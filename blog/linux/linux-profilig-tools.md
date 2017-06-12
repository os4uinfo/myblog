<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-26
title: linux系统负载分析常用工具
tags: Linux
images: https://os4u.info/blog/img/sun.png
category: Linux
status: publish
summary: 了解系统相关知识系列---性能工具
-->

##### 工具
1) uptime

```
# uptime
 10:28:41 up 202 days, 21:03,  2 users,  load average: 0.02, 0.06, 0.01
 
 10:28:41 系统当前时间
 up 202 days, 21:03 主机已运行时间
 2 users 当前用户连接数
 load average: 0.02, 0.06, 0.01 系统平均负载，统计最近1、5、15分钟系统平均负载
 
服务器有N核，负载小于N，就在于服务器可承受范围内。
也可以通过/proc/loadavg获取

# cat /proc/loadavg
0.04 0.06 0.01 1/332 19620
前三个数字是最近1、5、15分钟系统平均负载， 1/332表示系统当前总进程数332，1表示正在运行的进程数；19620表示最近一个启动运行的进程ID。
 
```
2) free 

```
# free
             total       used       free     shared    buffers     cached
Mem:       8060736    6637608    1423128       8032     296008    4982572
-/+ buffers/cache:    1359028    6701708
Swap:      8208380     791736    7416644
# free -m
             total       used       free     shared    buffers     cached
Mem:          7871       6484       1386          7        289       4865
-/+ buffers/cache:       1330       6541
Swap:         8015        773       7242
shared 指系统多个进程所共享的内存容量，反应系统中可被复用的内存量，在free命令中，shared实际没多大指导意义，被弃用的指标

buffers，cached区别
两者都属于内存的一部分，两者合空闲内存，已用内存构成了整个内存容量
目前市场上最快内存读写速度大概60GB/s，而SSD，最快速度大概600M/s 相差100倍。
由于磁盘和内存读写速度区别，减少时间等待，就增加了一个buffers缓冲区，写入缓冲。
从硬盘读取的数据，暂时存在cache里面，加速读取数据。

free是读取/proc/meminfo中对应的值。buffer是块设备I/O相关的缓存页，cached是普通文件相关的缓存页。

# free -m
             total       used       free     shared    buffers     cached
Mem:          7871       6480       1391          7        289       4865
-/+ buffers/cache:       1325       6546
Swap:         8015        773       7242

7871: 服务器的内存总量
6480: 含buffers/cached在内的内存的使用量
1391: 不包含buffers/cached在内的内存空闲
7: 系统多个进程所共享的内存容量
289: buffers占用内存量
4865: cached占用内存量
1325: 不含buffers/cached在内的内存使用量，真正内存使用量，因为buffer/cached可以随时被使用
6546: 包含buffers/cached在哪的内存使用量，真正内存闲置量，因为buffer/cached可以随时被使用
8015: 交换分区的总量
773: 交换分区的使用量
7242: 交换分区的闲置量

# free -om
             total       used       free     shared    buffers     cached
Mem:          7871       6483       1388          7        289       4865
Swap:         8015        773       7242
-o “-/+ buffers/cache:" 没有了，提高可读可易用性。

```

3）swap

```
设置swap，一般是实际无力内存的1～2.5倍之间
查看分区free -m
创建swap
独立分区：
# fdisk /dev/sda  指定分区类型为82
# partprobe
# mkswap /dev/sda2
# swapon /dev/sda2 开启swap
# swapoff /dev/sda2 关闭swap。

独立文件：
# dd if=/dev/zero of=/tmp/swap bs=1M count=256
# mkswap /tmp/swap
# swapon /tmp/swap
# swapoff /tmp/swap

内存情况的判定
cat /proc/sys/vm/swappiness 中的值范围0-100，不同策略，差异如下：
```

   swappiness 值       | swap策略
--------------------- |---------------
vm.swappiness = 0     | 几乎禁用swap，除非出现out of memory
vm.swappiness = 1     | 禁止swap之外最保守的swap策略
vm.swappiness = 10    | 内存空间充足，为提升整体性能，以便降低swap的频次
vm.swappiness = 60    | 默认值，中庸策略
vm.swappiness = 100   | 系统会激进使用swap。

