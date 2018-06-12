---
title: Docker共享volume的权限问题
date: 2018-06-12 16:29:41
updated: 2018-06-12 16:29:41
tags:
    - Docker
categories: 
    - Docker容器技术
description: docker容器volume目录 permission denied
top: false
---

有时候在创建Docke容器时，我一般都会先在host主机下创建一个用来映射到容器内的目录作为 `volume` 映射目录。但在某些情况下，我们可能会忘记提前创建该目录，而直接执行了 `docker run .... -v /data/sites:/app ....ubuntu:latest` 命令。该命令会自行在 `host` 主机下创建目录 `/data/sites` 。

但当向该文件中拷贝文件时，就会提示 `permission denied` 的错误，这是因为当前用户对该目录没有可操作权限导致的。

`volume` 的权限在于主机是怎么给的，如果你想要给phpfpm文件夹 `www-data:www-data` 权限，在你的主机挂载目录执行 `chown -R www-data:www-data /data/sites` 即可。

在宿主机为 `volume` 目录重新设置host主机下当前用户的权限即可：

如在 `host` 主机下，当前用户为 `tiger`：

```
$ chown -R tiger:tiger /data/sites
```
