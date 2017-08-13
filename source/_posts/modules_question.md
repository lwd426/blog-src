title: 学习笔记--模块化开发设计的几个比较重要的概念和AMD模块概述
date: 2012-06-09 09:10:09
tags:
- 模块化
categories: 模块化

---

`今天在知乎上看到个有关模块化的提问，底下的一个回答回答的很棒，就由这个提问引出今天的话题的---模块化`

### 提问

合并js 和 用requirejs 冲突吗?

如果把js都合并了 也就没有加载顺序的问题了,但是require能够做模块化和异步加载.

那么我现在有必要使用js合并吗?(或者说部分js合并).

我还不能理解 grunt 或者gulp 的requirejs 插件能干嘛.使用了grunt的话应该还是需要加载requirejs文件的吧?

### 回答一

简短一点， **requirejs 做模块化开发主要是方便维护，明确依赖关系。**

**define 是生成一个匿名函数，编译但不执行，需要的时候，再去执行，而且只在当前页面执行一次。**
<!-- more -->
看到define的属性就明白了，压缩和合并没有关系，因为define根本不会执行。。 只是一个匿名函数。

```javascript
var a = function(){ var cc=1; }
```

不主动调用a() 是不会执行的。所以可以将所有的define压缩到一起，没有冲突，甚至可以混合压缩到一起。只有在 require的时候，才会执行。 没有冲突。

### 回答二---很棒


这个问题设计到多个概念词汇：

1. **模块化开发**:模块化开发无非是为了解耦和代码重用；期间的优势你如果不能理解说明你还没达到那个水平；
2. **requirejs**；requirejs在模块化开发中作为落地方案之一的技术框架，主要功能是按需加载依赖模块。
3. 所有的加载器无非是实现几个功能：
    * 解析运行环境，解析主程序入口
    * 解析模块路径；加载模块代码并执行回调业务；

    所有类型模块加载器基本都会执行以上业务(amd，cmd，kmc……),在浏览器环境下的模块代码的加载跟在node环境下的模块代码加载的解析会有一些差异；
3. **代码合并**：在web前端静态文件上线之前必做的一个优化手段：减轻代码文件的体积，减少http请求；
4. **减轻代码体积**:的手段就是压缩代码（俗称ugly），在node环境下有很多类似的工具库可以用；
5. **减少http请求**的手段一般就是代码合并；如将a和b的代码放到一个js文件里面去（css同理）（俗称combo）；
6. 在执行amd标准时，一个js文件只允许一个模块的存在，故在减少http请求层面，这是相斥的，但**amd和cmd标准都有一个具名模块的定义方式，这个时候是允许一个js文件存在多个模块代码的**；

**注意下面这段！！**

你所不能理解的应该是r.js（requirejs的打包工具）。

* 它的工作其实就是解读出根据你的配置环境的目录结构下的代码存放目录结构，
* 然后把标准的代码结构转换成具名模块的代码结构，
* 如果你执行了合并，则把依赖的模块的代码合并到制定的一个文件中；

来自：[http://www.zhihu.com/question/34007485](http://www.zhihu.com/question/34007485)

### AMD的各种形式的模块定义

#### api定义

AMD设计出一个简洁的写模块API：

```javascript
define(id?, dependencies?, factory);
```

其中：

* id: 模块标识，可以省略。
* dependencies: 所依赖的模块，可以省略。
* factory: 模块的实现，或者一个JavaScript对象。
* id遵循CommonJS Module Identifiers 。dependencies元素的顺序和factory参数一一对应。
 
#### AMD中5种定义模块的方式

#### 普通方式

以下是使用AMD模式开发的简单三层结构（基础库/UI层/应用层）：
 
```javascript
//base.js
define(function() {
    return {
        mix: function(source, target) {
        }
    };
});
``` 
```javascript
//ui.js
define(['base'], function(base) {
    return {
        show: function() {
            // todo with module base
        }
    }
});
```
```javascript
//page.js
define(['data', 'ui'], function(data, ui) {
    // init here
});
``` 
```javascript
//data.js
define({
    users: [],
    members: []
});
```

以上同时演示了define的三种用法:

* 定义无依赖的模块（base.js）
* 定义有依赖的模块（ui.js，page.js）
* 定义数据对象模块（data.js）
 
##### 具名模块
细心的会发现，还有一种没有出现，即具名模块
 
* 具名模块⭐️

```javascript
define('index', ['data','base'], function(data, base) {
 	  // todo
});
```
具名模块多数时候是不推荐的，**一般由打包工具合并多个模块到一个js文件中时使用。**（与上文呼应）
 
##### 包装模块⭐️
前面提到dependencies元素的顺序和factory一一对应，其实不太严谨。AMD开始为摆脱CommonJS的束缚，开创性的提出了自己的模块风格。但后来又做了妥协，兼容了 CommonJS Modules/Wrappings 。即又可以这样写
 
* 包装模块

```javascript
//此种用法就是kibana使用的方式，也可以用于懒加载
define(function(require, exports, module) {
    var base = require('base');
    exports.show = function() {
        // todo with module base
    }
});
``` 

不考虑多了一层函数外，格式和Node.js是一样的：使用require获取依赖模块，使用exports导出API。（可以做懒加载）

#### 其他 
除了define外，AMD还保留一个关键字require。require 作为规范保留的全局标识符，可以实现为 module loader，也可以不实现。
 
目前，实现AMD的库有RequireJS 、curl 、Dojo 、bdLoad、JSLocalnet 、Nodules 等。
也有很多库支持AMD规范，即将自己作为一个模块存在，如MooTools 、jQuery 、qwery 、bonzo  甚至还有 firebug 。

此部分转自：[http://www.cnblogs.com/snandy/archive/2012/03/12/2390782.html](http://www.cnblogs.com/snandy/archive/2012/03/12/2390782.html)
