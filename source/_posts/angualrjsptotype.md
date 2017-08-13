title: javascript原型集成到angularJS继承
date: 2014-07-23 10:15:49
tags: 
- AngularJS
categories: AngularJS
---

### 一、遇到的问题
问题发生在使用 AngularJS 嵌套 Controller 的时候。因为每个 Controller 都有它对应的 Scope（相当于作用域、控制范围），所以 Controller 的嵌套，也就意味着 Scope 的嵌套。这个时候如果两个 Scope 内都有同名的 Model 会发生什么呢？从子 Scope 怎样更新父 Scope 里的 Model 呢？

这个问题很典型，比方说当前页面是一个产品列表，那么就需要定义一个 ProductListController

<!-- more -->

```javascript
function ProductListController($scope, $http) {
    $http.get('/api/products.json')
        .success(function(data){
            $scope.productList = data;
        });
    $scope.selectedProduct = {};
}
```
你大概看到了在 Scope 里还定义了一个 selectedProduct 的 Model，表示选中了某一个产品。这时会获取该产品详情，而页面通过 AngularJS 中的 $routeProvider 自动更新，拉取新的详情页模板，模板中有一个 ProductDetailController

```javascript
function ProductDetailController($scope, $http, $routeParams) {
    $http.get('/api/products/'+$routeParams.productId+'.json')
        .success(function(data){
            $scope.selectedProduct = data;
        });
}
```
有趣的事情发生了，在这里也有一个 selectedProduct ，它会怎样影响 ProductListController 中的 selectedProduct 呢？

答案是没有影响。在 AnuglarJS 里子 Scope 确实会继承父 Scope 中的对象，但当你试下对基本数据类型（string, number, boolean）的 双向数据绑定 时，就会发现一些奇怪的行为，继承并不像你想象的那样工作。子 Scope 的属性隐藏（覆盖）了父 Scope 中的同名属性，对子 Scope 属性（表单元素）的更改并不更新父 Scope 属性的值。这个行为实际上不是 AngularJS 特有的，JavaScript 本身的原型链就是这样工作的。开发者通常都没有意识到 ng-repeat, ng-switch, ng-view 和 ng-include 统统都创建了他们新的子 scopes，所以在用到这些 directive 时也经常出问题。

### 二、解决的办法
解决的办法就是不使用基本数据类型，而在 Model 里永远多加一个点.

```javascript
使用
<input type="text" ng-model="someObj.prop1">
来替代
<input type="text" ng-model="prop1">
```
是不是很坑爹？下面这个例子很明确地表达了我所想表达的奇葩现象

```javascript
app.controller('ParentController',function($scope){
    $scope.parentPrimitive = "some primitive"
    $scope.parentObj = {};
    $scope.parentObj.parentProperty = "some value";
});
app.controller('ChildController',function($scope){
    $scope.parentPrimitive = "this will NOT modify the parent"
    $scope.parentObj.parentProperty = "this WILL modify the parent";
});
```

但是我真的确实十分很非常需要使用 string number 等原始数据类型怎么办呢？2 个方法:

* 在子 Scope 中使用 $parent.parentPrimitive。 这将阻止子 Scope 创建它自己的属性。
* 在父 Scope 中定义一个函数，让子 Scope 调用，传递原始数据类型的参数给父亲，从而更新父 Scope 中的属性。（并不总是可行）

### 三、JavaScript 的原型链继承
吐槽完毕，我们来深入了解一下 JavaScript 的原型链。这很重要，特别是当你从服务器端开发转到前端，你应该会很熟悉经典的 Class 类继承，我们来回顾一下。

假设父类 parentScope 有如下成员属性 aString, aNumber, anArray, anObject, 以及 aFunction。子类 childScope 原型继承父类 parentScope，于是我们有：

![](/images/angularjs/angularjs1.png)

如果子 Scope 尝试去访问 parentScope 中定义的属性，JavaScript 会先在子 Scope 中查找，如果没有该属性，则找它继承的 scope 去获取属性，如果继承的原型对象 parentScope 中都没有该属性，那么继续在它的原型中寻找，从原型链一直往上直到到达 rootScope。所以，下面的表达式结果都是 ture：

```javascript
childScope.aString === 'parent string'childScope.anArray[1] === 20childScope.anObject.property1 === 'parent prop1'childScope.aFunction() === 'parent output'

```
假设我们执行下面的语句

```javascript
childScope.aString = 'child string'
```
原型链并没有被查询，反而是在 childScope 中增加了一个新属性 aString。这个新属性隐藏（覆盖）了 parentScope 中的同名属性。在下面我们讨论 ng-repeat 和 ng-include 时这个概念很重要。

![](/images/angularjs/angularjs2.png)


假设我们执行这个操作：

