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

需要局域网不同(lan1, lan2)的两个用户(user1, user2)能够通过一个公网服务器互连(user3, server)，
 
user1@lan1

user2@server

### user1连接公网


```
user1@lan1:$ ssh -NfR 20002:localhost:22 user2@server
```

参数解释：

1. -N 不执行任何指令
2. -f 后台执行
3. -R 使用远程端口转发

N 和 R 通常共同使用

### 使用公私钥登录

使用 ```ssh-keygen```生成公私钥

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

 ```-M 5678```表示通过 5678端口监视连接状态，连接有问题就会自动重载，




## ssh 端口转发

### 本地转发(-L)

```
ssh -L [<local host>]:<local port>:<remote host>:<remote port> <SSH hostname>
举例: ssh -L <A>:<A_port>:<B>:<B_port> <c>
-g 可以指定它与其他机器共享本地端口转发
```

将本地端口的消息 通过 ```ssh hostname``` 转发到远程主机和端口，```local host /A``` 可以省略

这种情况适合A是局域网地址，B和C 都具有公网地址的情况



### 远程转发(-R)

命令说明

```
ssh -R <local port>:<remote host>:<remote port> <SSH hostname>
ssh -R 9090:host_B:port_B C
```

这样访问 C:9090就等同与本机访问B:port_B;(表示很多博客写的模棱两可,)

如果B设置为localhost就可以从外网访问本机的port_B, 在反向代理时经常使用

### 动态转发(-D)

```
ssh -D <local port> <SSH Server>
```

动态转发无需再指定远程主机及其端口。它们由通过 SOCKS协议 连接到本地主机端口的那个主机



