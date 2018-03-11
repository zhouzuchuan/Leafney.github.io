---
title: Hexo博客功能优化二
date: 2018-03-11 16:35:36
updated: 2018-03-11 18:04:37
tags:
    - Hexo
categories: 
    - Hexo博客搭建
description: 介绍Hexo博客功能优化项。
---

介绍Hexo博客功能优化项。

#### 将“打赏”两字更改为“鼓励”

一直觉得 “打赏” 两个字不太适合博客这种语境，搞得像是杂耍完了向围观群众要赏钱的感觉：“有钱的捧个钱场，没钱的捧个人场...” 。写作，本来就是在学习技术的路上一次次总结，亦或和志同道合的技术人的一次次讨论，无关金钱或其他。
而“鼓励”则更适合这种语境。如果我的文章帮到了你，虽然不能改变世界，但也许为你节省了一些时间，亦或在你进入一个技术死角一直出不来的情况下，带来的一点点希望或是灵感。你觉得我的文章对你带来了帮助，送我一杯咖啡，我内心是非常高兴地。如果你还想多聊两句，我也会感到非常荣幸。

相应的修改方法是：

找到项目目录下 `/themes/next/languages/zh-Hans.yml` 文件，因为我的博客采用的是中文。这里可以按照自己的博客设置选择相应的语言文件。

```
reward:
  donate: 鼓励 #打赏
  wechatpay: 微信支付
  alipay: 支付宝
  bitcoin: 比特币
```

找到 `reward` 部分，将 `donate` 的值修改为自己想要的内容即可。这里我测试了一下，最长是4个汉字，否则就要修改按钮的样式了。

***

#### 关于评论

自此评论插件 “多说” 关闭之后，我就把博客中的评论功能去掉了，因为确实没有找到一款心仪的评论插件。

目前的话，也只是在博客的 “关于” 页面加了一个 `email` 地址能够立即联系到我。

因为我的邮箱手机客户端是24小时在线的，让我感到高兴的是确实还有一些朋友通过邮件向我咨询技术问题，我都一一为他们做了解答。之前还觉得这个邮箱地址放在“关于” 页面会比较隐蔽，今天稍微改版了一下，在每篇文章的末尾都加上了 `email` 地址，以方便交流。

这段提示文字我是直接加在了 `next` 主题配置文件 `_config.yml` 中的 `Reward 打赏功能` 部分。

原来的打赏功能提示文字 `reward_comment` 参数，如果添加的字符太多的话，就会导致自动换行。所以这里我修改了一下页面文件。

找到目录下 `/themes/next/layout/_macro/reward.swig` 文件，找到第二行的 

```
<div>{{ theme.reward_comment }}</div>
```

部分再复制一行，如下：

```
<div style="padding: 10px 0; margin: 20px auto; width: 90%; text-align: center;">
  <div>{{ theme.reward_comment }}</div>
  <div>{{ theme.reward_comment2 }}</div>
  <button id="rewardButton" disable="enable" onclick="var qr = document.getElementById('QR'); if (qr.style.display === 'none') {qr.style.display='block';} else {qr.style.display='none'}">
    <span>{{ __('reward.donate') }}</span>
  </button>
  <div id="QR" style="display: none;">
...
...
```

我这里就直接改成了 `theme.reward_comment2` 。然后在主题的配置文件中也添加一个 `reward_comment2` 部分即可。

```
# Reward 打赏功能
reward_comment: 坚持原创技术分享，您的支持将鼓励我继续创作！
reward_comment2: 如有疑问或需要技术讨论，请发邮件到 service@itfanr.cc
wechatpay: /images/wechat-reward-image.jpg
...
```
***

#### 页面载入进度

这个效果也是刚刚查看配置文件的时候偶然看到的。

之前就曾看别人的博客做的特别炫。页面头部可以显示一个加载进度条，非常的羡慕。

在 `next` 主题的配置文件 `_config.yml` 中 找到 `pace: false` 将其改为 `pace: true` 即可。

在下面可以选择不同的加载主题样式，通过 `pace_theme` 参数设置。

```
pace: true
# Themes list:
#pace-theme-big-counter
#pace-theme-bounce
#  ...
pace_theme: pace-theme-flash
```

***

#### 感谢支持

截止目前为止，共收到了3位朋友的扫码红包鼓励，在这里对他们表示感谢。也很高兴我的文章帮助到了他们。

不过由于微信扫码支付无法查看到支付者的微信账号信息，所以在这里就没有列出他们的昵称等信息。具体列表可查看 [关于](/about/#Thanks) 页面。

***
