---
layout: page
title: About
permalink: /about/
icon: heart
---
* content
  {:toc}

## 关于我

漏洞分析研究员

 java | php | python

* CVE-2024-9945
  * Fortra GoAnywhere MFT 是一种基于Web 的托管文件传输工具，Fortra 是一家知名的网络安全公司
  * 有限制任意文件读
  * https://www.fortra.com/security/advisories/product-security/fi-2024-014
  * https://www.cve.org/cverecord?id=CVE-2024-9945
* CVE-2024-41169
  * Apache Zeppelin是一款基于Web的Notebook产品，能够交互式数据分析。目前是 `Apache Software Foundation`的顶级项目。
  * 未授权任意文件读导致远程代码执行.
* CVE-2024-29007
  * Apache CloudStack 是一个开源的具有高可用性及扩展性的云计算平台。在 2011 年被 Citrix 以超过二亿美金价格收购，随后又被捐献给 Apache 基金会。目前是 `Apache Software Foundation`的顶级项目
  * Apache CloudStack ssrf 中危
  * https://lists.apache.org/thread/82f46pv7mvh95ybto5hn8wlo6g8jhjvp
* CVE-2024-29006
  * Apache CloudStack xff 伪造 中危
  * https://lists.apache.org/thread/82f46pv7mvh95ybto5hn8wlo6g8jhjvp
* CVE-2023-25573
  * metersphere 是一个很流行的开源测试平台，隶属于杭州飞致云信息科技有限公司。
  * 未授权任意文件下载，在默认环境部署环境下是可以直接 rce 的。
  * https://github.com/metersphere/metersphere/security/advisories/GHSA-mcwr-j9vm-5g8h
  * 还有致谢 https://mp.weixin.qq.com/s/8O4Bn0n1EE8VMBJfreF2OQ
* CVE-2023-28637
  * dataease 是一款开源数据可视化工具，隶属于杭州飞致云信息科技有限公司。
  * 普通权限用户rce
  * https://github.com/dataease/dataease/security/advisories/GHSA-8wg2-9gwc-5fx2
* CVE-2023-27025
  * 若依的一个后台任意文件下载，一个使用量超级高的开源CMS。
  * 在看历史洞的时候发现了这个绕过。能看到随着代码迭代，社区努力，安全性越来越高
  * https://github.com/advisories/GHSA-h4c9-rr5m-32fm
  * https://gitee.com/y_project/RuoYi/issues/I697Q5#note_16046476
* CVE-2022-24832
  * GoCD是一个开源的持续集成和持续交付系统，可以在持续交付过程中执行编译、自动化测试、自动部署等等。是 Thoughtworks 全球软件咨询公司 出品
  * 在使用ldap插件认证时，会导致 ldap 注入。挺有技巧性的一个漏洞，在具有一个ldap账户的前提下，可以渗出有效信息
  * https://github.com/gocd/gocd/security/advisories/GHSA-x5v3-x9qj-mh3h
* opera 2020 致谢
  * opera 浏览器地址栏欺骗
  * https://security.opera.com/en/hall-of-fame/
* CVE-2020-15651
  * unicode 字符控制字符视觉欺骗漏洞，可以调整火狐浏览器下载时的后缀名
  * [https://bugzilla.mozilla.org/show_bug.cgi?id=1649160](https://bugzilla.mozilla.org/show_bug.cgi?id=1649160)
* CVE-2018-8026
  * apache solr 是Apache Lucene项目的开源企业搜索平台
  * xxe 漏洞
  * [https://issues.apache.org/jira/browse/SOLR-12450](https://issues.apache.org/jira/browse/SOLR-12450)
  * [https://mail-archives.apache.org/mod_mbox/lucene-solr-user/201807.mbox/%3C0cdc01d413b7%24f97ba580%24ec72f080%24%40apache.org%3E](https://mail-archives.apache.org/mod_mbox/lucene-solr-user/201807.mbox/%3C0cdc01d413b7%24f97ba580%24ec72f080%24%40apache.org%3E)

## 联系我

* GitHub：[https://github.com/SuperXiaoxiong](https://github.com/SuperXiaoxiong)
* email：superxyyang@gmail.com

## 友情链接

[neargle](http://blog.neargle.com/)

## Comments

{% include comments.html %}
