which impala-shell

/data/xxx/cloudera/parcels/CDH/bin/impala-shell

```
echo ${SHELL_HOME}
echo "${EGG_PATH}${SHELL_HOME}/gen-py:${SHELL_HOME}/lib:${PYTHONPATH}"

/data/xxx/cloudera/parcels/CDH-5.14.4-1.cdh5.14.4.p0.3/bin/../lib/impala-shell
/data/xxx/cloudera/parcels/CDH-5.14.4-1.cdh5.14.4.p0.3/bin/../lib/impala-shell/ext-py/sqlparse-0.1.14-py2.7.egg:/data/xxx/cloudera/parcels/CDH-5.14.4-1.cdh5.14.4.p0.3/bin/../lib/impala-shell/ext-py/sasl-0.1.1-py2.7-linux-x86_64.egg:/data/xxx/cloudera/parcels/CDH-5.14.4-1.cdh5.14.4.p0.3/bin/../lib/impala-shell/ext-py/prettytable-0.7.1-py2.7.egg:/data/xxx/cloudera/parcels/CDH-5.14.4-1.cdh5.14.4.p0.3/bin/../lib/impala-shell/gen-py:/data/xxx/cloudera/parcels/CDH-5.14.4-1.cdh5.14.4.p0.3/bin/../lib/impala-shell/lib::/home/xxx/sp/pycommon:/home/xxx/sr/libs:/home/xxx/sp/pycommon:/home/xxx/sr/libs
```


/data/sa_cluster/cloudera/parcels/CDH-5.14.4-1.cdh5.14.4.p0.3/lib/impala-shell

impala_shell.py

```
TIPS=[
  "Press TAB twice to see a list of available commands.",
  
  "After running a query, type SUMMARY to see a summary of where time was spent.",
  
  "The SET command shows the current value of all shell and query options.",SET 查看所有 shell 和 查询可选设置的当前值
  
  "To see live updates on a query's progress, run 'set LIVE_SUMMARY=1;'.", 查看一个查询的进度
  
  "To see a summary of a query's progress that updates in real-time, run 'set LIVE_PROGRESS=1;'.", 一个查询的实时进度概要
  
  "The HISTORY command lists all shell commands in chronological order.",
  
  "The '-B' command line flag turns off pretty-printing for query results. Use this flag to remove formatting from results you want to save for later, or to benchmark Impala.", 去除结果中的格式化

  "You can run a single query from the command line using the '-q' option.",
  
  "When pretty-printing is disabled, you can use the '--output_delimiter' flag to set the delimiter for fields in the same row. The default is ','.", 移除格式后的结果中字段的分隔府，默认逗号
  
  "Run the PROFILE command after a query has finished to see a comprehensive summary of all the performance and diagnostic information that Impala gathered for that query. Be warned, it can be very long!",
  
  "To see more tips, run the TIP command.",
  
  "Every command must be terminated by a ';'.",
  
  "Want to know what version of Impala you're connected to? Run the VERSION command to find out!",
  
  "You can change the Impala daemon that you're connected to by using the CONNECT command."
  
  "To see how Impala will plan to run your query without actually executing it, use the EXPLAIN command. You can change the level of detail in the EXPLAIN output by setting the EXPLAIN_LEVEL query option.", 可以修改 EXPLAIN_LEVEL 设置级别
  
  "When you set a query option it lasts for the duration of the Impala shell session." 以上设置作用于当前会话
  ]
```
解析查询语句，提取为utf-8；"""Parse query file text to extract queries and encode into utf-8"""：
```
def parse_query_text(query_text, utf8_encode_policy='strict'):
  
  query_list = [q.encode('utf-8', utf8_encode_policy) for q in sqlparse.split(query_text)]
  # 清除注释，因为 sqlparse 只支持 识别 sql 语句前面的注释，后面的会当作 sql 一起执行
  if query_list and not sqlparse.format(query_list[-1], strip_comments=True).strip("\n"):
    query_list.pop()
  return query_list
```
解析命令行中做为参数传递的变量：
```
def parse_variables(keyvals):
```
非交互模式执行
```
def execute_queries_non_interactive_mode(options, query_options):
```
开始执行：
```
if __name__ == "__main__":
  """
  shell 和查询两类参数设置都可以在命令行设置，命令行设置的值优先级高于配置文件的(.impalarc). shell 默认在 impala_shell_config_defaults.py. 查询的默认在服务器端，它可以被 impala-shell 的  'set' 命令改写.
  """
  ```
