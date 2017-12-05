---
title: 百度人工智能开放平台DuerOS初体验
date: 2017-12-05 15:58:29
tags:
    - DuerOS
    - 智能家居
categories:
    - DuerOS开放平台
description: DuerOS开放平台是为企业及开发者提供的一整套对话式人工智能解决方案的开放平台
---

今年11月初的时候在百度DuerOS开放平台上申请了DuerOS的“开发套件个人版”的开发板，前几天正式收到了该开发板，经过几天的摸索，发现DuerOS开放平台还是有很多可挖掘的功能的。在此将这几天的研究成果记录一下。

双十一的时候在天猫上买了一个阿里的天猫精灵，通过对比小米的小爱同学，阿里的天猫精灵，和百度的DuerOS开发板比起来，个人感觉DuerOS开发板要更贴近于开发者，可以让开发者自己动手去实现想要的智能化的功能。

#### DuerOS唤醒

百度的DuerOS开发套件个人版需要用户自备一个树莓派3B来结合使用。开发板上带有2颗高灵敏度MEMS麦克风，搭载百度DuerOS SDK，可为用户提供百度海量的信息服务能力。

按照官方给的硬件安装文档和提供的镜像系统将设备组合成功，插入刷好系统的内存卡，通电等待系统开机。在初次连接时，可以使用百度提供的 “小度之家app” 将系统接入网络。 小度联网成功后，直接说 “小度小度+内容” 即可实现语音式的对话交互操作。

总体来说，唤醒小度的这一步非常的简单，附上一张设备的合影。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/1HHmf7HcHh.jpg?imageslim)

***

#### 接入Python SDK

DuerOS开发平台中也提供了相应的Python SDK，以便于个人开发者通过该SDK来实现想要的技能。

#### 通过SSH登陆

官方提供的DuerOS镜像系统是基于Raspbian系统的，所以我们可以按照在树莓派上安装Raspbian系统的方式来配置SSH服务。

为了找到树莓派的IP地址，我们可以使用 `Fing` 这个app来查看当前局域网上连接的所有设备。

然后通过SSH使用默认的用户名 `pi` 密码 `raspberry` 登陆DuerOS系统。我这里使用的是 `Xshell`，也可以选择 `Putty` 等其他软件。

在当前的用户目录下创建一个目录，用于后面的操作。比如我这里创建的目录名为 `Leafney`：

```
$ mkdir Leafney
$ cd Leafney
```

#### 停止现有小度功能，因为会占用MIC资源

```
$ sudo systemctl disable duer
$ sudo systemctl stop duer
```

#### 安装需要的依赖

##### 更换地址源

在操作之前，建议先更换地址源。因为DuerOS系统是基于 `Raspbian jessie` 版本的，操作如下：

```
$ sudo vim /etc/apt/sources.list
```

把原来的第一行用#注释掉，在末尾添加下面一行：

```
deb http://mirrors.aliyun.com/raspbian/raspbian/ jessie main contrib non-free rpi
```

还需要更改deb的源地址，这里可选择清华的源:

```
deb https://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ jessie main ui
```

或中科大的源：

```
deb http://mirrors.ustc.edu.cn/archive.raspberrypi.org/debian/ jessie main ui
```

编辑以下文件添加:
```
$ sudo vim /etc/apt/sources.list.d/raspi.list
```

##### update更新

修改完成后，更新：

```
$ sudo apt-get update
```

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/mhG80D6keF.jpg?imageslim)

##### 其他依赖包

安装其他的依赖包：

hyper库用来支持http2.0 client, pyaudio用来支持录音，tornado用来完成oauth认证。

```
$ sudo apt-get install python-dateutil gir1.2-gstreamer-1.0 python-pyaudio libatlas-base-dev python-dev
$ sudo pip install tornado hyper
```

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/Ha2dDjLkCl.jpg?imageslim)

#### 下载编译好的openssl和Python安装包

由于DuerOS运行所需要的依赖环境跟平台是相关的。比如DuerOS是基于Http2 ALPN的，但树莓派官方镜像的OpenSSL并不支持，而对应的Python库依赖于OpenSSL。为了在树莓派平台上支持Python的DuerOS SDK，专门交叉编译了OpenSSL和Python。

*从如下地址下载openssl安装包*(链接: https://pan.baidu.com/s/1skAP6WH 密码: wknz)
*从如下地址下载python2.7.14安装包*(链接: https://pan.baidu.com/s/1o8MHkzK 密码: ngx4)

将下载的两个文件用 `FileZilla` 传输到树莓派的 `/home/pi/Leafney` 目录下：

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/gead092HB7.jpg?imageslim)

然后分别解压：

```
$ sudo tar -zxvf openssl1.1.tar.gz -C /usr
```

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/9BLDE9GllL.jpg?imageslim)

```
$ sudo tar -zxvf python2.7.14.tar.gz -C /usr/local/
```

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/JkK98AlhB6.jpg?imageslim)

替换已有的python：

```
$ sudo rm -rf /usr/bin/python
$ sudo ln -s /usr/local/python2.7.14/bin/python /usr/bin/python
```

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/8A48988iE7.jpg?imageslim)

