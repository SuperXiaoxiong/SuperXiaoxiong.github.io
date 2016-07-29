---
layout: post
title:  "python网络请求urllib库学习"
date:   2016-07-29 12:19:05
categories: python
tags:  python urllib urllib2 request 
---


* content
{:toc}

## 简述

python网络请求之urllib(python2.7版本)





## urllib

### urlopen对象

#### 构造方法

创建一个url的类文件对象，然后进行类似文件操作

```
urllib.urlopen(url[, data[, proxies]])
```

1. url 链接，需要带上协议名
2. data 表示以post方式提交url数据
3. proxies 用于设置代理

#### 实例方法

1.  ```read()```,```readline()```,```readlines()```,```fileno()```,```close()``` 类似于文件使用方法
2.  ```info()```:返回```httplib.HTTPMessage```对象，远程服务器的头信息
3.  ```getcode()```:HTTP请求返回的状态码
4.  ```geturl()```：返回请求的url

### urlretrieve

urlretrieve方法将远程数据下载到本地。该方法返回一个包含两个元素的元组(filename, headers)，filename表示保存到本地的路径，header表示服务器的响应头。

```
urllib.urlretrieve(url[, filename[, reporthook[, data]]])
```

1. filename 指定了保存到本地的路径（如果未指定该参数，urllib会生成一个临时文件来保存数据）
2. reporthook 是一个回调函数，当连接上服务器、以及相应的数据块传输完毕的时候会触发该回调。我们可以利用这个回调函 数来显示当前的下载进度
3. data 指post到服务器的数据

 ```urllib.urlcleanup()```清除由于```urllib.urlretrieve()```所产生的缓存

实例

```
import urllib

def cbk(a, b, c):
    '''回调函数
    @a: 已经下载的数据块
    @b: 数据块的大小
    @c: 远程文件的大小
    '''
    per = 100.0 * a * b / c
    if per > 100:
        per = 100
    print '%.2f%%' % per

url = 'http://www.sina.com.cn'
local = 'd://sina.html'
urllib.urlretrieve(url, local, cbk)
```

回调函数的使用

TODO

### 编码

#### quote/unquote

对字符串进行编码/解码，使用%xx 转义替换string 中的特殊字符，字母、数字和'_.-' 字符永远不会转义

```
urllib.quote(string[, safe])
urllib.unquote(url)
```

可选的safe 参数指出其它不应该转义的字符 —— 默认值为'/'

#### quote_plus/unquote_plus

与urllib.quote类似，但这个方法用'+'来替换空格，'/'用%2F代替，而quote用'%20'来代替空格，'/'不替换

```
urllib.quote_plus(string[, safe])
urllib.unquote_plus(string)
```

#### urlencode

将dict或者包含两个元素的元组列表转换成url参数。例如 字典{'name':'a','age':20}将被转换为"name=a&age=20"

```
urllib.urlencode(query[, doseq])
```

#### pathname2url/url2pathname(path)

```
urllib.pathname2url(path)：将本地路径转换成url路径；
urllib.url2pathname(path)：将url路径转换成本地路径；
```

## 参考
* [python2.7官方中文版urllib库](http://python.usyiyi.cn/python_278/library/urllib.html)
* [urllib库学习 DarkBULL博文](http://python.jobbole.com/81478/)
* [urllib模块学习 中大博文](http://www.cnblogs.com/sysu-blackbear/p/3629420.html)