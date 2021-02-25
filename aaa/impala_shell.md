which impala-shell   
是个没有.sh 的 shell 文件   
```
echo ${SHELL_HOME}
echo "${EGG_PATH}${SHELL_HOME}/gen-py:${SHELL_HOME}/lib:${PYTHONPATH}"

/data/xxx/cloudera/parcels/CDH-5.14.4-1.cdh5.14.4.p0.3/bin/../lib/impala-shell
/data/xxx/cloudera/parcels/CDH-5.14.4-1.cdh5.14.4.p0.3/bin/../lib/impala-shell/ext-py/sqlparse-0.1.14-py2.7.egg:/data/xxx/cloudera/parcels/CDH-5.14.4-1.cdh5.14.4.p0.3/bin/../lib/impala-shell/ext-py/sasl-0.1.1-py2.7-linux-x86_64.egg:/data/xxx/cloudera/parcels/CDH-5.14.4-1.cdh5.14.4.p0.3/bin/../lib/impala-shell/ext-py/prettytable-0.7.1-py2.7.egg:/data/xxx/cloudera/parcels/CDH-5.14.4-1.cdh5.14.4.p0.3/bin/../lib/impala-shell/gen-py:/data/xxx/cloudera/parcels/CDH-5.14.4-1.cdh5.14.4.p0.3/bin/../lib/impala-shell/lib::/home/xxx/sp/pycommon:/home/xxx/sr/libs:/home/xxx/sp/pycommon:/home/xxx/sr/libs
```

