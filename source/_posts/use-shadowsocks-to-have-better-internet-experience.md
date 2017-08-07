---
title: 使用Shadowsocks轻松实现科学上网
date: 2016-10-02 10:24:44
tags:
	- Google
	- Shadowsocks
	- 科学上网
description: 在国内要想用Google来上网着实是要费一翻心思的，如果你有一台国外的服务器的话，一切都变得如此简单。
---

由于个人对百度的厌恶，平时上网很少用百度搜索。而又因为众所周知的原因，在国内要想用Google来上网着实是要费一翻心思的。所以也只能退而求其次，用微软的Bing来查询一些资料。

前几天刚刚把我在香港的云服务器进行了系统更换，从Windows Server换成了Linux系统，想起之前曾看过的使用Shadowsocks实现代理上网的文章，所以就想要亲自实现一下。

shadowsocks是一个著名的轻量级socket代理，原始版本是基于Python编写，后来又有了Go语言版本。不过该版本在Github上的源代码由于“你懂得”的原因，已经被开发者删除了。

这里我推荐安装的是 `shadowsocks-libev` 版本。`shadowsocks-libev` 是一个 shadowsocks 协议的轻量级实现，是 shadowsocks-android, shadowsocks-ios 以及 shadowsocks-openwrt 的上游项目。其特点如下：

* 体积小巧，静态编译并打包后只有 100 KB
* 高并发，基于 libev 实现的异步 I/O，以及基于线程池的异步 DNS，同时连接数可上万。
* 基于C语言实现，内存占用小（600k左右），低 CPU 消耗

***

#### shadowsocks-libev 的安装

我的云服务器系统为 `Ubuntu 14.04 TFS`

通常 `shadowsocks-libev` 版本有两种安装方式，从源码安装和通过软件源来安装。这里我推荐使用源码安装的方式。

##### 从源码安装 (Ubuntu/Debian系统下)

###### 安装必须的包

```
$ sudo apt-get udpate

$ mkdir shadowsocks-libev && cd shadowsocks-libev

$ sudo apt-get install build-essential autoconf libtool libssl-dev gawk debhelper dh-systemd init-system-helpers pkg-config asciidoc xmlto apg libpcre3-dev

```

安装过程会需要一些时间。


###### 通过Git下载源码

```
$ git clone https://github.com/shadowsocks/shadowsocks-libev.git
```

然后生成deb包并安装，依照以下步骤依次执行(如果出错请检查系统或者之前的步骤)：

```
$ cd shadowsocks-libev

$ dpkg-buildpackage -b -us -uc -i

$ cd ..

$ sudo dpkg -i shadowsocks-libev*.deb
```

在上面的第三步 `cd ..` 后，可以看到目录下编译生成了三个 `*.deb` 文件，我这里的是：

* `libshadowsocks-libev-dev_2.5.3-1_amd64.deb`
* `libshadowsocks-libev2_2.5.3-1_amd64.deb`
* `shadowsocks-libev_2.5.3-1_amd64.deb`

上面的步骤操作完成后，我们就已经安装成功了 `shadowsocks-libev` 。

通过如下命令来查看运行状态：

```
$ sudo service shadowsocks-libev status
 * shadowsocks-libev is not running
```

通过deb包安装的方式默认会开启自启。

*** 

##### 直接从作者提供的软件源安装（Ubuntu/Debian）

由于作者更新源码后并不一定及时更新这些预编译的包，所以无法保证最新版本，但操作步骤比较简单。

###### 先添加GPG Key

```
$ wget -O- http://shadowsocks.org/debian/1D27208A.gpg | sudo apt-key add -
```

###### 配置安装源，在/etc/apt/sources.list末尾添加

```
# Ubuntu 14.04 or above
$ deb http://shadowsocks.org/ubuntu trusty main

# Debian Wheezy, Ubuntu 12.04 or any distribution with libssl > 1.0.1
$ deb http://shadowsocks.org/debian wheezy main
```

###### 执行安装

```
$ apt-get update
$ apt-get install shadowsocks-libev
```

这里我在 配置 `wget -O- http://shadowsocks.org/debian/1D27208A.gpg | sudo apt-key add -` 时无法连接到 `http://shadowsocks.org` 站点，所以这种方法我就没有继续测试。

