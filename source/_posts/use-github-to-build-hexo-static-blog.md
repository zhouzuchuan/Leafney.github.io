---
title: 使用GitHub搭建Hexo静态博客
date: 2016-9-24 14:06:00
tags:
    - GitHub
    - Hexo
categories: 
    - Hexo博客搭建
description: 使用Hexo搭建静态博客并部署到GitHub
---

#### 说明

之前一直在用自己创建的.Net网站来写博客文章，只有简单的CRUD的功能，数据也是从数据库中直接查询的。自从接触了markdown之后，渐渐习惯了用markdown来记录自己在开发过程中遇到的问题和学到的新知识。但由于之前博客中是通过百度的 `ueditor` 编辑器来编辑文章，不能直接处理markdown，所以后来就一直考虑有没有其他的方法来更方便的管理和发布博客。

自从发现了 Hexo，不得不说这不就是我一直想要找的博客工具吗？

静态博客的特点如下：
   
* 不用数据库
* 访问速度快
* 支持markdown，更加注重博客内容

而且部署到 GitHub 上，我们还不需要去配置服务器，非常的方便。

下面详细记录我这个博客的搭建部署流程，希望能帮到有需要的朋友们！

#### 搭建流程

##### 开始前保证已安装：

* Node.js
	* Linux 下执行如下命令来安装：
		* `$ curl https://raw.github.com/creationix/nvm/master/install.sh | sh`
		* `$ nvm install stable`
	* Windows 下可以从 [Node.js Download](https://nodejs.org/en/download/) 处选择 `Windows Installer` 下载msi安装包
* Git
	* Linux(Ubuntu) 下执行如下命令来安装：
		* `$ sudo apt-get install git-core`
	* Windows 下可以从 [Git Download](https://git-scm.com/download/win) 处下载 `Git for Windows` 安装包

##### 安装Hexo

选择一个用来创建博客的目录，在该目录下使用 `CMD` 命令行窗口，这里推荐使用 [Cmder](http://cmder.net/) 来代替Windows上默认的命令行。

通过 `npm` 命令执行：

```
> npm install -g hexo-cli
```

##### 建站

执行如下命令，Hexo 将会在指定文件夹中新建所需要的文件

```
> hexo init <folder>
> cd <folder>
> npm install
```

这里假设我要创建的博客所在目录名为 `xblog` ,则命令为：

```
> hexo init xblog
> cd xblog
> npm install
```

##### 生成静态页

```
> hexo g
```

该命令的完整格式为:  `hexo generate`

##### 启动服务

```
> hexo s
```

该命令的完整格式为： `hexo server`

hexo默认使用 `4000` 端口进行预览，可以通过浏览器打开 `http://localhost:4000/` 查看。如果端口被占用，可以用 `hexo s -p 5000`  指定端口号为 `5000` 或者其他未被占用的端口。使用 `Ctrl+C` 来停止预览。

##### 查看Hexo版本

可以通过命令 `hexo -v` 查看当前hexo的版本信息：

```
> hexo -v
hexo: 3.2.2
hexo-cli: 1.0.2
os: Windows_NT 10.0.10586 win32 x64
http_parser: 2.5.2
node: 4.4.0
v8: 4.5.103.35
uv: 1.8.0
zlib: 1.2.8
ares: 1.10.1-DEV
icu: 56.1
modules: 46
openssl: 1.0.2g
```

*** 

#### Hexo配置

##### 更换主题 theme

以主题 [NexT](http://theme-next.iissnan.com/) 为例，首先下载该主题：

```
> cd your-hexo-site (我这里是 `cd xblog`)
> git clone https://github.com/iissnan/hexo-theme-next themes/next
```

在站点站点根目录下打开 `_config.yml`，找到 `theme` 字段，将 `theme: landscape` 改为 `theme: next` ，然后再次执行 `hexo g` 来重新生成。

##### 主题设置

关于主题的配置可直接参考NexT官网的配置流程 [开始使用-NexT](http://theme-next.iissnan.com/getting-started.html) ，这里我会把自己在配置时的操作简单记录，仅供大家参考。

在 Hexo 中有两份主要的配置文件，其名称都是 `_config.yml`。 其中，一份位于站点根目录下，主要包含 Hexo 本身的配置；另一份位于主题目录下，这份配置由主题作者提供，主要用于配置主题相关的选项。

为了描述方便，在以下说明中，将前者称为 `站点配置文件`， 后者称为 `主题配置文件`。

***

###### 选择Scheme

借助于 Scheme，NexT 提供了多种不同的外观。目前 NexT 支持三种 Scheme ：

* Muse - 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白
* Mist - Muse 的紧凑版本，整洁有序的单栏外观
* Pisces - 双栏 Scheme，小家碧玉似的清新

Scheme 的切换通过更改 `主题配置文件` ，搜索 `scheme` 关键字。 你会看到有三行 `scheme` 的配置，将你需要启用的 `scheme` 前面注释 `#` 去掉即可。

```
#scheme: Muse
#scheme: Mist
scheme: Pisces
```

***

###### 设置 语言

编辑 `站点配置文件`， 将 `language` 设置成你所需要的语言。这里我选择了简体中文，配置如下：

```
language: zh-Hans
```

***

###### 设置 菜单

菜单即是你的导航菜单，我们可以设置菜单项的 `显示名称`和 链接 以及 `菜单项对应的图标` 。

我们可以在 `主题配置文件` 中来修改，对应的字段是 `menu` 。我的配置如下：

```
menu:
  home: /   				# 主页
  categories: /categories   # 分类页
  tags: /tags   			# 标签页
  archives: /archives  		# 归档页
  about: /about 			# 关于页面
```

菜单项的显示文本放置在 `NexT` 主题目录下的 `languages/{language}.yml` （`{language}` 为你所使用的语言）。

菜单项的图标，对应的字段是 `menu_icons`。我们可以通过 `enable` 来控制是否显示图标。

我们也可以添加自己的链接，只要把上面的 `名称-链接-图标` 三者对应好即可。

> 请注意键值（如 `home`）的大小写要严格匹配

***

###### 设置 头像

编辑 `站点配置文件`， 新增字段 `avatar`， 值设置成头像的链接地址。

我们可以将自己的头像图片文件上传到主题目录下的 `source/uploads/` 目录下(新建uploads目录若不存在) 或 `source/images/` 目录下。完整的目录就是 `your-hexo-site\themes\next\source\images\` 下。

然后将 `avatar` 字段配置如下：

```
# avatar 个人头像
avatar: /images/avatar.jpg
```

或者也可以使用网络上的头像来进行设置：

```
avatar: http://example.com/avatar.png
```

***

###### 设置 作者昵称及站点描述

编辑 `站点配置文件`， 设置 `author` 为你的昵称。

编辑 `站点配置文件`， 设置 `description` 字段为你的站点描述。

***

###### 设置代码高亮主题

NexT共提供了5款主题可以选择，默认使用白色的 `normal` 主题。

`normal`，`night`， `night blue`， `night bright`， `night eighties`

在 `主题配置文件` 中找到 `highlight_theme` 字段，更改即可。

```
highlight_theme: night eighties
```

***

###### 开启打赏功能

NexT可以支持微信打赏和支付宝打赏，在 `主题配置文件` 中添加如下字段即可：

```
# 打赏功能
reward_comment: 坚持原创技术分享，您的支持将鼓励我继续创作！
wechatpay: /images/wechat-reward-image.jpg
alipay: /images/alipay-reward-image.jpg
```

微信个人收款二维码可以从微信APP右上角“+”处，选择 `收付款` -- `我要收款` -- `长按二维码` 来获取。
支付宝个人收款二维码可以从支付宝APP的右上角“+”处，选择 `我的二维码/收款` 来获取。

二维码图片可以和头像一样保存在 `source/images/` 目录下。

***

##### 主题栏目设置

在上面设置导航菜单时我们已经添加了导航菜单: `categories` 和 `tags` ，但在预览时会直接报错，因为我们还没有创建这两个分类页面。

###### 添加「标签」页面

「标签」页面将展示站点的所有标签，若你的所有文章都未包含标签，此页面将是空的。

这里先详细介绍两个命令：

```
> hexo n about

> hexo n page about
```

`hexo n` 的完整格式为 `hexo new` ，所以上面的命令也可以写成：

```
> hexo new about

> hexo new page about
```

两者的区别是：

`hexo new xxxx` 表示创建一个新的文章页面。在Hexo中, 你写的博客文章会默认存储在 `your-hexo-site/source/_posts` 下。比如上面的命令 `hexo new about` 我们就在 `your-hexo-site\source\_posts` 目录下新建了一个 `about.md` 的文件。

`hexo new page xxxx` 表示创建一个新的分类主页面。比如上面的命令 `hexo new page about` 我们就在 `your-hexo-site\source\` 目录下创建了一个名为 `about` 的目录，该目录下有一个名为 `index.md` 的文件。

***

在命令行窗口下，定位到 `Hexo` 站点目录下。使用 `hexo new page` 新建一个分类主页面，命名为 `tags` ：

```
> cd your-hexo-site
> hexo new page tags
```

然后打开 `your-hexo-site\source\tags` 目录下的 `index.md` 文件，将页面的类型设置为 `tags` ，主题将自动为这个页面显示标签云。

```
---
title: 标签
date: 2016-09-24 01:05:02
type: "tags"
---
```

> 注意：如果有启用 `多说` 或者 `Disqus` 评论，页面也会带有评论。 若需要关闭的话，请添加字段 `comments` 并将值设置为 `false` ，如：

```
---
title: 标签
date: 2016-09-24 01:05:02
type: "tags"
comments: false
---
```

那么我们在新创建的文章页面中要如何来显示标签呢？

我们可以在文章页面中通过添加 `tags` 标记来设置文章要显示的标签。

Hexo 中有 `Front-matter` 这个概念，是文件最上方以 `---` 分隔的区域，用于指定个别文件的变量。

`Front-matter` 支持 “数组” 方式或 “yaml” 方式来设置，如下：

```
title: hexo index
date: 2016-09-24 01:05:02
tags: [github,html5,css3]
```

```
title: hexo index
date: 2016-09-24 01:05:02
tags: 
- github
- html5
- css3
```

***

###### 添加「分类」页面

「分类」页面将展示站点的所有分类，若你的所有文章都未包含分类，此页面将是空的。 

和上面配置 `tags` 的方法一样，我们可以通过命令 `hexo new page xxxx` 来创建：

```
> cd your-hexo-site
> hexo new page categories
```

> 这里需要注意的一点是，我们命令中的名称 `tags` 和 `categories` 是在之前配置导航菜单时在 `menu` 字段下指定的名称，大小写不能错。

相应的页面类型设置如下：

```
---
title: 分类
date: 2016-09-24 01:08:49
type: "categories"
comments: false
---
```

同样的，在文章页面中，我们可以通过 `categories` 字段来为文章指定所属的分类：

```
---
title: 分类测试文章
categories: Testing
---
```

***

#### 关于 Hexo 下的文章页面

上面已经提到过，我们可以通过 `hexo n xxxx` 或 `hexo new xxxx` 来创建新的文章页面，默认会保存在 `your-hexo-site\source\_posts` 目录下。

一般的文章页面头部的 `Front-matter` 格式如下：

```
---
title: hexo next  		    # 文章标题
date: 2016-09-24 01:08:49   # 文章发布日期
tags: [github,html5]
tags:
	- github
	- html5 			  # tags 文章的标签，可以通过数组方式或yaml方式指定
categories: 前端   		  # 文章所属分类
description: 这里是文章描述，大概140个字左右  # 文章描述

---

下面是文章正文 ...
```

##### 需要注意的是

* 冒号后面与内容直接要有一个空格，否则无法编译生成
* 正文与 `Front-matter` 之间要有一个空行
* 为文章添加描述文字除了指定 `description` 之外，还可以在正文中通过添加 `<!--more-->` 标签分隔内容

##### 常见问题

* 修改配置文件时注意YAML语法，参数冒号:后一定要留一空格
* 中文乱码请修改文件编码格式为UTF-8
* yml文件中所有有空格的字段都用双引号括起来

默认情况下，新建文章的文件名和标题名是相同的，需要注意的是如果文件名是中文，那么生成的链接后面会自动添加末尾斜杠 `http://localhost:4000/2016/09/xxxxxxx/` 如果手动去掉该斜杠 `/` 会直接报错。所以推荐文件名最好用 `英文` 来写，文章内的标题可以改成中文。

另外，**文章文件名中文末尾斜杠** 的问题也可以通过配置 `Nginx` 来解决。(这里暂不讨论该方法)

***

#### Hexo 中常用命令

```
hexo generate (hexo g) 	  # 生成静态文件，会在当前目录下生成一个新的叫做public的文件夹
hexo server (hexo s)  	  # 启动本地web服务，用于博客的预览
hexo deploy (hexo d)      # 部署播客到远端（比如github, heroku等平台）
hexo new "postName"       # 新建文章
hexo new page "pageName"  # 新建页面
hexo clean    		  	  # 清理public文件夹
```

简写形式：

```
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy
hexo d -g # 生成部署
hexo s -g # 生成预览
```

*** 

#### 将 Hexo 部署到 Github

到 GitHub 新建一个项目，项目名为：`你的用户名.github.io` **必须为这个名字**

Hexo 目前没有自带 Git 部署模块，需手动安装:

```
> npm install hexo-deployer-git --save
```

然后，在 `站点配置文件` `_config.yml` 中设置 `deploy` 字段：

```
deploy:
  type: git
  repository: git@github.com:你的帐号/你的帐号.github.io.git  #例如我的：repository: git@github.com:Leafney/Leafney.github.io.git
  branch: master
```

> 注意：这个respository的地址你在GitHub创建同名仓库后，会在页面中给出，直接复制即可。另外，`branch` 要设置为 `master` 分支。

在此之前，还要注意你本地的 Git 已经通过 `SSH` 和你的 GitHub 连接起来了。

如何通过SSH连接GitHub可以查看我的另一篇文章 **[使用SSH密钥连接Github](/2016/09/24/using-ssh-key-connection-github/)** 来进行设置。

***

然后执行命令：

```
> hexo clean  # 先清理public文件夹
> hexo g      # 生成
> hexo d      # 部署

# 或通过下面一条命令直接生成并部署
> hexo g -d
```

执行完成之后会在你的博客根目录下生成一个文件夹：`.deploy_git`， 该目录下的文件会自动被发布到你的 GitHub 上，页面文件在 `master` 分支下。

然后打开浏览器，输入你的 GitHub pages 的地址 `xxxxx.github.io` 即可。

***

PublishTime: 2016-9-24 14:06:00
