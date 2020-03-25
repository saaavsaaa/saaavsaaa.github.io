conda deactivate     
conda activate dl     
ifconfig     
source /etc/profile     
chmod 755      
fg
lsof -i:9999
netstat -antp

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
pip install imageio     
pip install imageio-ffmpeg     
pip install moviepy     

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

Ctrl+L 地址栏  
jobs -l
nohup &
jupyter notebook --generate-config --allow-root     
/home/aaa/.jupyter/jupyter_notebook_config.py     
nohup jupyter notebook --allow-root >/dev/null 2>&1 &     
. /home/ubuntu/anaconda3/etc/profile.d/conda.sh     
conda create -n       
linux在执行shell命令之前，会确定好所有输入输出位置，并且从左往右依次执行重定向命令，所以>/dev/null 2>&1的作用就是让标准输出重定向到/dev/null中
sudo apt-get install cifs-utils (sudo apt-get install smbclient)
sudo mount -t cifs //10.10.19.37/share ~/share -o iocharset=utf8,username=lidongbo,dir_mode=777,file_mode=777     
sudo apt install ubuntu-unity-desktop     
keyboard--navigation--switch to workspace above & below 快捷键     

git clone https://github.com/kaldi-asr/kaldi.git kaldi-trunk --origin golden    
sudo apt-get install automake     
sudo apt-get install autoconf     
sudo apt-get install libtool     
sudo apt-get install g++     
cd kaldi-trunk/     
cd tools/     
extras/check_dependencies.sh     
sudo apt-get install sox subversion     
extras/check_dependencies.sh   
extras/install_mkl.sh   
extras/check_dependencies.sh   
make -j 4   
cd ..   
cd src/   
./configure --shared   
make depend -j 4   
make -j 4   

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
  
mkdir   
rm -rf   
ps -ef | grep tomcat   

git reset HEAD +文件名 取消add    

tail -f    
tail -fn 500 nohup.out   

cat /etc/sysctl.conf可修改   
cat /proc/sys/fs/file-nr   
cat /proc/sys/fs/file-max   
sysctl -a | grep fs.file-max最大文件描述符   
ulimit -n查看单进程最大文件描述符   
ulimit -n d更改为d   
ulimit -a查看限制   
lsof查看当前哪些进程打开哪些句柄，哪些文件被哪些进程占用   

pmap -d    

find / -name rocketmq_console.tar   
ps -auxww|grep usr|grep java   
cat /proc/meminfo   
free    
ll -h /proc/kcore    
/proc/meminfo 机器的内存使用信息   
statm 进程所占用的内存   
hostname -i    
netstat -anp | grep 6379   
netstat -tlnp   

| more   
| head -10   

grep error -m 10 === grep error  | head -10   
grep -v 排除   
grep -r "DataSourceActionInterceptor" *   
cat /proc/(pid)/status   
top命令下按f键   

ls -alt   

sort -u test1 同 sort test1 | uniq 排序后去重     
basename [pathname] 不加后缀可去除路径 basename /tmp/test/file.txt   :   file.txt     
basename [string] [suffix] 加后缀同时去除路径和后缀 basename /tmp/test/file.txt .txt    :   file

source /etc/profile   

git commit
put E:/Gitlab/loganalyzer/src/classes/logstreamlineparsersyslog.class.php

tar zxvf     
mkdir
rm -rf
ps -ef | grep tomcat

df -hl

tail -f
tail -fn 500 nohup.out
sftp aaa@ip

cat /etc/sysctl.conf可修改
cat /proc/sys/fs/file-nr
cat /proc/sys/fs/file-max
sysctl -a | grep fs.file-max最大文件描述符
ulimit -n查看单进程最大文件描述符
ulimit -n d更改为d
ulimit -a查看限制
lsof查看当前哪些进程打开哪些句柄，哪些文件被哪些进程占用

pmap -d 

find / -name rocketmq_console.tar
ps -auxww|grep usr|grep java
cat /proc/meminfo
free
ll -h /proc/kcore 
/proc/meminfo 机器的内存使用信息
statm 进程所占用的内存
hostname -i 
netstat -anp | grep 6379
netstat -tlnp

| more
| head -10

grep error -m 10 === grep error  | head -10
grep -v 排除
grep -r "DataSourceActionInterceptor" *
cat /proc/(pid)/status
top命令下按f键

ls -alt

source /etc/profile

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
chown -R jenkins:jenkins /var/build/

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
