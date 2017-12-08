---
title: MVC和WebAPI如何从Filter向Action中传递数据
date: 2016-04-17 14:06:00
tags:
    - RestfulApi
    - ASP.NET
categories: 
    - 开发笔记
description: MVC和WebAPI如何从Filter向Action中传递数据
---

MVC和WebAPI如何从Filter向Action中传递数据

#### 需求 

最近在策划实现MVC项目中用户身份验证的功能时，考虑用MVC中的`Filter`过滤器来先从url链接中获取传递过来的token，在`Filter`中通过token获取用户的信息后，如果用户信息正确，则传递到`Controller`的`Action`中进行用户数据的操作。

那么，要如何从Filter中向Action中传递数据呢？  

how to pass data from filter to controller?  

*** 

**注意：下面所提到的Filter都指实现`ActionFilterAttribute`的过滤器**

#### MVC中 从Filter过滤器向Action中传递数据 

##### 方法一 通过 `RouteData` 来传值

```
//赋值：
filterContext.RouteData.Values.Add("Tname",UName);

//获取：
 var nm = RouteData.Values["Tname"]; 
```

测试通过。

详见： [ASP.NET MVC Pass object from Custom Action Filter to Action - Stack Overflow](http://stackoverflow.com/questions/1809042/asp-net-mvc-pass-object-from-custom-action-filter-to-action)

*** 

##### 方法二 通过 `ActionParameters` 来传值  

另一种方法是通过 `ActionParameters` 来设置，但在Action中是通过添加参数获取值的：  

```
 //赋值：  
 filterContext.ActionParameters.Add("number", Id);
 //获取：  
 public ActionResult Index(int number, Person person)
```

详见： [Manipulating Action Method Parameters - You've Been Haacked](http://haacked.com/archive/2010/02/21/manipulating-action-method-parameters.aspx/)

通过测试，发现这种方法可以隐藏真实的Action方法：

比如：  
请求的链接是 `http://localhost:47760/home/show?id=3&name=abc` 
而实际的Action为：  

```
 public ActionResult Show(string aaa){}
```

那么可以通过添加一个 `ActionFilterAttribute` 过滤器，并设置：

```
  this.Uname=getquerystring.name;
  filterContext.ActionParameters["aaa"] = UName;
```

这样虽然url中请求的参数时id和name，而实际请求参数是`aaa`；  

而实际的请求链接 `http://localhost:47760/home/show?aaa=haha` 也是可以访问的。 

*** 

##### 方法三 通过 `HttpContext.Items` 来传值

```
//Filter中赋值：  
filterContext.HttpContext.Items["tname"] = UName+"2324";
//Action中取值：  
var nm= HttpContext.Items["tname"];
```

测试通过。

通过测试发现好像这种方式比较合适。因为：可看到`Items`的解释为：  

> 在派生类中重写时，获取一个键/值集合，该集合在 HTTP 请求过程中可以用于在模块与处理程序之间组织和共享数据。

详见： [asp.net mvc - Accessing Action Filter&#39;s data in Controller Action - Stack Overflow](http://stackoverflow.com/questions/7039231/accessing-action-filters-data-in-controller-action)

*** 

#### WebAPI中从Filter向Action中传递数据

如何从Filter向Action中传递数据？

##### 方法一 通过 `Request.Properties` 来传值

```
//Filter中赋值：  
actionContext.Request.Properties["id"] ="134";
//Action中获取： 
var id= Request.Properties["id"];

```

或：

```
//赋值:  
actionContext.Request.Properties.Add("mykey", myObject);
//获取：  
object myObject;
Request.Properties.TryGetValue("mykey", out myObject);
//cast to MyType
```

测试通过。

详见： 
* [asp.net web api - WebApi: how to pass state from filter to controller? - Stack Overflow](http://stackoverflow.com/questions/15059161/webapi-how-to-pass-state-from-filter-to-controller)
* [asp.net web api - Pass an object from ActionFilter.OnActionExecuting() to an ApiController - Stack Overflow](http://stackoverflow.com/questions/28357141/pass-an-object-from-actionfilter-onactionexecuting-to-an-apicontroller)

*** 

##### 总结

`MVC` 用 : 

```
filterContext.HttpContext.Items[UnitOfWorkRequestKey] = UnitOfWork;
```

`Web API` 用 :  

```
actionContext.Request.Properties[UnitOfWorkRequestKey] = UnitOfWork;
```

详见： [c# - Web API Action Filter - Controller.TempData equivalent? - Stack Overflow](http://stackoverflow.com/questions/14921041/web-api-action-filter-controller-tempdata-equivalent)
