
  去年，在接银行存管的时候重新设计了架构，使用了dubbo，因为原来的项目中有一些需要强一致性事务的功能，以及一些诸如量不特别大所以想尽量简单、不想引入外部依赖之类的原因，所以就自己实现了个简单的分布式事务。如今又由于换了存管的银行，和一些其他不方便拿出来丢人的原因，总之架构又重新做了，换了spring cloud，数据库中间件用了sharding-jdbc，所以我写的这玩意也没啥用了，于是打算写篇博客记录一下，由于代码毕竟是公司项目里的，就不方便全部公布出来了，反正很简单，有需要的同学可以自己实现一下。   

  首先，实现这个功能是基于一个假设，当所有业务代码都正常执行了，那事务就可以正常提交，所以如果业务代码正常执行完成，这时候数据库连接超时了，那造成的错误，我并没有避免。所以说实现的比较简单，量特别大的业务也不是很合适，其实挺鸡肋的，专门适应我们这种量不大，但是架子不小的项目。   

  实现的思路很简单，对于需要多个节点协作的业务，执行过程中的节点在业务执行完成，该节点事务提交前，将事务拦截下来，等到所有节点业务代码都执行完，统一提交所有代码。   

  拦截部分的代码也很简单，创建一个类继承DataSourceTransactionManager，使用的时候使用新创建的类就好了。我重写了其中newTransactionStatus、doBegin、doCommit、doCleanupAfterCompletion方法和JdbcTransactionObjectSupport内部类。   

  newTransactionStatus方法的重写用于保存事务的状态:   

-----

    @Override
    protected DefaultTransactionStatus newTransactionStatus(TransactionDefinition definition, Object transaction, boolean newTransaction,
                                                            boolean newSynchronization, boolean debug, Object suspendedResources) {
        DefaultTransactionStatus status = super.newTransactionStatus(definition, transaction, newTransaction, newSynchronization, debug, suspendedResources);
        waitStatus.put(IdCache.INSTANCE.getKey(), status);
        return status;
    }

-----

  waitStatus是一个ConcurrentHashMap。IdCache.INSTANCE.getKey()是一个全局唯一Id，在业务服务被调用时生成，服务间互相调用时，随着各子服务的调用传递。接到这个Id的服务将此Id使用ThreadLocal保存，以便在本地事务执行过程中使用，这个Id是最后业务统一提交时用于查找待提交的事务的唯一标识。在dubbo中，可以借助Filter来接受此Id：

-----

    @Activate(group = Constants.PROVIDER)
    @Component
    public class ReceiveIdFilter implements Filter {
        @Override
        public Result invoke(Invoker<?> invoker, Invocation invocation) {
            String path = invoker.getInterface().getName() + "." + invocation.getMethodName();

            Result result = null;
            try {
                if (invocation.getAttachment("transactionId") == null) {
                    ......
                } else {
                    IdCache.INSTANCE.set(invocation.getAttachment("transactionId"));
                }
                
                ......

                result = invoker.invoke(invocation);
            }
            catch (RpcException e){
                throw e;
            }

            return result;
        }
    }

-----

  doBegin方法，判断是否是新的分布式事务，如果是标识事务是自己启动的，同时设置事务状态为Start；如果不是，说明存在了本地服务在整个分布式事务中存在了多次被远程或本地服务调用的情况，虽然这种情况几乎没有，不过也可以处理一下，比如说同一个Key保存多个事务状态，可以新建一个事务状态类继承DefaultTransactionStatus并增加一个对下一个事务状态对象的引用，形成链表，不过这里对我基本没用，所以一直放着没管。     

  doCommit方法：

-----

    @Override
    protected void doCommit(DefaultTransactionStatus status) {
        if (GlobalTransaction.INSTANCE.owned()){

            try {
                DistributeService.INSTANCE.exec();
            } catch (Exception e) {
                throw new TransactionSystemException(e.getMessage());
            }
    
            super.doCommit(status);
        } else {
            ConnectionHolder conHolder = (ConnectionHolder) TransactionSynchronizationManager.getResource(this.getDataSource());
            this.setWaitDistributedObject(conHolder);
        }
    }

