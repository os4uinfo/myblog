<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-28
title: linux进程信号
tags: Linux
images: https://os4u.info/blog/img/sun.png
category: Linux
status: publish
summary: Linux运维中，常用命令解决实际的运维问题，持续更新，总结中。
-->

##### 1）fuser 

可以显示磁盘上文件／目录，甚至网络端口正在被什么程序使用，并可以展示这些程序详细信息。

```
fuser -v /root/
                     USER        PID ACCESS COMMAND
/root:               root      15205 ..c.. bash

USER 进程所属用户
PID 进程ID
COMMAND 进程名称
ACCESS 访问关系
  c 将此文件作为当前目录使用
  e 将此文件作为可执行对象使用
  r 将此文件作为根目录使用
  s 将此文件作为共享库（或其他可装载对象）使用
  m 将此文件作为映射文件或共享库
  f 打开此文件，默认不显示
  F 打开此文件，用于写操作，默认不显示

# fuser  -v .t.txt.swp
                     USER        PID ACCESS COMMAND
/root/.t.txt.swp:    root      15250 F.... vim
# fuser .t.txt.swp
/root/.t.txt.swp:    15250  

-m 表示查阅有多少程序正在使用某个目录第下的文件系统（目录下所有子目录和文件）


```
通过端口定位程序

```
# fuser -v -n tcp 80
                     USER        PID ACCESS COMMAND
80/tcp:              root      22979 F.... nginx
                     www       24501 F.... nginx
```

进程终止

```
$ fuser -v -k /root
```

fuser 和 lsof 对比

对比项 | lsof | fuser
------|------|-----
思路   | 通过进程查找文件| 通过文件查找进程
所属标准| -    | POSIX
接收参数| 文件、PID、网络端口| 文件、网络端口
进程的输出| 进程的详细信息| 进程ID
是否可发送信号| 不可以| 可以（-k或者-signal)

##### 2） ps命令

```
# ps
  PID TTY          TIME CMD
15274 pts/2    00:00:00 bash
15291 pts/2    00:00:00 ps

PID 进程ID
TTY 启动此进程终端名称
TIME 进程所占CPU时间总和单位为秒
CMD 进程命令

ps aux 组合命令
a 显示各终端上所有进程
u 展示进程所属用户
x 对于没有关联到终端上的进程也展示出来

ps aux 与ps -aux	区别
ps -aux 分为 ps -a -u x
-a 显示所有当前终端的所有进程
-u x 显示用户名为'x'的用户所有进程
系统不存在x用户，所以两者显示是一样的。

ps 有三种格式：
BSD 选项前不需要加短横线 -
UNIX 选项前需加短横线 
GNU 选项前加双短横线--

-e 显示全部进程
-F 显示详尽的进程信息

自定义显示列
$ ps -eo pid,user,cmd,start
修改列名
$ ps -eo pid,user,cmd=COMMAND,start
```

列出汇总信息

列名|内容
----|---
%CPU | 进程占用CPU时间占比
%MEM | 进程使用物理内存占比
ADDR | 进程的内存地址
C或CP | CPU利用率
CMD或COMMAND| 进程及参数
NI | 进程的NICE值，用于调整优先级
F  | 进程的旗标（flag），信号量，用于进程互斥加锁等
PID | 进程ID
PPID | 父进程ID
PRI | 进程优先级
RSS | 实际内存使用量，单位KB
S或STAT |进程状态（S-休眠，R-可运行， Z-僵尸进程，T-代表停止，0-正在运行）
START或STIME|进程开始时间
SZ | 虚拟内存使用量
TIME | 进程占用CPU时间总和，秒
TT或TTY | 启动此进程的终端名称
UID或USER |进程属主
WCHAN | 若此进程在休眠，显示休眠中的系统调用名

其他应用

