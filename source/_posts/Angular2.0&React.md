title: 有关Angular2.0和React的技术选型
date: 2015-12-10 10:10:10
tags: 
- react
categories: react

----

当前，前端领域最热的几个话题中一定有Angular2.0和React这两个。前端时间参加2015开发者大会，前端论坛也基本上是在讨论这两个主题，并极力推广react的使用。

鉴于明年部门一个产出是“组件化”，而这两个框架（react属于库）都适合于组件化的开发。本文就简单说说针对移动平台部，从基本特征和主要点的设计思路、学习成本、代码分析和相关生态上讲解Angular2.0和React的实用性和各种利弊。
<!-- more -->
### Angular2.0 

#### 基本特征和设计思路

值得一提的是，Angular2.0是一个MVVM框架，而React是一款专注于构建可复用组件的库。这是二者最显而易见的区别。

首先，Angular2.0与AngularJS 1.x有极大的不同，可以说从设计思路和架构上就进行了大手术。从名称上也可以看书，Angular2.0不能叫AngularJS2.0（这是Angular还有一个Dart版本）。在AngularJS 1.x中存在的诸如Controller、Service、Factory、指令包括那么重要的Scope都不见了。这么激进的革新，出发点在哪里呢？**除了解决自身的一些问题，更重要的意义在于对未来标准的迎合**，这些标准主要包括：

- module
- Web Components
- class
- observe
- promise

module的问题很好理解，JavaScript第一次有了语言上的模块机制，而Web Components则是约定了基于泛HTML体系构建组件库的方式，class增强了编程体验，observe提供了数据和展现分离的一种优秀方式，promise则是目前前端最流行的异步编程方式。

即使你很熟悉Angular 1.x，一下去读Angular2.0的Hello World代码，相信我，你看不懂。那这是为什么呢？我们一点点来说。

##### 语法：AtScript和ES6

Angular2.0 开发小组自己开发出一套新的语言AtScript。

AtScript属于JavaScript的扩展语言，它与JavaScript的关系如下图：

![](/images/atscript.png)

