<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-20
title: Test Kitchen管理沙盒测试环境
tags: Chef
images: https://os4u.info/blog/img/sun.png
category: Chef
status: publish
summary: Chef 是一款自动化服务器配置管理工具,理论上可以对服务器做任何配置,包括系统管理、安装软件等,近来已被越来越多地应用到云环境的自动化部署上。运维必备自动化神器之一，所以还是要学习下。
-->

![chef-logo](https://www.chef.io/wp-content/uploads/2017/02/sub-hero-logo-01.svg)

1.安装vagrant

```
$ sudo apt-get install vagrant
$ vagrant --version
```

2.安装virtualbox

```
$ sudo apt-get install virtualbox
$ VBoxManage --version

```

3.下载CentOS对应的vagrant box文件

```
$ wget http://cloud.centos.org/centos/7/vagrant/x86_64/images/CentOS-7-x86_64-Vagrant-1704_01.VirtualBox.box
```
4.把下载的box添加到vagrant中

编写metadata.json文件，格式如下：

```
{
    "name": "CentOS7",
    "versions": [{
        "version": "1704.01",
        "providers": [{
             "name": "virtualbox",
             "url": "file:///data/vagrant/CentOS-7-x86_64-Vagrant-1704_01.VirtualBox.box"
         }]
    }]
}

--------------------------------------------
说明：
name 是镜像名称
versions 版本描述信息
versions.version 是版本号
versions.providers 是配置提供者信息，如此处是virtualbox
versions.url 这里可以是http协议，也可以是本地file协议
```

5.初始化kitchen

```
$ kitchen init --create-gemfile
$ bundle install 

```

6.配置.kitchen.yml

```
---
driver:
  name: vagrant
  
provisioner:
  name: chef_solo

platforms:
  - name: CentOS7.3
    drivers:
      box: CentOS7 //此处的box名称是与上面导入的box名称一致
  - name: CentOS6.8 //默认会从互联网上下载对应的box
suites:
  - name: default
    run_list:
    attributres:
    
```

7.创建虚拟机，并登陆

```
$ kitchen list
$ kitchen create default-CentOS70
$ kitchen login default-CentOS70
```



至此，Test Kitchen管理沙盒测试环境 搭建完成。


#### vagrant 单独使用，创建沙盒环境

1.把下载的box添加到vagrant中

编写metadata.json文件，格式如下：

```
{
    "name": "CentOS7.0",
    "versions": [{
        "version": "1704.01",
        "providers": [{
             "name": "virtualbox",
             "url": "file:///data/vagrant/CentOS-7-x86_64-Vagrant-1704_01.VirtualBox.box"
         }]
    }]
}

--------------------------------------------
说明：
name 是镜像名称
versions 版本描述信息
versions.version 是版本号
versions.providers 是配置提供者信息，如此处是virtualbox
versions.url 这里可以是http协议，也可以是本地file协议
```

2.执行命令导入

```
$ vagrant box add metadata.json
```

3.导入验证

```
$ vagrant box list
```

4.创建机器

```
$ vagrant box list 
// 根据列举的box信息决定初始化什么系统
$ vagrant init CentOS7.0
$ vagrant up // 启动
$ vagrant ssh // 登陆机器

```