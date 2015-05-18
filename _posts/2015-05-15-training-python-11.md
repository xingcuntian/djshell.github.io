---
layout: post
category: "python"
title: "python技术分享[11]－ 日志－日志记录到mongodb中"
tags: ["python技术分享"]
---

#### 1.日志操作：
>
	日志文件操作在工作中最常用的操作类库之一，今天贴出来我常用的文件操作类，一直在用，也一直在不断优化，暂时可以满足业务


#### 2.日志操作代码：

```
import logging
import getpass
from datetime import datetime
from socket import gethostname
from pymongo.connection import Connection
from bson import InvalidDocument


class MongoFormatter(logging.Formatter):
    def format(self, record):
        """Format exception object as a string"""
        data = record.__dict__.copy()

        if record.args:
            record.msg = record.msg % record.args

        data.update(
            username=getpass.getuser(),
            time=datetime.now(),
            host=gethostname(),
            message=record.msg,
            args=tuple(unicode(arg) for arg in record.args)
        )
        if 'exc_info' in data and data['exc_info']:
            data['exc_info'] = self.formatException(data['exc_info'])
        return data
    

class MongoHandler(logging.Handler):
    """ Custom log handler

    Logs all messages to a mongo collection. This  handler is 
    designed to be used with the standard python logging mechanism.
    """

    @classmethod
    def to(cls, db, collection, host='localhost', port=None, level=logging.NOTSET):
        """ Create a handler for a given  """
        return cls(Connection(host, port)[db][collection], level)
        
    def __init__(self, collection, db='mongolog', host='localhost', port=None, level=logging.NOTSET):
        """ Init log handler and store the collection handle """
        logging.Handler.__init__(self, level)
        if (type(collection) == str):
            self.collection = Connection(host, port)[db][collection]
        else:
            self.collection = collection
        self.formatter = MongoFormatter()

    def emit(self,record):
        """ Store the record to the collection. Async insert """
        try:
            self.collection.save(self.format(record))
        except InvalidDocument, e:
            logging.error("Unable to save log record: %s", e.message ,exc_info=True)


```

>
- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2015年5月
