---
title: Golang实现http请求及代理设置
date: 2017-06-15 16:03:26
tags:
    - Golang
    - 爬虫
categories: 
    - Golang
description: 主要探究 1. 使用代理请求 2. 跳过https不安全验证 3. 自定义请求头User-Agent的实现
---
主要探究 1. 使用代理请求 2. 跳过https不安全验证 3. 自定义请求头User-Agent的实现

#### 主要研究的技术点

* 使用代理请求
* 跳过https不安全验证
* 自定义请求头 User-Agent

***

#### 静态数据请求并设置代理

##### 实例代码

```
package main

import (
	"crypto/tls"
	"fmt"
	"io/ioutil"
	"net/http"
	"net/url"
	"time"
)

func First() {
	/*
		1. 普通请求
	*/

	webUrl := "http://ip.gs/"
	resp, err := http.Get(webUrl)
	if err != nil {
		fmt.Println(err)
		return
	}
	// if resp.StatusCode == http.StatusOK {
	// 	fmt.Println(resp.StatusCode)
	// }

	time.Sleep(time.Second * 3)

	defer resp.Body.Close()
	body, _ := ioutil.ReadAll(resp.Body)
	fmt.Println(string(body))
}

func Second(webUrl, proxyUrl string) {
	/*
		1. 代理请求
		2. 跳过https不安全验证
	*/
	// webUrl := "http://ip.gs/"
	// proxyUrl := "http://115.215.71.12:808"

	proxy, _ := url.Parse(proxyUrl)
	tr := &http.Transport{
		Proxy:           http.ProxyURL(proxy),
		TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
	}

	client := &http.Client{
		Transport: tr,
		Timeout:   time.Second * 5, //超时时间
	}

	resp, err := client.Get(webUrl)
	if err != nil {
		fmt.Println("出错了", err)
		return
	}

	defer resp.Body.Close()
	body, _ := ioutil.ReadAll(resp.Body)
	fmt.Println(string(body))

}

func Third(webUrl, proxyUrl string) {
	/*
		1. 代理请求
		2. 跳过https不安全验证
		3. 自定义请求头 User-Agent

	*/
	// webUrl := "http://ip.gs/"
	// proxyUrl := "http://171.215.227.125:9000"

	request, _ := http.NewRequest("GET", webUrl, nil)
	request.Header.Set("Connection", "keep-alive")
	request.Header.Set("User-Agent", "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36")

	proxy, _ := url.Parse(proxyUrl)
	tr := &http.Transport{
		Proxy:           http.ProxyURL(proxy),
		TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
	}

	client := &http.Client{
		Transport: tr,
		Timeout:   time.Second * 5, //超时时间
	}

	resp, err := client.Do(request)
	if err != nil {
		fmt.Println("出错了", err)
		return
	}

	defer resp.Body.Close()
	body, _ := ioutil.ReadAll(resp.Body)
	fmt.Println(string(body))

}

func main() {
	webUrl := "http://httpbin.org/user-agent" //"http://ip.gs/"
	proxyUrl := "http://119.5.0.75:808"

	Second(webUrl, proxyUrl)
	// Third(webUrl, proxyUrl)
}

```

##### 相关参考

