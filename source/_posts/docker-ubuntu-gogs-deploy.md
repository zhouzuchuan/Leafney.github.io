---
title: Docker-Ubuntu-Gogs部署gogs容器过程记录
date: 2017-03-23 10:40:40
updated: 2017-03-23 10:40:40
tags:
    - Gogs
    - Docker
categories: 
    - Ubuntu-Gogs
description: Docker Gogs 用更简单的方式部署、升级或迁移Gogs容器服务。 Docker-Ubuntu-Gogs 系列文章主要记录我在Docker下部署Gogs代码管理项目的过程。系列文章包括Gogs容器的部署过程，部署时遇到的问题及解决方法，个性化配置等。
---

Docker Gogs 用更简单的方式部署、升级或迁移Gogs容器服务。

`Docker-Ubuntu-Gogs` 系列文章主要记录我在Docker下部署Gogs代码管理项目的过程。系列文章包括Gogs容器的部署过程，部署时遇到的问题及解决方法，个性化配置等。

这里底层系统选择了 `Ubuntu16.04` 版本，之前也曾尝试在 `Alpine` 系统下来部署Gogs，但安装完成后会报 `./gogs web is not found` 之类的错误，暂未找到解决方法。遂最后决定采用Ubuntu来部署。

对于 `Alpine` 系统下的部署方法，待后期再来完善。

另外，Gogs作者在Github中发布的Gogs容器版本是用Alpine系统来做的，如果比较在意容器的大小，可以直接用之。

***

#### 测试操作步骤

操作记录：`Ubuntu16.04` 系统

```bash
docker run -it --name ugg1 -p 3000:3000 -p 8080:22 ubuntu /bin/bash


RUN echo "deb http://cn.archive.ubuntu.com/ubuntu/ xenial main restricted universe multiverse" >> /etc/apt/sources.list


apt-get update

apt-get install -y git wget openssh-server

adduser git  (and set pwd git)

# 拷贝本地的gogs项目zip包到容器中
docker cp linux_amd64.zip ugg1:/home/git/gogs.zip

unzip gogs.zip

#***************

cd gogs
mkdir -p custom/conf
mkdir -p log

chmod -R 777 custom
chmod -R 777 log

cd ..
chown -R git:git gogs

#*******************

# gosu
docker cp gosu-amd64 ugg1:/usr/local/bin/gosu
chmod +x /usr/local/bin/gosu

# 启动项目
cd gogs
./gogs web


# 启动ssh服务（之前测试时把这项丢了，所以ssh功能一直无法使用）
service ssh start 
service ssh restart


gosu git /home/git/gogs/gogs web

# 项目文件目录
/app
	gogs-repositories
	gogs
		log
		ssh
		conf
		data


# 创建数据文件
mkdir -p /app/gogs-repositories /app/gogs/log /app/gogs/ssh /app/gogs/data /app/gogs/conf
chown -R git:git /app



ln -sf /home/git/gogs/custom/conf/app.ini  /app/gogs/conf/app.ini
chown -R git:git /app


$ docker run --name ugg1 -d -p 3002:3000 -p 8090:22 -v /home/tiger/xdk/dfile:/app gg1
```

***

```
# Link volumed data with app data
ln -sf /data/gogs/log  ./log
ln -sf /data/gogs/data ./data
```

参考自gogs官方github中的dockerfile，使用gosu调用git用户：

```
export USER=git
exec gosu $USER /app/gogs/gogs web
```

***

#### 问题一：不能使用 `gosu` 调用 `git` 用户来启动

发现不能使用 `gosu` 调用 `git` 用户来启动，`gosu git /home/git/gogs/gogs web` 会报如下错误：

```
gogs 运行系统用户非当前用户：git >
```

不推荐的解决方法是： 切换到git账户下执行：`su - git`  `$ ./gogs web`

通过测试，可以采用如下的方式来使用 `gosu` 调用 `git` 用户（参考自gogs官方github中的dockerfile）：

```
export USER=git
exec gosu $USER /app/gogs/gogs web
```

***

#### 问题二：`Fail to start SSH server: listen tcp 0.0.0.0:22: bind: permission denied`

启用内置SSH服务器会报该错误，暂未找到解决方法。

通过测试，将默认的 `22` 端口改成其他端口即可。

但是 **gogs官方的docker配置中建议不要在Docker容器中使用内置的SSH服务器**。

***

#### 问题三：`PANIC: session(start): mkdir data: permission denied`

详细错误信息如下：

