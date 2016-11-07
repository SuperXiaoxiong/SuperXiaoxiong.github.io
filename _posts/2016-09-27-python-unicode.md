---
layout: post
title:  "python2 unicode的使用"
date:   2016-09-27 19:50:44
categories: python
tags:  python unicode 
---


* content
{:toc}

## 简述

python2 unicode的学习和使用



## UTF-8和Unicode的关系

Unicode是一种字符集, 而UTF-8,UTF-16,UTF-32都是Unicode字符集实现的一种方式，在阮一峰的文章中有有关unicode编码的详细说明,http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html

## python2中使用如何Unicode

在python2中所有有关Unicode编码问题都可以统一成一个步骤

1. 外部输入编码，decode转成unicode
2. 逻辑处理(内部编码，统一unicode)
3. encode转成需要的目标编码，输出

### str和unicode的转换

 ```str```和```unicode```都是```basestring```的子串

```
unicode = str_obj.decode('str的编码')
str = unicode_obj.encode('希望得到的str的编码')
isinstance(u'中文', unicode) //判断编码方式
isinstance(u'中文', str) //判断编码方式
```

### 文本展示

使用声明

```
# -*- coding: utf-8 -*-
#coding=utf-8
```

若头部声明```coding=utf-8```,```a = '中文'```其编码为utf-8
若头部声明```coding=gb2312```,```a = '中文'``` 其编码为gbk

还要注意源文件的编码保存方式，与文件声明保持一致。

自定义字符串直接通过:```s=u'中文'```定义为Unicode编码

### 命令行输入

```
>>> s = raw_input('请输入:')
请输入:中文
>>> type(s)
<type 'str'>
```

可以看到从命令行输入的是```str```类型，所以在使用的时候按照原则需要转换成```unicode```编码，使用```s.decode(sys.stdin.encoding)```,转换成```unicode```编码,```sys.stdin.encoding```是读取当前环境的编码

```
>>> import sys
>>> print sys.stdin.encoding
UTF-8
>>> t = s.decode(sys.stdin.encoding)
>>> type(t)
<type 'unicode'>

```

### 文本输入

用```codecs```提供的```open```方法来指定打开的文件的语言编码，它会在读取的时候自动转换为内部```unicode```

```
info = codecs.open(file,'w','utf-8')
```

通过```codecs```指定输入文件打开的语言编码,会被默认转化，但使用标准文件读写方式读入的```str```需要自己```decode```成```Unicode```编码


```
>>> import chardet
>>> f = open("file")
>>> print chardet.detect(f.read())
{'confidence': 0.99, 'encoding': 'utf-8'}
```

判断文件的编码形式

### 字符串比较

比较按照原则来，采用```Unicode```编码进行比较，其实只要在输入阶段都转化成```Unicode```编码，这里也不用担心了

### 命令行输出

 ```sys.stdin.encoding```和```sys.stdout.encoding```是标准输入输出sdtin，stdout输入输出使用的编码，包命令行参数和print输出，由locale环境变量决定，所以一般print输出不需要将```Unicode```编码```encoding```。```sys.getdefaultencoding()```文件读写和字符串处理等操作使用的默认编码

```
>>> print sys.stdin.encoding
UTF-8
>>> print sys.stdout.encoding
UTF-8
>>> print u'中文'
中文
>>> print sys.getdefaultencoding()
ascii
>>> sys.setdefaultencoding('utf-8') //慎用
```


### 正则表达式

TODO 

http://blog.csdn.net/liweisnake/article/details/17325493

## 参考

* [编码处理小节python2](http://wklken.me/posts/2013/08/31/python-extra-coding-intro.html)
* [codecs介绍](http://san-yun.iteye.com/blog/1544123)
* [字节流的讲解](http://blog.jobbole.com/50345/)
* [unicode不同场景的使用](https://gist.github.com/x7hub/178c87f323fbad57ff91)
* [常用的处理方法](http://blog.csdn.net/eastmount/article/details/48841593)
