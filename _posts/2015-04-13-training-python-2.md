---
layout: post
category: "python"
title: "python培训[2]"
tags: ["友盟统计"]
---

* 定义:
	* list:链表,有序的项目, 通过索引进行查找,使用方括号”[]”;
	* tuple:元组,元组将多样的对象集合到一起,不能修改,通过索引进行查找, 使用括号”()”;
	* dict:字典,字典是一组键(key)和值(value)的组合,通过键(key)进行查找,没有顺序, 使用大括号”{}”;
	* set:集合,无序,元素只出现一次, 自动去重,使用”set([])”
	

```
#样例
mylist = [1, 2, 3, 4, 'Oh']  
mytuple = (1, 2, 'Hello', (4, 5))  
mydict = {'Wang' : 1, 'Hu' : 2, 'Liu' : 4}  
myset = set(['Wang', 'Hu', 'Liu', 4, 'Wang'])  
  
print(mylist)  
print(mytuple)  
print(mydict)  
print(myset)  

```
* 输出

	* [1, 2, 3, 4, 'Oh']  
	* (1, 2, 'Hello', (4, 5))  
	* {'Liu': 4, 'Wang': 1, 'Hu': 2}  
	* set(['Liu', 4, 'Wang', 'Hu']) 
	


我们重点讲字典和列表

####一、List操作
```
#!/usr/bin/python
# coding=utf-8
###################################################
#  
#  功能：python入门引导 
#        
####################################################

#定义函数
def list_test():
    #初始化列表
    sample_list = ['a','b',0,1,3]
    print sample_list
      
    #得到列表中的某一个值
    value_start = sample_list[1:3]
    print value_start
      
    end_value = sample_list[-1]
    print end_value
      
    #删除列表的第一个值
    del sample_list[0]
      
    #在列表中插入一个值
    sample_list[0] = ['sample value']
    sample_list[2] = 'second'
    sample_list.append('three')
      
    #得到列表的长度
    list_length = len(sample_list)
    print list_length
      
    #列表遍历
    for element in sample_list:
        print(element)
      
    #用同一个值初始化，形成一个含有六个元素的列表
    sample_list = ['test']*6
    for element in sample_list:
        print(element)
        
if __name__ == '__main__':
    #调用函数
    list_test()
	
```

```
#!/usr/bin/python
#coding=utf-8


###################################################
#  
#  功能：python入门引导 
#        
####################################################

def dict_test():
    #初始化字典    
    sample_dict = {'a':'a','2':'b',3:'c'}
    print sample_dict

    #获取指定的key的值
    pop = sample_dict.pop('2')
    print pop
    print sample_dict

    get = sample_dict.get('a')
    print get
    print sample_dict

    print sample_dict[3]

    #删除指定key的值
    pop = sample_dict.pop(3)
    print pop
    print sample_dict

    #在字典中插入一个值
    sample_dict.update({3:'c','2':'b'})
    print sample_dict

    #获取字典长度
    print len(sample_dict)

    #字典遍历
    for element in sample_dict:
        print element,sample_dict[element]

    for (k, v) in sample_dict.items():
        print "sample_dict[%s] =" % k, v

if __name__ == '__main__':
    dict_test()
    
```


>
- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2015年4月
