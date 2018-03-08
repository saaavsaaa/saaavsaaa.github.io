-->  对象或类间调用     
->  对象或类内调用     
[]  方法本身或方法体补充说明     
==  类型相同     
=== 值相等     
<<< 回到方法的被调用链某处继续向下执行

Insert 只分表：
org/springframework/jdbc/datasource/DataSourceTransactionManager.doBegin --> this.dataSource.getConnection()     
--> io/shardingjdbc/core/jdbc/core/datasource/ShardingDataSource.getConnection --> new ShardingConnection(shardingContext)     

org/apache/ibatis/binding/MapperMethod.execute case INSERT --> org/mybatis/spring/SqlSessionTemplate.insert ->     SqlSessionInterceptor.invoke - invoke --> sun/reflect/NativeMethodAccessorImpl.invoke0     
--> org/apache/ibatis/executor/CachingExecutor.update [delegate==SimpleExecutor]     
--> org/apache/ibatis/executor/BaseExecutor.update(MappedStatement ms, Object parameter)     
--> org/apache/ibatis/executor/SimpleExecutor.[abstract]doUpdate(ms, parameter)
->prepareStatement(StatementHandler handler, Log statementLog) :     
        Connection connection = getConnection(statementLog);     
        stmt = handler.prepare(connection, transaction.getTimeout());     
        这个connection根据开头可知是ShardingConnection类型     
--> org/apache/ibatis/executor/statement/RoutingStatementHandler.prepare (delegate==PreparedStatementHandler)     
--> org/apache/ibatis/executor/statement/BaseStatementHandler.prepare->[abstract]instantiateStatement     
--> org/apache/ibatis/executor/statement/PreparedStatementHandler.instantiateStatement     
--> io/shardingjdbc/core/jdbc/core/connection/ShardingConnection.prepareStatement(final String sql):     

-----

    @Override     
    public PreparedStatement prepareStatement(final String sql) throws SQLException {     
        return new ShardingPreparedStatement(this, sql);     
    }  
    
    public ShardingPreparedStatement(final ShardingConnection connection, final String sql) {     
        this(connection, sql, ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY, ResultSet.HOLD_CURSORS_OVER_COMMIT);     
    }     
    
    public ShardingPreparedStatement(final ShardingConnection connection, final String sql, final int resultSetType, final int resultSetConcurrency, final int resultSetHoldability) {
        this.connection = connection;
        this.resultSetType = resultSetType;
        this.resultSetConcurrency = resultSetConcurrency;
        this.resultSetHoldability = resultSetHoldability;
        routingEngine = new PreparedStatementRoutingEngine(sql, connection.getShardingContext());
    }

-----

    public ShardingPreparedStatement(final ShardingConnection connection, final String sql, final int resultSetType, final int resultSetConcurrency, final int resultSetHoldability) {
        this.connection = connection;
        this.resultSetType = resultSetType;
        this.resultSetConcurrency = resultSetConcurrency;
        this.resultSetHoldability = resultSetHoldability;
        routingEngine = new PreparedStatementRoutingEngine(sql, connection.getShardingContext());
    }

    public PreparedStatementRoutingEngine(final String logicSQL, final ShardingContext shardingContext) {
        this.logicSQL = logicSQL;
        sqlRouter = SQLRouterFactory.createSQLRouter(shardingContext);
    }
    
    public static SQLRouter createSQLRouter(final ShardingContext shardingContext) {
        return HintManagerHolder.isDatabaseShardingOnly() ? new DatabaseHintSQLRouter(shardingContext) : new ParsingSQLRouter(shardingContext);
    }
    
    public ParsingSQLRouter(final ShardingContext shardingContext) {
        shardingRule = shardingContext.getShardingRule();
        databaseType = shardingContext.getDatabaseType();
        showSQL = shardingContext.isShowSQL();
        generatedKeys = new LinkedList<>();
    }

-----
sql和路由规则都有了       
回到 <<< org/apache/ibatis/executor/statement/BaseStatementHandler.prepare->setFetchSize(statement)          
<<< org/apache/ibatis/executor/SimpleExecutor.prepareStatement:handler.parameterize(stmt : stmt==ShardingPreparedStatement)     
--> org/apache/ibatis/executor/statement/RoutingStatementHandler.parameterize     
--> org/apache/ibatis/executor/statement/PreparedStatementHandler.parameterize     
<<< org/apache/ibatis/executor/SimpleExecutor.doUpdate : return handler.update(stmt);     
--> org/apache/ibatis/executor/statement/PreparedStatementHandler.update(statement==ShardingPreparedStatement) :

