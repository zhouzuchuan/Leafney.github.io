---
title: 同一设备下多重SSH Keys管理
date: 2018-05-14 01:01:51
updated: 2018-07-01 23:54:38
tags:
    - GitHub
categories: 
    - Git操作系列
description: 一般我们在自己的个人电脑上使用git时都会通过SSH的方式来连接代码管理站点如Github等。但有时候我们也会遇到多重SSH的问题。该文章告诉你如何在同一个设备下快速方便的设置多重SSH Keys。
---

一般我们在自己的个人电脑上使用git时都会通过SSH的方式来连接代码管理站点如Github等。但有时候我们也会遇到多重SSH的问题。

#### 需求

最常见的情况就是一般的公司都会有自己的私有git仓库，比如用gitlab来搭建的git仓库站点。而公司里也会为每位员工设置一个个人专属的公司域名邮箱，这样的情况下就遇到问题了：我们在自己的电脑上已经配置了自己的邮箱账号作为主力的git账户，而在开发公司的项目时，用来提交git操作的时候就要切换为公司分配的邮箱账号了。虽然我们可以在项目目录内使用 `git config user.name` 和 `git config user.email` 来为该项目指定单独的git提交账户，但在 `git pull` 和 `git push` 时，每次都需要使用 `http` 的方式输入密码才行，非常的麻烦。

#### 准备

那么现在问题就来了，个人使用的git账户和公司分配的git账户应该如何在一台电脑上同时存在并保证都能分别使用SSH的方式来提交代码呢？

首先，我们指定下面示例使用的git账户：

我的个人git账户: `leafney` `leafney@gmail.com` git仓库：`github.com`
我的公司git账户: `wuyazi` `wuyazi@company.com` git仓库：`111.206.223.205:8080` （公司配置的gitlab对应ip地址）
我使用的Bash : `iTerm2+zsh`

我的需求是，因为我使用的是个人电脑，所以我希望的是在默认情况下，仍然使用我自己的git账户 `leafney@gmail.com` 作为全局账户来提交项目。对于公司的项目，则使用公司的git账户 `wuyazi@company.com` 作为局部账户来仅提交公司的项目。

之前在我的mac下已经设置了默认的git账号 `leafney@gmail.com` 对应的ssh密钥 。如果你还没有设置过SSH，可以参考我之前的文章：[Linux下使用SSH密钥连接Github
](/2017/03/03/using-ssh-key-connection-github-in-linux/)


#### 新增SSH key

现在要新增一个工作的账号，以 `wuyazi` `wuyazi@company.com` 为示例账户来添加。

##### 查看已添加SSH密钥

查看mac下已经添加的我的个人git账号信息:

```
~/.ssh
➜ ls
id_rsa      id_rsa.pub  known_hosts
```

查看 `id_rsa.pub` 内容:

```
➜ vim id_rsa.pub
ssh-ras xxxxxxxxxxxx  leafney@gmail.com
```

***

##### 新增SSH密钥

创建新的 ssh key：

```
// 切换到 .ssh 目录下
➜ cd ~/.ssh
// 输入工作邮箱来创建新的ssh key,git唯一认证标准是邮箱
➜ ssh-keygen -t rsa -C "wuyazi@company.com"
```

需要注意的一点是，当提示 `Enter file in which to save the key (/Users/leafney/.ssh/id_rsa): ` 时，我们不使用默认的名称 `id_rsa` ，因为该名称我们之前在设置个人git账户时已经使用过了，所以这里新设置一个针对于公司git账户的名称 `id_rsa_work`。

操作记录：

```
~/.ssh
➜ ssh-keygen -t rsa -C "wuyazi@company.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/leafney/.ssh/id_rsa): id_rsa_work
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in id_rsa_work.
Your public key has been saved in id_rsa_work.pub.
The key fingerprint is:
SHA256:gZO6tyeOrp0FLGT5q2kBn3+9Et+P1shzjuyv3rUXWY0 wuyazi@company.com
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|       o         |
|.     + .      . |
| o o .   .    E .|
|  B .   S    o . |
| o = ...    o    |
|  . B oo.o o ..  |
|   = Ao.oo*++. . |
|  +=B.o+.+BB=..  |
+----[SHA256]-----+

~/.ssh took 22s
```

##### 将密钥添加到ssh agent管理

通过ssh agent对密钥的管理我们可以实现免密码提交。使用 `ssh-add` 命令来将新增的密钥添加到 `ssh-agent` 的高速缓存中：

```
~/.ssh

//添加之前所新生成的用于work的密钥
➜ ssh-add ~/.ssh/id_rsa_work
Identity added: /Users/leafney/.ssh/id_rsa_work (/Users/leafney/.ssh/id_rsa_work)
```