```javacsript 
childScope.anArray[1] = '22'childScope.anObject.property1 = 'child prop1'
```
原型链被查询了，因为对象 anArray 和 anObject 在 childScope 中没有找到。它们在 parentScope 中被找到了，并且值被更新。childScope 中没有增加新的属性，也没有任何新的对象被创建。（注：在 JavaScript 中，array 和 function 都是对象）

![](/images/angularjs/angularjs3.png)


假设我们执行这个操作：

```javacsript 
childScope.anArray = [100, 555]childScope.anObject = { name: 'Mark', country: 'USA' }
```
原型链没有被查询，并且子 Scope 新加入了两个新的对象属性，它们隐藏（覆盖）了 parentScope 中的同名对象属性。

![](/images/angularjs/angularjs4.png)

应该可以总结

如果读取 childScope.propertyX，并且 childScope 有属性 propertyX，那么原型链没有被查询。
如果设置 childScope.propertyX，原型链不会被查询。
最后一种情况，

```javacsript 
delete childScope.anArraychildScope.anArray[1] === 22  // true
```
我们从 childScope 删除了属性，则当我们再次访问该属性时，原型链会被查询。删除对象的属性会让来自原型链中的属性浮现出来。

![](/images/angularjs/angularjs5.png)

四、AngularJS 的 Scope 继承
创建新的 Scope，并且原型继承：ng-repeat, ng-include, ng-switch, ng-view, ng-controller, directive withscope: true, directive with transclude: true
创建新的 Scope，但不继承：directive with scope: { ... }。它会创建一个独立 Scope。
注：默认情况下 directive 不创建新 Scope，即默认参数是 scope: false。

ng-include
假设在我们的 controller 中，

```javacsript 
$scope.myPrimitive = 50;$scope.myObject    = {aNumber: 11};
```
HTML 为：

```javacsript 
<script type="text/ng-template" id="/tpl1.html">
    <input ng-model="myPrimitive"></script><div ng-include src="'/tpl1.html'"></div> <script type="text/ng-template" id="/tpl2.html">
    <input ng-model="myObject.aNumber"></script><div ng-include src="'/tpl2.html'"></div>
```
每一个 ng-include 会生成一个子 Scope，每个子 Scope 都继承父 Scope。

![](/images/angularjs/angularjs6.png)

输入（比如”77″）到第一个 input 文本框，则子 Scope 将获得一个新的 myPrimitive 属性，覆盖掉父 Scope 的同名属性。这可能和你预想的不一样。

![](/images/angularjs/angularjs7.png)

输入（比如”99″）到第二个 input 文本框，并不会在子 Scope 创建新的属性，因为 tpl2.html 将 model 绑定到了一个对象属性（an object property），原型继承在这时发挥了作用，ngModel 寻找对象 myObject 并且在它的父 Scope 中找到了。

![](/images/angularjs/angularjs8.png)

如果我们不想把 model 从 number 基础类型改为对象，我们可以用 $parent 改写第一个模板：

<input ng-model="$parent.myPrimitive">
输入（比如”22″）到这个文本框也不会创建新属性了。model 被绑定到了父 scope 的属性上（因为 $parent 是子 Scope 指向它的父 Scope 的一个属性）。

![](/images/angularjs/angularjs9.png)

对于所有的 scope （原型继承的或者非继承的），Angular 总是会通过 Scope 的 $parent, $$childHead 和 $$childTail 属性记录父-子关系（也就是继承关系），图中为简化而未画出这些属性。

在没有表单元素的情况下，另一种方法是在父 Scope 中定义一个函数来修改基本数据类型。因为有原型继承，子 Scope 确保能够调用这个函数。例如，

```javacsript 
// 父 Scope 中$scope.setMyPrimitive = function(value) {
    $scope.myPrimitive = value;}
```

#### 总结
一共有四种 Scope：

* 普通进行原型继承的 Scope —— ng-include, ng-switch, ng-controller, directive with scope: true
* 普通原型继承的 Scope 但拷贝赋值 —— ng-repeat。 每个 ng-repeat 的循环都创建新的子 Scope，并且子 Scope 总是获得新的属性。
* 独立的 isolate scope —— directive with scope: {...}。它不是原型继承，但 ‘=’, ‘@’ 和 ‘&’ 提供了访问父 Scope 属性的机制。
* transcluded scope —— directive with transclude: true。它也遵循原型继承，但它同时是任何 isolate scope 的兄弟。
对于所有的 Scope，Angular 总是会通过 Scope 的 $parent, $$childHead 和 $$childTail 属性记录父-子关系。

参考链接
http://stackoverflow.com/questions/14049480/what-are-the-nuances-of-scope-prototypal-prototypical-inheritance-in-angularjs