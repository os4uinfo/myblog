<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-16
title: Openstack学习笔记（持续更新中）
tags: Openstack
images: https://os4u.info/blog/img/sun.png
category: Openstack
status: publish
summary: Openstack是一个趋势，每个运维都应该掌握的技能。以此为前提，不断学习，不断实验，不断总结，才能提高自己知识技能。
-->

##### 从导入ISO镜像到安装系统

1.导入镜像

```
$ openstack image create "gsm_ce_4.0.5" --public --disk-format iso \
--container-format bare --file gsm_ce_4.0.5.iso
```

2.检查flavor，（磁盘模板）

```
$ openstack flavor list
```

3.获取网络信息ID

```
$ openstack network list
```

4.创建系统

```
$ openstack server create --flavor m1.micro --image "gsm_ce_4.0.5" \
--nic net-id=ac0707a2-cee1-478a-835b-79a2b2291a92 \
--security-group default --key-name mykey "Greenbone Security Manager"

```
![greenbone-security-manager](https://www.os4u.info/blog/openstack/images/greenbone-security-manager.png)
