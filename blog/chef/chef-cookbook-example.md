<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-06-14
title: CentOS上创建使用cookbooks
tags: Chef
images: https://os4u.info/blog/img/sun.png
category: Chef
status: publish
summary: CentOS上创建使用cookbooks，多熟悉下。
-->


## 简介

Chef是一种专为自动化并控制大量计算机，可靠以及弹性而设计的配置管理系统。

前面几篇文章介绍了chef的环境搭建[https://os4u.info/blog/chef/chef-install-usage.html](https://os4u.info/blog/chef/chef-install-usage.html)

如何创建chef pushjobs服务[https://os4u.info/blog/chef/chef-push-jobs.html](https://os4u.info/blog/chef/chef-push-jobs.html)

在本文中将介绍一些cookbbok的基本使用。Cookbooks是能在远程节点上配置并运行的配置管理单元，可以创建一系列的操作并指定节点运行。



## Cookbook 基本概念

Cookbooks常用于管理指定服务，应用，以及一些功能。举例而言，它可以用作NTP同步服务，它还可以安装配置数据库应用，还可以安装一些基础包。

Cookbooks是在workstation机器上配置并上传至Chef server。因此，cookbook的recipes（菜谱）和policies（策略）可以指定到节点的run-list中，run-list是chef-client触发一系列recipes和roles（角色）且运行在node节点完成设置的policy的清单列表。

通过这种方式，你可以预先在cookbook中定义应用配置场景，cookbook是有一些默认的目录结构，下面来简要介绍下

### Recipes

recipes是cookbook的主要工作目录，一个cookbook包含多个recipe，或者依赖外部recipes。recipes使用来描述不同资源的状态。

resources用于描述系统以及设置的状态一部分。举例子，resources可以描述为："the package x should be installed"。另外一个resource可以描述为："the x service should be running"

recipe是一系列告知系统如何处理的resources清单，当chef运行recipe，将检查每个resource来完成声明的状态，如果匹配了，将继续下一个匹配操作，否则，将运行至指定状态。

resources中常见配置如下：

```
package: 管理node节点的pakcage
service: 管理node节点的服务
user: 管理节点用户
group: 管理节点群组
template: 管理嵌入的ruby模板文件
cookbook_file: Transfer files from the files subdirectory in the cookbook to a location on the node
file: 管理node节点上文件的内容
directory: 管理node节点上的目录
execute: 在node节点上运行命令
cron: 编辑node节点的一个计划任务
```

### Attributes

attributes（属性）是chef的基本配置，可以看出在使用cookbook时设置的健值对

有几种属性可以被应用，每一种属于不同的级别。在cookbook级别，我们只是定义默认配置服务或系统的属性，可以被覆盖。

### Files

files文件目录包含任何将被存放到node节点的文件。


### Templates

templates 与files相似，但区别在于，template不是静态的，template文件以.erb结尾。也就是说，这些文件是嵌套到ruby文件中的。

### Metadata.rb

metadata.rb文件是毫无疑问被用于管理package的metadata文件。主要包括package的名称，一些描述信息等等

metadata.rb也包含一些像依赖信息，可以指定哪些cookbooks需要操作。这些将允许chef server正确的为node创建run-list

## 创建一个简单的Cookbook

为了更深入的理解和使用cookbooks，下面将创建一个ntp服务的实际例子。

***说明：*** 工作目录为：/data/work-chef/chef-repo/cookbooks

```
$ cd /data/work-chef/chef-repo/cookbooks
```
使用chef创建cookbook。knife工具既可以在workstation上工作，又可以连接到chef server或者单独node节点。

chef创建cookbook的一些基本命令：

```
chef generate cookbook cookbook_name
```
***备注：*** knife（从12.19.36版本开始）创建cookbook命令已被弃用，主要使用chef命令创建cookbook。

创建名称为ntp的cookbook。

```
$ chef generate cookbook ntp
Generating cookbook ntp
- Ensuring correct cookbook file content
- Ensuring delivery configuration
- Ensuring correct delivery build cookbook content

Your cookbook is ready. Type `cd ntp` to enter it.

There are several commands you can run to get started locally developing and testing your cookbook.
Type `delivery local --help` to see a full list.

Why not start by writing a test? Tests for the default recipe are stored at:

test/smoke/default/default_test.rb

If you'd prefer to dive right in, the default recipe can be found at:

recipes/default.rb

```

***备注***

```
$ chef generate app APP_NAME (options)
$ chef generate cookbook COOKBOOK_PATH/COOKBOOK_NAME (options)
$ chef generate build-cookbook COOKBOOK_PATH/COOKBOOK_NAME (options)
$ chef generate file COOKBOOK_PATH NAME (options)
$ chef generate lwrp COOKBOOK_PATH NAME (options)
$ chef generate recipe COOKBOOK_PATH NAME (options)
$ chef generate repo REPO_NAME (options)
$ chef generate template COOKBOOK_PATH NAME (options)

Usage: chef generate GENERATOR [options]

Available generators:
  app             Generate an application repo
  cookbook        Generate a single cookbook
  recipe          Generate a new recipe
  attribute       Generate an attributes file
  template        Generate a file template
  file            Generate a cookbook file
  lwrp            Generate a lightweight resource/provider
  repo            Generate a Chef code repository
  policyfile      Generate a Policyfile for use with the install/push commands
  generator       Copy ChefDK's generator cookbook so you can customize it
  build-cookbook  Generate a build cookbook for use with Delivery
  
```

进入ntp目录，查看目录：

```
$ cd ntp/
$ ls
Berksfile  chefignore  metadata.rb  README.md  recipes  spec  test

```

### 创建简单的recipe

```
$ cd recipes
$ ls
default.rb
```

#### 1) 使用vim编辑工具打开，内容如下：

```
#
# Cookbook:: ntp
# Recipe:: default
#
# Copyright:: 2017, The Authors, All Rights Reserved.
#

```

添加如下内容：

```
package 'chrony' do
  action :install
end
```
该小段代码定义一个ntp package资源。action :install 是告知节点安装chrony

添加配置文件：

```
cookbook_file "/etc/chrony.conf" do
  source "chrony.conf"
  mode "0644"
end
```

安装完成后，需要开启该服务，添加以下内容：

```
service 'chronyd.service' do
  action [:enable, :start]
end

```


配置完成后，保存并退出。

#### 创建chrony.conf

```
$ cd /data/work-chef/chef-repo/cookbooks/ntp/
$ chef generate file .
$ cd ./files/default
$ vim chrony.conf
```

内容如下：

```
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server ntp.os4u.info iburst

# Ignore stratum in source selection.
stratumweight 0

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Enable kernel RTC synchronization.
rtcsync

# In first three updates step the system clock instead of slew
# if the adjustment is larger than 10 seconds.
makestep 10 3

# Allow NTP client access from local network.
#allow 192.168/16

# Listen for commands only on localhost.
bindcmdaddress 127.0.0.1
bindcmdaddress ::1

# Serve time even if not synchronized to any NTP server.
#local stratum 10

keyfile /etc/chrony.keys

# Specify the key used as password for chronyc.
commandkey 1

# Generate command key if missing.
generatecommandkey

# Disable logging of client accesses.
noclientlog

# Send a message to syslog if a clock adjustment is larger than 0.5 seconds.
logchange 0.5

logdir /var/log/chrony
#log measurements statistics tracking
```

配置完成后，保存退出。

### 创建一个Helper Cookbook

安装ntp服务之前，可能由于长期未更新软件仓库而导致安装出现失败等情况。因此，运行之前需要检查。

```
$ cd /data/work-chef/chef-repo/cookbooks
$ chef generate cookbook yum
```

编辑recipe

```
vim /data/work-chef/chef-repo/cookbooks/yum/recipes/default.rb
```
内容如下：

```
execute "yum update -y" do
  command "yum update -y"
end
```
完成后，保存退出。

完成该cookbook后，添加到ntp的cookbook中，编辑ntp的recipe。

```
$ cd /data/work-chef/chef-repo/cookbooks/ntp
$ vim recipes/default.rb
```
在顶部添加：

```
include_recipe "yum"
```

添加完成后，default.rb的内容如下：

```
#
# Cookbook:: ntp
# Recipe:: default
#
# Copyright:: 2017, The Authors, All Rights Reserved.
#

include_recipe "yum"

package 'chrony' do
  action :install
end

service 'nginx' do
  action [:enable, :start]
end

cookbook_file "/etc/chrony.conf" do
  source "chrony.conf"
  mode "0644"
end
```
编辑完成后保存退出。

编辑ntp的metadata.rb文件，添加yum cookbook的依赖说明。

```
$ cd /data/work-chef/chef-repo/cookbooks/ntp
$ vim metadata.rb
```
内容如下：

```
name 'ntp'
maintainer 'os4uinfo'
maintainer_email 'os4uinfo@gmail.com'
license 'All Rights Reserved'
description 'Installs/Configures ntp'
long_description 'Installs/Configures ntp'
version '0.1.0'
chef_version '>= 12.1' if respond_to?(:chef_version)

depends "yum"

```

完成编辑后，保存退出。至此解决了chrony软件安装的问题了。

### 将cookbooks添加到node节点

首先，将cookbook上传到chef server。使用如下命令：`knife cookbook upload COOKBOOKS_NAME`

```
$ knife cookbook upload yum
$ knife cookbook upload ntp
```
或者采用一键上传的命令：

```
$ knife cookbook upload -a
```
这种方式将上传所有的cookbook到chef server。

至此，可以修改node节点的run-list了。具体命令如下：

```
$ knife node edit name_of_node
```
查看node列表

```
$ knife node list
client1
```

本例子中测试目的，编辑client1

```
$ knife node edit client1
{
  "name": "client1",
  "chef_environment": "_default",
  "normal": {
    "tags": [

    ]
  },
  "run_list": [

  ]
}
```

将ntp添加其中，完成后结果如下：

```
{
  "name": "client1",
  "chef_environment": "_default",
  "normal": {
    "tags": [

    ]
  },
  "run_list": [
    "recipe[ntp]"
  ]
}
```

保存关闭编辑

使用cookbooks：

* 方案1: ssh登录到服务器执行`chef-client`命令。

```
$ sudo chef-client
Starting Chef Client, version 13.1.31
resolving cookbooks for run list: ["ntp"]
Synchronizing Cookbooks:
  - push-jobs (5.1.2)
  - yum-epel (2.1.1)
  - yum (0.1.0)
  - runit (3.0.5)
  - compat_resource (12.19.0)
  - chef-ingredient (2.1.2)
  - packagecloud (0.3.0)
  - ntp (0.1.1)
Installing Cookbook Gems:
Compiling Cookbooks...
/var/chef/cache/cookbooks/packagecloud/resources/repo.rb:10: warning: constant ::Fixnum is deprecated
Converging 10 resources
Recipe: yum::default
  * execute[yum update -y] action run
    - execute yum update -y
Recipe: ntp::default
  * yum_package[chrony] action install (up to date)
  * cookbook_file[/etc/chrony.conf] action create (up to date)
  * service[chronyd.service] action enable (up to date)
  * service[chronyd.service] action start (up to date)
 
```

* 方案2: 在workstation上运行(前提是client1上安装了push job）

```
添加recipes到client1
$ knife bootstrap client1  --ssh-user vagrant  --sudo --use-sudo-password  -r 'recipe[ntp]'
$ 执行cookbooks
knife job start --timeout 300 'chef-client' client1
Started.  Job ID: 5dcae323468a8954fbb68fb2ee10eeca
.Running (1/1 in progress) ...
.........Complete.
command:     chef-client
created_at:  Wed, 14 Jun 2017 00:44:08 GMT
env:
id:          5dcae323468a8954fbb68fb2ee10eeca
nodes:
  succeeded: client1
run_timeout: 300
status:      complete
updated_at:  Wed, 14 Jun 2017 00:44:18 GMT

```
 
***说明：*** 有多种方案使得node运行该cookbook。

查看是否成功

```
chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===========================================
^? ntp.os4u.info   0  8  0  10y  +0ns[  +0ns] +/-  0ns
```


## 总结

尽管是个简单的ntp服务同步的例子，但是入门chef的有用的例子。希望多理解多练习，熟练掌握。

参考资料：
> [how-to-create-simple-chef-cookbooks-to-manage-infrastructure-on-ubuntu](https://www.digitalocean.com/community/tutorials/how-to-create-simple-chef-cookbooks-to-manage-infrastructure-on-ubuntu)
> 
> [chef-generate](https://docs.chef.io/ctl_chef.html#chef-generate-template)
> 