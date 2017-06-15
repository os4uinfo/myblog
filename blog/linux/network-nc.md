<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-25
title: nc 工具介绍
tags: Network
images: https://os4u.info/blog/img/sun.png
category: Network
status: publish
summary: 了解网络相关知识系列---nc工具
-->

##### nc简介
nc 即netcat缩写，tcp／udp相关的一切操作。打开tcp连接，发送udp数据包，监听tcp/udp端口，端口扫描。

1）监听端口，构建c/s聊天服务器

服务器端
```
$ nc -l 12345
```

客户端

```
$ nc xxx.xxx.xxx.xxx 12345
```
ctr+d断开链接


2）端口扫描

```
$ nc -z -v -n -w 2 xxx.xxx.xxx.xxx 20-23
扫描20-23端口
-z 建立连接后，立即断开，不发送任何数据
-v 打印详细信息
-n 不做域名解析
-w 超时时间，单位为秒
-u udp，默认为tcp协议
```

3）传输文件

```
接收端
$ nc -v -l 12345 < out.txt

发送端
$ nc -v -n xxx.xxx.xxx.xxx 12345 > in.txt

监听端也可以发送文件
$ nc -v -l 12345 > in.txt
接收端
$ nc -v -n xxx.xxx.xxx.xxx 12345 < out.txt
```

4）传输文件夹

```
发送端
$ tar -cvPf - /root/filename | nc -l 12345
接收端
$ nc -n xxx.xxx.xxx.xxx 12345 | tar -xvPf -

压缩传输
$ tar -czvPf - /root/filename | nc -l 12345 
接收端
$ nc -n xxx.xxx.xxx.xxx 12345 | tar -xzvPf -
```


![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 