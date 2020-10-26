which spark2-submit

exec $LIB_DIR/spark2/bin/spark-submit "$@"  

exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@"

package org.apache.spark.deploy

......

object SparkSubmit extends CommandLineUtils with Logging {
