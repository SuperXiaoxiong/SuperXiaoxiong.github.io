---
layout: post
title:  记一次低版本禅道绕waf getshell
date: 2025-06-15
categories: 实战
excerpt: 记一次低版本禅道绕waf getshell。在针对某公司授权攻防项目时，外网有一禅道多年多次多支队伍都没能拿下来，此次缺乏有效突破点，无奈之下决定啃下这块硬骨头。waf策略较严格，利用多个php trick 绕过获得权限。
tags: php zentao
---

* content
{:toc}

## 简述

记一次低版本禅道绕waf getshell。在针对某公司授权攻防项目时，外网有一禅道多年多次多支队伍都没能拿下来，此次缺乏有效突破点，无奈之下决定啃下这块硬骨头。waf策略较严格，利用多个php trick 绕过获得权限。







## 低版本禅道绕waf

首先进行版本确认，企业版 3.3 版本

![3897326a63c2be7f10edec713693976e.png](/img/zentao/4405d8e5e4854fb0b8fcb4ccbba57d48.png)

首先得说明禅道及各种路径模式,  如果对禅道不熟悉的情况下，使用网上的poc快速检测响应为404 很容易认为漏洞不存在，其实是对应版本和路径模式不了解

### 不通版本访问路径说明

开源版本访问路径 `/zentao`  映射文件处理路径 `/opt/zbox/app/zentao/www/`

专业版 访问路径 `/pro`  映射文件处理路径 `/opt/zbox/app/zentaopro/www/`

企业版 访问路径 `/biz`  映射文件处理路径 `/opt/zbox/app/zentaoep/www/`

### 不同pathinfo配置说明

令人眼花缭乱的poc格式


在网上找了一些历史poc，长这样

```
/api-getModel-api-sql-sql=select+account,password+from+zt_user
```

仔细研究一下，发现禅道分静态访问和GET方式访问

可以在配置文件 `config/my.php` 中修改

```
$config->requestType = 'PATH_INFO'; 
```

也可配置都兼容 `PATH_INFO|GET`， 其中GET方式为常见的m=module&f=method形式传递模块和方法名，而PATH_INFO则是通过路径和分隔符的方式传递模块和方法名。路由方式及分隔符定义在config/config.php中，

其中，在目标版本的一键安装模式中，使用的是静态访问

```
/api-getModel-api-sql-sql=select+account,password+from+zt_user
```

url及参数介绍如下， 通过传参确定进入的control文件为api，对应的method为getModel，接着开始对参数进行赋值，其中moduleName为api，methodName=sql，最后的param为sql=select+account,password+from+zt_user

## 确认漏洞存在

历史：11.6 版本后台存在多个漏洞，授权用户可通过 module/api/control.php中getModel方法，调用module目录下所有的model模块和方法，从而实现SQL注入、任意文件读取、远程代码执行、文件包含等攻击

确认任意文件写，和任意文件包含两个漏洞存在

任意文件写：当前目录下写test文件。

![e6665e7d2f4b2fe9e1732260812fe04c.png](/img/zentao/0bac07d97e1042a19577c36c8773c984.png)

任意文件包含：包含test文件

![41a2f36cd2f91b61e186bae68d3b70bc.png](/img/zentao/e6b92e4c3a974c2a8b84605a4e2acfa8.png)

这不是轻轻松松getshell

boom，当上传字符出现 `<?` php的标签字符时，waf 直接阻拦提示 502
![85356c7d62a9e9ec1258a2f1181dd87c.png](/img/zentao/606213350ccb479380e2f22031088574.png)

### 有WAF，采用常规思路

小小WAF能拦住我

采用常规思路

* 分块传输 chunk 编码
* 垃圾字符填充
* url 编码

HTTP/1.1 502 Bad Gateway 响应岿然不动

## WAF策略非常严格

### 常规提权测试

如果可以确认环境，那么本地1比1模拟搭建，降低环境干扰

既然可以写文件，那我就先试试写文件的常规提权操作

这里有一个请求响应给了很大的误导

![cf2f86dd44db8cc00ec1a81e0e161688.png](/img/zentao/8d2d55f8ec4a4fc0972c3e1b2c3e8147.png)

可以将文件写到 `/tmp/` 目录，那必然台linux呀

* 定时任务
* .bashrc
* 公钥文件
* ......

全部报错无权限

禅道一键安装版本linux ，推荐目录是 /opt/zbox,
再尝试绝对路径、相对路径，全部返回无权限。

提示也是还是 chmod 777 -R xxxx，暗搓搓嘲讽，你没找对位置

![baef85243fe7878141130943f52709cb.png](/img/zentao/4a5f6fa9f1fe46ca92fce2e4c8c53c8e.png)

接近绝望时，在源码中找到一个奇怪命名的 `x.php` ，就想看眼到底啥作用，bingo！

是台windows 机器，安装目录在c盘，采用一键安装版本，。

![7bfe48b7c07393a09a9a3060feaa510d.png](/img/zentao/8df4f3932a024853abad8c2c735e3559.png)

### 禅道特性发掘

在确认目标环境后，可以在本地搭建环境，使用白盒角度观察

观察apache 配置 `etc/apache/http.conf`  文件，针对业务访问路径 `biz`，`pro`, `zentao` 配置一致

```
# setting for zentaoep.
Alias /biz "C:/zentao/xampp/zentaoep/www/"
<Directory "C:/zentao/xampp/zentaoep/www">
  Order deny,allow
  Allow from all
  AllowOverride All
  Require all granted
  
  <Files "index.php">
    SetHandler application/x-httpd-php
  </Files>
  .......

</Directory>
<DirectoryMatch "C:/zentao/xampp/zentaoep/www/.+/.*">
  <FilesMatch ".+\.ph(p[3457]?|t|tml)$">
    SetHandler text/plain
  </FilesMatch>
</DirectoryMatch>
```

