title: 思考：转-NodeJs 多核多进程并行框架实作
date: 2015-04-05 10:24:00
tags:
- nodejs
- 集群
- 多进程
categories: nodejs

---

>多核编程的重要性无需多说， 我们直奔主题，目前nodejs 的网络服务器有以下几种支持多进程的方式：



### 开启多个进程，每个进程绑定不同的端口
开启多个进程，每个进程绑定不同的端口，用反向代理服务器如 Nginx 做负载均衡，好处是我们可以借助强大的 Nginx 做一些过滤检查之类的操作，同时能够实现比较好的均衡策略，但坏处也是显而易见 —我们引入了一个间接层。

<!-- more -->

### 多进程绑定在同一个端口侦听

在nodejs 中，提供了进程间发送“文件句柄” 的功能，这个功能实在是太有用了（貌似是yahoo 的工程师提交的一个patch）

不明真相的群众可以看这里： [http://www.lst.de/~okir/blackhats/node121.html](http://www.lst.de/~okir/blackhats/node121.html)

在 node 中我们可以通过以下函数达到效果：

```javascript
stream.write(string, encoding=’utf8’, [fd]）
```
或在 node v0.5.9+ 中 fork 子进程之后： 

```javascript
child.send(message, [sendHandle])
```

所以我们设计以下方案：master 进程生成了listen 端口之后，发送这个 listenfd 给所有的worker 子进程，worker 子进程接收到handle 之后，执行listen 操作： 

```javascript
master ： 
function startWorker(handle){

  output(“start workers :” + WORKER_NUMBER);

  worker_succ_count = 0;

  for(var i=0; i<WORKER_NUMBER; i++){

    var c  =  cp.fork(WORKER_PATH);

    c.send({"server" : true}, handle);

  }

}

function startServer(){

  var tcpServer = net.createServer();

  tcpServer.on(“error", function(err){

    output(“server error ,check the port…”);

    about_exit();

  })


  tcpServer.listen(PORT , function(){

    startWorker(tcpServer._handle);

    tcpServer.close();

  });

}

startServer();
```
注意，因为我们只需要一个handle ，httpServer 其实是netServer 的一层封装，所以我们在master进程启动netServer ，发送这个listen套接字 “handle” 到各个子进程 

```javascript
worker ：
server = http.createServer(function(req, res){

  var i,r;

  for(i=0; i<10000; i++){

    r = Math.random();

  }


  res.writeHead(200 ,{"content-type” : “text/html"});

  res.end(“hello,world”);

  child_req_count++;

});

process.on(“message",function(m ,handle){

  if(handle){

    server.listen(handle, function(err){

      if(err){

        output(“worker listen error”);

      }else{

        process.send({"listenOK” : true});

        output(“worker listen ok”);

      }


   });

  …

 });
```

worker 进程收到handle后，立即进行listen ，这样就会有多个worker进程 listen同一个socket端口，即同一个套接字被加入到多个进程的epoll 监控结构中，当一个外部连接到来时，此时只有一个幸运的worker 进程得到激活事件，接收这个连接。（在UNP 中讲到这种情况下会导致 “惊群” 效应，但据江湖传闻2.6以上的Linux 系统中，阻塞式的listenfd 已消除惊群现象，非阻塞的listenfd 依然存在，即我们的epoll还是会存在这个问题的，但个人认为nodejs 的epoll 结构中往往有很多的监控句柄而非仅listenfd，所以这时候惊群造成的影响应该是比较小的…） 

我们开5个worker 测试 （以下测试均为开启keep-alive模式，本机测试）： 

测试业务如上代码所示：运行10K次 Math.random(), 然后输出 ”hello,world“;
 
```javascript
系统配置： 
Linux 2.6.18-164.el5xen x86_64 

CPU X5 ，Intel® Xeon® CPU E5620 @ 2.40GHz 

free -m 
total used free shared buffers cached 
Mem: 7500 3672 3827 0 863 1183 

siege -c 100 -r 1000 -b localhost:3458/ 

结果为： 

ransactions: 100000 hits 
Availability: 100.00 % 
Elapsed time: 10.95 secs 
Data transferred: 1.05 MB 
Response time: 0.01 secs 
Transaction rate: 9132.42 trans/sec 
Throughput: 0.10 MB/sec 
Concurrency: 55.61 
Successful transactions: 100000


5 个worker 处理的请求量分别是： 
child req total : 23000 
child req total : 16000 
child req total : 17000 
child req total : 22000 
child req total : 22000


再测一次： 
child req total : 13000 
child req total : 30000 
child req total : 14000 
child req total : 22000 
child req total : 21000
```

在这种情况下，我们的负载均衡是建立在各个worker“随机接收“的特征基础上的，由操作系统来保证的，长期运行情况下应该是均衡的，但短期内还是会有可能导致负载倾斜的现象，特别是在客户端使用keep-alive连接并长期不关闭的情况下。 


### 一个进程负责监听、接收连接，然后把接收到的连接平均发送到子进程中去处理 

我们先看一下正常情况下一个http server 服务的流程 ，其大体可分为几 个阶段:

```javascript
listenfd 绑定侦听 -> 接收到的Tcp 连接对象 → 包装成socket 对象 → 生成(req ,res)对象 -> 调用用户代码 
```

```javascript
(#1) TCP.bind — > TCP.listen (process.binding(“tcp_wrap”)) 
                                                | 
                                                |TCP.emit(“connection” ,handle) 
                                                |

(#2) Wrap TCP handle to Socket (Tcp.onconnection) 
                                                | 
                                                |Net.Server.emit(”connection” , socket) 
                                                | 
(#3)Create Req ,Res based on a Socket(net.server.connectionListen) 
                                                | 
                                                |Http.server.emit(“request” ,req ,res) 
                                                | 
(#4)your code writen here ：function(req ,res){ 
    res.writeHead(200 ,“content-type/text/html”); 
    res.end(“hello,world”) 
}
```
nodejs 的child.send(message, [sendHandle]) 函数 ，此处的 sendHandle 这时应该为一个 tcp_wrap 对象，所以我们不能直接使用 net.createServer 返回给我们的socket ，否则的话我们需要”回滚“ 从Tcp 到 Socket 这一步骤，不仅浪费资源，同时也是不安全的，所以我们在tcpMaster 中 直接使用 tcp_wrap : 

```javascript
var TCP = process.binding(“tcp_wrap”).TCP;

var childs = [];

var last_child_pos = 0;

function startWorker(){

  for(var i=0; i<WORKER_NUMBER; i++){

    var c  =  cp.fork(WORKER_PATH);

    childs.push©;

  }

}

function startServer(){

    server = new TCP();

    server.bind(ADDRESS, PORT);

    server.onconnection = onconnection;

    server.listen(BACK_LOG);

  }

function onconnection(handle){

    //output(“master on connection”);

    last_child_pos++;

    if(last_child_pos >= WORKER_NUMBER){

      last_child_pos = 0;

    }

    childs[last_child_pos].send({"handle” : true}, handle);

    handle.close();

}

startServer();

startWorker();

```
以上为tcpMaster 进程把接收的tcp 连接 均匀分配给 tcpWorkers ： 

```javascript
function onhandle(self, handle){

    if(self.maxConnections && self.connections >= self.maxConnections){

      handle.close();

      return;

    }

    var socket = new net.Socket({

      handle : handle,

      allowHalfOpen : self.allowHalfOpen

    });

    socket.readable = socket.writable = true;

    socket.resume();

    self.connections++;

    socket.server = self;

    self.emit(“connection", socket);

    socket.emit(“connect”);

  }

server = http.createServer(function(req, res){

    var r, i;

      for(i=0; i<10000; i++){

        r = Math.random();

      }


      res.writeHead(200 ,{"content-type” : "text/html"});

      res.end(“hello,world”);

      child_req_count++;

    });

}

 process.on(“message",function(m ,handle){

     if(handle){

        onhandle(server, handle);

     }


    if(m.status == “update”){

      process.send({"status” : process.memoryUsage()});

    }


  }); 
```

以上为tcpWorker 将接收到的tcp handle 封装成socket ，为了充分的与http.server类兼容，我们还对connections的数量进行检查，并把socket.server 设为当前的server ，然后激发http.server 的 ”connection“ 事件. 

通过这种方式，我们用尽量小的开销，在充分保证http.server 类的兼容性的前提下，用尽量少而优雅的代码实现了负载均衡与高效并行。 
测试结果如下： 
```javascript
ransactions: 100000 hits 
Availability: 100.00 % 
Elapsed time: 10.47 secs 
Data transferred: 1.05 MB 
Response time: 0.01 secs 
Transaction rate: 9551.10 trans/sec 
Throughput: 0.10 MB/sec 
Concurrency: 60.68 
Successful transactions: 100000

child req total : 20000 
child req total : 20000 
child req total : 20000 
child req total : 20000 
child req total : 20000
```
数据会有所起伏，qps 总体在 8000~11000 范围内，注意以上worker 数目均设为5个，适量增大worker数目，qps 可以稳定达到10k，但这时系统load比较高，使用时需谨慎选择。 

几次测试完成后，我们查看/proc/[tcpMaster]/fd , 其占用的端口如下：

```javascript 
0 -> /dev/pts/30 
1 -> /dev/pts/30 
10 -> socket:[71040] 
11 -> socket:[71044] 
12 -> socket:[71054] 
2 -> /dev/pts/30 
3 -> eventpoll:[71027] 
4 -> pipe:[71028] 
5 -> pipe:[71028] 
6 -> socket:[71030] 
8 -> socket:[71032] 
9 -> socket:[71036]

查看其中一个tcpWorker： 
0 -> socket:[71031] 
1 -> /dev/pts/30 
2 -> /dev/pts/30 
3 -> eventpoll:[71049] 
4 -> pipe:[71050] 
5 -> pipe:[71050]
```
tcpMaster 的fds 意义分别如下： 

1个socket为listenfd

5个socket 用作父子进程通信

2个pipe（一对）用于asyn_watcher/signal_watcher 的触发

剩余的不解释…



tcpWorker 的fds 意义分别如下： 

1个socket（这儿就是stdin）用作与父进程通信

其余fd与master中fd作用类似



所以tcpMaster/tcpWorker 端口占用正常，没有句柄泄露问题，负载均衡可控，但负责接收socket的master需要重新分配发送socket ，引入了额外的开销. 

小结： 

本文介绍了2种比较高效的多进程运行方式，两种方式各有优劣，需要使用者自行选择，在node v0.5.10+ 中，内置了cluster 库，不过在我看来，其宣传意义大于实用意义，因为这样官方就可理直气壮的宣称直接支持多进程运行方式，nodejs 官方为了让API 接口傻瓜化，用了一些比较trick 的方法，代码也比较绕，且这种多进程的方式，不可避免的要牵涉到进程通信、进程管理之类的东西，但我们往往有自己的需求，现在nodejs官方把它固化到lib中，我们就无法自由的更改添加一些功能。 

此外，有两个node 的module ，multi-node 和 cluster ，采用的策略和本文介绍的类似，但使用这些module往往有一些缺点： 

更新不及时

复杂庞大，往往绑定了很多其他的功能，用户往往被绑架

遇到问题难以hack




基于本文的介绍，你可以很方便的打造自己的高性能的、易维护的、最简的、优雅实用的cluster ，enjoy it！ 

源码地址：https://github.com/windyrobin/iCluster 

原文地址：https://cnodejs.org/topic/4f16442ccae1f4aa27001081