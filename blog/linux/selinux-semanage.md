<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-09
title: Selinux管理工具semanage 
tags: Security
images: https://os4u.info/blog/img/sun.png
category: Linux
status: publish
summary: Selinux极大的增强了Linux系统的安全性，能将用户权限关在笼子里。
-->

---
![image](https://os4u.info/blog/linux/images/nature.jpg)

```
Selinux极大的增强了Linux系统的安全性，能将用户权限关在笼子里。

如httpd服务，apache默认只能访问/var/www目录，并只能监听80和443端口，因此能有效的防范0-day类的攻击。

举例来说，系统上的 Apache 被发现存在一个漏洞，使得某远程用户可以访问系统上的敏感文件(比如 /etc/passwd 来获得系统已存在用户)。

而修复该安全漏洞的 Apache 更新补丁尚未释出。

此时 SELinux 可以起到弥补该漏洞的缓和方案。因为 /etc/passwd 不具有 Apache 的访问标签，所以 Apache 对于 /etc/passwd 的访问会被 SELinux 阻止。

CentOS系统自带的chcon工具只能修改文件、目录等的文件类型和策略，无法对端口、消息接口和网络接口等进行管理。

semanage能有效胜任SELinux的相关配置工作。
```

安装：

```
# yum instsall -y semanage
# yum provides /usr/sbin/semanage
# yum -y install policycoreutils-python

```
基本使用：

管理登录linux的用户和SELinux局限的用户之间的映射。

```
semanage login [-S store] -{a|d|m|l|n|D} [-sr] login_name | %groupname
```
管理策略模块：

```
semanage module [-S store] -{a|d|l} [-m [--enable | --disable] ] module_name
```
管理网络端口类型定义

```
semanage port [-S store] -{a|d|m|l|n|D} [-tr] [-p proto] port | port_range
```
例：如apache采用非标准端口，需执行如下命令：

```
emanage port -a -t http_port_t -p tcp port_number
semanage port -a -t http_port_t -p tcp 8091
```
查看当前允许的httpd端口：

```
# semanage port -l|grep http
http_cache_port_t tcp 3128, 8080, 8118, 8123, 10001-10010
http_cache_port_t udp 3130
http_port_t tcp 8888, 80, 443, 488, 8008, 8009, 8443
pegasus_http_port_t tcp 5988
pegasus_https_port_t tcp 5989
```
注意：8888是我刚才添加的

管理网络接口类型定义

```
semanage interface [-S store] -{a|d|m|l|n|D} [-tr] interface_spec
```
管理网络节点类型定义

```
semanage node [-S store] -{a|d|m|l|n|D} [-tr] [ -p protocol ] [-M netmask] address
```
管理文件中映射定义

```
semanage fcontext [-S store] -{a|d|m|l|n|D} [-frst] file_spec
semanage fcontext [-S store] -{a|d|m|l|n|D} -e replacement target
```
例：让 Apache 可以访问位于非默认目录下的网站文件

首先，用 semanage fcontext -l | grep '/var/www' 获知默认 /var/www 目录的 SELinux 上下文：

```
/var/www(/.*)? all files system_u:object_r:httpd_sys_content_t:s0
```
从中可以看到 Apache 只能访问包含 httpd_sys_content_t 标签的文件。

假设希望 Apache 使用 /srv/www 作为网站文件目录，那么就需要给这个目录下的文件增加 httpd_sys_content_t 标签，分两步实现。

首先为 /srv/www 这个目录下的文件添加默认标签类型：semanage fcontext -a -t httpd_sys_content_t '/srv/www(/.*)?' 然后用新的标签类型标注已有文件：restorecon -Rv /srv/www 之后 Apache 就可以使用该目录下的文件构建网站了。

其中 restorecon 在 SELinux 管理中很常见，起到恢复文件默认标签的作用。比如当从用户主目录下将某个文件复制到 Apache 网站目录下时，Apache 默认是无法访问，因为用户主目录的下的文件标签是 user_home_t。此时就需要 restorecon 将其恢复为可被 Apache 访问的 httpd_sys_content_t 类型：

```
restorecon -v /srv/www/html/file.html
restorecon reset /srv/www/html/file.html context unconfined_u:object_r:user_home_t:s0->system_u:object_r:httpd_sys_content_t:s0
semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?" //新建一条规则，指定/web目录及其下的所有文件的扩展属性为httpd_sys_content_t
```

![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 