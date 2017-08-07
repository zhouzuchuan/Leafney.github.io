### Python操作SQLAlchemy


#### 安装

通过pip安装SQLAlchemy

```
$ pip install sqlalchemy
```

***

##### 初始化数据库连接

```
# 初始化数据库连接
engine=create_engine('sqlite:///./cnblogblog.db',echo=True)
```

其中，`echo=True` 表示 是否将执行过程中的sql语句进行输出显示

##### 常用数据库连接写法

###### 整理常用-直接拷贝

1. sqlite内存：`engine = create_engine('sqlite:///:memory:', echo=True)`
2. sqlite文件: `engine=create_engine('sqlite:///./cnblogblog.db',echo=True)`
3. mysql+pymysql：`engine = create_engine("mysql+pymysql://username:password@hostname:port/dbname",echo=True)`
4. mssql+pymssql: `engine = create_engine('mssql+pymssql://username:password@hostname:port/dbname',echo=True)`


***

```
>>> from sqlalchemy import create_engine
```

`create_engine()` 用来初始化数据库连接。SQLAlchemy用一个字符串表示连接信息：

```
'数据库类型+数据库驱动名称://用户名:口令@机器地址:端口号/数据库名'
```


1. sqlite 内存 示例 `engine = create_engine('sqlite:///:memory:', echo=True)`
2. sqlite 文件 示例 `engine=create_engine('sqlite:///./cnblogblog.db',echo=True)`
3. mysql通用 `engine = create_engine('mysql+mysqlconnector://root:password@localhost:3306/test')`
4. mysql+pymysql 示例 `engine = create_engine("mysql+pymysql://username:password@hostname/dbname", encoding="utf8", echo=True)`
5. postgresql 示例 `engine = create_engine('postgresql://scott:tiger@localhost:5432/mydatabase')`


**PostgreSQL**

```
# default
engine = create_engine('postgresql://scott:tiger@localhost/mydatabase')

# psycopg2
engine = create_engine('postgresql+psycopg2://scott:tiger@localhost/mydatabase')

# pg8000
engine = create_engine('postgresql+pg8000://scott:tiger@localhost/mydatabase')
```

**MySQL**

```
# default
engine = create_engine('mysql://scott:tiger@localhost/foo')

# mysql-python
engine = create_engine('mysql+mysqldb://scott:tiger@localhost/foo')

# MySQL-connector-python
engine = create_engine('mysql+mysqlconnector://scott:tiger@localhost/foo')

# OurSQL
engine = create_engine('mysql+oursql://scott:tiger@localhost/foo')
```

**Oracle**

```
engine = create_engine('oracle://scott:tiger@127.0.0.1:1521/sidname')

engine = create_engine('oracle+cx_oracle://scott:tiger@tnsname')
```

**MS SQL**

```
# pyodbc
engine = create_engine('mssql+pyodbc://scott:tiger@mydsn')

# pymssql
engine = create_engine('mssql+pymssql://scott:tiger@hostname:port/dbname')
```

**SQLite**

```
# sqlite://<nohostname>/<path>
# where <path> is relative:
engine = create_engine('sqlite:///foo.db')

#Unix/Mac - 4 initial slashes in total
engine = create_engine('sqlite:////absolute/path/to/foo.db')
#Windows
engine = create_engine('sqlite:///C:\\path\\to\\foo.db')
#Windows alternative using raw string
engine = create_engine(r'sqlite:///C:\path\to\foo.db')

#memory
engine = create_engine('sqlite://')
```

