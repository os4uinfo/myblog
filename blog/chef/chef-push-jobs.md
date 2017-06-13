<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-06-13
title: Chef使用之push-jobs安装使用
tags: Chef
images: https://os4u.info/blog/img/sun.png
category: Chef
status: publish
summary: Chef 是一款自动化服务器配置管理工具,理论上可以对服务器做任何配置,包括系统管理、安装软件等,近来已被越来越多地应用到云环境的自动化部署上。运维必备自动化神器之一，所以还是要学习下。
-->


![](https://www.os4u.info/blog/chef/images/overview_push_jobs_states.png)

1) 在chef server端安装push-job-server和reporing组件，方便后续web查看执行报告。

```
$ sudo yum install opscode-push-jobs-server.x86_64
$ sudo yum install opscode-reporting.x86_64
$ sudo opscode-push-jobs-server-ctl reconfigure
$ sudo opscode-reporting-ctl reconfigure
$ sudo chef-server-ctl reconfigure  
```

2) 创建push-jobs的cookbbok（下载）

```
$ cd /data/work-chef/chef-repo/cookbooks
$ knife cookbook site search push-jobs
$ knife cookbook site download push-jobs
Downloading push-jobs from Supermarket at version 5.1.1 to /data/work-chef/chef-repo/cookbooks/push-jobs-5.1.1.tar.gz
Cookbook saved: /data/work-chef/chef-repo/cookbooks/push-jobs-5.1.1.tar.gz
$ tar zxf push-jobs-5.1.1.tar.gz
```

3）修改

```
$ vim push-jobs/attributes/default.rb
default['push_jobs']['package_url'] = 'https://www.os4u.info/centos/7/centosplus/push-jobs-client-2.1.4-1.el7.x86_64.rpm'
default['push_jobs']['package_checksum']  = '09a4cdf1ec01536e6ecc3bedb018f9677889171ff8853106e4ff47374e6ce951'

```

4）将push-jobs上传到chef-server

```
$ knife cookbook upload push-jobs 
```

5) 将需要安装的（chef node）客户端添加菜单

```
$ knife node run_list add openstack-block  recipe[push-jobs]
$ knife node show openstack-block
Node Name:   openstack-block
Environment: _default
FQDN:        openstack-block
IP:          10.0.2.15
Run List:    recipe[push-jobs]
Roles:
Recipes:     push-jobs, push-jobs::default, push-jobs::install, push-jobs::package, push-jobs::config, push-jobs::service
Platform:    centos 7.3.1611
Tags:
```

6) 运行

方法1: 在chef node（客户端）执行

```
$ sudo chef-client
```

方法2: 在chef workstation执行

```
$ knife bootstrap 10.0.2.15 --ssh-user vagrant  --sudo --use-sudo-password -N openstack-block -r 'recipe[push-jobs]'

```

7) 查看chef node的push job状态

```
$ knife node status
openstack-block	available
```

8）push job的一些常用命令

```
knife job list 
knife job output <job id> <node> [<node> ...]
knife job start <command> [<node> <node> ...]
knife job status <job id>


knife job output
knife job start --timeout 10 'chef-client' openstack-block
```

***参考资料：***

[https://d2avhevdwylo4t.cloudfront.net/reporting.html](https://d2avhevdwylo4t.cloudfront.net/reporting.html)

[https://blog.chef.io/2015/03/16/using-chef-supermarket-a-guided-tour/](https://blog.chef.io/2015/03/16/using-chef-supermarket-a-guided-tour/)

[https://docs.chef.io/knife_bootstrap.html](https://docs.chef.io/knife_bootstrap.html)

[https://docs.chef.io/push_jobs.html](https://docs.chef.io/push_jobs.html)

![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 