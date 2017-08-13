title: 读书笔记—《JavaScript设计模式与开发实践》- 深入原型的道理
date: 2015-01-09 16:24:09
tags:
- 读书笔记
- 学习笔记
categories: 读书笔记

----

读了读这本书，感觉作者从新的角度去解释了JS中很传统的一些概念和道理。虽然，ES6已经得到使用，但我认为把基础弄清楚了还是大有好处的。下面是自己的读书笔记，作为一个记录。

<!-- more -->

### 面向对象的JavaScript

第一章设计几个比较重要的概念：多态、封装、原型。

#### 封装
静态语言（比如java）的封装是通过“类”这个概念做到的，而js中没有类的概念，是弱类型语言，它的封装是通过闭包做到的。

#### 多态

* 多态背后的思想是“做什么”和“谁去做以及怎样做”分离开来
* 归根结底是要消除类的耦合关系
* 是通过把过程化的条件分支语句转化为对象的多态性。
* 这给予了我们的代码扩展能力，代码看起来是可生长的，也是符合开放-封闭原则的，更优雅和安全。

静态语言的多态是通过父类、抽象类、接口这些概念通过封装和继承做到。而对于JavaScript来说，一个变量的类型在运行时是可变的，所以多态是与生俱来的。

#### 原型

> JavaScript是一门基于原型继承的语言

说实话，这句话我读了好久都没太清楚，太抽象了，那么我就用大白话解释下：

* js是弱类型的语言
* js创建一个对象的过程就是：以一个对象为原型对象，把它克隆一份，然后在做其他的处理。不会实例化一个类。
* 每个对象都有一个隐藏属性__proto__：它指向自己的**构造函数的原型对象**。
* 原型链是原型继承的基础。每个对象的__proto__属性又是原型链实现的基础。一个属性在对象上找不到，就会去它的构造函数的原型函数上去找，再找不到就继续向上。这个过程的进行就实现了继承。这也就是JS的继承方式。代码举例：

	```js
	function A(){...}
	var a = new A();
	
	function B(){}
	var b = new B();
	//让b继承a实际上就是让b对象的构造函数的原型对象指向a
	b._proto__ = B.prototype = a;
	a = new A();
	//所以只需要让 
	B.prototype = new A();
	//就完成了b继承a
	
	```
	（在ES6中定义了class和extends来完成继承）
	
	```js
	class A(){
		constructor(name){
			this.name = name
		}
		getName(){
			return this.name;
		}
	}
	
	class B() extends A{
		construtor(){
			super(this);
		}
		getName(){
		
		}
	}
	```
	
* 当一个函数被new调用，那么它就是构造器函数，注意，它总会返回一个对象：如果构造器函数中，显式地返回了一个对象，那么这就是结果；如果没显式返回内容或显式返回的非对象，结果就是this。它的执行过程如下：
	
	```js
	var objectFactory = function(){
		var obj = new Object(), // 从 Object.prototype 上克隆一个空的对象
		Constructor = [].shift.call( arguments ); // 取得外部传入的构造器,此例是 Person
		obj.__proto__ = Constructor.prototype; // 指向正确的原型
		var ret = Constructor.apply( obj, arguments ); // 借用外部传入的构造器给 obj 设置属性return typeof ret === 'object' ? ret : obj;// 确保构造器总是会返回一个对象
};
	```

### this、apply和call

apply和call的两个最用：

1. 改变this的指向（也可用Function.prototype.bind函数给一个函数绑定this对象的值）
2. 借用其他对象的方法（其中arguments对象最常用）

### 闭包和高阶函数

#### 闭包和面向对象的设计可以互转

过程与数据的结合是形容面向对象中的“对象”时经常使用的表达。**对象以方法的形式包含了过程,而闭包则是在过程中以环境的形式包含了数据。通常用面向对象思想能实现的功能,用闭包也能实现。**反之亦然。在JavaScript语言的祖先Scheme语言中,甚至都没有提供面向对象 的原生设计,但可以使用闭包来实现一个完整的面向对象系统。

下面来看看这段跟闭包相关的代码:

```js

var extent = function(){ var value = 0;
         return {
             call: function(){
					value++;
					console.log( value ); 
					}
} };
var extent = extent();
extent.call(); 
extent.call(); 
extent.call();
// 输出:1 
// 输出:2 
// 输出:3
如果换成面向对象的写法,就是:
var extent = { 
		value: 0,
		call: function(){ 
			this.value++;
			console.log( this.value ); 
			}
};
extent.call(); 
extent.call(); 
extent.call();
// 输出:1 // 输出:2 // 输出:3
或者:

var Extent = function(){ 
	this.value = 0;
};
Extent.prototype.call = function(){ 
	this.value++;
	console.log( this.value ); 
};
var extent = new Extent();
extent.call(); 
extent.call();

```


#### 高阶函数实现AOP-面向切面

AOP(面向切面编程)的主要作用是**把一些跟核心业务逻辑模块无关的功能抽离出来**,这些 跟业务逻辑无关的功能通常包括日志统计、安全控制、异常处理等。把这些功能抽离出来之后, 再通过“动态织入”的方式掺入业务逻辑模块中。这样做的好处首先是可以保持业务逻辑模块的纯净和高内聚性,其次是可以很方便地复用日志统计等功能模块。
在 Java 语言中,可以通过反射和动态代理机制来实现 AOP 技术。而在 JavaScript 这种动态 语言中,AOP的实现更加简单,这是 JavaScript 与生俱来的能力。
通常,在 JavaScript 中实现 AOP,都是指把一个函数“动态织入”到另外一个函数之中,具体的实现技术有很多,本节我们通过扩展 Function.prototype 来做到这一点。代码如下:

```js
Function.prototype.before = function( beforefn ){
	var __self = this; // 保存原函数的引用
	return function(){ // 返回包含了原函数和新函数的"代理"函数
		beforefn.apply( this, arguments ); // 执行新函数,修正 this
		return __self.apply( this, arguments );// 执行原函数
	}
};

Function.prototype.after = function( afterfn ){ 
	var __self = this;
	return function(){
 		var ret = __self.apply( this, arguments ); 		afterfn.apply( this, arguments );
		return ret;
	} 
};
var func = function(){ 
	console.log( 2 );
};
func = func.before(function(){ 
	console.log( 1 );
}).after(function(){ 
	console.log( 3 );
}); func();
```

我们把负责打印数字 1 和打印数字 3 的两个函数通过 AOP 的方式动态植入 func 函数。通过 执行上面的代码,我们看到控制台顺利地返回了执行结果 1、2、3。这种使用 AOP 的方式来给函数添加职责,也是 JavaScript 语言中一种非常特别和巧妙的装饰
者模式实现。这种装饰者模式在实际开发中非常有用
...未完待续...