---
title: Scrapy爬虫初探
date: 2016-06-27 21:26:00
tags:
    - Python
    - Scrapy
    - 爬虫
categories: 
    - 开发笔记
description: Scrapy 是一个快速的高层次的屏幕抓取和网页爬虫框架
---

Scrapy 是一个快速的高层次的屏幕抓取和网页爬虫框架，爬取网站，从网站页面得到结构化的数据，它有着广泛的用途，从数据挖掘到监测和自动测试，Scrapy完全用Python实现，完全开源，代码托管在Github上，可运行在Linux，Windows，Mac和BSD平台上，基于Twisted的异步网络库来处理网络通讯，用户只需要定制开发几个模块就可以轻松的实现一个爬虫，用来抓取网页内容以及各种图片。

#### 安装Scrapy

```
$ sudo pip install Scrapy
```

scrapy依赖 `lxml` ,请保证已经安装了 `lxml` 库。

#### 创建

初始化项目框架

这里以抓取 `http://www.cnbeta.com/topics/9.htm` 页面中的文章标题和简介为例，创建项目名称为 `cnbetaSpider` 。

```
$ scrapy startproject cnbetaSpider
```

该命令将会创建包含下列内容的 `cnbetaSpider` 目录:

```
cnbetaSpider/
    scrapy.cfg
    cnbetaSpider/
        __init__.py
        items.py
        pipelines.py
        settings.py
        spiders/
            __init__.py
            ...
```

这些文件分别是:

* `scrapy.cfg` : 项目的配置文件
* `cnbetaSpider/` : 该项目的python模块。之后您将在此加入代码。
* `cnbetaSpider/items.py` : 项目中的item文件.
* `cnbetaSpider/pipelines.py` : 项目中的pipelines文件.
* `cnbetaSpider/settings.py` : 项目的设置文件.
* `cnbetaSpider/spiders/` : 放置spider代码的目录.

*** 

#### 定义Item

`Item` 是保存爬取到的数据的容器。我们需要从网页中提取 文章的标题，链接，描述，对此，在 `item` 中定义相应的字段。编辑 `cnbetaSpider` 目录中的 `items.py` 文件:

```
import scrapy

class CnbetaspiderItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    title=scrapy.Field()
    link=scrapy.Field()
    desc=scrapy.Field()
    pass

```

#### 创建Spider

`Spider` 是用户编写用于从单个网站(或者一些网站)爬取数据的类。
其包含了一个用于下载的初始URL，如何跟进网页中的链接以及如何分析页面中的内容， 提取生成 `item` 的方法。

为了创建一个 `Spider` ，您必须继承 `scrapy.Spider` 类， 且定义以下三个属性:

* `name` 指定Spider的名称，唯一
* `start_urls`  包含了Spider在启动时进行爬取的url列表
* `parse()` 接收完成下载后生成的 `Response` 对象，该方法负责解析返回的数据，提取数据(生成item)以及生成需要进一步处理的URL的 `Request` 对象。

在目录 `cnbetaSpider/spiders` 下创建文件 `cnbetaspider.py`:

```
# coding:utf-8

import scrapy

class cnbetaSpider(scrapy.Spider):
    """docstring for cnbetaSpider"""
    name="cnbeta"
    allowed_domains=["cnbeta.com"]
    start_urls=[
        "http://www.cnbeta.com/topics/9.htm"
    ]

    def parse(self,response):
        filename=response.url.split("/")[-2]
        with open(filename,'wb') as f:
            f.write(response.body)
    
```

#### 爬取

进入项目根目录，执行下列命令启动spider:

```
$ ls
cnbetaSpider  scrapy.cfg
# 执行命令
$ scrapy crawl cnbeta
```

第三个参数为 `cnbetaspider.py` 中的 `name` 属性值。

执行完成后可在当前目录下看到生成的文件 `topics`,里面保存的获取到的网页的内容。

*** 

#### 提取

