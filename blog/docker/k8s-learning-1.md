<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-07-13
title: 使用ReplicationController，Replica Set，Deployment管理Pod
tags: k8s 
images: https://os4u.info/blog/img/sun.png
category: Docker
status: publish
summary: kubernetes学习。
-->


 
本文主要讲k8s自动管理Pod的伸缩。主要利用ReplicationController、Replica Set、Deploymen来实现，分为以下三部分：

>
### 1. 使用ReplicationController
>
### 2. Deployment — 更加方便的管理Pod和Replica Set 

### 1.使用ReplicationController（RC）

RC保证在同一时间能够运行指定数量的Pod副本，并且保证Pod总是可用。如果实际Pod数量比指定的多就结束掉多余的，如果实际数量比指定的少就启动缺少的。当Pod失败、被删除或被终结时，RC会自动创建新的Pod来保证副本数量，所以即使只有一个Pod，也应该使用RC来进行管理。

1) 我们先看以下这个nginx-rc.yaml文件

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-frontend
spec:
  replicas: 1
  selector:
    app: nginx-frontend
  template:
    metadata:
      name: nginx-frontend
      labels:
        app: nginx-frontend
    spec:
      containers:
      - name: nginx
        image: 192.168.81.210:5000/uet/nginx:centos7
        ports:
        - containerPort: 80
```

yaml字段的含义： 

```
spec.replicas：副本数量1 
spec.selector：RC通过spec.selector来筛选要控制的Pod 
spec.template：这里写Pod的定义（但不需要apiVersion和kind） 
spec.template.metadata.labels：Pod的label，可以看到这个label与spec.selector相同
```
这个文件的意思： 

```
* 定义一个RC对象，它的名字是nginx-frontend（metadata.name:frontend）。
* 保证有1个Pod运行（spec.replicas:1），Pod的镜像是192.168.81.210:5000/uet/nginx:centos7
```

关键在于spec.selector与spec.template.metadata.labels，这两个字段必须相同，否则下一步创建RC会失败。（也可以不写spec.selector，这样默认与spec.template.metadata.labels相同）


2) 通过kubectl创建rc

```
# kubectl create -f nginx-rc.yaml
```
3）查看rc信息

```
# kubectl describe rc frontend
kubectl describe rc nginx-frontend
Name:	nginx-frontend
Namespace:	default
Image(s):	192.168.81.210:5000/uet/nginx:centos7
Selector:	app=nginx-frontend
Labels:		app=nginx-frontend
Replicas:	1 current / 1 desired
Pods Status:	1 Running / 0 Waiting / 0 Succeeded / 0 Failed
No volumes.
Events:
FirstSeen LastSeen Count  From  SubObjectPath Type Reason Message
---------	-------- ----- ----	------------- -------- ------  -------
26s 26s 1 {replication-controller }  Normal SuccessfulCreate  Created pod: nginx-frontend-hfjlx
```

4) 通过rc修改Pod副本数量（需要修改yaml文件的spec.replicas字段到目标值为2，然后替换旧的yaml文件）

```
# kubectl replace -f nginx-rc.yaml
```
或

```
# kubect edit replicationcontroller nginx-frontend

# kubectl get pods
NAME                   READY     STATUS    RESTARTS   AGE
nginx-frontend-hfjlx   1/1       Running   0          5m
nginx-frontend-s1vhg   1/1       Running   0          15s
```


5）对rc使用滚动升级.

```
# kubectl rolling-update nginx-frontend --image=192.168.81.210:5000/uet/nginx:latest
```

当Pod中只有一个容器时，通过–image参数指定新的Tag完成滚动升级，但如果有多个容器或其他字段修改时，需要指定yaml文件

```
# kubectl rolling-update nginx-frontend -f update.yaml
```

6) 如果在升级过程中出现问题（如发现配置错误、长时间无响应），可以使用CTRL+C退出，再进行回滚

```
# kubectl rolling-update nginx-frontend  --rollback
```
但如果升级完成后出现问题（比如新版本程序出core），此命令就无能为力了。我们需要使用同样方法，利用原来的镜像，“升级”为旧版本。 


### 2. Deployment—更加方便的管理Pod和Replica Set

k8s是一个高速发展的项目，在新的版本中，官方推荐使用Replica Set和Deployment来代替rc。

rc只支持基于等式的selector（env=dev或environment!=qa），而Replica Set还支持基于集合的selector（version in (v1.0, v2.0)或env notin (dev, qa)），这对复杂的运维管理很方便。

使用Deployment升级Pod，只需要定义Pod的最终状态，k8s会为你执行必要的操作，虽然能够使用命令`# kubectl rolling-update`完成升级。

但它是在客户端与服务端多次交互控制RC完成的，所以REST API中并没有rolling-update的接口，这为定制自己的管理系统带来了一些麻烦。

Deployment拥有更加灵活强大的升级、回滚功能。

目前，Replica Set与rc的区别只是支持的selector不同，后续肯定会加入更多功能。Deployment使用了Replica Set，它是更高一层的概念。除非用户需要自定义升级功能或根本不需要升级Pod，在一般情况下，我们推荐使用Deployment而不直接使用Replica Set。

使用子命令create，创建Deployment

```
# cat nginx-deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx-dep
    spec:
      containers:
      - name: nginx-dep
        image: 192.168.81.210:5000/uet/nginx:latest
        ports:
        - containerPort: 80
          hostPort: 7777

# kubectl create -f nginx-deployment.yaml --record          
```

注意–record参数，使用此参数将记录后续创建对象的操作，方便管理与问题追溯

使用子命令edit，编辑spec.replicas/spec.template.spec.container.image字段，完成deployment的扩缩容与滚动升级（这要比子命令rolling-update速度快很多）

```
# kubectl edit deployment nginx-deployment
```

使用rollout history命令，查看Deployment的历史信息

```
# kubectl rollout history deployment nginx-deployment
deployments "nginx-deployment"
REVISION	CHANGE-CAUSE
1		kubectl create -f nginx-deployment.yaml --record
```

上面提到RC在rolling-update升级成功后不能直接回滚，而使用Deployment却可以回滚到上一版本，但要加上–revision参数，指定版本号

```
# kubectl rollout history deployment nginx-deployment --revision=2
```

使用rollout undo回滚到上一版本

```
# kubectl rollout undo deployment nginx-deployment 
```
使用–to-revision可以回滚到指定版本

```
# kubectl rollout undo deployment nginx-deployment --to-revision=2
```

参考资料：
>[http://blog.csdn.net/yjk13703623757/article/details/53746273](http://blog.csdn.net/yjk13703623757/article/details/53746273)
>
> [http://www.dockerinfo.net/1128.html](http://www.dockerinfo.net/1128.html)
> 
> [https://kubernetes.io/cn/docs/tutorials/object-management-kubectl/object-management/](https://kubernetes.io/cn/docs/tutorials/object-management-kubectl/object-management/)
> 
> gcr.io的国内仓库 [https://uk8s.com/](https://uk8s.com/)
> 


![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 