<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-07-22
title: Linux一些常见问题
tags: Linux
images: https://os4u.info/blog/img/sun.png
category: Linux
status: publish
summary: Linux一些常见问题
-->

常用的负载均衡开源软件有nginx、lvs、haproxy，商业的硬件负载均衡设备F5

### 1.linux如何挂在windows下的共享目录

```
mount.cifs //192.168.1.3/server /mnt/server -o user=administrator,pass=123456
```
linux 下的server需要自己手动建一个  后面的user与pass 是windows主机的账号和密码 注意空格 和逗号

### 2.查看http的并发请求数与其TCP连接状态

```
netstat -n | awk '/^tcp/ {++b[$NF]} END {for(a in b) print a, b[a]}'
```

还有ulimit -n 查看linux系统打开最大的文件描述符，这里默认1024，不修改这里web服务器修改再大也没用。若要用就修改很几个办法，这里说其中一个：
修改/etc/security/limits.conf

```
* soft nofile 10240
* hard nofile 10240
```

重启后生效

### 3.用tcpdump嗅探80端口的访问看看谁最高

```
tcpdump -i eth0 -tnn dst port 80 -c 1000 | \
awk -F"." '{print $1"."$2"."$3"."$4}' | sort | uniq -c | sort -nr |head -5 ;
```

### 4.查看/var/log目录下文件数

```
ls /var/log/ -lR| grep "^-" |wc -l
```

### 5.查看当前系统每个IP的连接数

```
netstat -n | awk '/^tcp/ {print $5}'| awk -F: '{print $1}' | sort | uniq -c | sort -rn
```

### 6.shell下32位随机密码生成

```
cat /dev/urandom | head -1 | md5sum | head -c 32 >> /pass
将生成的32位随机数 保存到/pass文件里了
```

### 7.统计出apache的access.log中访问量最多的5个IP

```
 cat access_log | awk  '{print $1}' | sort | uniq -c | sort -n -r | head -5
```

### 8.如何查看二进制文件的内容

```
我们一般通过hexdump命令 来查看二进制文件的内容。
hexdump -C XXX(文件名)  -C是参数 不同的参数有不同的意义
-C  是比较规范的 十六进制和ASCII码显示
-c  是单字节字符显示
-b  单字节八进制显示
-o  是双字节八进制显示
-d  是双字节十进制显示
-x  是双字节十六进制显示
等等等等
```

### 9.ps aux 中的VSZ代表什么意思，RSS代表什么意思

```
VSZ:虚拟内存集,进程占用的虚拟内存空间
RSS:物理内存集,进程战用实际物理内存空间
```

### 10.检测并修复/dev/hda5

```
fsck用来检查和维护不一致的文件系统。若系统掉电或磁盘发生问题，可利用fsck命令对文件系统进行检查。
```

### 11.Linux系统的开机启动顺序

```
加载BIOS–>读取MBR–>Boot Loader–>加载内核–>用户层init一句inittab文件来设定系统运行的等级(一般3或者5，3是多用户命令行，5是界面)–>init进程执行rc.syninit–>启动内核模块–>执行不同级别运行的脚本程序–>执行/etc/rc.d/rc.local(本地运行服务)–>执行/bin/login,就可以登录了。
```

### 12.符号链接与硬链接的区别

```
我们可以把符号链接，也就是软连接 当做是 windows系统里的 快捷方式。
硬链接 就好像是 又复制了一份.
ln 3.txt 4.txt   这是硬链接，相当于复制，不可以跨分区，但修改3,4会跟着变，若删除3,4不受任何影响。
ln -s 3.txt 4.txt  这是软连接，相当于快捷方式。修改4,3也会跟着变，若删除3,4就坏掉了。不可以用了。
```

### 13.保存当前磁盘分区的分区表

dd 命令是以个强大的命令，在复制的同时进行转换
```
dd if=/dev/sda of=./mbr.txt bs=1 count=512
```

### 14.如何在文本里面进行复制、粘贴，删除行，删除全部，按行查找和按字母查找。

