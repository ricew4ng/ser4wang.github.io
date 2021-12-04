---
title: "SQL注入-报错注入原理"
description: 读书时的碎碎念
date: 2018-03-26T21:14:23+08:00
tags:
    - Security
Categories:
    - security
---

先解释几个概念和语句

## 聚合函数

> 定义：SQL基本函数，聚合函数对一组值执行计算，并返回单个值。除了 COUNT 以外，聚合函数都会忽略空值。 聚合函数经常与 SELECT 语句的 GROUP BY 子句一起使用。

> 性质: 所有聚合函数都具有确定性。任何时候用一组给定的输入值调用它们时，都返回相同的值。

举例:

count() 返回指定组中项目的数量

> select count(id) from users; // 11

max() 返回指定数据的最大值

> select max(id) from users; //20

min()同理，其它还有像sum()求和，一般都是用于数字列。

## group by 语句

“Group By”从字面意义上理解就是根据“By”指定的规则对数据进行分组，所谓的分组就是将一个“数据集”划分成若干个“小区域”，然后针对若干个“小区域”进行数据处理。

实质上就是按照某种属性分类汇总

比如说要统计在全体中统计几个小类的各自情况，可以

> select 列名(能化成几个分类的)，某种属性比如sum(数量) as 数量之和 from table_name group by 列名

group by 列名就是按列名来分组。

## count(*)

> select count(*) from table_name;

返回table_name的行数；

也可以

> select count(*) from table_name where id > 10;

## #正文

我找来一句报错语句:



```
select count(),concat((select version()),floor(rand()2))a from test_table group by a;
```


解释



> concat：为聚合函数，连接字符串功能

> floor：取float的整数值

> rand:取0~1之间随机浮点值，长度测试大概为16-19的样子，官方手册也没说明 ：P

> group by：为聚合函数,根据一个或多个列对结果集进行分组并有排序功能

> a为as a 别名的简写方式

> floor(rand()*2) rand为0~1，rand（）*2为0~2，那么整个语句就是取0，1，2三个数字，rand（）为2的几率可以忽律不计，所以是 0，1两个随机数

ps: 测试语句中的test_table是我自建的一个表，行数为11，所以count(*)返回11；

知道了语句作用之后，测试语句，在sql中执行发现，有时会报错有时却输出正常。这是为什么?

不报错的话，结果是两行数据，如图：

![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/50036228.jpg)

![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/29323571.jpg)

就放两张，报错的话，如图：

![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/8582190.jpg)

就放一张，其它报错信息类似，除了最后随机数不同之外..

------

分析一下，发现test_table的行数是11，而count(*) 每次都是两行数据和是11。

报错信息说，Error: 重复录入 ‘10.1.21-MariaDB0’ for key ‘group_key’

也就是说 group by x 的这个x是作为新表的一个主键存在的，

我们可以把语句精简一下，来测试看看

> select count(*) from test_table group by floor(rand()*2);

情况跟上面有点不一样，报错情况剧增，先来看为什么会出现这个报错呢?

我们知道

> group by floor(rand()*2)

会随机产生两个值，（0，1），test_table中有11行数据，而select count(*)会产生两种总和为11的结果，结果集根据group by的值进行分组并且排序。

所以正常结果有4种情况，(0,0) (0,1) (1,0) (1,1)，只是group by把(0,1)和(1,0)排序了。

想不出来接下去怎么测试，参考前辈文章，把count(*)换成1继续测试，发现返回结果都是正常的。
也就是说可不可以有这样一个结论，count(*)是聚合函数，只要是聚合函数就会有这类报错

于是测试

> select max(test_column) from test_table group by floor(rand()*2);

又报错了，ok。测试其他函数像min(),sum(),avg()报错相同。

那么为什么呢?

…

我不知道.

前辈猜测：
（1）无其他聚合函数时，不会生成字典，只对指定字段进行分组
（2）有其他聚合函数时，根据指定字段生成字典，进行其他聚合函数计算，即使其他聚合函数没有使用该分组数据

------

那么最后，我来做个总结:

## 报错几率

这里我发现前辈结论是错的，floor(rand()*2)产生两个随机值，而对于group by floor(rand()*2)我的猜测是，比如目标列有11行，则它进行11次取值，以随机种子数为2举例，每次有(0,0) (0,1) (1,0) (1,1) 4种情况，取11次值，其中有2次值相同就会产生报错。
至少2次取到相同值的概率就是p=1-0.25 ^11次方，这个0.25的基数是跟随机种子数成正比，跟报错几率成反比。