4）vmstat 

查看CPU_IDLE, WAIT, 磁盘I/O

```
# vmstat 1 4
procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0 791736 1423896 296008 4982836    0    0     0     1    0    0  0  0 100  0  0
 0  0 791736 1423880 296008 4982836    0    0     0     0   31   34  0  0 100  0  0
 0  0 791736 1423880 296008 4982836    0    0     0     0   35   27  0  0 100  0  0
 0  0 791736 1423880 296008 4982836    0    0     0     0   22   36  0  0 100  0  0
每隔1秒输出系统状态信息。共输出4次。
======================================================================================
procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
进 程--------------内 存---------- ---交换分区-- ---块设备-- --系统---- ----CPU------
进程：
r： 进程运行队列中的进程个数
b： 处于不可中断的睡眠状态的进程个数

内存：
swpd: 虚拟内存使用量
free: 空闲内存量(不含buffer和cached）
buff: 内存中的buffers使用量
cache: 内存中的cache使用量

交换分区：
si: 每秒从交换分区写入内存的量
so: 每秒从内存写入交换分区的量

块设备：
bi: 每秒从块设备读取的块（block）数量
bo: 每秒向块设备写入的块（block）数量

系统：
in: 每秒中断数（含时钟中断）
cs: 每秒上下文切换次数

CPU：
us: 用户进程CPU消耗时间百分比
sy: 内核进程CPU消耗时间百分比
id: CPU空闲状态时间百分比
wa: IO等待消耗时间百分比
st: 虚机管理程序占用时间百分比

第一行数据是自统计以来的一个平均值，可以忽略。

参数：
-a 非活跃内存（inactive memroy)
   活跃内存 (active memroy)
   减少了buffers和cache信息

查看服务器有多少forks(累计数量）
# vmstat -f
 47211808 forks
forks是由 fork，vfork和clone三类系统调用（system calls)产生的，可以理解为系统创建的总任务数（tasks）
可以通过获取该数据差值，判断系统是不是因为forks骤增导致性能问题。

```
选项  | 用法                 | 举例
-----|---------------------|-------
-m   | vmstat -m           | 展示内存slabinfo
-s   | vmstat -s           | 展示内存指标及系统事件信息
-d   | vmstat -d           | 展示各磁盘统计信息
-p   | vmstat -p /dev/sda1 | 展示某一特定分区的I/O信息

vmstat 服务器问题查看原则

```
1. cache数据较大，说明系统缓存了较多的磁盘数据，利于磁盘I/O性能的提升，bi相对较小，磁盘读写都有cache承担
2. si和so是swap的读写，如果长期大于0，则说明系统经常读写交换分区，会消耗CPU和I/O，需要关注下。看是否是内存性能问题。
3. free值较小，接近0，不一定是系统内存耗尽，也要同时查看buff和cache的量，大部分情况是buffer和cache占用量较多。
4. bi和bo数据值很大，说明系统进行磁盘读写操作
5. us数值经常大于50%，说明用户占用的CPU时间较多，开发程序就需要优化了
6. sy系统内核消耗较多cpu时间，如果较高，则系统存在问题
7. wa 说明cpu总是等待I/O，表明磁盘成为瓶颈，可以将磁盘升级，或者查看程序读取磁盘的策略，是随机读还是顺序读可以调整下
8. r表示正在运行队列中的任务数，若总数超过cpu个数，则说明cpu成为瓶颈，考虑开启超线程，更换多核CPU，调整某些进程的nice优先级等
9. us 服务器视频编码任务，有可能超过95%，说明正常。产生随机数的程序，或者其他包含系统调用的程序，sy也可能很高
10. 开启了大型软件，同时又开启了大型游戏，so的值身高，系统将内存闲置数据写入到swap。如果希望减少写入，可增大内存。
```

5) mpstat

多处理器统计工作

