---
title: Golang网站微框架Gin
date: 2017-07-30 14:05:28
tags:
	- Golang
	- Gin
description: Gin is a web framework written in Golang.
---

Gin is a web framework written in Golang.

#### 安装 Gin

获取包：

```
> go get github.com/gin-gonic/gin
```

添加引用：

```
import "github.com/gin-gonic/gin"
```

***

#### Gin web 入门

##### 一个简单的示例

```
package main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()

	router.GET("/", func(c *gin.Context) {
		c.String(200, "hello world!")
	})

	//http://localhost:8081  postdata: name=tom
	router.POST("/", postHome)

	router.Run(":8081")
}

func postHome(c *gin.Context) {
	uName := c.PostForm("name")
	c.JSON(200, gin.H{
		"say": "Hello " + uName,
	})
}
```

##### 请求方法名必须全部是大写字母

```
	router.GET("/someGet", getting)
	router.POST("/somePost", posting)
	router.PUT("/somePut", putting)
	router.DELETE("/someDelete", deleting)
	router.PATCH("/somePatch", patching)
	router.HEAD("/someHead", head)
	router.OPTIONS("/someOptions", options)
```

##### 获取路由参数

通过 `Context` 的 `Param` 方法来获取：

```
...
// http://localhost:8081/user/article/tommy
router.GET("/user/:type/:name", getRouteStr)
...

func getRouteStr(c *gin.Context) {
	ctype := c.Param("type")
	cname := c.Param("name")
	c.JSON(200, gin.H{
		"typeName": ctype,
		"username": cname,
	})
}

//result:
{
    "typeName": "article",
    "username": "tommy"
}
```

##### 获取url参数

通过 `DefaultQuery` 或 `Query` 方法获取：

* `Query('xxx')` 如果没有相应值，默认为空字符串
* `DefaultQuery("xxx","defaultValue")` 可设置默认值,string类型

```
...
	router.GET("/user", getQueryStrs)
...

func getQueryStrs(c *gin.Context) {
	name := c.Query("name")           //如果没有相应值，默认为空字符串
	age := c.DefaultQuery("age", "0") //可设置默认值,string类型
	c.JSON(200, gin.H{
		"name": name,
		"age":  age,
	})
}

//result:
//http://localhost:8081/user?name=tom&age=23
{
    "age": "23",
    "name": "tom"
}

//result2:
//http://localhost:8081/user?name=tom
{
    "age": "0",
    "name": "tom"
}
```

***

`c.Request.URL.Query()` 可获取所有url请求参数 `map[]` 集合：

```
		//获取所有url请求参数
		reqData := c.Request.URL.Query()
		fmt.Printf("[info] req url data is %s\n", reqData)

//result:
//http://localhost:8080?id=3&name=zhangsan&address=beijing
[info] req url data is map[name:[zhangsan] address:[beijing] id:[3]]
```

##### 获取表单参数

表单参数通过 `PostForm` 或 `DefaultPostForm` 方法获取：

* `PostForm()`
* `DefaultPostForm("xxx","defaultValue")`

```
	...
	router.POST("/", getFormStr)
	...

func getFormStr(c *gin.Context) {
	title := c.PostForm("title")
	cont := c.DefaultPostForm("cont", "没有内容")
	c.JSON(200, gin.H{
		"title": title,
		"cont":  cont,
	})
}

//result:
//http://localhost:8081  postData: title:这是一个标题
{
    "cont": "没有内容",
    "title": "这是一个标题"
}
```

***

`c.Request.Body` 获取所有 `post body` 数据：

```
		//获取post body
		x, _ := ioutil.ReadAll(c.Request.Body)
		fmt.Printf("[info] %s", string(x))

//result:
// post body type :x-www-form-urlencoded (user=tom pwd=123)
[info] user=tom&pwd=123

//result:
// post body type: raw application/json
[info] {"name":"zhangfei","id":32}
```