##### 下载Python SDK和示例代码

```
$ git clone https://github.com/MyDuerOS/DuerOS-Python-Client.git
$ cd DuerOS-Python-Client
$ git checkout raspberry-dev
```

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/1H0kJCiD7i.jpg?imageslim)

#### 初次授权

如果直接按照官方给出的教程配置 [Step by Step带你玩转DuerOS - Python DuerOS SDK[树莓派平台] (3)](http://open.duer.baidu.com/forum/topic/show?topicId=244796)，下一步就是授权操作了。

```
$ ./auth.sh
```

执行后在 `Xshell` 中有提示 `A web page should is opened. If not, go to http://127.0.0.1:3000 to start` 。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/GBIHDg2A9k.jpg?imageslim)

因为这里是要求访问 `127.0.0.1` ，所以必须在树莓派系统中通过浏览器来访问。我在Windows系统下通过 `树莓派IP+端口3000` 的方式访问，会提示 “授权回调页地址错误” 的错误页面。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/AedIAK9chJ.jpg?imageslim)

我并没有多余的HDMI数据线来直接连接树莓派和显示器，所以这里我用远程桌面的方式来配置。

#### 安装远程桌面

树莓派下的远程桌面我们选择 `xrdp` 或者 `VNC` 来实现。

##### xrdp

xrdp 可以使用 windows下的远程桌面直接连接，不过这种方式只适合于Windows系统下连接。

在树莓派下执行安装：

```
$ sudo apt-get install xrdp
```

打开windows系统的 “远程桌面连接” 程序，输入树莓派的IP地址进行连接。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/mDL3h3G9GE.jpg?imageslim)

在弹出的 `Login to xrdp` 窗口中，输入树莓派的用户名和密码，点击 `OK` 连接。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/ee14cebf7G.jpg?imageslim)

##### VNC

如果你是MAC系统或者不喜欢Windows自带的远程桌面，可以使用适合于全平台的 `VNC` 。

###### VNC初始化

在树莓派下执行安装：

```
$ sudo apt-get install tightvncserver
```

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/fLbc7ehf6B.jpg?imageslim)

增加一个桌面，执行：

```
$ tightvncserver
```

会要求设置一个连接的密码并重复输入。

会询问是否设置一个只读方式的密码，一般选择否 `n` 。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/7d91GlFC56.jpg?imageslim)

###### 连接

从网站 [vncViewer](http://www.realvnc.com/download/viewer/) 下载vncViewer。打开程序后连接：

```
your Pi IP:1

# 比如我的设置：
192.168.5.130:1
```

###### 关闭桌面

关闭VNC桌面只需要在树莓派中将VNC的服务kill掉即可。在 `Xshell` 中操作：

```
$ vncserver -kill :1
```

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/JDkd7Idcak.jpg?imageslim)

***

#### 再次授权

再次进入树莓派的 `/home/pi/Leafney/DuerOS-Python-Client` 目录，启动授权：

```
$ cd /home/pi/Leafney/DuerOS-Python-Client

$ ./auth.sh
```

通过远程桌面访问，在树莓派的桌面系统下打开浏览器，访问 `127.0.0.1:3000` 地址，会出现 “百度账号的登陆授权页面” 。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/1BBDKhaC6c.jpg?imageslim)

不过这个是官方的测试账号 `GitHub项目测试账号`。如果我们想要配置自己的设备，还是需要去申请自己的client_id和client_secret来调用。

这里我不在继续往下操作，先去申请自己的ClientID信息。

在 `Xshell` 中按 `Ctrl+C` 停止启动的web服务。

#### 创建设备

打开 DuerOS开放平台官网 [DuerOS开放平台](https://dueros.baidu.com/open) ，选择右上角 “控制台” -- “设备控制台” -- 在打开的新页面选择 “配置新设备” 。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/BKfi41AImL.jpg?imageslim)

然后在 `请选择终端场景` 中选择 “音箱” 点击 “下一步” 。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/BfD84IEJJ1.jpg?imageslim)

在 `请选择操作系统` 界面选择第一项 “Linux” 或者也可以选择最下面的 “点击这里” ，没有太大区别。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/GF9559eF6j.jpg?imageslim)

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/926bH7lJBl.jpg?imageslim)

输入 “产品名称”，比如这里我取名叫 `贾维斯` （电影钢铁侠里的人工智能系统），点击 “申请ClientID” ，下面会显示出相应的 client_id 和 client_secret等信息。这里，我们先将这两项记下来以待后面使用。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/ik2hLA9K0C.jpg?imageslim)

接下来是配置 “端能力”的页面，可以自定义选择，或者直接保持默认下一步即可。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/F2jhkif846.jpg?imageslim)

然后会弹出 `BOT配置` 页面。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/14d0Ik0ddi.jpg?imageslim)

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/im394L0fA7.jpg?imageslim)

​可以看到上面是一些 `音乐` `有声点播` `有声直播` 等等选项；下面有 `聊天定制` `语音唤醒服务` `自定义控制指令` 这些，如果看不懂呢可以不用管，直接下一步。后面会询问是否下载SDK，也不用管，直接点击下面的 “完成” 会提示 “创建产品成功” 。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/ldkA768AdC.jpg?imageslim)