```
mpstat -P ALL 1 每秒展示每一个CPU核的运行状态
查看第一个核CPU状态（编号0）， mpstat -P 0 1 每秒展示第一个核的运行状态
```
```
# mpstat -P ALL
Linux 2.6.32-573.el6.x86_64 (lj-mail-server) 	05/26/2017 	_x86_64_	(8 CPU)

01:05:00 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
01:05:00 PM  all    0.21    0.00    0.14    0.00    0.00    0.01    0.00    0.00   99.64
01:05:00 PM    0    0.30    0.00    0.14    0.00    0.02    0.06    0.00    0.00   99.48
01:05:00 PM    1    0.32    0.00    0.25    0.00    0.00    0.00    0.00    0.00   99.43
01:05:00 PM    2    0.23    0.00    0.12    0.00    0.00    0.00    0.00    0.00   99.64
01:05:00 PM    3    0.14    0.00    0.10    0.00    0.00    0.00    0.00    0.00   99.76
01:05:00 PM    4    0.19    0.00    0.13    0.00    0.00    0.00    0.00    0.00   99.67
01:05:00 PM    5    0.24    0.00    0.23    0.00    0.00    0.00    0.00    0.00   99.53
01:05:00 PM    6    0.17    0.00    0.09    0.00    0.00    0.00    0.00    0.00   99.75
01:05:00 PM    7    0.07    0.00    0.04    0.00    0.00    0.00    0.00    0.00   99.88
```
指标     | 说明
-------- | ------
 %usr    | 用户进程所使用CPU的百分比 
 %nice   | 对进程进行降级时CPU的百分比
 %sys    | 内核进程使用CPU百分比
 %iowait | 等待进行I/O所使用的CPU时间百分比
 %irq    | 处理系统中断的CPU百分比
 %soft   | 软件中断的CPU百分比
 %steal  | 虚机管理程序占用的CPU百分比
 %guest  | 运行虚拟处理器占用的CPU百分比
 %idle   | CPU空闲时间


中断，提供了一种机制，是的读取硬盘这样的操作可以交个硬件来完成，CPU挂起当前进程，将控制权交给其他进程，等硬件处理完后通知CPU，操作系统把当前进程设置为活动，从而允许该进程继续运行，处理读取硬盘结果。

```
-I interrupts 获取CPU中断处理情况 可选参数
* SUM 展示所有CPU中断之和
* CPU 展示每个CPU的中断处理情况，不展示所有CPU总和
* ALL SUM和CPU的内容一起展示
```

6) top

linux常用性能监控工具，可以了解到cpu负载情况，内存状态，swap使用情况，以及详细的进程运行状态

常用功能：1.查找占用大量CPU资源的进程；2.查找占用大量内存的进程；3.查看占用CPU时间最多的进程。

```
# top
top - 13:17:48 up 202 days, 23:52,  3 users,  load average: 0.07, 0.02, 0.00
Tasks: 251 total,   1 running, 250 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.2%us,  0.1%sy,  0.0%ni, 99.6%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   8060736k total,  6633608k used,  1427128k free,   296024k buffers
Swap:  8208380k total,   791736k used,  7416644k free,  4984832k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
19682 root      20   0 15160 1244  824 R  3.3  0.0   0:00.13 top
    1 root      20   0 19364 1276 1004 S  0.0  0.0   2:20.47 init
    2 root      20   0     0    0    0 S  0.0  0.0   0:09.94 kthreadd
    3 root      RT   0     0    0    0 S  0.0  0.0   0:07.14 migration/0
    4 root      20   0     0    0    0 S  0.0  0.0   0:49.12 ksoftirqd/0
    5 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 stopper/0
    6 root      RT   0     0    0    0 S  0.0  0.0   0:33.86 watchdog/0
    7 root      RT   0     0    0    0 S  0.0  0.0   6:45.33 migration/1
    8 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 stopper/1
```

数据      | 位置   | 内容
---------| ----  | ----
系统级数据 | 上部分 | 1.系统负载信息；2.CPU信息； 3.内存信息
进程级数据 | 下部分 | 包含每个进程详细信息。 PID，USER，PR，NI，VIRT，RES，SHR，S，%CPU，%MEM，TIME+，COMMAND