也就是说, 只要随机种子数够小，就会一直产生报错。

好，于是以后我们就可以直接 rand()*0 这样就是百分百报错了。

------

第二天想了下，可能不严谨，感觉结论推出过程有问题…比如group by 随机数 是这样取11次值吗之类的问题。需要研究一下mysql的实现源码，看看group by具体是如何实现的。

测试了一下…结论肯定有问题…

------

3/28:

测试语句1：

> select count(*) from test_table group by floor(rand()*2);

测试语句2：

> select count(*) from test_table group by floor(rand(0)*2);

这里补充一些对rand()函数的解释，当rand()函数有参数时，结果是确定的

比如rand(0),rand(1),rand(2)，它们都会返回一个确定的浮点数，没有了随机性。

对于一个表，执行测试1和测试2，

表中只有一条数据时，2个测试都是没有报错

2条时:语句1有报错，但几率不高，语句2无报错

3条时:语句1有报错，几率变高，语句2有报错，近乎百分百报错

因为我认为floor(rand(0)*2)是一个确定的数字，所以我尝试替换成0或者floor(0)，结果是不再报错了，可以推测 报错注入中 count(*)和 group by floor(rand(0)*2)可能是必要的一个搭配。

而floor(rand()*2)和floor(rand(0)*2)之间的差别只是，前者具有不确定性(0~1间取值)，而后者具有确定性。

那么对于 select count(*) from test_table group by key;//key是表中主键的名字

我们猜测mysql遇到该语句时会建立一个虚拟表(实际上就是会建立虚拟表)

对于这句语句来说，虚拟表有两列，一列是key(主键)，另一类是count(*)

mysql执行此语句，开始查询数据，取数据库数据，然后查看虚拟表中key列下对应的值存在与否，不存在则插入新记录，存在则count(*)字段直接加1。

mysql官方有给过提示，就是查询的时候使用rand()的话，该值会被计算多次，什么意思呢，就是在使用group by的时候，其实就是建立虚拟表然后查询，floor(rand(0)*2)会被执行一次，如果虚拟表不存在记录，插入虚表的时候会再被执行一次，我们来瞰下floor(rand(0)*2)报错的过程就知道了，

首先，我们来看这句sql语句

> select floor(rand(0)*2) from test_table;（test_table有11条数据）

![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/82566196.jpg)

结果是这个的结果也有11条，怎么算的，之前说，查询的时候使用rand()，该值会被计算多次，这个语句可以理解成它执行了11次查询的结果为0 1 1 0 1 1 0 0 1 1 1

那么再来看

> select count(*) from test_table group by floor(rand(0)*2);

查询前，mysql建立了一个虚拟表(空的)，取第一条记录，执行floor(rand(0)*2)，此时floor(rand(0)*2)结果为0，发现值不存在，所以要将0插入虚表，此时先执行一次floor(rand(0)*2)，结果变为1，最后其实插入了1。

取第二条记录，执行floor(rand(0)*2)，此时floor(rand(0)*2)结果为1，值存在，于是count(*) +1 ， 取第三条记录，执行floor(rand(0)*2)，此时结果为0，查询记录，不存在，于是要将0插入虚表，但插入时又要先执行一次floor(rand(0)*2)，此时结果变为1，跟先前的1就发生了冲突，也就是主键必须唯一这个规则被违反了，就造成了dumplicate entry，重复输入主键。

floor(rand(0)*2) 具有确定性，它的值可以被推算出来，所以有三条记录时就必定能产生报错。(前提是 如果之前没有不小心查询建了个虚表，并且虚表中已经有了0和1，那么此时再执行就不会报错了)

而floor(rand()*2)具有随机性，它的序列是随机的，所以有2条记录时就有几率产生报错了，比如第一次插了个0，第二次算出来结果是个1，然后要插了，又计算一次正好结果是0，于是报错。

以上，就是原理。

总之，报错注入需要 count(*),rand(),group by ，三者缺一不可。

构造出来的语句一般是，

> select count(*) from target_table group by floor(rand(0)*2);

最后，感谢前辈白帽子们的分享和研究经验。

参考链接:

> https://blog.csdn.net/kuangmang/article/details/43675061

> https://mp.weixin.qq.com/s?__biz=MzA5NDY0OTQ0Mw==&mid=403404979&idx=1&sn=27d10b6da357d72304086311cefd573e&scene=1&srcid=04131X3lQlrDMYOCntCqWf6n#wechat_redirect