```
[Macaron] 2017-03-21 06:05:40: Started GET / for 192.168.137.1
[Macaron] PANIC: session(start): mkdir data: permission denied
/usr/local/go/src/runtime/panic.go:489 (0x4340bf)
/home/vagrant/gopath/src/github.com/gogits/gogs/vendor/github.com/go-macaron/session/session.go:156 (0x8f478e)
/home/vagrant/gopath/src/github.com/gogits/gogs/vendor/gopkg.in/macaron.v1/context.go:79 (0x89ba01)
/home/vagrant/gopath/src/github.com/gogits/gogs/vendor/github.com/go-macaron/inject/inject.go:157 (0x87dc92)
/home/vagrant/gopath/src/github.com/gogits/gogs/vendor/github.com/go-macaron/inject/inject.go:135 (0x87da8b)
/home/vagrant/gopath/src/github.com/gogits/gogs/vendor/gopkg.in/macaron.v1/context.go:121 (0x89bc62)
/home/vagrant/gopath/src/github.com/gogits/gogs/vendor/gopkg.in/macaron.v1/context.go:112 (0x89bb86)
/home/vagrant/gopath/src/github.com/gogits/gogs/vendor/gopkg.in/macaron.v1/recovery.go:161 (0x8af50b)
/home/vagrant/gopath/src/github.com/gogits/gogs/vendor/gopkg.in/macaron.v1/logger.go:40 (0x89f118)
```

该问题导致的原因是，当 `git` 用户运行 `./gogs web` 时，会在 `gogs` 项目的主目录下 ( 这里是 `/home/git/gogs`）创建一个 `data` 目录用于存放session缓存等临时文件。如果当前工作的主目录不是在 `/home/git/gogs` 目录，则git账户就没有权限来创建目录，从而导致权限错误。
解决方法是在Dockerfile中指定工作目录 `WORKDIR /home/git/gogs` 即可。 

后经查证，在 `Gogs` 项目目录下的 `custom` `data` 和 `log` 三个目录是用来存放项目运行期间产生的日志、配置文件、数据等信息的。当Gogs项目需要升级时，直接拷贝这三个目录到新项目目录下即可。
这里的 `data` 目录需要提前创建好。

***

#### 相关参考

* [在ubuntu上安装gogs](http://amazingw.github.io/2016/03/22/ubuntu-with-gogs.html)
* [用Gogs搭建自己的Git服务器 - LibHappy](https://libhappy.com/2016/01/build-gogs-service/)
* [How To Set Up Gogs on Ubuntu 14.04 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-gogs-on-ubuntu-14-04)
* [chmod 777 修改权限 - 日光之下无新事 - 博客园](http://www.cnblogs.com/sipher/articles/2429772.html)
* [linux - 安装gogs时报错：运行系统用户非当前用户：git -&gt; root。 不知道是什么意思 - SegmentFault](https://segmentfault.com/q/1010000004608054)

***

#### Install时注意事项

* `Database Type` 选择 `SQLite3` （目前仅测试了sqlite3数据库）
* `Sqlite Database Path` 使用固定路径 `/app/gogs/data/gogs.db` （后期改成默认目录也可以，详见Github）
* `Repository Root Path` 使用固定路径 `/app/gogs-repositories`
* `Run User` 使用默认的 `git`
* `Domain` 填写Docker宿主机的主机名或物理地址 类似于 `192.168.137.140`
* `SSH Port` 不要勾选使用内置SSH服务器（Don't user `Use Builtin SSH Server
`） 如果你映射Docker外部端口如 `10022:22` 那么这里填写 `10022`
* `HTTP Port` 如果映射外部端口 `10080:3000` 这里仍然使用 `3000`
* `Application URL` 使用域和公开的HTTP端口值的组合 如 `http://192.168.137.140:10080`
* `Log Path` 填写固定路径 `/app/gogs/log`  （后期改成默认目录也可以，详见Github）


**注意：**

Dockerfile中必需添加 `WORKDIR /home/git/gogs` 即必需指定当前工作目录为 `gogs` 目录下，因为gogs在install完成后会在当前目录下创建一个 `data` 目录保存 `sessions` 信息。如果当前目录不是这里，则会报 `[Macaron] PANIC: session(start): mkdir data: permission denied` 的错误。

***

#### 成果

最终项目见：[GitHub - Leafney/ubuntu-gogs: Docker + Ubuntu + Gogs](https://github.com/Leafney/ubuntu-gogs)

***
