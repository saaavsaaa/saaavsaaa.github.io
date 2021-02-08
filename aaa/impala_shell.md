impala_shell.py

  "Press TAB twice to see a list of available commands.",
  "After running a query, type SUMMARY to see a summary of where time was spent.",
  "The SET command shows the current value of all shell and query options.",
  "To see live updates on a query's progress, run 'set LIVE_SUMMARY=1;'.",
  "To see a summary of a query's progress that updates in real-time, run 'set \
LIVE_PROGRESS=1;'.",
  "The HISTORY command lists all shell commands in chronological order.",
  "The '-B' command line flag turns off pretty-printing for query results. Use this flag \
to remove formatting from results you want to save for later, or to benchmark Impala.",
  "You can run a single query from the command line using the '-q' option.",
  "When pretty-printing is disabled, you can use the '--output_delimiter' flag to set \
the delimiter for fields in the same row. The default is ','.",
  "Run the PROFILE command after a query has finished to see a comprehensive summary of \
all the performance and diagnostic information that Impala gathered for that query. Be \
warned, it can be very long!",
  "To see more tips, run the TIP command.",
  "Every command must be terminated by a ';'.",
  "Want to know what version of Impala you're connected to? Run the VERSION command to \
find out!",
  "You can change the Impala daemon that you're connected to by using the CONNECT \
command."
  "To see how Impala will plan to run your query without actually executing it, use the \
EXPLAIN command. You can change the level of detail in the EXPLAIN output by setting the \
EXPLAIN_LEVEL query option.",
  "When you set a query option it lasts for the duration of the Impala shell session."