读配置参数：
```
from impala_shell_config_defaults import impala_shell_defaults
from option_parser import get_option_parser, get_config_from_file

  # 默认配置
  parser = get_option_parser(impala_shell_defaults)
  options, args = parser.parse_args()
  # 用户指定配置文件 in config_file option
  user_config = os.path.expanduser(options.config_file);
  # by default, use the .impalarc in the home directory
  config_to_load = impala_shell_defaults.get("config_file")

  # verify user_config, if found
  if os.path.isfile(user_config) and user_config != config_to_load:
    if options.verbose:
      print_to_stderr("Loading in options from config file: %s \n" % user_config)
    # Command line overrides loading ~/.impalarc
    config_to_load = user_config
  elif user_config != config_to_load:
    print_to_stderr('%s not found.\n' % user_config)
    sys.exit(1)

  # 加载默认 impala_shell_config_defaults.py
  # 覆盖 in config file
  try:
    loaded_shell_options, query_options = get_config_from_file(config_to_load)
    impala_shell_defaults.update(loaded_shell_options)
  except Exception, e:
    print_to_stderr(e)
    sys.exit(1)

  parser = get_option_parser(impala_shell_defaults)
  options, args = parser.parse_args()
``
一堆参数验证：
```
  if len(args) > 0:
    print_to_stderr('Error, could not parse arguments "%s"' % (' ').join(args))
    parser.print_help()
    sys.exit(1)

  if options.version:
    print VERSION_STRING
    sys.exit(0)

  if options.use_kerberos and options.use_ldap:
    print_to_stderr("Please specify at most one authentication mechanism (-k or -l)")
    sys.exit(1)

  if not options.ssl and not options.creds_ok_in_clear and options.use_ldap:
    print_to_stderr("LDAP credentials may not be sent over insecure " +
                    "connections. Enable SSL or set --auth_creds_ok_in_clear")
    sys.exit(1)

  if not options.use_ldap and options.ldap_password_cmd:
    print_to_stderr("Option --ldap_password_cmd requires using LDAP authentication " +
                    "mechanism (-l)")
    sys.exit(1)

  if options.use_kerberos:
    print_to_stderr("Starting Impala Shell using Kerberos authentication")
    print_to_stderr("Using service name '%s'" % options.kerberos_service_name)
    # Check if the user has a ticket in the credentials cache
    try:
      if call(['klist', '-s']) != 0:
        print_to_stderr(("-k requires a valid kerberos ticket but no valid kerberos "
                         "ticket found."))
        sys.exit(1)
    except OSError, e:
      print_to_stderr('klist not found on the system, install kerberos clients')
      sys.exit(1)
  elif options.use_ldap:
    print_to_stderr("Starting Impala Shell using LDAP-based authentication")
  else:
    print_to_stderr("Starting Impala Shell without Kerberos authentication")

  options.ldap_password = None
  if options.use_ldap and options.ldap_password_cmd:
    try:
      p = subprocess.Popen(shlex.split(options.ldap_password_cmd), stdout=subprocess.PIPE,
                           stderr=subprocess.PIPE)
      options.ldap_password, stderr = p.communicate()
      if p.returncode != 0:
        print_to_stderr("Error retrieving LDAP password (command was '%s', error was: "
                        "'%s')" % (options.ldap_password_cmd, stderr.strip()))
        sys.exit(1)
    except Exception, e:
      print_to_stderr("Error retrieving LDAP password (command was: '%s', exception "
                      "was: '%s')" % (options.ldap_password_cmd, e))
      sys.exit(1)

  if options.ssl:
    if options.ca_cert is None:
      print_to_stderr("SSL is enabled. Impala server certificates will NOT be verified"\
                      " (set --ca_cert to change)")
    else:
      print_to_stderr("SSL is enabled")

  if options.output_file:
    try:
      # Make sure the given file can be opened for writing. This will also clear the file
      # if successful.
      open(options.output_file, 'wb')
    except IOError, e:
      print_to_stderr('Error opening output file for writing: %s' % e)
      sys.exit(1)

  options.variables = parse_variables(options.keyval)
```



----
[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/impala_shell.md)
