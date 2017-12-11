---
title: Celery分布式任务队列入门(二)-环境配置
date: 2017-12-10 23:57:14
updated: 2017-12-11 16:16:18
tags:
    - Celery
    - Docker
    - RabbitMQ
categories: 
    - Celery分布式任务队列入门
description: 实践在Docker容器中配置Celery运行环境。
---

实践在Docker容器中配置Celery运行环境。


#### 创建Docker容器

在容器 `Ubuntu:16.04` 系统中来搭建，创建容器：

```
$ docker run -it --name celery1 -p 5672:5672 -p 15673:15672 -v /home/tiger/dckerfile/celery1:/app ubuntu /bin/bash
```

需要注意的是在Docker容器中不需要 `sudo` 命令，默认即是 `root` 权限。下面的命令请根据自己所在系统类型自行添加 `sudo` 操作。

##### 更新软件源

```
# echo "deb http://cn.archive.ubuntu.com/ubuntu/ xenial main restricted universe multiverse" >> /etc/apt/sources.list
# echo "deb http://cn.archive.ubuntu.com/ubuntu/ xenial-security main restricted universe multiverse" >> /etc/apt/sources.list


# apt-get update
```

##### 安装Erlang依赖

Erlang可以通过包管理器来安装，或者直接从官方网站下载安装包来安装。

执行如下命令安装：

```
cd /tmp
wget http://packages.erlang-solutions.com/ubuntu/erlang_solutions.asc
# apt-key add erlang_solutions.asc
# apt-get update
# apt-get install erlang
# apt-get install erlang-nox
```

