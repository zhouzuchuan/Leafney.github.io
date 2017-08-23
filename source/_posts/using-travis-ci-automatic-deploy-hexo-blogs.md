---
title: 使用Travis CI自动部署Hexo博客
date: 2017-08-09 18:01:35
tags:
	- Hexo
	- Travis-CI
description: 自从使用GitHub Pages和Hexo来发布博客之后，不得不说方便了许多，只需要几个简单的命令博客就发布了。但在不断的使用中发现每次的发布操作也挺耗时的。 我一般的操作是将平时整理好的md文件放到私有的git仓库中（感兴趣可了解 "Ubuntu Gogs 用更简单的方式部署、升级或迁移Gogs服 ...
---

自从使用GitHub Pages和Hexo来发布博客之后，不得不说方便了许多，只需要几个简单的命令博客就发布了。但在不断的使用中发现每次的发布操作也挺耗时的。

我一般的操作是将平时整理好的md文件放到私有的git仓库中（感兴趣可了解 [Ubuntu-Gogs 用更简单的方式部署、升级或迁移Gogs服务](https://github.com/Leafney/ubuntu-gogs)），每次发布的时候都要先将文件 `clone` 到本地，然后配置一下hexo的运行环境，接着再执行 `hexo s -g` 来预览和调整，最后执行 `hexo d` 命令将博客发布上去，在这之前如果你没有配置过GitHub的 `SSH Key`,还要花一些时间来弄权限的问题。久而久之就发现这样操作起来实在是太繁琐了。

后来看到一篇文章介绍可以使用Travis CI来自动部署hexo的博客，只需要将md文件 `pull` 到仓库中博客就自动发布好了。趁着这几天工作任务不太着急，研究了一下，特纪录在此，希望能帮到有需要的朋友。

Travis CI 是目前新兴的开源持续集成构建项目，用来构建托管在GitHub上的代码。它提供了多种编程语言的支持，包括Ruby，JavaScript，Java，Scala，PHP，Haskell和Erlang在内的多种语言。


#### 配置GitHub Pages

如果你是新手或者还没有自己的 `GitHub Pages` 博客站点，可以先看我之前的文章 [使用GitHub搭建Hexo静态博客 | IT范儿](http://www.itfanr.cc/2016/09/24/use-github-to-build-hexo-static-blog/) 了解如何配置，具体过程这里不再详述。

##### 创建 hexo 分支

因为我之前的博客源文件是存放在私有的git管理工具下，如果我们要使用Travis CI自动部署，必须将这些博客的源码文件放到GitHub上才能被Travis访问到。因为 `GitHub Pages` 默认要求必须使用 `master` 分支存放静态文件，我们可以在该仓库下使用其他分支来存放博客源码文件，或者新创建一个仓库来单独保存。这里我们把hexo博客的源码放在 `hexo` 分支下，博客的静态文件部署在 `master` 分支下。

对于如何在GitHub上创建分支，相关操作命令如下，仅供参考：

```
# 克隆项目到本地
> git clone https://github.com/Leafney/Leafney.github.io.git

# 创建并切换到 hexo 分支
> git checkout -b hexo
```

当切换到 `hexo` 分支后，因为我们是需要用 `hexo` 分支来存放博客源码文件的，所以，将 `hexo` 分支下的文件除 `.git` 目录外全部删除，然后将博客源码文件拷贝到该目录下，并 `commit` 到 `hexo` 分支.

然后我们需要将本地的 `hexo` 分支提交到远程仓库中

```
# 提交本地hexo分支到远程仓库的hexo分支
> git push origin hexo:hexo
```

这样我们在GitHub的仓库下就能看到 `hexo` 分支为博客源文件，`master` 分支为静态文件。

这里需要注意一点，当我们新增博客md文件时，获取远程分支时要指定分支的名称，否则会默认获取 `master` 分支：

```
> git pull origin hexo
```

***

#### 设置 Travis CI

使用 GitHub账户登录 [Travis CI官网](https://travis-ci.org/) ，进去后能看到已经自动关联了 GitHub 上的仓库。这里我们选择需要启用的项目，即 `yourname/yourname.github.io` 。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/170809/gmgDm96jGa.jpg?imageslim)

然后点击后面的齿轮图标进入设置界面。

如果你之前已经勾选过项目，可以进到项目主页中，在右上角找到 `More options` 选项下的 `Settings` 进入设置界面。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/170809/26B1D9j4eI.jpg?imageslim)

***

##### 通用设置

在 `General` 区域开启：`Build only if .travis.yml is present` 表示“只有当 `.travis.yml` 存在时才构建” ；开启：`Build branch updates` 表示 “当分支更新时构建” 两个选项，如下：

![mark](http://ouej55gp9.bkt.clouddn.com/blog/170809/34fAK0D1cH.jpg?imageslim)

Travis CI在自动构建完成后需要push静态文件到仓库的 `master` 分支下，而访问GitHub的仓库是需要权限的，下面来看看如何配置权限。

##### 配置 Access Token

如下图，`Environment Variables` 区域就是用来添加权限信息的。我们需要填写一个Token的名称和值，该名称可以在配置文件中以 `${变量名}` 来引用，该Token我们需要从Github中获取。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/170809/37cGl0m6me.jpg?imageslim)

***

##### 从GitHub获取Access Token

之前我们在使用命令 `hexo d` 部署hexo博客到GitHub上时，是因为本地有 `SSH key`，当交给 Travis 去自动部署时我们也需要设置可操作权限，这里我们使用GitHub提供的token变量来实现。

登陆 `GitHub` --`Settings` 选项，找到 `Personal access tokens` 页面。

点击右上角的 `Generate new token` 按钮会生成新的token，点击后提示输入密码后继续，然后来到如下界面，取个名字（我这里取 `Travis_Token` 下面的配置文件中会用到)，勾选相应权限，这里只需要 `repo` 下全部和 `user` 下的 `user:email` 即可。

 ![mark](http://ouej55gp9.bkt.clouddn.com/blog/170809/L8IgF7ijH5.jpg?imageslim)

生成完成后，将该token拷贝下来。这里需要注意的是该token只有这个时候才能看到，当再次进入这个页面时就只会显示之前设置的名称了。如果忘记了只能重新生成一个。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/170809/dHK41IbL4h.jpg?imageslim)

***

##### 在Travis CI中配置

将上面获取到的token添加到 `Environment Variables` 部分，值为该 `token` ,而名称即为上面设置的 `Travis_Token` (请更改为个人所设置名称)。**不勾选**后面的 `Display value in build log` . 否则会在日志文件中暴露你的 `token` 信息，而日志文件是公开可见的。

至此我们已经配置好了要构建的仓库和访问的token，接下来就是如何构建的问题了。

***

##### 创建 .travis.yml 文件

之前的步骤中我们勾选了一项 `Build only if .travis.yml is present`,所以我们要在博客源码文件的 `hexo` 分支下新增一个 `.travis.yml` 配置文件，其内容如下：

```
language: node_js # 设置语言

node_js: stable # 设置相应版本

install:
    - npm install # 安装hexo及插件

script:
    - hexo clean # 清除
    - hexo g # 生成

after_script:
    - cd ./public
    - git init
    - git config user.name "yourname" # 修改name
    - git config user.email "your email" # 修改email
    - git add .
    - git commit -m "Travis CI Auto Builder"
    - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master # GH_TOKEN是在Travis中配置token的名称

branches:
    only:
        - hexo #只监测hexo分支，hexo是我的分支的名称，可根据自己情况设置

env:
    global:
        - GH_REF: github.com/yourname/yourname.github.io.git #设置GH_REF，注意更改yourname
```

注意：需要将配置文件中的 `GH_TOKEN` 换成我们自己设定的名称，这里我的配置应该是 `Travis_Token` 即 `- git push --force --quiet "https://${Travis_Token}@${GH_REF}" master:master # GH_TOKEN是在Travis中配置token的名称`。 还要更改 `GH_REF` 中我们的博客仓库的地址。


配置文件中的操作也很简单，这也是网上找到的比较常见的一种配置格式了。然而，这份配置文件中却隐藏着一个大坑。至于如何跳过去，后面再详说。

***

##### 实现自动部署

当 `.travis.yml` 配置文件修改完成后，将其提交到远程仓库的 `hexo` 分支下，此时如果之前的配置一切ok，我们应该能在 `Travis CI` 的博客项目主页页面中看到自动构建已经在开始执行了。上面会显示出构建过程中的日志信息及状态等。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/170809/ii4E0ca8G4.jpg?imageslim)

