---
title: 通过路由器找回忘记的宽带密码
date: 2017-11-13 09:50:46
tags:
    - Skill
description: 宽带账号的密码忘记了怎么办? 下面说说通过路由器找回密码的方法
---

可能是因为赶上了双十一的缘故,最近我的宽带网络总是时好时坏的,给宽带客服打了电话,说会找师傅过来给我看看.突然间想起来好像我的宽带自从安装上以后,我就没再改动过,怎么也想不起来宽带的密码了.账号的话可以在路由器中直接看到,而密码却是显示成星号,还不能复制出来.

所以特意上网找了找方法,记录于此.

#### 备份路由器配置

我的路由器是 `TP-Link WR847N` 型号的，其他路由器的方法类似。

第一步：在浏览器输入路由器网关地址（一般是192.168.1.1）进入路由器登录界面

第二步：输入路由器账号和密码登录（如果未更改过一般都是admin）到路由器管理界面

第三步：在左菜单栏点击 “系统工具” -- “备份和载入配置” 

第四步：在右侧对话框中点击 “备份配置文件” 按钮

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171112/bEf4IBJgeA.jpg?imageslim)

第五步：保存配置文件，名称为 `config.bin` 。

#### 通过配置文件找回

第六步：从网站 [RouterPassView](http://www.nirsoft.net/utils/router_password_recovery.html#DownloadLinks) 下载软件RouterPassView。

RouterPassView 是 NirSoft 出品的一款路由密码恢复软件，可以查看绝大多数家用路由的配置文件中保存的密码。

第七步：用 RouterPassView打开备份的配置文件 `config.bin`, 就能看到当前路由器上已配置的所有账号和密码了。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171112/6jeAcDFc4K.jpg?imageslim)

#### 附件

百度网盘链接: [Download](https://pan.baidu.com/s/1qYkKaVq) 密码: hwpx
