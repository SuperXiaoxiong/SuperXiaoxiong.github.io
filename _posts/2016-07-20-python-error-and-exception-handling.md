---
layout: post
title:  "错误和异常-python核心编程笔记"
date:   2016-07-20 16:14:54
categories: python
tags:  python error exception try-except
---

* content
{:toc}

## 简述

掌握一门语言，精通一种技术，才是个人竞争的核心价值。

本篇是对python核心编程第二版中，python错误和异常处理的总结，python采用"try/尝试"块和"catching/捕获"块来处理异常





## 异常处理语句

这是一个完整的try-except(-else)(-finally)语句块

```
try:
	try_suite 							# watch for exceptions here 监控这里的异常
except Exception1[, reason]:
	suite_for_Exception1 				# exception-handling code 异常处理代码
except Exception2[, reason2]:						# 可以连接多个异常，处理一个try块发生的多种异常
	suite_for_exception_Exception2
except (Exc3[, Exc4[, ... ExcN]])[, reason]:		# 处理多个异常的 except 语句，放在一个元祖中
	suite_for_exceptions_Exc3_to_ExcN
else：
	no_exceptions_detected_suite					# 在try范围内没有异常被检测到，执行else子句
finally：
	finally_suite 						#无论如何都执行
```

## 异常参数

标准内建异常至少提供一个参数，指示异常原因的一个字符串

reason 只包含一个字符串或是由错误编号和字符串组成的元组, 调用 str(reason) 总会返回一个良好可读的错误原因.
 
reason 是一个类实例 - 这样做你其实是调用类的特殊方法 __str__() .

为了获得更多的关于异常的信息,我们可以调用该实例的 __class__ 属性,它标示了实例是从什么类实例化而来. 类对象也有属性, 比如文档字符串(documentation string)reason.__class__.__doc__和进一步阐明错误类型的名称字符串reason.__class__.__name__

## 触发异常

程序员希望在遇到错误的输入时触发异常

```
raise [SomeException [, args [, traceback]]]
```

SomeExcpetion,是触发异常的名字.如果有,它必须是一个字符串,类或实例

第二个符号为可选的 args(比如参数,值),来传给异常,可能是字符串来指示错误的原因，如果是元祖，通常的组成是一个错误字符串,一个错误编号,可能还有一个错误的地址,比如文件。

第三个traceback 则是当异常触发时新生成的一个用于异常-正常化(exception—normally)的追踪(traceback)对象

raise 语法

```
raise exclass 					# 触发一个异常,从 exclass 生成一个实例(不含任何异常参数)
raise exclass() 				# 同上,除了现在不是类;通过函数调用操作符(function calloperator:"()")作用于类名生成一个新的 exclass 实例,同样也没有异常参数
raise exclass, args 			#同上,但同时提供的异常参数 args,可以是一个参数也可以元组
raise exclass(args) 			#同上
raise exclass,args, tb 			#同上,但提供一个追踪(traceback)对象 tb 供使用
raise exclass,instance 			#通过实例触发异常(通常是 exclass 的实例);如果实例是 exclass的子类实例,那么这个新异常的类型会是子类的类型(而不是exclass);如果实例既不是 exclass 的实例也不是 exclass 子类的实例,那么会复制此实例为异常参数去生成一个新的 exclass 实例.
raise instance 					#通过实例触发异常:异常类型是实例的类型;等价于raise
instance.__class__, instance (同上).
raise string 					#(过时的) 触发字符串异常
raise string, args 				#同上,但触发伴随着 args
raise string, args, tb 			#同上,但提供了一个追踪(traceback)对象 tb 供使用
raise 							#(1.5 新增)重新触发前一个异常,如果之前没有异常,触发 TypeError.
```

## 断言

断言语句等于断言成功不采取任何措施，否则触发断言错误AssertionError的异常。

```
assert expression[, arguments]
#例如
assert 1 == 1
```

AssertionError 异常和其他的异常一样可以用 try-except 语句块捕捉,但是如果没有捕捉,它将终止程序运行而且提供一个如下的 traceback:

```
>>> assert 1 == 0 ,'学习断言语句'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AssertionError: 学习断言语句
```

用try/exception的断言语句

```
>>> try:
...     assert 1== 0 , '学习断言语句'
... except AssertionError, diag:
...     print '%s : %s' % (diag.__class__.__name__,diag)
...
AssertionError : 学习断言语句
```

##标准异常

python当前的异常标准集

