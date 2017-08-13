title: 玩转nodejs多进程
date: 2015-04-03 14:34:00
tags:
- nodejs
- 集群
- 多进程
categories: nodejs

---

>nodejs的单线程单进程的模型避免了线程之间的状态同步或者状态锁之类的问题，但是这也使node饱受诟病，比如一个node进程只能利用一个核，不能利用现在越来越普遍的多核CPU；单进程一旦有异常没有被捕获则会让整个应用死掉，稳定性十分差。所以引用《nodejs深入浅出》问题抛出：
>
* 如何充分利用多核cpu服务器？
* 如何保证进程的健壮性和稳定性？

以下是自己总结的《nodejs深入浅出》第9章-玩转进程的阅读笔记：
<!-- more -->
### 服务模型的变迁
服务器的主要性能指标就是`服务器处理客户端请求的并发量`。
服务器的模型经历了以下几个阶段：

1. 同步：服务器服务多个请求的`数据同步问题`。最早的服务器的执行模型是同步的，其一次只为一个请求服务，其余请求都处于耽误的状态。这类架构如今已基本淘汰，只在一些无并发要求的应用中存在。
2. 复制进程：为了解决同步架构的并发问题，一个简单的改进是`通过进程的复制来同时服务更多的请求`。每有一个连接，就复制一个进程来提供服务。这个模型不具备伸缩性，一旦并发请求过高，内存会随着进程数的增长耗尽。
3. 多线程：类似多进程模式，为了解决进程复制过程中的浪费问题，`多线程被引入`，对每一个连接都创建一个线程去服务。线程相对进程开销要小很多，而且线程间可以共享数据。但是多线程还是会随着并发数的增多而耗尽内存，缺乏强大的伸缩性。
4. 事件驱动：单线程的事件驱动避免了不必要的内存开销和上下文切换开销，不受资源上限的影响，伸缩性远比前两者高。

### 多进程架构
理想状态下，每个进程各自利用一个cpu,以实现多核cpu的有效利用。Node提供了child_process模块，使用`child_process.fork()`函数供我们实现进行的复制。

* node采用Master-Worker模式（又称主从模式）。把进程分为两种：`主进程和工作进程`。这种分布式架构采用并行的方式处理业务的模式，具备较好的伸缩性和稳定性。
* node通过child_process.fork()复制的进程都是`独立的进程`，这些进程都有着独立而全新的V8实例。（*fork()进程是昂贵的*）
* node启动多个进程就是为了`充分利用cpu资源`，而不是为了解决并发问题，并发问题是通过事件驱动方式在单线程上解决的。

*注意：复制出的进程都是“独立的进程”，所以不能同时监听同一个端口（socket）,node采用`传递句柄`的方式解决了这个问题，让多个进程监听同一个端口，加上主线程的负载均衡，可以让node性能很棒。*

#### 创建子进程的方法
 
 面对单进程单线程对多核利用不足的问题，前人的经验是启动多个进程即可，理想状态下每个进程各自利用一个CPU。child_process模块赋予了Node可以随意创建子进程（child_process）的能力，他提供了4个方法用于创建子进程：
 
 * spawn()：其他三个方法的基础
 * exec()
 * execFile()
 * fork()
 
#### 进程间的通信
 
 1. 子进程对象使用send()方法实现主进程向子进程发送数据
 2. message()事件实现收听子进程发送来的数据
 3. 进程间通过`消息`传递内容，而不是共享或者直接操作资源
 4. 消息通过`IPC（进程间通信）`通道传递消息（send（）和message()）
 5. 进程间通信的目的是为了让不同的进程能够互相访问资源并进行协调工作。
 6. 在node中，IPC通道被抽象为`Stream对象`，在调用send()时发送数据，接收到的消息会通过message事件触发给应用层。
 
 *机制描述：*父进程在创建子进程之前，会创建IPC通道并监听它，然后才真正创建子进程，并通过环境变量NODE_CHANNEL_FD告诉子进程这个IPC通道的文件描述符。子进程在启动的过程中，根据文件描述符去连接这个IPC通道，从而完成父子进程之间的链接。

#### 句柄传递

应为node的child_process模块fork()复制出的进程都是独立，不能同时监听同一个端口，但我们想让`多个进程监听同一个端口`，从而可以让多个进程同时服务请求。所以Node在v5.0.9版本中引入了`进程间发送句柄`的功能。

1. 进程间通信的send()方法在发送数据的同时，也可以通过第二个参数发送句柄。

  ```javascript
  child.send(message, [sendHandle]);
  ```

