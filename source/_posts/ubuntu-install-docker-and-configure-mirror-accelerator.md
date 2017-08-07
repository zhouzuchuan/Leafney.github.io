---
title: Ubuntu16.04安装Docker及配置镜像加速器
date: 2017-01-14 18:25:34
tags: 
	- Ubuntu
	- Docker
description:  最近将虚拟机中的Ubuntu系统从14.04更新到了16.04.1版本，又要重新安装Docker服务，在安装的时候发现Docker的官方安装教程又有了更新，之前的安装方法已经过时。
---

最近将虚拟机中的Ubuntu系统从14.04更新到了16.04.1版本，又要重新安装Docker服务，在安装的时候发现Docker的官方安装教程又有了更新，之前的安装方法已经过时。

#### 过时的安装方法

之前官网中提供的安装方法为：

```
$ curl -sSL https://get.docker.com/ | sudo sh
```

现在如果再执行该命令，会直接报错。

#### 官方推荐安装方法

##### 查看系统内核

Docker需要安装在Linux 64位系统下，内核版本在 3.10 以上。可以通过 `uname -r` 来查看内核信息：

```
$ uname -r
4.4.0-31-generic
```

##### 更新源，安装CA证书

```
$ sudo apt-get update
$ sudo apt-get install apt-transport-https ca-certificates
```

##### 导入 GPG 密钥

```
$ sudo apt-key adv \
               --keyserver hkp://ha.pool.sks-keyservers.net:80 \
               --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
```

##### 添加docker源

根据当前系统版本，添加docker源命令：

```
$ echo "<REPO>" | sudo tee /etc/apt/sources.list.d/docker.list
```

只要将 `<REPO>` 替换成相应系统的源地址即可。

我这里当前系统Ubuntu16.04，源地址为：`deb https://apt.dockerproject.org/repo ubuntu-xenial main`，所以只需如下命令：

```
$ echo "deb https://apt.dockerproject.org/repo ubuntu-xenial main" | sudo tee /etc/apt/sources.list.d/docker.list
```

其他版本系统源地址如下：

| Ubuntu version | Repository |
| --------  | --------- |
| Precise 12.04 (LTS) | deb https://apt.dockerproject.org/repo ubuntu-precise main |
| Trusty 14.04 (LTS) | deb https://apt.dockerproject.org/repo ubuntu-trusty main |
| Wily 15.10 | deb https://apt.dockerproject.org/repo ubuntu-wily main |
| Xenial 16.04 (LTS) | deb https://apt.dockerproject.org/repo ubuntu-xenial main |

##### 更新源列表

```
$ sudo apt-get update
```

##### 验证 APT 能否正确获取

执行如下命令会从docker官方仓库中列出所有docker的可安装版本：

```
$ apt-cache policy docker-engine
docker-engine:
  Installed: null
  Candidate: 1.12.6-0~ubuntu-xenial
  Version table:
 *** 1.12.6-0~ubuntu-xenial 500
        500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
        100 /var/lib/dpkg/status
     1.12.5-0~ubuntu-xenial 500
        500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
     1.12.4-0~ubuntu-xenial 500
        500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
     1.12.3-0~xenial 500
        500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
     ...
     ...
```

##### 安装docker

默认会安装推荐的版本 `Candidate` 项列出的，也是最新的版本

```
$ sudo apt-get install -y docker-engine
```

##### 启动docker服务

```
$ sudo service docker start
```

##### 验证

在命令行下输入 `docker` ,如提示docker的 `[OPTIONS]` 说明，则表示docker服务已经安装成功了。

```
$ docker
Usage: docker [OPTIONS] COMMAND [arg...]
       docker [ --help | -v | --version ]

A self-sufficient runtime for containers.

Options:

  --config=~/.docker              Location of client config files
  -D, --debug                     Enable debug mode
  -H, --host=[]                   Daemon socket(s) to connect to
  -h, --help                      Print usage
  -l, --log-level=info            Set the logging level
  --tls                           Use TLS; implied by --tlsverify
  --tlscacert=~/.docker/ca.pem    Trust certs signed only by this CA
  --tlscert=~/.docker/cert.pem    Path to TLS certificate file
  --tlskey=~/.docker/key.pem      Path to TLS key file
  ...
  ...
```
***

#### 将当前用户添加到docker的用户组

Docker安装成功后，如果想查看docker的信息，执行 `docker info` 命令时可能会提示如下信息：

```
$ docker info
Cannot connect to the Docker daemon. Is the docker daemon running on this host?
```

这是因为必需以管理员权限或使用 `sudo` 来运行命令才可以。为了以后执行命令时不用每次都必需添加 `sudo`，可以将当前用户加入到docker用户组中。

##### 创建 `docker` 分组

```
$ sudo groupadd docker
```

##### 将当前用户添加到组

```
$ sudo usermod -aG docker $USER
```

注意：这里不用更改 `$USER` 这个参数，`$USER` 这个环境变量就是指当前用户名

##### 重启系统

更改完成后，还需要重启系统才能看到效果。

```
$ sudo reboot
```

##### 创建一个测试容器

可通过如下命令来创建一个测试容器：

```
$ docker run hello-world
```

输出：

```
Hello from Docker.
This message shows that your installation appears to be working correctly.
...
```

***

#### 配置阿里云Docker镜像加速器

打开 [开发者平台](https://dev.aliyun.com/) -- `管理中心` -- `加速器` 。可以看到 “您的专属加速器地址” 即 `https://xxxxxxx.mirror.aliyuncs.com` 。 

注意：这里以 `Ubuntu 16.04` 系统为例，其他系统请到上述页面中查看相应操作命令。

##### 配置Docker加速器

通过 `docker info` 命令可以知道上面安装好的Docker的版本为 `1.12.6` 。所以请通过修改daemon配置文件 `/etc/docker/daemon.json` (没有时新建该文件) 来使用加速器：

```
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xxxxxxx.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker
```

##### 注意事项

针对于Docker版本在1.10以下的情况，可以使用如下的配置方法：

```
sudo mkdir -p /etc/systemd/system/docker.service.d

sudo tee /etc/systemd/system/docker.service.d/mirror.conf <<-'EOF'
[Service]
ExecStart=/usr/bin/docker daemon -H fd:// --registry-mirror=https://xxxxxxx.mirror.aliyuncs.com
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

但是该方法并不适用于1.12.0版本之后的Docker上。因为Docker的可执行文件名称从 `docker` 改成了 `dockerd`。如果使用了以上脚本，可能会报如下的错误：

```
$ sudo systemctl restart docker
Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.
```

所以在配置加速器时一定要按照相应版本来设置。

还要注意一点：**上文代码段中给出的镜像加速器地址中的 `xxxxxxx` 为阿里云在你注册账户后分配的指定地址名称，切记要修改为自己账户的给定地址。**

***

#### 相关参考

* [Install Docker on Ubuntu - Docker](https://docs.docker.com/engine/installation/linux/ubuntulinux/)
* [How To Install and Use Docker on Ubuntu 16.04 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04)
* [ubuntu16.04安装docker - 程序园](http://www.voidcn.com/blog/gsls200808/article/p-6323832.html)
* [Ubuntu 16.04安装docker加速器后无法启动docker-问答-云栖社区-阿里云](https://yq.aliyun.com/ask/43505)

