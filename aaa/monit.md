 几年前用它作过监控，最近又有这个打算了，把以前的记录搬过来 https://www.cnblogs.com/saaav/p/5504803.html，监控某些服务，失败自动重启，同时监控特定的日志文件，如果有变化，就发邮件报警

  安装不细写了，网上好多

  我先用cat /proc/version看了下我的系统是el6的，于是wget http://pkgs.repoforge.org/monit/monit-5.5-1.el6.rf.x86_64.rpm

  启动什么的就不管了，直接上配置文件:


set daemon  10 

set logfile /var/monit/monit.log

set idfile /var/monit/id

set statefile /var/monit/state

  set mailserver smtp.xxxxxxxxx.com port 25 USERNAME "xxxxxxxx@xxxxxxxx.cn" PASSWORD "xxxxxxxx"
  set mail-format {
          from: xxxxxx@xxxxxxx.cn
          subject: monit alert --  $EVENT $SERVICE
          message: $EVENT Service $SERVICE
                Date:        $DATE
                Action:      $ACTION
                Host:        $HOST
                Description: $DESCRIPTION

    }

  set alert xxxxxxxxx@xxxxxxxx.com with reminder on 3 cycles

set httpd port 2812 and
    use address 192.168.1.45
    allow xxxxx:*****
    allow @users readonly

include /etc/monit.d/*

两个例子：

check process nginx with pidfile /var/run/nginx/nginx.pid
start program = "/etc/init.d/nginx start"
stop program = "/etc/init.d/nginx stop"
if failed host localhost port 80 protocol http
then restart
和文件变化的监控

check file test.log with path /tmp/test.log
if changed size then alert
 ##########################################################

今天又测试了下监控目录变化：

从github上找了下代码，代码里的文档比官网上的细多了。。。。

check directory root with path /root
if changed timestamp then alert
alert ****@******.com
然后干脆把rocket的监控也加上了

check host 192.168.1.45 with address 192.168.1.45
#check system 192.168.1.45
      # if failed icmp type echo count 1 with timeout 3 seconds then alert
       if failed host 192.168.1.45 port 9876  type tcp then alert
       if failed host 192.168.1.45 port 10911 type TCP then alert
       alert xxxxxxx@xxxxxxxxxx.com

![Image](https://images2015.cnblogs.com/blog/445166/201605/445166-20160521143543810-1880566177.png)

##########################################################

#从http://blog.chinaunix.net/uid-21516619-id-1825022.html copy 过来的tomcat监控

check process tomcat with pidfile /var/run/catalina.pid     # 这个要另外说明【2】

    start program = "/etc/init.d/tomcat start"              # 设置启动命令

    stop program  = "/etc/init.d/tomcat stop"               # 设置停止命令

    if 9 restarts within 10 cycles then timeout             # 设置在10个监视周期内重

                                                            # 启了9次则超时,不再监视

                                                            # 这个服务。原因另外说明【3】

        if cpu usage > 90% for 5 cycles then alert          # 如果在5个周期内该服务

                                                            # 的cpu使用率都超过90%

                                                            # 则提示

# 若连续5个周期打开url都失败（120秒超时，超时也认为失败）

# 则重启服务

        if failed url http://127.0.0.1:4000/ timeout 120 seconds for 5 cycles then restart

        if failed url http://127.0.0.1:5000/ timeout 120 seconds for 5 cycles then restart

 

使用/var/run/catalina.pid这个pid文件来检查tomcat这个服务(服务名可以随便起),tomcat进程默认是不使用pid文件的,pid文件需要显式为tomcat设置,可以打开tomcat目录下的bin目录,打开catalina.sh文件,在开头（但不是第一行）处加入:

CATALINA_PID=/var/run/catalina.pid

即可指定pid文件,然后重启tomcat,这样就可以monit的配置中指定pid文件了。

参考:

http://blog.chinaunix.net/uid-24774106-id-3705291.html

http://blog.csdn.net/fan_hai_ping/article/details/41733783

http://blog.chinaunix.net/uid-21516619-id-1825022.html

https://github.com/arnaudsj/monit

http://mmonit.com/monit/
