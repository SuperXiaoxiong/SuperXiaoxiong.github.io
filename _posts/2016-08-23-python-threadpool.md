---
layout: post
title:  "python 线程池原理和使用"
date:   2016-08-25 12:13:05
categories: python
tags:  python  pool
---

* content
{:toc}

## 简述

在使用threading模块中遇到了一些问题，可以采用一些高级API来轻松的解决这些问题





## 线程池的原理和使用

问题的提出：在使用多线程的时候面对着一个问题，如果使用断点下载，把每个range设置为指定值，文件大或者range小都会面对一个任务数爆炸的状态。使用一个线程来完成一个任务会造成线程数过多，线程数过多不仅影响效率，也会影响程序的健壮性

### 线程池的原理

1. 线程池首先会维护一个任务队列
2. 生成工作使用的线程(可以是自定义个数，也可以是系统默认)
3. 线程分别从队列中取出任务，并执行，一个任务执行完成需要告诉主线程完成一个任务
4. 再从任务队列中取出任务，直到所有任务为空，退出线程

### 线程池模型

```
def worker():
    while True:
        item = q.get()
        do_work(item)
        q.task_done()
q = Queue()
for item in source():
    q.put(item)
for i in range(num_threads):
     t = Thread(target=worker)
     t.daemon = True
     t.start()
q.join()   
```

1. 首先创建队列```q```，将原始任务```source```加入到队列q中
2. 循环创建```num_threads```个线程，执行```worker```函数
3. 在```worker```函数中，```while True```一直循环，从队列```q```中取得元素，执行对q的操作。
4.  ```Queue.task_done()```方法告诉之前队列中的一个任务被完成
5.  ```Queue.join()```阻塞调用进程，直到队列中所有任务都被处理，然后继续主线程
6. 由于之前设置了```thread.daemon = True```,所以主线程结束,不用等待还在活动的子线程，并结束子线程

线程池的使用过程中运用了大量的队列知识[Queue官方中文文档](http://python.usyiyi.cn/translate/python_278/library/queue.html)和[线程池大量的实现文章](http://www.cnblogs.com/goodhacker/p/3359985.html)

### 线程池的使用

```
import urllib2 
from multiprocessing.dummy import Pool as ThreadPool 
urls = [
        ...
		... 
        ]
pool = ThreadPool(4) 
results = pool.map(urllib2.urlopen, urls)
pool.close() 
pool.join()
#调用join之前，先调用close函数，否则会出错。执行完close后不会有新的进程加入到pool,join函数等待所有子进程结束
```

 ```multiprocessing```在python中是一个多进程模块，而```multiprocessing.dummy```则是一个多线程模块

pool = ThreadPool(4)是在线程池中创建了4个子线程，pool._processes是线程使用的数量

对于```map(function, iterable, ...)```这个函数,最初步的看就是对iterable每一个元素使用function做处理，结果作为list返回

```
>>> def add100(x):
...     return x+100
... 
>>> hh = [11,22,33]
>>> map(add100,hh)
[111, 122, 133]
```

有点类似于```[f(x) for x in iterable]```

```
>>> [add100(i) for i in list1]
[101, 102, 103]
```

这里有一篇很好的[python map()的解析](http://my.oschina.net/zyzzy/blog/115096)
和一篇[进程池的简介](http://www.cnblogs.com/kaituorensheng/p/4465768.html)

## 参考

1. [Queue官方中文文档](http://python.usyiyi.cn/translate/python_278/library/queue.html)
2. [线程池大量的实现文章](http://www.cnblogs.com/goodhacker/p/3359985.html)
3. [python map()的解析](http://my.oschina.net/zyzzy/blog/115096)
4. [进程池的简介](http://www.cnblogs.com/kaituorensheng/p/4465768.html)