```
系统级数据

top - 13:17:48 up 202 days, 23:52,  3 users,  load average: 0.07, 0.02, 0.00
命令名称 系统时间 系统开启时间          用户连接数   最近1，5，15分钟系统平均负载

Tasks: 251 total,   1 running, 250 sleeping,   0 stopped,   0 zombie
系统进程总数，        处于运行状态的进程数量1，处于睡眠状态进程250，停止状态的进程0个，处于僵尸状态进程0个

Cpu(s):  0.2%us,  0.1%sy,  0.0%ni, 99.6%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
         用户cpu占比，内核占比，改变过nice进程占比，空闲占比， io等待占比，硬中断占比，软中断占比，虚机管理占比
         
Mem:   8060736k total,  6633608k used,  1427128k free,   296024k buffers
       内存总量     使用中的内存总量（包括buffers和cached），空闲内存总量（不含buffers和cached），buffers占用的内存量
Swap:  8208380k total,   791736k used,  7416644k free,  4984832k cached
       swap总量，         swap使用量，swap空闲量       ， cached占用内存量

top参数
f 进入进程信息列选择页面
Current Fields:  AEHIOQTWKNMbcdfgjplrsuvyzX  for window 1:Def
Toggle fields via field letter, type any other key to return

* A: PID        = Process Id
* E: USER       = User Name
* H: PR         = Priority
* I: NI         = Nice value
* O: VIRT       = Virtual Image (kb)
* Q: RES        = Resident size (kb)
* T: SHR        = Shared Mem size (kb)
* W: S          = Process Status
* K: %CPU       = CPU usage
* N: %MEM       = Memory usage (RES)
* M: TIME+      = CPU Time, hundredths
  b: PPID       = Parent Process Pid
  c: RUSER      = Real user name
  d: UID        = User Id
  f: GROUP      = Group Name
  g: TTY        = Controlling Tty
  j: P          = Last used cpu (SMP)
  p: SWAP       = Swapped size (kb)
  l: TIME       = CPU Time
  r: CODE       = Code size (kb)
  s: DATA       = Data+Stack size (kb)
  u: nFLT       = Page Fault count
  v: nDRT       = Dirty Pages count
  y: WCHAN      = Sleeping in Function
  z: Flags      = Task Flags <sched.h>
* X: COMMAND    = Command name/line

Flags field:
  0x00000001  PF_ALIGNWARN
  0x00000002  PF_STARTING
  0x00000004  PF_EXITING
  0x00000040  PF_FORKNOEXEC
  0x00000100  PF_SUPERPRIV
  0x00000200  PF_DUMPCORE
  0x00000400  PF_SIGNALED
  0x00000800  PF_MEMALLOC
  0x00002000  PF_FREE_PAGES (2.5)
  0x00008000  debug flag (2.5)
  0x00024000  special threads (2.5)
  0x001D0000  special states (2.5)
  0x00100000  PF_USEDFPU (thru 2.4)
添加所需的列，直接按键盘上对应的字母即可，回车确认。

o 参数排序
Current Fields:  AEHIOQTWKNMbcdfgjplrsuvyzX  for window 1:Def
Upper case letter moves field left, lower case right

* A: PID        = Process Id
* E: USER       = User Name
* H: PR         = Priority
* I: NI         = Nice value
* O: VIRT       = Virtual Image (kb)
* Q: RES        = Resident size (kb)
* T: SHR        = Shared Mem size (kb)
* W: S          = Process Status
* K: %CPU       = CPU usage
* N: %MEM       = Memory usage (RES)
* M: TIME+      = CPU Time, hundredths
  b: PPID       = Parent Process Pid
  c: RUSER      = Real user name
  d: UID        = User Id
  f: GROUP      = Group Name
  g: TTY        = Controlling Tty
  j: P          = Last used cpu (SMP)
  p: SWAP       = Swapped size (kb)
  l: TIME       = CPU Time
  r: CODE       = Code size (kb)
  s: DATA       = Data+Stack size (kb)
  u: nFLT       = Page Fault count
  v: nDRT       = Dirty Pages count
  y: WCHAN      = Sleeping in Function
  z: Flags      = Task Flags <sched.h>
* X: COMMAND    = Command name/line

Flags field:
  0x00000001  PF_ALIGNWARN
  0x00000002  PF_STARTING
  0x00000004  PF_EXITING
  0x00000040  PF_FORKNOEXEC
  0x00000100  PF_SUPERPRIV
  0x00000200  PF_DUMPCORE
  0x00000400  PF_SIGNALED
  0x00000800  PF_MEMALLOC
  0x00002000  PF_FREE_PAGES (2.5)
  0x00008000  debug flag (2.5)
  0x00024000  special threads (2.5)
  0x001D0000  special states (2.5)
  0x00100000  PF_USEDFPU (thru 2.4)
对应的星号字母可以上下移动。
保存配置W。会将配置写入~/.toprc
```
进程级数据


