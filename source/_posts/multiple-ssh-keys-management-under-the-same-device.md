---
title: 同一设备下多重SSH Keys管理
date: 2018-05-14 01:01:51
updated: 2018-05-14 01:01:51
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
* `HostName` 主机名 : 主机名可以是ip也可以是域名(如:github.com或者bitbucket.org)
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
    
将项目SSH地址： `git@111.206.223.205:8080:how_to_use_multiple_git_ssh/you_can_by_this_way.git` 更改为 `work:how_to_use_multiple_git_ssh/you_can_by_this_way.git`

第二步：`work-git-clone`

```
➜ work-git-clone work:how_to_use_multiple_git_ssh/you_can_by_this_way.git
```

而操作个人的git项目，命令只需要一步：

```
➜ git clone git@github.com:Leafney/docker-alpine-mysql.git
```

****

未完待续。。。

****

#### 相关参考

* [同一设备多个git账号的ssh管理](https://blog.kinpzz.com/2016/02/12/muti-git-ssh-management/)
* [Multiple SSH Keys settings for different github account](https://gist.github.com/jexchan/2351996)
* [Git多帐号配置](https://gist.github.com/yeungeek/596984fd9e53d6c36c0d)
* [利用SSH的用户配置文件Config管理SSH会话](https://www.hi-linux.com/posts/14346.html) --config配置项说明
* [一个客户端设置多个GitHub账号](https://www.jianshu.com/p/cd20ac5b2a3e)
