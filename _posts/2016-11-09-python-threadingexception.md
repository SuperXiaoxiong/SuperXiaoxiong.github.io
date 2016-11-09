
---
layout: post
title:  python2 线程异常处理所遇到的问题
date: 2016-11-09 21:57:23
categories: python
tags:  python exception 
---

* content
{:toc}

## 问题描述

记录和解决在用python线程时候使用的问题





## ctrl+c退出被忽略

在使用线程编程时，使用ctrl+c结束进程，结果告知键盘中断异常被忽略

下面是报错信息

```
Exception KeyboardInterrupt in <module 'threading' from '/usr/local/Cellar/python/2.7.9/Frameworks/Python.framework/Versions/2.7/lib/python2.7/threading.pyc'> ignored
```

解决方式，设置子线程的```daemon```属性

```
t.setDaemon(True)
```

在子线程```start```之前，将Daemon属性设为真，就表示在主线程要退出时，不用等待子线程结束，并且结束子线程

## 主线程捕获KeyboardInterrupt失败

TODO

不知道是不是win环境问题，待解决

## 使用线程池报错太简略

在使用```multiprocessing.dummy```的线程模块的过程中,对```Pool().map()```线程池执行的线程报错并不会到线程运行的函数中去，给调试带来很大的困扰