```
-u 按有效用户ID（EUID）查看用户运行的进程
-U 按实际用户ID（RUID）筛选进程
u：按用户名和进程号的顺序来显示进程，输出由
USER、PID、%CPU、%MEM、VSZ、RSS、TTY、STAT、START、TIME、COMMAND组成
-C 根据命令查找
ps显示排序
ps --sort 
参数：
-pcpu CPU使用率降序
+pcpu CPU使用率生序
-pmem 内存降序
+pmem 内存生序

# ps aux --sort -pmem
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
nobody   24518  0.0  9.9 286064 101580 ?       Sl   May27   0:53 varnishd -f /etc/varnish/default.vcl -s malloc,512M -T 127.0.0.1:2000 -a 0.0.0.0:8080
root     24517  0.0  8.6 122776 87948 ?        SLs  May27   0:03 varnishd -f /etc/varnish/default.vcl -s malloc,512M -T 127.0.0.1:2000 -a 0.0.0.0:8080
root      6495  0.0  1.6 553160 16448 ?        Ssl  11:20   0:00 /usr/bin/python -Es /usr/sbin/tuned -l -P
polkitd   6464  0.0  1.3 527516 13904 ?        Ssl  11:20   0:00 /usr/lib/polkit-1/polkitd --no-debug
root       328  0.0  0.9  45012  9424 ?        Ss   May27   0:01 /usr/lib/systemd/systemd-journald
root     15203  0.0  0.5 145696  5116 ?        Ss   11:21   0:00 sshd: root@pts/0
root     15348  0.0  0.5 145696  5116 ?        Ss   12:26   0:00 sshd: root@pts/3
root     15231  0.0  0.5 145696  5112 ?        Ss   11:26   0:00 sshd: root@pts/1
root     15272  0.0  0.5 145696  5112 ?        Ss   11:34   0:00 sshd: root@pts/2
root     15406  0.0  0.5 145696  5112 ?        Ss   12:39   0:00 sshd: root@pts/4
root     15250  0.0  0.5 151424  5096 pts/1    S+   11:26   0:00 vim t.txt

查找线程
-L -T 选项
例子：
# ps -ef | grep varnish
root     15445 15408  0 12:42 pts/4    00:00:00 grep --color=auto varnish
root     24517     1  0 May27 ?        00:00:03 varnishd -f /etc/varnish/default.vcl -s malloc,512M -T 127.0.0.1:2000 -a 0.0.0.0:8080
nobody   24518 24517  0 May27 ?        00:00:53 varnishd -f /etc/varnish/default.vcl -s malloc,512M -T 127.0.0.1:2000 -a 0.0.0.0:8080
# ps -L 24517
  PID   LWP TTY      STAT   TIME COMMAND
24517 24517 ?        SLs    0:03 varnishd -f /etc/varnish/default.vcl -s malloc,512M -T 127.0.0.1:2000 -a 0.0.0.0:8080
# ps -T 24517
  PID  SPID TTY      STAT   TIME COMMAND
24517 24517 ?        SLs    0:03 varnishd -f /etc/varnish/default.vcl -s malloc,512M -T 127.0.0.1:2000 -a 0.0.0.0:8080

* LWP light weight process 轻量级进程，即用户线程
* SPID 系统中的线程ID

进程树
BSD方式
ps axjf

* a 显示终端上所有进程
* x 显示没有控制终端的进程
* j 用任务格式来显示
* f 用ascii显示树状结构

UNIX方式
ps -ejH
* -e 显示终端上所有进程
* -j 用任务格式来显示
* -H 树状结构

```

##### 3） kill命令

```
kill 信号
#  kill -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
63) SIGRTMAX-1	64) SIGRTMAX

查看信号对应编号
kill -l SIGILL
kill -l 9
```
常用信号

