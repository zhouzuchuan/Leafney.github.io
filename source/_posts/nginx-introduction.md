---
title: Nginx初级入门
date: 2016-07-03 22:06:00
tags:
    - Linux
categories: 
    - 开发笔记
description: Nginx初级入门
---


安装

```
$ sudo apt-get install nginx
```

在Ubuntu下安装后的文件结构：

* 所有的配置文件都在 `/etc/nginx/` 下
    * `nginx.conf` 为主配置文件
    * `sites-available/default` 为默认配置文件
* 程序文件在 `/usr/sbin/nginx` 下
* 日志放在了 `/var/log/nginx` 下
* 启动脚本 `/etc/init.d/nginx`
* 默认的虚拟主机的目录设置在了 `/var/www/` 下（参考默认配置文件）


在 `nginx.conf` 配置文件中，可看到下面两行：

```
include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*;
```

网上找到的自定义配置文件的设置方法为：

1. 我们可以在 `/etc/nginx/sites-available` 目录下添加自定义配置文件，然后为该文件创建软链接到 `/etc/nginx/sites-enabled` 目录中。
2. 也可以在 `/etc/nginx/conf.d` 目录下创建自定义配置文件并以 `.conf` 为扩展名。

*** 

nginx 的启动 暂停 重启 

* 启动 `sudo /etc/init.d/nginx start`
* 暂停 `sudo /etc/init.d/nginx stop`
* 重启 `sudo /etc/init.d/nginx restart`

*** 

nginx 基本的配置文件写法：

```
sever {
    listen 8080;
    server_name _;

    root /home/tiger/myweb/site;
    index index.html;

    location / {
            try_files $uri $uri/ =404;
    }

}
```

示例文件 `web.conf` ,所在目录为 `/etc/nginx/conf.d` :

```
sever {
    listen 8081;
    server_name _;

    root /home/tiger/myweb/site2;
    index index.html aaa.html;

    location / {
            try_files $uri $uri/ =404;
    }

}

server {
    listen 8082;
    server_name _; # 访问 localhost

    error_page 404 /404.html; # 指定404错误页

    root /home/tiger/myweb/site2/aaa;
    index abc.html;
    location / {
            allow all;
    }
}
```

查看目录下文件：

```
$ ls /home/tiger/myweb/site2
aaa aaa.html
$ ls /home/tiger/myweb/site2/aaa
404.html abc.html
```

*** 