可以通过命令 `ssh-add -l` 来查看存在于 `ssh-agent` 密钥管理器中所有的密钥列表：

```
~/.ssh
➜ ssh-add -l
2048 SHA256:gZO6tyeOrp0FLGT5q2kBn3+9Et+P1shzjuyv3rUXWY0 /Users/leafney/.ssh/id_rsa_work (RSA)
2048 SHA256:My6EL2r7d20oq01x1PeuRiYJSK2aID3sQlR+xCagPnE /Users/leafney/.ssh/id_rsa (RSA)
```

这里如果发现之前的 `id_rsa` 的密钥不存在，可以重新添加一下。

如果要删除所有已添加的密钥，可以使用命令：`ssh-add -D` 。

#### 配置ssh config

##### 编辑config

`SSH` 程序可以从以下途径获取配置参数： 

用户配置文件：`~/.ssh/config`
系统配置文件：`/etc/ssh/ssh_config`

配置文件可分为多个配置区段，每个配置区段使用 `Host` 来区分。我们可以在命令行中输入不同的Host来加载不同的配置段。

在 `~/.ssh` 目录下打开配置文件 `config`(没有则新增该文件)，如下是我的配置示例：

新增：

```
➜ cd ~/.ssh/
➜ touch config
```

`config` 内容:

```
Host work
    HostName 111.206.223.205:8080
    User git
    IdentityFile ~/.ssh/id_rsa_work

Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa
```

##### 配置项说明

常用的SSH配置项：

* `Host` 别名 : `Host myhost` 表示自定义的host简称，以后连接远程服务器就可以使用命令 `ssh myhost` (注意下面有缩进)
* `HostName` 主机名 : 主机名可以是ip也可以是域名(如:github.com或者bitbucket.org);如果主机名中包含 `%h` ，则实际使用时会被命令行中的主机名替换。
* `Port` 端口 : 表示服务器open-ssh端口，默认22，默认时一般不写此行
* `User` 用户名 : `User git` 表示登录ssh的用户名，如 `git`
* `IdentityFile` ： 表示证书文件路径（如 `~/.ssh/id_rsa_*`)

其他SSH配置项：

* `IdentitiesOnly` 只接受 SSH Key 登录  可选 `yes` or `no`
* `PreferredAuthentications` 强制使用Public Key验证


##### 测试

如上，我用别名 `work` 指代了公司gitlab的ip地址 `111.206.223.205:8080` ，使用 `ssh -T` 命令来验证一下：

```
➜ ssh -T git@work
The authenticity of host '111.206.223.205:8080 (111.206.223.205:8080)' can't be established.
RSA key fingerprint is SHA256:/UwiMSDZVkgF+3yLAxHwMJfYGxK2XGk5DU3txRr+LNg.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '111.206.223.205:8080' (RSA) to the list of known hosts.
Welcome to GitLab, 无崖子!
```

****

#### 第一版示例

##### 项目clone

如上的步骤配置完成后，我们就可以通过ssh别名的方式来区分使用不同的SSH Key了。

在 `clone` 项目到本地之前，我们需要进行一个小小的改动。

之前的clone命令：

```
➜ git clone git@111.206.223.205:8080:how_to_use_multiple_git_ssh/you_can_by_this_way.git
```

更改后的clone命令：

```
➜ git clone git@work:how_to_use_multiple_git_ssh/you_can_by_this_way.git
```

即我们将原来ssh地址中的域名用我们在配置文件中的别名来代替了，而相应的别名又对应着ssh密钥文件。

当然，我们还可以更精简一下，因为我们在`config` 文件中已经指定了用户为 `git` ，所以可将路径中的用户名也省略掉：

```
➜ git clone work:how_to_use_multiple_git_ssh/you_can_by_this_way.git
```

这样，看起来是不是非常的舒服呢？

而如果我要clone自己的个人项目，则仍然按照之前的操作即可：

```
➜ git clone git@github.com:Leafney/docker-alpine-mysql.git
```

##### 设置项目提交账户

在将项目 `clone` 到本地之后，还要为该项目单独设置commit提交时使用的git账户。

因为我们在之前已经按照 [Linux下使用SSH密钥连接Github
](/2017/03/03/using-ssh-key-connection-github-in-linux/) 操作中的方法将我的个人账户 `leafney` `leafney@gmail.com` 设置为了全局的git账户，在提交项目时如果没有设置项目单独的git账户，就会使用全局的，那这样就不对了。

不加 `--global` 参数，为刚刚clone下来的公司项目设置公司分配的git账户：

```
➜ git config user.name "wuyazi"   # 双引号是为了防止name中含有空格而导致错误
➜ git config user.email "wuyazi@company.com"
```

这样就配置完成了。

****

#### 还能再快一点吗

总结上面的操作步骤，无非是设置了如下两项：

