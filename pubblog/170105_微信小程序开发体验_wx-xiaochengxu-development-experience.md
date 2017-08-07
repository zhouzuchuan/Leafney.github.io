### 微信小程序开发体验

https://mp.weixin.qq.com/debug/wxadoc/dev/

#### 简易教程

app.js、app.json、app.wxss 这三个，.js后缀的是脚本文件，.json后缀的文件是配置文件，.wxss后缀的是样式表文件。

* app.js是小程序的脚本代码。我们可以在这个文件中监听并处理小程序的生命周期函数、声明全局变量。调用框架提供的丰富的 API，如本例的同步存储及同步读取本地数据。
* app.json 是对整个小程序的全局配置。我们可以在这个文件中配置小程序是由哪些页面组成，配置小程序的窗口背景色，配置导航条样式，配置默认标题。注意该文件不可添加任何注释。详见：[配置](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/config.html?t=1475052047016)
* app.wxss 是整个小程序的公共样式表。


##### 创建页面

页面都在 `pages` 目录下。

微信小程序中的每一个页面的【路径+页面名】都需要写在 app.json 的 pages 中，且 pages 中的第一个页面是小程序的首页。

每一个小程序页面是由同路径下同名的四个不同后缀文件的组成，如：index.js、index.wxml、index.wxss、index.json。.js后缀的文件是脚本文件，.json后缀的文件是配置文件，.wxss后缀的是样式表文件，.wxml后缀的文件是页面结构文件。

页面的样式表是非必要的。当有页面样式表时，页面的样式表中的样式规则会层叠覆盖 app.wxss 中的样式规则。

***

#### 框架

框架提供了自己的视图层描述语言 WXML 和 WXSS，以及基于 JavaScript 的逻辑层框架，并在视图层与逻辑层间提供了数据传输和事件系统，可以让开发者可以方便的聚焦于数据与逻辑上。

##### 响应的数据绑定

框架的核心是一个响应的数据绑定系统。

整个系统分为两块视图层（View）和逻辑层（App Service）

***

#### 文件结构

框架程序包含一个描述整体程序的 app 和多个描述各自页面的 page。

一个框架程序主体部分由三个文件组成，必须放在项目的根目录：app.js app.json app.wxss


一个框架页面由四个文件组成，分别是：*.js  *.wxml *.wxss *.json

#### 配置

使用app.json文件来对微信小程序进行全局配置，决定页面文件的路径、窗口表现、设置网络超时时间、设置多 tab 等。

app.json 配置项列表

* `pages` 设置页面路径
* `window` 设置默认页面的窗口表现
* `tabBar` 设置底部 tab 的表现
* `networkTimeout` 设置网络超时时间 
* `debug` 设置是否开启 debug 模式

##### pages

* 接受一个数组，每一项都是字符串，来指定小程序由哪些页面组成。
* 对应页面的【路径+文件名】信息
* 数组的第一项代表小程序的初始页面。
* 小程序中新增/减少页面，都需要对 pages 数组进行修改
* 文件名不需要写文件后缀

##### window

设置小程序的状态栏、导航条、标题、窗口背景色。

* `navigationBarBackgroundColor`   导航栏背景颜色，如"#000000"
* `navigationBarTextStyle`  导航栏标题颜色，仅支持 black/white
* `navigationBarTitleText`  导航栏标题文字内容
* `backgroundColor`  窗口的背景色
* `backgroundTextStyle`   下拉背景字体、loading 图的样式，仅支持 dark/light
* `enablePullDownRefresh`  是否开启下拉刷新 false/true

##### tabBar

多 tab 应用（客户端窗口的底部有tab栏可以切换页面）, 可以通过 tabBar 配置项指定 tab 栏的表现，以及 tab 切换时显示的对应页面。

tabBar 是一个数组，**只能配置最少2个、最多5个 tab**，tab 按数组的顺序排序。

* `color`  tab 上的文字默认颜色
* `selectedColor`  tab 上的文字选中时的颜色
* `backgroundColor`  tab 的背景色
* `borderStyle`  tabbar上边框的颜色， 仅支持 black/white
* `list`  tab 的列表，最少2个、最多5个 tab

list 属性值：

* `pagePath`  页面路径，必须在 pages 中先定义
* `text`  tab 上按钮文字
* `iconPath`  图片路径，icon 大小限制为40kb
* `selectedIconPath`  选中时的图片路径，icon 大小限制为40kb

