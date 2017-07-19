<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-07-19
title: Ceph 源码打包
tags: Ceph
images: https://os4u.info/blog/img/sun.png
category: Ceph
status: publish
summary: Ceph是一个分布式开源块存储，文件存储，对象存储的存储解决方案。开放、可扩展、分布式的本质使得ceph逐渐变成一个主流的云存储解决方案。
-->

### 1. 创建ceph包
为方便学习ceph，特根据官方网站介绍，自行创建一个ceph包。

基于CentOS来完成实践。
首先，下载或克隆Ceph仓库。Debian/Ubuntu系统使用dpkg-buildpackage工具。

***【Tips】*** 在多核CPU中，使用-j 参数cpu核数*2。比如 -j4 是双核的参数。


### 2. ADVANCED PACKAGE TOOL (APT)

Debian/Ubuntu系统创建.deb安装包的话，方法如下：

```
#sudo apt-get install debhelper
```
安装完成debhelper，继续如下操作：

```
sudo dpkg-buildpackage
```



### 3. RPM PACKAGE MANAGER

使用rpm-build及rpmdevtools命令，具体步骤如下：

```
yum install rpm-build rpmdevtools
```

创建编译目录
```
rpmdev-setuptree
```

下载ceph源码包
```
wget -P ~/rpmbuild/SOURCES/ http://ceph.com/download/ceph-<version>.tar.bz2
```
或者使用欧洲镜像：

```
wget -P ~/rpmbuild/SOURCES/ http://eu.ceph.com/download/ceph-<version>.tar.bz2

```
解压rpm包所需的spec文件

```
tar --strip-components=1 -C ~/rpmbuild/SPECS/ --no-anchored -xvjf ~/rpmbuild/SOURCES/ceph-<version>.tar.bz2 "ceph.spec"
```

创建rpm包

```
rpmbuild -ba ~/rpmbuild/SPECS/ceph.spec
```