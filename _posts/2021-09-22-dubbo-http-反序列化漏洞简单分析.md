---
layout: post
title: dubbo-http-反序列化漏洞简单分析
date: 2021-09-21
categories: 漏洞分析
excerpt: 看了一个月的weblogic，尝试挖两周利用链，无果，心情很低落；工作需要对小众进行漏洞挖掘，项目间隙进行一段快节奏的漏洞复现，切换心情，然后进行一波简单的知识积累。
tags: java dubbo
---

* content
{:toc}

# cve-2020-1948 分析

看了一个月的weblogic，尝试挖两周利用链，无果，心情很低落；工作需要对小众进行漏洞挖掘，项目间隙进行一段快节奏的漏洞复现，切换心情，然后进行一波简单的知识积累。

影响版本：dubbo 版本 2.7.3 及以下。

一句话解释就是：dubbo 使用Spring HTTP Invoker RPC框架作为数据交换，但是该框架存在风险，默认会调用java原生反序列化机制，导致反序列化漏洞

## 漏洞调用整体逻辑

调用栈如图所示（很清晰），dubbo 在处理数据时使用Spring HTTP Invoker RPC调用框架进行序列化数据传输处理，server 在处理时进行反序列化，造成反序列化漏洞
![474917ea70d5d04000a03a46dbed69ca.png](/img/dubbo/853e9eef1ee94cb8956cab83cdc6544d.png)

调用代码如下

### DispatcherServlet#service 

service:61, DispatcherServlet (org.apache.dubbo.remoting.http.servlet)  调用  handle:216, HttpProtocol$InternalHandler (org.apache.dubbo.rpc.protocol.http)

![0b753a888f47e75190f89b9f9e8963ec.png](/img/dubbo/9e992b164f3c4a6e8a20814c9b22abec.png)

###  HttpProtocol$InternalHandler#handle

![2db8ac78b12a3901181384838eb5c24e.png](/img/dubbo/f533af3d1e024a42925a0af24444f02d.png)

### HttpInvokerServiceExporter (org.springframework.remoting.httpinvoker) 处理请求

![747812943b069bd691569425dbf50bb9.png](/img/dubbo/44a3f8ea147946a1894192c028d9c373.png)

### RemoteInvocationSerializingExporter (org.springframework.remoting.rmi) 造成反序列化

![c8b158e453f700e4793e53a77f35def3.png](/img/dubbo/43c9ffd888e14de0a7fcfa260c09051e.png)




## 两个疑问

关注调用链，产生两个疑惑

1.  疑问 `DispatcherServlet` 是在哪里注册的
2.  handlers.put(port, processor); 为什么加上 http 协议 `org.apache.dubbo.rpc.protocol.http.HttpProtocol`

org.apache.dubbo.rpc.protocol.http.HttpProtocol 使用了 org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter

![a50b55169d6cb12abb506aa1064673a0.png](/img/dubbo/1113936844a6448d861f0864387ef775.png)

### DispatcherServlet 注册逻辑