-----

  提交方法首先判断是不是本地启动的分布式事务，这个标志使用ThreadLocal保存以避免本地服务被多次调用的情况。如果不是当前服务启动的分布式事务，则将数据库的连接和事务对象保存在内存中；如果是当前服务启动的，则说明所有的业务代码已经执行完成，通知所有服务可以进行提交了。所有被通知的服务都在之前调用时将其引用保存下来，在这时候调用，我因为方便，保存引用也是在dubbo的Filter中做的：

-----

    @Activate(group = Constants.CONSUMER)
    @Component
    public class SendIdFilter implements Filter {
        @Override
        public Result invoke(Invoker<?> invoker, Invocation invocation) {

            long start = System.currentTimeMillis();
            Result result = null;
            try {
                if (StringUtils.isEmpty(IdCache.INSTANCE.get())){
                    if (invocation.getAttachment("transactionId") == null){
                        String newId = IdGenerator.INSTANCE.createNewId();
                        ((RpcInvocation) invocation).setAttachment("transactionId", newId);
                    }
                } else {
                    ((RpcInvocation) invocation).setAttachment("transactionId", IdCache.INSTANCE.get());
                }

                if (StringUtils.isNotEmpty(GlobalTransaction.INSTANCE.getOwnId())){
                    if (GlobalTransaction.INSTANCE.owned()){
                        Map services = ApplicationContextUtil.getContext().getBeansOfType(invoker.getInterface());
                        if (services != null && !services.isEmpty()){
                            services.forEach(
                                    (k,v) -> {
                                        if (v instanceof BaseService){
                                            DistributeService.INSTANCE.add((BaseService) v);
                                        }
                                    }
                            );
                        }
                    }
                }


                result = invoker.invoke(invocation);

                // TODO: 17-8-14 DistributeService 的远程调用也会进拦截器，导致执行多次
                if (result.hasException()){
                    DistributeService.INSTANCE.adviseRollback();
                }
            }
            catch (Exception e){

                throw e;
            }

            return result;
        }
    }

-----

  生成全局事务Id的方法为了方便也放这里了，不过其实在事务启动的时候生成更好，生成全局Id的方法[大概是这样](https://github.com/saaavsaaa/warn-report/blob/master/src/main/java/util/IdGenerator.java)。     
  提交的逻辑很简单，就是用key取出事务状态和对象，直接调用jdbc的提交，之后再清理一下所有与这个key相关的数据还有ThreadLocal等，回滚也是类似的。exec的代码：

-----

    public void exec() throws Exception {
        List<BaseService> services = waitServiceHolder.get();
        if (services != null){
            try {
                for (BaseService service : services) {
                    service.adviseCommit();
                }
            } catch (Throwable e){
                System.out.println("rollback : " + e.getMessage());
                adviseRollback();
                throw e;
            }
        }
    }

-----

  doCleanupAfterCompletion方法没啥好说的：

-----

    @Override
    protected void doCleanupAfterCompletion(Object transaction) {
        if (GlobalTransaction.INSTANCE.owned()){
            super.doCleanupAfterCompletion(transaction);
            IdCache.INSTANCE.clear();
        } else {
            this.clearThreadLocal();
        }
    
        DistributeService.INSTANCE.clear();
        GlobalTransaction.INSTANCE.stop();
        
        System.out.println();
    }
    
    private void clearThreadLocal(){
        DistributedTransactionObject distributedTransactionObject = waitDistributedObject.get(IdCache.INSTANCE.getKey());
        if (distributedTransactionObject.isNewConnectionHolder()) {
            TransactionSynchronizationManager.unbindResource(this.getDataSource());
        }
    }

-----

    大概就是这样了，其实原本还打算用redis做服务间的同步控制的，但是做到一半就感觉，本来就是一次网络请求变成两次，就不要再增加网络IO了。另外，对数据库的连接也是，将每一个数据库连接的占用时间都延长了。感觉只有比较畸形的项目，比如在没多大量就还是赶潮流，追微服务什么的，这种情况下才用的上。两段式也是因为类似的原因不得人心吧。

-----

微信公众号：

![Image](/ppp/20170902204445.jpg)

[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Own_Distribute_Transaction.md)


```markdown
```
