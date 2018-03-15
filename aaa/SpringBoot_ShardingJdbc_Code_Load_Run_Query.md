[前略](https://saaavsaaa.github.io/aaa/SpringBoot_ShardingJdbc_Code_Load_Run_Insert.html) 

select语句（简单版）

org/apache/ibatis/binding/MapperMethod.execute [case SELECT]

因为是简单版，所以依然是CachingExecutor == SimpleExecutor

org/apache/ibatis/executor/BaseExecutor.query -> queryFromDatabase -> doQuery
--> org/apache/ibatis/executor/statement/PreparedStatementHandler.query [ps.execute();]

--> io/shardingjdbc/core/jdbc/core/statement/ShardingPreparedStatement.execute -> route     

--> io/shardingjdbc/core/routing/PreparedStatementRoutingEngine.route [参数就是sql的参数 == final List<Object> parameters]
除了类型是select，前面这大部分和前略的insert都是类似的

