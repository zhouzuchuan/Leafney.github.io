---
title: Ubuntu系统下安装终极Shell-zsh
date: 2016-10-05 10:39:10
update: 2018-09-05 14:46:59
tags:
	- zsh
description: 如果你还在为linux系统自带的bash而苦恼，那快试试zsh吧
---

#### 查看系统 shell

终端输入 `echo $SHELL` ，可以输出当前使用的shell。

终端输入 `cat /etc/shells` ，可以输出当前系统已经安装的shell。

*** 

#### 安装 zsh

安装 `zsh` 需要 `git` 环境支持，请先确保已安装 `git` 环境：

```
$ sudo apt-get install git
```

##### 安装 zsh

```
$ sudo apt-get update
$ sudo apt-get install zsh
```

##### 安装增强插件 oh-my-zsh  

可以通过 `wget` 或者 `curl` 来安装。下面的命令是在 [Oh My Zsh](https://ohmyz.sh/) 官网中查看到的最新安装命令，建议用官网中的推荐安装方法：

```
// wget
$ sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

```
// curl
$ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

执行完上面的命令后，可能会出现两种情况。

###### 输入密码后立即生效

当提示停留在 `Password` 处要求输入当前账户的密码时，直接输入密码，等待自动配置完成即可。`oh-my-zsh` 会立即生效。

```
ubuntu@VM-46-228-ubuntu:~$ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
Cloning Oh My Zsh...
Cloning into '/home/ubuntu/.oh-my-zsh'...
remote: Counting objects: 891, done.
remote: Compressing objects: 100% (756/756), done.
remote: Total 891 (delta 20), reused 708 (delta 7), pack-reused 0
Receiving objects: 100% (891/891), 590.35 KiB | 111.00 KiB/s, done.
Resolving deltas: 100% (20/20), done.
Checking connectivity... done.
Looking for an existing zsh config...
Using the Oh My Zsh template file and adding it to ~/.zshrc
Time to change your default shell to zsh!
Password:
         __                                     __
  ____  / /_     ____ ___  __  __   ____  _____/ /_
 / __ \/ __ \   / __ `__ \/ / / /  /_  / / ___/ __ \
/ /_/ / / / /  / / / / / / /_/ /    / /_(__  ) / / /
\____/_/ /_/  /_/ /_/ /_/\__, /    /___/____/_/ /_/
                        /____/                       ....is now installed!


Please look over the ~/.zshrc file to select plugins, themes, and options.

p.s. Follow us at https://twitter.com/ohmyzsh.

p.p.s. Get stickers and t-shirts at https://shop.planetargon.com.

➜  ~
```

可以看到前面的提示符已经变了。

###### 需要重启生效

如果提示如下信息，则直接忽略即可，继续执行下面的步骤。

```
Looking for an existing zsh config...
Using the Oh My Zsh template file and adding it to ~/.zshrc
Time to change your default shell to zsh!
Password: chsh: PAM: Authentication failure
```

如果提示 `Authentication failure`，则还需要执行如下命令将zsh更改为默认shell,根据提示输入当前用户的密码,重新登录终端或重启后生效：

```
$ chsh -s /bin/zsh
//或:
$ chsh -s `which zsh`


$ sudo reboot
```

***

#### zsh 主题

zsh的默认配置项都在 `~/.zshrc` 文件中，例如里面的`ZSH_THEME="robbyrussell"` 表示当前zsh的主题为`robbyrussell`.

配置完之后，我们需要重启终端或打开新的标签，或者用以下命令刷新配置： 
```
source ~/.zshrc
```

`oh-my-zsh` 提供了数十种主题，我们可以在目录 `~/.oh-my-zsh/themes` 中看到他们：  

```
~/.oh-my-zsh/themes
```

如果你不知道选哪个好，我们可以设置成随机项：  

```
ZSH_THEME="random"
```

oh-my-zsh官方提供的主题如下：  
[Themes · robbyrussell/oh-my-zsh Wiki · GitHub](https://github.com/robbyrussell/oh-my-zsh/wiki/themes)


***

#### zsh 下的后台程序

在 `zsh` 下，如果有后台运行的程序，此时执行 `exit` 会提示如下：

```
➜  ~ exit
zsh: you have running jobs.
```

在一般的 `Bash` 下，我们设置后台运行程序用 `&`:

```
$ python cnblog.py &
```

而在 `zsh` 下，我们设置后台运行程序则需要用 `&!`:

```
$ python cnblog.py &!
```

StackOverflow上的提示：

```
Start the program with—
dolphin &!
The &! (or equivalently, &|) is a zsh-specific shortcut to both background and disown the process, such that exiting the shell will leave it running.
```

详见：[bash - Exit zsh, but leave running jobs open? - Stack Overflow](http://stackoverflow.com/questions/19302913/exit-zsh-but-leave-running-jobs-open)

*** 

#### 相关参考

* [在Ubuntu上安装zsh](http://logicmd.net/2012/11/installing-zsh-on-ubuntu/)
* [Ubuntu 上安装 zsh](https://ldsink.com/archives/install-zsh-on-ubuntu.html)
* [[Linux] ubuntu安装zsh - TangShangWen - SegmentFault](https://segmentfault.com/a/1190000003019439)
* [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)
