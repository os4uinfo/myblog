<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-28
title: Linux 命令之安全
tags: Security
images: https://os4u.info/blog/img/sun.png
category: Linux
status: publish
summary: Linux运维中，常用命令解决实际的运维问题，持续更新，总结中。
-->

#### 1. Hidden Server
[knockd](https://github.com/jvinet/knock)

#### 2. File
[tripwire](https://github.com/Tripwire/tripwire-open-source)

[aide](https://aide.sourceforge.net)

```
创建系统后，建立md5文件md5sum.list 
md5sum -c md5sum.list

创建系统后，建立sha256sum文件sha256sum.list
sha256sum -c sha256sum.list
```

[md5deep](http://md5deep.sourceforge.net/)

[rkhunter](http://rkhunter.sourceforge.net/)

```
Usage:
rkhunter --propupd
rkhunter --check
```

#### 3. Netcat

[netcat](https://nmap.org/ncat)

```
Usage:
a)
$ ncat -C www.os4u.info 80
GET / HTTP/1.0
[ENTER]
[ENTER]

b) 传输文件
发送端
$ ncat -l 1234 > bootstrap.txt
接收端
$ ncat --send-only xxx.xxx.xxx.xxx 12345 < bootstrap.orig

压缩传输
发送端
# ncat -l | tar xzv
接收端
# tar czv <list of files> | ncat --send-only xxx.xxx.xxx.xxx


发送端
# ncat -l | bzip2 -d > massive.file.bz
接收端
# cat massive.file.orig | bzip2 | ncat --send-only xxx.xxx.xxx.xxx

c)聊天
Server
# ncat -l 1234
Client
# ncat xxx.xxx.xxx.xxx 1234

d)命令拼接
[root@hosta] # ncat -l 1234 > my_new_big_file.txt
[root@hostb] # ncat -l 1234 | ncat hosta 1234
[root@hostc] # ncat --send-only hostb 1234 < lengthy_file.txt

[root@hosta] # ncat -l 1234 > newlog.txt
[root@hostb] # ncat -l 1234 --sh-exec "ncat hosta 1234"
[root@hostc] # ncat --send-only hostb 1234 < logfile.log

d)端口转发
[root@hostd] # ncat -l localhost 80 --sh-exec "ncat www.os4u.info 80"

e)安全通信
[root@host] # ncat -C --ssl www.os4u.info 443
[root@host] # ncat -C --ssl-verify www.os4u.info 443
[root@host] # ncat -C --ssl-verify --ssl-trustfile <custom-certs.pem> <server> 443
[root@host] # ncat -l localhost 143 --sh-exec "ncat --ssl www.os4u.info 443"
[root@host] # ncat -v --listen -ssl

f)可执行文件
[root@hosta] # ncat --exec "/bin/bash" -l 1234 --keep-open
[root@hostb] # ncat hosta 1234

g)访问控制列表
[root@host] # ncat --exec "/bin/bash" --max-conns 3 --allow 192.168.0.0/24 -l 8081 --keep-open
[root@host] # ncat -l --deny 1222:cb88::3b
[root@host] # ncat -l --allowfile trusted-hosts.txt

h) Others
[root@host] # ncat --sctp
sctp(stream control transmission protocol)

```
[more usage](https://www.os4u.info/blog/linux/network-nc.html)

#### 4. DDoS

```
NTP反射攻击
[root@host] # iptables -A INPUT -s 0/0 -d 0/0 -p udp --source-port 123:123 -m stat --state ESTABLISHED -j ACCEPT
[root@host] # iptables -A OUTPUT -s 0/0 -d 0/0 -p udp --destination-port 123:123 -m state --state NEW,ESTABLISHED -j ACCEPT

SNMP反射攻击

DNS 解析器

```
[Reference]

[ATLAS](https://www.arbornetworks.com/threats/)

[digitalattackmap](http://www.digitalattackmap.com/gallery)

#### 5. Nping 

[nping](https://nmap.org/nping)

```
[root@host] # nping -c1 --tcp -p 80,443 localhost

Starting Nping 0.7.40 ( https://nmap.org/nping ) at 2017-05-29 13:01 CST
SENT (0.0391s) TCP 127.0.0.1:2061 > 127.0.0.1:80 S ttl=64 id=55801 iplen=40  seq=2741805035 win=1480 
RCVD (0.0392s) TCP 127.0.0.1:80 > 127.0.0.1:2061 RA ttl=64 id=16531 iplen=40  seq=0 win=0 
SENT (1.0393s) TCP 127.0.0.1:2061 > 127.0.0.1:443 S ttl=64 id=55801 iplen=40  seq=2741805035 win=1480 
RCVD (1.0394s) TCP 127.0.0.1:443 > 127.0.0.1:2061 RA ttl=64 id=16668 iplen=40  seq=0 win=0 
 
Max rtt: 0.046ms | Min rtt: 0.044ms | Avg rtt: 0.045ms
Raw packets sent: 2 (80B) | Rcvd: 2 (80B) | Lost: 0 (0.00%)
Nping done: 1 IP address pinged in 1.07 seconds

[root@host] # nping -c1 --tcp -p 80,443 www.os4u.info
Starting Nping 0.7.40 ( https://nmap.org/nping ) at 2017-05-29 13:02 CST
SENT (0.0564s) TCP 192.168.100.4:38704 > 113.107.219.16:80 S ttl=64 id=30394 iplen=40  seq=1486415092 win=1480 
RCVD (0.0702s) TCP 113.107.219.16:80 > 192.168.100.4:38704 SA ttl=52 id=0 iplen=44  seq=2896013213 win=14600 <mss 1412>
SENT (1.0574s) TCP 192.168.100.4:38704 > 113.107.219.16:443 S ttl=64 id=30394 iplen=40  seq=1486415092 win=1480 
RCVD (1.0719s) TCP 113.107.219.16:443 > 192.168.100.4:38704 SA ttl=51 id=0 iplen=44  seq=2308804094 win=14600 <mss 1412>
 
Max rtt: 14.417ms | Min rtt: 12.980ms | Avg rtt: 13.698ms
Raw packets sent: 2 (80B) | Rcvd: 2 (88B) | Lost: 0 (0.00%)
Nping done: 1 IP address pinged in 1.13 seconds


[root@host] # nping -c1 --tcp -p 0-1024 www.os4u.info

[root@host] # nping --tcp -p 80 --flags rst -c1 www.os4u.info

Starting Nping 0.7.40 ( https://nmap.org/nping ) at 2017-05-29 13:05 CST
SENT (0.2066s) TCP 192.168.100.4:43593 > 113.107.219.16:80 R ttl=64 id=58072 iplen=40  seq=2141850229 win=1480 
 
Max rtt: N/A | Min rtt: N/A | Avg rtt: N/A
Raw packets sent: 1 (40B) | Rcvd: 0 (0B) | Lost: 1 (100.00%)
Nping done: 1 IP address pinged in 1.25 seconds

[root@host] # nping --tcp -p 80 --flags syn -c1 www.os4u.info
[root@host] # nping --tcp -p 80 --flags ack -c1 www.os4u.info

UDP
[root@host] # nping --udp www.os4u.info

ICMP
[root@host] # nping  www.os4u.info

ARP
[root@host] # nping --arp www.os4u.info

Others
--data
--data-string 
--data-length

Echo
[root@host] # nping -e eth0 -vvv --echo-server "please_connect"

```

##### 6.Logging

```
tcpdump
[root@host] # tcpdump ip proto \\icmp

iptables
[root@host] # iptables -I INPUT -p icmp --icmp-type 8 -m stast --state NEW, ESTABLISHED,RELATED -j LOG --log-level=1 --log-prefix "pings logged"

```
types | code
------|------
0       | echo reply
3       | desination unreachable*
4       | source Quench*
5       | redirect
8       | echo request
B       | time exceeded
C      | parameter problem
D      | time stamp request
E       | time stamp reply
F       | info request
G       | info reply
H      | address mask request
I        |address mask reply

```
net.ipv4.icmp_echo_ignore_broadcasts = 1
echo "1" > /proc/sys/net/ipv4/icmp_echo_ignore_broadcasts
[root@host] # iptables -I INPUT -p icmp -m limit --limit 30/minute --limit-burst 60 -j ACCEPT
[root@host] # iptables -I INPUT -p icmp -m limit --limit 6/minute --limit-burst 10 -j LOG 
[root@host] # iptables -I INPUT -p icmp -j DROP

```

#### 7. NSE

```
[root@host] # nmap -PN xxx.xxx.xxx.xxx
[root@host] # nmap -sT www.os4u.info
[root@host] # nmap xxx.xxx.xxx.xxx -excludefile  ipaddress.txt
[root@host] # nmap -p U:53, T:0-1024, 88 xxx.xxx.xxx.xxx
[root@host] # nmap -sP xxx.xxx.xxx.xxx
[root@host] # nmap -p0-1024 -T 4 xxx.xxx.xxx.xxx

Zenmap
```

#### 8. Malware check

[R-fs](https://www.rfxn.com)

LMD usage

#### 9. Hashcat 

[Hashcat](https://hashcat.net/hashcat)

[howsecureismypassword.net](https://howsecureismypassword.net)

```
/etc/shadow
```

符号 | 哈希算法
------|-----------
$0     | DES
$1     | MD5 hashing 
$2     | Blowlish
$2A  | Eksblowfish
$5     | SHA256
$6     | SHA512


Other tools

```
oclHashcat 
cudaHashcat 
Hashcat-utils
```

#### 10. SQL

[OWASP](https://www.owasp.org/index.php/Top_10_2013-Top_10)



![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 