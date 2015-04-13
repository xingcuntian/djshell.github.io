---
layout: post
category: "python"
title: "python培训[2]"
tags: ["友盟统计"]
---
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
