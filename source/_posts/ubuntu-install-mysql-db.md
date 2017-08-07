---
title: Ubuntu下安装及配置MySql数据库
date: 2016-10-06 23:50:30
tags:
	- Ubuntu
	- Mysql
description: Ubuntu下安装Mysql数据库及一些必要设置
---


#### 安装过程

测试系统为 `Ubuntu 16.04 LTS`

##### 更新Ubuntu软件安装源

```
$ echo "deb http://cn.archive.ubuntu.com/ubuntu/ xenial main restricted universe multiverse" >> /etc/apt/sources.list
```

##### 安装mysql-server

执行如下命令安装mysql:

```
$ sudo apt-get update
$ sudo apt-get install mysql-server
```

安装过程会弹出提示框，输入root用户的密码，这里设置密码为 `mysql` 。

安装完成后，通过命令 `mysql -V` 查看mysql版本信息：

```
# mysql -V
mysql  Ver 14.14 Distrib 5.7.15, for Linux (x86_64) using  EditLine wrapper
```

***

#### 解决mysql连接错误 ERROR 2002 (HY000)

在使用 `root` 账户连接mysql时，报了如下的 `2002` 错误：

```
# mysql -uroot -p
Enter password: 
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)
```

查看 `mysql` 服务是否启动，执行如下命令查看：

```
# ps -aux |grep mysqld
root       984  0.0  0.0  11276   728 ?        S+   07:23   0:00 grep --color=auto mysqld
# ps -aux |grep mysql 
root       986  0.0  0.0  11276   728 ?        S+   07:23   0:00 grep --color=auto mysql
```

可见mysql的服务并没有启动，然后我们尝试启动服务：

```
$ sudo service mysql start

# service mysql start
 * Starting MySQL database server mysqld                                                                                                                       No directory, logging in with HOME=/

```

mysql的服务居然无法启动。

最终找到原因是当前用户对 `/var/run/mysqld` 目录没有操作权限导致的。

先查看 `/var/run/` 下是否存在 mysqld目录，没有先创建。

执行如下命令：

```
$ sudo chown -R mysql:mysql /var/run/mysqld
```

然后再次尝试启动mysql服务：

```
$ sudo service mysql start

# service mysql status
 * /usr/bin/mysqladmin  Ver 8.42 Distrib 5.7.15, for Linux on x86_64
Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Server version		5.7.15-0ubuntu0.16.04.1
Protocol version	10
Connection		Localhost via UNIX socket
UNIX socket		/var/run/mysqld/mysqld.sock
Uptime:			7 sec

Threads: 1  Questions: 8  Slow queries: 0  Opens: 105  Flush tables: 1  Open tables: 98  Queries per second avg: 1.142
```

可以看到mysql服务启动成功了。

网上查找各种解决该问题的方法，也只有这一种方法是根本原因。

所以当再次遇到该问题的时候，先查看一下目录是否有操作权限：

```
$ ls -al /var/run/mysqld/
```

**chown命令将指定文件的拥有者改为指定的用户或组**

