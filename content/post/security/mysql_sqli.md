---
title: "Mysql的注入姿势集"
description: 读书时的碎碎念
date: 2017-11-30T21:14:23+08:00
tags:
    - Security
Categories:
    - security
---



一直想找个时间好好整理下搞的这些东西，只能想起来的时候偶尔写一下。

起因是因为最近看到有个poc利用了报错注入, 而我却发现对它没多少记忆了….就把碰到的都记下来吧..

1. http头注入：

   应用场景：某些网站有某些功能，这些功能会收集你的http头信息，带入数据库。

   基本原理：php有个$_SERVER变量可以接收http头的所有信息，它是个数组，具体某个信息比如 User-Agent头，就是 变量 $_SERVER[‘HTTP_USER_AGENT’]。网页某些功能可能会将它带入数据库查询，比如

   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/83450459.jpg)

注入测试：利用抓包工具抓包修改User-Agent信息，在后面添加注入语句比如’and 1=1# 完成利用。

1. Cookie注入：

   应用场景：其实这跟http头注入非常类似，我认为它们的区别在于，cookie注入比http头注入更常见，因为网站经常会验证用户的cookie，来维持登录状态或其他操作。

   基本原理：Cookie也是在http头里的，但独立出$_SERVER数组，cookie中的数据在php中被$_COOKIE变量接收为一个数组。 调用方法 $_SERVER[‘cookiename’] 。

注入测试：在cookie中找到对应变量，其后添加注入语句。