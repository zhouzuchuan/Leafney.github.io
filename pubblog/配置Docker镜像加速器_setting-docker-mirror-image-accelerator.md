---
title: 配置Docker镜像加速器
date: 2016-12-26 16:04:40
tags: 
	- Docker
description: 配置阿里云Docker镜像加速器和DaoCload镜像加速器
---

由于“你懂得”的原因，在国内获取Docker镜像时经常因为网络原因而下载失败，有些人会选择“番茄”，但是配置起来着实麻烦。所以选择国内的镜像加速器是目前解决该问题的最好方法。这里我主要介绍阿里云的Docker镜像加速器和DaoCload的镜像加速器的配置方法。

注意：针对于 Ubunt16.04系统下的Docker配置镜像加速器，可直接参考整理的最新文章：[Ubuntu16.04安装Docker及配置镜像加速器](/2017/01/14/ubuntu-install-docker-and-configure-mirror-accelerator/)

#### 配置阿里云Docker镜像加速器

打开 `官方地址` [开发者平台](https://dev.aliyun.com/)  --  `管理中心` -- `加速器` 。可以看到 “您的专属加速器地址” 即 `https://xxxxxxx.mirror.aliyuncs.com` 。

##### Ubuntu系统下如何配置

因为我的系统为 `Ubuntu 15.04` , 所以这里仅以Ubuntu系统下的配置方法为例，其他系统可参考官网中的说明。

###### 安装或升级Docker

这里要求必须是 1.6.0 以上版本的Docker。可以从阿里云的镜像仓库下载 [mirrors.aliyun.com/help/docker-engine](mirrors.aliyun.com/help/docker-engine)

```
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
```

###### 配置Docker加速器

如果Ubuntu系统是 `12.04` `14.04` ，`Docker 1.9` 以上， 执行：

```
echo "DOCKER_OPTS=\"\$DOCKER_OPTS --registry-mirror=https://xxxxxxx.mirror.aliyuncs.com\"" | sudo tee -a /etc/default/docker
sudo service docker restart
```

如果Ubuntu系统是 `15.04` `16.04` ，`Docker 1.9` 以上， 执行：

```
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo tee /etc/systemd/system/docker.service.d/mirror.conf <<-'EOF'
[Service]
ExecStart=
ExecStart=/usr/bin/docker daemon -H fd:// --registry-mirror=https://xxxxxxx.mirror.aliyuncs.com
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

等待Docker服务重启后，再次下载镜像则非常快了。

***

#### 配置DaoCload的镜像加速器

打开 `官方地址` [DaoCloud](http://www.daocloud.io/) -- `产品` -- `加速器` -- `立即使用` 。

##### Linux系统配置 Docker 加速器

###### 命令脚本自动配置 （推荐）

DaoCloud也会为你分配一个专属的加速器地址，可以直接拷贝页面中给出的代码：

```
# curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://xxxxxx.m.daocloud.io
```

该脚本可以将 `--registry-mirror` 加入到你的 Docker 配置文件 `/etc/default/docker` 中。适用于 Ubuntu14.04、Debian、CentOS6 、CentOS7、Fedora、Arch Linux、openSUSE Leap 42.1，其他版本可能有细微不同。

###### 手动配置

也可以自己手动修改。

在 `/etc/default/docker` 文件底部添加如下内容：

```
$ sudo vim /etc/default/docker
```

添加：

```
DOCKER_OPTS="$DOCKER_OPTS --registry-mirror=http://xxxxxx.m.daocloud.io"
```

###### 重启Docker服务

配置完成后需要重启docker服务。

```
$ sudo service docker restart
```

***

#### 注意事项

* 如果你的系统当前登陆用户不是管理员账户的话，记得添加 `sudo` 以免执行失败。
* **上文代码段中给出的镜像加速器地址中的 `xxxxxxx` 为阿里云或DaoCloud在你注册账户后分配的指定地址名称，切记要修改为自己账户的给定地址。**

***

#### 相关链接

* [阿里云开发者平台](https://dev.aliyun.com/)
* [DaoCloud](http://www.daocloud.io/)
* [Docker 镜像加速器-博客-云栖社区-阿里云](https://yq.aliyun.com/articles/29941)
* [使用DaoCloud安装Docker和镜像 - 简书](http://www.jianshu.com/p/bc35ea82b61d)
