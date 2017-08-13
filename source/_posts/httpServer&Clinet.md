title: HTTP服务的那些事---node对http模块的基础实现
date: 2015-03-22 10:09:10
tags:
- nodejs
- HTTP
categories: nodeJS

---
## 题记
>一直对node的服务器和客户端稀里糊涂的，只是会用，知道可以用`createServer(function( req, res){....}).listen(3000);`
的方式来建立一个服务器程序，监听客户端连接过来、数据（req）发送过来然后处理数据、然后通过res把数据返回给客户端。再者就是可以使用诸如connect或者express的这种对http封装过的框架，采用中间件的方式在服务器程序处理请求之前、之后对数据进行操作。就这么多了，再深入就不知道怎么回事了。

## 待解决问题
所以，这次特意来搞明白：

1. http的服务器和客户端到底张什么玩意？
2. connect对http模块进行了哪些封装？
3. express又对http模块进行了哪些封装？

今天就先来搞明白读一个问题！~
<!-- more -->
## http服务器和http客户端

简要的说：

* Node.js标准库提供了http模块，其中封装了一个高效的（高并发）http服务器和一个简易的客户端。
* http.Server是一个`基于事件的http服务器`
* http.Server核心由Node.js下层的C++部分实现的，而上层接口是由javascript封装，兼顾了高性能和简易性。
* http.request是一个客户端工具，用于向http服务器发起请求，如内容获取等。

从特性上说，http模块是：

* Node的http模块是对HTTP处理逻辑的封装（采用事件机制）。* Node中，HTTP服务(http模块)继承自TCP服务（net模块）。
* 是`基于事件驱动`的，继承自EventEmitter类，所以可以在node的单线程上做到高并发。
* HTTP服务以request为单位，而TCP服务以connecttion为单位进行服务，http模块是将connection到request的过程进行了封装.
 
### http服务器
http.Server是一个基于事件驱动的服务器，它把服务器相关的`逻辑都用事件进行封装`，用户只需要实现相关事件对应的响应函数，就可以完成服务器逻辑逻辑的处理，实现服务器的所有功能。



在使用过程中，最常用的就是当请求到来时，request封装出的request对象和response对象。他们两个分别是http.ServerRequest和http.ServerResponse的实例，表示请求和响应信息。

#### http服务器事件

http.Sever继承自EventEmitter，提供了以下几个事件，从而完成服务器的基本功能。

1. `request`:当客户端请求到来时，该事件被触发，提供两个参数req和res，分别是http.ServerRequest和http.ServerResponse的实例，表示请求和响应信息。
2. `connection`:当TCP连接建立时，该事件被触发，提供一个参数socket，为net.Socket的实例。connection事件的粒度要大于request，因为客户端在Keep-Alive模式下可能会在同一个连接内发送多次请求。
3. `close`：当服务器关闭时，该事件触发。注意不是在用户连接断开时。
4. 除此之外还有checkContinue、upgrade、clientError事件，通常我们不需要关心，只有实现复杂的HTTP服务器的时候才会用到。

在这些事件中，最常用的就是request了，因此http提供了一个捷径：
`http.createServer([requestListener])`，功能是创建一个HTTP服务器并将requestListener作为request事件的监听函数，这也是上面例子中的实现方式。

*注意:无论服务器端在处理业务逻辑时是否发生异常，务必在结束时调用res.end()结束请求，否则客户端将一直处于等待状态。当然，也可以通过延迟res.end()的方式实现客户端与服务端之间的长连接，但结束时务必关闭连接。*

下面是显式方式注册request事件：

```javascript
//httpserver.js
var http =require(‘http’);
var server = http.Server();
sever.on(‘request’,function(req,res){
   res.writeHead(200,{‘Content-Type’:’text/html’});
   res.write(‘<h1>Node.js</h1>’);
  res.end(‘<p>Hello World</p>’);
});
sever.listen(3000);
console.log(“HTTPserver is listening at port 3000.”);
```
>http请求对象和http响应对象是相对较底层的封装，现行的Web框架如Connect和Express都是这两个对象的基础之上进行高层封装完成的。

