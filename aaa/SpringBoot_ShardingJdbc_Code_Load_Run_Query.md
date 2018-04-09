跟着调式看代码的笔记，其中我觉得不重要的部分就直接用符号略过了，关于符号可参见
[前略](https://saaavsaaa.github.io/aaa/SpringBoot_ShardingJdbc_Code_Load_Run_Insert.html)      
我是mybatis+shardingjdbc+druid

select语句（简单版）

org/apache/ibatis/binding/MapperMethod.execute [case SELECT]

因为是简单版，所以依然是CachingExecutor == SimpleExecutor

org/apache/ibatis/executor/BaseExecutor.query -> queryFromDatabase -> doQuery
--> org/apache/ibatis/executor/statement/PreparedStatementHandler.query [ps.execute();]

--> io/shardingjdbc/core/jdbc/core/statement/ShardingPreparedStatement.execute -> route     

--> io/shardingjdbc/core/routing/PreparedStatementRoutingEngine.route [参数就是sql的参数 == final List\<Object> parameters]

-----

    public SQLRouteResult route(final List<Object> parameters) {
        if (null == sqlStatement) {
            sqlStatement = sqlRouter.parse(logicSQL, parameters.size());
        }
        return sqlRouter.route(logicSQL, parameters, sqlStatement);
    }

-----

除了类型是select，前面这大部分和前略的insert都是类似的     
[io/shardingjdbc/core/parsing/parser/sql/SQLParserFactory.newInstance return SelectParserFactory.newInstance(dbType, shardingRule, lexerEngine);]

--> io/shardingjdbc/core/routing/type/simple/SimpleRoutingEngine.routeDataSources      
与前略的insert不同会走到根据分表分库的策略选库上[shardingRule.getDatabaseShardingStrategy(tableRule).doSharding(availableTargetDatabases, databaseShardingValues);]     
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
对已有分片键值使用自定义的分片算法选出所有符合自定义算法的数据库     

<<< SimpleRoutingEngine.routeDataSources <<< route -> each : routeTables     
从所有数据源[库.表]中选出刚刚策略计算出的库中对应的所有表。当分表键存在时，对这些表使用自定义分表策略（与上面分库逻辑基本一样），键不存在直接使用刚选出的库中所有匹配的表。再用这些库和表创建DataNode。

简单版的select后面没说的就都参见上面的前略吧，下面看个复杂的。

-----

上面简单版解析sql的部分都跳过了，这里看一下(以下所有解析出来的全都放在一个sqlStatement对象里了)：
io/shardingjdbc/core/routing/PreparedStatementRoutingEngine.route     
--> io/shardingjdbc/core/routing/router/ParsingSQLRouter.parse     
--> SQLParsingEngine.parse [SQLParserFactory.newInstance(dbType, lexerEngine.getCurrentToken().getType(), shardingRule, lexerEngine) == SQLParser == MySQLSelectParser] 

--> io.shardingjdbc.core.parsing.parser.sql.dql.select.AbstractSelectParser.parse -> parseInternal      
--> io/shardingjdbc/core/parsing/parser/dialect/mysql/sql/MySQLSelectParser.parseInternal     

-> parseDistinct不支持Distinct     
-> parseSelectOption [io/shardingjdbc/core/parsing/parser/dialect/mysql/clause/MySQLSelectOptionClauseParser.parse]     
-> parseSelectList [io/shardingjdbc/core/parsing/parser/dialect/mysql/sql/MySQLSelectParser.parse -> parseSelectItem:分别处理不同类型的列，如行号、\*、聚合函数（Count等,要处理括号，以及计算如count()+等数值计算）和正常情况(parseCommonSelectItem拼不同列的列名或别名)]     

-> parseFrom [
不支持INTO,parseTable(如果跟着左括号,处理子查询) --> io/shardingjdbc/core/parsing/parser/clause/TableReferencesClauseParser.parse --> MySQLTableReferencesClauseParser.parseTableReference和前略里的处理差不多，不同的就是这里有别名AS和连表，连表还要处理ON和USING的连接条件]

-> parseWhere [io/shardingjdbc/core/parsing/parser/clause/WhereClauseParser.parse -> parseConditions -> parseConditions 条件是分左右两个SQLExpression对象存在condition里的，这里判断了OR目前是不支持的]     
-> parseGroupBy [io/shardingjdbc/core/parsing/parser/clause/GroupByClauseParser.parse]     
-> parseHaving [不支持having]     
-> parseOrderBy [io/shardingjdbc/core/parsing/parser/clause/OrderByClauseParser.parse 默认排序是ASC]     
-> parseLimit [io/shardingjdbc/core/parsing/parser/dialect/mysql/clause/MySQLLimitClauseParser.parse]     
-> parseSelectRest [io/shardingjdbc/core/parsing/parser/clause/SelectRestClauseParser.parse 一些不支持的关键字:UNION,INTERSECT,EXCEPT,MINUS,PROCEDURE,INTO]     

<<< AbstractSelectParser.parse --> io/shardingjdbc/core/parsing/parser/sql/dql/select/SelectStatement [containsSubQuery, 判断是否包含子查询，如果包含，执行mergeSubQueryStatement]：

-----

    public SelectStatement mergeSubQueryStatement() {
        SelectStatement result = processLimitForSubQuery();
        processItems(result);
        processOrderByItems(result);
        return result;
    }

-----

-> appendDerivedColumns [和appendDerivedOrderBy有todo注释(move to rewrite)，处理聚合函数，如遇到avg会增加count和sum函数列到avg的derived项中，并以AVG_DERIVED_COUNT/SUM\_+ derived项序号等形式做别名]  

<<< -> appendDerivedOrderColumns [根据参数不同分别为order和group项加别名]     

<<< io/shardingjdbc/core/routing/PreparedStatementRoutingEngine.route     
--> io/shardingjdbc/core/routing/router/ParsingSQLRouter.route [取出逻辑表名，构造ComplexRoutingEngine路由引擎对象]       

--> io/shardingjdbc/core/routing/type/complex/ComplexRoutingEngine.route      
[在ShardingRule中找对应逻辑表的规则配置，并对单个逻辑表构建SimpleRoutingEngine对象，执行route方法（此处参见上面那个简单的SQL）]   

<<< ComplexRoutingEngine.route [return new CartesianRoutingEngine(result).route();CartesianRoutingEngine的route -> getDataSourceLogicTablesMap返回数据源交集共有的逻辑表 -> getIntersectionDataSources这个方法对数据源取交集返回，取交集就是说同一个CartesianTableReference中的tableUnits都应该在同一个数据库，ComplexRoutingEngine.route所有库的逻辑表对应真实库和表对应关系List\<CartesianTableReference>放到CartesianRoutingResult对象实例中]     

--> io/shardingjdbc/core/routing/type/complex/CartesianRoutingEngine.route [用join的表构造笛卡尔组合的TableUnits的Set，放在RoutingResult的routingTableReferences中返回]   

<<< io/shardingjdbc/core/routing/router/ParsingSQLRouter.route     
-> processLimit     
<<< ParsingSQLRouter.route --> io/shardingjdbc/core/rewrite/SQLRewriteEngine.rewrite [用sqlToken构建SQLBuilder对象实例]   

<<< ParsingSQLRouter.route [循环数据源创建SQLExecutionUnit生成完整SQL,routingResult instanceof CartesianRoutingResult的情况，else见前略]:

-----

            for (CartesianDataSource cartesianDataSource : ((CartesianRoutingResult) routingResult).getRoutingDataSources()) {
                for (CartesianTableReference cartesianTableReference : cartesianDataSource.getRoutingTableReferences()) {
                    result.getExecutionUnits().add(new SQLExecutionUnit(cartesianDataSource.getDataSource(), rewriteEngine.generateSQL(cartesianTableReference, sqlBuilder)));
                }
            }

-----

<<< io/shardingjdbc/core/jdbc/core/statement/ShardingPreparedStatement.execute [参见前略]     
<<< io/shardingjdbc/core/jdbc/core/statement/ShardingPreparedStatement.query [return resultSetHandler.\<E> handleResultSets(ps);]     
--> org/apache/ibatis/executor/resultset/DefaultResultSetHandler.handleResultSets -> getFirstResultSet [stmt.getResultSet()]     
--> <<< io/shardingjdbc/core/jdbc/core/statement/ShardingPreparedStatement.getResultSet [多结果集拼到一起]

-----

        for (PreparedStatement each : routedStatements) {
            resultSets.add(each.getResultSet());
        }
        if (routeResult.getSqlStatement() instanceof SelectStatement) {
            currentResultSet = new ShardingResultSet(resultSets, new MergeEngine(resultSets, (SelectStatement) routeResult.getSqlStatement()).merge(), this);
        } else {
            currentResultSet = resultSets.get(0);
        }

-----

--> io/shardingjdbc/core/merger/MergeEngine.merge     
-> build [合并结果集 GroupByStreamResultSetMerger > OrderByStreamResultSetMerger > IteratorStreamResultSetMerger这里顺序会有影响，前面的resultSetMerger会传给后面，在最后mybatis拼装结果集时会被类似链式的方式调用]     
-> decorate [处理limit, LimitDecoratorResultSetMerger]     

<<< ShardingPreparedStatement.getResultSet     
<<< DefaultResultSetHandler.getFirstResultSet [返回ShardingResultSet，包装成new ResultSetWrapper:rsw构建结果的元数据，列什么的]     
<< DefaultResultSetHandler.handleResultSets     
-> handleResultSet -> handleRowValues      
-> handleRowValuesForSimpleResultMap [此处DefaultResultSetHandler的Limit没什么用是最开始DefaultSqlSession处的RowBounds.DEFAULT，循环的判断条件中rsw.getResultSet()是返回ShardingResultSet，我的这个例子next方法:return mergeResultSet.next() --> LimitDecoratorResultSetMerger.next 此处控制从多数据源返回的结果集合并后只返回前Limit行记录，这里Limit还会调用前面OrderByStreamResultSetMerger的next]
each :     
-> getRowValue -> applyAutomaticMappings [向元数据对象里填查询回来的结果值]     
-> storeObject -> callResultHandler [结果放进泛型对象实例]      
后面似乎也没什么关系了。

备注：
sharding-jdbc 配置tag :io/shardingjdbc/spring/namespace/constants/ShardingStrategyBeanDefinitionParserTag.class(META-INF/spring.handlers)


-----

增加sharding-jdbc对MySQL强制索引Force INDEX语法的支持     
看过了前面的代码之后，偶然发现一个比较简单的ISSUE(https://github.com/shardingjdbc/sharding-jdbc/issues/671)，于是决定试着改一下，同时熟悉sharding-jdbc     
这个功能主要就是在表名之后增加对Force INDEX的支持，最开始我就判断如果表名（代码在TableReferencesClauseParser.parseTableFactor）之后跟着MySQLKeyword.FORCE(我查了下，这个语法只有MySQL支持)和DefaultKeyword.INDEX，就直接对之后括号及括号内跳过，这样这个语法就可以正常执行了。不过看了张亮（sharding-jdbc开源社区的管理员）的代码建议才知道sharding-jdbc本身有创建索引的功能，而兼容pg这种全库索引名唯一，于是sj自己创建的索引是使用了逻辑索引的，像表名一样，对分表会进行真实索引与逻辑索引的映射，比如配置了logicIndex: order_index，如果语句中包含order_index，就会把它映射成"order_index_真实表名"。     
于是就将原来直接跳过改为，通过分片规则判断括号里的索引是否是逻辑索引，如果是逻辑索引，就创建一个它的IndexToken实例并加入sqlStatement的SqlTokens中，这样最后在创建执行单元的时候就会将它映射为真实的索引，代码如下：

-----

        boolean skipIfForce = lexerEngine.skipIfEqual(MySQLKeyword.FORCE) && this.lexerEngine.skipIfEqual(DefaultKeyword.INDEX);
        if (skipIfForce) {
            lexerEngine.accept(Symbol.LEFT_PAREN);
            do {
                String literals = lexerEngine.getCurrentToken().getLiterals();
                if (shardingRule.isLogicIndex(literals, tableName)) {
                    int beginPosition = lexerEngine.getCurrentToken().getEndPosition() - literals.length();
                    sqlStatement.getSqlTokens().add(new IndexToken(beginPosition, literals, tableName));
                }
                lexerEngine.nextToken();
            } while (lexerEngine.skipIfEqual(Symbol.COMMA));
            lexerEngine.accept(Symbol.RIGHT_PAREN);
        }

-----

还有通过分片规则判断是否为逻辑索引，代码在ShardingRule中，原本我是分几行写的，最后根据代码建议合为一行了：

-----

    /**
     * Adjust is logic index or not.
     *
     * @param logicIndexName logic index name
     * @param logicTableName logic table name
     * @return is logic index or not
     */
    public boolean isLogicIndex(final String logicIndexName, final String logicTableName) {
        return logicIndexName.equals(getTableRule(logicTableName).getLogicIndex());
    }

-----

基本功能就是这些了，之后就是加单元测试和集成测试了，sharding-jdbc中封装好了对sql的测试功能，所以这部分我这个简单的代码就需要配置就可以了

-----

    parser-rule.yaml中加入logicIndex: order_index

    <sql-case id="assertSelectWithJoinForceIndex" value="SELECT o.*,i.* FROM t_order o FORCE INDEX(order_index) JOIN t_order_item i WHERE o.order_id = %s" db-types="MySQL" />

    <parser-result sql-case-id="assertSelectWithJoinForceIndex" parameters="1000">
        <tables>
            <table name="t_order" alias="o" />
            <table name="t_order_item" alias="i"/>
        </tables>
        <tokens>
            <table-token begin-position="20" original-literals="t_order" />
            <table-token begin-position="60" original-literals="t_order_item" />
            <index-token begin-position="42" original-literals="order_index" table-name="t_order" />
        </tokens>
        <conditions>
            <condition column-name="order_id" table-name="t_order" operator="EQUAL">
                <value index="0" literal="1000" type="int" />
            </condition>
        </conditions>
    </parser-result>
    
    集成测试：
    <sql id="assertSelectWithJoinForceIndex">
        <sharding-rule value="tbl">
            <data parameter="1000" expected="select/SelectWithJoinForceIndex.xml" />
        </sharding-rule>
    </sql>

-----

    提交的pr地址（其中有沟通和根据建议修改的过程）:https://github.com/shardingjdbc/sharding-jdbc/pull/683

-----

[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/SpringBoot_ShardingJdbc_Code_Load_Run_Query.md)

