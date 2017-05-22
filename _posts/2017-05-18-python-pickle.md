---
layout: post
title:  "python2 pickle���л�"
date:   2017-05-19 19:23:52
categories: python
tags:  python pickle
---


* content
{:toc}

## ����

python2 pickle���л���ѧϰ��ʹ��







## ��ֲ��

pickle �ļ���ʽ�����ڻ�������ϵ�ṹ�����������ڲ�ͬϵͳ�µ�python����(��Ȼ����������������)������֧�ֵĸ�ʽ

```
>>> import pickle
>>> pickle.format_version
'2.0'
>>> pickle.compatible_formats
['1.0', '1.1', '1.2', '1.3', '2.0']
```

## ����ʹ��

��Python�������л�Ϊһ���ֽ������Ա㱣�浽�ļ����洢�����ݿ����ͨ�����紫�䣬���л������ձ�������ͨ��pickleģ�顣

���л�����������������

### ʹ��˵��

```
pickle.dump(data, file) //������洢���ļ��У�file:һ����д���ļ�����(wb)
s = pickle.dumps(data) //������ת����һ���洢����

data1 = pickle.load(file) //���ļ��ж��� ��file:һ���ɶ����ļ�����(rb)
data2 = pickle.loads(s) 
```

### ���Ա����л�������

python 2�汾

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

### �쳣

�������л��ͷ����л�����ʱ���ܷ���```PicklingError```�쳣;�����쳣ʱ�������������ֽڱ�д�뵽�˵ײ��ļ��С����������л��߶ȵݹ�����ݽṹ���ܳ������ĵݹ������������raiseһ��```RuntimeError```�쳣���޸ĵݹ����������һ��ҪС��```sys.setrecursionlimit()```.

### �����������л�

```
class Foo:
    attr = 'a class attr'

picklestring = pickle.dumps(Foo)

>>> print pickle.loads(picklestring)
__main__.Foo
```

����ֻ�к������ֱ����л����Ͷ������������ģ�顣��������ͺ������Զ�û�б����л���

��Щ���ƾ���Ϊʲô �����л��ĺ����������Ҫ������ģ��Ķ�����

��ͬ�ģ������ʵ�������л���ʱ�����ǵĴ��������û�б����л���ֻ������ʵ�������ݱ����л���

���ں�����������л�����������ʶ��ģ�������Ҫimport��Ӧ��module

## ��ʵ�����л�

��```unpickle```���ʵ��ʱ��ͨ�������ٵ������ǵ�```_init_()```�������෴��```Python```����һ��ͨ����ʵ������Ӧ���ѽ��й�```pickle```��ʵ�����ԣ�ͬʱ���ø�ʵ����```_class_```���ԣ�ʹ��ָ��ԭ�����ࡣ

### getstate��setstate

����```__getstate__()```�����л�ʱ����:Ĭ�����л���```__dict__```,����```__getstate__()```��returnֵ��Ϊ���л�������

����```__setstate__()```���ڷ����л�ʱ```load```���������ò����ָ�����

ע�⣺����ʽ���У����```__getstate__()```������һ�������ֵ,```__setstate__()```�������������

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

### ������ȡ

```
def save(obj):
    return (obj.__class__, obj.__dict__)

def load(cls, attributes):
    obj = cls.__new__(cls)
    obj.__dict__.update(attributes)
    return obj
	
```

��ʽ��```object.__getinitargs__()```

�������������������л�ʱʱ����```__init__()```,����Զ���```__getinitargs__()```,���᷵��һ������Ԫ�飬���Ԫ��ᴫ�ݸ�```__init__()```��ע�⣬�������ֻ�����ھ�ʽ�ࡣ��֧��keyword��__getinitargs__()�����л�ʱ���á������ص�Ԫ�鱻���뵽pickle��ʵ����

��ʽ��```object.__getnewargs__()```
	
����ʽ����˵�������ͨ����������ı����ڷ����л�ʱ���ݸ�```__new__()```�Ĳ������������Ӧ�÷���һ������Ԫ�顣

����һ������ʽ�� C��ʵ��

```
obj = C.__new__(C, *args)
```

����args����ԭʼ�����ϵ���__ getnewargs __()�Ľ����

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


ע�⣺�ڷ����л�ʱ���Ǹ�ʵ����һЩ������ ```__getattr__()```,```__getattribute__()```,```__setattr__()```�ᱻ��������������·���������һЩ�ⲿ�Ĳ���������ȷ�ģ���Щ������Ҫʵ��```__getinitargs__()```,```__getnewargs__()```ȥ����һ�������������򣬽��������```__new__()```��```__init__()```��Щ����

### object.__reduce__()
	
ʵ��```__reduce__()```����,���Ը���Ч����ȷ�����л��������л���ʱ����޲ε���```__reduce__()```���������ұ��뷵��һ���ַ���������Ԫ�顣
	
����ֵ��һ������ȫ�����Ƶ��ַ�����```Python```����Ҳ�```pickle```��
	
����ֵ��һ��Ԫ�飬������2��5��Ԫ�ء���ѡԪ�ؿ��Ա�ɾ����```None```Ҳ���Ա�����ֵ��Ԫ������ݱ����������л���
	
Ԫ���Ԫ�ض��壺

