<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-15
title: Docker容器的故障检测和排除
tags: Docker
images: https://os4u.info/blog/img/sun.png
category: Docker 
status: publish
summary: 有时候，仪表化并不够，我们应该以一种可扩展方式对Docker部署进行故障检测和排除。然而，有时候，我们只能等来到Docker宿主机上来查看Docker容器，除此之外，别无他法。
-->


![nature water](https://www.os4u.info/blog/docker/images/nature-water.jpg)
##### 检查容器

典型工作流：

```
1.登入到宿主机；
2. 使用docker exec进入指定、运行中的容器进程命名空间。作为调试应用最后的一种手段，还是很有效的。
```

常用命令:

```
1.执行基础命令
$ docker exec dockername /bin/ss -l
$ docker exec dockername /usr/bin/netstat -tlnp 

2.查看docker引擎日志
$ journalctl -u docker.service -o cat 

3.交互docker
$ docker exec -it  dockername /bin/bash

```

docker exec 命令的局限性：

```
1. 停止当前容器，重建容器，在执行过程中安装的一些命令会失效;
2. 如果在镜像中打包所有的调试工具，那么镜像会增大；
3. 有些最小化容器可能没有bash shell.
```

##### 从外部调试

```
尽管Docker隔离了容器的网络、内存、存储资源。但是每个单独的容器仍需要Docker宿主机操作系统来执行命令。
利用这一点，我们从外部检查调试Docker容器。
```

###### 追踪系统调用

```
$ PID=`docker inspect -f '{{ .State.Pid }}' dockername`
$ strace -p $PID

```

###### 分析网络数据包

```
大多数Docker容器都提供了某种形式的网络服务，无论何种容器，数据包最终通过Docker容器出发。
通过打印并分析数据包的内容，我们就可以了解容器的本质。
$ tcpdump -i docker0
¥ ip addr show dev docker0

```

###### 观察块设备
```
Docker将数据存储在物理设备上，我们观察特殊的I/O行为和性能问题，使用blktrace工具来追踪，检查故障
和排除块设备中的问题。
需要开启文件系统的调试选项，在Docker宿主机上，输入如下命令：
$ mount -t debugfs debugfs /sys/kernel/debug

查询docker运行时的根文件在哪，后续便能追踪容器的I/O事件。
$ df -h
可以看出，docker宿主机的/var/lib/docker目录时在这个/分区下。这就是blktrace的监听事件的位置。
$ blktrace -d /dev/dm-0 -o dump

例子：
创建一次I/O事件，
$ docker run -d --name dump debian:jessie \
/bin/dd if=/dev/zero of=/root/dump bs=65000
获取容器PID
$ docker inspect -f '{{.State.Pid}}' dump

得到产生I/O事件的容器PID，可以使用blktrace的辅助工具blkparse。blktrace程序知识监听linux
内核的块I/O层，并将结果输出到文件中。blkparse可以查看和分析这些事件。
$ blkparse -d dump.blktrace.0 | grep --color " $PID "
可以看出有多少字节的写入（W），并且已经完成（C）的，进一步调查的话，可以查看产生的偏移量位置
$ blkparse -i dump.blktrace.0 | grep xxxx
可以看到写入（W）已经被kworker进程放入了设备的队列（Q）中，也就是说，写入操作被内核加入到了
队列中，在40毫秒之后，docker容器进程的该写请求完成了。

$ docker stats产生的性能指标相比，这些真实的事件更能深入故障检查和排除。
```

##### 故障检测和排除工具

```
RedHat的rhel-tools Docker镜像是一个巨大的容器，集成了我们之前提到的工具，
CoreOS的toolbox是一个精简的脚本工具
nsenter工具可以进入linux控制组（control group）的进程空间，它是docker exec前身。
```