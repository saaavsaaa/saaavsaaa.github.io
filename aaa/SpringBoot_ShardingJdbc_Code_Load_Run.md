/org/springframework/beans/factory/support/AbstractBeanFactory.java     
line 306 : beanName--> shardingDataSource     
           mbd --> Root bean: class [io.shardingjdbc.spring.datasource.SpringShardingDataSource]; scope=singleton; abstract=false;lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false;factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=close     
           args --> null  

org/springframework/beans/factory/support/AbstractAutowireCapableBeanFactory.java     
line 1095 : ctors --> [0] public io.shardingjdbc.spring.datasource.SpringShardingDataSource(java.util.Map,io.shardingjdbc.core.api.config.ShardingRuleConfiguration,java.util.Map,java.util.Properties) throws java.sql.SQLException     

line 271: this.beanFactory
          argsToUse --> [0][0] --> "www_0" -> "{CreateTime:"2018-03-06 16:45:49", ActiveCount:0, PoolingCount:0, CreateCount:0, DestroyCount:0, CloseCount:0, ConnectCount:0, Connections:[] }"
          [0][1] --> "www_1" -> "{CreateTime:"2018-03-06 16:45:49", ActiveCount:0, PoolingCount:0, CreateCount:0, DestroyCount:0, CloseCount:0, ConnectCount:0, Connections:[]}"
