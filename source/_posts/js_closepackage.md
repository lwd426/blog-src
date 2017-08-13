title: 对"闭包"最深刻的剖析
date: 2015-11-01 10:08:34
tags: 
- javascript
categories: 基础

---

### 题记--前言
一直在研究闭包，是它让JavaScript变得强大，也是它让我们对JavaScript感到很困惑。真所谓是"成也萧何败萧何"（PS:此句表达不是很准确）。

大多数JS书籍和博客都无非是告诉我以下几点：

* 闭包是外层函数返回值为内层函数
* 闭包是能访问外层函数作用域内参数和变量的函数（除了this和arguments）
* 闭包可以访问它被创建时的作用域对象
* 封装性：外界要访问（读取或修改）闭包内的私有变量，唯一的办法就是通过--特权方法。
* 闭包可以把传进来的参数保存在作用域链，这常用于偏函数和柯里化函数
* 只有当闭包不在引用所有外层作用域对象的变量和参数，外层作用域对象才会被销毁
* 闭包是JS做模块化开发的基础
* 闭包常用来做数据访问的控制（特权方法）
* 闭包让JS变得十分灵活，是它让函数在JS中成了"first-class object"
<!-- more -->

其他也就没有太多了，无非就是再举一个（下文）我们极为熟悉的给DOM循环设置监听的那个老掉牙而特别具有代表性的例子，但似乎没有几个地方真正讲明白了为什么闭包能解决问题。知道前段时间，我还一直理解为闭包是缓存了这个i值。但读了下面这个文章，才真正明白了，这个问题为什么能被解决：在for循环过程中产生了许多闭包，两种方法的闭包的作用域链不同：

```javascript
"use strict";

var elems = document.getElementsByClassName("myClass"), i;

for (i = 0; i < elems.length; i++) {
  elems[i].addEventListener("click", function () {
    this.innerHTML = i;
  });
}
//解释问题：如果按照第一种方法，所有的闭包的外层作用域对象都是全局对象，他们访问的是同一个i，而作用域链里的作用域对象的变量（i）是会被改变的。

//解决方法

"use strict";

var elems = document.getElementsByClassName("myClass"), i;
var clickhandler = function (val) {
    return function(){
    		this.innerHTML = val;
    	};
}
for (i = 0; i < elems.length; i++) {
  elems[i].addEventListener("click", clickhandler);
}
//解释原理：如果按照第二种方法，所有的闭包都指向了自己专属的外层作用域对象（相当于在全局对象和自己的作用域对象之间加了一个作用域对象），所以他们都有自己的i变量。这样问题就解决了。
```

以下是John Wu的原文，他是翻译的老外的文章，很值得读一读：

这篇文章告诉我：

* 可以这么说：JavaScript中所有对象都是闭包
* 闭包是什么时候创建的：定义一个函数的时候，就创建了一个闭包
* 闭包是什么时候被销毁的：当它不被任何其他的对象引用时，就可以被销毁了
* 作用域对象的创建时间点：当函数被调用的时候，就创建了一个函数作用域
* this:是函数调用时的上下文环境的引用

我研究JavaScript闭包（closure）已经有一段时间了。我之前只是学会了如何使用它们，而没有透彻地了解它们具体是如何运作的。那么，究竟什么是闭包？

Wikipedia给出的解释并没有太大的帮助。闭包是什么时候被创建的，什么时候被销毁的？具体的实现又是怎么样的？

```javascript
"use strict";

var myClosure = (function outerFunction() {

  var hidden = 1;

  return {
    inc: function innerFunction() {
      return hidden++;
    }
  };

}());

myClosure.inc();  // 返回 1
myClosure.inc();  // 返回 2
myClosure.inc();  // 返回 3

// 相信对JS熟悉的朋友都能很快理解这段代码
// 那么在这段代码运行的背后究竟发生了怎样的事情呢？
```
现在，我终于知道了答案，我感到很兴奋并且决定向大家解释这个答案。至少，我一定是不会忘记这个答案的。

>Tell me and I forget. Teach me and I remember. Involve me and I learn.
© Benjamin Franklin

并且，在我阅读与闭包相关的现存的资料时，我很努力地尝试着去在脑海中想想每个事物之间的联系：对象之间是如何引用的，对象之间的继承关系是什么，等等。我找不到关于这些负责关系的很好的图表，于是我决定自己画一些。

