---
title: "Python-threading库的使用"
description: 读书时的碎碎念
date: 2018-01-27T21:14:23+08:00
tags:
    - Python
Categories:
    - school_notes
---

之前看过的多线程相关的记不大清了，重新学习吧。

1. 线程的概念

   

   线程是CPU分配资源的基本单位。但一个程序开始运行，这个程序就变成了一个进程，而一个进程有至少一个线程。

   当没有多线程编程时，一个进程也是一个主线程，但有多线程编程时，一个进程包含多个线程，包括主线程。

   使用线程可以实现程序的并发。

2. 创建线程

   第一种，创建thread.Thread实例

   ```
    import threading
   
    def func_thread(x,y): #定义线程运行函数
        for i in range(x,y):
            print(i)
    #创建线程t1，用来跑func_thread(1,6)
    t1 = threading.Thread(target=func_thread,args=(1,6))
   
    #创建线程t2,用来跑func_thread(10,15)
    t2 = threading.Thread(target=func_thread,args=(10,15))
   
    t1.start() #调用start(),启动t1
    t2.start() #调用start(),启动t2
    
   ```

   刚接触，实际过程中，除了能传参，其它感觉不是特别方便.. 第二种，继承thread.Thread类 要点是是要重载threading.THread类的run方法，然后调用start()启动线程

   ```
    import threading
   
    class mythread(threading.Thread):
        def run(self):
            #这里写要运行的代码
            print(111)
   
    t1 = mythread() #创建mythread()实例t1
    t2 = mythread() #创建mythread()实例t2
    t1.start() #启动mythread
    t2.start() #启动mythread
    
   ```

3. threading.Thread类的一些相关属性

   join()方法：调用join()后，直到调用join的线程运行完了其他线程才能运行

   ```
    import threading
   
    class mythread(threading.Thread):
        def run(self):
            for i in range(30):
                print(i)
   
    t1 = mythread()
    t2 = mythread()
    t1.start()
    t2.start()
   
    t1.join() #调用join()方法，这时候只能运行t1线程
    
   ```

   isAlive()方法： 用于判断线程是否运行，未调用start或者已经执行并结束，返回False

   name属性：表示线程名，默认是Thread-x x为序号，从1开始，第一个创建的县城就是Thread-1

   daemon属性：用来设置线程是否随主线程退出而退出

   daemon = False,默认，线程不会随主线程退出而推出，

   daemon = True，主线程结束时，其他子线程就会被强制结束

4. 锁

   如果在一个进程里创建多个线程，如果各个线程之间有一定顺序关系，涉及到谁先谁后操作数据的时候，这时候就要用到锁这个概念了。

   python3中的threading模块有RLock()对象，实例化后就是一个锁对象。
   对于这个lock，它有acquire()方法和release()方法，acquire()方法相当于上锁，release()方法相当于解锁。

   在lock.acquire()和lock.release()语句之间一次只能有一个线程进入，其余线程只能在acquire()处等待。

   代码:

   ```
    import threading
   
    class mythread(threading.Thread):
        def run(self):
            global x #声明全局变量
            lock.acquire() #上锁
            x+=10
            print('%s:%d'%(self.name,x))
            lock.release() #解锁
   
    x = 0
    lock = threading.RLock() #创建一个锁对象
   
    thread_list = []
    for i in range(5):
        thread_list.append(mythread())
    for i in thread_list:
        i.start()
    
   ```

   一般lock.acquire()和lock.release()都放在thread对象的run方法里，毕竟，每个thread都是要运行run方法的。

5. 停止thread

   在继承threading.Thread类基础上，可以设置一个running标志，然后循环，为它编写要给terminate()方法很简单，running设置成False就ok了。