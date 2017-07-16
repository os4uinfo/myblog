<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-07-16
title: Golang 使用笔记（不断更新中）
tags: golang
images: https://os4u.info/blog/img/sun.png
category: golang
status: publish
summary: Go是Google开发的一种编译型，可并行化，并具有垃圾回收功能的编程语言。本篇文章主要记录工作学习中遇到的各种golang问题，以便后续查询使用，提高自己技能。
-->

### 1. Golang []uint8 转换为json

![golang](https://www.os4u.info/blog/programing/golang/images/golang.jpg)

[]unit8切片内容如下：

```
{"SortAs": "SGML","GlossTerm": "Markup Language", "Acronym": "SGML""}

```
#### 转换如下：
---
```
package main
import (
	"fmt"
	"encoding/json"
)

func main() {
	js := []byte(`{"SortAs": "SGML","GlossTerm": "Markup Language", "Acronym": "SGML"`)
	m := map[string]interface{}{}
	if err := json.Unmarshal(js, &m); err != nil {
	    panic(err)
	}
	fmt.Printf("%q",m)
}
```
---
### 2. 时间转换

```
package main

import (
	"fmt"
	"time"
)

func main() {

	//获取时间戳
	timestamp := time.Now().Unix()
	fmt.Println(timestamp)

	//格式化为字符串,tm为Time类型
	tm := time.Unix(timestamp, 0)
	fmt.Println(tm.Format("2006-01-02 15:04:05"))
	fmt.Println(tm.Format("02/01/2006 15:04:05 PM"))

	//从字符串转为时间戳，第一个参数是格式，第二个是要转换的时间字符串
	tm2, _ := time.Parse("01/02/2006", "02/08/2015")
	fmt.Println(tm2.Unix())

}
```
运行后结果为：

```
1497836281
2017-06-19 09:38:01
19/06/2017 09:38:01 AM
1423353600
```

### 3. Channel

```
package main

import "fmt"

func sum(values []int, resultChan chan int) {
	sum := 0
	for _, value := range values {
		sum += value
	}
	resultChan <- sum
}

func main() {
	values := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	resultChan := make(chan int, 3)
	go sum(values[len(values)/2:], resultChan)
	go sum(values[:len(values)/2], resultChan)
	fmt.Println(values[:len(values)/4])
	sum1, sum2 := <-resultChan, <-resultChan
	fmt.Println("result:", sum1, sum2, sum1+sum2)
}
-----------------------------VS----------------------------------------
package main

import "fmt"

func sum(values []int) int {
	sum := 0
	for _, value := range values {
		fmt.Println(value)
		sum += value
	}
	return sum
}

func main() {
	values := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	sum1 := sum(values[len(values)/2:])
	sum2 := sum(values[:len(values)/2])
	fmt.Println("Result: ", sum1, sum2, sum1+sum2)
}

```


![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 
