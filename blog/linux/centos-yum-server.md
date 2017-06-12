<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-06-04
title: yum server on centos.
tags: yum
images: https://os4u.info/blog/img/sun.png
category: yum
status: publish
summary: 搭建CentOS在线yum源镜像服务器
-->

#### 搭建CentOS在线yum源镜像服务器

```
说明：

操作系统：CentOS 7.x

IP地址：192.168.30.62

实现目的：同步CentOS镜像站点的内容到此服务器，并且通过配置http服务器，能够向外提供yum服务
```

##### 1) 安装http服务器

```
a. compile
# mkdir /data/tmp/ && cd /data/tmp;
# wget http://nginx.org/download/nginx-1.12.0.tar.gz
# tar zxf nginx-1.12.0.tar.gz 
# cd nginx-1.12.0
# ./configure --prefix=/data/apps/nginx --with-http_stub_status_module\
 --with-http_ssl_module --with-file-aio --with-http_realip_module

b. configure
--------------------------------------------------------------
server {
        listen       80;
        server_name  mirrors.os4u.info;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            autoindex on;
            root   /data/repos;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
}

--------------------------------------------------------------

c.start nginx
# nginx 
d. 
# yum install createrepo
# reposync -p /data/repos/centos/7/  ### long time to sync
# createrepo -p /data/repos/centos/7/base/Packages
# createrepo -p /data/repos/centos/7/extras/Packages
# createrepo -p /data/repos/centos/7/updates/Packages
# createrepo -p /data/repos/centos/7/epel
# crontab -e
----
1 2 * * * /usr/bin/reposync -np /data/repos/centos/7/

----

# rm -rf  /etc/localtime 
# ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

##### 2) 

```
----------------------------------------------------------------------------------
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
name=CentOS-$releasever - Base - ue.cn
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
baseurl=http://mirrors.os4u.info/centos/$releasever/base/Packages/
gpgcheck=1
gpgkey=http://mirrors.os4u.info/centos/$releasever/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates - ue.cn
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
baseurl=http://mirrors.os4u.info/centos/$releasever/updates/Packages/
gpgcheck=1
gpgkey=http://mirrors.os4u.info/centos/$releasever/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - ue.cn
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
baseurl=http://mirrors.os4u.info/centos/$releasever/extras/Packages/
gpgcheck=1
gpgkey=http://mirrors.os4u.info/centos/$releasever/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - ue.cn
baseurl=http://mirrors.os4u.info/centos/$releasever/centosplus/
gpgcheck=0
enabled=1

----------------------------------------------------------------------------------

```


![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 