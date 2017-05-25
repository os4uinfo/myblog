<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-25
title: tcpdump相关知识
tags: Network
images: https://os4u.info/blog/img/sun.png
category: Network
status: publish
summary: 了解tcpdump相关知识系列---tcpdump应用
-->

#### tcpdump使用
1）基本使用

```
$ tcpmdump -i eth0 -nn -X 'port 80' -c 1

-i interface
-nn 不要将协议号或端口号转换成对于服务缩写。例如：80端口不显示为http
-X 显示协议头和包内容原有格式显示
'port 80' 过滤只抓取80端口数据包
-c 抓取包的个数
```

2）其他选项

三大部分：第一项“选项”，第二项“过滤表达式”，第三项“输出信息”

```
-e 增加以太网头部信息输出
$ tcpdump -i enp3s0 -c 1
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp3s0, link-type EN10MB (Ethernet), capture size 65535 bytes
11:18:39.548759 IP bogon.ssh > bogon.62571: Flags [P.], seq 3786419316:3786419504, ack 2007260194, win 316, options [nop,nop,TS val 542575193 ecr 863335485], length 188
1 packet captured
7 packets received by filter
0 packets dropped by kernel

$ tcpdump -i enp3s0 -e -c 1
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp3s0, link-type EN10MB (Ethernet), capture size 65535 bytes
11:18:44.825541 38:2c:4a:6e:5d:54 (oui Unknown) > 58:6a:b1:0f:68:e8 (oui Unknown), ethertype IPv4 (0x0800), length 254: bogon.ssh > bogon.62571: Flags [P.], seq 3786420316:3786420504, ack 2007260558, win 316, options [nop,nop,TS val 542580470 ecr 863340788], length 188
1 packet captured
6 packets received by filter
0 packets dropped by kernel

oui -- 组织唯一标志符，就是网卡制造商的信息

-l 让输出变为行缓冲
$ tcpdump -i enp3s0 -l  | awk '{print $1}'
如果不设置，那么只有当缓冲区全部占满，才会将缓冲区中的内容输出出来，会导致输出时总是间断不顺畅。

-t 输出时不打印时间戳

-v 输出更详细的信息
在原有输出内容的基础之上，还会到tos值，ttl值，ID值，总长度，校验值等

-F 指定过滤表达式所在的文件夹

# tcpdump -i enp3s0 -nn -F  filter.txt -c 1
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp3s0, link-type EN10MB (Ethernet), capture size 65535 bytes
14:17:28.272424 IP 192.168.1.50.50733 > 114.114.114.114.53: 29477+ A? www.baidu.com. (31)
1 packet captured
1 packet received by filter
0 packets dropped by kernel

[root@k8s-master ~]# cat filter.txt
port 53


```

3) 保存与回放

```
-w 保存到本地raw packets(原始网络包）
-r 读取raw packets
```

4)过滤流量

抓取UDP的包

```
# tcpdump -i enp3s0 -c 10 'udp'
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp3s0, link-type EN10MB (Ethernet), capture size 65535 bytes
14:31:19.314148 IP bogon.40591 > public1.114dns.com.domain: 63778+ A? www.baidu.com. (31)
14:31:19.314626 IP bogon.49723 > public1.114dns.com.domain: 8703+ PTR? 114.114.114.114.in-addr.arpa. (46)
14:31:19.317457 IP public1.114dns.com.domain > bogon.40591: 63778 3/0/0 CNAME www.a.shifen.com., A 163.177.151.110, A 163.177.151.109 (90)
14:31:19.317565 IP public1.114dns.com.domain > bogon.49723: 8703 1/0/0 PTR public1.114dns.com. (78)
14:31:19.317765 IP bogon.37272 > public1.114dns.com.domain: 53432+ PTR? 50.1.168.192.in-addr.arpa. (43)
14:31:19.321012 IP public1.114dns.com.domain > bogon.37272: 53432* 1/0/0 PTR bogon. (62)
14:31:19.326493 IP bogon.54784 > public1.114dns.com.domain: 32002+ PTR? 110.151.177.163.in-addr.arpa. (46)
14:31:19.329767 IP public1.114dns.com.domain > bogon.54784: 32002 NXDomain 0/1/0 (101)
14:31:31.978325 IP bogon.bootpc > 255.255.255.255.bootps: BOOTP/DHCP, Request from d4:3d:7e:13:ae:89 (oui Unknown), length 300
14:31:31.978530 IP bogon.51230 > public1.114dns.com.domain: 59420+ PTR? 255.255.255.255.in-addr.arpa. (46)
10 packets captured
13 packets received by filter
0 packets dropped by kernel

```
指定目的

