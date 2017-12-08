---
title: Ubuntu下配置Flask运行环境
date: 2016-05-09 21:15:00
tags:
    - Python
    - Flask
categories: 
    - 开发笔记
description: Ubuntu下配置Flask运行环境
---

Virtualenv + Flask + Gunicorn + Supervisor + Nginx

#### 配置

假设我所在的当前账户名为 `tiger` 。

##### 准备`python`环境

```
$ sudo apt-get udpate
// 安装python 和 pip (如果已经安装可以忽略)
$ sudo apt-get install python-dev python-pip -y
```

##### 创建virtualenv虚拟环境

Virtualenv可以为每个Python应用创建独立的开发环境，使他们互不影响，Virtualenv能够做到：

* 在没有权限的情况下安装新套件
* 不同应用可以使用不同的套件版本
* 套件升级不影响其他应用

```
//安装或通过执行：
$ sudo pip install virtualenv
```

我通常创建一个包含 `venv` 文件夹的项目文件夹:

```
$ mkdir myproject
$ cd myproject
$ virtualenv venv
New python executable in venv/bin/python2
Also creating executable in venv/bin/python
Installing setuptools, pip...done.
```

现在，每次需要使用项目时，必须先激活相应的环境。

```
$ ls
-- venv
$  . venv/bin/activate

//结果：
(venv)tiger@VirtualBox:~/xbox/myflask$ 
```

你现在进入你的 `virtualenv` （注意查看你的 shell 提示符已经改变了）。

##### 安装Flask

现在可以开始在你的 `virtualenv` 中安装 `Flask` 了:  

```
$ pip install Flask

//结果：
(venv)tiger@VirtualBox:~/xbox/myflask$ pip install Flask
Downloading/unpacking Flask
  Downloading Flask-0.10.1.tar.gz (544kB): 544kB downloaded
    ......
  Successfully installed Flask Werkzeug Jinja2 itsdangerous MarkupSafe
Cleaning up...
```

几秒钟后就安装好了。

##### 退出虚拟环境

可通过`deactivate`退出虚拟环境：

```
(venv)tiger@VirtualBox:~/xbox/myflask$ deactivate
tiger@VirtualBox:~/xbox/myflask$
```

