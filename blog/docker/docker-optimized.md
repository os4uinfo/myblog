<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-09
title: Docker优化
tags: Docker
images: https://os4u.info/blog/img/sun.png
category: Docker 
status: publish
summary: Docker优化
-->

1.降低部署时间

场景1:
```
1.编写Dockerfile创建一个“大”镜像：
FROM debian:jessie
RUN dd if=/dev/urandom of=/largefile bs=1024 count=524288

2.编译该dockerfile 附上标签hubuser/largeapp
$ docker build -t hubuser/largeapp .

3.检查大小
$ docker images

4.上传到Docker hub计算时间
$ time docker push hubuser/largeapp
```
此时耗费大量时间。优化如下：
```
优化1：创建私有Docker Registry
$ docker tag hubuser/largeapp dockerhost:5000/largeapp
$ docker push dockerhost:5000/largeapp
```

![image](https://os4u.info/blog/docker/images/bluebird.jpg)

2.改善镜像编译时间

```
1）采用registry镜像
2）复用镜像
例子：
---app
   |__config.ru
   |
   |__Dockerfile
   |
   |__Gemfile
三个文件内容如下：
config.ru
--------------------
app = proc do |env|
   [200, {}, %w(hello, world)]
end
run app
---------------------
Gemfile:
---------------------
source 'https://rubygems.org'
gem 'rack'
gem 'nokogiri'
---------------------
Dockerfile
---------------------
FROM ruby:2.2.2
ADD . /app
WORKDIR /app
RUN bundle install
EXPOSE 9292
CMD rackup -E none
---------------------
```
为了优化这个工作流，可以将准备应用程序依赖的阶段从整个应用程序的镜像构建中剥离出来。

```
1.改变Dockerfile内容，如下：
FROM ruby:2.2.2
ADD Gemfile /app/Gemfile
WORKDIR /app
RUN bundle install
ADD ./app
EXPOSE 9292
CMD rackup -E none

2.编译刚刚改造过的Docker镜像。
3.记录step3的镜像ID，然后重新编辑config.ru内容，再重新编译镜像
从输出的记录中可以看出，编译时间节省了很多。
```
3.减小构建上下文大小

假定基于Git的版本控制管理中有一个Dockerfile文件，在某种程度上.git占了很大磁盘空间。编译镜像会处理这部分内容，故需要大量时间。

优化如下：
```
1.在Dockerfile所在目录下创建一个.dockerignore文件，并将如下内容写入该文件：
.git
2.然后重新编译镜像
```

4.使用缓存代理
```
1.在docker宿主机中搭建apt-cacher-ng缓存代理
$ docker run -d -p 3142:3142 sameersbn/apt-cacher-ng
2.修改Dockerfile,内容如下：
FROM debian:jessie

RUN echo Acquire::http { \
Proxy\"http://dockerhost:3142\"\;\
}\; > /etc/apt-apt.conf.d/01proxy

3.编译之前创建并标记为hubuser/debian:jessie的Dockerfile
$ docker build -t hubuser/debian:jessie

4.更新hubuser/debian:jessie为新的Docker基础镜像，其中包含了要安装的一些列依赖包：
FROM hubuser/debian:jessie

RUN echo deb http://xxx.xx.xx.xx/debian > /etc/apt/sources.list.d/jessie-backports.list

RUN apt-get update && \
apt-get --no-install-recommends install -y openjdk-8-jre-headless

5.确认新的工作流，执行初始化编译使得缓存生效
$ docker build -t argercaching .
6.移除刚才编译创建的镜像，并重新编译该镜像验证缓存生效
$ docker rmi aftwercaching
$ time docker build -t aftercaching .

其他可选择的缓存代理如下：
apt-cacher-ng
sonatype nexus
polipo
squid
```

5.减小Docker镜像尺寸
```
a)链式指令
优化方法：尽量合并执行命令

b)分离编译镜像和部署镜像
例子：
一个简单的web应用：
文件夹内容：Dockerfile，hello.go
hello.go内容如下：
package main
import (
   "fmt"
   "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "hello, world")
}

fun main() {
    http.HandleFun("/", haandler)
    http.ListenAndServe(":8080", nil)
}

Dockerfile内容如下：
FROM golang:1.4.2
ADD hello.go hello.go
RUN go build hello.go
EXPOSE 8080
ENTRYPOINT ["./hello"]

首先看如何操作：
$ docker build -t largeapp .
$ docker run --name large -d largeapp
$ docker exec -it large/bin/ls -lh
优化步骤如下：
1.复制运行中可执行文件到Docker宿主机
$ docker cp -L large:/go/hello ../build
2.查看依赖库
$ docker exec -it large /usr/bin/ldd hello
3.拷贝出其依赖库文件
$ docker cp -L large:/xxx/xxx.so.o ../build
...
4.创建一个新Dockerfile
FROM scratch
ADD hello /app/hello
ADD xx.so.0 /lib/xxxx/xxx.so.0
...
EXOSE 8080
ENTRYPOINT ["/app/hello"]

5.目录结构如下：
---build
  |___Dockerfile
  |___hello
  |___xxx.so.0
  ...
---src
  |___Dockerfile
  |___hello.go
6.编译
$ cd build
$ docker build -t binary .
```



![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 