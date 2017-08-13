title: 学习笔记—异步流程控制库和回调处理
date: 2015-10-09 16:24:09
tags:
- 异步流程
- Promise
categories: Promise

----

虽然一直在用Async和Promise，但因为工作很忙，始终没有对其中的奥秘有个窥探，今天特此捣鼓捣鼓。

### 概念

首先应该明白两个概念：

* **Async是一个JavaScript的异步流程库**：它解决了JS在使用过程中为了进行异步执行从而导致的回调地狱（主要是控制权转义频繁）；同时能够更简单的进行错误处理（只要有错误发生就不往下执行了，进入done函数传出error信息）这个库最常用的方法就是：
	1. series: 串行执行，一个函数数组（N个）中的每个函数，每一个函数执行完成之后才能执行下一个函数,最后done中得到的results为N个结果的数组。
	2. parallel: 并行执行多个函数（N个），每个函数都是立即执行，不需要等待其它函数先执行。传给最终callback的数组中的数据按照tasks中声明的顺序，而不是执行完成的顺序（done中得到的results为N个结果的数组）。
	3. waterfall: 按顺序依次执行一组函数。每个函数产生的值，都将传给下一个，最后done中只会有一个结果值。
* **Promise是一个JavaScript用来处理回调的规范**：它更侧重于处理回调函数，对流程控制（异步函数数组是怎么执行的）的处理略显单一，以我现在的理解，可以理解为Promise规范把流程限制于串行执行。

<!-- more -->
### Async库的实现原理

简单来说：Async这个异步流程库对多个函数数组的流程的控制是：

1. 通过把传入的函数数组缓存在一个数组对象里
2. 然后通过一个next函数来对流程进行控制
3. 如果一个方法处理正常，就进行下一步：
   
   * 同步执行：就取数组中下一个元素然后执行
   * 异步执行：就直接把执行结果放到results里，等待在done函数里处理
   * 瀑布式执行：要把上一个函数执行结果传给下一个元素，然后继续执行
4. 如果一个方法处理错误，就直接取数组的最后一个函数（done）,调用done函数。

这里具体实现就不在这里写了。

### Promise规范实现--bluebird的实现原理

现在最常用的Promise规范的一个实现库应该就是bluebird了。今天把官方文档仔细读了读，还看了几个常用方法的api和反模式。

#### 反模式一：重复实现promise
bluebird已经把基本上所有能用的的方式都进行了封装和处理，这样就把使用Promise规范来进行回调处理的一大弊端（需要自己根据业务实现promise）给解决掉了。所以它**不推荐在使用bluebird的同时再在自己的代码里去实现调用resolve和reject的逻辑**。如下，是一个反模式（官方示例）：

```js
myApp.factory('Configurations', function (Restangular, MotorRestangular, $q) {
    var getConfigurations = function () {
        //这里根本就不用自己再去实现
        var deferred = $q.defer();
        MotorRestangular.all('Motors').getList().then(function (Motors) {
            //Group by Config
            var g = _.groupBy(Motors, 'configuration');
            //Map values
            var mapped = _.map(g, function (m) {
                return {
                    id: m[0].configuration,
                    configuration: m[0].configuration,
                    sizes: _.map(m, function (a) {
                        return a.sizeMm
                    })
                }
            });
            deferred.resolve(mapped);//
        });
        return deferred.promise;
    };

    return {
        config: getConfigurations()
    }

});

```
应该如下：

```js
myApp.factory('Configurations', function (Restangular, MotorRestangular, $q) {
    var getConfigurations = function () {
        //Just return the promise we already have!
        return MotorRestangular.all('Motors').getList().then(function (Motors) {
            //Group by Cofig
            var g = _.groupBy(Motors, 'configuration');
            //Return the mapped array as the value of this promise
            return _.map(g, function (m) {
                return {
                    id: m[0].configuration,
                    configuration: m[0].configuration,
                    sizes: _.map(m, function (a) {
                        return a.sizeMm
                    })
                }
            });
        });
    };

    return {
        config: getConfigurations()
    }

});
```

#### 反模式二：不要用.them(success,fail)

原因跟上一个反模式相似，bluebird反对以.then(success,fail)这种方式使用。因为它已经可以让你使用.catch()方法轻松处理错误，还能够根据错误详情，多次调用.catch()。所以，正确的方式应该是这样的：

```js
promise
	.then(function(data){})
	.then(function(data){})
	...
	.catch(MyError,function(err){})
	...
	.catch(function(err){});
```

#### 常用的三个方法

bluebird也有几个比较常用的方法：

* promisify()：传入一个异步函数，返回一个promise对象
* promisifyAll()：传入一个异步函数库，返回一个promise对象（注意此方法有一个参数{suffix}决定了这个库被bluebird处理后，每个方法的后缀名，默认为Async）。

具体方法网上一大推，就不赘述了。

下面自己实现了下promisify这个方法，代码如下：

promisify.js

```js
function promisify2(){
	var args = Array.prototype.slice.apply(arguments);
	var f = args[0];
	return function(){
		var args2 = Array.prototype.slice.apply(arguments);
		return new Promise(function(resolve, reject){
				args2.push(function(err,results){
					if(err){
						reject(err);
					}else{
						resolve(results);
					}
					
				});
				f.apply(null, args2);
			});
	}
}
	
module.exports = {

	promisify2 : promisify2
};
```

test.js

```js
var promise = require('./promisify'),
    read = promise.promisify2(require('fs').readFile);

read('lwd.txt')
    .then(function(data){
		console.log(data+'');
    })
    .catch(function(e){
		console.log(e);
    });
```