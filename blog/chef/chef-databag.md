<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-21
title: Chef 数据包使用
tags: Chef
images: https://os4u.info/blog/img/sun.png
category: Chef
status: publish
summary: 了解下Chef data bag。
-->

##### 使用knife在命令行进行数据包的基本操作

工作目录为：/data/work-chef/chef-repo

```
$ mkdir -p /data/work-chef/chef-repo/data_bags/users
$ cd /data/work-chef/chef-repo/data_bags/users
$ vim alice.json
//内容如下：
{
  "id": "alice",
  "comment": "Alice Jones",
  "uid": 2000,
  "gid": 0,
  "home": "/home/alice",
  "shell": "/bin/bash"
}

$ knife data_bag create users
$ knife data_bag from file users alice.json
$ knife search user "*:*"
$ knife search user "id:alice"

$ knife search node "*:*"
$ knife search node "*:*" -a ipaddress
```

##### 加密数据包

```

$ mkdir -p /data/work-chef/chef-repo/data_bags/api_keys
$ cd  /data/work-chef/chef-repo/data_bags/api_keys
$ vim payment.json
//内容如下：
{
  "id": "payment",
  "api_key": "8c728347-b326-4d41-b7b5-f8e748e65db0"
}
$ openssl rand -base64 512 | tr -d '\r\n' > encrypted_data_bag_secret
$ knife data bag create api_keys
$ knife data bag from file api_keys payment.json --secret-file encrypted_data_bag_secret
// 查看api
$ knife data bag show api_keys payment
WARNING: Encrypted data bag detected, but no secret provided for decoding. Displaying encrypted data.
api_key:
  cipher:         aes-256-cbc
  encrypted_data: iD4FvtLb5ud0lebKLyAAKR/ZU+Ym+MBMIz3NAvMbM7yPAlIybcUsd9+QkqLx
  cN4gCURplwre008ysE3beghSWg==

  iv:             DG/EgLhFyac/24BUQEp3HQ==

  version:        1
id:      payment

$ knife data bag show api_keys payment --secret-file encrypted_data_bag_secret

```


*** 

![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 