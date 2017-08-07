---
title: 重新安装Ubuntu系统后的必要设置
date: 2016-10-01 10:15:30
tags:
	- Ubuntu
description: 重新安装Ubuntu系统后的一些必要设置：1. 升级软件库及系统内核；2. 新增账户及权限设置；3. 修改系统时区；4. SSH能连接而SFTP不能连接；
---

以下为我在重装Linux系统后进行的一些必要的操作，特此记录。

#### ubuntu 14.04 升级到 16.04 

如果直接执行 `# do-release-upgrade` 升级命令，会遇到：`需要的依赖关系未安装` 的报错信息：

```
The required dependency 'apt (>= 1.0.1ubuntu2.13)' is not installed. 
```

我们需要先更新 `apt` 到 `1.0.1ubuntu2.13` 以上才能进行升级操作。

通过以下操作来更新：

1. 保持软件源指向 14.04(trusty) 不变
2. `sudo apt-get update && sudo apt-get upgrade`

此时 `apt` 应已升级到 `1.0.1ubuntu2.13`，可以继续 `do-release-upgrade` 了。

总结需要执行的命令如下：

```
# sudo apt-get update
# sudo apt-get upgrade
# sudo do-release-upgrade
```

操作完成后，重启系统即可：`# sudo reboot` 。

****

#### 更新软件包及升级系统到最新的内核

##### 更新软件包

刚装完linux系统后，软件及内核版本都比较低，需要先更新一下：

刚装完后系统版本为： `Ubuntu 16.04.1 LTS`

因为我的服务器在香港，所以能连接到ubuntu官方的软件源，如果你的系统是在国内，可能需要更新一下软件源，以免下载太慢或更新失败，设置软件源：

```
# echo "deb http://cn.archive.ubuntu.com/ubuntu/ trusty main restricted universe multiverse" >> /etc/apt/sources.list
```

