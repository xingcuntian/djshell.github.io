---
layout: post
category: "数据仓库"
title: "阿里云[4]－ ODPS UDF Python API"
tags: ["数据仓库"]
---
####ODPS_ele—UDF Python API
#####自定义函数(UDF)
- UDF全称User Defined Function，即用户自定义函数。ODPS提供了很多内建函数来满足用户的计算需求，同时用户还可以通过创建自定义函数来满足不同的计算需求。UDF在使用上与普通的 SQL内建函数 类似。
- 在ODPS中，用户可以扩展的UDF有三种，分别是：
- UDF 分类 |  描述
- User Defined Scalar Function 通常也称之为UDF
    - 自定义函数，准确的说是用户自定义标量函数 (User Defined Scalar Function)。UDF的输入与输 出是一对一的关系，即读入一行数据， 写出一条输出值。
- UDAF(User Defined Aggregation Function)
    - 自定义聚合函数，其输入与输出是多对一的关系， 即将多条输入记录聚合成一条输出值。可以与 SQL中的Group By语句联用。具体语法请参考聚合函数 。
- UDTF(User Defined Table Valued Function)
    - 自定义表函数，是用来解决一次函数调用输出 多行数据场景的，也是唯一能返回多个字段的自定 义函数。而UDF及UDAF只能一次计算输出一条 返回值。
- 注解
	- UDF广义的说法代表了自定义标量函数，自定义聚合函数及自定义表函数三种类型的自定义函数的集合。狭义来说，仅代表用户自定义标量函数。文档会经常使用这一名词，请读者根据文档上下文判断具体含义。
- 受限环境
	- ODPS UDF的Python版本为2.7，并以沙箱模式执行用户代码，即代码是在一个受限的运行环境中执行的，在这个环境中，被禁止的行为包括：

    读写本地文件
    启动子进程
    启动线程
    使用socket通信
    其他系统调用

基于上述原因，用户上传的代码必须都是纯Python实现，C扩展模块是被禁止的。

此外，Python的标准库中也不是所有模块都可用，涉及到上述功能的模块都会被禁止。具体标准库可用模块说明如下：
所有纯Python实现(不依赖扩展模块)的模块都可用
C实现的扩展模块中下列模块可用

```
    array ；audioop ；
    binascii ；_bisect ；
    cmath ；_codecs_cn ；_codecs_hk ；_codecs_iso2022 ；_codecs_jp ；_codecs_kr
    _codecs_tw ；_collections ；cStringIO ；
    datetime ；
    _functools ；future_builtins ；
    _hashlib ；_heapq ；
    itertools ；
    _json ；
    _locale ；_lsprof ；
    math ；_md5 ；_multibytecodec
    operator ；
    _random ；
    _sha256 ；_sha512 ；_sha ；_struct ；strop
    time ；
    unicodedata ；
    _weakref ；
    cPickle；
```

- 部分模块功能受限。比如沙箱限制了用户代码最多能往标准输出和标准错误输出写出数据的大小，即``sys.stdout/sys.stderr``最多能写20Kb，多余的字符会被忽略。

第三方库

运行环境中还安装了除标准库以外比较常用的三方库，做为标准库的补充。支持的三方库列表如下：
numpy

警告

三方库的使用同样受到禁止本地、网络IO或其他在受限环境下的限制，因此三方库中涉及到相关功能的API也是被禁止的。
参数与返回值类型

@odps.udf.annotate(signature)

Python UDF目前支持ODPS SQL数据类型有：bigint, string, double, boolean和datetime。SQL语句在执行之前，所有函数的参数类型和返回值类型必须确定。因此对于Python这一动态类型语言，需要通过对UDF类加decorator的方式指定函数签名。

函数签名signature通过字符串指定，语法如下：

```
arg_type_list '->' type_list
arg_type_list: type_list | '*' | ''
type_list: [type_list ','] type
type: 'bigint' | 'string' | 'double' | 'boolean' | 'datetime'
```
箭头左边表示参数类型，右边表示返回值类型。
只有UDTF的返回值可以是多列, UDF和UDAF只能返回一列。
‘*’代表变长参数，使用变长参数，UDF/UDTF/UDAF可以匹配任意输入参数。

下面是合法的signature的例子：

```
'bigint,double->string'            # 参数为bigint、double，返回值为string
'bigint,boolean->string,datetime'  
```

# UDTF参数为bigint、boolean，返回值为string,datetime

'*->string'                        # 变长参数，

输入参数任意

，返回值为string
'->double'                         # 参数为空，返回值为double

