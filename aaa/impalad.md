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
  impala_server->Join(); // 阻塞直到进程退出
  ... 
```
be/src/service/impala-server.cc
```
Status ImpalaServer::Start(int32_t thrift_be_port, int32_t beeswax_port,
   int32_t hs2_port) {
  exec_env_->SetImpalaServer(this);
  必须在向 ExecEnv 注册 ImpalaServer 后注册HTTP handlers。否则，HTTPhandlers 将尝试通过 ExecEnv 单例解析 ImpalaServer，报nullptr。
  
```
