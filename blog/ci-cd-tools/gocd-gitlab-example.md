<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-06-03
title: CI,CD技术
tags: CI-CD-CD
images: https://os4u.info/blog/img/sun.png
category: CI-CD-CD
status: publish
summary: 持续集成，持续部署，持续发布，提高开发效率，降低运维成本。
-->

##### 利用gocd集成gitlab

场景：golang代码发布到gitlab，然后通过gocd编译。

注意：go-agent需要golang支持。

##### 1）登录gocd
![gocd-login](https://www.os4u.info/blog/ci-cd-tools/images/gocd-login.png)

##### 2)添加pipelines
![gocd-login](https://www.os4u.info/blog/ci-cd-tools/images/gocd-add-pipeline.png)

##### 3) 在默认组下创建pipelines
![gocd-login](https://www.os4u.info/blog/ci-cd-tools/images/gocd-add-pipelines-1.png)

##### 4）填写pipelines相关信息

```
a. 填写名称为helloworld
```
![gocd-login](https://www.os4u.info/blog/ci-cd-tools/images/gocd-add-pipelines-2.png)

```
b. 选择源
```

![gocd-login](https://www.os4u.info/blog/ci-cd-tools/images/gocd-add-pipelines-3.png)

```
c. 设置执行任务方式
```
![gocd-login](https://www.os4u.info/blog/ci-cd-tools/images/gocd-add-pipelines-4.png)

```
d. 配置material
```
![gocd-login](https://www.os4u.info/blog/ci-cd-tools/images/gocd-add-pipelines-5.png)

![gocd-login](https://www.os4u.info/blog/ci-cd-tools/images/gocd-add-pipelines-6.png)

```
e. 配置gopath
```
![gocd-login](https://www.os4u.info/blog/ci-cd-tools/images/gocd-add-pipelines-7.png)

```
f. 配置编译后文件存放路径
```

![gocd-login](https://www.os4u.info/blog/ci-cd-tools/images/gocd-add-pipelines-8.png)

##### 5）配置完成，触发任务

![gocd-login](https://www.os4u.info/blog/ci-cd-tools/images/gocd-add-pipelines-9.png)

##### 6) 查看任务执行情况
![gocd-login](https://www.os4u.info/blog/ci-cd-tools/images/gocd-add-pipelines-10.png)

##### 7）查看编译后文件存放
![gocd-login](https://www.os4u.info/blog/ci-cd-tools/images/gocd-add-pipelines-11.png)



![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 