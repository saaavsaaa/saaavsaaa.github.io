源码部分的注释有些翻译了，有些就直接贴的英文，我想，能看下去的人应该不会受多大影响，毕竟...也挺随意的

通过命令找脚本：
which spark2-submit

找到实际提交的脚本：
exec $LIB_DIR/spark2/bin/spark-submit "$@"  

找到提交的类：
exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@"

```
package org.apache.spark.deploy

......

private[spark] class SparkSubmit extends Logging {

......

object SparkSubmit extends CommandLineUtils with Logging {
  override def main(args: Array[String]): Unit = {
    val submit = new SparkSubmit() {
      self =>

      override protected def parseArguments(args: Array[String]): SparkSubmitArguments = {
        new SparkSubmitArguments(args) {
          override protected def logInfo(msg: => String): Unit = self.logInfo(msg)

          override protected def logWarning(msg: => String): Unit = self.logWarning(msg)
        }
      }

      override protected def logInfo(msg: => String): Unit = printMessage(msg)

      override protected def logWarning(msg: => String): Unit = printMessage(s"Warning: $msg")

      override def doSubmit(args: Array[String]): Unit = {
        try {
          super.doSubmit(args)
        } catch {
          case e: SparkUserAppException =>
            exitFn(e.exitCode)
        }
      }

    }

    submit.doSubmit(args)
  }

```

