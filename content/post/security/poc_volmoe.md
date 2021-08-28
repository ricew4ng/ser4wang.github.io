---
title: "利用越权漏洞下载vol.moe的所有漫画"
description: 上学时的碎碎念
date: 2018-08-05T21:14:23+08:00
tags:
    - Security
Categories:
    - Security
---

前几天想看一部漫画找到一个不错的站点，顺便就发现了一个越权漏洞，普通用户可以直接具有高级用户的下载权限。

为了隐私，很多细节都不能放，因为暂时未允许公开，所以就先看一下效果吧。

代码:

![img](http://p6jpvwsnk.bkt.clouddn.com/18-8-5/9313906.jpg)

执行代码

![img](http://p6jpvwsnk.bkt.clouddn.com/18-8-5/84685411.jpg)

获取到比如id为10000010的用户的cookie，我们放到bp里

![img](http://p6jpvwsnk.bkt.clouddn.com/18-8-5/51303251.jpg)

提交然后发现权限获取成功可以下任意漫画了~

![img](http://p6jpvwsnk.bkt.clouddn.com/18-8-5/16401895.jpg)

总结:

这漏洞原理很简单，就是cookie明文未加密，解决方案就是cookie+salt再用个md5，sha1加密一波，加密过程最好后端执行。