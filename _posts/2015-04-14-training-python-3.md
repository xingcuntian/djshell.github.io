---
layout: post
category: "python"
title: "python培训[3]"
tags: ["友盟统计"]
---
##python基本类型在业务中的应用实践

```
#!/usr/bin/python
# coding=utf-8
###################################################
#  
#  功能：python入门引导 
#        
####################################################

#统计游戏中玩家及玩家登陆次数
games = ['a','b','a','c','d','a','b','c','b']  
from collections import Counter  
d = Counter(games) 
print d


#排位赛规则，如果积分相同，则按胜利场次，再相同按失败场次
games =[{'f':3,'s':10,'b':12},{'f':3,'s':10,'b':12},{'f':13,'s':20,'b':16},{'f':13,'s':12,'b':12},{'f':13,'s':10,'b':12},]
from operator import itemgetter  
print sorted(games ,key = itemgetter('f','s','b'),reverse=True)


#选择某一天，然后以这天为准，次日留存，3日留存，7日留存，14日留存，30日留存
from datetime import datetime,timedelta  
def GetNextDay(baseday,n):  
    return str((datetime.strptime(str(baseday),'%Y-%m-%d')+timedelta(days=n)).date())  

import functools  
selected_day = '2015-02-01'
nday = functools.partial(GetNextDay,selected_day)
print nday(1)
print nday(2)
print nday(6)
print nday(13)
print nday(29)

#用来计算游戏包裹里面的变化情况
def symmetric_difference(_oldobj,_newobj):  
    _oldkeys = _oldobj.keys()  
    _newkeys = _newobj.keys()  
    _diff = {}  
    for _key in set(_oldkeys + _newkeys):  
        _val = _newobj.get(_key,0) - _oldobj.get(_key,0)  
        if _val:  
            _diff[_key] = _val   
    return _diff   
      
oldobj = {'a':1,'b':2,'c':3}  
newobj = {'a':1,'b':3,'d':4}  
print symmetric_difference(oldobj,newobj) 

```


- 输入：python 3.py
- 输出：
    - Counter({'a': 3, 'b': 3, 'c': 2, 'd': 1})
	- [{'s': 20, 'b': 16, 'f': 13}, {'s': 12, 'b': 12, 'f': 13}, {'s': 10, 'b': 12, 'f': 13},  
	- {'s': - 10, 'b': 12, 'f': 3}, {'s': 10, 'b': 12, 'f': 3}]- 
	- 2015-02-02- 
	- 2015-02-03- 
	- 2015-02-07- 
	- 2015-02-14- 
	- 2015-03-02- 
	- {'c': -3, 'b': 1, 'd': 4}- 

>
- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2015年4月
