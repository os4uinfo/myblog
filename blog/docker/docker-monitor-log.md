<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-10
title: Docker在ELK栈中整合日志
tags: Docker
images: https://os4u.info/blog/img/sun.png
category: Docker 
status: publish
summary: 监控提供了对Docker部署的一种反馈，回答了从底层操作系统性能到高层业务目标的一系列问题。在Docker宿主机中插入合适的工具对于确定系统工作状态很关键，我们可以使用这种反馈来确定应用是否正常工作。
-->

![images](https://os4u.info/blog/docker/images/docker-log.jpg)

### Docker在ELK栈中整合日志

```
ELK栈是一个弹性的，解决将文本转化为易懂的非结构化日志问题的包组合。主要包括组件如下：
L-Logstash： logstash用于收集管理日志和事件的模块，是收集不同日志源的中心节点，允许日志中包含丰富信息。
E-Elasticsearch: Elasticsearch 是一个高扩展性分布式搜索引擎，分片功能允许随着数据增长，扩展日志存储。
K-Kibana： Kibana是Elasticsearch的分析和搜索仪表盘。给关心的人们提供有价值的内部关系。
```

#### 配置ELK栈

##### 1.创建elasticsearch映像
```
$ docker run -d --name=elastic elasticseach:1.7.1
```

##### 2.运行映像跟上一步骤的elsticsearch连接起来
```
$ docker run -d --link elastic:elasticsearch -p 80:5601 kibana:4.1.1
```

##### 3.准备logstash dockerfile
```
FROM logstash:1.5.3
ADD logstash.conf /etc/logstash.conf
EXPOSE 1514/udp
```

##### 4.配置文件：
```
input {
    syslog {
        port => 1514 type => syslog
    }
}

output {
    elasticsearch {
        host => "elasticsearch"
    }
}
```

##### 5.生成hubuser/logstash映像
```
$ docker build -t hubuser/logstash .
```

##### 6.执行logstash。
```
上面暴露了1514端口给Docker宿主机作为syslog端口。还链接了之前生成的elastic容器。
目标名称为elasticsearch，
因为起主机名在之前配置的logstash.conf中配置为elasticsearch，并将接收日志数据。
$ docker run --link elastic:elasticsearch \
-d -p 1514:1514/udp hubuser/logstash \
-f /etc/logstash.conf
```
##### 7.配置Docker宿主机的syslog服务转发到logstash容器。
```
添加配置如下(/etc/rsyslog.d/100-logstash.conf)：
*.* @dockerhost:1514
```

##### 8.重启syslog服务（Docker宿主机上操作）
```
$ systemctl restart rsyslog.service
```

##### 9.测试
```
$ logger -t test 'message to elasticsearch'
```
##### 10.访问仪表盘 
```
http://dockerhost
```

##### 11.查看日志
```
http://dockerhost/#discover
```

#### 转发Docker容器日志

##### 1.docker引擎已经通过Debian Jessie主机systemd配置完毕，更新生成Systemd文件
```
/etc/systemd/system/docker.service.d/10-syslog.conf,代码如下：
[service]
ExecStart=
ExecStart=/usr/bin/docker daemon -H fd:// --log-drive=syslog
```

##### 2.重启systemd服务
```
$ systemctl daemon-reload
```

##### 3.重启docker引擎
```
$ systemctl restart docker.service
```
4。可选操作，如果像定制化Docker容器中的日志注释，可以应用任何logstash过滤器。

##### 5.测试
```
docker run --rm busybox echo message to elk

```
其他日志监控方案
```
cAdvisor
InfluxDB
Sensu
Fluentd
Graylog
Splunk

代理
New Relic
Datadog
Librato
Elastic's Found
Treasure Data
Splunk Cloud
```


![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 