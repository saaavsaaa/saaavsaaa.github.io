gdb ./ccccc       r(:run)p[option: args]      bt(:backtrace)     up/down(:step backtrace)/p arg(:print arg)        br filename:line_num (:set breakpoint)            n(:single step pass)   s(:one step in)       finish(:to the return)     list [option: line_number](:list code)         info locals(:local arg)    p var_(:print variable)      info breakpoints(:list all breakpoints)   delete breakpoints         break ... if ... (:condition interrupt)   until line(:go to line)   https://gcc.gnu.org/onlinedocs/gcc-5.1.0/gcc/Function-Specific-Option-Pragmas.html#Function-Specific-Option-Pragmas                  
gcore [-o filename] pid;gdb -c core_file program_binary;gdb:thread find thread_id (查找对应线程id在gdb中的线程编号)，thread id (跳到对应编号的线程)。gdb attach pid   
cat cols.txt |tr "\n" ","   |sed -e 's/,$/\n/'   
cat cols.txt |awk '{print $1}'|xargs |sed 's/ /,/g'                
pmap -d   
ps -mp xxxx -o THREAD   
jobs -l   
fg   
env   
sudo -i   
sudo passwd user   
find / -name *    
lsof -i:9999
netstat -antp   
df -h   
du -h --max-depth=1   
cat /proc/cpuinfo   
g++ -std=c++11 test_lock.cc -o test_lock -lpthread   
mkdir   
rm -rf   
mv   
tail -f   
ps -ef    
sftp    
sed 替换全文(g)时，第一个匹配替换完，是从下一个字符开始的，例如sed -e 's/⊙datetime⊙/⊙timestamp⊙/ig'只能替换⊙datetime⊙datetime⊙的第一个，要想两个都替换，需要⊙datetime⊙⊙datetime⊙ （i忽略大小写）   

watch -n 1 cat /proc/net/dev   
watch -n 1 "ifconfig eth0"   
```
rs=`cat /proc/net/dev | grep "em1" | sed 's/:/ /g' | awk '{print $2" "$10}'`
p_rec=`echo ${rs} | awk '{print $1}'`
p_send=`echo ${rs} | awk '{print $2}'`
sleep 1
cat /proc/net/dev | grep "em1" | sed 's/:/ /g' | awk  -v p_rec="${p_rec}" -v p_send="${p_send}" 'BEGIN{rec=0;send=0;} {rec=$2-p_rec;send=$10-p_send;if(rec/1024>1024){rec=rec/1048576 "MB/s"}else{rec=rec/1024 "KB/s"};if(send/1024>1024){send=send/1048576 "MB/s"}else{send=send/1024 "KB/s"}} END{print "rec:"rec",   send:"send;}' 
```
cat /etc/passwd   

jstack -l pid | grep -a40 "locked.\*T4CStatement" | grep -a2 RUNNABL | grep nid   
ps -Lf pid |wc -l   
netstat -an | grep 8080 |awk '{count[$6]++} END{for (i in count) print(i,count[i])}'   

hadoop fs -du -s -h   
hadoop fs -df -h   

sudo apt --fix-broken install     
chmod +x      
ifconfig   
chmod 755     
audacity     
grep -a 二进制文件   
grep ^aaa  以aaa开头的行   
grep aaa$  以aaa结尾的行   
grep aaa[^a] 除aaaa外aaa* ： aaab,aaac...   
sox in.wav out.wav speed 2.0 两倍速     

cat /etc/sysctl.conf可修改   
cat /proc/sys/fs/file-nr   
cat /proc/sys/fs/file-max   
sqoop import -D mapreduce.map.memory.mb=40960  -Dmapreduce.map.cpu.vcores=10

conda install --file requirements.txt
conda search
conda install package=version 
conda install -c conda package=version -y

comm 文件对比，合并等    
git fetch origin   
git diff HEAD FETCH_HEAD   