#### 请求对象http.ServerRequest

http.ServerRequest是HTTP请求的信息，是后端开发者最关注的内容。`它一般由http.Server的request事件发送，作为第一个参数传递，通常简称为request或req。`
属性有：

| 名称           | 含义           
| ------------- |:-------------:| 
| complete      | 客户端请求是否已经发送完成 | 
| httpVersion      | HTTP协议版本，通常是1.0或1.1      |
| method | HTTP请求方法，如GET POST PUT DELETE      |
| url | 原始请求路径     |
| headers | HTTP请求头      |
| trailers | HTTP请求尾（不常见）     |
| connection | 当前http连接套接字，为net.Socket的实例     |
| socket | connection属性的别名      |
| client | client属性的别名      |
  
HTTP请求一般可以分为两部分：请求头和请求体。以上内容由于长度较短都可以`在请求头解析完成后立即读取`。

而请求体可能相对较长，需要一定的时间传输，因此http.ServerRequest提供了以下3个事件用于控制请求体传输。

* `data`:当请求体数据到来时，该事件触发。该事件提供一个参数chunk，表示接收到的数据，如果该事件没有被监听，那么请求体将被抛弃。该事件可能会被调用多次。
* end:当请求体数据传输完成时，该事件被触发，此后将不会再有数据到来。
* close:用户当前请求结束时，该事件触发。不同于end，如果用户强制终止了传输，也还是调用close。

对请求参数的解析分为两种情况：get请求和post请求
##### 解析请求属性（get）
使用url模块的parse方法来解析get请求参数，实例如下：

```javascript
var http =require(‘http’);
var url = require(‘url’);
http.createServer(function(req,res){
   res.writeHead(200,{‘Content-Type’:’text/html’});
   res.end(url.parse(req.url,true));
}).listen(3000);

//parse后返回的对象包含
{
    search:’?name=aaa&key=…’,
    query:{name:’aaa’,key:’’}
    pathname:’问号前的如/user’
    path:’’,
    href:’’
}
```
##### 解析请求体(post)
HTTP协议1.1版本提供了8种标准的请求方法，其中最常见的就是GET和POST。`POST请求的内容全部都在请求体中`。http.ServerRequest并没有一个属性内容为请求体，原因是等待请求体传输可能是一件耗时的工作，譬如上传文件。而很多时候我们可能并不需要理会请求的内容，恶意的POST请求会消耗服务器的资源。所以Node.js默认是不会解析请求体的，当你需要的时候需要手动来做。

```javascript
//httpserverrequestpost.js
var http =require(‘http’);
var querystring =require(‘querystring’);
var util =require(‘util’);
http.createServer(function(req,res){
   var post = ‘’;
   req.on(‘data’,function(chunk){
         post+=chunk;
   });
   req.on(‘end’,function(){
            post = questring.parse(post);
            res.end(util.inspect(post));
   });
}).listen(3000);
```
通过querystring.parse将post解析为真正的POST请求格式.

#### 响应对象http.SeverResponse

http.ServerResponse是返回给客户端的信息，决定了用户最终能看到的结果。`它也是由http.Server的request事件发送的，作为第二个参数传递，一般简称为response或res。`

httpServerResponse有三个重要的成员函数，用于`返回响应头`、`响应内容`以及`结束请求`。

* `response.writeHead(statusCode,[headers]):向请求的客户端发送响应头`。statusCode是HTTP状态码，如：200，404。headers是一个类似关联数组的对象，表示响应头的每个属性。该函数在一个请求内最多只能调用一次，如果不调用，则会自动生成一个响应头。
* `response.write(data,[encoding])：向请求的客户端发送响应内容`。data是一个Buffer或字符串，表示要发送的内容。如果data是字符串，那么需要指定encoding来说明它的编码方式，默认是utf-8。在response.end调用之前，response.write可以被多次调用。
* `response.end([data],[encoding])：结束响应，告知客户端所有发送已经完成`。当所有返回的内容发送完毕的时候，该函数必须被调用一次。如果不调用该函数，客户端将永远处于等待状态。`注意高级框架express的很多方法比如res.json都已经包含了end方法，所以不需要再次end.`

