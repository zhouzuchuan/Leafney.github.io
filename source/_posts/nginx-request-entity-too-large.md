---
title: HTTP 413 curl 22 The requested URL returned error 413 Request Entity Too Large
date: 2016-12-30 10:50:55
tags: 
	- Nginx
description: HTTP 413 curl 22 The requested URL returned error 413 Request Entity Too Large
---

#### 问题原因

在使用gogs时，`git push` 代码报如下错误：

```
error: RPC failed; HTTP 413 curl 22 The requested URL returned error: 413 Request Entity Too Large
```

经查证，是服务器上的nginx默认情况下只允许上传最大 `1m` 大小的文件。nginx默认配置如下：

```
Syntax:		client_max_body_size size;
Default:	client_max_body_size 1m;
Context:	http, server, location
```

***

#### 解决方法

在 `nginx` 的配置文件 `nginx.conf` 中的 `http` 段内，添加 `client_max_body_size` 配置：

```
http {
	...
	client_max_body_size 50m;
	...
	}
```

后面的 `50m` 表示最大允许上传 `50M` 大小的文件。

然后重新加载 `nginx` 配置信息：

```
$ sudo service nginx reload
```

***

#### 相关参考

* [Module ngx_http_core_module](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size)
* [nginx client_max_body_size 的问题 | yanghao&#039;s blog](http://yanghao.org/blog/archives/365)
* [git - Github Push Error: RPC failed; result=22, HTTP code = 413 - Stack Overflow](http://stackoverflow.com/questions/7489813/github-push-error-rpc-failed-result-22-http-code-413)

