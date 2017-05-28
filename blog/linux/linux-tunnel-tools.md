<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-28
title: tunnel tools
tags: Security
images: https://os4u.info/blog/img/sun.png
category: Linux
status: publish
summary: Linux security
-->

##### Tunnel tools

```
* tcp tunnel: stunnel, tcp tunnel, tor, ssh, wintunnel, sixtynine, zebedee
* udp tunnel: ssh, netcallback, cipe, tunnel, zebedee
* icmp tunnel: ptunnel, itun, itunnel, skeeve, icmptx
```
[stunnel uage](https://www.stunnel.org/index.html)

[tcptunnel](http://www.vakuumverpackt.de/tcptunnel/)

[tcptunnel usage](http://t.cn/RLkzdBR)

[icmptx usage](http://thomer.com/icmptx/)

##### Other methods

login into localhost
```
step 1:
[user@localhost]$ ssh -R 1337:localhsot:22 root@xxx.xxx.xxx.xx
step 2:
[root@xxx.xxx.xxx.xxx]# ssh -L 31337:localhost:1337 -f -N -g root@xxx.xxx.xxx.xxx
step 3:
[anonymous@anyhost]$ ssh -l user -p 31337 xxx.xxx.xxx.xxx

[notice]: sshd_config
TCPKeepAlive yes
ClientAliveInterval 30
ClientAliveCountMax 9999


``` 

##### steelcap ?
[reference](http://www.educatedguesswork.org/2007/05/steelcape_and_f.html)


##### Mail 

[MIMEDefang](http://www.mimedefang.org/)

[amavis](http://amavis.sourceforge.net/)

[mail-ip-check](http://www.rbls.org)

[SPF](http://www.openspf.org/)

[DomainKey](http://www.dkim.org/)

##### Kernel Security

[cfengine](https://cfengine.com/)

-----
Reference: [Hacking Exposed Linux 3rd - JOEL SCAMBRAY_STUART MCCLURE_GEORGE KURT](http://www.cs.ucf.edu/~hlugo/cis4361/private/supplementals/Hacking%20Exposed%20Linux%203rd%20-%20JOEL%20SCAMBRAY_STUART%20MCCLURE_GEORGE%20KURT.pdf)
