### Ubuntu下安装及配置MySql数据库

#### 需要修改MySQL服务器的配置：

* 将默认编码改为utf8
* 运行远程访问
* 新增账户及设置权限
* 更改数据库文件保存目录


Ubuntu设置软件源: 

```
echo "deb http://archive.ubuntu.com/ubuntu/ precise universe" >> /etc/apt/sources.list
```

通过 apt-get 安装: 

```
sudo apt-get update
sudo apt-get install mysql-server
```

安装过程会弹出提示框，输入root用户的密码，我在这里设置密码为mysql。


通过安装过程可以看到 安装的为 mysql5.7 的版本

mysql-server-5.7 (5.7.15-0ubuntu0.16.04.1)

查看mysql服务器状态：

```
service mysql status
```

测试登陆：

```
mysql -uroot -p
Enter password:
```


***

#### 将字符编码设置为UTF-8

默认情况下，MySQL的字符集是latin1，因此在存储中文的时候，会出现乱码的情况，所以我们需要把字符集统一改成UTF-8。

查看mysql默认编码：

mysql> show variables like "%character%";show variables like "%collation%";


[client]
default-character-set=utf8

[mysqld]
character-set-server=utf8

[mysql]
default-character-set=utf8



sudo service mysql restart


也有评论说在 [mysqld] 下加如下配置：

[mysqld] 
init_connect='SET collation_connection = utf8_unicode_ci' 
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake


也有评论说只加下面两行：

...
[mysqld]
character-set-server=utf8
collation-server=utf8_general_ci
...


个人感觉靠谱的可能是添加下面这三行:

```
[mysqld]
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
```

所以我的配置如下：

vim /etc/mysql/my.cnf

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

然后查看：

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


详细可参考：

