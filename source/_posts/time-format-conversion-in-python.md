---
title: Python中时间格式转换
date: 2016-05-17 22:44:51
tags:
    - Python
categories: 
    - 开发笔记
description: Python中时间格式转换
---


```
YYYY-MM-DDTHH:MM:SS+HH:MM
2016-04-05T13:31:00+08:00

2014-09-18T10:42:16.126Z
```


```
$ time = '2012-03-01T00:05:55+00:00'
$ datetime.strptime(time, "%Y-%m-%dT%H:%M:%S+00:00")
# => datetime.datetime(2012, 3, 1, 0, 5, 55)
```

`strftime()` 用于时间格式转换
`strptime()` 用于字符串格式转换


#### 代码段

```

# 将UTC时间转换为本地时间
# 2016-04-04T23:58:00+08:00
def _utc_datetime(value):
    # value为传入的值为UTC时间，如：2016-04-04T23:58:00+08:00
    format='%Y-%m-%d %H:%M:%S'
    utc_format='%Y-%m-%dT%H:%M:%S+08:00'
    local= datetime.strptime(value,utc_format)
    dt= datetime.strftime(local,format)
    return dt


'''
将unix时间戳转为标准时间格式
'''
def _timestamp_datetime(value):
    format = '%Y-%m-%d %H:%M:%S'
    # value为传入的值为时间戳(整形)，如：1332888820
    value = time.localtime(value)
    ## 经过localtime转换后变成
    ## time.struct_time(tm_year=2012, tm_mon=3, tm_mday=28, tm_hour=6, tm_min=53, tm_sec=40, tm_wday=2, tm_yday=88, tm_isdst=0)
    # 最后再经过strftime函数转换为正常日期格式。
    dt = time.strftime(format, value)
    return dt

```

***

#### 相关链接

* 答案见这里：[Convert UTC time to python datetime - Stack Overflow](http://stackoverflow.com/questions/13662789/convert-utc-time-to-python-datetime)
* [datetime - [ Python 3零起点教程 ] - 看云](http://www.kancloud.cn/thinkphp/python-guide/39410)
* [Python中Timestamp、Datetime和UTC时间相互转化的方法 - OPEN 开发经验库](http://www.open-open.com/lib/view/open1412994489608.html)
* [SQLite 日期类型(转) - 深海的小鱼儿 - 博客园](http://www.cnblogs.com/xmphoenix/archive/2011/05/23/2054022.html)
* [python本地时间与UTC时间转换_丶Source_新浪博客](http://blog.sina.com.cn/s/blog_4da051a60102v221.html)
* [Python将UTC时间转化为Local时间 - 降龍        - 博客频道 - CSDN.NET](http://blog.csdn.net/wuxianglong/article/details/7061568)
* [python时间处理之datetime - 码农老毕的学习笔记        - 博客频道 - CSDN.NET](http://blog.csdn.net/wirelessqa/article/details/7973121)
* [Python 时间戳和日期相互转换 - 李林克斯](http://liyangliang.me/posts/2012/10/python-timestamp-to-timestr/) **☆**
* [Python中Timestamp、Datetime和UTC时间相互转化的方法_python_ThinkSAAS](http://www.thinksaas.cn/topics/0/593/593950.html)
