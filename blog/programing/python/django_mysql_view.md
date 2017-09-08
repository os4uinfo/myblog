<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-09-08
title: Python使用笔记（不断更新中）
tags: python
images: https://os4u.info/blog/img/sun.png
category: python
status: publish
summary: 使用django进行网站开发，做个笔记。
-->

### django使用mysql视图


例子：

```
class MyViewModel(models.Model):
    field1 = models.IntegerField()
    [...]

    class Meta:
        managed = False
        db_table = 'MyView' # your view name
managed = False will cause python manage.py syncdb to ignore that model for sync'ing.
```


文章来源：[https://stackoverflow.com/questions/30579015/how-to-use-mysql-view-in-django](https://stackoverflow.com/questions/30579015/how-to-use-mysql-view-in-django)




![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 