***

#### 遇到的问题

##### 问题一：提示 `.travis.yml` 文件格式错误

在 `Travis CI` 的日志文件中，如果遇到下面的错误提示，那可能就是 `.travis.yml` 文件的格式有问题。

```
ERROR: An error occured while trying to parse your .travis.yml file.
Please make sure that the file is valid YAML.
http://lint.travis-ci.org can check your .travis.yml.
The log message was: Build config file had a parse error: found character that cannot start any token while scanning for the next token at line 6 column 1.
```

通过在github上查询，我发现这个问题是我在配置文件中的缩进使用了 `tab` 键导致的。因为在不同的编辑器下，`tab` 键表示的宽度可能不同。

这里建议是：**不要用 `tab` 键，而是用适当的空格实现缩进**

* [found character 	&#39;\t&#39; that cannot start any token while scanning for the next token at line · Issue #136 · ruby/psych · GitHub](https://github.com/ruby/psych/issues/136)

***

##### 问题二：Travis CI的自动构建成功，但是构建完成后的项目没有推送到github中

```
...
...
git commit -m "Travis CI Auto Builder"
git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master
remote: Anonymous access to Leafney/Leafney.github.io.git denied.
fatal: Authentication failed for 'https://@github.com/Leafney/Leafney.github.io.git/'
```

查看日志提示是权限问题。

这里的问题是我在 `.travis.yml` 配置文件中没有把 `${GH_TOKEN}` 部分换成自己在 `Travis CI` 中填写的token名称而导致的。执行时找不到token，也就没法设置权限了。

***

##### 问题三：`master commit` 树被清空 ☆

如果你按照上面的 `travis.yml` 配置文件的设置去自动构建你的博客，你会发现 `master` 分支的提交记录只有当前提交的这一条，而且无论操作多少次，也仅仅只有一条。这还真的是一个大坑呀！

比如下面这位网友的站点： [GitHub - hhstore/hhstore.github.io: 个人技术博客](https://github.com/hhstore/hhstore.github.io) 在 `master` 分支下就只有一条提交记录。

***

`.travis.yml` 部分配置内容：

```
after_script:
  - cd ./public
  - git init
  - git config user.name "yourname"
  - git config user.email "your email"
  - git add .
  - git commit -m "update"
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master

```

仔细查看上面的配置文件，我们发现每次都是将 `public` 目录下的文件重新生成了一个git项目，然后强制覆盖提交到了 `master` 分支下，这就是问题的所在。

为了解决这个问题，我将配置文件改为了如下的内容：

```
after_script:
    - git clone https://${GH_REF} .deploy_git
    - cd .deploy_git
    - git checkout master
    - cd ../
    - mv .deploy_git/.git/ ./public/
    - cd ./public
    - git config user.name "yourname"
    - git config user.email "your email"
    - git add .
    - git commit -m "Travis CI Auto Builder"
    - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master
```

在 `after_script` 部分，我先将博客项目 `clone` 到本地的 `.deploy_git` 目录下（目录名可自定义）,然后切换到 `master` 分支，将 `master` 分支下的 `.git` 目录拷贝到了 `public` 目录下，接着继续后面的 `commit` 操作。

这里算是采用了一种 `换位` 的方式。之前我们通过git管理文件时并不会改动 `.git` 目录，而只是更改文件。但在这种情况下，我们需要提交的是 `public` 目录下的新文件。这样，就会保留之前的提交记录了。

***

附上我在使用的配置文件内容：

```
language: node_js # 设置语言

node_js: stable # 设置相应版本

cache:
    apt: true
    directories:
        - node_modules # 缓存不经常更改的内容

before_install:
    - npm install hexo-cli -g

install:
    - npm install # 安装hexo及插件

script:
    - hexo clean # 清除
    - hexo g # 生成

after_script:
    - git clone https://${GH_REF} .deploy_git
    - cd .deploy_git
    - git checkout master
    - cd ../
    - mv .deploy_git/.git/ ./public/
    - cd ./public
    - git config user.name "your name"
    - git config user.email "your email"
    - git add .
    - git commit -m "Travis CI Auto Builder"
    - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master

branches:
    only:
        - hexo # 只监测hexo分支

env:
    global:
        - GH_REF: github.com/yourname/yourname.github.io.git #设置GH_REF，注意更改成自己的仓库地址
```

**注意上面配置文件中的某些参数改为自己的。**

***

##### 问题四：添加 commit 时间戳

> 2017-8-23 11:25:34 Update:

按照上面的方法配置 `travis.yml` 的内容，我在一段时间后发现在 `master` 分支下的提交记录是这样的：

```
Travis CI Auto Builder

Travis CI Auto Builder

Travis CI Auto Builder

....
```

而之前在使用 `hexo d` 直接部署的时候的提交记录是这样的：

```
Site updated: 2017-06-22 22:29:10

Site updated: 2017-04-19 08:13:36

Site updated: 2017-03-27 20:54:40

...
```

看到每次的提交记录中没有提交的时间戳，感觉似乎缺少了些什么，所以考虑着要把 `commit` 的时间戳给加上。

通过查看 `travis.yml` 的文档，并没有找到如何直接获取当前时间或者和 `date` 有关的方法，但是 `script` 命令下是可以执行 `shell` 命令的，所以对 `travis.yml` 文件进行了修改。

在 `shell` 中获取当前的时间戳，可以这样：

```
#/bin/bash

> date +"%Y-%m-%d %H:%M"
2017-08-23 11:07
```

修改后的 `travis.yml` 内容：

```
language: node_js # 设置语言

node_js: stable # 设置相应版本

cache:
    apt: true
    directories:
        - node_modules # 缓存不经常更改的内容

before_install:
    - npm install hexo-cli -g
    - chmod +x ./publish-to-gh-pages.sh

install:
    - npm install # 安装hexo及插件

script:
    - hexo clean # 清除
    - hexo g # 生成

after_script:
    - ./publish-to-gh-pages.sh

branches:
    only:
        - hexo # 只监测hexo分支

env:
    global:
        - GH_REF: github.com/yourname/yourname.github.io.git #设置GH_REF，注意更改成自己的仓库地址
```

将 `after_script` 端中的命令移到了单独的shell文件中：

文件 `publish-to-gh-pages.sh` 内容：

```
#!/bin/bash
set -ev

git clone https://${GH_REF} .deploy_git
cd .deploy_git
git checkout master

cd ../
mv .deploy_git/.git/ ./public/

cd ./public

git config user.name  "your name"
git config user.email "your email"

# add commit timestamp
git add .
git commit -m "Travis CI Auto Builder at `date +"%Y-%m-%d %H:%M"`"

git push --force --quiet "https://${TravisCIToken}@${GH_REF}" master:master
```

**注意上面配置文件中的某些参数改为自己的。**

需要注意的是：命令中要为 `publish-to-gh-pages.sh` 文件赋予可执行权限，否则会报无权限错误：

```
# travis-ci log

$ ./publish-to-gh-pages.sh
/home/travis/.travis/job_stages: line 57: ./publish-to-gh-pages.sh: Permission denied
```

* [Customizing the Build - Travis CI](https://docs.travis-ci.com/user/customizing-the-build/)
* [travis ci - Permission denied for build.sh file - Stack Overflow](https://stackoverflow.com/questions/42154912/permission-denied-for-build-sh-file)

***

##### 问题五：使用 x-oauth-basic

在网上看到一位网友解决 “`master commit` 树被清空” 的问题时采用了另外一种方法，即在 `after_script` 部分调用执行 `hexo d` 命令来发布。这样的方式遇到的问题是需要设置 `SSH Key` 或者必须获得权限才能进行 `push` 操作。

有一种授权的方式是通过https使用OAuth验证的方式将token添加到url中来提交。即需要更改 `_config.yml` 中的如下部分：

```
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: git@github.com:Leafney/Leafney.github.io.git
  branch: master
```

为：

```
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://<token>:x-oauth-basic@github.com/owner/repo.git
  branch: master
```

而这样一来 token 就暴露在配置文件中了。所以还需要在操作命令中使用替换的方式只在自动部署时更改该token。

这里仅做介绍，更详细可访问：

* [使用Travis Ci使hexo自动生成并部署 | xingo&#39;s private plot](https://blog.xingoxu.com/2016/12/use-travis-ci-your-blog/)
* [Easier builds and deployments using Git over HTTPS and OAuth · GitHub](https://github.com/blog/1270-easier-builds-and-deployments-using-git-over-https-and-oauth)

***

##### 问题六：`git branch` 分支操作相关命令

```
# 查看本地所有分支(分之名称前面带*表示当前分支)
> git branch

# 查看远程所有分支
> git branch -r

# 创建分支 blog
> git branch blog

# 切换到 blog 分支
> git checkout blog

# 创建并切换到新分支
> git checkout -b blog

# 删除分支
> git branch -d blog

# 提交本地test分支作为远程的test分支
> git push origin test:test

# 合并分支(将名称为[blog]的分支与当前分支合并)
> git merge blog

# 获取远程指定分支
> git pull origin blog
```

***

#### 相关参考

* [手把手教你使用Travis CI自动部署你的Hexo博客到Github上 - 简书](http://www.jianshu.com/p/e22c13d85659)
* [使用 Travis CI 自动部署 Hexo - 简书](http://www.jianshu.com/p/5e74046e7a0f)
* [使用 Travis-CI 来自动化部署 Hexo · ZHOU](http://zhzhou.me/2017/02/20/auto-deploy-hexo-on-travis-ci/)
* [用TravisCI来做持续集成 | 进击的马斯特](http://pinkyjie.com/2016/02/27/continuous-integration-with-travis-ci/)
* [Customizing the Build - Travis CI](https://docs.travis-ci.com/user/customizing-the-build/)

***

该文章同步发表在：

* [使用Travis CI自动部署Hexo博客 - 酷小孩 - 博客园](http://www.cnblogs.com/babycool/p/7326722.html)
* [使用Travis CI自动部署Hexo博客 | IT范儿](http://www.itfanr.cc/2017/08/09/using-travis-ci-automatic-deploy-hexo-blogs/)
