---
layout: post
title:  "python 文件操作"
date:   2016-08-23 23:20:44
categories: python
tags:  python  file
---

* content
{:toc}

## 简述

文件的操作，我将其分为文件打开，文件读取，文件写入，文件缓存，文件关闭和文件指针等几个方面




## 文件读写

在进行保存文件时，文件处理也是一个很重要的过程

### 斜杠和反斜杠

要将文件保存需要对目录进行操作。在windows系统中```C:\Users\user\Desktop```显示是反斜杠，```'\'```一般用作转义符，但实际上在window系统中```'/' 和 '\'``` 都支持分隔子目录的操作。在linux和web只支持```'/'```来标志来分隔子目录。

window 四种路径方式

```
path = r"C:\Windows\temp\readme.txt"
path1 = r"c:\windows\temp\readme.txt"
path2 = "c:\\windows\\temp\\readme.txt"
path3 = "c:/windows/temp/readme.txt"
```

1. path："\"为转义字符，加上r后变为原始字符串，则不会对字符串中的"\t"、"\r"进行字符串转义
2. path1：大小写不影响windows定位到文件，linux会影响
3. path2：用一个"\"取消第二个"\"的特殊转义作用，即为"\\"
4. path3：用正斜杠做目录分隔符也可以转到对应目录

### 文件打开

```
file object = open(file_name [, access_mode][, buffering])
```

这里对于文件打开方式做一下简要说明

```
'r'   只读      文件必须存在
'r+'  读写      同上
'w'   只写      文件不存在则创建/存在则清空
'w+'  读写      同上
'a'   追加      文件不存在则创建文件
'a+'  追加和读写 
```

例如'rb'等后面添加b的方式都是以二进制的方式打开

### 文件读取

```
read([size]):      读取文件
readline([size]):  读取一行
readlines([size]): 读取完文件，返回每一行所组成的列表  
iter:              使用迭代器读取文件
```

当对size进行设置的时候：read和readline都会返回size或者小于size字节的内容

readlines 当size设置时会返回接近于缓冲的字节数(会将一行全部读完)。缓存是io.DEFAULT_BUFFER_SIZE

```
f = open('a.txt')
iter_f = iter(f)
for line in iter_f:
	do_something(line)
f.close()
```

迭代器并不是一次性将文件全部读入，而是每一次对line操作时才会读入一个line。

### 文件写入

```
write(str):       将字符串写入文件
writelines(sequence of strings): 写多行到文件，参数为字符串所组成的序列
```

在写文件的过程中，首先会将文件写入文件缓存，当写入的数据量大于或等于写缓存，写缓存同步到磁盘

调用文件关闭```f.close()```或者```f.flush()```会将写缓存同步到磁盘

### 文件指针操作

```
seek(offset[, whence])
```

offset:偏移量，可以为负数
whence：偏移相对位置

文件指针定位方式whence：

```
os.SEEK_SET(0):   相对文件起始位置 
os.SEEK_CUR(1):  相对文件当前位置  
os.SEEK_END(2):  相对文件结束位置 
```

```file.tell()```，返回文件当前的偏移

## 常用文件属性

```
file.fileno()  文件描述符
file.mode      文件打开权限
file.encoding  文件编码格式
file.closed    文件是否关闭
```

### python 标准文件

sys模块中

文件标准输入:sys.stdin

文件标准输出:sys.stdout

文件标准错误:sys.stderr

### 系统命令行参数

sys.argv 是一个命令行参数的列表。

sys.argv[0]表示代码本身文件路径，所以参数从1开始

### 中文编码

1.将unicode转换成utf-8

```
a = unicode.encode(u'汉字','utf-8')
```

2.创建codecs模块创建指定编码格式文件

```
open(fname,mode,encoding,errors,buffering)
```

使用指定编码格式打开文件

## TODO

只是简单的对文件操作的常用部分做了简介，列一个关于文件操作的TODO列表

1. 目录操作
2. 各种注意点

## 参考
1. imooc python学习 