有关AtScript的讲解，可以看下知乎的这个帖子说的比较清晰：[贴1](http://www.zhihu.com/question/28310183)

从上图也可以看出，Angular2.0也是基于ES6的。自从今年ES6推出，新出来的框架也必定开始面向未来，所以包括React也推荐大家用ES6语法去书写代码。有关ES6的语法可以看下阮一峰老师的ES6入门，当然，能够阅读英文版规范的更好。

值得一提的是，目前编写任何ES6代码都要在上线前使用编译工具（如gulp或grunt）把代码编译成ES5代码，原因是很多浏览器目前还不支持ES6语法。

##### 基于ES6语法

ECMAScript 6（以下简称ES6）是JavaScript语言的下一代标准，已经在2015年6月正式发布了。它的目标，是使得JavaScript语言可以用来编写复杂的大型应用程序，成为企业级开发语言。

现今，越来越多的框架和库都在使用ES6语法来进行代码书写。Angular2.0和React都鼓励用ES6。ES6加入了以下新特性：

- let和const命令
- 变量的解构赋值
- 字符串、正则、数值、数组、函数和对象进行了扩展
- 加入了Set和Mao数据解构
- Iterator和for...of循环
- Generator函数
- Promise对象
- 异步操作和Async函数
- 引入了class
- 引入了module

这些基本都可以理解为ES5的语法糖。为什么做了这么多新的东西？原因就是近些年来随着库和框架越来越多，加上人们越来越多的使用javascript开发大型应用，就产生出了很多好用的工具、库、框架和思想。比如适用于后端的mvc和模块的概念思路都在前端开发中被使用。JS是异步的，这大家都知道，所以一些基本的异步操作和流程控制函数的出现就变得很自然了。js饱受诟病的一点就是没有类，所以es终于引入了类的概念，让代码编写起来更美观，阅读性更高了。
如果想系统的了解这些新特性，可以阅读阮一峰老师的[《ECMAScript6入门》](http://es6.ruanyifeng.com/#docs/intro)。当然，我们要支持正版，最好是买一本。

##### 基于注解

有关注解的基本使用在上面的帖子里说的比较清晰了。这里引用一下：
**注解的意义主要在于为类型添加特定的约束，好比一个类型是一张纸牌，有注解的类型相当于在纸牌背面贴了额外的标签**。如果是正常使用这个类，可以就当注解不存在，只看正面就行了，如果想要拿这个类来作特殊用途，就可以把牌反过来看看，看哪些标签上给了什么额外的信息。比如说，在组件化体系中，某个类被当做组件类的时候，可能需要额外的配置信息，如果没有注解的话，也可以在类上面放静态变量来实现，但不优雅。语言支持注解的话，写到注解上，就比较优雅了，类的实现中只写具体功能的实现。

```js
@Component({selector: 'foo'})
export class MyComponent {
  constructor(server:Server) {
      this.server = server;
  }
}
```

看这个例子里面那个@Component的一段，如果以MyComponent类的本身功能来说，有没有这句都不会影响这个类主体功能的实现，但我们可能在特定框架（Angular）中，需要把它与界面上某种DOM元素的选择器关联起来，就可以添加这么一句注解，然后，仅仅在Angular的框架中去读取并使用它即可，其他框架则无视之。此外，注解对代码文档生成的作用应当比注释更大，因为注解格式可以有编译期检查，而注释没有，根据注解，甚至还能在运行时生成文档。

##### 组件化

因为历史原因，Angular 1.x架构设计思路是很乱的，组件基本就是通过指令的定义来做的。自从Web Components规范出来，所有的框架都向之靠拢，Angular2.0开发小组也借鉴了react的一些思想，正式引入了组件的概念。组件基本上是用ES6的Class来定义的。代码如下：

```js
//从angualr2框架中引入基本的组件、核心指令、表单指令和启动命令
import {
	Component,
	CORE_DIRECTIVES,
    FORM_DIRECTIVES,
	bootstrap
} from 'angular2/angular2';

//从services/NameList引入模块作为数据源
import { NameList } from 'services/NameList';

//给SampleApp类加注解
@Component({
    selector: 'sample-app',//指定sampleApp的选择器
    viewProviders: [NameList],//指定数据源
    templateUrl: './templates/sample-app.html',//指定模板
    directives: [CORE_DIRECTIVES, FORM_DIRECTIVES]//指定组件会使用的核心指令
})
//定义组件类
class SampleApp {
    names: Array<string>;//定义类型为数组的names属性
    name: string;//定义类型为String的name属性
    constructor(service:NameList) {//定义构造函数：属性初始化
        this.names = service.getList();
        this.name = '';
 
    }
    //定义类函数
    addName(name:string) {
        this.names.push(name);
        this.name = "";
    }
	//定义类函数
    removeName(name:string) {
        var index = this.names.indexOf(name);
        index !== -1 && this.names.splice(index, 1);
    }
}

//导出模块
export function main(){
	bootstrap(SampleApp);//启动组件
}
```

以上是我找的一段例子代码，这段代码定义了一个sample-app元素组件。这段代码十分有代表性在html里这样使用：

```js
<section class="container sample-form-content">
        <sample-form>Loading</sample-form>
</section>
```

这段代码很好的讲解了Angular2.0中如何定义组件类、使用注解给组件类作标签以及其他基本用法。当然这是最基本的用法，还有很多深层次的东西都是没有用到，比如路由等等。

这里只说这些基本用法我想就足够了，在深层次的使用，我们再摸索中前进。更多详细的内容，大家还是移步汇智网的[这个课程](http://www.hubwiz.com/course/5599d367a164dd0d75929c76/)，我认为挺不错的。

#### 学习成本

要想把Angular2.0用起来，需要了解学习的东西还是比较多的。以下的东要会：

1. ES6语法
2. AtScript语法
3. 编译工具gulp或grunt

#### 对移动平台的支持

AngularJS1.x并没有针对移动平台做特别的优化，但随着近些年来移动设备的普及，让Angular2.0开发团队开始思考让Angular更适合于移动设备，所以在做设计的时候，他们提出“Angular2.0是面向移动设备的。”但很多现在都处于设计阶段，一切尚未盖棺定论。

#### 生态圈

因为现在Angular2.0推出的只是alpha版本，很多优秀的提议和理念也都在计划之中。所以Angular2.0的生态还很薄弱，就国内而言，目前国内大部分的生态还是Angular1.x的。不过倒是出了几个课程，书籍我至今还没看到过。由于Angular2.0的断层式升级，大家更多的是在议论它到底好不好，我们是不是还要做升级等等诸如此类的话题，真正的干活还是比较少的。

而作为对比，React在这方面就会好的多。以下就来说说React相关的东西。

### React

#### 基本特征和设计思路

自2013年Facebook发布以来，React吸引了越来越多的开发者，基于它的衍生技术，如React Native、React Canvas等也层出不穷。React最初来自Facebook内部的广告系统项目，项目实施过程中前端开发遇到了巨大挑战，代码变得越来越臃肿且混乱不堪，难以维护。于是痛定思痛，他们决定抛开很多所谓的“最佳实践”，重新思考前端界面的构建方式，于是就有了React。

>以简单直观、符合习惯的（idiomatic）方式去编程，让代码更容易被理解，从而易于维护和不断演进。这正是React的设计哲学。

##### 编写可预测，符合习惯的代码

所谓可预测（predictable），即容易理解的代码。在年初的React开发者大会上，React项目经理Tom Occhino进一步阐述React诞生的初衷，在演讲中提到，React最大的价值究竟是什么？是高性能虚拟DOM、服务器端Render、封装过的事件机制、还是完善的错误提示信息？尽管每一点都足以重要。但他指出，其实React最有价值的是声明式的，直观的编程方式。

软件工程向来不提倡用高深莫测的技巧去编程，相反，如何写出可理解可维护的代码才是质量和效率的关键。试想，一个月之后你回头看你写的代码，是否一眼就明白某个变量，某个if判断的含义；一个新加入的同事想去增加一个小小的新功能或是修复某个Bug，他是否对自己的代码有足够的信心不引入任何副作用？随着功能的增加，代码很容易变得越来越复杂，这些问题也将越来越严重，最终导致一份难以维护的代码。而React号称，新同事甚至在加入的第一天就能开始开发新功能。

那么React是如何做的呢？

##### 使用JSX直观的定义用户界面

JSX是React的核心组成部分，它使用XML标记的方式去直接声明界面，界面组件之间可以互相嵌套。但是JSX给人的第一印象却是相当“丑陋”。当下面这样的例子被第一次展示的时候，甚至很多人称之为“巨大的退步（Huge Step Backwards）”：

```js
var React = require(‘React’);
var message =
  <div class=“hello” onClick={someFunc}>
    <span>Hello World</span>
  </div>;
React.renderComponent(message, document.body);
```

将HTML直接嵌入到JavaScript代码中看上去确实是一件足够疯狂的事情。人们花了多年时间总结出的界面和业务逻辑相互分离的“最佳实践”就这么被彻底打破。那么React为何要如此另类？

模板出现的初衷是让非开发人员也能对界面做一定的修改。但这个初衷在当前Web程序里已完全不适用，每个模板背后的代码逻辑严重依赖模板中的内容和DOM结构，两者是紧密耦合的。即使做到文件位置的分离，实际上两者还是一体的，并且为了两者之间的协作而不得不引入很多机制和概念。以Angularjs的首页示例代码为例：

```js
<ul class="unstyled">
  <li ng-repeat="todo in todoList.todos">
    <input type="checkbox" ng-model="todo.done">
    <span class="done-{{todo.done}}">{{todo.text}}</span>
  </li>
</ul>
```

尽管我们很容易看懂这一小段模板的含义，但你却无法开始写这样的代码，因为你需要学习这一整套语法。比如说，你得知道有ng-repeat这样的标记的准确含义，其中的”todo in todoList.todos”看上去是repeat语法的一部分，或许还有其它语法存在；可以看到有{{todo.text}}这样的数据绑定，那么如果要对这段文本格式化（加一个formatter）该怎么做；另外，ng-model背后又需要什么样的数据结构？

现在来看React怎么写这段逻辑：

```js
//...
render: function () {
  var lis = this.todoList.todos.map(function (todo) {
    return  (
      <li>
        <input type="checkbox" checked={todo.done}>
        <span className="done-{todo.done}">{todo.text}</span>
      </li>);
  });
  return (
    <ul class="unstyled">
      {lis}
    </ul>
  );
}
//...
```

可以看到，JSX中除了另类的HTML标记之外，并没有引入其它任何新的概念（事实上HTML标记也可以完全用JavaScript去写）。Angular中的repeat在这里被一个简单的数组方法map所替代。在这里你可以利用熟悉的JavaScript语法去定义界面，在你的思维过程中其实已经不需要存在模板的概念，需要考虑的仅仅是如何用代码构建整个界面。这种自然而直观的方式直接降低了React的学习门槛并且让代码更容易理解。

##### 简化的组件模型：所谓组件，其实就是状态机器

组件并不是一个新的概念，它意味着某个独立功能或界面的封装，达到复用、或是业务逻辑分离的目的。而React却这样理解界面组件：

>所谓组件，就是状态机器

>React将用户界面看做简单的状态机器。当组件处于某个状态时，那么就输出这个状态对应的界面。通过这种方式，就很容易去保证界面的一致性。

>在React中，你简单的去更新某个组件的状态，然后输出基于新状态的整个界面。React负责以最高效的方式去比较两个界面并更新DOM树。

这种组件模型简化了我们思考的方式：对组件的管理就是对状态的管理。不同于其它框架模型，React组件很少需要暴露组件方法和外部交互。例如，某个组件有只读和编辑两个状态。一般的思路可能是提供beginEditing()和endEditing()这样的方法来实现切换；而在React中，需要做的是setState({editing: true/false})。在组件的输出逻辑中负责正确展现当前状态。这种方式，你不需要考虑beginEditing和endEditing中应该怎样更新UI，而只需要考虑在某个状态下，UI是怎样的。显然后者更加自然和直观。

组件是React中构建用户界面的基本单位。它们和外界的交互除了状态（state）之外，还有就是属性（props）。事实上，状态更多的是一个组件内部去自己维护，而属性则由外部在初始化这个组件时传递进来（一般是组件需要管理的数据）。React认为属性应该是只读的，一旦赋值过去后就不应该变化。关于状态和属性的使用在后续文章中还会深入探讨。

##### 每一次界面变化都是整体刷新

数据模型驱动UI界面的两层编程模型从概念角度看上去是直观的，而在实际开发中却困难重重。一个数据模型的变化可能导致分散在界面多个角落的UI同时发生变化。界面越复杂，这种数据和界面的一致性越难维护。在Facebook内部他们称之为“Cascading Updates”，即层叠式更新，意味着UI界面之间会有一种互相依赖的关系。开发者为了维护这种依赖更新，有时不得不触发大范围的界面刷新，而其中很多并不真的需要。React的初衷之一就是，既然整体刷新一定能解决层叠更新的问题，那我们为什么不索性就每次都这么做呢？让框架自身去解决哪些局部UI需要更新的问题。这听上去非常有挑战，但React却做到了，实现途径就是通过虚拟DOM（Virtual DOM）。

关于虚拟DOM的原理我在去年底的文章有过比较详细的介绍，这里不再重复。简而言之就是，UI界面是一棵DOM树，对应的我们创建一个全局唯一的数据模型，每次数据模型有任何变化，都将整个数据模型应用到UI DOM树上，由React来负责去更新需要更新的界面部分。事实证明，这种方式不但简化了开发逻辑并且极大的提高了性能。

以这种思路出发，我们在考虑不断变化的UI界面时，仅仅需要整体考虑UI的构成。编程模型的简化带来的是代码的精简和易于理解，也即React不断提到的可预测（Predictable）的代码，代码的功能一目了然易于理解。Tom Occhino在2015 React开发者大会上也分享了React在Facebook内部的应用案例，随着新功能被不断的添加到系统中，开发进度非但没有变慢，甚至越来越快。

##### 单向数据流动：Flux

既然已经有了组件机制去定义界面，那么还需要一定的机制来定义组件之间，以及组件和数据模型之间如何通信。为此，Facebook提出了Flux框架用于管理数据流。Flux是一个相当宽松的概念框架，同样符合React简单直观的原则。不同于其它大多数MVC框架的双向数据绑定，Flux提倡的是单向数据流动，即永远只有从模型到视图的数据流动。

![](/images/flux.jpg)

##### React思想的衍生：React Native, React Canvas等等

在前几天的Facebook F8开发者大会上，React Native终于众望所归的发布，它将React的思想延伸到了原生移动开发。它的口号是“Learn Once, Write Anywhere”，有React开发经验的开发人员将可以无缝的进行React Native开发。无论是组件化的思想，调试工具，动态代码加载等React具有的强大特性都可以应用在React Native。相信这会对以后的移动开发布局产生重要影响。

React对UI层进行了完美的抽象，写Web界面时甚至能够做到完全的去DOM化：开发者可以无需进行任何DOM操作。因此，这也让对UI层进行整体替换成为了可能。React Native正是将浏览器基于DOM的UI层换成了iOS或者Android的原生控件。而Flipboard则将UI层换成了Canvas。

React Canvas是Flipboard出品的一套前端框架，所有的界面元素都通过Canvas来绘制，infoQ之前也有文章对其进行了介绍。Flipboard追求极致的性能和用户体验，因此对浏览器的缓慢DOM操作深恶痛绝，不惜大刀阔斧彻底舍弃了DOM，而完全用Canvas实现了整套UI控件。有兴趣的同学不妨一试。

#### 学习成本

React的学习成本跟Angular2.0比起来就少的多了。要想上手react进行开发，只需要了解ES6的语法和React库的相关知识，然后就可以上手写helloworld了。如果要做react-native，那还要会用andriod ui和ios ui。

#### 对移动平台的支持

React-native的发布大大优化了React在移动平台上的便利性。React-native可以让我们使用react的开发模式直接调用ios UI或Android ui。

#### 生态圈

目前，React的生态已经相当完善。无论是论坛、书籍、个人博客或者各种开发者组织，都已经积累了相当的一大批人。不过国内有关react的书籍还是不多的。可以参考的几个论坛：

* [React中文技术站](http://www.reactjs-china.com/)
* [React](http://www.reactjs.cn/)
* [开源中国社区-React-Native](http://www.oschina.net/news/61214/11-react-native-ui-components)

### 结论

从学习成本、移动平台和生态上来说，目前我们部门更适合React。


此文只做抛砖引玉用途，若同事们有兴趣，请移步到更专业的论坛和社区。