1. 指定git项目使用的SSH Key
2. 指定git项目提交时使用的git账户

那么第一步我们已经在ssh的 `config` 中做了指定，貌似已经是最快的操作了。那就看看第二项如何优化呢？

我们是在git项目clone到本地之后，再去为该项目添加了使用的git账户，那是否能将这两步合并为一步呢？

**答案当然是可以的。**

#### 设置Base/Zsh方便clone操作

为了以后方便使用不同git账户 `git clone` 项目，我们在 `bash` 的 `~/.bashrc` 或者 `zsh` 的 `~/.zshrc` 内加入：

```
alias work-git-clone='git clone --config user.name="wuyazi" --config user.email="wuyazi@company.com" $@'
```

以后就只要输入 `work-git-clone REPO-SSH-URL` ，效果就相当于执行了如下的命令:

```
git clone REPO-SSH-URL
cd REPO
git config user.name "wuyazi"
git config user.email "wuyazi@company.com"
```

然后，更新 `Bash/Zsh` 的设置：

```
# 更新Bash设置
➜ source ~/.bashrc

# 更新zsh设置
➜ source ~/.zshrc
```

参考自；[多重 SSH Keys 與 Github 帳號](https://kuanyui.github.io/2016/08/01/git-multiple-ssh-key/)

#### 第二版示例

经过上面的操作，我们在 `git clone` 公司的git项目时，只需要执行如下简单的两步即可：

第一步：更改SSH地址

```
# 将项目SSH地址：
git@111.206.223.205:8080:how_to_use_multiple_git_ssh/you_can_by_this_way.git
# 更改为:
work:how_to_use_multiple_git_ssh/you_can_by_this_way.git
```

第二步：执行 `work-git-clone`

```
➜ work-git-clone work:how_to_use_multiple_git_ssh/you_can_by_this_way.git
```

而操作个人的git项目，命令只需要一步：

```
➜ git clone git@github.com:Leafney/docker-alpine-mysql.git
```

****

#### 新的复杂需求

上面的需求可能是相对来说比较常见而且普通的一种情况了，那么下面的需求算是一种稍微复杂的情况了。

因为在Github上发布私有的项目，是需要付费的，所以一般针对于个人的私有项目，我们一般会选择自己购买服务器来搭建属于自己的私有个人仓库，比如我自己的 `gogit.itfanr.cc` 就是我用开源项目 `Gogs` 来搭建的。有兴趣可查看我的开源项目：[Docker Ubuntu-Gogs 用更简单的方式部署、升级或迁移Gogs服务。](https://github.com/Leafney/ubuntu-gogs) 。

那么，现在就出现了下面的四种情况：

1. 对于我个人想要开源的代码项目，我会选择使用个人账户发布到 `github.com` 站点下；
2. 对于我个人的私有代码项目，我会选择使用个人账户发布到我自己搭建的 `gogit.itfanr.cc` 仓库站点下；
3. 对于公司的私有项目，我要选择使用公司分配的git账户发布到公司搭建的私有仓库 `111.206.223.205:8080` 站点下；
4. 对于我在工作中的一些积累或私人的项目，我想使用公司分配的git账户发布到我个人的 `gogit.itfanr.cc` 仓库站点下；

##### 修改ssh config

经过上面的步骤，我们很容易的就知道想要实现上面的需求，只需要通过修改SSH的配置文件 `config` 中的配置即可实现。

对于公司的项目，针对于上面第3个情况，我们仍然如之前的配置即可：

`config` :

```
# company repo account
Host work
    HostName 111.206.223.205:8080
    User git
    IdentityFile ~/.ssh/id_rsa_work
```

那么在clone项目时的操作如下：

1. 更改项目SSH地址中的域名部分；

2. 执行命令：

```
➜ work-git-clone work:how_to_use_multiple_git_ssh/you_can_by_this_way.git
```

****

对于工作中的私人项目，我们要使用公司的git账户来发布到我的个人仓库站点 `gogit.itfanr.cc`。配置时我们还需要指定使用密钥 `id_rsa_work` ：

`config` :

```
# wuyazi private repo account
Host wuyazigogit
    HostName gogit.itfanr.cc
    User git
    IdentityFile ~/.ssh/id_rsa_work
```

那么在clone项目时的操作如下：

1. 更改项目SSH地址中的域名部分；

```
# 如将: 
ssh://git@gogit.itfanr.cc:9527/wuyazi/repo_for_myself.git
# 更改为:
ssh://git@wuyazigogit:9527/wuyazi/repo_for_myself.git
```

2. 执行命令：

```
work-git-clone ssh://git@wuyazigogit:9527/wuyazi/repo_for_myself.git
```

****

因为我的个人账户是作为全局账户来使用的，就是说在 `config` 文件中如果上面的 `Host` 部分没有匹配上，那么要保证最后一个 `Host` 匹配到我的个人git账户。再次回顾 `config` 配置项中的常用参数，我们发现 `HostName` 除了可以设置域名或ip地址之外，还可以设置为 `%h` ，表示**实际使用时会被命令行中的主机名替换**。

所以，为了实现上面的第1种和第2种情况，以及实现对未特殊说明的项目的默认匹配，我用如下的方式来配置：

```
# leafney default account for github.com or gogit.itfanr.cc
Host github.com gogit.itfanr.cc
    HostName %h
    User git
    IdentityFile ~/.ssh/id_rsa
```

如上，针对于不同主机地址使用同一私钥进行登录的情况，可以在 `Host` 中指定多个别名来匹配，而 `HostName` 中的 `%h` 会自动匹配用户输入的ssh地址中的域名部分，来匹配到对应的密钥。

那么在clone项目时的操作如下：

```
git clone git@github.com:Leafney/ubuntu-gogs.git
```

综上，我的 `config` 配置内容如下：

```
# company repo account
Host work
    HostName 111.206.223.205:8080
    User git
    IdentityFile ~/.ssh/id_rsa_work

# wuyazi private repo account
Host wuyazigogit
    HostName gogit.itfanr.cc
    User git
    IdentityFile ~/.ssh/id_rsa_work

# leafney default account for github.com or gogit.itfanr.cc
Host github.com gogit.itfanr.cc
    HostName %h
    User git
    IdentityFile ~/.ssh/id_rsa
```

****

#### 新增项目配置

上面的操作中说到的方法一般适用的场景比如你去到一家新公司，然后会给你分配公司的邮箱及git账号，以及公司项目的私有git站点，直接从站点上面 `clone` 项目到你的开发电脑上，这些项目一般都是公司已有的项目了。

某些情况下，可能需要你新增一个公司的项目，那对于新增公司的项目时，我们要如何使用指定的SSH账户来操作呢？

一般的步骤如下：

##### 初始化项目

在你的开发电脑上，新增一个git管理的项目。在项目目录下使用 `git init` 来初始化。

##### 设置项目git账户

注意，这里就是最关键的一步了。我们要为该新增的项目创建针对于该项目的git账户。在上面的流程中也提到过，要使用不加 `--global` 的命令来设置：

```
git config user.name "wuyazi"
git config user.email "wuyazi@company.com"
```

##### 创建远端项目

然后就是在公司的私有git站点上创建一个空项目。一般在创建完成后都会有类似于下面的一个提示页面：

```
从命令行创建一个新的仓库
touch README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin ssh://git@111.206.223.205:8080:how_to_use_multiple_git_ssh/new.git
git push -u origin master

从命令行推送已经创建的仓库
git remote add origin ssh://git@111.206.223.205:8080:how_to_use_multiple_git_ssh/new.git
git push -u origin master
```

##### 提交到远端仓库

对于只有一个SSH密钥的情况下，我们要将本地项目提交到远端，只要执行页面中后面这两句就可以了。而现在我们要将本地新增的公司项目使用我的公司git账户提交到公司的私有git站点上，首先呢还是需要更改一下提交的项目地址：

```
# 将项目SSH地址：
ssh://git@111.206.223.205:8080:how_to_use_multiple_git_ssh/new.git
# 更改为:
ssh://work:how_to_use_multiple_git_ssh/new.git
```

然后我们就可以执行上面两条命令了：

```
git remote add origin ssh://work:how_to_use_multiple_git_ssh/new.git
git push -u origin master
```

##### 有没有简洁命令

可能有的朋友会想了，在之前 `clone` 已有项目时，我们使用了一条简洁的命令： `work-git-clone work:how_to_use_multiple_git_ssh/you_can_by_this_way.git` 来直接省略了单独设置git操作账户的步骤，那在新增时，是不是也可以有类似的操作呢？

答案是否定的。

因为 `git clone` 命令是可以接收 `--config` 参数的，以便在clone的同时指定配置；而 `git push` 命令却是没有该项的。具体的可以通过命令 `git clone help` 和 `git push help` 来详细了解。

至此，后面的操作就是常用的 `pull` 和 `push` 等操作了。

***

#### 相关参考

* [同一设备多个git账号的ssh管理](https://blog.kinpzz.com/2016/02/12/muti-git-ssh-management/)
* [Git多帐号配置
](https://gist.github.com/yeungeek/596984fd9e53d6c36c0d)
* [利用SSH的用户配置文件Config管理SSH会话](https://www.hi-linux.com/posts/14346.html)
* [多重 SSH Keys 與 Github 帳號](https://kuanyui.github.io/2016/08/01/git-multiple-ssh-key/)