以下操作全部在命令行状态操作，不要在编辑状态操作。
在文本里 移动到想要复制的行  按yy  想复制到哪就移动到哪，然后按P  就黏贴了
删除行  移动到改行 按dd
删除全部  dG  这里注意G一定要大写
按行查找  :90 这样就是找到第90行
按字母查找 /path  这样就是 找到path这个单词所在的位置，文本里可能存在多个,多次查找会显示在不同的位置。

### 15.手动安装grub

```
grub-install /dev/sda
```

### 16.修改内核参数

```
vi /etc/sysctl.conf  这里修改参数
sysctl -p  刷新后可用
```

### 17.在1-39内取随机数


expr $[$RANDOM%39] + 1

RANDOM 随机数
%39 取余数 范围 0-38

### 18.限制apache每秒新建连接数为1，峰值为3

每秒新建连接数 一般都是由防火墙来做，apache本身好像无法设置每秒新建连接数，只能设置最大连接：

```
iptables -A INPUT -d 172.16.100.1 -p tcp --dport 80 -m limit --limit 1/second  -j ACCEPT
```

硬件防火墙设置更简单，有界面化，可以直接填写数字。。。
最大连接 apache本身可以设置
MaxClients 3  ,修改apache最大连接 前提还是要修改系统默认tcp连接数。

### 19.FTP的主动模式和被动模式

FTP协议有两种工作方式：PORT方式和PASV方式，中文意思为主动式和被动式。
PORT（主动）方式的连接过程是：客户端向服务器的FTP端口（默认是21）发送连接请 求，服务器接受连接，建立一条命令链路。当需要传送数据时，客户端在命令链路上用PORT 命令告诉服务器：“我打开了XX端口，你过来连接我”。于是服务器从20端口向客户端的 XX端口发送连接请求，建立一条数据链路来传送数据。
PASV（被动）方式的连接过程是：客户端向服务器的FTP端口（默认是21）发送连接请 求，服务器接受连接，建立一条命令链路。当需要传送数据时，服务器在命令链路上用PASV 命令告诉客户端：“我打开了XX端口，你过来连接我”。于是客户端向服务器的XX端口 发送连接请求，建立一条数据链路来传送数据。
从上面可以看出，两种方式的命令链路连接方法是一样的，而数据链路的建立方法就完 全不同。


### 20.显示/etc/inittab中以#开头，且后面跟了一个或者多个空白字符，而后又跟了任意非空白字符的行

```
grep "^# \{1,\}[^ ]" /etc/inittab
```

### 21.显示/etc/inittab中包含了:一个数字:(即两个冒号中间一个数字)的行

```
grep "\:[0-9]\{1\}\:" /etc/inittab
```

### 22.怎么把脚本添加到系统服务里，即用service来调用

在脚本里加入

```
#!/bin/bash
# chkconfig: 345 85 15
# description: httpd

```
然后保存

chkconfig httpd –add  创建系统服务
现在就可以使用service 来 start or restart

### 23.写一个脚本，实现批量添加20个用户，用户名为user01-20，密码为user后面跟5个随机字符

```
#!/bin/bash
#description: useradd
for i in `seq -f"%02g" 1 20`;do
useradd user$i
echo "user$i:`echo $RANDOM|md5sum|cut -c 1-5`"|passwd –stdinuser$i >/dev/null 2>&1
done
```

### 24.写一个脚本，实现判断192.168.1.0/24网络里，当前在线的IP有哪些，能ping通则认为在线

```
#!/bin/bash
for ip in `seq 1 255`
  do
    {
     ping -c 1 192.168.1.$ip > /dev/null 2>&1
     if [ $? -eq 0 ]; then
          echo 192.168.1.$ip UP
     else
          echo 192.168.1.$ip DOWN
     fi
   }&
done
wait

```
### 25.写一个脚本，判断一个指定的脚本是否是语法错误；如果有错误，则提醒用户键入Q或者q无视错误并退出其它任何键可以通过vim打开这个指定的脚本