```
BaseException　　　　　　　　　　　　　　所有异常基类
 +-- SystemExit　　　　　　　　　　　　　　　　python解释器请求退出
 +-- KeyboardInterrupt　　　　　　　　　　　　用户中断执行(通常是输入ctrl+C)
 +-- GeneratorExit　　　　　　　　　　　　　　生成器(generator)发生异常来通知退出　
 +-- Exception　　　　　　　　　　　　　　　　常规错误的基类
      +-- StopIteration　　　　　　　　　　　迭代器没有更多的值
      +-- StandardError　　　　　　　　　　　所有的内建标准异常的基类
      |    +-- BufferError　　　　　　　　　
      |    +-- ArithmeticError　　　　　　　所有数值计算错误的基类
      |    |    +-- FloatingPointError　　 浮点计算错误
      |    |    +-- OverflowError　　　　　数值运算超出最大限制
      |    |    +-- ZeroDivisionError　　　除(或取模)零(所有数据类型)
      |    +-- AssertionError　　　　　　　断言语句失败
      |    +-- AttributeError　　　　　　　 对象没有这个属性
      |    +-- EnvironmentError　　　　　  操作系统错误的基类
      |    |    +-- IOError　　　　　　　   输入输出失败
      |    |    +-- OSError　　　　　　　   操作系统错误　
      |    |         +-- WindowsError (Windows)　　　　windows系统调用失败
      |    |         +-- VMSError (VMS)　　　　　　　　　
      |    +-- EOFError　　　　　　　　　　没有内建输入,到达EOF标记　　　　　　　　　　
      |    +-- ImportError　　　　　　　　导入模块/对象 失败
      |    +-- LookupError　　　　　　　　无效数据查询的基类
      |    |    +-- IndexError　　　　　　序列中没有此索引(index)
      |    |    +-- KeyError　　　　　　　映射中没有这个键
      |    +-- MemoryError　　　　　　　　内存溢出错误(对于Python解释器不是致命的)
      |    +-- NameError　　　　　　　　　未声明/初始化对象(没有属性)
      |    |    +-- UnboundLocalError　　访问未初始化的本地变量
      |    +-- ReferenceError　　　　　　 弱引用(Weak reference)试图访问已经垃圾回收了的对象
      |    +-- RuntimeError　　　　　　　　一般的运行时错误
      |    |    +-- NotImplementedError　尚未实现的方法
      |    +-- SyntaxError　　　　　　　　 Pythony语法错误
      |    |    +-- IndentationError　　　缩进错误

      |    |         +-- TabError　　　　　Tab和空格混用
      |    +-- SystemError　　　　　　　　　一般的解释器系统错误
      |    +-- TypeError　　　　　　　　　　对类型无效的操作
      |    +-- ValueError　　　　　　　　　传入无效的参数
      |         +-- UnicodeError　　　　　Unicode相关错误
      |              +-- UnicodeDecodeError　　　　Unicode解码时错误
      |              +-- UnicodeEncodeError　　　　Unicode编码时错误
      |              +-- UnicodeTranslateError　　 Unicode转换时错误
      +-- Warning　　　　　　　　　　　　　　　警告的基类　　　　　　　
           +-- DeprecationWarning　　　　　　关于被启用的特征的警告
           +-- PendingDeprecationWarning　　关于构造将来语义会有改变的警告
           +-- RuntimeWarning 　　　　　　　 可疑的运行时行为的警告
         +-- SyntaxWarning　　　　　　　可疑的语言的警告　　
           +-- UserWarning　　　　　　　　用户代码生成警告
           +-- FutureWarning
	   +-- ImportWarning　　　　　　导入模块/对象警告
	   +-- UnicodeWarning　　　　　　Unicode警告
	   +-- BytesWarning　　　　　　　Bytes警告
　　　　　　 +-- Overflow Warning　　　　　　旧的关于自动提升为长整型(long)的警告
 
```

每一层缩进都代表一次异常类的派生.

允许如 except Exception 的语句捕获所有非控制程序退出的异常.

## 创建异常

创建新的异常类型来命名自己的异常，直接或者间接从Exception类派生

```
>>> class MyError(Exception):
...     def __init__(self, value):
...             self.value=value
...     def __str__(str):
...             return repr(self.value)
...
>>> try:
...     raise MyError('这儿有错误')
... except MyError as e:
...     print 'My exception occurred, %s' % (e.value)
...
My exception occurred, 这儿有错误
```

在这个例子中，Exception 默认的 init() 被覆盖。新的方式简单的创建 value 属性。这就替换了原来创建 args 属性的方式。

## sys模块

从sys模块中exc_info()中获取异常信息，此功能提供了一个3元组

```
>>> try:
...     float('happy')
... except:
...     import sys
...     exc_tuple = sys.exc_info()
...
>>> print exc_tuple
(<type 'exceptions.ValueError'>, ValueError('could not convert string to float: happy',), <traceback object at 0x0000000002483648>)
>>>
>>> for item in exc_tuple:
...     print item
...
<type 'exceptions.ValueError'>
could not convert string to float: happy
<traceback object at 0x0000000002483648>
```

我们从 sys.exc_info()得到的元组中是:

* exc_type: 异常类
* exc_value: 异常类的实例
* exc_traceback: 追踪(traceback)对象

