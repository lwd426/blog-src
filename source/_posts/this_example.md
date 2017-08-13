title: 笔记：构造函数中的this
date: 2013-04-03 10:15:49
tags: 
categories: javascript
---
总是搞不清this的用法，自己简单记录了下笔记:

<!-- more -->

```javascript
function myConstructor(message){
    this.message = message; //实例属性
    var sepatator = '_'; // 私有属性
    var myOwmer = this;
    function alertmessage(){
        alert(myOwmer .message);
    }
   //特权方法：可访问私有成员，并且可以被实例调用
    this.myfunc = function(){
           alert(sepatator+this.message);
    }
}
//共有方法
myConstructor.prototype.publicfunc = function(){
}
```

这是一个`构造函数`，可以用new运算符创建一个实例

```javascript
var obj = new myConstructor('dddd');
alert(obj.message); //'dddd'
alert(obj.separator); // undefined 因为是myConstructor的对象私有属性，实例是没有的
```
进一步说明，myConstructor里的私有方法 alertmessage方法体中的this指向的是 myContructor对象，而不是他的实例，所以，为了有意义，要把指向实例的this复制给间接变量myOwner.
this指向构造函数的实例。