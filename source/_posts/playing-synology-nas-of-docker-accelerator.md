---
title: 玩转群晖NAS--Docker加速
date: 2017-11-17 22:28:04
tags: 
    - SynologyNAS
categories: 
    - 玩转群晖NAS
description: 群晖下Docker配置镜像加速器
---

群晖的DSM系统上安装Docker套件非常的简单，只要点击一个按钮就行了。

但是由于某些 “你懂的” 原因，DockerHub的网站是在国外的，而在国内访问起来就会特别的慢。之前我也写过在其他Linux系统如Ubuntu下配置Docker加速器的方法，但经过一番研究发现群晖NAS下的Docker套件配置加速器的方法还是有一定区别的。

#### 启用Docker套件

进入群晖的 DSM 系统后，选择桌面的 “套件中心” 图标（或在 “主菜单” 界面中选择 “套件中心”），在左侧找到 “实用工具” 一项，右侧往下拉在 “第三方” 一栏下找到 “Docker” 的图标，点击 “安装套件” ，等待安装完成。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/Dg018bm2b9.png?imageslim)

安装完成后，选择 “主菜单” 找到 “Docker” 图标并打开。

具体的操作可以查看 DSM 的帮助界面，这里主要说明重点的项：

* `总览` 能看到当前群晖的 “CPU 使用率” 和 “内存使用率” 以及正在运行的 Docker 容器。
* `注册表` 对应于 `Docker` 来说就是“Docker Hub”。我们可以在这里搜索以及下载镜像。
* `映像` 对应于 `Docker` 来说就是“镜像”，用来管理镜像，创建容器等操作。
* `容器` 是对创建的容器进行管理。

#### 未启用加速器

要在Docker套件中创建容器，我们可以在左侧菜单 “注册表” 项搜索相应的镜像名称，双击下载。但是我们发现下载中的镜像右侧的下载图标在一段时间之内一直显示 “0 B” 的情况，然后就自动消失了。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/6K1C7bKB4a.png?imageslim)

同时在 “映像” 中也会提示 “您未下载任何映像，请进入注册表选项卡以下载。”：

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/aG1A4jmC25.png?imageslim)

一连试过几次，都是下载中途镜像就自动消失了。

#### 启用SSH

要配置Docker的配置文件，还得需要在命令行下来操作。群晖NAS默认没有开启SSH功能，得需要我们先开启才行。

打开 “控制面板” 图标，选择 “应用程序” -- “终端机和SNMP” -- 勾选 “启用 SSH 功能” 。端口可以选择默认或自定义，然后点击应用。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/9j5LlAIEHc.png?imageslim)

#### 启用用户主目录服务

这时如果你以任何用户身份通过终端使用SSH方式访问NAS的ip地址，登陆后一般会看到一条警告提示：

```
Could not chdir to home directory /var/services/homes/tiger: No such file or directory
```

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/jddEE7660m.png?imageslim)

发生此警告是因为主目录由DSM的“用户主目录服务”控制，默认情况下该主目录服务是禁用的。要防止错误，请通过选中 “控制面板” -- “用户账户”菜单 -- “高级设置”选项卡 -- “家目录”组 -- “启用家目录服务” 复选框来启用用户主目录服务。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/KfKI9kDI9j.png?imageslim)

这样再次尝试登陆，会看到警告信息没有了。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/aLk3B8mjAI.png?imageslim)

即使您不打算使用该家目录，但还是建议您选择启用用户主目录服务，以防影响其他某些程序的运行。

#### 临时性Docker加速

如果是临时性的想要 “加速” 下载镜像，可以选择通过命令的方式，执行 `docker pull` 时加入国内源地址，格式为：

```
$ docker pull registry.docker-cn.com/myname/myrepo:mytag
```

例如:

```
$ docker pull registry.docker-cn.com/library/ubuntu:16.04
```

