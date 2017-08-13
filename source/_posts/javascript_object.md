title: JavaScript的对象和继承者们
date: 2012-08-04 14:50:08
tags:
- javascript
- 基础
categories: javascript

---

JavaScript的创建对象和继承什么的真的好绕人啊！今天把它们搞明白！

* JavaScript生成对象的几种方式
* JavaScript继承对象的两种方式

### 怎么生成JS对象？
这个问题好简单啊！直接`var obj = {};`不就生成了一个对象么？还可以用构造函数`var obj = new F();`额...应该就这两种了吧。我相信很多js初学者都熟悉这两种生成JS对象的方式。但你真的明白么？或者说，你知道什么叫“明白”么？下面就来说说吧。
<!-- more -->
1. 通过对象字面量直接创建对象
2. 通过new 构造函数()创建对象
3. 通过原型继承（其实是高级的构造函数的使用）
4. 通过Object.create创建对象

#### 对象字面量
创建对象的最简单的方式就是使用对象字面量，就如上面的例子，这种方法`最好，效率也最好，推荐使用`以下用代码说明：

```javascript
//使用对象字面量生成对象
var obj = {'name':'lwd426','age':26,'sex':male};
```
但这种方式有一个特别大的短板，就是可能生成一个对象，如果要生成很多具有共性的对象，那就得一个一个的码字了。**当需要生成简单的少数的对象时，推荐使用这种简便的做法。**

#### 构造函数
当我们稀里糊涂的使用java的new新对象的时候，似乎觉得js使用这种方式生成类的实例很正常，似乎一切也行得通，代码照跑不误。其实不是这样的，只是‘貌似’，‘神’可不似。为了解决`对象字面量只能创建单一对象`的问题，JS让我们可以使用构造函数（当函数被new操作符调用，函数就成为了构造函数，他的行为有些不一样了）来创建构造函数的`实例`（构造函数返回的对象我们可以成为‘实例’）。同时，我们也可以通过传参的方式让我们的对象有一些个性化预定义属性。

```javascript
//构造函数
function F(proa, pron){
	this.proa = proa;
	this.pron = pron;
	this.proarray = [];
	this.getProa = function(){
		return this.proa;
	}
}

var f1 = new F('pro1', 3);
var f2 = new F('pro2', 1);
console.log(f.pron); //2
console.log(f.pron); //2
```

* 构造函数可以创造多个共享特定属性和行为的对象。
* new让函数执行过程有点不一样：
	1. 创建新对象F，让继承自构造器函数的原型对象，F连接到构造器prototype的新对象
	2. 执行构造器函数，绑定函数的this到新对象上
	3. 返回新对象this
	
	```javascript
	//new的实现逻辑（模拟）
	//method方法为扩展原型的工具方法
	//Object.create为生成一个原对象为原型的新对象
	Function.method('new'， function(){
		var obj = Object.create(this.prototype);
		this.apply(obj, arguments);
		return obj;
	});
	```

但是，每一个对象都要给自己实例化一遍所有方法，挂到this下，这好浪费资源啊。我要是能把方法提出来就好了，咱要‘面向对象’嘛，搞个类出来不就ok了么！现在多流行啊。

#### 原型继承

不过，`JavaScript是基于原型继承的，没有类的概念`。于是牛逼的设计者就为JS设计出了个对象，叫原型。把所有对象共有的属性和方法都放到原型对象中，来共享对象的属性和方法，再让对象继承这个原型对象，这样就解决了上面的问题。

```javascript
//构造函数
function F(proa, pron){
	this.proa = proa;
	this.pron = pron;
	this.proarray = [];
}

F.prototype.getProa = function(){
	return this.proa;
}

var f1 = new F('pro1', 3);
var f2 = new F('pro2', 1);
console.log(f.pron); //2
console.log(f.pron); //2
```

