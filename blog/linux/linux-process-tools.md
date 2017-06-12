<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-26
title: linux进程
tags: Linux
images: https://os4u.info/blog/img/sun.png
category: Linux
status: publish
summary: Linux运维中，常用命令解决实际的运维问题，持续更新，总结中。
-->
#### 进程相关
1）获取进程id

```
获取程序pid
$ pidof program
获取shell脚本pid
$ pidof -x shell.sh
获取第一个pid
$ pidof -s program

pidof 与killall5关系是一样的程序
pidof 返回值 0，找到至少一个对应的pid 1，没有找到任何pid
```
2）服务器系统性能

```
sar 参数 时间间隔和输出次数
时间间隔是秒
输出次数，默认为1，设置为0的话，将一直运行下去。

$ sar 2 3

输出内容存到文件
$ sar 2 3 -o filename
从文本读取内容
$ sar -f filename
sar -o 不接文件名，则会默认保存到/var/log/sa/saDD (DD当天日期数字）

展示每一个CPU核的信息，以1秒为间隔，总共展示1次
$ sar -P ALL 1 1

展示编号为2的CPU,以1秒为间隔，总共展示1次
$ sar -P 1 1 1

参数：
-b: 报告I/O使用情况以及传输速率(只适用于2.5之前内核）
-B: 报告”页“使用情况
-c: 报告进程创建情况
-d: 报告每个块设备使用情况（DEV列类似dev1-7字符串，1表示设备主序号，n表示设备从序号，而rd_sec/s和wr_sec/s单位都是512bytes，即512B，也就是0.5KB)
-I: 报告中断情况
-n: 报告网络情况
-P: 设定CPU
-q: 报告队列长度和负载信息
-r: 报告内存和交换分区使用情况
-R: 报告内存情况
-u: 报告CPU使用情况
-v: 报告i节点、文件和其他内核表信息
-w: 报告系统上下文切换情况
-x: 针对某个特定pid给出统计信息，可以直接给进程号，也可以指定SELF，检查sar本身，若为ALL，则报告所有系统进程信息
-X: 报告特定PID子进程信息
-y: 设定TTY设备信息
-A: 展示所有sar服务器性能信息相当于：-bBcdqrRuvwWy -I SUM -n FULL -P ALL

指定结束时间
$ sar -e 19:30:00 -o record.log 2

查看网络信息
-n 参数：
* DEV 网络设备相关的信息 lo，eth0
* EDEV 
* SOCK 
* FULL

# sar -n DEV 1 1
Linux 3.10.0-514.10.2.el7.x86_64 (k8s-minion-3.ue.cn) 	05/26/2017 	_x86_64_	(4 CPU)

01:47:18 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
01:47:19 PM vetheb81b14      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:47:19 PM      eth0      2.00      2.00      0.98      0.25      0.00      0.00      0.00
01:47:19 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:47:19 PM  flannel0      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:47:19 PM   docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00

Average:        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
Average:    vetheb81b14      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:         eth0      2.00      2.00      0.98      0.25      0.00      0.00      0.00
Average:           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:     flannel0      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:      docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00

IFACE    网络设备名称
rxpck/s  每秒接收到的包数目
txpck/s  每秒发送出去的包数目
rxkB/s   每秒接收到的字节数
txkB/s   每秒发送出去的字节数  
rxcmp/s  每秒钟接收到压缩包数目  
txcmp/s  每秒发送出去的压缩包数目
rxmcst/s 每秒接收到的组播包的包数目

EDEV 网络设备失败情况

sar -n EDEV 1 1
Linux 3.10.0-514.10.2.el7.x86_64 (k8s-minion-3.ue.cn) 	05/26/2017 	_x86_64_	(4 CPU)

01:52:41 PM     IFACE   rxerr/s   txerr/s    coll/s  rxdrop/s  txdrop/s  txcarr/s  rxfram/s  rxfifo/s  txfifo/s
01:52:42 PM vetheb81b14      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:52:42 PM      eth0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:52:42 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:52:42 PM  flannel0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:52:42 PM   docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

Average:        IFACE   rxerr/s   txerr/s    coll/s  rxdrop/s  txdrop/s  txcarr/s  rxfram/s  rxfifo/s  txfifo/s
Average:    vetheb81b14      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:         eth0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:     flannel0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:      docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

rxerr/s   每秒钟接收到的损坏的包数目
txerr/s   发送包时，每秒钟发生的错误数
coll/s    发送包时，每秒发送冲突（collisions）次数，半双工模式才有
rxdrop/s  由于缓冲区满，网络设备接收端每秒钟丢弃包数目
txdrop/s  由于缓冲区满，网络设备发送端每秒钟丢失包数目
txcarr/s  发送数据包时，每秒钟载波错误发生次数
rxfram/s  接收数据包时，每秒钟缓冲区溢出错误发生的次数
rxfifo/s  发送数据包时，每秒钟缓冲区溢出错误发送的次数

socket连接情况报告
sar -n SOCK 1 1
Linux 3.10.0-514.10.2.el7.x86_64 (k8s-minion-3.ue.cn) 	05/26/2017 	_x86_64_	(4 CPU)

01:58:01 PM    totsck    tcpsck    udpsck    rawsck   ip-frag    tcp-tw
01:58:02 PM       293        19         4         0         0         0
Average:          293        19         4         0         0         0
totsck   被使用socket的总数目
tcpsck   当前正在被使用于TCP的socket数目
udpsck   当前正在被使用于UDP的socket数目 
rawsck   当前正在被使用于RAW的socket数目
ip-frag  当前IP分片的数目
tcp-tw   ？
```

