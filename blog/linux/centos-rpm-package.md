<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-06-06
title: 制作自定义rpm软件包
tags: yum
images: https://os4u.info/blog/img/sun.png
category: yum
status: publish
summary: 本例子是将私有repository打包成rpm，方便后续使用。
-->

##### 制作自定义rpm软件包

![summer-nature](https://www.os4u.info/blog/linux/images/summer-nature.jpg)
```

说明：
系统环境：CentOS 7

```

##### 1.打包安装软件安装

```
# yum install rpmdevtools yum-utils rpm-build

```

##### 2.构建环境
```
# cd ~
# rpmdev-setuptree
执行完成后有以下目录：
# tree -L 1
.
|-- BUILD
|-- RPMS
|-- SOURCES
|-- SPECS
`-- SRPMS 
```
##### 3.开始创建

```
例子说明：本例子是将私有repository打包成rpm，方便后续使用。

a. 准备原材料
   # cd SOURCES
   //创建要求：软件名+版本
   # mkdir centos-ue-yum-source-1
   # cd centos-ue-yum-source-1
   # vim CentOS-Base-Ue.repo
   ------------------------------------------------------------------------
		# CentOS-Base.repo
		#
		# The mirror system uses the connecting IP address of the client and the
		# update status of each mirror to pick mirrors that are updated to and
		# geographically close to the client.  You should use this for CentOS updates
		# unless you are manually picking other mirrors.
		#
		# If the mirrorlist= does not work for you, as a fall back you can try the
		# remarked out baseurl= line instead.
		#
		#
		[base]
		name=CentOS-$releasever - Base - os4u.info
		#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
		baseurl=http://mirrors.os4u.info/centos/$releasever/base/Packages/
		gpgcheck=1
		gpgkey=http://mirrors.os4u.info/centos/$releasever/RPM-GPG-KEY-CentOS-7
		
		#released updates
		[updates]
		name=CentOS-$releasever - Updates - os4u.info
		#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
		baseurl=http://mirrors.os4u.info/centos/$releasever/updates/Packages/
		gpgcheck=1
		gpgkey=http://mirrors.os4u.info/centos/$releasever/RPM-GPG-KEY-CentOS-7
		
		#additional packages that may be useful
		[extras]
		name=CentOS-$releasever - Extras - os4u.info
		#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
		baseurl=http://mirrors.os4u.info/centos/$releasever/extras/Packages/
		gpgcheck=1
		gpgkey=http://mirrors.os4u.info/centos/$releasever/RPM-GPG-KEY-CentOS-7
		
		#additional packages that extend functionality of existing packages
		[centosplus]
		name=CentOS-$releasever - Plus - os4u.info
		baseurl=http://mirrors.os4u.info/centos/$releasever/centosplus/
		gpgcheck=0
		enabled=1
	------------------------------------------------------------------------	# cd ..
	# tar czf centos-ue-yum-source-1.tar.gz centos-ue-yum-source-1/

b. 编写spec文件
	# cd SPECS/
	# vim ue-yum.spec
	------------------------------------------------------------------------
		Name:		centos-ue-yum-source
		Version:	1
		Release:	0
		Summary:	CentOS yum server by UE.ltd
		
		Group:		UE
		License:	GPL
		URL:		http://mirrors.os4u.info
		Source0:	centos-ue-yum-source-1.tar.gz
		
		# 系统定义
		BuildArch:	noarch
		
		%description
		Add private CentOS repository.
		
		# 这两句是要写的，否则会报错
		%define debug_package %{nil}
		%define _binaries_in_noarch_packages_terminate_build   0
		
		%prep
		%setup -q
		
		
		%build
		
		%install
		install -m 0755 -d $RPM_BUILD_ROOT/etc/yum.repos.d/
		install -m 0755 CentOS-Base-Ue.repo $RPM_BUILD_ROOT/etc/yum.repos.d/
		
		%clean
		rm -rf $RPM_BUILD_ROOT
		
		%post
		echo .Greate, Thank you use our repository.
		
		%files
		/etc/yum.repos.d/
		/etc/yum.repos.d/CentOS-Base-Ue.repo
		
		
		
		%changelog
	------------------------------------------------------------------------
	# cd ..

c. 构建
	# rpmbuild -ba SPECS/ue-yum.spec
	... 输出省略

d. 拷贝出打包好的rpm文件,即可安装到系统
   # cp RPMS/noarch/centos-ue-yum-source-1-0.noarch.rpm  ~
   	
```
##### 4. 将打包好的rpm包，放到私有仓库中

```
# mv centos-ue-yum-source-1-0.noarch.rpm /data/repos/centos/7/centosplus/
更新repository
# createrepo -p /data/repos/centos/7/centosplus
```
##### 5. 在客户端查询
```
# yum search centos-ue
Loaded plugins: fastestmirror, ovl
Loading mirror speeds from cached hostfile
======================= N/S matched: centos-ue ================================
centos-ue-yum-source.noarch : CentOS yum server by UE.ltd

  Name and summary matches only, use "search all" for everything.
  
# yum install centos-ue-yum-source.noarch
```
##### 6. 参考
[rpm-build-package-example](http://www.thegeekstuff.com/2015/02/rpm-build-package-example/)
