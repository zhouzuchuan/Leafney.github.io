---
title: 从0到1学Golang之基础--Go 包
date: 2017-05-12 20:07:48
tags:
    - Golang
categories: 
    - 从0到1学Golang
description: Golang下的包管理操作
---

《Go In Action》 中文版 《Go语言实战》 读书笔记

#### 什么是包

go语言的包其实就是我们计算机里的目录

#### 包的命名

所有的 `.go` 文件，除了空行和注释，都应该在第一行声明自己所属的包。

使用 `package` 关键字声明包。如： `package main` 。

每个包都在一个单独的目录里。也就是说，同一个目录下的所有 `.go` 文件必须声明同一个包名。

以 `net/http` 包为例，在 `http` 目录下的所有文件都属于 `http` 包。

go语言中给包及其目录命名，遵循 `简洁` 、`清晰` 、`全小写` 及 `和所在目录同名` 的原则。

```
package main

import "net/http"

func main() {
    http.ListenAndServe("127.0.0.1:80",handler);
}
```

如上 `net/http`，在导入包时采用 `全路径` 的方式，所以可以区分同名的不同包，只要保证全路径不同就可以。
使用全路径的导入，也增加了包名命名的灵活性。

#### main 包

在go语言中，命名为 `main` 的包会被尝试编译为一个二进制可执行文件。

所有用go语言编译的可执行程序都必须有一个名为 `main` 的包。

一个 `main` 的包，一定会包含一个 `main()` 函数。

在Go语言里同时要满足 `main` 包和包含 `main()` 函数，才会被编译成一个可执行文件。

Go程序编译时，会使用声明 `main` 包的代码所在的目录的目录名作为二进制可执行文件的文件名。

#### 包的导入

使用 `import` 关键字来导入包。导入的包必须是一个全路径的包，也就是包所在的位置。

导入单个包：

```
import "fmt"
```

导入多个包时，使用一对括号包含的导入块，每个包独占一行。

```
import (
    "fmt"
    "net/http"
    "strings"
)
```

对于多于一个路径的包名，在代码中引用的时候，使用全路径最后一个包名作为引用的包名，比如 `net/http` ,我们在代码使用的是 `http` ，而不是 `net` 。

Go有两个很重要的环境变量 `GOROOT` 和 `GOPATH` ,这是两个定义路径的环境变量，GOROOT是安装Go的路径，比如/usr/local/go；GOPATH是我们自己定义的开发者个人的工作空间，比如 `/home/myproject/go` 。

标准库中的包会在 `GOROOT` 去查找，Go开发者创建的包会在 `GOPATH` 指定的目录中查找。

对于包的查找，是有优先级的，编译器会优先在 `GOROOT` 里搜索，其次是 `GOPATH` ,一旦找到，就会马上停止搜索。如果最终都没找到，就报编译异常了。

#### 远程导入

Go语言的工具链支持从Github、Bitbucket等类似网站获取源代码。

```
import "github.com/spf13/viper"
```

用导入路径编译程序时，gobuild 命令会先在 `GOPATH` 下搜索这个包，如果没有找到，就会使用 `go get` 工具通过URL从网络获取，并把包的源代码保存在 `GOPATH` 目录下对应URL的目录里。

`go get` 工具可以递归获取依赖包。

#### 命名导入

如果要导入的多个包具有相同的名字，可以通过 `命名导入` 的方式来导入。

**命名导入**是指在 `import` 语句给出的包路径的左侧定义一个名字，将导入的包命名为新名字。

```
import (
    "fmt"
    myfmt "mylib/fmt"
)
```

Go语言规定，导入的包必须要使用，否则会报编译错误。Go语言的这个特性可以防止导入了未被使用的包，避免代码变得臃肿。

但是有时候我们需要导入一个包，但是不需要引用这个包的标识符。这种情况下，可以使用空白标识符 `_` 来重命名导入的包即可。

```
package main

import (
    "fmt"
    _ "mylib/fmt"
)
```

> 空白标识符(_) 用来抛弃不想继续使用的值，如给导入的包赋予一个空名字，或忽略函数返回的不感兴趣的值。

#### init函数

每个包都可以有任意多个init函数，这些init函数都会在main函数之前执行。init函数通常用来做初始化变量、设置包或者其他需要在程序执行前的引导工作。init函数用在设置包、初始化变量或者其他要在程序运行前优先完成的引导工作。

以数据库驱动为例， database 下的驱动在启动时执行 init 函数会将自身注册到 sql 包里，因为 sql 包在编译时并不知道这些驱动的存在，等启动之后 sql 才能调用这些驱动。

```
package postgres

import (
    "database/sql"
)

func init() {
    sql.Register("postgres", new(PostgresDriver))
}
```

因为我们只是想执行这个 postgres 包的 `init` 方法，并不想使用这个包，所以我们在导入的时候，需要使用空白标识符 `_` 来重命名包名，避免编译错误。

```
package main

import (
    "database/sql"
    _ "github.com/goinaction/code/chapter3/dbdriver/postgres"
)

func main() {
    sql.Open("postgres", "mydb")
}
```

如上非常简洁，剩下很对数据库的操作，都是使用 `database/sql` 标准接口。如果我们想换其他数据库驱动，只需要换个导入即可，灵活方便。

