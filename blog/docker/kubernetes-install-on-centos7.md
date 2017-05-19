<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-19
title: Centos 7 上安装Kubernetes集群（转载）
tags: Docker
images: https://os4u.info/blog/img/sun.png
category: Docker
status: publish
summary: Docker容器技术是当下比较热门的技术，业务应用逐渐转向容器化，掌握容器集群管理也是运维技能必要条件。从Kubernetes开始，走进一个容器的世界吧。
-->




![kubernetes](https://severalnines.com/sites/default/files/kube7-logo.png)

##### Kubernets 组件
Kubernetes工作模式是C/S模式。即一组master提供集中控制一组minions的架构。本文中我们将使用1台master和3台minions机器。
架构如下图：


![1-master-3-minions](https://severalnines.com/sites/default/files/kube7-arch.png)

```
etcd - 一个共享配置和服务器发现的高可用键值对存储服务。
flannel - 为容器提供以etcd为后端的网络服务。
kube-apiserver - 为Kubernetes业务流程提供API。
kube-controller-manager - 管理kubernetes服务。
kube-scheduler - 主机上容器调度服务。
kubelet - ，使得容器根据描述启动运行容器。
kube-proxy - 提供网络代理服务。
```

##### 部署需求

```
1.4台服务器
2.运行CentOS 7.1 64位
```

##### 开始部署

1.禁用每台机器上的iptables避免与Docker上iptables规则冲突

```
$ systemctl stop firewalld
$ systemctl disable firewalld
```

2. 安装NTP服务器

```
$ yum -y install ntp
$ systemctl start ntpd
$ systemctl enable ntpd
```

##### 设置master

在master服务器上设置

1. 通过yum命令安装etcd以及kubernetes服务。

```
$ yum -y install etcd kubernetes
```

2. 配置/etc/etcd/etcd.conf

```
ETCD_NAME=default
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://localhost:2379"
```

3.配置/etc/kubernetes/apiserver

```
KUBE_API_ADDRESS="--address=0.0.0.0"
KUBE_API_PORT="--port=8080"
KUBELET_PORT="--kubelet_port=10250"
KUBE_ETCD_SERVERS="--etcd_servers=http://127.0.0.1:2379"
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
KUBE_ADMISSION_CONTROL="--admission_control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"
KUBE_API_ARGS=""
```

4. 开启etcd, kube-apiserver, kube-controller-manager 以及 kube-scheduler

```
$ for SERVICES in etcd kube-apiserver kube-controller-manager kube-scheduler; do
    systemctl restart $SERVICES
    systemctl enable $SERVICES
    systemctl status $SERVICES 
done
```

5. 设置flannel网络配置，该配置将被minions拉取

```
$ etcdctl mk /atomic.io/network/config '{"Network":"172.17.0.0/16"}'
```

6. 由于还未安装minions，所以获取不到nodes信息

```
$ kubectl get nodes
NAME             LABELS              STATUS
Setting up Kubernetes Minions (Nodes)
```


##### 在minions上操作

1. 用yum命令安装flannel和Kubernetes

```
$ yum -y install flannel kubernetes
```

2. 配置/etc/sysconfig/flanneld使得能连接到master

```
FLANNEL_ETCD="http://192.168.50.130:2379"
```

3. 配置Kubernetes/etc/kubernetes/config，确保KUBE_MASTER配置能连接到kubernetes api服务器

```
KUBE_MASTER="--master=http://192.168.50.130:8080"
```

4. 配置/etc/kubernetes/kubelet

minion1:

```
KUBELET_ADDRESS="--address=0.0.0.0"
KUBELET_PORT="--port=10250"

### change the hostname to this host’s IP address
KUBELET_HOSTNAME="--hostname_override=192.168.50.131"
KUBELET_API_SERVER="--api_servers=http://192.168.50.130:8080"
KUBELET_ARGS=""
```

minion2:
```
KUBELET_ADDRESS="--address=0.0.0.0"
KUBELET_PORT="--port=10250"
# change the hostname to this host’s IP address
KUBELET_HOSTNAME="--hostname_override=192.168.50.132"
KUBELET_API_SERVER="--api_servers=http://192.168.50.130:8080"
KUBELET_ARGS=""
```

minion3:

```
KUBELET_ADDRESS="--address=0.0.0.0"
KUBELET_PORT="--port=10250"
# change the hostname to this host’s IP address
KUBELET_HOSTNAME="--hostname_override=192.168.50.133"
KUBELET_API_SERVER="--api_servers=http://192.168.50.130:8080"
KUBELET_ARGS=""
```


5. 开启kube-proxy, kubelet, docker 和 flanneld services:

```
$ for SERVICES in kube-proxy kubelet docker flanneld; do
    systemctl restart $SERVICES
    systemctl enable $SERVICES
    systemctl status $SERVICES 
done
```

6. 在minion服务器上可以看到有docker0 和 flannel0接口， 可以获取到不同的IP。


minion1:

```
$ ip a | grep flannel | grep inet
inet 172.17.45.0/16 scope global flannel0
```

minion2:

```
$ ip a | grep flannel | grep inet
inet 172.17.38.0/16 scope global flannel0
```

minion3:

```
$ ip a | grep flannel | grep inet
inet 172.17.93.0/16 scope global flannel0
```

7. 登录到master验证minions状态

```
$ kubectl get nodes
NAME             LABELS                                  STATUS
192.168.50.131   kubernetes.io/hostname=192.168.50.131   Ready
192.168.50.132   kubernetes.io/hostname=192.168.50.132   Ready
192.168.50.133   kubernetes.io/hostname=192.168.50.133   Ready
```

至此，kubernetes 的集群环境搭建完成。

原文地址：[点击访问](https://severalnines.com/blog/installing-kubernetes-cluster-minions-centos7-manage-pods-services)
