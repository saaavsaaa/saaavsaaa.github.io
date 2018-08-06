  前略：[关于SS注册中心ZK客户端的封装](https://saaavsaaa.github.io/aaa/S_S_ZK_Registry_Center.html)
  在做前略的事时，最初因为[assertWatch这个单元测试](https://github.com/sharding-sphere/sharding-sphere/blob/dev/sharding-jdbc-orchestration/src/test/java/io/shardingsphere/jdbc/orchestration/reg/newzk/client/zookeeper/UsualClientTest.java)对watcher产生了误解，当然也是之前对zk的监听事件了解不够。
  因为这个单元测试跑过了，所以我误以为可以再启动时注册的watcher里获取所有的事件。当然，确实是可以的，只不过并没有解决一个监听器只会触发一次的问题。只是因为这个单元测试中最开头的exist和没执行一次更新就assert一下数据是否正确的getData导致每次都重新注册了一个监听。可以归类为对节点的查询都会注册响应的时间监听。而增删改就不一样了，删除是每次都会触发事件，无论是否监听；创建只会对监听的节点触发创建事件；修改事件会对监听的节点和其子节点触发对应NodeDataChanged和NodeChildrenChanged。
  