* 原型是存放继承特征的地方（也就是共享属性和方法的地方）
* JS的所有对象都连接到一个原型对象，并且新对象可以从原型对象中继承属性和方法
* JS的所有对象都有一个属性叫prototype,这个属性指向自己的原型对象。
* 原型关系是一种动态的关系。如果在原型对象中添加一个新属性，那么该属性会立即对所有基于该原型创建的对象生效可见。

有了原型，一切似乎变得简单起来：

* 对象可以直接从另一个对象`继承`属性，只需要以旧对象为原型，做‘差异性继承’就好了。
* 使用原型可以轻松完成`创建`一个对象的另一个对象。

所以我们又有了第三种创建对象的方法：

#### var obj = Object.create(OriginObj);
此为道格拉斯发明的方法，推荐使用这种方法来创建对象：

```javascript
Object.create方法的实现
if(typeof Object.beget !== 'function'){
	Object.create = function(o){
		var F = function(){};
		F.prototype = o;
		return new F();
	}
}

//旧对象
var originObj = {'first_name':'lwd426','second_name':'Liu'};

var obj1 = Object.create(originObj);
var obj2 = Object.create(originObj);
```

### 怎么继承对象？

>其实js设计出来是没有想做继承的，因为它被设计出来仅仅是为了满足简单的浏览器和用户交互的功能，没想到会今天单页面程序这种大型的前端架构的出现。

怎么新建对象已经搞得挺明白，起码够用了。现在有一个更实用的问题摆在面前：怎么继承对象呢？

>JavaScript是一门基于`原型`的语言，这意味着对象可以直接从其他对象继承
>
>在一个纯粹的原型继模式中，我们会摒弃类，转而专注于对象。
>
>基于原型的继承先对基于类的继承在概念上更加简单：一个对象可以继承以旧的对象。

有两种继承的方法：

1. 伪类继承：没有私有环境，不能访问父类方法
2. 模块模式完成继承：可以保护私有信息，可以访问父类

#### 伪类

简单来说就是：

* 原型对象用作是存放继承特征的地方
* 定义一个中间伪类（子类）继承父类构造函数实例
* 然后添加/覆盖伪类（子类）新方法/属性。
* 最后，新建子类的实例

```javascript
  //引用《JavaScript精粹》的例子代码
  //我们可以定义一个构造器并扩充它的原型以作父类
   var Mammal = function (name) {
		this.name = name;
	};
	
	Mammal.prototype.get_name = function () {
		return this.name;
	};
	
	Mamal.prototype.says = function () {
		return this.saying || '';
	};
	
	//再构造一个实例：
	var myMammal = new Mammal('Herb the Mamal');
	var name = myMammal.get_name(); // 'Herb the Mammal';
	document.writeln(name);
	
	//构造一个伪类来继承Mammal，这是通过定义它的constructor函数并替换它的prototype为一个Manmal的实例来实现的：
	var Cat = function (name) {
		this.name = name;
		this.saying = 'meow';
	};
	
	// 替换 Cat.prototype 为一个新的 Mammal 实例
	Cat.prototype = new Mammal();
	// 扩充新原型对象，增加 purr 和 get_name 方法
	Cat.prototype.purr = function (n) {
		var i, s = '';
		for (i = 0; i < n; i += 1) {
			if (s) {
				s +- '-';
			}
			s += 'r';
		}
		return s;
	};
	Cat.prototype.get_name	= function () {
		return this.says() + ' ' + this.name + ' ' + this.says();
	};
	
	var myCat = new Cat('Henrietta');
	var says = myCat.says(); // 'meow'
	var purr = myCat.purr(5); // 'r-r-r-r-r'
	var name = myCat.get_name(); // 'meow Henrietta meow'
```
好费劲啊，这种方式是想模拟面向对象的思想，但是真的很生硬，我们为什么要继承父‘类’的实例，而不是直接继承父‘类’呢，这不是更好么？所以我们可以通过使用method方法来定义一个inherits方法来封装以上的逻辑实现伪类的继承：

