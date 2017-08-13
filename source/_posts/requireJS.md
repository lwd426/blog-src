title: 学习笔记--RequireJS的一些使用理解（结合kibana源码）
date: 2015-10-02 09:10:09
tags: 
- 模块化
- 自己的笔记
categories: 模块化

----------

### 题记--js模块化

js的模块化一直在用，用的就是RequireJS。但是很多牛逼的用法，确是在不经意间使用的，特此查明白为什么。

### 模块化：前端和后端

模块化分为前端模块化和后端模块化。

#### 后端模块化---npm
后端模块化使用的最多的当属npm(node 模块管理器)了，它采用的是CommonJS规范定义和加载模块：

1. 定义模块：你只需要在定义模块的js文件里require()来加载其他模块（npm会为你找到对应的模块），然后使用module.exports导出自身供其他模块使用。
2. 只是使用模块：不需要导出，只是使用就好了

<!-- more -->

```javascript
module.js
var _ = require('lodash');//引入第三方模块
var myself = require(__dirname+'/myself/index');//引入自定义模块

.....使用加载的模块....

（如果module.js也是一个自定义的模块，那么就要module.exports = {}来导出自己，供其他模块引入）
module.exports = {
	func = func();
}
```

#### 前端模块化---requireJS
前端模块化使用的最多的就是requireJS这个库了。它采用的是AMD(异步加载模块)规范来异步加载模块。特点如下：

4. **requireJS使用的模块必须符合AMD规范**，所以当你使用不符合此规范的第三方库时，你就要使用require.config(配置对象);一般此配置都单独放到一个文件里(require.config.js)。配置对象有两个参数对象：paths和shim。
5. **paths**: 为了方便管理，把所有的模块可以在此处做一个路径映射，这样，在定义和使用模块的时候，只需要引入对应的模块key值就行了。
6. **shim**:简单来说就是对requirejs要引用的第三方非AMD规范的插件、类库的特殊处理。**为那些没有使用define()来声明依赖关系、设置模块的"浏览器全局变量注入"型脚本做依赖和导出配置。**通常有两种方法：
  * A: AMD化.通过define封装一下
  * B: config shim. 
7. **加载require.js**:要在视图里引入require.js和require.config.js。
8. **每个模块定义在一个js文件中**
9. **定义模块**：defind(['a','b','c'],function(a, b ,c){ ... return {}}); (还有一种懒加载的方式引入模块，见下文)
10. **使用模块**：require(['a','b','c'],function(a,b,b){...});(还有一种懒加载的方式引入模块，见下文)
11. 加载完毕基础文件后，不需要加载模块文件，只需要**加载主模块文件**，有以下两种方式：
   * 直接在require.js引入引入标签里，定义一个data-main属性，值为主模块（入口模块）的路径
   * 直接在加载完require.config.js后，用require(['主模块']，function(){});
 
以下举例说明这些要点。

### RequireJS的基本用法--demo

这块不用啰嗦，直接上段demo，然后指出几点就ok了。

```javascript
//定义模块:使用define()方法（基础方法）  main.js
define(['jquery', 'underscore', 'angular', 'first', 'app','ctrl'], function($, _ , angular, f, app, ctrl){
		var kibana = {};
		kibana.init = function(){
		 	$(document).ready(function(){
		 		angular.bootstrap(document, ["appccc"]);
		 	});
		}
		return kibana;
});
 
```

```javascript
//定义映射文件：使用require.config({});
require.config({
	paths: {
		"jquery": "/bower_components/jquery/dist/jquery.min",
		"underscore": "/bower_components/underscore/underscore-min",
		"angular": "/bower_components/angular/angular.min",
		"angular-route": "/bower_components/angular-route/angular-route.min",
		"first": "amd_modules/first",
		"app": "amd_modules/app",
		"ctrl": "amd_modules/ctrl",
		"lwd426": "main"
	},
	shim: {
		"angular":{
			exports: "angular"
		},
		"angular-route": {
			deps: ["angular"],
			exports: "angular-route"
		}
	}
});
```

```javascript
//加载require.js、配置文件和引用模块
<!DOCTYPE html>
<html>
  <head>
    <title>这是我的测试</title>
    <link rel='stylesheet' href='/stylesheets/style.css' />
    <link rel='stylesheet' href='/bower_components/bootstrap/dist/css/bootstrap.min.css' />
  </head> 
  <body>
    <div class="container">
      <div ng-controller="hahaControler">
        <h1 ng-bind="text.message"></h1>
      </div>
    </div>
    <!-- 这是一种加载入口模块的方法 -->
    <script src="bower_components/requirejs/require.js" data-main='xxx/main'></script>
    <script type="text/javascript" src="require.config.js"></script>
    <script type="text/javascript">
    	//这是另一种加载入口模块的方法
    	require(['lwd426'],function(app){
    		app.init();
    	});
    </script>
	</body>
</html>
```
### 灵活加载模块的方式

为了让代码更灵活，可以采用commonJS加载模块的方式。下面，使用这种方式来重写定义模块的代码：

```javascript
define(function(require){
		var $ = require('jquery');
		var _ = require('underscore');
		var angular = require('angular');
		var f = require('first');
		var app = require('app');
		var ctrl = require('ctrl');
		var kibana = {};
		 //var app = angular.module("app", []);
		 kibana.init = function(){
		 	$(document).ready(function(){
		 		angular.bootstrap(document, ["appccc"]);
		 	});
		 }
		 return kibana;
});
```
#### 懒加载模块
在模块使用时才加载模块能够提高性能，以下写个小demo:
```javascript
define(function(require){
	var _ = require('lodash');
	//以下为伪代码，释义即可
	xxx.onclick = function(){
		require('click_uitls').click(function(){
			...
		});
	}
});
```

### require()和define()的概念问题

可以参考[http://www.zhihu.com/question/21260764](http://www.zhihu.com/question/21260764)

### 有关前端模块化打包和使用Browerify来在前端使用npm的话题，以后在写吧