Query语义解析阶段会将检查到不符合函数签名的用法，抛出错误禁止执行。执行期，UDF函数的参数会以函数签名指定的类型传给用户。用户的返回值类型也要与函数签名指定的类型一致，否则检查到类型不匹配时也会报错。ODPS SQL数据类型对应Python类型如下：

image

- 注解

   - Datetime类型是以int的形式传给用户代码的，值为epoch utc time起始至今的毫秒数。用户可以通过Python标准库中的datetime模块处理日期时间类型。
    NULL值对应Python里的None。

odps.udf.int(value[, silent=True])

Python builtin函数 int 的修改。增加了参数 silent 。当 silent 为 True 时，如果 value 无法转为 int ，不会抛出异常，而是返回 None 。
UDF

实现Python UDF非常简单，只需要定义一个new-style class，并实现 evaluate 方法。下面是一个例子：

```
from odps.udf import annotate

@annotate("bigint,bigint->bigint")
class MyPlus(object):

   def evaluate(self, arg0, arg1):
       if None in (arg0, arg1):
           return None
       return arg0 + arg1
```
注解：Python UDF必须通过annotate指定函数签名
UDAF

class odps.udf.BaseUDAF

继承此类实现Python UDAF。

BaseUDAF.new_buffer()

实现此方法返回聚合函数的中间值的buffer。buffer必须是mutable object(比如list, dict)，并且buffer的大小不应该随数据量递增，在极限情况下，buffer marshal过后的大小不应该超过2Mb。

BaseUDAF.iterate(buffer[, args, ...])

实现此方法将args聚合到中间值buffer中。

BaseUDAF.merge(buffer, pbuffer)

实现此方法将两个中间值buffer聚合到一起，即将pbuffer merge到buffer中。

BaseUDAF.terminate(buffer)

实现此方法将中间值buffer转换为ODPS SQL基本类型。

下面是一个UDAF求平均值的例子。

```
#coding:utf-8
from odps.udf import annotate
from odps.udf import BaseUDAF

@annotate('double->double')
class Average(BaseUDAF):

    def new_buffer(self):
        return [0, 0]

    def iterate(self, buffer, number):
        if number is not None:
            buffer[0] += number
            buffer[1] += 1

    def merge(self, buffer, pbuffer):
        buffer[0] += pbuffer[0]
        buffer[1] += pbuffer[1]

    def terminate(self, buffer):
        if buffer[1] == 0:
            return 0
        return buffer[0] / buffer[1]
```

UDTF

class odps.udf.BaseUDTF

   Python UDTF的基类，用户继承此类，并实现 process, close 等方法。

BaseUDTF.__init__()

   初始化方法，继承类如果实现这个方法，则必须在一开始调用基类的初始化方法 super(BaseUDTF,self).__init__() 。

   __init__ 方法在整个UDTF生命周期中只会被调用一次，即在处理第一条记录之前。当UDTF需要保存内部状态时，可以在这个方法中初始化所有状态。

BaseUDTF.process([args, ...])

   这个方法由ODPS SQL框架调用，SQL中每一条记录都会对应调用一次 process ， process 的参数为SQL语句中指定的UDTF输入参数。

BaseUDTF.forward([args, ...])

   UDTF的输出方法，此方法由用户代码调用。每调用一次 forward ，就会输出一条记录。 forward 的参数为SQL语句中指定的UDTF的输出参数。

BaseUDTF.close()

   UDTF的结束方法，此方法由ODPS SQL框架调用，并且只会被调用一次，即在处理完最后一条记录之后。

下面是一个UDTF的例子。

```
#coding:utf-8
# explode.py

from odps.udf import annotate
from odps.udf import BaseUDTF

@annotate('string -> string')
class Explode(BaseUDTF):
   """将string按逗号分隔输出成多条记录"""
   def process(self, arg):
       props = arg.split(',')
       for p in props:
           self.forward(p)
```

注解

Python UDTF也可以不加annotate指定参数类型和返回值类型。这样，函数在SQL中使用时可以匹配任意输入参数，但返回值类型无法推导，所有输出参数都将认为是string类型。因此在调用 forward 时，就必须将所有输出值转成 str 类型。
引用资源

Python UDF可以通过 odps.distcache 模块引用资源文件，目前支持引用文件资源和表资源。

odps.distcache.get_cache_file(resource_name)

release-2012.09.03 新版功能.

返回指定名字的资源内容。 resource_name 为 str 类型，对应当前Project中已存在的资源名。如果资源名非法或者没有相应的资源，会抛出异常。

返回值为 file-like object ，在使用完这个object后，调用者有义务调用 close 方法释放打开的资源文件。

下面是使用 get_cache_file 的例子：