-----

        @Override
        public int update(Statement statement) throws SQLException {
                PreparedStatement ps = (PreparedStatement) statement;
                ps.execute();
                int rows = ps.getUpdateCount();
                Object parameterObject = boundSql.getParameterObject();
                KeyGenerator keyGenerator = mappedStatement.getKeyGenerator();
                keyGenerator.processAfter(executor, mappedStatement, ps, parameterObject);
                return rows;
        }

-----

ps.execute == io/shardingjdbc/core/jdbc/core/statement/ShardingPreparedStatement.execute:

-----

    @Override
    public boolean execute() throws SQLException {
        try {
            Collection<PreparedStatementUnit> preparedStatementUnits = route();
            return new PreparedStatementExecutor(
                    getConnection().getShardingContext().getExecutorEngine(), routeResult.getSqlStatement().getType(), preparedStatementUnits, getParameters()).execute();
        } finally {
            clearBatch();
        }
    }
    
    private Collection<PreparedStatementUnit> route() throws SQLException {
        Collection<PreparedStatementUnit> result = new LinkedList<>();
        routeResult = routingEngine.route(getParameters());
        for (SQLExecutionUnit each : routeResult.getExecutionUnits()) {
            SQLType sqlType = routeResult.getSqlStatement().getType();
            Collection<PreparedStatement> preparedStatements;
            if (SQLType.DDL == sqlType) {
                preparedStatements = generatePreparedStatementForDDL(each);
            } else {
                preparedStatements = Collections.singletonList(generatePreparedStatement(each));
            }
            routedStatements.addAll(preparedStatements);
            for (PreparedStatement preparedStatement : preparedStatements) {
                replaySetParameter(preparedStatement);
                result.add(new PreparedStatementUnit(each, preparedStatement));
            }
        }
        return result;
    }

-----

--> io/shardingjdbc/core/routing/PreparedStatementRoutingEngine.route:

-----

    public SQLRouteResult route(final List<Object> parameters) {
        if (null == sqlStatement) {
            sqlStatement = sqlRouter.parse(logicSQL, parameters.size());
        }
        return sqlRouter.route(logicSQL, parameters, sqlStatement);
    }

-----

io/shardingjdbc/core/routing/router/ParsingSQLRouter.parse [例:logicSQL=insert into aaatest(a,b,c) values(?,?,?), parameters=1,1,1] :

-----

    @Override
    public SQLStatement parse(final String logicSQL, final int parametersSize) {
        SQLParsingEngine parsingEngine = new SQLParsingEngine(databaseType, logicSQL, shardingRule); [databaseType==MySQL]
        SQLStatement result = parsingEngine.parse();
        if (result instanceof InsertStatement) {
            ((InsertStatement) result).appendGenerateKeyToken(shardingRule, parametersSize);
        }
        return result;
    }

-----

io/shardingjdbc/core/parsing/SQLParsingEngine.parse[lexer:词法分析器]：

-----

        public SQLStatement parse() {
                LexerEngine lexerEngine = LexerEngineFactory.newInstance(dbType, sql);
                lexerEngine.nextToken();
                return SQLParserFactory.newInstance(dbType, lexerEngine.getCurrentToken().getType(), shardingRule, lexerEngine).parse();
        }

        public MySQLLexer(final String input) {
                super(input, dictionary);
        }

        public final void nextToken() {
                skipIgnoredToken();
                if (isVariableBegin()) {
                    currentToken = new Tokenizer(input, dictionary, offset).scanVariable();
                } else if (isNCharBegin()) {
                    currentToken = new Tokenizer(input, dictionary, ++offset).scanChars();
                } else if (isIdentifierBegin()) {
                    currentToken = new Tokenizer(input, dictionary, offset).scanIdentifier();
                } else if (isHexDecimalBegin()) {
                    currentToken = new Tokenizer(input, dictionary, offset).scanHexDecimal();
                } else if (isNumberBegin()) {
                    currentToken = new Tokenizer(input, dictionary, offset).scanNumber();
                } else if (isSymbolBegin()) {
                    currentToken = new Tokenizer(input, dictionary, offset).scanSymbol();
                } else if (isCharsBegin()) {
                    currentToken = new Tokenizer(input, dictionary, offset).scanChars();
                } else if (isEnd()) {
                    currentToken = new Token(Assist.END, "", offset);
                } else {
                    throw new SQLParsingException(this, Assist.ERROR);
                }
                offset = currentToken.getEndPosition();
        }

-----

--> io/shardingjdbc/core/parsing/lexer/analyzer/Tokenizer.scanIdentifier 截出Token，例如INSERT  
<<< io/shardingjdbc/core/parsing/parser/sql/SQLParserFactory.newInstance:     

