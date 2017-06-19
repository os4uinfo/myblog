<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-06-10
title: ssh的一些使用
tags: ssh 
images: https://os4u.info/blog/img/sun.png
category: Linux
status: publish
summary: ssh学习，本节主要介绍ssh的一些技巧。
-->

![learning-ssh](https://www.os4u.info/blog/linux/images/learning-ssh.png)
##### 1.ssh 代理

1）配置文件修改

```
/etc/ssh/ssh_config
```

2)用户自定义配置文件修改

```
~/.ssh/config
```
3) 运行命令时修改

```
ssh -o "ForwardAgent yes" ...
ssh -A ...
```
禁用代理 `ssh -a ...`

##### 2.创建ssh透明连接

模型： A---> B ---> C

###### a)在A机器上配置~/.ssh/config

```
host C
  hostname B
  user root
```

###### b)在B机器上关联一个强制命令~/.ssh/authorized_keys

```
command="ssh -l root C" ...key...
```

##### 3.跨网关使用SCP

在上述模型中B为网关

###### 1)解决传递远程命令
在B机器上：~/.ssh/authorized_keys
```
command="sh -c 'ssh -l root C ${SSH_ORIGINAL_COMMAND:-}'" ...key...
```
###### 2）解决认证
在B机器上代理转发打开
```
~/.ssh/config
ForwardAgent yes
```

##### 4.端口转发

在A机器上执行

```
$ ssh -L2001:C:22 B
```
在A机器上重开一个shell

```
$ ssh -p 2001 localhost
```

在A机器上~/.ssh/config

```
host C
  hostname localhost
  port 2001
```
如果配置后，上述`$ ssh -p 2001 localhost`命令可以替换为：`ssh C`


参考：
[ssh-agent](http://linux.101hacks.com/unix/ssh-agent/)


![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 