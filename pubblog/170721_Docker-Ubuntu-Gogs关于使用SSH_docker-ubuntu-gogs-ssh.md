### Docker-Ubuntu-Gogs关于使用SSH

#### SSH key passphrases

在本地电脑上(我这里是Windows系统)创建 `SSH key` 时，会要求你为key设置一个密码：

```
λ ssh-keygen -t rsa -C "xxxxx@qq.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/You/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/You/.ssh/id_rsa.
Your public key has been saved in /c/Users/You/.ssh/id_rsa.pub.
The key fingerprint is:
...
```

即其中的：

```
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

一般情况下我们可以选择直接回车即不设置密码。

如果没有设置key的密码，我们在通过SSH提交代码时，可以直接操作。如果设置了key的密码，那我们在每次 `pull` 或 `push` 时都会要求你输入key的密码：

```
$ git pull
Enter passphrase for key '/c/Users/You/.ssh/id_rsa':

```

***

#### Docker-Ubuntu-Gogs容器使用SSH提交时要求输入git密码

在使用SSH获取或提交代码时，偶尔会遇到要求输入 `git` 密码的情况：

```
$ git clone ssh://git@gogit.itfanr.cc/haha/wohaha.git
Cloning into 'wohaha'...
git@gogit.itfanr.cc's password:
Permission denied, please try again.
git@gogit.itfanr.cc's password:
Permission denied, please try again.
git@gogit.itfanr.cc's password:
Permission denied (publickey,password).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

或者：

```
$ git pull
git@gogit.itfanr.cc's password:
Permission denied, please try again.
git@gogit.itfanr.cc's password:
Permission denied, please try again.
git@gogit.itfanr.cc's password:
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

```

但是我们通过Docker创建的gogs容器中，并没有为git账户设置密码，所以这里无论输入什么都是错误的。

通过查看 [Issues · gogits/gogs · GitHub](https://github.com/gogits/gogs/issues) 中相关的 `issues` 可以了解到，但凡是要求输入git密码的问题，十有八九是 **关于 `.ssh/authorize_keys` 文件的权限** 问题。

gogs的文档中要求：

* `.ssh/` 目录权限为 `0700`
* `.ssh/authorize_keys` 文件的权限为 `0600`

相关的操作命令为：

```
$ chmod 0700 /home/git/.ssh
$ chmod 0600 /home/git/.ssh/*
```

***

##### 方法一

登陆gogs站点的管理员账户，选择 `管理面板` 访问 `/admin` 页面。在 `管理员操作` 区域选择 `重新生成 '.ssh/authorized_keys' 文件（警告：不是 Gogs 的密钥也会被删除）` 点击 `执行` 按钮。

然后再次尝试看是否能够成功操作。

##### 方法二

如果 `方法一` 的操作无效，那么我们需要登陆该gogs容器所在的服务器来进入如下操作：

1. 进入该gogs容器，然后删除 `.ssh/authorized_keys` 文件 。
2. 重复方法一的操作：登陆管理员账户，选择 `管理面板` -- `管理员操作` -- 点击 `重新生成 '.ssh/authorized_keys' 文件（警告：不是 Gogs 的密钥也会被删除）` 后的 `执行` 按钮。

然后再次尝试看是否能够成功操作。

我的操作记录：

```
$ docker exec -it gogs /bin/bash
root@564c5628c7e9:/home/git/gogs# cd ..
root@564c5628c7e9:/home/git# cd .ssh/
root@564c5628c7e9:/home/git/.ssh# ls
authorized_keys
root@564c5628c7e9:/home/git/.ssh# rm authorized_keys 
root@564c5628c7e9:/home/git/.ssh# ls

# 此时在管理后台重新生成 authorized_keys
root@564c5628c7e9:/home/git/.ssh# ls
authorized_keys
root@564c5628c7e9:/home/git/.ssh# ls -al
total 12
drwx------ 2 git git 4096 Jul 21 14:20 .
drwxr-xr-x 6 git git 4096 Jul 21 11:32 ..
-rw------- 1 git git  549 Jul 21 14:20 authorized_keys
```

***

我在 `git push` 时遇到要求输入git密码的问题时，先将 `authorized_keys` 文件删除，然后重新生成，这样操作后就能正常获取和提交了。


#### 相关参考

* [ssh 的链接地址不可以使用 · Issue #545 · gogits/gogs · GitHub](https://github.com/gogits/gogs/issues/545)
