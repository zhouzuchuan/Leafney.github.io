---
title: C#脱离IronPython中执行python脚本
date: 2016-06-10 20:06:00
tags: 
    - ASP.NET
categories: 
    - 开发笔记
description: C#脱离IronPython中执行python脚本
---

给客户安装程序时除了安装 `.net framework` 还要安装 `IronPython` ，是不是觉得很麻烦？

上面这一切都弱爆了，下面我来介绍一种不安装 `IronPython` 只需要引入几个 `IronPython` 的 `dll` 就可以在c#中执行 `python` 脚本的方法。

1：引入IronPython中的几个dll

    * `IronPython.dll`
    * `IronPython.Modules.dll`
    * `Microsoft.Dynamic.dll`
    * `Microsoft.Scripting.dll`
    * `Microsoft.Scripting.Metadata.dll`

2：进入IronPython的Lib文件夹，把Lib中的内容打包成zip，名字任意既可。打包好后放到c#项目下
我把它放到了和py文件同一个目录中 

3：很关键的一步，程序初始化时执行下段代码

```
ScriptEngine engine = Python.CreateEngine(); 
ScriptScope scope = engine.CreateScope(); 
ScriptSource source = engine.CreateScriptSourceFromString( 
    @"import sys" "\n"  
    @"sys.path.append("".\scripts\pythonlib.zip"")" "\n"
    @"sys.path.append("".\scripts"")" "\n"
); 
source.Execute(scope);
```

将zip文件加入python库路径。这样能保证py脚本可以正确搜索到python库的位置。

4：尽情享用脚本语言带来的便利吧。为其他人安装程序时也不用安装讨厌的IronPython环境了。

*** 

链接

* [C#脱离IronPython中执行python脚本-gsbhzh 的个人空间 - 开源中国社区](http://my.oschina.net/gsbhz/blog/361140)
