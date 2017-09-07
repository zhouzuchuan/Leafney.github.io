---
title: Ubuntu16.04下安装Docker-CE社区版
date: 2017-08-25 23:03:33
tags: 
	- Ubuntu
	- Docker
description: 在2017年3月份，Docker公司宣布Docker企业版（Enterprise Edition, EE），并将开源版本重命名为Docker社区版（Community Edition, CE）；同时公布了产品迭代计划，这会为企业客户提供透明的生命周期支持计划、并对Docker技术的稳定性和可维护性提升带来了帮助。
---

在2017年3月份，Docker公司宣布Docker企业版（Enterprise Edition, EE），并将开源版本重命名为Docker社区版（Community Edition, CE）；同时公布了产品迭代计划，这会为企业客户提供透明的生命周期支持计划、并对Docker技术的稳定性和可维护性提升带来了帮助。

注：文章写于 `2017年8月` ,文中所讲方法可能会过时，请查看Docker官方最新安装文档 [Get Docker CE for Ubuntu | Docker Documentation](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)，本文仅供参考。

#### Docker CE 还是 Docker EE

##### Docker CE

Docker CE表示社区版，是免费的Docker产品的新名称，Docker CE包含了完整的Docker平台，非常适合开发人员和运维团队构建容器APP。


##### Docker EE

Docker EE表示企业版，由公司支持，可在经过认证的操作系统和云提供商中使用，并可运行来自Docker Store的、经过认证的容器和插件。

Docker EE提供三个服务层次：

| 服务层级 | 功能 |
| -------- | ----- |
| Basic	| 1. 包含用于认证基础设施的Docker平台<br> 2. Docker公司的支持<br> 3. 经过认证的、来自Docker Store的容器与插件 |
| Standard | 1. 添加高级镜像与容器管理<br> 2. LDAP/AD用户集成<br> 3. 基于角色的访问控制(Docker Datacenter) |
| Advanced | 1. 添加Docker安全扫描<br> 2. 连续漏洞监控 |


##### 版本迭代

Docker从17.03开始，转向基于时间的 `YY.MM` 形式的版本控制方案，类似于Canonical为Ubuntu所使用的版本控制方案。

Docker CE有两种版本：

* edge版本每月发布一次，主要面向那些喜欢尝试新功能的用户。
* stable版本每季度发布一次，适用于希望更加容易维护的用户（稳定版）。

edge版本只能在当前月份获得安全和错误修复。而stable版本在初始发布后四个月内接收关键错误修复和安全问题的修补程序。这样，Docker CE用户就有一个月的窗口期来切换版本到更新的版本。

Docker EE和stable版本的版本号保持一致，每个Docker EE版本都享受为期一年的支持与维护期，在此期间接受安全与关键修正。

***

#### 官方安装方法

##### 系统要求

安装Docker CE,需要64位的Ubuntu系统：

* Zesty 17.04
* Xenial 16.04 (LTS)
* Trusty 14.04 (LTS)

我的系统是 `Ubuntu 16.04.2 LTS` 版本，通过命令 `lsb_release -a` 我们可以查看到:

```
$ sudo lsb_release -a
[sudo] password for tiger: 
LSB Version:	core-9.20160110ubuntu0.2-amd64
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.2 LTS
Release:	16.04
Codename:	xenial
```

通过 `uname -r` 来查看内核信息：

```
$ uname -r
4.4.0-85-generic
```

***

##### 卸载旧版本Docker

旧版本的docker被称为 `docker` 或者 `docker-engine`，而现在最新的Docker CE包被称为 `docker-ce`。在安装之前，需要先卸载旧版本(如果之前有安装)：

```
$ sudo apt-get remove docker docker-engine docker.io
```

另外原来 `/var/lib/docker/` 目录下的镜像，容器，数据卷，网络等都会保留，新安装的docker任然可以使用这些内容。

##### 14.04 Trusty 需要安装额外包

在 14.04 系统版本下，需要安装 `linux-image-extra-*` 包以允许Docker使用 `aufs` 存储驱动程序：

```
$ sudo apt-get update

$ sudo apt-get install \
    linux-image-extra-$(uname -r) \
    linux-image-extra-virtual
```

***

#### 安装 Docker CE

##### 更新源

