---
layout: post
title:  python2 序列化pickle模块
date: 2017-05-18 12:20:56
categories: python
tags:  python pickle
---

* content
{:toc}

## 简述

python2 pickle序列化的学习和使用







## 移植性

pickle 文件格式独立于机器的体系结构，可以适用于不同系统下的python程序(当然不适用于其他语言)，检索支持的格式

```
>>> import pickle
>>> pickle.format_version
'2.0'
>>> pickle.compatible_formats
['1.0', '1.1', '1.2', '1.3', '2.0']
```

## 基本使用

将Python对象序列化为一个字节流，以便保存到文件、存储到数据库或者通过网络传输，序列化的最普遍做法是通过pickle模块。

序列化的做法都基本相似

### 使用说明

```
pickle.dump(data, file) //将对象存储到文件中，file:一个可写的文件对象(wb)
s = pickle.dumps(data) //将对象转换成一个存储对象

data1 = pickle.load(file) //从文件中读入 ，file:一个可读的文件对象(rb)
data2 = pickle.loads(s) 
```

### 可以被序列化的类型

python 2版本

```
1. None, True, and False
2. integers, long integers, floating point numbers, complex numbers
3. normal and Unicode strings
4. tuples, lists, sets, and dictionaries containing only picklable objects
5. functions defined at the top level of a module
6. built-in functions defined at the top level of a module
7. classes that are defined at the top level of a module
8. instances of such classes whose __dict__ or the result of calling __getstate__() is picklable (see section The pickle protocol for details).
```

### 异常

尝试序列化和反序列化对象时可能发生```PicklingError```异常;发生异常时，数量不明的字节被写入到了底层文件中。当尝试序列化高度递归的数据结构可能超过最大的递归层数，这样会raise一个```RuntimeError```异常。修改递归层数的限制一定要小心```sys.setrecursionlimit()```.

### 函数和类序列化

```
class Foo:
    attr = 'a class attr'

picklestring = pickle.dumps(Foo)

>>> print pickle.loads(picklestring)
__main__.Foo
```

仅仅只有函数名字被序列化，和定义这个函数的模块。函数代码和函数属性都没有被序列化。

这些限制就是为什么 可序列化的函数和类必须要定义在模块的顶层中

相同的，当类的实例是序列化的时候，他们的代码和数据没有被序列化。只有他们实例的数据被序列化。

对于函数或类的序列化是以名字来识别的，所以需要import相应的module

## 类实例序列化

当```unpickle```类的实例时，通常不会再调用它们的```_init_()```方法。相反，```Python```创建一个通用类实例，并应用已进行过```pickle```的实例属性，同时设置该实例的```_class_```属性，使其指向原来的类。

### getstate和setstate

方法```__getstate__()```在序列化时调用:默认序列化成```__dict__```,调用```__getstate__()```后return值作为序列化数据流

方法```__setstate__()```用于反序列化时```load```操作，适用参数恢复属性

注意：在新式类中，如果```__getstate__()```返回了一个错误的值,```__setstate__()```方法将不会调用

```
class Countdown(object):
    a = 1
    def __init__(self, n):
        self.n = n
        
    def test(self):
        print 'test'
        print self.n
        
    def __getstate__(self):
        return self.a, self.n 
       
    def __setstate__(self, state):
        self.a, self.n = state
        print 'hahaha'
        print self.a
        

c = Countdown(30)
import pickle

s = pickle.dumps(c)
print s 

t = pickle.loads(s)
t.__class__ = Countdown
t.test() 
```

### 参数获取

```
def save(obj):
    return (obj.__class__, obj.__dict__)

def load(cls, attributes):
    obj = cls.__new__(cls)
    obj.__dict__.update(attributes)
    return obj
	
```

旧式类```object.__getinitargs__()```

如果你想让你的类在序列化时时调用```__init__()```,你可以定义```__getinitargs__()```,它会返回一个参数元组，这个元组会传递给```__init__()```。注意，这个方法只能用于旧式类。不支持keyword。__getinitargs__()在序列化时调用。它返回的元组被并入到pickle的实例中

新式类```object.__getnewargs__()```
	
对新式类来说，你可以通过这个方法改变类在反序列化时传递给```__new__()```的参数。这个方法应该返回一个参数元组。

创建一个新样式类 C的实例

```
obj = C.__new__(C, *args)
```

其中args是在原始对象上调用__ getnewargs __()的结果；

```
class Countdown(object):
    a = 1
    b = 2
    def __init__(self, n):
        print 'construct************'
        
    def test(self):
        print 'test***************'
        print self.a
        
    def __getstate__(self):
        return (self.__class__, self.__dict__)
       
    def __setstate__(self, state):
        class_ , dict_ = state
        obj = self.__new__(class_)
        obj.__dict__= dict_
        return obj
        

c = Countdown(30)
import pickle

s = pickle.dumps(c)
print s 

t = pickle.loads(s)
print t.__class__
t.test() 
'''
```


注意：在反序列化时，那个实例的一些方法， ```__getattr__()```,```__getattribute__()```,```__setattr__()```会被调用在这种情况下方法历来与一些外部的不变量是正确的，这些类型需要实现```__getinitargs__()```,```__getnewargs__()```去建立一个不变量，否则，将不会调用```__new__()```和```__init__()```这些方法

### reduce()
	
实现```__reduce__()```方法,可以更有效和明确的序列化。在序列化的时候会无参调用```__reduce__()```方法，而且必须返回一个字符串或者是元组。
	
返回值是一个代表全局名称的字符串，```Python```会查找并```pickle```。
	
返回值是一个元组，必须是2到5个元素。可选元素可以被删除，```None```也可以被当做值。元组的内容被正常的序列化。
	
元组的元素定义：

