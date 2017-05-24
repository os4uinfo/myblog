<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-24
title: dig使用相关知识
tags: DNS
images: https://os4u.info/blog/img/sun.png
category: DNS
status: publish
summary: 了解DNS相关知识系列---dig应用
-->

![dns-dig-usage](https://www.os4u.info/blog/linux/images/dns-dig-usage.jpg)

##### dig使用

1）dig 后加一个点.

```
查询根域
```

2）用google开放DNS来查询

```
$ dig @8.8.8.8 www.os4u.info A

; <<>> DiG 9.8.3-P1 <<>> @8.8.8.8 www.os4u.info A
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37589
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 2, ADDITIONAL: 4

;; QUESTION SECTION:
;www.os4u.info.			IN	A

;; ANSWER SECTION:
www.os4u.info.		3600	IN	CNAME	os4u.info.
os4u.info.		3600	IN	A	113.107.219.16

;; AUTHORITY SECTION:
os4u.info.		3600	IN	NS	ns74.domaincontrol.com.
os4u.info.		3600	IN	NS	ns73.domaincontrol.com.

;; ADDITIONAL SECTION:
ns73.domaincontrol.com.	171161	IN	A	216.69.185.47
ns73.domaincontrol.com.	171761	IN	AAAA	2607:f208:206::2f
ns74.domaincontrol.com.	150088	IN	A	208.109.255.47
ns74.domaincontrol.com.	162599	IN	AAAA	2607:f208:302::2f

;; Query time: 2870 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Wed May 24 22:04:22 2017
;; MSG SIZE  rcvd: 204

```

命令格式：

```
$ dig @dnsserver domain querytype
```

3) 批量查询

```
$ dig -f querylist -t A
querylist 为域名列表
-t 指定查询类型(默认为A）
```

4）显式设置查询域名-q

主要是区分极短且极易与参数混淆的域名

```
$ dig -q www.os4u.info -t A

; <<>> DiG 9.8.3-P1 <<>> -q www.os4u.info -t A
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64519
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 2, ADDITIONAL: 4

;; QUESTION SECTION:
;www.os4u.info.			IN	A

;; ANSWER SECTION:
www.os4u.info.		3600	IN	CNAME	os4u.info.
os4u.info.		3600	IN	A	113.107.219.16

;; AUTHORITY SECTION:
os4u.info.		3599	IN	NS	ns73.domaincontrol.com.
os4u.info.		3599	IN	NS	ns74.domaincontrol.com.

;; ADDITIONAL SECTION:
ns73.domaincontrol.com.	169935	IN	A	216.69.185.47
ns73.domaincontrol.com.	151871	IN	AAAA	2607:f208:206::2f
ns74.domaincontrol.com.	151871	IN	A	208.109.255.47
ns74.domaincontrol.com.	47921	IN	AAAA	2607:f208:302::2f

;; Query time: 23 msec
;; SERVER: 114.114.114.114#53(114.114.114.114)
;; WHEN: Wed May 24 22:20:08 2017
;; MSG SIZE  rcvd: 204

```

5） -x反解域名
```
$ dig -x 8.8.8.8

; <<>> DiG 9.8.3-P1 <<>> -x 8.8.8.8
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 34489
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 2

;; QUESTION SECTION:
;8.8.8.8.in-addr.arpa.		IN	PTR

;; ANSWER SECTION:
8.8.8.8.in-addr.arpa.	3600	IN	PTR	google-public-dns-a.google.com.

;; AUTHORITY SECTION:
8.in-addr.arpa.		75417	IN	NS	ns1.level3.net.
8.in-addr.arpa.		75417	IN	NS	ns2.level3.net.

;; ADDITIONAL SECTION:
ns1.level3.net.		672	IN	A	209.244.0.1
ns2.level3.net.		672	IN	A	209.244.0.2

;; Query time: 10 msec
;; SERVER: 114.114.114.114#53(114.114.114.114)
;; WHEN: Wed May 24 22:23:29 2017
;; MSG SIZE  rcvd: 160
```
查看ANSWER SECTION部分

6）用tcp代替udp

```
$ dig +tcp @114.114.114.114 www.os4u.info

; <<>> DiG 9.8.3-P1 <<>> +tcp @114.114.114.114 www.os4u.info
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 31341
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 2, ADDITIONAL: 4

;; QUESTION SECTION:
;www.os4u.info.			IN	A

;; ANSWER SECTION:
www.os4u.info.		2183	IN	CNAME	os4u.info.
os4u.info.		600	IN	A	113.107.219.16

;; AUTHORITY SECTION:
os4u.info.		2182	IN	NS	ns73.domaincontrol.com.
os4u.info.		2182	IN	NS	ns74.domaincontrol.com.

;; ADDITIONAL SECTION:
ns73.domaincontrol.com.	268	IN	A	216.69.185.47
ns73.domaincontrol.com.	171621	IN	AAAA	2607:f208:206::2f
ns74.domaincontrol.com.	171621	IN	A	208.109.255.47
ns74.domaincontrol.com.	46504	IN	AAAA	2607:f208:302::2f

;; Query time: 77 msec
;; SERVER: 114.114.114.114#53(114.114.114.114)
;; WHEN: Wed May 24 22:25:12 2017
;; MSG SIZE  rcvd: 204
```

