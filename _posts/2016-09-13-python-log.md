---
layout: post
title: "python 日志"
date: 2016-09-13 22:21:00
categories: python
tags: python log
---

* content
{:toc}

## 简述

日志系统是软件开发，调试，运维的过程必不可缺的元素

本文从简单的日志使用，日志各模块详细介绍python日志系统




## 简单的例子

### 输出到控制台

```
>>> import logging
>>> logging.warning("watch out!")
WARNING:root:watch out!
```

### 输出到文件

```
import logging
logging.basicConfig(filename='tmp.log',level=logging.DEBUG)
logging.debug('log file')
```

### 日志级别

```
DEBUG     调试详细信息
INFO      事情按照预期工作
WARNING   警告
ERROR     错误，软件一部分不能执行
CRITICAL  严重错误，表明软件已经不能继续运行
```

## logging库详细介绍

```
logger  可以直接调用的接口
handler 日志记录发送到合适的目的地
filter  过滤器，决定输出哪条记录
formatter 输出记录的最终格式
```

### logger

日志记录是通过调用logger的实例方法实现的，用```.```做分隔符

创建方法```logger = logging.getLogger(logger_name)```

