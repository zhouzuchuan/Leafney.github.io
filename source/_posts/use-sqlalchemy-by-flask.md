---
title: 在Flask中使用SQLAlchemy
date: 2017-01-06 15:21:30
tags:
	- Flask
	- SQLAlchemy
description: 在Flask中使用SQLAlchemy操作数据库
---

在Flask中使用SQLAlchemy操作数据库。

#### 安装 flask-sqlalchemy

```
$ pip install flask
```

```
$ pip install flask-sqlalchemy
```

***

#### 基础

你所有模型的基类叫做 `db.Model` 。它存储在你必须创建的 `SQLAlchemy` 实例上。

在 Flask-sqlalchemy中，表名已自动设置好，除非自己重载它：eg: 

```
__tablename__ = 'students' #指定表名
```

***

##### Column类型

|类型名|Python 类型|说明|
|:----    |:---|:----- |
| `Integer` | int |  整数  |
| `String(size)` |  str | 有最大长度的字符串 |
| `Text` | str  | 长 unicode 文本 |
| `Float` | float | 存储浮点值 |
| `Boolean` | bool | 存储布尔值 |
| `Date` | datetime.date | 日期 |
| `Time` | datetime.time | 时间 |
| `DateTime` | datetime.datetime | 日期和时间 |
| `PickleType` | 任何python对象 | 存储一个持久化 Python 对象 |
| `LargeBinary` | str | 存储任意大的二进制数据 |

##### 常用字段

|选项名|说明| 示例 |
|:----    |:---|:----- |
| `primary_key` | 设置主键 | `primary_key=True` |
| `unique` | 是否唯一 | `unique=True` |
| `index` |  是否创建索引 | `index=True` |
| `nullable` | 是否允许为空 |  `nullable=True` |
| `default` | 设置默认值 | `default=datetime.datetime.utcnow` |

