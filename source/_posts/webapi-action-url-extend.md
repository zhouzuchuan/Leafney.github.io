---
title: Web API中路由中加了action后其他get、post开头的方法如何直接访问
date: 2016-04-17 10:06:00
tags:
    - ASP.NET
categories: 
    - 开发笔记
description: Web API中路由中加了action后其他get、post开头的方法如何直接访问
---

#### 需求 

通常我们在访问Web API接口时，默认自带的Action方法为以下这些：

```
 public IEnumerable<string> Get()
 public string Get(int id)
 public void Post([FromBody]string value)
 public void Put(int id, [FromBody]string value)
 public void Delete(int id)
```

默认的路由表规则为：

WebApiConfig.cs文件 

```
            config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional }
            );
```

匹配的Url为：

| 动作 | HTTP方法 | 相对路径 |
|---- | ----- | ---- |
| 获取全部| GET | /api/products |
|指定 id 获取 | GET | /api/products/id |
| 添加 | POST | /api/products |
| 更新 | PUT | /api/products/id |
| 删除 | DELETE | /api/products/id |

*** 

#### 特殊需求

我们在同一个 `Controller` 中，有时候可能需要定义多个相同 `Method` 的 `Action` 方法，比如：

```
 public string Get(int id)
 public string GetList(string name)
 public string GetPeople(int id)

```

那么这种情况下，我们就要再添加一个 `Route` 路由规则，添加一个 `{action}` 的占位符：

```
            config.Routes.MapHttpRoute(
              name: "DefaultApiAction",
              routeTemplate: "api/{controller}/{action}/{id}",
              defaults: new { id = RouteParameter.Optional }
          );
```

但这样我们就不能再用之前的那种方式来直接访问默认的`Get`或`Post`等方法了，每个Url链接都要加上`action`名称：

```
/api/products/get
/api/products/getlist
/api/products/post
```

这样的话我们就觉得反而更麻烦了。

*** 

#### 解决方法

那是不是有一种什么样的方法能让我们在添加多个相同类型的请求方法时，既能直接访问默认的方法，又能通过添加`action`名称来访问自定义添加的方法呢？

其实，我们只要修改路由规则如下即可：

```
using System.Net.Http;
using System.Web.Http.Routing;

            //可用的路由表

            config.Routes.MapHttpRoute("DefaultApiWithId", 
                "api/{controller}/{id}", 
                new { id = RouteParameter.Optional },
                new { id = @"\d+" });
            config.Routes.MapHttpRoute("DefaultApiWithAction",
                "api/{controller}/{action}");

            config.Routes.MapHttpRoute("DefaultApiGet",
                "Api/{controller}",
                new { action = "Get" }, 
                new { httpMethod = new HttpMethodConstraint(HttpMethod.Get) });
            config.Routes.MapHttpRoute("DefaultApiPost", 
                "Api/{controller}",
                new { action = "Post" },
                new { httpMethod = new HttpMethodConstraint(HttpMethod.Post) });


```

详细说明见： 
[wcf web api - Single controller with multiple GET methods in ASP.NET Web API - Stack Overflow](http://stackoverflow.com/questions/9499794/single-controller-with-multiple-get-methods-in-asp-net-web-api)

*** 

#### 再啰嗦一句  

如果同一种请求方式下，有多个同类型的方法，会请求默认状态下的那个：
比如 get方法：

```
 public string Get()
 public string GetTop()
```

如果请求：`http://localhost:47760/api/product` 
则访问到的是`Get()`方法，即使在顺序上`GetTop()`排在`Get()`方法前面。
