Hexo博客

[![Build Status](https://travis-ci.org/Leafney/Leafney.github.io.svg?branch=hexo)](https://travis-ci.org/Leafney/Leafney.github.io)

#### 部署

##### 直接部署

下载项目文件后，在Hexo博客项目根目录下即 `_config.xml` 文件所在目录下打开 `CMD` 命令窗口执行：

```
> npm install
``` 

等待安装hexo的依赖项。

##### 重新部署

```
> npm install hexo-cli -g

# 建站
> hexo init <folder>
> cd <folder>
> npm install
```

***

#### 基础操作

##### 查看hexo版本

```
> hexo -v
```

##### 创建新页面

```
> hexo new "blog name"
```

##### 创建新分类

```
> hexo new page "page name"
```

##### 清理public文件夹

```
> hexo clean
```

##### 生成

```
> hexo g
```

##### 预览

```
> hexo s
```

##### 部署

```
> hexo d
```

##### 生成并预览

```
> hexo s -g
```

##### 生成并部署

```
> hexo d -g
```

***

#### Git分支操作

##### 查看本地所有分支(分之名称前面带*表示当前分支)

```
> git branch
```

##### 查看远程所有分支

```
> git branch -r
```

##### 创建分支 blog

```
> git branch blog
```

##### 切换到 blog 分支

```
> git checkout blog
```

##### 创建并切换到新分支

```
> git checkout -b blog
```

##### 删除分支

```
> git branch -d blog
```

##### 提交本地test分支作为远程的test分支

```
> git push origin test:test
```

##### merge合并分支(将名称为[blog]的分支与当前分支[master]合并)

```
> git branch
* master
  blog

> git merge blog
```

##### 获取远程指定分支

```
> git pull origin blog
```

***