更详细的配置可参考 [Flask Web Development —— 数据库（上） - young - SegmentFault](https://segmentfault.com/a/1190000002362175)

***

##### 连接URI格式

```
dialect+driver://username:password@host:port/database
```

* Mysql `mysql://scott:tiger@localhost/mydatabase`
* Postgres `postgresql://scott:tiger@localhost/mydatabase`
* SQLite `sqlite:////absolute/path/to/foo.db`


##### Flask-SQLAlchemy Sqlite连接路径问题

```
sqlite:////tmp/test.db
```

Sqlite连接字符串中的`/`斜杠说明：三斜杠为相对路径，四斜杠为绝对路径。

```
 'sqlite:////tmp/test.db'                #表示指向绝对路径在Ｔｍｐ目录的test.db文件
 'sqlite:///Data/test.db'                #表示指向相对路径在当前Py文件同目录的Data目录下test.db文件
```

[Flask小记一：Flask-SQLAlchemy Sqlite连接路径问题 - 不折腾难受斯基](https://my.oschina.net/zhangzhe/blog/415084)

***

##### 过滤器

|过滤器|说明|
|:----    |:---|
| `filter` | 把过滤器添加到原查询上，返回一个新查询 |
| `filter_by` | 把等值过滤器添加到原查询上，返回一个新查询 |
| `limit` | 使用指定的值限制返回的结果数量，返回一个新查询 |
| `offset` | 便宜原查询返回的结果， 返回一个新查询 |
| `order_by` |根据指定条件对原查询结果进行排序，返回一个新查询 |
| `group_by` | 根据指定条件对原查询结构进行分组，返回一个新查询 |

##### 执行器

|方法|说明|
|:----    |:---|
| `all` | 以列表形式返回查询的所有结果 |
| `first` | 返回查询的第一个结果，如果没有结果，则返回 None |
| `first_or_404` | 返回查询的第一个结果，如果没有结果，则终止请求，返回 404 错误输出 |
| `get` | 返回指定主键对应的行，如果没有对应的行，则返回 None |
| `get_or_404` | 返回指定主键对应的行，如果没找到指定的主键，则终止请求，返回 404 错误输出 |
| `count` | 返回查询结果的数量 |
| `paginate` | 返回一个 Paginate 对象，它包含指定范围内的结果 |

***

#### 简单示例

##### 入门示例 

参考自官方入门教程示例代码 [Quickstart Flask-SQLAlchemy Documentation (2.1)](http://flask-sqlalchemy.pocoo.org/2.1/quickstart/)

**flsksql.py**

```
# coding:utf-8

from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app=Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI']='sqlite:///test.db'  # 连接当前项目同目录下的test.db数据库文件


app.config['SQLALCHEMY_TRACK_MODIFICATIONS']=True

db=SQLAlchemy(app)

class User(db.Model):
	"""docstring for User"""
	__tablename__='users'
	id=db.Column(db.Integer,primary_key=True)
	username=db.Column(db.String(80),unique=True)
	email=db.Column(db.String(64),unique=True)

	def __init__(self, username,email):
		self.username=username
		self.email=email

	def __repr__(self):
		return '<User %r>' % self.username
```

**app.py**

```
# coding:utf-8

from flask import Flask
from flsksql import db
from flsksql import User

if __name__ == '__main__':
	
	# 初始化数据库
	# db.create_all()
	
	# 新增
	# user1=User('abc','abc@124.com')
	# user2=User('def','def@129.com')
	# db.session.add(user1)
	# db.session.add(user2)
	# 使用commit提交更改
	# db.session.commit()

	# 查询所有数据信息
	# users=User.query.all()
	# print(users)

	# 条件查询
	# admin=User.query.filter_by(username='abc').first()
	# print(admin)

	# 模糊查询 
	# user2_query=User.query.filter(User.username.endswith('f')).first()
	# print(user2_query)
	
	
	# 删除
	# u3=User.query.first()
	# print(u3.username)
	# db.session.delete(u3)
	# db.session.commit()

	# 更改
	# u4=User.query.first()
	# u4.username=u'专升本'  # 中文必须为unicode类型，而不是str类型
	# db.session.commit()
```

***

##### 通过上下文的方式-init_app()


###### database.py--数据操作方法

```
# coding:utf-8

from flask import Flask
from flask_sqlalchemy import SQLAlchemy

db=SQLAlchemy()

def create_app(config_name=None):
	app=Flask(__name__)

	# 在此处加载配置文件
	if config_name is not None:
		# app.config.from_object()  # 默认的config.py
		app.config.from_pyfile(config_name) # 通过配置文件名称加载配置文件

	db.init_app(app)

	# 在此处加载蓝图设置

	with app.app_context():
		# 添加数据对象的引用
		from models import *
		# 初始化数据库
		db.create_all()

	return app

```

###### models.py--Model实体

```
# coding:utf-8

from database import db


class User(db.Model):
	"""docstring for User"""
	__tablename__='users'
	id=db.Column(db.Integer,primary_key=True)
	username=db.Column(db.String(80))
	email=db.Column(db.String(64))

	def __init__(self, username,email):
		self.username=username
		self.email=email


class Address(db.Model):
	"""docstring for Address"""

	id=db.Column(db.Integer,primary_key=True)
	name=db.Column(db.String(32))
	def __init__(self, name):
		self.name=name

```

###### config.py--sql配置文件

```
DEBUG=True
SQLALCHEMY_DATABASE_URI='sqlite:///testabc.db'
SQLALCHEMY_TRACK_MODIFICATIONS=True
```

###### app.py--项目

```
# coding:utf-8

from flask import Flask
from database import db
from database import create_app
from models import User,Address

app=create_app('config.py')

@app.route('/')
def index():
	# user1=User('aaa','aaa@124.com')
	# user2=User(u'马云','mayun@111.com')
	add1=Address('beijing motuoluola')
	# db.session.add(user1)
	db.session.add(add1)
	db.session.commit()
	return "create complate"

@app.route('/show')
def show():
	u=User.query.filter(User.email.startswith('m')).first()
	return u.username

if __name__ == '__main__':
	app.run()
```

###### 项目目录

```
flaskdemo
	app.py
	database.py
	models.py
	config.py
	testabc.db
```

###### init_app() 相关参考

* [【Flask】Flask和SQLAlchemy：init_app - 阿秀的学习笔记        - 博客频道 - CSDN.NET](http://blog.csdn.net/yannanxiu/article/details/53426359)
* [Flask and SQLAlchemy: init_app &middot; Blog](http://piotr.banaszkiewicz.org/blog/2012/06/29/flask-sqlalchemy-init_app/)
* [Introduction into Contexts &#8212; Flask-SQLAlchemy Documentation (2.1)](http://flask-sqlalchemy.pocoo.org/2.1/contexts/)

***

#### 一对多、多对多关系

##### 一对多

待完善。

##### 多对多

待完善。

***

#### 文中所用各类库版本

```
Flask==0.12
Flask-SQLAlchemy==2.1
Jinja2==2.8.1
SQLAlchemy==1.1.4
```

#### 相关参考

* [Flask-SQLAlchemy &mdash; Flask-SQLAlchemy 0.16 documentation--中文版](http://docs.jinkan.org/docs/flask-sqlalchemy/index.html)
* [Flask-SQLAlchemy &mdash; Flask-SQLAlchemy Documentation (2.1)--英文版](http://flask-sqlalchemy.pocoo.org/2.1/) （这两个文档比对着参考，0.16版中有些方法已经过时了，代码按照2.1的来，中文释义参考0.16版的）
* [在 Flask 中使用 SQLAlchemy &mdash; Flask 0.10 documentation](http://dormousehole.readthedocs.io/en/latest/patterns/sqlalchemy.html)
* [flask-sqlalchemy 简单笔记 - 简书](http://www.jianshu.com/p/a52cf3907f29)
* [Flask学习记录之Flask-SQLAlchemy - agmcs - 博客园](http://www.cnblogs.com/agmcs/p/4445583.html)
* [Flask-SQLAlchemy 学习总结 - python 学习 - SegmentFault](https://segmentfault.com/a/1190000004618621)
* [Flask Web Development —— 数据库（上） - young - SegmentFault](https://segmentfault.com/a/1190000002362175) **这个配置说明比较详细**
* [使用 Flask 框架写用户登录功能的Demo时碰到的各种坑（一）——创建应用 - cjnmy36723 - 博客园](http://www.cnblogs.com/cjnmy36723/p/5201551.html)
* [Flask-SQLAlchemy--GitBook](https://wizardforcel.gitbooks.io/flask-extension-docs/content/flask-sqlalchemy.html)
