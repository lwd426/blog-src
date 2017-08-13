title: 做过的一个angular+node产品实录
date: 2014-11-20 10:15:49
tags: 
- 前端
categories: 前端

---

### 题记：
分享一下自己之前做的一个产品，这个产品有个特点：`所有前端工作，包括页面及交互设计、css、所有js代码包括后端部分node接口都是我一个人搞的。现在想想，好像当年很强大的样子T_T`

>ps:由于本文只是用来讲解页面构成和组件划分。为了避免侵权行为，特意把所有文字都p掉了，影响了美观，请谅解。
    
### 下面是实际页面图：
<!-- more -->

![](/images/map.png)

图上已经进行了模块划分。模块基本是以功能组件为单位划分的，分别是header(当然还有footer组件balabala,不赘述)、selector、regioner和maper。

### 架构说明
产品前端整体架构是angularJS+jquery的组合。

>ps: 课本上不是说angularJS和jquery最好不要一起用么!!!这里特殊说明下，请注意maper组件，这个组件是封装了百度地图api，很多东西都要用jquery比较方便，而且把angularJS的jQlite换成jQuery也不是难事，所以我就毅然的加上了jQuery。

#### 目录结构基本如下：

```javascript:
mapcenter
- node_moudles
- src
- -server
- - - utils
- - - interceptor
- - - config
- - - libs
- - - routes
- - - app.js
- - - index.js
- - fontor
- - - styles
- - - fonts
- - - images
- - - filters
- - - services
- - - factories
- - - components
- - - plugins
- - - index.js
```
#### 实现细节

* 单页面应用
* 采用`组件分治`的原则，做到组件可拔插，想用就用，想卸载就卸载
* 各个组件都有自己的样式目录、静态资源目录、指令的实现目录、控制器实现目录。使后期维护和同步开发的效率大大提升
* 所有的公共代码都移到service中，service都放到了service目录里
* 组件之间通过service和广播传递数据
* 把调用百度地图的api接口封装出mapService，来作为地图公共服务，来被指令和控制器调用
* 没有使用bootstap样式库，而是自己实现效果，采用了css3的盒式等新特性
* 使用html5的媒体查询做到自适应屏幕大小调整布局

自己记录下，在各位面前板门弄斧，实在惭愧。