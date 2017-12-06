---
title: Hexo博客功能优化
date: 2017-12-06 11:58:33
tags:
    - Hexo
categories: 
    - Hexo博客搭建
description: 介绍Hexo博客功能优化项，如 文章置顶、显示版权、访问统计、字数统计、显示更新时间等
---

介绍Hexo博客功能优化项，如 文章置顶、显示版权、访问统计、字数统计、显示更新时间等

#### 文章置顶

在Hexo博客中，有时候我们想要将一些特别的文章一直置顶在首页。Hexo博客中，默认的情况是按照时间倒序来排列的，即新发布的文章排在前面。虽然有一种很简单的方法，就是更改文章的发布时间到一个“未来”的时间点，这样虽然能让文章一直置顶，但是给人的体验和感觉是非常不好的。

今天介绍一种非常简单而且体验上也非常好的方法。

##### 安装node插件

```
$ npm uninstall hexo-generator-index --save
$ npm install hexo-generator-index-pin-top --save
```

##### 添加标记

在需要置顶的文章的 `Front-matter` 中加上 `top: true` 即可。

比如：

```
---
title: 从0到1学Golang之基础--Go 数组
date: 2017-05-24 22:07:58
tags:
    - Golang
categories: 
    - 从0到1学Golang
description: Golang下的数组操作
top: true
---

```

ok，现在发布文章，就能看到我们设置的文章已经置顶显示了，即使是之前发布的文章，同时日期也不会被更改。

##### 相关参考

