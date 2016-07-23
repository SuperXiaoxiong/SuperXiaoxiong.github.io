---
layout: post
title:  "面向对象-python核心编程笔记"
date:   2016-07-22 12:33:04
categories: python
tags:  python OOP 
---

* content
{:toc}

## 简述

记录阅读python核心编程第二版的笔记，本篇是对python面向对象的总结




### 介绍

核心笔记：类名以大写字幕开头。数据值使用名词作为名字，方法使用动词，python推荐使用骆驼记法的下划线方式，比如update_email,类名：AddrBookEntry.

类声明和函数声明相似。两者都允许在声明中创建函数，闭包或者内部函数

```
def function(args):
	'fuction document string'		#函数文档字符串
	function_suite		#函数体

class ClassNmae(object):
	'class document string'		#类文档字符串
	class_suite		#类体
```

类通常在一个模块的顶层定义，以便类实例能够在类所定义的源代码中任意地方被创建。

对于python来说，声明和定义类没什么区别，总是同时进行的。

基类是一个或多个用于继承的父类的集合；类体由所有的声明语句，类成员定义，数据属性和函数构成。

### 类属性

#### 数据类型
数据属性仅仅是定义的类的变量，即静态变量，表示这些数据与他们所属的类对象绑定，不依赖任何类实例，相当于Java和C++中变量前添加static

```
>>> class C(object):
...     foo = 100
...
>>> print C.foo
100
>>> C.foo = C.foo + 1
>>> print C.foo
101
```

#### 绑定方法

绑定方法指的是方法只有在其所属的类拥有实例时，才能被调用，任何一个绑定方法的第一个参数变量都是self，表示调用此方法的实例

self 在每一个方法声明中都是作为第一个参数传递的。当你在实例中调用一个绑定的方法时，self 不需要明确地传入了

#### 非绑定方法

调用非绑定方法一个常见的场景是：你在派生一个之类，想要覆盖父类的方法，但是需要调用父类中已有的构造方法。

```
class EmplAddrBookEntry(AddrBookEntry):
	'Employee Address Book Entry class' # 员工地址记录条目
	def __init__(self, nm, ph, em):
		AddrBookEntry.__init__(self, nm, ph)
		self.empid = id
		self.email = em
```

我们将在子类构造器中调用父类的构造器并且明确地传递（父类）构造器所需要的 self 参数（因为我们没有一个父类的实例）。子类中 ```__init__()``` 的第一行就是对父类```__init__()```的调用。我们通过父类名来调用它，并且传递给它 self 和其他所需要的参数。一旦调用返回，我们就能定义那些与父类不同的仅存在我们的（子）类中的（实例）定制.

#### 特殊的类属性

```
C.__name__ 		类Ｃ的名字（字符串）
C.__doc__ 		类Ｃ的文档字符串
C.__bases__ 		类Ｃ的所有父类构成的元组
C.__dict__ 		类Ｃ的属性
C.__module__ 		类Ｃ定义所在的模块（1.5 版本新增）
C.__class__ 		实例Ｃ对应的类（仅新式类中）
```

！！！注意 ```__doc__```是类的文档字符串，不能被派生类继承。

```
>>> class MyClass(object):
...     'MyClass class definition'
...     myVersion='1.1'
...     def showMyVersion(self):
...             print MyClass.myVersion
...
>>> MyClass.__name__
'MyClass'
>>> MyClass.__doc__
'MyClass class definition'
>>> MyClass.__bases__
(<type 'object'>,)
>>> MyClass.__dict__
dict_proxy({'__module__': '__main__', 'showMyVersion': <function showMyVersion at 0x0000000002AFFB38>, '__dict__': <attribute '__dict__' of 'MyClass' objects>, 'myVersion': '1.1', '__weakref__': <attribute '__weakref__' of 'MyClass' objects>, '__doc__': 'MyClass class definition'})
>>> MyClass.__module__
'__main__'
>>> MyClass.__class__
<type 'type'>
>>> MyClass
<class '__main__.MyClass'>
```

可以看到类MyClass全名是```__main__.MyClass```，类名完全由模块名限定,使用```___module___```可以看到定义类所在的模块

对于新式类而言，访问任何类的```__class__```属性，会发现这是一个类型对象的实例。

### 实例

#### 初始化

```
>>> class MyClass(object):
...     pass
...
>>> mc = MyClass()
```

实例化对象

#### 特殊方法

当类被调用时，实例化的第一步是创建实例对象，如果实现了```__init__()```方法 ，实例化的第一步是对构造器的调用，```__init__```应当返回None

类的```__new__()```方法，一个静态方法，并且传入的参数是在类实例化操作时生成的。```__new__()```会调用父类的```__new__()```来创建对象（向上代理）

```__new__()```必须返回一个合法的实例，这样解释器在调用```__init__()```时，就可以把这个实例作为 self 传给它。调用父类的```__new__()```来创建对象，正像其它语言中使用 new 关键字一样。

NEW方法 TODO

```__del__()``` "解构器"方法，该函数在该实例对象所有的引用都被清除掉后才会执行。解构器只能被调用一次，一旦引用计数为0，则对象就被清除了 使用del来删除引用