```
[root@localhost  tmp]# cat checksh.sh
 #!/bin/bash
 read -p "please input check script-> " file
 if [ -f $file ]; then
    sh -n $file > /dev/null 2>&1
    if [ $? -ne 0 ]; then
        read -p "You input $file syntax error,[Type q to exit or Type vim to  edit]" answer
        case $answer in
        q | Q)
           exit 0
           ;;
        vim )
           vim $file
           ;;
        *）
         exit 0
         ;;
        esac
   fi
 else
    echo "$file not exist"
    exit 1
 fi
```
### 26、写一个脚本：(26包括3个小题)

1、创建一个函数，能接受两个参数：
1)第一个参数为URL，即可下载的文件；第二个参数为目录，即下载后保存的位置；
2)如果用户给的目录不存在，则提示用户是否创建；如果创建就继续执行，否则，函数返回一个51的错误值给调用脚本；
3)如果给的目录存在，则下载文件；下载命令执行结束后测试文件下载成功与否；如果成功，则返回0给调用脚本，否则，返回52给调用脚本；


```
[root@localhost tmp]# cat downfile.sh
#!/bin/bash
url=$1
dir=$2
download()
  {
    cd $dir >> /dev/null 2>&1
    if [ $? -ne 0 ];then
        read -p "$dir No such file or directory,create?(y/n)" answer
        if [ "$answer" == "y" ];then
            mkdir -p $dir
            cd $dir
            wget $url 1> /dev/null 2>&1
        else
            return "51"
        fi
    fi
    if [ $? -ne 0 ]; then
        return "52"
    fi
}
download $url $dir
echo $?
```

### 27、写一个脚本：（27包括2个小题）

1、创建一个函数，可以接受一个磁盘设备路径（如/dev/sdb）作为参数;在真正开始后面步骤之前提醒用户有危险，并让用户选择是否继续；而后将此磁盘设备上的所有分区清空（提示，使用命令dd if=/dev/zero of=/dev/sdb bs=512 count=1实现，注意其中的设备路径不要写错了；
如果此步骤失败，返回67给主程序；
接着在此磁盘设备上创建两个主分区，一个大小为100M，一个大小为1G；如果此步骤失败，返回68给主程序；
格式化此两分区，文件系统类型为ext3；如果此步骤失败，返回69给主程序；
如果上述过程都正常，返回0给主程序；
2、调用此函数；并通过接收函数执行的返回值来判断其执行情况，并将信息显示出来；

```
local Darray=(`ls /dev/sd[a-z]`)
for i in ${Darray};do
  [[ "$i" == "$1" ]] && Sd=$i &&break
done
  else
  return66
  fi
#当匹配成功，进入选择，告诉用户，是否继续，输错的话进入无限循环，当用户选择Y,则清空目标分区，且跳出while循环
while :;do
    read -p "Warning!!!This operation will clean $Sd data.Next=y,Quit=n [y|n]:" Choice
    case $Choice in
y)
   dd if=/dev/zero of=$Sd bs=512 count=1 &> /dev/null &&break || return 67 ;;
n)
   exit 88 ;;
*)
   echo "Invalid choice,please choice again." ;;
esac
done

#使用echo传递给fdisk进行分区，如果此命令失败，则跳转出去，错误值68，需要注意的是，
有时候这个返回值很诡异，笔者之前成功与否都是返回的1，后来重启之后，就好了，
如果慎重的话，可以对创建的分区，进行判断，不过就需要使用其他工具截取相关字段了，虽有些小麻烦，但无大碍
`partprobe`
Part=`fdisk -l /dev/$Sd|tail -2|cut -d” ” -f1`
for M in ${Part};do
   mke2fs -j $M &> /dev/null && ErrorPart=$M &&return 69
done
  return 0
}
```


```
#下面代码，调用函数，接收函数返回值，根据返回值进行判断哪里出错。

Disk_Mod $1
Res=$?
[ $Res-eq 0 ] && exit 0
[ $Res-eq 66 ] && echo "Error! Invalid input."
[ $Res-eq 67 ] && echo "Error! Command -> dd <- Faild."
[ $Res-eq 68 ] && echo "Error! Command -> fdisk <- Faild."
[ $Res-eq 69 ] && echo "Error! Command -> mke2fs <- Faild."
```