具体可详见：[Flask-安装](http://dormousehole.readthedocs.io/en/latest/installation.html)

##### 启动falsk

在当前目录下新建一个 `hello.py` 的文件：

```
(venv) $ vim hello.py
(venv) $ ls
venv hello.py
```

创建一个简单的Flask程序：

```
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    app.run()
```

修改`hello.py` 的权限：

```
(venv) $ chmod a+x hello.py
```

启动 flask:

```
(venv) $ python hello.py
```

此时，用浏览器访问 `http://127.0.0.1:5000` 就能看到网页显示 `hello world`。

*** 

#### 配置Gunicorn

Gunicorn是用于部署WSGI应用的，任何支持WSGI的都可以，虽说直接执行 `python hello.py` 这样网站也能跑起来，但那是方便开发而已，在线上环境，还是需要更高效的组件来做。

##### 安装

```
(venv)$ pip install gunicorn
```

然后可以执行：

```
gunicorn -w 4 -b 127.0.0.1:8000 hello:app

//结果：
(venv) $ gunicorn -w 4 -b 127.0.0.1:8000 hello:app
[2016-05-08 16:07:36 +0000] [1337] [INFO] Starting gunicorn 19.4.5
[2016-05-08 16:07:36 +0000] [1337] [INFO] Listening at: http://127.0.0.1:8000 (1337)
[2016-05-08 16:07:36 +0000] [1337] [INFO] Using worker: sync
[2016-05-08 16:07:36 +0000] [1342] [INFO] Booting worker with pid: 1342
[2016-05-08 16:07:36 +0000] [1343] [INFO] Booting worker with pid: 1343
[2016-05-08 16:07:36 +0000] [1344] [INFO] Booting worker with pid: 1344
[2016-05-08 16:07:36 +0000] [1345] [INFO] Booting worker with pid: 1345

```

> “hello” is the name of the file (without extension). And “app” is the name of the Flask object.

这里 "hello" 是python文件的名称(不包含扩展名)，冒号后面的"app" 是flask程序中`app = Flask(__name__)`创建的这个Flask对象的名称。

这里我们用了 `8000` 的端口进行访问，原先的 `5000` 并没有启用。

`-w`表示开启多少个 `worker`，`-b` 表示 `gunicorn` 开发的访问地址  

想要结束 `gunicorn` 只需执行 `pkill gunicorn` : 

```
(venv)  $ pkill gunicorn

//结果：
[2016-05-08 16:09:28 +0000] [1337] [INFO] Handling signal: term
[2016-05-08 16:09:28 +0000] [1344] [INFO] Worker exiting (pid: 1344)
[2016-05-08 16:09:28 +0000] [1342] [INFO] Worker exiting (pid: 1342)
[2016-05-08 16:09:28 +0000] [1343] [INFO] Worker exiting (pid: 1343)
[2016-05-08 16:09:28 +0000] [1345] [INFO] Worker exiting (pid: 1345)
[2016-05-08 16:09:29 +0000] [1337] [INFO] Shutting down: Master

```

我们还可以在一个独立的配置文件中来设置，新增配置文件 `gunicorn.conf`：

```
(venv) $ ls
-- hello.py  hello.pyc  venv
(venv) $ vim gunicorn.conf

//写入下列内容：
workers=4
bind='127.0.0.1:8000'
```

上面使用`pkill gunicorn`的方式来停止进程，太过于繁琐，因此出现了另外一个神器---`supervisor`，一个专门用来管理进程的工具，还可以管理系统的工具进程。

***

#### Supervisor

Supervisor 是用Python实现的一款非常实用的进程管理工具。

##### 安装supervisor

```
(venv) $ pip install supervisor
```

###### 常用操作

```
## 启动服务
$ sudo service supervisor start
## 停止服务
$ sudo service supervisor stop
## 也可以直接 kill pid
$ ps -A | grep supervisor
```

###### 生成supervisor默认配置文件

```
(venv) $ echo_supervisord_conf > supervisor.conf

## 添加一个`logs`目录，用来后面存放日志：
(venv) $ mkdir logs
(venv) $ ls
gunicorn.conf  hello.py  hello.pyc  logs  supervisor.conf  venv 
```

###### 修改supervisor配置文件，添加gunicorn进程管理

```
(venv) $ vim supervisor.conf
```

在 `supervisor.conf` 配置文件底部添加：(假设我的工作目录为： `/home/tiger/myflask/myproject/` )

```
##文件内容
[program:hello]  ## 服务的名称，后面操作会用到
command=/home/tiger/myflask/myproject/venv/bin/gunicorn hello:app -c /home/tiger/myflask/myproject/gunicorn.conf      ; supervisor启动命令
directory=/home/tiger/myflask/myproject     ; 项目的文件夹路径
user=tiger
autostart=true                           ; 是否自动启动
autorestart=true                         ; 是否自动重启
##log文件的位置
stdout_logfile=/home/tiger/myflask/myproject/logs/gunicorn_supervisor.log     ; log 日志
stderr_logfile=/home/tiger/myflask/myproject/logs/gunicorn_supervisor.err     ; 错误日志
```

`supervisor`的基本使用命令

```
supervisord -c supervisor.conf            通过配置文件启动supervisor
supervisorctl -c supervisor.conf status   查看supervisor的状态
supervisorctl -c supervisor.conf reload                    重新载入 配置文件
supervisorctl -c supervisor.conf start [all]|[appname]     启动指定/所有 supervisor管理的程序进程
supervisorctl -c supervisor.conf stop [all]|[appname]      关闭指定/所有 supervisor管理的程序进程

```

添加完上面的内容后，保存退出。执行操作：

当前目录：

```
(venv) $ ls
-- gunicorn.conf  hello.py  hello.pyc  logs  supervisor.conf  venv  
```

通过配置文件 启动 `supervisor` :      

```
(venv) $ supervisord -c supervisor.conf 
```

查看 `supervisor` 的状态：    

```
(venv) $ supervisorctl -c supervisor.conf status
-- hello              RUNNING   pid 1550, uptime 0:04:08
```

停止 设置的 服务 `hello` ：   

```
(venv) $ supervisorctl -c supervisor.conf stop hello 
-- hello: stopped
```

再次查看 服务 `hello` 的状态 ：   

```
(venv) $ supervisorctl -c supervisor.conf status
-- hello              STOPPED   May 08 05:23 PM
```

*** 

自动启动：

那么，如果想开机时自动启动怎么办呢？或者说，如果机器重启了，那WEB服务就断了。
其实呢，也很简单，只要在 `/etc/rc.d/rc.local` 中加入一句就可以了：

```
supervisord -c /home/tiger/myflask/myproject/supervisor.conf
```

有了Gunicorn、Supervisor，本地的环境的算是搭好了，但是我们需要让VPS上的网站从外网可以访问，这时候需要Nginx。

现在，我们只能在本机上通过`http://127.0.0.1:8000` 的方式来访问的，但在外网上是无法访问到的。

*** 

#### 配置Nginx

在安装Nginx之前，要先退出上面步骤中操作所在的`venv`环境，执行：

```
(venv) $ deactivate
```

##### 安装nginx

```
$ sudo apt-get install nginx
```

安装完成后访问`localhost`可以看到 **Welcome to nginx on Ubuntu!** 的页面。

`nginx` 的默认网站目录 `/usr/share/nginx/html`

##### 常用配置命令

```
##启动服务
$ sudo service nginx start
##重启和暂停服务
$ sudo service nginx restart
$ sudo service nginx stop
##查看状态
$ sudo service nginx status
```

或下面的命令：

```
$ sudo /etc/init.d/nginx start
$ sudo /etc/init.d/nginx stop
$ sudo /etc/init.d/nginx restart
$ sudo /etc/init.d/nginx status
```

Nginx的配置文件和Supervisor类似，不同的程序可以分别配置，然后被总配置文件`include`：

```
## Nginx的主配置文件地址
/etc/nginx/nginx.conf

## 新建单独的配置文件
## 在conf.d目录下
$ sudo vim /etc/nginx/conf.d/helloweb.conf

## 然后写入下列内容：
server {
        listen   80;             //端口
        server_name 192.168.1.144;   //访问域名
        root /home/tiger/myflack/myproject;
        access_log /home/tiger/myflask/myproject/logs/nginx_access.log;
        error_log /home/tiger/myflask/myproject/logs/nginx_error.log;
        location / {
                proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
                proxy_redirect off;
                if (!-f $request_filename) {
                        proxy_pass http://127.0.0.1:8000;  //这里是flask的gunicorn端口
                        break;
                }
        }
}

```

也可以用下面的最简配置：

```
server {
    listen 80;
    server_name _; # _表示 localhost 

    root /home/tiger/myflack/myproject; # hello.py 文件所在的目录

    location / {
        proxy_pass http://127.0.0.1:8000;  //这里是flask的gunicorn端口
    }
}
```

配置完成之后，`sudo service nginx restart` 重启一下服务。

现在，在浏览器中输入 `http://127.0.0.1` 即可正常访问。

*** 

#### 总结

* Flask应用的基本部署依赖包括一个应用容器（比如Gunicorn）和一个反向代理（比如Nginx）。 
* Gunicorn应该退居Nginx幕后并监听127.0.0.1（内部请求）而非0.0.0.0（外部请求）。
* 仍有疑问的地方：在虚拟环境下安装的`supervisor` 并非是全局的，经测试 `kill` 掉 `supervisor` 进程后也不会自动重启，或如何随系统启动？该问题待进一步测试

*** 

#### 相关链接

* [virtualenv &mdash; virtualenv 1.7.1.2.post1 documentation](http://virtualenv-chinese-docs.readthedocs.io/en/latest/)
* [GitHub - defshine/flaskblog: Learn python and flask,just a tony blog system](https://github.com/defshine/flaskblog)
* [Gunicorn - Python WSGI HTTP Server for UNIX](http://gunicorn.org/)
* [安装 Flask 0.10 documentation](http://dormousehole.readthedocs.io/en/latest/installation.html)

*** 

* [Flask + Gunicorn + Nginx 部署 - Ray Liang - 博客园](http://www.cnblogs.com/Ray-liang/p/4837850.html)
* [阿里云ECS上环境搭建(virtualenv+flask+gunicorn+supervisor+nginx) - 菩提本无树 - 博客园](http://www.cnblogs.com/herryly/p/4517852.html)
* [阿里云部署 Flask  + WSGI + Nginx 详解 - Ray Liang - 博客园](http://www.cnblogs.com/Ray-liang/p/4173923.html)
* [Supervisor 管理后台守护进程](http://blog.csdn.net/beerium/article/details/8721906)

*** 

* [python web 部署：nginx + gunicorn + supervisor + flask 部署笔记 - 简书](http://www.jianshu.com/p/be9dd421fb8d)
* [VPS环境搭建详解 (Virtualenv+Gunicorn+Supervisor+Nginx) | BeiYuu.com](http://beiyuu.com/vps-config-python-vitrualenv-flask-gunicorn-supervisor-nginx/)
* [基于 Flask 实现 RESTful API | Ross's Page](http://xhrwang.me/2014/12/13/restful-api-by-flask.html)
* [Ubuntu 14.04 系统基于 Gunicorn 和 Nginx 部署 Flask 应用 | Ross's Page](http://xhrwang.me/2014/12/17/deploy-flask-app-with-gunicorn-and-nginx.html)
* [Flask+Nginx+Gunicorn+Redis+Mysql搭建一个小站 | Alex&#039;s Blog](http://codingnow.cn/server/539.html)
* [How to Run Flask Applications with Nginx Using Gunicorn &#8211; Onur Güzel](http://www.onurguzel.com/how-to-run-flask-applications-with-nginx-using-gunicorn/) *
* [dnsmasq配置域名重定向和dns缓存 | Alex&#039;s Blog](http://codingnow.cn/server/431.html)
* [在阿里云CentOS7中配置基于Nginx+Supervisor+Gunicorn的Flask项目 - simpleapples](http://www.simpleapples.com/2015/06/configure-nginx-supervisor-gunicorn-flask/)
* [virtualenv 环境下 Django + Nginx + Gunicorn+ Supervisor 搭建 Python Web](http://news.oneapm.com/flask-nginx-gunicorn-supervisor-python/)
* [部署 | Flask之旅](https://spacewander.github.io/explore-flask-zh/14-deployment.html)
* [CentOS 6 下使用 Nginx，Gunicorn 以及 Supervisor 部署 Flask 应用 - 杜顺帆的个人博客](http://dushunfan.com/2013/03/17/deploy-flask-in-centos6-nginx-gunicorn-supervisor/)
* [在 Ubuntu 中 Nginx,Gunicorn 部署 Flask | autarch](http://autarch.me/archives/2016/01/%E5%9C%A8-Ubuntu-%E4%B8%AD-Nginx-Gunicorn-%E9%83%A8%E7%BD%B2-Flask.html) * 
* [阿里云Python+Flask环境搭建](https://zhuanlan.zhihu.com/p/22126999)
