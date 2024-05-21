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

* CVE-2024-29007
  * Apache CloudStack ssrf 中危
  * https://lists.apache.org/thread/82f46pv7mvh95ybto5hn8wlo6g8jhjvp
* CVE-2024-29006
  * Apache CloudStack xff 伪造 中危
  * https://lists.apache.org/thread/82f46pv7mvh95ybto5hn8wlo6g8jhjvp
* CVE-2023-25573
  * metersphere 是一个很流行的开源测试平台。未授权任意文件下载，实际上是可以直接 rce 的。
  * https://github.com/metersphere/metersphere/security/advisories/GHSA-mcwr-j9vm-5g8h
  * 还有致谢 https://mp.weixin.qq.com/s/8O4Bn0n1EE8VMBJfreF2OQ
* CVE-2023-28637
  * dataease 是一款开源数据可视化工具。普通权限用户rce
  * https://github.com/dataease/dataease/security/advisories/GHSA-8wg2-9gwc-5fx2
* CVE-2023-27025
  * 若依的一个后台任意文件下载，在看历史洞的时候发现了这个绕过。能看到随着代码迭代，社区努力，安全性越来越高
  * https://github.com/advisories/GHSA-h4c9-rr5m-32fm
  * https://gitee.com/y_project/RuoYi/issues/I697Q5#note_16046476
* CVE-2022-24832
  * 在使用ldap插件认证时，会导致 ldap 注入。还挺有意思的，在具有一个ldap账户的前提下，可以渗出有效信息
  * https://github.com/gocd/gocd/security/advisories/GHSA-x5v3-x9qj-mh3h
* opera 2020 致谢
  * https://security.opera.com/en/hall-of-fame/
* CVE-2020-15651
  * [https://bugzilla.mozilla.org/show_bug.cgi?id=1649160](https://bugzilla.mozilla.org/show_bug.cgi?id=1649160)
* CVE-2018-8026
  * [https://issues.apache.org/jira/browse/SOLR-12450](https://issues.apache.org/jira/browse/SOLR-12450)
  * [https://mail-archives.apache.org/mod_mbox/lucene-solr-user/201807.mbox/%3C0cdc01d413b7%24f97ba580%24ec72f080%24%40apache.org%3E](https://mail-archives.apache.org/mod_mbox/lucene-solr-user/201807.mbox/%3C0cdc01d413b7%24f97ba580%24ec72f080%24%40apache.org%3E)

## 联系我

* GitHub：[https://github.com/SuperXiaoxiong](https://github.com/SuperXiaoxiong)
* email：superxyyang@gmail.com

## 友情链接

[neargle](http://blog.neargle.com/)

## Comments

{% include comments.html %}