doSubmit ：private[spark] class SparkSubmit extends Logging {

```
  def doSubmit(args: Array[String]): Unit = {
    
    ...
    
    appArgs.action match {
      case SparkSubmitAction.SUBMIT => submit(appArgs, uninitLog)
      case SparkSubmitAction.KILL => kill(appArgs)
      case SparkSubmitAction.REQUEST_STATUS => requestStatus(appArgs)
      case SparkSubmitAction.PRINT_VERSION => printVersion()
    }
  }
```
SparkSubmitAction.SUBMIT : submit
```
  /**
   * Submit the application using the provided parameters, ensuring to first wrap
   * in a doAs when --proxy-user is specified.
   使用提供的参数提交应用程序，确保在指定--proxy user时首先在proxyUser.doAs中包装。
   */
  @tailrec
  private def submit(args: SparkSubmitArguments, uninitLog: Boolean): Unit = {

    def doRunMain(): Unit = {
      if (args.proxyUser != null) {
        val proxyUser = UserGroupInformation.createProxyUser(args.proxyUser,
          UserGroupInformation.getCurrentUser())
        try {
          proxyUser.doAs(new PrivilegedExceptionAction[Unit]() {
            override def run(): Unit = {
              runMain(args, uninitLog)
            }
          })
        } catch {
          case e: Exception =>
            // Hadoop's AuthorizationException suppresses the exception's stack trace, which
            // makes the message printed to the output by the JVM not very helpful. Instead,
            // detect exceptions with empty stack traces here, and treat them differently.
            if (e.getStackTrace().length == 0) {
              error(s"ERROR: ${e.getClass().getName()}: ${e.getMessage()}")
            } else {
              throw e
            }
        }
      } else {
        runMain(args, uninitLog)
      }
    }
    
     // In standalone cluster mode, there are two submission gateways:
    //   (1) The traditional RPC gateway using o.a.s.deploy.Client as a wrapper
    //   (2) The new REST-based gateway introduced in Spark 1.3
    // The latter is the default behavior as of Spark 1.3, but Spark submit will fail over
    // to use the legacy gateway if the master endpoint turns out to be not a REST server.
    //（1）传统RPC网关使用o.a.s.deploy.Client包装
    //（2）Spark 1.3中引入了新的基于REST的网关
    //后者是Spark 1.3的默认行为，但如果主端点不是REST服务器，则Spark Submit将故障转移到使用旧网关。
    if (args.isStandaloneCluster && args.useRest) {
      try {
        logInfo("Running Spark using the REST application submission protocol.")
        doRunMain()
      } catch {
        // Fail over to use the legacy submission gateway
        case e: SubmitRestConnectionException =>
          logWarning(s"Master endpoint ${args.master} was not a REST server. " +
            "Falling back to legacy submission gateway instead.")
          args.useRest = false
          submit(args, false)
      }
    // In all other modes, just run the main class as prepared 在其他模式下，只需按准备好的方式运行主类
    } else {
      doRunMain()
    }
```
runMain : 
```
/**
   * Run the main method of the child class using the submit arguments.
   *
   * This runs in two steps. First, we prepare the launch environment by setting up
   * the appropriate classpath, system properties, and application arguments for
   * running the child main class based on the cluster manager and the deploy mode.
   * Second, we use this launch environment to invoke the main method of the child
   * main class.
   *
   * Note that this main class will not be the one provided by the user if we're
   * running cluster deploy mode or python applications.
   使用submit参数运行子类的main方法。

  这分为两个步骤。首先，我们通过设置适当的类路径、系统属性和应用程序参数来准备启动环境，以便基于集群管理器和部署模式运行子主类。
  其次，我们使用这个启动环境来调用子main类的main方法。

  请注意，如果我们运行的是集群部署模式或python应用程序，那么这个主类将不是用户提供的类。
   */
  private def runMain(args: SparkSubmitArguments, uninitLog: Boolean): Unit = {
    val (childArgs, childClasspath, sparkConf, childMainClass) = prepareSubmitEnvironment(args)
    // Let the main class re-initialize the logging system once it starts.
    if (uninitLog) {
      Logging.uninitialize()
    }

    if (args.verbose) {
      logInfo(s"Main class:\n$childMainClass")
      logInfo(s"Arguments:\n${childArgs.mkString("\n")}")
      // sysProps may contain sensitive information, so redact before printing
      logInfo(s"Spark config:\n${Utils.redact(sparkConf.getAll.toMap).mkString("\n")}")
      logInfo(s"Classpath elements:\n${childClasspath.mkString("\n")}")
      logInfo("\n")
    }

    val loader =
      if (sparkConf.get(DRIVER_USER_CLASS_PATH_FIRST)) {
        new ChildFirstURLClassLoader(new Array[URL](0),
          Thread.currentThread.getContextClassLoader)
      } else {
        new MutableURLClassLoader(new Array[URL](0),
          Thread.currentThread.getContextClassLoader)
      }
    Thread.currentThread.setContextClassLoader(loader)

    for (jar <- childClasspath) {
      addJarToClasspath(jar, loader)
    }

    var mainClass: Class[_] = null

    try {
      mainClass = Utils.classForName(childMainClass)
    } catch {
      case e: ClassNotFoundException =>
        logWarning(s"Failed to load $childMainClass.", e)
        if (childMainClass.contains("thriftserver")) {
          logInfo(s"Failed to load main class $childMainClass.")
          logInfo("You need to build Spark with -Phive and -Phive-thriftserver.")
        }
        throw new SparkUserAppException(CLASS_NOT_FOUND_EXIT_STATUS)
      case e: NoClassDefFoundError =>
        logWarning(s"Failed to load $childMainClass: ${e.getMessage()}")
        if (e.getMessage.contains("org/apache/hadoop/hive")) {
          logInfo(s"Failed to load hive class.")
          logInfo("You need to build Spark with -Phive and -Phive-thriftserver.")
        }
        throw new SparkUserAppException(CLASS_NOT_FOUND_EXIT_STATUS)
    }

    val app: SparkApplication = if (classOf[SparkApplication].isAssignableFrom(mainClass)) {
      mainClass.newInstance().asInstanceOf[SparkApplication]
    } else {
      // SPARK-4170
      if (classOf[scala.App].isAssignableFrom(mainClass)) {
        logWarning("Subclasses of scala.App may not work correctly. Use a main() method instead.")
      }
      new JavaMainApplication(mainClass)
    }
```
val (childArgs, childClasspath, sparkConf, childMainClass) = prepareSubmitEnvironment(args)
准备提交所需环境变量：childArgs：子进程参数、childClasspath：子classpath列表、sparkConf：系统参数的map集合、childMainClass：子的主类；   
设置集群管理器，目前支持：YARN,STANDLONE,MESOS,KUBERNETES,LOCAL，--master yarn；设置部署模式--deploy-mode，默认client,--deploy mode cluster/client；支持和不支持的模式、各种资源、配置的全局路径的、下载远程文件、各种jar、R或Python等执行所需、忽略无效的host、执行所在环境如shell相关等；   
CLIENT childMainClass = args.mainClass 【localPrimaryResource localJars】JavaMainApplication  

isYarnCluster childMainClass = YARN_CLUSTER_SUBMIT_CLASS;
CLUSTE 时sparkConf.remove("spark.driver.host") https://mvnrepository.com/artifact/org.apache.spark/spark-yarn  
```
yarn-cluster 模式下
submit -> runMain -> YarnClusterApplication.start
// SparkSubmit would use yarn cache to distribute files & jars in yarn mode,
// so remove them from sparkConf here for yarn mode.
new Client(new ClientArguments(args), conf).run()  

// If set spark.yarn.submit.waitAppCompletion to true, it will stay alive reporting the application's status until the application has exited for any reason. Otherwise, the client process will exit after submission.
def run(): Unit = {
    this.appId = submitApplication()

  /**
   * Submit an application running our ApplicationMaster to the ResourceManager.向 ResourceManager 提交运行我们ApplicationMaster的应用程序
   *
   * The stable Yarn API provides a convenience method (YarnClient#createApplication) for
   * creating applications and setting up the application submission context. This was not
   * available in the alpha API.
   stable版的Yarn API为创建应用程序和设置应用程序提交上下文提供了一种方便的方法（YarnClient#createApplication）。这在alpha API中不可用。
   */
  def submitApplication(): ApplicationId = {
    var appId: ApplicationId = null
    try {
      launcherBackend.connect()
      yarnClient.init(hadoopConf)
      yarnClient.start()

      logInfo("Requesting a new application from cluster with %d NodeManagers"
        .format(yarnClient.getYarnClusterMetrics.getNumNodeManagers))

      // Get a new application from our RM 
      // 从ResourceManager获取一个newApp用于运行AM。通过getNewApplicationResponse()返回newApp需要资源情况(newAppResponse)。
      val newApp = yarnClient.createApplication()
      // The response sent by the <code>ResourceManager</code> to the client for a request to get a new {@link ApplicationId} for submitting applications.
      val newAppResponse = newApp.getNewApplicationResponse()
      appId = newAppResponse.getApplicationId()

      // 用于为HDFS和Yarn设置Spark调用程序上下文。当Spark应用程序在Yarn和HDFS上运行时，它的调用程序上下文将被写入Yarn RM审计日志和hdfs-audit.log. 这可以帮助用户更好地诊断和理解特定的应用程序如何影响Hadoop系统的各个部分，以及他们可能产生的潜在问题（例如，NN过载）。正如HDFS-9184中提到的HDFS，对于给定的HDFS操作，跟踪哪个上层作业发出它非常有用。
      new CallerContext("CLIENT", sparkConf.get(APP_CALLER_CONTEXT),
        Option(appId.toString)).setCurrentContext()

      // Verify whether the cluster has enough resources for our AM 验证集群是否有足够的资源来运行AM
      verifyClusterResources(newAppResponse)

      // Set up the appropriate contexts to launch our AM 设置合适的上下文来启动AM
      val containerContext = createContainerLaunchContext(newAppResponse)
      val appContext = createApplicationSubmissionContext(newApp, containerContext)

      // Finally, submit and monitor the application  向yarn提交任务启动的请求，并监控application
      logInfo(s"Submitting application $appId to ResourceManager")
      // 提交应用给YARN。阻塞调用，提交成功并被 ResourceManager 认可才会返回
      yarnClient.submitApplication(appContext)
      launcherBackend.setAppId(appId.toString)
      reportLauncherState(SparkAppHandle.State.SUBMITTED)

      appId
    } catch {
      case e: Throwable =>
        if (appId != null) {
          cleanupStagingDir(appId)
        }
        throw e
    }
  }
```
createContainerLaunchContext : Set up a ContainerLaunchContext to launch our ApplicationMaster container.This sets up the launch environment, java options, and the command for launching the AM.
```
  private def createContainerLaunchContext(newAppResponse: GetNewApplicationResponse)
    : ContainerLaunchContext = {
    logInfo("Setting up container launch context for our AM")
    val appId = newAppResponse.getApplicationId
    val appStagingDirPath = new Path(appStagingBaseDir, getAppStagingDir(appId))
    val pySparkArchives =
      if (sparkConf.get(IS_PYTHON_APP)) {
        findPySparkArchives()
      } else {
        Nil
      }

    val launchEnv = setupLaunchEnv(appStagingDirPath, pySparkArchives)
    val localResources = prepareLocalResources(appStagingDirPath, pySparkArchives)

    val amContainer = Records.newRecord(classOf[ContainerLaunchContext])
    amContainer.setLocalResources(localResources.asJava)
    amContainer.setEnvironment(launchEnv.asJava)

    val javaOpts = ListBuffer[String]()

    // Set the environment variable through a command prefix
    // to append to the existing value of the variable
    var prefixEnv: Option[String] = None

    // Add Xmx for AM memory
    javaOpts += "-Xmx" + amMemory + "m"

    val tmpDir = new Path(Environment.PWD.$$(), YarnConfiguration.DEFAULT_CONTAINER_TEMP_DIR)
    javaOpts += "-Djava.io.tmpdir=" + tmpDir

    // TODO: Remove once cpuset version is pushed out.
    // The context is, default gc for server class machines ends up using all cores to do gc -
    // hence if there are multiple containers in same node, Spark GC affects all other containers'
    // performance (which can be that of other Spark containers)
    // Instead of using this, rely on cpusets by YARN to enforce "proper" Spark behavior in
    // multi-tenant environments. Not sure how default Java GC behaves if it is limited to subset
    // of cores on a node.
    val useConcurrentAndIncrementalGC = launchEnv.get("SPARK_USE_CONC_INCR_GC").exists(_.toBoolean)
    if (useConcurrentAndIncrementalGC) {
      // In our expts, using (default) throughput collector has severe perf ramifications in
      // multi-tenant machines
      javaOpts += "-XX:+UseConcMarkSweepGC"
      javaOpts += "-XX:MaxTenuringThreshold=31"
      javaOpts += "-XX:SurvivorRatio=8"
      javaOpts += "-XX:+CMSIncrementalMode"
      javaOpts += "-XX:+CMSIncrementalPacing"
      javaOpts += "-XX:CMSIncrementalDutyCycleMin=0"
      javaOpts += "-XX:CMSIncrementalDutyCycle=10"
    }

    // Include driver-specific java options if we are launching a driver
    if (isClusterMode) {
      sparkConf.get(DRIVER_JAVA_OPTIONS).foreach { opts =>
        javaOpts ++= Utils.splitCommandString(opts)
          .map(Utils.substituteAppId(_, appId.toString))
          .map(YarnSparkHadoopUtil.escapeForShell)
      }
      val libraryPaths = Seq(sparkConf.get(DRIVER_LIBRARY_PATH),
        sys.props.get("spark.driver.libraryPath")).flatten
      if (libraryPaths.nonEmpty) {
        prefixEnv = Some(createLibraryPathPrefix(libraryPaths.mkString(File.pathSeparator),
          sparkConf))
      }
      if (sparkConf.get(AM_JAVA_OPTIONS).isDefined) {
        logWarning(s"${AM_JAVA_OPTIONS.key} will not take effect in cluster mode")
      }
    } else {
      // Validate and include yarn am specific java options in yarn-client mode.
      sparkConf.get(AM_JAVA_OPTIONS).foreach { opts =>
        if (opts.contains("-Dspark")) {
          val msg = s"${AM_JAVA_OPTIONS.key} is not allowed to set Spark options (was '$opts')."
          throw new SparkException(msg)
        }
        if (opts.contains("-Xmx")) {
          val msg = s"${AM_JAVA_OPTIONS.key} is not allowed to specify max heap memory settings " +
            s"(was '$opts'). Use spark.yarn.am.memory instead."
          throw new SparkException(msg)
        }
        javaOpts ++= Utils.splitCommandString(opts)
          .map(Utils.substituteAppId(_, appId.toString))
          .map(YarnSparkHadoopUtil.escapeForShell)
      }
      sparkConf.get(AM_LIBRARY_PATH).foreach { paths =>
        prefixEnv = Some(createLibraryPathPrefix(paths, sparkConf))
      }
    }

    // For log4j configuration to reference
    javaOpts += ("-Dspark.yarn.app.container.log.dir=" + ApplicationConstants.LOG_DIR_EXPANSION_VAR)

    val userClass =
      if (isClusterMode) {
        Seq("--class", YarnSparkHadoopUtil.escapeForShell(args.userClass))
      } else {
        Nil
      }
    val userJar =
      if (args.userJar != null) {
        Seq("--jar", args.userJar)
      } else {
        Nil
      }
    val primaryPyFile =
      if (isClusterMode && args.primaryPyFile != null) {
        Seq("--primary-py-file", new Path(args.primaryPyFile).getName())
      } else {
        Nil
      }
    val primaryRFile =
      if (args.primaryRFile != null) {
        Seq("--primary-r-file", args.primaryRFile)
      } else {
        Nil
      }
    val amClass =
      if (isClusterMode) {
        Utils.classForName("org.apache.spark.deploy.yarn.ApplicationMaster").getName
      } else {
        Utils.classForName("org.apache.spark.deploy.yarn.ExecutorLauncher").getName
      }
    if (args.primaryRFile != null && args.primaryRFile.endsWith(".R")) {
      args.userArgs = ArrayBuffer(args.primaryRFile) ++ args.userArgs
    }
    val userArgs = args.userArgs.flatMap { arg =>
      Seq("--arg", YarnSparkHadoopUtil.escapeForShell(arg))
    }
    val amArgs =
      Seq(amClass) ++ userClass ++ userJar ++ primaryPyFile ++ primaryRFile ++ userArgs ++
      Seq("--properties-file", buildPath(Environment.PWD.$$(), LOCALIZED_CONF_DIR, SPARK_CONF_FILE))

    // Command for the ApplicationMaster
    val commands = prefixEnv ++
      Seq(Environment.JAVA_HOME.$$() + "/bin/java", "-server") ++
      javaOpts ++ amArgs ++
      Seq(
        "1>", ApplicationConstants.LOG_DIR_EXPANSION_VAR + "/stdout",
        "2>", ApplicationConstants.LOG_DIR_EXPANSION_VAR + "/stderr")

    // TODO: it would be nicer to just make sure there are no null commands here
    val printableCommands = commands.map(s => if (s == null) "null" else s).toList
    amContainer.setCommands(printableCommands.asJava)

    logDebug("===============================================================================")
    logDebug("YARN AM launch context:")
    logDebug(s"    user class: ${Option(args.userClass).getOrElse("N/A")}")
    logDebug("    env:")
    if (log.isDebugEnabled) {
      Utils.redact(sparkConf, launchEnv.toSeq).foreach { case (k, v) =>
        logDebug(s"        $k -> $v")
      }
    }
    logDebug("    resources:")
    localResources.foreach { case (k, v) => logDebug(s"        $k -> $v")}
    logDebug("    command:")
    logDebug(s"        ${printableCommands.mkString(" ")}")
    logDebug("===============================================================================")

    // send the acl settings into YARN to control who has access via YARN interfaces
    val securityManager = new SecurityManager(sparkConf)
    amContainer.setApplicationACLs(
      YarnSparkHadoopUtil.getApplicationAclsForYarn(securityManager).asJava)
    setupSecurityToken(amContainer)
    amContainer
  }
```
其中 org.apache.spark.deploy.yarn.ExecutorLauncher ： This object does not provide any special functionality. It exists so that it's easy to tell apart the client-mode AM from the cluster-mode AM when using tools such as ps or jps.
```
  def main(args: Array[String]): Unit = {
    SignalUtils.registerLogger(log)
    val amArgs = new ApplicationMasterArguments(args)
    master = new ApplicationMaster(amArgs)
    System.exit(master.run())
  }
```

应用代码中如SparkSession.builder()... .getOrCreate()创建SparkContext：
```
def getOrCreate(): SparkSession = synchronized {
      assertOnDriver()
      // Get the session from current thread's active session.
      var session = activeThreadSession.get()
      if ((session ne null) && !session.sparkContext.isStopped) {
        applyModifiableSettings(session)
        return session
      }

      // Global synchronization so we will only set the default session once.
      SparkSession.synchronized {
        // If the current thread does not have an active session, get it from the global session.
        session = defaultSession.get()
        if ((session ne null) && !session.sparkContext.isStopped) {
          applyModifiableSettings(session)
          return session
        }

        // No active nor global default session. Create a new one.
        val sparkContext = userSuppliedContext.getOrElse {
          val sparkConf = new SparkConf()
          options.foreach { case (k, v) => sparkConf.set(k, v) }

          // set a random app name if not given.
          if (!sparkConf.contains("spark.app.name")) {
            sparkConf.setAppName(java.util.UUID.randomUUID().toString)
          }

          SparkContext.getOrCreate(sparkConf)
          // Do not update `SparkConf` for existing `SparkContext`, as it's shared by all sessions. 不要修改已有SparkContext的SparkConf，它被所有session共享
        }

        // Initialize extensions if the user has defined a configurator class.
        val extensionConfOption = sparkContext.conf.get(StaticSQLConf.SPARK_SESSION_EXTENSIONS)
        if (extensionConfOption.isDefined) {
          val extensionConfClassName = extensionConfOption.get
          try {
            val extensionConfClass = Utils.classForName(extensionConfClassName)
            val extensionConf = extensionConfClass.newInstance()
              .asInstanceOf[SparkSessionExtensions => Unit]
            extensionConf(extensions)
          } catch {
            // Ignore the error if we cannot find the class or when the class has the wrong type.
            case e @ (_: ClassCastException |
                      _: ClassNotFoundException |
                      _: NoClassDefFoundError) =>
              logWarning(s"Cannot use $extensionConfClassName to configure session extensions.", e)
          }
        }

        session = new SparkSession(sparkContext, None, None, extensions)
        options.foreach { case (k, v) => session.initialSessionOptions.put(k, v) }
        setDefaultSession(session)
        setActiveSession(session)

        // Register a successfully instantiated context to the singleton. This should be at the
        // end of the class definition so that the singleton is updated only if there is no
        // exception in the construction of the instance.
        sparkContext.addSparkListener(new SparkListener {
          override def onApplicationEnd(applicationEnd: SparkListenerApplicationEnd): Unit = {
            defaultSession.set(null)
          }
        })
      }

      return session
    }
```
根据 SparkConf 启动调度，提交应用
```
class SparkContext(config: SparkConf) :
  // 根据SparkConf中的各种属性（如spark.***），
  try {
    _conf = config.clone()
    _conf.validateSettings()
    
    ...... //各种验证相关等 

    // Set Spark driver host and port system properties. This explicitly sets the configuration
    // instead of relying on the default value of the config constant.
    _conf.set(DRIVER_HOST_ADDRESS, _conf.get(DRIVER_HOST_ADDRESS))
    _conf.setIfMissing("spark.driver.port", "0")

    _conf.set("spark.executor.id", SparkContext.DRIVER_IDENTIFIER)

    _jars = Utils.getUserJars(_conf)
    _files = _conf.getOption("spark.files").map(_.split(",")).map(_.filter(_.nonEmpty))
      .toSeq.flatten

    _eventLogDir = ...// conf.get("spark.eventLog.dir", EventLoggingListener.DEFAULT_LOG_DIR)

    _eventLogCodec = ...// _conf.getBoolean("spark.eventLog.compress", false)

    _listenerBus = new LiveListenerBus(_conf)

    // Initialize the app status store and listener before SparkEnv is created so that it gets all events.
    _statusStore = AppStatusStore.createLiveStore(conf)
    listenerBus.addToStatusQueue(_statusStore.listener.get)

    // 创建spark执行环境 Create the Spark execution environment (cache, map output tracker, etc)
    _env = createSparkEnv(_conf, isLocal, listenerBus)
    SparkEnv.set(_env)

    // If running the REPL, register the repl's output dir with the file server.
    _conf.getOption("spark.repl.class.outputDir").foreach { path =>
      val replUri = _env.rpcEnv.fileServer.addDirectory("/classes", new File(path))
      _conf.set("spark.repl.class.uri", replUri)
    }

    _statusTracker = new SparkStatusTracker(this, _statusStore)

    _progressBar // shows the progress of stages 显示stage进度

    _ui = SparkUI.create(Some(this), _statusStore, _conf, _env.securityManager, appName, "",
          startTime))
    // Bind the UI before starting the task scheduler to communicate
    // the bound port to the cluster manager properly
    _ui.foreach(_.bind())

    _hadoopConfiguration = SparkHadoopUtil.get.newConfiguration(_conf)

    // Add each JAR given through the constructor jars.foreach(addJar) files.foreach(addFile)

    _executorMemory = _conf.getOption("spark.executor.memory")
      .orElse(Option(System.getenv("SPARK_EXECUTOR_MEMORY")))
      .orElse(Option(System.getenv("SPARK_MEM"))
      .map(warnSparkMem))
      .map(Utils.memoryStringToMb)
      .getOrElse(1024)

    // Convert java options to env vars as a work around since we can't set env vars directly in sbt.
    // ......
    // The Mesos scheduler backend relies on this environment variable to set executor memory.
    // ......

    // We need to register "HeartbeatReceiver" before "createTaskScheduler" because Executor will retrieve "HeartbeatReceiver" in the constructor. (SPARK-6640)
    _heartbeatReceiver = env.rpcEnv.setupEndpoint(HeartbeatReceiver.ENDPOINT_NAME, new HeartbeatReceiver(this))

    // Create and start the scheduler 创建并启动调度
    val (sched, ts) = SparkContext.createTaskScheduler(this, master, deployMode)
    _schedulerBackend = sched
    _taskScheduler = ts
    _dagScheduler = new DAGScheduler(this)
    _heartbeatReceiver.ask[Boolean](TaskSchedulerIsSet)

    // start TaskScheduler after taskScheduler sets DAGScheduler reference in DAGScheduler's  constructor
    _taskScheduler.start()

    _applicationId = _taskScheduler.applicationId()
    _applicationAttemptId = taskScheduler.applicationAttemptId()
    _conf.set("spark.app.id", _applicationId)
    if (_conf.getBoolean("spark.ui.reverseProxy", false)) {
      System.setProperty("spark.ui.proxyBase", "/proxy/" + _applicationId)
    }
    _ui.foreach(_.setAppId(_applicationId))
    // 运行在每个节点（驱动程序和执行器）上的管理器，它提供接口，用于本地和远程将块放入各种存储（内存、磁盘和堆外）
    _env.blockManager.initialize(_applicationId)

     ......
    _env.metricsSystem.start()
    // Attach the driver metrics servlet handler to the web ui after the metrics system is started.
    _env.metricsSystem.getServletHandlers.foreach(handler => ui.foreach(_.attachHandler(handler)))

    _eventLogger = // isEventLogEnabled EventLoggingListener listenerBus.addToEventLogQueue(logger)

    // ExecutorAllocationManager : An agent that dynamically allocates and removes executors based on the workload.
    // ExecutorAllocationClient : A client that communicates with the cluster manager to request or kill executors. This is currently supported only in YARN mode.

     ......
    // 注册监听、发布环境和应用事件

    // 初始化成功后调用（通常在spark上下文中）。Yarn用它来引导基于首选位置的资源分配，等待slave注册等
    ......
  } catch {
	......
  }
```
SparkContext中创建调度 createTaskScheduler，以及用于调度的后端接口 SchedulerBackend   
yarn-cluster：client向RM(Yarn Resource Manager)申请一个Container来启动AM（ApplicationMaster）进程，SparkContext运行在AM进程中   
yarn-client：在提交节点上执行SparkContext初始化，由JavaMainApplication调用   
```
  /**
   * Create a task scheduler based on a given master URL.
   * Return a 2-tuple of the scheduler backend and the task scheduler.
   */
  private def createTaskScheduler(
      sc: SparkContext,
      master: String,
      deployMode: String): (SchedulerBackend, TaskScheduler) = {
    import SparkMasterRegex._

    // When running locally, don't try to re-execute tasks on failure.
    val MAX_LOCAL_TASK_FAILURES = 1

    master match {
      case "local" =>
        val scheduler = new TaskSchedulerImpl(sc, MAX_LOCAL_TASK_FAILURES, isLocal = true)
        val backend = new LocalSchedulerBackend(sc.getConf, scheduler, 1)
        scheduler.initialize(backend)
        (backend, scheduler)

      case LOCAL_N_REGEX(threads) =>
        def localCpuCount: Int = Runtime.getRuntime.availableProcessors()
        // local[*] estimates the number of cores on the machine; local[N] uses exactly N threads.
        val threadCount = if (threads == "*") localCpuCount else threads.toInt
        if (threadCount <= 0) {
          throw new SparkException(s"Asked to run locally with $threadCount threads")
        }
        val scheduler = new TaskSchedulerImpl(sc, MAX_LOCAL_TASK_FAILURES, isLocal = true)
        // LocalSchedulerBackend master都运行在本地同一个JVM中，只用一个executor
        val backend = new LocalSchedulerBackend(sc.getConf, scheduler, threadCount)
        scheduler.initialize(backend)
        (backend, scheduler)

      case LOCAL_N_FAILURES_REGEX(threads, maxFailures) =>
        def localCpuCount: Int = Runtime.getRuntime.availableProcessors()
        // local[*, M] means the number of cores on the computer with M failures
        // local[N, M] means exactly N threads with M failures
        val threadCount = if (threads == "*") localCpuCount else threads.toInt
        val scheduler = new TaskSchedulerImpl(sc, maxFailures.toInt, isLocal = true)
        val backend = new LocalSchedulerBackend(sc.getConf, scheduler, threadCount)
        scheduler.initialize(backend)
        (backend, scheduler)

      case SPARK_REGEX(sparkUrl) =>
        val scheduler = new TaskSchedulerImpl(sc)
        val masterUrls = sparkUrl.split(",").map("spark://" + _)
        // A [[SchedulerBackend]] implementation for Spark's standalone cluster manager.
        val backend = new StandaloneSchedulerBackend(scheduler, sc, masterUrls)
        scheduler.initialize(backend)
        (backend, scheduler)

      case LOCAL_CLUSTER_REGEX(numSlaves, coresPerSlave, memoryPerSlave) =>
        // Check to make sure memory requested <= memoryPerSlave. Otherwise Spark will just hang.
        val memoryPerSlaveInt = memoryPerSlave.toInt
        if (sc.executorMemory > memoryPerSlaveInt) {
          throw new SparkException(
            "Asked to launch cluster with %d MB RAM / worker but requested %d MB/worker".format(
              memoryPerSlaveInt, sc.executorMemory))
        }

        val scheduler = new TaskSchedulerImpl(sc)
        val localCluster = new LocalSparkCluster(
          numSlaves.toInt, coresPerSlave.toInt, memoryPerSlaveInt, sc.conf)
        val masterUrls = localCluster.start()
        // Create a scheduler backend for the given SparkContext and scheduler.
        val backend = new StandaloneSchedulerBackend(scheduler, sc, masterUrls)
        scheduler.initialize(backend)
        backend.shutdownCallback = (backend: StandaloneSchedulerBackend) => {
          localCluster.stop()
        }
        (backend, scheduler)

      case masterUrl =>
        val cm = getClusterManager(masterUrl) match {
          case Some(clusterMgr) => clusterMgr
          case None => throw new SparkException("Could not parse Master URL: '" + master + "'")
        }
        try {
          val scheduler = cm.createTaskScheduler(sc, masterUrl)
          // ClusterManager
          val backend = cm.createSchedulerBackend(sc, masterUrl, scheduler)
          cm.initialize(scheduler, backend)
          (backend, scheduler)
        } catch {
          case se: SparkException => throw se
          case NonFatal(e) =>
            throw new SparkException("External scheduler cannot be instantiated", e)
        }
    }
  }
```
根据不同的master创建scheduler和backend     
TaskSchedulerImpl 通过SchedulerBackend来为多种类型的集群调度任务,调度不同的job，接收被分配的资源来给每一个Task分配资源。它还可以通过使用“LocalSchedulerBackend”并将isLocal设置为true来使用本地设置。它处理常见的逻辑，如确定跨作业的调度顺序、唤醒启动推测性任务等。CAUTION:SPARK-31485   
SchedulerBackend负责获取资源，本地与ClusterManager等   
case masterUrl 除了local和standelone的外部资源管理方式，ExternalClusterManager各种实现：YarnClusterManager、MesosClusterManager、KubernetesClusterManager 等   
```
ExternalClusterManager：
canCreate(masterURL: String):Boolean  Create a task scheduler instance for the given SparkContext
createTaskScheduler(sc: SparkContext, masterURL: String):TaskScheduler  Create a task scheduler instance for the given SparkContext
createSchedulerBackend(sc: SparkContext,masterURL: String,scheduler: TaskScheduler): SchedulerBackend Create a scheduler backend for the given SparkContext and scheduler.
initialize(scheduler: TaskScheduler, backend: SchedulerBackend): Unit Initialize task scheduler and backend scheduler. 

YarnClusterManager：
  override def createTaskScheduler(sc: SparkContext, masterURL: String): TaskScheduler = {
    sc.deployMode match {
      case "cluster" => new YarnClusterScheduler(sc)
      case "client" => new YarnScheduler(sc)
      case _ => throw new SparkException(s"Unknown deploy mode '${sc.deployMode}' for Yarn")
    }
  }

  override def createSchedulerBackend(sc: SparkContext,
      masterURL: String,
      scheduler: TaskScheduler): SchedulerBackend = {
    sc.deployMode match {
      case "cluster" =>
        new YarnClusterSchedulerBackend(scheduler.asInstanceOf[TaskSchedulerImpl], sc)
      case "client" =>
        new YarnClientSchedulerBackend(scheduler.asInstanceOf[TaskSchedulerImpl], sc)
      case  _ =>
        throw new SparkException(s"Unknown deploy mode '${sc.deployMode}' for Yarn")
    }
  }
```
SparkContext 中 _taskScheduler.start()，TaskSchedulerImpl start() 中 backend.start()

StandaloneSchedulerBackend.start() The scheduler backend should only try to connect to the launcher when in client mode. In cluster mode, the code that submits the application to the Master needs to connect。使用--driver-url、sc.conf.getOption("spark.executor.extraClassPath")等参数启动 executor，StandaloneAppClient.start() Just launch an rpcEndpoint; it will call back into the listener.提交应用的过程到此完成

YarnClusterManager:

YarnClientSchedulerBackend.start() 创建 Yarn client 提交应用给 ResourceManager;SchedulerExtensionServiceBinding:The attempt ID will be set if the service is started within a YARN application master;there is then a different attempt ID for every time that AM is restarted.
SchedulerExtensionServices:Container for [[SchedulerExtensionService]] instances.sparkContext = binding.sparkContext...instance.start(binding)...
```
 /**
   * Create a Yarn client to submit an application to the ResourceManager.
   * This waits until the application is running.
   */
  override def start() {
    val driverHost = conf.get("spark.driver.host")
    val driverPort = conf.get("spark.driver.port")
    val hostport = driverHost + ":" + driverPort
    sc.ui.foreach { ui => conf.set("spark.driver.appUIAddress", ui.webUrl) }
    ......
    totalExpectedExecutors = SchedulerBackendUtils.getInitialTargetExecutorNumber(conf)
    client = new Client(args, conf)
    // submitApplication上面贴过内容了,那个是cluster模式的
    bindToYarn(client.submitApplication(), None)

    // SPARK-8687: Ensure all necessary properties have already been set before we initialize our driver scheduler backend, which serves these properties to the executors
    super.start()
    waitForApplication()

    monitorThread = asyncMonitorApplication()
    monitorThread.start()
  }
```
YarnClusterSchedulerBackend.start()
```
  override def start() {
    val attemptId = ApplicationMaster.getAttemptId
    bindToYarn(attemptId.getApplicationId(), Some(attemptId))
    super.start()
    totalExpectedExecutors = SchedulerBackendUtils.getInitialTargetExecutorNumber(sc.conf)
  }
```
yarnClient.submitApplication 结束后就提交到yarn集群了，客户端的提交就结束了。
yarn-cluster模式下，driver运行在AM(Application Master)中，它负责向YARN申请资源，并监督作业的运行状况。当用户提交了作业之后，就可以关掉Client，作业会继续在YARN上运行。
yarn-cluster模式Application Master仅向YARN请求executor，client会和请求的container通信来调度他们工作，Client不能关

-----

 [Edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Spark-Submit.md)  