* [go - gin/golang - Empty Req Body - Stack Overflow](https://stackoverflow.com/questions/31911579/gin-golang-empty-req-body)
* [Print Request Body empty · Issue #401 · gin-gonic/gin · GitHub](https://github.com/gin-gonic/gin/issues/401)

***

获取所有 `post data` 数据 （`map[]` 类型）

```
		c.Request.ParseForm()
		reqBodyData := c.Request.PostForm
		fmt.Printf("[info] req body data is %s \n", reqBodyData)

//result:
// post body type :x-www-form-urlencoded (user=tom pwd=123)
[info] req body data is map[user:[tom] pwd:[123]]
```

* [go - Gin Gonic array of values from PostForm - Stack Overflow](https://stackoverflow.com/questions/39984575/gin-gonic-array-of-values-from-postform)

##### 路由群组

支持 `一级` 或 `多级` 分组的路由规则：

```
	...
	//路由分组
	articleGroup := router.Group("/article")
	{
		articleGroup.GET("/one/:id", getArticleByid)
		articleGroup.GET("/list", getArticleList)
	}

	//多级路由分组
	apiGroup := router.Group("/api")
	apiv1Group := apiGroup.Group("/v1")
	{
		apiv1Group.GET("/user", getApiV1User)
	}

	apiv2Group := apiGroup.Group("/v2")
	{
		apiv2Group.GET("/order", getApiV2Order)
	}
	...



func getArticleByid(c *gin.Context) {
	a_id := c.Param("id")
	c.String(200, a_id)
}
//result:
//http://localhost:8081/article/one/3
3

func getArticleList(c *gin.Context) {
	c.JSON(200, gin.H{
		"a": "1",
		"b": "2",
	})
}
//result:
//http://localhost:8081/article/list
{
    "a": "1",
    "b": "2"
}

func getApiV1User(c *gin.Context) {
	c.String(200, "api/v1/user")
}
//result:
//http://localhost:8081/api/v1/user
api/v1/user

func getApiV2Order(c *gin.Context) {
	c.String(200, "api/v2/order")
}
//result:
//http://localhost:8081/api/v2/order
api/v2/order

```

##### 数据绑定

* `Bind()` 
* `BindJSON()`

```
	...
	//绑定普通表单 (user=tom&&pwd=123)
	router.POST("/loginform", func(c *gin.Context) {
		var form Login
		if c.Bind(&form) == nil {
			if form.User == "tom" && form.Password == "123" {
				c.JSON(200, gin.H{"status": "form logined in"})
			} else {
				c.JSON(201, gin.H{"status": "form no login"})
			}
		}
	})

	//绑定JSON ({"user":"tom","pwd":"123"}) Content-Type:application/json
	router.POST("/loginjson", func(c *gin.Context) {
		var json Login
		if c.BindJSON(&json) == nil {
			if json.User == "tom" && json.Password == "123" {
				c.JSON(200, gin.H{"status": "json logined in"})
			} else {
				c.JSON(201, gin.H{"status": "json no login"})
			}
		}
	})
	...

type Login struct {
	User     string `form:"user" json:"user"`
	Password string `form:"pwd" json:"pwd"`
}

```

##### POST上传文件

```
	...
	//表单上传文件  http://localhost:8081/upload 
	// key:upload value: file....
	//result:
	//{
    //"filename": "1009e3ee4bc0919e11d32e00ccf55cdf.jpg"
	//}

	router.POST("/upload", func(c *gin.Context) {
		_, header, _ := c.Request.FormFile("upload")
		filename := header.Filename
		c.JSON(200, gin.H{
			"filename": filename,
		})
	})
	...
```

##### 根据客户端的请求类型，返回对应的响应格式

```
	...
	router.GET("/getdata", func(c *gin.Context) {
		contentType := c.Request.Header.Get("Content-Type")

		switch contentType {
		case "application/json":
			c.JSON(200, gin.H{"user": "张飞", "address": "长坂坡"})
		case "application/xml":
			c.XML(200, gin.H{"user": "张飞", "address": "长坂坡"})
		case "application/x-www-form-urlencoded":
			c.String(200, "张飞 长坂坡")
		}
	})
	...
```

##### 字符串响应

```
import "net/http"

c.String(200, "some string")
c.String(http.StatusOK, "some string")
```

##### JSON/XML/YAML等格式响应

```
	...
	c.JSON(http.StatusOK, msg)
	c.XML(http.StatusOK, msg)
	c.YAML(http.StatusOK, msg)
	...
```

##### 视图响应

```
	...
	//加载模板
	router.LoadHTMLGlob("templates/*")
	// router.LoadHTMLFiles("templates/index.html","templates/article.html")
	router.GET("/", func(c *gin.Context) {
		//根据完整文件名渲染模板，并传递参数
		c.HTML(200, "index.html", gin.H{
			"say": "Hello World!",
		})
	})
	...

//index.html:
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<h3>{{ .say }}</h3>
</body>
</html>


//result:
<!DOCTYPE html>
<html>
    <head>
        <title></title>
    </head>
    <body>
        <h3>Hello World!</h3>
    </body>
</html>
```

##### 加载多层模板路径

经测试：如果是多层级的模板文件，要在模板文件中使用  
```
{{define xxx}} {{end}}
```
将该模板作为嵌套模板：

```
	//加载模板
	router.LoadHTMLGlob("templates/**/*")
	// router.LoadHTMLFiles("templates/index.html","templates/article.html")

	router.GET("/articles/index", func(c *gin.Context) {
		//根据完整文件名渲染模板，并传递参数
		c.HTML(200, "articles/index.html", gin.H{
			"say": "Article index!",
		})
	})

	router.GET("/users/index", func(c *gin.Context) {
		c.HTML(200, "/users/index.html", gin.H{
			"say": "User index!",
		})
	})
```

templates/articles/index.html:

```
{{define "articles/index.html"}}
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<h3>{{ .say }}</h3>
</body>
</html>
{{end}}
```

templates/users/index.html:

```
{{define "users/index.html"}}
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<h3>{{ .say }}</h3>
</body>
</html>
{{end}}
```

##### 提供静态文件

* `Static()`
* `StaticFS()`
* `StatucFile()`

示例:

```
func main() {
	router := gin.Default()
	router.Static("/assets", "./assets")
	router.StaticFS("/more_static", http.Dir("my_file_system"))
	router.StaticFile("/favicon.ico", "./resources/favicon.ico")

	// Listen and serve on 0.0.0.0:8080
	router.Run(":8080")
}
```

待详细研究。

##### 重定向

支持 `内部` 和 `外部` 地址的重定向

```
	...
	//跳转到外部地址
	router.GET("/abc", func(c *gin.Context) {
		c.Redirect(302, "http://www.baidu.com")
	})

	//跳转到内部地址
	router.GET("def", func(c *gin.Context) {
		c.Redirect(302, "/home")
	})

	router.GET("/home", func(c *gin.Context) {
		c.String(200, "home page")
	})
	...
```

##### 自定义中间件及中间件的使用方式

```
	...

func main() {
	// router := gin.Default()

	router := gin.New()
	//全局中间件
	router.Use(MyLogger())

	router.GET("/test", func(c *gin.Context) {
		example := c.MustGet("example").(string)
		c.String(200, example)
	})

	//单路由中间件
	router.GET("/abc",MyMiddelware(),getAbc)

	//群组路由中间件
	aGroup:=router.Group("/",MyMiddelware())

	//中间件可以同时添加多个
	aGroup:=router.Group("/",MyMiddelware(),My2Middelware(),My3Middelware())

	//或者：群组路由中间件
	bGroup:=router.Group("/")
	bGroup.Use(MyMiddelware())
	{
		bGroup.GET("/v1",getV1)
	}


	router.Run(":8081")

}

func MyLogger() gin.HandlerFunc {
	return func(c *gin.Context) {
		c.Set("example", "123465")

		c.Next()

	}
}

```

##### 异步处理

`goroutine` 中只能使用只读的上下文 `c.Copy()`

```
	...
	//异步
	router.GET("/async", func(c *gin.Context) {
		// goroutine 中只能使用只读的上下文 c.Copy()
		cCp := c.Copy()
		go func() {
			time.Sleep(5 * time.Second)
			//需要使用只读上下文
			log.Println("Done! in path " + cCp.Request.URL.Path)
		}()

	})

	//同步
	router.GET("/sync", func(c *gin.Context) {
		time.Sleep(5 * time.Second)
		//可以使用原始上下文
		log.Println("Done! in path " + c.Request.URL.Path)
	})
	...
```

##### 初始化不带中间件和带有默认中间件的路由

* `r := gin.New()` 创建不带中间件的路由
* `r := gin.Default()` 创建带有默认中间件的路由:日志与恢复中间件


##### Cookie

* http://www.grdtechs.com/2016/03/29/gin-setcookie/

****

#### 其他示例

##### 获取所有请求参数

* 获取所有URL请求参数

```
		reqData := c.Request.URL.Query()
		fmt.Printf("[info] req url data is %s\n", reqData)
```

* 获取所有Body请求参数

```
		c.Request.ParseForm()
		reqBodyData := c.Request.PostForm
		fmt.Printf("[info] req body data is %s \n", reqBodyData)
```

##### 获取请求头信息

```
		// 获取请求头Header中的 key sign timestamp
		token := c.Request.Header.Get("X-Auth-Token")
		key := c.Request.Header.Get("X-Auth-Key")
		timestamp := c.Request.Header.Get("X-Auth-TimeStamp")
		fmt.Printf("[info] key is %s ,timestamp is %s\n", key, timestamp)

		//获取Post put请求模式下的Content-length
		conlength := c.Request.Header.Get("Content-Length")
		fmt.Printf("[info] Content-Length is %s\n", conlength)
```

##### 获取请求 Method Host URL ContentLength

```
		//判断请求Method
		// GET POST PUT DELETE  OPTIONS
		fmt.Println("[info]", c.Request.Method)
		fmt.Println("[info]", c.Request.Host) // localhost:8080
		fmt.Println("[info]", c.Request.URL) ///?id=3&name=zhangsan&address=beijing
		//获取请求头中数据长度 ContentLength
		fmt.Println("[info]", c.Request.ContentLength) // POST for 16  or  GET for 0
```

##### CORS 跨域请求   (未测试)

```
  // CORS middleware
    g.Use(CORSMiddleware())


func CORSMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Writer.Header().Set("Access-Control-Allow-Origin", "*")
        c.Writer.Header().Set("Access-Control-Allow-Headers", "Content-Type, Content-Length, Accept-Encoding, X-CSRF-Token, Authorization")
        if c.Request.Method == "OPTIONS" {
            c.Abort(200)
            return
        }
        c.Next()
    }
}
```
* [Json not work · Issue #149 · gin-gonic/gin · GitHub](https://github.com/gin-gonic/gin/issues/149)

##### 定义 struct 时，参数的首字母要大写才能被访问到

```
type ReturnMsg struct {
	Code int         `json:"code"`
	Msg  string      `json:"msg"`
	Data interface{} `json:"data"`
}

```

* [Go json.Marshal(struct) returns &quot;{}&quot; - Stack Overflow](https://stackoverflow.com/questions/26327391/go-json-marshalstruct-returns)


##### 输出一个struct对象

```
	...
	c.JSON(403, ReturnMsg{Code: 1, Msg: "req error"})
	...

type ReturnMsg struct {
	Code int         `json:"code"`
	Msg  string      `json:"msg"`
	Data interface{} `json:"data"`
}

```

***

##### gin中间件阻止请求继续访问

`c.Abort()`

```
		token := c.Request.Header.Get("X-Auth-Token")
		key := c.Request.Header.Get("X-Auth-Key")
		timestamp := c.Request.Header.Get("X-Auth-TimeStamp")

		//判断请求头中是否含有必须的三个参数
		if token == "" || key == "" || timestamp == "" {

			//经测试，c.Abort() 在前在后均可
			// c.Abort()
			// c.JSON(403, ReturnMsg{Code: 1, Msg: "req error"})

			c.JSON(403, ReturnMsg{Code: 1, Msg: "req error"})
			c.Abort()
			return //一定要加return
		}

		c.Next()

```

* [go - Failed to Abort() context - gin - Stack Overflow](https://stackoverflow.com/questions/32467002/failed-to-abort-context-gin)

##### gin安装报错

安装 `gin` 包时可能会报 `x/net` 相关错误，可以先下载该必须包：

```
$ mkdir -p $GOPATH/src/golang.org/x/

$ cd $GOPATH/src/golang.org/x/

$ git clone https://github.com/golang/net.git net

$ go install net
```

* [Gin 安装报错 - 简书](http://www.jianshu.com/p/371f627f4dda)

***

#### golang中相关易错点

##### 时间戳转换

```
package main

import (
    "fmt"
    "time"
    "strconv"
)

func main() {
    i, err := strconv.ParseInt("1405544146", 10, 64)
    if err != nil {
        panic(err)
    }
    tm := time.Unix(i, 0)
    fmt.Println(tm)
}
```

* [date - How to parse unix timestamp in golang - Stack Overflow](https://stackoverflow.com/questions/24987131/how-to-parse-unix-timestamp-in-golang)

***

##### 什么类型可以声明为常量及在func外部声明变量时不能使用`:=`

数字类型，字符串或布尔类型可以声明为 `const` 常量；`array` ,` slice` 或 `map` 不能声明为常量。

map类型不能声明为常量:

```
const myString = "hello"
const pi = 3.14 // untyped constant
const life int = 42 // typed constant (can use only with ints)
const ( 
   First = 1
   Second = 2
   Third = 4
)

```

***

在函数func外部声明变量，要用完整模式：

```
var romanNumeralDict = map[int]string{
	1:"a",
	2:"b",
}
```

在函数func内部声明变量，可以使用缩写模式：

```
romanNumeralDict := map[int]string{
...
}
```

* [go - How to declare constant map in golang - Stack Overflow](https://stackoverflow.com/questions/18342195/how-to-declare-constant-map-in-golang)


##### 判断map中是否存在某值，可以写在一行

```
// 查找键值是否存在
if v, ok := m1["a"]; ok {
	fmt.Println(v)
} else {
	fmt.Println("Key Not Found")
}
```

##### map排序

`map` 是无序的:

```

import (
	"fmt"
	"sort"
)

func main() {
	// fmt.Printf("Hello,是按揭")
	// fmt.Printf("ni好呀")

	m := map[string]string{
		"sign":      "1399dke",
		"user":      "zhangsan",
		"timestamp": "36644747373",
		"id":        "3",
	}
	fmt.Println(m)
	var keys []string
	for k := range m {
		// fmt.Println(k)
		keys = append(keys, k)
	}

	sort.Strings(keys)
	// fmt.Println(keys)
	for _, k := range keys {
		fmt.Println("key:", k, "Value :", m[k])
	}
}
```

* [go - sort golang map values by keys - Stack Overflow](https://stackoverflow.com/questions/23330781/sort-golang-map-values-by-keys)

***

##### for range 

`for range` 可以遍历 `slice` 或 `map`。并通过两个参数(index和value)，分别获取到slice或者map中某个元素所在的index以及其值。

在Go的 `for…range` 循环中，Go始终使用**值拷贝**的方式代替被遍历的元素本身。

```
for index, value := range mySlice {
    fmt.Println("index: " + index)
    fmt.Println("value: " + value)
}
```

##### go语言string、int、int64互相转换

```
#string到int
int,err:=strconv.Atoi(string)
#string到int64
int64, err := strconv.ParseInt(string, 10, 64)
#int到string
string:=strconv.Itoa(int)
#int64到string
string:=strconv.FormatInt(int64,10)
```

##### http.Request

```
func getURL(w http.ResponseWriter, r *http.Request) {
    url := r.URL.String()
}
```

* [How to convert *url.URL to string in GO, Google App Engine - Stack Overflow](https://stackoverflow.com/questions/13896592/how-to-convert-url-url-to-string-in-go-google-app-engine)


##### 中文 url 编码问题

```
import "net/url"

	//url编码
    str := "中文-_."
    unstr := "%2f"
    fmt.Printf("url.QueryEscape:%s", url.QueryEscape(str))
    fmt.Println()
    s, _ := url.QueryUnescape(unstr)
    fmt.Printf("url.QueryUnescape:%s", s)
    fmt.Println()
```

go 中url编码和字符转码(类似php中的urlencode 和htmlspecialchars):

```
package main

import (
    "fmt"
    "html"
    "net/url"
    "testing"
)

func Test_Escape(t *testing.T) {
//url编码
    str := "中文-_."
    unstr := "%2f"
    fmt.Printf("url.QueryEscape:%s", url.QueryEscape(str))
    fmt.Println()
    s, _ := url.QueryUnescape(unstr)
    fmt.Printf("url.QueryUnescape:%s", s)
    fmt.Println()
//字符转码
    hstr := "<"
    hunstr := "&lt"
    fmt.Printf("html.EscapeString:%s", html.EscapeString(hstr))
    fmt.Println()
    fmt.Printf("html.UnescapeString:%s", html.UnescapeString(hunstr))
    fmt.Println()
}
```

* [go 中url编码和字符转码(类似php中的urlencode 和htmlspecialchars) - coolaf](http://ouapi.com/article/55.html)

***

##### golang中字符串拼接

一种说法：

> 如果是少量小文本拼接，用 “+” 就好  
> 如果是大量小文本拼接，用 strings.Join  
> 如果是大量大文本拼接，用 bytes.Buffer  

* [字符串连接哪一种方式最高效 - Go 技术社区 - golang](https://gocn.io/question/265)
* [golang 高效字符串拼接  - Go语言中文网 - Golang中文社区](http://studygolang.com/articles/3427)
* [go语言中高效字符串拼接](https://www.birdcat.cn/golang%E5%AD%A6%E4%B9%A0/go%E8%AF%AD%E8%A8%80%E4%B8%AD%E9%AB%98%E6%95%88%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%8B%BC%E6%8E%A5.html)

***

##### strings 包

* [Go语言开发-字符串-strings包 | Plum Wine Blog - 青梅酒博客](https://plumwine.me/programming-in-go-fmt-strings-package/)


##### 获取字符串的MD5值

```
import (
    "crypto/md5"
    "encoding/hex"
)

func GetMD5Hash(text string) string {
    hasher := md5.New()
    hasher.Write([]byte(text))
    return hex.EncodeToString(hasher.Sum(nil))
}
```

* [md5 example](https://gist.github.com/sergiotapia/8263278)

***

#### 相关链接

* [GitHub - gin-gonic/gin: Gin is a HTTP web framework written in Go (Golang). It features a Martini-like API with much better performance -- up to 40 times faster. If you need smashing performance, get yourself some Gin.](https://github.com/gin-gonic/gin)
* [Gin Web Framework](https://gin-gonic.github.io/gin/)
* [gin-doc-cn/README.md at master · ningskyer/gin-doc-cn · GitHub](https://github.com/ningskyer/gin-doc-cn/blob/master/README.md)
* [Go语言web框架 gin | shanshanpt](http://shanshanpt.github.io/2016/05/03/go-gin.html)
* [Gin 安装报错 - 简书](http://www.jianshu.com/p/371f627f4dda)
* [Building Go Web Applications and Microservices Using Gin - Semaphore](https://semaphoreci.com/community/tutorials/building-go-web-applications-and-microservices-using-gin)
* [Build a RESTful API Server with Golang and MySQL - Jinchuriki](http://blog.narenarya.in/build-rest-api-go-mysql.html)
* [Golang 微框架 Gin 简介 - 简书](http://www.jianshu.com/p/a31e4ee25305)
* [How to create a basic Restful API in Go – Etienne Rouzeaud – Medium](https://medium.com/@etiennerouzeaud/how-to-create-a-basic-restful-api-in-go-c8e032ba3181)
* [Gin middleware examples - Dan Sosedoff](https://sosedoff.com/2014/12/21/gin-middleware.html)
* [REST Microservices in Go with Gin](http://txt.fliglio.com/2014/07/restful-microservices-in-go-with-gin/)
* [Go: Templating with the Gin Web Framework](http://www.markhneedham.com/blog/2016/12/23/go-templating-with-the-gin-web-framework/)

***

#### 相关问题

* 关于数据绑定 [how to bind query string? · Issue #742 · gin-gonic/gin · GitHub](https://github.com/gin-gonic/gin/issues/742)
* 关于模板文件 [go - How to make templates work with gin framework? - Stack Overflow](https://stackoverflow.com/questions/38042181/how-to-make-templates-work-with-gin-framework)
