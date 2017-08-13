title: 自己对javascript异步IO和事件循环的理解
date: 2014-04-13 9:15:49
tags: 
- 事件驱动
- 异步
categories: javascript

---
### 题记
> 最近在了解javascript的异步IO和事件循环机制。最近有幸浏览了阮一峰老师的[JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)，感觉阮老师把一直困扰我的这个问题弄明白了，所以非常感谢阮老师。以下是我自己对javascript异步io和事件循环的一些理解和笔记。

相信我们在一开始接触javascript的时候都被各种贴子填鸭般的教导说“javascript是单线程的”。这没有错，javascript也只能是单线程，因为当初javascript被设计出来主要是为了操作DOM，设计成单线程，也就巧妙的避开了很多状态同步问题。有关状态的最近很火的一个库，大家可以看下`react`是怎么操作状态的，很巧妙。额，扯得有点远了，那么单线程的javascript是怎么做到`异步IO和并发`的呢？这多亏了`javascript的事件循环机制`。
<!-- more -->

### 实例代码
以下用代码的方式大概的说明下：

	<html>
	<head>
	<javascript src="jquery.1.7.0.min.js"></javascript>
	<javascript>
	 //1.ajax异步任务
    function getUserList(){
        $.get("/getusersinfo",{},function(error,response){
            ......
        });
    }
    
    $(function(){
        //2.定时任务
    	 setTimout(test(),200); 
		 function test() { 
		 	alert(1); 
		 } 
    	 //3.DOM注册click事件        
    	 $("#submit").click(function(){
            getUserList();
        });
    });
	</javascript>
	</head>
	<body>
 		<button id="submit"></button>
	</body>
	</html> 
 
### 来张图吧
下面这张图是我对事件循环的了解。
![](/images/javascript-eventloop1.png)

 1. 所有任务都在主线程上执行，形成一个`执行栈`。换句话说，我们写的所有js代码（函数声明、变量声明/赋值、注册监听等）都是一个个的代码块，跑在主线程上。
 2. 除了主线程，还有一个`任务队列`。系统通过调用各种接口完成了注册监听（给DOM元素，比如onclick、onload）、注册事件、调用ajax、设置定时任务、操作IO等）`把这些异步任务放到任务队列，执行栈继续向下执行后面的任务`。
 3. 在主线程执行栈和任务队列中间有一个很牛的东西叫“`事件循环`”。
 4. 一旦"执行栈"中的所有任务执行完毕，事件循环就会读取"任务队列"。如果这个时候，异步任务已经`结束了等待状态`，就会从"任务队列"`进入执行栈，恢复执行`。
 5. 主线程不断重复上面的第三步。

### 大神画的图
以上是我个人的一个理解，是给各个小白看的。对这个过程描述的而更为透彻是下图（转引自Philip Roberts的演讲《Help, I'm stuck in an event-loop》）

![](/images/javascript-eventloop2.png)

上图中，主线程运行的时候，产生堆（heap）和栈（stack），栈中的代码调用各种外部API，它们在"任务队列"中加入各种事件（click，load，done）。只要栈中的代码执行完毕，主线程就会去读取"任务队列"，依次执行那些事件所对应的回调函数。
### 结尾了
这是我对javascript的异步IO和事件循环的理解。接下来我还会了解下nodeJS的异步IO机制。
