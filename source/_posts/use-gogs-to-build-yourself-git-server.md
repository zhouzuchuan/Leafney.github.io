---
title: Linux系统下使用Gogs搭建Git Service
date: 2016-12-26 20:18:53
tags:
	- Gogs
	- Git Server
	- 代码管理
description: 目前为止最好的自搭建Git服务端项目-Gogs
---

#### 基本环境搭建

* 新建用户

Gogs 默认以 `git` 用户运行（你应该也不会想一个能修改 `ssh` 配置的程序以 `root` 用户运行吧？）。
运行 `sudo adduser git` 新建好 `git` 用户并设置密码。

然后 `su - git` 切换至 `git` 用户登录。

具体操作如下：

```
$ sudo adduser git
```

根据提示信息，输入新账户 `git` 密码，其他用户信息直接敲回车即可。

*** 

* 安装git

因为新创建的用户 `git` 没有设置管理员权限，所以我们先在 `root` 账户或其他管理员账户下安装 `git` :

```
$ sudo apt-get update
$ sudo apt-get install git
$ git --version  //检查git是否安装成功
```

***

切换到新创建的git用户：

```
$ su - git
```

进入用户git的根目录下：

```
$ cd ~
```

***

* 下载解包

我使用的是预编译的二进制安装包。需要从源码编译的话，请参考一般 Go 语言项目的编译。

数据库采用 `Sqlite3` 数据库，如想使用`Mysql` 获取其他数据库，请参考官网的安装方法。

创建gogs应用的解压目录：

```
$ mkdir goapp
$ cd goapp
$ pwd
/home/git/goapp
```

从 [官网](https://gogs.io/docs/installation/install_from_binary.html) 或从 [GitHub Tags](https://github.com/gogits/gogs/tags) 下载当前最新的版本 `v0.9.13` 版，[linux amd64](https://dl.gogs.io/gogs_v0.9.13_linux_amd64.zip) 并解压：

```
$ wget https://dl.gogs.io/gogs_v0.9.13_linux_amd64.zip
$ unzip gogs_v0.9.13_linux_amd64.zip
$ ls
gogs gogs_v0.9.13_linux_amd64.zip
```

进入 `gogs` 目录：

```
$ cd gogs
$ ls
gogs LICENSE public README.md README_ZH.md scripts templates
```

创建自定义配置文件目录并修改文件夹权限：

```
$ mkdir custom
$ mkdir custom/conf

$ sudo chmod -R 777 custom
```

在当前 `git` 用户下，如果提示没有 `sudo` 权限，可以先临时更新为其他用户，更改目录读写权限，改完后再切换回 `git` 用户：

```
$ su root
$ sudo chmod -R 777 custom
$ su - git
```

创建日志目录并修改文件夹权限：

```
$ mkdir log

$ sudo chmod -R 777 log
```

***

* 启动gogs:

```
$ pwd
/home/git/goapp/gogs

$ ./gogs web
```

执行命令后，出现 `Listen:http://0.0.0.0:3000` 提示信息，表示 `gogs` 启动成功。

然后访问 `http://服务器IP:3000/` 来进行安装，填写好表单之后提交就可以了。默认第一个创建的账户为管理员账户。

表单中指定了 `Database Settings` -- `Path` 为数据库的存放目录。`Application General Settings` -- `Repository Root Path` 为仓库文件的存放目录。

***

#### 配置Nginx

在管理员账户下执行：

```
$ sudo apt-get install nginx
```

在 `/etc/nginx/conf.d` 目录下添加 `gogsweb.conf` 文件，填入如下内容：

```
server {		
	server_name git.****.com;
	listen 80;

	location / {
		proxy_pass http://127.0.0.1:3000/;
	}
}

```

然后通过 `sudo service nginx restart` 重启 `nginx` 服务。

***

#### 配置 supervisor 启动

在管理员账户下执行：

```
$ sudo apt-get install supervisor
```

在 `/etc/supervisor/conf.d` 目录下添加 `gogsweb.conf` 文件，填入如下内容：

```
[program:gogs]
directory=/home/git/goapp/gogs/
command=/home/git/goapp/gogs/gogs web
autostart=true
autorestart=true
startsecs=10
stdout_logfile=/home/git/goapp/gogs/log/stdout.log
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
stdout_capture_maxbytes=1MB
stderr_logfile=/home/git/goapp/gogs/log/stderr.log
stderr_logfile_maxbytes=1MB
stderr_logfile_backups=10
stderr_capture_maxbytes=1MB
user = git
environment = HOME="/home/git", USER="git"
```

以上的配置信息在 `gogs` 目录下的 `Scripts` 文件夹下有给出，可参考。

##### 开启 supervisor UI 管理台

编辑 `/etc/supervisor/supervisor.conf` 主配置文件，修改或添加(通过apt-get的方式安装后不包含该配置) 以下内容：

```
[inet_http_server]         ; inet (TCP) server disabled by default
port=*:9001        ; (ip_address:port specifier, *:port for all iface)
username=user              ; (default is no username (open server))
password=123               ; (default is no password (open server))
```

`port` 中 `*.9001` 表示接受任意网络的请求，如果设置成 `127.0.0.1` 则只接受本地访问请求。

通过通过 `sudo supervisorctl reload` 重启服务。

重启之后，如果之前的配置也没有问题的话，现在在浏览器上通过域名即可浏览该站点了。

在浏览器中输入 `服务器IP:9001` 来访问 supervisor UI 的管理端页面。

***

#### Gogs的个性化配置

1. 顶部导航菜单中 “帮助” 链接文字，未登录用户不显示 “帮助” 按钮：
	* 目录:   `gogs\templates\base\head.tmpl`
	* 位置：  ```
	<a class="item" target="_blank" href="http://gogs.io/docs" rel="noreferrer">{{.i18n.Tr "help"}}</a>
	```
	* 将 `{{else}}` 标签下面的改行注释掉。
2. 底部右下角显示 “官方网站” 字样，修改为 “Gogs官方网站”：
	* 目录： `gogs\templates\base\footer.tmpl`
	* 位置： ```
	<a target="_blank" href="http://gogs.io">{{.i18n.Tr "website"}}</a>
	```
	* 更改为：```
	<a target="_blank" href="https://gogs.io">Gogs{{.i18n.Tr "website"}}</a>
	```
3. 首页，首页样式改版：
	* 目录： `gogs\templates\home.tmpl`

4. 为md文件中的a标签链接添加target属性，在新页面打开

	在 `goapp/gogs/public/js/` 目录下，添加名为 `mdlinktarget-1.0.min.js` 内容如下：

```
$(document).ready(function(){$('#file-content a[href^="http"]').each(function(){$(this).attr("target","_blank")})});
```

在项目目录 `gogs\templates\base` 下找到 `footer.tmpl` 文件，在最下面添加js引用：

```
<script src="{{AppSubUrl}}/js/mdlinktarget-1.0.min.js"></script>
```

*** 

#### 参考

* [Installation - Gogs - Go Git Service](https://gogs.io/docs/installation)
* [使用 Gogs 搭建自己的 Git 服务器 - My Nook](https://mynook.info/blog/post/host-your-own-git-server-using-gogs)

