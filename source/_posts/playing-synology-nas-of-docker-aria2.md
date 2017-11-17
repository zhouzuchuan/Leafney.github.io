---
title: 玩转群晖NAS--下载神器aria2
date: 2017-11-17 22:28:45
tags: 
    - SynologyNAS
categories: 
    - 玩转群晖NAS
description: 群晖下Docker配置下载神器aria2
---

上一篇文章说了如何配置Docker加速器，现在下载Docker镜像文件已经非常的快了。但是对于其他的一些文件比如电影、程序文件等来说，如何在NAS中来快速的下载呢？

虽然群晖中已经自带了下载套件，不过看到那个界面我就有种不想用的感觉。这里推荐一个开源的下载神器 -- `aria2`，号称迅雷的替代者。

这里我还选择在Docker中来配置，选择的镜像为我之前创建的 `leafney/debian-aria2-kode` 镜像。该镜像自带了 aria2下载程序、ariaNg管理页面以及KodExplorer文件管理页面。

具体可以访问：[Leafney/debian-aria2-kode](https://github.com/Leafney/debian-aria2-kode)

***

#### 创建aria2容器

打开群晖的docker套件，选择 “注册表” 项，搜索并下载镜像 `leafney/debian-aria2-kode` 。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/h0GeCiKJG3.png?imageslim)

下载完成后，选中该镜像，点击 “启动” 菜单，打开 “创建容器” 界面。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/Dc9aFm6f6c.png?imageslim)

为该容器设置一个自定义的名称，我这里命名为 `aria2-kode`，然后打开 “高级设置” 窗口。

在 “高级设置” 选项卡，选中 “启用自动重新启动” 及 “创建桌面快捷方式” 。

自动重新启动是在容器不当关机的情况下回尝试自动重启的操作。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/EcA9f5A3Bl.png?imageslim)

在 “卷” 菜单中，为创建的容器添加一个文件夹用来管理和查看我们通过aria2下载的文件。因为要存储新文件，所以这里不要勾选 “只读” 项。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/b2aick4cmg.png?imageslim)

在 “端口设置” 菜单中，已经列出了镜像中预设的端口信息，在 “本地端口” 项下，我们为其指定相应的端口，不选择默认的 “自动” 。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/A8JD9Behmg.png?imageslim)

然后点击 “应用” 按钮。

回到 “创建容器” 界面，点击 “下一步” 。查看我们设置的容器信息，勾选左下角的 “向导完成后运行此容器” 项，然后点击 “应用” 等待容器启动。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/kD4GleC0Aa.png?imageslim)

***

#### 查看容器信息

选择左侧 “容器” 项，可以看到我们刚刚创建的容器已经启动了。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/ih68G0fH77.png?imageslim)

点击顶部的 “详情” 选项，可以查看容器 `aria2-kode` 的信息。

在 “日志” 项下，可以查看当前容器运行时输出的日志记录。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/lEiH8f74eJ.png?imageslim)

***

#### 配置KodExplorer

在浏览器中输入 `群晖ip:6860` ，打开 `KodExplorer` 的登录界面。看到 “运行环境检测” 下输出 “Successful!” 说明我们的容器已经正常的跑起来了。

首先要设置 `KodExplorer` 资源管理器的管理员 `admin` 的密码。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/kaaL1fL1aE.png?imageslim)

然后使用管理员账号登录。登录后可以看到 `KodExplorer` 的文件管理页面和我们平时使用的资源管理器页面非常的相似，操作起来也没有什么难度。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/AG7Bj3LAaB.png?imageslim)

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/f4J96e8C8l.png?imageslim)

***

#### 配置aria2

在浏览器中输入 `群晖ip:6801` ,打开 `AriaNg` 的管理页面。进入后会弹出 “认证失败” 的错误弹窗，不用管它。

选择左侧 “系统设置” 下的 “AriaNg 设置” 项。在右侧选择 “RPC(192.168.x.xx...” 的菜单，然后配置之前创建容器时设置的 “Aria2 RPC 地址” 端口号和 “Aria2 RPC 密钥” 项。

RPC密钥默认是 `123456` 。设置完成后点击 “重新加载页面” 应用配置。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/jAGbI57kfb.png?imageslim)

然后可以看到 `Aria2 状态` 已经显示为 “已连接” 的状态了。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/EI19DI0LF8.png?imageslim)

至此，aria2就配置完成了。选择左侧的 “正在下载” 项新建下载任务即可。

***

#### 管理下载文件

