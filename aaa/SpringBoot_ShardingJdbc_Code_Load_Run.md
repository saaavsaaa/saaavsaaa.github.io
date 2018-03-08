Insert 只分表：
org/springframework/jdbc/datasource/DataSourceTransactionManager.doBegin --> this.dataSource.getConnection()
--> io/shardingjdbc/core/jdbc/core/datasource/ShardingDataSource.getConnection --> new ShardingConnection(shardingContext)

org/apache/ibatis/binding/MapperMethod.execute case INSERT --> org/mybatis/spring/SqlSessionTemplate.insert -> SqlSessionInterceptor.invoke - invoke --> sun/reflect/NativeMethodAccessorImpl.invoke0
--> org/apache/ibatis/executor/CachingExecutor.update
















-----


[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/SpringBoot_ShardingJdbc_Code_Load_Run.md)
