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

当没有显示的创建时，会默认创建root logger，默认日志级别warning，处理器handler将日志信息打印到标准输出，默认格式第一个简单实用程序中输出的格式

```
logger.setLevel(logging.INFO)    #设置日志级别为INFO，只有日志级别大于INFO的才会输出
logger.addHandler(handler_name)  #为logger对象增加处理器
logger.removeHandler(handler_name) 
```

### handler

 ```handler```将日志分发到合适的目的地（以日志级别为区分）

 ```logger```通过```addHandler()```方法给它自己添加零个或者多个处理器

创建handler之后通过下面的方法来设定日志级别，设定格式，添加一个或多个过滤器

```
setLevel(logging.WARN) # 指定日志级别，低于WARN级别的日志将被忽略
setFormatter(formatter_name) # 设置一个格式化器formatter
addFilter(filter_name) # 增加一个过滤器，可以增加多个
removeFilter(filter_name) # 删除一个过滤器
```

为什么会有两个setLevel()方法,记录器的级别决定了消息是否要传递给处理器。每个处理器的级别决定了消息是否要分发。

#### logging.StreamHandler

StreamHandler将日志输出发送到```sys.stdout```, ```sys.stderr```这样的流对象

 ```sh = logging.StreamHandler(stream=None)```

 ```stream```是一个文件对象，默认是sys.stderr

#### logging.FileHandler 

 ```fh = logging.FileHandler(filename, mode='a', encoding=None, delay=False)```

 默认为追加方式打开文件

#### RotatingFileHandler和TimedRotatingFileHandler

```
logging.handlers.RotatingFileHandler(filename, mode='a', maxBytes=0, backupCount=0, encoding=None, delay=0)
```

以文件大小为阈值，改名并重建新的日志文件，

改名规则：日志名字为tmp.log,达到阈值更改为tmp.log.1存储，若tmp.log.1存在类推为tmp.log.2或者后续数字，tmp.log就像临时存储文件

 ```maxBytes```指定阈值大小，当maxBytes=0，日志文件可以无限大

 ```backupCount```只保存指定数量的日志，会覆盖前面的日志 #TODO

```
logging.handlers.TimedRotatingFileHandler(filename, when='h', interval=1, backupCount=0, encoding=None, delay=False, utc=False)
```
  
 间隔一定时间创建日志
 
 ```interval```是时间间隔

 ```when```时间间隔单位，不区分大小写，有以下取值

```
S       秒
M       分
H       小时
D       天
W       每星期（interval==0时代表星期一）
midnight 每天凌晨
```

#### SocketHandler

```
logging.handlers.SocketHandler
logging.handlers.DatagramHandler
```

以上两个```Handler```类似，都是将日志信息发送到网络。不同的是前者使用```TCP```协议，后者使用```UDP```协议。它们的构造函数是：

```Handler(host, port)```

其中```host```是主机名，```port```是端口名

#### TODO

还有多种不常用handler，以及delay参数

### Formatter


 ```logging.Formatter(fmt=None, datefmt=None)```

使用```Formatter```对象设置日志信息最后的规则、结构和内容，默认的时间格式为```%Y-%m-%d %H:%M:%S```。

其中，```fmt```是消息的格式化字符串，```datefmt```是日期字符串。如果不指明```fmt```，将使用```%(message)s```。如果不指明```datefmt```，将使用```ISO8601```日期格式。


日志系统可以输出的各种信息

```
%(name)s	    Logger的名字
%(levelno)s	    数字形式的日志级别
%(levelname)s	文本形式的日志级别
%(pathname)s	调用日志输出函数的模块的完整路径名，可能没有
%(filename)s	调用日志输出函数的模块的文件名
%(module)s	    调用日志输出函数的模块名
%(funcName)s	调用日志输出函数的函数名
%(lineno)d	    调用日志输出函数的语句所在的代码行
%(created)f	    当前时间，用UNIX标准的表示时间的浮点数表示
%(relativeCreated)d	   输出日志信息时的，自Logger创建以来的毫秒数
%(asctime)s	    字符串形式的当前时间。默认格式是“2003-07-08 16:49:45,896”。逗号后面的是毫秒
%(thread)d	    线程ID。可能没有
%(threadName)s	线程名。可能没有
%(process)d	    进程ID。可能没有
%(message)s	    用户输出的消息
```

### Filter

 ```Filter```基类只允许特定```Logger```层次以下的事件。例如用```A.B```初始化的```Filter```允许```Logger``` ```A.B```, ```A.B.C```, ```A.B.C.D```, ```A.B.D```等记录的事件，```logger``` ```A.BB```, ```B.A.B``` 等就不行。 如果用空字符串来初始化，所有的事件都接受。

 ```filter = logging.Filter(name='')```

## 配置日志

### 使用源码

```
import logging

formatter = logging.Formatter('%(asctime)s %(filename)s %(name)s %(message)s ' )

sh = logging.StreamHandler(stream=None)
sh.setLevel(logging.INFO)
sh.setFormatter(formatter)

logger = logging.getLogger('test_logger')
logger.setLevel(logging.INFO)
logger.addHandler(sh)
logger.warning("warning !")

2016-09-14 22:20:39,693 <stdin> test_logger warning !
```


### 使用配置文件

配置文件必需包含名为```[loggers]```, ```[handlers]``` 和 ```[formatters]```的节，标识了文件中定义的各种类型的对象的名字

```
[loggers]
keys=root,log1

[handlers]
keys=handler0,handler1

[formatters]
keys=form0,form1
```

对于每一个实体，会一个独立的节标识如何配置该实体：[loggers]节中有名为log1的logger，就需要有[logger_log1]的节并会包含了相关的配置细节

根logger必需指定级别和handler列表。

```
level=DEBUG
handlers=handler0
```

 ```level```可以是```DEBUG```,```INFO```,```WARNING```,```ERROR```,```CRITICAL```或```NOTSET```其中之一。```NOTSET```表示所有的消息都要记录，这只对根```logger```有效

对于非根```logger```来说，需要一些额外的信息。

```
[logger_log1]
level=WARNING
handlers=handler1
propagate=1
qualname=log_1
```

如果非根logger的级别为NOTSET，系统参考高层次的logger来决定logger的有效级别。propagate为1表示将消息传递给高层次logger的handler，为0表示不传播。qualname是logger在层次中的名字，应用通过该名字得到logger,对于不存在的名字均为跟logger

```
[handler_handler0]
class=StreamHandler
level=INFO
formatter=form0
args=(sys.stderr,)
```

class表示handler的类

```
[formatter_form1]
format= %(asctime)s %(levelname)s %(message)s
datefmt= "%a %d %b %Y %H:%M:%S"
class=logging.Formatter
```

在调用配置文件代码中需要调用```logging.config.fileConfig()```方法：

```
import logging.config
logging.config.fileConfig('./logging_emp.conf')
```

### TODO

字典形式配置

## 参考

* [python指南](http://pythonguidecn.readthedocs.io/zh/latest/writing/logging.html)
* [python官方中文文档](http://python.usyiyi.cn/documents/python_278/library/logging.config.html#logging-config-api)
* [python logging模块使用教程_好吃的野菜](http://www.jianshu.com/p/feb86c06c4f4)
* [python log 老鱼的博客](http://hgoldfish.com/blogs/article/7/)
