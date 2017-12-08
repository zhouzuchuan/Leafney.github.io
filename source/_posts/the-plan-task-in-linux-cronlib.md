---
title: Linux下的计划任务-Crontab
date: 2016-03-28 09:53:10
tags:
    - Linux
    - Crontab
categories: 
    - 开发笔记
description: Linux下的计划任务 Crontab 是用于设置周期性被执行的任务的工具
---

Crontab  用于设置周期性被执行的任务的工具

#### 检查cron服务

检查crontab工具是否安装

```
$ crontab -l 
```

检查crond服务是启动

```
$service crond status
```

注意: 在Ubuntu系统下,查看crontab服务是: `service cron status`

#### 安装cron

```
$ sudo apt-get install vixie-cron
$ sudo apt-get install crontabs
```

如果在输入`crontab -l`之后,提示"`no crontab for root`"的信息,则说明系统中没有设置crontab,需要先进行设置.

crontab是一个文本文件，用来存放你要运行的命令。

执行命令`crontab -e`,会提示:
```
no crontab for root - using an empty one

Select an editor. ....
.....
```
这样的提示,然后选择一个编辑器(这里我选 vim:tiny)即可.然后会进入crontab编辑页面,在编辑页面中直接输入`shift+:`,然后输入`wq`保存,一个新的crontab就生成了. 
然后再执行`crontab -l` 就能看到刚刚编辑过的crontab文件了.

参考:
[Linux提示no crontab for root的解决办法](http://www.2cto.com/os/201110/109339.html)
[Ubuntu下crontab命令的用法](http://www.cnblogs.com/daxian2012/articles/2589894.html)

*** 

#### 实例

执行`crontab -e`打开crontab文件 

写入以下命令:

```
*/1 * * * * date >> /tmp/log.txt
```

表示:每分钟将当前时间写入到tmp目录下的log.txt文件中  

查看log.txt文件,执行命令:

```
$ tail -f /tmp/log.txt
```

表示查看log.txt文件的最后几行  


查看crontab服务运行状态:

```
$ service cron status
```

*** 

#### crontab 配置文件的格式

```
* * * * * COMMAND
```

* 1* 分钟 0~59
* 2* 小时 0~23
* 3* 日期 1~31
* 4* 月份 1~12
* 5* 星期 1-7 (0或者7表示星期天)

##### 案例

1. 每晚 21:30重启apache
```
30 21 * * * service httpd restart
```

2. 每月 1 10 22日的4:45重启apache
```
45 4 1,10,22 * * service httpd restart
```

3. 每月1到10日的4:45重启apache
```
45 4 1-10 * * service httpd restart
```
用`-`表示间隔

4. 每隔2分钟重启Apache服务
```
*/2 * * * * service httpd restart  //偶数分钟内
1-59/2 * * * * service httpd restart  //奇数分钟内
```

5. 晚上11点到早上7点之间,每隔一小时重启apache
```
0 23-7/1 * * * * service httpd retart
```

6. 每天18点到23点之间每隔30分钟重启apache
```
0-59/30 18-23 * * * service httpd restart
0,30 18-23 * * * service httpd restart
```

##### 总结

1. `*` 表示任何时间都匹配
2. 可以用 `A,B,C` 表示A或者B或者C时执行命令
3. 可以用 `A-B` 表示A到B之间时执行命令
4. 可以用 `*/A` 表示每A分钟(小时等)执行一次命令

##### 注意

1. 第三和第五个域之间是"或"的关系:
四月的第一个星期天早晨1时59分运行a.sh
```
59 1 1-7 4 * test `date + \%w` -eq && /root/a.sh
```

*** 

#### Crontab格式

![crontab格式](http://images.cnitblog.com/blog/34483/201301/08090352-4e0aa3fe4f404b3491df384758229be1.png)  

minute   hour   day   month   week   command  

其中：
* minute：表示分钟，可以是从0到59之间的任何整数。  
* hour：表示小时，可以是从0到23之间的任何整数。  
* day：表示日期，可以是从1到31之间的任何整数。  
* month：表示月份，可以是从1到12之间的任何整数。  
* week：表示星期几，可以是从0到7之间的任何整数，这里的0或7代表星期日。  
* command：要执行的命令，可以是系统命令，也可以是自己编写的脚本文件。  

##### 在以上各个字段中，还可以使用以下特殊字符：

* 星号（*）：代表所有可能的值，例如month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。
* 逗号（,）：可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”
* 中杠（-）：可以用整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”
* 正斜线（/）：可以用正斜线指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次。同时正斜线可以和星号一起使用，例如*/10，如果用在minute字段，表示每十分钟执行一次。

*** 

#### Crontab定时调用Python脚本

* 指定python脚本文件中的头信息  `#!/bin/sh`

***

#### 相关链接

* [每天一个linux命令（50）：crontab命令 - peida - 博客园](http://www.cnblogs.com/peida/archive/2013/01/08/2850483.html)
* [crontab命令_Linux crontab 命令用法详解：提交和管理用户的需要周期性执行的任务](http://man.linuxde.net/crontab)
* [crontab 中 python 脚本执行失败的解决方法 | CoCo的小黑屋](http://www.acwind.net/blog/archives/1304)