详见官网文档：[Engine Configuration](http://docs.sqlalchemy.org/en/latest/core/engines.html)

***

##### 如何设置初始化表结构时字段的 `主键` `自增` 等属性

sqlite中如果设置主键自增，还需要添加 `__table_args__` 参数，示例如下：

```
class Person(Base):
    __tablename__ = "person"
    __table_args__ = {'sqlite_autoincrement': True}

	id=Column(Integer,primary_key=True,autoincrement=True)
```

* [sqlalchemy - Pylons, SQlite and autoincrementing fields - Stack Overflow](http://stackoverflow.com/questions/4567574/pylons-sqlite-and-autoincrementing-fields)

##### 设置表结构的 `不可空`  `默认值`  `唯一` 等属性

```
# 定义User对象
class User(Base):
	"""Users table"""
	# 表的名字
	__tablename__='users'
	__table_args__={'sqlite_autoincrement': True}
	# 表结构
	id=Column(Integer,primary_key=True,autoincrement=True)
	name=Column(String(32),nullable=False)
	age=Column(Integer,default=0)
	password=Column(String(64),unique=True)
```

##### 插入中文数据

直接插入中文数据，可能会报如下错误信息：

> sqlalchemy.exc.ProgrammingError: (sqlite3.ProgrammingError) You must not use 8-bit bytestrings unless you use a text_factory that can interpret 8-bit bytestrings (like text_factory = str). It is highly recommended that you instead just switch your application to Unicode strings.

相应的解决方法是：将 `str` 类型的中文转成 `unicode` 类型再插入即可。

示例代码如下:

```
# 添加一条数据
def addUserForZhCn():
	session=DBSession()
	new_user=User(name=u'关羽2',password='12322233')
	session.add(new_user)
	session.commit()
	session.close()
```

参考自：

> Python 官方文档中不建议使用这种方式：use of sys.setdefaultencoding() has always been discouraged，在文件头写上 # coding: utf-8 之类的注释并且在 Unicode 字符串前加上 `u` 就可以了。

[Flask Sqlalchemy中文模糊搜索错误](http://python-china.org/t/1148)

***

##### Mysql 指定表的引擎和编码格式

```
from sqlalchemy import Column, Integer, String

class User(BaseModel):
	__tablename__='users'
	__table_args__={
		"mysql_engine":"InnoDB",   # 表的引擎
		"mysql_charset":"utf8"   # 表的编码格式
	}

	id=Column("id",Integer,primary_key=True,autoincrement=True)
	name=Column("name",String(50),nullable=False)
	age=Column("age",Integer,default=0)

```

***

##### 如果记录存在则修改，不存在则添加

```
session.merge()
```

##### 模型的属性名称和表的字段名称不一致

```
	id=Column("id",Integer,primary_key=True,autoincrement=True)
	name=Column("name",String(50),nullable=False)
```

##### 增删改  查询

增删改需要 `commit` 操作 :

```
	session=DBSession()
	duser=session.query(User).filter(User.id==2).delete()
	session.commit()
	session.close()
```

查询不需要 `commit` 操作: 

```
	session=DBSession()
	quser=session.query(User).filter(User.id==4).one()
	print('name:',quser.name)
	session.close()
```

***

##### first() 和 one() 的区别

query.first()：返回第一个元素
query.one()有且只有一个元素时才正确返回

first()方法限制并仅作为标量返回结果集的第一条记录

one()方法，完整的提取所有的记录行，并且如果没有明确的一条记录行(没有找到这条记录)或者结果中存在多条记录行，将会引发错误异常NoResultFound或者MultipleResultsFound。

当没有数据行返回时，使用 `one()` 方法会报错，可以使用 `one_or_none()` 方法来代替，当没有数据时，会返回 `None` 而不是异常。


***

##### 执行sql语句

绑定参数也可以用基于字符串的SQL指派，使用冒号来标记替代参数，然后再使用params()方法指定相应的值。

```
session.query(User).filter("id<:value and name=:name").\
params(value=224, name='fred').order_by(User.id).one() 
```

**execute() 方法**

```
	s=DBSession()
	# 不能用 `?` 的方式来传递参数 要用 `:param` 的形式来指定参数
	# s.execute('INSERT INTO users (name, age, password) VALUES (?, ?, ?)',('bigpang',2,'1122121'))  
	# 这样执行报错 
	
	# s.execute('INSERT INTO users (name, age, password) VALUES (:aa, :bb, :cc)',({'aa':'bigpang2','bb':22,'cc':'998'}))
	# s.commit()
	# 这样执行成功

	res=s.execute('select * from users where age=:aaa',{'aaa':4})
	# print(res['name'])  # 错误
	# print(res.name)    # 错误
	# print(type(res))   # 错误
	for r in res:
		print(r['name'])
		
	s.close()
```

可参考：[python - How to execute raw SQL in SQLAlchemy-flask app - Stack Overflow](http://stackoverflow.com/questions/17972020/how-to-execute-raw-sql-in-sqlalchemy-flask-app)

***

##### 执行sql语句 高级

* 执行sql语句，可以使用传统的 `connection` 方式，也可以使用 `session` 方式
* sqlalchemy下的传统connection方式，执行sql语句时不需要 `cursor` 光标，执行增删改直接生效，执行sql语句不需要 `commit` 操作。
* sqlalchemy下的传统connection方式，参数形式与传统方式相同，使用 `?` 占位，元祖形式传值
* sqlalchemy下的session方式，执行增删改需要 `commit` 操作。
* sqlalchemy下的session方式，参数形式为 dict, 在sql语句中使用 `:key` 占位，dict形式传值

***

#### 示例代码

```
	# **传统 connection方式**
	# 创建一个connection对象，使用方法与调用python自带的sqlite使用方式类似
	# 使用with 来创建 conn，不需要显示执行关闭连接
	# with engine.connect() as conn:
	# 	res=conn.execute('select * from users')
	# 	data=res.fetchone()
	# 	print('user is %s' %data[1])

	# 与python自带的sqlite不同，这里不需要 cursor 光标，执行sql语句不需要commit。如果是增删改，则直接生效，也不需要commit.
	
	# **传统 connection 事务**
	with engine.connect() as conn:
		trans=conn.begin()
		try:
			r1=conn.execute("select * from users")
			print(r1.fetchone()[1])
			r2=conn.execute("insert into users (name,age,password) values (?,?,?)",('tang',5,'133444'))
			trans.commit()
		except:
			trans.rollback()
			raise

	# **session**
	session=DBSession()

	session.execute('select * from users')
	session.execute('insert into users (name,age,password) values (:name,:age,:password)',{"name":'dayuzhishui','age':6,'password':'887'})
	# 注意参数使用dict，并在sql语句中使用:key占位

	# 如果是增删改，需要 commit
	session.commit()
	# 用完记得关闭，也可以用 with
	session.close()
```

详情可见：[sqlalchemy学习笔记 - python学习笔记 - SegmentFault](https://segmentfault.com/a/1190000006949536)

***

#### 完整测试代码

```
# coding:utf-8

from sqlalchemy import create_engine

from sqlalchemy.ext.declarative import declarative_base

from sqlalchemy import Column, Integer, String

from sqlalchemy.orm import sessionmaker

# ***************************

# 初始化数据库连接
engine=create_engine('sqlite:///./cnblogblog.db',echo=True)

# 创建对象的基类
Base=declarative_base()
# 创建会话类
DBSession=sessionmaker(bind=engine)

# ******************

# 定义User对象
class User(Base):
	"""Users table"""
	# 表的名字
	__tablename__='users'
	__table_args__={'sqlite_autoincrement': True}
	# 表结构
	id=Column(Integer,primary_key=True,autoincrement=True)
	name=Column(String(32),nullable=False)
	age=Column(Integer,default=0)
	password=Column(String(64),unique=True)


class Blog(Base):
	"""docstring for Blog"""

	__tablename__='blogs'

	id=Column(Integer,primary_key=True)
	title=Column(String(100))
	desc=Column(String(500))

class Tips(Base):
	"""docstring for Tips"""
	
	__tablename__='tips'
		
	id=Column(Integer,primary_key=True)
	name=Column(String(32))

# ***********************

# 添加一条数据
def newUser():
	# 创建会话对象
	session=DBSession()

	new_user=User(name='Jery',password='123')

	session.add(new_user)

	session.commit()
	session.close()

# 添加一条数据
def addUserForZhCn():
	session=DBSession()
	new_user=User(name=u'关羽2',password='12322233')
	session.add(new_user)
	session.commit()
	session.close()

# 新增多条数据
def addmoreUser():
	session=DBSession()
	session.add_all([
		User(name='guanyu',age=4,password='11111'),
		User(name='zhangfei',password='2233'),
		User(name='zhenji',password='44556')
		])
	session.commit()
	session.close()

# 查询
def queryUser():
	session=DBSession()
	quser=session.query(User).filter(User.id==4).one()
	print('name:',quser.name)
	session.close()

# 删除
def deleteUser():
	session=DBSession()
	duser=session.query(User).filter(User.id==2).delete()
	session.commit()
	session.close()

# 执行sql语句
def SQlUser():
	s=DBSession()
	# 不能用 `?` 的方式来传递参数 要用 `:param` 的形式来指定参数
	# s.execute('INSERT INTO users (name, age, password) VALUES (?, ?, ?)',('bigpang',2,'1122121'))  
	# 这样执行报错 
	
	# s.execute('INSERT INTO users (name, age, password) VALUES (:aa, :bb, :cc)',({'aa':'bigpang2','bb':22,'cc':'998'}))
	# s.commit()
	# 这样执行成功

	res=s.execute('select * from users where age=:aaa',{'aaa':4})
	# print(res['name'])  # 错误
	# print(res.name)    # 错误
	# print(type(res))   # 错误
	for r in res:
		print(r['name'])

	s.close()

# 执行sql语句
def SQlUser2():

	# **传统 connection方式**
	# 创建一个connection对象，使用方法与调用python自带的sqlite使用方式类似
	# 使用with 来创建 conn，不需要显示执行关闭连接
	# with engine.connect() as conn:
	# 	res=conn.execute('select * from users')
	# 	data=res.fetchone()
	# 	print('user is %s' %data[1])

	# 与python自带的sqlite不同，这里不需要 cursor 光标，执行sql语句不需要commit。如果是增删改，则直接生效，也不需要commit.
	
	# **传统 connection 事务**
	with engine.connect() as conn:
		trans=conn.begin()
		try:
			r1=conn.execute("select * from users")
			print(r1.fetchone()[1])
			r2=conn.execute("insert into users (name,age,password) values (?,?,?)",('tang',5,'133444'))
			trans.commit()
		except:
			trans.rollback()
			raise

	# **session**
	session=DBSession()

	session.execute('select * from users')
	session.execute('insert into users (name,age,password) values (:name,:age,:password)',{"name":'dayuzhishui','age':6,'password':'887'})
	# 注意参数使用dict，并在sql语句中使用:key占位

	# 如果是增删改，需要 commit
	session.commit()
	# 用完记得关闭，也可以用 with
	session.close()


# 更多操作
def TestUser():
	session=DBSession()

	# test1
	# 使用merge方法，如果存在则修改，如果不存在则插入（只判断主键，不判断unique列）
	# t1=session.query(User).filter(User.name=='zhenji').first()
	# t1.age=34
	# session.merge(t1)
	# session.commit()

	# test2
	# merge方法，如果数据库中没有则添加
	# t2=User()
	# t2.name='haha'
	# session.merge(t2)
	# session.commit()


	# test3
	# 获取第2-3项
	# tUser=session.query(User)[1:3]   
	# for u in tUser:
	# 	print(u.id)

	# test4
	# 


if __name__ == '__main__':
	
	# 删除全部数据库
	# Base.metadata.drop_all(engine)
	
	# 初始化数据库
	# Base.metadata.create_all(engine)

	# 删除全部数据库
	# Base.metadata.drop_all(engine)

	# 删除指定的数据库
	# 如删除 Blogs表
	# 详见 ：http://stackoverflow.com/questions/35918605/how-to-delete-a-table-in-sqlalchemy
	# Blog.__table__.drop(engine)
	

	# 新增数据
	# newUser()

	# 新增多条数据
	# addmoreUser()

	# 新增数据含中文
	# addUserForZhCn()

	# 查询数据
	# queryUser()
	
	# 删除
	# deleteUser()

	# 测试
	# TestUser()
	 
	# 执行sql语句
	# SQlUser()
	
	# 执行sql语句2
	SQlUser2()

	print('ok')

```

***

#### sqlalchemy 教程

##### 入门

* [Test.py](https://gist.github.com/tuxmartin/ea5783d1ebb99057dd81)
* [作为一个Pythoner，不会SQLAlchemy](https://zhuanlan.zhihu.com/p/23190728)  **☆**
* [SQLAlchemy ORM教程之一：Create - 简书](http://www.jianshu.com/p/0d234e14b5d3)  **☆**
* [SQLAlchemy ORM教程之二：Query - 简书](http://www.jianshu.com/p/8d085e2f2657)  **☆**
* [SQLAlchemy ORM教程之三：Relationship - 简书](http://www.jianshu.com/p/9771b0a3e589) **☆**

##### 增删改查常用命令

* [sqlalchemy学习笔记 - python学习笔记 - SegmentFault](https://segmentfault.com/a/1190000006949536) **☆**
* [SQLAlchemy 使用经验](https://www.keakon.net/2012/12/03/SQLAlchemy%E4%BD%BF%E7%94%A8%E7%BB%8F%E9%AA%8C)
* [note/sqlalchemy.md at master · lzjun567/note · GitHub](https://github.com/lzjun567/note/blob/master/note/python/sqlalchemy.md)

***
