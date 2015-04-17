---
layout: post
category: "python"
title: "python培训[4]"
tags: ["python培训"]
---
* pylint是一个Python代码风格的检查工具, 它依据的标准是Guido van Rossum的PEP8
* Pylint:  <http://www.logilab.org/project/pylint>
* Pep8:    <https://www.python.org/dev/peps/pep-0008/>
* Pep8中文: <http://code.google.com/p/zhong-wiki/wiki/PEP8> 
* 原谅我用2007年译的版本，我只是觉得当年这篇比较靠近原意。


#### 扩展安装：
* pip install pylint
* 可以这样用 pylint demo.py 
* 
* pip install pep8
* pep8 --show-source --show-pep8  demo.py 


#### Pylint 的输出

* MESSAGE_TYPE 有如下几种：
* (C) 惯例。违反了编码风格标准
* (R) 重构。写得非常糟糕的代码。
* (W) 警告。某些 Python 特定的问题。
* (E) 错误。很可能是代码中的错误。
* (F) 致命错误。阻止 Pylint 进一步运行的错误。


>
- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2015年4月