### 28. LVS的组成

LVS 由2部分程序组成，包括 ipvs 和 ipvsadm。

```
1. ipvs(ip virtual server)：一段代码工作在内核空间，叫ipvs，是真正生效实现调度的代码。
2. ipvsadm：另外一段是工作在用户空间，叫ipvsadm，负责为ipvs内核框架编写规则，
定义谁是集群服务，而谁是后端真实的服务器(Real Server)



1. DS：Director Server。指的是前端负载均衡器节点。
2. RS：Real Server。后端真实的工作服务器。
3. VIP：向外部直接面向用户请求，作为用户请求的目标的IP地址。
4. DIP：Director Server IP，主要用于和内部主机通讯的IP地址。
5. RIP：Real Server IP，后端服务器的IP地址。
6. CIP：Client IP，访问客户端的IP地址。
```
### 29lvs一些原理


```
LVS/NAT原理和特点
(a). 当用户请求到达Director Server，此时请求的数据报文会先到内核空间的PREROUTING链。 此时报文的源IP为CIP，目标IP为VIP 
(b). PREROUTING检查发现数据包的目标IP是本机，将数据包送至INPUT链
(c). IPVS比对数据包请求的服务是否为集群服务，若是，修改数据包的目标IP地址为后端服务器IP，然后将数据包发至POSTROUTING链。 此时报文的源IP为CIP，目标IP为RIP 
(d). POSTROUTING链通过选路，将数据包发送给Real Server
(e). Real Server比对发现目标为自己的IP，开始构建响应报文发回给Director Server。 此时报文的源IP为RIP，目标IP为CIP 
(f). Director Server在响应客户端前，此时会将源IP地址修改为自己的VIP地址，然后响应给客户端。 此时报文的源IP为VIP，目标IP为CIP

2. LVS-NAT模型的特性

RS应该使用私有地址，RS的网关必须指向DIP
DIP和RIP必须在同一个网段内
请求和响应报文都需要经过Director Server，高负载场景中，Director Server易成为性能瓶颈
支持端口映射
RS可以使用任意操作系统
缺陷：对Director Server压力会比较大，请求和响应都需经过director server

```
```
(a) 当用户请求到达Director Server，此时请求的数据报文会先到内核空间的PREROUTING链。 此时报文的源IP为CIP，目标IP为VIP
(b) PREROUTING检查发现数据包的目标IP是本机，将数据包送至INPUT链
(c) IPVS比对数据包请求的服务是否为集群服务，若是，将请求报文中的源MAC地址修改为DIP的MAC地址，将目标MAC地址修改RIP的MAC地址，然后将数据包发至POSTROUTING链。 此时的源IP和目的IP均未修改，仅修改了源MAC地址为DIP的MAC地址，目标MAC地址为RIP的MAC地址 
(d) 由于DS和RS在同一个网络中，所以是通过二层来传输。POSTROUTING链检查目标MAC地址为RIP的MAC地址，那么此时数据包将会发至Real Server。
(e) RS发现请求报文的MAC地址是自己的MAC地址，就接收此报文。处理完成之后，将响应报文通过lo接口传送给eth0网卡然后向外发出。 此时的源IP地址为VIP，目标IP为CIP 
(f) 响应报文最终送达至客户端

2. LVS-DR模型的特性

特点1：保证前端路由将目标地址为VIP报文统统发给Director Server，而不是RS
RS可以使用私有地址；也可以是公网地址，如果使用公网地址，此时可以通过互联网对RIP进行直接访问
RS跟Director Server必须在同一个物理网络中
所有的请求报文经由Director Server，但响应报文必须不能进过Director Server
不支持地址转换，也不支持端口映射
RS可以是大多数常见的操作系统
RS的网关绝不允许指向DIP(因为我们不允许他经过director)
RS上的lo接口配置VIP的IP地址
缺陷：RS和DS必须在同一机房中
3. 特点1的解决方案：

在前端路由器做静态地址路由绑定，将对于VIP的地址仅路由到Director Server
存在问题：用户未必有路由操作权限，因为有可能是运营商提供的，所以这个方法未必实用
arptables：在arp的层次上实现在ARP解析时做防火墙规则，过滤RS响应ARP请求。这是由iptables提供的
修改RS上内核参数（arp_ignore和arp_announce）将RS上的VIP配置在lo接口的别名上，并限制其不能响应对VIP地址解析请求。
```
```
(a) 当用户请求到达Director Server，此时请求的数据报文会先到内核空间的PREROUTING链。 此时报文的源IP为CIP，目标IP为VIP 。
(b) PREROUTING检查发现数据包的目标IP是本机，将数据包送至INPUT链
(c) IPVS比对数据包请求的服务是否为集群服务，若是，在请求报文的首部再次封装一层IP报文，封装源IP为为DIP，目标IP为RIP。然后发至POSTROUTING链。 此时源IP为DIP，目标IP为RIP 
(d) POSTROUTING链根据最新封装的IP报文，将数据包发至RS（因为在外层封装多了一层IP首部，所以可以理解为此时通过隧道传输）。 此时源IP为DIP，目标IP为RIP
(e) RS接收到报文后发现是自己的IP地址，就将报文接收下来，拆除掉最外层的IP后，会发现里面还有一层IP首部，而且目标是自己的lo接口VIP，那么此时RS开始处理此请求，处理完成之后，通过lo接口送给eth0网卡，然后向外传递。 此时的源IP地址为VIP，目标IP为CIP
(f) 响应报文最终送达至客户端

LVS-Tun模型特性

RIP、VIP、DIP全是公网地址
RS的网关不会也不可能指向DIP
所有的请求报文经由Director Server，但响应报文必须不能进过Director Server
不支持端口映射
RS的系统必须支持隧道
其实企业中最常用的是 DR 实现方式，而 NAT 配置上比较简单和方便，后边实践中会总结 DR 和 NAT 具体使用配置过程。
```
### 30 lvs调度算法

