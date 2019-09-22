---
layout: post
title:  http_request_smuggling 翻译
date: 2019-09-22
categories: 漏洞分析
tags:  HTTP
---

* content
{:toc}

## 简述

翻译：https://portswigger.net/web-security/request-smuggling

这篇文章主要描述 HTTP request smuggling 攻击 和 smuggling 攻击是如何产生的












## 什么是 HTTP 请求 smuggling

HTTP request smuggling 是一项用于干扰HTTP 请求的技术，网站会处理来自用户的多个 HTTP 请求，使用 request smuggling 可以干扰其他用户的请求。 请求 smuggling 漏洞在实践场景中经常发生，允许攻击者绕过控制，获得未授权的数据，并且直接攻陷应用的其他用户

![](https://github.com/SuperXiaoxiong/SuperXiaoxiong.github.io/blob/master/img/http_request_smuggling_1.png?raw=true)


HTTP 请求 smuggling 第一次在2005 年提出，最近被 [PortSwigger](https://portswigger.net/blog/http-desync-attacks-request-smuggling-reborn)的研究提上了头条 


## HTTP request smuggling  攻击时怎样的

现在的网络多数在用户和最终的逻辑应用之间使用 HTTP 服务器链，用户发送请求给前端（通常叫负载均衡和代理服务器），前端服务器把请求发送给一个或者多个后端。在现有基于云的应用中，这种类型的架构已经越来越常见。

当前端服务器转发 HTTP 请求给后端时，会一次把多个请求转发给后端，因为这样可以更有效率。协议很简单，HTTP 请求一个接着一个发送，后端服务器根据HTTP 请求头决定 一个请求结束另一个开始。

![](https://github.com/SuperXiaoxiong/SuperXiaoxiong.github.io/blob/master/img/http_request_smuggling_2.png?raw=true)

在这种情况下，前端服务器和后端服务器需要定义两个请求的边界。不然攻击者可以发送让前后端系统混淆的请求。

![](https://github.com/SuperXiaoxiong/SuperXiaoxiong.github.io/blob/master/img/http_request_smuggling_3.png?raw=true)

比如，攻击者可以让一部分前端服务器处理的请求 被后端服务器当作另一个请求的开头处理。通过伪装成后续的请求，可以干扰应用处理HTTP 请求，这就是 request smuggling 攻击，可以带来毁灭性的效果

## HTTP request smuggling 漏洞如何发送

总舵的HTTP request smuggling 漏洞 是因为 HTTP 明确提供了两个不同的方式来表示请求结束，请求头中的 `Content-Length` 字段和 `Transfer-Encoding` 字段

`Content-Length` 字段时直接的，明确了请求体的字节数 

比如 

    POST /search HTTP/1.1
    Host: normal-website.com
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 11

    q=smuggling

`Transfer-Encoding` 头可以用来明确请求体的块编码方式，这表明 消息体 包含了一个或者多个数据块，每一个数据块都用十六进制数据写明了大小，在块内容之后新的一行中表明了块的大小。使用 size为0来表示块结束了

    POST /search HTTP/1.1
    Host: normal-website.com
    Content-Type: application/x-www-form-urlencoded
    Transfer-Encoding: chunked

    b
    q=smuggling
    0

**Note**：  

chunked encoding 很少见是因为如下两个原因：

* Burp Suite 自动解压缩chunked 编码，使消息更容易查看和编辑
* 浏览器在请求时不怎么使用 chunked 编码，chunked编码在服务器响应时比较常见

因为 HTTP 规范使用两种不同的方式明确 HTTP 消息，如果在一次请求中同时使用两种方式，就会造成冲突。HTTP 规范说明，如果同时使用 这两种方式，`Content-Length`请求头就会被忽略。当只有一个服务器运行时，可以避免歧义，当两个或者多个服务器被串起来时，情况就不一样了，可能有如下两个原因：

* 一些服务器不支持请求头中的`Transfer-Encoding` 
* 如果以某种方式混淆了文件，一些服务器会被诱导不处理`Transfer-Encoding` 请求头字段

如果前端和后端服务器在处理`Transfer-Encoding`时明显不同，就会造成请求的边界识别出差异，导致 request smuggling 漏洞

## 如果发起 HTTP request smuggling 攻击

Request smuggling 攻击同时包含 `Content-Length` 头和 `Transfer-Encoding` 在同一个请求中，通过操纵这两个字段使前后端服务器在处理请求时表现不同。确切的方式取决于两个服务器

* `CL.TE`:前端服务器使用 `Content-Length` 头，后端使用`Transfer-Encoding` 头
* `TE.CL`：前端服务器使用`Transfer-Encoding`头 后端服务器使用`Content-Length`头
* `TE.TE` 前后端服务器都使用 `Transfer-Encoding`头，但是两个服务器被诱导后会产生不同行为


### `CL.TE`漏洞

前端服务器使用 `Content-Length` 头，后端使用`Transfer-Encoding` 头，我们可以使用如下简单的HTTP request smuggling 攻击

    POST / HTTP/1.1
    Host: vulnerable-website.com
    Content-Length: 13
    Transfer-Encoding: chunked

    0

    SMUGGLED

前端服务器处理 `Content-Length`请求头字段，认为 请求体为13bytes，请求体以 `SMUGGLED` 结束，然后将请求转发给后端

后端服务器 处理 `Transfer-Encoding` 所以认为请求体使用块编码。当处理到第一个请求块时，size 为0，认为请求结束，SMGUGGLED 字串会被遗留，后端服务器会认为这个字段是另外一个请求开始

[**LAB**](https://portswigger.net/web-security/request-smuggling/lab-basic-cl-te)

### `TE.CL`漏洞

因为前端服务器使用`Transfer-Encoding`头，而后端服务器使用 `Content-Length`头，我们使用如下请求包进行攻击

   POST / HTTP/1.1
    Host: vulnerable-website.com
    Content-Length: 3
    Transfer-Encoding: chunked

    8
    SMUGGLED
    0 

**Note** 在使用Burp Repeater 的时候，使用 Repeater 菜单保证 `Update Content-Length`选项是没有开启的

需要在 0 的末尾加上 `\r\n\r\n` 两个换行

前端处理 `Transfer-Encoding` 请求头，使用块编码来处理，所以第一个块长度为8 包含了 SMUGGLED。第二个块长度为0 请求结束，并转发给后端服务器

后端服务器处理 `Content-Length` 请求头，认为请求体长度只有3个bytes，所以处理到8 换行，认为请求结束。SMUGGLED 被遗弃，后端服务器会认为这是第二个请求开始

[**LAB **](https://portswigger.net/web-security/request-smuggling/lab-basic-te-cl)

### `TE.TE`漏洞

前后端服务器都支持 `Transfer-ENcoding`.但是其中一个被诱导导致请求头被混淆，以下有好几种方式混淆`Transfer-Encoding`头：

    Transfer-Encoding: xchunked

    Transfer-Encoding : chunked

    Transfer-Encoding: chunked
    Transfer-Encoding: x

    Transfer-Encoding:[tab]chunked

    [space]Transfer-Encoding: chunked

    X: X[\n]Transfer-Encoding: chunked

    Transfer-Encoding
    : chunked

这些技术中的每一种都和 HTTP 规范略有不同。真实世界代码很少准确的遵守实现协议规范，不同的实现会适用不同的规范变化。为了 发现`TE.TE`漏洞，有必要发现不同前后端服务器处理 `Transfer-Encoding`头的区别。

根据是否可以诱导前端后端服务器不处理混淆的传输编码头，攻击的其余部分将采用与CL.TE/TE.CL相同的形式。

[**LAB**](https://portswigger.net/web-security/request-smuggling/lab-ofuscating-te-header)

相似文章
* [如何寻找request smuggling 漏洞](https://portswigger.net/web-security/request-smuggling/finding)
* [如何利用 request smuggling漏洞](https://portswigger.net/web-security/request-smuggling/exploiting)


## 如何阻止 HTTP request smuggling 漏洞

HTTP request smuggling 漏洞主要发生在一次网络请求中前端服务器一次转发多个HTTP 请求到后端，这个协议对于两个或者多个服务器在处理请求时容易混淆边界存在风险，一些阻止 requst smuggling 攻击方式如下

* 禁用后端链接重用，前后端一次请求转发指挥发送一个请求
* 对于后端链接使用 HTTP/2 ， 这个协议阻止了请求的混淆
* 使用明确相同的前后端服务器软件，在处理请求边界时没有差异

在一些情况下，可以通过前端服务器清理有混淆性的请求或者使用后端服务器拒绝混淆性的请求来阻止攻击。然而，这些方式相对于上面的缓解方式更容易出错。


