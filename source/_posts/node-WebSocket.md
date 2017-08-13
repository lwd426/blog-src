title: 自己对WebSocket+nodeJS的理解
date: 2014-07-20 19:15:49
tags: 
- nodejs
- WebSocket
categories: nodeJS

---

### 题记
>WebSocket最初是作为html5的一个重要特性出现的，但最终在w3c和IETF的推动下，形成了标准，目前，大部分浏览器都得以支持。

### WebSocket的优势
    
* 服务端和客户端只要建立一次TCP连接就可以完成双向通信，可以使用`更少的连接`。
* WebSocket服务端可以`推送数据`到客户端，不需要传统的长连接或者轮循的方式。这也比HTTP的请求/响应模式更加灵活、更加高效。
* 协议头更轻量，`减少数据传送量`。

<!-- more -->

### node跟ws更配哦
nodeJS是事件驱动的、非阻塞的。所以，采用nodeJS实现WebSocket可以更好的发挥nodeJS和WebSocket的优势。原因如下：

* WebScocket客户端基于`事件`的编程模型与Node中的自定义事件（事件驱动）十分相似。
* Node的`事件驱动`的方式使得它特别适合应对WebScocket这类`长连接`的应用场景，可以轻松的处理`高并发`的请求。

WebScocket协议主要分为两个部分：`握手`和`数据传输`。

### WebSocket讲解
#### 握手
客户端建立连接时，通过HTTP发起请求报文，如下：

```javascript
GET /xxx HTTP/1.1  
Host: sever.example.com 
Upgrade: websocket 请求服务器更新协议为websocket
Connection: Upgrade
Sec-WebScocket-key: xxxxxxxxx 安全校验
Sec-WebScocket-Protocol: chat, superchat 子协议
Sec-WebScocket-Version: 13 版本
```
服务端处理完请求后，响应报文如下：
```javascript
HTTP/1.1 101 Switching Protocols  
Upgrade: websocket
Connection: Upgrade
Sec-WebScocket-key: dddddddddd 安全校验
Sec-WebScocket-Protocol: chat, superchat 子协议
```
上面的报文告诉客户端，正在更新协议，更新应用层协议为WebSocket，其中，dddddddd为服务端基于客户端发过来的xxxxxxxxx生成的安全校验字符串。客户端将会校验ddddddddd,如果成功则可以进行数据传输。一旦握手成功，服务器端和客户端将会呈现对等的效果，都能接收（onmessage）和发送(send)信息。

真实请求和响应报文如下图

![](/images/nodewebsocket.png)

#### 数据传输
当握手成功，当前连接将不再进行HTTP的交互，而是开始WebSocket的数据协议传输数据。`需要注意的是所有的通信数据都是以”\x00″开头以”\xFF”结尾的，并且都是UTF-8编码的。`

客户端通过websocket API提供的如下4个事件进行编程：

* onopen 建立连接后触发
* onmessage 收到消息后触发
* onerror 发生错误时触发
* onclose 关闭连接时触发

#### 工具-Socket.io
以下用Socket.io来实例Websocket在Node的使用

Socket.IO是一个开源的WebSocket库，它实现了WebSocket协议，并对WebSocket的4个事件进行了封装。它通过Node.js实现WebSocket服务端，同时也提供客户端JS库。Socket.IO支持以事件为基础的实时双向通讯，它可以工作在任何平台、浏览器或移动设备。

npm socket.io后，服务端的代码如下：