#### 实例属性

使用内建函数dir()可以显示类属性,还可以打印所有实例的属性，实例也有```__dict__```特殊属性,他是实例属性构成的字典, ```__class__```实例所属的类

```
>>> class C(object):
...     pass
...
>>> c = C()
>>> dir(c)
['__class__', '__delattr__', '__dict__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__module__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__']
>>> c.__dict__
{}
>>> c.__class__
<class '__main__.C'>
```

#### 类属性的持久类

类的静态成员不会因为实例属性的变化而改变，但是类属性的更改会影响到所有的实例。


### 组合

```
class NewAddrBookEntry(object): 	# 类定义
	'new address book entry class'
	def __init__(self, nm, ph): 	# 定义构造器
		self.name = Name(nm) 	# 创建 Name 实例
		self.phone = Phone(ph) 	# 创建 Phone 实例
		print 'Created instance for:', self.name
```
NewAddrBookEntry 类由它自身和其它类组合而成。这就在一个类和其它组成类之间定义了一种"has-a/有一个"的关系。 比如NewAddrBookEntry类"有一个"Name 类实例和一个 Phone实例。

### 子类和派生

创建子类,创建子类的语法看起来与普通(新式)类没有区别， 一个类名，后跟一个或多个需要从其中派生的父类

```
class SubClassName (ParentClass1[, ParentClass2, ...]):
	'optional class documentation string'
	class_suite
```

### 继承

```__bases__类属性```它是一个包含其父类（parent）的集合的元组。

子类重新定义和父类相同的方法，可以对父类方法进行覆盖

#### 调用被覆盖的基类方法

调用一个未绑定的基类方法，明确给出子类的实例

```
>>> class P(object):
...     def foo(self):
...             print 'this is P_foo()'
...
>>> class C(P):
...     def foo(self):
...             print 'this is C_foo()'
...
>>> c = C()
>>> P.foo(c)
this is P_foo()
```

在子类的重写方法中显示的调用基类方法

```
class C(P):
	def foo(self):
		P.foo(self)
		print 'this is C_foo()'
```

使用super内建方法

```
class C(P):
	def foo(self):
		super(C, self).foo()
		print 'this is c_foo()'
```

super()不但能找到基类方法，而且还为我们传进 self，这样我们就不需要做这些事了。现在我们只要调用子类的方法，它会帮你完成一切.

使用 super()的重点，是你不需要明确提供父类

在python中重写```__init__```不会自动调用基类的```__init__```

#### 从标准类型派生

TODO


#### 多重继承

经典类和新式类的多重继承有着区别

经典类采用python2.2以前的方法，深度优先，从左至右进行搜索，取得在子类中使用的属性。

新式类它首先查找同胞兄弟，采用一种广度优先的方式。新式类有一个__mor__属性，显示继承的查找顺序

### 类、实例和其他对象的内建函数

#### issubclass()

issubclass() 布尔函数判断一个类是另一个类的子类或子孙类。

```
issubclass(sub, sup)
```

issubclass() 返回 True 的情况：给出的子类 sub 确实是父类 sup 的一个子类 （反之，则为 False）。这个函数也允许“不严格”的子类，意味着，一个类可视为其自身的子类，所以，这个函数如果当sub 就是 sup，或者从 sup 派生而来，则返回 True。

#### isinstance()

isinstance() 布尔函数在判定一个对象是否是另一个给定类的实例时，非常有用。

```
isinstance(obj1, obj2)
```

isinstance()在 obj1 是类 obj2 的一个实例，或者是 obj2 的子类的一个实例时，返回 True（反之，则为 False）

#### attr()系列函数

hasattr()函数是 Boolean 型的，它的目的就是为了决定一个对象是否有一个特定的属性，一般用于访问某属性前先作一下检查。getattr()和 setattr()函数相应地取得和赋值给对象的属性，getattr()会在你试图读取一个不存在的属性时，引发 AttributeError 异常，除非给出那个可选的默认参数。setattr()将要么加入一个新的属性，要么取代一个已存在的属性。而 delattr()函数会从一个对象中删除属性。

```
class myClass(object):
    def __init__(self):
        self.foo = 100
        
myInst = myClass()
hasattr(myInst,'foo')		#True
getattr(myInst,'foo')         #100
setattr(myInst,'bar',200)
getattr(myInst,'bar')          #200
delattr(myInst,'bar')
hasattr(myInst,'bar')         #False
```

#### dir()

dir(): 作用在实例上时，显示实例变量，还有实例所在的类及所有它的基类中定义的方法和类属性；作用在类上时，则显示类以及它的所有基类的__dict__中的内容。作用在模块上时，则显示模块的__dict__的内容。不带参数时，则显示调用者的局部变量。

#### super()

super(type[,obj]) 函数：帮助找出相应的父类，然后方便调用相关的属性；super(MyClass,self).__init__()。

#### var()

var()函数：与dir()类似。

## 参考
* python核心编程第二版
* [python笔记](http://www.cnblogs.com/NNUF/archive/2013/01/28/2880451.html)
* [__new__()和__init__()区别]（http://www.cnblogs.com/tuzkee/p/3540293.html）