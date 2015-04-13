---
layout: post
category: "android"
title: "python培训[1]"
tags: ["友盟统计"]
---
脚本下载：<http://pan.baidu.com/s/1jGDvFzo>  

####一、前言部分

- os系统：centos6.5

- python:2.7.7 或者 2.7.8

- 扩展要求:
        setuptools ,pip	 ,MySQL-python		
		
		  

####二、Pythonic八荣八耻

- 以动手实践为荣 , 以只看不练为耻;
- 以打印日志为荣 , 以单步跟踪为耻;
- 以空格缩进为荣 , 以制表缩进为耻;
- 以单元测试为荣 , 以人工测试为耻;
- 以模块复用为荣 , 以复制粘贴为耻;
- 以多态应用为荣 , 以分支判断为耻;
- 以Pythonic为荣 , 以冗余拖沓为耻;
- 以总结分享为荣 , 以跪求其解为耻;

####三、实践这些原则
```

#!/usr/bin/python2.7
 
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

