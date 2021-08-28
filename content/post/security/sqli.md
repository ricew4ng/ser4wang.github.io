---
title: "初识SQL注入"
description: 上学时的碎碎念
date: 2017-09-19T21:14:23+08:00
tags:
    - Security
Categories:
    - Security
---

盲注，报错注入，有时间再写写。

sql注入的作用：

```
它可以获取信息（用户名密码），也可以写信息（shell）
```



sql注入根据数据库的不同也有区别，很流行的有两个组合，一是asp+access，二是php+mysql.

今天总结的是后者

## php+mysql

我们知道mysql是这样分的：

Mysql->选择数据库->选择表->选择列->存储信息

要理解sql注入，首先要知道后台脚本语言和基本sql语句的知识。

我们的目的就是获取最后一步的那个存储的信息，接下来我说一下一次比较完整的sql注入的过程：

它首先可以在url中进行也可以抓包 (使用burpsuite之类的抓包工具，或者fx-hackbar) 进行，根据提交方式(get,post)和注入手段(cookie)的不同也要不同对待。

比如目标网站有个文章功能，这个url是

```
http://www.target.com/article.php?p=1
```

问号后面就是传递的参数p，值为1（这里根据参数的不同也要注意注入的区别，下面介绍最简单的纯数字参数的注入）

对p=1做文章

1. 首先对目标url进行sql注入测试，像这样

```
http://www.target.com/article.php?p=1 and 1=1
```

和

```
http://www.target.com/article.php?p=1 and 1=2 (下面都省略成？后面的内容)
```

根据这两句话判断页面变化，如果第一句返回结果是正常，第二句返回结果是无（页面正常，应该有的内容没显示）那就说明有sql注入。

2. order by 语句判断当前表有多少列

```
p=1 order by 12 
```

如果显示错误，那就说明当前表的列数<12，依次递减，

如果order by 8显示正确，那就说明当前表的列数为8，然后才能进行下一步操作。

3. union select 1,2,3 联合查询语句

mysql中有几个函数对于注入很有用

```
database():显示当前使用的数据库名
user():显示当前的数据库用户名
version():显示当前的数据库版本
```

语句可以像

```
p=1 union select database(),user(),version()
```

这样就可以爆出数据库名，用户名和数据库版本的信息。

另外提一点，mysql5.0以上会自带一个叫做information_schema的数据库。 如果目标没有删除这个数据库的话，就可以根据此来进行下一步操作。

用database()获取了数据库名，我们离目标还隔着一层表、列，所以接下来先获取表

```
union select table_name,2,3 from information_schema.tables where table_schema='数据库名' 
```

这样就获取到了数据库下的所有表名

接下来根据表名获取它下面的列名

```
union select column_name,2,3 from information_schema.columns where table_name='表名'
```

这样我们就获取到了目标表下的所有列名信息

然后最后执行 union select 列名1，2，3 from 表名，就可以爆出数据了。

一些可能遇到的安全检查，

黑名单：

因为mysql是不区分大小写的，所以像select可以写成SeleCt

php magic quotes：

这是php的一个函数，类似于addslashes（）函数，给单双引号，null字符前加上反斜杠\，以此来消除危险的输入信息，对此可以采用宽字节注入

因为中文和乱码占两个字节，符号和英文占一个字节，所以在单引号前加上像 %df这样就可以变成 %df‘，会把反斜杠结合成一个乱码或者中文字符。

最后，结合burpsuite或者wireshark和sql注入工具来让自己的攻击上一层台阶。

sql注入工具对漏洞进行注入的时候用抓包工具抓下它的语句，分析，一次来更好的理解sql注入工具的攻击思路，也可以在工具注入失败的时候分析它语句的问题进行一些修改，来完善攻击。