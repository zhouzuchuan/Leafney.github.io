Hexo博客

#### 部署

##### 直接部署

下载项目文件后，在Hexo博客项目根目录下即 `_config.xml` 文件所在目录下打开 `CMD` 命令窗口执行：

```
> npm install
``` 

等待安装hexo的依赖项。

##### 重新部署

```
> npm install -g hexo-cli

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

