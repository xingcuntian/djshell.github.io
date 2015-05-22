---
layout: post
category: "python"
title: "python技术分享[13]－ Flask实现restful接口"
tags: ["python技术分享"]
---

#### 1.安装扩展
- pip install flask
- pip install flask-restful
- pip install sqlalchemy
- 网方文档地址：
<http://flask-restful.readthedocs.org/en/0.3.3/>


#### 2.models代码：
```
#!/usr/bin/python
#coding=utf-8

‘’‘
功能：定义数据表结构，生成需要生成数据表时，直接python models.py
’‘’
from sqlalchemy import Column
from sqlalchemy import Integer
from sqlalchemy import String
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class Todo(Base):
    __tablename__ = 'todos'

    id = Column(Integer, primary_key=True)
    task = Column(String(255))
‘’‘
if __name__ == "__main__":
    from sqlalchemy import create_engine
    from settings import DB_URI
    engine = create_engine(DB_URI)
    Base.metadata.drop_all(engine)
    Base.metadata.create_all(engine)

’‘’
```


#### 3.db代码：
```
#!/usr/bin/env python
# coding=utf-8


from sqlalchemy import create_engine
from sqlalchemy.orm import scoped_session
from sqlalchemy.orm import sessionmaker
#from settings import DB_URI

DB_URI = 'mysql://root:@localhost:3306/test?charset=utf8'

Session = sessionmaker(autocommit=False,
                       autoflush=False,
                       bind=create_engine(DB_URI))
session = scoped_session(Session)

```


#### 4.resources代码：
```
#!/usr/bin/env python
# coding=utf-8
'''
功能：引入上面的models文件，将数据表数据进行对象化，定义restful资源，返回前数据利用marshal_with装饰器做序列化
'''
from models import Todo
from db import session

from flask.ext.restful import reqparse
from flask.ext.restful import abort
from flask.ext.restful import Resource
from flask.ext.restful import fields
from flask.ext.restful import marshal_with

todo_fields = {
    'id': fields.Integer,
    'task': fields.String,
    'uri': fields.Url('todo', absolute=True),
}

parser = reqparse.RequestParser()
parser.add_argument('task', type=str)

class TodoResource(Resource):
    @marshal_with(todo_fields)
    def get(self, id):
        todo = session.query(Todo).filter(Todo.id == id).first()
        if not todo:
            abort(404, message="Todo {} doesn't exist".format(id))
        return todo

    def delete(self, id):
        todo = session.query(Todo).filter(Todo.id == id).first()
        if not todo:
            abort(404, message="Todo {} doesn't exist".format(id))
        session.delete(todo)
        session.commit()
        return {}, 204

    @marshal_with(todo_fields)
    def put(self, id):
        parsed_args = parser.parse_args()
        print parsed_args
        todo = session.query(Todo).filter(Todo.id == id).first()
        todo.task = parsed_args['task']
        session.add(todo)
        session.commit()
        return todo, 201


class TodoListResource(Resource):
    @marshal_with(todo_fields)
    def get(self):
        todos = session.query(Todo).all()
        return todos

    @marshal_with(todo_fields)
    def post(self):
        parsed_args = parser.parse_args()
        todo = Todo(task=parsed_args['task'])
        session.add(todo)
        session.commit()
        return todo, 201

```

#### 5.app代码：
```
#!/usr/bin/env python
# coding=utf-8


from flask import Flask
from flask.ext.restful import Api

app = Flask(__name__)
api = Api(app)

from resources import TodoListResource
from resources import TodoResource

api.add_resource(TodoListResource, '/todos', endpoint='todos')
api.add_resource(TodoResource, '/todos/<string:id>', endpoint='todo')

if __name__ == '__main__':
    app.run(debug=True)
    

```

#### 5.test代码：
```
import requests, json
print requests.get('http://localhost:5000/todos').json()
print requests.post('http://localhost:5000/todos',
                 headers={'Content-Type': 'application/json'},
                 data=json.dumps({'task': 'go outside!'})).json()
print requests.get('http://localhost:5000/todos/1').json()
print requests.put('http://localhost:5000/todos/1',
                headers={'Content-Type': 'application/json'},
                data=json.dumps({'task': 'go to the gym'})).json()
print requests.delete('http://localhost:5000/todos/1')
print requests.get('http://localhost:5000/todos').json()
    

```
>
- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2015年5月