### http客户端

>客户端是我们最常用的，就是要连接服务器前，要声明url、method、请求头、参数列表（get或post）用户名、密码等基本信息，然后用一个request连过去，就ok了。接下来就可以在回调函数（或事件中）接受服务器返回的结果，然后处理了。

http模块提供了两个函数http.request和http.get，功能是`作为客户端向HTTP服务器发起请求`。

#### http.request(options,callback)

http.request(options,callback)发起HTTP请求，接受两个参数，option是一个类似关联数组的对象，表示请求的参数，callback是请求的回调函数。

*  option常用参数如下:

	1. host:请求网站的域名或IP地址。
	2. port:请求网站的端口，默认80。
	3. method:请求方法，默认是GET
	4. path:请求相对根的路径，默认是“/”QueryString应该包含在其中。
	5. headers：一个关联数组对象，为请求头的内容
 
*  callback接收一个参数，为http.ClientResponse的实例

http.request返回一个http.ClientRequest的实例。如下通过http.request发送POST请求代码：

```javascript
//httprequest.js
var http =require(‘http’);
var querystring =require(‘querystring’);
var contents =querystring.stringify({
      name:’will’,
      email:’aa@qq.com’,
      adress:’fsfsdfsdfs’;
});
var options = {
      host:’www.byvoid.com’,
      path:’/application/node/post.php’,
      method:’POST’,
      headers:{
         ‘Content-Type’:’application/x-www.form-urlencoded’,
         ‘Content-Length:contents.length,
      }
};
var req =http.request(options,function(res){
      res.setEncoding(‘utf-8’);
      res.on(‘data’,function(data){
          console.log(data);
      });
});
req.write(contents);
req.end();
```

#### http.get(options,callback) 
http模块还提供了一个更加简便的方法便于处理GET请求：http.get。它是http.request的简化版，唯一区别在于http.get自动将请求方法设备成GET请求，同时不需要手动调用req.end();

```javascript
httpget.js
var http =require(‘http’);
http.get({host:’www.baidu.com’},function(res){
    res.setEncoding(‘utf8’);
    res.on(‘data’,function(data){
           console.log(data);
    });
});
```
#### 请求对象http.ClientRequest
http.ClientRequest是由http.request或http.get返回产生对象，表示`一个已经产生而且正在进行中的HTTP请求`。它提供一个response事件，即http.request或http.get第二个参数指定的回调函数的绑定对象。也可以显式地绑定这个事件的监听函数

```javascript
//httpresponse.js
var http =require(‘http’);
var req =http.get({host:’www.baidu.com’});
req.on(“response”,function(res){
     res.setEncoding(‘utf8’);
     res.on(‘data’,function(data){
             console.log(data);
     }
})
```
http.ClientRequest像http.ServerResponse一样也提供了write和end函数，用于向服务器发送请求体，通常用于POST、PUT等操作。所有写结束以后必须调用end函数以通知服务器，否则请求无效。
它还提供以下函数：

```javascript
request.abort():终止正在发送的请求
request.setTimeout(timeout,[callback])设置请求超时时间，
request.setNoDelay([noDelay])
request.setSocketKeepAlive([enable],[initialDelay])等函数
```
 
#### 响应对象http.ClientResponse
http.ClientResponse与http.ServerRequest相似，提供三个事件data,end,和close，分别在数据到达、传输结束和连接结束时触发，其中data事件传递一个参数chunk,表示接收到数据。
http.ClientResponse也提供了一些属性，用于表示请求的结果状态，

| 名称           | 含义           
| ------------- |:-------------:| 
| statusCode      | 200/404 | 
| httpVersion      | HTTP协议版本，通常是1.0或1.1      |
| headers | HTTP请求头      |
| trailers | HTTP请求尾（不常见）     |
 
还提供以下特殊函数：

```javascript
response.setEncoding([encoding]);当data事件被触发时，数据将会以encoding编码。默认值为null,即不编码，以Buffer形式存储。常用utf8
response.pause():暂停接收数据和发送事件，方便实现下载功能
response.resume():从暂停状态中恢复。
```