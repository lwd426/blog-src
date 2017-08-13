title: JSON对象和字符串之间的相互转换
date: 2013-09-01 10:09:09
tags:
- javascript
categories: 基础

---

JSON和字符串的互转在javascript中是极为常用的，尤其是在使用java做数据接口，使用json作为主要的数据传输格式的时候。这次就总结下吧~

比如我有两个变量，我要将a转换成字符串，将b转换成JSON对象：

```javascript
var a={"name":"tom","sex":"男","age":"24"};
var b='{"name":"Mike","sex":"女","age":"29"}';
```
### 常用方法
在Firefox，chrome，opera，safari，ie9，ie8等高级浏览器直接可以用JSON对象的stringify()和parse()方法。
<!-- more -->
* JSON.stringify(obj)将JSON转为字符串。
* JSON.parse(string)将字符串转为JSON格式；

上面的转换可以这么写：

```javascript
var a={"name":"tom","sex":"男","age":"24"};
var b='{"name":"Mike","sex":"女","age":"29"}';
var aToStr=JSON.stringify(a);
var bToObj=JSON.parse(b);
alert(typeof(aToStr));  //string
alert(typeof(bToObj));//object
```
### IE8|IE7|IE6
1. ie8(兼容模式),ie7和ie6没有JSON对象，不过[http://www.json.org/](http://www.json.org/)提供了一个json.js,这样ie8(兼容模式),ie7和ie6就可以支持JSON对象以及其stringify()和parse()方法；你可以在https://github.com/douglascrockford/JSON-js上获取到这个js，一般现在用`json2.js`。

2. ie8(兼容模式),ie7和ie6可以使用eval()将字符串转为JSON对象，

```javascript
var c='{"name":"Mike","sex":"女","age":"29"}';
var cToObj=eval("("+c+")");
alert(typeof(cToObj));
```

### jQuery中也有将字符串转为JSON格式的方法
```javascript
jQuery.parseJSON( json )
```
接受一个标准格式的 JSON 字符串，并返回解析后的 JavaScript （JSON）对象。当然如果有兴趣可以自己封装一个jQuery扩展，jQuery.stringifyJSON(obj)将JSON转为字符串。