```
$ sudo apt-get update
```

##### 允许通过HTTPS使用存储库

```
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

##### 导入官方 GPG 密钥

```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

验证密钥指纹是否正确 `9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88`

```
$ sudo apt-key fingerprint 0EBFCD88
```

操作记录如下：

```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
OK
$ sudo apt-key fingerprint 0EBFCD88
pub   4096R/0EBFCD88 2017-02-22
      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22
```

***

##### 选择稳定版本

使用如下命令安装稳定版本的docker-ce,64位系统： `amd64` or `x86_64`:

```
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

##### 更新源列表

```
$ sudo apt-get update
```

##### 安装最新版本的Docker CE

```
$ sudo apt-get install docker-ce
```

##### 安装特定版本的Docker CE

使用命令 `$ apt-cache madison docker-ce` 查看可安装的版本列表：

```
$ apt-cache madison docker-ce
 docker-ce | 17.06.1~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.06.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.03.2~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.03.1~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 docker-ce | 17.03.0~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
 ```

中间一项为版本名称，执行命令选择安装指定版本 `$ sudo apt-get install docker-ce=<VERSION>` ,比如：

```
$ sudo apt-get install docker-ce=17.06.1~ce-0~ubuntu
[sudo] password for tiger: 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
docker-ce is already the newest version (17.06.1~ce-0~ubuntu).
0 upgraded, 0 newly installed, 0 to remove and 34 not upgraded.
```

***

安装完成后docker守护进程会自动启动。

***

##### 验证

执行命令 `docker` 查看安装是否成功：

```
$ docker 

Usage:	docker COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default "/home/tiger/.docker")
  -D, --debug              Enable debug mode
      --help               Print usage
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/home/tiger/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/home/tiger/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/home/tiger/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  config      Manage Docker configs
  container   Manage containers
  image       Manage images
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.
```

***

#### 为当前用户添加管理员权限

Docker进程启动后，执行docker命令都必须带上 `sudo` 才行，否则会报 `permission denied` 的错误：

```
$ docker info
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.30/info: dial unix /var/run/docker.sock: connect: permission denied
```

解决方法是将当前用户加入到 `docker` 用户分组下。

***

##### 将当前用户添加到 `docker` 分组下

```
$ sudo usermod -aG docker <your-user>
```

或者直接用 `$USER` 表示当前用户：

```
$ sudo usermod -aG docker $USER
```

然后重启系统：

```
$ sudo reboot
```

再执行时就不会报错了：

```
$ docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 17.06.1-ce
Storage Driver: aufs
 Root Dir: /var/lib/docker/aufs
 Backing Filesystem: extfs
 Dirs: 0
 Dirperm1 Supported: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins: 
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
Swarm: inactive
```

***

#### 使用阿里云Docker CE镜像源安装

##### Ubuntu 14.04 16.04 (使用apt-get进行安装)

```
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装 Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce


# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# apt-cache madison docker-ce
#   docker-ce | 17.03.1~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
#   docker-ce | 17.03.0~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
# Step 2: 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.1~ce-0~ubuntu-xenial)
# sudo apt-get -y install docker-ce=[VERSION]
```

##### CentOS 7 (使用yum进行安装)

```
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo service docker start


# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2 : 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]
```

***

#### 配置阿里云Docker镜像加速器

打开阿里云 [开发者平台](https://dev.aliyun.com/) - `管理中心` - `Docker Hub 镜像站点`。可以看到 `您的专属加速器地址 https://xxxxx.mirror.aliyuncs.com`

##### 配置Docker加速器

通过修改daemon配置文件 `/etc/docker/daemon.json` (没有时新建该文件) 来使用加速器：

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xxxxx.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

注意：**上文代码段中给出的镜像加速器地址中的 `xxxxx` 为阿里云在你注册账户后分配的指定地址名称，切记要修改为自己账户的给定地址。**

***

#### 相关参考

* [Get Docker CE for Ubuntu | Docker Documentation](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)
* [Docker 17.03系列教程（一）Docker EE/Docker CE简介与版本规划 | 周立|Spring Cloud](http://www.itmuch.com/docker/docker-1/)
* [Docker CE 镜像源站-博客-云栖社区-阿里云](https://yq.aliyun.com/articles/110806)
