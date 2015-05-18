---
layout: post
category: "python"
title: "python技术分享[12]－ 利用BeautifulSoup进行采集"
tags: ["python技术分享"]
---

#### 1.BeautifulSoup操作：
- Beautiful Soup 是一个可以从HTML或XML文件中提取数据的Python库.它像其它网络爬虫一样，可以便捷的操作html，
如果你有jquery的使用经验，请使用Beautiful Soup 4.
- 安装时：pip install Beautiful Soup 4 
- 网方文档地址：
<http://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html>


#### 2.糗百内容采集代码：
```
#!/usr/bin/python
#coding=utf-8
‘’‘
功能：演示BeautifulSoup  MySQLdb Process的使用
’‘’
from multiprocessing import Process, Pool, Pipe, cpu_count, active_children
import time,requests
from bs4 import BeautifulSoup
from db.mysql import Custom_MySQL

class RebotQB(Process):

    def __init__(self, child_conn, data):
        Process.__init__(self)
        self.data = data
        self.child_conn = child_conn

    def insert_qiubai_content(self, datas):
        '''
        记录采集入口到数据库
        '''
        self.mysql = Custom_MySQL('test')
        # 查看当年当月当天的入口
        # 演示 mysql count
        sql = 'select count(*) as count from qiubai\
            where year = %(year)s and month = %(month)s and day = %(day)s and page = %(page)s and `index`= %(index)s'% datas

        # print sql
        result = self.mysql.count(sql)
        count = result['count']
        # 如果不存在
        if int(count) == 0:
            # 演示insert
            self.mysql.insert('qiubai', **datas)
        # 如果存在
        else:
            # 演示update
            self.mysql.update('qiubai',
                              ' year = %(year)s and  month =%(month)s and day = %(day)s and  page=%(page)s and `index`=%(index)s' % datas,
                              **datas)
    def run(self):
        print self.data['url']
        # 获取页面内容
        content = requests.get(self.data['url']).text
        if content == None:
            self.child_conn.put({'count': 0})
            return False
        # 变成soup可认内容
        soup = BeautifulSoup(content, from_encoding="utf-8")
        # 抓取笑话内容
        # 规范请参考http://www.crummy.com/software/BeautifulSoup/bs4/doc/
        # 同jquery选择器一样
        tag_index = 0
        for i in soup.select('[id^="qiushi_tag"]'):
            # 确定是第几个笑话
            tag_index = tag_index + 1
            # 去掉内容和图片上次的内容
            try:
                del self.data['content']
                del self.data['img']
            except:
                pass

            # 内容和图片变量
            content = None
            img = None
            # 连接数据库并确认是本页的第几个笑话
            self.mysql = Custom_MySQL('test')
            self.data.update({'index': tag_index})
            # 采集笑话内容
            try:
                content = i.find("div", attrs={'class': "content"})
                content = content.string
                self.data.update({'content': content.encode('gbk')})
            except:
                pass
            # 采集笑话中的图片
            try:
                img = i.find("div", attrs={'class': "thumb"})
                img = img.find('img').attrs['src']
                self.data.update({'img': img})
            except:
                pass
            # 内容入库
            result = self.insert_qiubai_content(self.data)
        # 统计每页要采集多少个笑话
        self.child_conn.put({'count': tag_index})

if __name__ == '__main__':
    from multiprocessing import Process, Queue
    q = Queue()
    data ={'id':2,'url':'http://www.qiushibaike.com/history'}
    r = RebotQB(q,data)
    r.start()

```

#### 3.利用进程池代码：
```
#!/usr/bin/env python
# coding=utf-8
'''
功能：采集糗百笑话，利用进程池,更改了一下系统pool的默认行为支持daemon模式
'''

import multiprocessing
#我们必须显示声明引进multiprocessing模块，而不是Process
import multiprocessing.pool
import time
from random import randint
from multiprocessing import Process


class TaskProcess(Process):
	'''
	替成自己的业务类即可
	'''
    def __init__(self):
        Process.__init__(self)
    def __del__(self):
        pass
    def run(self):
        print 'TaskProcess is running '


class NoDaemonProcess(multiprocessing.Process):
    # 使进程总是daemon模式
    def _get_daemon(self):
        return False
    def _set_daemon(self, value):
        pass
		
    daemon = property(_get_daemon, _set_daemon, None,
                      doc='make "daemon" attribute always return False')

#我们用multiprocessing.pool.Pool 代替 multiprocessing.Pool
#因为这里只有最新的包装器函数, 而不是一个类.
class CustomPool(multiprocessing.pool.Pool):
    Process = NoDaemonProcess

def work(num_procs):
    print("Creating %i (daemon) workers and jobs in child." % num_procs)
    # 自己的业务进程
    p = TaskProcess()
    p.start()

def test():
    print("Creating 5 (non-daemon) workers and jobs in main process.")
    year = [x for x in range(2008, 2014)]
    pool = CustomPool(len(year) * 4)
    result = pool.map(work, year)
    pool.close()
    pool.join()
    # print(result)

if __name__ == '__main__':
    test()

```

>
- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2015年5月
