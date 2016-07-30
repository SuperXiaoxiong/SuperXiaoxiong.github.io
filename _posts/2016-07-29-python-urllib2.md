---
layout: post
title:  "python 网络urllib2模块"
date:   2016-07-29 17:20:44
categories: python
tags:  python urllib2 网络 
---


* content
{:toc}

## 简述

python网络之urllib2学习(python2.7版本)

相对于urllib模块，urllib2模块提供了很多辅助的接口用于处理基本认证，重定向，cookies等相关问题




## urlopen

### 构造方法

```
urllib2.urlopen(url[, data][, timeout])
```

url 参数可以是字符串也可以是Request对象

data 是一个标准```application/x-www-form-urlencoded```格式的缓存. ```urllib.urlencode()```使用一个词典或者二元组的序列参数以这种格式返回这样格式的字符串. urllib2使用包含```Connection:close```头信息的HTTP/1.1来发送请求.

timeout 以秒为单位指定像访问链接阻塞操作的超时时间 (如果不指定这个时间，那么就使用所设置的全局默认超时时间). 这个设置只用于HTTP, HTTPS和FTP连接

### 实例方法

1. 构造方法返回一个类文件对象，可用类似文件方法操作```read()```,```readline()```,```readlines()```,```fileno()```,```close()``` 
2.  ```geturl()```通常决定是否跟随者一个重定向
3.  ```info()```返回的头部信息
4.  ```getcode()```响应的状态码

## Request

Request类是一个抽象的URL请求

```
class urllib2.Request(url[, data][, headers][, origin_req_host][, unverifiable])
```

URL 链接地址，字符串

data 也是字符串，与urlopen中的data参数相同,Encode需要通过urllib来完成

headers 字典类型,可以通过实例方法```add_header()```方法添加键值对，```headers = { 'User-Agent' : user_agent }```,在使用服务器提供的 RESTful 或 SOAP 服务时， Content-Type 设置错误会导致服务器拒绝服务

有关headers的属性：

* User-Agent : 有些服务器或 Proxy 会通过该值来判断是否是浏览器发出的请求
* Content-Type : 在使用 REST 接口时，服务器会检查该值，用来确定 HTTP Body 中的内容该怎样解析。
* application/xml ： 在 XML RPC，如 RESTful/SOAP 调用时使用
* application/json ： 在 JSON RPC 调用时使用
* application/x-www-form-urlencoded ： 浏览器提交 Web 表单时使用

origin_req_host 是RFC2965定义的源交互的request-host。默认的取值是cookielib.request_host(self)。这是由用户发起的原始请求的主机名或IP地址。例如，如果请求的是一个HTML文档中的图像，这应该是包含该图像的页面请求的request-host。

unverifiable 代表请求是否是无法验证的，它也是由RFC2965定义的。默认值为false。一个无法验证的请求是，其用户的URL没有足够的权限来被接受。例如，如果请求的是在HTML文档中的图像，但是用户没有自动抓取图像的权限，unverifiable的值就应该是true。

## install_opener/build_opener

```
urllib2.install_opener(opener)
urllib2.build_opener([handler, ...])
```

