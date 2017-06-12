<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-10
title: Docker应用负载均衡
tags: Docker
images: https://os4u.info/blog/img/sun.png
category: Docker 
status: publish
summary: 无论我们如何调优Docker应用，总会碰到性能瓶颈。所以为了服务不断增长的用户，我们需要水平扩展Docker应用。
-->

![images](https://os4u.info/blog/docker/images/poly.jpg)

### 负载均衡

无论我们如何调优Docker应用，总会碰到性能瓶颈。所以为了服务不断增长的用户，我们需要水平扩展Docker应用。
本节内容包含如下：
```
1.准备Docker宿主机集群
2.使用nginx来做负载均衡
3.水平扩展Docker应用
4.使用负载均衡器来实现零停机发布
```

#### 1.准备Docker宿主机集群
```
1.准备web应用
app.js内容如下：
var http = require('http');

http.createServer(function (request, response) {

	// 发送 HTTP 头部 
	// HTTP 状态值: 200 : OK
	// 内容类型: text/plain
	response.writeHead(200, {'Content-Type': 'text/plain'});

	// 发送响应数据 "Hello World"
	response.end('Hello World\n');
}).listen(8000);

// 终端打印如下信息
console.log('Server running at http://127.0.0.1:8000/');

2.编写Dockerfile文件来发布该web应用
FROM node:4.0.0
COPY app.js /app/app.js
EXPOSE 8000
cmd ["node", "/app/app.js"]

3.构建镜像
dockerhost$ docker build -t hubuser/app:1.0.0 .
dockerhost$ docker push hubuser/app:1.0.0

4.部署
greenhost00$ docker run -d -p 8000:8000 --log-driver syslog --log-opt syslogtag=webapp hubuser/app:1.0.0
greenhost01$ docker run -d -p 8000:8000 --log-driver syslog --log-opt syslogtag=webapp hubuser/app:1.0.0
greenhost02$ docker run -d -p 8000:8000 --log-driver syslog --log-opt syslogtag=webapp hubuser/app:1.0.0
[更好的方法是：使用chef cookbook来部署docker应用]

elasticsearch使用logstash过滤
filter {
    if [program] == "docker/webapp" {
        json {
            source => "message"
        }
    }
}
```

#### 2.使用nginx来做负载均衡
```
1.nginx 配置如下：
events { }
http {
    upstream app_server {
        server greenhost00:8000;
        server greenhost01:8000;
        server greenhost02:8000;
    }
    server {
        location / {
            proxy_pass http://app_server
        }
    }
}
2. 运行docker容器
dockerhost$ docker run -p 80:80 -d --name=blancer --volume=/root/nginx.conf:/etc/nginx/nginx.conf:ro nginx:1.9.4

3.测试
$ while true; do curl http://dockerhost && sleep 0.1 ;done

4.观察日志情况
```
参考：[ELK日志收集平台](https://os4u.info/blog/docker/docker-monitor-log.html) 

#### 3.水平扩展Docker应用
```
1.增加宿主机来扩展greenhost03，greenhost04

2.在新增机器上部署webapp
greenhost03$ docker run -d -p 8000:8000 --log-driver syslog --log-opt syslogtag=webapp hubuser/app:1.0.0
greenhost04$ docker run -d -p 8000:8000 --log-driver syslog --log-opt syslogtag=webapp hubuser/app:1.0.0

3. 更新nginx配置

events { }
http {
    upstream app_server {
        server greenhost00:8000;
        server greenhost01:8000;
        server greenhost02:8000;
        server greenhost03:8000;
        server greenhost04:8000;
    }
    server {
        location / {
            proxy_pass http://app_server
        }
    }
}

4.重新加载nginx
dockerhost$ docker kill -s HUP balancer

5.测试观察（步骤如上，此处略）
```

#### 4.零停机部署
blue-green部署技术来实现零停机部署
```
1. 更新app.js内容
var http = require('http');

http.createServer(function (request, response) {

	// 发送 HTTP 头部 
	// HTTP 状态值: 200 : OK
	// 内容类型: text/plain
	response.writeHead(200, {'Content-Type': 'text/plain'});

	// 发送响应数据 "Hello World"
	response.end('Hello Webapps!\n');
}).listen(8000);

// 终端打印如下信息
console.log('Server running at http://127.0.0.1:8000/');

2.重新编译新版本
dockerhost$ docker build -t hubuser/app:2.0.0. .
dockerhost$ docker push hubuser/app:2.0.0

3.准备Docker宿主机池，称为宿主机池blue

4.在新宿主机上部署新版本
bluehost00$ docker run -d -p 8000:8000 --log-driver syslog --log-opt syslogtag=webapp hubuser/app:2.0.0
bluehost01$ docker run -d -p 8000:8000 --log-driver syslog --log-opt syslogtag=webapp hubuser/app:2.0.0
bluehost02$ docker run -d -p 8000:8000 --log-driver syslog --log-opt syslogtag=webapp hubuser/app:2.0.0

5.确认宿主机池blue部署成功后，对该组服务器进行preflight checks和测试

6.确认正常工作后，上线，添加到nginx配置中

events { }
http {
    upstream app_server {
        server greenhost00:8000;
        server greenhost01:8000;
        server greenhost02:8000;
        server greenhost03:8000;
        server greenhost04:8000;
        server bluehost00:8000;
        server bluehost01:8000;
        server bluehost02:8000;
    }
    server {
        location / {
            proxy_pass http://app_server
        }
    }
}

7.重载nginx负载均衡器
dockerhost$ docker kill -s HUP balancer

8.此时nginx会把流量同时发往老的版本（1.0.0）和新版本（2.0.0），此时检验是否能正常工作，不能正常工作，则删除bluehost机器，重载nginx回滚。
如果正常，那就可以逐步下线老版本。

9.测试检查（步骤如上，此处略）
```
#### 5.其他负载均衡器
```
Redx
HAProxy
Apache HTTP Server
Vulcand
CloudFoundry's GoRoter
dotCloud's Hipache
```



![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 