---
title: Golang运行环境配置
date: 2017-04-02 10:09:32
tags:
    - Golang
categories: 
    - Golang
description: Golang运行环境配置
---

#### Win系统下配置Go语言环境

1. 从网站 [Downloads - The Go Programming Language](https://golang.org/dl/) 下载 `go1.6.3.windows-amd64.msi` 安装包。
2. 运行并安装到默认目录 `C:\Go` 目录下。
3. 配置 `GOROOT` 和 `GOPATH` :
    * 在 系统的环境变量中的 `PATH` 变量中添加Go的目录 `C:\Go\bin;` (通过安装包安装后已默认设置该项)
    * 在 系统的环境变量中添加 `GOROOT` 变量，设置值为 `C:\Go\` (通过安装包安装后已默认设置该项)
    * 要验证 `GOROOT` 是否设置成功，我们可以在命令行窗口中输入 `go version`，如果输出了Go的版本信息则说明配置正确。
    * `GOPATH` 就需要我们手动配置一下，`GOPATH` 就是你的工作目录，用于存放项目和go依赖包等。假设目录 `E:\GOProject` 为我的Go项目工作目录，该工作目录下默认包含三个子目录：`bin/` `pkg/` `src` 。然后我们在 系统的环境变量中 新建 `GOPATH` 变量，设置值为 `E:\GOProject;`。默认情况下 `GOPATH` 目录可以设置多个，之间用分号 `;` 分隔。
    * 然后在命令行窗口中输入 `echo %GOPATH%` 如果打印出了上面设置的 `GOPATH` 的目录，则说明配置成功。
4. 为了验证Go开发环境是否设置成功，我们可以在命令行下输入如下命令：(保证已经安装Git)

```
go get github.com/golang/example/hello
```
5. 然后执行命令：

```
%GOPATH%/bin/hello
```

如果输出了 `Hello,Go examples!` 则说明Go语言的开发环境搭建成功。

参考自：[Easy Go Programming Setup for Windows Wade Wegner](http://www.wadewegner.com/2014/12/easy-go-programming-setup-for-windows/)

***


#### Mac下配置Golang开发环境


##### 通过brew安装

```
$ brew install go
```

##### 通过pkg包安装

从 [golang官网](https://golang.org/) 下载Mac下的pkg安装包直接安装.


##### 环境变量配置

GOPATH允许多个目录，当有多个目录时，请注意分隔符，多个目录的时候Windows是分号，Linux系统是冒号，当有多个GOPATH时，默认会将 `go get` 的内容放在第一个目录下。

```
$ go env
GOARCH="amd64"
GOBIN=""
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOOS="darwin"
GOPATH=""
GORACE=""
GOROOT="/usr/local/go"
GOTOOLDIR="/usr/local/go/pkg/tool/darwin_amd64"
CC="clang"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/mk/7c0w24ts0ps1g06q78gzd7dc0000gn/T/go-build098119519=/tmp/go-build -gno-record-gcc-switches -fno-common"
CXX="clang++"
CGO_ENABLED="1"
```


假如我的go项目开发主目录为:
`/Users/xxx/Learn/Go`
在该目录下,第一个目录为 `xgo`  第二个目录为 `xgo_workspace`


```
# GOPATH
export GOPATH=$HOME/Learn/Go/xgo:$HOME/Learn/Go/xgo_workspace

# GOPATH bin
export PATH=$PATH:$GOPATH/bin
```

添加完成后,重启终端即可生效.如果想立即生效,则可执行如下命令:

```
$ source ~/.bash_profile
```

再次查看go环境变量:

```
$ go env
GOARCH="amd64"
GOBIN=""
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOOS="darwin"
GOPATH="/Users/xxx/Learn/Go/xgo:/Users/xxx/Learn/Go/xgo_workspace"
GORACE=""
GOROOT="/usr/local/go"
GOTOOLDIR="/usr/local/go/pkg/tool/darwin_amd64"
CC="clang"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/mk/7c0w24ts0ps1g06q78gzd7dc0000gn/T/go-build346222074=/tmp/go-build -gno-record-gcc-switches -fno-common"
CXX="clang++"
CGO_ENABLED="1"
```

***

* http://blog.helloarron.com/2015/08/29/go/mac-install-go/
* https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/01.2.md
* http://blog.studygolang.com/2013/01/%E5%86%8D%E7%9C%8Bgopath/  (设置多个目录时只会将最后一个目录添加上bin 这里要说明)

***

