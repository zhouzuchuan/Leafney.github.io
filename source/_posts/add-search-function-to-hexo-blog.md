---
title: Hexo博客添加搜索功能
date: 2017-10-27 18:23:32
tags:
    - Hexo
    - Search
categories: 
    - Hexo博客搭建
description: 渐渐的随着hexo博客中文章越来越多了之后,平时想要查找一篇文章时,一般都是记得部分标题内容或者某些关键词,而在"归档"下通过标题一页一页的找又非常的麻烦.所以考虑为博客增加搜索功能.
---

渐渐的随着hexo博客中文章越来越多了之后,平时想要查找一篇文章时,一般都是记得部分标题内容或者某些关键词,而在"归档"下通过标题一页一页的找又非常的麻烦.所以考虑为博客增加搜索功能.

从Next主题网站上我们可以搜索多款搜索插件 [第三方服务集成 -- Next文档](http://theme-next.iissnan.com/third-party-services.html#search-system),个人感觉 "Local Search" 和 "Algolia" 这两款搜索插件比较和我的心意.

下面简要的介绍为hexo博客添加搜索插件的过程.

#### 升级Next主题版本

我的hexo主题安装的是 `Next` 主题,当前版本为 `v5.0.1` .最新版本为 `5.1.3`.

因为我的hexo主题版本差距太大,而且我在 `next` 主题目录下的 `.git` 目录我已经删除了,所以我采用完全更新的方式.

先备份本地的 `hexo` 主题目录 `your-hexo-site/themes/next` ,其实只需要备份 `_config.yml` 一个文件即可,为了保险这里我将整个目录都备份一下.

另外还要注意如果你之前添加了自定义头像及打赏功能等所需图片,添加的图片是在 `/themes/next/source/images/` 目录下的,也要记得做好备份.

***

如果你本地的主题 `next` 目录下的 `.git` 目录没有删除,你可以直接通过 `git pull` 命令来更新:

```
$ cd themes/next
$ git pull
```

##### 完全更新

先删除本地现在的 `next` 主题目录:

```
$ cd <your-hexo-site>
$ rm -rf ./themes/next
```

从github下载 hexo 主题文件最新版本:

```
$ cd <your-hexo-site>
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

比照新的主题配置文件 `_config.yml` ,将旧的主题文件中的内容添加到新文件中.

我这里更改的地方有:

* `menu`
* `scheme`
* `social`
* `sidebar`
* `highlight_theme`
* `tencent_analytics`
* `打赏功能 reward_comment wechatpay alipay`
* `站点建立时间 since`

更改后,清除缓存,然后再查看:

```
$ hexo clean

$ hexo s -g
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```

打开浏览器,查看页面中是否有错误并修改.

***

#### LocalSearch搜索

安装 `hexo-generator-searchdb`，在站点的根目录下执行以下命令：

```
$ npm install hexo-generator-searchdb --save
```

编辑 `站点配置文件`，新增以下内容到任意位置：

```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

编辑 `主题配置文件`，启用本地搜索功能：

```
# Local search
local_search:
  enable: true
```

然后 重新生成 查看:

```
$ hexo clean

$ hexo s -g
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```

这样,搜索功能就添加上了.

***

#### Algolia搜索

详情可参考官方教程,这里不再详述. 

* [Algolia -- Next文档](http://theme-next.iissnan.com/third-party-services.html#algolia-search)

***

#### 相关链接

* [搜索服务 -- Next文档](http://theme-next.iissnan.com/third-party-services.html#search-system)
* [hexo-generator-search](https://github.com/PaicHyperionDev/hexo-generator-search)
* [hexo-algoliasearch](https://github.com/LouisBarranqueiro/hexo-algoliasearch)
* [Hexo集成Algolia搜索插件](https://jobbym.github.io/2017/01/16/Hexo%E9%9B%86%E6%88%90Algolia%E6%90%9C%E7%B4%A2%E6%8F%92%E4%BB%B6/)