7) 默认追加域

```
$ dig +domain=os4u.info www

; <<>> DiG 9.8.3-P1 <<>> +domain=os4u.info www
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61884
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 2, ADDITIONAL: 4

;; QUESTION SECTION:
;www.os4u.info.			IN	A

;; ANSWER SECTION:
www.os4u.info.		3600	IN	CNAME	os4u.info.
os4u.info.		3600	IN	A	113.107.219.16

;; AUTHORITY SECTION:
os4u.info.		3599	IN	NS	ns73.domaincontrol.com.
os4u.info.		3599	IN	NS	ns74.domaincontrol.com.

;; ADDITIONAL SECTION:
ns73.domaincontrol.com.	169935	IN	A	216.69.185.47
ns73.domaincontrol.com.	151871	IN	AAAA	2607:f208:206::2f
ns74.domaincontrol.com.	151871	IN	A	208.109.255.47
ns74.domaincontrol.com.	47921	IN	AAAA	2607:f208:302::2f

;; Query time: 12 msec
;; SERVER: 114.114.114.114#53(114.114.114.114)
;; WHEN: Wed May 24 22:26:03 2017
;; MSG SIZE  rcvd: 204

```

8) 跟踪dig查询过程

```
$ dig +trace www.os4u.info

; <<>> DiG 9.8.3-P1 <<>> +trace www.os4u.info
;; global options: +cmd
.			175000	IN	NS	f.root-servers.net.
.			175000	IN	NS	m.root-servers.net.
.			175000	IN	NS	d.root-servers.net.
.			175000	IN	NS	k.root-servers.net.
.			175000	IN	NS	g.root-servers.net.
.			175000	IN	NS	j.root-servers.net.
.			175000	IN	NS	c.root-servers.net.
.			175000	IN	NS	l.root-servers.net.
.			175000	IN	NS	a.root-servers.net.
.			175000	IN	NS	e.root-servers.net.
.			175000	IN	NS	i.root-servers.net.
.			175000	IN	NS	b.root-servers.net.
.			175000	IN	NS	h.root-servers.net.
//从本地DNS查询到根域DNS列表
;; Received 508 bytes from 114.114.114.114#53(114.114.114.114) in 171 ms

info.			172800	IN	NS	b2.info.afilias-nst.org.
info.			172800	IN	NS	b0.info.afilias-nst.org.
info.			172800	IN	NS	a0.info.afilias-nst.info.
info.			172800	IN	NS	a2.info.afilias-nst.info.
info.			172800	IN	NS	c0.info.afilias-nst.info.
info.			172800	IN	NS	d0.info.afilias-nst.org.
//使用192.112.36.4域名服务器来查找.info域名DNS列表
;; Received 434 bytes from 192.112.36.4#53(192.112.36.4) in 308 ms

os4u.info.		86400	IN	NS	ns73.domaincontrol.com.
os4u.info.		86400	IN	NS	ns74.domaincontrol.com.
;; Received 86 bytes from 199.254.31.1#53(199.254.31.1) in 210 ms

www.os4u.info.		3600	IN	CNAME	os4u.info.
os4u.info.		600	IN	A	113.107.219.16
os4u.info.		3600	IN	NS	ns74.domaincontrol.com.
os4u.info.		3600	IN	NS	ns73.domaincontrol.com.
;; Received 116 bytes from 208.109.255.47#53(208.109.255.47) in 470 ms
```

9）精简输出

```
输出最精简的CNMAE和A记录
$ dig +short www.os4u.info
os4u.info.
113.107.219.16

其他精简冗余内容
$ dig +nocmd +nocomment +nostat www.os4u.info
;www.os4u.info.			IN	A
www.os4u.info.		3600	IN	CNAME	os4u.info.
os4u.info.		3600	IN	A	113.107.219.16
os4u.info.		3599	IN	NS	ns73.domaincontrol.com.
os4u.info.		3599	IN	NS	ns74.domaincontrol.com.
ns73.domaincontrol.com.	169935	IN	A	216.69.185.47
ns73.domaincontrol.com.	151871	IN	AAAA	2607:f208:206::2f
ns74.domaincontrol.com.	151871	IN	A	208.109.255.47
ns74.domaincontrol.com.	47921	IN	AAAA	2607:f208:302::2f
```