```javascript
Function.method('inherits', function(Parent){
    this.prototype = new Parent();
    return this;
});
```
```javascript
var Cat = function(name){
	this.name = name;
	this.saying = 'meow';
}
.inherits(Mammal)
.method('purr', function (n) {
		var i, s = '';
		for (i = 0; i < n; i += 1) {
			if (s) {
				s +- '-';
			}
			s += 'r';
		}
		return s;
})
.method('get_name', function () {
		return this.says() + ' ' + this.name + ' ' + 				this.says();
});
```
伪类模式继承的问题：

1. 没有私有环境，所有属性都是公开的。无法访问super（父类）方法。
2. 如果在调用构造器函数时候忘记调用new操作符，那么this将不会绑定到新的对象上，而是全局window上。
3. “伪类”的形式可以给不收悉js的程序员便利，但它也隐藏了该语言的真实本质。借鉴类的表示法可能误导程序员去编写过于深入与复杂的层次结构。
4. construction的指向错误。

#### 模块模式
大部分所看到的继承模式的一个弱点就是`没办法去保护隐私`。对象的属性都是可见的。没有办法得到私有的变量和函数。所以我们使用`模块模式`利用闭包保护隐私信息，提供特殊函数做为接口对外服务。

```javascript
var consturctor = function(spec, my){
    var that,   // 其他的私有实例变量
        my = my || {};

    // 把共享的变量和函数添加到 my 中
    // 给 that = 一个新的对象
    // 添加给 that 的特权方法

    // 最后把 that 对象返回
    return that;
};
```
1. 创建一个对象。
2. 定义私有实例的变量和方法。
3. 给这个新的对象扩充方法，这些方法拥有特权去访问参数。
4. 返回那个对象。

`spec`对象包含构造器需要构造一个新的实例的所有信息。spec的可能会被复制到私有变量中，或者被其他函数改变，或者方法可以在需要的时候访问spec的信息。

声明该对象私有的实例变量的方法。通过简单地声明变量就可以做到了。构造器的变量和内部函数变成了该实例的私有成员。

`my`对象是一个为继承链中的构构造器提供的秘密共享的容器。通过给my对象添加共享秘密成员：

```javascript
my.member = value;
```

然后构造一个新的对象并且把它赋值给that。接着扩充that，加入组成该对象接口的特权方法。可以分配一个新函数成为that的成员方法，然后再把它分配给that：

```javascript
var methodical = function(){ ... };

that.methodical = methodical;
```
分两步去定义methodical的好处就是，如果其他方法想要调用methodical,它们可以直接调用methodical()而是不是that.methodical()。如果实例遭到破坏或修改，调用methodical照样会继续工作，因为它们私有的methodical不会该实例被修改的影响。

最后把that返回。

函数化模式有很大的灵活性。它相比伪类模式不仅带来的工作更少，还让我们得到更好的封装和信息隐藏，以及访问父类方法的能力。

如果对象的所有状态都是私有的，那么该对象就成为一个“防伪”对象。该对象的属性可以被替换或删除，但该对象的完整性不会受到伤害。

### 工具方法
好了，让我们总结所有用到的工具方法吧：

#### new的实现逻辑（模拟):

```javascript
//method方法为扩展原型的工具方法
//Object.create为生成一个原对象为原型的新对象
Function.method('new'， function(){
    var obj = Object.create(this.prototype);
    this.apply(obj, arguments);
    return obj;
});
```

#### 创建对象方法的实现:

```javascript
if(typeof Object.beget !== 'function'){
	Object.create = function(o){
		var F = function(){};
		F.prototype = o;
		return new F();
	}
}
```

#### 继承方法的实现：

```javascript
Function.method('inherits', function(Parent){
    this.prototype = new Parent();
    return this;
});
```

#### 扩展原型方法的实现：

```javascript
Function.prototype.method = function(name, func){
	if(!this.prototype[name]){
		this.prototype[name] = func;
	}
	return this;
}
```