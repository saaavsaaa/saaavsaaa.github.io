  最近的业余时间多用于做这件事：https://github.com/sharding-sphere/sharding-sphere/issues/717 。     
  
  由于是以完成现有功能未目标的，同时业余时间也不太充足，所以其中有些用不到地方处理的有些粗糙，另外还有原本打算用上，但最后没用上的，还有干脆就没做的部分，比如zk的一些一步回调执行的接口。我打算在这里把开发的过程，思路介绍一下，大家如果有兴趣，大家一起来把它做好。   
  
### 首先来个草图，大致看一下组成结构：

-----

                                    ClientFactory
               作用是为了将Client的创建逻辑封装起来，原本也是有同样参数创建多个客户端的逻辑，不过重构掉不用了
                                          |
                                          |
                                          |
              ------------------------------------------------------------
              |                                                           |
              |                                                           |
              |                                                           |
         CacheClient                                                 UsualClient
    原本是用于把缓存的使用细节封装起来的            现在用的客户端，功能包括初始化执行策略，选择执行策略，在调用过程中可以切换策略
    原本还想把它做成一个执行策略                   主要执行逻辑是调用执行策略接口执行
    不过暂时用不上，所以是个半成品                 此外创建和删除时会判断是否根节点（namespace），如果是会更新BaseClient的rootExist
                                                                           |
                                                                           |
                                                                           |
               ---------------------------------------------------------------------------------------------------------
               |                             |                                    |                                    |
               |                             |                                    |                                    |
               |                             |                                    |                                    |
          UsualStrategy             SyncRetryStrategy                       AsyncRetryStratety                   ContentStrategy
     最基本zk功能的封装或直接         目前主要使用的策略           原本想用的重试策略，后来发现原有功能是同步重试   TransactionContentStrategy
     使用最基础接口的直接实现      用Callable包装了执行来实现重试     把执行包装为BaseOperation的子类对象           Content是指竞争节点的写权限
               |                 这里的有一点不顺的地方后面会说      发送到DelayQueue，开线程做异步重试            这实现比较糙，后面细说
               |                             |                                    |                                    |
               |                             |                                    |                                    |
               ---------------------------------------------------------------------------------------------------------
                                                              |
                                                              |
                                                              |
                                                           BaseProvider
                               包装了基本的zk操作，现在正准备加一个TransactionProvider把事务部分独立

-----                                                                                    

### 包结构：
    action 主要的接口，为了找代码方便就放一处了
    cache 主要用于缓存zk节点树数据
    election 节点竞争，未来选举的功能准备放这
    retry 重试功能，之后把Callable移到这个包里，同步和异步重试的功能就基本都在这了
    utility 工具
    zookeeper zk客户端
      base 各种基类和基础组件Holder
      operation 用于异步重试的操作对象
      section 各种没想好怎么归类的组件
      strategy 执行策略的各种实现
      transaction 事务封装
    
### 下面是开发细节:
  UsualClient的基类BaseClient中有一个必要重要的类型Holder，因为没有顺眼的地方现在放在base包里。最开始是没有这个类型的，zk的连接是由BaseClient维护的，但是在做重试功能的时候觉得怎么实现怎么别扭，因为重试功能按说是Client内部实现的，但是如果连接由Client维护，那重试策略势必要操作Client，这引用关系就相当凌乱了。而且由于先做的异步重试（其实同步重试一样），在重试过程中会涉及到连接和监听，如果传递Client，重试的Operation立场就十分尴尬了。于是，根据书上有别扭就必然是因为有隐含的概念没有抽取出来的理论来寻找一个合适解耦的中间层抽取出来，那就比较明显了，把zk连接的维护独立出来，Client中只负责对节点的操作，由Holder来维护节点连接。同时在Client中创建Provider实例的时候可以把Holder实例传递给Provider，这样重试策略Provider就完全hold得住了。
  
  Holder里还有一个比较重要的部分是Watcher。一般情况下zk原生的Watcher功能只能使用一次，不过有一个地方除外，就是new的时候通过zk构造传进去的那个Watcher，由于是用于监听连接状态的，所以一直存在。于是我就通过Client提供一个registerWatch方法，把注册的监听放到一个Map里，有新的event的时候会遍历Map执行监听，此处有一个需要晚上的地方，就是遍历Map时候的判断，现在只是简单的判断了路径为null或者匹配，并没有根据对应的事件类型过滤监听器，这个后续一定会有过滤的需要。

  另外Client里会看到有Context的使用，但是其实这玩意是做重试功能时候，怎么试怎么不顺眼，就拿来传参数用了，其实抽出Holder以后我本来是有想法把它去掉的，不过看着也不太讨厌就留下了，如果能优雅的去掉，我也是欢迎的。
  
  BaseClient主要功能有start、close、注册、认证和创建删除根节点。start这有一个blockUntilConnected方法，主要功能是如果在一定时间内启动不成功就返回启动失败。这个方法是在curator同名方法上稍微简化了一下，我比较不喜欢这种实现方式，holder里实现的start是用CountDownLatch阻塞的，CountDownLatch的await本身就有时间参数的重载，我现在在准备用await的重载方法替代blockUntilConnected，功能是没问题了，只是两个单元测试出现了互相影响的现象，还没调好。创建和删除根节点是为了实现namespace的功能的，使用namespace做根节点，不同的namespace下的节点就不会互相影响了。
  
  现在注册中心的Client使用的执行策略是同步重试SyncRetryStrategy，是用Callable实现的。Callable会从section包移到retry包里去的。主要逻辑就是根据延迟策略创建延迟策略的执行者，每次执行失败先判断失败的异常是否是需要重新连接zk server的，如果是resetConnection后延迟重试，否则直接延迟重试期待延迟过程中zk自动重连，之后由执行者判断是否还有下一次执行。
  
  异步重试策略AsyncRetryStrategy是由AsyncRetryCenter执行的，在该策略的api执行时，如果捕捉到需要reset
  

  
         
         
zk事务,分成两个是因为想事务和非事务的拆开，现有没拆开的,之后也想拆开，因为没事务的部分是可以支持全版本的zk
ICallbackProvider
LeaderElection
后续要做的 todo
