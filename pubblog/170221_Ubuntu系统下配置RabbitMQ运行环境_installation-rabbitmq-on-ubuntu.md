### Ubuntu 16.04 安装 RabbitMQ

该文章主要记录在 `Ubuntu 16.04.1 LTS` 系统下安装及配置 RabbitMQ 的方法。

#### 更新软件源

```
sudo echo "deb http://cn.archive.ubuntu.com/ubuntu/ xenial main restricted universe multiverse" > /etc/apt/sources.list
sudo echo "deb http://cn.archive.ubuntu.com/ubuntu/ xenial-security main restricted universe multiverse" >> /etc/apt/sources.list

sudo apt-get update
```

#### 安装Erlang依赖

```
cd /tmp
wget http://packages.erlang-solutions.com/ubuntu/erlang_solutions.asc
sudo apt-key add erlang_solutions.asc
sudo apt-get update
sudo apt-get install erlang
sudo apt-get install erlang-nox
```

***

#### 安装RabbitMQ

```
sudo echo "deb http://www.rabbitmq.com/debian/ testing main" >> /etc/apt/sources.list
sudo wget https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
sudo apt-key add rabbitmq-release-signing-key.asc
sudo apt-get update
sudo apt-get install rabbitmq-server
```

#### 管理RabbitMQ服务

##### 管理服务

可以使用 `rabbitmqctl` 或系统服务 `service` 或者 `systemctl` 来管理.

**rabbitmqctl** :

```
$ sudo rabbitmqctl [status|start|stop|reset]
```

**systemctl** :

```
$ sudo systemctl [status|start|stop|restart] rabbitmq-server
```

**service** :

```
$ sudo service rabbitmq-server [status|start|stop|restart]
```

如果 `status` 状态显示无法连接rabbitmq服务，需要先启动该服务。

```
$ sudo rabbitmqctl status
Status of node rabbit@localhost ...
Error: unable to connect to node rabbit@localhost: nodedown


$ sudo rabbitmqctl start


$ sudo rabbitmqctl status
Status of node rabbit@localhost ...
[{pid,9286},
 {running_applications,[{rabbit,"RabbitMQ","3.6.6"},
                        {mnesia,"MNESIA  CXC 138 12","4.13.3"},
                        {os_mon,"CPO  CXC 138 46","2.4"},
                        {rabbit_common,[],"3.6.6"},
                        {xmerl,"XML parser","1.3.10"},
                        {ranch,"Socket acceptor pool for TCP protocols.",
                               "1.2.1"},
                        ...
                        ...
```

***

##### 在Docker下管理

注意：在Ubuntu系统的 `Docker` 下 ，使用 `systemctl` 命令会报错，所以在docker下还是推荐使用 `service` 来管理。

```
root@6b16517eab27:/tmp# systemctl
Failed to connect to bus: No such file or directory

root@6b16517eab27:/tmp# service rabbitmq-server status
Status of node rabbit@6b16517eab27 ...
[{pid,12184},
 {running_applications,
     [{rabbitmq_management,"RabbitMQ Management Console","3.6.6"},
      {rabbitmq_web_dispatch,"RabbitMQ Web Dispatcher","3.6.6"},
      {webmachine,"webmachine","1.10.3"},
      {mochiweb,"MochiMedia Web Server","2.13.1"},
      {amqp_client,"RabbitMQ AMQP Client","3.6.6"},
      {rabbitmq_management_agent,"RabbitMQ Management Agent","3.6.6"},
      {rabbit,"RabbitMQ","3.6.6"},
      ...
      ...
```

> So the systemctl can not run inside docker, right? thanks!

* [Running systemctl in container fails with &quot;Failed to get D-Bus connection&quot; · Issue #2296 · docker/docker · GitHub](https://github.com/docker/docker/issues/2296)
* [systemd and systemctl within Ubuntu Docker images - Stack Overflow](http://stackoverflow.com/questions/39169403/systemd-and-systemctl-within-ubuntu-docker-images)

***

#### RabbitMQ Web管理接口

##### 启用rabbitmq-management

启用rabbitmq-management插件：

```
$ sudo rabbitmq-plugins enable rabbitmq_management
```

```
$ sudo rabbitmq-plugins enable rabbitmq_management
The following plugins have been enabled:
  mochiweb
  webmachine
  rabbitmq_web_dispatch
  amqp_client
  rabbitmq_management_agent
  rabbitmq_management

Applying plugin configuration to rabbit@6b16517eab27... started 6 plugins.
```

重启RabbitMQ:

```
$ sudo systemctl restart rabbitmq-server
```

使用浏览器访问 `http://localhost:15672` ，使用默认的 `guest/guest` 用户登录。

***

##### 使用guest账户远程访问

注意：使用远程访问或在Ubuntu系统的Docker下使用所在服务器的地址访问时会报权限错误：

```
{error: "not_authorised", reason: "User can only log in via localhost"}
```

这是因为 rabbitmq从3.3.0开始禁止使用 `guest/guest` 权限通过除 `localhost` 外的访问。

如果想使用 `guest/guest` 通过远程机器访问，需要在rabbitmq配置文件 `(/etc/rabbitmq/rabbitmq.config)` 中设置 `loopback_users为[]` 。

`/etc/rabbitmq/rabbitmq.config` (不存在先创建) 文件完整内容如下（注意后面的半角句号）：

```
[{rabbit, [{loopback_users, []}]}].
```

操作步骤如下：

```
# ls /etc/rabbitmq/
enabled_plugins

$ sudo vim /etc/rabbitmq/rabbitmq.config
    [{rabbit, [{loopback_users, []}]}].

$ sudo service rabbitmq-server restart
```

***

##### 创建新账户

如果不想使用默认的 `guest` 账户，可以创建一个新的具有管理员权限的账户，如创建一个 `test/test` 账户操作如下：

```
rabbitmqctl add_user test test
rabbitmqctl set_user_tags test administrator
rabbitmqctl set_permissions -p / test ".*" ".*" ".*"
```

***

* [RabbitMQ - Access Control (Authentication, Authorisation) in RabbitMQ](https://www.rabbitmq.com/access-control.html)
* [Can&#39;t access RabbitMQ web management interface after fresh install - Stack Overflow](https://stackoverflow.com/questions/22850546/cant-access-rabbitmq-web-management-interface-after-fresh-install/22854222#22854222)
* [RabbitMQ 3.3.1 can not login with guest/guest - Stack Overflow](http://stackoverflow.com/questions/23669780/rabbitmq-3-3-1-can-not-login-with-guest-guest)
* [rabbitmq问题之HTTP access denied: user &#39;guest&#39; - User can only log in via localhost - 布雷泽 - 博客园](http://www.cnblogs.com/lazyboy/p/3853371.html)

***

#### 相关链接

* [How To Install RabbitMQ on Ubuntu 16.04 - idroot](http://idroot.net/linux/install-rabbitmq-ubuntu-16-04/)
* [Ubuntu 16.04 安装 RabbitMQ](http://blog.topspeedsnail.com/archives/4750)
