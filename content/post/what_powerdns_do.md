---
title: 【WIP】开源DNS服务器源码解析
description: DNS是网络的基础服务，通过阅读开源的DNS Server源码，可以更好地帮助我们了解其工作机制及原理。本篇的撰写源于一次事故，下面就听在下娓娓道来。
date: 2019-09-18 20:32:56
slug: what_powerdns_do
tags: 
    - DNS
    - PowerDNS
categories: 
    - DNS
---

我们选择一个开源的DNS服务器，这里笔者选择的是 [PowerDNS](https://github.com/PowerDNS/pdns)，也是很多组织或企业搭建DNS服务的一个常见选择。



## 编译安装

暂略

分支：rec-4.0.x



## 递归解析 源码

分析的几点，

1. 开启Lua与否的区别

2. 

入口看 pdns_recursor.cc 文件，main() 函数主要读取各种配置以及各种初始化。

### startDoResolve 函数

#### 1. Line 690-760 

![image-20200110211550470](/Users/wangwenqi/Library/Application Support/typora-user-images/image-20200110211550470.png)

主要初始化一些变量，介绍一部分。

edns是rfc里用来储存DNS额外信息（客户端IP）。默认不开。

pw是 DNSPacketWriter，顾名思义，用来写返回包的。

740 - 746行 设置了一些DNS包的标志位。

下面初始化了一个SyncRes类（核心），初始化了Lua引擎

756行 因为DNSSEC的默认值是 process-no-validate，所以默认会进行DNSSEC行为。



#### 2. Line 779-821

![](https://ser4wang.oss-cn-beijing.aliyuncs.com/20200110212045.png)

781行 shouldNotValidate变量设置默认值false



#### 3. Line 822-973

到了一个if判断。这是关键位置。

![](https://ser4wang.oss-cn-beijing.aliyuncs.com/20200110212817.png)

if的条件是 没开Lua hook 或者 preresolve 这个Lua hook 直接return false，则执行。

先进行了一次 wantsRPZ (默认true)，根据不同policy进行相应处理。

下面的try catch会进入一个 beginResolve方法，即进行**递归解析，先不跟进**。

874行 判断res，是否等于 -2。代码会switch policy的值进行不同操作。

918行 如果res == RCode::NoError ，遍历结果，按情况会执行一个 nodata hook.

935行 又执行了一次wantsRPZ



#### 4. Line 974-1105

![](https://ser4wang.oss-cn-beijing.aliyuncs.com/20200112164805.png)

975 判断res值是否等于DROP，是则直接return。

981 如果错误会打Log

993 判断res值是否等于 -1，是则返回包设置为Servfail

若res不等于 -1，写返回的rcode（设置为res值）

1002 若需要validate（执行lua hook后则不需要，这是一个区分点），则会进行使用validateRecords(ret)验证，根据state值执行不同操作。

1057 结果不为空，会调用 orderAndShuffle(ret)









DNSComboWriter类似一个生产者，获取请求的 域名 d_qname 和 请求类型 d_qtype，下面是处理EDNS。



再看下面

![](https://ser4wang.oss-cn-beijing.aliyuncs.com/20191217204919.png)

738行，初始化了一个DNSPacketWriter，用来写返回的packet。

750行，初始化了解析器。用来做递归解析的。

752行， 设置了Lua引擎指针 t_pdl，pdns通过Lua的形式提供了各种hook。



再往下看，784行初始化了res，即返回码，服务端和客户端都会根据这个判断请求状态。如 Rcode为 SERVFAIL 时，很多开源的dns库就会报错。

![](https://ser4wang.oss-cn-beijing.aliyuncs.com/20191218103654.png)



813行，会调用lua的一个hook，prerpz()方法。

往下看

![](https://ser4wang.oss-cn-beijing.aliyuncs.com/20191218105837.png)

823行，会调用Lua Hook，preresolve方法，在每次递归解析前会执行这个Hook。

看if条件，如果没有启用 Lua 或者 preresolve 方法返回了false，会进行默认解析。



825 - 860 行，看默认解析，是 RPZ 和 policy 的判断，这块暂略。不同policy会直接影响解析结果。

![](https://ser4wang.oss-cn-beijing.aliyuncs.com/20191218110645.png)



try的块中，可以看到默认解析会调用 syncres.cc文件中的SyncRes类的beginResolve方法，shouldNotValidate的值是用来判断是否OOB的（OOB暂略）。中间报错会直接返回SERVFAIL。



暂时不跟进，主要看Lua，往下看



873行，判断返回码res是否等于-2（-2表示命中了过滤引擎的策略）

> syncres.cc

![](https://ser4wang.oss-cn-beijing.aliyuncs.com/20191218111229.png)

同之前一样，下面就会根据不同policy进行处理。



往下看，默认解析最后，如果没有报错的话，会执行Lua Hook，nodata方法

![](https://ser4wang.oss-cn-beijing.aliyuncs.com/20191218111629.png)



第二个if 执行的是 nxdomain hook。（NXDOMAIN 暂略）



最后再调用 Lua Hook，postresolve方法。（即解析完后执行）



往下看，如果返回码res 不等于-1，都会执行

![](https://ser4wang.oss-cn-beijing.aliyuncs.com/20191218113239.png)

如果shouldNotValidate为true，即比如postresolve 中返回了false才会执行这段code。



往下看，

![](https://ser4wang.oss-cn-beijing.aliyuncs.com/20191218113958.png)

会调用validateRecords方法（判断CSPF，暂略）判断解析记录的状态。

然后根据不同状态设置ad码。



下面是写到返回的packet中。















