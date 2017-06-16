<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-06-14
title: 搭建第一个Docker应用栈
tags: Docker
images: https://os4u.info/blog/img/sun.png
category: Docker 
status: publish
summary: 搭建第一个Docker应用栈
-->

## 应用栈架构
本应用栈架构如下：

![](https://www.os4u.info/blog/docker/images/docker-app-stack.png)

***说明：*** HAProxy时负载均衡代理节点；Redis构建一个非关系型数据库；APP是由python的Django构建的web应用。

## 动手搭建

### 1.获取应用栈各节点所需镜像
具体操作如下：

```
$ sudo docker pull ubuntu
$ sudo docker pull django
$ sudo docker pull haproxy
$ sudo docker pull redis
$ sudo docker images 
redis		latest		83744227b191		6 days ago     98.9MB
ubuntu		latest		7b9b13f7b9c0		12 days ago    118MB
haproxy		latest		c481d2544260		5 weeks ago    136MB
django		latest		eb40dcf64078		5 months ago   436MB
```

### 2.应用栈容器节点互联

本操作中需要用到--link选项，简要介绍下。--link选项能够进行容器间的安全交互通信，使用格式为name:alias。具体使用如下：

```
$ sudo docker run --link redis:redis --name console ubuntu bash
```
***说明：*** 上例中将在ubuntu镜像上启动一个容器，并命名为console，同时将新启动的console容器连接到名为redis的容器上。在使用--link选项时，连接通过容器名来确定容器，这里建议启动容器时自定义容器名。通过--link选项建立容器间的连接，不但可以避免容器的IP和端口暴露到外网导致的安全问题，还可以防止容器在重启后IP地址变化导致的访问失效。它的原理类似于DNS服务器的域名和地址映射。当容器IP地址变化时，Docker将自动维护映射关系中的IP地址。这一关系维护主要体现在容器的/etc/hosts中。

通过上述原理可以讲--link设置理解为一条IP地址的单向记录信息，因此在搭建容器应用栈时，需要注意各个容器的启动顺序，以及对应的--link参数设置。应用栈各节点的连接信息如下：

* 启动redis-master容器节点
* 两个redis-slave容器节点启动时要连接到redis-master上
* 两个APP容器节点启动时要连接到redis-master上
* HAProxy容器节点启动时要连接到两个APP容器节点上

因此容器启动顺序为：redis-master --> redis-slave --> APP --> HAProxy 

同时需要将HAProxy的端口暴露在外部网络，需要用-p参数。

### 3.应用栈容器节点启动
通过上述分析，各个容器节点启动命令如下：

a) 启动redis-master及redis-slave

```
$ sudo docker run -d -it --name redis-master redis /bin/bash
$ sudo docker run -d -it --name redis-slave1 --link redis-master:master redis /bin/bash
$ sudo docker run -d -it --name redis-slave2 --link redis-master:master redis /bin/bash
```

b) 启动APP，即Django

```
$ sudo mkdir -p /data/Django/App1
$ sudo mkdir -p /data/Django/App2
$ sudo docker run -d -it --name APP1 --link redis-master:db  \
-v /data/Django/App1:/usr/src/app django /bin/bash
$ sudo docker run -d -it --name APP2 --link redis-master:db  \
-v /data/Django/App2:/usr/src/app django /bin/bash
```

c) 启动HAProxy容器

```
$ sudo mkdir /data/HAProxy
$ sudo docker run -d -it --name HAProxy --link APP1:APP1 \
--link APP2:APP2 -p 80:80 -v /data/HAProxy:/tmp haproxy /bin/bash
```

d) 启动后docker ps查看，结果如下：