***

##### shadowsocks-libev 一键安装

待完善，详见：https://github.com/iMeiji/shadowsocks_install/wiki/shadowsocks-libev-%E4%B8%80%E9%94%AE%E5%AE%89%E8%A3%85

***

#### 配置与启动

##### 配置文件

`shadowsocks-divev` 生成的默认配置文件在目录 `/etc/shadowsocks-libev` 下，找到 `config.json` 文件并编辑：

将配置信息：

```
{
    "server":"127.0.0.1",
    "server_port":8388,
    "local_port":1080,
    "password":"OikIryahoa",
    "timeout":60,
    "method":null
} 
```

修改为如下：

```
{
    "server":"0.0.0.0",
    "server_port":8388,
    "local_port":1080,
    "password":"OikIryahoa",
    "timeout":60,
    "method":"aes-256-cfb"
}
```

其中：

* `server` ：主机域名或者IP地址，尽量填IP (可以为服务器实际的IP地址或 `0.0.0.0` ) 
* `server_port` ：服务器监听端口
* local_port: 客户端连接端口
* `password` ：密码
* `timeout` ：连接超时时间，单位秒。要适中
* `method` ：加密方式 默认为table,其他有rc4,rc4-md5,aes-128-cfb, aes-192-cfb, aes-256-cfb,bf-cfb, camellia-128-cfb, camellia-192-cfb,camellia-256-cfb, cast5-cfb, des-cfb


**注意：** 

* 如果客户端有OpenWRT路由器等设备，推荐 `rc4-md5` ，性能更好；否则可以选用安全性更好的 `aes-256-cfb` 等。
* `server` 配置项表示主机的域名或IP地址，这里默认情况下是 `127.0.0.1` 但不建议设置成 `127.0.0.1` ，测试时发现无法正确连通。你可以设置成 **`0.0.0.0`** 或 **真实的服务器所在的IP地址**。修改配置文件重启后生效。
* 默认的客户端的IP地址为 `127.0.0.1`


***

##### 启动

上面有提到，通过deb包安装后就默认启动了，通过如下命令来控制：  

```
$ sudo service shadowsocks-libev start
$ sudo service shadowsocks-libev stop
$ sudo service shadowsocks-libev status
$ sudo service shadowsocks-libev restart
```

安装后的shadowsocks程序名为 `ss-server` ，程序目录为 `/usr/bin/ss-server` 。

查看 `ss-server` 的启动信息：

```
$ sudo service shadowsocks-libev status
 * shadowsocks-libev is not running

$ ps ax |grep ss-server
40160 ?        Ss     0:00 /usr/bin/ss-server -c /etc/shadowsocks-libev/config.json -a root -u -f /var/run/shadowsocks-libev/shadowsocks-libev.pid -u
40162 ?        S+     0:00 grep --color=auto ss-server

```

注意其中有 `-u`，表示会通过udp的方式进行连接。

通过 `netstat -lnp` 可以查看 `ss-server` 监听的端口：

```
$ netstat -lnp
(No info could be read for "-p": geteuid()=1000 but you should be root.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:8388            0.0.0.0:*               LISTEN      -                          
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -               
tcp6       0      0 :::80                   :::*                    LISTEN      -               
tcp6       0      0 :::22                   :::*                    LISTEN      -                        
udp        0      0 0.0.0.0:8388            0.0.0.0:*                           -       
```

可以看到 `ss-server` 通过 `tcp` 和 `udp` 两种方式监听了 `8388` 端口。

***

#### shadowsocks 客户端的设置

