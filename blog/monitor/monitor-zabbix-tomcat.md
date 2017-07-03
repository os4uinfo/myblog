<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-07-03
title: 使用Zabbix 监控tomcat
tags: monitor
images: https://os4u.info/blog/img/sun.png
category: monitor
status: monitor
summary: 使用Zabbix监控tomcat。
-->
##### 一、	环境准备

```1. 系统：centos 6.7zabbix版本：3.0.4zabbix server端java支持（需要jar，java，javac命令）防火墙关闭
```
```2. 下载zabbix源码编译参数如下（具体安装略）：./configure --prefix=/opt/ljops/apps/monitors/zabbix --enable-server --enable-proxy --enable-agent --enable-java --with-mysql --with-ldap --enable-java --with-net-snmp```
##### 二、	zabbix server端操作
#### 在zabbix服务器端配置如下：1）	开启java gateway.

在/opt/ljops/apps/monitors/zabbix/sbin/zabbix_java目录中修改配置setting.sh，有效配置如下：```[root@lj-zabbix zabbix_java]# sed  -e '/^#/d' -e '/^$/d' settings.sh======配置分割线开始=======LISTEN_IP="0.0.0.0"LISTEN_PORT=10052PID_FILE="/tmp/zabbix_java.pid"START_POLLERS=0======配置分割线结束=======开启服务[root@lj-zabbix zabbix_java]# ./startup.sh[root@lj-zabbix zabbix_java]#启动后，检查10052端口是否开启```
2）	zabbix server配置如下

/opt/ljops/apps/monitors/zabbix/etc/zabbix_server.conf
```[root@lj-zabbix etc]# sed -e '/^#/d;/^$/d' zabbix_server.conf======配置分割线开始=======LogFile=/tmp/zabbix_server.logDBHost=localhostDBName=zabbixDBUser=zabbixDBPassword=passwordStartPollers=5JavaGateway=172.16.61.32JavaGatewayPort=10052StartJavaPollers=5SNMPTrapperFile=/tmp/zabbix_traps.tmpStartSNMPTrapper=1ListenIP=0.0.0.0Timeout=4AlertScriptsPath=/opt/ljops/apps/monitors/zabbix/alertscriptsLogSlowQueries=3000======配置分割线结束=======
```
开启zabbix server服务
```[root@lj-zabbix etc]# /etc/init.d/zabbix_server start```3）	下载cmdline-jmxclient-0.10.3.jar后面使用java测试##### 三、	被监控服务器（zabbix agentd）端操作
被监控端操作1）	下载catalina-jmx-remote.jar
```进入tomcat目录 /data/apps/apache-tomcat-7.0.70/lib/按照使用的版本下载(https://archive.apache.org/dist/tomcat/)该服务器上使用的是tomcat 7.0.70版本，下载地址为：https://archive.apache.org/dist/tomcat/tomcat-7/v7.0.70/bin/extras/catalina-jmx-remote.jar具体下载操作如下：[root@testing-software-server ~]# cd /data/apps/apache-tomcat-7.0.70/lib/[root@testing-software-server lib]# wget https://archive.apache.org/dist/tomcat/tomcat-7/v7.0.70/bin/extras/catalina-jmx-remote.jar--2016-11-30 15:34:53--  https://archive.apache.org/dist/tomcat/tomcat-7/v7.0.70/bin/extras/catalina-jmx-remote.jarResolving archive.apache.org... 163.172.17.199Connecting to archive.apache.org|163.172.17.199|:443... connected.HTTP request sent, awaiting response... 200 OKLength: 13012 (13K) [application/java-archive]Saving to: “catalina-jmx-remote.jar.1”100%[==========================================>] 13,012      --.-K/s   in 0s2016-11-30 15:34:55 (106 MB/s) - “catalina-jmx-remote.jar” saved [13012/13012][root@testing-software-server lib]#```2）	修改catalina.sh(/data/apps/apache-tomcat-7.0.70/bin)文件

```添加内容如下(添加在脚本前面，确保执行命令能生效)：======配置分割线开始=======export CATALINA_OPTS="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false  -Djava.rmi.server.hostname=172.16.66.13"======配置分割线结束=======（其中172.16.66.13是服务器IP）```
3）	修改server.xml(/data/apps/apache-tomcat-7.0.70/conf)文件```添加内容如下：======配置分割线开始=======<Listener className="org.apache.catalina.mbeans.JmxRemoteLifecycleListener"  rmiRegistryPortPlatform="12345" rmiServerPortPlatform="12345"/>======配置分割线结束=======```4）	启动zabbix agentd和tomcat服务

```[root@testing-software-server ~]# /etc/init.d/zabbix_agentd start[root@testing-software-server ~]# cd /data/apps/apache-tomcat-7.0.70/bin[root@testing-software-server ~]# ./startup.sh```
##### 四、	测试1）	方法1:在zabbix server服务器端测试是否能正常获取信息（配置服务器端时，下载的cmdline-jmxclient-0.10.3.jar文件）

```[root@lj-zabbix zabbix_java]# java -jar cmdline-jmxclient-0.10.3.jar - 172.16.66.13:12345 java.lang:type=Memory NonHeapMemoryUsage11/30/2016 15:46:07 +0800 org.archive.jmx.Client NonHeapMemoryUsage:committed: 31916032init: 24576000max: 136314880used: 17985496[root@lj-zabbix zabbix_java]#```
2）	zabbix官方提供脚本```
脚本内容如下：======脚本内容分割线开始=======#!/usr/bin/env bashZBXGET="/opt/ljops/apps/monitors/zabbix/bin/zabbix_get"if [ $# != 5 ]then    echo "Usage: $0 <JAVA_GATEWAY_HOST> <JAVA_GATEWAY_PORT> <JMX_SERVER> <JMX_PORT> <KEY>"    exit;fiQUERY="{\"request\": \"java gateway jmx\",\"conn\": \"$3\",\"port\": $4,\"keys\": [\"$5\"]}"#echo $ZBXGET -s $1 -p $2 -k "$QUERY"$ZBXGET -s $1 -p $2 -k "$QUERY"======脚本内容分割线结束=======保存为：zabbix_get_jmx使用如下：[root@lj-zabbix zabbix]# bash zabbix_get_jmx 172.16.61.32  10052 172.16.66.13 12345  jmx["java.lang:type=Runtime",Uptime]{"data":[{"value":"19914637"}],"response":"success"}[root@lj-zabbix zabbix]#```	##### 五、	添加模版，并将模版应用到服务器上
在zabbix server的web界面1）	下载并导入模版

```使用zabbix自带的监控项目部分监控项不能获取到数据，故需要自定义一个模版。模板下载地址如下：http://172.16.66.13/scripts/JMX_templates.xml```2）	在zabbix server添加被监控服务器的jmx接口```配置端口为：12345配置如图：```
![monitor-java-](https://www.os4u.info/blog/monitor/images/monitor-java-setting.jpeg)
3）	将模版应用到具体服务器（略）
4）	效果如下图
![monitor-java-effect](https://www.os4u.info/blog/monitor/images/monitor-java-effect.jpeg)##### 六、	问题处理：

```查看jmx参数图时会出现乱码。zabbix图片中中文乱码解决如下：http://blog.chinaunix.net/uid-11121450-id-3296646.html
```