Scrapy使用了一种基于 `XPath` 和 `CSS` 表达式机制: `Scrapy Selectors` 。详见：[选择器(Selectors) &mdash; Scrapy 0.24.1 文档](http://scrapy-chs.readthedocs.io/zh_CN/latest/topics/selectors.html#topics-selectors)

选择器方法( `.xpath()` or `.css()` )

为了提取真实的原文数据，你需要调用 `.extract()` 方法：

```
>>> response.xpath('//title/text()').extract()
[u'Example website']
```

这里给出XPath表达式的例子及对应的含义:

* `/html/head/title`  : 选择HTML文档中 `<head>` 标签内的 `<title>` 元素
* `/html/head/title/text()` : 选择上面提到的 `<title>` 元素的文字
* `//td` : 选择所有的 `<td>` 元素
* `//div[@class="mine"]` : 选择所有具有 `class="mine"` 属性的 `div` 元素

Selector有四个基本的方法:

* `xpath()` : 传入xpath表达式，返回该表达式所对应的所有节点的selector list列表 。
* `css()` : 传入CSS表达式，返回该表达式所对应的所有节点的selector list列表.
* `extract()` : 序列化该节点为unicode字符串并返回list。
* `re()` : 根据传入的正则表达式对数据进行提取，返回unicode字符串list列表。

分析网页 `http://www.cnbeta.com/topics/9.htm` 中的文章：

文章列表在 `item` 元素：

```
//*[@class="all_news_wildlist"]/div[@class="items"]/div[@class="item"]
```

标题：

```
xpath('//*[@class="all_news_wildlist"]/div[@class="items"]/div[@class="item"]/*[@class="hd"]/div[@class="title"]/a/text()').extract()
```

链接：

```
xpath('//*[@class="all_news_wildlist"]/div[@class="items"]/div[@class="item"]/*[@class="hd"]/div[@class="title"]/a/@href').extract()
```

描述：

```
xpath('//*[@class="all_news_wildlist"]/div[@class="items"]/div[@class="item"]/*[@class="hd"]/*[@class="newsinfo"]/p/text()').extract()
```

修改我们之前定义的 `cnbetaspider.py` 中的 `parse()` 方法如下：

```
import scrapy

class cnbetaSpider(scrapy.Spider):
    """docstring for cnbetaSpider"""
    name="cnbeta"
    allowed_domains=["cnbeta.com"]
    start_urls=[
        "http://www.cnbeta.com/topics/9.htm"
    ]

    def parse(self,response):
        for sel in response.xpath('//*[@class="all_news_wildlist"]/div[@class="items"]/div[@class="item"]'):
            title=sel.xpath('*[@class="hd"]/div[@class="title"]/a/text()').extract()
            link=sel.xpath('*[@class="hd"]/div[@class="title"]/a/@href').extract()
            desc=sel.xpath('*[@class="hd"]/*[@class="newsinfo"]/p/text()').extract()
            print(title,link,desc)
            
```

再次执行，能看到爬取到的网站信息被成功输出:

```
$ scapy crawl cnbeta
```

*** 

#### 结果

将之前设置的 `Item` 对象引入，使用标准的字典语法来保持获取到的每个字段的值。最终的代码为：

```
# coding:utf-8

import scrapy
from cnbetaSpider.items import CnbetaspiderItem

class cnbetaSpider(scrapy.Spider):
    """docstring for cnbetaSpider"""
    name="cnbeta"
    allowed_domains=["cnbeta.com"]
    start_urls=[
        "http://www.cnbeta.com/topics/9.htm"
    ]

    def parse(self,response):
        items=[]
        for sel in response.xpath('//*[@class="all_news_wildlist"]/div[@class="items"]/div[@class="item"]'):
            item=CnbetaspiderItem()
            item['title']=sel.xpath('*[@class="hd"]/div[@class="title"]/a/text()').extract()
            item['link']=sel.xpath('*[@class="hd"]/div[@class="title"]/a/@href').extract()
            item['desc']=sel.xpath('*[@class="hd"]/*[@class="newsinfo"]/p/text()').extract()
            # yield item
            items.append(item)

        return items
```

*** 

如果想把得到的结果保存在临时文件中，可以：

```
$ scrapy crawl cnbeta > abc.html
```

这样就把结果保存在当前目录下的 `abc.html` 中了。

*** 

#### Scrapy爬虫运行时报错 “Forbidden by robots.txt”

解决该问题只需要将 `settings.py` 文件中的 `ROBOTSTXT_OBEY` 值改为 `False` 即可。

```
ROBOTSTXT_OBEY = False
```

#### 相关链接

* [Scrapy 0.25 文档 &mdash; Scrapy 0.24.1 文档](http://scrapy-chs.readthedocs.io/zh_CN/latest/index.html)
* [Scrapy 1.0 文档(未完成,只更新了intro部分,请谨慎参考) &mdash; Scrapy 1.0.5 文档](http://scrapy-chs.readthedocs.io/zh_CN/1.0/index.html)
* [使用Scrapy抓取数据](https://segmentfault.com/a/1190000000583419)
* [爬虫出现Forbidden by robots.txt - 菜鸡瞎讲- 博客频道 - CSDN.NET](http://blog.csdn.net/zzk1995/article/details/51628205)
* [Scrapy用Cookie实现模拟登录 - 简书](http://www.jianshu.com/p/887af1ab4200)