```javascript
var app = require('express')();
var http = require('http').Server(app);
var io = require('socket.io')(http);
 
app.get('/', function(req, res){
    res.send('<h1>Welcome Realtime Server</h1>');
});
 
//在线用户
var onlineUsers = {};
//当前在线人数
var onlineCount = 0;
 
io.on('connection', function(socket){
    console.log('a user connected');
     
    //监听新用户加入
    socket.on('login', function(obj){
        //将新加入用户的唯一标识当作socket的名称，后面退出的时候会用到
        socket.name = obj.userid;
         
        //检查在线列表，如果不在里面就加入
        if(!onlineUsers.hasOwnProperty(obj.userid)) {
            onlineUsers[obj.userid] = obj.username;
            //在线人数+1
            onlineCount++;
        }
         
        //向所有客户端广播用户加入
        io.emit('login', {onlineUsers:onlineUsers, onlineCount:onlineCount, user:obj});
        console.log(obj.username+'加入了聊天室');
    });
     
    //监听用户退出
    socket.on('disconnect', function(){
        //将退出的用户从在线列表中删除
        if(onlineUsers.hasOwnProperty(socket.name)) {
            //退出用户的信息
            var obj = {userid:socket.name, username:onlineUsers[socket.name]};
             
            //删除
            delete onlineUsers[socket.name];
            //在线人数-1
            onlineCount--;
             
            //向所有客户端广播用户退出
            io.emit('logout', {onlineUsers:onlineUsers, onlineCount:onlineCount, user:obj});
            console.log(obj.username+'退出了聊天室');
        }
    });
     
    //监听用户发布聊天内容
    socket.on('message', function(obj){
        //向所有客户端广播发布的消息
        io.emit('message', obj);
        console.log(obj.username+'说：'+obj.content);
    });
   
});
 
http.listen(3000, function(){
    console.log('listening on *:3000');
});
```
客户端代码实现：

```javascript

<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta name="format-detection" content="telephone=no"/>
        <meta name="format-detection" content="email=no"/>
<meta content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=0" name="viewport">
        <title>多人聊天室</title>
        <link rel="stylesheet" type="text/css" href="./style.css" />
        <!--[if lt IE 8]><script src="./json3.min.js"></script><![endif]-->
        <script src="http://realtime.plhwin.com:3000/socket.io/socket.io.js"></script>
    </head>
    <body>
        <div id="loginbox">
            <div style="width:260px;margin:200px auto;">
                请先输入你在聊天室的昵称
                <br/>
                <br/>
                <input type="text" style="width:180px;" placeholder="请输入用户名" id="username" name="username" />
                <input type="button" style="width:50px;" value="提交" onclick="CHAT.usernameSubmit();"/>
            </div>
        </div>
        <div id="chatbox" style="display:none;">
            <div style="background:#3d3d3d;height: 28px; width: 100%;font-size:12px;">
                <div style="line-height: 28px;color:#fff;">
                    <span style="text-align:left;margin-left:10px;">Websocket多人聊天室</span>
                    <span style="float:right; margin-right:10px;"><span id="showusername"></span> |
                    <a href="javascript:;" onclick="CHAT.logout()" style="color:#fff;">退出</a></span>
                </div>
            </div>
            <div id="doc">
                <div id="chat">
                    <div id="message" class="message">
<div id="onlinecount" style="background:#EFEFF4; font-size:12px; margin-top:10px; margin-left:10px; color:#666;">
</div>
                    </div>
                    <div class="input-box">
                        <div class="input">
<input type="text" maxlength="140" placeholder="请输入聊天内容，按Ctrl提交" id="content" name="content">
                        </div>
                        <div class="action">
                            <button type="button" id="mjr_send" onclick="CHAT.submit();">提交</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <script type="text/javascript" src="./client.js"></script>
    </body>
</html>
```

client.js实现代码：

