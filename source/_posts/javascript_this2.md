title: 对this新的认识--更深刻
date: 2015-04-24 10:25:00
tags: 
- javascript
categories: 基础

---

>今天，在cnode上看到一篇真的是很不错的文章[《图解Javascript上下文与作用域》](https://cnodejs.org/topic/5599521293cb46f578f0a57e)，已经被设置为精华帖了。

### 自己之前的理解：
改变了我之前对js的基础概念的以下简单的看法：

* 执行上下文: 就是函数的执行环境，比如`a.func();`里，func函数执行时的上下文就是对象a。
* this: 就是在新建对象时，把新建的对象的引用赋给了this变量；
* 作用域链: 这个理解是对的，简单的说就是`函数的作用域链就是函数以及他上面一直到global的所有定义的变量对象串成的链`。
* 闭包: 就是在返回值是函数的情况下，返回函数里拥有外层函数的作用域链。

### 引文重要观点摘录

1. 每一个上下文都会绑定一个变量对象（variable object），它就像一个容器，用来存`当前上下文中所有已定义或可获取的变量、函数等`
2. Js解释器总是在上下文堆栈的顶上下文中执行
3. 在生成新的上下文时，首先会:
<!-- more -->
	* `绑定该上下文的变量对象`，其中包括arguments和该函数中定义的变量；
	* 之后会创建属于该上下文的`作用域链（scope chain）`
	* 最后将`this赋予这一function所属的Object`
4. 作用域链实际上就是自下而上地将所有嵌套定义的上下文所绑定的变量对象串接到一起
5. 内层function所包含的作用域链将会一起返回`，即使内层function在其他上下文中执行，其内部的作用域链仍然保持着原有的数据，而`当前的上下文可能无法获取原先外层function中的数据，使得function内部的作用域链被保护起来，从而形成“闭包”`
6. function中的this与作用域链不同，它是由执行该function时当前所处的Object环境所决定的
7. context与函数调用执行相关，相对scope，可以理解为一个动态的概念。context的值为拥有此函数代码的this所代表的值
8. scope与函数调用时的this是谁无关。应该说，函数代码本身决定了scope的范围

### 引用正文：从另一个角度看js

本文尝试阐述Javascript中的上下文与作用域背后的机制，主要涉及到执行上下文（execution context）、作用域链（scope chain）、闭包（closure）、this等概念。

#### Execution context

* 执行上下文（简称上下文）决定了Js执行过程中可以获取哪些变量、函数、数据，一段程序可能被分割成许多不同的上下文
* 每一个上下文都会绑定一个变量对象（variable object），它就像一个容器，用来存`当前上下文中所有已定义或可获取的变量、函数等`。
* 位于最顶端或最外层的上下文称为全局上下文（global context），全局上下文取决于执行环境，如Node中的global和Browser中的window：

![](/images/js_global_context.jpg)

需要注意的是，上下文与作用域（scope）是不同的概念。Js本身是单进程的，每当有function被执行时，就会产生一个新的上下文，这一上下文会被压入`Js的上下文堆栈（context stack）`中，function执行结束后则被弹出，因此`Js解释器总是在栈顶上下文中执行`。

在生成新的上下文时，首先会:

1. `绑定该上下文的变量对象`，其中包括arguments和该函数中定义的变量；
2. 之后会创建属于该上下文的`作用域链（scope chain）`
3. 最后将`this赋予这一function所属的Object`

这一过程可以通过下图表示：

![](/images/js_global_context2.jpg)

#### this

上文提到this被赋予function所属的Object，具体来说，当function是定义在global对中时，this指向global；当function作为Object的方法时，this指向该Object：

```javascript
var x = 1;
var f = function(){
  console.log(this.x);
}
f();  // -> 1

var ff = function(){
  this.x = 2;
  console.log(this.x);
}
ff(); // -> 2
x     // -> 2

var o = {x: "o's x", f: f};
o.f(); // "o's x"
```

#### Scope chain

上文提到，在function被执行时生成新的上下文时会先绑定当前上下文的变量对象，再创建作用域链。我们知道function的定义是可以嵌套在其他function所创建的上下文中，也可以并列地定义在同一个上下文中（如global）。`作用域链实际上就是自下而上地将所有嵌套定义的上下文所绑定的变量对象串接到一起`，使嵌套的function可以“继承”上层上下文的变量，而并列的function之间互不干扰：

![](/images/js_global_context3.jpg)

```javascript
var x = 'global';
function a(){
  var x = "a's x";
  function b(){
    var y = "b's y";
    console.log(x);
  };
  b();
}
function c(){
  var x = "c's x";
  function d(){
    console.log(y);
  };
  d();
}
a();  // -> "a's x"
c();  // -> ReferenceError: y is not defined
x     // -> "global"
y     // -> ReferenceError: y is not defined
```

#### Closure

如果理解了上文中提到的上下文与作用域链的机制，再来看闭包的概念就很清楚了。每个function在调用时会创建新的上下文及作用域链，而作用域链就是将外层（上层）上下文所绑定的变量对象逐一串连起来，使当前function可以获取外层上下文的变量、数据等。如果我们在function中定义新的function，同时将内层function作为值返回，那么`内层function所包含的作用域链将会一起返回`，即使内层function在其他上下文中执行，其内部的作用域链仍然保持着原有的数据，而`当前的上下文可能无法获取原先外层function中的数据，使得function内部的作用域链被保护起来，从而形成“闭包”`。看下面的例子：

```javascript
var x = 100;
var inc = function(){
  var x = 0;
  return function(){
    console.log(x++);
  };
};

var inc1 = inc();
var inc2 = inc();

inc1();  // -> 0
inc1();  // -> 1
inc2();  // -> 0
inc1();  // -> 2
inc2();  // -> 1
x;       // -> 100
```

执行过程如下图所示，inc内部返回的匿名function在创建时生成的作用域链包括了inc中的x，即使后来赋值给inc1和inc2之后，直接在global context下调用，它们的作用域链仍然是由定义中所处的上下文环境决定，而且由于x是在function inc中定义的，无法被外层的global context所改变，从而实现了闭包的效果：

![](/images/js_global_context4.jpg)

#### this in closure

我们已经反复提到执行上下文和作用域实际上是通过function创建、分割的，而function中的this与作用域链不同，它是由执行该function时当前所处的Object环境所决定的，这也是this最容易被混淆用错的一点。一般情况下的例子如下：

```javascript
var name = "global";
var o = {
  name: "o",
  getName: function(){
    return this.name
  }
};
o.getName();  // -> "o"
```

由于执行o.getName()时getName所绑定的this是调用它的o，所以此时this == o；更容易搞混的是在closure条件下：

```javascript
var name = "global";
var oo = {
  name: "oo",
  getNameFunc: function(){
    return function(){
      return this.name;
    };
  }
}
oo.getNameFunc()();  // -> "global"
此时闭包函数被return后调用相当于：

getName = oo.getNameFunc();
getName();  // -> "global"
换一个更明显的例子：

var ooo = {
  name: "ooo",
  getName: oo.getNameFunc() // 此时闭包函数的this被绑定到新的Object
};
ooo.getName();  // -> "ooo"
当然，有时候为了避免闭包中的this在执行时被替换，可以采取下面的方法：

var name = "global";
var oooo = {
  name: "ox4",
  getNameFunc: function(){
    var self = this;
    return function(){
       return self.name;
    };
  }
};
oooo.getNameFunc()(); // -> "ox4"
或者是在调用时强行定义执行的Object：

var name = "global";
var oo = {
  name: "oo",
  getNameFunc: function(){
    return function(){
      return this.name;
    };
  }
}
oo.getNameFunc()();  // -> "global"
oo.getNameFunc().bind(oo)(); // -> "oo"
```

### context和scope的区别：

#### context:

context与函数调用执行相关，相对scope，可以理解为一个动态的概念。context的值为拥有此函数代码的this所代表的值。举例来说，var obj = new foo()，则context代表了obj。

#### scope:

scope与函数定义相关，相对context，可以理解为一个静态的概念。scope定义了函数内部的变量作用域。如果函数内部引用了其它函数中的变量，就形成了作用域链(scope chain)。scope与函数调用时的this是谁无关。应该说，函数代码本身决定了scope的范围。

scope就在那里，不离不弃。函数有没有调用，被谁调用，与它都无关系。

#### context和scope的关系:

每个函数调用都涉及到 scope和context这样一对形影不离的兄弟。他们俩不是相互对立的关系，分别代表了函数创建定义时和执行过程时的不同的形态。
在创建阶段，函数、变量、参数都会被定义好，此时也就形成了scope和scope chain。而当代码被执行时，也就产生了context。

引自：[《图解Javascript上下文与作用域》](https://cnodejs.org/topic/5599521293cb46f578f0a57e)