<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-06-01
title: iptables学习笔记
tags: iptables
images: https://os4u.info/blog/img/sun.png
category: iptables
status: publish
summary: 学习iptables
-->

#### 1） iptables 表

```
iptables 共有4个表：filter、nat、mangle和raw

filter: 过滤规则 
nat: NAT规则
mangle: 修改分组数据的特定规则
raw: 独立于Netfilter连接跟踪子系统起作用的规则
```

#### 2）iptables 链

```
每张表都有自己的一组内置链，用户可以自定义链，从而建立一组规则。它们都关联到一个共同的标签如INPUT_ESTABLISHED或DMZ_NETWORK。
本文中，最重要的内置链是filter的INPUT、OUTPUT和FORWARD链
* 数据包指向一个本地套接字，它将经过INPUT链检查
* OUTPUT链保留给linux系统自身生成的数据包
* FORWARD链管理经过Linux系统路由的数据包（即iptables防火墙用于连接两个必经防火墙网络）

nat表中的PREROUTING和POSTROUTING链
* PREROUTING 内核进行IP路由计算之前的数据包头
* POSTROUTING 内核进行IP路由计算之后的数据包头
	
```
![iptables-pics-1](https://www.os4u.info/blog/iptables/images/iptables_pics_1.gif)
#### 3） 匹配

每个iptables规则都包含一组匹配以及一个目标，后者告诉iptables对于符合规则的数据包应该采取什么动作。

iptables匹配指的是数据包必须匹配的条件。只有数据包满足所有匹配条件时，iptables才能根据该规则的目标锁指定的动作来处理数据包。

每个匹配都在iptables命令行中指定。

```
* --source(-s)        ---------匹配源IP地址或网络
* --destination(-d)   ---------匹配目标IP地址或网络
* --protocol(-p)      ---------匹配协议
* --in-interface(-i)  ---------流入接口（如：eth0）
* --out-interface(-o) ---------流出接口
* --state             ---------匹配一组连接状态
* --string            ---------匹配应用层数据字节序列
* --comment           ---------在内核内存中为一个规则关联多大256歌字节的注释
```

#### 4） 目标

iptables的目标用于在数据包匹配一条规则时候触发一个动作。具体如下：

```
* ACCEPT  ----------允许数据包通过
* DROP    ----------丢弃数据包，不对该数据包作进一步的处理，对接收栈而言，就好像该数据包从未被接收
* LOG     ----------将数据包信息记录到syslog
* REJECT  ----------丢弃数据包，同时发送适当的响应报文（如，针对TCP连接的TCP重置数据包，或针对UDP数据包的ICMP端口不可达）
* RETURN  ----------在调用链中继续处理数据包
```


![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 