```javascript
(function () {
    var d = document,
    w = window,
    p = parseInt,
    dd = d.documentElement,
    db = d.body,
    dc = d.compatMode == 'CSS1Compat',
    dx = dc ? dd: db,
    ec = encodeURIComponent;
     
     
    w.CHAT = {
        msgObj:d.getElementById("message"),
        screenheight:w.innerHeight ? w.innerHeight : dx.clientHeight,
        username:null,
        userid:null,
        socket:null,
        //让浏览器滚动条保持在最低部
        scrollToBottom:function(){
            w.scrollTo(0, this.msgObj.clientHeight);
        },
        //退出，本例只是一个简单的刷新
        logout:function(){
            //this.socket.disconnect();
            location.reload();
        },
        //提交聊天消息内容
        submit:function(){
            var content = d.getElementById("content").value;
            if(content != ''){
                var obj = {
                    userid: this.userid,
                    username: this.username,
                    content: content
                };
                this.socket.emit('message', obj);
                d.getElementById("content").value = '';
            }
            return false;
        },
        genUid:function(){
            return new Date().getTime()+""+Math.floor(Math.random()*899+100);
        },
        //更新系统消息，本例中在用户加入、退出的时候调用
        updateSysMsg:function(o, action){
            //当前在线用户列表
            var onlineUsers = o.onlineUsers;
            //当前在线人数
            var onlineCount = o.onlineCount;
            //新加入用户的信息
            var user = o.user;
                 
            //更新在线人数
            var userhtml = '';
            var separator = '';
            for(key in onlineUsers) {
                if(onlineUsers.hasOwnProperty(key)){
                    userhtml += separator+onlineUsers[key];
                    separator = '、';
                }
            }
            d.getElementById("onlinecount").innerHTML = '当前共有 '+onlineCount+' 人在线，在线列表：'+userhtml;
             
            //添加系统消息
            var html = '';
            html += '<div class="msg-system">';
            html += user.username;
            html += (action == 'login') ? ' 加入了聊天室' : ' 退出了聊天室';
            html += '</div>';
            var section = d.createElement('section');
            section.className = 'system J-mjrlinkWrap J-cutMsg';
            section.innerHTML = html;
            this.msgObj.appendChild(section);  
            this.scrollToBottom();
        },
        //第一个界面用户提交用户名
        usernameSubmit:function(){
            var username = d.getElementById("username").value;
            if(username != ""){
                d.getElementById("username").value = '';
                d.getElementById("loginbox").style.display = 'none';
                d.getElementById("chatbox").style.display = 'block';
                this.init(username);
            }
            return false;
        },
        init:function(username){
            /*
            客户端根据时间和随机数生成uid,这样使得聊天室用户名称可以重复。
            实际项目中，如果是需要用户登录，那么直接采用用户的uid来做标识就可以
            */
            this.userid = this.genUid();
            this.username = username;
             
            d.getElementById("showusername").innerHTML = this.username;
            this.msgObj.style.minHeight = (this.screenheight - db.clientHeight + this.msgObj.clientHeight) + "px";
            this.scrollToBottom();
             
            //连接websocket后端服务器
            this.socket = io.connect('ws://realtime.plhwin.com:3000');
             
            //告诉服务器端有用户登录
            this.socket.emit('login', {userid:this.userid, username:this.username});
             
            //监听新用户登录
            this.socket.on('login', function(o){
                CHAT.updateSysMsg(o, 'login'); 
            });
             
            //监听用户退出
            this.socket.on('logout', function(o){
                CHAT.updateSysMsg(o, 'logout');
            });
             
            //监听消息发送
            this.socket.on('message', function(obj){
                var isme = (obj.userid == CHAT.userid) ? true : false;
                var contentDiv = '<div>'+obj.content+'</div>';
                var usernameDiv = '<span>'+obj.username+'</span>';
                 
                var section = d.createElement('section');
                if(isme){
                    section.className = 'user';
                    section.innerHTML = contentDiv + usernameDiv;
                } else {
                    section.className = 'service';
                    section.innerHTML = usernameDiv + contentDiv;
                }
                CHAT.msgObj.appendChild(section);
                CHAT.scrollToBottom(); 
            });
 
        }
    };
    //通过“回车”提交用户名
    d.getElementById("username").onkeydown = function(e) {
        e = e || event;
        if (e.keyCode === 13) {
            CHAT.usernameSubmit();
        }
    };
    //通过“回车”提交信息
    d.getElementById("content").onkeydown = function(e) {
        e = e || event;
        if (e.keyCode === 13) {
            CHAT.submit();
        }
    };
})();
```

摘录：
部分内容来自《深入浅出NodeJS》