<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-10
title: Docker 私有仓库搭建
tags: Docker
images: https://os4u.info/blog/img/sun.png
category: Docker 
status: publish
summary: Dockers不仅提供了一个中央仓库，同时也允许我们使用registry搭建本地私有仓库。本文主要介绍私有仓库的搭建，以及一些问题处理。

-->
![images](https://os4u.info/blog/docker/images/docker-group.png)
#### docker 私有仓库搭建
```
Dockers不仅提供了一个中央仓库，同时也允许我们使用registry搭建本地私有仓库。
目前Docker Registry已经升级到了v2，最新版的Docker已不再支持v1。
Registry v2使用Go语言编写，在性能和安全性上做了很多优化，重新设计了镜像的存储格式。本文是是用v2版本安装。

```

##### 一、使用私有仓库有许多优点：
```
1.节省网络带宽，针对于每个镜像不用每次都去中央仓库上面去下载，只需从私有仓库中下载即可；
2.提供镜像资源利用，针对于公司内部使用的镜像，推送到本地的私有仓库中，以供公司内部相关人员使用。

```
##### 二、下面在本地搭建私有仓库


##### 1) 环境准备
```
环境：两个装有Docker(版本1.10.3以上), 操作系统：CentOS 7
服务器一：docker-server-1  172.16.66.215 用作私有仓库
服务器二：docker-server-2  172.16.66.216 用户开发机 

此处准备了两个虚拟机，分别都安装了Docker，其中216机器用作开发机，215机器用作registry私有仓库机器。
环境准备好之后接下来我们就开始搭建私有镜像仓库。

```
##### 2) 开始搭建

首先在172.16.66.215机器上下载registry镜像

```

[root@docker-server-1 ~]# docker pull registry
Using default tag: latest
Trying to pull repository docker.io/library/registry ...
latest: Pulling from docker.io/library/registry

b7f33cc0b48e: Pull complete
46730e1e05c9: Pull complete
b7574c100797: Downloading [=======================] 3.179 MB/6.726 MB
2e5b74c7b611: Download complete
b7574c100797: Downloading [=======================] 3.506 MB/6.726 MB

b7574c100797: Pull complete
2e5b74c7b611: Pull complete
ba42bd458a59: Pull complete
Digest: sha256:946480a23b33480b8e7cdb89b82c1bd6accae91a8e66d017e21e8b56551f6209
Status: Downloaded newer image for docker.io/registry:latest
```
下载完之后我们通过该镜像启动一个容器

编辑registry的配置文件

在docker-server-1（172.16.66.215 ）的/opt/docker/registry/config.yml中，内容如下：
```
**************配置文件分割线**************
version: 0.1
log:
  fields:
    service: registry
storage:
  delete:
    enabled: true
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3   
**************配置文件分割线**************
```
编辑完成后，保存。
```
[root@docker-server-1 ~]# docker run -d -p 5000:5000 --restart=always --name="registry-server" -v /opt/data/registry:/tmp/registry -v /opt/docker/registry/config.yml:/etc/docker/registry/config.yml docker.io/registry
15eecbe63028ef25de62ac344be36233c22e52cec07c1636d8cbe5f4b12add4a
[root@docker-server-1 ~]# docker ps -a
CONTAINER ID        IMAGE                            COMMAND                  CREATED             STATUS                      PORTS                    NAMES
15eecbe63028        docker.io/registry               "/entrypoint.sh /etc/"   16 seconds ago      Up 15 seconds               0.0.0.0:5000->5000/tcp   registry-server
```
```
【注】
默认情况下，会将仓库存放于容器内的/tmp/registry目录下，这样如果容器被删除，则存放于容器中的镜像也会丢失。
所以我们一般情况下会指定本地一个目录挂载到容器内的/tmp/registry下。

--restart 标志会检查容器的退出代码，并据此来决定是否要重启容器，默认是不会重启。
--restart的参数说明
always：无论容器的退出代码是什么，Docker都会自动重启该容器。
on-failure：只有当容器的退出代码为非0值的时候才会自动重启。
另外，参数还接受一个可选的重启次数参数。
--restart=on-fialure:5`表示当容器退出代码为非0时。Docker会尝试自动重启该容器，最多5次。
```
可以看到我们启动了一个容器，地址为：172.16.66.215:5000。

查看配置：
```
[root@docker-server-1 ~]# docker exec registry-server cat /etc/docker/registry/config.yml
version: 0.1
log:
  fields:
    service: registry
storage:
  delete:
    enabled: true
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

##### 3) 测试

接下来我们就要操作把一个本地镜像push到私有仓库中。首先在172.16.66.216机器下pull一个比较小的镜像来测试（此处使用的是busybox）。
```
[root@docker-server-2 ~]# docker pull busybox
Using default tag: latest
latest: Pulling from library/busybox

fdab12439263: Pull complete
Digest: sha256:f102731ae8898217038060081c205aa3a4ae3f910c2aaa7b3adeb6da9841d963
Status: Downloaded newer image for busybox:latest

[root@docker-server-2 ~]# docker images
REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
busybox                            latest              1efc1d465fd6        2 weeks ago         1.095 MB
```
接下来修改一下该镜像的tag。
```
[root@docker-server-2 ~]# docker tag busybox 172.16.66.215:5000/busybox
[root@docker-server-2 ~]# docker images
REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
busybox                            latest              1efc1d465fd6        2 weeks ago         1.095 MB
172.16.66.215:5000/busybox         latest              1efc1d465fd6        2 weeks ago         1.095 MB
```
接下来把打了tag的镜像上传到私有仓库。
```
[root@docker-server-2 ~]# docker push 172.16.66.215:5000/busybox
The push refers to a repository [172.16.66.215:5000/busybox]
68737f5031b3: Pushed
latest: digest: sha256:f102731ae8898217038060081c205aa3a4ae3f910c2aaa7b3adeb6da9841d963 size: 527
[root@docker-server-2 ~]#
```
```
【注】可能有push失败的情况如下：

2017/01/05 11:01:17 Error: Invalid registry endpoint https://172.16.66.215:5000/v1/: Get https://172.16.66.215:5000/v1/_ping: dial tcp 172.16.66.215:5000: connection refused. If this private registry supports only HTTP or HTTPS with an unknown CA certificate, please add `--insecure-registry 172.16.66.215:5000` to the daemon's arguments. In the case of HTTPS, if you have access to the registry's CA certificate, no need for the flag; simply place the CA certificate at /etc/docker/certs.d/172.16.66.215:5000/ca.crt 

因为Docker从1.3.X之后，与docker registry交互默认使用的是https，然而此处搭建的私有仓库只提供http服务，所以当与私有仓库交互时就会报上面的错误。为了解决这个问题需要在启动docker server时增加启动参数为默认使用http访问。修改docker启动配置文件（此处是修改172.16.66.216机器的配置）CentOS下配置文件地址为：/etc/sysconfig/docker，在其中增加–insecure-registry 172.16.66.215:5000如下所示：

[root@docker-server-2 ~]# vi /etc/sysconfig/docker
====docker version 1.10.3配置=====
OPTIONS='--storage-driver=devicemapper --selinux-enabled --log-driver=journald -g /data/docker --insecure-registry 172.16.66.215:5000'
====docker version 1.12.5配置=====
# 配置/etc/docker 添加文件daemon.json。内容如下：
{ "insecure-registries":["172.16.66.215:5000"] }
```
修改完之后，重启Docker服务。
```
[root@docker-server-2 ~]# systemctl restart docker
```
重启完之后我们再次运行推送命令，把本地镜像推送到私有服务器上。

接下来我们删除本地镜像，然后从私有仓库中pull下来该镜像。
```
[root@docker-server-2 ~]# docker rmi 172.16.66.215:5000/busybox
[root@docker-server-2 ~]# docker rmi busybox
[root@docker-server-2 ~]# docker pull 172.16.66.215:5000/busybox
Using default tag: latest
latest: Pulling from busybox
fdab12439263: Pull complete
Digest: sha256:f102731ae8898217038060081c205aa3a4ae3f910c2aaa7b3adeb6da9841d963
Status: Downloaded newer image for 172.16.66.215:5000/busybox:latest
```
到此就搭建好了Docker私有仓库。上面搭建的仓库是不需要认证的，我们可以结合nginx和https实现认证和加密功能。


##### 4) 管理仓库中的镜像

###### a) 查询
```
如果我们想要查询私有仓库中的所有镜像，使用docker search命令：

# docker search registry_ip:5000/
如果要查询仓库中指定账户下的镜像，则使用如下命令：

# docker search registry_ip:5000/account/
同时也可以指定镜像查询。

curl -XGET http://172.16.66.215:5000/v2/_catalog
curl -XGET http://172.16.66.215:5000/v2/busybox/tags/list
curl -XGET http://172.16.66.215:5000/v2/busybox/manifests/latest
```
```
[root@docker-server-2 ~]# curl -XGET http://172.16.66.215:5000/v2/_catalog
{"repositories":["busybox"]}
[root@docker-server-2 ~]# curl -XGET http://172.16.66.215:5000/v2/busybox/tags/list
{"name":"busybox","tags":["latest"]}
[root@docker-server-2 ~]# curl -XGET http://172.16.66.215:5000/v2/busybox/manifests/latest
{
   "schemaVersion": 1,
   "name": "busybox",
   "tag": "latest",
   "architecture": "amd64",
   "fsLayers": [
      {
         "blobSum": "sha256:a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4"
      },
      {
         "blobSum": "sha256:fdab12439263ca89c449f09a489b6fbf40af8526a3dda279c8954451703641c3"
      }
   ],
   "history": [
      {
         "v1Compatibility": "{\"architecture\":\"amd64\",\"config\":{\"Hostname\":\"f2fbc9d370cd\",\"Domainname\":\"\",\"User\":\"\",\"AttachStdin\":false,\"AttachStdout\":false,\"AttachStderr\":false,\"Tty\":false,\"OpenStdin\":false,\"StdinOnce\":false,\"Env\":[\"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin\"],\"Cmd\":[\"sh\"],\"Image\":\"sha256:0e65a4041bdf2c31576d2c88af271d10bb03634cac58d8d99a79422b69f23c1f\",\"Volumes\":null,\"WorkingDir\":\"\",\"Entrypoint\":null,\"OnBuild\":null,\"Labels\":{}},\"container\":\"e9c043e65360fe1b62846a0a88e0217ef03e2e1abd84fd092417256015155b18\",\"container_config\":{\"Hostname\":\"f2fbc9d370cd\",\"Domainname\":\"\",\"User\":\"\",\"AttachStdin\":false,\"AttachStdout\":false,\"AttachStderr\":false,\"Tty\":false,\"OpenStdin\":false,\"StdinOnce\":false,\"Env\":[\"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin\"],\"Cmd\":[\"/bin/sh\",\"-c\",\"#(nop) \",\"CMD [\\\"sh\\\"]\"],\"Image\":\"sha256:0e65a4041bdf2c31576d2c88af271d10bb03634cac58d8d99a79422b69f23c1f\",\"Volumes\":null,\"WorkingDir\":\"\",\"Entrypoint\":null,\"OnBuild\":null,\"Labels\":{}},\"created\":\"2016-12-22T19:21:03.23977775Z\",\"docker_version\":\"1.12.3\",\"id\":\"eeaaafdf6e984582bc1781743850afdff246b6e1774503711dcd29585e8095a2\",\"os\":\"linux\",\"parent\":\"81b09006921b48e15519b9fe0ade9c1addc70d161867bfa7366fed2b7d50cdde\",\"throwaway\":true}"
      },
      {
         "v1Compatibility": "{\"id\":\"81b09006921b48e15519b9fe0ade9c1addc70d161867bfa7366fed2b7d50cdde\",\"created\":\"2016-12-22T19:21:02.783485892Z\",\"container_config\":{\"Cmd\":[\"/bin/sh -c #(nop) ADD file:aee188d5d0618c8fd71cbe237ca14869d041a08f41ea22ebf72ad12b8a727c0b in / \"]}}"
      }
   ],
   "signatures": [
      {
         "header": {
            "jwk": {
               "crv": "P-256",
               "kid": "4IPF:QVMP:5NOR:D7YI:GUVG:65CW:HPUV:V7VZ:3NF6:EYOH:RLUM:VEZF",
               "kty": "EC",
               "x": "LBs6AMAQ5F698xFX8D3WNxvFZ6GKzuu4_Vc78ZuSVSg",
               "y": "rfOovTfKLpkkTqQL_xiwCO9uMfxVxK-zf1OoICuh-0M"
            },
            "alg": "ES256"
         },
         "signature": "hWIF2JLJJNV786jwTJKsFP6OwZb-XvLwnzjU6JSwLLarnBm02FMRo220WZBCzu3M9XMe1E9R4ebQ-4ASixEE4w",
         "protected": "eyJmb3JtYXRMZW5ndGgiOjIwODcsImZvcm1hdFRhaWwiOiJDbjAiLCJ0aW1lIjoiMjAxNy0wMS0wNlQwNDozNTo1NVoifQ"
      }
   ]
}[root@docker-server-3 ~]#
```
###### b) 删除

```
目前尚未找到方法删除私有仓库中的镜像，尝试过直接从仓库存储目录中删除镜像文件，但是并不能成功删除镜像。

Registry V2

Registry V2.2的安装脚本如下：

#!/bin/bash
# Description: create a private registry v2.2
# Version: 0.2
#
# Author: Jun 651516215@qq.com
# Date: 2017/01/03

set -o xtrace

if [[ $UID -ne 0 ]]; then
    echo "Not root user. Please run as root."
    exit 0
fi

# Install Docker if not
docker -v
if [[ $? -ne 0 ]]; then
    echo "Please install Docker first."
    exit 0
fi

REGISTRY_VERSION=2.2

# Download registry image v2.2
docker pull registry:${REGISTRY_VERSION}

# Start registry container
mkdir /opt/registry
docker run -d -p 5000:5000 --restart=always -v /opt/registry:/var/lib/registry --name hummer_registry registry:${REGISTRY_VERSION}

Registry的存放目录在Docker Hub上显示的是/tmp/registry-dev，但是映射之后发现并没有存放在该目录，查看源码发现，镜像信息存放在/var/lib/registry目录下，因此这里修改为将/opt/registry目录映射到/var/lib/registry。
```
###### c) 上传镜像
```
# docker push registry:5000/image_name
```
###### d) 查看镜像
```
# curl -XGET http://registry:5000/v2/_catalog
# curl -XGET http://registry:5000/v2/image_name/tags/list
```
###### e) 删除镜像 

获取registry中镜像的digest方法
```
imageSha256=`docker push 172.16.66.215:5000/busybox | grep digest | awk '{print $3}'

# 官方建议获取方法
imageSha256=`curl -s --header "Accept: application/vnd.docker.distribution.manifest.v2+json" -I  -X HEAD http://172.16.66.215:5000/v2/busybox/manifests/latest | grep Docker-Content-Digest  | awk '{print $2}'` 

curl  -I -X DELETE http://172.16.66.215:5000/v2/busybox/manifests/${imageSha256}
[root@docker-server-3 ~]# curl  -I -X DELETE http://172.16.66.215:5000/v2/busybox/manifests/sha256:f102731ae8898217038060081c205aa3a4ae3f910c2aaa7b3adeb6da9841d963
HTTP/1.1 202 Accepted
Docker-Distribution-Api-Version: registry/2.0
X-Content-Type-Options: nosniff
Date: Fri, 06 Jan 2017 04:37:37 GMT
Content-Length: 0
Content-Type: text/plain; charset=utf-8
```
检验删除结果
```
[root@docker-server-2 ~]# docker pull 172.16.66.215:5000/busybox
Using default tag: latest
Pulling repository 172.16.66.215:5000/busybox
Error: image busybox:latest not found
```
#### 导出本地镜像上传到私有仓库

###### 查看docker当前镜像
```
[root@docker-server-2 ~]# docker images
REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
docker.io/swarm                  latest              942fd5fd357e        3 months ago        19.47 MB
docker.io/lemonbar/centos6-ssh   latest              efd998bd6817        2 years ago         296.9 MB
[root@docker-server-2 ~]#
```
###### 查看当前运行的容器
```
[root@docker-server-2 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
284f6e5f917e        test-server:v1.0    "/bin/bash"         2 days ago          Exited (137) 22 hours ago                       test-cdn-server-1
```
###### 导出当前容器到镜像
```
[root@docker-server-2 ~]# docker commit 284f6e5f917e 172.16.66.215:5000/ljdemo/centos:latest
sha256:dec2b3b9d72da57b2cf7684603baa8923151ebabc4c2846e741d77e6ccec7d54
[root@docker-server-2 ~]# docker images
REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
172.16.66.215:5000/ljdemo/centos   latest              dec2b3b9d72d        5 seconds ago       1.178 GB
```
###### 查看docker当前镜像
```
[root@docker-server-3 ~]# docker images
REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
172.16.66.215:5000/ljdemo/centos   latest              dec2b3b9d72d        5 seconds ago       1.178 GB
```
###### 将本地镜像推送到私有仓库
```
[root@docker-server-3 ~]# docker push  172.16.66.215:5000/ljdemo/centos:latest
The push refers to a repository [172.16.66.215:5000/ljdemo/centos]
555adeeaa523: Pushed
34a5b31d8f9f: Pushed
5f70bf18a086: Pushed
5dde9e7abaae: Pushed
0d2c9d3bef41: Pushed
26231980c1a5: Pushed
7ec2a9eb1017: Pushed
29e0db08def6: Pushed
9225ac3d0295: Pushed
0f26de94dce0: Pushed
821f107b9dc4: Pushed
latest: digest: sha256:19ff89368d00b3f5beb64c83f62881faf24411831e1f95801f7f6e19312cee5a size: 3442
[root@docker-server-3 ~]#
```

##### 三、配置带用户权限
到上面为止，registry已经可以使用了。如果想要控制registry的使用权限，使其只有在登录用户名和密码之后才能使用的话，还需要做额外的设置。

registry的用户名密码文件可以通过htpasswd来生成：
```
[root@docker-server-2 registry]# mkdir /opt/docker/registry/auth/  
[root@docker-server-2 registry]# docker run --entrypoint htpasswd docker.io/registry:latest -Bbn ljkj ljkj2016 >> /opt/docker/registry/auth/htpasswd
```
上面这条命令是为ljkj用户名生成密码为ljkj2016的一条用户信息，保存在/opt/docker/registry/auth/htpasswd文件里面，文件中存的密码是被加密过的。

使用带用户权限的registry时候，容器的启动命令就跟上面不一样了，将之前的容器停掉并删除，然后执行下面的命令：
```
[root@docker-server-2 registry]# docker run -d -p 5000:5000 \
  --restart=always --name="registry-server" \
  -v /opt/docker/registry/auth/:/auth/ \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm"  \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  -v /opt/data/registry:/tmp/registry \
  -v /opt/docker/registry/config.yml:/etc/docker/registry/config.yml \
  docker.io/registry
```
这时，如果直接想查看仓库信息、pull或push都会出现权限报错。必须先使用docker login 命令来登录私有仓库：
```
[root@docker-server-2 registry]#  docker login 172.16.66.215:5000  
```
根据提示，输入用户名和密码即可。如果登录成功，会在/root/.docker/config.json文件中保存账户信息，这样就可以继续使用了。
```
[root@docker-server-2 registry]# cat ~/.docker/config.json
{
	"auths": {
		"172.16.66.215:5000": {
			"auth": "bGprajpsamtqMjAxNg==",
			"email": "ljkj@ue.cn"
		}
	}
}
```



![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 