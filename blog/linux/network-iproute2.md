<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-24
title: iproute2相关知识
tags: Network
images: https://os4u.info/blog/img/sun.png
category: Network
status: publish
summary: 了解iproute2相关知识系列---iproute2应用
-->

![dns-dig-usage](https://www.os4u.info/blog/linux/images/dns-dig-usage.jpg)

##### iproute2 知识

netstat 可以用来显示网络连接，路由表，接口统计，无效连接和组播成员信息，但逐渐被ss和ip命令替代
替换如下：

|     作用    | 原命令            |   新命令      |
|:-----------: |:---------------:| :-------------:|
| 网络连接     |  netstat -a      |  ss         |
| 路由表       |  netstat -r      |    ip route |
| 统计接口     |  netstat -i      |  ip -s link |
| 伪连接       | netstat -M       | ss          |
| 组播成员      | netstat -g       | ip maddr    |

##### ss命令
```
ss命令是用来查看sockets信息的命令，速度比netstat更快，更高效。netstat直接读取/proc/net/tcp文件，执行
速度非常慢，而ss命令使用TCP协议中的tcp_diag模块，确保了ss高效。
```

|     作用    | 原命令            |   新命令      |
|:-----------: |:---------------:| :-------------:|
| 地址和链路配置     |  ifconfig     |  ip addr, ip link         |
| 路由表            |  route        |  ip route        |
| arp 表           |  arp          |  ip neigh        |
| VLAN             | vconfig       |  ip link         |
| 隧道              |  iptunnel     |  ip tunnel        |
| 组播              |  ipmaddr      |  ip maddr         |
| 统计              |  ifconfig     |  ss               |

1）ss统计选项

场景1:查看服务器网络连接

```
$ ss -s
```
场景2:查看打开的网络端口

```
$ ss -lp
```
场景3:查看所有sockets连接

```
$ ss -a
-ta 查看tcp sockets
-ua 查看udp sockets
-w 查看RAW sockets
-x 查看UNIX sockets
```

##### 一些工具

旧软件命令

```
ipmaddr 管理组播地址，
mii-tool管理网络接口状态 
nameif设置基于mac地址的网络接口名称 
plipconfig管理PLIP协议设备参数
slattach指定网络接口关联到特定的串行线路
```
新软件命令

```
ip 管理路由，设备，策略和隧道
ss展示系统套接字相关信息
tc 管理流量控制策略
nstat 网络统计
bridge 管理网络地址和设备
ifcfg 进行ip管理，以替代ifconfig
lnstat 展示网络状态
```

iproute2系列工具对GRE隧道等新技术支持非常好。

|     作用         | 原命令                                 |   新命令      |
|:--------------: |:-------------------------------------:| :------------------------:|
| 本机网络接口     |  ifconfig                              |  ip link show                      |
| 开启某个网络接口  |  ifconfig eth0 up                      |  ip link set up eth0               |
| 停止某个网络接口  |  ifconfig eth0 down                    |  ip link set down eth0             |
| 配置IP          |  ifconfig eth0 10.0.0.1/24             |  ip addr add 10.0.0.1/24 dev eth0  |
| 删除网络IP地址    |  ifconfig eth0 0                      |   ip addr del 10.0.0.1/24 dev eth0  |
| 显示接口IP       |  ifconfig eth0                         |  ip addr show dev eth0             |
| 显示路由表       |  route -n                              |   ip route show                          |
| 添加网关         |  route add default gw 192.168.1.1 eth0 |   ip route add default via 192.168.1.1 eth0 |
| 删除网关         |  route del default gw 192.168.1.1 eth0 |   ip route del default via 192.168.1.1 eth0 (ip route replace default via 192.186.1.1 dev eth0)|
| arp        |  arp -an |   ip neigh |
| 添加arp        |  arp -s 192.168.1.100 00:0c:29:c1:51:ef |   ip neigh add 192.168.1.100 lladdr 00:0c:29:c1:51:ef dev eth0|
| 删除arp      |  arp -d 192.168.1.100 00:0c:29:c1:51:ef |   ip neigh del 192.168.1.100  dev eth0|
| 查看组播        |  ipmaddr show dev eth0 |   ip maddr list dev eth0|
| 展示套接字        |  netstat -l |  ss -l |


##### ipv6

```
$ ip -6 addr show dev eth0
```