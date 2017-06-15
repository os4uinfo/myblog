<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-05-25
title:rsync工具介绍
tags: Network
images: https://os4u.info/blog/img/sun.png
category: Network
status: publish
summary: 了解网络相关知识系列---rsync工具
-->

##### rsync命令

```
$ rsync file xxx@remotehost:/dir/

文件不一致便会同步，总会修改modified time为最新时间，且不关注目的端文件rwx权限，若目的端无文件，则权限与源服务器相同。
rsync只能以登陆目的端账号来创建文件，没有权限保持权限一致性。
```

##### rsync参数

```
-t 同步修改时间，出现modified time一致，目的端文件不会传输。
-I 确保数据一致性，速度变慢，挨个文件同步，目的端文件总会被更新到当前时间
quick check策略，先检查文件时间戳和文件大小，排除一批认为相同的文件。
-vvvv显示传输详细信息
-r 文件夹递归同步
-l 软连接的同步，但不会follow link，-L 会follow link
-P （preserve permission) 权限同步
-p 只会尽量保证目的端和源端权限一致
-g -o 保持稳健的属主和属组一致。 root可用
-a = -rlptgoD archive option 归档选项 但无法同步“硬连接”，使用-H解决
--delete 删除
  --delete 如果源端没有此文件，目的端也不会有（与-r搭配使用）
  --delete-excluded 指定需要在目的端删除的文件
  --delete-after 默认情况下，rsync先清理目的端文件再开始同步数据，此选项是先同步数据，完成后，再删除需清理的文件。
  -n搭配使用，不会实际删除，只是提示会删除的文件。
--exclude和--exclude-from
不希望同步一些文件到目的端，用--exclue隐藏。 --exclude-from从文件读取隐藏文件
--partial 断点续传，默认会删除传输中断的文件，然后重传。
--progress 显示传输进度。
```


![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 