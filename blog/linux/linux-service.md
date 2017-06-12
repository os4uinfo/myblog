<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-28
title: linux系统硬件管理
tags: Linux
images: https://os4u.info/blog/img/sun.png
category: Linux
status: publish
summary: Linux运维中，常用命令解决实际的运维问题，持续更新，总结中。
-->


##### 1）系统信息

```
$ uname -a
Linux xxx 3.10.0-514.16.1.el7.x86_64 #1 SMP Wed Apr 12 15:04:24 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
* Linux  -s  内核名称 
* xxx    -n  主机名称
* 3.10.0-514.16.1.el7.x86_64 -r 内核发行版本号
* #1 SMP Wed Apr 12 15:04:24 UTC 2017 -v 操作系统具体版本
* x86_64 -m 机器硬件名称
* x86_64 -p 处理器名称
* x86_64 -i 硬件平台名称
* GNU/Linux -o 操作系统名称

实际用户和有效用户
实际用户 real user id 就是登陆shell那个时刻所使用的用户ID， 用who am i就是实际用户
登陆shell后，用su 或su - 切换到的用户ID，有效用户
$ who am i # 展示实际用户
$ whoami # 展示有效用户
用户组groups命令查看
who are you = who am i = who -m
w
who
who -H 便于阅读方式展示
who -b 上次系统启动时间
who -r 当前系统运行级别(runlevel)
who -l（--login) 系统登录进程
who -a 列出所有信息

```

service -运行 System V初始化脚本的工具

```
System V 

# service [service name] [start/stop/restart/status]

```

chkconfig 等级管理

标识数字| 等级名称| 描述
-------|------|-----
0 | 停机 | 关机使用
1| 单机模式| 系统管理维护
2| 无网络多用户模式| 禁用网络相关功能
3| 有网络多用户模式| 正常状态
4| 未使用，预留| 一般不使用
5| 图形化模式| 在等级3上，图形化展示
6| 重启| 系统重启

```
chkconfig --list 

chkconfig --level 35 ntpd on
chkconfig ntpd off 
chkconfig原理
在/etc/rc.d/有7个文件夹init.d  rc0.d  rc1.d  rc2.d  rc3.d  rc4.d  rc5.d  rc6.d对应的是7个等级所以可以理解了。
其实就是对应每个等级，软连接到哪个目录
* /etc/init.d/中包含所有可用服务
* /etc/rc.d/rcN.d 对应的等级
* /etc/rc.d/rcN.d全部是服务的软连接
* /etc/rc.d/rcN.d 软连接规则： 
   K + 整数 + 服务名 表示要关闭对应的服务
   S + 整数 + 服务名 表示要开启对应的服务
   K、S后的整数，可以是两位数，三位数，决定启动或关闭顺序，本质上是按ASCII顺序。
   S11，一定是在S15前启动
   S110，是S11后启动，S15前启动
   多个S99那么机会按照整个符号连接文件名来排序

```

##### 硬件查看

dmidecode

```
-t 硬件类型
# dmidecode -t
dmidecode: option requires an argument -- 't'
Type number or keyword expected
Valid type keywords are:
  bios         BIOS信息
  system       系统信息
  baseboard    主板信息
  chassis      机箱
  processor    处理器
  memory       内存
  cache        缓存
  connector    连接器
  slot         插槽
  
DMTF Distribued Management Task Force 分布式管理任务组
DMI Desktop Management Interface 桌面管理接口
DMI和SMBIOS System Management BIOS 是一套数据结构标准，规范各个厂商的BIOS信息存储格式，1999年被纳入到DMTF。

-s string 
dmidecode -t 16 查看当前设备能支持的最大内存，以及当前的内存插槽情况
内存插槽情况 
dmidecode -t 17 
```

##### 内核模块

