官方文档 : http://livy.incubator.apache.org/docs/latest/rest-api.html

通过livy提交spark job时可以配置的参数，因为是间接提交给spark的，所以并不是所有spark参数都支持

kind ：spark, pyspark or sparkr等，这个参数我没用过，官方文档上说现在是可选的，也没什么特别的用处，有想限制提交语法的可以自己看上面官方文档

proxyUser : 启动Session的用户

jars : 会话中要提交的jar包

pyFiles : 会话中要提交的py文件

files : 随便什么文件了

driverMemory : 对应spark的spark.driver.memory参数，Driver进程的内存

driverCores ：对应spark的spark.driver.cores参数，CPU核数