虽然能实现加速效果，但是对于在群晖NAS中操作Docker来说，每次下载镜像都要先去登陆SSH，在命令行中下载好了镜像再回到 DSM 界面来操作，这样的流程未免有些太繁琐了。

#### 配置Docker加速器

我们可以通过配置 `Docker` 守护进程默认使用 `Docker` 官方镜像加速。

##### 查看群晖下Docker版本

这里我使用 `admin` 账号通过SSH登陆到群晖的命令模式下来操作。

使用命令 `docker info` 查看docker详细信息：

如果提示 `Cannot connect to the Docker daemon. Is the docker daemon running on this host?` 说明当前的账号没有 `root` 权限，可以使用 `sudo` 提权来操作，或者可以通过切换到 `root` 账户下来操作，这里我们选择后者。

通过 `admin` 账号登录后，执行 `sudo su -` 切换到 `root` 账户下(注意这一步输入的是admin账号的密码)：

示例命令如下：

```
admin@HomeNAS:/etc$ docker info
Cannot connect to the Docker daemon. Is the docker daemon running on this host?
admin@HomeNAS:~$ sudo su -
Password:    # 注意这一步输入的是admin账号的密码
root@HomeNAS:~# docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 1.11.2
Storage Driver: btrfs
Logging Driver: db
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: host bridge null
Kernel Version: 4.4.15+
Operating System: <unknown>
OSType: linux
Architecture: x86_64
CPUs: 2
Total Memory: 1.801 GiB
Name: HomeNAS
ID: A3DQ:M62X:NLZP:RYMF:NINR:5QBY:7OIJ:L425:3WDR:4V2N:FEFL:OV42
Docker Root Dir: /volume1/@docker
Debug mode (client): false
Debug mode (server): false
Registry: https://index.docker.io/v1/
WARNING: No kernel memory limit support
WARNING: No cpu cfs quota support
WARNING: No cpu cfs period support
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
```

我们可以看到 `Server Version` 目前版本是 `1.11.2` 的。

要退出 `root` 账户模式，执行 `exit` 即可：

```
root@HomeNAS:~# exit
logout
admin@HomeNAS:~$
```

##### 配置加速器