##### networkTimeout

设置各种网络请求的超时时间。

* `request`  wx.request的超时时间，单位毫秒
* `connectSocket`  wx.connectSocket的超时时间，单位毫秒
* `uploadFile`   wx.uploadFile的超时时间，单位毫秒
* `downloadFile`  wx.downloadFile的超时时间，单位毫秒

##### debug

在控制台面板中调试信息以 `info` 的形式给出，其信息有Page的注册，页面路由，数据更新，事件触发 。

#### page.json

每一个小程序页面也可以使用.json文件来对本页面的窗口表现进行配置。 页面的配置比app.json全局配置简单得多，只是设置 app.json 中的 window 配置项的内容，页面中配置项会覆盖 app.json 的 window 中相同的配置项。

页面的.json只能设置 window 相关的配置项,以决定本页面的窗口表现，无需写 window 这个键。

***

#### 逻辑层

##### App

###### App()

App() 函数用来注册一个小程序。接受一个 object 参数，其指定小程序的生命周期函数等。

object 参数说明：

* `onLaunch`   当小程序初始化完成时，会触发 onLaunch（全局只触发一次）
* `onShow`   当小程序启动，或从后台进入前台显示，会触发 onShow
* `onHide`   当小程序从前台进入后台，会触发 onHide
* 其他   开发者可以添加任意的函数或数据到 Object 参数中，用 this 可以访问

前台、后台定义： 当用户点击左上角关闭，或者按了设备 Home 键离开微信，小程序并没有直接销毁，而是进入了后台；当再次进入微信或再次打开小程序，又会从后台进入前台。

```
App({
  onLaunch: function() { 
    // Do something initial when launch.
  },
  onShow: function() {
      // Do something when show.
  },
  onHide: function() {
      // Do something when hide.
  },
  globalData: 'I am global data'
})
```

###### App.prototype.getCurrentPage()

getCurrentPage() 函数用户获取当前页面的实例。

###### getApp()

全局的 getApp() 函数，可以获取到小程序实例。

注意：

* App() 必须在 app.js 中注册，且不能注册多个。
* 不要在定义于 App() 内的函数中调用 getApp() ，使用 this 就可以拿到 app 实例。
* 不要在 onLaunch 的时候调用 getCurrentPage()，此时 page 还没有生成。
* 通过 getApp() 获取实例之后，不要私自调用生命周期函数。

***

##### Page

Page() 函数用来注册一个页面。接受一个 object 参数，其指定页面的初始数据、生命周期函数、事件处理函数等。

object参数说明：

* `data`  页面的初始数据
* `onLoad`  生命周期函数--监听页面加载
* `onReady`  生命周期函数--监听页面初次渲染完成
* `onShow`  生命周期函数--监听页面显示
* `onHide`  生命周期函数--监听页面隐藏
* `onUnload`  生命周期函数--监听页面卸载
* `onPullDownRefreash`  页面相关事件处理函数--监听用户下拉动作
* 其他  开发者可以添加任意的函数或数据到 object 参数中，用 this 可以访问

###### 初始化数据

初始化数据将作为页面的第一次渲染。数据必须是可以转成 JSON 的格式：字符串，数字，布尔值，对象，数组。


###### Page.prototype.setData()

setData 函数用于将数据从逻辑层发送到视图层，同时改变对应的 this.data 的值。

注意：

1. 直接修改 `this.data` 无效
2. 单次设置的数据不能超过 1024 KB 

###### setData() 参数格式

接受一个对象，以 key，value 的形式表示将 this.data 中的 key 对应的值改变成 value。


###### 页面的路由


***

#### 文件作用域

* 在 JavaScript 文件中声明的变量和函数只在该文件中有效；
* 不同的文件中可以声明相同名字的变量和函数，不会互相影响。

* 通过全局函数 getApp() 可以获取全局的应用实例
* 如果需要全局的数据可以在 App() 中设置

#### 模块化

模块只有通过 module.exports 才能对外暴露接口。

```
// common.js
function sayHello(name) {
  console.log('Hello ' + name + '!')
}
module.exports = {
  sayHello: sayHello
}
```

在需要使用这些模块的文件中，使用 require(path) 将公共代码引入。

```
var common = require('common.js')
Page({
  helloMINA: function() {
    common.sayHello('MINA')
  }
})
```

*** 

