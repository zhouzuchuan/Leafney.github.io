---
title: Docker-Ubuntu-Gogs部署及配置时遇到的问题
date: 2017-03-24 14:19:37
tags: 
    - Gogs
    - Docker
description: Docker Gogs 用更简单的方式部署、升级或迁移Gogs容器服务。 这里主要记录在Docker下部署Gogs项目过程中遇到的问题及解决方法。
---

Docker Gogs 用更简单的方式部署、升级或迁移Gogs容器服务。

这里主要记录在Docker下部署Gogs项目过程中遇到的问题及解决方法。

#### 目录

1. Install页面中的注意问题
2. `Connection timed out` 问题
3. `hooks/update: No such file or directory`
4. 添加 `SSH key` 时显示报错页面 `500`
5. 要求输入git账户密码
6. `SSH Connection refused` 问题
7. `Git SSH` 使用非默认22端口时，如何隐藏端口号
8. `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`
9. echo 输出
10. Docker Gogs 使用 `HTTPS`


#### Install页面中的注意问题

* domain: ensure that is the IP address of your docker host
* SSH port: that is the forwarded ssh port. So if you forward 22 to 10022, it's 10022.
* HTTP port: my error, here, you must enter the HTTP in the container. So it's 3000. Even if you forward it to 10080 ;)
* Application URL: this is a combination of domain + forwarded HTTP port (I got this one wrong too). So, if you're forwarding 3000 to 10080, then it's `http://domain:10080`.
* ssh端口使用外部端口，而http端口使用的是容器内部端口.

着重需要说明的是：

* `Domain` 填写Docker宿主机的物理IP地址，或者域名地址,注意这里是不带 `http`的 如： `192.168.137.140` 或 `git.mydomain.com`
* `SSH port` 假如Docker映射的端口是 `10022:22` 那么这里就填写宿主机开放的端口 `10022`
* `HTTP port` 假如Docker映射的端口是 `10080:3000` 这里要填容器内的监听端口 `3000`
* `Application URL` 这里要填写的格式为 `http(s):// + Domain + HTTP port` ，比如：`http://git.mydomain.com/10080` 。还需要注意的一点是，如果你用了nginx来映射宿主机的 `10080` 端口，这里要去掉后面的端口，即 `http://git.mydomain.com/`，说白了就是你在外部浏览器上访问的地址。

