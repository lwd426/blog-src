title: 学习笔记--Javascript模块化那些事（总结导图）
date: 2015-09-12 13:10:09
tags: 
- 模块化
- 自己的笔记
categories: 模块化

----------

### 题记

自己抽空把js所有的能能涉及到的模块化有关的东西总结了下。作为知识贮备吧。

### 模块定义的几种方式

1. IIFE(立即执行函数)--模块模式
2. 纯闭包
3. CJS或AMD规范

#### IIFE -- 模块模式

IIFE是Immediately-Invoked Function Expression的简称。如果只需要闭包，避免全局变量污染，就可以使用IIFE。定义结构如下：

```
(function(){
	新作用域
})();
```
注意：此种方式还有一种补充，比较常用于使用闭包处理DOM的处理，就是原型的模块化:在闭包中声明原型的模块，然后在闭包里原型的外部维护私有状态。（详见JavaScript Web应用开发94页）

#### 纯闭包

如果想要模块完全不依赖闭包之外的代码，就把需要使用的变量导出其中。如果要提供公开的API，那么就导入全局对象。各种牛逼的库就是这么做的比如jQuery和Lodash。这样做，一切都完美的包在闭包中，可以使用JSHint找出因为声明的变量导致的问题。

定义结构如下：

```javascript
(function(win){
	var privateThing;
	
	function privateFunction(){
		...
	}
	
	win.api = {
		//公开接口们
		...
	}
})(window);
```
#### 使用CommonJS、AMD等规范定义模块
自己做了个思维导图 

![模块化思维导图](/images/modules_image.png)

地址：[模块化思维导图](http://naotu.baidu.com/file/b5d5f3f6e4898d8c5de1fcda12ff2045?token=a80d6b601914e91d)