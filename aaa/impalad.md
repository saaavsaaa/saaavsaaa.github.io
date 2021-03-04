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