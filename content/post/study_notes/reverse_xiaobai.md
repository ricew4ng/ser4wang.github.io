---
title: "入门逆向"
description: 读书时的碎碎念
date: 2018-11-19T21:14:23+08:00
tags:
    - Security
Categories:
    - study_notes
---

我想成为魔法师！ 这个逆向专题用来记录自己学习逆向魔法的过程。



## 关于魔法

有人问教主: “ 现在都是win10了，win7连debug都没有，学习汇编有什么用，对于手机app开发有什么帮助？ “

教主:

> “ 学习魔法辛苦又困难， 而且似乎对当个铁匠、农夫、牧羊人也毫无帮助。 “ ———— 2016-01-18

作为入门篇，这次逆向的是一个小pe，目标是改变它的执行逻辑，从而达到我们想要的效果（比如破解）。

## 用C写一个小程序

crack.c 代码如下:

```
#include <stdio.h>

#define PASSWORD "1234567"

int verify_password(char *password){
    int authenticated;
    authenticated = strcmp(password,PASSWORD);
    return authenticated;
}

main(){
    int valid_flag = 0;
    char password[1024];
    while(1){
        printf("please input password:    ");
        scanf("%s",password);
        valid_flag = verify_password(password);
        if(valid_flag){
            printf("incorrect password!\n\n");
        }else{
            printf("Congradulation! You have passed the vrification\n");
            break;
        }
    }
}
```

编译获得 crack.exe ，也就是今天要破解的目标了。

## IDA PRO静态分析

打开ida pro，把这个exe拖进去，就获得了一张清晰的执行逻辑结构图

![img](https://77sera.oss-cn-beijing.aliyuncs.com/18-11-20/62836004.jpg)

看到这句汇编指令，执行的是跳转，也就是if语句，有两个分支，一个会break，一个则会返回while判断。

![img](https://77sera.oss-cn-beijing.aliyuncs.com/18-11-20/80649460.jpg)

按space可以看到对应的汇编语句，

![img](https://77sera.oss-cn-beijing.aliyuncs.com/18-11-20/76865807.jpg)

记住红框中的这两个地址，上面的是汇编指令在内存中的虚拟地址VA，下面那个是它在文件中的偏移地址。 关于如何换算VA和FOA，可以参考下面文章

https://blog.csdn.net/baishuiniyaonulia/article/details/78510997

## hex编辑PE文件

这里我用的notepad的插件hex editor，我们找到刚才那句if语句的偏移地址处，它的逻辑是，如果valid_password是true就继续循环，那么我们可以考虑把这句if的跳转语句修改一下，也就是把je改成比如 jne，je是相等跳转，jne是不相等则跳转。也就是将对应je的(十六进制 74)改成对应jne的（十六进制 75）

![img](https://77sera.oss-cn-beijing.aliyuncs.com/18-11-20/9798884.jpg)

上图，我是已经将74修改为了75，这个偏移地址的计算方式可以自行百度，而在IDA pro中已经给出了这个偏移地址，也省去了自己计算的麻烦

（刚开始我没有发现，自己计算的时候是004015AF-00400000-BAF）逆推的，发现节偏移是600，不知道原因是什么，我看别的文章都是400…

## 查看结果

我们运行crack.exe，发现随便输一个密码就直接跳出循环了，而原来表示正确的密码 1234567，输入后却提示错误了。

至此，破解完毕。

## xdbg 动态分析

打开xdbg，把crack.exe拖进去…原理同上，f7/f8 跟着查看汇编，对应着修改关键的跳转指令，或者test指令，就可以做到修改pe的执行逻辑了。

------

本篇介绍至此，时间匆忙，还请看官们多多谅解。