* [Mocking a HTTP access with http.Transport in Golang - oinume journal](http://oinume.hatenablog.com/entry/mocking-http-access-in-golang)
* [Go http访问使用代理](http://www.cnblogs.com/damir/archive/2012/05/06/2486663.html)
* [GO HTTP client客户端使用 - 海运的博客](https://www.haiyun.me/archives/1051.html)
* [Making Tor HTTP Requests with Go | DevDungeon](http://www.devdungeon.com/content/making-tor-http-requests-go)
* [go - golang: How to do a https request with proxy - Stack Overflow](https://stackoverflow.com/questions/42662369/golang-how-to-do-a-https-request-with-proxy)
* [go - Set UserAgent in http request - Stack Overflow](https://stackoverflow.com/questions/13263492/set-useragent-in-http-request)

***

#### 动态数据请求并设置代理

动态数据请求使用golang调用phantomjs来请求网页数据内容实现。

通过命令参数方式：

```
phantomjs [options] somescript.js [arg1 [arg2 [...]]]
```

代理相关的配置参数：

* `--load-images=[true|false]`  (default is `true`)
* `--proxy=address:port`  (eg `--proxy=192.168.1.42:8080`)
* `--proxy-type=[http|socks5|none]`  (default is `http`)
* `--proxy-auth` (eg `--proxy-auth=username:password`)

其他常用配置参数：

* `--load-images=[yes|no]`             Load all inlined images (default is 'yes').
* `--load-plugins=[yes|no]`            Load all plugins (i.e. 'Flash', 'Silverlight', ...) (default is 'no').
* `--proxy=address:port`               Set the network proxy.
* `--disk-cache=[yes|no]`              Enable disk cache (at desktop services cache storage location, default is 'no').
* `--ignore-ssl-errors=[yes|no]`       Ignore SSL errors (i.e. expired or self-signed certificate errors).

也可以通过配置文件方式来设置：

```
phantomjs --config=/path/to/config.json somescript.js [arg1 [...]]
```

##### 常用配置相关参考

* [Command Line Interface | PhantomJS](http://phantomjs.org/api/command-line.html)
* [Proxy Auth In Phantomjs &middot; David Blooman](http://www.dblooman.com/network/2014/05/27/Proxy-Auth-in-phantomjs/)

****

##### 实例代码

* 经过测试，发现： `phantomjs --proxy=address:port somescript.js [args]` 这种方式无法执行成功。
* 经过测试，发现：在 `somescript.js` 中设置 `page.setProxy("http://119.5.0.75:808/");` 这种方式也无效。
* 经过测试，发现：在 `somescript.js` 中设置 `phantom.setProxy("139.224.237.33", "8888", 'manual', '', '');` 这种方式可行。

###### 不使用代理的动态数据请求

`dynamicproxy.go`:

```
package main

import (
	"fmt"
	"io/ioutil"
	"os/exec"
)

func First(webUrl, jsFileName string) {
	cmd := exec.Command("phantomjs.exe", jsFileName, webUrl)
	out, err := cmd.Output()
	if err != nil {
		fmt.Println(err)
	}

	fmt.Println(string(out))
}

func Second(webUrl, jsFileName string) {
	// cmd := exec.Command("phantomjs.exe", "test.js", "https://www.cnblogs.com/")
	cmd := exec.Command("phantomjs.exe", jsFileName, webUrl)
	stdout, err := cmd.StdoutPipe()
	if err != nil {
		fmt.Println(err)
	}
	cmd.Start()
	content, err := ioutil.ReadAll(stdout)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(string(content))
}

func main() {
	webUrl := "http://httpbin.org/ip" //"http://httpbin.org/ip" // "http://ip.gs/" // "http://httpbin.org/user-agent"
	jsfileName := "somescript.js"
	First(webUrl, jsfileName)
	// Second(webUrl, jsfileName)
}

```

`somescript.js`:

```
var page =require('webpage').create();
system=require('system');
url=system.args[1];

page.settings.userAgent = 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/37.0.2062.120 Safari/537.36';
page.open(url,function(status){
	console.log('Loading '+system.args[1]);
	if (status=="success") {
		// page.render('a.png'); //网页截屏
		system.stdout.writeLine(page.content); // 获取网页内容
		// system.stdout.writeLine(page.title);
		// console.log(page.plainText); // 文本内容
		// console.log(page.title); // 网页标题
	}else{
		system.stdout.writeLine("request error");
	};
	phantom.exit();
});
```

****

###### 使用http代理的动态数据请求

`dynamicproxy.go`:

```
package main

import (
	"fmt"
	"io/ioutil"
	"os/exec"
)

func Third(webUrl, jsFileName,proxyHost,proxyPort string) {
	// cmd := exec.Command("phantomjs.exe", jsFileName, webUrl)
	cmd := exec.Command("phantomjs.exe", jsFileName,proxyHost,proxyPort, webUrl)
	out, err := cmd.Output()
	if err != nil {
		fmt.Println(err)
	}

	fmt.Println(string(out))
}
func main() {
	webUrl := "http://httpbin.org/ip" //"http://httpbin.org/ip" // "http://ip.gs/" // "http://httpbin.org/user-agent"
	jsfileName := "somescript.js"
	Third(webUrl, jsfileName,"139.224.237.33", "8888")
}
```

`somescript.js`:

```
var page =require('webpage').create();
system=require('system');

if (system.args.length<4) {
	system.stdout.writeLine("somescript.js <proxyHost> <proxyPort> <URL>");
	phantom.exit(1);
}else{
	host=system.args[1];
	port=system.args[2];
	url = system.args[3];

// page.setProxy("http://119.5.0.75:808/");//这样设置请求失败

// phantom.setProxy("139.224.237.33", "8888", 'manual', '', '');//这样设置可以
phantom.setProxy(host, port, 'manual', '', '');
page.settings.userAgent = 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/37.0.2062.120 Safari/537.36';

page.open(url,function(status){
	if (status=="success") {
		system.stdout.writeLine(page.content); // 获取网页内容
	}else{
		console.log(status);
		system.stdout.writeLine("error");
	};
	phantom.exit();
});
}
```

说明：`phantom.setProxy("139.224.237.33", "8888", 'manual', '', '');`  参数1为代理host;参数2为代理port;参数3可以保持默认定值`manual`,参数4为代理类型http socket5等可为空;参数5不知道为空。

***

###### 忽略https安全验证的动态数据请求

* 暂时只找到一种方法，直接请求https网址不使用代理能够请求成功。
* 即忽略https安全验证，又使用代理经测试请求失败。
* 直接在命令窗口中执行可以，通过golang调用请求失败。

使用参数：`--ignore-ssl-errors=true --ssl-protocol=any` 设置。

如下方式执行成功，示例代码：

```
phantomjs.exe --ignore-ssl-errors=true --ssl-protocol=any test.js https://kyfw.12306.cn/otn/
```

`test.js`:

```
var page =require('webpage').create();
system=require('system');
url=system.args[1];

page.settings.userAgent = 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/37.0.2062.120 Safari/537.36';
page.open(url,function(status){
	console.log('Loading '+system.args[1]);
	if (status=="success") {
		system.stdout.writeLine(page.content); // 获取网页内容
	}else{
		console.log("request error");
	};
	phantom.exit();
});
```

***

##### 相关参考

* [--ignore-ssl-errors not working · Issue #12181 · ariya/phantomjs · GitHub](https://github.com/ariya/phantomjs/issues/12181)
* [phantomjs · GitHub](https://github.com/ariya/phantomjs/blob/master/examples/openurlwithproxy.js)
* [phantomjs/examples at master · ariya/phantomjs · GitHub](https://github.com/ariya/phantomjs/tree/master/examples)
