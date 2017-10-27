---
title: Mac系统下配置Hexo博客运行环境遇到的问题
date: 2017-10-27 14:38:51
tags:
    - Node.js
    - Hexo
categories: 
    - Hexo博客搭建
description: 今天在Mac下更新hexo博客时,遇到了一些安装 Node.js 和安装 hexo 相关的小问题,特此记录下来.
---

今天在Mac下更新hexo博客时,遇到了一些安装 Node.js 和安装 hexo 相关的小问题,特此记录下来.

#### Mac下配置Node.js环境

安装 node.js 有多种方法：使用 `homebrew` 安装或者直接下载 `安装包`。 

##### 官网下载安装包

从node.js官网 [Node.js](https://nodejs.org/en/) 下载对应系统的安装包,打开会提示安装位置:

```
This package will install:
  • Node.js v8.8.1 to /usr/local/bin/node
  • npm v5.4.2 to /usr/local/bin/npm
```

按照步骤安装即可.

##### homebrew安装

通过 `homebrew` 安装,直接执行如下命令:

```
$ brew install node
```

##### 检测安装是否成功 

终端输入 `-v` , 成功则显示版本号:

```
$ node -v
v8.8.1
$ npm -v
5.4.2
$
```

***

#### Mac系统下Finder显示隐藏文件

涉及到一些以 `.` 开头的文件或目录(如.git目录)或者隐藏文件,默认在Finder下是看不到的.

我们可以通过命令 `Command+Shift+.` 来在Finder中快速的切换显示出隐藏的文件或文件夹,再按一次,恢复隐藏.

***

#### 更换Node.js镜像源

由于npm的官方镜像源在国外,而由于国内"众所周知的"的网络原因,访问默认的官方镜像源常常会出问题.我们可以更改为国内的镜像源来加速软件的安装.

##### 淘宝npm镜像

目前国内推荐的是淘宝的npm镜像:

* 搜索地址：[http://npm.taobao.org/](ttp://npm.taobao.org/)
* registry地址：[http://registry.npm.taobao.org/](http://registry.npm.taobao.org/)

##### 如何使用

###### 临时使用

以下载 `express` 软件为例:

```
npm --registry https://registry.npm.taobao.org install express
```

###### 持久使用

```
npm config set registry https://registry.npm.taobao.org
```

配置后可通过下面方式来查看是否设置成功:

```
npm config get registry
```

我的操作记录:

```
$ npm config get registry
https://registry.npmjs.org/
$ npm config set registry https://registry.npm.taobao.org
$ npm config get registry
https://registry.npm.taobao.org/
$
```

**提醒** : 我在实际操作时发现淘宝的npm镜像源有时候也会请求失败,然后又切换回了官方源(`npm config set registry https://registry.npmjs.org/`)发现能够操作成功了.所以是否更换镜像源还要根据实际情况来定.

```
$ npm config get registry
https://registry.npm.taobao.org/
$ npm install
npm ERR! code ENOTFOUND
npm ERR! errno ENOTFOUND
npm ERR! network request to https://registry.npm.taobao.org/hexo-generator-category failed, reason: getaddrinfo ENOTFOUND registry.npm.taobao.org registry.npm.taobao.org:443
npm ERR! network This is a problem related to network connectivity.
npm ERR! network In most cases you are behind a proxy or have bad network settings.
npm ERR! network
npm ERR! network If you are behind a proxy, please make sure that the
npm ERR! network 'proxy' config is set properly.  See: 'npm help config'

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/xxx/.npm/_logs/2017-10-26T14_59_23_770Z-debug.log
$ 
$ 
$ npm config set registry https://registry.npmjs.org/
$ npm install
npm WARN deprecated swig@1.4.2: This package is no longer maintained

> dtrace-provider@0.8.5 install /Users/xxx/Project17/Leafney.github.io/node_modules/dtrace-provider
> node scripts/install.js


> fsevents@1.1.2 install /Users/xxx/Project17/Leafney.github.io/node_modules/fsevents
> node install

[fsevents] Success: "/Users/xxx/Project17/Leafney.github.io/node_modules/fsevents/lib/binding/Release/node-v57-darwin-x64/fse.node" already installed
Pass --update-binary to reinstall or --build-from-source to recompile

> hexo-util@0.6.1 postinstall /Users/xxx/Project17/Leafney.github.io/node_modules/hexo-util
> npm run build:highlight


> hexo-util@0.6.1 build:highlight /Users/xxx/Project17/Leafney.github.io/node_modules/hexo-util
> node scripts/build_highlight_alias.js > highlight_alias.json

npm notice created a lockfile as package-lock.json. You should commit this file.
added 430 packages in 119.035s
```

***

#### Mac install hexo use sudo but sitll permission denied

##### 安装报错

参照hexo官网 [Hexo](https://hexo.io/zh-cn/index.html) 安装hexo时,使用命令 `npm install hexo-cli -g` 却报没有权限:

```
$ npm install hexo-cli -g
npm WARN checkPermissions Missing write access to /usr/local/lib/node_modules
npm ERR! path /usr/local/lib/node_modules
npm ERR! code EACCES
npm ERR! errno -13
npm ERR! syscall access
npm ERR! Error: EACCES: permission denied, access '/usr/local/lib/node_modules'
npm ERR!  { Error: EACCES: permission denied, access '/usr/local/lib/node_modules'
npm ERR!   stack: 'Error: EACCES: permission denied, access \'/usr/local/lib/node_modules\'',
npm ERR!   errno: -13,
npm ERR!   code: 'EACCES',
npm ERR!   syscall: 'access',
npm ERR!   path: '/usr/local/lib/node_modules' }
npm ERR!
npm ERR! Please try running this command again as root/Administrator.

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/xxx/.npm/_logs/2017-10-27T01_21_01_871Z-debug.log
```

然后我换用管理员权限,加上 `sudo` ,执行如下:

```
$ sudo npm install hexo-cli -g
Password:
/usr/local/bin/hexo -> /usr/local/lib/node_modules/hexo-cli/bin/hexo

> dtrace-provider@0.8.5 install /usr/local/lib/node_modules/hexo-cli/node_modules/dtrace-provider
> node scripts/install.js

fs.js:768
  return binding.rename(pathModule._makeLong(oldPath),
                 ^

Error: EACCES: permission denied, rename '/usr/local/lib/node_modules/hexo-cli/node_modules/dtrace-provider/compile.py' -> '/usr/local/lib/node_modules/hexo-cli/node_modules/dtrace-provider/binding.gyp'
    at Object.fs.renameSync (fs.js:768:18)
    at Object.<anonymous> (/usr/local/lib/node_modules/hexo-cli/node_modules/dtrace-provider/scripts/install.js:14:4)
    at Module._compile (module.js:612:30)
    at Object.Module._extensions..js (module.js:623:10)
    at Module.load (module.js:531:32)
    at tryModuleLoad (module.js:494:12)
    at Function.Module._load (module.js:486:3)
    at Function.Module.runMain (module.js:653:10)
    at startup (bootstrap_node.js:187:16)
    at bootstrap_node.js:608:3

> fsevents@1.1.2 install /usr/local/lib/node_modules/hexo-cli/node_modules/fsevents
> node install

[fsevents] Success: "/usr/local/lib/node_modules/hexo-cli/node_modules/fsevents/lib/binding/Release/node-v57-darwin-x64/fse.node" already installed
Pass --update-binary to reinstall or --build-from-source to recompile

> hexo-util@0.6.1 postinstall /usr/local/lib/node_modules/hexo-cli/node_modules/hexo-util
> npm run build:highlight


> hexo-util@0.6.1 build:highlight /usr/local/lib/node_modules/hexo-cli/node_modules/hexo-util
> node scripts/build_highlight_alias.js > highlight_alias.json

sh: highlight_alias.json: Permission denied
npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! hexo-util@0.6.1 build:highlight: `node scripts/build_highlight_alias.js > highlight_alias.json`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the hexo-util@0.6.1 build:highlight script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

┌────────────────────────────────────────────────────────┐
│                npm update check failed                 │
│          Try running with sudo or get access           │
│          to the local update config store via          │
│ sudo chown -R $USER:$(id -gn $USER) /Users/xxx/.config │
└────────────────────────────────────────────────────────┘
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: dtrace-provider@0.8.5 (node_modules/hexo-cli/node_modules/dtrace-provider):
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: dtrace-provider@0.8.5 install: `node scripts/install.js`
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: Exit status 1

npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! hexo-util@0.6.1 postinstall: `npm run build:highlight`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the hexo-util@0.6.1 postinstall script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/xxx/.npm/_logs/2017-10-27T02_56_29_887Z-debug.log
```

##### 解决方法

第一步,赋予目录权限:

```
$ sudo chown -R `whoami` /usr/local/lib/node_modules
```

第二步,安装hexo:

```
$ npm install hexo-cli -g
```

**需要注意的点**: 在安装hexo时,不要用 `sudo` 命令.

***

#### 相关参考

* [国内优秀npm镜像推荐及使用](http://riny.net/2014/cnpm/)
* [Mac install hexo use sudo but sitll permission denied](https://github.com/hexojs/hexo/issues/2785)
* [`npm update -g` fails and causes `/usr/local/lib/node_modules` to be deleted](https://github.com/npm/npm/issues/8165)