2. 句柄：是一种可以用来标识资源的引用，它的内部包含了指向对象的文件描述符。比如句柄可以标识一个服务器socket对象，一个客户端socket对象，一个UPD套接字或一个管道等。
3. node的child_process模块的主进程接收到socket请求后，将这个socket直接发送给工作进程，从而让各个独立的进程能够监听同一个端口。

##### 句柄传递机制

1. 发送：代码显示sever是把服务器socket对象send给了工作进程，但实际上node发送到IPC管道中的实际是我们要发送的`句柄的文件描述符`，最终发送到通道里的信息都是`字符串`（*注意：send()方法只能发送消息和句柄*）
2. 接收：连接了IPC管道的子进程在`读到了父进程发来的消息`（句柄的文件描述符的字符串）后，`解析还原为对象`后，`触发message事件`将消息体作为参数传给回调函数（即应用层）使用。
3. node进程之间只有消息传递，不会真正的传递对象。

##### 为什么多个进程可以监听同一个端口而不报EADDRINUSE错误？
1. EADDRINUSE错误原因：独立启动的进程中，`TCP服务器端socket套接字的文件描述符并不相同`，导致监听到相同的端口会抛出异常。
2. node进行了特殊设置，可以让不同的进程监听相同的网卡和端口。
3. 当我们通过send()发送句柄后，工作进程从IPC中还原出句柄服务后，他们的文件描述符是相同的，所以监听相同的端口不会引发异常。
4. 多个进程监听同一端口时，文件描述符同一时间只能被某个进程所用，所以这些进程的服务是`抢占式`的。

### 代码演示演变的过程

#### 1. 多进程架构:主从模式

```javascript
//worker.js
var http = require('http');
http.createServer(function(req, res){
	res.writeHead('Content-Type','text/plain');
	res.end('Hello World\n');
}).listen(Math.round(1 + Math.random()) * 1000, 127.0.0.1);

//master.js
var fork = require('child_process').fork;
var cpus = require('os').cpus();
for(var i=0;i<cpus.length;i++){
	fork('./worker.js');
}
```

#### 2. 进程间通信:主进程和工作进程之间可以通过send和messge配合发送接收消息

```javascript
//parent.js
var cp = require('child_process');
var n = cp.fork(__dirname + 'sub.js');

n.on('message', function(m){
	console.log('parent got a message:' + m);
});

n.send({hello : 'world'});

//sub.js
process.on('message', function(m){
	console.log('child get a message:' + m);
});

process.send({foo ;: 'bar'});
```
*注意：process是node的全局变量，是用于描述当前node进程状态的对象，可以理解为当前的node进程实例。
*

#### 3. 句柄传递：主进程可以通过send()发送句柄给工作进程，从而主进程和工作进程可以同时服务请求

```javascript
//parent.js
var cp = require('child_process');
var child1 = cp.fork('sub.js');
var child2 = cp.fork('sub.js');

//打开一个服务器对象，发送句柄
var server = require('net').createServer();
server.on('connection', function(socket){
	socket.end('handle by parent\n');
});
server.listen(1377, function(){
	child1.send('server', server);
	child2.send('server', server);
});

//sub.js
process.on('message', function(m, server){
	if(m === 'server'){
		server.on('connection', function(socket){
			socket.end('handle by sub, pid is ' + process.id + '\n');	
		});
	}
});
```

#### 4. 主进程更轻量，让工作进程处理请求：关闭服务器的监听，只让工作进程监听端口处理请求

```javascript
//parent.js
var cp = require('child_process');
var child1 = cp.fork('sub.js');
var child2 = cp.fork('sub.js');

//打开一个服务器对象，发送句柄
var server = require('net').createServer();
//server.on('connection', function(socket){
//	socket.end('handle by parent\n');
//});
server.listen(1377, function(){
	child1.send('server', server);
	child2.send('server', server);
	//关闭
	sever.close();
});

//sub.js
var http = require('http');
http.createServer(function(req, res){
	res.writeHead('Content-Type','text/plain');
	res.end('Hello World\n');
});

process.on('message', function(m, tcp){
	if(m === 'server'){
		//给当前启动的工作进程服务器注册connection事件，当来请求时，把socket对象作为参数触发主进程服务器的connection事件
		tcp.on('connection', function(socket){
			server.emit('connection', socket);
		});
	}
});
```
[另一个人对本章做的笔记，觉得比我做的好，拜读下](http://raytaylorlin.com/Tech/web/Node.js/node-process-and-cluster/)