```
# tcpdump -i enp3s0 'dst www.baidu.com'
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp3s0, link-type EN10MB (Ethernet), capture size 65535 bytes
14:35:30.269680 IP bogon > 163.177.151.110: ICMP echo request, id 13093, seq 1, length 64

```

关注特定端口

```
tcpdump -i enp3s0 'dst www.os4u.info or port 53 or dst port 80'

host: 指定主机， host xxx.xxx.xxx.xxx
net： 指定网络 src net 192.168 或dst net 192.168
portrange： 指定端口区域 src or dst portrange 6000-7000
```
获取主机三次握手中第一个网络包，即带有SYN标记的网络包，并排出主机www.os4u.info

```
tcpdump -i enp3s0  'tcp[tcpflags] & tcp-syn != 0 and not dst host www.os4u.info'

```

打印IP包长度超过576字节的网络包

```
tcpdump -i enp3s0 -X  'ip[2:2] > 576'

proto[expr:size] proto-protocol, expr用来指数据报偏移量，即从某个协议的数据报的第多少位开始提取内容。
默认起始位置0，size是从偏移量的位置开始提取多少个字节。
若设置了expr，而未设置size，则默认提取1个字节。ip[2:2]表示提取3，4个字节。ip[0]则表示提取ip协议头的第一个字节。
比较操作符：> < >= <= = != 6个

协议。有以下几种：
ether：链路层协议
fddi: 链路层协议
tr: 链路层协议
wlan: 链路层协议
ppp: 链路层协议
slip: 链路层协议
link: 链路层协议
ip: 网络层
arp: 网络层
rarp: 网络层
tcp: 传输层
udp: 传输层
icmp: 传输层
ip6: 传输层
```
![image](http://image.lxway.com/upload/8/70/870822343e58645afbb8dbf734e4a172.jpg)

```

ip[0] & 0xf 提取的是IP的“首部长度”
!或not：表示“否定”
&& 与 and: 表示“与”
|| 与 or: 表示“或”

'tcp[tcpflags] & tcp-syn != 0 and not dst host os4u.info'
'ip[2:2] > 576'
'ether[0] & 1 = 0 and ip[16] >= 224'
```


打印广播包或组播包，同时数据链路层不是通过以太网


```
tcpdump -i enp3s0 'ether[0] & 1 = 0 and ip[16] >= 224'
```

tcpdump输出内容解读

```
# tcpdump -i enp3s0 -nn -X 'port 53' -c 1
1.  tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
2.  listening on enp3s0, link-type EN10MB (Ethernet), capture size 65535 bytes
3.  15:23:46.534276 IP 192.168.1.50.43143 > 114.114.114.114.53: 46047+ A? www.os4u.info. (31)
4.   	0x0000:  4500 003b f868 4000 4011 9b8a c0a8 0132  E..;.h@.@......2
5.  	0x0010:  7272 7272 a887 0035 0027 c1cf b3df 0100  rrrr...5.'......
6.  	0x0020:  0001 0000 0000 0000 0377 7777 046f 7334  .........www.os4
7.  	0x0030:  7504 696e 666f 0000 0100 01              u.info.....
8.  1 packet captured
9.  1 packet received by filter
10. 0 packets dropped by kernel

4-7行是IP包内容，左侧是十六进制内容，右侧是ASCII内容。
用wireshark分析
```

![tcpdump-usage](https://www.os4u.info/blog/linux/images/tcpdump.png)