我将假设读者对JavaScript已经比较熟悉了，知道什么是全局对象，知道函数在JavaScript当中是“first-class objects”，等等。

### 作用域链（Scope Chain）

当JavaScript在运行的时候，它需要一些空间让它来存储本地变量（local variables）。我们将这些空间称为作用域对象（Scope object），有时候也称作LexicalEnvironment。例如，当你调用函数时，函数定义了一些本地变量，这些变量就被存储在一个作用域对象中。你可以将作用域函数想象成一个普通的JavaScript对象，但是有一个很大的区别就是你不能够直接在JavaScript当中直接获取这个对象。你只可以修改这个对象的属性，但是你不能够获取这个对象的引用。

作用域对象的概念使得JavaScript和C、C++非常不同。在C、C++中，本地变量被保存在栈（stack）中。在JavaScript中，作用域对象是在堆中被创建的（至少表现出来的行为是这样的），所以在函数返回后它们也还是能够被访问到而不被销毁。

正如你做想的，作用域对象是可以有父作用域对象（parent scope object）的。当代码试图访问一个变量的时候，解释器将在当前的作用域对象中查找这个属性。如果这个属性不存在，那么解释器就会在父作用域对象中查找这个属性。就这样，一直向父作用域对象查找，直到找到该属性或者再也没有父作用域对象。我们将这个查找变量的过程中所经过的作用域对象乘坐作用域链（Scope chain）。

在作用域链中查找变量的过程和原型继承（prototypal inheritance）有着非常相似之处。但是，非常不一样的地方在于，当你在原型链（prototype chain）中找不到一个属性的时候，并不会引发一个错误，而是会得到undefined。但是如果你试图访问一个作用域链中不存在的属性的话，你就会得到一个ReferenceError。

在作用域链的最顶层的元素就是全局对象（Global Object）了。运行在全局环境的JavaScript代码中，作用域链始终只含有一个元素，那就是全局对象。所以，当你在全局环境中定义变量的时候，它们就会被定义到全局对象中。当函数被调用的时候，作用域链就会包含多个作用域对象。

### 全局环境中运行的代码

好了，理论就说到这里。接下来我们来从实际的代码入手。

```javascript
// my_script.js
"use strict";

var foo = 1;
var bar = 2;
```

我们在全局环境中创建了两个变量。正如我刚才所说，此时的作用域对象就是全局对象。

![](/images/js_closure_1.png)

在全局环境中创建两个变量
在上面的代码中，我们有一个执行的上下文（myscript.js自身的代码），以及它所引用的作用域对象。全局对象里面还含有很多不同的属性，在这里我们就忽略掉了。

### 没有被嵌套的函数（Non-nested functions）

接下来，我们看这段代码

```javascript
"use strict";
var foo = 1;
var bar = 2;

function myFunc() {
  //-- define local-to-function variables
  var a = 1;
  var b = 2;
  var foo = 3;

  console.log("inside myFunc");
}

console.log("outside");

//-- and then, call it:
myFunc();
```

当myFunc被定义的时候，myFunc的标识符（identifier）就被加到了当前的作用域对象中（在这里就是全局对象），并且这个标识符所引用的是一个函数对象（function object）。函数对象中所包含的是函数的源代码以及其他的属性。其中一个我们所关心的属性就是内部属性[[scope]]。[[scope]]所指向的就是当前的作用域对象。也就是指的就是函数的标识符被创建的时候，我们所能够直接访问的那个作用域对象（在这里就是全局对象）。

>“直接访问”的意思就是，在当前作用域链中，该作用域对象处于最底层，没有子作用域对象。

所以，在console.log("outside")被运行之前，对象之间的关系是如下图所示。

![对象之间的关系](/images/js_closure_2.png)

温习一下。**myFunc所引用的函数对象其本身不仅仅含有函数的代码，并且还含有指向其被创建的时候的作用域对象。这一点非常重要！**

当myFunc函数被调用的时候，一个新的作用域对象被创建了。新的作用域对象中包含myFunc函数所定义的本地变量，以及其参数（arguments）。这个新的作用域对象的父作用域对象就是在运行myFunc时我们所能直接访问的那个作用域对象。

所以，当myFunc被执行的时候，对象之间的关系如下图所示。

