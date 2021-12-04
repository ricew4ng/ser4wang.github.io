---
title: "PHP-FileInclusion"
description: 读书时的碎碎念
date: 2018-03-26T21:14:23+08:00
tags:
    - Security
    - PHP
Categories:
    - security
---

文件包含(File Inclusion)可能会出现在JSP、PHP、ASP等语言中，常见的导致文件包含的函数如下:

PHP:

> include(),include_once(),require(),require_once().fopen(),readfile()…

JSP/Servlet:

> ava.io.File(),java.io.FileReader()…

ASP:

> include file,include virtual…

------

以PHP为例，一般用上述函数来导入一个文件，比如一个全是函数的function.php。当使用上述函数包含一个新的文件时，该文件将作为PHP代码执行(重点!)，PHP内核不会在意被包含的文件类型是什么，所以就算包含了txt文件，图片文件，远程URL，也都将作为PHP代码执行。这一特性，在实施攻击时将非常有用。

比如以下代码:

```
<-?php include($_get['test']);="" ?-="">
```

执行[www.test.com/test.php?test=atk.jpg](http://www.test.com/test.php?test=atk.jpg)

若atk.jpg中含有PHP攻击代码，则会被执行。

可见，要想成功利用LFI漏洞，需满足条件:

1. include()等函数通过动态变量的方式引入需要包含的文件；
2. 用户能够控制该动态变量；

如果web应用没有正确防御，可能会被读出重要本地数据，或者直接被执行了一个webshell等等。

------

再深入看看文件包含漏洞的后果。

1.本地文件包含

能够打开并包含本地文件的漏洞，被称为本地文件包含漏洞(Local File Inclusion,简称LFI)

比如一下代码就存在LFI漏洞:

```
<-?php $file="$__GET['file'];" "..="" ..="" etc="" passwd\0"="" if(file_exists('="" home="" wwwrun="" '.$file.'.php')){="" include="" '="" '.$file.'.php';="" }="" ?-="">
```

此时如果用户控制$file变量为 ‘../../etc/passwd’,则会执行 include ‘/home/wwwrun/../../etc/passwd.php’。

但passwd.php文件并不存在,所以什么事都没有了吗?

…

不，因为PHP内核是用C实现的，因此使用了C语言中的一些字符串处理函数。比如在连接字符串时，0字节(\x00)将作为字符串结束符。所以在这个地方，攻击者只要在最后加入一个0字节，就能截断file变量之后的字符，即

> ../../etc/passwd\0

通过web输入时，只需要Url编码，变成

> ../../etc/passwd%00

字符串截断配合LFI口味更佳

所以web应用可以禁用0字节，因为用户其实是不需要使用它的。

但这样实际上仍没有解决问题，还有个技巧就是利用操作系统对目录最大长度的限制，可以不需要0字节而达到截断的目的。目录字符串，在windows下256字节，linux下4096字节会达到最大值，最大值后的字符会被丢弃，于是构造 n个 ‘./‘即可

比如:

> ../../etc/./././././ …… passwd.php
> 或者
> ../../etc/////////// …… /////passwd.php

------

除了include()等4个函数，php能对文件进行操作的函数都有可能出现漏洞。虽然大多数情况下不能执行php代码，但是能够读取敏感文件（比如源代码，比如账号密码文件）带来的后果不言而喻。

某些情况，目录遍历漏洞还可以用一些编码来绕过服务器端的逻辑

比如

%2e%2e%2f 等同于 ../

%2e%2e/ 等同于 ../

..%2f 等同于 ../

..%5c 等同于 ..\

之类

解决此类漏洞方法:

应该尽量避免包含动态的变量，禁用某些php导入外部url的函数，还有一种方式就是枚举用户变量(case…default)