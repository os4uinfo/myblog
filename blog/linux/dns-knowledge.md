<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-20
title: DNS相关知识
tags: DNS
images: https://os4u.info/blog/img/sun.png
category: DNS
status: publish
summary: 了解DNS相关知识
-->

![images](https://www.os4u.info/blog/linux/images/step-by-step.jpg)
##### DNS -- nslookup 输出解析

以www.os4u.info来作为例子。
```
$ nslookup www.os4u.info
Server:		114.114.114.114
Address:	114.114.114.114#53

Non-authoritative answer:
www.os4u.info	canonical name = os4u.info.
Name:	os4u.info
Address: 113.107.219.16
```

域名解析分成了上下两部分

```
Server:		114.114.114.114
Address:	114.114.114.114#53

```
第一行为本次DNS服务器，默认使用/etc/resolv.conf配置的第一个DNS服务器

第二行为负责解析的服务器和端口

```
Non-authoritative answer:
www.os4u.info	canonical name = os4u.info.
Name:	os4u.info
Address: 113.107.219.16
```

Non-authoritative answer,非权威返回结果，可能是因为时间差原因导致不是最新的。

www.os4u.info	canonical name = os4u.info.这就是常说的别名。

因此，解析到的IP，才是服务器的地址。

##### DNS协议中的五元组

{ DomainName, TimeToLive, Class, Type, Value}
翻译过来即：{域名，生存期限，类别，类型，值}

```
DomainName: 指我们要查询的的域名
TimeToLive：表示域名在各个DNS服务器缓存中应保存的时长
Class: 通常为IN，即Internet，还有CH（Chaos）和HS（Hesiod）两类，但至今已被淘汰
Type: 指出这条记录的类型，包括8种，即SOA，A，MX，NS，CNAME，PTR，HINFO和TXT
Value: 针对不同类型，会有不同值
```

举个例子：

```
$ dig www.os4u.info

; <<>> DiG 9.8.3-P1 <<>> www.os4u.info
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 41431
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 2, ADDITIONAL: 4

;; QUESTION SECTION:
;www.os4u.info.			IN	A

;; ANSWER SECTION:
www.os4u.info.		3600	IN	CNAME	os4u.info.
os4u.info.		3600	IN	A	113.107.219.16

;; AUTHORITY SECTION:
os4u.info.		3600	IN	NS	ns73.domaincontrol.com.
os4u.info.		3600	IN	NS	ns74.domaincontrol.com.

;; ADDITIONAL SECTION:
ns73.domaincontrol.com.	159788	IN	A	216.69.185.47
ns73.domaincontrol.com.	11542	IN	AAAA	2607:f208:206::2f
ns74.domaincontrol.com.	159788	IN	A	208.109.255.47
ns74.domaincontrol.com.	131980	IN	AAAA	2607:f208:302::2f

;; Query time: 16 msec
;; SERVER: 114.114.114.114#53(114.114.114.114)
;; WHEN: Tue May 23 22:53:46 2017
;; MSG SIZE  rcvd: 204
```
中的

```
www.os4u.info.		3600	IN	CNAME	os4u.info.
os4u.info.		3600	IN	A	113.107.219.16
```
第一行表示www.os4u.info是os4u.info的别名， 其超时时限是3600秒，第二行表示对应IP及超时时限3600秒

##### DNS的八种类型

```
SOA: Start of Authority 授权起始
A：IP地址
MX： 邮件交换
NS：域名服务器
CNAME： 别名，也叫规范名
PTR: 指针，用于反解析
HINFO：主机描述信息，包括CPU和OS等信息
TXT：其他一些文本信息
```

逐项说明：

```
$ nslookup -type=soa os4u.info
Server:		114.114.114.114
Address:	114.114.114.114#53

Non-authoritative answer:
os4u.info
	origin = ns73.domaincontrol.com
	mail addr = dns.jomax.net
	serial = 2017050900
	refresh = 28800
	retry = 7200
	expire = 604800
	minimum = 600

Authoritative answers can be found from:
os4u.info	nameserver = ns73.domaincontrol.com.
os4u.info	nameserver = ns74.domaincontrol.com.
ns73.domaincontrol.com	internet address = 216.69.185.47
ns73.domaincontrol.com	has AAAA address 2607:f208:206::2f
ns74.domaincontrol.com	internet address = 208.109.255.47
ns74.domaincontrol.com	has AAAA address 2607:f208:302::2f
```
SOA类型中：

```
mail addr:管理员邮箱地址
serial：版本号一般为年月日次，2017050900表示为：2017年05月09日的第0个版本
refresh：表示slave的DNS服务器多久向master服务器要更新一次数据
retry：slave连接到master，重试次数
expire：当发起refresh时，如果始终无法连接到master多久后放弃重试
minimum：即TTL，表示外部DNS服务器如果缓存本DNS服务器的授权数据，保存多久
```

类型A中：表示从域名解析到IP地址

```
$ nslookup -type=a www.os4u.info
Server:		114.114.114.114
Address:	114.114.114.114#53

Non-authoritative answer:
www.os4u.info	canonical name = os4u.info.
Name:	os4u.info
Address: 113.107.219.16
```

类型MX，表示有限服务器设置有关，用来表示当前域名对应的邮箱服务器。

类型NS，表示给定域名下所包含的DNS服务器信息

```
$ nslookup -type=ns os4u.info
Server:		114.114.114.114
Address:	114.114.114.114#53

Non-authoritative answer:
os4u.info	nameserver = ns74.domaincontrol.com.
os4u.info	nameserver = ns73.domaincontrol.com.

Authoritative answers can be found from:
```

类型CNAME，别名

```
nslookup -type=cname  www.os4u.info
Server:		114.114.114.114
Address:	114.114.114.114#53

Non-authoritative answer:
www.os4u.info	canonical name = os4u.info.

Authoritative answers can be found from:

```

类型PTR：用来表示反解析，即从IP地址查询对应的域名映射关系

类型HINFO：包含CPU和OS等信息

类型TXT：即文本信息，表示有关此域名的一些信息。


##### 查询域名A记录

```
nslookup
> www.os4u.info
Server:		114.114.114.114
Address:	114.114.114.114#53

Non-authoritative answer:
www.os4u.info	canonical name = os4u.info.
Name:	os4u.info
Address: 113.107.219.16
```

##### 更改上联DNS地址
```
$ nslookup
> www.os4u.info
Server:		114.114.114.114
Address:	114.114.114.114#53

Non-authoritative answer:
www.os4u.info	canonical name = os4u.info.
Name:	os4u.info
Address: 113.107.219.16
> server 8.8.8.8
Default server: 8.8.8.8
Address: 8.8.8.8#53
> www.os4u.info
Server:		8.8.8.8
Address:	8.8.8.8#53

Non-authoritative answer:
www.os4u.info	canonical name = os4u.info.
Name:	os4u.info
Address: 113.107.219.16
>
```

##### 查看当前DNS配置
```
$ nslookup
> set all
Default server: 8.8.8.8
Address: 8.8.8.8#53

Set options:
  novc			nodebug		nod2
  search		recurse
  timeout = 0		retry = 3	port = 53
  querytype = A       	class = IN
  srchlist =
>

```
##### 进入调试模式
```
$ nslookup
> set debug
```

##### 查询中默认的域后缀
```
$ nslookup
> set all
Default server: 8.8.8.8
Address: 8.8.8.8#53

Set options:
  novc			nodebug		nod2
  search		recurse
  timeout = 0		retry = 3	port = 53
  querytype = A       	class = IN
  srchlist =
>
> set domain=os4u.info
> set all
Default server: 8.8.8.8
Address: 8.8.8.8#53

Set options:
  novc			nodebug		nod2
  search		recurse
  timeout = 0		retry = 3	port = 53
  querytype = A       	class = IN
  srchlist = os4u.info
> www
Server:		8.8.8.8
Address:	8.8.8.8#53

Non-authoritative answer:
www.os4u.info	canonical name = os4u.info.
Name:	os4u.info
Address: 113.107.219.16
>
```

##### 在交互模式下设置type
```
A: 查看主机IPv4地址
AAAA： 查看主机IPv6地址
ANY: 查看关于主机域的所有信息
CNAME: 别名
HINFO: 主机的CPU等信息
MINFO：邮箱信息
MX： 邮件交换信息
NS: 主机域的域名服务器
PTR: IP 查域名
RP: 域负责人记录
SOA: 域内的SOA地址
UINFO:用户信息
```

例子：

```
> set type=CNAME
> www.os4u.info
Server:		8.8.8.8
Address:	8.8.8.8#53

Non-authoritative answer:
www.os4u.info	canonical name = os4u.info.

Authoritative answers can be found from:
os4u.info	nameserver = ns73.domaincontrol.com.
os4u.info	nameserver = ns74.domaincontrol.com.
ns73.domaincontrol.com	internet address = 216.69.185.47
ns73.domaincontrol.com	has AAAA address 2607:f208:206::2f
ns74.domaincontrol.com	internet address = 208.109.255.47
ns74.domaincontrol.com	has AAAA address 2607:f208:302::2f
>
```