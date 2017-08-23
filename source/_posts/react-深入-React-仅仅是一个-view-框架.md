---
title: '[react]深入 - React 仅仅是一个 view 框架'
date: 2017-08-23 10:18:14
tags:
- react
- reactjs
- redux
- mvc
categories:
- 技术
---


**React 框架的终极目的是：高效的用 js 生成HTML**。为了实现这个目的，react 维护了一个 virtual DOM ，然后再根据 virtual DOM 生成真正的 DOM，来达到「高效」的目的。说到高效，其实就是回答这两个问题：

1. When do I re-render? Answer: When I observe that the data is dirty.
2. How do I re-render efficiently? Answer: Using a virtual DOM to generate a real DOM patch.

[摸我了解更多关于 virtual DOM ][virtual_dom_stack_overflow]

下面，我们讲： React 为了实现如上的构想，为我们提供了哪些东西。

### jsx 标记语言

React 使用 jsx 标记语言。它跟 html 语言非常相似，使用起来非常方便。如果没有它，我们只能通过 `Rect.createElement( 'canvas', {…props} )` 的方式来生成一个 `cavans` 的描述，然后，react 会把这个描述「渲染」成真正的 Html `<canvas ...props />`。甚至，如果没有 react 框架，我们只能用 `let element = document.createElement('canvas', [, options])` 来创建一个 Html 元素。也许，你会觉得这三种方式代码量差不多。但是，当有多层嵌套的视图结构时，jsx 标记语言的优势就显现出来了。你可以试着把以下 jsx 语言「翻译」为后面两种写法。

```jsx
render() {
    const taskCount = this.props.taskCount;

    return (<div className="step-content create-data-content">
        <div className="info-bar" >
            <div>
                <div className="ui-font-secondary4">{'总共统计'}</div>
                <div className="ui-font-h2">{`${taskCount}条任务`}</div>
            </div>
            <div>
                {this.renderTabBar()}
            </div>
        </div>
        {this.renderContent()}
    </div>);
}
```

### 组件抽象 Component

React 提供了一个类 Component 作为所有组件的基类。它的意义是：
1. 它为我们创造了一个「组件」的概念，代表可重用的可视元素的最小单位。我们按照这个概念去开发，能很容易写出高复用性的视图代码。
2. 它提供了基本的生命周期回调函数，满足我们多样的需求。比如我们常用的: `componentDidMount`, `componentWillReceiveProps`, `componentWillUnmount`。
3. 提供了 props 接口，为父容器和子容器之间信息交流提供标准通道。
4. 提供 state，作为容器内部状态管理的落脚地。

### 视组组件的描述

每个组件都有一个 render 方法。

### 高效的翻译器：描述 => 

[virtual_dom_stack_overflow]: https://stackoverflow.com/questions/21109361/why-is-reacts-concept-of-virtual-dom-said-to-be-more-performant-than-dirty-mode "virtual-dom module"