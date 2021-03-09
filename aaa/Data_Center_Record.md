清理历史分区的时候发现有一个带编码字符的：
ts=20200921
ts=20200922%0D
ts=20200925

百度%0D得知是换行，creturn
试验了各种百度上的办法不行
并且自己写了各种生成%0D对应字符的shell脚本，并没有什么效果
最后想，反正暂时只留最后一个分区就可以，并不影响使用
于是：hive --database ods -S -e "alter table edw.******** drop partition(ts!='20200925');"

hive 表创建时，如果表备注中有"# �"（暂时发现去掉这两个就好了），在 impala 中进行 invalidate metadata 或 refresh 后也无法访问 impala 表，会报错说数据版本不对，[impala-shell](https://github.com/saaavsaaa/saaavsaaa.github.io/blob/master/aaa/impala_shell.md) 

在 [impala_client.py](https://github.com/cloudera/Impala/blob/cdh6.3.0/shell/impala_client.py) 的 wait_to_finish 方法中 print 出以下结果（while true 轮询 impalad）。

self.query_state ================{'EXCEPTION': 5, 'CREATED': 0, 'COMPILED': 2, 'FINISHED': 4, 'RUNNING': 3, 'INITIALIZED': 1}   
wait_to_finish================= 2   
wait_to_finish================= 3   
wait_to_finish================= 5   

其中 get_query_state 方法主要调用 imp_service.get_state(last_query_handle)，通过 BeeswaxService.py 的  
  def get_state(self, handle):   
    self.send_get_state(handle)   
    return self.recv_get_state()    
方法，借助 thrift 的协议，从 impalad 获取状态。   
错误信息是从 rpc_result = self._do_rpc(   
        lambda: self.imp_service.get_log(last_query_handle.log_context))   
    log, status = rpc_result   
rpc_result 中获取的   

WARNINGS: File 'hdfs://nameservice01/user/hive/warehouse/edw.db/error_table/000027_0' has an invalid version number: �
This could be due to stale metadata. Try running "refresh edw.error_table".   

报错与未执行 refresh 差不多
```
Query: SELECT count(*) FROM table
Query submitted at: 2021-02-05 13:51:50 (Coordinator:http: //xxxxxx:25000) 
Query progress can be monitored at: http://xxxxxxx:25000/query_plan?query_id=e0485597184609cd:7283bcb600000000
WARNINGS: File 'hdfs://nameservice01/user/hive/warehouse/edw.db/table/000005_0' has an invalid version nunbe:♐%
This could be due to stale metadata. Try running "refresh edw.table".
```

hive 表在被使用时，无法用 sqoop 导数据，由于表上有锁，只能等锁被释放，后续的导数据或insert才会依次执行。show locks [OPTION:table_name];unlock table table_name;unlock table table_name partition(partition='partition_name'); lock table table_name exclusive ;如果show locks无法执行，需要指定 LockManager，set hive.support.concurrency=true;set hive.txn.manager = org.apache.hadoop.hive.ql.lockmgr.DummyTxnManager; 实在不行就直接该 mysql 中的 hive 元数据 select * from HIVE_LOCKS; delete from HIVE_LOCKS where HL_DB = 'db' and HL_TABLE = 'table'; 注意：表名和字段大写。

临时文件：   
sqoop 临时目录 this.tempRootDir = System.getProperty(OLD_SQOOP_TEST_IMPORT_ROOT_DIR, "\_sqoop");   
```
org/apache/sqoop/util/AppendUtils.java : public static Path getTempAppendDir(String salt, SqoopOptions options) {
  String tempDir = options.getTempRootDir() + Path.SEPARATOR + uuid + "_" + salt;（hdfs://nameservice01/user/aaa/_sqoop/61ae9e0e660f4198922e8f38f8d7a621_a343472f）
org/apache/sqoop/tool/ImportTool.java : private Path getOutputPath : outputPath = AppendUtils.getTempAppendDir(salt, options);
-->
  protected void lastModifiedMerge
  private boolean initIncrementalConstraints //初始化增量导入数据范围的约束
  protected boolean importTable(SqoopOptions options) throws IOException, ImportException { 
     if (!initIncrementalConstraints(options, context)) {return false;} // 增量导入需设置过滤条件以获取最后一条记录
     if options.isAppendMode() else if (options.getIncrementalMode() == SqoopOptions.IncrementalMode.DateLastModified) { lastModifiedMerge(options, context);

```