这里我选择使用阿里云的镜像加速器。打开阿里云的 [开发者平台](https://dev.aliyun.com/) , 选择 “管理中心” -- “镜像加速器” ，可以看到 “您的专属加速器地址” 。

而且下面也给出了具体的操作方法。

通过上一步我们看到群晖下的 Docker 版本是大于 `1.10.0` 的，按照文档我们可以通过修改daemon配置文件 `/etc/docker/daemon.json` 来使用加速器。

但是，这里一定要说但是，文档中的方法是对应于在 `Ubuntu` 等Linux系统下通过 `Docker` 官方的安装方式安装的Docker而言的，对于群晖下的Docker来说，并不是这样的。

通过查找我发现群晖中Docker的配置文件地址在 `/var/packages/Docker/etc/dockerd.json` 下，

使用vim编辑：

```
root@HomeNAS:~# vim /var/packages/Docker/etc/dockerd.json
```

可以看到内容如下：

```
{
    "ipv6": true,
    "registry-mirrors": []
}
```

然后将从阿里云获得的加速器地址填入 `registry-mirrors` 部分即可：

```
{
    "ipv6": true,
    "registry-mirrors": ["https://xxxxxx.mirror.aliyuncs.com"]
}
```

注意：网址要用英文的双引号引起来再添加到中括号中。

当然，也可以使用其他的加速器地址。比如使用Docker中国官方镜像的加速地址：

```
{
    "ipv6": true,
    "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

然后需要重启群晖下的Docker服务。

#### 重启群晖下的Docker服务

上面也说到，群晖的DSM系统并不像其他的linux系统如 `Ubuntu` 那样，管理服务可以使用 `systemctl`(Ubuntu16.04后版本) 或 `service` 来操作: 

```
root@HomeNAS:~# systemctl
-ash: systemctl: command not found
root@HomeNAS:~# service
-ash: service: command not found
root@HomeNAS:~#
```

可以看到这两个命令在群晖下都是找不到的。

那是因为在群晖下的操作命令都要加上 `syno` 前缀来操作，执行命令 `synoservice` 或 `synoservice --help`：

```
root@HomeNAS:~# synoservice
Copyright (c) 2003-2017 Synology Inc. All rights reserved.

SynoService Tool Help (Version 15217)
Usage: synoservice
	--help							Show this help
	--help-dev						More specialty functions for deveplopment
	--is-enabled		[ServiceName]			Check if the service is enabled
	--status		[ServiceName]			Get the status of specified services
	--enable		[ServiceName]			Set runkey to yes and start the service (alias to --start)
	--disable		[ServiceName]			Set runkey to no and stop the service (alias to --stop)
	--hard-enable		[ServiceName]			Set runkey to yes and start the service and its dependency (alias to --hard-start)
	--hard-disable		[ServiceName]			Set runkey to no and stop the service and its dependency (alias to --hard-stop)
	--restart		[ServiceName]			Restart the given service
	--reload		[ServiceName]			Reload the given service
	--pause			[ServiceName]			Pause the given service
	--resume		[ServiceName]			Resume the given service
	--pause-by-reason	[ServiceName]	[Reason]	Pause the service by given reason
	--resume-by-reason	[ServiceName]	[Reason]	Resume the service by given reason
	--pause-all		(-p)	[Reason]	(Event)	Pause all service by given reason with optional event(use -p to include packages)
	--pause-all-no-action	(-p)	[Reason]	(Event)	Set all service runkey to no but leave the current service status(use -p to include packages)
	--resume-all		(-p)	[Reason]		Resume all service by given reason(use -p to include packages)
	--reload-by-type	[type]		(buffer)	Reload services with specified type
	--restart-by-type	[type]		(buffer)	Restart services with specified type
								Type may be {file_protocol|application}
								Sleep $buffer seconds before exec the command (default is 0)
root@HomeNAS:~#
```

好的，现在已经知道了如何在群晖下管理服务，那么按照步骤，下一步只需要重启Docker服务使其应用上加速器地址即可。

按照上面的规律可想而知，在群晖下Docker的守护进程服务名称肯定会和在 `Ubuntu` 下的名称不一样，那我们如何来找到呢？

可以通过 `synoservicecfg --list` 命令来查看当前群晖系统下所有运行的服务：

```
root@HomeNAS:~# synoservicecfg --list
DSM
apparmor
atalk
avahi
bluetoothd
bonjour
...
...
pkgctl-Docker
pkgctl-FileStation
pkgctl-LogCenter
pkgctl-PDFViewer
pkgctl-PHP7.0
pkgctl-PhotoStation
...
...
```

可以看到通过群晖的 “套件中心” 添加的套件程序的服务名称均以 `pkgctl-` 为前缀来命名。

然后重启群晖的docker服务：

```
root@HomeNAS:~# synoservice --restart pkgctl-Docker
root@HomeNAS:~#
```

如果没有错误提示，说明docker服务重启正常。

现在我们再次回到 `DSM` 操作界面中，重新下载我们需要的Docker镜像即可。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/Bkae1Hj4E4.png?imageslim)

可以看到现在后面的容量大小一直在增加，很快我们就看到 “消息通知” 里提示我们镜像下载完成了。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/f2dHLjjm9d.png?imageslim)

后面就是通过镜像来创建容器了，后文继续。

***

#### 相关参考

* [Synology DSM 6 (terminal) service control](https://diktiosolutions.eu/en/synology/synology-dsm-6-terminal-service-control-en/)
* [restart WebServer bash command](https://forum.synology.com/enu/viewtopic.php?t=119012)
* [Documentation for /var/packages/Docker/etc config files?](https://forum.synology.com/enu/viewtopic.php?t=126938)
* [Docker 中国官方镜像加速](https://www.docker-cn.com/registry-mirror)
