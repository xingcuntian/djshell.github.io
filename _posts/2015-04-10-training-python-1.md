---
layout: post
category: "python"
title: "python技术分享[1]－python初步感观"
tags: ["python技术分享"]
---
###1、环境简介
- os系统：centos6.5
- python:2.7.7 或者 2.7.8
- 扩展要求:
        setuptools ,pip	 ,MySQL-python	
- 如果你的python版本过低，可以用下面的脚本升级为2.7.7
- 脚本下载：<http://pan.baidu.com/s/1jGDvFzo>  	

- 		
###1.1 python使用方式
     - 进入python shell环境
     - 执行脚本模式 python demo.py

###2、初步感观
```

#coding=utf-8
#!/usr/bin/python2.7

###################################################
#
#      python 第一个显示程序，用于初步感观
#
###################################################

def fib():
  '''
  a generator that produces the fibonacci series's elements
  '''
 
  a = 1
  b = 1
  while True:
    a, b = a + b, a
    yield a
 
def nth(series, n):
  '''
  returns the nth element of a series,
  consuming the series' earlier elements.
  '''
 
  for x in series:
    n -= 1
    if n <= 0: return x
 
if __name__ == '__main__':
	print('Executed from the command line')
	print nth(fib(), 10)
	
```
	
- 执行 
- python demo.py 
- 输出
- Executed from the command line
- 144

###3、Python格言

- 输入python 进入python shell环境
- import this 
- 显示python之禅

- 
##### 3.1 Python之禅

   - Beautiful is better than ugly.  
   - 漂亮的代码要比丑陋的代码要好得多。  
   - Explicit is better than implicit.  
   - 明确的定义比 隐式定义更好。  
   - Simple is better than complex.  
   - 简单比负责要好。  
   - Complex is better than complicated.  
   - 负责要比搞复杂要好。  
   - Flat is better than nested.  
   - 扁平结构要比嵌套结构好。  
   - Sparse is better than dense.  
   - 简洁明了的代码要比稠密的代码要好。  
   - Readability counts.  
   - 可读写的计数。  
   - Special cases aren't special enough to break the rules.  
   - 专门的用例不是特殊到足以违反规则。  
   - Although practicality beats purity.  
   - 是的，实用性练就纯度。  
   - Errors should never pass silently.  
   - 错误永远都不会沉默。  
   - Unless explicitly silenced.  
   - 除非明确啥也不干。  
   - In the face of ambiguity, refuse the temptation to guess.  
   - 面对模糊定义、拒绝视图拍脑袋猜。  
   - There should be one-- and preferably only one --obvious way to do it.  
   - Although that way may not be obvious at first unless you're Dutch.  
   - 虽然一开始不那面明确,我们会选择更清晰一条到走。  
   - Now is better than never.  
   - 现在开始总比不开始的要好。  
   - Although never is often better than *right* now.  
   - 虽然从不尝试总比现在开始尝试好。  
   - If the implementation is hard to explain, it's a bad idea.  
   - 如果实现难以说明，那它是个坏主意。  
   - If the implementation is easy to explain, it may be a good idea.  
   - 如果实现容易说明，那它是个好主意。  
   - Namespaces are one honking great idea -- let's do more of those!  
   - 名称空间是一个好东西——让我们做更多那样的东西!  

- 
##### 3.2 Python“八荣八耻”

   - 以动手实践为荣 , 以只看不练为耻;
   - 以打印日志为荣 , 以单步跟踪为耻;
   - 以空格缩进为荣 , 以制表缩进为耻;
   - 以单元测试为荣 , 以人工测试为耻;
   - 以模块复用为荣 , 以复制粘贴为耻;
   - 以多态应用为荣 , 以分支判断为耻;
   - 以Pythonic为荣 , 以冗余拖沓为耻;
   - 以总结分享为荣 , 以跪求其解为耻;





<!--
![Alt text](/images/029_hooded_k_w_1.jpg)
-->

>
- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2015年4月
