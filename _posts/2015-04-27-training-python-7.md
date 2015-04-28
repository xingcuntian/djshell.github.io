---
layout: post
category: "python"
title: "python技术分享[7]－ 自定义数据库连接池"
tags: ["python技术分享"]
---

#### 1.数据库连接池原理：
>
连接池基本的思想是在系统初始化的时候，将数据库连接作为对象存储在内存中，当用户需要访问数据库时，并非建立一个新的连接，而是从连接池中取出一个已建立的空闲连接对象。使用完毕后，用户也并非将连接关闭，而是将连接放回连接池中，以供下一个请求访问使用。而连接的建立、断开都由连接池自身来管理。同时，还可以通过设置连接池的参数来控制连接池中的初始连接数、连接的上下限数以及每个连接的最大使用次数、最大空闲时间等等。也可以通过其自身的管理机制来监视数据库连接的数量、使用情况等。


#### 2.代码：

```
#!/usr/bin/env python
#coding=utf-8
 


class PooledConnection:

    def __init__(self, maxconnections, dbtype):
        from Queue import Queue
        self._pool = Queue(maxconnections) # create the queue
    
        self.dbtype=dbtype
        self.maxconnections=maxconnections
        try:
            for i in range(maxconnections):
                self.fillConnection(self.CreateConnection(dbtype))
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
            self.fillConnection(self.CreateConnection(dbtype))
        except Exception,e:
            raise "CloseConnection error:"+str(e)

    def CreateConnection(self, dbtype):
        try:
            conndb=MySQLdb.connect(db='test',host='127.0.0.1',user='root',passwd='');
            conndb.clientinfo = 'datasync connection pool from datasync.py';
            conndb.ping();
            return conndb
        except Exception, e:
            raise 'conn targetdb datasource Excepts,%s!!!(%s).'%('127.0.0.1',str(e))
            return None
			
DB = PooledConnection(10,"mysql")
    
def sub_mod( ip, data):
	
	global DB
	conn = DB.getConnection()
	cursor = conn.cursor()
	cursor.execute("""insert into pool (name) values('wwwww')""")
	#result = cursor.fetchall();
	#print  cursor.description
	

    
class MtaskConnection(object):
     
     
    stream_set = set([])
    
 
    def __init__(self,stream, address):
        
        self.stream = stream
        self.address = address
        self.stream_set.add(self.stream)
        self.stream.set_close_callback(self._on_close)
        self.stream.read_until('\n', self._on_read_line)
        
 
    def _on_read_line(self, data):
 

        sub_mod(self.address[0], data)
      
        
    def _on_close(self):
   
        MtaskConnection.stream_set.remove(self.stream)
         
class MonitorTCPServer(TCPServer):
    
    def __init__(self):
        TCPServer.__init__(self)  
		
    def handle_stream(self, stream, address):
        MtaskConnection(stream,address)
		
	#def __del__(self):
	#	self.pool.ColseConnection()
         

     
def main():
    
    from tornado import process
	from tornado import web,netutil
	from tornado.tcpserver import TCPServer
	from tornado import ioloop

	import MySQLdb
	import json,time
	import sys,os,string
	import socket, select
	import syslog
	import base64

    sockets = netutil.bind_sockets(32777)
    process.fork_processes(0)
    server = MonitorTCPServer()
    server.add_sockets(sockets)
    ioloop.IOLoop.instance().start()


#经两次fork实现脱终端，成为守护进程
def daemon():
    try:
        pid = os.fork()
        if pid > 0:
            sys.exit(0)
        os.setsid()
        os.umask(0)
        pid = os.fork()
        if pid > 0: # exit from second parent
            sys.exit(0)


        for i in range(65):
            try:
                os.close(i)
            except:
                continue

        sys.stdin = open("/dev/null", "r+")
        sys.stdout = sys.stdin
        sys.stderr = sys.stdin

    except OSError, e:
        syslog.syslog(syslog.LOG_ERR, "fork failed: (%s)" % e)
        sys.exit(1)

    
     
if __name__ == '__main__':
	 main()
     #syslog.openlog("monitor_svr", 0, syslog.LOG_LOCAL6)
     #daemon()
     #syslog.closelog()

```
>
- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2015年4月
