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




## json的使用

在```python```中自带有```json```的处理模块，只需要```import json```就行

在```json```数据中```false```和```true```会转化为```False```, ```True```; ```null```为转为```None```


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

1. 用```pprint()```代替```print```输出
2. 添加indent参数```json.dumps(data, indent=4)```

## 参考

* [Python解析json学习](http://crazyof.me/blog/archives/368.html)
* [cookboook,python3](http://python3-cookbook.readthedocs.io/zh_CN/latest/c06/p02_read-write_json_data.html)