```
linux内核是宏内核（和微内核相对），整个内核是一个单独的，非常庞大的程序。
为了应对每天新产品出现的变化，Linux引入了module的技术实现动态把某些功能加入到内核中。大大改善Linux内核对新硬件的支持能力。
```

名称| 位置
---| ---
内核| /boot/vmlinuz或/boot/vmlinz-version
内核解压所需的RAMDisk | /boot/initrd 或/boot/initrd-version
内核模块| /lib/modules/version/kernel /lib/modules/$(uname -r)/kernel
内核源代码| ／usr/src/Linux或/usr/src/kernels

内核分类管理

分类名称| 描述
------| -----
arch | 与硬件平台相关的项目，如CPU的等级
crypto | 核心所支持的加密技术，md5等
drivers| 硬件驱动，显卡，网卡等
fs | 核心所支持的文件系统,vfat,reiserfs等
lib | 函数库
mm | 内存管理相关
net | 网络有关的各项协议数据
sound| 与音效相关的模块
virt | 虚拟化相关的函数库

查看内核依赖关系

```
cat /lib/modules/$(uname -r)/modules.dep

```

显示所有的内核模块

```
lsmod 
* 第一列 模块名称
* 第二列 模块大小
* 第三列 被引用的数量
   如果有autoclean，表示该模块可以在空闲时自动卸载，后面若有unused，则表示该模块当前没有被使用，同时有两个标志，可以rmmod -a自动卸载
* 第四例表示此模块被哪些模块所引用

```

modinfo查看模块具体信息

```
$ modinfo floppy
```

##### chroot

```
* 测试和开发使用
* 依赖控制
* 兼容性
* 修复系统
* 特权分离

```

chroot创建空间例子


```
# mkdir -p newroot/{bin,lib,lib64}
# ls newroot/
bin  lib  lib64
# cd newroot/
# cp -v /bin/{bash,ls} ./bin/
‘/bin/bash’ -> ‘./bin/bash’
‘/bin/ls’ -> ‘./bin/ls’
# cd ..
# chroot
chroot: missing operand
Try 'chroot --help' for more information.
# chroot newroot/
chroot: failed to run command ‘/bin/bash’: No such file or directory
# ldd ./newroot/bin/bash
	linux-vdso.so.1 =>  (0x00007fffff7f7000)
	libtinfo.so.5 => /lib64/libtinfo.so.5 (0x00007f8237265000)
	libdl.so.2 => /lib64/libdl.so.2 (0x00007f8237061000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f8236c9f000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f8237498000)
# cp /lib64/libtinfo.so.5 newroot/lib64/
# cp /lib64/libdl.so.2 newroot/lib64/
# cp /lib64/libc.so.6 newroot/lib64/
# cp /lib64/ld-linux-x86-64.so.2 newroot/lib64/
# chroot newroot/
bash-4.2# ls
bash: ls: command not found
bash-4.2# exit

ls命令同样时缺少依赖，可以手动拷贝过去

```
##### 关机重启

```
* shutdown 
* reboot
* halt
* poweroff
* init

```

命令| 作用
----|----
shutdown | 关机，重启，支持定时和通知
reboot | 重启
halt| 停止系统
poweroff | 关机
init | init 0 用于关机 init 6用于重启

```
立即关机
# shutdown -h now
立即重启
# shutdown -r now
定时关机
# shutdown -h 23:30
15分钟后关机
# shutdown -h +15

关机并关闭电源
# halt -p

关机不留关机痕迹
# halt -d

假关机
# halt -w

重启
# reboot 
# init 6
关机切开电源
# pwoeroff
# init 0


reboot halt poweroff 是同一个程序

查看电源管理标准
# dmesg | grep -i acpi
# dmesg | grep -i apm
# ps auxww | grep acpi

wtmp和utmp 与重启关机有关的文件
/var/log目录下
last命令从wtmp获取，utmp当前用户的信息w命令来源
shutdown -k  now "warning, I'll shutdown" 提示而已，不会真关机

```



![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 