* [Docker gogs web/ssh does not restart after reboot · Issue #3039 · gogits/gogs · GitHub](https://github.com/gogits/gogs/issues/3039)

***

#### `Connection timed out` 问题

##### 使用HTTP方式 Push 代码时报错？

使用HTTP方式获取：

```
$ git clone http://gogit.itfanr.cc/xueer/HelloWorld.git
Cloning into 'HelloWorld'...
remote: Counting objects: 8, done.
remote: Compressing objects: 100% (6/6), done.
remote: Total 8 (delta 1), reused 0 (delta 0)
Unpacking objects: 100% (8/8), done.
Checking connectivity... done.
```

使用HTTP方式提交：

```
$ git push
Counting objects: 3, done.
Writing objects: 100% (3/3), 379 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
error: RPC failed; HTTP 504 curl 22 The requested URL returned error: 504 Gateway Time-out
fatal: The remote end hung up unexpectedly
fatal: The remote end hung up unexpectedly
Everything up-to-date
```

***

##### 使用SSH方式获取代码时报错？

使用SSH方式获取：

```
$ git clone ssh://git@gogit.itfanr.cc:10022/xueer/HelloWorld.git
Cloning into 'HelloWorld'...
ssh: connect to host gogit.itfanr.cc port 10022: Connection timed out
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```


对于 `Connection timed out` 的问题，可能是使用的端口已经被其他程序占用导致，也可能是缓存文件导致。

最后的解决方法是将 `data/sessions` 中的 `session` 文件删除，然后更换了其他端口后再次运行则能够提交数据了。

***

#### 报错 `hooks/update: No such file or directory`

将旧的配置和数据库拷贝到新的Gogs项目下（我这里是从v0.8.25.0129版本迁移到v0.10.18.0313版本），提交代码时会报如下错误：

```
$ git push origin master
Counting objects: 3, done.
Writing objects: 100% (3/3), 432 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote: hooks/update: line 2: /home/git/gogs/goapp/gogs/gogs: No such file or directory
remote: error: hook declined to update refs/heads/master
To http://gogit.itfanr.cc/xueer/HelloWorld.git
 ! [remote rejected] master -> master (hook declined)
error: failed to push some refs to 'http://gogit.itfanr.cc/xueer/HelloWorld.git'
```

***

解决方法：

根据Gogs官网中的`故障排查`的说明：

```
Update 钩子指向错误的二进制路径

可能原因：您升级 `Gogs` 后将其移动到了和之前安装位置不同的目录
解决方案：到管理员控制面板（`/admin`）执行以下操作：
重新生成 '.ssh/authorized_keys' 文件
重新同步所有仓库的 `pre-receive`、`update` 和 `post-receive` 钩子
```

所以直接使用管理员账户在管理后台中操作即可解决。

详见：[故障排查](https://gogs.io/docs/intro/troubleshooting#update-%E9%92%A9%E5%AD%90%E6%8C%87%E5%90%91%E9%94%99%E8%AF%AF%E7%9A%84%E4%BA%8C%E8%BF%9B%E5%88%B6%E8%B7%AF%E5%BE%84)

***

之前的解决方法：

在新的Gogs位置执行 `gogs fix location <old Gogs path>`

经测试，该方法似乎已过时，执行时会报如下错误：

```
/home/git/gogs# ./gogs fix location /home/git/gogs/goapp/gogs
No help topic for 'fix'
```

* [remote: hooks/update: line 2: /path/to/dir: No such file or directory · Issue #659 · gogits/gogs · GitHub](https://github.com/gogits/gogs/issues/659)
* [Question: How do I move servers? · Issue #654 · gogits/gogs · GitHub](https://github.com/gogits/gogs/issues/654)

***

#### 添加 SSH key 时显示报错页面 `error 500`

查看日志，找到如下错误信息：

```
AddPublicKey: addKey: open /home/git/.ssh/authorized_keys: permission denied
```

按照如下说法操作成功：

> So I changed the .ssh/ folder to 0700 and .ssh/authorize_keys to 0600. It works.

详细来源见：[ssh 的链接地址不可以使用](https://github.com/gogits/gogs/issues/545)


修改 `/home/git/.ssh` 的权限为 `700`   
修改 `/home/git/.ssh/authorized_keys` 的权限为 `600`   


我的操作：

1. 登陆Gogs站点管理员账户，访问 `/admin` 页面，选择 `Rewrite '.ssh/authorized_keys' file (caution: non-Gogs keys will be lost)` 项 ，点击 `Run` 报错：`open /home/git/.ssh/authorized_keys.tmp: permission denied` 。
2. 查看日志 `gogs.log` 文件，发现错误信息：`AddPublicKey: addKey: open /home/git/.ssh/authorized_keys: permission denied` 。
3. 进入容器内部：`docker exec -it gogs /bin/bash` ,切换至git用户：`su git`,然后进入 `/home/git/` 目录，更改 `.ssh` 目录权限：`chmod 0700 .ssh` 。（此时，该 .ssh 目录内为空）
4. 然后再次访问 `/admin` 页面，再次执行 `Run` 操作，提示“所有公钥重新生成成功！”信息。查看容器内的 `.ssh/` 目录下生成了一个 `authorized_keys` 文件。然后更改该文件权限：`chmod 0600 authorized_keys` 。
5. 然后重启该容器。

重启容器后该问题解决。

***

#### 要求输入git账户密码

添加成功SSH密钥后，`git clone` 项目会报如下错误：

```
$ git clone ssh://git@gogit.itfanr.cc:10022/xueer/HelloWorld.git
Cloning into 'Hello'...
The authenticity of host '[gogit.itfanr.cc]:10022 ([gogit.itfanr.cc]:10022)' can't be established.
ECDSA key fingerprint is SHA256:+LHU3V00S0g9kNnpByc9ysAM5n6DWutT51YOldIcf88.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[gogit.itfanr.cc]:10022' (ECDSA) to the list of known hosts.
git@gogit.itfanr.cc's password:
Permission denied, please try again.
git@gogit.itfanr.cc's password:
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

一种可能的解决方式是 `Check if /home/git/.ssh/.authorized_keys's permission is 600.`


这个问题还是关于 `.ssh/authorize_keys` 的权限问题。

我的解决方法：

1.  登陆Gogs站点管理员账户，访问 `/admin` 页面，选择 `Rewrite '.ssh/authorized_keys' file (caution: non-Gogs keys will be lost)` 项 ，点击 `Run` ，会提示更新成功。
2. 重启该容器即可。

2017-7-21 add:

如果以上方法无效，可以进入容器后将 `authorize_keys` 文件删除，然后在管理员操作页面中重新生成。这样应该能解决。

***

* [ssh 的链接地址不可以使用 · Issue #545 · gogits/gogs · GitHub](https://github.com/gogits/gogs/issues/545)
* [SSH prompts for password in Docker · Issue #2409 · gogits/gogs · GitHub](https://github.com/gogits/gogs/issues/2409)
* [How troubleshoot SSHd on Docker ?](https://github.com/gogits/gogs/issues/1893)

***

#### `SSH Connection refused` 问题


添加成功SSH密钥后，`git clone` 项目会报如下错误：

```
$ git clone ssh://git@192.168.137.140:10022/xueer/HelloWorld.git
Cloning into 'HelloWorld'...
ssh: connect to host 192.168.137.140 port 10022: Connection refused
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

我的探索：

进入容器内部：`docker exec -it gogs /bin/bash`

查看ssh连接：

```
root@b48dfa9583f9:/home/git/gogs# ssh localhost
ssh: connect to host localhost port 22: Connection refused
```

那现在这个问题就是在ubuntu下配置ssh链接的问题了。  

发现问题的原因是安装 `openssh` 后，并没有启动 `SSH` 服务。执行如下命令启动：

```bash
/etc/init.d/ssh start
# 或
service ssh start
service ssh status
```

通过如下命令查看ssh服务是否启动：`ps -e |grep ssh`

```
root@b48dfa9583f9:/home/git/gogs# ps -e |grep ssh
   81 ?        00:00:00 sshd
```

然后使用 `ssh localhost` 命令查看是否能够连接本地：

```
root@b48dfa9583f9:/home/git/gogs# ssh localhost
The authenticity of host 'localhost (::1)' can't be established.
ECDSA key fingerprint is SHA256:eKk78mnEXpwvFhbzC6BgM70jZx3be4Fz8okyagHA6QA.
Are you sure you want to continue connecting (yes/no)? no
Host key verification failed.
```

可以看到SSH服务已经正常运行了。

然后在回到宿主机下，再次执行 `git clone` 命令，测试是否能够连通。

***

这里我整理了两种连接方法：

连接方法一：

`git clone ssh://git@192.168.137.140:10022/qqq/Xweixin.git`   Pull and Push Test Ok.

连接方法二：

如果嫌上面带有端口号的链接太不极客范儿，可以在客户端电脑上当前用户主目录的 `.ssh` 目录下创建一个没有扩展名的 `config` 文件（注意文件格式为 `UTF-8` 且行结束标识为 `Unix(LF)`），填写如下内容：

`~/.ssh/config`:

```
Host 192.168.137.140
HostName 192.168.137.140
Port 10022
User git
```

这里，

* `Host` 是远程git仓库所在宿主机的IP地址或域名；
* `HostName` 不太重要，仅起到标识的作用；
* `Port` 表示远程git仓库使用的端口号；
* `User` 表示远程git仓库的默认用户，这里填写默认的 `git` 即可。

然后就可以直接请求不带端口号的ssh仓库地址了：

`git clone ssh://git@192.168.137.140/qqq/Xweixin.git` Pull and Push Test Ok.


相关参考

* [ssh: connect to host localhost port 22: Connection refused 问题](http://blog.csdn.net/jszhangyili/article/details/8881807)
* [ubuntu下如何安装使用SSH？](http://os.51cto.com/art/201109/291634.htm)
* [How to Enable SSH in Ubuntu 16.04 LTS | UbuntuHandbook](http://ubuntuhandbook.org/index.php/2016/04/enable-ssh-ubuntu-16-04-lts/)
* [SSH doesn't work on ports other than 22 in dockerized gogs](https://github.com/gogits/gogs/issues/1788)

***

#### Git SSH 使用非默认22端口时，如何隐藏端口号

Note that you can also add an entry to your `~/.ssh/config` file:

```
Host git.example.com
Port 2222
User git
```

and then use the normal git clone `git@git.example.com:myuser/myproject.git` command.

相关参考

* [git - Using a remote repository with non-standard port - Stack Overflow](http://stackoverflow.com/questions/1558719/using-a-remote-repository-with-non-standard-port)
* [Change port git is using for ssh](https://prestongarrison.com/change-port-git-is-using-for-ssh/)
* [Specify SSH Port for Git - Server Fault](http://serverfault.com/questions/218256/specify-ssh-port-for-git)

***

#### 报错 `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`

使用 SSH 获取时报错：

```
$ git clone ssh://git@192.168.137.140:10022/xueer/HelloWorld.git
Cloning into 'HelloWorld'...
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:+LHU3V00S0g9kNnpByc9ysAM5n6DWutT51YOldIcf88.
Please contact your system administrator.
Add correct host key in /c/Users/Yx/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /c/Users/Yx/.ssh/known_hosts:5
ECDSA host key for [192.168.137.140]:10022 has changed and you have requested strict checking.
Host key verification failed.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

该问题是由于客户端电脑上的 `.ssh/known_hosts` 文件中记录了重复或冲突的ssh信息，删除该文件重新设置ssh连接即解决。

***

#### echo 输出

下面两行命令的输出结果是不一样的：

```
ab=$(service ssh status);echo $ab;

ab=$(service ssh status);echo "$ab";
```

```
you can get output of third solution in good way:

echo "$var"
and also in nasty way:

echo $var
```

当 `echo` 后面的内容带有引号时，只会输出变量值的内容。当没有引号时，如果当前所在目录下有其他文件，这些文件的文件名也会被输出：

```
ab=$(service ssh status);echo $ab;
LICENSE README.md README_ZH.md custom data gogs log public scripts templates sshd is running
```

* [linux - How to set a variable to the output from a command in Bash? - Stack Overflow](http://stackoverflow.com/questions/4651437/how-to-set-a-variable-to-the-output-from-a-command-in-bash)

***

#### Gogs 使用 HTTPS 

待完善。

* [使用 HTTPS 部署 Gogs · Issue #12 · Unknwon/wuwen.org · GitHub](https://github.com/Unknwon/wuwen.org/issues/12)
* [gogsi/gogs-nginx-ssl.conf at master · richardskumat/gogsi · GitHub](https://github.com/richardskumat/gogsi/blob/master/gogs-nginx-ssl.conf)

***

#### One More Thing...

根据以上遇到的问题，这里主要提一点：

但凡是SSH相关的问题，就是要保证目录 `/home/git/.ssh` 具有 `0700` 的权限，目录 `/home/git/.ssh/` 下的文件具有 `0600` 的权限。

或者直接尝试如下操作:

1. 登陆Gogs站点管理员账户，访问 `/admin` 页面，选择 `Rewrite '.ssh/authorized_keys' file (caution: non-Gogs keys will be lost)` 项 ，点击 `Run` 按钮，如果报错：`open /home/git/.ssh/authorized_keys.tmp: permission denied` 。否则直接跳至步骤5。
2. 查看日志 `gogs.log` 文件，发现错误信息：`AddPublicKey: addKey: open /home/git/.ssh/authorized_keys: permission denied` 。
3. 进入容器内部：`docker exec -it gogs /bin/bash` ,切换至git用户：`su git`,然后进入 `/home/git/` 目录，更改 `.ssh` 目录权限：`chmod 0700 .ssh` 。（此时，该 .ssh 目录内为空）
4. 然后再次访问 `/admin` 页面，再次执行 `Run` 操作，提示“所有公钥重新生成成功！”信息。查看容器内的 `.ssh/` 目录下生成了一个 `authorized_keys` 文件。然后更改该文件权限：`chmod 0600 authorized_keys` 。
5. 最后重启该容器。

***