![对象之间的关系（函数执行后）](/images/js_closure_3.png)


现在我们就拥有了一个作用域链。当我们试图在myFunc当中访问某些变量的时候，JavaScript会先在其能直接访问的作用域对象（这里就是myFunc() scope）当中查找这个属性。如果找不到，那么就在它的父作用域对象当中查找（在这里就是Global Object）。如果一直往上找，找到没有父作用域对象为止还没有找到的话，那么就会抛出一个`ReferenceError`。

例如，如果我们在myFunc中要访问a这个变量，那么在myFunc scope当中就可以找到它，得到值为1。

如果我们尝试访问foo，我们就会在myFunc() scope中得到3。只有在myFunc() scope里面找不到foo的时候，JavaScript才会往Global Object去查找。所以，这里我们不会访问到Global Object里面的foo。

如果我们尝试访问bar，我们在myFunc() scope当中找不到它，于是就会在Global Object当中查找，因此查找到2。

很重要的是，只要这些作用域对象依然被引用，它们就不会被垃圾回收器（garbage collector）销毁，我们就一直能访问它们。当然，当引用一个作用域对象的最后一个引用被解除的时候，并不代表垃圾回收器会立刻回收它，只是它现在可以被回收了。

所以，当myFunc()返回的时候，再也没有人引用myFunc() scope了。当垃圾回收结束后，对象之间的关系变成回了调用前的关系。

![对象之间的关系恢复](/images/js_closure_4.png)

接下来，为了图表直观起见，我将不再将函数对象画出来。但是，请永远记着，函数对象里面的[[scope]]属性，保存着该函数被定义的时候所能够直接访问的作用域对象。

### 嵌套的函数（Nested functions）

正如前面所说，当一个函数返回后，没有其他对象会保存对其的引用。所以，它就可能被垃圾回收器回收。但是如果我们在函数当中定义嵌套的函数并且返回，被调用函数的一方所存储呢？（如下面的代码）

```javascript
function myFunc() {
  return innerFunc() {
    // ...
  }
}

var innerFunc = myFunc();
```

你已经知道的是，函数对象中总是有一个[[scope]]属性，保存着该函数被定义的时候所能够直接访问的作用域对象。所以，当我们在定义嵌套的函数的时候，这个嵌套的函数的[[scope]]就会引用外围函数（Outer function）的当前作用域对象。

如果我们将这个嵌套函数返回，并被另外一个地方的标识符所引用的话，那么这个嵌套函数及其[[scope]]所引用的作用域对象就不会被垃圾回收所销毁。

```javascript
"use strict";

function createCounter(initial) {
  var counter = initial;

  function increment(value) {
    counter += value;
  }

  function get() {
    return counter;
  }

  return {
    increment: increment,
    get: get
  };
}

var myCounter = createCounter(100);

console.log(myCounter.get());   // 返回 100
myCounter.increment(5);
console.log(myCounter.get());   // 返回 105
```

当我们调用createCounter(100)的那一瞬间，对象之间的关系如下图

![调用createCounter(100)时对象间的关系](/images/js_closure_5.png)

注意increment和get函数都存有指向createCounter(100) scope的引用。如果createCounter(100)没有任何返回值，那么createCounter(100) scope不再被引用，于是就可以被垃圾回收。但是因为createCounter(100)实际上是有返回值的，并且返回值被存储在了myCounter中，所以对象之间的引用关系变成了如下图所示

调用createCounter(100)后对象间的关系
所以，createCounter(100)虽然已经返回了，但是它的作用域对象依然存在，可以且仅只能被嵌套的函数（increment和get）所访问。

让我们试着运行myCounter.get()。刚才说过，函数被调用的时候会创建一个新的作用域对象，并且该作用域对象的父作用域对象会是当前可以直接访问的作用域对象。所以，当myCounter.get()被调用时的一瞬间，对象之间的关系如下。

![调用myCounter.get()对象间的关系](/images/js_closure_6.png)

在myCounter.get()运行的过程中，作用域链最底层的对象就是get() scope，这是一个空对象。所以，当myCounter.get()访问counter变量时，JavaScript在get() scope中找不到这个属性，于是就向上到createCounter(100) scope当中查找。然后，myCounter.get()将这个值返回。

调用myCounter.increment(5)的时候，事情变得更有趣了，因为这个时候函数调用的时候传入了参数。

