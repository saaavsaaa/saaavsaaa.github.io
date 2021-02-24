清理历史分区的时候发现有一个带编码字符的：
ts=20200921
ts=20200922%0D
ts=20200925

百度%0D得知是换行，creturn
试验了各种百度上的办法不行
并且自己写了各种生成%0D对应字符的shell脚本，并没有什么效果
最后想，反正暂时只留最后一个分区就可以，并不影响使用
于是：hive --database ods -S -e "alter table edw.******** drop partition(ts!='20200925');"

hive 表创建时，如果表备注中有"#"（暂时只发现井号），在 impala 中进行 invalidate metadata 或 refresh 后也无法访问 impala 表，会报错说数据版本不对，推测时由于 [impala-shell](https://github.com/saaavsaaa/saaavsaaa.github.io/blob/master/aaa/impala_shell.md) 是python 开发的并且没有对注释符号做特殊处理导致的，正在确认，后续如果有结果或改正再补充
报错与未执行 refresh 差不多
```
Query: SELECT count(*) FROM table
Query submitted at: 2021-02-05 13:51:50 (Coordinator:http: //xxxxxx:25000) 
Query progress can be monitored at: http://xxxxxxx:25000/query_plan?query_id=e0485597184609cd:7283bcb600000000
WARNINGS: File 'hdfs://nameservice01/user/hive/warehouse/edw.db/table/000005_0' has an invalid version nunbe:♐%
This could be due to stale metadata. Try running "refresh edw.table".
```

hive 表在被使用时，无法用 sqoop 导数据，由于表上有锁，只能等锁被释放，后续的导数据或insert才会依次执行。show locks [OPTION:table_name];unlock table table_name;unlock table table_name partition(partition='partition_name'); lock table table_name exclusive ;如果show locks无法执行，需要指定 LockManager，set hive.support.concurrency=true;set hive.txn.manager = org.apache.hadoop.hive.ql.lockmgr.DummyTxnManager; 实在不行就直接该 mysql 中的 hive 元数据 select * from HIVE_LOCKS; delete from HIVE_LOCKS where HL_DB = 'db' and HL_TABLE = 'table'; 注意：表名和字段大写。
