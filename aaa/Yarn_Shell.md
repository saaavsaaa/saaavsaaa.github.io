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
        yarn rmadmin [-refreshQueues] [-refreshNodes] [-refreshNodesResources] [-refreshSuperUserGroupsConfiguration] [-refreshUserToGroupsMappings] [-refreshAdminAcls] [-refreshServiceAcl] [-getGroup [username]] [-addToClusterNodeLabels <"label1(exclusive=true),label2(exclusive=false),label3">] [-removeFromClusterNodeLabels <label1,label2,label3>] [-replaceLabelsOnNode <"node1[:port]=label1,label2 node2[:port]=label1">] [-directlyAccessNodeLabelStore]] [-updateNodeResource [NodeID] [MemSize] [vCores] ([OvercommitTimeout]) [-transitionToActive [--forceactive] <serviceId>] [-transitionToStandby <serviceId>] [-getServiceState <serviceId>] [-checkHealth <serviceId>] [-help [cmd]]   

           -refreshQueues: Reload the queues' acls, states and scheduler specific properties.
                        ResourceManager will reload the mapred-queues configuration file.
           -refreshNodes: Refresh the hosts information at the ResourceManager.
           -refreshNodesResources: Refresh resources of NodeManagers at the ResourceManager.
           -refreshSuperUserGroupsConfiguration: Refresh superuser proxy groups mappings
           -refreshUserToGroupsMappings: Refresh user-to-groups mappings
           -refreshAdminAcls: Refresh acls for administration of ResourceManager
           -refreshServiceAcl: Reload the service-level authorization policy file.
                        ResoureceManager will reload the authorization policy file.
           -getGroups [username]: Get the groups which given user belongs to.
           -addToClusterNodeLabels <"label1(exclusive=true),label2(exclusive=false),label3">: add to cluster node labels. Default exclusivity is true
           -removeFromClusterNodeLabels <label1,label2,label3> (label splitted by ","): remove from cluster node labels
           -replaceLabelsOnNode <"node1[:port]=label1,label2 node2[:port]=label1,label2">: replace labels on nodes (please note that we do not support specifying multiple labels on a single host for now.)
           -directlyAccessNodeLabelStore: This is DEPRECATED, will be removed in future releases. Directly access node label store, with this option, all node label related operations will not connect RM. Instead, they will access/modify stored node labels directly. By default, it is false (access via RM). AND PLEASE NOTE: if you configured yarn.node-labels.fs-store.root-dir to a local directory (instead of NFS or HDFS), this option will only work when the command run on the machine where RM is running.
           -updateNodeResource [NodeID] [MemSize] [vCores] ([OvercommitTimeout]): Update resource on specific node.
           -transitionToActive [--forceactive] <serviceId>: Transitions the service into Active state
           -transitionToStandby <serviceId>: Transitions the service into Standby state
           -getServiceState <serviceId>: Returns the state of the service
           -checkHealth <serviceId>: Requests that the service perform a health check.
        The HAAdmin tool will exit with a non-zero exit code
        if the check fails.
           -help [cmd]: Displays help for the given command or all commands if none is specified.

        Generic options supported are
        -conf <configuration file>     specify an application configuration file
        -D <property=value>            use value for given property
        -fs <local|namenode:port>      specify a namenode
        -jt <local|resourcemanager:port>    specify a ResourceManager
        -files <comma separated list of files>    specify comma separated files to be copied to the map reduce cluster
        -libjars <comma separated list of jars>    specify comma separated jar files to include in the classpath.
        -archives <comma separated list of archives>    specify comma separated archives to be unarchived on the compute machines.

        The general command line syntax is
        bin/hadoop command [genericOptions] [commandOptions]
    
    sharedcachemanager                    run the SharedCacheManager daemon   
    scmadmin                              SharedCacheManager admin tools   
    version                               print the version   
    jar <jar>                             run a jar file   
    application                           prints application(s)   
                                          report/kill application   

         -appStates <States>             Works with -list to filter applications based on input comma-separated list of   
                                         application states. The valid application state can be one of the following:   
                                         ALL,NEW,NEW_SAVING,SUBMITTED,ACCEPTED,RUNNING,FINISHED,FAILED,KILLED   
         -appTypes <Types>               Works with -list to filter applications based on input comma-separated list of   
                                         application types.   
         -help                           Displays help for all commands.   
         -kill <Application ID>          Kills the application.   
         -list                           List applications. Supports optional use of -appTypes to filter applications based   
                                         on application type, and -appStates to filter applications based on application state.   
         -movetoqueue <Application ID>   Moves the application to a different queue.   
         -queue <Queue Name>             Works with the movetoqueue command to
                                         specify which queue to move an
                                         application to.
         -status <Application ID>        Prints the status of the application.
   
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