这样我们就创建好了自己的client_id和client_secret。

#### 设置个人的ClientID信息

使用 `FileZilla` 软件，在树莓派的目录 `/home/pi/Leafney/DuerOS-Python-Client` 下找到 `app/auth.py` 这个文件，因为在控制台界面下不太方便编辑文件，所以这里我选择将该文件下载到Windows本地来编辑。

将 `auth.py` 下载到本地后，推荐使用 `SublimeText` 或 `NotePad++` 来进行编辑。

找到 `开发者注册信息` 部分，替换成刚刚申请的信息：

```
# 开发者注册信息
CLIENT_ID = 'XXXXX'
CLIENT_SECRET = 'XXXXXX'
```

然后将下面的 `使用开发者注册信息` 一行下面代码段前面的井号 `#` 去掉，解注释这一行。

```
    # 使用开发者注册信息
    auth.auth_request(CLIENT_ID, CLIENT_SECRET)
```

再将下面的 `使用默认的CLIENT_ID和CLIENT_SECRET` 一行下面代码行前面加一个井号 `#` 注释掉这一行。

```
    # 使用默认的CLIENT_ID和CLIENT_SECRET
    # auth.auth_request()
```

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/jLciAm6A4f.jpg?imageslim)

然后使用 `FileZilla` 将我们刚刚改好的 `auth.py` 上传到树莓派中。

#### 设置授权回调地址

在浏览器中访问 `控制台` -- `设备控制台` 页面 [设备控制台](https://dueros.baidu.com/didp/product/listview) , 选择我们刚刚创建的产品点击 “编辑” 选项。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/3ebjjK0LkG.jpg?imageslim)

在 `基础信息` 页面，可以查看刚刚创建设备的 client_id 等信息。这里我们点击 `OAUTH CONFIG URL` 这个链接：

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/Ba8jgec5kl.jpg?imageslim)

在新页面的左侧点击 `安全设置` 选项，在 `授权回调页` 的输入框中输入如下内容，然后点击 `确定` 保存修改。

```
http://127.0.0.1:3000/authresponse
```

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/fgmEh6ei60.jpg?imageslim)

#### 使用个人设备授权

完成上面的配置后，回到 `Xshell` 中，在树莓派的 `/home/pi/Leafney/DuerOS-Python-Client` 目录下，再次执行授权命令：

```
$ ./auth.sh
A web page should is opened. If not, go to http://127.0.0.1:3000 to start
```

然后在树莓派系统浏览器输入 `127.0.0.1:3000` 访问。

可以看到页面右侧的授权应用变成了我自己创建的设备名称。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/744die8bjc.jpg?imageslim)

如果你的授权页面中这里显示的是空的，那是因为你用的是中文名称。在“基础信息” 的 “名称”那里需要再次添加一下名称。如果是英文的话，这个名称会直接显示。我觉得这里可能是一个bug。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/1E5cDdb1jg.jpg?imageslim)

​在授权页面输入我的百度账号和密码进行授权。

看到提示 `Succeed to login DuerOS Voice Service` 的信息就说明授权成功了。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/e6dBAefhid.jpg?imageslim)

同时在 `Xshell` 下我们会看到输出相应的授权信息。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/KgGHdJGJ89.jpg?imageslim)

#### 语言唤醒

执行如下命令：

```
$ ./wakeup_trigger_start.sh
```

使用唤醒词 `小度小度` 就能唤醒了。

因为我在 `Xshell` 下操作时发现命令行下的中文会有乱码的情况，所以我改用远程桌面下树莓派上自带的 `Terminal` 程序来执行。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/e6iEB7LKEm.jpg?imageslim)

也可以使用enter按键唤醒，执行命令：

```
$ ./enter_trigger_start.sh
```

使用enter键回车唤醒。

这里我尝试了上面的两种唤醒方式，发现不知道是哪里的问题，音箱都没有声音输出。查看输出的日志信息是能看到有音频文件下载成功并播放的。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171205/mFK66gbHId.jpg?imageslim)

#### 解决没有声音的问题

我使用 `alsamixer` 然后按 `F6` 切换使用的声卡，发现无论如何切换，似乎都没有效果。

后来我考虑**将音箱线换到树莓派本身的音频接口**上，发现居然有声音输出了。不过树莓派自带的音频输出杂音还是很吵的。

这里要注意的是不能在树莓派通电的情况下切换音频口，我发现如果直接将音频线从DuerOS板子的音频口换到树莓派的音频口上时，刚一接触的时候噪音是非常大的，所以最后我是将树莓派关机然后切换的。

#### 感受

1. 使用Python SDK 最后唤醒的时候需要将音频接口插到树莓派的音频接口上，这一点在论坛的文档中没有说明，可能会给一些人操作时带来困惑。
2. 个人认为应该是有方法使用DuerOS开发版的音频接口的，毕竟没有杂音嘛。需要进一步研究一下。
3. 发现在使用 Python SDK 唤醒小度时，语言识别的效果不如镜像系统中语音的识别准确度高。