[Change MySQL default character set to UTF-8 in my.cnf? - Stack Overflow](http://stackoverflow.com/questions/3513773/change-mysql-default-character-set-to-utf-8-in-my-cnf)

***

在 Ubuntu 14.04 系统下，mysql的配置文件目录为：

```
/etc/mysql/my.cnf
```

在  Ubuntu 16.04 系统下，mysql的配置文件目录为：

```
/etc/mysql/mysql.conf.d/mysqld.cnf
```

[MySQL istallation problem on Ubuntu 16.04 (my.cnf public ip problem) - Ask Ubuntu](http://askubuntu.com/questions/763774/mysql-istallation-problem-on-ubuntu-16-04-my-cnf-public-ip-problem)

***

#### 存储格式及可访问地址设置

```
[mysqld]
default-storage-engine         = InnoDB


bind-address                   = 127.0.0.1
```

待完善。

***

#### 改变数据存储位置

直接修改配置文件 `/etc/mysql/my.cnf`，找到datadir属性修改目录。

```
[mysqld]
datadir         = /var/lib/mysql
```

***

#### 让MySQL服务器被远程访问 (已测试通过)

##### 第一步

编辑 `/etc/mysql/my.cnf` 将绑定地址行注释掉或者修改为指定IP：

#注释bind-address
# bind-address                   = 127.0.0.1


如果这时通过外网连接mysql，在连接时出现错误 “'Host XXX is not allowed to connect to this MySQL server' ” ，则还需要按照下面的方法来进行设置。

##### 第二步

mysql -u root -p
Enter password: <enter password>

use mysql;

select host,user from user;

mysql> select host,user from user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| localhost | debian-sys-maint |
| localhost | mysql.sys        |
| localhost | root             |
+-----------+------------------+
3 rows in set (0.00 sec)

可以看到，root账户默认下不允许从远程登陆，只能从 `localhost` 来访问，我们还要为 `root` 账户添加访问权限。

这里有两种方法，一种是将上面的 `mysql` 数据库中的 `user` 表里的 `host` 项，将 `localhost` 改为 `%`，

```
mysql> use mysql;
mysql> update user set host ='%' where user ='root';
```

另外一种是为账号 `root` 添加一个新的远程访问授权。

这里我们采用第二种方法。

执行如下命令：

```
mysql -u root -p
Enter password: <enter password>
mysql>GRANT ALL PRIVILEGES ON *.* to root@'%' IDENTIFIED BY 'put-your-password' WITH GRANT OPTION;
mysql>FLUSH PRIVILEGES;
mysql>exit
```

然后重启数据库服务：

sudo service mysql restart


如果想让账户 myuser 使用 mypwd 从任何主机连接到 mysql 服务器，执行：

```
GRANT ALL PRIVILEGES ON *.* to myuser@'%' IDENTIFIED BY 'mypwd' WITH GRANT OPTION;
``` 

如果想让账户 myuser 使用 123456 从 ip为 123.123.123.123 的主机连接到 mysql 服务器，执行：

```
GRANT ALL PRIVILEGES ON *.* to myuser@'123.123.123.123' IDENTIFIED BY '123456' WITH GRANT OPTION;
```

即  '%' 表示任何主机。

这里我执行：

GRANT ALL PRIVILEGES ON *.* to root@'%' IDENTIFIED BY 'put-your-password' WITH GRANT OPTION;


然后再次查看：

```
mysql> select host,user from user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| %         | root             |
| localhost | debian-sys-maint |
| localhost | mysql.sys        |
| localhost | root             |
+-----------+------------------+
4 rows in set (0.01 sec)
```

现在，再尝试通过外网来连接 `mysql` 数据库，发现连接成功。


* [Host &#39;xxx.xx.xxx.xxx&#39; is not allowed to connect to this MySQL server - Stack Overflow](http://stackoverflow.com/questions/1559955/host-xxx-xx-xxx-xxx-is-not-allowed-to-connect-to-this-mysql-server)
* [Ubuntu Mysql开通外网访问权限 - SmartFramework - 博客园](http://www.cnblogs.com/micro-chen/p/5452786.html)
* [mysql 远程访问不行解决方法 Host is not allowed to connect to this MySQL server - Ьchen3888015.taobao.com - 51CTO](http://chen3888015.blog.51cto.com/2693016/986841) **☆**

***

#### 相关参考

##### Ubuntu 14.04

* [How To Install MySQL on Ubuntu 14.04 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-14-04)
* [Install MySQL on Ubuntu 14.04](https://www.linode.com/docs/databases/mysql/install-mysql-on-ubuntu-14-04)
* [Linux(Ubuntu)下MySQL的安装与配置 - ZuqingLi        - 博客频道 - CSDN.NET](http://blog.csdn.net/lizuqingblog/article/details/18423751)
* [Install MySQL Server on Ubuntu](https://support.rackspace.com/how-to/installing-mysql-server-on-ubuntu/)


* [How To Backup MySQL Databases on an Ubuntu VPS | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-backup-mysql-databases-on-an-ubuntu-vps)
* [How To Import and Export Databases and Reset a Root Password in MySQL | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-import-and-export-databases-and-reset-a-root-password-in-mysql)


通过 apt-get 的方式安装：

* [How to Install MySQL on Ubuntu 14.04 LTS | Liquid Web Knowledge Base](https://www.liquidweb.com/kb/how-to-install-mysql-on-ubuntu-14-04-lts/)
* [Installing MySQL 5.6 on Ubuntu 14.04 (Trusty Tahr) | Master MySQL](http://www.tocker.ca/2014/04/21/installing-mysql-5-6-on-ubuntu-14-04-trusty-tahr.html)
* [How to install mysql server 5.6 on Ubuntu 14.04 LTS ( Trusty Tahr )](http://sharadchhetri.com/2014/05/07/install-mysql-server-5-6-ubuntu-14-04-lts-trusty-tahr/)
* [How To Install MySQL on Ubuntu 14.04 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-14-04)
* [在Ubuntu中安装MySQL | 粉丝日志](http://blog.fens.me/linux-mysql-install/) **这个不错**


 ##### mysql-6.0.11-alpha

如何通过 tar 安装 tar.gz 的mysql安装包:

* [12.04 - How to install mysql-6.0.10-alpha? - Ask Ubuntu](http://askubuntu.com/questions/156182/how-to-install-mysql-6-0-10-alpha)
* [Index of /MySQL/Downloads/MySQL-6.0](http://download.softagency.net/MySQL/Downloads/MySQL-6.0/)  **从这里下载**
* [MySQL :: MySQL 5.7 Reference Manual :: 2.2 Installing MySQL on Unix/Linux Using Generic Binaries](http://dev.mysql.com/doc/refman/5.7/en/binary-installation.html) **参考这个安装方法**

***