---
title: Netdata-Linux下性能监视工具
date: 2016-05-06 22:17:57
tags: 
    - Linux
categories: 
    - 开发笔记
description: Netdata-Linux性能监视工具
---

#### 编译依赖

```
# Debian/Ubuntu
$ sudo apt-get install zlib1g-dev gcc make git autoconf autogen automake pkg-config
```

#### Install netdata

```
# download it - the directory 'netdata.git' will be created
$ git clone https://github.com/firehol/netdata.git --depth=1
cd netdata

# build it
./netdata-installer.sh
```

一旦编译安装完毕，`netdata` 将执行 `/usr/sbin/netdata` 启动 `daemon` 程序，并监听本机的 `19999` 端口。

直接访问：`127.0.0.1:19999` 即可打开监控界面。


#### 链接

* 官方安装教程：[Installation · firehol/netdata Wiki · GitHub](https://github.com/firehol/netdata/wiki/Installation)
* [GitHub - firehol/netdata: Real-time performance monitoring, done right!](https://github.com/firehol/netdata)
* [netdata：实时监视 Linux 系统性能](https://linuxtoy.org/archives/netdata.html)
