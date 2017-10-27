---
title: Linux下使用SSH密钥连接Github
date: 2017-03-03 15:43:38
tags:
	- Linux
	- GitHub
	- SSH
description: 在 Linux 系统下通过 SSH 密钥来连接 GitHub (Mac系统下设置方法相同)
---

在 Linux 系统下如何通过 SSH 密钥来连接 GitHub (Mac系统下设置方法相同)。

#### 引申

* Linux下 ：[Linux下使用SSH密钥连接Github](/2017/03/03/using-ssh-key-connection-github-in-linux/)
* Windows下：[使用SSH密钥连接Github](/2016/09/24/using-ssh-key-connection-github/)

***

#### Ubuntu 系统初始化配置git环境

测试系统版本 : `Ubuntu 16.04 LTS`

##### 更新软件源

```
$ echo "deb http://cn.archive.ubuntu.com/ubuntu/ xenial main restricted universe multiverse" >> /etc/apt/sources.list && \
echo "deb http://cn.archive.ubuntu.com/ubuntu/ xenial-security main restricted universe multiverse" >> /etc/apt/sources.list && \
echo "deb http://cn.archive.ubuntu.com/ubuntu/ xenial-updates main restricted universe multiverse" >> /etc/apt/sources.list
```

##### 安装git依赖

```
$ apt-get update
$ apt-get install git
```

##### 设置时区

```
$ ln -sf /usr/share/zoneinfo/Asia/ShangHai /etc/localtime
$ echo "Asia/Shanghai" > /etc/timezone
$ dpkg-reconfigure -f noninteractive tzdata
```

***

#### Alpine 系统初始化配置git环境

测试系统版本 : `Alpine 3.5`

##### 更新软件源

```
$ echo "http://dl-4.alpinelinux.org/alpine/v3.5/main" >> /etc/apk/repositories
$ echo "http://dl-4.alpinelinux.org/alpine/v3.5/community" >> /etc/apk/repositories
```

##### 安装依赖

The `ssh-keygen` command is part of `OpenSSH` (package "openssh").

```
$ apk update
$ apk add git openssh 
```

##### 设置时区

```
$ apk add tzdata
$ ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ echo "Asia/Shanghai" > /etc/timezone
```

***

#### Mac 系统初始化配置git环境

Mac系统上安装git,可以直接从git网站下载安装包,访问 [Git - Downloads](https://git-scm.com/download/mac) 安装.

也可以通过 `homebrew` 进行安装:

```
$ brew install git
```

***

#### 设置git账户

执行如下两条命令设置git账户的用户名和密码：

```
$ git config --global user.name "Your Name"
$ git config --global user.email "youremail@domain.com"

$ git config --list
```

***

#### 生成SSH公钥

SSH 公钥默认储存在账户的主目录下的 `~/.ssh` 目录。先确认是否已经有一个公钥了：

```
$ cd ~/.ssh
/bin/sh: cd: can't cd to /root/.ssh
```

主要是看是否存在 `id_dsa` 或 `id_rsa` 文件。有 `.pub` 后缀的文件就是 `公钥`，另一个文件则是密钥。

##### 创建新的SSH密钥

如果已经存在公钥，可跳过这步。如果没有，使用 `ssh-keygen` 来创建：

```
$ cd ~
$ ssh-keygen -t rsa -C "youremail@domain.com"
```

示例：(`xxxxxx@126.com` 为我的账户邮箱)

```
~ # ssh-keygen -t rsa -C "xxxxxx@126.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): # 直接回车，则将密钥按默认路径及文件名进行存储。此时也可以输入特定的文件名
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):  # 根据提示，你需要输入密码和确认密码。可以不填，设置为空值，直接回车
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:yFt14TcP0H+ixy9VKiILPPJ6DVevkKgrbxVFqk7mn5k xxxxxx@126.com
The key's randomart image is:
+---[RSA 2048]----+
|            o.   |
|      .    . o.  |
|     o    . o +. |
|    .... ... ..++|
|   . o .So .   o+|
|  + o Bo= . + + .|
| =   *.* +   o o |
|  ++o o o .   . .|
|  E=++         . |
+----[SHA256]-----+
```

查看生成的文件：

```
$ cd ~/.ssh
~/.ssh $ ls
id_rsa      id_rsa.pub
```

文件 `id_rsa.pub` 就是公钥。

***

#### 在 GibHub 中添加你的公钥

复制公钥 `id_rsa.pub` 文件中的内容。

我这里使用 `XShell` 来登录的linux服务器，可以直接复制出来。或在 `vim` 下，可通过命令 `ggVG` 全选，`+y` 复制选中内容到+寄存器，也就是系统的剪贴板，供其他程序使用。

登陆Github网站，选择 `Settings` –> `SSH and GPG keys` 菜单，点击 `New SSH key` 按钮。
粘贴你的密钥到 `Key` 输入框中并设置 `Title` 信息，点击 `Add SSH key` 按钮完成。

***

#### 连接测试

测试 SSH keys 是否设置成功，执行如下命令：

```
$ ssh -T git@github.com
```

当提示如下信息时，说明正常连通了github:

```
Hi xxxxxx! You've successfully authenticated, but GitHub does not provide shell access.
```

如果你是第一次设置连接github.com,会询问你是否继续,输入 `yes` 即可,这样就会将连接地址记录在本地:

```
$ ssh -T git@github.com
The authenticity of host 'github.com (192.30.253.112)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpxxxxxxxxARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,192.30.253.112' (RSA) to the list of known hosts.
Hi xxxxxx! You've successfully authenticated, but GitHub does not provide shell access.
```

***

然后就可以将本地的项目用github来管理了。

***

#### 更新日志

* 2017-10-27 - 完善"连接测试"内容; 添加Mac系统下安装git配置

