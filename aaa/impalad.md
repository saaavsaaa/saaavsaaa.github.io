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
```
