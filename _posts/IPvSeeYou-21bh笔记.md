---
layout: post
title: IPvSeeYou-21bh笔记
date: 2021-08-09
categories: 技术调研
excerpt: IPvSeeYou：通过IPv6地址中泄漏的标识进行街道级地理位置定位
tags: IPv6
---

* content
{:toc}

## 简述

在21年的blackhat会议上看到一个议题 `IPvSeeYou: Exploiting Leaked Identifiers in IPv6 for Street-Level Geolocation`，通过IPv6地址中泄漏的标识进行街道级地理位置定位，感觉很点意思，记录一下。

## 数据关联流程

1. IPv6 地址和 MAC 地址存在相关关系

![1628492391392](/img/IPvSeeYou/ca4f6742c3354f1bb35b3cb0ba7777d6.png)


2. WiFi 地理地址和 BSSID 存在相关关系 

![1628492450525](/img/IPvSeeYou/b65ef39abc884852b18326f807adf2de.png)

3. WAN MAC 地址 和 BSSID 存在一定相关关系

![1628492528407](/img/IPvSeeYou/dbc3af567750439482dd3c1df49de0a8.png)

通过这三个相关关系，将 IPv6地址和地理位置关联。

## IPv6 地址和 MAC 相关关系介绍

背景知识：

1. ipv6 地址长度为128位，最后64位地址称为接口ID，接口 ID 用来标识特定节点的接口。接口 ID 必须在子网内唯一。IPv6 主机可以使用相邻节点搜索协议自动生成其自身的接口 ID
2. MAC 地址：用于网络适配器的传统接口标识可使用称为 IEEE 802 地址的 48 位地址, 由 24 位公司 ID（也称为制造商 ID）和 24位扩展ID（也称为底板 ID）组成, 即可生成全局唯一的 48 位地址。这个 48 位地址也称为物理地址、硬件地址或媒体访问控制(MAC) 地址
3. EUI-64: IEEE EUI-64 地址代表网络接口寻址的新标准。公司 ID 仍然是 24 位长度，但扩展 ID 是 40 位，IEEE 802 地址创建 EUI-64 地址，则 16 位的 11111111 11111110 (0xFFFE) 将被插入到公司 ID 和扩展 ID 之间的 IEEE 802 地址中。EUI-64地址一般是唯一的
4. EUI-64的构造规则：根据 RFC2373，接口的MAC地址再加上固定的前缀来生成一个IPv6的地址（扫描发现，有超过 60M 的设备使用该规则）

实例：
* 2001:abcd:1234:fedc:fe75:16ff:fe01:2345
* 固定前缀 2001:abcd:1234:fedc
* mac地址 fe7516012345

**局限性**
* 不一定所有设备都是用 EUI-64
* 很多 icmp6 请求没有响应

## WiFi 地理位置数据库

背景知识

1. BSSID = WiFi interface MAC address
2. BSSID 地理位置映射数据库项目及数据库，超过 450M 的条目数据
    * wigle.net
    * www.mylnikov.net
    * openWifi.su
    * openBMap
    * gs-loc.apple.com
    * api.mylnikov.org

**局限性**

* 有的设备没有 BSSID  IOT 设备
* BSSID 没有在地理位置数据库中

##  WAN MAC 和 WiFi BSSID 映射关系

* 每个接口都有自己的`MAC`地址
* `MAC` 地址比较接近
* 设备分配的所有MAC地址由 厂商在一个偏移区间内

![1628496785231](/img/IPvSeeYou/f132383276a34446b1b877ffdf705fcf.png)

**局限性**
* 分配给有线和无线接口的mac地址不连续或在不同oui（组织唯一标志符）中
* 每个oui（组织唯一标志符）有多个偏移量
* 2.4/5 GHz BSSID使偏移推断复杂化

## 结果验证

* 通过数据融合算法
    * 定位了 60M 设备中的 12M
    * 分布 147 个国家
    * 超过 1000+ 不同的 oui（组织唯一标志符）
* 只用少部分真实数据做了验证，还可行
* 大数观察
    * MitraStar OUI 的设备显示出国家不一致性（mac地址区别）
    * 其他的 OUI 只在微小处有区别
    * Askey Corp OUI 在瑞士有很多
    * Swisscom 是一家主要的瑞士 ISP，提供 Askey 路由器作为其标准家用 WLAN 设备
* 可以推断 ISP 的覆盖区域，推断非EUI-64终端设备的大致位置