这里我以下载 `BaiduExporter` 为例来示范如何管理下载的文件。

[BaiduExporter](https://github.com/acgotaku/BaiduExporter)

##### 下载文件

打开 `BaiduExporter` 的github页面，在master分支下，选择右侧的 `Clone or download` 项下的 `Download Zip` ，右击选择 “复制链接地址” 。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/j7blJKDCib.png?imageslim)

打开 “AriaNg” 页面，在 “正在下载” 页面 “新建” 下载任务。粘贴下载链接，点击 “立即下载” 开始。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/EH82F8J37d.png?imageslim)

下载完成后，会在 “已完成/已停止” 菜单中显示。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/73fCaB1Dmg.png?imageslim)

##### KodExplorer文件管理

要查看我们刚刚下载的文件，在浏览器打开 `KodExplorer` 页面，选择上面的目录路径，点击根目录项，查看所有的文件及目录。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/jBB9DF8b40.png?imageslim)

找到 `app` 目录，打开里面的 `aria2down` 目录即可查看到我们刚刚下载的文件了。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/llG1fjCAB8.png?imageslim)

##### 在群晖DSM中查看下载文件

在创建容器时我们为容器指定了群晖本地的下载文件目录。打开群晖DSM界面 -- “File Station” 文件管理器，找到我们设置的目录，可以看到容器为我们自动创建了三个目录，在 `aria2down` 下就能找到我们刚刚下载的文件了。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/6EB6Kjc45f.png?imageslim)

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/G4j3hE8KH9.png?imageslim)

***

#### 百度网盘文件下载

知道了如何下载和如何管理文件，接下来我们看看具体的应用。因为之前网盘刚兴起的时候，我把大部分的文件都放在到了百度网盘里，但后来网盘逐渐衰落，百度网盘的客户端下载文件还会限速，除非你冲超级会员才行。

今天，我们就用aria2来解决这个问题。

##### 安装插件

想要下载百度网盘中的文件，首先需要安装一个插件，也就是上面我们已经下载的 `BaiduExporter`。

在 `KodExplorer` 管理界面或群晖的 `DSM` 界面，选中文件 `BaiduExporter-master.zip` 右击选择下载均可将该文件下载到当前电脑上，解压后看到一个名为 `BaiduExporter.crx` 的文件。

打开 `Chrome` 浏览器 -- “更多工具” -- “扩展程序” 界面。将 `BaiduExporter.crx` 拖放到该页面以安装。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/4ja3030bB1.png?imageslim)

打开百度网盘页面，在顶部菜单栏中可以看到多出了一项 “导出下载” 的按钮。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/94ELDGH5hG.png?imageslim)

##### 设置ARIA2 RPC

仍在百度网盘页面，选择菜单 “导出下载” -- “设置” 项，在 `ARIA2 RPC` 右侧输入RPC地址，格式为 ：`http://192.168.5.120:6800/jsonrpc` 。

因为我的aria2是添加了密钥的，所以最后的rpc地址格式应为：`http://token:RPC密钥@192.168.5.120:6800/jsonrpc` ，即

> 设置密码以后需要在导出下面的设置里在 JSONRPC 的地址的 `http://` 后面 `localhost` 前面加上 `token:你的密码@`。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/AkL6GebFaB.png?imageslim)

##### 下载

点击应用后，勾选百度网盘中要下载的文件或文件夹，选择 “导出下载” 菜单下的 “ARIA2 RPC”，会弹出 “下载成功，赶紧去看看吧！” 的提示信息。切换到 `AriaNg` 页面，我们可以看到在百度网盘上选择的文件已经在依次下载了。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/GlLGJIkd74.png?imageslim)

另外在 `KodExplorer` 和群晖DSM的资源管理界面，都可以看到正在下载的文件。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171117/lggD4efImf.png?imageslim)

##### 注意

这里发现一个问题：如果在 `Mac10.13` 系统上使用release下的 `v0.8.5` 的版本在百度网盘页面选中文件后，“导出下载” 的按钮就消失了。更新成当前的master版本 `v0.9.10` 后没有问题。所以上面直接推荐安装master版本。

具体可以看 github issues：[mac os 10.13 无法使用了](https://github.com/acgotaku/BaiduExporter/issues/492)

***

#### 相关参考

* [Aria2c那边设置rpc-secret后，chrome里的aria按钮点击后就不能无法下载了，报：是不是没有启动aria2](https://github.com/acgotaku/BaiduExporter/issues/299)
