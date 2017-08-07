### Python独立运行环境Virtualenv

Virtualenv可以为每个Python应用创建独立的开发环境，使他们互不影响，Virtualenv能够做到：

* 在没有权限的情况下安装新套件
* 不同应用可以使用不同的套件版本
* 套件升级不影响其他应用

***

#### 安装virtualenv

##### 使用pip安装（推荐）

```
$ sudo pip install virtualenv
```

##### 使用 easy_install 安装：

```
$ sudo easy_install virtualenv
```

***

#### 初始化

我通常创建一个包含虚拟名称为 `venv` 文件夹的项目文件夹:

```
$ mkdir myproject
$ cd myproject
$ virtualenv venv
New python executable in venv/bin/python2
Also creating executable in venv/bin/python
Installing setuptools, pip...done.
```

#### 激活虚拟环境

现在，每次需要使用项目时，必须先激活相应的环境。

##### 在Linux系统下执行

```
$ ls
-- venv
$ source ./venv/bin/activate

//结果：
(venv)tiger@VirtualBox:~/xbox/myflask$ 
```

##### 在Win系统下执行

```
> ls
venv/
> venv\Scripts\activate.bat
(venv) D:\YYYY
```

你现在就进入你的 `virtualenv` 虚拟环境了（**注意查看你的 `shell` 提示符已经改变了**）。

***

#### 退出虚拟环境

通过 `deactivate` 命令退出虚拟环境。

*** 

#### virtualenv 命令整理

##### 安装

```
pip install virtualenv 
```

##### 创建

```
virtualenv <EnvName>
```

##### *nix 

```
$ source ./venv/bin/activate
```

此处 `venv` 为 `<EnvName>`

##### Win 

```
> venv\Scripts\activate
```

此处 `venv` 为 `<EnvName>`

##### 退出

```
deactivate
```

***

#### 相关参考

* [virtualenv &mdash; virtualenv 1.7.1.2.post1 documentation](https://virtualenv-chinese-docs.readthedocs.io/en/latest/)

***