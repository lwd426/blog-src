title: Node.js核心模块-events
date: 2014-11-03 14:34:00
tags:
- nodejs
categories: nodejs

---

>events是Node.js最重要的模块，没有“之一”，原因是Node.js本身架构就是事件式的，而它提供唯一 的接口，所以堪称Node.js事件编程的基石。events模块不仅用于用户代码与Node.js下层事件循环的交互，还几乎被所有的模块依赖。

### 事件发射

events模块只提供一个对象：events.EventEmitter。EventEmitter的核心就是事件发射与事件监听器功能的封装。EventEmitter的每个事件由一个事件名和若干参数组成，事件名是一个字符串，通常表达一定的语义。对于每个事件，EventEmitter支持若干个事件监听器。当事件发射时，注册到这个事件的事件监听器被依次调用，事件参数作为回调函数参数传递。
<!-- more -->
让我们以下面的例子解释这个过程：

```javascript
var events =require(‘events’);
var emitter = newevents.EventEmitter();
emmitter.on(“someEvent”,function(arg1,arg2){
     console.log(“listener1”,arg1,arg2);
});
emmitter.on(“someEvent”,function(arg1,arg2){
     console.log(“listener2”,arg1,arg2);
});
 
emmiter.emit(“someEvent”,’byVoid’,1991);
//运行的结果是：
listener1 byVoid1991
listener2 byVoid1991
```

emitter为事件someEvent注册了两个事件监听器，然后发射了someEvent事件，运行结果中可以看到两个事件监听回调函数被先后调用。

### EventEmitter的简单API

* `EventEmitter.on(event,listener)` :接受一个字符串和一个回调函数listener回调函数
* `EventEmitter.emit(evnet,[arg1],[…])`发射event事件，传递若干可选参数到事件监听的参数表
* `EventEmitter.once(event,listener);`指定事件注册一个单次监听器，即监听器最多只会触发一次，触发后立即解除该监听器。
* `EventEmitter.removeListener(event,listener);`移除指定事件的某个监听器，listener必须是该事件已经注册过的监听器
* `EventEmitter.removeAllListeners([evnet]);`移除所有事件的所有监听器。
* API:http://nodejs.org/api/events.html
 
### error事件
我们在遇到异常的时候通常会发射error事件。当error被发射时，EventEmitter规定如果没有响应的监听器，Node.js会把它当作异常，退出程序并打印调用栈。**我们一般要为会发射error事件的对象设备监听器，避免遇到错误后整个程序崩溃**：
var events =require(“events”);
var emitter = newevent.EventEmitter();
emitter.emit(“error”);
 
运行时会显示以下错误：
node.js:201
         throw e; //process.nextTick error, or ‘error’event on first tick
Error:Uncaught,unspecified‘error’ event.
        at EventEmitter.emit(events.js 50:15);
         …