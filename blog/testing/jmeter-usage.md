<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-17
title: Jmeter压测工具使用简介
tags: Testing
images: https://os4u.info/blog/img/sun.png
category: Testing
status: publish
summary: 测试，是运维上线前的一道工序。压测，是对服务器进行有效评估的重要一步。因此，测试和压测两种工具运维是需要熟悉和掌握的。
-->


1.安装JMeter
```
# apt-get install jmeter
```

2.运行JMeter

```
$ jmeter
```

运行效果如下图：
![jmeter-screenshot](https://www.os4u.info/blog/testing/images/jmeter-screenshot.png)

3.制定测试计划

```
1) 重命名"test plan" 为 "Unicorn Capacity"

2) 增加进程
步骤如下：Unicorn Capacity--> Add --> Threads(Users) --> Thread Group
重命名 "Thread Group" 为 "Application Users" 
```
![thread-group-parameters](https://www.os4u.info/blog/testing/images/thread-group-parameters.png)

```
3)创建http访问请求：
Application Users --> Add --> Sampler --> HTTP Request
```
![thread-group-parameters-1](https://www.os4u.info/blog/testing/images/thread-group-parameters-1.png)

```
4)收集访问过程中的数据
Unicorn Capacity--> Add --> Listener --> Simple Data Writer
保存到 resutl.jtl
```
![save-test-result](https://www.os4u.info/blog/testing/images/save-test-result.png)

```
5) 执行测试
```

4.结果分析

```
创建新项目，并命名为 "Analyze Results"
```
![analyze-results](https://www.os4u.info/blog/testing/images/analyze-results.png)

```
1) 查看带宽情况
Analyze Results --> Add --> Listener --> Summary Report
加载上面的测试结果 'result.jtl'
```
![analyze-results-bandwith](https://www.os4u.info/blog/testing/images/analyze-results-bandwith.png)

```
2) 吞吐量
Analyze Results --> Add --> Listener --> Graph Results
重命名 "Graph Results" 为 "Throughput over time"（也可以不重命名）
加载测试结果文件'result.jtl'
```
![analyze-resutls-throughput](https://www.os4u.info/blog/testing/images/analyze-resutls-throughput.png)

```
3) 响应时间
Analyze Results --> Add --> Listener --> Response Time Graph
加载测试结果文件 'result.jtl'
```
![analyze-resutls-response-time-graph-1](https://www.os4u.info/blog/testing/images/analyze-resutls-response-time-graph-1.png)

![analyze-resutls-response-time-graph-2](https://www.os4u.info/blog/testing/images/analyze-resutls-response-time-graph-2.png)

3. 利用工具观察性能

```
Graphite 或者 Kibana
```

4. 更多压测

通过压测，分析，找出性能瓶颈，对相应的部分进行优化。

1)增加并发

```
需要调整的参数是loop count，增加更多并发请求使我们看出应用对负载相应情况。这样会增加性能基准的精度。
为了增加JMeter的压力，需要增加thread group中的线程数，如果想模拟逐渐增加用户量，可以设置ramp-up时段变长
想瞬间产生大量用户访问，可以设置ramp-up时间为0
如果想调整稳定负载后，突然出现一个峰值，可以使用Stepping Thread Group plugin插件

```


2) 分布式测试

```
在分布式docker容器中运行JMeter

a) 创建Dockerfile文件
-------------------------
FROM java:8u66-jre

# Download URL for JMeter
RUN culr http://www.apache.org/dist/jmeter/binaries/apache-jmeter-2.13.tgz | tar xz 
WORKDIR /apache-jmeter-2.13

EXPOSE 1099
EXPOSE 1100 

ENTRYPOINT ["./bin/jmeter", "-j", "/dev/stdou", "-s", \
"-Dserver_port=1099", "-Jserver.rmi.localport=1100"]
-------------------------

b) 部署更多的docker

c)运行docker
$ docker run -p 1099:1099 -p 1100:1100 \
hubuser/jmeter -Djava.rmi.server.hostname=dockerhost1

$ docker run -p 1099:1099 -p 1100:1100 \
hubuser/jmeter -Djava.rmi.server.hostname=dockerhost2

d)运行JMeter客户端
$ jmeter -Jremote_hosts=dockerhost1, dockerhost2
```

更多关于分布式测试参考：[https://jmeter.apache.org/usermanual/remote-test.html](https://jmeter.apache.org/usermanual/remote-test.html)