或者直接从网站 [Erlang Downloads](https://packages.erlang-solutions.com/erlang/#tabs-debian) 下载 `.deb` 安装包来安装。

##### 安装RabbitMQ

执行如下命令通过包管理器来安装：

```
# echo "deb https://dl.bintray.com/rabbitmq/debian xenial main" | tee /etc/apt/sources.list.d/bintray.rabbitmq.list

# wget -O- https://www.rabbitmq.com/rabbitmq-release-signing-key.asc | apt-key add -

# apt-get update
# apt-get install rabbitmq-server
```

发现通过上面方法安装的 `Erlang Version` 为 `Erlang/OTP 18 [erts-7.3] [source] [64-bit] [smp:2:2]` ， 
`RabbitMQ Version` 为 `"RabbitMQ","3.5.7"`

都不是官网上的最新版本，但在软件源中来说已是可安装的最新版本。所以如果想要安装官方的最新版本，可以采用直接从官网获取安装包的方式来安装。

##### 安装官方最新版


在Ubuntu系统下直接下载 `rabbitmq` 的 `*.deb` 安装包：

从地址 [Installing on Debian / Ubuntu](https://www.rabbitmq.com/install-debian.html) 看到最新版本是 3.7.0 [rabbitmq-server_3.7.0-1_all.deb](https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.0/rabbitmq-server_3.7.0-1_all.deb)

也可以从网址 [Released Artifacts](https://www.rabbitmq.com/releases/rabbitmq-server/) 选择其他指定版本。

下面以 `rabbitmq-server_3.7.0-1_all.deb` 为例安装：

```
# wget https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.0/rabbitmq-server_3.7.0-1_all.deb
# dpkg -i rabbitmq-server_3.7.0-1_all.deb
```

如果提示依赖其他的包，执行如下命令安装依赖包:

```
# apt-get -f install
```

然后再次执行安装：

```
# dpkg -i rabbitmq-server_3.7.0-1_all.deb
```

我在执行 `apt-get -f install` 时遇到问题，输出提示是要移除 `rabbitmq-server` ，并没有自动安装其他依赖：

```
root@b792ae940e3e:/app# dpkg -i rabbitmq-server_3.7.0-1_all.deb 
Selecting previously unselected package rabbitmq-server.
(Reading database ... 7744 files and directories currently installed.)
Preparing to unpack rabbitmq-server_3.7.0-1_all.deb ...
Unpacking rabbitmq-server (3.7.0-1) ...
dpkg: dependency problems prevent configuration of rabbitmq-server:
 rabbitmq-server depends on erlang-nox (>= 1:19.3) | esl-erlang (>= 1:19.3); however:
  Package erlang-nox is not installed.
  Package esl-erlang is not installed.
 rabbitmq-server depends on logrotate; however:
  Package logrotate is not installed.
 rabbitmq-server depends on socat; however:
  Package socat is not installed.

dpkg: error processing package rabbitmq-server (--install):
 dependency problems - leaving unconfigured
Processing triggers for systemd (229-4ubuntu21) ...
Errors were encountered while processing:
 rabbitmq-server
root@b792ae940e3e:/app# apt-get -f install
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Correcting dependencies... Done
The following packages will be REMOVED:
  rabbitmq-server
0 upgraded, 0 newly installed, 1 to remove and 2 not upgraded.
1 not fully installed or removed.
After this operation, 13.3 MB disk space will be freed.
Do you want to continue? [Y/n]
```

所以，我选择手动安装 `Erland` 的 `.deb` 包。

从 网站 [RabbitMQ Erlang Version Requirements](https://www.rabbitmq.com/which-erlang.html) 中可以看到RabbitMQ和Erlang版本之间的对应关系。这里上面的我按照的是RabbitMQ的 `3.7.0` 版本，所以我可以选择Erlang的最新版即 `20.1.7` 版本安装。下载地址见：[Erlang Downloads](https://packages.erlang-solutions.com/erlang/#tabs-debian) 。

```
# wget http://packages.erlang-solutions.com/site/esl/esl-erlang/FLAVOUR_1_general/esl-erlang_20.1.7-1~ubuntu~xenial_amd64.deb

# dpkg -i esl-erlang_20.1.7-1~ubuntu~xenial_amd64.deb
```

这次安装时也提示缺少依赖，所以我执行：

```
# apt-get -f install 
```

结果是找到了相关的依赖包，输入 `y` 进行安装。

```
root@b792ae940e3e:/app# ls
esl-erlang_20.1.7-1~ubuntu~xenial_amd64.deb  rabbitmq-server_3.7.0-1_all.deb
root@b792ae940e3e:/app# dpkg -i esl-erlang_20.1.7-1~ubuntu~xenial_amd64.deb 
Selecting previously unselected package esl-erlang.
(Reading database ... 7744 files and directories currently installed.)
Preparing to unpack esl-erlang_20.1.7-1~ubuntu~xenial_amd64.deb ...
Unpacking esl-erlang (1:20.1.7) ...
dpkg: dependency problems prevent configuration of esl-erlang:
 esl-erlang depends on libwxbase2.8-0 | libwxbase3.0-0 | libwxbase3.0-0v5; however:
  Package libwxbase2.8-0 is not installed.
  Package libwxbase3.0-0 is not installed.
  Package libwxbase3.0-0v5 is not installed.
 esl-erlang depends on libwxgtk2.8-0 | libwxgtk3.0-0 | libwxgtk3.0-0v5; however:
  Package libwxgtk2.8-0 is not installed.
  Package libwxgtk3.0-0 is not installed.
  Package libwxgtk3.0-0v5 is not installed.
 esl-erlang depends on libsctp1; however:
  Package libsctp1 is not installed.

dpkg: error processing package esl-erlang (--install):
 dependency problems - leaving unconfigured
Errors were encountered while processing:
 esl-erlang
root@b792ae940e3e:/app# apt-get -f install
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Correcting dependencies... Done
The following additional packages will be installed:
......
......
```

然后再次安装 `esl-erlang_20.1.7-1~ubuntu~xenial_amd64.deb` 包：

```
# dpkg -i esl-erlang_20.1.7-1~ubuntu~xenial_amd64.deb
```

最后，安装RabbitMQ的包：

```
# dpkg -i rabbitmq-server_3.6.14-1_all.deb
```

如果中途再出现缺少依赖包的问题，通过 `apt-get -f install` 来解决。

```
root@b792ae940e3e:/app# dpkg -i rabbitmq-server_3.7.0-1_all.deb 
Selecting previously unselected package rabbitmq-server.
(Reading database ... 29232 files and directories currently installed.)
Preparing to unpack rabbitmq-server_3.7.0-1_all.deb ...
Unpacking rabbitmq-server (3.7.0-1) ...
dpkg: dependency problems prevent configuration of rabbitmq-server:
 rabbitmq-server depends on logrotate; however:
  Package logrotate is not installed.
 rabbitmq-server depends on socat; however:
  Package socat is not installed.

dpkg: error processing package rabbitmq-server (--install):
 dependency problems - leaving unconfigured
Processing triggers for systemd (229-4ubuntu21) ...
Errors were encountered while processing:
 rabbitmq-server
root@b792ae940e3e:/app# apt-get -f install 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Correcting dependencies... Done
The following additional packages will be installed:
  cron libpopt0 libwrap0 logrotate socat tcpd
Suggested packages:
  anacron checksecurity exim4 | postfix | mail-transport-agent mailx
The following NEW packages will be installed:
  cron libpopt0 libwrap0 logrotate socat tcpd
0 upgraded, 6 newly installed, 0 to remove and 2 not upgraded.
1 not fully installed or removed.
Need to get 522 kB of archives.
After this operation, 1674 kB of additional disk space will be used.
Do you want to continue? [Y/n] 
```

* [Downloading and Installing RabbitMQ](https://www.rabbitmq.com/download.html)
* [RabbitMQ安装方式及常用命令 ](https://www.36nu.com/post/240.html) ☆
* [Erlang Downloads](https://packages.erlang-solutions.com/erlang/#tabs-debian)

***

##### Run RabbitMQ Server

启动 RabbitMQ 服务:
```
# service rabbitmq-server start
```

安装 RabbitMQWeb 管理插件：
```
# rabbitmq-plugins enable rabbitmq_management  
# service rabbitmq-server restart
```

打开浏览器登录：http://127.0.0.1:15672，登录账号密码默认都是 `guest`

```
root@730778dc65bd:/app# rabbitmq-plugins enable rabbitmq_management
The following plugins have been enabled:
  mochiweb
  webmachine
  rabbitmq_web_dispatch
  amqp_client
  rabbitmq_management_agent
  rabbitmq_management

Applying plugin configuration to rabbit@730778dc65bd... started 6 plugins.
root@730778dc65bd:/app# service rabbitmq-server restart
 * Restarting RabbitMQ Messaging Server rabbitmq-server                                                      [ OK ] 
root@730778dc65bd:/app#
```

测试代码：

```
## 查看rabbitmq状态
root@730778dc65bd:/app# rabbitmqctl status
Status of node rabbit@730778dc65bd ...
Error: unable to connect to node rabbit@730778dc65bd: nodedown

DIAGNOSTICS
===========

attempted to contact: [rabbit@730778dc65bd]

rabbit@730778dc65bd:
  * connected to epmd (port 4369) on 730778dc65bd
  * epmd reports: node 'rabbit' not running at all
                  no other nodes on 730778dc65bd
  * suggestion: start the node

current node details:
- node name: 'rabbitmq-cli-9223@730778dc65bd'
- home dir: /var/lib/rabbitmq
- cookie hash: MwPrvM8WUeAkWCiIWYw2fg==

root@730778dc65bd:/app# 

root@730778dc65bd:/app# service rabbitmq-server start
 * Starting RabbitMQ Messaging Server rabbitmq-server                                                        [ OK ] 
root@730778dc65bd:/app# rabbitmqctl status
Status of node rabbit@730778dc65bd ...
[{pid,9527},
 {running_applications,[{rabbit,"RabbitMQ","3.5.7"},
                        {mnesia,"MNESIA  CXC 138 12","4.13.3"},
                        {xmerl,"XML parser","1.3.10"},
                        {os_mon,"CPO  CXC 138 46","2.4"},
                        {sasl,"SASL  CXC 138 11","2.7"},
                        {stdlib,"ERTS  CXC 138 10","2.8"},
                        {kernel,"ERTS  CXC 138 10","4.2"}]},
 {os,{unix,linux}},
 {erlang_version,"Erlang/OTP 18 [erts-7.3] [source] [64-bit] [smp:2:2] [async-threads:64] [kernel-poll:true]\n"},
 {memory,[{total,83922208},
          {connection_readers,0},
          {connection_writers,0},
```

****

##### RabbitMQ中的vitrual host

`Virtual host`，是起到隔离作用的。每一个 `vhost` 都有自己的 `exchanges` 和 `queues`，它们互不影响。不同的应用可以跑在相同的 `rabbitmq` 上，使用 `vhost` 把它们隔离开就行。默认情况下，`rabbitmq` 安装后，默认的 `vhost` 是 `/`。


##### 创建用户并设置虚拟主机

可以发现上面我们通过 `guest` 用户在其他电脑上或外网段访问时，会提示 `User can only log in via localhost ` ，这是因为 `guest` 是仅允许在 `localhost` 下才能登陆的。如果我们想在外部访问，可以创建一个新的账户。

创建用户的同时为该用户指定允许访问的虚拟主机 `myvhost`

```
# rabbitmqctl add_user myuser mypassword
# rabbitmqctl add_vhost myvhost
# rabbitmqctl set_permissions -p myvhost myuser ".*" ".*" ".*"
```

此时，新创建的账户 `myuser` 也并没有权限在外网访问，可以用 `set_user_tags` 为用户设置角色：

```
# rabbitmqctl set_user_tags myuser administrator
```

然后我们就能在外网通过地址 `http://192.168.5.107:15673/` 来访问管理端了。

示例：

```
# User: myuser 
# UserPwd: hello
# VHost: hellohost

root@b792ae940e3e:/app# rabbitmqctl add_user myuser hello
Adding user "myuser" ...
root@b792ae940e3e:/app# rabbitmqctl add_vhost hellohost
Adding vhost "hellohost" ...
root@b792ae940e3e:/app# rabbitmqctl set_permissions -p hellohost myuser ".*" ".*" ".*"
Setting permissions for user "myuser" in vhost "hellohost" ...
root@b792ae940e3e:/app# 
```

然后重启服务：

```
# service rabbitmq-server restart
```

***

#### Celery 

Celery官方推荐使用 `RabbitMQ` 或 `Redis` 来作为中间件。设置也很简单，通过 `broker` 和 `backend` 参数即可绑定。


##### broker 和 backend

可以用RabbitMQ和Redis来作为broker或backend：

```
app = Celery('tasks', backend='amqp', broker='amqp://')
```


```
app = Celery('tasks', backend='redis://localhost', broker='amqp://')
```

注意，虽然推荐使用RabbitMQ来作为 `broker`，但不推荐其作为 `backend` 。具体原因我会在后面的文章中说明。

##### 中间人RabbitMQ

RabbitMQ 功能完备、稳定，是一个非常可靠的选择。

```
BROKER_URL =transport://userid:password@hostname:port/virtual_host

BROKER_URL = 'amqp://guest:guest@localhost:5672//'
```

完整的格式为：

```
CELERY_BROKER_URL = 'amqp://[YOUR_NAME]:[PASSWORD]@localhost:[PORT]/[VHOST_NAME]' 
```

##### 中间人Redis

与 `RabbitMQ` 相比，使用 `Redis` 作为 `broker` 缺点是可能因为掉电或异常退出导致数据丢失，优点是使用简单。

以下命令可以同时安装 `celery` 和 `redis` 相关的依赖，但是 `redis server` 还是必须单独安装的。

```
$ pip install -U celery[redis] # -U 的意思是把所有指定的包都升级到最新的版本
```

```
BROKER_URL = 'redis://localhost:6379//'
```

##### 安装celery

先安装 `python3`  `pip3` 等依赖:

```
# apt-get install -y python3 python3-pip
```


```
# pip3 install celery
# 或者：
# pip3 install -U Celery
```

创建一个 `tasks.py` 文件:

```
from celery import Celery

app = Celery('tasks', broker='amqp://guest@localhost//')

@app.task
def add(x, y):
    return x + y
```
注意，其中的：

```
app = Celery('tasks', broker='amqp://guest@localhost//')
```

中 `broker` 要改为上面设置的RabbitMQ的信息，所以结果为：

```
app = Celery('tasks',broker='amqp://myuser:hello@localhost:5672/hellohost')
```

Celery 的第一个参数是当前模块的名称，这个参数是必须的，这样的话名称可以自动生成。
第二个参数是中间人关键字参数，指定你所使用的消息中间人的 URL。

##### 保存结果

执行完成后的结果，Celery 需要在某个地方存储或发送任务处理后的状态，可以通过 `backend` 参数来指定。格式和 `broker` 一致。


完整的 `tasks.py`:

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

from celery import Celery

app = Celery('tasks',
             broker='amqp://myuser:hello@localhost:5672/hellohost',
             backend='amqp://myuser:hello@localhost:5672/hellohost'
             )


@app.task
def add(x, y):
    return x + y

```

##### 运行Celery

用 worker 参数执行程序:

```
$ celery -A tasks worker --loglevel=info
```

输出：

```
root@b792ae940e3e:/app# celery -A tasks worker --loglevel=info
/usr/local/lib/python3.5/dist-packages/celery/platforms.py:795: RuntimeWarning: You're running the worker with superuser privileges: this is
absolutely not recommended!

Please specify a different user using the -u option.

User information: uid=0 euid=0 gid=0 egid=0

  uid=uid, euid=euid, gid=gid, egid=egid,
/usr/local/lib/python3.5/dist-packages/celery/backends/amqp.py:68: CPendingDeprecationWarning: 
    The AMQP result backend is scheduled for deprecation in     version 4.0 and removal in version v5.0.     Please use RPC backend or a persistent backend.

  alternative='Please use RPC backend or a persistent backend.')
 
 -------------- celery@b792ae940e3e v4.1.0 (latentcall)
---- **** ----- 
--- * ***  * -- Linux-4.4.0-42-generic-x86_64-with-Ubuntu-16.04-xenial 2017-12-09 13:50:06
-- * - **** --- 
- ** ---------- [config]
- ** ---------- .> app:         tasks:0x7f97a8360dd8
- ** ---------- .> transport:   amqp://myuser:**@localhost:5672/hellohost
- ** ---------- .> results:     amqp://
- *** --- * --- .> concurrency: 2 (prefork)
-- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
--- ***** ----- 
 -------------- [queues]
                .> celery           exchange=celery(direct) key=celery
                

[tasks]
  . tasks.add

[2017-12-09 13:50:06,121: INFO/MainProcess] Connected to amqp://myuser:**@127.0.0.1:5672/hellohost
[2017-12-09 13:50:06,137: INFO/MainProcess] mingle: searching for neighbors
[2017-12-09 13:50:07,178: INFO/MainProcess] mingle: all alone
[2017-12-09 13:50:07,228: INFO/MainProcess] celery@b792ae940e3e ready.
```

可以看到 `celery` 的 `worker` 已经准备就绪了。


查看 `worker` 完整的命令行参数列表:

```
$  celery worker --help
## 或者：
$ celery help
```

##### 调用任务

使用 `delay()` 方法来调用任务。

新打开一个控制台界面，

```
$ docker exec -it celery1 /bin/bash
```

执行：

```
# python3

>>> from tasks import add
>>> add.delay(4, 4)
```

示例：

```
root@b792ae940e3e:/app# python3
Python 3.5.2 (default, Nov 23 2017, 16:37:01) 
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from tasks import add
>>> add.delay(3,4)
<AsyncResult: e1ae8ea3-8a8f-47c5-befb-e6ba975f0580>
>>> 
```

执行结果：

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171211/cc9H40dDJ5.jpg?imageslim)

同时，也可以在 `RabbitMQ web` 管理页面看到新增了一个任务并存储了处理结果：

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171211/9JEB2m262l.jpg?imageslim)

为了得到调用任务后返回的 `AsyncResult` 实例，通过一个参数来接收：

```
>>> result=add.delay(3,4)
```

`ready()` 方法查看任务是否完成处理:

```
>>> result.ready()
True #结果返回 `True` 表示任务处理完成
```

这里是异步调用，如果我们需要返回的结果，那么要等 `ready` 状态为 `True` 才行。

执行结果：

```
[2017-12-09 13:50:07,228: INFO/MainProcess] celery@b792ae940e3e ready.
[2017-12-09 14:00:33,132: INFO/MainProcess] Received task: tasks.add[e1ae8ea3-8a8f-47c5-befb-e6ba975f0580]  
[2017-12-09 14:00:33,163: INFO/ForkPoolWorker-1] Task tasks.add[e1ae8ea3-8a8f-47c5-befb-e6ba975f0580] succeeded in 0.02956800399988424s: 7
[2017-12-09 14:17:21,033: INFO/MainProcess] Received task: tasks.add[c178619e-3af3-41ed-8d2c-6371de80a601]  
[2017-12-09 14:17:21,058: INFO/ForkPoolWorker-1] Task tasks.add[c178619e-3af3-41ed-8d2c-6371de80a601] succeeded in 0.024445844999718247s: 7
```

#### 相关参考

* [Celery 初步](http://docs.jinkan.org/docs/celery/getting-started/first-steps-with-celery.html)
