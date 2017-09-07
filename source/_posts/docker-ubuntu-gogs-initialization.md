---
title: Docker-Ubuntu-Gogs数据库及初始化配置
date: 2017-09-07 11:27:54
tags: 
    - Gogs
    - Docker
categories: 
    - Ubuntu-Gogs
description: Ubuntu-Gogs 首次运行安装程序配置及多数据库配置方法整理。
---

Ubuntu-Gogs 首次运行安装程序配置及多数据库配置方法整理。

#### app.ini中数据库配置说明

| 名称 | 描述 |
| ---- | ----- |
| DB_TYPE | 数据库类型，可以是 mysql、postgres、mssql 或 sqlite3 |
| HOST | 数据库主机地址与端口 |
| NAME | 数据库名称 |
| USER | 数据库用户名 |
| PASSWD | 数据库用户密码 |
| SSL_MODE | 仅限 PostgreSQL 使用 |
| PATH | 仅限 SQLite3 使用，数据库文件路径 |


***

#### 初次启动时数据库设置

Gogs 要求安装 MySQL、PostgreSQL、SQLite3、MSSQL 或 TiDB。

##### SQLite3

###### 数据库设置

* 数据库类型 ：`SQLite3`
* 数据库文件路径 ：可使用绝对路径：`/app/gogs/data/gogs.db` 或者 相对路径：`data/gogs.db` （推荐使用绝对路径）

###### app.ini中配置结果

```
[database]
DB_TYPE  = sqlite3
HOST     = 127.0.0.1:3306
NAME     = gogs
USER     = root
PASSWD   = 
SSL_MODE = disable
PATH     = /app/gogs/data/gogs.db
```

###### 示例容器

```
$ docker run --name gogs1 -d -p 10080:3000 -p 10022:22 -v /home/tiger/gogsfile:/app leafney/ubuntu-gogs
```

***

##### MySQL

###### 数据库设置

* 数据库类型 ：`MySQL`
* 数据库主机 ：`127.0.0.1:3306`
* 数据库用户 ：`root`
* 数据库用户密码 : `*******`
* 数据库名称 ：`gogs`


###### app.ini中配置结果

```
[database]
DB_TYPE  = mysql
HOST     = 127.0.0.1:3306
NAME     = gogs
USER     = root
PASSWD   = `123456`
SSL_MODE = disable
PATH     = data/gogs.db
```

###### 示例容器

**第一种方法：创建mysql容器和gogs容器，让gogs容器通过 `--link` 直接链接mysql容器。**

创建mysql容器，并设置root账户密码：`123456`；新用户：`gogs123`；密码：`gogs123`；新用户数据库：`gogs` ：

```
$ docker run --name mysqlgogs -v /home/tiger/mysqldb/:/var/lib/mysql -v /home/tiger/mysqldbase/:/home/mysqldbase/ -d -e MYSQL_ROOT_PWD="123456" -e MYSQL_USER=gogs123 -e MYSQL_USER_PWD="gogs123" -e MYSQL_USER_DB="gogs" leafney/docker-alpine-mysql
```

创建gogs容器并链接：

```
$ docker run --name gogs2 -d -p 10080:3000 -p 10022:22 --link mysqlgogs:mydb -v /home/tiger/gogsfile:/app leafney/ubuntu-gogs
```

相对应的配置信息如下：

* 数据库类型 ：`MySQL`
* 数据库主机 ：`mydb:3306`
* 数据库用户 ：`gogs123`
* 数据库用户密码 : `*******`
* 数据库名称 ：`gogs`


**第二种方法：让gogs容器链接已有mysql地址。**

在创建gogs容器之前，先创建mysql数据库：

在下载的 `gogs` 压缩包中，我们可以找到一个名为 `mysql.sql` 的文件，是用来初始化mysql数据库的，内容如下：

