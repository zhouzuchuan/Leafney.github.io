---
title: Python时间差
date: 2016-05-17 10:41:00
tags:
    - Python
categories: 
    - 开发笔记
description: Python中计算时间差值
---

#### datetime.timedelta

`datetime.timedelta`对象代表两个时间之间的的时间差，两个date或datetime对象相减时可以返回一个timedelta对象。

构造函数:

```
class datetime.timedelta([days[, seconds[, microseconds[, milliseconds[, minutes[, hours[, weeks]]]]]]])
```

所有参数可选，且默认都是0，参数的值可以是整数，浮点数，正数或负数。

`timedelta` 可以和 ` date，datetime` 对象进行加减操作

`timedelta.total_seconds()` 用于计算秒数。

***

#### 当前的时间上加一天或一年减一天等操作

```
#!/usr/bin/env python   
# -*- coding:utf-8 -*-   
  
from datetime import datetime,timedelta   
  
now = datetime.now()   
  
yestoday = now - timedelta(days=1)   
tommorow = now + timedelta(days=1)   
  
next_year = now + timedelta(days = 365)  
```

#### 相关链接

* [Python中时间的处理之——timedelta篇 - Goodspeed - 博客园](http://www.cnblogs.com/goodspeed/archive/2011/11/06/python_timedelta.html)
