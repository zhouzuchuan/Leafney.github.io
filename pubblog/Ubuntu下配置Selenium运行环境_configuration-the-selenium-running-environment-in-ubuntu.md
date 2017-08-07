### Ubuntu下配置Selenium运行环境

Selenium，自动化测试工具。


#### 配置环境

* Ubuntu 16.04 TLS


#### 安装依赖

##### 设置软件安装源

```
$ echo "deb http://cn.archive.ubuntu.com/ubuntu/ xenial main restricted universe multiverse" >> /etc/apt/sources.list
```

也可以改成：

```
$ echo "deb http://cn.archive.ubuntu.com/ubuntu/ xenial main restricted universe multiverse" > /etc/apt/sources.list
```

##### 安装依赖包

执行以下命令：

```
$ sudo apt-get update
$ sudo apt-get install python python-pip curl unzip -y
$ sudo pip install selenium
```

***

#### 支持的浏览器及 WebDriver 驱动

Selenium可以调用Chrome 、Firefox 、Safari 等浏览器。

WebDriver 支持以下的

* ChromeDriver
* EventFiringWebDriver
* FirefoxDriver
* HtmlUnitDriver
* InternetExplorerDriver
* PhantomJSDriver
* RemoteWebDriver
* SafariDriver

这里我以常用的Chrome 和 Firefox 为例，来配置运行环境。

我们需要安装相对应的浏览器和浏览器驱动 WebDriver 。比如 Selenium 无法直接启动 Chrome ，需要用第三方插件 ChromeDriver 来调用。

#### Selenium调用Chrome浏览器

Chrome浏览器我们可以使用官方的 Google Chrome 浏览器 或者 开源的 Chromium 浏览器

##### 安装 Google Chrome

执行如下命令安装，这里我们选择的是 `google-chrome-stable` 版本：

```
$ wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -

$ sudo sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list'

sudo apt-get update
sudo apt-get install google-chrome-stable
```

详细可访问：