指标 | 说明|参考值
-------|-------------| ----  
  PID  | 进程ID        |
  PPID | 进程父ID       |
  USER | 进程拥有者用户名 |
  PR   | 进程优先级      |
  NI   | 进程的NICE值，用户调节优先级 | -20到20之间整数 负数调高优先级。正数降低
  VIRT  |  进程使用虚拟内存量   |
  RES | 进程使用的且未被换出的物理内存量 |
  SHR |共享内存量 |
  S  |进程状态| R：运行，S:睡眠，D:不可中断的睡眠状态，T:跟踪，停止，Z:僵尸
  %CPU | 进程所占CPU时间百分比   |
  %MEM   |  进程所使用物理内存占比| 
  TIME+  |进程占用CPU时间总和，单位为1/100秒 |
  RUESR | 进程拥有者实际用户名，即登录到shell时的用户|
  UID  |进程所有者的用户ID，即USER所对应的ID| 
  GROUP | 进程所有者的用户组名称 |
  TTY | 启动进程的终端名称| 若对应终端，则显示为问号（？）
  P   | 此进程最近一个时刻使用CPU编号 |
  SWAP | 进程中使用且已被换出的虚拟内存量| 
  TIME | 进程所占CPU总时间，单位秒 |
  CODE | 进程对应可执行代码占用物理内存量 |
  DATA | 进程对应的数据部分（数据段、栈等）占用物理内存量 | 
  nFLT | 页面错误次数 |
  nDRT | 从最后一次写入到此时此刻，被修改过的页面数（脏数据？） | 
  WCHAN| 若进程在睡眠，则显示睡眠中系统调用名 |
  Flags | 进程标志 |
  COMMAND | 进程对应的命令名称 |

```
进程数据排序 F或者O实现

```
快捷键 | 作用
------|--------
b     | 高亮当前运行的进程
m     | 隐藏内存信息
l     | 隐藏系统平均负载
t     | 隐藏tasks和cpu信息
c     | 查看完整命令
M     | 内存使用量排序
P     | CPU使用量排序

top问题分析定位

```
top - 14:00:47 up 203 days, 35 min,  3 users,  load average: 0.20, 0.07, 0.02
Tasks: 256 total,   1 running, 255 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.0%us,  0.1%sy,  0.0%ni, 99.9%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   8060736k total,  6636928k used,  1423808k free,   296024k buffers
Swap:  8208380k total,   791736k used,  7416644k free,  4984984k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
 1797 root      20   0  581m  972  784 S  0.3  0.0  12:21.93 automount
32064 root      20   0 15164 1356  924 R  0.3  0.0   0:00.07 top
    1 root      20   0 19364 1276 1004 S  0.0  0.0   2:20.68 init
    2 root      20   0     0    0    0 S  0.0  0.0   0:09.94 kthreadd
    3 root      RT   0     0    0    0 S  0.0  0.0   0:07.16 migration/0
    4 root      20   0     0    0    0 S  0.0  0.0   0:49.15 ksoftirqd/0
    5 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 stopper/0
    6 root      RT   0     0    0    0 S  0.0  0.0   0:33.86 watchdog/0
    7 root      RT   0     0    0    0 S  0.0  0.0   6:45.74 migration/1
    8 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 stopper/1
    
场景：某一个cpu负载很高
1. f命令，选中"j: P=Last used cpu(SMP)"，展示出来
2. b,高亮运行的进程
3. 观察运行在该核上的进程，通过/proc相关信息深入调查
4. 可能原因：进程绑定了内核
5. 解决方案，kill，优化程序，解绑。

内存指标：
VIRT/RES/SHR/DATA/CODE/SWAP/%MEM
$ cat /proc/2814/statm
23903 1009 760 55 0 185 0
VIRT   RES SHR  Trs Lrs Drs Dt
```
指标   | 说明
------|----
VIRT  | virtual memory 虚拟内存，进程通过malloc/calloc系列函数申请的内存，或是堆栈所占用的内存或是全局变量占用的内存
RES   | Resident Memory 常驻内存，一个进程真正在使用的内存
SHR   | shared Memory 共享内存
CODE  | 机器指令部分大小，称Trs Text Resident Set
DATA  | 除了CODE之外的内存空间，也叫Drs，Data Resident Set
SWAP  | 进程被置换的虚拟内存空间大小