3）检测文件各种特性

```
lsof | grep filename
文件描述符定位进程
找出所有打开标准错误输出的进程
$ lsof -d 3

进程定位文件
1. 查找pid
2. lsof -p pid

查看用户打开的文件
$ lsof -u guestuser

查看程序占用端口
lsof -i[46] [protocol][@hostname|hostaddr][:service|prot]
46 ipv4 or ipv6 
$ lsof -i:22

释放已删除磁盘空间
$ lsof -n | grep deleted
找到对应进程，并kill -9 pid杀掉进程

文件不停增大，查看
$ lsof filename
如果未知进程，则kill掉。

查找占据的端口
$ lsof -i:22 的到pid
然后ps aux | grep pid的到进程

lsof查找原理其实就是从/proc/$PID/fd查
COMMAND  进程的名称，显示前9个字符
PID   进程id， -R可以显示-R 显示PPID
TID   ？
USER  进程所有者  -g参数显示进程所属组
FD    文件描述符  
TYPE  文件类型
   DIR： 目录
   REG： 普通文件
   CHR： 字符类型
   BLK： 块设备
   UNIX： UNIX域套接字
   FIFO：先进先出（FIFO）队列
   	IPv4/IPv6：网际协议（IP）套接字             
DEVICE  磁盘名称
SIZE/OFF  文件大小
NODE 索引节点（文件在磁盘上的标识）
NAME 文件名

FD 文件描述符
第一类是文件描述符
第二类是描述文件特征的标识
第一类：
* 0 表示标准输入
* 1 标准输出
* 2 标准错误输出
* n 其他文件描述符的数值
$ lsof | grep /var/spool/postfix/pid/master.pid
master     1311           root   10uW     REG              253,1        33   12583529 /var/spool/postfix/pid/master.pid

10uW说明：
10【文件描述符】
--见上述解释
u【文件状态模式】
* u 该文件打开且处于读取/写入模式
* r 文件打开且处于只读模式
* w 文件打开且处于可写模式
* 空格 文件状态模式未知，且没有锁定
* - 文件状态模式未知，且被锁定

W【文件锁】
* r 文件的部分读锁
* R 整个文件的读锁
* w 文件的部分写锁
* W 整个文件的写锁

第二类：
cwd 应用程序当前工作目录，也是该应用程序启动的目录
txt 该类型文件时程序代码或数据
mem 内存映射文件
pd 父目录 
rtd 根目录
DEL 文件已经被进程删除，但还在内存中

```





![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 