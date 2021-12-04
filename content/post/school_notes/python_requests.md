---
title: "Python-requests库的使用"
description: 读书时的碎碎念
date: 2018-01-29T21:14:23+08:00
tags:
    - Python
Categories:
    - school_notes
---

requests库的官方中文文档半夜让我笑出声，可爱的编写人员!

感谢kennethreitz大神的requests库，足够方便，足够有用，足够pythonic!

官方中文文档链接:http://cn.python-requests.org/zh_CN/latest/

official doc肯定比我的详细和细致，而我从一个requests库初学者的角度来写，当然，不管哪种方式，都欢迎你接触到了requests库!

ps：有一些高级用法算是requests库的进阶，可以自行阅览。

------

> import requests #导入requests库

1. 发送请求

   常用的GET方式和POST方式举例:

   ```
    response = requests.get(url=’https://www.baidu.com/')
    response = requests.post(url=’https://www.baidu.com/')
    
   ```

   其它http请求类型(put,delete,head,options)类似.

   

2. 传递url参数

   get方式 请求x.com?key1=value1&key2=value2

   ```
    params = {'key1':'value1','key2':'value2'}
    response = requests.get(url='https://www.baidu.com/',params=params)
    
   ```

3. text属性(界面内容)

   requests相当方便的一点，它可以自动解码来自服务器的内容，大多数时候都不需要自己再因为目标url的编码问题而烦恼了!

   ```
    response = requests.get(url='https://www.baidu.com/')
    print(response.text) #打印解码后的界面内容
    
   ```

   response.encoding 返回解码界面内容的相应的编码(比如ISO-8859-1)，可以改变值，再次打印时也会变成改变了的解码方式。

   注:
   然而我在实际爬取过程中，偶尔也会遇到明明对应页面是charset=’utf8’，结果解码时变成’ISO-8859-1’，所以根据实际情况还是要有所变化，可以自己写一个识别编码头的函数一劳永逸…

   response.content 不解码直接得到界面内容的二进制相应数据,返回值类似于：b’界面内容’.

4. 定制http请求头

   ```
    url = 'http://www.baidu.com/'
    headers = {'User-Agent':'xxx','Cookie':'xxxx'}
    response = requests.get(url=url,headers=headers)
    
   ```

5. post请求传递表单数据

   ```
    data = {'key1':'value1','key2':'value2'}
    response = requests.post(url=url,data=data)
    
   ```

6. 保持会话

   主要涉及requests.Session()对象,直接看例子

   ```
    test_session = requests.Session() #实例化requests.Session对象
   ```

   

   url1 = ‘http://x.com/page1.php'
   url2 = ‘http://x.com/page2.php'

   test_session.get(url=url1) #获取page1的会话

   \#这时再用获取过session信息的Session对象,test_session的get方法打开新url时，就能保持page1上的会话。
   res = test_session.get(url=url2) #保持了会话

   \#而如果仍然用requests的get方法打开新url，那相当于无会话打开一个新的url，并不能维持会话。
   res = requests.get(url=url2) #新会话

   cookie也一样通过这个方法来保持

7. 设置代理 proxy

   ```
    proxies = {
        'http':'http://x.x.x.x:xx'
        'https':'https://xx.xx.xx.xx:xxx'
    }
    res = requests.get('http://target.com/',proxies=proxies)
    
   ```

1. response内容的其它有用属性

   status_code 返回响应状态码(int) 如:200

   url 返回当前页面的url

   headers 返回响应头(dict) 如: {‘User-Agent’:’xx’,’Cookie’:’xxx’}

   headers有两种方式直接得到其字典值
   eg:

   ```
    response.headers.get(‘Content-Type’) # ‘application/json’
    response.headers[‘Content-Type’] #同上
    
   ```

   

   request.headers 返回请求头(dict)
   eg:

   ```
   response.request.headers
   ```

