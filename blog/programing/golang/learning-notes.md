<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-11
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
### 2. 补充中...
![golang](https://www.os4u.info/blog/programing/golang/images/golang_blog.jpg)
