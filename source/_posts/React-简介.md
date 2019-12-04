---
title: React 简介
date: 2019-05-24 16:22:32
tags: 前端
categories: 技术
---

React是Facrbook内部的一个JavaScript类库并开源，可用于创建Web用户交互界面。它引入了一种新的方式来处理浏览器DOM。那些需要手动更新DOM、费力地记录每一个状态的日子一去不复返。

<!-- more -->

> React是Facrbook内部的一个JavaScript类库并开源，可用于创建Web用户交互界面。它引入了一种新的方式来处理浏览器DOM。那些需要手动更新DOM、费力地记录每一个状态的日子一去不复返。React使用很新颖的方式解决了这些问题。你只需要声明地定义各个时间点的用户界面，而无序关系在数据变化时，需要更新哪一部分DOM。在任何时间点，React都能以最小的DOM修改来更新整个应用程序。

**React主要有四个主要概念构成，下面分别来简单介绍一下**


#### Virtual DOM

- 之所以引入虚拟DOM，一方面是性能的考虑。Web应用和网站不同，一个Web应用 中通常会在单页内有大量的DOM操作，而这些DOM操作很慢

- 在React中，应用程序在虚拟DOM上操作，这让React有了优化的机会。简单说， React在每次需要渲染时，会先比较当前DOM内容和待渲染内容的差异， 然后再决定如何最优地更新DOM

- 除了性能的考虑，React引入虚拟DOM更重要的意义是提供了一种一致的开发方式来开发服务端应用、Web应用和手机端应用

![avatar](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/20190106120609.png)

> 因为有了虚拟DOM这一层，所以通过配备不同的渲染器，就可以将虚拟DOM的内容 渲染到不同的平台。而应用开发者，使用JavaScript就可以通吃各个平台了。相当棒的思路！且**虚拟DOM是内存数据，性能是极高的，而对实际DOM进行操作的仅仅是 Diff部分，因而能达到提高性能的目的**

#### React组件
对于React而言，则完全是一个新的思路，开发者从功能的角度出发，将**UI分成不同的组件，每个组件都独立封装**。
在React中，你按照界面模块自然划分的方式来组织和编写你的代码，对于评论界面而言，整个UI是一个通过小组件构成的大组件，每个组件只关心自己部分的逻辑，彼此独立。

![avatar](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/component1.png)
![avatar](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/20190106120653.png)

- 组件化开发特性:
React认为一个组件应该具有如下特征：

1.**可组合（Composeable）**：一个组件易于和其它组件一起使用，或者嵌套在另一个组件内部。如果一个组件内部创建了另一个组件，那么说父组件拥有（own）它创建的子组件，通过这个特性，一个复杂的UI可以拆分成多个简单的UI组件；

2.**可重用（Reusable）**：每个组件都是具有独立功能的，它可以被使用在多个UI场景；

3.**可维护（Maintainable）**：每个小的组件仅仅包含自身的逻辑，更容易被理解和维护；

4.**可测试（Testable）**：因为每个组件都是独立的，那么对于各个组件分别测试显然要比对于整个UI进行测试容易的多。

- 组件定义
在React中定义一个组件也是相当的容易，组件就是一个 实现预定义接口的JavaScript类：

1. 组件渲染

> ReactDOM.render 是 React 的最基本方法，用于将模板转为 HTML 语言，并插入指定的 DOM 节点。

```javascript
ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('example')
);
```
而这个方法， 必须而且只能返回一个有效的React元素。这意味着，如果你的组件是由多个元素构成的，那么你必须在外边包一个顶层 元素，然后返回这个顶层元素。比如我们创建一个布局组件：
```javascript
render:function(){
    return React.createElement(
        "div",null,
        React.createElement("div",null,"header"),
        React.createElement("div",null,"content"),
        React.createElement("div",null,"footer")
    );
}
```
2. ES5方式定义组件
```javascript
"use strict";
var HelloMessage = React.createClass({
  displayName: "HelloMessage",
    render: function render() {
        return React.createElement(
            "div",
            null,
            "Hello ",
            this.props.name
        );
    }
});
```

3. ES6方式定义组件
```javascript
import './Hello.css';
import './Hello.scss';
import React, {Component} from 'react';
// 内联样式
let style={
    backgroundColor:'blue'
}
export default class Hello extends Component {
    constructor(props) {
        super(props);
        this.state = { count: 'es6'};
    }
    render() {
        return (
            <div>
                <h1 style={style}>Hello world{this.state.count}</h1>
                <br/>
                <image/>
            </div>
        )
    }
}
```

4. JSX方式定义组件
```javascript
var HelloMessage = React.createClass({
  render: function() {
    return <div>Hello {this.props.name}</div>;
  }
});
ReactDOM.render(<HelloMessage name="John" />, mountNode);
```

#### Jsx语法
- 什么是jsx
> 在用React写组件的时候，通常会用到JSX语法，粗看上去，像是在Javascript代码里直接写起了XML标签，实质上这只是一个语法糖，每一个 XML标签都会被JSX转换工具转换成纯Javascript代码，当然你想直接使用纯Javascript代码写也是可以的，只是**利用JSX，组件的结构和组件之间的关系看上去更加清晰**

- Jsx语法使用
HTML 语言直接写在 JavaScript 语言之中，不加任何引号，这就是 JSX 的语法，它允许 HTML 与 JavaScript 的混写。
```javascript
var names = ['Alice', 'Emily', 'Kate'];
ReactDOM.render(
  <div>
  {
    names.map(function (name) {
      return <div>Hello, {name}!</div>
    })
  }
  </div>,
  document.getElementById('example')
);
```
**上面代码体现了 JSX 的基本语法规则：遇到 HTML 标签（以 < 开头），就用 HTML 规则解析；遇到代码块（以 { 开头），就用 JavaScript 规则解析。** 

---

JSX 允许直接在模板插入 JavaScript 变量。如果这个变量是一个数组，则会展开这个数组的所有成员
```javascript
var arr = [
  <h1>Hello world!</h1>,
  <h2>React is awesome</h2>,
];
ReactDOM.render(
  <div>{arr}</div>,
  document.getElementById('example')
);
```
上面代码的arr变量是一个数组，结果 JSX 会把它的所有成员，添加到模板，运行结果如下。

![helloword](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/20190106121530.png)

#### Data Flow（单向数据流）

- 单向数据流
先来了解一下 Flux 的核心“单向数据流“怎么运作的：
Action -> Dispatcher -> Store -> View
更多时候 View 会通过用户交互触发 Action，所以一个简单完整的数据流类似这样：
![Flux](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/20190106121955.png)

整个流程如下：

1. 首先要有 action，通过定义一些 action creator 方法根据需要创建 Action 提供给 dispatcher
2. View 层通过用户交互（比如 onClick）会触发 Action
3. Dispatcher 会分发触发的 Action 给所有注册的 Store 的回调函数
4. Store 回调函数根据接收的 Action 更新自身数据之后会触发一个 change 事件通知 View 数据更改了
5. View 会监听这个 change 事件，拿到对应的新数据并调用 setState 更新组件 UI

所有的状态都由 Store 来维护，通过 Action 传递数据，构成了如上所述的单向数据流循环，所以应用中的各部分分工就相当明确，高度解耦了。
这种单向数据流使得整个系统都是透明可预测的。
