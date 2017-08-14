---
title: Golang下通过Gin体验WebSocket框架Melody
date: 2017-08-14 13:52:19
tags:
	- Golang
	- Gin
description: Melody 是一个 Go 语言的微型 WebSocket 框架，基于 github.com/gorilla/websocket 开发.
---

Melody 是一个 Go 语言的微型 WebSocket 框架，基于 github.com/gorilla/websocket 开发.

#### Gin-Gonic

获取包：

```
> go get github.com/gin-gonic/gin
```

添加引用：

```
inport "github.com/gin-gonic/gin"
```

创建Gin测试站点：

```
package main

import (
    "github.com/gin-gonic/gin"
)

func main() {

    r := gin.Default()

    r.GET("/", func(c *gin.Context) {
        c.String(200, "Hello Gin")
    })

    r.Run(":8080")
}
```

运行：

```
> go run main.go
```

***

#### Melody

获取包：

```
> go get gopkg.in/olahol/melody.v1
```

添加引用：

```
import "gopkg.in/olahol/melody.v1"
```

***

#### Simple Chat Demo

main.go：

```
package main

import (
	"github.com/gin-gonic/gin"
	"gopkg.in/olahol/melody.v1"
	"net/http"
)

func main() {
	r := gin.Default()
	m := melody.New()

	r.GET("/", func(c *gin.Context) {
		http.ServeFile(c.Writer, c.Request, "templates/index.html")
	})

	//websocket
	r.GET("/ws", func(c *gin.Context) {
		m.HandleRequest(c.Writer, c.Request)
	})

	m.HandleMessage(func(s *melody.Session, msg []byte) {
		m.Broadcast(msg)
	})

	r.Run(":8080")
}

```

index.html：

```
<html>
<head>
	<meta charset="UTF-8">
	<title>WebSocket</title>
	 <style>
    #chat {
      text-align: left;
      background: #f1f1f1;
      width: 500px;
      min-height: 300px;
      padding: 20px;
    }
  </style>
</head>
<body>
	<center>
      <h3>Chat</h3>
      <pre id="chat"></pre>
      <input placeholder="say something" id="text" type="text">
    </center>

    <script>
      var url = "ws://" + window.location.host + "/ws";
      var ws = new WebSocket(url);

      var name = "Guest" + Math.floor(Math.random() * 1000);

      var chat = document.getElementById("chat");
      var text = document.getElementById("text");

      var now = function () {
        var iso = new Date().toISOString();
        return iso.split("T")[1].split(".")[0];
      };

      ws.onmessage = function (msg) {
        var line =  now() + " " + msg.data + "\n";
        chat.innerText += line;
      };

      text.onkeydown = function (e) {
        if (e.keyCode === 13 && text.value !== "") {
          ws.send("<" + name + "> " + text.value);
          text.value = "";
        }
      };

    </script>
</body>
</html>
```

运行：

```
> go run main.go
```

***

* [GitHub - olahol/melody: Minimalist websocket framework for Go](https://github.com/olahol/melody)
