---
title: "ThinkPHP 漏洞总结"
description: 读书时的碎碎念
date: 2019-02-02T21:14:23+08:00
tags:
    - Security
Categories:
    - security
---

对已经公开的tp漏洞的分析总结

# 2019/1/11 thinkphp 5.0.0~5.0.23 RCE漏洞

payload:

> http://localhost:89/index.php?s=captcha

post:

> _method=__construct&filter[]=system&method=get&get[]=whoami

![img](https://i.imgur.com/X7igLUm.png)

在thinkphp中，访问index.php，会调用App.php中的run方法

![img](https://i.imgur.com/zePr6tz.png)

其中，会调用routeCheck方法检测路由信息，跟进

![img](https://i.imgur.com/x1ulDuX.png)

此处，会调用check方法检测路由，跟进

![img](https://i.imgur.com/UmmOw6V.png)

857行调用了request类的method方法，跟进

这处就比较关键了，我们发现，此method方法如果post了一个伪装变量 `_method` 就可以调用request类的任意方法，并且参数传递的是$_POST数组。

config设置中的默认伪装变量为_method

![img](https://i.imgur.com/W9ao6Il.png)

然后我们看看payload是如何构造并实现的。

# **PAYLOAD**

再看看payload，即post过去的数组

> _method=__construct&filter[]=system&method=get&get[]=whoami

通过上文可知，tp会调用 伪装变量`_method`的值即`__construct`方法，参数是$_POST数组

![img](https://i.imgur.com/kvoHXqr.png)

我们可以看到，在这个__construct方法其实只过了一件事情，就是在第一个foreach循环里将POST数组，赋值给了$this->$name这个可变变量。

接下来，我们一步步继续后，跑完了method，跑完了check，跑完了routeCheck，我们回到App.php的run方法。

此处dispatch值会是method，这里不是特别清楚。 看到说是因为vendor/topthink/think-captcha/src/helper.php中配置了路由。

![img](https://i.imgur.com/0fLqL8k.png)

继续往下看，执行了exec方法，跟进

![img](https://i.imgur.com/9SHEy3y.png)

这里因为dispatch是method，所以到case method处继续执行代码，跟进param()

![img](https://i.imgur.com/SDzt2E9.png)

重点关注$this->param变量，它是空值，但是在红框处，通过一个array_merge，将$this->get(false)，即$this->get的值赋给了$this->param变量。也即用户输入直接传递给了$this->param

再跟进末尾 return的input方法，我们继续跟进。

![img](https://i.imgur.com/zGhDKKD.png)

1028行的getFilter会将$this->filter，即我们之前的输入赋给filter变量，最后在1034行调用filterValue，跟进

![img](https://i.imgur.com/kdYraIA.png)

这里会遍历filter，如果它是一个可用方法，则会调用它，而它的确是的，这里$filter的值是 system，$value的值是 whoami，代码执行成功。

# 2018/12/9 thinkphp 5.x 全版本 任意代码执行漏洞

官方12.9 github上发布了修复代码，对controller类进行过滤

![img](https://i.imgur.com/m9CiMHl.png)

payload:

> http://localhost:89/index.php?s=index/\think\Container/invokefunction&function=call_user_func_array&vars[0]=phpinfo&vars[1][]=1

下面从App.php的run方法开始分析

![img](https://i.imgur.com/kr736rs.png)

我们跟进routeCheck

![img](https://i.imgur.com/WvN3sCM.png)

这里关键的是两个方法，我们先跟进第一个path

![img](https://i.imgur.com/VvdUXw6.png)

这里pathinfo方法会将默认的兼容模式值s赋给$pathinfo并返回。 即 path方法最终会获取到 GET提交的s参数值

之后跟进check方法

![img](https://i.imgur.com/gBOvsAu.png)

最关键的是return处，它最终会调用think\route\dispatch\Url.php的初始化方法，跟进

![img](https://i.imgur.com/jKqfkja.png)

关键代码有两行，跟进parseUrl方法

![img](https://i.imgur.com/lMVVfzS.png)

这里会对刚才获取的path进行处理，parseUrlPath方法，会根据正斜杠对path进行切割，然后返回切割的数组

然后tp会按顺序，pop刚才获取的path数组，依次得到$module，$controller，和$action的值

在parseUrl这个方法里，接下来还会获取其它提交的参数。

最后将module,controller,action封装并返回

从parseUrl出来后，我们继续看刚才的第二个关键方法，就是Module类的初始化init方法，跟进

![img](https://i.imgur.com/4tDPrA0.png)

这里没有对controller做什么过滤，返回后，

![img](https://i.imgur.com/fsGwemq.png)

这里通过中间件一路执行，一直到最关键的Module类的exec方法，

![img](https://i.imgur.com/LpuomOw.png)

跟进controller方法

![img](https://i.imgur.com/kPeRESZ.png)

跟进parseModule方法

![img](https://i.imgur.com/LkFlUvx.png)

因为存在反斜杠，所以直接返回类名，从而直接创建了任意可控实例 \think\container

接下来看payload是如何构造的。

# PAYLOAD

再复习一下payload

> http://localhost:89/index.php?s=index/\think\Container/invokefunction&function=call_user_func_array&vars[0]=phpinfo&vars[1][]=1

![img](https://i.imgur.com/onJWIZh.png)

就在exec方法中，获取到了我们可控的 \think\container实例以及GET传递的方法和参数后，调用了invokeReflectMethod方法，跟进

![img](https://i.imgur.com/ggdxLlP.png)

最终通过调用invokeArgs方法，任意代码执行成功。

![img](https://i.imgur.com/q6pyLQq.png)

成功调用 \think\container类的invokeFunction方法，

最后的代码调用相当于

```
call_user_func_array("call_user_func_array",["phpinfo",[1]]);
```

