---
layout: post
category: "python"
title: "python技术分享[4]－使用pylint约束代码规范"
tags: ["python技术分享"]
---
* pylint是一个Python代码风格的检查工具, 它依据的标准是Guido van Rossum的PEP8
* Pylint:  <http://www.logilab.org/project/pylint>
* Pep8:    <https://www.python.org/dev/peps/pep-0008/>
* Pep8中文: <http://code.google.com/p/zhong-wiki/wiki/PEP8> 
* 原谅我用2007年译的版本，我只是觉得当年这篇比较靠近原意。


#### 1.扩展安装：
* pip install pylint
* 可以这样用 pylint demo.py 
* 
* pip install pep8
* pep8 --show-source --show-pep8  demo.py 


#### 2.Pylint 的输出
- 对于每一个 Python 模块，Pylint 的结果中首先显示一些"*"字符 , 后面紧跟模块的名字，然后是一系列的 message, 
- message 的格式如下：
- MESSAGE_TYPE: LINE_NUM:[OBJECT:] MESSAGE 
	- MESSAGE_TYPE 有如下几种：
	- (C) 惯例。违反了编码风格标准
	- (R) 重构。写得非常糟糕的代码。
	- (W) 警告。某些 Python 特定的问题。
	- (E) 错误。很可能是代码中的错误。
	- (F) 致命错误。阻止 Pylint 进一步运行的错误。
	
####3.样本实例
demo.py在 <http://djshell.github.io/python/training-python-1.html>
######3.1检查python格式
- pylint demo.py 
	- No config file found, using default configuration
	- ************* Module demo
	- W: 11, 0: Bad indentation. Found 2 spaces, expected 4 (bad-indentation)
	- W: 15, 0: Bad indentation. Found 2 spaces, expected 4 (bad-indentation)
	- W: 16, 0: Bad indentation. Found 2 spaces, expected 4 (bad-indentation)
	- W: 17, 0: Bad indentation. Found 2 spaces, expected 4 (bad-indentation)
	- W: 18, 0: Bad indentation. Found 4 spaces, expected 8 (bad-indentation)
	- W: 19, 0: Bad indentation. Found 4 spaces, expected 8 (bad-indentation)
	- W: 22, 0: Bad indentation. Found 2 spaces, expected 4 (bad-indentation)
	- W: 27, 0: Bad indentation. Found 2 spaces, expected 4 (bad-indentation)
	- W: 28, 0: Bad indentation. Found 4 spaces, expected 8 (bad-indentation)
	- W: 29, 0: Bad indentation. Found 4 spaces, expected 8 (bad-indentation)
	- C: 32, 0: Unnecessary parens after u'print' keyword (superfluous-parens)
	- C:  1, 0: Missing module docstring (missing-docstring)
	- C: 15, 2: Invalid variable name "a" (invalid-name)
	- C: 16, 2: Invalid variable name "b" (invalid-name)
	- C: 18, 4: Invalid variable name "a" (invalid-name)
	- C: 18, 7: Invalid variable name "b" (invalid-name)
	- C: 21, 0: Invalid argument name "n" (invalid-name)
	- C: 27, 6: Invalid variable name "x" (invalid-name)
	- C: 29,15: More than one statement on a single line (multiple-statements)
	- ............
	- ............省略多行，下面是得分
	- Your code has been rated at -2.67/10
<br><br><br>
- 可能是在页面复制贴粘时空格数据发生改变了，我们只得了负分现在我们来按照提示整理格式
	- Bad indentation. Found 2 spaces, expected 4 (bad-indentation) 空格期望四个，错误缩进
	-  Unnecessary parens after u'print' keyword (superfluous-parens) print 之后多余的小括号
	-  Missing module docstring (missing-docstring) 没有模块文档说明
	-  Invalid variable name "a" (invalid-name) 非法的变量
	-  No space allowed around keyword argument assignment 不要在用于指定
关键字参数 (keyword argument) 或默认参数值的 '=' 号周围使用空格
	- Trailing whitespace (trailing-whitespace) 尾随空白，行尾有空白字符，需要删除

<br><br>
- 按照提示更改代码直到得到10分
	- Your code has been rated at 10.00/10 (previous run: 10.00/10, +0.00)

####更改完的代码如下：

```
# coding=utf-8
#!/usr/bin/python2.7

'''
python pylint demo
'''

def fib():
    '''
    a generator that produces the fibonacci series's elements
    '''

    int_a = 1
    int_b = 1
    while True:
        int_a, int_b = int_a + int_b, int_a
        yield int_a


def nth(series, num):
    '''
   returns the nth element of a series, consuming the series' earlier elements.    '''

    for key in series:
        num -= 1
        if num <= 0:
            return key

if __name__ == '__main__':
    print 'Executed from the command line'
    print nth(fib(), 10)
```
>
- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2015年4月
