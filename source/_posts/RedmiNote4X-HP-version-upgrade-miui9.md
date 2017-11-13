---
title: 红米Note4X高通版升级MIUI9
date: 2017-11-12 09:16:56
tags:
    - XiaoMI
description: 红米Note4X高通版 MIUI8稳定版升级MIUI9开发版 采用卡刷的方式 不需要BL解锁
---

双十一当天在京东上买了一个红米Note4X高通版的手机,收到货后发现系统仍然是MIUI8的稳定版.之前看过MIUI9的介绍视频,官方给出的标语是"快如闪电",所以就想体验一把.我的方法是采用 `卡刷` 的方式，不需要BL解锁。

#### 当前系统版本 

在手机的"设置"--"我的设备" 下查看当前的MIUI版本:

```
MIUI 8.5 稳定版   8.5.6.0(MCFCNED)
```

#### 先从稳定版升级开发版

##### 不能直接刷最新版

因为小米Note4x稳定版8.5的系统是基于Android6.0开发的，而最新的MIUI9是基于Android7.0开发的，Android版本不一致也就导致无法直接卡刷到最新版，会报错。

可行的方法是可以先卡刷MIUI8 `7.4.6` 的开发版（基于Android6.0），然后再刷MIUI9最新的开发版。

##### 卡刷Android6.0开发版

下载 `7.4.6` 开发版卡刷包,然后拷贝到手机的内置存储中。

在手机端选择 `设置` -- `我的设备` -- `MIUI版本` -- 进入 “系统升级” 界面

点击 `右上角三点` ，选择 `手动选择安装包`

选择刚刚下载的卡刷包，确定。等待其解密并自动升级。

升级完成后，在 `设置` -- `我的设备` -- `MIUI版本` -- 进入 “系统升级” 界面

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171113/IKDBk7kbEa.png?imageslim)

这时如果点击 “检查更新” 的话，收到的应该是MIUI8的最新开发版的更新包,所以这里不点击“立即更新”。(MIUI论坛中官方给出的说法是现在MIUI8开发版可以直接自动检查升级到MIUI9的最新开发版了,不知道我这里为什么不行?如果你的可以,那就直接升级即可,否则继续下面的操作)

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171113/lL6DH0cfdC.png?imageslim)

##### 卡刷MIUI9最新开发版

从网址 [http://www.miui.com/download-326.html](http://www.miui.com/download-326.html) 下载 `红米Note4X(高通平台)` 的最新开发版的完整卡刷包，然后拷贝到手机中，依照上一步的操作方法通过 `手动更新` 的方式来升级。

我这里下载到的是当前的最新版本 `MIUI9开发版 7.11.9` 的版本 : `miui_HMNote4X_7.11.9_71db0b04ec_7.0.zip` 。

等待其解密安装包并自动更新完成即可。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171113/Ck2ElHG14d.png?imageslim)

***

上面的整个升级过程要保证手机电量充足,按照步骤操作即可,非常适合小白升级.

***

#### 附件

百度网盘 [Download](https://pan.baidu.com/s/1c2GqHVy)  密码: d2v7
1. MIUI8(Android6.0)_7.4.6.zip
2. MIUI9(Android7.0)_7.11.9.zip
