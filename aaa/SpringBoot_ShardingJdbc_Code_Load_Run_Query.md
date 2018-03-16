[前略](https://saaavsaaa.github.io/aaa/SpringBoot_ShardingJdbc_Code_Load_Run_Insert.html) 

select语句（简单版）

org/apache/ibatis/binding/MapperMethod.execute [case SELECT]

因为是简单版，所以依然是CachingExecutor == SimpleExecutor

org/apache/ibatis/executor/BaseExecutor.query -> queryFromDatabase -> doQuery
--> org/apache/ibatis/executor/statement/PreparedStatementHandler.query [ps.execute();]

--> io/shardingjdbc/core/jdbc/core/statement/ShardingPreparedStatement.execute -> route     

--> io/shardingjdbc/core/routing/PreparedStatementRoutingEngine.route [参数就是sql的参数 == final List\<Object> parameters]
除了类型是select，前面这大部分和前略的insert都是类似的

[io/shardingjdbc/core/parsing/parser/sql/SQLParserFactory.newInstance return SelectParserFactory.newInstance(dbType, shardingRule, lexerEngine);]

--> io/shardingjdbc/core/routing/type/simple/SimpleRoutingEngine.routeDataSources      
与insert不同会走到根据分表分库的策略选库上[shardingRule.getDatabaseShardingStrategy(tableRule).doSharding(availableTargetDatabases, databaseShardingValues);]     
--> io/shardingjdbc/core/routing/strategy/standard/StandardShardingStrategy.doSharding
[参数中有所有待选的数据库:final Collection\<String> availableTargetNames，另外还有逻辑表名（例如：配置的没有后缀数字的表名）和列以及参数:List\<PreciseShardingValue>]     

-----

    @Override
    public Collection<String> doSharding(final Collection<String> availableTargetNames, final Collection<ShardingValue> shardingValues) {
        ShardingValue shardingValue = shardingValues.iterator().next();
        Collection<String> shardingResult = shardingValue instanceof ListShardingValue
                ? doSharding(availableTargetNames, (ListShardingValue) shardingValue) : doSharding(availableTargetNames, (RangeShardingValue) shardingValue);
        Collection<String> result = new TreeSet<>(String.CASE_INSENSITIVE_ORDER);
        result.addAll(shardingResult);
        return result;
    }

-----

shardingValue instanceof ListShardingValue判断（如果是就循环，不是就直接走分库逻辑）后会走进自定义的分库策略逻辑，例如：
<sharding:standard-strategy id="databaseStrategy" sharding-column=" \*** " precise-algorithm-class="\*\*\*" \> 中的配置。    
根据分库策略返回一个或多个库     
<<< SimpleRoutingEngine.routeDataSources <<< route -> each : routeTables     
从所有数据源[库.表]中选出刚刚策略计算出的库中对应的所有表。当分表键存在时，对这些表使用自定义分表策略（与上面分库逻辑基本一样），键不存在直接使用刚选出的库中所有匹配的表。再用这些库和表创建数据节点。
简单版的select后面就都参见上面的前略吧，下面看个复杂的。

-----

 上面简单版解析sql的部分都跳过了，这里看一下：
 io/shardingjdbc/core/routing/PreparedStatementRoutingEngine.route     
 --> io/shardingjdbc/core/routing/router/ParsingSQLRouter.parse     
 --> SQLParsingEngine.parse [SQLParserFactory.newInstance(dbType, lexerEngine.getCurrentToken().getType(), shardingRule, lexerEngine) == SQLParser == MySQLSelectParser]     
 --> io.shardingjdbc.core.parsing.parser.sql.dql.select.AbstractSelectParser.parse -> parseInternal      
 --> io/shardingjdbc/core/parsing/parser/dialect/mysql/sql/MySQLSelectParser.parseInternal     

-----


[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/SpringBoot_ShardingJdbc_Code_Load_Run_Query.md)