* [解决Hexo置顶问题](http://www.netcan666.com/2015/11/22/%E8%A7%A3%E5%86%B3Hexo%E7%BD%AE%E9%A1%B6%E9%97%AE%E9%A2%98/)
* [hexo-generator-index-pin-top](https://github.com/netcan/hexo-generator-index-pin-top)
* [使用Hexo基于GitHub Pages搭建个人博客（三）](https://ehlxr.me/2016/08/30/%E4%BD%BF%E7%94%A8Hexo%E5%9F%BA%E4%BA%8EGitHub-Pages%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2%EF%BC%88%E4%B8%89%EF%BC%89/)
* [如何置顶post？](https://github.com/iissnan/hexo-theme-next/issues/415)

***

#### 显示版权信息

一般在网络上发表文章时，都要时刻提防着网络爬虫的抓取。特别是有些网站在抓取到你的文章后进行一些词语、段落的修改，公然改为自己发表的文章。完全无视原作者的辛苦。

为了更好的标明文章的版权，一般我们都会在文章中添加上文章的链接、版权声明等信息，虽然不能完全彻底的抵制文章抄袭的情况，也算是“防君子不防小人”吧。

##### 启用版权

我使用的是 `Hexo` 的 `Next` 主题。找到主题目录下的 `_config.yml` 文件，更改以下部分：

```
# Declare license on posts
post_copyright:
  enable: false
  license: CC BY-NC-SA 3.0
  license_url: https://creativecommons.org/licenses/by-nc-sa/3.0/

```

将其中的 `enable: false` 改为 `enable: true` 即可。

但是改完后，使用 `hexo s -g` 预览，发现 “本文链接” 部分有问题。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171206/Blk7gcB7CI.png?imageslim)

这就需要我们修改主站点的配置文件了。打开主站点的 `_config.yml` 文件，修改：

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171206/LC91BEHEcg.png?imageslim)

将 `url` 部分改成自己站点的域名地址即可。

##### 相关参考

* [Hexo持续优化-在文章尾部添加版权声明信息](http://www.crocutax.com/2017/05/20/Hexo%E6%8C%81%E7%BB%AD%E4%BC%98%E5%8C%96-%E5%9C%A8%E6%96%87%E7%AB%A0%E5%B0%BE%E9%83%A8%E6%B7%BB%E5%8A%A0%E7%89%88%E6%9D%83%E5%A3%B0%E6%98%8E%E4%BF%A1%E6%81%AF/)

***

#### 访问统计功能

在博客中我们一般都比较在意自己博客的访问量，或者哪篇文章比较受欢迎之类的。

在Hexo的 `Next` 主题下带有多种统计和分析的功能。这里我选择 `不蒜子统计`来显示文章的访客数、浏览量等信息。

##### 启用统计

找到 `Next` 主题下的配置文件 `_config.yml` ，找到 `busuanzi_count` 部分：

```
# Show PV/UV of the website/page with busuanzi.
# Get more information on http://ibruce.info/2015/04/04/busuanzi/
busuanzi_count:
  # count values only if the other configs are false
  enable: true
  # custom uv span for the whole site
  site_uv: true
  site_uv_header: 访客数 <i class="fa fa-user"></i>
  site_uv_footer: 人次
  # custom pv span for the whole site
  site_pv: true
  site_pv_header: 访问量 <i class="fa fa-eye"></i>
  site_pv_footer: 次
  # custom pv span for one page only
  page_pv: true
  page_pv_header: 阅读量 <i class="fa fa-file-o"></i>
  page_pv_footer: 次

```

当 `enable: true` 时，代表开启全局开关。

当 `site_uv: true` 时，代表在页面底部显示站点的UV值。
当 `site_pv: true` 时，代表在页面底部显示站点的PV值。
当 `page_pv: true` 时，代表在文章页面的标题下显示该页面的PV值（阅读数）。

##### 相关参考

* [不蒜子统计](http://theme-next.iissnan.com/third-party-services.html#analytics-busuanzi)

***

#### 显示文章更新时间

在文章列表中我们一般都能看的文章的发布时间。对于一些文章来说，比如涉及到文章中的内容过期，或者软件的升级等等，我们都会进行一些修改。这种情况下，我们就像把文章的更新日期也显示处理，也能让读者看的我们写的之前的文章也是有更新的，不会过时的。

##### 显示更新日期

在 `Next` 主题下添加显示更新时间非常简单，找到主题下的配置文件 `_config.yml` 的 `post_meta` 部分：

```
# Post meta display settings
post_meta:
  item_text: true
  created_at: true
  updated_at: false
  categories: true

```

将 `updated_at: false` 修改为 `updated_at: true` 即可。 

通过 `hexo s -g` 预览，可以看到已经自动添加上了更新日期。

##### 自定义显示更新日期

对于某些特殊的文章，我们也想能够自定义这个更新的日期。当然，更改起来也非常的简单，Hexo默认就支持更新日期的配置。

在每一篇文章的 `Front-matter` 部分，只要添加 `updated` 参数即可。

```
---
title: 从0到1学Golang之基础--Go 数组
date: 2017-05-24 22:07:58
updated: 2017-12-01 10:35:18
tags:
    - Golang
categories: 
    - 从0到1学Golang
description: Golang下的数组操作
---
```

这样我们就自定义了这篇文章的更新时间。

##### 相关参考

* [Front-matter](https://hexo.io/zh-cn/docs/front-matter.html)

***

#### 添加文章字数统计

一般为了让读者大概估计阅读文章的时间，有的文章在头部会显示总的字数统计。

##### 启用字数统计

首先安装一个依赖插件：

```
npm i --save hexo-wordcount
```

然后修改主题配置文件 `_config.yml` 中的 `post_wordcount` 部分：

```
# Post wordcount display settings
# Dependencies: https://github.com/willin/hexo-wordcount
post_wordcount:
  item_text: true   //底部是否显示“总字数”字样
  wordcount: false  //文章字数统计
  min2read: false  //文章预计阅读时长（分钟）
  totalcount: false  //网站总字数，位于底部
  separated_meta: true //是否将文章的字数统计信息换行显示
```

将 `wordcount: false` 改为 `wordcount: true` 即可显示单篇文章的总字数了。
另外，`totalcount` 是用来统计整站总的字数的。

##### 相关参考

* [hexo-wordcount](https://github.com/willin/hexo-wordcount)
* [畅玩Hexo——2：骚起来吧，NexT](https://zcore.coding.me/%E7%95%85%E7%8E%A9Hexo%E2%80%94%E2%80%942%EF%BC%9A%E9%AA%9A%E8%B5%B7%E6%9D%A5%E5%90%A7%EF%BC%8CNexT/)