* [UbuntuUpdates-Google Chrome](http://www.ubuntuupdates.org/ppa/google_chrome)
* [All "google-chrome-stable" versions](http://www.ubuntuupdates.org/pm/google-chrome-stable)

或者可以通过以下命令直接下载 `*.deb` 安装包：

* x64 -- `$ curl http://dl.google.com/linux/chrome/deb/pool/main/g/google-chrome-stable/google-chrome-stable_54.0.2840.59-1_amd64.deb -O`
* x32 -- 32位版本已不可用

安装deb安装包：

```
$ dpkg -i google-chrome-stable_54.0.2840.59-1_amd64.deb
```

安装过程中可能会安装失败，缺少依赖：

```
dpkg: error processing package google-chrome-stable (--install):
 dependency problems - leaving unconfigured
Processing triggers for mime-support (3.59ubuntu1) ...
Errors were encountered while processing:
 google-chrome-stable
```

通过如下命令解决：

```
$ apt-get -f install
```

* [software installation - How to install Google Chrome? - Ask Ubuntu](http://askubuntu.com/questions/510056/how-to-install-google-chrome)

***

##### 安装 Chromium

Google Chrome isn't in the repositories - however, Chromium is.

因为在软件库中存在 Chromium 的包，所以可以直接通过 `apt-get` 来安装：

```
$ sudo apt-get update
$ sudo apt-get install chromium-browser
```

##### 未安装Chrome浏览器异常

如果未安装 Google-Chrome 或  Chromium 浏览器，会提示如下错误信息：

```
selenium.common.exceptions.WebDriverException: Message: unknown error: cannot find Chrome binary
```

##### 安装 ChromeDriver

ChromeDriver是一个实现了WebDriver与Chromium联接协议的独立服务。

###### 下载 ChromeDriver

访问站点 [chromedriver](http://chromedriver.storage.googleapis.com/index.html) 下载当前系统对应的 `chromedriver` 程序, 页面中给出了不同Chrome版本对应的 ChromeDriver 程序。

执行命令如下，下载并解压：

```
$ curl http://chromedriver.storage.googleapis.com/2.24/chromedriver_linux64.zip -O

$ unzip chromedriver_linux64.zip

$ ls
chromedriver
```

解压后我们得到了一个 `chromedriver` 执行程序。

###### 将 ChromeDriver 放到系统 PATH 目录下

我们可以在程序中指定具体的 ChromeDriver 所在的目录，不指定的话会默认去系统PATH目录下找。为了编程方便，我们将其放到系统PATH目录下。

查看系统目录：

```
$ echo $PATH
```

这里我将其放到 `/usr/local/bin/` 目录下，并添加可执行权限：

```
$ sudo mv ./chromedriver /usr/local/bin/

$ sudo chmod a+x /usr/local/bin/chromedriver
```

######  测试 ChromeDriver

执行 `chromedriver` 查看是否能正常运行。

```
$ chromedriver 
Starting ChromeDriver 2.24.417424 (c5c5ea873213ee72e3d0929b47482681555340c3) on port 9515
Only local connections are allowed.
```

当提示 `Starting ChromeDriver xxx on port 9515` 信息时，说明 `ChromeDriver` 设置成功。  

如果提示如下错误信息，则是在系统PATH下找不到 `ChromeDriver` :

> selenium.common.exceptions.WebDriverException: Message: 'chromedriver' executable needs to be in PATH. Please see https://sites.google.com/a/chromium.org/chromedriver/home

***

#### Selenium调用Firefox浏览器

##### 安装 Firefox 浏览器

可以通过 `apt-get` 直接安装 Firefox 浏览器：

```
$ sudo apt-get install firefox
```

##### 未安装Firefox浏览器异常

如果未安装 Firefox 浏览器程序，则会提示如下错误：

> selenium.common.exceptions.WebDriverException: Message: Expected browser binary location, but unable to find binary in default location, no 'moz:firefoxOptions.binary' capability provided, and no binary flag set on the command line


##### 安装 geckodriver

###### 下载 geckodriver

访问站点 [geckodriver](https://github.com/mozilla/geckodriver/releases) 下载当前系统对应 geckodriver 程序。

执行如下命令：

```
$ curl https://github.com/mozilla/geckodriver/releases/download/v0.11.1/geckodriver-v0.11.1-linux64.tar.gz -O

$ tar -zxvf geckodriver-v0.11.1-linux64.tar.gz

$ ls
geckodriver
```

解压后我们得到了一个 `geckodriver` 执行程序。

该 `geckodriver` 压缩包可能由于网络原因下载失败，可通过迅雷等软件下载后拷贝到Linux系统中。

###### 将 geckodriver 放到系统 PATH 目录下

我们可以在程序中指定具体的 geckodriver 所在的目录，不指定的话会默认去系统PATH目录下找。为了编程方便，我们将其放到系统PATH目录下。

查看系统目录：

```
$ echo $PATH
```

这里我将其放到 `/usr/local/bin/` 目录下，并添加可执行权限：

```
$ sudo mv ./geckodriver /usr/local/bin/

$ sudo chmod a+x /usr/local/bin/geckodriver
```

######  测试 geckodriver

执行 `geckodriver` 查看是否能正常运行。

```
$ geckodriver 
1476443497207	geckodriver	INFO	Listening on 127.0.0.1:4444
```

当提示 `Listening on 127.0.0.1:4444` 信息时，说明 `geckodriver` 设置成功。  

如果提示如下错误信息，则是在系统PATH下找不到 `geckodriver` :

> selenium.common.exceptions.WebDriverException: Message: 'geckodriver' executable needs to be in PATH.

***

#### 安装虚拟显示器 xvfb

##### 为什么要用 xvfb??

xvfb 这个工具相当于一个wrapper，给应用程序提供虚拟的 X server

##### 执行如下命令安装

```
$ sudo apt-get install xvfb
$ sudo pip install pyvirtualdisplay
```

***

#### 测试selenium调用浏览器获取网页

##### Chrome 版本

```
#coding:utf-8

import time
from selenium import webdriver
from pyvirtualdisplay import Display

display=Display(visible=0,size=(800,800))
display.start()

driver=webdriver.Chrome()
driver.get('http://www.cnblogs.com/')
time.sleep(5)
title=driver.title
print(title.encode('utf-8'))
driver.close()

display.stop()
```

将以上代码保存为 `chrome.py` ，执行：

```
$ python chrome.py
博客园 - 开发者的网上家园
```

##### Firefox 版本

```
#coding:utf-8

import time
from selenium import webdriver
from pyvirtualdisplay import Display

display=Display(visible=0,size=(800,800))
display.start()

driver=webdriver.Firefox()
driver.get('http://www.cnblogs.com/')
time.sleep(5)
title=driver.title
print(title.encode('utf-8'))
driver.close()

display.stop()
```

将以上代码保存为 `firefox.py` ，执行：

```
$ python firefox.py
博客园 - 开发者的网上家园
```

***

#### 代码详解

```
#coding:utf-8

import time
from selenium import webdriver
from pyvirtualdisplay import Display

# 设置虚拟显示器的窗口大小
display=Display(visible=0,size=(800,800))
display.start()

driver=webdriver.Chrome()
driver.get('http://www.cnblogs.com/')
time.sleep(5)

# 打印网页的标题
title=driver.title
print(title.encode('utf-8'))
# 退出浏览器
driver.close()

# 关闭虚拟显示器窗口
display.stop()
```



#### 相关参考

* [ChromeDriver](http://chromedriver.storage.googleapis.com/index.html)
* [WebDriver - Mozilla | MDN](https://developer.mozilla.org/en-US/docs/Mozilla/QA/Marionette/WebDriver)
* [GitHub - mozilla/geckodriver: WebDriver &lt;-&gt; Marionette proxy](https://github.com/mozilla/geckodriver) **☆**
* [python - How do I run Selenium in Xvfb? - Stack Overflow](http://stackoverflow.com/questions/6183276/how-do-i-run-selenium-in-xvfb) **☆**
* [Selenium2.0](http://blog.163.com/he_junwei/blog/static/1979376462013126105119274/)
