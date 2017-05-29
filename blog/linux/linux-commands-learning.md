<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-17
title: Linux 命令之网络
tags: Network
images: https://os4u.info/blog/img/sun.png
category: Network
status: publish
summary: Linux运维中，常用命令解决实际的运维问题，持续更新，总结中。
-->
#### Linux 命令之网络

![step-by-step](https://www.os4u.info/blog/linux/images/step-by-step.jpg)

1.ping

1).ping 次数

```
ping -c 3 www.os4u.info
ping 三次
```

2)直接查结果

```
ping -q -c 3 www.os4u.info
mdev 是mean Deviation的缩写，表示ICMP包的RTT偏离平均值的程度，主要来判定网速的稳定性。
mdev 的值越大，说明网速越不稳定。
```
3)指定ping数据包大小

```
ping -s 65500 -c 3 www.os4u.info
使用65500字节的数据包来测试网络，通常-s用来发现网络环境中有关MTU的问题。
```
4)指定ping的TTL

```
ping -t 255 www.os4u.info
```
5)指定ping的时间间隔

```
ping -i 0.1 -c www.os4u.info
```
6)快速ping

```
ping -f -c 3 www.os4u.info
-f 就是flood ping潮水模式ping
```

RTT参考

场景                  | RTT参考值           |
---------------------|--------------------|
ping本机              |     0.01ms         | 
ping同机房             |     0.1ms          |
ping同城              |    1ms             |
ping不同城市           |    20ms            |
北（南）方ping南（北）方 |    50ms            |
从国内ping国外机器      |    200ms           |

2.nslookup

1) 连接到公网DNS服务器

```
$ nslookup - 114.114.114.114
> www.os4u.info
Server:		114.114.114.114
Address:	114.114.114.114#53

Non-authoritative answer:
www.os4u.info	canonical name = os4u.info.
Name:	os4u.info
Address: 113.107.219.16
>

```
2) DNS 查询方式(需复习，后续补充）

```
a.DNS递归查询

b.DNS迭代查询

c.DNS递归和迭代查询
```
