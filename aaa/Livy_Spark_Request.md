官方文档 : http://livy.incubator.apache.org/docs/latest/rest-api.html

通过livy提交spark job时可以配置的参数，因为是间接提交给spark的，所以并不是所有spark参数都支持

kind ：spark, pyspark or sparkr等，这个参数我没用过，官方文档上说现在是可选的，也没什么特别的用处，有想限制提交语法的可以自己看上面官方文档

./bin/spark-submit \
  --master spark://xxx.xxx.xxx.xxx:xxxx 
  --proxy-user aaa                                     //模拟提交Job的用户
  --jars                                               //逗号分隔的本地jar包，包含在driver和executor的classpath下
  --py-files                                           //逗号分隔的”.zip”,”.egg”或者“.py”文件，这些文件放在python app的PYTHONPATH下面
  --files                                              //逗号分隔的文件，这些文件放在每个executor的工作目录下面
  --num-executors 50 
  --executor-memory 512m                               //spark.executor.memory
  --executor-cores 4                                   //spark.executor.cores
　--total-executor-cores 400                           //standalone default all cores
  --driver-memory 1G                                   //spark.driver.memory
  --driver-cores 1                                     //spark.driver.cores仅在standalone集群deploy模式下使用
  --queue                                              //YARN队列的名称
  --name
  --conf <key>=<value> 
  
jars : 对应上面 --jars

pyFiles : 对应上面 --py-files

files : 对应上面 --files

driverMemory : 对应driver-memory参数，Driver进程的内存

proxyUser : 对应proxy-user

driverCores ：对应spark.driver.cores参数，CPU核数

executorMemory : 对应executor-memory，每个executor进程使用的内存数

executorCores : 对应executor-cores，每个executor进程使用的CPU核数

numExecutors : 对应num-executors，executor个数

archives : Archives to be used in this session

queue : 对应 --queue

name : 对应 --name

conf : 对应 --conf

heartbeatTimeoutInSecond : 心跳
