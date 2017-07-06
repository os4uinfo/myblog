<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-06-24
title: Chef使用之配置修改
tags: Chef
images: https://os4u.info/blog/img/sun.png
category: Chef
status: publish
summary: Chef Server配置修改，方便公司内部使用。
-->


1.修改发件人邮箱

默认邮箱发件人地址为：notifications@chef.com

修改配置文件路径为：

```
/opt/chef-manage/embedded/cookbooks/omnibus-chef-manage/libraries/config.rb
```

2.发件内容修改

具体修改文件为：

```
/opt/chef-manage/embedded/service/chef-manage/app/views/password_reset_mailer

```

3.修改完成后需重新配置才能生效

chef-manage-ctl reconfigure
chef-server-ctl reconfigure

找回密码邮件中可以看到发送邮件的内容已有部分改变（发件人，以及邮件内容）

![chef-sendmail-message-change](https://www.os4u.info/blog/chef/images/chef-sendmail-message-change.png)

4.其他配置修改，可以在上述文件中修改


***参考文件***
> [https://d2avhevdwylo4t.cloudfront.net/config_rb_manage.html](https://d2avhevdwylo4t.cloudfront.net/config_rb_manage.html)


![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 