关注重点  `.htaccess` 配置可以覆盖

```
Options FollowSymLinks：
AllowOverride All：
```

指定下列文件被当做php解析

```
  <Files "index.php">
    SetHandler application/x-httpd-php
  </Files>
指定了 index.php、upgrade.php、install.php、checktable.php、api.php、x.php 这几个文件被当作 PHP 文件解析。
```

其他php 文件只按照文本类型

```
<DirectoryMatch "/opt/zbox/app/zentaoep/www/.+/.*">
  <FilesMatch ".+\.ph(p[3457]?|t|tml)$">
    SetHandler text/plain
  </FilesMatch>
</DirectoryMatch>
```

### 挑战

从这里可以看到两个难点

1. 无法使用php标签  `<?`
2. 非指定php文件不会解析

### php 标签绕过

#### php 标签选择

首先需要在标签处进行绕过，先了解背景知识，有哪些php 标签, 是否能使用其他的php标签

1. 标准标签（Standard Tags）

```
<?php ... ?>
```

2. 短标签（Short Tags）

```
<? ... ?>
```

需在 php.ini 中开启 short_open_tag=On（PHP 5.3 及之前默认关闭，PHP 5.4+ 默认开启部分功能）

3. ASP 风格标签（ASP-style Tags）

```
ASP 风格标签（ASP-style Tags）
```

* 使用条件：
  * 需在 php.ini 中开启 asp_tags=On
  * PHP 7.0 起已完全移除（不再支持）

5. Script 标签（Script Tags）

```
<script language="php"> ... </script>
```

* 使用条件
  * PHP 7.0 起已移除（不再支持）

本地搭建一键安装版，可以看到php 版本为7.0.19 ，无法使用其他标签

![18bc664925f8eb983b0a50ca797a9e20.png](/img/zentao/a15dea28994d426faebdf6b4bf07db0e.png)

#### 编码绕过

可以看到，无其他可选标签，那还有一个办法，编码绕过。

查看php 资料，是否支持其他编码解析

![1681feb5661a424335a0e238446b4fa1.png](/img/zentao/05aaff73ac294a0999b723d0661d7621.png)

关注 `zend.multibyte` 配置,在配置 multibyte 为true 后，可以设置 `script_encoding `编码

编码可选为 支持多种编码

zend.script_encoding 支持以下编码（取决于 PHP 版本和 mbstring 扩展）：

* "UTF-8"
* "SJIS"（Shift-JIS，日文编码）
* "EUC-JP"（日文编码）
* "ASCII"
* "UTF-7"
  其他 mbstring 支持的编码（如 "GBK"、"BIG5" 等，实际支持有限）

`<` 属于ASCII安全字符，在 Python标准库、RFC 2152标准下，`<` 和 `?` 都是原样输出

只有在微软/Outlook/PHP mbstring等“变体”UTF-7实现下，< 可能会被编码为 `+ADw-`，> 为 `+AD4-`,  php script_encoding 就是使用 mbstring 实现，在这里可以绕过 waf 对于 `<?` 绕过

如何修改特定的编码, 回到 apache 中重点关注配置,  htaccess 配置可以覆盖

```
Options FollowSymLinks：
AllowOverride All：
```

只要在子目录htaccess 中进行配置，配置环境变量，使php 可以使用utf-7编码解析，绕过waf 检测

```
php_value zend.multibyte 1
php_value zend.script_encoding "UTF-7"
```

只要配置中有 .htaccess 就会 502

![186e7aab4f51c96ec6aed19aebd88363.png](/img/zentao/327e7fb277464b96b799e0a436d3b9b3.png)

#### 挑战1

apache 中配置 AccessFileName 来替代 .htaccess, 这里使用 ztaccess ,刚好绕过 waf

```
# security.
AccessFileName .ztaccess
```

#### 挑战2

该配置会对当前目录及子目录解析都造成影响，使php 都只能使用utf-7 编码解析，影响正常功能使用

```
php_value zend.multibyte 1
php_value zend.script_encoding "UTF-7"
```

找到禅道根htdocs web路径

```
C:\zentao\xampp\htdocs
```

1. 其中只有一个index.php 关键起效代码使 js 请求跳转到 /biz/ 目录
2. 和项目代码属于两个目录，不影响关键代码运行，可以通过 `.ztaccess` 重复覆盖保证不会影响正常功能使用

在这里定义了最后上webshell 策略

1. 覆盖子目录 .ztaccess,修改解析编码为utf-7
2. 绕过waf，写入utf-7小马
3. 访问小马：落地正常webshell
4. 再次覆盖 .ztaccess, 修改编码为utf-8
5. 连接webshell

#### 挑战3

apache 配置中限定 其他目录下文件只能以txt进行解析

```
<DirectoryMatch "/opt/zbox/app/zentaoep/www/.+/.*">
  <FilesMatch ".+\.ph(p[3457]?|t|tml)$">
    SetHandler text/plain
  </FilesMatch>
</DirectoryMatch>
```

使用 htaceess 覆盖策略

```
<Files "3.php">
    SetHandler application/x-httpd-php
</Files>
```

## webshell 落地上传截图

写入 .ztaccess

![fd2e0c0d7462c31f95238556a0a12cf5.png](/img/zentao/4fd30052c15b4b3fbc443d1832f8ba9e.png)

写webshell
![ded43fe2e9c731dc83b287a0e18bba8b.png](/img/zentao/a28cbb1f5ef545618dae77ec6b923d03.png)

webshell 上线
![4c72bf34b313e37349bf7e96bc4e3b62.png](/img/zentao/cc55a4e1c089414981ce605ba39c5fed.png)
