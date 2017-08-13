title: JavaScript应该使用的编码风格
date: 2012-07-09 12:10:09
tags:
- javascript
categories: 基础

---

写代码一直都是‘随心而写’，并没有什么风格和过多的思考，这也导致后来阅读代码和修改bug的过程变得十分困难，所以，还是尽早的注意下编码风格这些东西吧。这里借鉴《javascript语言精粹》的第9章，和自己平时的经验，总结一下：
<!-- more -->

1. 要在逗号，和冒号：左右都加一个空格，这样有较好的阅读便利；
	
	```javascript
	obj = {
		name : 'lwd426'
	}
	
	obj.say( 'hello', 'nacy');
	```
2. 在if和(之间也加一个空格，这能告诉我们‘这不是个调用语句’
3. 只有在调用的时候才紧密连接，比如obj.saysth();和array[0];
4. 在if语句中，无论有多少条语句（即便是一条）也要用{}框住;
    
    ```javascript
    if (valid) {
    
    }
    ```
5. 使用K&R风格，把{放到一行的结尾，而不是下一行的开头，这能避免js的return语句中的一个可怕的设计错误：
>因为js会自动为行尾插入分号，所以如下代码就会让人好难过啊：

	```javascript
	function(name) {
		return 
		{
			name : name
		}
	}
	到return就直接返回了undefined,一直是undefined.
	```
6. 代码一定要有注释，并保持其是最新的。
7. 不要用无谓的初始化代码。比如

 	```javascript
 	var i = 0;
 	```
8. 在函数最起始部分声明所有的变量；
9. 不在if的条件语句中使用赋值表达式，因为会于比较表达式弄混：
	
	```javascript
	if (a = b) { ... }
	可能的本意是：
	if (a === b) { ... }
	```
10. 在switch case语句中，每个case都用break，不要穿越；
11. 最多只用一个全局变量，像很多库一样（jQuery）;
12. 使用闭包来隐藏信息，增强模块的健壮性；
13. 简单的if语句可以用三项表达式表示：
	
	```javascript
	var c = false;
	if (a > 0) {
		c = true;
	} else {
		c = false;
	}
	可以等价于
	var c = a > 0 ? true : false;
	```
14. 用 || 来确保变量有默认值：
	
	```javascript
	var a = b || 0;
	```
15. parseInt是一个把字符串转换为整数的函数。
    
    * 当遇到非数字就停止解析，所以:
    
    	```javascript
    parseInt('16') === parseInt('16 tons')。
    ```
    * 当第一个字符是0，那么它会用八进制解析。所以一定要带上此函数的第二个参数---基数参数。
    
      ```javascript
    parseInt('08', 10) === 8 //true
    ```
16. **把所有的变量声明都放到函数体的顶端。**
	 
	 js执行环境分为声明阶段和执行阶段:
	 
	 * 声明阶段:JavaScript引擎会为所有的变量与函数声明`创建标识符`,然后`定义函数`。
	 * 执行阶段：`函数均已被定义`，但`所有变量（包括函数表达式）`的值均`未定义`
	 
	 原因：`变量提升`指函数中所有变量声明会在函数执行时被“提升”至函数体顶端。规律如下：
	 * 变量的声明会被提升，赋值不会
	 * 函数声明时，函数标示符和函数体都会被提升，所以函数不受影响
	 * 函数表达式与变量一样，只是标示符声明被提升。

17. DOM操作为更新列表内容时，不使用html()方法，而是用$("ol").empty().append(列表数组);原因如下：
	* 已存在的DOM元素理应被清理掉，这样做利于解除相关事件监听器的引用，使内存得到回收
	* 这是一项比html()方法更稳妥的一种清理方法。html()方法在富客户端js应用中使用，可能会导致一些极为怪异的bug

18. 优化switch的臃肿结构的通常做法是采用对象的键值对的方式,这样代码嵌套变浅了，而且代码看起来更清爽：

	```javascript
	switch(name){
		case 'A' : //todo1 break;
		case 'B' : //todo2 break;
		case 'C' : //todo3 break;
		case 'D' : //todo4 break;	
		default: //todo
	}
	
	//优化
	var nameList = {
		'A' : function(){ //todo1 },
		'B' : function(){ //todo2 },
		'C' : function(){ //todo3 },
		'D' : function(){ //todo4 },
	}
	
	nameList[name];

	```
19. 类数组对象转成数组对象：Javascript中存在一种名为伪数组的对象结构。比较特别的是 arguments 对象。我们能通过 Array.prototype.slice.call 转换为真正的数组的带有 length 属性的对象，这样 类数组对象 就可以应用 Array 下的所有方法了。

	```javascript
	function log(){
  		var args = Array.prototype.slice.call(arguments);
  		args.unshift('(app)');
  		console.log.apply(console, args);
	};
	```	
	
20. **this的设计错误**：当一个函数并非做为一个对象的方法时，那么它就是被当做一个函数来调用的：

	```javascript
	var sum = add(3, 4); //sum 的值为7
	``` 
	
  当函数以此模式调用时，this被绑定到全局对象。这是语言设计上的一个错误，倘若语言设计正确，**当内部函数被调用时，this应该仍然绑定到外部函数的this变量**。这个设计错误错误的后果是方法**不能利用内部函数来帮助它工作，因为内部函数的this被绑定了错误的值，所以不能共享该方法对对象的访问权**。幸运的是，有一个很容易的解决方案：如果该方法定义一个变量并给他赋值为this，那么内部函数就可以通过那个变量访问到this。
  
  ```javascript
	//给myObject增加一个double方法
	myObject.double = function(){
   	 	var that = this; //解决方案
   	 	var helper = function(){
        	that.value = add(that.value,that.value);
    	};
    	helper();//以函数的形式调用helper
	};
	//以方法的形式调用double
	myObject.double();
	document.writeln(myObject.getValue()); //6	```
	
21. **利用模块定义组件**：（来自《JavaScript应用程序设计》-89页）。思路如下：
    * 在模块内定义DOM结构并给DOM注册事件，完成整个组件静态DOM的编写
    * 在公开接口函数中传入外部数据，函数的实现逻辑就是利用外部数据来填充DOM，实现DOM。
    * 最后通过module.exports = 公开接口函数导出组件实例。
    
    ```javascript
    var $ = require('jquery-browserify'),
    	checkedinClass = 'icon-check',
    	listClass = 'dropdown-menu',
    	guestClass = 'guest';
  		
  		toggleCheckedIn = function toggleCheckedIn(e){
  			$(this).toggleClass(checkedinClass):
  		}
  		
  		$listView = $('<ol>', {
  			id : 'guestlist-view',
  			'class' : listClass
  		}).on('click', '.' + guestClass, toggleCheckedIn),
  		
  		render = function render(guestlist){
  			guestlist.forEach(function (guest){
  				$guest = $('<li class=>"' + guestClass + '">' + '<span class="name">' + guest + '</span></li>');
  				$guest.appendTO($listView);
  			});
  			
  			return $listView;
  		},
  		
  		api = {
  			render : render
  		};
  		
  	module.exports = api;	
  		
    ```
	
22. parseInt与parseFloat的区别：parseInt()和parseFloat()两个方法都是从左边的字符串开始查找，如果第一个字符不是数字或者负号（在parseFloat()还可以是个小数点)。一旦它们遇到了这样的一个字符，它们就返回自己提取的数字:
	
	```js	
	parseInt("1234blue"); //returns 1234 
	parseInt("0xA"); //returns 10 
	parseInt("22.5"); //returns 22 
	parseInt("blue"); //returns NaN
	parseInt("010"); //returns 8 
	parseInt("010", 8); //returns 8 
	parseInt("010", 10); //returns 10
	parseInt("-10.9Blue"); //return -10
	parseInt(".9Blue"); //return 0
	parseInt("-2.1e4xyz") //-2
	
	parseFloat("1234blue"); //returns 1234.0 
	parseFloat("0xA"); //returns NaN 
	parseFloat("22.5"); //returns 22.5 
	parseFloat("22.34.5"); //returns 22.34 
	parseFloat("0908"); //returns 908 
	parseFloat("blue"); //returns NaN
	parseInt("-10.9Blue"); //return -10.9
	parseInt(".9Blue"); //return .9
	parseFloat("-2.1e4xyz") //-21000
	```
继续完善....