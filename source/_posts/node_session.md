title: express框架的session应用
date: 2013-09-10 01:15:49
tags: 
- nodejs
- session
categories: 后端框架
---

### 题记
很常用的一个场景：我们做用户权限验证的时候，势必要用到session。在node下，最好用的应该就是express-session了。以下记录下它的使用方法。

>express-session 是基于express框专门用于处理session的中间件。这里不谈express-session怎么安装，只给出相应的实例代码。另外，`session的认证机制离不开cookie`（原因见我的另一篇[session和cookie](http://blog.lwdle.xin/2013/02/20/session/)），需要同时使用cookieParser 中间件，有关的介绍可以专门参考https://github.com/expressjs/session/blob/master/README.md，或者参考http://blog.modulus.io/nodejs-and-express-sessions，这个博客上讲的比较清楚。
<!-- more -->
### 简单实例

```javascript
var express = require('express');
var session = require('express-session');
var cookieParser = require('cookie-parser');

var app = express();

app.use(cookieParser());
app.use(session({
    secret: '12345',
    name: 'testapp',   //这里的name值得是cookie的name，默认cookie的name是：connect.sid
    cookie: {maxAge: 80000 },  //设置maxAge是80000ms，即80s后session和相应的cookie失效过期
    resave: false,
    saveUninitialized: true,
}));
.....
```
### express-session中间件的使用：

* 只需要用express app的use方法将session挂载在‘/’路径即可，这样`所有的路由都可以访问到session	`，框架会对所有路由进行预处理，从http请求头中解析出session并塞到req.session中。
* 可以给要挂载的session传递不同的option参数，来控制session的不同特性。具体可以参见官网：https://github.com/expressjs/session/blob/master/README.md。

### session内容的存储和更改：

To store or access session data, simply use the request property req.session, which is (generally) serialized as JSON by the store, so nested objects are typically fine.

一旦我们将express-session中间件用use挂载后，我们可以很方便的通过req参数来存储和访问session对象的数据。`req.session是一个JSON格式的JavaScript对象`，我们可以在使用的过程中随意的增加成员，这些成员会自动的被保存到option参数指定的地方，`默认即为内存中去`。

### session的生命周期

　　`session与发送到客户端浏览器的生命周期是一致的`。而我们在挂载session的时候，通过option选项的`cookie.maxAge`成员，我们可以设置session的过期时间，以ms为单位（但是，如果session存储在mongodb中的话，任何低于60s(60000ms)的设置是没有用的，下文会有详细的解释）。如果maxAge不设置，默认为null，这样的expire的时间就是浏览器的关闭时间，即每次关闭浏览器的时候，session都会失效。
　
### session的存储方式
session有两种存储方式：

* 一种是存在内存中，这是默认的方式，生命周期段。
* 另一种是在options中制定一种持久化存储方式，express-session会自动把session信息持久化到持久化媒介中。适合特殊应用场景。
　　

有时候，我们需要session的声明周期要长一点，比如好多网站有个免密码两周内自动登录的功能。基于这个需求，session必须寻找内存之外的存储载体，数据库能提供完美的解决方案。这里，我选用的是mongodb数据库，作为一个NoSQL数据库，它的基础数据对象时database-collection-document 对象模型非常直观并易于理解，针对node.js 也提供了丰富的驱动和API。express框架提供了针对mongodb的中间件：connect-mongo，我们只需在挂载session的时候在options中传入mongodb的参数即可，程序运行的时候, express app 会自动的替我们管理session的存储，更新和删除。具体可以参考：

https://github.com/kcbanner/connect-mongo
　　
```javascript 
var express = require('express');
var session = require('express-session');
var cookieParser = require('cookie-parser');
var MongoStore = require('connect-mongo')(session);
var app = express();

app.use(cookieParser());
app.use(session({
    secret: '12345',
    name: 'testapp',
    cookie: {maxAge: 80000 },
    resave: false,
    saveUninitialized: true,
    store: new MongoStore({   //创建新的mongodb数据库
        host: 'localhost',    //数据库的地址，本机的话就是127.0.0.1，也可以是网络主机
        port: 27017,          //数据库的端口号
        db: 'test-app'        //数据库的名称。
    })
}));
```

跟session的内存存储一样，只需增加红色部分的store选项即可，app会自动替我们把session存入到mongodb数据，而非内存中。

### session的生命周期：

　　由于session是存在服务器端数据库的，所以的它的生命周期可以持久化，而不仅限于浏览器关闭的时间。`具体是由cookie.maxAge 决定`：如果maxAge设定是1个小时，那么从这个因浏览器访问服务器导致session创建开始后，session会一直保存在服务器端，即使浏览器关闭，session也会继续存在。如果此时服务器宕机，只要开机后数据库没发生不可逆转的破坏，maxAge时间没过期，那么session是可以继续保持的。当maxAge时间过期后，session会自动的数据库中移除，对应的还有浏览器的cookie。不过，由于connect-mongo的特殊机制（每1分钟检查一次过期session），session的移除可能在时间上会有一定的`滞后`。当然，由于cookie是由浏览器厂商实现的，cookie不具有跨浏览器的特性，例如，我用firefox浏览器在京东上购物时，勾选了2周内免密码输入，但是当我第一次用IE登陆京东时，同样要重新输入密码。所以，这对服务器的同一个操作，不同的浏览器发起的请求，会产生不同的session-cookie。