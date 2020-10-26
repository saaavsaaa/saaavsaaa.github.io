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
```

