impalad 启动   
impalad-main.cc
```
int ImpaladMain(int argc, char** argv) {
  InitCommonRuntime(argc, argv, true);      // be/src/common/init.cc CPU、硬盘、内存等信息，检查CPU是否符合要求，设置默认主机名，可以被覆盖
  
  ABORT_IF_ERROR(LlvmCodeGen::InitializeLlvm());   // be/src/codegen/llvm-codegen.cc
  JniUtil::InitLibhdfs();   // be/src/util/jni-util.cc 
  ABORT_IF_ERROR(HBaseTableScanner::Init());    // be/src/exec/hbase-table-scanner.cc   获取当前进程对应的 JNIEnv*  （hdfsJniHelper.h）
  ABORT_IF_ERROR(HBaseTable::InitJNI());        // be/src/runtime/hbase-table.cc
  ABORT_IF_ERROR(HBaseTableWriter::InitJNI());  // be/src/exec/hbase-table-writer.cc
  ABORT_IF_ERROR(HiveUdfCall::InitEnv());       // be/src/exprs/hive-udf-call.cc
  ABORT_IF_ERROR(JniCatalogCacheUpdateIterator::InitJNI());   //be/src/catalog/catalog-util.cc
  InitFeSupport();    // be/src/service/fe-support.cc org/apache/impala/service/FeSupport
  ...
  CommonMetrics::InitCommonMetrics(exec_env.metrics());   // be/src/util/common-metrics.cc  kudu-client.version
  ...
  tzdata_path->SetValue(TimezoneDatabase::GetPath());
  ABORT_IF_ERROR(StartMemoryMaintenanceThread()); // Memory metrics 已经在 Init() 创建了.
  ABORT_IF_ERROR(StartThreadInstrumentation(exec_env.metrics(), exec_env.webserver(), true));
  InitRpcEventTracing(exec_env.webserver(), exec_env.rpc_mgr());
  boost::shared_ptr<ImpalaServer> impala_server(new ImpalaServer(&exec_env));   // be/src/service/impala-server.cc
  Status status = impala_server->Start(FLAGS_be_port, FLAGS_beeswax_port, FLAGS_hs2_port);  // 参见 be/src/service/impala-server.cc
  ...
  impala_server->Join(); // 阻塞直到进程退出 thrift_be_server_ beeswax_server_ hs2_server_ beeswax_server_ hs2_server_ 
  ... 
```
be/src/service/impala-server.cc
```
Status ImpalaServer::Start(int32_t thrift_be_port, int32_t beeswax_port,
   int32_t hs2_port) {
  exec_env_->SetImpalaServer(this);
  必须在向 ExecEnv 注册 ImpalaServer 后注册HTTP handlers。否则，HTTPhandlers 将尝试通过 ExecEnv 单例解析 ImpalaServer，报nullptr。
  http_handler_.reset(new ImpalaHttpHandler(this));
  http_handler_->RegisterHandlers(exec_env_->webserver());
  
  // 订阅statestore。协调器需要订阅 catalog 主题，并等待初始的 catalog 更新
  RETURN_IF_ERROR(exec_env_->StartStatestoreSubscriberService());     // be/src/runtime/exec-env.cc
  if (FLAGS_is_coordinator) exec_env_->frontend()->WaitForCatalog();
  ...
  // 启动内部服务（这代码看命名就可以，没必要注释了）
  if (thrift_be_port > 0 || (TestInfo::is_test() && thrift_be_port == 0)) {
    boost::shared_ptr<ImpalaInternalService> thrift_if(new ImpalaInternalService());  // be/src/service/impala-internal-service.cc
    boost::shared_ptr<TProcessor> be_processor(new ImpalaInternalServiceProcessor(thrift_if));
    boost::shared_ptr<TProcessorEventHandler> event_handler(new RpcEventHandler("backend", exec_env_->metrics()));
    be_processor->setEventHandler(event_handler);

    ThriftServerBuilder be_builder("backend", be_processor, thrift_be_port);

    if (IsInternalTlsConfigured()) {
      LOG(INFO) << "Enabling SSL for backend";
      be_builder.ssl(FLAGS_ssl_server_certificate, FLAGS_ssl_private_key).pem_password_cmd(FLAGS_ssl_private_key_password_cmd)
          .ssl_version(ssl_version).cipher_list(FLAGS_ssl_cipher_list);
    }
    ThriftServer* server;
    RETURN_IF_ERROR(be_builder.metrics(exec_env_->metrics()).Build(&server));
    thrift_be_server_.reset(server);
  }
  
  if (!FLAGS_is_coordinator) {
    LOG(INFO) << "Initialized executor Impala server on "
              << TNetworkAddressToString(GetThriftBackendAddress());
  } else {
    // Initialize the client servers.
    boost::shared_ptr<ImpalaServer> handler = shared_from_this();
    ... // beeswax_port  hs2 HiveServer2
  }
  LOG(INFO) << "Initialized coordinator/executor Impala server on "
      << TNetworkAddressToString(GetThriftBackendAddress());
  // 启动 RPC 服务
  RETURN_IF_ERROR(exec_env_->StartKrpcService());
  ... // thrift_be_server_(Impala InternalService) hs2_server_(Impala HiveServer2 Service) beeswax_server_(Impala Beeswax Service)  listening on server_->port();
  
  ImpaladMetrics::IMPALA_SERVER_READY->SetValue(true);
  LOG(INFO) << "Impala has started.";

  return Status::OK();
}
```

