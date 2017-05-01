---
layout: post
title:  python 高阶函数
date: 2017-05-01 15:47
categories: python
tags:  python
---

* content
{:toc}

## 问题描述

对高阶函数，闭包，装饰器做详解






## 高阶函数


### 定义：

高阶函数:能接收函数做参数的函数
 
* 变量可以指向函数
* 函数的参数可以接收变量
* 一个函数可以接收另一个函数作为参数
* 能够接收函数作参数的函数就是高阶函数

```
#定义能够接收函数的高阶函数
#取x,y绝对值
#将abs()作为参数带入
def add(x,y,f):
	return f(x) + f(y)
	
add(-5,9,abs)
```

### map()

 ```map()```是```Python```内置的高阶函数，接收一个函数```f```,和一个```list```,通过函数依次作用在```list```每个元素上，得到新的```list```并返回

```
#请利用map()函数，把一个list（包含若干不规范的英文名字）变成一个包含规范英文名字的list：
#输入：['adam', 'LISA', 'barT']
#输出：['Adam', 'Lisa', 'Bart']

list = ['adam','LISA','barT']
def fun_align(str_in):
    return  str_in.title()
    
print map(fun_align,list)
```



```
python string 大小写转换
s = 'hEllo pYthon'
>>> print s.upper()         #全大写
HELLO PYTHON
>>> print s.lower()         #全小写	
hello python	
>>> print s.capitalize()    #首字母大写其余全小写
Hello python
>>> print s.title()         #所有单词首字母大写,其余小写
Hello Python
```

### reduce()

 ```reduce()```函数接收的参数和```map()```类似，一个函数```f```，一个```list```，但行为和```map()```不同，```reduce()```传入的函数```f```必须接收两个参数,```reduce()```对```list```的每个元素反复调用函数```f```，并返回最终结果值。

```
#利用recude()来求积
list = [2,4,5,7,12]
def f(x,y):
    return x*y
print reduce(f,list,1)
#第三个可选参数 是计算的初始值
```

###  filter()

 ```filter()```函数接收一个函数```f```和一个```list```，这个函数```f```的作用是对每个元素进行判断，返回```True```或```False```,```filter()```根据判断结果自动过滤掉不符合条件的元素，返回由符合条件元素组成的新```list```。

例如，要从一个list [1, 4, 6, 7, 9, 12, 17]中删除偶数，保留奇数，首先，要编写一个判断奇数的函数：

```
def is_odd(x):
    return x % 2 == 1
#然后，利用filter()过滤掉偶数：

filter(is_odd, [1, 4, 6, 7, 9, 12, 17])
#结果：[1, 7, 9, 17] 
``` 

```
#请利用filter()过滤出1~100中平方根是整数的数，
import math
def is_sqrt(x):
    temp = int(math.sqrt(x))
    return temp * temp == x
    
print filter(is_sqrt,xrange(1,101))
```

## 匿名函数lambda

 ```Python```中匿名函数```lambda```,关键字```lambda```表示匿名函数，只能有一个表达式，不写```return```，返回值就是该表达式的结果
 
```
#lambda求绝对值
myabs = lambda x: -x if x < 0 else x

#lambda求积
lambda x: x * x 

#相等于
def f(x):
    return x * x
``` 

在其他高阶函数中使用lambda

```
def is_not_empty(s):
    return s and len(s.strip()) > 0
filter(is_not_empty, ['test', None, '', 'str', '  ', 'END'])
#用匿名函数简化

print filter(lambda s:s and len(s.strip()) > 0 , ['test', None, '', 'str', '  ', 'END'])
```

## 装饰器

### 返回函数

函数不仅可以返回int,str,list,dict等数据类型，还可以返回函数

定义函数f(),返回函数g

```
def f():
    print 'call f()...'
    # 定义函数g:
    def g():
        print 'call g()...'
    # 返回函数g:
    return g
	
#调用函数f,得到f的返回函数
>>> x = f()   # 调用f()
call f()...
>>> x   # 变量x是f()返回的函数：
<function g at 0x1037bf320>
>>> x()   # x指向函数，因此可以调用
call g()...   # 调用x()就是执行g()函数定义的代码
```


编写一个函数calc_prod(lst)，它接收一个list，返回一个函数，返回函数可以计算参数的乘积。

```
def calc_prod(lst):
    def cal_g():
        return reduce((lambda x,y: x*y),lst)
    return cal_g

f = calc_prod([1, 2, 3, 4])
print f()
```

