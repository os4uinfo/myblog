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
$MODPROBE ip_conntrack
$MODPROBE iptable_nat
$MODPROBE ip_conntrack_ftp
$MODPROBE ip_nat_ftp

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
$IPTABLES -A INPUT -i eth1 ! -s $INT_NET -j LOG --log-prefix "SPOOFED PKT"
$IPTABLES -A INPUT -i eth1 ! -s $INT_NET -j DROP

### ACCEPT rules
$IPTABLES -A INPUT -i eth1 -p tcp -s $INT_NET --dport 22 --syn -m state --state NEW -j ACCEPT
$IPTABLES -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

### default INPUT LOG rule
$IPTABLES -A INPUT ! -i lo -j LOG --log-prefix "DROP" --log-ip-options --log-tcp-options

```
##### 4) OUTPUT链
OUTPUT链允许iptables对由本地系统产生的网络数据包进行内核级的控制，OUTPUT链可用于允许或拒绝外出的SYN数据包

```
###### OUTPUT chain ######
echo "[+] Seting up OUTPUT chain..."
### state tracking rules
$IPTABLES -A OUTPUT -m state --state INVALID -j LOG --log-prefix "DROP INVALID" --log-ip-options \
--log-tcp-options
$IPTABLES -A OUTPUT -m state --state INVALID -j DROP
$IPTABLES -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

### ACCEPT rules for allowing connections out
$IPTABLES -A OUTPUT -p tcp --dport 21 --syn -m state --state NEW -j ACCEPT
$IPTABLES -A OUTPUT -p tcp --dport 22 --syn -m state --state NEW -j ACCEPT
$IPTABLES -A OUTPUT -p tcp --dport 25 --syn -m state --state NEW -j ACCEPT
$IPTABLES -A OUTPUT -p tcp --dport 43 --syn -m state --state NEW -j ACCEPT
$IPTABLES -A OUTPUT -p tcp --dport 80 --syn -m state --state NEW -j ACCEPT
$IPTABLES -A OUTPUT -p tcp --dport 443 --syn -m state --state NEW -j ACCEPT
$IPTABLES -A OUTPUT -p tcp --dport 4321 --syn -m state --state NEW -j ACCEPT
$IPTABLES -A OUTPUT -p udp --dport 53 -m state --state NEW -j ACCEPT
$IPTABLES -A OUTPUT -p icpm --icmp-type echo-request -j ACCEPT

### default OUTPUT LOG rule
$IPTABLES -A OUTPUT ! -o lo -j LOG --log-prefix "DROP " --log-ip-options --log-tcp-options

```

##### 5) FORWARD 链
filter表中的FORWARD链提供了对通过防火墙接口转发数据包进行访问控制的能力

```
###### FORWARD chain ######
echo "[+] Setting up FORWARD chain..."
### state tracking rules
$IPTABLES -A FORWARD -m state --state INVALID -j LOG --log-prefix "DROP INVALID" --log-ip-options \
--log-tcp-options
$IPTABLES -A FORWARD -m state --state INVALID -j DROP
$IPTABLES -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT

### anti-spoofing rules
$IPTABLES -A FORWARD -i eth1 ! -s $INT_NET -j LOG --log-prefix "SPOOFED PKT"
$IPTABLES -A FORWARD -i eth1 ! -s $INT_NET -j DROP

### ACCEPT rules
$IPTABLES -A FORWARD -p tcp -i eth1 -s $INT_NET --dport 21 --syn -m state --state NEW -j ACCEPT
$IPTABLES -A FORWARD -p tcp -i eth1 -s $INT_NET --dport 22 --syn -m state --state NEW -j ACCEPT
$IPTABLES -A FORWARD -p tcp -i eth1 -s $INT_NET --dport 25 --syn -m state --state NEW -j ACCEPT
$IPTABLES -A FORWARD -p tcp -i eth1 -s $INT_NET --dport 43 --syn -m state --state NEW -j ACCEPT
$IPTABLES -A FORWARD -p tcp --dport 80 --syn -m state --state NEW -j ACCEPT
$IPTABLES -A FORWARD -p tcp --dport 443 --syn -m state --state NEW -j ACCEPT
$IPTABLES -A FORWARD -p tcp -i eth1 -s $INT_NET --dport 4321 --syn -m state --state NEW -j ACCEPT
$IPTABLES -A FORWARD -p udp --dport 53 -m state --state NEW -j ACCEPT
$IPTABLES -A FORWARD -p icmp --icmp-type echo-request -j ACCEPT

### default log rule
$IPTABLES -A FORWARD ! -i lo  -j LOG --log-prefix "DROP " --log-ip-options --log-tcp-options

```

##### 6) 网络地址转换

构建iptables策略所需的最后一步，是将不可路由的网络转换为可路由的外部地址。iptables的nat表专用雨定义所有的NAT规则

```
###### NAT rules ######
echo "[+] Setting up NAT rules..."
$IPTABLES -t nat -A PREROUTING -p tcp --dport 80 -i eth0 -j DNAT --to 192.168.10.3:80
$IPTABLES -t nat -A PREROUTING -p tcp --dport 443 -i eth0 -j DNAT --to 192.168.10.3:443
$IPTABLES -t nat -A PREROUTING -p tcp --dport 53 -i eth0 -j DNAT --to 192.168.10.4:53

$IPTABLES -t nat -A POSTROUTING -s $INT_NET -o eth0 -j MASQUERADE

###### forwarding ######
echo "[+]Enabling IP forwarding..."
echo 1 > /proc/sys/net/ipv4/ip_forward

```

##### 7) 保存与恢复

```
iptables-save
iptables-restore

```
待续...