```
DROP DATABASE IF EXISTS gogs;
CREATE DATABASE IF NOT EXISTS gogs CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

使用 `root` 账户登录，然后执行 `mysql -u root -p < mysql.sql` （需要输入密码）即可初始化好数据库。

还要注意：使用 `MySQL` 数据库时，必须要保证mysql的存储引擎为 `INNODB` 且 编码格式为 `utf8_general_ci` 。可以使用如下语句来设置：

```
use gogs;
set global storage_engine=INNODB;
```

***

或者使用如下命令创建数据库 `gogs` 及新用户 `gogsUser`, 并将数据库 `gogs` 的所有权限都赋予该用户,密码 `123456`:

```
mysql -u root -p
mysql> SET GLOBAL storage_engine = 'InnoDB';
mysql> CREATE DATABASE gogs CHARACTER SET utf8 COLLATE utf8_bin;
mysql> GRANT ALL PRIVILEGES ON gogs.* TO 'gogsUser'@'localhost' IDENTIFIED BY '123456';
mysql> FLUSH PRIVILEGES;
mysql> QUIT；
```


* [使用 Gogs 搭建自己的 Git 服务器 - My Nook](https://blog.mynook.info/post/host-your-own-git-server-using-gogs/)

***

##### PostgreSQL

###### 数据库设置

* 数据库类型 ：`PostgreSQL`
* 数据库主机 ：`127.0.0.1:5432`
* 数据库用户 ：`gogs`
* 数据库用户密码 : `*******`
* 数据库名称 ：`gogs`
* SSL 模式 : `Disable`  (可选：Disable  Require  Verify Full)

###### app.ini中配置结果

```
[database]
DB_TYPE  = postgres
HOST     = 127.0.0.1:5432
NAME     = gogs
USER     = gogs
PASSWD   = gogs
SSL_MODE = disable
PATH     = data/gogs.db
```

###### 示例容器

**第一种方法：创建postgersql容器和gogs容器**

创建postgresql容器，这里使用容器 `docker pull ananthhh/postgress`：

```
docker run --name postgress -p 5432:5432 -e POSTGRES_PASSWORD=gogs -e POSTGRES_USER=gogs -d ananthhh/postgress
```

该postgresql容器创建的用户为：`gogs`；用户密码：`gogs`; 数据库：`gogs` 。

创建gogs容器并通过 `--link` 参数链接：

```
$ docker run --name gogs3 -d -p 10080:3000 -p 10022:22 -v /home/tiger/gogsfile:/app --link postgress:psqldb leafney/ubuntu-gogs
```

相对应的配置信息如下：

* 数据库类型 ：`PostgreSQL`
* 数据库主机 ：`psqldb:5432`
* 数据库用户 ：`gogs`
* 数据库用户密码 : `*******`
* 数据库名称 ：`gogs`
* SSL 模式 : `Disable` 

**第二种方法：让gogs容器链接已有PostgreSQL地址。**

使用指定数据库账户登录PostgreSQL，先创建 `gogs` 数据库，链接成功后会自动创建所需表：

```
> CREATE DATABASE gogs
```


***

##### MSSql

###### 数据库设置

* 数据库类型 ：`MSSQL`
* 数据库主机 ：`127.0.0.1, 1433`
* 数据库用户 ：`sa`
* 数据库用户密码 : `*******`
* 数据库名称 ：`gogs`

###### app.ini中配置结果

```
[database]
DB_TYPE  = mssql
HOST     = 127.0.0.1, 1433
NAME     = gogs
USER     = sa
PASSWD   = 123456
SSL_MODE = disable
PATH     = data/gogs.db
```

###### 示例容器

创建gogs容器链接已有MSSql地址：

```
docker run --name gogs4 -d -p 10080:3000 -p 10022:22 -v /home/tiger/gogsfile:/app leafney/ubuntu-gogs
```

使用指定数据库账户登录MSSql，先创建gogs数据库，链接成功后会自动创建所需表：

```
> CREATE DATABASE gogs
```

***

#### 应用基本设置

以如下命令创建容器为例：

```
$ docker run --name mygogs -d -p 10080:3000 -p 10022:22 -v /home/tiger/gogsfile:/app leafney/ubuntu-gogs
```

* `仓库根目录`： 更改为绝对路径  `/app/gogs-repositories`
* `运行系统用户`：  使用默认用户  `git`
* `域名`： 填写Docker宿主机的主机名或物理地址或要使用的域名(不带http/https) 如  `192.168.137.140`
* `SSH 端口号`： 如果你映射Docker外部端口如 `10022:22` 那么这里就填写 `10022` ；不要勾选“使用内置SSH服务器”（Don't tick Use Builtin SSH Server）
* `HTTP 端口号`： 如果映射Docker外部端口如 `10080:3000` 这里要使用：`3000`
* `应用 URL`： 使用域名和公开的HTTP端口值的组合(带http/https) 如 `http://192.168.137.140:10080`
* `日志路径`： 可使用路径 `/app/gogs/log`(推荐) 或默认值 `/home/git/gogs/log`

***

#### 可选设置

app.ini中邮件(mailer)配置说明

| 名称 | 描述 |
| ---- | ----- |
| ENABLED | 启用该选项以激活邮件服务 |
| DISABLE_HELO  |  禁用 HELO 操作 |
| HELO_HOSTNAME |  HELO 操作的自定义主机名 |
| HOST  |  SMTP 主机地址与端口 |
| FROM  |  邮箱的来自地址，遵循 RFC 5322规范，可以是一个单纯的邮箱地址或者 "名字" <email@example.com> 的形式 |
| USER  |  邮箱用户名 |
| PASSWD | 邮箱密码 |
| SKIP_VERIFY | 不验证自签发证书的有效性 |
| USE_PLAIN_TEXT | 使用 `text/plain` 作为邮件内容格式 |


##### 邮件服务设置

* `SMTP 主机`： 以163为例 如 `smtp.163.com:25`
* `邮件来自`： 格式为 `"Name" <email@example.com>` 如 `GitAdmin <xxxxx@163.com>`
* `发送邮箱`： 邮箱地址 如 `xxxxx@163.com`
* `发送邮箱密码` : 邮箱密码

app.ini中配置结果

```
[mailer]
ENABLED = true
HOST = smtp.163.com:25
FROM = GitAdmin <xxxxx@163.com>
USER = xxxxx@163.com
PASSWD = 123456
```

***

##### 服务器和其它服务设置

* `禁止用户自主注册`  激活该选项来禁止用户注册功能，只能由管理员创建帐号
* `启用验证码服务`  要求在用户注册时输入预验证码
* `启用登录访问限制`  只有已登录的用户才能够访问页面，否则将只能看到登录或注册页面

***

##### 管理员账号设置

创建管理员帐号并不是必须的，因为 `ID=1` 的用户将自动获得管理员权限。

建议在此处直接创建管理员账户。

***

#### 相关参考

* [How To Set Up Gogs on Ubuntu 14.04 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-gogs-on-ubuntu-14-04)
* [配置文件手册 - Gogs](https://gogs.io/docs/advanced/configuration_cheat_sheet)