![调用myCounter.increment(5)对象间的关系](/images/js_closure_7.png)

正如你所见，increment(5)的调用创建了一个新的作用域对象，并且其中含有传入的参数value。当这个函数尝试访问value的时候，JavaScript立刻就能在当前的作用域对象找到它。然而，这个函数试图访问counter的时候，JavaScript无法在当前的作用域对象找到它，于是就会在其父作用域createCounter(100) scope中查找。

我们可以注意到，在createCounter函数之外，除了被返回的get和increment两个方法，没有其他的地方可以访问到value这个变量了。这就是用闭包实现“私有变量”的方法。

我们注意到initial变量也被存储在createCounter()所创建的作用域对象中，尽管它没有被用到。所以，我们实际上可以去掉var counter = initial;，将initial改名为counter。但是为了代码的可读性起见，我们保留原有的代码不做变化。

需要注意的是作用域链是不会被复制的。每次函数调用只会往作用域链下面新增一个作用域对象。所以，如果在函数调用的过程当中对作用域链中的任何一个作用域对象的变量进行修改的话，那么同时作用域链中也拥有该作用域对象的函数对象也是能够访问到这个变化后的变量的。

这也就是为什么下面这个大家都很熟悉的例子会不能产出我们想要的结果。

```javascript
"use strict";

var elems = document.getElementsByClassName("myClass"), i;

for (i = 0; i < elems.length; i++) {
  elems[i].addEventListener("click", function () {
    this.innerHTML = i;
  });
}
```

在上面的循环中创建了多个函数对象，所有的函数对象的[[scope]]都保存着对当前作用域对象的引用。而变量i正好就在当前作用域链中，所以循环每次对i的修改，对于每个函数对象都是能够看到的。

### “看起来一样的”函数，不一样的作用域对象

现在我们来看一个更有趣的例子。

```javascript
"use strict";

function createCounter(initial) {
  // ...
}

var myCounter1 = createCounter(100);
var myCounter2 = createCounter(200);
```

当myCounter1和myCounter2被创建后，对象之间的关系为

![myCounter1和myCounter2被创建后，对象之间的关系](/images/js_closure_8.png)

在上面的例子中，myCounter1.increment和myCounter2.increment的函数对象拥有着一样的代码以及一样的属性值（name，length等等），但是它们的[[scope]]指向的是不一样的作用域对象。

这才有了下面的结果

```javascript
var a, b;
a = myCounter1.get();   // a 等于 100
b = myCounter2.get();   // b 等于 200

myCounter1.increment(1);
myCounter1.increment(2);

myCounter2.increment(5);

a = myCounter1.get();   // a 等于 103
b = myCounter2.get();   // b 等于 205
```

### 作用域链和this

this的值不会被保存在作用域链中，this的值取决于函数被调用的时候的情景。

译者注：对这部分，译者自己曾经写过一篇更加详尽的文章，请参考《用自然语言的角度理解JavaScript中的this关键字》。原文的这一部分 以及“this在嵌套的函数中的使用”译者便不再翻译。

### 总结

让我们来回想我们在本文开头提到的一些问题。

* 什么是闭包？闭包就是同时含有对函数对象以及作用域对象引用的最想。实际上，所有JavaScript对象都是闭包。
* 闭包是什么时候被创建的？因为所有JavaScript对象都是闭包，因此，当你定义一个函数的时候，你就定义了一个闭包。
* 闭包是什么时候被销毁的？当它不被任何其他的对象引用的时候。

###专有名词翻译表

本文采用下面的专有名词翻译表，如有更好的翻译请告知，尤其是加*的翻译

*全局环境中运行的代码：top-level code

参数：arguments

作用域对象：Scope object

作用域链：Scope Chain

栈：stack

原型继承：prototypal inheritance

原型链：prototype chain

全局对象：Global Object

标识符：identifier

垃圾回收器：garbage collector

### 著作权声明

本文经授权翻译自[How do JavaScript closures work under the hood。](http://dmitryfrank.com/articles/js_closures?utm_source=javascriptweekly&utm_medium=email)

译者对原文进行了描述上的一些修改。但在没有特殊注明的情况下，译者表述的意思和原文保持一致。

原文地址：[http://blog.leapoahead.com/2015/09/15/js-closure/](http://blog.leapoahead.com/2015/09/15/js-closure/)