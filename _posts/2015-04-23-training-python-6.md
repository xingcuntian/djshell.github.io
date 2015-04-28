---
layout: post
category: "python"
title: "python技术分享[6]－ 自定义数据库连接池"
tags: ["python技术分享"]
---

#### 1.数据库连接池原理：
>
连接池基本的思想是在系统初始化的时候，将数据库连接作为对象存储在内存中，当用户需要访问数据库时，并非建立一个新的连接，而是从连接池中取出一个已建立的空闲连接对象。使用完毕后，用户也并非将连接关闭，而是将连接放回连接池中，以供下一个请求访问使用。而连接的建立、断开都由连接池自身来管理。同时，还可以通过设置连接池的参数来控制连接池中的初始连接数、连接的上下限数以及每个连接的最大使用次数、最大空闲时间等等。也可以通过其自身的管理机制来监视数据库连接的数量、使用情况等。


#### 2.代码：

```
#!/usr/bin/python
# coding=utf-8

import MySQLdb
import time
import string
 
class PooledConnection:
 
    def __init__(self, maxconnections, connstr,dbtype):
        from Queue import Queue
        self._pool = Queue(maxconnections) # create the queue
        self.connstr = connstr
        self.dbtype=dbtype
        self.maxconnections=maxconnections
        try:
            for i in range(maxconnections):
                self.fillConnection(self.CreateConnection(connstr,dbtype))
        except Exception,e:
            raise e
 
    def fillConnection(self,conn):
        try:
            self._pool.put(conn)
 
        except Exception,e:
            raise "fillConnection error:"+str(e)
 
    def returnConnection(self, conn):
        try:
            self._pool.put(conn)
        except Exception,e:
            raise "returnConnection error:"+str(e)
 
    def getConnection(self):
        try:
            return self._pool.get()
        except Exception,e:
            raise "getConnection error:"+str(e)
 
    def ColseConnection(self,conn):
        try:
            self._pool.get().close()
            self.fillConnection(self.CreateConnection(connstr,dbtype))
        except Exception,e:
            raise "CloseConnection error:"+str(e)
 
    def CreateConnection(self,connstr,dbtype):
        try:
            conndb=MySQLdb.connect(db='test',host='127.0.0.1',user='root',passwd='');
            conndb.clientinfo = 'datasync connection pool from datasync.py';
            conndb.ping();
            return conndb
        except Exception, e:
            print 'ssssssssss'
            raise 'conn targetdb datasource Excepts,%s!!!(%s).'%('127.0.0.1',str(e))
            return None
 
#创建连接池：
connstring="rooot#' '#127.0.0.1:3306/test"
pool=PooledConnection(10,connstring,"mysql");
#获取连接：
conn = pool.getConnection()
time.sleep(15)
cursor = conn.cursor()
cursor.execute("""select * from emp""")
result = cursor.fetchall();
print  cursor.description
conn.close();

```
>
- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2015年4月
