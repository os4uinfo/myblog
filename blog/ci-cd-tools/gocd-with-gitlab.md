<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-06-05
title: Gitlab 与GoCD集成
tags: CI-CD-CD 
images: https://os4u.info/blog/img/sun.png
category: CI-CD-CD 
status: publish
summary: gitlab与GoCD的集成。
-->

#### Gitlab 与GoCD集成

##### 1）下载插件

[gocd-oauth-login](https://github.com/gocd-contrib/gocd-oauth-login/releases)

此处下载gitlab的插件 [点击下载 gitlab-oauth-login-2.3.jar](https://github.com/gocd-contrib/gocd-oauth-login/releases/download/v2.3/gitlab-oauth-login-2.3.jar)

下载完成后，插件存放位置：/var/lib/go-server/plugins/external/gitlab-oauth-login-2.3.jar

重启go-server

##### 2) 配置gitlab，创建一个api keys
![create keys](https://www.os4u.info/blog/ci-cd-tools/images/gocd-gitlab-step-0.png)

```
【tips】用gitlab的管理员账号设置的api keys，全部gitlab成员可以以普通用户身份登录gocd。
```
##### 3）配置go-server

```
a. 设置管理员账号
   	在go-server的配置目录下，配置password.txt文件，主要是设置用户名密码，内容如下：
   	----------------------------------------------
   	testuser:{SHA}EpG8/BCCKKIVQl8GdjapOigbSrU=
   	----------------------------------------------
   	配置完成后，重启go-server，使用该账号密码登录。

b. 配置gitlab插件
```
![configure plugins](https://www.os4u.info/blog/ci-cd-tools/images/gocd-gitlab-step-1.png)

![configure plugins](https://www.os4u.info/blog/ci-cd-tools/images/gocd-gitlab-step-2.png)


```
c. 登录成功界面
```
![configure plugins](https://www.os4u.info/blog/ci-cd-tools/images/gocd-gitlab-step-3.png)


![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 