```
1. 轮叫调度 rr
这种算法是最简单的，就是按依次循环的方式将请求调度到不同的服务器上，该算法最大的特点就是简单。轮询算法假设所有的服务器处理请求的能力都是一样的，调度器会将所有的请求平均分配给每个真实服务器，不管后端 RS 配置和处理能力，非常均衡地分发下去。

2. 加权轮叫 wrr
这种算法比 rr 的算法多了一个权重的概念，可以给 RS 设置权重，权重越高，那么分发的请求数越多，权重的取值范围 0 – 100。主要是对rr算法的一种优化和补充， LVS 会考虑每台服务器的性能，并给每台服务器添加要给权值，如果服务器A的权值为1，服务器B的权值为2，则调度到服务器B的请求会是服务器A的2倍。权值越高的服务器，处理的请求越多。

3. 最少链接 lc
这个算法会根据后端 RS 的连接数来决定把请求分发给谁，比如 RS1 连接数比 RS2 连接数少，那么请求就优先发给 RS1 

4. 加权最少链接 wlc
这个算法比 lc 多了一个权重的概念。

5. 基于局部性的最少连接调度算法 lblc
这个算法是请求数据包的目标 IP 地址的一种调度算法，该算法先根据请求的目标 IP 地址寻找最近的该目标 IP 地址所有使用的服务器，如果这台服务器依然可用，并且有能力处理该请求，调度器会尽量选择相同的服务器，否则会继续选择其它可行的服务器

6. 复杂的基于局部性最少的连接算法 lblcr
记录的不是要给目标 IP 与一台服务器之间的连接记录，它会维护一个目标 IP 到一组服务器之间的映射关系，防止单点服务器负载过高。

7. 目标地址散列调度算法 dh
该算法是根据目标 IP 地址通过散列函数将目标 IP 与服务器建立映射关系，出现服务器不可用或负载过高的情况下，发往该目标 IP 的请求会固定发给该服务器。

8. 源地址散列调度算法 sh
与目标地址散列调度算法类似，但它是根据源地址散列算法进行静态分配固定的服务器资源。
```

### 31 nginx lvs对比

