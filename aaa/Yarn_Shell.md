yarn -help
  Usage: yarn [--config confdir] [COMMAND | CLASSNAME]
    CLASSNAME                             run the class named CLASSNAME
   or
    where COMMAND is one of:
    resourcemanager                       run the ResourceManager
                                          Use -format-state-store for deleting the RMStateStore.
                                          Use -remove-application-from-state-store <appId> for
                                              removing application from RMStateStore.
    nodemanager                           run a nodemanager on each slave
    timelineserver                        run the timeline server
    rmadmin                               admin tools
    sharedcachemanager                    run the SharedCacheManager daemon
    scmadmin                              SharedCacheManager admin tools
    version                               print the version
    jar <jar>                             run a jar file
    application                           prints application(s)
                                          report/kill application
    applicationattempt                    prints applicationattempt(s)
                                          report
    container                             prints container(s) report
    node                                  prints node report(s)
    queue                                 prints queue information
    logs                                  dump container logs
    classpath                             prints the class path needed to
                                          get the Hadoop jar and the
                                          required libraries
    cluster                               prints cluster information
    daemonlog                             get/set the log level for each
                                          daemon
    envvars                               display computed Hadoop environment variables
    top                                   run cluster usage tool

  Most commands print help when invoked w/o parameters.
