title: 自己对nodeJS的异步I/O模型的理解
date: 2014-07-10 01:15:49
tags:
- nodejs
- 异步
categories: nodeJS
---
### 题记
> 终于把node的异步IO给搞的稍微明白些了，之前看朴灵老师的《深入浅出NodeJS》还有些吃力，最近终于可以下定决心把它搞明白。以下是我参阅朴灵老师的《深入浅出NodeJS》后，自己对node异步io的理解，画了一张图，记录一下。
<!-- more -->

### 大神给了张图
这张是朴灵老师在他的Infoq中画的一张图：

![](/images/nodeio2.png)

### 事件循环和异步调用

朴灵老师对事件循环的描述如下：
> 在调用uv_fs_open方法的过程中实际上应用到了事件循环。以在Windows平台下的实现中，启动Node.js时，便创建了一个基于IOCP的事件循环loop，并一直处于执行状态。

`uv_run(uv_default_loop());`

> 每次循环中，它会调用IOCP相关的*GetQueuedCompletionStatus*方法检查是否线程池中有执行完的请求，如果存在，poll操作会将请求对象加入到loop的pending_reqs_tail属性上。 另一边这个循环也会不断检查loop对象上的pending_reqs_tail引用，如果有可用的请求对象，就取出请求对象的result属性作为结果传递给oncomplete_sym执行，以此达到调用JavaScript中传入的回调函数的目的。 至此，整个异步I/O的流程完成结束。其流程如下：

![事件循环和异步调用](/images/nodeio3.png)

> `事件循环`和`请求对象`构成了Node.js的异步I/O模型的两个基本元素，这也是典型的消费者生产者场景。在Windows下通过IOCP的GetQueuedCompletionStatus、PostQueuedCompletionStatus、QueueUserWorkItem方法与事件循环实。对于*nix平台下，这个流程的不同之处在与实现这些功能的方法是由libeio和libev提供。

### 以下是我个人的理解：
   
![NodeJS异步IO和事件循环图解](/images/nodeio.png)

#### 参与者们
完成NodeJS的整个异步I/O环节的有事件循环、观察者（队列）、IOCP和请求对象等。
	
 1. `事件循环`：nodejs自身的执行模型--事件循环。它使在node启动时创建的，每执行一次循环体称做Tick.每个Tick的过程就是`看观察者队列里是否有事件要处理`。
 2. `观察者（队列）`：在Node中，事件主要来源于网络请求、文件I/O等，所以`Node用观察者把事件分类`，这些事件对应的观察者有网络请求观察者、文件I/O观察者等，当线程池中的请求对象执行完毕，就会把执行结果req-->result后，调用`PostQueueCompletionStatus()`通知IOCP该请求对象执行完毕。
 3. `IOCP`:事件观察者会在每次Tick的过程中，都会`调用GetQueueCompletionStatus()询问IOCP线程池中是否有执行完的请求`。如果有，则把它加入到对应的`观察者队列`中，等待被事件循环执行。
 4. `请求对象`：它可以理解为node从js发起异步调用到内核执行完异步操作的一个`中间载体`。它在libuv层被组装，组装的时候把各种参数、回调函数、执行状态都封装了进去。当请求对象执行完毕后，会把执行结果封装到他的result中。最终由`事件循环调用回调函数（result）`.

### 使用异步IO构建web服务器
异步I/O不仅仅应用在文件操作中，对于网络套接字，node也用到了I/O。以下描述`node构建web服务器的原理`：

1. 开始
2. 监听端口（套接字）
3. 事件循环会不断的处理这些网络I/O事件：
	- 接收到网络请求
	- 把请求对象发给I/O观察者形成事件
	- 请求对象进入到事件循环
	- 执行I/O观察者中事件对象的回调函数
	- 判断是否有业务层逻辑的回调函数：如果是，则执行回调，如果没有回调，则结束

以上是我在阅读《深入浅出NodeJS》后，查询很多资料后的一些个人理解，其中肯定肯定有理解上的不足，姑且记录下，慢慢理解提升。


参考：

* 《nodejs异步IO的实现》[http://cnodejs.org/blog/?p=244](http://cnodejs.org/blog/?p=244)
* 《深入浅出Node.js（五）：初探Node.js的异步I/O实现》[http://www.infoq.com/cn/articles/nodejs-asynchronous-io](http://www.infoq.com/cn/articles/nodejs-asynchronous-io)
