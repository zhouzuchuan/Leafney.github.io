### 使用SSH密钥连接Github

Windows下操作：

#### 第一步 查看是否已经存在SSH秘钥(keys)

打开Git Bash,并运行：

```
$ cd ~/.ssh
```

如果提示如下信息为 `No such file or directory`，则说明不存在SSH秘钥，如果已经存在，可以直接进入第三步。

```
$ cd ~/.ssh
sh.exe": cd: /c/Users/Xue/.ssh: No such file or directory
```

如果无提示信息，进入 `.ssh` 目录执行 `ls` 命令，可查看本机已经存在的SSH的公钥和私钥。

*** 

#### 第二步 创建新的SSH密钥(keys)

输入如下命令：

```
$ cd ~  # 保证当前路径在 `~` 下，即 `C:/Users/用户名` 目录
$ ssh-keygen -t rsa -C "your_email@example.com"  # 这将根据你提供的邮箱地址，创建一对密钥
```

提示信息如下：

```
$ ssh-keygen -t rsa -C "your_email@example.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/用户名/.ssh/id_rsa):   # 直接回车，则将密钥按默认路径及文件名进行存储。此时也可以输入特定的文件名
Created directory '/c/Users/用户名/.ssh'.
Enter passphrase (empty for no passphrase):   # 根据提示，你需要输入密码和确认密码。可以不填，设置为空则，直接回车
Enter same passphrase again:
Your identification has been saved in /c/Users/用户名/.ssh/id_rsa.
Your public key has been saved in /c/Users/用户名/.ssh/id_rsa.pub.
The key fingerprint is:
6d:40:da:xx:xx:xx:xx:b8:60:4a:bd:61:5f:c5:d6:db your_email@example.com
The key's randomart image is:
+--[ RSA 2048]----+
|   .   .=oo.     |
|  . * .=.*o .    |
| . + =oo+..  o   |
|  . ..o. o  . E  |
|      . S o      |
|         .       |
|                 |
|                 |
|                 |
+-----------------+

```

然后在目录 `~/.ssh` 下，就新创建了两个文件：

```
$ cd ~/.ssh
$ ls
id_rsa  id_rsa.pub
```

*** 

#### 第三步 在GitHub账户中添加你的公钥

执行如下命令，将公钥的内容复制到系统剪切板中(或直接打开该文件进行复制操作)：

```
clip < ~/.ssh/id_rsa.pub
```

登陆Github网站，选择 `Settings` --> `SSH and GPG keys` 菜单，点击 `New SSH key` 按钮。
粘贴你的密钥到 `Key` 输入框中并设置 `Title` 信息，点击 `Add SSH key` 按钮完成。

至此，添加完毕。

*** 

#### 第四步 连接测试

先保证本地 `Git` 已设置好git账户的 `用户名` 和 `邮箱` 信息：

```
$ git config --global user.name "your_username"   # 设置用户名
$ git config --global user.email "your_email@example.com"  # 设置邮箱地址
```

测试SSH keys 是否设成功，执行如下命令：

```
$ ssh -T git@github.com
```

提示信息如下：

```
$ ssh -T git@github.com
The authenticity of host 'github.com (192.30.253.113)' can't be established.
RSA key fingerprint is 16:27:xx:xx:xx:xx:xx:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)? yes  # 确认你是否继续联系，输入yes
Warning: Permanently added 'github.com,192.30.253.113' (RSA) to the list of know
n hosts.
Hi xxx! You've successfully authenticated, but GitHub does not provide shell
 access.  #  出现这句话，说明设置成功

```

当提示如下信息，说明连通Github成功：

```
Hi xxx! You've successfully authenticated, but GitHub does not provide shell
 access. 
```

*** 

#### 第五步 将本地项目通过 SSH push 到 GitHub

在github新建一个仓库，如 `test_ssh` 。

执行以下命令：

```
## 创建目录
$ mkdir test
$ cd test
## 初始化git仓库
$ git init
## 创建readme.md文件
$ echo "this is a test ssh keys" > README.md
## 提交到本地
## 提交当前目录下的所有文件
$ git add .
## 提交记录说明
$ git commit -m "add readme.md"
## 提交到github
$ git remote add origin git@github.com:your_github_name/test_ssh.git
$ git push -u origin master
```

刷新 `test_ssh` 仓库，就能看到提交的文件了。

如果是本地已经存在的git项目，只需要执行以下命令即可：

```
## 提交到github
$ git remote add origin git@github.com:your_github_name/test_ssh.git
$ git push -u origin master
```

*** 

#### 相关链接

* [使用SSH密钥连接Github【图文教程】 - 轩枫阁 – 前端开发 | web前端技术博客](http://www.xuanfengge.com/using-ssh-key-link-github-photo-tour.html)
