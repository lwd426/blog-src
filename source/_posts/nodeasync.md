title: 自己对node异步编程方案的理解
date: 2014-06-20 9:15:49
tags: 异步

---
### 题记
因为node是单线程，所以要用异步来提升性能。但编程一牵扯到“异步”两个字，那么什么时候调用回调函数就不是我们自己说了算的了。当我们进行顺序性特别强的多异步任务协作时，一切就会显得“不那么美好”了。所以我们要想办法更好更方便的进行异步编程。

### 三种异步编程方案
<!-- more -->
>朴灵写到：
目前，异步编程的主要解决方案有如下3种:
1. 事件发布/订阅模式
2. Promise/Defered模式
3. 流程控制库

#### 事件发布/订阅模式
其中事件发布模式我已经在我之前的博客中
[自己对node事件驱动的理解](http://blog.lwdle.xin/2015/09/15/nodeevent/#more "自己对node事件驱动的理解")中大致描述清楚了。算是比较原始的方式。
#### Promise模式
Promise/Defered模式很巧妙地使用这种事件订阅/发布模式抽象出了一套需要`预先定义好`的异步任务模型：`对异步调用的部分进行封装，暴露出自定义部分（成功执行函数和失败执行函数）`。这种模式需要封装，但很强大，有侵入性。各种Promise规范的实现库（Q、Bluebird等）也对`多个异步协作任务的异步调用部分也进行了很好的封装`，使用者可以十分方便的直接调用api，可读性也十分好，可以链式调用等，所以，个人推荐此种方法进行异步编程。
```javascript
var fs = require('fs')
var Promise = require('bluebird')
//改造fs.readFile为Promise版本
var readFileAsync = function(path){
	//返回一个Promise对象，初始状态pending
	return new Promise(function(fulfill, reject){
		fs.readFile(path,  'utf8', function(err, content){
			//由pending状态进入rejected状态
			if(err)return reject(err)
			//由pending状态进入fulfilled状态
			return fulfill(content)
		})
	})
}
//开始使用，调用其then方法，回调接受执行成功的返回值
readFileAsync('./promise-1.js').then(function(content){
	console.log(content)
});
```
#### 流程控制库
对于流程控制库，与Promise的思路完全不同，是另外一种解决方案。（Promise的重点在于`封装异步调用`的部分，流程控制库则没有显示的模式，将处理重点放到了`回调函数注入`（使用高阶函数注入）上。）它与事件订阅/发布模式基本无关，而是在回调函数上做文章，通过对所有异步操作数组里的结果进行”缓存“--最终向使用者暴露出总回调函数（封装了缓存的结果数组）来达到流程控制的目的。如果有一个异步任务失败，则都会把失败信息放到总回调中的第一个参数中。比较流行的有async库。这种方案更为灵活。
async示例：
```javascript
//异步的串行任务（异步的并行任务只需要把series方法换成parallel就ok了）
async.series([
        function (callback) {
          fs.readFile('file1.txt','utf-8',callback);
        },
        function(callback) {
          fs.readFile('file2.txt','utf-8',callback);
        }
      ],function(err, results){
        //results => [file1.txt,file2.txt]
        TODO
});
```