/data/xxx/cloudera/parcels/CDH-5.14.4-1.cdh5.14.4.p0.3/lib/impala-shell/
[impala_shell.py](https://github.com/cloudera/Impala/blob/cdh6.3.0/shell/impala_shell.py)

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
```

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
用命令行的设置盖掉 config file 的 query_options ，以及拼加欢迎信息：
```
  query_options.update(
     [(k.upper(), v) for k, v in parse_variables(options.query_options).items()])

  if options.query or options.query_file:
    if options.print_progress or options.print_summary:
      print_to_stderr("Error: Live reporting is available for interactive mode only.")
      sys.exit(1)

    execute_queries_non_interactive_mode(options, query_options)
    sys.exit(0)

  intro = WELCOME_STRING
  if not options.ssl and options.creds_ok_in_clear and options.use_ldap:
    intro += ("\n\nLDAP authentication is enabled, but the connection to Impala is "
              "not secured by TLS.\nALL PASSWORDS WILL BE SENT IN THE CLEAR TO IMPALA.\n")

  if options.refresh_after_connect:
    intro += REFRESH_AFTER_CONNECT_DEPRECATION_WARNING

  shell = ImpalaShell(options, query_options)
  while shell.is_alive:
    try:    # 套了两层 try，外层只是清空介绍信息
      try:
        shell.cmdloop(intro)
      except KeyboardInterrupt:
        intro = '\n'
      # 最后捕获未在 shell 中捕获的 rpc 抛出的异常
      except socket.error, (code, e):
        # if the socket was interrupted, reconnect the connection with the client
        if code == errno.EINTR:
          print shell.CANCELLATION_MESSAGE
          shell._reconnect_cancellation()
        else:
          print_to_stderr("Socket error %s: %s" % (code, e))
          shell.imp_client.connected = False
          shell.prompt = shell.DISCONNECTED_PROMPT
      except DisconnectedException, e:
        # 如果 socket 中断，重新与客户端建立连接
        print_to_stderr(e)
        shell.imp_client.connected = False
        shell.prompt = shell.DISCONNECTED_PROMPT
      except QueryStateException, e:
        # 执行查询时出现异常
        shell.imp_client.close_query(shell.last_query_handle,
                                     shell.query_handle_closed)
        print_to_stderr(e)
      except RPCException, e:
        # could not complete the rpc successfully
        print_to_stderr(e)
      except IOError, e:
        # 忽略系统中断(例如因为取消)造成的异常
        if e.errno != errno.EINTR: raise
    finally:
      intro = ''
```
ImpalaShell:
```
基本的用法 connect <host:port> 连接 impalad. Tab自动显示可用命令.实现shell命令的方法返回a boolean tuple (stop, status)，其中stop是用于提示命令循环是否继续的标识；status 同志调用者成功完成执行。
class ImpalaShell(object, cmd.Cmd): 
  # 如果没连接上 impalad
  UNKNOWN_SERVER_VERSION = "Not Connected"
  DISCONNECTED_PROMPT = "[Not connected] > "
  UNKNOWN_WEBSERVER = "0.0.0.0"
  
  # Number of times to attempt cancellation before giving up.
  CANCELLATION_TRIES = 3
  # Commands are terminated with the following delimiter.
  CMD_DELIM = ';'
  # 有效的变量名格式
  VALID_VAR_NAME_PATTERN = r'[A-Za-z][A-Za-z0-9_]*'
  # SET命令前面会被移除掉的注释的匹配模式
  COMMENTS_BEFORE_SET_PATTERN = r'^(\s*/\*(.|\n)*?\*/|\s*--.*\n)*\s*((un)?set)'
  COMMENTS_BEFORE_SET_REPLACEMENT = r'\3'
  
  # 变量名前缀
  VAR_PREFIXES = [ 'VAR', 'HIVEVAR' ]
  DEFAULT_DB = 'default'
  # 匹配 DML .
  DML_REGEX = re.compile("^(insert|upsert|update|delete)$", re.I)
  # 用于查询历史文件的分隔府.
  HISTORY_FILE_QUERY_DELIM = '_IMP_DELIM_'
  # true 的四种写法 "true", "TRUE", "True", "1"
  VALID_SHELL_OPTIONS = {
    'LIVE_PROGRESS' : (lambda x: x in ("true", "TRUE", "True", "1"), "print_progress"),
    'LIVE_SUMMARY' : (lambda x: x in ("true", "TRUE", "True", "1"), "print_summary")
  }
  # 两次执行 summary 之间的最小时间间隔，单位秒.
  PROGRESS_UPDATE_INTERVAL = 1.0
  
  def __init__(self, options, query_options):
    ... 各种初始化值
    self.current_db = options.default_db
    self.history_file = os.path.expanduser("~/.impalahistory")
    # 遇到分隔符前保存用户输入状态
    self.partial_cmd = str()
    # 用户输入不完整时，保存旧的状态
    self.cached_prompt = str()
    ...
    self.progress_stream = OverwritingStdErrOutputStream()      # from shell_output import OverwritingStdErrOutputStream
    ...
    self._populate_command_list()  # 填充命令列表，每个命令 command 对应的方法名 do_<command>，可以在类目录中找到   self.commands = [cmd[3:] for cmd in dir(self.__class__) if cmd.startswith('do_')]
    ...
    # 由于 readline 在 centos/rhel7 中有bug, 会导致控制字符的print。这会破坏非交互模式下的任何shell脚本的编写。因为非交互模式下不需要 readline - do not import it。
    if options.query or options.query_file:
      self.interactive = False
      self._disable_readline()      #  self.readline = None 设置 readline 模块不可用.readline 模块用于跟踪命令的历史记录
    else:
      self.interactive = True
      try:
        self.readline = __import__('readline')
        try:
          self.readline.set_history_length(int(options.history_max))
        except ValueError:
          print_to_stderr("WARNING: history_max option malformed %s\n"
            % options.history_max)
          self.readline.set_history_length(1000)
      except ImportError:
        self._disable_readline()
    ...
    # impala_shell 自己处理 Ctrl-C, 在 handler 和 main shell 之间使用一个事件对象发送取消请求的信号
    signal.signal(signal.SIGINT, self._signal_handler)
  
  def do_shell(self, args):
  def __do_dml(self, args): # Executes a DML query
  # 各种 do_<command>：
  def do_summary(self, args):
  def do_set(self, args):
  def do_unset(self, args):
  def do_quit(self, args):
  def do_exit(self, args):
  def do_connect(self, args):  
    # 位指定连接串时默认连接本地impalad.连接kerberized impalad需指定全限定域名 fqdn 作主机名
    # from impala_client import (ImpalaClient, DisconnectedException, QueryStateException, RPCException, TApplicationException) 
    # (./lib/impala_client.py) 参见:https://github.com/cloudera/Impala/blob/cdh6.3.0/shell/impala_client.py
    # 初始化 imp_client self.imp_client = self._new_impala_client()
    
  def do_alter(self, args):
  def do_create(self, args):
  def do_drop(self, args):
  def do_load(self, args):
  def do_profile(self, args):
  
  def do_select(self, args):
    # 执行 select 查询，获取所有返回行， args：例如 count(*) from db.table
    query = self._create_beeswax_query(args)   # query BeeswaxService.Query() 例如: Query(query='select count(*) from db.table', configuration=[], hadoop_user='aaa')
    return self._execute_stmt(query, print_web_link=True)

  def do_compute(self, args):
  def do_values(self, args):  # Executes a VALUES(...) query, fetching all rows
  def do_with(self, args):
  def do_use(self, args):
  def do_show(self, args):
  def do_describe(self, args):
  def do_desc(self, args):
  def do_upsert(self, args):  # def __do_dml(self, args):
  def do_update(self, args):  # def __do_dml(self, args):
  def do_delete(self, args):  # def __do_dml(self, args):
  def do_insert(self, args):  # def __do_dml(self, args):
  def do_explain(self, args):
  def do_history(self, args):
  def do_rerun(self, args):  # Rerun a command with an command index in history Example: @1;
  def do_tip(self, args):
  def do_src(self, args):
  def do_source(self, args):
  def do_version(self, args):
  
  
  # 各种检查清理 input 的方法
  def _remove_comments_before_set(self, line):
  def sanitise_input(self, args):
  def _shlex_split(self, line):
  def _cmd_ends_with_delim(self, line):
  def _check_for_command_completion(self, cmd):
  
  
  # 脚手架？
  def __do_dml(self, args):
    """Executes a DML query"""
    query = self._create_beeswax_query(args)
    return self._execute_stmt(query, is_dml=True, print_web_link=True)

  def _print_options(self, print_mode):
  def _get_query_option_grouping(self):
  def _print_option_group(self, query_options):
  def _print_variables(self):
  def _print_shell_options(self):
  def _create_beeswax_query(self, args):
    # 运行前保存命令，通常用于 do_* 方法，保存在 precmd() 
    # self.imp_client.create_beeswax_query("%s %s" % (command, args), self.set_query_options)
    # imp_client.create_beeswax_query 参见 do_connect
    
  def _new_impala_client(self):
  def _signal_handler(self, signal, frame):
  def _replace_variables(self, query): 用对应的值替换查询语句中的变量
  def precmd(self, args):
  def onecmd(self, line):
  def postcmd(self, status, args): status向shell传递shell应该如何继续执行，并且始终为CmdStatus
  def _handle_shell_options(self, token, value):
  def _get_var_name(self, name):
  def _print_with_set(self, print_level):
  def _connect(self):
  def _reconnect_cancellation(self):
  def _validate_database(self, immediately=False):
  def _print_if_verbose(self, message):
  def print_runtime_profile(self, profile, status=False):
  def _parse_table_name_arg(self, arg):
  def _format_outputstream(self):
  def _periodic_wait_callback(self):
  
  def _default_summary_table(self):     # self.construct_table_with_header(["Operator", "#Hosts", "Avg Time", "Max Time","#Rows", "Est. #Rows", "Peak Mem","Est. Peak Mem", "Detail"]
  def _execute_stmt(self, query, is_dml=False, print_web_link=False):
    # 执行查询逻辑。客户端执行查询，在开始执行的同时返回query_handle。如果查询不是dml，当结果流式输入时，通过使用生成器从客户端获取结果。打印执行时间，如果执行未完成，关闭查询?。The execution time is printed and the query is closed if it hasn't been already
    self.last_query_handle = self.imp_client.execute_query(query)     # imp_service.query(query) 参见结尾处;ImpalaService.Client(protocol)  ImpalaService 代码由thrift 生成，可以在 gen-py 下找到，它的 Client 继承了 beeswaxd.BeeswaxService.Client, Iface，BeeswaxService 同样由 thrift 生成，参见结尾处。
    self.query_handle_closed = False
      self.last_summary = time.time()
      if print_web_link:
        self._print_if_verbose(
            "Query progress can be monitored at: %s/query_plan?query_id=%s" % (self.webserver_address, self.last_query_handle.id)) # 例如 Query progress can be monitored at: http://xxxxxxx:25000/query_plan?query_id=e0485597184609cd:7283bcb600000000
      wait_to_finish = self.imp_client.wait_to_finish(self.last_query_handle, self._periodic_wait_callback)
      # Reset 进度 stream.
      self.progress_stream.clear()
      
      if is_dml:
        # retrieve the error log
        warning_log = self.imp_client.get_warning_log(self.last_query_handle)
        (num_rows, num_row_errors) = self.imp_client.close_dml(self.last_query_handle)
      else:
        # impalad 不支持某型查询类型的元数据获取
        if not self.imp_client.expect_result_metadata(query.query):
          # Close the query
          self.imp_client.close_query(self.last_query_handle)
          self.query_handle_closed = True
          return CmdStatus.SUCCESS

        self._format_outputstream()
        # fetch 返回一个生成器
        rows_fetched = self.imp_client.fetch(self.last_query_handle)
        num_rows = 0

        for rows in rows_fetched:
          # IMPALA-4418: 中断循环以防止打印不必要的空行。
          if len(rows) == 0:
            break
          self.output_stream.write(rows)
          num_rows += len(rows)
          
        warning_log = self.imp_client.get_warning_log(self.last_query_handle)

      end_time = time.time()




  def construct_table_with_header(self, column_names):
  def preloop(self):
  def postloop(self):
  def parseline(self, line):
  def _replace_history_delimiters(self, src_delim, tgt_delim):
  def default(self, args):
  def emptyline(self):
  def completenames(self, text, *ignored):
  def execute_query_list(self, queries):
```









```
# Tarball / packaging build makes impala_build_version available
try:
  from impala_build_version import get_git_hash, get_build_date, get_version
  VERSION_STRING = VERSION_FORMAT % {'version': get_version(),
                                     'git_hash': get_git_hash()[:7],
                                     'build_date': get_build_date()}
except Exception:
  pass

class CmdStatus:
  """Values indicate the execution status of a command to the cmd shell driver module
  SUCCESS and ERROR continue running the shell and ABORT exits the shell
  Since SUCCESS == None, successful commands do not need to explicitly return
  anything on completion
  """
  SUCCESS = None
  ABORT = True
  ERROR = False

class ImpalaPrettyTable(prettytable.PrettyTable):
  """Patched version of PrettyTable that TODO"""
  def _unicode(self, value):
    if not isinstance(value, basestring):
      value = str(value)
    if not isinstance(value, unicode):
      # If a value cannot be encoded, replace it with a placeholder.
      value = unicode(value, self.encoding, "replace")
    return value

class QueryOptionLevels:
  """These are the levels used when displaying query options.
  The values correspond to the ones in TQueryOptionLevel"""
  REGULAR = 0
  ADVANCED = 1
  DEVELOPMENT = 2
  DEPRECATED = 3

class QueryOptionDisplayModes:
  REGULAR_OPTIONS_ONLY = 1
  ALL_OPTIONS = 2
```







----
[impala_client.py](https://github.com/cloudera/Impala/blob/cdh6.3.0/shell/impala_client.py)
```
from beeswaxd import BeeswaxService

  def create_beeswax_query(self, query_str, set_query_options):
    """Create a beeswax query object from a query string"""
    query = BeeswaxService.Query()
    query.hadoop_user = self.user
    query.query = query_str
    query.configuration = self._options_to_string_list(set_query_options)
    return query

  def execute_query(self, query):
    self.is_query_cancelled = False
    rpc_result = self._do_rpc(lambda: self.imp_service.query(query))
    last_query_handle, status = rpc_result
    if status != RpcStatus.OK:
      raise RPCException("Error executing the query")
    return last_query_handle

  def connect(self):
    # 创建到 Impalad 实例的连接，然后ping impala service的实例测试连接是否成功，并获取服务器版本(ps -aux | grep impalad，impalad-main.cc)
    ......
    sock, self.transport = self._get_socket_and_transport()
    ......
    protocol = TBinaryProtocol.TBinaryProtocol(self.transport)
    self.imp_service = ImpalaService.Client(protocol)     # BeeswaxService.py
    result = self.ping_impala_service()
    self.connected = True
    return result

  def wait_to_finish(self, last_query_handle, periodic_callback=None):
    loop_start = time.time()
    while True:
      query_state = self.get_query_state(last_query_handle)
      if query_state == self.query_state["FINISHED"]:
        break
      elif query_state == self.query_state["EXCEPTION"]:
        if self.connected:
          raise QueryStateException(self.get_error_log(last_query_handle))
        else:
          raise DisconnectedException("Not connected to impalad.")

      if periodic_callback is not None: periodic_callback()
      time.sleep(self._get_sleep_interval(loop_start))
```
BeeswaxService.py
```
class Client(Iface):
  def query(self, query):
    # 提交查询并返回句柄(QueryHandle)，异步运行
    self.send_query(query)
    return self.recv_query()

  def send_query(self, query):
    self._oprot.writeMessageBegin('query', TMessageType.CALL, self._seqid) # _oprot：impala_client.py : protocol = TBinaryProtocol.TBinaryProtocol(self.transport)
    args = query_args()
    args.query = query
    args.write(self._oprot)
    self._oprot.writeMessageEnd()
    self._oprot.trans.flush()

  def recv_query(self, ):
    (fname, mtype, rseqid) = self._iprot.readMessageBegin()
    if mtype == TMessageType.EXCEPTION:
      x = TApplicationException()
      x.read(self._iprot)
      self._iprot.readMessageEnd()
      raise x
    result = query_result()
    result.read(self._iprot)
    self._iprot.readMessageEnd()
    if result.success is not None:
      return result.success
    if result.error is not None:
      raise result.error
    raise TApplicationException(TApplicationException.MISSING_RESULT, "query failed: unknown result");

class query_args:
  def write(self, oprot):
    if oprot.__class__ == TBinaryProtocol.TBinaryProtocolAccelerated and self.thrift_spec is not None and fastbinary is not None:
      oprot.trans.write(fastbinary.encode_binary(self, (self.__class__, self.thrift_spec)))
      return
    oprot.writeStructBegin('query_args')
    if self.query is not None:
      oprot.writeFieldBegin('query', TType.STRUCT, 1)
      self.query.write(oprot)
      oprot.writeFieldEnd()
    oprot.writeFieldStop()
    oprot.writeStructEnd()

class query_result:
  """
  Attributes:
   - success
   - error
  """

  thrift_spec = (
    (0, TType.STRUCT, 'success', (QueryHandle, QueryHandle.thrift_spec), None, ), # 0
    (1, TType.STRUCT, 'error', (BeeswaxException, BeeswaxException.thrift_spec), None, ), # 1
  )

  def __init__(self, success=None, error=None,):
    self.success = success
    self.error = error

  def read(self, iprot):
    if iprot.__class__ == TBinaryProtocol.TBinaryProtocolAccelerated and isinstance(iprot.trans, TTransport.CReadableTransport) and self.thrift_spec is not None and fastbinary is not None:
      fastbinary.decode_binary(self, iprot.trans, (self.__class__, self.thrift_spec))
      return
    iprot.readStructBegin()
    while True:
      (fname, ftype, fid) = iprot.readFieldBegin()
      if ftype == TType.STOP:
        break
      if fid == 0:
        if ftype == TType.STRUCT:
          self.success = QueryHandle()
          self.success.read(iprot)
        else:
          iprot.skip(ftype)
      elif fid == 1:
        if ftype == TType.STRUCT:
          self.error = BeeswaxException()
          self.error.read(iprot)
        else:
          iprot.skip(ftype)
      else:
        iprot.skip(ftype)
      iprot.readFieldEnd()
    iprot.readStructEnd()

  def write(self, oprot):
    if oprot.__class__ == TBinaryProtocol.TBinaryProtocolAccelerated and self.thrift_spec is not None and fastbinary is not None:
      oprot.trans.write(fastbinary.encode_binary(self, (self.__class__, self.thrift_spec)))
      return
    oprot.writeStructBegin('query_result')
    if self.success is not None:
      oprot.writeFieldBegin('success', TType.STRUCT, 0)
      self.success.write(oprot)
      oprot.writeFieldEnd()
    if self.error is not None:
      oprot.writeFieldBegin('error', TType.STRUCT, 1)
      self.error.write(oprot)
      oprot.writeFieldEnd()
    oprot.writeFieldStop()
    oprot.writeStructEnd()

```


----
[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/impala_shell.md)
