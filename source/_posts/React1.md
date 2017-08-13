title: 学习笔记--对reactjs的初感
date: 2015-04-02 9:30:09
tags: 
- React
- 自己的笔记
categories: React

----------

PS:这是之前纯用手机打的啊~~~特此发出来，记录下。

### 题记

刚把react的官网的新手指南，做了一个小组件的demo应用。感觉react确实是把dom的逻辑放到了js里，把二者混在一起。**思想就是组件的思想，只不过每个大小组件都用react自己的库函数React.createClass搞成个对象，然后通过一层层父子组件的拼装，最后用ReactDOM.render(组件，要注入的dom节点)完成所有的渲染。**

### 概念理解
为了更好的理解概念，注意的几点：

1. 在发送请求前就提前改变对应组件的state状态，使应用感觉更快
2. 父子组件之间通过props传递数据，pros是不可变的：它们从父组件传递过来，“属于”父组件。比如方法或者data之类的
3. 为了交互，每个组件都有自己的状态state，这是组件私有的
4. 组件的状态用过在组件里this.setState()方法改变，注意，一旦组件的状态改变，react就会调用该组件的render方法进行重新渲染。
5. render方法依赖输入（this.state和this.pros），框架会确保渲染出来的ui界面总是与**输入幂等**的。
6. 在父组件里通过ref属性来读取子组件，ref相当于子组件的id一样。比如：
   ```javascript
    <input type="text" ref="myname">
    //父组件就可以通过this.ref.myname的方式访问到该节点。
    ```
7. 用回调函数作为pros可以从子组件传数据到父组件：我们在父组件的render方法体内会这样做，传递一个新的回调函数到子组件，绑定它到子组件的某个属性事件上。
8. getInitialState()在组件的生命周期中仅执行一次，用于设置组件的初始化状态。
9. componentDidMount()是一个组件渲染的时候被react自动调用的方法。这个方法是在组件渲染前最后一次改变组件状态的地方。可以在这里用轮训或websocket技术从后台读取数据保持状态最新。
10. react的生命周期

	![](/images/react_lifeCircle.png)