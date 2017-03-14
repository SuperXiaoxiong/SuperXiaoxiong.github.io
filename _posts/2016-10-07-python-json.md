---
layout: post
title:  "python2 json的处理"
date:   2016-10-07 11:05:44
categories: python
tags:  python json 
---


* content
{:toc}

## 简述

python2 json的学习和使用


## 注意

python 中对于字符串，不管是键还是值都用双引号，

使用单引号报错：

```
print json.loads(dict_msg)
  File "C:\Python27\lib\json\__init__.py", line 339, in loads
      return _default_decoder.decode(s)
        File "C:\Python27\lib\json\decoder.py", line 364, in decode
	    obj, end = self.raw_decode(s, idx=_w(s, 0).end())
	    TypeError: expected string or buffer
```

可以用下面的方法转换

```
json_string=json.dumps(s)
python_obj=json.loads(json_string)
```


## json的使用

在```python```中自带有```json```的处理模块，只需要```import json```就行


 ```python```和```json```转换表

python 转 json

|     python     |      json      |
|:---------------|:---------------|
|dict            |object          |
|list, tuple     |array           |
|str, unicode    |string          |
|int, long, float|number          |
|True            |true            |
|False           |false           |
|None            |null            |



json 转 python

|     json       |     python     |
|:---------------|:---------------|
|object          |dict            |
|array           |list            |
|string          |unicode         |
|number(int)     |int, long       |
|number(real)    |float           |
|true            |True            |
|false           |False           |
|null            |None            |


在```json```数据中```false```和```true```对应python中的```False```, ```True```; ```null```对应python中的```None```

注意:通过表可以看见在转换的时候```tuple```会转换成```array```，并且在通过```load```方式也只会转换成```list```，对于```str```编码都会转换成```unicode```,如果使用则需要在使用前添加适当处理

### 基本使用

```
json.dumps(obj, skipkeys=False, ensure_ascii=True, check_circular=True, allow_nan=True,cls=None, indent=None, separators=None, encoding="utf-8″, default=None, sort_keys=False, **kw)

json.loads(s[, encoding[, cls[, object_hook[, parse_float[, parse_int[, parse_constant[,object_pairs_hook[, **kw]]]]]]]])
```

#### 将python对象生成json

```
json.dumps(data)    #生成json字符串
json.dump(data,file)     #生成json类文件对象
```

#### 将json对象转换成python

```
json.loads(str)    #解析json字符串
json.load(file)    #解析json类文件对象
```

注意在使用```loads```方法时，若```str```为空会报错。

### json中unicode使用

#### json中utf-8编码输出

```
>>> js = json.loads('{"language": "中文 / Chinese "}')
>>> print json.dumps(js)
{"language": "\u4e2d\u6587 / Chinese "}
>>> print json.dumps(js,ensure_ascii=False)
{"language": "中文 / Chinese "}
```

#### 其他编码转换成json对象

可以看见在```json.dumps(data)```中默认```encoding=utf-8```转换成unicode编码，所以其他编码输入需要设置encoding

```
>>> str = u'中文'
>>> gbk_str = str.encode('gb2312')
>>> type(gbk_str)
<type 'str'>
>>> json_str = json.dumps(gbk_str)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/lib/python2.7/json/__init__.py", line 243, in dumps
    return _default_encoder.encode(obj)
  File "/usr/lib/python2.7/json/encoder.py", line 201, in encode
    return encode_basestring_ascii(o)
UnicodeDecodeError: 'utf8' codec can't decode byte 0xd6 in position 0: invalid continuation byte
>>> json_str = json.dumps(gbk_str,encoding='gb2312')
>>> type(json_str)
<type 'str'>
>>> print json_str
"\u4e2d\u6587"
```

## 漂亮的输出

1. 用```pprint()```代替```print```输出,输出json
2. 添加indent参数,输出python对象```json.dumps(data, indent=4, ensure_ascii=False)```

## 参考

* [Python解析json学习](http://crazyof.me/blog/archives/368.html)
* [cookboook,python3](http://python3-cookbook.readthedocs.io/zh_CN/latest/c06/p02_read-write_json_data.html)
