<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-06-07
title: 防火墙学习笔记（1）
tags: iptables 
images: https://os4u.info/blog/img/sun.png
category: iptables
status: publish
summary: iptables学习，本节主要介绍防火墙的默认策略
-->

防火墙学习笔记
-
---



##### 1)学习防火墙网络架构如下：

![learning-iptables-topology](https://www.os4u.info/blog/iptables/images/learning-iptables-topology.png)

##### 2)iptables.sh

文件名：iptables.sh

```
内容如下：
-------------------------------------
#!/bin/bash
IPTABLES=/sbin/iptables
MODPROBE=/sbin/modprobe
INT_NET=192.168.10.0/24

echo "[+] Flushing existing iptables rules..."
$IPTABLES -F
$IPTABLES -F -t nat
$IPTABLES -X
$IPTABLES -P INPUT DROP
$IPTABLES -P OUTPUT DROP
$IPTABLES -P FORWARD DROP
### load connection-tracking modules
$MODEPROBE ip_contrack
$MODEPROBE iptable_nat
$MODEPROBE ip_conntrack_ftp
$MODEPROBE ip_nat_ftp

-------------------------------------
[注] 
1.变量指定iptables，modprobe二进制文件路径，并设置内网子网地址和掩码
2.将已有的iptables规则从运行内核中移除，INPUT、OUTPUT和FORWARD链过滤策略设为DROP
3.使用modprobe命令加载连接跟踪模块

```
##### 3) INPUT 链

INPUT链过滤是IP及以上协议的数据包，2.6以上内核INPUT可基于数据链路层MAC地址来过滤IP数据包。下面继续编写INPUT链规则

```
###### INPUT chain ######
echo "[+] Setting up INPUT chain..."
### state tracking rules
$IPTABLES -A INPUT -m state --state INVALID -j LOG --log-prefix "DROP INVALID" \
--log-ip-options --log-tcp-options
$IPTABLES -A INPUT -m state --state INVALID -j DROP
$IPTABLES -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

### anti-spoofing rules
$IPTABLES -A INPUT -i eth1 -s ! $INT_NET -j LOG --log-prefix "SPOOFED PKT"
$IPTABLES -A INPUT -i eth1 -s ! $INT_NET -j DROP

### ACCEPT rules
$IPTABLES -A INPUT -i eth1 -p tcp -s $INT_NET --dport 22 --syn -m state --state NEW -j ACCEPT
$IPTABLES -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

### default INPUT LOG rule
$IPTABLES -A INPUT -i ! log -j LOG --log-prefix "DROP" --log-ip-options --log-tcp-options

```

待续...