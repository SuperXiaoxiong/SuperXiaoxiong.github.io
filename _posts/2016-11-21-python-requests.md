---
layout: post
title:  python2 requests使用心得
date: 2016-11-21 20:17:23
categories: python
tags:  python requests urllib 
---

* content
{:toc}

## 问题描述

记录在使用requests库中的小技巧





## 官方文档

* [requests官方中文文档链接](http://cn.python-requests.org/zh_CN/latest/)
* [requests github地址](https://github.com/kennethreitz/requests)
* [requests github学习地址](https://github.com/wangshunping/read_requests)
* imooc上有学习视频


### response常用对象

```
response.request.headers 查看发送的headers
response.request.bogy 查看发送的body
response.status_code 查看返回的状态码
response.url 查看发送的url
response.headers响应头
response.history 是Response对象的列表，用allow_redirects=False屏蔽重定向
response.elapsed 发出响应到响应到达的时间

response.raw   原始响应内容 
response.content  二进制响应体，下载图片等常用
response.text     根据编码自动解码,使用res.encoding可以改变
response.json    json编码后
```

### 使用的超时异常处理

```
try:
    response = requests.get(url, timeout=1)
    response.raise_for_status()
except requests.exceptons.Timeout as e:
    print e.message
except requests.exceptions.HTTPError as e
    print e.message
```

### prepare使用

```
req = requests.Request('GET', url, auth, headers)
req.prepare().body
req.prepare().headers
// 构造prepare

resp = Session().send(req.prepare(), timeout=5)
resp.status_code
resp.request.headers
resp.text
```

### cookie的保存和使用

在使用```session```的过程中有很大需求将cookies保存下来,直接附上代码

```
def save_cookies_lwp(cookiejar, filename):
    lwp_cookiejar = cookielib.LWPCookieJar()
    for c in cookiejar:
	args = dict(vars(c).items())
	args['rest'] = args['_rest']
	del args['_rest']
	c = cookielib.Cookie(**args)
	lwp_cookiejar.set_cookie(c)
    lwp_cookiejar.save(filename, ignore_discard=True)


def load_cookies_from_lwp(filename):
    lwp_cookiejar = cookielib.LWPCookieJar()
    lwp_cookiejar.load(filename, ignore_discard=True)
    return lwp_cookiejar
```



### urllib.quote使用注意

requests库中的```params```和```data```都会自动将编码转换成utf-8模式
 
但构造特殊的url可能会用到```url```可能会用到```urllib.quote()```函数,```def quote(s, safe='/')```

1. quote方法 编码字符串```s```不支持unicode,使用会出错

```
urllib.quote(u'中文')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
    File "C:\Python27\lib\urllib.py", line 1299, in quote
        return ''.join(map(quoter, s))
	KeyError: u'\u4e2d'
```

必须编码再使用

```
urllib.quote(u'中文'.encode('utf-8'))
'%E4%B8%AD%E6%96%87'
```

2. 默认```safe```不会编码```/```

```
urllib.quote('hi/')
'hi/'
urllib.quote('hi/', safe='')
'hi%2F'
```

### 响应体工作内容流


使用```urllib1/2```的时候，默认```read/readline()```使用时才读取响应内容，而在```requests```库，默认是获得所有响应体才会返回

如果使用响应体工作内容流的方式就可以推迟下载响应体,直到访问```Response.content```,一般可以用于下载前获得文件总大小,等headers属性

```
tarball_url = 'https://github.com/kennethreitz/requests/tarball/master'
r = requests.get(tarball_url, stream=True)
```

### SSL的使用

使用```verify=False```忽略SSL验证，使用下面代码忽略警告

```
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
```

### TODO

也有一些内容没有使用，等待理解

#### prepare使用

```
req = requests.Request('GET', url, auth, headers)
req.prepare().body
req.prepare().headers
// 构造prepare

resp = Session().send(req.prepare(), timeout=5)
resp.status_code
resp.request.headers
resp.text

#### 钩子函数

TODO

## pip 使用小技巧 

```
pip --version 查看版本
pip --help 查看帮助
pip freeze 查看安装了哪些Python包


安装了virtualenv 后
可以专门配置独立虚拟的环境
在当前目录下 配置 virtualenv  .env
source .env/bin/activate
进入了新的环境
```