需要排查的查询功能：
```
class ImpalaServer : public ImpalaServiceIf,
                     public ImpalaHiveServer2ServiceIf,
                     public ThriftServer::ConnectionHandlerIf,
                     public boost::enable_shared_from_this<ImpalaServer>,
                     public CacheLineAligned {
......
  /// ImpalaService rpcs: Beeswax API (implemented in impala-beeswax-server.cc)
  virtual void query(beeswax::QueryHandle& query_handle, const beeswax::Query& query);
```
impala-beeswax-server.cc

```
void ImpalaServer::query(QueryHandle& query_handle, const Query& query) {
  VLOG_QUERY << "query(): query=" << query.query;
  RAISE_IF_ERROR(CheckNotShuttingDown(), SQLSTATE_GENERAL_ERROR);

  ScopedSessionState session_handle(this);
  shared_ptr<SessionState> session;
  RAISE_IF_ERROR(
      session_handle.WithBeeswaxSession(ThriftServer::GetThreadConnectionId(), &session),
      SQLSTATE_GENERAL_ERROR);
  TQueryCtx query_ctx;
  // raise general error for request conversion error;
  RAISE_IF_ERROR(QueryToTQueryContext(query, &query_ctx), SQLSTATE_GENERAL_ERROR);

  // raise Syntax error or access violation; it's likely to be syntax/analysis error
  // TODO: that may not be true; fix this
  shared_ptr<ClientRequestState> request_state;
  RAISE_IF_ERROR(Execute(&query_ctx, session, &request_state),
      SQLSTATE_SYNTAX_ERROR_OR_ACCESS_VIOLATION);

  // start thread to wait for results to become available, which will allow
  // us to advance query state to FINISHED or EXCEPTION
  Status status = request_state->WaitAsync();
  if (!status.ok()) {
    discard_result(UnregisterQuery(request_state->query_id(), false, &status));
    RaiseBeeswaxException(status.GetDetail(), SQLSTATE_GENERAL_ERROR);
  }
  // Once the query is running do a final check for session closure and add it to the
  // set of in-flight queries.
  status = SetQueryInflight(session, request_state);
  if (!status.ok()) {
    discard_result(UnregisterQuery(request_state->query_id(), false, &status));
    RaiseBeeswaxException(status.GetDetail(), SQLSTATE_GENERAL_ERROR);
  }
  TUniqueIdToQueryHandle(request_state->query_id(), &query_handle);
}
```
be/src/service/impala-server.cc
```
Status ImpalaServer::Execute(TQueryCtx* query_ctx,
    shared_ptr<SessionState> session_state,
    shared_ptr<ClientRequestState>* request_state) {
  PrepareQueryContext(query_ctx);
  ScopedThreadContext debug_ctx(GetThreadDebugInfo(), query_ctx->query_id);
  ImpaladMetrics::IMPALA_SERVER_NUM_QUERIES->Increment(1L);

  // Redact the SQL stmt and update the query context
  string stmt = replace_all_copy(query_ctx->client_request.stmt, "\n", " ");
  Redact(&stmt);
  query_ctx->client_request.__set_redacted_stmt((const string) stmt);

  bool registered_request_state;
  Status status = ExecuteInternal(*query_ctx, session_state, &registered_request_state,
      request_state);
  if (!status.ok() && registered_request_state) {
    discard_result(UnregisterQuery((*request_state)->query_id(), false, &status));
  }
  return status;
}


Status ImpalaServer::ExecuteInternal(
    const TQueryCtx& query_ctx,
    shared_ptr<SessionState> session_state,
    bool* registered_request_state,
    shared_ptr<ClientRequestState>* request_state) {
  DCHECK(session_state != nullptr);
  *registered_request_state = false;

  request_state->reset(new ClientRequestState(query_ctx, exec_env_, exec_env_->frontend(), this, session_state)); //托管 ClientRequestState
  // client-request-state.h RuntimeProfile::EventSequence* query_events() const { return query_events_; }
  (*request_state)->query_events()->MarkEvent("Query submitted");

  TExecRequest result;
  {
    // 对 request_state 加锁，保证注册和设置 result_metadata 是原子的
    lock_guard<mutex> l(*(*request_state)->lock());

    RETURN_IF_ERROR(RegisterQuery(session_state, *request_state)); // 使用全局唯一 query_id 向 client_request_state_map_ 注册 查询执行状态
    *registered_request_state = true;
......

    RETURN_IF_ERROR((*request_state)->UpdateQueryStatus(exec_env_->frontend()->GetExecRequest(query_ctx, &result)));
    (*request_state)->query_events()->MarkEvent("Planning finished");
    (*request_state)->set_user_profile_access(result.user_has_profile_access);
    (*request_state)->summary_profile()->AddEventSequence(result.timeline.name, result.timeline);
    (*request_state)->SetFrontendProfile(result.profile); // 设置frontend产生的概要profile，frontend 在planning期间产生并通过TExecRequest返回给后端
    if (result.__isset.result_set_metadata) {(*request_state)->set_result_metadata(result.result_set_metadata);}
  }
  VLOG(2) << "Execution request: " << ThriftDebugString(result);

  // 开始执行查询; also starts fragment status reports
  RETURN_IF_ERROR((*request_state)->Exec(&result));
  Status status = UpdateCatalogMetrics();
  if (!status.ok()) {
    VLOG_QUERY << "Couldn't update catalog metrics: " << status.GetDetail();
  }

  if ((*request_state)->schedule() != nullptr) {
    const PerBackendExecParams& per_backend_params =
        (*request_state)->schedule()->per_backend_exec_params();
    if (!per_backend_params.empty()) {
      lock_guard<mutex> l(query_locations_lock_);
      for (const auto& entry : per_backend_params) {
        const TNetworkAddress& host = entry.first;
        query_locations_[host].insert((*request_state)->query_id());
      }
    }
  }

  return Status::OK();
}
```
be/src/service/client-request-state.cc
```
ClientRequestState::ClientRequestState(
    const TQueryCtx& query_ctx, ExecEnv* exec_env, Frontend* frontend,
    ImpalaServer* server, shared_ptr<ImpalaServer::SessionState> session)
  : query_ctx_(query_ctx),
    last_active_time_ms_(numeric_limits<int64_t>::max()),
    child_query_executor_(new ChildQueryExecutor),
......
  query_events_ = summary_profile_->AddEventSequence("Query Timeline");
  query_events_->Start();

```
