title: 解决方案：node解析markdown
date: 2015-07-24 15:15:49
tags: 
- nodejs
categories: nodeJS

---


>目前，越来越多的博客开始采用markdown来撰写。它的轻量、书写方便、易读易写的特点让我着迷。真的很简便。所以我们打算用markdown来写产品的使用说明。

### 问题产生

    我们的产品采用angularJS做前端MVVM框架，后端采用Nodejs。
目前markdown浏览器还不能解析markdown,最终也要转换成html才能被浏览器解析。所以采用markdown来做文档说明，那就要解决以下两个问题：

<!-- more -->

* 前端解析markdown，还是放到后端用node做？
* 采用什么解析markdown?有没有工具去做？

### 解决方案

以下就来一个一个的解决：

`软件开发中没有银弹。`
任何技术或者解决方案都有自己使用的情境，因为markdown文件也可以任务是一种静态资源，所以为了提高性能，最好是采用服务端启动解析成html文件放到静态资源目录里。或者是直接推给angularJS。所以针对第一个问题的答案：

`放到node端做解析`

经过技术选型，我们筛选出了marked来在node端做markdown文档的解析，在npm上有很高的使用率。具体使用方法请看[marked](https://www.npmjs.com/package/marked)。所以第二个问题的答案就是：

`使用较为成熟的marked.js`

### 解决步骤

具体的在node端安装marked.js模块的方法就不赘述了，可以去api上查阅。着重说下我们是怎么做的。

有两种使用方式来解析markdown文档：

* 直接使用marked命令行：这种做法太笨拙，需要手动去搞，所以没有用
* 在node代码里使用marked的api：`bingo!`

```javascript
var marked = require('marked')
    ,fs = require("fs")
    ,http = require('http');

marked.setOptions({
    renderer: new marked.Renderer(),
    gfm: true,
    tables: true,
    breaks: true,
    pedantic: false,
    sanitize: true,
    smartLists: true,
    smartypants: false
});

fs.readFile('m1.md', 'utf-8', function (err, data) {
    if (err) throw err;
    write2static(marked(data));
});

function write2static(data){
	fs.writeFile(staticPath+'m.md', data, function 	(err) {
 	 if (err) throw err;
 	 console.log('It\'s saved!');
	});
}

```
以上代码仅是示意，如果考虑到效率需要优化方案。

注意：此过程当然可以在build的过层中做，但是在node启动的时候解析markdown并生成到静态资源目录有一个好处：`可以直接修改markdown.md文档后，重启node立即就解析输出了，再配合pm2守护进程，世界一下就变得简单而又美好了。有没有。。`