```
$ sudo docker ps -a
CONTAINER ID IMAGE   COMMAND                 CREATED        STATUS        PORTS       NAMES
63ffd0a22daa haproxy "/docker-entrypoin..."  54 seconds ago Up 51 seconds 0.0.0.0:80->80/tcp HAProxy
f0d3b4b611ee django  "/bin/bash"             2 minutes ago  Up 2 minutes                     APP2  
37beec3780e5 django  "/bin/bash"             3 minutes ago  Up 3 minutes                     APP1
90047d05f60f redis   "docker-entrypoint..."  5 minutes ago  Up 5 minutes  6379/tcp           redis-slave2
ecac9a1a14cb redis   "docker-entrypoint..."  5 minutes ago  Up 5 minutes  6379/tcp           redis-slave1
47e0b0814021 redis   "docker-entrypoint..."  6 minutes ago  Up 6 minutes  6379/tcp           redis-master
```

### 4.应用栈容器节点的配置

#### 1) redis master主数据库容器节点的配置

找到master的配置文件路径

```
$ sudo docker inspect 47e0b0814021  | grep data
                "Source": "/var/lib/docker/volumes/6353f08cf4bc3117aaf3715c6ccaee4c483a551287e84325d8bc021b3f720b13/_data"

进入该目录，将配置好的redis.conf配置文件拷贝到此处
# cd /var/lib/docker/volumes/6353f08cf4bc3117aaf3715c6ccaee4c483a551287e84325d8bc021b3f720b13/_data
# cp ~/redis.conf .
```
***说明：***redis.conf 配置如下：
```
bind 0.0.0.0
protected-mode yes
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize yes
supervised no
pidfile "/var/run/redis.pid"
loglevel notice
logfile "/var/log/redis.log"
databases 16
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename "dump.rdb"
dir "/data/redis/data"
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
maxclients 100000
maxmemory 32G
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
```

启动redis-master

```
$ sudo docker attach 47e0b0814021
root@47e0b0814021:/data# ls
redis.conf
root@47e0b0814021:/data# mkdir redis/data -p
root@47e0b0814021:/data# redis-server redis.conf

```

#### 2) redis slave 从数据库容器节点的配置

与redis master相同的操作，将宿主机上的配置文件拷贝到redis slave，并指向master

```
$ sudo docker inspect redis-slave1 | grep data
                "Source": "/var/lib/docker/volumes/31720daf5190d75d4db8dcba609dc07474991487b488671c433c5d77fb393f7a/_data",
```
redis.conf配置需修改地方如下：

```
daemonize yes
pidfile /var/run/redis.pid
slaveof master 6379

```

启动redis

```
# docker attach redis-slave1
root@ecac9a1a14cb:/data#
root@ecac9a1a14cb:/data# ls
redis.conf
root@ecac9a1a14cb:/data# mkdir redis/data -p
root@ecac9a1a14cb:/data# redis-server redis.conf

# docker attach redis-slave2
root@90047d05f60f:/data#
root@90047d05f60f:/data# ls
redis.conf
root@90047d05f60f:/data# mkdir redis/data -p
root@90047d05f60f:/data# redis-server redis.conf
```

#### 3) redis数据库容器节点的测试
在redis master上测试

```
# docker attach redis-master
root@47e0b0814021:~# redis-cli
127.0.0.1:6379> set master 47e0b0814021
OK
127.0.0.1:6379> get master
"47e0b0814021"
127.0.0.1:6379>
```
在redis slave上检查

```
# docker attach redis-slave1
root@ecac9a1a14cb:/data# redis-cli
127.0.0.1:6379> get master
"47e0b0814021"
127.0.0.1:6379>

```

#### 4) APP容器节点的配置
在app容器内执行

