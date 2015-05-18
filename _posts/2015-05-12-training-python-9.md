---
layout: post
category: "python"
title: "python技术分享[9]－ 数据库操作之MongoDB"
tags: ["python技术分享"]
---

#### 1.数据库操作：
>
	数据操作在工作中最常用的操作类库之一，今天贴出来我常用的pymongo操作类，一直在用，也一直在不断优化，暂时可以满足业务


#### 2.接口代码：

```
#!/usr/bin/env python
#coding=utf-8


from abc import ABCMeta  
from abc import abstractproperty
from abc import abstractmethod 


class Custom_Interface(object):
    
    __metaclass__ = ABCMeta



    @abstractmethod
    def close(self):
        """
        关闭当前数据库句柄
        """
        return
    @abstractmethod
    def query(self, query, *parameters):
        """
        @return: 返回一个list，多个结果。
        @param query:SQL语句
        @param parameters:SQL语句参数
        """
        return
       
    @abstractmethod
    def get(self, query, *parameters):
        """
        @return: 返回单个结果
        @param query:SQL语句
        @param parameters:SQL语句参数
        """
        return
       
    @abstractmethod
    def count(self,query, *parameters):
        """
        @return: 返回单个结果
        @param query:SQL语句
        @param parameters:SQL语句参数
        """
        return
       
    @abstractmethod
    def insert(self,table,**datas):
        '''
        @param table:表名
        @param datas:｛字段：值｝
        '''
        return
       
    @abstractmethod
    def update(self,table,where,**datas):
        '''
        @param table:表名
        @param datas:｛字段：值｝
        '''
        return
        
    @abstractmethod
    def delete(self,table,where):
        '''
        @param table:表名
        @param datas:｛字段：值｝
        '''
        return
  

```

#### 2.类实现代码：

```

#/usr/bin/python
#coding=utf-8

"""
auth:.启
createtime:2014-6-17下午12:13:07
usege:

"""

import sys


try:
    import pymongo

except ImportError:
    print >> sys.stderr,"""\

There was a problem importing Python modules(pymongo) required.
The error leading to this problem was:
%s Please install a package which provides this module, or
verify that the module is installed correctly.
you can use this cmd :pip install pymongo

It's possible that the above module doesn't match the current version of Python,
which is:
%s
""" % (sys.exc_info(), sys.version)

from interface import Custom_Interface

class Custom_Mongo(Custom_Interface):
    
    def __init__(self,using=''):

        """
        @param cursor_hander:数据库句柄
        """
        self.cursor = None
        self.cursor_hander = using
        self.connections = None
        self.conn  = None
        
        if str(self.cursor_hander).rstrip() == '':
            print 'please write Custom_MySQL`s using param'
            exit(0)
   
            

        databases ={
            'logs':{'host':'127.0.0.1', 'user':'readonly','password':'readonly', 'database':'local','charset':'utf8','port':27017,'connect_timeout':50},
            
            }
        
        database = databases[self.cursor_hander]
        
        #建立和数据库系统的连接,创建Connection时，指定host及port参数
        self.conn   = pymongo.Connection(host=database['host'],port=database['port'])
        self.database = self.conn[database['database']]
        #admin 数据库有帐号，连接-认证-切换库
        #db_auth = conn.admin
        #db_auth.authenticate('sa','sa')

        
    def __del__(self):
        
        self.close()
        
     
    def close(self):
        """
        关闭当前数据库句柄
        """
        if self.conn != None:
            return self.conn.disconnect()
     
    def query(self, table_name, parameters,skip=None,limit=None):
        """
        @return: 返回一个list，多个结果。
        @param table_name:表名
        @param parameters:SQL语句参数
        """
        result = self.database[table_name].find(parameters)
        
        if skip !=None:
            result.skip(skip)
        if limit !=None:
                
            result.limit(limit) 
            
        return result
       
     
    def get(self, table_name,**parameters):
        """
        @return: 返回单个结果
        @param table_name:表名
        @param parameters:SQL语句参数
        """
        return self.database[table_name].find_one(parameters)
       
     
    def count(self,table_name, **parameters):
        """
        @return: 返回单个结果
        @param table_name:表名
        @param parameters:SQL语句参数
        """
        return self.database[table_name].find(parameters).count()
       
     
    def insert(self,table,**datas):
        '''
        @param table:表名
        @param datas:｛字段：值｝
        '''
        return self.database[table].insert(datas)
       
     
    def update(self,table,where,**datas):
        '''
        @param table:表名
        @param datas:｛字段：值｝
        '''
        return self.database[table].update(where,datas)
        
     
    def delete(self,table,where):
        '''
        @param table:表名
        @param datas:｛字段：值｝
        '''
        return self.database[table].remove(where)

```

#### 3.用例代码：

```
#/usr/bin/python
#coding=utf-8

'''
date:2014-06-17 12:00

'''
from custom.db.mongo import Custom_Mongo

n = Custom_Mongo(using='logs')

param = {'sex':'man','name':'wuqichao'}
print n.insert('users',**param)

param = {'sex':'man','name':'wuqichao'}
print n.get('users',**param)

param = {'sex':'man','name':'wuqichao'}
print n.count('users', **param)

n.update('users',{'sex':'man','name':'wuqichao'},**{'sex':'wman','name':'wuqichao'})

param = {'sex':'man','name':'wuqichao'}
res =  n.query('users',param,skip=2,limit=2)
for r in res:
    print r

```

>
- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2015年5月
