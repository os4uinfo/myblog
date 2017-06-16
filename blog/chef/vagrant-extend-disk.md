<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-06-16
title: vagrant管理虚拟机--扩展磁盘
tags: Chef
images: https://os4u.info/blog/img/sun.png
category: Chef
status: publish
summary: 了解下vagrant管理虚拟机扩展磁盘。
-->


1. 原理：
   转换原始vmdk镜像为可扩展的vdi镜像，挂载虚拟磁盘，利用pv(物理卷)、vg(卷组)、lv(逻辑卷)实现根目录或其他目录的扩展。
 
2. 转换并扩展vdi
   停止虚拟机，此处mytest虚拟机为例

## 1) 关闭虚拟机

```
$ cd ~/VirtualBox VMs/mytest 
$ vagrant halt
```

## 2) 创建磁盘

```
// 进入虚拟机目录
$ cd ~/VirtualBox VMs/mytest  
$ VBoxManage clonehd box-disk1.vmdk box-disk1.vdi --format vdi  
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Clone medium created in format 'vdi'. UUID: 85a965b3-d756-4ec4-98b1-94d12503b3e0
//扩展磁盘 (从 2GB 到 100GB => 1024*100)
$ VBoxManage modifyhd box-disk1.vdi --resize 102400
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100% 
```

## 3) 添加到虚拟机
```
# VBoxManage storageattach mytest_default_1496903743943_83055 --storagectl "IDE" --port 0 --device 1 --type hdd --medium box-disk1.vdi  
```

## 4) 开启机器

```
$ cd ~/VirtualBox VMs/mytest 
$ vagrant up
```

## 5) 格式化磁盘

```
略
```

### 参考资料：
> [https://tvi.al/resize-sda1-disk-of-your-vagrant-virtualbox-vm/](https://tvi.al/resize-sda1-disk-of-your-vagrant-virtualbox-vm/)
> 
> [http://blog.csdn.net/zxjxingkong/article/details/62419379](http://blog.csdn.net/zxjxingkong/article/details/62419379)


![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 