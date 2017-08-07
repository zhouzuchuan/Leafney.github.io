---
title: Docker+Flask搭建微信公众平台之一
date: 2016-09-24 22:02:06
tags:
	- Docker
	- Flask
	- Python
categories: 
	- 微信公众平台
description: 在虚拟机中的Docker容器内结合Flask来配置微信公众平台的本地开发环境
---

之前申请的个人公众号在申请通过后用.NET开发了一版比较简单的交互逻辑功能，最近在知乎中看到有关Python开发微信公众号的文章，正好前几天也在学习Docker技术，所以就想研究一下在Docker下如何用Flask配置微信公众平台的开发环境。

因为我的公众号是 **个人号且是未认证** 的，可获得的权限有限，所以目前我只做了消息的发送和接收相关的功能，更多功能后期再考虑加入。

下面测试Demo是申请的微信公众平台的测试号来开发的。

#### 环境配置

* **主系统** 为 `Win10`
* **宿主机** 为 在 `Win10` 下的 `Hyper-V` 虚拟机中安装的 `Ubuntu 14.04 LTS` 系统 宿主机ip地址为 `http://192.168.137.219/`
* **测试Docker容器** 为在 `宿主机` 下安装的 `Ubuntu 14.04 LTS` 系统 
* 使用的软件是 `Sublime Text` 和 `Xshell 4` 、`FileZilla`
* Python环境为 `2.7.12`

这里提一点，如果你不知如何配置Docker环境，或从未接触过Docker技术，在我之后的文章中我会有详细的介绍，欢迎关注。

*** 

#### 安装流程

##### 配置 Flask 运行环境

在 宿主机中，当前的账户为 `tiger`。在主目录下创建 `xweixin` 目录，添加的 `Flask` 程序为 `app.py` 文件。 

启动一个前台运行的容器，将宿主机目录 `xweixin` 映射到容器的 `weixin` 目录：

```
docker run -it --name weixin01 -p 80:80 -v /home/tiger/xweixin:/weixin ubuntu /bin/bash
```

查看容器中主目录下是否存在 `weixin` 目录，及该目录下是否存在 `app.py` 文件：

```
$ ls 
 ****** weixin
```

**注意：** 下面的操作主要在 `测试Docker容器` 中进行，因为在Docker中默认是 `root` 账户，所以下面的命令前面都没有加 `sudo` 。

设置安装源：

```
$ echo "deb http://archive.ubuntu.com/ubuntu/ precise universe" >> /etc/apt/sources.list
```

更新，安装python ：

```
$ apt-get update

$ apt-get install python -y  # 默认安装的是 2.7.12
```

安装 virtualenv : (由于是在容器内操作，只用来搭建Flask的运行环境，所以可以不安装虚拟环境)

```
$ apt-get install python-virtualenv
```

安装 pip ：

```
$ apt-get install python-pip -y
```

安装 flask:

```
$ pip install Flask
```

安装 vim ：(主要用于查看)

```
$ apt-get install vim
```

总结上面的操作：从 `Win10` 系统创建 flask 程序 `app.py` 后，通过 SFTP 传输到 宿主计算机的 `/home/tiger/xweixin` 目录下，该目录直接映射到了容器的 `/weixin` 目录。

app.py 内容 ：

```
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    app.run(host='0.0.0.0',port=80,debug=True)
```

在容器中执行 `python app.py` 运行。

在 `主系统Win10` 下访问 宿主机 的ip地址 `http://192.168.137.219/`，看到输出 `Hello World!` 说明Flask配置成功。

*** 

##### 使用 ngrok 实现外网访问