可以看一下[the5fire](https://www.the5fire.com/read-urllib2-source-code-02-html.html)博文中对于urllib2库的分析

```
def urlopen(url, data=None, timeout=socket._GLOBAL_DEFAULT_TIMEOUT):
    """
    对外的访问函数
    """
    global _opener
    if _opener is None:
        _opener = build_opener()
    return _opener.open(url, data, timeout)
```

在程序第一次执行urlopen操作的时候，其实就是构建了一个全局的_opener对象，然后用这个_opener对象来处理url以及data。避免多次构建_opener对象,urllib2库提供了build_opener来创建_opener对象，install_opener来加载

_opener是OpenerDirector的一个实例,其中存放着用于处理request，打开request，处理response的句柄handler(handler也就是处理特种协议如http，ftp或者cookie等具体句柄,如HTTPBasicAuthHandler、HTTPCookieProcessor、ProxyHandler实例)

对于build_opener函数，就是把所有的handler都实例化通过opener.add_handler方法添加给OpenerDirector。最后返回构建好的OpenerDirector实例。build_opener 用来构建处理器（handlers），建造一些默认的或者通过参数传递进来的handler到openerdirector中待用。

build_opener默认会加入许多handlers，它提供了一个快速的方法添加更多东西和使默认的handler失效。

```
import urllib2
opener = urllib2.build_opener()
opener.addheaders = [('User-agent', 'Mozilla/5.0')]
opener.open('http://www.example.com/')
```

install_opener如上所述也能用于创建一个opener对象，但是这个对象是（全局）默认的opener。这意味着调用urlopen将会用到你刚创建的opener。这段代码最终还是使用的默认opener。一般情况下我们用build_opener为的是生成自定义opener，没有必要调用install_opener，除非是为了方便

```
import urllib2
req = urllib2.Request('http://www.python.org/')
opener=urllib2.build_opener()
urllib2.install_opener(opener)
f = urllib2.urlopen(req)
```

## 密码验证

HTTPBasicAuthHandler()处理程序可用add_password()来设置密码。

实例方法
```
add_password(realm,url,user,passwd)
```

realm是与验证相关联的名称或描述信息，取决于远程服务器。uri是基URL。user和passwd分别指定用户名和密码。

```
import urllib2
auth=urllib2.HTTPBasicAuthHandler()
auth.add_password('Administrator','http://www.example.com','Dave','123456')
opener=urllib2.build_opener(auth)
u=opener.open('http://www.example.com/evilplan.html')
```

## Cookie

使用HTTPCookieProcessor,这样可以得到某个cookie项的值

```
import urllib2,cookielib
cookie=cookielib.CookieJar()
cookiehand=urllib2.HTTPCookieProcessor(cookie)
opener=urllib2.build_opener(cookiehand)
response = opener.open('http://www.google.com')
for item in cookie:
    if item.name == 'some_cookie_item_name':
        print item.value
```

## 代理设置

urllib2 默认会使用环境变量 http_proxy 来设置 HTTP Proxy。如果想在程序中明确控制 Proxy 而不受环境变量的影响，可以使用下面的方式

```
import urllib2

enable_proxy = True
proxy_handler = urllib2.ProxyHandler({"http" : 'http://some-proxy.com:8080'})
null_proxy_handler = urllib2.ProxyHandler({})

if enable_proxy:
    opener = urllib2.build_opener(proxy_handler)
else:
    opener = urllib2.build_opener(null_proxy_handler)
```

## 异常处理

当不能处理一个response时，urlopen抛出一个URLError
HTTPError是HTTP URL在特别的情况下被抛出的URLError的一个子类。

### URLError

handlers当运行出现问题时（通常是因为没有网络连接也就是没有路由到指定的服务器，或在指定的服务器不存在），抛出这个异常.

这个抛出的异常包括一个‘reason’ 属性,他包含一个错误编码和一个错误文字描述。

### HTTPError

HTTPError是URLError的子类。每个来自服务器HTTP的response都包含"status code". 有时status code不能处理这个request. 默认的处理程序将处理这些异常的responses。例如，urllib2发现response的URL与你请求的URL不同时也就是发生了重定向时，会自动处理。对于不能处理的请求, urlopen将抛出HTTPError异常. 典型的错误包含'404' (没有找到页面), '403' (禁止请求),'401' (需要验证)等。它包含2个重要的属性reason和code。

当一个错误被抛出的时候，服务器返回一个HTTP错误代码和一个错误页。你可以使用返回的HTTP错误示例。这意味着它不但具有code和reason属性，而且同时具有read，geturl，和info等方法

如果我们想同时处理HTTPError和URLError，因为HTTPError是URLError的子类，所以应该把捕获HTTPError放在URLError前面，如不然URLError也会捕获一个HTTPError错误， 

```
import urllib2
req = urllib2.Request('http://www.python.org/fish.html')
try:
    response=urllib2.urlopen(req)
except urllib2.HTTPError,e:
    print 'The server couldn\'t fulfill the request.'
    print 'Error code: ',e.code
    print 'Error reason: ',e.reason   
except urllib2.URLError,e:
    print 'We failed to reach a server.'
    print 'Reason: ', e.reason
else:
    # everything is fine
    response.read()
```

## Redirect

urllib2 默认情况下会针对 HTTP 3XX 返回码自动进行 redirect 动作，无需人工配置。要检测是否发生了 redirect 动作，只要检查一下 Response 的 URL 和 Request 的 URL 是否一致就可以了。

```
import urllib2
response = urllib2.urlopen('http://www.google.cn')
redirected = response.geturl() == 'http://www.google.cn'
```

如果不想自动 redirect，除了使用更低层次的 httplib 库之外，还可以自定义 HTTPRedirectHandler 类。

```
import urllib2
class RedirectHandler(urllib2.HTTPRedirectHandler):
    def http_error_301(self, req, fp, code, msg, headers):
        pass
    def http_error_302(self, req, fp, code, msg, headers):
        pass
opener = urllib2.build_opener(RedirectHandler)
opener.open('http://www.google.cn')
```

## 使用 HTTP 的 PUT 和 DELETE 方法

urllib2只支持HTTP的GET和POST方法，可以使用低层次的httplib库来使用PUT和DELETE方法

```
import urllib2

request = urllib2.Request(uri, data=data)
request.get_method = lambda: 'PUT' # or 'DELETE'
response = urllib2.urlopen(request)
```

Hack 的方式,来使用PUT和DELETE方法
	
## Debug Log

使用 urllib2 时，可以通过下面的方法把 debug Log 打开，这样收发包的内容就会在屏幕上打印出来，方便调试，有时可以省去抓包的工作

```
import urllib2

httpHandler = urllib2.HTTPHandler(debuglevel=1)
httpsHandler = urllib2.HTTPSHandler(debuglevel=1)
opener = urllib2.build_opener(httpHandler, httpsHandler)

urllib2.install_opener(opener)
response = urllib2.urlopen('http://www.google.com')
```

## 参考

* [python中文官方文档urllib2模块2.7](http://python.usyiyi.cn/python_278/library/urllib2.html)
* [urllib2学习总结 风行隐者](http://www.cnblogs.com/wly923/archive/2013/05/07/3057122.html)
* [Python 标准库 urllib2 的使用细节zhuoqiang](http://zhuoqiang.me/python-urllib2-usage.html)
* [python urllib2源码解读the5fire的技术博客](https://www.the5fire.com/read-urllib2-source-code-02-html.html)