Hexo博客

[![Build Status](https://travis-ci.org/Leafney/Leafney.github.io.svg?branch=hexo)](https://travis-ci.org/Leafney/Leafney.github.io)

#### 配置Node.js环境

##### Win和Linux下

从node.js官网 [Node.js](https://nodejs.org/en/) 下载对应系统的安装包安装即可.

##### Mac下

安装 node.js 有多种方法：使用 `brew` 安装或者直接下载 `安装包`。 

从node.js官网 [Node.js](https://nodejs.org/en/) 下载对应系统的安装包,打开会提示安装位置:

```
This package will install:
	•	Node.js v8.8.1 to /usr/local/bin/node
	•	npm v5.4.2 to /usr/local/bin/npm
```

按照步骤安装即可.

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

#### 获取项目文件

使用git获取项目文件:

```
$ git clone https://github.com/Leafney/Leafney.github.io.git
```

切换到 `hexo` 分支:

```
$ git checkout hexo
```

***

#### 部署

##### 直接部署

下载项目文件后，在Hexo博客项目根目录下即 `_config.xml` 文件所在目录下打开 `CMD` 命令窗口或 `shell` 窗口执行：

```
> npm install
``` 

等待安装hexo的依赖项。

##### 重新部署

安装好Node.js环境后, 执行 `npm install hexo-cli -g` 安装hexo相关的依赖包:

```
> npm install hexo-cli -g

# 建站
> hexo init <folder>
> cd <folder>
> npm install
```

***

#### 基础操作

##### 查看hexo版本

```
> hexo -v
```

##### 创建新页面

```
> hexo new "blog name"
```

##### 创建新分类

```
> hexo new page "page name"
```

##### 清理public文件夹

```
> hexo clean
```

##### 生成

```
> hexo g
```

##### 预览

```
> hexo s
```

##### 部署

```
> hexo d
```

##### 生成并预览

```
> hexo s -g
```

##### 生成并部署

```
> hexo d -g
```

***

#### Git分支操作

##### 查看本地所有分支(分之名称前面带*表示当前分支)

```
> git branch
```

##### 查看远程所有分支

```
> git branch -r
```

##### 创建分支 hexo

```
> git branch hexo
```

##### 切换到 hexo 分支

```
> git checkout hexo
```

##### 创建并切换到新分支

```
> git checkout -b hexo
```

##### 删除分支

```
> git branch -d hexo
```

##### 提交本地test分支作为远程的test分支

```
> git push origin test:test
```

##### merge合并分支(将名称为[hexo]的分支与当前分支[master]合并)

```
> git branch
* master
  hexo

> git merge hexo
```

##### 获取远程指定分支

```
> git pull origin hexo
```

***