```
# docker attach APP1
# pip install redis
Collecting redis
  Downloading redis-2.10.5-py2.py3-none-any.whl (60kB)
    100% |████████████████████████████████| 61kB 234kB/s
Installing collected packages: redis
Successfully installed redis-2.10.5
root@37beec3780e5:/#
root@37beec3780e5:/# cd /usr/src/app/
root@37beec3780e5:/usr/src/app# ls
root@37beec3780e5:/usr/src/app# mkdir dockerweb
root@37beec3780e5:/usr/src/app# cd dockerweb/
root@37beec3780e5:/usr/src/app/dockerweb# ls
root@37beec3780e5:/usr/src/app/dockerweb# django-admin.py startproject redisweb
root@37beec3780e5:/usr/src/app/dockerweb# ls
redisweb
root@37beec3780e5:/usr/src/app/dockerweb# cd redisweb/
root@37beec3780e5:/usr/src/app/dockerweb/redisweb# ls
manage.py  redisweb
root@37beec3780e5:/usr/src/app/dockerweb/redisweb# python manage.py startapp helloworld
root@37beec3780e5:/usr/src/app/dockerweb/redisweb# ls
helloworld  manage.py  redisweb
```
在宿主机上

```
# cd /data/Django/App1/
# ls
dockerweb
# cd /data/Django/App1/dockerweb/redisweb/helloworld
```
编辑views.py，内容如下：

```
from django.shortcuts import render
from django.http import HttpResponse

# Create your views here.
import redis

def hello(request):
    str = redis.__file__
    str += "<br>"
    r = redis.Redis(host='db', port = 6379, db=0)
    info = r.info()
    str += ("Set Hi <br>")
    r.set('Hi', 'HelloWorld-APP1')
    str += ("Get Hi: %s <br>" % r.get('Hi'))
    str += ("Redis Info: <br>")
    str += ("Key: Info Value")
    for key in info:
        str += ("%s: %s <br>" % (key, info[key]))
    return HttpResponse(str)

```
配置setting.py中INSTALLED_APPS

```
# cd /data/Django/App1/dockerweb/redisweb/redisweb
# vim settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'helloworld',
]
```
修改urls.py模式

```
# cd /data/Django/App1/dockerweb/redisweb/redisweb
# vim urls.py
from django.conf.urls import url
from django.contrib import admin
from helloworld.views import hello

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^helloworld$', hello),
]
```

进入容器完成项目的生成

```
root@37beec3780e5:~# cd /usr/src/app/dockerweb/redisweb
root@37beec3780e5:/usr/src/app/dockerweb/redisweb# python manage.py makemigrations
No changes detected
root@37beec3780e5:/usr/src/app/dockerweb/redisweb# python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying sessions.0001_initial... OK
```
至此完成APP1配置，APP2 配置与APP1一样，此处省略。

启动app

```
root@37beec3780e5:/usr/src/app/dockerweb/redisweb# python manage.py runserver 0.0.0.0:8001
Performing system checks...

System check identified no issues (0 silenced).
June 15, 2017 - 15:41:32
Django version 1.10.4, using settings 'redisweb.settings'
Starting development server at http://0.0.0.0:8001/
Quit the server with CONTROL-C.
Invalid HTTP_HOST header: '192.168.4.208'. You may need to add '192.168.4.208' to ALLOWED_HOSTS.
[15/Jun/2017 15:41:36] "GET /helloworld HTTP/1.1" 400 58618

```
#### 5) HAProxy容器节点的配置
在宿主机中编辑

```
# cd /data/HAProxy
# cat haproxy.cfg
global
	log 127.0.0.1 local0
	maxconn 4096
	chroot /usr/local/sbin
	daemon
	nbproc 4
	pidfile /usr/local/sbin/haproxy.pid

defaults
	log	127.0.0.1 local3
	mode http
	option	dontlognull
	option	redispatch
	retries 2
	maxconn	2000
	balance roundrobin
	timeout connect 5000ms
	timeout client 50000ms
	timeout server 50000ms

listen redis_proxy
	bind 0.0.0.0:80
	stats enable
	stats uri /haproxy-stats
	server APP1 APP1:8001	check inter 2000 rise 2 fall 5
	server APP2 APP2:8002	check inter 2000 rise 2 fall 5
```
#### 6) 应用栈外部访问
结果如下图

![](https://www.os4u.info/blog/docker/images/docker-app-stack-result.png)


![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 