* [Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock'的解决](http://www.faceye.net/search/79276.html)

*** 

#### 新增账户及权限设置

#####  第一种方法

###### 以管理员身份登陆mysql

```
$ mysql -uroot -p
```
输入之前设置的密码，登陆mysql命令模式。

###### 选择 `mysql` 数据库

```
mysql> use mysql;
```

###### 创建用户并设定密码

先查看默认存在哪些账户：

```
mysql> select host,user from user;

+-----------+------------------+
| host      | user             |
+-----------+------------------+
| localhost | debian-sys-maint |
| localhost | mysql.sys        |
| localhost | root             |
+-----------+------------------+
3 rows in set (0.00 sec)
```

执行以下命令来创建新用户账户及密码：

```
mysql> create user 'testuser1'@'localhost' identified by 'testpassword';
```

将 `testpassword` 替换为你自己的密码。

执行如下命令使操作生效：

```
mysql> flush privileges;
```

示例操作如下：

```
mysql> create user 'testuser1'@'localhost' identified by '123456';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> select host,user from user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| localhost | debian-sys-maint |
| localhost | mysql.sys        |
| localhost | root             |
| localhost | testuser1        |
+-----------+------------------+
4 rows in set (0.00 sec)
```

###### 为新账户创建数据库

```
mysql> create database testdb;
Query OK, 1 row affected (0.03 sec)
```

###### 为新账户赋予操作新建数据库 `testdb` 的权限 

```
mysql> grant all privileges on testdb.* to 'testuser1'@'localhost' identified by '123456';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

* `testdb.*` 表示操作 `testdb` 这个数据库中所有的表
* `'testuser1'@'localhost'` 表示使用账户 `testuser1` 登陆到 `localhost`
* `'123456'` 表示登陆密码

###### 使用新账户登陆

我们在上面的步骤中创建的新用户为 `testuse1` 密码为 `123456` 管理的数据库为 `testdb`。

使用 `exit` 退出 `root` 账户的登陆，然后使用新账户登陆：

```
mysql> exit
Bye

$ mysql -u testuser1 -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
```

登陆成功后，查看数据库列表：

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| testdb             |
+--------------------+
2 rows in set (0.00 sec)
```

***

##### 第二种方法  通过GRANT授权的方式新增用户

###### 以root账户登录：

```
$ mysql -uroot -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
```

###### 新增数据库

为新账户创建数据库 `testdb2`

```
mysql> create database testdb2;
Query OK, 1 row affected (0.00 sec)
```

###### 新增账户并设置密码

新增账户 `testuser2` 密码为 `123456` 管理 `testdb2` 数据库。

通过 `grant all privileges on` 语句来操作：

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| testdb             |
| testdb2            |
+--------------------+
6 rows in set (0.00 sec)

mysql> grant all privileges on testdb2.* to 'testuser2'@'localhost' identified by '123456';
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

`grant all` 语句不需要使用 `flush privilege;` 刷新系统权限表，该操作立即生效。

###### 使用新账户 `testuser2` 登陆

```
mysql> exit
Bye

$ mysql -utestuser2 -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| testdb2            |
+--------------------+
2 rows in set (0.00 sec)

```

***

#### 将默认编码改为utf8

默认情况下，MySQL的字符集是 `latin1` ，因此在存储中文的时候，会出现乱码的情况，所以我们需要把字符集统一改成 `UTF-8` 。

##### mysql 默认编码

通过如下命令查看mysql默认编码：

```
mysql> show variables like "%character%";show variables like "%collation%";

+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | latin1                     |
| character_set_connection | latin1                     |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | latin1                     |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.01 sec)

+----------------------+-------------------+
| Variable_name        | Value             |
+----------------------+-------------------+
| collation_connection | latin1_swedish_ci |
| collation_database   | latin1_swedish_ci |
| collation_server     | latin1_swedish_ci |
+----------------------+-------------------+
3 rows in set (0.00 sec)
```

##### 更改配置文件

一般情况下，在 Ubuntu 14.04 系统中，mysql的配置文件目录为：

```
/etc/mysql/my.cnf
```

在 Ubuntu 16.04 系统下，mysql的配置文件目录为：

```
/etc/mysql/my.cnf

/etc/mysql/mysql.conf.d/mysqld.cnf
```

我当前的系统为 `Ubuntu16.04` 。其实在 ubuntu 16.04 系统中，mysql的配置文件路径也为 `/etc/mysql/my.cnf` ，只不过这个 `my.cnf` 是全局配置文件，在该文件内部可以看到如下配置：

```
# 
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```

具体的配置文件是存放在上面两个目录下的。所以我们可以更改 `/etc/mysql/my.cnf` 这个文件，也可以更改 `/etc/mysql/mysql.conf.d/mysqld.cnf` 这个文件。或者也可以自己新增一个扩展名为 `*.cnf` 的配置文件放在上面包含的两个目录内。

从网上找到各种说法的修改编码为utf-8的方法，经测试后需要修改的配置如下：

```
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
```

编辑 `/etc/mysql/my.cnf` 配置文件，依次添加上面的编码设置。

```
#

[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci


!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/

```

***

然后重启 mysql 服务：

```
$ service mysql restart
```

再次登录mysql命令模式查看默认编码：

```
$ mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.

mysql> show variables like "%character%";show variables like "%collation%";
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

+----------------------+-----------------+
| Variable_name        | Value           |
+----------------------+-----------------+
| collation_connection | utf8_general_ci |
| collation_database   | utf8_unicode_ci |
| collation_server     | utf8_unicode_ci |
+----------------------+-----------------+
3 rows in set (0.00 sec)
```

可见，我们已经更改成功了。

* [Change MySQL default character set to UTF-8 in my.cnf? - Stack Overflow](http://stackoverflow.com/questions/3513773/change-mysql-default-character-set-to-utf-8-in-my-cnf)

*** 

#### 让MySQL服务器被远程访问

默认情况下，root账户只能从 `localhost` 即本机下来访问mysql的服务。而在正式使用时，mysql数据库都是放在远程的数据库服务器上，这样也就需要我们通过远程的方式能够访问到mysql服务。

##### 开启绑定端口

编辑配置文件 `/etc/mysql/my.cnf` 或 `/etc/mysql/mysql.conf.d/mysqld.cnf` ，将绑定地址行注释掉或者修改为指定IP：

```
#注释bind-address
# bind-address                   = 127.0.0.1
```

* 注释掉则允许所有ip都能够访问，也可以设置成 `0.0.0.0`
* 修改为指定的IP地址，则只允许该IP网段可以访问

修改配置文件后，重启 mysql 服务生效：

```
$ service mysql restart
```

如果这时通过外网连接mysql，在连接时会出现错误 “'Host XXX is not allowed to connect to this MySQL server' ” ，则还需要修改数据库中用户的访问权限。

##### 修改数据库中账户访问权限

这里以 `root` 账户为例来设置远程访问。

###### 查看root账户可访问权限

以 `root` 账户登录：

```
mysql -u root -p
Enter password: <enter password>
```

切换到 `mysql` 数据库，并查询 `user` 表中的账户设置：

```
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

mysql> select host,user from user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| localhost | debian-sys-maint |
| localhost | mysql.sys        |
| localhost | root             |
| localhost | testuser1        |
| localhost | testuser2        |
+-----------+------------------+
5 rows in set (0.00 sec)

```

可以看到，root账户默认下不允许从远程登陆，只能从 `localhost` 来访问，我们还要为 `root` 账户添加访问权限。

###### 添加远程访问授权

这里有两种方法，一种是将上面的 `mysql` 数据库中的 `user` 表里的 `host` 项，将 `localhost` 改为 `%`，

```
mysql> use mysql;
mysql> update user set host ='%' where user ='root';
```

另外一种是为账号 `root` 添加一个新的远程访问授权。

这里我们采用第二种方法。

通过命令 `GRANT ALL PRIVILEGES ON *.* to root@'%' IDENTIFIED BY 'put-your-password' WITH GRANT OPTION;` 来操作。

执行如下命令：

```
mysql -u root -p
Enter password: <enter password>

mysql> GRANT ALL PRIVILEGES ON *.* to root@'%' IDENTIFIED BY 'mysql' WITH GRANT OPTION;

mysql> FLUSH PRIVILEGES;

```

再次查看：

```
mysql> select host,user from user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| %         | root             |
| localhost | debian-sys-maint |
| localhost | mysql.sys        |
| localhost | root             |
| localhost | testuser1        |
| localhost | testuser2        |
+-----------+------------------+
6 rows in set (0.00 sec)
```

现在再尝试通过外网来连接 `mysql` 数据库，就能连接成功了。

***

###### 本地和远程访问使用不同权限或密码

在上面的表中我们可以知道，`root` 账户有两个 `host` 配置项，一个本地的，一个远程的。其实我们可以将两项设置成不同的密码，以防止本地或远程的密码泄露问题。也可以在 `grant` 后跟详细的查询条件 `select,delete` 等，为本地或远程访问设置不同的访问权限。比如：

```
mysql> GRANT SELECT,UPDATE,INSERT,DELETE on *.* to root@'%' IDENTIFIED BY 'mysql' WITH GRANT OPTION;
```

* [mysql grant 命令三种常用 - redfox - 博客园](http://www.cnblogs.com/redfox241/archive/2009/08/07/1541212.html)
* [mysql Grant 语法详解](http://5iwww.blog.51cto.com/856039/267499)

***

###### 添加特定远程访问权限

假设账户 `myuser` 密码 `mypwd`

1. `grant all privileges on *.* to 'myuser'@'localhost' identified by 'mypwd' `
2. `grant all privileges on *.* to 'myuser'@'%' identified by 'mypwd' `
3. `grant all privileges on *.* to 'myuser'@'10.22.255.18' identified by 'mypwd' `

说明：

1. 添加一个本地用户 `myuser` ,一般用于web服务器和数据库服务器在一起的情况
2. 添加一个用户 `myuser` ,只要能连接数据库服务器的机器都可以使用，这个比较危险，一般不用
3. 在数据库服务器上给 `10.22.255.18` 机器添加一个用户 `myuser`，一般用于web服务器和数据库服务器分离的情况

**注意**：真正使用的时候不会用 `grant all PRIVILEGES on *.* ` ，而是根据实际需要设定相关的权限。

***

###### 特定访问权限

* 如果想让账户 `myuser` 使用 `mypwd` 从任何主机连接到 `mysql` 服务器，执行：  
```
GRANT ALL PRIVILEGES ON *.* to myuser@'%' IDENTIFIED BY 'mypwd' WITH GRANT OPTION;
```
* 如果想让账户 `myuser` 使用密码 `123456` 从 ip为 `123.123.123.123` 的主机连接到 `mysql` 服务器，执行：
```
GRANT ALL PRIVILEGES ON *.* to myuser@'123.123.123.123' IDENTIFIED BY '123456' WITH GRANT OPTION;
```

即  '%' 表示任何主机。

***

###### `WITH GRANT OPTION` 是啥意思

`WITH GRANT OPTION` 表示具有授予权限的权利。比如 上面的： 

```
GRANT ALL PRIVILEGES ON *.* to myuser@'%' IDENTIFIED BY 'mypwd' WITH GRANT OPTION;
```

为 `root` 用户赋予了 `ALL PRIVILEGES` 的权限，那么 `root` 账户就可以为其他的账户比如 `testuser1` 设置不同的权限。

***
