---
title: Golang连接MongoDB数据库
date: 2017-06-28 17:28:24
tags:
	- Golang
	- MongoDB
categories: 
    - Golang
description: 目前go支持MongoDB最好的驱动就是mgo。
---

目前go支持MongoDB最好的驱动就是mgo。

#### 下载与引用

```
go get gopkg.in/mgo.v2

import "gopkg.in/mgo.v2"
```

```
go get labix.org/v2/mgo

import "labix.org/v2/mgo"
```

安装时发现，地址 `labix.org/v2/mgo` 中的mgo版本中有些方法不全，而地址 `gopkg.in/mgo.v2`中的方法是全的，但是该地址下载超时而失败，必须翻墙才可以访问。
如 `labix.org/v2/mgo` 中就没有 `mongo, err := mgo.ParseURL(MongoDBUrl)` 的 `ParseURL()` 方法。

如果无法翻墙，还可以从 github 下载：

因为我们要使用 mgo 的 `v2` 版本，所以需要迁出 `branch:v2` 分支，不能直接使用 `master` 分支。

在 `go get` 命令后添加 `-d` 参数可以只下载而不会执行安装命令。

操作步骤为：

1. 执行命令 `go get -d github.com/go-mgo/mgo`
2. 找到上面 `clone` 的目录，迁出分支 `v2`
3. 再次运行 `go get` 命令，这时会在迁出的分支上执行命令

```
go get -d github.com/go-mgo/mgo

git checkout v2

go get github.com/go-mgo/mgo
```

* [mgo - Rich MongoDB driver for Go](http://labix.org/mgo)
* [mgo.v2 - gopkg.in/mgo.v2](http://gopkg.in/mgo.v2)
* [git - How to do &quot;go get&quot; on a specific tag of a github repository - Stack Overflow](https://stackoverflow.com/questions/30188499/how-to-do-go-get-on-a-specific-tag-of-a-github-repository)

***

#### 安装bzr工具

Bazaar是一款开源的分布式版本控制工具。

安装mgo之前，需要先安装 bzr 工具，否则直接执行或报错：

```
> go get labix.org/v2/mgo
go: missing Bazaar command. See https://golang.org/s/gogetcmd
package labix.org/v2/mgo: exec: "bzr": executable file not found in %PATH%
```

##### Win

从网址 `http://bazaar.canonical.com/en/` 下载安装包，选择 `Standalone`版本或其他版本。

##### Ubuntu

> sudo apt-get install bzr

* [多平台系统下如何安装Bazzar工具](http://fpliu-blog.chinacloudsites.cn/it/software/Bazaar)

***

#### 连接字符串

```
MONGODB_URL="mongodb://user:pass@server.compose.io/db_name"

mongodb://localhost:27017/articles_demo_dev
mongodb://myuser:mypass@localhost:40001,otherhost:40001/mydb
```

或者：

```
session, err := mgo.Dial("") 
session, err := mgo.Dial("localhost") 
session, err := mgo.Dial("127.0.0.1") 
session, err := mgo.Dial("localhost:27017")
session, err := mgo.Dial("127.0.0.1:27017")
```

***

#### 创建连接

通过Session.DB()来切换相应的数据库

```
 db := session.DB("xtest")   //数据库名称
```

通过Database.C()方法切换集合（Collection）：

```
collection := db.C("xtest") // 集合名称
```

或直接一步：

```
c := session.DB("xtest").C("xtest")
```

***

#### 查询

通过func (c *Collection) Find(query interface{}) *Query来进行查询
通过Query.All()可以获得所有结果
通过Query.One()可以获得一个结果
条件用 `bson.M{key: value}` ，注意key必须用MongoDB中的字段名，而不是struct的字段名。

***

#### 示例

```
package main

import (
	"fmt"
	"labix.org/v2/mgo"
	"labix.org/v2/mgo/bson"
)

const (
	MONGODB_URL = "127.0.0.1:27017"
)

func main() {
	//创建连接
	session, err := mgo.Dial(MONGODB_URL)
	if err != nil {
		panic(err)
	}
	defer session.Close()

	session.SetMode(mgo.Monotonic, true)
	// db := session.DB("xtest")   //数据库名称
	// collection := db.C("xtest") // 集合名称
	c := session.DB("xtest").C("xtest")

	// //插入数据
	// err = c.Insert(&Person{"Tommy", "123456"}, &Person{"Hanleilei", "98765"},
	// 	&Person{"喜洋洋", "98765"}, &Person{"灰太狼", "46577"},
	// )
	// if err != nil {
	// 	panic(err)
	// }

	// //查询并赋值 Find().One()
	// result := Person{}
	// err = c.Find(bson.M{"name": "Tommy"}).One(&result)
	// if err != nil {
	// 	panic(err)
	// }
	// //输出
	// fmt.Println("Phone ", result.Phone)

	// //集合中元素数量 Count()
	// countNum, err := c.Count()
	// fmt.Println("obj numbers ", countNum)

	// //查询多条数据 Find().Iter()
	// var onep = Person{}
	// iter := c.Find(nil).Iter()
	// for iter.Next(&onep) {
	// 	fmt.Println("姓名 ", onep.Name)
	// }

	// //查询多条数据 Find().All()
	// var personAll []Person
	// err = c.Find(nil).All(&personAll)
	// for i := 0; i < len(personAll); i++ {
	// 	fmt.Println("Person ", personAll[i].Name, personAll[i].Phone)
	// }

	// //更新数据 Update()
	// abc := Person{}
	// err = c.Find(bson.M{"name": "Tommy"}).One(&abc)
	// fmt.Println("Tommy phone is ", abc.Phone)

	// err = c.Update(bson.M{"name": "Tommy"}, bson.M{"$set": bson.M{"phone": "10086"}})
	// err = c.Find(bson.M{"name": "Tommy"}).One(&abc)
	// fmt.Println("Tommy phone is ", abc.Phone)

	// //删除数据 Remove()
	// fmt.Println(c.Count())
	// err = c.Remove(bson.M{"phone": "46577"})
	// fmt.Println(c.Count())

	fmt.Println("end")
}

type Person struct {
	Name  string `bson:"name"`
	Phone string `bson:"phone"`
}

```

***

#### 相关

* [GitHub - go-mgo/mgo: The MongoDB driver for Go. See http://labix.org/mgo for details.](https://github.com/go-mgo/mgo)
* [golang使用mgo连接MongoDB](http://www.it165.net/database/html/201403/5661.html)