shadowsocks 默认支持多种客户端。可以从 [Shadowsocks - Clients](https://shadowsocks.org/en/download/clients.html) 下载对应平台的客户端软件。

##### windows 客户端

windows用户可以从 [Releases · shadowsocks/shadowsocks-windows · GitHub](https://github.com/shadowsocks/shadowsocks-windows/releases) 下载安装包。解压后得到 `Shadowsocks.exe` 程序。

运行，配置 `服务器地址` `服务器端口` `密码` `加密方式` 即可。按照配置文件中的设置，默认监听客户端所在本地系统 `127.0.0.1` 的 `1080` 端口。

更加详细的内容可以参考如下文章：

* [Shadowsocks Windows 使用说明 · shadowsocks/shadowsocks-windows Wiki · GitHub](https://github.com/shadowsocks/shadowsocks-windows/wiki/Shadowsocks-Windows-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E)

***

##### Chrome+SwitchyOmega实现科学上网

待续。。。

***

#### shadowsocks-libev 的多用户配置

C语言编写的shadowsocks客户端/服务端软件shadowsocks-libev并不像go版本或python版本的shadowsocks客户端/服务端软件那样直接支持多实例配置，具体可以查看如下说明：

* [please support multi-port config.json · Issue #5](https://github.com/shadowsocks/shadowsocks-libev/issues/5)

`shadowsocks-libev` 版本默认不支持在同一个配置文件 `config.json` 中一次设置多个端口和密码，如果想要设置多个，可以通过添加多个配置文件来实现。

##### 方式一

先停止 `ss-server` 服务：

```
$ sudo service shadowsocks-libev status
 * shadowsocks-libev is running
$ sudo service shadowsocks-libev stop
$ sudo service shadowsocks-libev status
 * shadowsocks-libev is not running
```

然后，拷贝一份原来的配置文件，自定义新的文件名，只要保证扩展名为 `.json` 即可，我这里命名为 `configuser1.json` ：

```
$ cd /etc/shadowsocks-libev
$ sudo cp config.json configuser1.json
$ sudo vim configuser1.json
```

修改配置参数中的端口号，密码等：

```
{
    "server":"0.0.0.0",
    "server_port":8398,
    "local_port":1080,
    "password":"OikIrya3oyt",
    "timeout":60,
    "method":"aes-256-cfb"
}
```

然后启动 `ss-server` 服务：

```
$ sudo service shadowsocks-libev start
$ sudo service shadowsocks-libev status
 * shadowsocks-libev is running
```

执行如下命令添加新的配置文件设置 ：

```
$ setsid ss-server -c /etc/shadowsocks-libev/***.json -u
```

注意将其中的 `***` 替换为你的配置文件名称。

***

##### 方式二

如果你嫌上面的“停止-拷贝已有配置文件-重启”操作太麻烦，也可以直接新建一个json配置文件，然后填入如下配置信息：

```
{
    "server":"0.0.0.0",
    "server_port":8398,
    "local_port":1080,
    "password":"OikIrya3oyt",
    "timeout":60,
    "method":"aes-256-cfb"
}
```

注意 `server_port` 要设置成新的端口号。

然后直接执行如下命令即可：

```
$ setsid ss-server -c /etc/shadowsocks-libev/***.json -u
```

查看启动信息：

```
$ ps ax |grep ss-server
40103 ?        Ss     0:00 ss-server -c /etc/shadowsocks-libev/configuser1.json -u
40160 ?        Ss     0:00 /usr/bin/ss-server -c /etc/shadowsocks-libev/config.json -a root -u -f /var/run/shadowsocks-libev/shadowsocks-libev.pid -u
40162 ?        S+     0:00 grep --color=auto ss-server
```

可以看到比之前多了一条后台服务。

通过 `netstat -lnp` 来查看 `ss-server` 是否监听了多个端口：

```
$ netstat -lnp
(No info could be read for "-p": geteuid()=1000 but you should be root.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:8388            0.0.0.0:*               LISTEN      -                          
tcp        0      0 0.0.0.0:8398            0.0.0.0:*               LISTEN      -      
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -               
tcp6       0      0 :::80                   :::*                    LISTEN      -               
tcp6       0      0 :::22                   :::*                    LISTEN      -                        
udp        0      0 0.0.0.0:8388            0.0.0.0:*                           -     
udp        0      0 0.0.0.0:8398            0.0.0.0:*                           -     
```

这样，就实现了监听多个端口，实现多用户连接了。如果想要停止新增的监听端口，只需要重启shadowsocks服务就又恢复默认，只会监听的 `config.json` 中配置的端口了。

***

#### 相关链接

* [shadowsocks libev](https://github.com/iMeiji/shadowsocks_install/wiki/shadowsocks-libev) **☆**
* [shadowsocks client](https://shadowsocks.org/en/download/clients.html)
* [Shadowsocks Windows 使用说明](https://github.com/shadowsocks/shadowsocks-windows/wiki/Shadowsocks-Windows-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E)

