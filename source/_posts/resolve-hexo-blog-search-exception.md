---
title: 解决Hexo博客搜索异常
date: 2017-11-24 10:49:28
tags:
    - Hexo
categories: 
    - Hexo博客搭建
description: 解决Hexo博客使用Local Search搜索功能时一直显示加载中的问题
---

最近在更新博客文章时发现之前新添加的搜索功能不太好用了。每次点击了搜索按钮之后，搜索弹框一直显示 “加载中” 的状态。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171124/1KiH0C4BaA.png?imageslim)

#### 尝试

因为我使用的是 `hexo` 的 `Next` 主题中的 `Local Search` 搜索功能，所以就去 `Next` 主题的github中查找了类似的 `issues` ，发现类似问题下作者是建议重新安装该搜索组件来解决的。

于是我就卸载了该组件，然后重新安装：

```
$ npm uninstall hexo-generator-searchdb --save

$ npm install hexo-generator-searchdb --save

```

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171124/eHlhg3am05.png?imageslim)

结果问题依旧。

#### 探究

后来我发现 `Local Search` 的搜索功能是加载的项目目录下的 `search.xml` 文件：`http://localhost:4000/search.xml`。于是我在浏览器中打开，居然有报错提示。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171124/68cam2i3k8.png?imageslim)

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171124/jmJc73ClH5.png?imageslim)

按照错误信息的说明，我找到了出错的第 `47` 行第 `35` 列，发现和其他内容不同的是这里居然多了一个 “红点”，那么搜索弹窗出不来的问题应该就是这个 “红点” 搞的鬼了。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171124/imFDgH4mmE.png?imageslim)

我通过 `Sublime Text` 打开了源博客文件，发现在段落的开头居然多了两个奇葩的字符：`BS`。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171124/km78g4i39h.png?imageslim)

我觉得可能是什么时候复制文件时给加上的。删除后，再次生成。问题解决。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171124/06F5a7GEaL.png?imageslim)

再次访问搜索的xml文件 `http://localhost:4000/search.xml` ，发现已经不会再报之前的错误了。

![mark](http://ouej55gp9.bkt.clouddn.com/blog/171124/B85CK5774h.png?imageslim)


需要注意的一点：我在查看源博客文件时也使用了 `VS Code` 编辑器，但是 `VS Code` 却无法显示出来前面的特殊字符 `BS`，通过 `Sublime Text` 才查看到。

#### 扩展

在Hexo博客文件的项目目录下有一个 `node_modules` 目录。每次在windows系统下删除 (拷贝或者移动) 该目录时都会报 `文件名或扩展名太长，目录层次超过限制` 等错误而导致操作失败。

解决这个问题只需要使用 `unix` 或者 `linux` 下的 `rm -rf`（强制删除） 命令来删除即可，但要注意操作时一定要慎重，不要误删其他文件。

在 `node_modules` 文件夹所在目录下右键打开 `Git Bash` 窗口，执行：

```
$ rm -rf ./node_modules/
```

等待完成，即可。

#### 相关参考

* [how to uninstall npm modules in node js?](https://stackoverflow.com/questions/13066532/how-to-uninstall-npm-modules-in-node-js)
* [windows删除node_modules[文件名或扩展名太长，目录层次超过无法删除的问题]](http://blog.csdn.net/crper/article/details/50458369)
