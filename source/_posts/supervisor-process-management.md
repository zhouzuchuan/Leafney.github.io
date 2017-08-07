---
title: Linux下进程管理Supervisor
date: 2016-10-04 10:35:15
tags:
	- Supervisor
description: 在Linux系统下管理后台进程，使用Supervisor更省心，更省力
---

#### Supervisor 

> Supervisor,是一个进程控制系统，是一个客户端/服务器端系统允许用户在UNIX-LIKE 操作系统中去监控，控制一些进程。Supervisor作为主进程，Supervisor下管理的时一些子进程，当某一个子进程异常退出时，Supervisor会立马对此做处理，通常会守护进程，重启该进程。

`Supervisor` 有两个主要的组成部分：

* `supervisord`，运行 `Supervisor` 时会启动一个进程 `supervisord`，它负责启动所管理的进程，并将所管理的进程作为自己的子进程来启动，而且可以在所管理的进程出现崩溃时自动重启。
* `supervisorctl`，是命令行管理工具，可以用来执行 `stop`、`start`、`restart` 等命令，来对这些子进程进行管理。

*** 

#### `Supervisor` 安装

通常除了通过源码的 `setup.py` 的方式来安装(详见官网文档 [Supervisor: A Process Control System &mdash; Supervisor 3.3.0 documentation](http://supervisord.org/index.html) )，还有以下两种方式：

```
$ sudo apt-get install supervisor
$ sudo pip install supervisor
```

但这两种安装方式是有区别的。

通过 `pip` 的方式安装后不会安装为默认服务，还需要自己将supervisor程序设置为后台服务。而通过 `apt-get` 的方式安装后就默认创建为了后台服务，可以直接通过 `service supervisor restart` 的方式来管理。



可见：[python - supervisor.conf default location - Stack Overflow](http://stackoverflow.com/questions/12226113/supervisor-conf-default-location)

***

#### 配置文件

supervisor的配置文件通常命名为 `supervisord.conf`。

配置文件检测顺序如下(默认会使用找到的第一个):

* `$CWD/supervisord.conf`
* `$CWD/etc/supervisord.conf`
* `/etc/supervisord.conf`
* `/etc/supervisor/supervisord.conf (since Supervisor 3.3.0)`
* `../etc/supervisord.conf (Relative to the executable)`
* `../supervisord.conf (Relative to the executable)`

*** 

#### 通过 `apt-get install supervisor` 方式安装

执行 ：

```
$ sudo apt-get install supervisor
```

安装完成后会默认将 `supervisord` 启动为后台服务：

```
$ ps -ef|grep supervisor
root      1455     1  0 15:46 ?        00:00:00 /usr/bin/python /usr/bin/supervisord -c /etc/supervisor/supervisord.conf
tiger     1470  1298  0 15:46 pts/1    00:00:00 grep --color=auto supervisor
```

通过 `apt-get` 安装的 `supervisord` 程序位于 `/usr/bin/` 目录下：

```
$ sudo whereis supervisord
[sudo] password for tiger:
supervisord: /usr/bin/supervisord
```

安装完成后查看 `ls /etc/supervisor/` :

```
$ ls /etc/supervisor/
conf.d  supervisord.conf
```

我们看到会生成一个默认的 `supervisord.conf` 配置文件，也可以在 `conf.d` 目录下创建自己的配置文件。


查看文件 `supervisord.conf` 内容：

```
; supervisor config file

[unix_http_server]
file=/var/run/supervisor.sock   ; (the path to the socket file)
chmod=0700                       ; sockef file mode (default 0700)

[supervisord]
logfile=/var/log/supervisor/supervisord.log ; (main log file;default $CWD/supervisord.log)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
childlogdir=/var/log/supervisor            ; ('AUTO' child log dir, default $TEMP)

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///var/run/supervisor.sock ; use a unix:// URL  for a unix socket

; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.

[include]
files = /etc/supervisor/conf.d/*.conf

```

该默认配置文件中是包含着 `conf.d` 目录下的所有 `*.conf` 文件的。我们可以对于不同的项目，使用各自独立的配置文件，放置在 `/etc/supervisor/conf.d` 目录下。

*** 

我们在 `conf.d` 目录下使用 `echo_supervisord_conf` 命令来创建一个 `hello.conf` 配置文件(或者直接通过 `vim` 创建一个空的 `.conf` 文件也可)：

```
$ tiger@localhost:/etc/supervisor/conf.d$ echo_supervisord_conf > hello.conf
-bash: Hello.conf: 权限不够
```

如果提示没有权限的问题，可以使用下面的命令：

```
$ sudo su - root -c "echo_supervisord_conf > /etc/supervisor/conf.d/hello.conf"
```

查看生成的 `hello.conf` :

```
;[program:theprogramname]
;command=/bin/cat              ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
;directory=/tmp                ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=999                  ; the relative start priority (default 999)
;autostart=true                ; start at supervisord start (default: true)
;autorestart=unexpected        ; whether/when to restart (default: unexpected)
;startsecs=1                   ; number of secs prog must stay running (def. 1)
;startretries=3                ; max # of serial start failures (default 3)
;exitcodes=0,2                 ; 'expected' exit codes for process (default 0,2)
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;stopasgroup=false             ; send stop signal to the UNIX process group (default false)
;killasgroup=false             ; SIGKILL the UNIX process group (def false)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=true          ; redirect proc stderr to stdout (default false)
;stdout_logfile=/a/path        ; stdout log path, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
;stdout_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
;stderr_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A=1,B=2           ; process environment additions (def no adds)
;serverurl=AUTO                ; override serverurl computation (childutils)

```

我们需要管理的程序只要依照上面的说明进行配置即可：

假如在目录 `/home/tiger/py` 下有一个需要在后台运行的 Python 程序 `hello.py` 。日志文件保存在 `/home/tiger/py/logs` 下。

```
# coding:utf-8

import time

def make_log():
	while True:
		time.sleep(2)
		with open('hello.log','a') as f:
			f.write('hello_'+time.strftime("%H:%M:%S")+'\r')

if __name__ == '__main__':
	make_log()
```

我们向上面创建的配置文件 `/etc/supervisor/conf.d/hello.conf` 中写入如下数据(可把已有的数据清空，分号 `;` 开头的表示注释信息)：

```
[program:hello]  ;服务的名称
command=python hello.py     				; supervisor启动命令
directory=/home/tiger/py 			        ; 项目的文件夹路径
user=tiger  								; 进程执行的用户身份
autostart=true                           	; 是否自动启动
autorestart=true                         	; 是否自动重启
startsecs=1  								; 自动重启间隔
;log日志文件的位置
stdout_logfile=/home/tiger/py/logs/hellopy.log     ; log 日志
stderr_logfile=/home/tiger/py/logs/hellopy.err     ; 错误日志
```

使文件具有可写权限:

```
$ sudo chmod 777 /etc/supervisor/conf.d/hello.conf
```

通过如下命令启动 supervisor：

```
$ sudo supervisord -c /etc/supervisor/supervisord.conf
```

通过 `apt-get` 安装的 `supervisor` 在安装完成后已经作为了后台服务启动了，修改了配置文件后只需要重新加载即可生效。

重新加载配置文件使用命令：

```
$ sudo supervisorctl reload
```

其他操作命令：

```
$ sudo supervisorctl -c /etc/supervisor/supervisord.conf status  查看管理进程状态
$ sudo supervisorctl -c /etc/supervisor/supervisord.conf reload  重新载入配置项
$ sudo supervisorctl -c /etc/supervisor/supervisord.conf start [all]|[appname]     启动指定/所有程序进程
$ sudo supervisorctl -c /etc/supervisor/supervisord.conf stop [all]|[appname]      关闭指定/所有程序进程
```

注意：

`supervisor` 默认情况下如果不指定要执行的配置文件路径会按照默认的顺序去查询相应的配置文件，按找到的第一个为准。
所以，执行以上代码时，精简的代码为：

```
$ sudo supervisorctl status
```

完整的代码为：

```
$ sudo supervisorctl -c /etc/supervisor/supervisord.conf status
```

`-c` 参数指定使用的配置文件目录 更多参数请通过 `supervisorctl -h/--help` 查看。

*** 

~~测试时发现在 `Ubuntu 16.04` 系统下通过 `apt-get` 方式安装的 `Supervisor 3.3.0` 之前版本(eg:3.2.0.x) 默认不会注册为后台服务，3.3.0.x后的版本会默认注册为后台服务。~~  待考证


***

#### 通过 `pip install supervisor` 的方式安装

通过 `sudo pip install supervisor` 安装完成后不会在 `/etc/` 下生成 `supervisor` 目录及其下的文件。应用程序是存在于 `/usr/local/bin/` 目录中:

```
$ ls /usr/local/bin/
echo_supervisord_conf  pidproxy  supervisorctl  supervisord
```

可见安装时的脚本如下：

```
 ......
 ......
	Installing /usr/local/lib/python2.7/dist-packages/supervisor-3.3.0-nspkg.pth
    Installing echo_supervisord_conf script to /usr/local/bin
    Installing pidproxy script to /usr/local/bin
    Installing supervisorctl script to /usr/local/bin
    Installing supervisord script to /usr/local/bin
```

可见通过 `pip` 安装的 `supervisord` 程序位于 `/usr/local/bin/` 目录下(或者通过如下命令查找)。

```
$ sudo whereis supervisord
[sudo] password for tiger:
supervisord: /usr/local/bin/supervisord
```

*** 

##### 配置文件

通常默认配置文件位于 `/etc/supervisord.conf`。

通过以下命令来创建默认配置文件：

```
$ echo_supervisord_conf > /etc/supervisord.conf
```

如果出现没有权限的问题，可以使用这条命令:

```
$ sudo su - root -c "echo_supervisord_conf > /etc/supervisord.conf"
```

*** 

默认的配置文件是下面这样的，但是这里有个坑需要注意：`supervisord.pid` 以及 `supervisor.sock` 是放在 `/tmp` 目录下，但是 `/tmp` 目录是存放临时文件，里面的文件会被linux系统删除的，一旦这些文件丢失，就无法再通过 `supervisorctl` 来执行 `restart` 和 `stop` 命令了，将只会得到 `unix:///tmp/supervisor.sock` 不存在的错误。(通过 `apt-get` 方式安装的配置文件中不是 `/tmp` 而是`/var/run`)


```
[unix_http_server]
file=/tmp/supervisor.sock   ; (the path to the socket file)
;chmod=0700                 ; socket file mode (default 0700)
;chown=nobody:nogroup       ; socket file uid:gid owner
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

;[inet_http_server]         ; inet (TCP) server disabled by default
;port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for all iface)
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

[supervisord]
logfile=/tmp/supervisord.log ; (main log file;default $CWD/supervisord.log)
logfile_maxbytes=50MB        ; (max main logfile bytes b4 rotation;default 50MB)
logfile_backups=10           ; (num of main logfile rotation backups;default 10)
loglevel=info                ; (log level;default info; others: debug,warn,trace)
pidfile=/tmp/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
nodaemon=false               ; (start in foreground if true;default false)
minfds=1024                  ; (min. avail startup file descriptors;default 1024)
minprocs=200                 ; (min. avail process descriptors;default 200)
;umask=022                   ; (process file creation umask;default 022)
;user=chrism                 ; (default is current user, required if root)
;identifier=supervisor       ; (supervisord identifier, default is 'supervisor')
;directory=/tmp              ; (default is not to cd during start)
;nocleanup=true              ; (don't clean up tempfiles at start;default false)
;childlogdir=/tmp            ; ('AUTO' child log dir, default $TEMP)
;environment=KEY="value"     ; (key value pairs to add to environment)
;strip_ansi=false            ; (strip ansi escape codes in logs; def. false)

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ; use a unix:// URL  for a unix socket
;serverurl=http://127.0.0.1:9001 ; use an http:// url to specify an inet socket
;username=chris              ; should be same as http_username if set
;password=123                ; should be same as http_password if set
;prompt=mysupervisor         ; cmd line prompt (default "supervisor")
...
...
```

将默认配置文件中的以下项进行修改：

```
;file=/tmp/supervisor.sock
;修改为 `/var/run` 目录
file=/var/run/supervisor.sock

;logfile=/tmp/supervisord.log
;修改为 `/var/log` 目录
logfile=/var/log/supervisord.log

;pidfile=/tmp/supervisord.pid
;修改为 `/var/run` 目录
pidfile=/var/run/supervisord.pid

;serverurl=unix:///tmp/supervisor.sock
;修改为 `/var/run` 目录
serverurl=unix:///var/run/supervisor.sock
```

然后在配置文件中添加启动的程序配置项：

```
[program:hello]   							;服务的名称
command=python hello.py     				; supervisor启动命令
directory=/home/tiger/py 			        ; 项目的文件夹路径
user=tiger  								; 进程执行的用户身份
autostart=true                           	; 是否自动启动
autorestart=true                         	; 是否自动重启
startsecs=1  								; 自动重启间隔
;log日志文件的位置
stdout_logfile=/home/tiger/py/logs/hellopy.log     ; log 日志
stderr_logfile=/home/tiger/py/logs/hellopy.err     ; 错误日志
```

*** 

##### 启动 supervisord

执行 `supervisord` 命令，将会启动 `supervisord` 进程，同时我们在配置文件中设置的进程也会相应启动。

使用如下命令来启动:

```
$ sudo supervisord -c /etc/supervisord.conf
```

示例操作命令：

```
tiger@localhost:~/py$ ps -ef | grep supervisor

tiger@localhost:~/py$ sudo supervisord -c /etc/supervisord.conf

tiger@localhost:~/py$ ps -ef | grep supervisor
root      7363     1  0 11:42 ?        00:00:00 /usr/bin/python /usr/local/bin/supervisord -c /etc/supervisord.conf

tiger@localhost:~/py$ ps -ef | grep hello
tiger     7364  7363  0 11:42 ?        00:00:00 python hello.py

```

我们使用 `supervisord` 来启动管理进程，之后所有的操作都用 `supervisorctl` 来控制。 (default /etc/supervisord.conf)

##### supervisorctl 命令介绍

```
# appname 为 [program:x] 里的 x
$ sudo supervisorctl start [appname]|[all]      启动指定/所有程序进程
$ sudo supervisorctl stop [appname]|[all]       停止指定/所有程序进程
$ sudo supervisorctl status  					查看管理所有进程状态
$ sudo supervisorctl status [appname]  			查看管理指定进程状态
$ sudo supervisorctl reload  	载入最新的配置文件，停止原有进程并按新的配置启动、管理所有进程
$ sudo supervisorctl update  	根据最新的配置文件，启动新配置或有改动的进程，配置没有改动的进程不会受影响而重启
$ sudo supervisorctl restart [appname]  		重启某个进程
$ sudo supervisorctl shutdown  	关闭所有管理进程

$ sudo supervisorctl start/stop/restart/status groupworker:    		管理所有属于名为 groupworker 这个分组的进程
$ sudo supervisorctl start/stop/restart/status groupworker:name1    管理分组里指定的进程

```

~~注意：显示用 `stop` 停止掉的进程，用 `reload` 或者 `update` 都不会自动重启。~~  该问题待考证

示例操作命令：

```
tiger@localhost:~/py$ sudo supervisorctl status
hello                            RUNNING   pid 7382, uptime 0:01:50
tiger@localhost:~/py$ sudo supervisorctl stop hello
hello: stopped
tiger@localhost:~/py$ sudo supervisorctl status
hello                            STOPPED   Jul 27 11:47 AM

```

*** 

##### 开机自动启动 Supervisord

通过 `pip` 安装的 `Supervisord` 默认情况下并没有被安装成服务，它本身也是一个进程。我们可以使用安装脚本将supervisord设置为服务。

```
# 下载脚本 (需要root权限)
$ sudo su - root -c "sudo curl https://gist.githubusercontent.com/howthebodyworks/176149/raw/d60b505a585dda836fadecca8f6b03884153196b/supervisord.sh > /etc/init.d/supervisord"
# 设置该脚本为可以执行
$ sudo chmod +x /etc/init.d/supervisord
# 设置为开机自动运行
% sudo update-rc.d supervisord defaults
# 试一下，是否工作正常
$ sudo service supervisord stop
$ sudo service supervisord start
# 查看supervisord进程
$ sudo ps -ef| grep supervisor
```

注意：下载了 `supervisord.sh` 文件后，请核对好里面的配置参数和本地文件所在目录是否一致(主要是以下部分)：

```
	...
# PATH should only include /usr/* if it runs after the mountnfs.sh script
PATH=/sbin:/usr/sbin:/bin:/usr/bin
DESC="Description of the service"
NAME=supervisord
DAEMON=/usr/local/bin/supervisord
DAEMON_ARGS=""
PIDFILE=/var/run/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
	...
```

详细安装方法及脚本文件见下面两个链接说明：

* [python - How to automatically start supervisord on Linux (Ubuntu) - Server Fault](http://serverfault.com/questions/96499/how-to-automatically-start-supervisord-on-linux-ubuntu)
* [an init.d script for supervisord · GitHub](https://gist.github.com/howthebodyworks/176149)

还有第二种方法将supervisor随系统启动而启动，Linux 在启动的时候会执行 `/etc/rc.local` 里面的脚本，所以只要在这里添加执行命令即可：

```
# 如果是 Ubuntu 添加以下内容（这里要写全路径，因为此时PATH的环境变量未必设置）
/usr/local/bin/supervisord -c /etc/supervisord.conf

# 如果是 Centos 添加以下内容
/usr/bin/supervisord -c /etc/supervisord.conf
```

*** 

##### 测试 supervisor 管理的进程能自动重启

将 `supervisor` 管理的进程用 `kill` 命令杀掉，看是否能够自动重启。

```
tiger@localhost:~/py$ ps -ef | grep hello
tiger     7364  7363  0 11:42 ?        00:00:00 python hello.py
tiger     7368  1255  0 11:42 pts/0    00:00:00 grep --color=auto hello
tiger@localhost:~/py$ kill 7364
tiger@localhost:~/py$ ps -ef | grep hello
tiger     7382  7363  2 11:45 ?        00:00:00 python hello.py
tiger     7384  1255  0 11:45 pts/0    00:00:00 grep --color=auto hello

```

***

##### 使用 include

在配置文件的最后，有一个 `[include]` 的配置项。我们可以 `include` 某个文件夹下的所有配置文件，这样就能为每个进程或相关的几个进程的配置单独写成一个文件。

配置文件的后缀名可以为 `.conf` 或 `.ini`。

在 `/etc/supervisord.conf` 末尾添加如下：

```
[include]
files = /etc/supervisord.d/*.conf
```

我们在 `/home/tiger/py` 目录下再建立一个 `world.py` 的python程序来做测试。

然后在 `/etc` 目录下创建 `supervisord.d` 目录，添加一个 `world.conf` 配置文件，写入如下配置信息。

```
[program:world]  ;服务的名称，后面操作会用到
command=python world.py     				; supervisor启动命令
directory=/home/tiger/py 			        ; 项目的文件夹路径
user=tiger  								; 进程执行的用户身份
autostart=true                           	; 是否自动启动
autorestart=true                         	; 是否自动重启
startsecs=1  								; 自动重启间隔
;log日志文件的位置
stdout_logfile=/home/tiger/py/logs/worldpy.log     ; log 日志
stderr_logfile=/home/tiger/py/logs/worldpy.err     ; 错误日志
```

然后重新加载配置文件：

```
$ sudo supervisorctl reload
```

查看运行状态：

```
$ sudo supervisorctl status
hello                            RUNNING   pid 1549, uptime 0:00:28
world                            RUNNING   pid 1548, uptime 0:00:28
$ sudo supervisorctl stop hello
hello: stopped
$ sudo supervisorctl status
hello                            STOPPED   Jul 27 04:53 PM
world                            RUNNING   pid 1573, uptime 0:00:21
```

这样我们就通过不同的配置文件来管理不同的程序进程了。

***

#### Supervisor UI 管理台：

在默认配置文件中我们可以找到下面的配置项：

```
;[inet_http_server]         ; inet (TCP) server disabled by default
;port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for all iface)
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))
```

去除 `[inet_http_server]` 和 `port=127.0.0.1:9001` 前面的分号，然后执行 `sudo supervisorctl reload` 重新加载配置文件。之后在浏览器中访问 `http://localhost:9001` 就能查看到Web版的进程管理界面了。

注意：如果设置为 `port=127.0.0.1:9001`，则只能在本机访问；如果为 `port=*:9001` 则可以在外网进行访问。

注意：`;[inet_http_server]` 这个前面的分号必须去掉，要不然不管用。

***

#### Supervisor 集群管理

集中进程管理，可在一台机器下管理多台机器的进程。

详见：Supervisor集群管理开发 文档


* [XML-RPC API Documentation &mdash; Supervisor 3.3.0 documentation](http://supervisord.org/api.html)
* [supervisord 的 XML-RPC API 使用说明 - yexiaoxiaobai - SegmentFault](https://segmentfault.com/a/1190000000606682)
* [Supervisor集群管理WEB UI (monitor) - WisZhou的想到啥写啥        - 博客频道 - CSDN.NET](http://blog.csdn.net/u013411478/article/details/25387587)
* [GitHub - WisZhou/supervisord-monitor: Supervisord Monitoring Tool](https://github.com/WisZhou/supervisord-monitor)
* [GitHub - luxbet/supervisorui: Supervisor multi-server dashboard](https://github.com/luxbet/supervisorui)

*** 

#### `unix:///var/run/supervisor.sock no such file`

某些情况下，可能会出现如下错误：`unix:///var/run/supervisor.sock no such file` 

对于该问题，我的操作是，执行命令：

```
$ sudo supervisord
```

* [unix:///var/run/supervisor.sock no such file #480](https://github.com/Supervisor/supervisor/issues/480)

***

#### 注意：被监控的进程要以非daemon方式运行

该问题暂未研究。略。

***

#### supervisor深入研究之 **多进程**

略。

***

#### supervisor深入研究之 **group** 分组管理 

略。

***

#### supervisor调用 virtualenv 环境中的python项目

配置 .conf 文件时，加上 `environment` 参数，指定 virtualenv 的目录:

```
command = /home/www/flasky/env/bin/gunicorn -w 2 -b 0.0.0.0:8080 --max-requests 2000 --log-level debug --name %(program_name)s "app:create_app('development')"
directory = /home/www/flasky
environment=PATH="/home/www/flasky/env/bin", GEVENT_RESOLVER="ares"
user = root
numprocs=1
autostart=false
autorestart=true
...
...
```

*** 


#### 相关链接

* [Python 进程管理工具 Supervisor 使用教程 - restran - 博客园](http://www.cnblogs.com/restran/p/4854623.html)
* [Python 进程管理工具 Supervisor 使用教程 | 淡水网志](https://www.restran.net/2015/10/04/supervisord-tutorial/)

* [python - How to automatically start supervisord on Linux (Ubuntu) - Server Fault](http://serverfault.com/questions/96499/how-to-automatically-start-supervisord-on-linux-ubuntu)

* [ubuntu下supervisor安装与使用笔记 - 为程序员服务](http://ju.outofmemory.cn/entry/208687)
* [使用 supervisor 管理进程 - 李林克斯](http://liyangliang.me/posts/2015/06/using-supervisor/)
* [使用Supervisor简化进程管理工作 - EverET.org](http://everet.org/supervisor.html)

* [python - supervisor.conf default location - Stack Overflow](http://stackoverflow.com/questions/12226113/supervisor-conf-default-location) supervisor的配置文件查询目录 
* [linux 后台进程管理利器supervisor - youxin - 博客园](http://www.cnblogs.com/youxin/p/4147384.html)
* [Supervisor使用教程 | Snow Memory](http://andrewliu.in/2016/02/19/Supervisor%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/) **☆**
* [Linux进程管理工具supervisor安装及使用 | cpper](http://cpper.info/2016/04/14/supervisor-usage.html) **☆**
* [Supervisor: A Process Control System &mdash; Supervisor 3.3.0 documentation](http://supervisord.org/index.html) 官方文档

* [How To Install and Manage Supervisor on Ubuntu and Debian VPS | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-manage-supervisor-on-ubuntu-and-debian-vps)
* [supervisor初体验 - 简书](http://www.jianshu.com/p/9abffc905645)

* [使用 supervisor 管理进程 - 李林克斯](http://liyangliang.me/posts/2015/06/using-supervisor/)  排版好看
* [supervisord 部署 Flask &#8211; Angiris Council](https://liuliqiang.info/deploy-flask-gunicorn-by-supervisord/) 好

* [Supervisor手册  - GitBook](https://www.gitbook.com/book/wohugb/supervisor/details)
* [supervisord 部署 Flask](https://liuliqiang.info/deploy-flask-gunicorn-by-supervisord/)





* [virtualenv wrapper](https://liuliqiang.info/post/125/)