%MEM = （常驻内存/总内存大小) 100%

7) iostat

iostat 显示CPU的统计信息，以及整个系统，适配器，tty设备核硬盘输入输出信息

```
# iostat
// 系统信息，内核版本，架构信息
Linux 2.6.32-573.el6.x86_64 (lj-mail-server) 	05/26/2017 	_x86_64_	(8 CPU)

// CPU 信息
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.21    0.00    0.15    0.00    0.00   99.64

//磁盘信息，展示磁盘I/O情况，块的读写量和读写速度
Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
sda               0.99         0.25        17.09    4318238  299724704
dm-0              2.06         0.14        16.48    2490322  289115888
dm-1              0.04         0.10         0.20    1796672    3548736
dm-2              0.05         0.00         0.40       5858    7059936

场景：查看TPS和吞吐量
iostat -d -k 1 3
-d 显示磁盘使用状态
-k KB做单位
1 每秒采样
3 采样3次

```
名称 | 说明
----| ---
tps | 每秒进程的IO读写请求总数
kB_read/s     | 每秒读取的字节数（KB单位）
kB_wrtn/s     | 每秒写入的字节数 (KB单位）
kB_read       | 读取的字节总数（KB单位）
kB_wrtn       | 写入的字节总数（KB单位）

```
不显示第一组统计数据 iostat -d -k -y 1 3
iostat -d -x -k 1 3 
-x 显示更多的磁盘统计数据

```
名称| 说明
----|----
rrqm/s   | 每秒对该设备的读请求被合并次数
wrqm/s   | 每秒对该设备的写请求被合并次数
r/s      | 每秒完成读I/O的次数
w/s      | 每秒完成写I/O的次数 
rsec/s   | 每秒读扇区数，每扇区512字节
wsec/s   | 每秒写扇区数，每扇区512字节
rkB/s    | 每秒读千字节数
wkB/s    | 每秒写千字节数
avgrq-sz | 平均每次I/O操作的数据大小（扇区）即（rsec/s + wsec/s)/(r/s+w/s)
avgqu-sz | 平均等待处理I/O请求队列长度
await    | 平均每次I/O请求等待时间
svctm    | 平均每次I/O操作的服务时间，单位毫秒
%util    | 周期内用于I/O操作的时间比。（r/s+w/s)*(svctm/1000)

```
* 如果%util较大，代表I/O请求太多，硬盘可能存在瓶颈
* await 大于svctm，差值越小，说明队列时间越短；反之差值越大，则说明队列时间越长，说明系统问题
* svctm接近 awati。说明I/O几乎没有等待时间
* await远大于svctm,说明I/O队列太长，响应时间也变长
* avgqu-sz队列长度也可衡量I/O负载指标。单位时间内的平均值

```

场景：查看CPU

```
iostat -c 1 3

```
指标| 说明
---|---
%user   | 用户级占用CPU百分比
%nice   | 程序使用NICE权限执行时百分比
%system | 内核占用CPU百分比
%iowait | CPU空闲且系统有未完成的磁盘I/O请求的百分比
%steal  | hypervisor服务另一个虚拟处理器时，虚拟CPU等待实际CPU的时间百分比
%idle   | cpu空闲

```
* steal 值比较高的话，需要扩展cpu，可能是别的虚拟机占用太多cpu时间片，从而
* 若升级后，还出现steal值较高，说明宿主机上虚拟机过多。

%iowait 高，硬盘存在瓶颈
%idle 高，CPU空闲
$idle 高，系统响应慢，可能是CPU等待分配内存
%idle 小于70，I/O压力大
%idle 持续低于10，那么CPU瓶颈

```

![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 