```
from odps.udf import annotate
from odps.distcache import get_cache_file

@annotate('bigint->string')
class DistCacheExample(object):

def __init__(self):
    cache_file = get_cache_file('test_distcache.txt')
    kv = {}
    for line in cache_file:
        line = line.strip()
        if not line:
            continue
        k, v = line.split()
        kv[int(k)] = v
    cache_file.close()
    self.kv = kv

def evaluate(self, arg):
    return self.kv.get(arg)

odps.distcache.get_cache_table(resource_name)
```

release-2012.11.14 新版功能.

返回指定资源表的内容。 resource_name 为 str 类型，对应当前Project中已存在的资源表名。如果资源名非法或者没有相应的资源，会抛出异常。

返回值为 generator 类型，调用者通过遍历获取表的内容，每次遍历得到的是以 tuple 形式存在的表中的一条记录。

下面是使用 get_cache_table 的例子：

```
from odps.udf import annotate
from odps.distcache import get_cache_table

@annotate('->string')
class DistCacheTableExample(object):
    def __init__(self):
        self.records = list(get_cache_table('udf_test'))
        self.counter = 0
        self.ln = len(self.records)

    def evaluate(self):
        if self.counter > self.ln - 1:
            return None
        ret = self.records[self.counter]
        self.counter += 1
        return str(ret)
```

注意事项
表达式优化

当一个Query中有多个相同UDF，并且他们的参数也都一致时，这些UDF在执行时会被优化成只执行一次。例如：

```
random.seed(12345)
@annotate('bigint->bigint')
class MyRand(object):
    def evaluate(self, a):
        return random.randint(0, 10)
```

实现一个Rand函数，希望每次调用Rand时返回一个随机值。

> select MyRand(c_int_a), MyRand(c_int_a) from udf_test;

+------------+------------+

| _c0        | _c1        |

+------------+------------+

| 4          | 4          |

| 0          | 0          |

| 9          | 9          |

| 3          | 3          |

+------------+------------+


可以看到默认情况下，同一行的两次Rand调用返回值结果一样，这是因为被优化后只执行一次导致的。如果不想要这个优化，可以通过设置配置项odps.sql.udf.optimize.reuse 取消这个优化：

> set odps.sql.udf.optimize.reuse=false;
> select MyRand(c_int_a), MyRand(c_int_a) from udf_test;


+------------+------------+

| _c0        | _c1        |

+------------+------------+

| 4          | 0          |

| 9          | 3          |

| 4          | 2          |

| 6          | 1          |

+------------+------------+


总结

ODPS为Python提供的类有
1. 参数与返回值类型

@odps.udf.annotate(signature)，ODPS SQL数据类型对应Python类型如下：

image

odps.udf.int(value[, silent=True])
2. UDF

####定义一个new-style class，并实现 evaluate 方法

```
from odps.udf import annotate
@annotate("bigint,bigint->bigint")
class MyPlus(object):
   def evaluate(self, arg0, arg1):
       if None in (arg0, arg1):
           return None
       return arg0 + arg1
```

3. UDAF

class odps.udf.BaseUDAF—继承此类实现Python UDAF。

BaseUDAF类拥有的四个方法如下：

```
BaseUDAF.new_buffer()

BaseUDAF.iterate(buffer[, args, ...])

BaseUDAF.merge(buffer, pbuffer)

BaseUDAF.terminate(buffer)

```

下面是一个UDAF求平均值的例子。


```
#coding:utf-8
from odps.udf import annotate
from odps.udf import BaseUDAF

@annotate('double->double')
class Average(BaseUDAF):

    def new_buffer(self):
        return [0, 0]

    def iterate(self, buffer, number):
        if number is not None:
            buffer[0] += number
            buffer[1] += 1

    def merge(self, buffer, pbuffer):
        buffer[0] += pbuffer[0]
        buffer[1] += pbuffer[1]

    def terminate(self, buffer):
        if buffer[1] == 0:
            return 0
        return buffer[0] / buffer[1]
        

```

4.UDTF

class odps.udf.BaseUDTF—Python UDTF的基类，用户继承此类，并实现 process , close 等方法。

BaseUDTF类拥有的四个方法

```
BaseUDTF.__init__()

BaseUDTF.process([args, ...])

BaseUDTF.forward([args, ...])

BaseUDTF.close()

```
    
下面是一个UDTF的例子。

```
#coding:utf-8
# explode.py

from odps.udf import annotate
from odps.udf import BaseUDTF

@annotate('string -> string')
class Explode(BaseUDTF):
   """将string按逗号分隔输出成多条记录
   """

   def process(self, arg):
       props = arg.split(',')
       for p in props:
           self.forward(p)
```