```
对软件实现负载均衡的几个软件,从性能和稳定上还是LVS最强
不过就因为LVS忒牛了，配置也最麻烦了，而且健康检测需要另外配置Ldirector，其他HAPROXY和NGINX自己就用，而且配置比较简单。
 
门户级别的用HAPROXY或者NGINX就OK了，到了门户级别在用LVS+Idirector吧
下面来分析一下两者：

一、lvs的优势：

1、抗负载能力强，因为lvs工作方式的逻辑是非常之简单，而且工作在网络4层仅做请求分发之用，没有流量，所以在效率上基本不需要太过考虑。在我手里的 lvs，仅仅出过一次问题：在并发最高的一小段时间内均衡器出现丢包现象，据分析为网络问题，即网卡或linux2.4内核的承载能力已到上限，内存和 cpu方面基本无消耗。

2、配置性低，这通常是一大劣势，但同时也是一大优势，因为没有太多可配置的选项，所以除了增减服务器，并不需要经常去触碰它，大大减少了人为出错的几率。

3、工作稳定，因为其本身抗负载能力很强，所以稳定性高也是顺理成章，另外各种lvs都有完整的双机热备方案，所以一点不用担心均衡器本身会出什么问题，节点出现故障的话，lvs会自动判别，所以系统整体是非常稳定的。

4、无流量，上面已经有所提及了。lvs仅仅分发请求，而流量并不从它本身出去，所以可以利用它这点来做一些线路分流之用。没有流量同时也保住了均衡器的IO性能不会受到大流量的影响。

5、基本上能支持所有应用，因为lvs工作在4层，所以它可以对几乎所有应用做负载均衡，包括http、数据库、聊天室等等。

另：lvs也不是完全能判别节点故障的，譬如在wlc分配方式下，集群里有一个节点没有配置VIP，会使整个集群不能使用，这时使用wrr分配方式则会丢掉一台机。目前这个问题还在进一步测试中。所以，用lvs也得多多当心为妙。

Nginx的优点:
性能好，可以负载超过1万的并发。
功能多，除了负载均衡，还能作Web服务器，而且可以通过Geo模块来实现流量分配。
社区活跃，第三方补丁和模块很多
支持gzip proxy
缺点:
不支持session保持。
对后端realserver的健康检查功能效果不好。而且只支持通过端口来检测，不支持通过url来检测。
nginx对big request header的支持不是很好，如果client_header_buffer_size设置的比较小，就会返回400bad request页面。
Haproxy的优点:
它的优点正好可以补充nginx的缺点。支持session保持，同时支持通过获取指定的url来检测后端服务器的状态。
支持tcp模式的负载均衡。比如可以给MySQL的从服务器集群和邮件服务器做负载均衡。
缺点：
不支持虚拟主机(这个很傻啊)
目前没有nagios和cacti的性能监控模板
LVS的优点:
性能好，接近硬件设备的网络吞吐和连接负载能力。
LVS的DR模式，支持通过广域网进行负载均衡。这个其他任何负载均衡软件目前都不具备。
缺点：
比较重型。另外社区不如nginx活跃。


二、nginx和lvs作对比的结果

1、nginx工作在网络的7层，所以它可以针对http应用本身来做分流策略，比如针对域名、目录结构等，相比之下lvs并不具备这样的功能，所以 nginx单凭这点可利用的场合就远多于lvs了；但nginx有用的这些功能使其可调整度要高于lvs，所以经常要去触碰触碰，由lvs的第2条优点 看，触碰多了，人为出问题的几率也就会大。

2、nginx对网络的依赖较小，理论上只要ping得通，网页访问正常，nginx就能连得通，nginx同时还能区分内外网，如果是同时拥有内外网的 节点，就相当于单机拥有了备份线路；lvs就比较依赖于网络环境，目前来看服务器在同一网段内并且lvs使用direct方式分流，效果较能得到保证。另 外注意，lvs需要向托管商至少申请多一个ip来做Visual IP，貌似是不能用本身的IP来做VIP的。要做好LVS管理员，确实得跟进学习很多有关网络通信方面的知识，就不再是一个HTTP那么简单了。

3、nginx安装和配置比较简单，测试起来也很方便，因为它基本能把错误用日志打印出来。lvs的安装和配置、测试就要花比较长的时间了，因为同上所述，lvs对网络依赖比较大，很多时候不能配置成功都是因为网络问题而不是配置问题，出了问题要解决也相应的会麻烦得多。

4、nginx也同样能承受很高负载且稳定，但负载度和稳定度差lvs还有几个等级：nginx处理所有流量所以受限于机器IO和配置；本身的bug也还是难以避免的；nginx没有现成的双机热备方案，所以跑在单机上还是风险较大，单机上的事情全都很难说。

5、nginx可以检测到服务器内部的故障，比如根据服务器处理网页返回的状态码、超时等等，并且会把返回错误的请求重新提交到另一个节点。目前lvs中 ldirectd也能支持针对服务器内部的情况来监控，但lvs的原理使其不能重发请求。重发请求这点，譬如用户正在上传一个文件，而处理该上传的节点刚 好在上传过程中出现故障，nginx会把上传切到另一台服务器重新处理，而lvs就直接断掉了，如果是上传一个很大的文件或者很重要的文件的话，用户可能 会因此而恼火。

6、nginx对请求的异步处理可以帮助节点服务器减轻负载，假如使用apache直接对外服务，那么出现很多的窄带链接时apache服务器将会占用大 量内存而不能释放，使用多一个nginx做apache代理的话，这些窄带链接会被nginx挡住，apache上就不会堆积过多的请求，这样就减少了相 当多的内存占用。这点使用squid也有相同的作用，即使squid本身配置为不缓存，对apache还是有很大帮助的。lvs没有这些功能，也就无法能 比较。

7、nginx能支持http和email（email的功能估计比较少人用），lvs所支持的应用在这点上会比nginx更多。

在使用上，一般最前端所采取的策略应是lvs，也就是DNS的指向应为lvs均衡器，lvs的优点令它非常适合做这个任务。

重要的ip地址，最好交由lvs托管，比如数据库的ip、webservice服务器的ip等等，这些ip地址随着时间推移，使用面会越来越大，如果更换ip则故障会接踵而至。所以将这些重要ip交给lvs托管是最为稳妥的，这样做的唯一缺点是需要的VIP数量会比较多。

nginx可作为lvs节点机器使用，一是可以利用nginx的功能，二是可以利用nginx的性能。当然这一层面也可以直接使用squid，squid的功能方面就比nginx弱不少了，性能上也有所逊色于nginx。

nginx也可作为中层代理使用，这一层面nginx基本上无对手，唯一可以撼动nginx的就只有lighttpd了，不过lighttpd目前还没有 能做到nginx完全的功能，配置也不那么清晰易读。另外，中层代理的IP也是重要的，所以中层代理也拥有一个VIP和lvs是最完美的方案了。

nginx也可作为网页静态服务器，不过超出了本文讨论的范畴，简单提一下。

具体的应用还得具体分析，如果是比较小的网站（日PV<1000万），用nginx就完全可以了，如果机器也不少，可以用DNS轮询，lvs所耗费的机器还是比较多的；大型网站或者重要的服务，机器不发愁的时候，要多多考虑利用lvs。
**************************************************************************
现在网络中常见的的负载均衡主要分为两种：一种是通过硬件来进行进行，常见的硬件有比较昂贵的NetScaler、F5、Radware和Array等商用的负载均衡器，也有类似于LVS、Nginx、HAproxy的基于Linux的开源的负载均衡策略。

另一种负载均衡的方式是通过软件：比较常见的有LVS、Nginx、HAproxy等，其中LVS是建立在四层协议上面的，而另外Nginx和HAproxy是建立在七层协议之上的，下面分别介绍关于
LVS：使用集群技术和linux操作系统实现一个高性能、高可用的服务器，它具有很好的可伸缩性（Scalability）、可靠性（Reliability）和可管理性（Manageability）。
LVS的特点是：
1、抗负载能力强、是工作在网络4层之上仅作分发之用，没有流量的产生；
2、配置性比较低，这是一个缺点也是一个优点，因为没有可太多配置的东西，所以并不需要太多接触，大大减少了人为出错的几率；
3、工作稳定，自身有完整的双机热备方案；
4、无流量，保证了均衡器IO的性能不会收到大流量的影响；
5、应用范围比较广，可以对所有应用做负载均衡；
6、LVS需要向IDC多申请一个IP来做Visual IP，因此需要一定的网络知识，所以对操作人的要求比较高。
Nginx的特点是：
1、工作在网络的7层之上，可以针对http应用做一些分流的策略，比如针对域名、目录结构；
2、Nginx对网络的依赖比较小；
3、Nginx安装和配置比较简单，测试起来比较方便；
4、也可以承担高的负载压力且稳定，一般能支撑超过1万次的并发；
5、Nginx可以通过端口检测到服务器内部的故障，比如根据服务器处理网页返回的状态码、超时等等，并且会把返回错误的请求重新提交到另一个节点，不过其中缺点就是不支持url来检测；
6、Nginx对请求的异步处理可以帮助节点服务器减轻负载；
7、Nginx能支持http和Email，这样就在适用范围上面小很多；
8、不支持Session的保持、对Big request header的支持不是很好，另外默认的只有Round-robin和IP-hash两种负载均衡算法。
HAProxy的特点是：
1、HAProxy是工作在网络7层之上。
2、能够补充Nginx的一些缺点比如Session的保持，Cookie的引导等工作
3、支持url检测后端的服务器出问题的检测会有很好的帮助。
4、更多的负载均衡策略比如：动态加权轮循(Dynamic Round Robin)，加权源地址哈希(Weighted Source Hash)，加权URL哈希和加权参数哈希(Weighted Parameter Hash)已经实现
5、单纯从效率上来讲HAProxy更会比Nginx有更出色的负载均衡速度。
6、HAProxy可以对mysql进行负载均衡，对后端的DB节点进行检测和负载均衡。
```