### 闭包

像这种内层函数引用了外层函数的变量（参数也算变量），然后返回内层函数的情况，称为闭包（Closure）。

闭包的特点是返回的函数还引用了外层函数的局部变量，所以，要正确使用闭包，就要确保引用的局部变量在函数返回后不能变。举例如下：

```
# 希望一次返回3个函数，分别计算1x1,2x2,3x3:
def count():
    fs = []
    for i in range(1, 4):
        def f():
             return i*i
        fs.append(f)
    return fs

f1, f2, f3 = count()
```

实际结果全部都是 9（请自己动手验证）。

原因就是当count()函数返回了3个函数时，这3个函数所引用的变量 i 的值已经变成了3。由于f1、f2、f3并没有被调用，所以，此时他们并未计算 i*i，当 f1 被调用时为9

```
def count():
    fs = []
    for i in range(1, 4):
        def f(m = i):
            return m*m
        fs.append(f)
    return fs

f1, f2, f3 = count()
print f1(), f2(), f3()
```

方法：问题的产生是因为函数只在执行时才去获取外层参数i，若函数定义时可以获取到i，问题便可解决。而默认参数正好可以完成定义时获取i值且运行函数时无需参数输入的功能，所以在函数f()定义中改为f(m = i),函数f返回值改为m*m即可.


### 装饰器(无参数)

Python的 decorator 本质上就是一个高阶函数，它接收一个函数作为参数，然后，返回一个新函数。

使用 decorator 用Python提供的 @ 语法，这样可以避免手动编写 f = decorate(f) 这样的代码。

```
#编写一个@performance，它可以打印出函数调用的时间
import time

def performance(f):
    def fn(*args, **kw):
        print 'call ' + f.__name__ + '() in ' + str(time.time())
        return f(*args, **kw)
    return fn

@performance
def factorial(n):
    return reduce(lambda x,y: x*y, range(1, n+1))

print factorial(10)
```

### 装饰器(带参数)

对于 一个简单的日志系统来说

```
def log(f):
    def fn(x):
        print 'call ' + f.__name__ + '()...'
        return f(x)
    return fn
	
@log
def factorial(n):
    return reduce(lambda x,y: x*y, range(1, n+1))
print factorial(10)
```

log 打印的语句不变，希望打印出'[INFO] call xxx()...'，有的函数不太重要，希望打印出'[DEBUG] call xxx()...'，这时，log函数本身就需要传入'INFO'或'DEBUG'这样的参数，类似这样：

```
@log('DEBUG')
def my_func():
    pass
```

使用高阶函数的调用

```
my_func = log('DEBUG')(my_func)
```

再展开

```
log_decorator = log('DEBUG')
my_func = log_decorator(my_func)
```

语句相当于

```
log_decorator = log('DEBUG')
@log_decorator
def my_func():
    pass
```

所以，带参数的log函数首先返回一个decorator函数，再让这个decorator函数接收my_func并返回新函数：

```
def log(prefix):
    def log_decorator(f):
        def wrapper(*args, **kw):
            print '[%s] %s()...' % (prefix, f.__name__)
            return f(*args, **kw)
        return wrapper
    return log_decorator

@log('DEBUG')
def test():
    pass
print test()
```

### 完善decorator

```
def log(f):
    def wrapper(*args, **kw):
        print 'call...'
        return f(*args, **kw)
    return wrapper
@log
def f2(x):
    pass
print f2.__name__
```

输出： wrapper

可见，由于decorator返回的新函数函数名已经不是'f2'，而是@log内部定义的'wrapper'。这对于那些依赖函数名的代码就会失效。decorator还改变了函数的__doc__等其它属性。如果要让调用者看不出一个函数经过了@decorator的“改造”，就需要把原函数的一些属性复制到新函数中

```
def log(f):
    def wrapper(*args, **kw):
        print 'call...'
        return f(*args, **kw)
    wrapper.__name__ = f.__name__
    wrapper.__doc__ = f.__doc__
    return wrapper
```

这样写decorator很不方便，因为我们也很难把原函数的所有必要属性都一个一个复制到新函数上，所以Python内置的functools可以用来自动化完成这个“复制”的任务：

```
import functools
def log(f):
    @functools.wraps(f)
    def wrapper(*args, **kw):
        print 'call...'
        return f(*args, **kw)
    return wrapper
```

## 参考

参考imooc python进阶