其他系统可参考 ：[Ubuntu 源列表](http://wiki.ubuntu.org.cn/%E6%BA%90%E5%88%97%E8%A1%A8)

执行如下命令进行更新：

```
# sudo apt-get update
# sudo apt-get upgrade
```

更新后的系统显示：

```
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 3.16.0-30-generic x86_64)
```

***

##### 升级系统内核

升级内核版本，执行如下命令：

```
# sudo apt-get install linux-generic-lts-xenial linux-image-generic-lts-xenial
```

安装完成后重启：

```
# sudo reboot
```

登陆后我们可以看到，内核已经更新了：

```
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-38-generic x86_64)
```

***

#### Ubuntu新增普通管理员账户并设置管理员权限

在Linux系统下，`$` 是普通管理员命令标识，`#` 是系统管理员命令标识

##### 更改已有用户账户密码

我们可以使用 `passwd` 命令来更改账户的命令，执行后输入两次新密码即可。例如为 `root` 账户修改密码：

```
$ sudo passwd root
```

##### 新增用户账户

可以通过 `adduser` 命令来新增账户：

```
$ sudo adduser username
```

同样的需要输入两次密码，其他的设置项直接按回车即可。

新创建的用户只有普通用户权限，如果想要安装软件或更新软件包，还需要赋予账户管理员权限才行。

***

##### 使新增用户具有 `root` 权限--命令法(不推荐)

在网上查看为新增用户账户可以通过如下的命令来添加管理员权限，但我在实际操作后并没有生效，所以这里暂不推荐该方法。

```
sudo usermod -G root username 
```

要添加新用户到 `sudo`，最简单的方式就是使用 `usermod` 命令。运行 :

```
$sudo usermod -G root username 
```

然而，如果用户已经是其他组的成员，你需要添加 `-a` 这个选项，象这样：

``` 
$sudo usermod -a -G root username 
```

这里暂作记录，待以后找到为何无效的原因再来修改。

***

##### 使新增用户具有 `root` 权限--修改文件法(推荐)

使用新创建的用户账户登陆。这里我新建的用户是 `tiger` 账户。执行更新命令：

```
tiger@MyServer:~$ apt-get update
E: Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)
E: Unable to lock directory /var/lib/apt/lists/
E: Could not open lock file /var/lib/dpkg/lock - open (13: Permission denied)
E: Unable to lock the administration directory (/var/lib/dpkg/), are you root?
tiger@MyServer:~$ sudo apt-get update
[sudo] password for tiger: 
tiger is not in the sudoers file.  This incident will be reported.
tiger@MyServer:~$ 
```

报错信息：`tiger is not in the sudoers file.  This incident will be reported.` 表名我们新建的账户 `tiger` 在 `sudoers` 文件中并没有指定权限，所以也就无法以管理员权限执行命令。

执行下面的操作：

先切换回 `root` 账户下(注意 `su -` 后面的 `-` 和后面的账户名之间有一个空格)：

```
tiger@MyServer:~$ su - root
```

为文件 `/etc/sudoers` 添加写权限，默认情况下该文件为只读属性。执行命令：`$ chmod u+w /etc/sudoers` 。

然后编辑该文件，找到这一行 :  

```
root  ALL=(ALL:ALL)  ALL

```

在下面按照同样的格式添加:  

```
xxx  ALL=(ALL:ALL)  ALL

```

(这里的xxx是你要设置的用户名)，然后保存退出。

撤销文件的写权限，执行命令：`$ chmod u-w /etc/sudoers` 。

完整的命令如下：

```
root@MyCloudServer:/home/tiger# chmod u+w /etc/sudoers
root@MyCloudServer:/home/tiger# vim /etc/sudoers
root@MyCloudServer:/home/tiger# ls -al /etc/sudoers 
-rw-r----- 1 root root 770 Sep 28 02:15 /etc/sudoers
root@MyCloudServer:/home/tiger# chmod u-w /etc/sudoers
root@MyCloudServer:/home/tiger# su - tiger
tiger@MyCloudServer:~$ apt-get update

E: Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)
E: Unable to lock directory /var/lib/apt/lists/
E: Could not open lock file /var/lib/dpkg/lock - open (13: Permission denied)
E: Unable to lock the administration directory (/var/lib/dpkg/), are you root?
tiger@MyCloudServer:~$ sudo apt-get update

Get:1 http://security.ubuntu.com trusty-security InRelease [65.9 kB]
Ign http://us.archive.ubuntu.com trusty InRelease          
Get:2 http://us.archive.ubuntu.com trusty-updates InRelease [65.9 kB]
Get:3 http://security.ubuntu.com trusty-security/main Sources [120 kB] 
Get:4 http://us.archive.ubuntu.com trusty-backports InRelease [65.9 kB]        
Get:5 http://security.ubuntu.com trusty-security/restricted Sources [4,064 B]  
Hit http://us.archive.ubuntu.com trusty Release.gpg        
Get:6 http://security.ubuntu.com trusty-security/universe Sources [42.5 kB]
Get:7 http://us.archive.ubuntu.com trusty-updates/main Sources [383 kB]
Get:8 http://security.ubuntu.com trusty-security/multiverse Sources [2,749 B]
...
...

```

这样，我们就为新创建的账户 `tiger` 设置了管理员权限，执行命令时就可以通过 `sudo` 来提权了。

参考：

* [xx is not in the sudoers file 问题解决【转载】 - evasnowind - 博客园](http://www.cnblogs.com/evasnowind/archive/2011/02/04/1949113.html)
* [Ubuntu技巧之 is not in the sudoers file解决方法_Linux教程_Linux公社-Linux系统门户网站](http://www.linuxidc.com/Linux/2010-12/30386.htm)


***

#### Ubuntu sudo Error:unable to resolve host

为新增用户设置了管理员权限后，每次执行 `$ sudo xxxx` 命令时都会弹出下面一条信息：

```
Ubuntu sudo Error:unable to resolve host
```

这种情况是 系统的主机名和配置文件中的主机名不一致造成的。

编辑 `/etc/hostname` 更改主机名(hostname)
主机名是在命令行中 `tiger@MyServer:~$` @符合后面的。

编辑 `/etc/hosts` 文件中：

```
127.0.0.1	localhost

127.0.1.1	ubuntu
```

更改 `ubuntu` 为你的 `hostname` 的名称。

然后重启系统。`sudo reboot` 。必需重启之后才有效。

* [ubuntu 下修复使用sudo命令后出现主机名字不能解析的错误：Fix Ubuntu sudo Error:unable to resolve host - wzb56的资料库        - 博客频道 - CSDN.NET](http://blog.csdn.net/wzb56_earl/article/details/6289988)

***

#### Ubuntu 14.04 修改时区

在系统下通过 `date` 命令查看时间，可能会与本地的之间不一致。

比如 我当前系统的时间为 `2016-10-5 15:53:54` 而 服务器的时间为 `Wed Oct  5 03:58:37 EDT 2016` 。按理说，Linux系统在重启后会自动同步时间的，而这种情况可能是时区不一致的原因。

执行下面命令，并按照提示选择 “Asia/Shanghai”：

```
# sudo dpkg-reconfigure tzdata
```

选择完成后，会输出如下结果：

```
Current default time zone: 'Asia/Shanghai'
Local time is now:      Wed Oct  5 16:01:12 CST 2016.
Universal Time is now:  Wed Oct  5 08:01:12 UTC 2016.
```

再次查看时间：

```
tiger@MyServer:~$ date
Wed Oct  5 16:02:13 CST 2016
```

* [Ubuntu 14.04 修改时区 - MyPy的个人页面 - 开源中国社区](https://my.oschina.net/zhangrf/blog/224758)

***

#### Linux 下切换用户 

`su` 和 `su -` 的区别

```
su abc
su - abc
```

注意，`su -` 在 `-` 和账户名后面还有一个空格分隔。




* [（总结）Linux下su与su -命令的本质区别](http://www.ha97.com/4001.html)
* [linux - Why do we use su - and not just su? - Unix &amp; Linux Stack Exchange](http://unix.stackexchange.com/questions/7013/why-do-we-use-su-and-not-just-su)

***

#### Ubuntu下SSH能连接而SFTP不能连接

[ssh能够连接而sftp不能连接的解决方法](http://m.2cto.com/os/201211/169172.html)


用FileZilla一直不能登录远程的服务器，ssh的登录就OK

```
# locate sftp-server
```

locate一下ftp-server，发现目录跟配置文件中的不同

```
# vi /etc/ssh/sshd_config
```

```
# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

#Subsystem sftp /usr/lib/openssh/sftp-server

```

上面的 locate 查看到的列表，我更改成了如下的设置：

```
# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

Subsystem sftp /usr/lib/sftp-server
```

即把它默认成第二个改成了列表中的第一项。

然后重启ssh服务：

```
sudo service ssh restart
```

再次尝试，则能够正常连接了。

***

