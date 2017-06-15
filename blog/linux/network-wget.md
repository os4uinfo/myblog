<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-26
title: wget 工具介绍
tags: Network
images: https://os4u.info/blog/img/sun.png
category: Network
status: publish
summary: 了解网络相关知识系列---wget工具
-->

##### wget 工具介绍
1） 下载文件

```
$ wget https://www.os4u.info/blog/linux/images/dns-dig-usage.jpg

wget 配置文件位置
/etc/wgetrc 及家目录.wgetrc配置

-X 排除下载目录，
也可以在配置文件中添加 exclude_directories=xxx,xxx
-r 递归目录下载

--background 后台下载

--execute robots=off 避开robots协议

-nH --no-host-directiories 不创建以域名为名称的文件夹

--cut-dirs=number 指定目录层级
```

| 参数| 结果|
|:--:|:--|
|-r | www.os4u.info/blog/linux/images/|
| -nH | /blog/linux/images|
| -nH --cut-dirs=1 | linux/images|
| -nH --cut-dirs=2 | images|
| -nH --cut-dirs=3 | .|
| --cut-dirs=1 | www.os4u.info/linux/images|

```
-nd --no-directories 下载文件，不下载文件夹

-x --force-directories 下载文件夹

--protocol-directories 创建以协议名为名称的文件夹，然后再下载  tcp://www.os4u.info/blog/

-t --tries=numbers 下载重试次数

-o --output-file=logfile 下载过程中日志记录
-O --output-document-file 指定下载文件位置，重命名。

-nc 禁止文件重复下载

-N --timestamping 启用时间戳机制，对比本地远程文件时间戳，只将远端最新文件下载。

-c 断点续传

--limit-rage=N 限速 N bytesBytes/second 也可以用k或m单位k/m作为单位表示，小文件不会起作用。

-w --wait=seconds 等待间隔，降低远端服务器负载。

--waitretry=seconds 请求失败重试时间
```


![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 