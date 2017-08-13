title: 自己对node事件驱动的理解
date: 2014-06-21 9:15:49
tags: 
- nodejs
- 事件驱动
categories: nodeJS

---

### 题记
>觉得Node的书里，貌似只有朴灵的《深入浅出NodeJS》一直在讲比较深刻的东西，章节顺序安排的也比较合理（毕竟是从博客转过来的，从另一个角度也放映了老师当年的研究路线），之前也是关注他的博客，所以我的学习路线基本是按照这本书来的。

今天了解了下Node的异步编程。因为NodeJS的单线程、非阻塞（外）、异步执行（内）和事件驱动的特点，最终决定了他的编程方式很大程度采用了事件发布/订阅模式，可以说就是基于事件发布/订阅模式，node才大放光彩。
<!-- more -->

### 事件发布/订阅
以下是对这个模式的描述：

1. node中大半的对象都继承了events模块（`EventEmitter类`）
2. 由于第1点导致node中的众多模块都有addListener/on()、once()、removeListener()、removeAllListeners()和emit()等基本事件监听模式的方法的实现。但比DOM简单，不存在事件冒泡，也就没有preventDefault()、stopPropagation()和stopImmediatePropagation()等控制事件传递的方法。
3. 采用事件发布/订阅模式能够`解耦业务逻辑`、有`黑盒`的特点
4. 也是一种`钩子机制`（利用钩子暴露出内部数据或状态给外部调用），让开发者只需要把注意力放在事件点上即可。比如http模块的error、data、end等业务事件点，其实react的各种生命周期事件也是钩子函数。
5. node为了健壮性，EventEmitter对象会在运行时出错时，先看用户是否对`error`事件设置了监听器，如果有，则执行，如果没有则抛出异常，如果程序员也没有处理该error,则node线程就退出了。
6. node设定最多对一个时间设置`10个监听器`，要是多了，会报错，可以通过setMaxListeners(0)移除限制。

### 高级用法
更高级的Promise/Deffered以及各种库（bluebird、Q等）也都是以它为基础(在定义Promise的时候首先继承EventEmitter对象)，无非就是`设置成功状态的监听和失败状态的监听`，然后`在resolve()的时候触发成功事件的监听`，`reject()的时候触发失败状态的监听事件`。事件是node能够完成异步的很重要的一环。所以“事件是头等重要的”“一切以事件先行”。膜拜。