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
          scmadmin is the command to execute shared cache manageradministrative commands.
          The full syntax is:

          hadoop scmadmin [-runCleanerTask] [-help [cmd]]

          -runCleanerTask: Run cleaner task right away.

          -help [cmd]:    Displays help for the given command or all commands if none
                          is specified.


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
   
    applicationattempt                    prints applicationattempt(s) report   
          usage: applicationattempt
           -help                              Displays help for all commands.
           -list <Application ID>             List application attempts for
                                              aplication.
           -status <Application Attempt ID>   Prints the status of the application
                                              attempt.

    container                             prints container(s) report   
          usage: container
           -help                            Displays help for all commands.
           -list <Application Attempt ID>   List containers for application attempt.
           -status <Container ID>           Prints the status of the container.

    node                                  prints node report(s)   
          usage: node
           -all               Works with -list to list all nodes.
           -help              Displays help for all commands.
           -list              List all running nodes. Supports optional use of
                              -states to filter nodes based on node state, all -all
                              to list all nodes, -showDetails to display more
                              details about each node.
           -showDetails       Works with -list to show more details about each node.
           -states <States>   Works with -list to filter nodes based on input
                              comma-separated list of node states.
           -status <NodeId>   Prints the status report of the node.

    queue                                 prints queue information   
          usage: node
           -all               Works with -list to list all nodes.
           -help              Displays help for all commands.
           -list              List all running nodes. Supports optional use of
                              -states to filter nodes based on node state, all -all
                              to list all nodes, -showDetails to display more
                              details about each node.
           -showDetails       Works with -list to show more details about each node.
           -states <States>   Works with -list to filter nodes based on input
                              comma-separated list of node states.
           -status <NodeId>   Prints the status report of the node.
          [manager@SL010A-AnalysisDB1 ~]$ yarn queue -help
          19/10/08 15:28:22 INFO client.AHSProxy: Connecting to Application History server at sl010a-hopdb4/10.0.2.128:10200
          usage: queue
           -help                  Displays help for all commands.
           -status <Queue Name>   List queue information about given queue.

    logs                                  dump container logs   
          usage: yarn logs -applicationId <application ID> [OPTIONS]

          general options are:
           -am <AM Containers>                          Prints the AM Container logs for this application. Specify comma-separated value to get logs for related AM Container. For example, If we specify -am 1,2, we will get the logs for the first AM Container as well as the second AM Container. To get logs for all AM Containers, use -am ALL. To get logs for the latest AM Container, use -am -1. By default, it will print all available logs. Work with -log_files to get only specific logs.
           -appOwner <Application Owner>                AppOwner (assumed to be current user if not specified)
           -client_max_retries <Max Retries>            Set max retry number for a retry client to get the container logs for the running applications. Use a negative value to make retry forever. The default value is 30.
           -client_retry_interval_ms <Retry Interval>   Work with --client_max_retries to create a retry client. The default value is 1000.
           -containerId <Container ID>                  ContainerId. By default, it will print all available logs. Work with -log_files to get only specific logs. If specified, the applicationId can be omitted
           -help                                        Displays help for all commands.
           -list_nodes                                  Show the list of nodes that successfully aggregated logs. This option can only be used with finished applications.
           -log_files <Log File Name>                   Specify comma-separated value to get exact matched log files. Use "ALL" or "*" to fetch all the log files for the container.
           -log_files_pattern <Log File Pattern>        Specify comma-separated value to get matched log files by using java regex. Use ".*" to fetch all the log files for the container.
           -nodeAddress <Node Address>                  NodeAddress in the format nodename:port
           -out <Local Directory>                       Local directory for storing individual container logs. The container logs will be stored based on the node the container ran on.
           -show_application_log_info                   Show the containerIds which belong to the specific Application. You can combine this with --nodeAddress to get containerIds for all the containers on the specific NodeManager.
           -show_container_log_info                     Show the container log metadata, including log-file names, the size of the logfiles. You can combine thiwith --containerId to getlog metadata for thespecific container, or with--nodeAddress to get logmetadata for all thecontainers on the specific NodeManager.
           -size <size>                                 Prints the log file's first 'n' bytes or the last 'n' bytes. Use negative values as bytes to read from the end and positive values as bytes to read from the beginning.
           -size_limit_mb <Size Limit>                  Use this option to limit the size of the total logs which could be fetched. By default, we only allow to fetch at most 10240 MB logs. If the total log size is larger than the specified number, the CLI would fail. The user could specify -1 to ignore the size limit and fetch all logs.

    classpath                             prints the class path needed to   
                                          get the Hadoop jar and the   
                                          required libraries   
    cluster                               prints cluster information   
    daemonlog                             get/set the log level for each   
                                          daemon   
    envvars                               display computed Hadoop environment variables   
    top                                   run cluster usage tool   