1. һ�����Ա����õĶ���(��)���ؽ�ʱʹ��(python2��Ҫ�� ��ע����õİ�ȫ���������߱�����```__safe_for_unpickling__```Ϊ��ķ����л����ԡ����򣬽���raiseһ��UnpicklingError��); 

2. ����Ԫ�飬�������ؽ�ʱ����(��������ղ�����һ����Ԫ��); 
		
3. �����״̬�����ᱻ����```__setstate__()```����������������û��```__setstate__()```���������ֵ������һ���ֵ䣬���һᱻ��������```__dict__```(��ѡ)
		
4. һ�������б�Ԫ�صĵ���������, ��```append(item)```����```extend(list_of_items)```������������о������Ǻ���Ҫ������Ҳ�������������ֻ࣬Ҫ��```append����```��```extend����```�ķ���(��ѡ)
		
5. һ�������ֵ�Ԫ�صĵ���������```(key, value)```���Ϊ```obj[key] = value```����������Ϊ�ֵ����࣬ʵ����```__setitem__()```����Ҳ������(��ѡ)
		
### object.__reduce_ex__(protocol)

__reduce_ex__ �Ĵ�����Ϊ�˼����ԡ�����������壬��```pickle```ʱ```__reduce_ex__```�����```__reduce__```�����á�```__reduce__```Ҳ���Ա����壬���ڲ�֧��```__reduce_ex__```�ľɰ�pickle��API���á�

## ����ȫ�ֱ���

pickle���л���ʹ�ù����л�����ܶలȫ�����⣬������ʹ�õĹ����н����Զ����```Unpickler.find_class()```���Կ���ʹ�õ�ģ�������������ƣ���֤��ȫ��

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

�����������ֻ����ʹ��```builtins```ģ���м�����ȫ��

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

## ���л��ͷ����л��ⲿ����(������)

���ڶ���ĳ־��Ե����ƣ����л�ģ��֧�����ö����ⲿ���������뷨���ܶ�������ó־���id���ã��������ֽ���������Ŀ��Դ�ӡ��ascii�ַ������ֲ����ǰ������л�ģ�鶨������������������û��������л��ͷ����л��Զ��庯����
	
�����ⲿ�־û�ʵ��id����Ҫ����pickle��persistent_id���ԣ��ͷ����л���persistent_load������
	
���л�������һ���ⲿʵ��id�����л��������Զ���persistent_id()�������Ѷ�����Ϊ����������None���߳־û�id��������None��ʱ�����л��������Ѷ�����������һ��pickle����������һ���־û�id�ַ���������һ����ǣ������л���ʱ��ͻᱻ��Ϊ�־û�id
	
�����л��ⲿʵ���ʱ�򣬷����л����������Զ����persistent_load()���ܣ����־û�id��Ϊ��������һ���������õĶ���
	
��cPickleģ���У������л���persistent_load����Ҳ���Ա����ó�ΪPython �б�����������£������л���⵽�˳־û�id���־û�id���������뵽list�С�������ܴ����������л����ݿ��Ա���̽����ʹ�ö��������ʱ�������ǰ����еĶ���װ��Ϊһ��pickle������persistent_load��һ���б��� ���������ڷ����л��� noload() �����ϵ�ʱ��
	
Subclassing Unpicklers	
	
Ĭ�ϵģ������л����ᵼ�루pickle data�У����е��ࡣ����Ծ�ȷ���Ʊ������л������ݺͱ����õ�����Ӧ�����ݡ����ҵ��ǣ�pickle��cpickle�ǲ�һ����

��pickleģ���У�����Ҫ�õ�unpickle��֮�࣬����load_global()������load_global()��Ҫ��ȡ����pickle data������һ����ģ�������������֣��ڶ�����ʵ�������֡�Ȼ����������࣬���ܻᵼ��ģ��������ԣ����뵽�����л���ջ�С�Ȼ������ཫ�ᱻ������__class__����Ϊ���࣬�Զ��Ĵ���һ��ʵ�������ǵ������__init__()��ʼ������������Ҫ��load_global()����push�������л�ջ�У�ѡһ����֪��ȫ�İ汾ȥ�����л�����ȡ��������ι�������ࡣ���������raiseһ����������㲻���������л����е�ʵ����
	
cpickleģ������������������find_global����Ϊ��������None�����Ʒ����л����������ΪNone�������л�ʵ����raiseһ��UnpicklingError�������һ����������ô�ͻ����һ��ģ����������һ�����������ҷ�����Ӧ������󡣲��ұ�Ҫ������ߵ��������Ҫ�ģ����ҿ��ܻ����ɴ����ڷ����л�ʵ����ʱ��
	
	
## �ο�

* [python2.7�ٷ��ĵ�](https://docs.python.org/2.7/library/pickle.html#what-can-be-pickled-and-unpickled)
* [python3�ĵ�](https://docs.python.org/3/library/pickle.html#module-pickle)
* [python3�ĵ�2](http://docspy3zh.readthedocs.io/en/latest/library/pickle.html?highlight=pickle)
* [python3cookbook](http://python3-cookbook.readthedocs.io/zh_CN/latest/c05/p21_serializing_python_objects.html)
* [ibm�����ĵ�](https://www.ibm.com/developerworks/cn/linux/l-pypers/)
* [python����](http://pyzh.readthedocs.io/en/latest/python-magic-methods-guide.html#pickling)