dubbo-samples-rest 项目中配置了 web.xml 文件，使用配置文件方式配置了使用的 DispatcherServlet，配置路径为 /service/*

![fe3abc0c652d452f5832bef54950b74a.png](/img/dubbo/c556a2759d4247a6bd478a90a5b103eb.png)

在dubb-samples-http 项目中没有 web.xml 文件，那servlet 是如何注册的了，配置生效地址是什么了，原来在 `org.apache.dubbo.remoting.http.tomcat.TomcatHttpServer` 中对 servlet 进行了注册(注：内嵌式tomcat 添加servlet)

![bca38ea12ba4f245309abb74a7b2f933.png](/img/dubbo/9b8c62edf020426f875cbe55e7476de6.png)

第二个问题也呼之欲出，为什么会调用 HTTPProtocol handle 

### HTTPProtocol 调用逻辑

Dispatcher addhandler 中加入了 `org.apache.dubbo.rpc.protocol.http.HttpProtocol$InternalHandler`
![d7312eebbde8bf156717ad4f83b8db05.png](/img/dubbo/8d78851ee29b4a30a8c80e7de0a9f1cc.png)

## 修复

前面说过dubbo的http处理是通过org.apache.dubbo.rpc.protocol.http.HttpProtocol，然后是实例化一个JsonRpcServer对象skeleton来处理uri，紧接着调用 skeleton.handle
也就是com.googlecode.jsonrpc4j.JsonRpcBasicServer#handle

暂代梳理调用逻辑

![8c29f3ef42e0900b33ebf278574320ff.png](/img/dubbo/e6a5e11487f74725a2180d91046b3761.png)

## 背景知识:Spring HTTP Invoker 介绍

>> Spring HTTP Invoker一种JAVA远程方法调用框架实现，原理与JDK的RMI基本一致。Spring使用各种技术为远程处理提供支持。远程处理支持简化了远程支持服务的开发，通过Java接口和对象作为输入和输出实现。远程处理技术：

* 远程方法调用（RMI）：通过使用RmiProxyFactoryBean和RmiServiceExporter，Spring支持传统的RMI（使用java.rmi.Remote接口和java.rmi.RemoteException)以及通过RMI调用器（使用任何Java接口）进行透明远程处理。RMI：使用JRMP协议(基于TCP/IP)，不允许穿透防火墙，使用JAVA系列化方式，使用于任何JAVA应用之间相互调用。
* Spring HTTP Invoker：Spring提供了一种特殊的远程处理策略，允许通过HTTP进行Java序列化，支持任何Java接口（就像RMI调用器那样）。相应的支持类是HttpInvokerProxyFactoryBean和HttpInvokerServiceExporter。使用HTTP协议，允许穿透防火墙，使用JAVA系列化方式，但仅限于Spring应用之间使用，即调用者与被调用者都必须是使用Spring框架的应用。
* Hessian：通过使用Spring的HessianProxyFactoryBean和HessianServiceExporter，你可以通过Caucho提供的基于HTTP的轻量级二进制协议透明地公开你的服务。使用HTTP协议，允许穿透防火墙，使用自己的系列化方式，支持JAVA、C++、.Net等跨语言使用。
* Java Web Services：Spring通过JAX-WS为Web服务提供远程处理支持。
* JMS：spring-jms模块中的JmsInvokerServiceExporter和JmsInvokerProxyFactoryBean类支持通过JMS作为底层协议进行远程处理。
* AMQP：通过AMQP作为底层协议的远程处理由单独的 Spring AMQP项目支持。
* 
* Burlap: 与Hessian相同，只是Hessian使用二进制传输，而Burlap使用XML格式传输（两个产品均属于caucho公司的开源产品)。（spring 好像不支持）


源码分析时从客户端和服务端配置两个对象 HttpInvokerServiceExporter、HttpInvokerProxyFactoryBean下手

![97ab3fd375b2086925a7862064214235.png](/img/dubbo/2e6e361ddd2942e6906307e88bf2cdce.png)

HttpInvokerServiceExporter 继承 HttpRequestHandler 并实现 handleRequest 方法。

```
public void handleRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        try {
            RemoteInvocation invocation = this.readRemoteInvocation(request);
            RemoteInvocationResult result = this.invokeAndCreateResult(invocation, this.getProxy());
            this.writeRemoteInvocationResult(request, response, result);
        } catch (ClassNotFoundException var5) {
            throw new NestedServletException("Class not found during deserialization", var5);
        }
    }
```

>> 首先从 http 请求中读取远程调用对象，然后调用服务对应方法并组织执行结果，最后将执行结果写入到 http 返回。   

>> 这几个过程你追溯到底层代码，你会发现 Java ObjectInputStream 反序列化对象、Java Method 反射对象。
   
   
## 参考链接
1. Dubbo HttpInvoker反序列化分析  https/img/dubbo//mp.weixin.qq.com/s/tdWrbQc5sGzdTqFLZ8prXA
2. Apache Dubbo反序列化漏洞（CVE-2019-17564）http/img/dubbo//www.lmxspace.com/2020/02/16/Apache-Dubbo%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E%EF%BC%88CVE-2019-17564%EF%BC%89/#%E6%BC%8F%E6%B4%9E%E4%BF%AE%E5%A4%8D
3. 浅析Dubbo HttpInvokerServiceExporter反序列化漏洞（CVE-2019-17564）https/img/dubbo//mp.weixin.qq.com/s/FU2sqIAVkN_6arXCWMRm7g


## TODO

1. JsonRpcBasicServer 的处理逻辑
2. 其他远程调用框架的调用逻辑