信号名称| 信号编号 | 说明
-------|---------|----
HUP   | 1 | 终端断线
INT | 2 |中断（同ctrl+c）
QUIT | 3| 退出（同ctrl+\）
TERM | 15 | 终止
KILL | 9 | 强制终止
CONT | 18 | 继续（与STOP相反，fg／bg命令）
STOP | 19 |暂停(同ctrl+Z）


kill 用法

```
kill [选项] [进程号]
选项：
* -l 列出信号类型
* -s 要发出的信号，向目标发送指定的信号类型
* 无选项，向目标进程发送默认终止型号，即SIGTERM，编号15
进程号：
* 进程号> 0向目标发送信号，进程号可以是几个进程的集合
* 进程号=0 向当前进程组的所有进程发送信号
* 进程号=-1 则是向当前kill和init除外的所有进程发送信号
* 进程号< -1 向进程组PGID 的所有进程发送信号

kill -9 pid
kill -KILL pid
kill -SIGKILL pid
kill -s 9 pid
进程id为1的，无法kill

信号编号为0，实际为test信号
$ ps -ef | grep nginx
www      15465 22979  0 12:46 ?        00:00:00 nginx: worker process
www      15466 22979  0 12:46 ?        00:00:00 nginx: worker process
www      15467 22979  0 12:46 ?        00:00:00 nginx: worker process
www      15468 22979  0 12:46 ?        00:00:00 nginx: worker process
www      15579 15563  0 13:03 pts/1    00:00:00 grep --color=auto nginx
root     22979     1  0 May27 ?        00:00:00 nginx: master process nginx
$ kill -0 22979
bash: kill: (22979) - Operation not permitted

终结后台作业
kill -信号类型 %后台工作编号

$ ping www.baidu.com > /tmp/output.txt &
[1] 15583
$ jobs
[1]+  Running                 ping www.baidu.com > /tmp/output.txt &
$ kill -9 %1
[1]+  Killed                  ping www.baidu.com > /tmp/output.txt
$
```

##### 4)作业控制

```
后台命令（&）
Ctrl+Z 让作业转到后台并停止
jobs 查看当前作业列表
fg 将后台一个作业切换到前台并运行
bg 将作业切换到后台运行
kill 终止一个作业

$ bg %1

```

符号 | 含义|例子
----|----|---
%Number | 某一作业| fg %1
%String | 匹配命令行已String开头的作业，多个会报错| kill %abc
%?String| 模糊匹配，默认匹配到第一个| kill %?abc
%% | 最近一个被切换到后台的作业| kill %%
%+ | 与%% 相同| kill %+
%- | 排在最前面的作业| fg %-


##### 5)trap 
信号是一种进程间通信手段，为进程提供了一种异步的软件中断机制。当一个进程接受到信号后，有三种可能处理方式：忽略，触发，捕捉信号处理相应函数

```
trap "commands" signal-list

例子：
# cat t.sh
#!/bin/bash

trap 'echo "hello world"' 2

tail -f /var/log/messages
# sh t.sh
May 28 13:01:17 iZ2ze6ma90hm43zezbqasdZ systemd-logind: Removed session 195.
May 28 13:02:32 iZ2ze6ma90hm43zezbqasdZ systemd: Started Session 197 of user root.
May 28 13:02:32 iZ2ze6ma90hm43zezbqasdZ systemd-logind: New session 197 of user root.
May 28 13:02:32 iZ2ze6ma90hm43zezbqasdZ systemd: Starting Session 197 of user root.
May 28 13:03:30 iZ2ze6ma90hm43zezbqasdZ su: (to www) root on pts/1
May 28 13:10:01 iZ2ze6ma90hm43zezbqasdZ systemd: Started Session 198 of user root.
May 28 13:10:01 iZ2ze6ma90hm43zezbqasdZ systemd: Starting Session 198 of user root.
May 28 13:16:06 iZ2ze6ma90hm43zezbqasdZ systemd: Started Session 199 of user root.
May 28 13:16:06 iZ2ze6ma90hm43zezbqasdZ systemd-logind: New session 199 of user root.
May 28 13:16:06 iZ2ze6ma90hm43zezbqasdZ systemd: Starting Session 199 of user root.
^Chello world

信号捕捉应用：
* 动态读取并更新配置
* 定期清除临时文件
* 忽略某信号对程序可能的影响，如Ctrl+C
* 在用户退出是，再次确认

屏蔽信号
$ trap "" signal-list
此时
$ trap "" INT
$ tail -f /var/log/messages
用ctrl+c无法退出，用ctrl+\退出即可
恢复信号
$ trap INT

```

常见信号

信号名称| 信号数 | 描述
-------| ------|-----
SIGHUP | 1 | 在用户终端连接结束时发出，通知同一session内的各个作业，这个信号默认操作为终止进程，重新读取配置可用
SIGINT | 2 | 程序终止（intrerupt）在用户输入INTR字符（即Ctrl+C）
SIGQUIT |3 | 和SIGINT类似，但由QUIT字符（Ctrl+\)控制，进程收到该信号会产生core文件，类似于程序错误信号
SIGFPE | 8 | 发生致命的算术错误时发出
SIGKILL | 9| 立即结束程序，不能被阻塞、处理和忽略
SIGALRM | 14 | 时钟定时信号，计算的时实际的时间或时钟时间.alarm函数使用
SIGTERM | 15 | 程序结束信号（terminate），正常退出程序

```
$ trap  -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
63) SIGRTMAX-1	64) SIGRTMAX
```

##### 6) nohup

执行方式

```
$ nohup 执行的命令 [&]
获取nohup pid
echo $!

screen setsid disown
让进程后端运行思路：
* 让进程对SIGHUP信号免疫 nohup，disown
* 让进程在新的会话中运行，setid，screen

例子
场景：忘记将进程放后台运行
# ping www.baidu.com >  /tmp/ping.txt
^Z   // Ctrl + Z
[1]+  Stopped                 ping www.baidu.com > /tmp/ping.txt
# jobs
[1]+  Stopped                 ping www.baidu.com > /tmp/ping.txt
# bg %1
[1]+ ping www.baidu.com > /tmp/ping.txt &
#  disown -h %1
# jobs
[1]+  Running                 ping www.baidu.com > /tmp/ping.txt &

新会话

setsid命令

# setsid ping www.baidu.com
# PING www.a.shifen.com (220.181.112.244) 56(84) bytes of data.
64 bytes from 220.181.112.244 (220.181.112.244): icmp_seq=1 ttl=53 time=4.61 ms
64 bytes from 220.181.112.244 (220.181.112.244): icmp_seq=2 ttl=53 time=4.63 ms

logout
Connection to xxx.xxx.xxx.xxx closed.
$ ssh -l root xxx.xxx.xxx.xxx
Last login: Sun May 28 13:37:20 2017 from xxx.xxx.xxx.xxx
Welcome to Alibaba Cloud Elastic Compute Service !
# ps -ef | grep ping
root     15739     1  0 13:37 ?        00:00:00 ping www.baidu.com
root     15760 15742  0 13:38 pts/6    00:00:00 grep --color=auto ping



screen 命令
# screen -A -m -d -S screen_ping ping www.baidu.com &
[1] 15778
# screen  -list
There is a screen on:
	15779.screen_ping	(Detached)
1 Socket in /var/run/screen/S-root.

```



![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 