---
title: Python操作SQLServer数据库
date: 2016-06-20 20:06:00
tags:
    - Python
categories: 
    - 开发笔记
description: Python操作SQLServer数据库
---

#### pymssql

Python操作SQLServer需要使用 `pymssql` 模块

Windows系统下的Python类库文件 `*.whl`文件，需要用 `pip install *.whl` 的方式来安装。

下载pymssql2.7 library ：[pythonlibs](http://www.lfd.uci.edu/~gohlke/pythonlibs/#pymssql)

第一次下载了 `pymssql-2.1.2-cp27-cp27m-win_amd64.whl` 文件，安装时报错，提示如下错误：

```
λ pip install pymssql-2.1.2-cp27-cp27m-win_amd64.whl
pymssql-2.1.2-cp27-cp27m-win_amd64.whl is not a supported wheel on this platform.
```

即使成功安装了 `pymssql`，但是 `import pymssql` 还是会报错

```
import pymssql
ImportError: DLL load failed: 找不到指定的模块。
```

所以又再次下载了 `pymssql-1.0.3-cp27-none-win32.whl` 文件，再次安装成功。
具体原因可能和操作系统或python版本有关，具体待深入研究。

#### Demo

```
# coding:utf-8
"""
http://www.pymssql.org/en/stable/
http://www.lfd.uci.edu/~gohlke/pythonlibs/#pymssql 

pymssql-1.0.3-cp27-none-win32.whl    # 只有这个才能在windows下执行
pymssql-2.1.2-cp27-cp27m-win_amd64.whl # 这个不能执行

安装 whl文件 执行 `pip install *.whl`


异常：
不能用 DB-Library (如 ISQL)或 ODBC 3.7 或更早版本将 ntext 数据或仅使用 Unicode 排序规则的 Unicode 数据发送到客 户端。
这个问题是数据库中字段为ntext类型，这种类型目前的c-library不支持，需要转为nvchar或text类型才可以。
1. 建议：将ntext修改为nvarchar或text.
2. 既然不支持ntext但支持text，那么我们只需要在输出时将ntext转换为text就好了 
`SELECT cast ( field_name AS TEXT ) AS field_name`
`select convert(varchar(50),guid) as guid, convert(text,content) as content from news`

"""
import pymssql

class MSSQL:
    def __init__(self,host,user,pwd,db):
        self.host=host
        self.user=user
        self.pwd=pwd
        self.db=db

    def __GetConnect(self):
        if not self.db:
            raise(NameError,"没有设置数据库信息")

        self.conn=pymssql.connect(host=self.host,user=self.user,password=self.pwd,database=self.db,charset="utf-8")
        cur=self.conn.cursor()
        if not cur:
            raise(NameError,"连接数据库失败")
        else:
            return cur

    def ExecQuery(self,sql):
        cur=self.__GetConnect()
        cur.execute(sql.encode("utf-8"))
        resList=cur.fetchall()
        # 查询完毕后关闭连接
        self.conn.close()
        return resList

    def ExecNonQuery(self,sql):
        cur=self.__GetConnect()
        cur.execute(sql.encode("utf-8"))
        self.conn.commit()
        self.conn.close()



if __name__=="__main__":

    ms=MSSQL(host="Test-FPC\sqlexpress",user="sa",pwd="1",db="its")
    resList=ms.ExecQuery("select * from [SchemeCategory] where IsShow=1")   
    for i in resList:
        print(i["Name"])

```

#### 错误处理

在操作是报出了如下异常信息：

> 不能用 DB-Library (如 ISQL)或 ODBC 3.7 或更早版本将 ntext 数据或仅使用 Unicode 排序规则的 Unicode 数据发送到客 户端。

这个问题是数据库中字段为ntext类型，这种类型目前的 `c-library` 不支持，需要转为 `nvchar` 或 `text` 类型才可以。

1. 建议：将ntext修改为nvarchar或text.
2. 既然不支持ntext但支持text，那么我们只需要在输出时将ntext转换为text就好了 
```
SELECT cast ( field_name AS TEXT ) AS field_name
select convert(varchar(50),guid) as guid, convert(text,content) as content from news
```
3. 使用另一个类库 `pyodbc` 来操作，未遇到该问题。

*** 

#### 相关链接

[Python操作SQLServer示例 | 大爱](http://lovesoo.org/python-example-sqlserver.html)
[利用python简化sql server数据导入导出 - qianlifeng - 博客园](http://www.cnblogs.com/qianlifeng/archive/2012/02/08/2343286.html)
* [python 使用pymssql连接sql server数据库 - Hello World!](http://blog.csdn.net/shanliangliuxing/article/details/8632909)

[pymssql &mdash; pymssql 2.2.0.dev documentation](http://pymssql.org/en/latest/index.html)
[FreeTDS &mdash; pymssql 2.2.0.dev documentation](http://pymssql.org/en/latest/freetds.html#windows)
[GitHub - pymssql/pymssql: Official home for the pymssql source code.](https://github.com/pymssql/pymssql)
[FreeTDS.org](http://www.freetds.org/)
[Python Extension Packages for Windows - Christoph Gohlke](http://www.lfd.uci.edu/~gohlke/pythonlibs/#pymssql)

******

#### pyodbc

##### SQL Server ODBC drivers

* {SQL Server} - released with SQL Server 2000
* {SQL Native Client} - released with SQL Server 2005 (also known as version 9.0)
* {SQL Server Native Client 10.0} - released with SQL Server 2008
* {SQL Server Native Client 11.0} - released with SQL Server 2012

 [sqlservertests]
 `connection-string=DRIVER={SQL Server};SERVER=localhost;UID=uid;PWD=pwd;DATABASE=db`

The connection string above will use the 2000/2005 driver, even if SQL Server 2008
is installed:
* 2000: DRIVER={SQL Server}
* 2005: DRIVER={SQL Server}
* 2008: DRIVER={SQL Server Native Client 10.0}


##### connection strings

```
DRIVER={SQL Server Native Client 11.0};SERVER=test;DATABASE=test;UID=user;PWD=password
```

in Python:

```
conn = pyodbc.connect(r'DRIVER={SQL Server Native Client 11.0};SERVER=test;DATABASE=test;UID=user;PWD=password')
```

##### 连接字符串的两种写法

###### 字符串方式

```
connSqlServer = pyodbc.connect('DRIVER={SQL Server Native Client 10.0};SERVER=192.106.0.102\instance1;DATABASE=master;UID=sql2008;PWD=password123')

connSqlServer = pyodbc.connect('DRIVER={SQL Server Native Client 10.0};SERVER=192.106.0.102,1443;DATABASE=master;UID=sql2008;PWD=password123')
```

###### 关键字方式

```
connSqlServer = pyodbc.connect(driver='{SQL Server Native Client 10.0}',
                               server='192.106.0.102\instance1',
                               database='master',
                               uid='sql2008',pwd='password123')

connSqlServer = pyodbc.connect(driver='{SQL Server Native Client 10.0}',
                               server='192.106.0.102,1443',
                               database='master',
                               uid='sql2008',pwd='password123')
```

详见：[sql - python pyodbc : how to connect to a specific instance - Stack Overflow](http://stackoverflow.com/questions/25505081/python-pyodbc-how-to-connect-to-a-specific-instance)

##### 我的写法

###### 第一种方式

```
    def __init__(self,driver,server,db,uid,pwd):
        self.driver="{{{0}}}".format(driver)  # 两个{表示一个
        self.server=server
        self.db=db
        self.uid=uid
        self.pwd=pwd

    def __GetConnect(self):
        if not self.db:
            raise(NameError,"没有设置数据库信息")

        # r'DRIVER={SQL Server Native Client 11.0};SERVER=test;DATABASE=test;UID=user;PWD=password'
        self.conn=pyodbc.connect(driver=self.driver,server=self.server,database=self.db,uid=self.uid,pwd=self.pwd)

    ms=MSSQLODBC(driver="SQL Server",server="Test-FPC\sqlexpress",db="its",uid="sa",pwd="1")
```

###### 第二种方式

```
self.conn = pyodbc.connect('DRIVER={SQL Server};SERVER=Test-FPC\sqlexpress;PORT=1433;DATABASE=its;UID=sa;PWD=1')
```

***

* [pyodbc](https://mkleehammer.github.io/pyodbc/)
* [Connecting to SQL Server from Windows · mkleehammer/pyodbc Wiki · GitHub](https://github.com/mkleehammer/pyodbc/wiki/Connecting-to-SQL-Server-from-Windows)
* [sql - python pyodbc : how to connect to a specific instance - Stack Overflow](http://stackoverflow.com/questions/25505081/python-pyodbc-how-to-connect-to-a-specific-instance)

#### Demo

```
# coding:utf-8
"""

"""
import pyodbc

class MSSQLODBC:
    def __init__(self,driver,server,db,uid,pwd):
        self.driver="{{{0}}}".format(driver)  # 两个{表示一个
        self.server=server
        self.db=db
        self.uid=uid
        self.pwd=pwd

    def __GetConnect(self):
        if not self.db:
            raise(NameError,"没有设置数据库信息")

        # r'DRIVER={SQL Server Native Client 11.0};SERVER=test;DATABASE=test;UID=user;PWD=password'
        self.conn=pyodbc.connect(driver=self.driver,server=self.server,database=self.db,uid=self.uid,pwd=self.pwd)
        # self.conn = pyodbc.connect('DRIVER={SQL Server};SERVER=Test-FPC\sqlexpress;PORT=1433;DATABASE=its;UID=sa;PWD=1')
        cur=self.conn.cursor()
        if not cur:
            raise(NameError,"连接数据库失败")
        else:
            return cur

    def ExecQuery(self,sql):
        cur=self.__GetConnect()
        cur.execute(sql.encode("utf-8"))
        resList=cur.fetchall()
        # 查询完毕后关闭连接
        self.conn.close()
        return resList

    def ExecNonQuery(self,sql):
        cur=self.__GetConnect()
        cur.execute(sql.encode("utf-8"))
        self.conn.commit()
        self.conn.close()



if __name__=="__main__":

    ms=MSSQLODBC(driver="SQL Server",server="Test-FPC\sqlexpress",db="its",uid="sa",pwd="1")
    resList=ms.ExecQuery("select top 10 * from [MallProductItem]")
    for m in resList:
        print(m[4])
```