1. 一个可以被调用的对象(类)，重建时使用(python2中要求 被注册调用的安全构造器或者必须有```__safe_for_unpickling__```为真的反序列化属性。否则，将会raise一个UnpicklingError。); 

2. 参数元组，供对象重建时调用(如果不接收参数是一个空元组); 
		
3. 对象的状态，将会被传到```__setstate__()```方法。如果这个对象没有```__setstate__()```方法，这个值必须是一个字典，而且会被加入对象的```__dict__```(可选)
		
4. 一个产生列表元素的迭代器对象, 用```append(item)```或者```extend(list_of_items)```加入这个对象。列举子类是很重要，但是也可以用作其他类，只要有```append（）```和```extend（）```的方法(可选)
		
5. 一个产生字典元素的迭代器对象，```(key, value)```会成为```obj[key] = value```可以用来作为字典子类，实现了```__setitem__()```的类也可以用(可选)
		
### reduce_ex(protocol)

__reduce_ex__ 的存在是为了兼容性。如果它被定义，在```pickle```时```__reduce_ex__```会代替```__reduce__```被调用。```__reduce__```也可以被定义，用于不支持```__reduce_ex__```的旧版pickle的API调用。

## 限制全局变量

pickle序列化在使用过程中会带来很多安全性问题，所以在使用的过程中进行自定义的```Unpickler.find_class()```，对可以使用的模块做白名单限制，保证安全性

```
import builtins
import io
import pickle

safe_builtins = {
    'range',
    'complex',
    'set',
    'frozenset',
    'slice',
}

class RestrictedUnpickler(pickle.Unpickler):

    def find_class(self, module, name):
        # Only allow safe classes from builtins.
        if module == "builtins" and name in safe_builtins:
            return getattr(builtins, name)
        # Forbid everything else.
        raise pickle.UnpicklingError("global '%s.%s' is forbidden" %
                                     (module, name))

def restricted_loads(s):
    """Helper function analogous to pickle.loads()."""
    return RestrictedUnpickler(io.BytesIO(s)).load()
```	

上面这个例子只允许使用```builtins```模块中几个安全类

```
>>> restricted_loads(pickle.dumps([1, 2, range(15)]))
[1, 2, range(0, 15)]
>>> restricted_loads(b"cos\nsystem\n(S'echo hello world'\ntR.")
Traceback (most recent call last):
  ...
pickle.UnpicklingError: global 'os.system' is forbidden
>>> restricted_loads(b'cbuiltins\neval\n'
...                  b'(S\'getattr(__import__("os"), "system")'
...                  b'("echo hello world")\'\ntR.')
Traceback (most recent call last):
  ...
pickle.UnpicklingError: global 'builtins.eval' is forbidden
```

## 序列化和反序列化外部对象(待完善)

对于对象的持久性的优势，序列化模块支持引用对象外部数据流的想法。很多对象都是用持久性id引用，但是名字仅仅是任意的可以打印的ascii字符。名字并不是按照序列化模块定义这个方法；代表了用户对于序列化和反序列化自定义函数。
	
定义外部持久化实体id，需要设置pickle的persistent_id属性，和反序列化的persistent_load的属性
	
序列化对象有一个外部实体id，序列化必须有自定义persistent_id()方法，把对象作为参数，返回None或者持久化id。当返回None的时候，序列化器仅仅把对象和正常情况一样pickle。当返回了一个持久化id字符串，会有一个标记，反序列化的时候就会被视为持久化id
	
反序列化外部实体的时候，反序列化器必须有自定义的persistent_load()功能：将持久化id作为参数返回一个可以引用的对象
	
在cPickle模块中，反序列化的persistent_load属性也可以被设置成为Python 列表。在这种情况下，反序列化检测到了持久化id，持久化id仅仅被加入到list中。这个功能存在所以序列化数据可以被嗅探到在使用对象的引用时，而不是把所有的对象安装成为一个pickle。设置persistent_load到一个列表中 经常备用在反序列化的 noload() 在联合的时候
	
Subclassing Unpicklers	
	
默认的，反序列化将会导入（pickle data中）所有的类。你可以精确控制被反序列化的内容和被调用的自适应的内容。不幸的是，pickle和cpickle是不一样的

在pickle模块中，你需要得到unpickle的之类，重载load_global()方法。load_global()需要读取两行pickle data流，第一行是模块包含的类的名字，第二行是实例的名字。然后会查找这个类，可能会导入模块查明属性，加入到反序列化的栈中。然后，这个类将会被标明成__class__属性为空类，自动的创造一个实例而不是调用类的__init__()初始化方法。你需要将load_global()方法push到反序列化栈中，选一个已知安全的版本去反序列化。这取决以你如何构造这个类。或者你可以raise一个错误如果你不想允许反序列化所有的实例。
	
cpickle模块会更清楚。你可以设置find_global属性为函数或者None来控制反序列化。如果设置为None，反序列化实例会raise一个UnpicklingError。如果是一个函数，那么就会接受一个模块名或者是一个类名，而且返回相应的类对象。查找必要的类或者导入包是重要的，而且可能会生成错误在反序列化实例类时。
	
	
## 参考

* [python2.7官方文档](https://docs.python.org/2.7/library/pickle.html#what-can-be-pickled-and-unpickled)
* [python3文档](https://docs.python.org/3/library/pickle.html#module-pickle)
* [python3文档2](http://docspy3zh.readthedocs.io/en/latest/library/pickle.html?highlight=pickle)
* [python3cookbook](http://python3-cookbook.readthedocs.io/zh_CN/latest/c05/p21_serializing_python_objects.html)
* [ibm开发文档](https://www.ibm.com/developerworks/cn/linux/l-pypers/)
* [python翻译](http://pyzh.readthedocs.io/en/latest/python-magic-methods-guide.html#pickling)









