---
layout: post
title:  ssh 反向代理配置
date: 2016-12-24 16:17:23
categories: ssh
tags:  ssh proxy 
---

* content
{:toc}

## 问题描述

学习笔记记录,配置ssh反向代理,远程登录局域网主机






## 反向代理建立流程


### 背景说明：

需要从不同局域网访问局域网lan1中的用户user1，通过一个公网服务器进行流量转发user2@server
 
### user1反向连接

![反向连接代理图](https://raw.githubusercontent.com/SuperXiaoxiong/SuperXiaoxiong.github.io/master/img/SSH_R2.PNG)

```
user1@lan1:$ ssh -NfR 20002:localhost:22 user2@server
```

这样访问user2@server:20002 就相当于访问user1@lan1:22

参数解释：

1. -N 不执行任何指令
2. -f 后台执行
3. -R 使用远程端口转发

N 和 R 通常共同使用

### 使用公私钥登录


使用```ssh-keygen```生成公私钥

复制 内网主机上的公钥到外网主机

待解决 一直permission denied

```
~/.ssh/ 文件夹 权限700 
~/.ssh/authorized_keys 权限600
sudo vi /etc/ssh/sshd_config
RSAAuthentication yes  
PubkeyAuthentication yes  
AuthorizedKeysFile     .ssh/authorized_keys 
seLinus 关闭
```

### autossh 自动连接

```
autossh -M 5678 -NfR 20001:localshost:22 user2@server
```

参数 ```-M 5678```表示通过 5678端口监视连接状态，连接有问题就会自动重载，

## ssh 端口转发

### 本地转发(-L)

![本地转发图](https://raw.githubusercontent.com/SuperXiaoxiong/SuperXiaoxiong.github.io/master/img/SSH_L.PNG)

命令说明，在host_A上执行

```
ssh -L <local port>:<remote host>:<remote port> <SSH hostname>
举例: ssh -L <A_port>:<C>:<C_port> <B>
-g 可以指定它与其他机器共享本地端口转发
```

访问A_port,将流量通过B转发到C:C_port,当把C设置成```localhost```表示表示访问B:B_port;然后流量再原路返回。

B为SSH Server，A为Client，所以A必须可以访问到B。


### 远程转发(-R)

![远程转发图](https://raw.githubusercontent.com/SuperXiaoxiong/SuperXiaoxiong.github.io/master/img/SSH_R.PNG)

访问C:C_port,经过B转发，可以访问到A:A_port

命令说明,在host_A上执行

```
ssh -R <remote port>:<destinative host>:<destinative port> <SSH hostname>
ssh -R 9090:host_C:port_C B
```

B监听9090端口,这样访问 B:9090就等同通过A访问C:port_C

如果C设置为localhost就等同访问A:port_A,在反向代理时经常使用

### 动态转发(-D)

```
ssh -D <local port> <SSH Server>
ssh -qTfnN -D 7070 -p 22 user@host
-D 绑定本地端口
-p 远程服务器的端口
-q Quiet mode，ssh的诊断信息以及警告等信息被抑制
-T 禁用伪终端分配
-f 该选项是后台执行的ssh在规定时间(10秒)内进行连接，如果超过该时间ssh将退出。
-n 重定向标准输入到/dev/null中，为了防止从标准输入中读入。ssh进行后台执行时必须使用该参数。该选项对于要求输入密码不起作用
-N 不执行远程命令

```

在client端上执行命令，并在浏览器出配置SOCKS代理

动态转发无需再指定远程主机及其端口。它们由通过 SOCKS协议 连接到本地主机端口的那个主机


## 参考

* [ibm实战 SSH 端口转发](https://www.ibm.com/developerworks/cn/linux/l-cn-sshforward/index.html)
