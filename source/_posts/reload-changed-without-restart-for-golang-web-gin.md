---
title: reload-changed-without-restart-for-golang-web-gin
date: 2017-08-01 15:07:37
tags:
	- Golang
	- Gin
description: 在调试Gin项目时，每次更改了文件内容后都需要重新运行 [go run main.go] 命令才能看到更改。项目 [codegangsta/gin] 和 [pilu/fresh] 是通过采用热更新的方式来调试Gin项目推荐度较高的两个，且看哪个在操作上更加的方便。
---

Gin is a HTTP web framework written in Go (Golang)。但是在调试Gin项目时，每次更改了文件内容后都需要重新运行 `go run main.go` 命令才能看到更改。项目 `codegangsta/gin` 和 `pilu/fresh` 是通过采用热更新的方式来调试Gin项目推荐度较高的两个，且看哪个在操作上更加的方便。

#### gin 

##### github 地址

[GitHub - codegangsta/gin: Live reload utility for Go web servers](https://github.com/codegangsta/gin)

##### 下载并安装

```
go get github.com/codegangsta/gin
```

将 `GOPATH/bin` 目录添加到系统的 `PATH` 中。


##### 项目测试

```
package main
import "github.com/gin-gonic/gin"
func main() {
        r := gin.Default()
        r.GET("/", func(c *gin.Context) {
                c.String(200, "hello world\n")
        })
        r.Run(":3001")
}

# 运行
> gin run main.go

# 浏览器访问
http://localhost:3000
```

##### 扩展

> 因为项目 `codegangsta/gin` 和 `gin-gonic/gin` 重名，所以这里我用 `Reload gin` 代指 `codegangsta/gin`;用 `Web gin` 代指 `gin-gonic/gin`。

默认情况下，`Reload gin`的默认监听端口为 `3000`,内部导向的go web项目端口为 `3001`。如果采用 `Reload gin` 的默认端口，则需要将 `Web gin` 的监听端口改为 `3001`,即：`r.Run(":3001")` 。

如果需要自定义端口，通过 `gin -h` 可以看到 `Reload gin` 的常用配置项。其中:

```
 --port value, -p value        port for the proxy server (default: 3000)
 --appPort value, -a value     port for the Go web server (default: 3001)
```

可以分别指定监听端口和映射端口。

不过，经过测试，自定义端口时报如下错误：

```
λ gin run -p 8082 -a 8080 main.go
Incorrect Usage: flag provided but not defined: -p

NAME:
   gin run - Run the gin proxy in the current working directory

USAGE:
   gin run [arguments...]
```

好像目前只能使用默认的 `3000` 和 `3001` 端口。

***

#### fresh

##### github地址

[GitHub - pilu/fresh: Build and (re)start go web apps after saving/creating/deleting source files.](https://github.com/pilu/fresh)

##### 下载及安装

```
go get github.com/pilu/fresh
```

将 `GOPATH/bin` 目录添加到系统的 `PATH` 中。

##### 项目测试

```
package main
import "github.com/gin-gonic/gin"
func main() {
        r := gin.Default()
        r.GET("/", func(c *gin.Context) {
                c.String(200, "hello world\n")
        })
        r.Run(":8080")
}

# 进入项目所在目录
> cd /path/to/myapp

# 运行
> fresh

# 浏览器访问
http://localhost:8080
```

##### 扩展

原项目不需要做任何改动，只需要在原项目的目录下执行命令 `fresh` 即可。

***

经以上测试，推荐使用 `fresh` 来运行 `Gin` 项目。

***

#### 相关链接

* [GitHub - codegangsta/gin: Live reload utility for Go web servers](https://github.com/codegangsta/gin)
* [GitHub - pilu/fresh: Build and (re)start go web apps after saving/creating/deleting source files.](https://github.com/pilu/fresh)
* [GitHub - gin-gonic/gin: Gin is a HTTP web framework written in Go (Golang). It features a Martini-like API with much better performance -- up to 40 times faster. If you need smashing performance, get yourself some Gin.](https://github.com/gin-gonic/gin)