```
现在网站发展的趋势对网络负载均衡的使用是随着网站规模的提升根据不同的阶段来使用不同的技术：
第一阶段：利用Nginx或者HAProxy进行单点的负载均衡，这一阶段服务器规模刚脱离开单服务器、单数据库的模式，需要一定的负载均衡，但是 仍然规模较小没有专业的维护团队来进行维护，也没有需要进行大规模的网站部署。这样利用Nginx或者HAproxy就是第一选择，此时这些东西上手快， 配置容易，在七层之上利用HTTP协议就可以。这时是第一选择
第二阶段：随着网络服务进一步扩大，这时单点的Nginx已经不能满足，这时使用LVS或者商用F5就是首要选择，Nginx此时就作为LVS或者 F5的节点来使用，具体LVS或者F5的是选择是根据公司规模，人才以及资金能力来选择的，这里也不做详谈，但是一般来说这阶段相关人才跟不上业务的提 升，所以购买商业负载均衡已经成为了必经之路。
第三阶段：这时网络服务已经成为主流产品，此时随着公司知名度也进一步扩展，相关人才的能力以及数量也随之提升，这时无论从开发适合自身产品的定制，以及降低成本来讲开源的LVS，已经成为首选，这时LVS会成为主流。
最终形成比较理想的状态为：F5/LVS<—>Haproxy<—>Squid/Varnish<—>AppServer。
```

### 32 编程语言 
```
Go应该是最年轻，它设计的目标主要是应用于系统编程，这也显现的它的优点：速度一流，同时它的并行也是这几个中最好的。

Ruby的速度相对慢点，它的目标是让程序员快乐的编程，所以在使用的过程中是比较舒服的。在Ruby里的面对对象是最完全的，比如说正则表达式也是对象，真正的一切皆对象。而且还可以修改标准库里面的类和方法，这也就是它元编程的强大特性。同时还有很多函数式编程的特性，代码块的改善程序员更快乐的编程。

Python在实际的应用是最广泛的，人生苦短，我用Python，说明它相对简单，容易入门，而且有很多不同类型的库和框架，在实际应用工程中比较广。

```