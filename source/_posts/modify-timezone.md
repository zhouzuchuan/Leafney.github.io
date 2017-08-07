---
title: Linux下修改时区
date: 2017-02-23 14:57:58
tags:
	- Linux
	- Docker
description: 一般通过默认方式安装的linux系统显示的都是UTC时间，这样导致一些依赖时间的程序就会出现时差问题。下面介绍在Ubuntu和Alpine系统下如何更改UTC时区为CST时区。
---

一般通过默认方式安装的linux系统显示的都是UTC时间，这样导致一些依赖时间的程序就会出现时差问题。下面介绍在Ubuntu和Alpine系统下如何更改UTC时区为CST时区。

#### Alpine

##### alpine 下修改UTC时间为CST时间 (测试通过)

```
$ apk add tzdata 
$ ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 
$ echo "Asia/Shanghai" > /etc/timezone
```

##### Docker + Alpine 下修改utc时间为cst时间

Dockerfile :

```
RUN apk update && apk add ca-certificates && \
    apk add tzdata && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone
```

#### Ubuntu

##### Ubuntu 下手动修改配置 (图形化界面)

```
$ sudo dpkg-reconfigure tzdata
```

或：

```
$ sudo tzselect
```

##### Ubuntu下通过命令更改（已测试）

```
$ sudo ln -sf /usr/share/zoneinfo/Asia/ShangHai /etc/localtime
$ sudo echo "Asia/Shanghai" > /etc/timezone
$ sudo dpkg-reconfigure -f noninteractive tzdata
```

**经测试，如果不加第一行，系统重启后又恢复UTC时间了** 

##### Docker + Ubuntu 下修改UTC时间为CST时间

Dockerfile:

```
# Setting timezone
RUN ln -sf /usr/share/zoneinfo/Asia/ShangHai /etc/localtime && \
	echo "Asia/Shanghai" > /etc/timezone && \
	dpkg-reconfigure -f noninteractive tzdata
```

***

#### 相关参考

* [docker 中设置时区 | 水能载舟 亦可赛艇](https://lengzzz.com/note/timezone-in-docker)
* [Ubuntu: 以命令行方式修改时区](https://lesca.me/archives/set-timezone-with-command-line-on-ubuntu.html)
* [docker 中设置时区 - 作业部落 Cmd Markdown 编辑阅读器](https://www.zybuluo.com/zwh8800/note/337111)
