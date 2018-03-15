[前略](https://saaavsaaa.github.io/aaa/SpringBoot_ShardingJdbc_Code_Load_Run_Insert.html) 

select语句（简单版）

org/apache/ibatis/binding/MapperMethod.execute [case SELECT]

因为是简单版，所以依然是CachingExecutor == SimpleExecutor

org/apache/ibatis/executor/BaseExecutor.query -> doQuery