-----

    public static SQLParser newInstance(final DatabaseType dbType, final TokenType tokenType, final ShardingRule shardingRule, final LexerEngine lexerEngine) {
        if (!(tokenType instanceof DefaultKeyword)) {
            throw new SQLParsingUnsupportedException(tokenType);
        }
        switch ((DefaultKeyword) tokenType)  {
            case SELECT:
                return SelectParserFactory.newInstance(dbType, shardingRule, lexerEngine);
            case INSERT:
                return InsertParserFactory.newInstance(dbType, shardingRule, lexerEngine);
            case UPDATE:
                return UpdateParserFactory.newInstance(dbType, shardingRule, lexerEngine);
            case DELETE:
                return DeleteParserFactory.newInstance(dbType, shardingRule, lexerEngine);
            case CREATE:
                return CreateParserFactory.newInstance(dbType, shardingRule, lexerEngine);
            case ALTER:
                return AlterParserFactory.newInstance(dbType, shardingRule, lexerEngine);
            case DROP:
                return DropParserFactory.newInstance(dbType, shardingRule, lexerEngine);
            case TRUNCATE:
                return TruncateParserFactory.newInstance(dbType, shardingRule, lexerEngine);
            case SET:
            case COMMIT:
            case ROLLBACK:
            case SAVEPOINT:
            case BEGIN:
                return TCLParserFactory.newInstance(dbType, shardingRule, lexerEngine);
            default:
                throw new SQLParsingUnsupportedException(lexerEngine.getCurrentToken().getType());
        }
    }

-----

这个例子是INSERT。
--> io/shardingjdbc/core/parsing/parser/sql/dml/insert/InsertParserFactory.newInstance      
--> return new MySQLInsertParser(shardingRule, lexerEngine);      
--> io/shardingjdbc/core/parsing/parser/dialect/mysql/clause/facade/MySQLInsertClauseParserFacade     
--> io/shardingjdbc/core/parsing/parser/dialect/mysql/clause/MySQLTableReferencesClauseParser      
aliasExpressionParser === io/shardingjdbc/core/parsing/parser/dialect/ExpressionParserFactory.createAliasExpressionParser [return new MySQLAliasExpressionParser(lexerEngine);basicExpressionParser === io/shardingjdbc/core/parsing/parser/dialect/ExpressionParserFactory.createBasicExpressionParser]     
<<< MySQLInsertClauseParserFacade --> io/shardingjdbc/core/parsing/parser/dialect/mysql/clause/MySQLInsertValuesClauseParser : basicExpressionParser     
<<< MySQLInsertClauseParserFacade --> io/shardingjdbc/core/parsing/parser/dialect/mysql/clause/MySQLInsertSetClauseParser

回到上面SQLParsingEngine.parse 方法中 InsertParserFactory.newInstance 之后还有个链式的parse调用[return SQLParserFactory.newInstance(dbType, lexerEngine.getCurrentToken().getType(), shardingRule, lexerEngine).parse();]     
--> io/shardingjdbc/core/parsing/parser/sql/dml/insert/AbstractInsertParser.parse : 

-----

    @Override
    public final DMLStatement parse() {
        lexerEngine.nextToken();
        InsertStatement result = new InsertStatement();
        insertClauseParserFacade.getInsertIntoClauseParser().parse(result);
        insertClauseParserFacade.getInsertColumnsClauseParser().parse(result);
        if (lexerEngine.equalAny(DefaultKeyword.SELECT, Symbol.LEFT_PAREN)) {
            throw new UnsupportedOperationException("Cannot INSERT SELECT");
        }
        insertClauseParserFacade.getInsertValuesClauseParser().parse(result);
        insertClauseParserFacade.getInsertSetClauseParser().parse(result);
        appendGenerateKey(result);
        return result;
    }

-----

--> io/shardingjdbc/core/parsing/parser/clause/InsertIntoClauseParser :     

-----

    public void parse(final InsertStatement insertStatement) {
        lexerEngine.unsupportedIfEqual(getUnsupportedKeywordsBeforeInto());
        lexerEngine.skipUntil(DefaultKeyword.INTO);
        lexerEngine.nextToken();
        tableReferencesClauseParser.parse(insertStatement, true);
        skipBetweenTableAndValues(insertStatement);
    }

-----

--> io/shardingjdbc/core/parsing/parser/clause/TableReferencesClauseParser

-----

    public final void parse(final SQLStatement sqlStatement, final boolean isSingleTableOnly) {
        do {
            parseTableReference(sqlStatement, isSingleTableOnly);
        } while (lexerEngine.skipIfEqual(Symbol.COMMA));
    }

-----

-----


[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/SpringBoot_ShardingJdbc_Code_Load_Run.md)