git rm --cached *.iml     
git rm -r --cached */.idea/*     
git rm -r --cached target/   
git rm -f --cached .idea/*   

set fileformat=unix   

git remote add upstream http://xxxxxxxxxxxxxxxxxx
git fetch upstream
git merge upstream/master

git commit
git push origin master

git log -n 1 --stat -p

git fetch
git merge

git fetch --all
git reset --hard origin/master

git stash
git stash list
git pull
git stash pop stash@{0}

grep error -m 10 === grep error  | head -10   
grep -v 排除   
grep -r "DataSourceActionInterceptor" *   
cat /proc/(pid)/status   
top命令下按f键   
sysctl -a | grep fs.file-max最大文件描述符   
ulimit -n查看单进程最大文件描述符   
ulimit -n d更改为d   
ulimit -a查看限制   
lsof查看当前哪些进程打开哪些句柄，哪些文件被哪些进程占用   

pmap -d    

tail -fn 500 nohup.out     
chown -R jenkins:jenkins /var/build/

sar -d 10 3 –p
怀疑CPU存在瓶颈，可用 sar -u 和 sar -q 等来查看   
怀疑内存存在瓶颈，可用 sar -B、sar -r 和 sar -W 等来查看   
怀疑I/O存在瓶颈，可用 sar -b、sar -u 和 sar -d 等来查看   
-A：所有报告的总和  
-u：输出CPU使用情况的统计信息   
-v：输出inode、文件和其他内核表的统计信息   
-d：输出每一个块设备的活动信息   
-r：输出内存和交换空间的统计信息   
-b：显示I/O和传送速率的统计信息   
-a：文件读写情况   
-c：输出进程统计信息，每秒创建的进程数   
-R：输出内存页面的统计信息   
-y：终端设备活动情况   
-w：输出系统交换活动信息   
 
ps -auxww|grep usr|grep java  : D    不可中断     Uninterruptible sleep (usually IO)
    R    正在运行，或在队列中的进程
    S    处于休眠状态
    T    停止或被追踪
    Z    已退出但描述符还在
    W    进入内存交换（从内核2.6开始无效）
    X    挂掉的进程
    <    高优先级
    N    低优先级
    L    有些页被锁进内存
    s    包含子进程
    +    位于后台的进程组；
    l    多线程，克隆线程  multi-threaded (using CLONE_THREAD, like NPTL pthreads do) 
cat /proc/meminfo   
free    
ll -h /proc/kcore    
/proc/meminfo 机器的内存使用信息   
statm 进程所占用的内存   
hostname -i    
netstat -anp | grep 6379   
netstat -tlnp   
cat /proc/swaps   
sudo slabtop   

tar zxvf     
sudo tar zcvf aaa.tar.gz aaa/    
source /etc/profile   

| more   
ls -alt   

sudo -u hdfs  hadoop jar .../hadoop-mapreduce-client-jobclient-...-tests.jar TestDFSIO -write -nrFiles 2 -fileSize 128MB -resFile /tmp/TestDFSIOresults.txt   
hdfs getconf -confkey  mapreduce.map.output.compress   

sort -u test1 同 sort test1 | uniq 排序后去重     
basename [pathname] 不加后缀可去除路径 basename /tmp/test/file.txt   :   file.txt       
basename [string] [suffix] 加后缀同时去除路径和后缀 basename /tmp/test/file.txt .txt    :   file   
nl 输出内容加行号
awk -F"_" '{print "" $1}' 以_为分隔符，打印第一项

http://www.ruanyifeng.com/blog/2019/08/xargs-tutorial.html
xargs命令的作用，是将标准输入转为命令行参数     echo "one two three" | xargs mkdir    同   mkdir one two three创建三个目录，如果要创建一个可以 -d 加分隔符  xargs -d "\t"    
-p参数打印出要执行的命令，询问用户是否要执行    
-t参数打印出最终要执行的命令，然后直接执行，不需要用户确认     
-0参数表示用null当作分隔符     
-L参数指定多少行作为一个命令行参数   
-n参数指定每次将多少项，作为命令行参数   
-I指定每一项命令行参数的替代字符串   
--max-procs参数指定同时用多少个进程并行执行命令     
xargs [-options] [command]      
$ xargs 等同于 $ xargs echo    $ xargs find -name "*.txt"    :   ./foo.txt。。。   

echo '一二三四五六七八九十' | LC_ALL="en_US.utf8" grep -oE '[一-十]'     
grep 
[[:alnum:]] - 字母数字字符
[[:alpha:]] - 字母字符
[[:blank:]] - 空字符: 空格键符 和 制表符
[[:digit:]] - 数字: '0 1 2 3 4 5 6 7 8 9'
[[:lower:]] - 小写字母: 'a b c d e f g h i j k l m n o p q r s t u v w x y z'
[[:space:]] - 空格字符: 制表符、换行符、垂直制表符、换页符、回车符和空格键符
[[:upper:]] - 大写字母: 'A B C D E F G H I J K L M N O P Q R S T U V W X Y Z'

pgrep进程grep   
pstree   
bc精度比较高的数学运算   
split文件分割   
nl == cat+行号   
mkfifo创建命名管道，|匿名管道   
ldd查看可执行文件所使用的动态链接库   

dpkg --list | grep "^rc" | cut -d " " -f 3   
dpkg --list | grep "^rc" | cut -d " " -f 3 | xargs sudo dpkg --purge   

ps -e -o 'pid,comm,args,pcpu,rsz,vsz,stime,user,uid' |  sort -nrk5     
-k 5 :按第5个参数 rsz实际内存大小进行排序     
-r：逆序     
-n：numeric，按数字来排序     

vim /etc/apt/sources-list   
sh Anaconda3-2019.10-Linux-x86_64.sh     
conda install jupyter notebook         
conda install tensorflow     
sudo apt install git     
sudo apt-get install openjdk-8-jdk   
curl -s "https://get.sdkman.io" | bash     
sdk install scala     
pip install findspark     
core-site.xml     
hdfs-site.xml     

pip install opencv-python  
https://pypi.org/project/opencv-python/#files   
pip install imageio     
pip install imageio-ffmpeg     
pip install moviepy     

cat /usr/local/cuda/version.txt

pip install tensorflow   : pip install -i https://pypi.tuna.tsinghua.edu.cn/simple tensorflow     
pip install torch torchvision     
sudo apt-get install vim     
pip install sklearn     
conda install pandas     
pip install Flask     
cd /home/aaa/Github/ScatteredCode/python/web/     
export FLASK_APP=hello.py     
python -m flask run --host=0.0.0.0     

c.NotebookApp.iopub_data_rate_limit = 10000000000    
c.NotebookApp.port = 9999    
#c.NotebookApp.notebook_dir = ''    
c.NotebookApp.ip = '*'    
c.NotebookApp.password = ''  
c.ContentsManager.allow_hidden = True

Ctrl+L 地址栏  
jobs -l   
nohup &   
jupyter notebook --generate-config --allow-root     
vim /home/aaa/.jupyter/jupyter_notebook_config.py     
nohup jupyter notebook --allow-root >/dev/null 2>&1 &     
. /home/ubuntu/anaconda3/etc/profile.d/conda.sh     
conda create -n       
linux在执行shell命令之前，会确定好所有输入输出位置，并且从左往右依次执行重定向命令，所以>/dev/null 2>&1的作用就是让标准输出重定向到/dev/null中
sudo apt-get install cifs-utils (sudo apt-get install smbclient)
sudo mount -t cifs //10.10.19.37/share ~/share -o iocharset=utf8,username=lidongbo,dir_mode=777,file_mode=777     
sudo apt install ubuntu-unity-desktop     
keyboard--navigation--switch to workspace above & below 快捷键     

slurmctld -Dvvv   
slurmd -Dvvv   
scontrol show partition   
scontrol show node ...2,...3   

sudo apt-get install automake     
sudo apt-get install autoconf     
sudo apt-get install libtool     
sudo apt-get install g++     
If you really want to use python 3.7.6 as default, add an empty file /export1/kaldi/tools/python/.use_default_python and run this script again   

extras/install_mkl.sh   
sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak   
vim /etc/apt/sources.list   

deb http://http.kali.org/kali kali-rolling main contrib non-free
deb http://old.kali.org/kali sana main non-free contrib

sudo apt-key add archive-key.asc   
sudo wget https://archive.kali.org/archive-key.asc   
cat /etc/apt/apt.conf  apt代理   
sudo apt install gnutls-bin   
gnutls-cli -V -p 443 apt.repos.intel.com   
apt policy apt-transport-https   
docs.kali.org/general-use/kali-linux-sources-list-repositories   

mkdir online   
cd online/   
cd ..   
cp ../voxforge/online_demo/run.sh online/   
cd online/   
mkdir online-data   
mkdir work   
cd online-data/   
mkdir audio   
mkdir models   
cd models/   
cp -r ../../../s5/exp/tri1 .   
cp -r ../../../s5/exp/tri2 .   
cp -r ../../../s5/exp/tri3a .   
cp -r ../../../s5/exp/tri4a .   
cp -r ../../../s5/exp/tri5a .   
cd ../..   
bash run.sh    
cp ../../thchs30/online_demo_tri4b_ali/online-data.tar.bz2 .   
cp online-data/models/tri3a/graph/words.txt online-data/models/tri3a   
cp online-data/models/tri3a/graph/HCLG.fst online-data/models/tri3a   

sudo ln -s /home/ubuntu/anaconda3/etc/profile.d/conda.sh /etc/profile.d/conda.sh     
echo "conda activate" >> ~/.bashrc     
export PATH="/home/ubuntu/anaconda3/bin:$PATH"     
https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=debnetwork     

git clone https://github.com/kaldi-asr/kaldi.git kaldi-trunk --origin golden   
cd kaldi-trunk/   
git clone https://github.com/alibaba/Alibaba-MIT-Speech   
git checkout 04b1f7d6658bc035df93d53cb424edc127fab819   
git apply --stat Alibaba-MIT-Speech/Alibaba_MIT_Speech_DFSMN.patch   
git apply --check Alibaba-MIT-Speech/Alibaba_MIT_Speech_DFSMN.patch   
git am --signoff < Alibaba-MIT-Speech/Alibaba_MIT_Speech_DFSMN.patch   
cd tools/   
extras/check_dependencies.sh      
cat /proc/cpuinfo |grep "cores"|uniq   
cat /proc/cpuinfo |grep "physical id"|sort |uniq|wc -l   
make -j 4   
cd ../src    
./configure --shared   
make depend -j 4   
make -j 4   
make ext   
cd ../egs/yesno/s5/   
./run.sh   
  
ps -ef | grep tomcat      
| head -10  
git reset HEAD +文件名 取消add      

put E:/Gitlab/loganalyzer/src/classes/logstreamlineparsersyslog.class.php

git init     
git remote add origin    
git branch --set-upstream-to=origin/master     
git pull origin master --allow-unrelated-histories       
git commit

sh mqshutdown namesrv
sh mqshutdown broker

nohup sh mqnamesrv > ns.log &
nohup sh mqbroker -c ../conf/2m-noslave/broker-a.properties > bk.log &


cat nohup.out

sh mqadmin topicStatus -n 192.168.1.44:9876 -t 'topicTest'
sh mqadmin updateTopic -b 192.168.1.44:10911 -n 192.168.1.44:9876 -t 'testTopic'
sh mqadmin brokerStatus -b 192.168.1.44:10911 -n 192.168.1.44:9876
sh mqadmin updateTopic -n 192.168.1.45:9876 -b 192.168.1.45:10911 -t 'registerTopic'
sh mqadmin brokerStatus -n 192.168.1.45:9876 -b 192.168.1.45:10911
sh mqadmin consumerStatus -g cgr -t registerTopic -n 192.168.1.45:9876
sh mqadmin deleteTopic -c DefaultCluster  -n 192.168.1.45:9876 -t 'registerTopic'
sh mqadmin topicStatus -n 192.168.1.45:9876 -t 'registerTopic'

sh mqadmin queryMsgByKey -k 201605301 -t testTopic -n 192.168.1.44:9876
sh mqadmin queryMsgByKey -k 201605301 -t testTopic -n 192.168.1.45:9876

sh mqadmin topicStatus -n 10.25.90.180:9876 -t 'registerTopic'
sh mqadmin topicStatus -n 10.171.82.8:9876 -t 'registerTopic'
sh mqadmin topicStatus -n 10.45.43.45:9876 -t 'registerTopic'

sh mqadmin queryMsgByKey -k 18672001963 -t registerTopic -n 10.25.90.180:9876
sh mqadmin queryMsgByKey -k 18672001963 -t registerTopic -n 10.171.82.8:9876

sh mqadmin queryMsgByOffset -b broker-a -i 1 -o 0 -t registerTopic -n 10.25.90.180:9876

sh mqadmin consumerProgress -g cgr -n 10.25.90.180:9876
sh mqadmin consumerProgress -g cgr -n 10.25.90.180:9876
sh mqadmin consumerProgress -g activitycgr1 -n 10.25.90.180:9876

sh mqadmin queryMsgByKey -k 1* -t activityTopic -n 10.25.90.180:9876

sh mqadmin updateTopic -b 10.25.90.180:10911 -n 10.25.90.180:9876 -t activityTopic
sh mqadmin updateTopic -b 10.45.43.45:10911 -n 10.45.43.45:9876 -t activityTopic
sh mqadmin deleteTopic -c DefaultCluster  -n 10.171.82.8:9876 -t 'registerTopic'

sh mqadmin topicStatus -n 10.25.90.180:9876 -t '%RETRY%activitycgr1'
sh mqadmin queryMsgByOffset -b broker-a -i 0 -o 9157 -t '%RETRY%activitycgr1' -n 10.25.90.180:9876

123.57.36.172

不输出  >/dev/null 2>&1 &     

service iptables status
（1） 重启后永久性生效：
　　开启：chkconfig iptables on
　　关闭：chkconfig iptables off
（2） 即时生效，重启后失效：
　　开启：service iptables start
　　关闭：service iptables stop

chmod +x /usr/local/soft/tomcat/tomcat7/bin/*.sh
chmod 777 *.sh 


curl https://www.caijinquan.cn/app/appv4/firstShowV2/queryFirstShow.action -d "{\"hmac\":\"a749b161651e12f67452f928d58b1bc29d1e2b97\",\"params\": {}}"


curl http://101.201.210.164/app/appv4/firstShowV2/queryFirstShow.action
rinetd -c /etc/rinetd.conf


jmap -dump:format=b,file=heapDump
jstat -gc
jstat -gccapacity
jhat heapDump


JAVA_OPTS='-Xloggc:/var/log/tomcat_gc.log'

du -h --max-depth=1 /root/store/



【一】从第3000行开始，显示1000行。即显示3000~3999行
cat filename | tail -n +3000 | head -n 1000
【二】显示1000行到3000行
cat filename| head -n 3000 | tail -n +1000
sed -n '5,10p' filename 这样你就可以只查看文件的第5行到第10行
    tail -n +1000：从1000行开始显示，显示1000行以后的
    head -n 1000：显示前面1000行


1、ps -mp xxxx -o THREAD
     在当前用户下，列出pid包含的所有线程。
 
2、ps -mp xxxx -o THREAD  >> /tmp/thread.txt
     在当前用户下，列出pid包含的所有线程。并把结果增量 输出到文件/tmp/thread.txt。
 
3、ps -mp xxxx -o THREAD,tid
     在当前用户下，列出pid包含的所有线程信息及本地线程ID (tid)。

4、ps -mp xxxx -o THREAD |wc -l
     在当前用户下，列出pid包含的所有线程的个数 。
     “wc -l”是统计记录的行数。

uname -a    
cat /proc/version   
cat /etc/issue   

perf record -F 99 -p PID -g — sleep 10   
perf script | ./stackcollapse-perf.pl > out.perf-folded   
./flamegraph.pl out.perf-folded>ou.svg   

$0  :  当前脚本的文件名       
$n  :  传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。   
$#  :  传递给脚本或函数的参数个数。   
$*  :  传递给脚本或函数的所有参数。   
$@  :  传递给脚本或函数的所有参数。   
$* 和 $@ 的区别：
当它们在双引号中，”$*" 会将所有的参数作为一个整体，以"$1 $2 … n"的形式输出所有参数；"@" 会将各个参数分开，以"$1" “$2"…"$n”的形式输出   
$?  :  上个命令的退出状态，或函数的返回值。   
$$  :  当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。   

-e filename  判断是否存在 
-d 判断是否为目录  
-f 判断是否为常规文件  [ -f /usr/bin/grep ] 
-L 判断是否为符号链接  [ -L /usr/bin/grep ]
-r 判断是否可读 
-w 判断是否可写 
-x 判断是否可执行 

trap有三种形式分别对应三种不同的信号回应方式.
第一种:
　trap 'commands' signal-list
当脚本收到signal-list清单内列出的信号时,trap命令执行双引号中的命令.
第二种:
　trap signal-list 
trap不指定任何命令,接受信号的默认操作.默认操作是结束进程的运行.
第三种:
　trap ' ' signal-list
trap命令指定一个空命令串,允许忽视信号.
INT（快速关闭）----当用户键入<Control-C>时由终端驱动程序发送的信号。这是一个终止当前操作的请求，如果捕获了这个信号，一些简单的程序应该退出，或者允许自己被终止，这也是程序没有捕获到这个信号时的默认处理方法。拥有命令行或者输入模式的那些程序应该停止它们在做的事情，清除状态，并等待用户的再次输入。
TERM（快速关闭）----是请求彻底终止某项执行操作，期望接收进程清除自给的状态并退出。
HUP---- 平滑启动，重新加载配置文件：
1：许多守护进程将其理解为一个重新设置的请求。如果一个进程不用重新启动就能重新读取它的配置文件并调整自己以适应变化的话，那么通常用HUP触发。
2：HUP信号有时由终端指挥生成，试图"清除"("终止")某特定终端的进程。如:某终端会话结束时，shell后台不接受HUP的信号，有时可以使用nohup来模仿这种行为。
PIPE 终止进程 向一个没有读进程的管道写数据

Linux通过locale来设置程序运行的不同语言环境，locale由ANSI C提供支持
LC_ALL是一个宏，如果该值设置了，则该值会覆盖所有LC_*的设置值。注意，LANG的值不受该宏影响。C"是系统默认的locale，"POSIX"是"C"的别名。所以当我们新安装完一个系统时，默认的locale就是C或POSIX。程序会去除所有locale sensitive的设置，以保证命令能正确执行。

#mktemp命令专门用来创建临时文件，并且其创建的临时文件是唯一的。shell会根据mktemp命令创建临时文件，但不会使用默认的umask值（管理权限的）。它会将文件的读写权限分配给文件属主，一旦创建了文件，在shell脚本中就拥有了完整的读写权限，其他人不可访问（除了root）
默认情况下，mktemp会在本地当前目录创建一个临时文件，创建临时文件时只需要创建模板文件，模板可以包含任意的文件名，文件末尾可以根据需要添加n个X


cat cols.txt |awk '{print $1}'|xargs|sed 's/`//g'|sed 's/ /,/g'   

`

ps:
%CPU 进程的cpu占用率
%MEM 进程的内存占用率
VSZ 进程所使用的虚存的大小
RSS 进程使用的驻留集大小或者是实际内存的大小
TTY 与进程关联的终端（tty）
STAT 检查的状态：进程状态使用字符表示的，如R（running正在运行或准备运行）、S（sleeping睡眠）、I（idle空闲）、Z (zombie)、D（不可中断的睡眠，通常是I/O）、P（等待交换页）、W（换出,表示当前页面不在内存）、N（低优先级任务）T(terminate终 止)、W has no resident pages

START （进程启动时间和日期）
TIME ;（进程使用的总cpu时间）
COMMAND （正在执行的命令行命令）
NI (nice)优先级
PRI 进程优先级编号
PPID 父进程的进程ID（parent process id）
SID 会话ID（session id）
WCHAN 进程正在睡眠的内核函数名称；该函数的名称是从/root/system.map文件中获得的。
FLAGS 与进程相关的数字标识

常用参数
-A 显示所有进程（等价于-e）(utility)
-a 显示一个终端的所有进程，除了会话引线
-N 忽略选择。
-d 显示所有进程，但省略所有的会话引线(utility)
-x 显示没有控制终端的进程，同时显示各个命令的具体路径。dx不可合用。（utility）
-p pid 进程使用cpu的时间
-u uid or username 选择有效的用户id或者是用户名
-g gid or groupname 显示组的所有进程。
U username 显示该用户下的所有进程，且显示各个命令的详细路径。如:ps U zhang;(utility)
-f 全部列出，通常和其他选项联用。如：ps -fa or ps -fx and so on.
-l 长格式（有F,wchan,C 等字段）
-j 作业格式
-o 用户自定义格式。
v 以虚拟存储器格式显示
s 以信号格式显示
-m 显示所有的线程
-H 显示进程的层次(和其它的命令合用，如：ps -Ha)（utility）
e 命令之后显示环境（如：ps -d e; ps -a e）(utility)
h 不显示第一行