`ngrok.exe` 程序可以从 [ngrok - download](https://ngrok.com/download) 下载。

在 `主系统Win10` 下使用 `CMD` 命令运行 `ngrok` 工具，执行如下命令：

```
> ngrok http 192.168.137.219:80
```

后面的 ip地址不要带 `http://` , 端口号不要丢。

可以看到 ngrok 生成了一个 外网可以访问到的地址 `http://c62f4a09.ngrok.io/` ,在浏览器中打开，看到输出 `Hello World!` 说明可以实现外网访问了。

*** 

#### 搭建微信公众平台处理逻辑

##### 准备工作

由于 `ngrok` 每次重新启动就会重新分配一个新的域名地址，所以我们在启动ngrok之后，就不需要再去管它了。Flask中启用了 `debug=True` 的选项后，每次更新文件 `app.py` 都会自动重新加载(除非程序报错导致异常退出)，所以这样下来我们只需要负责修改 `app.py` 改完了上传到宿主机，直接刷新即可，不需要再进行重启ngrok，重新执行 `python app.py`启动程序等操作，非常方便。

修改 `app.py` 的框架，假设网站的二级目录 `/weixin` 来实现我们的微信公众平台的接口操作。

```
# coding:utf-8
"""
微信公众平台
"""
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
	return 'Hello World!'

# 微信公众平台接口
@app.route('/weixin')
def weixin_interface():
	return "这是微信接口"

if __name__ == '__main__':
	app.run(host='0.0.0.0',port=80,debug=True)
```

通过 `ngrok` 分配的域名为 `http://d0dd1f96.ngrok.io/weixin` ,该域名用于之后填入 微信公众平台 的接口配置的URL处(此时如果尝试添加，在微信公众平台页面一直会报配置错误的问题，因为在点击确定后微信会向我们的服务器发送验证消息，只有验证通过后才能保存)。

*** 

##### 处理逻辑

###### 在 Flask 中通过不同的 `method` 处理微信的验证请求和交互请求

```
# 微信公众平台接口
@app.route('/weixin',methods=['GET','POST'])
def weixin_interface():
	# return "这是微信接口"
	if request.method=='GET':
		# 处理验证
	else:
		# 处理逻辑交互

```

###### 设置微信验证请求 - 验证消息的确来自微信服务器

修改 `weixin_interface` 代码如下:

```
....
省略
....

from flask import Flask
from flask import request
import hashlib
import time


# 配置参数
X_TOKEN='leafney' #这里改写你在微信公众平台里输入的token

....
省略
....

# 微信公众平台接口
@app.route('/weixin',methods=['GET','POST'])
def weixin_interface():
	# return "这是微信接口"
	if request.method=='GET':
		# 处理验证

		# 接收参数
		wx_signature=request.args.get('signature')
		wx_timestamp=request.args.get('timestamp')
		wx_nonce=request.args.get('nonce')
		wx_echostr=request.args.get('echostr')
		# 自己的token
		wx_token=X_TOKEN
		# 字典序排序
		w_list=[wx_token,wx_timestamp,wx_nonce]
		w_list.sort()
		# sha1加密算法
		w_sha1=hashlib.sha1()
		map(w_sha1.update,w_list)
		w_hashcode=w_sha1.hexdigest()

		# 如果是来自微信的请求，则返回echostr
		if w_hashcode == wx_signature:
			return wx_echostr

	else:
		# 处理逻辑交互
		pass


```

因为我用的是微信的测试号来进行配置，将程序上传并启动之后，在 “测试号配置” 页面中的 “接口配置信息” 处 填入 `URL` 和 `Token` 然后点击 “确定” 。

如果程序接入成功，会提示 “配置成功” ，否则可以在之前配置的 `测试Docker容器` 下查看调试信息，来进行相应的修改。

*** 

###### 实现业务逻辑

​当普通微信用户向公众账号发消息时，微信服务器将POST消息的XML数据包到开发者填写的URL上。

接收消息内容，在 Flask 中可以使用 `request.data` 来获取：

```
xml_data=request.data
```

根据微信公众平台开发文档的说明 [接收普通消息 - 微信公众平台开发者文档](http://mp.weixin.qq.com/wiki/10/79502792eef98d6e0c6e1739da387346.html) 我们可以通过 `MsgType` 参数来区分接收的消息的类型，

例如文本消息的数据包如下：

```
 <xml>
 <ToUserName><![CDATA[toUser]]></ToUserName>
 <FromUserName><![CDATA[fromUser]]></FromUserName> 
 <CreateTime>1348831860</CreateTime>
 <MsgType><![CDATA[text]]></MsgType>
 <Content><![CDATA[this is a test]]></Content>
 <MsgId>1234567890123456</MsgId>
 </xml>
```

* ToUserName 表示消息的接收者
* FromUserName 表示消息的发送者
* CreateTime 表示消息创建的时间戳
* MsgType 表示消息的类型
* Content 表示消息的内容
* MsgId 表示消息的id，可以用来对消息排重

我们可以使用 `lxml` 来解析xml文档，获取相应的参数值。

添加引用：

```
import lxml
from lxml import etree
```

解析得到所需的参数：

```
wx_xml=etree.fromstring(xml_data) # 进行xml解析
print(etree.tostring(wx_xml,pretty_print=True))  # 获取请求内容

# 获取请求参数
wx_msgType=wx_xml.find('MsgType').text
wx_fromUser=wx_xml.find('FromUserName').text  # 微信公众号
wx_toUser=wx_xml.find('ToUserName').text # 用户
```

在向微信服务器回复消息时，也要按照微信的规定来返回特定XML结构的数据，详细可见 [被动回复用户消息 - 微信公众平台开发者文档](http://mp.weixin.qq.com/wiki/14/89b871b5466b19b3efa4ada8e577d45e.html) ，比如回复文本消息格式如下：

```
<xml>
<ToUserName><![CDATA[toUser]]></ToUserName>
<FromUserName><![CDATA[fromUser]]></FromUserName>
<CreateTime>12345678</CreateTime>
<MsgType><![CDATA[text]]></MsgType>
<Content><![CDATA[你好]]></Content>
</xml>
```

在向微信端回复消息时，`ToUserName` 和 `FromUserName` 我们可以从接收时的消息内容中来得到，只需要把接收者和发送者的角色互换一下即可，`Content` 为我们要回复的内容。

详细的处理代码如下：

```
		# 根据请求类型来返回不同的处理结果
		if wx_msgType == 'text':
			# 文本消息

			# 获取文本消息内容
			wx_content=wx_xml.find('Content').text
			content=wx_content.encode('utf-8')

			print(content)
			if content == '天气':
				# return '北京天气挺好的！'
				# 注意回复消息时，接收者和发送者的位置要互换一下
				return TextReply(wx_fromUser,wx_toUser,u'北京天气挺好的！').render()

			else:
				return TextReply(wx_fromUser,wx_toUser,wx_content).render()

		elif wx_msgType == 'image':
			return 'success'
		else:
			return 'success'
```

消息模板：

```
class TextReply(object):
	"""回复文本消息"""

	TEMPLATE=u"""
	<xml>
    <ToUserName><![CDATA[{target}]]></ToUserName>
    <FromUserName><![CDATA[{source}]]></FromUserName>
    <CreateTime>{time}</CreateTime>
    <MsgType><![CDATA[text]]></MsgType>
    <Content><![CDATA[{content}]]></Content>
    </xml>
	"""

	def __init__(self, target,source,content):
		self.target=target
		self.source=source
		self.content=content
		self.time=int(time.time())

	def render(self):
		return TextReply.TEMPLATE.format(target=self.target,source=self.source,time=self.time,content=self.content)

```

至此，接收回复文本消息的功能我们就做好了，上传到 `宿主机` 的 `xweixin` 目录下即可。

***

在运行代码之前，先要安装依赖包 `lxml` ,不然会报 `No module named lxml` 的错误。

##### 安装 lxml

先安装 lxml 的依赖包：

```
$ apt-get install python-dev libxml2-dev libxslt1-dev zlib1g-dev 
```

再安装 lxml :

```
$ pip install lxml
```

*** 

##### 完整的代码实现

app.py (主程序)

```
# coding:utf-8
"""
微信公众平台
"""

from flask import Flask
from flask import request
import hashlib
import lxml
from lxml import etree
import time
 
# 消息返回模板
from reply import TextReply

# 配置参数
X_TOKEN='leafney' #这里改写你在微信公众平台里输入的token


app = Flask(__name__)

@app.route('/')
def hello_world():
	return 'Hello World!'

# 微信公众平台接口
@app.route('/weixin',methods=['GET','POST'])
def weixin_interface():
	# return "这是微信接口"
	if request.method=='GET':
		# 处理验证

		# 接收参数
		wx_signature=request.args.get('signature')
		wx_timestamp=request.args.get('timestamp')
		wx_nonce=request.args.get('nonce')
		wx_echostr=request.args.get('echostr')
		# 自己的token
		wx_token=X_TOKEN
		# 字典序排序
		w_list=[wx_token,wx_timestamp,wx_nonce]
		w_list.sort()
		# sha1加密算法
		w_sha1=hashlib.sha1()
		map(w_sha1.update,w_list)
		w_hashcode=w_sha1.hexdigest()

		# 如果是来自微信的请求，则返回echostr
		if w_hashcode == wx_signature:
			return wx_echostr

	else:
		# 处理逻辑交互

		xml_data=request.data # 获得post来的数据
		wx_xml=etree.fromstring(xml_data) # 进行xml解析
		# print(etree.tostring(wx_xml,pretty_print=True))  # 获取请求内容
		
		# 获取请求参数
		wx_msgType=wx_xml.find('MsgType').text
		wx_fromUser=wx_xml.find('FromUserName').text  # 微信公众号
		wx_toUser=wx_xml.find('ToUserName').text # 用户

		# 根据请求类型来返回不同的处理结果
		if wx_msgType == 'text':
			# 文本消息

			# 获取文本消息内容
			wx_content=wx_xml.find('Content').text
			content=wx_content.encode('utf-8')

			print(content)
			if content == '天气':
				# return '北京天气挺好的！'
				# 注意回复消息时，接收者和发送者的位置要互换一下
				return TextReply(wx_fromUser,wx_toUser,u'北京天气挺好的！').render()

			else:
				return TextReply(wx_fromUser,wx_toUser,wx_content).render()

		elif wx_msgType == 'image':
			return 'success'
		else:
			return 'success'


if __name__ == '__main__':
	app.run(host='0.0.0.0',port=80,debug=True)
```

reply.py (消息模板)

```
# coding:utf-8
"""
微信公众平台 消息回复模板
"""
import time

class TextReply(object):
	"""回复文本消息"""

	TEMPLATE=u"""
	<xml>
    <ToUserName><![CDATA[{target}]]></ToUserName>
    <FromUserName><![CDATA[{source}]]></FromUserName>
    <CreateTime>{time}</CreateTime>
    <MsgType><![CDATA[text]]></MsgType>
    <Content><![CDATA[{content}]]></Content>
    </xml>
	"""

	def __init__(self, target,source,content):
		self.target=target
		self.source=source
		self.content=content
		self.time=int(time.time())

	def render(self):
		return TextReply.TEMPLATE.format(target=self.target,source=self.source,time=self.time,content=self.content)

```

***

#### 未完待续

至此，上面我们就实现了最基本的微信公众号的验证和简单文本消息的接收和回复功能。

之后我会介绍如何来处理复杂的消息内容，图片、语音、图文等等。然后通过一些有趣的小功能来让我们这即使没有特殊权限的公众号也能玩得更有意思，欢迎关注！

*** 

PublishTime: 2016-9-24 22:35:16
