---
title: '[react] 细数 React 的原罪'
date: 2017-08-09 09:27:36
tags:
- react
- reactjs
- redux
- mvc
categories:
- 技术
---

1. **Props & onChange 的原罪** 。「props & onChange 接口规范」它不是一个典型的「程序接口规范」。
    1. 当你拿到一个可视组件的 ref，却没有类似 `setProps()` 这样的方法来改变其 `props`，你只能在 render() 方法中，通过 jsx 语言来设置其 `props`。这意味着父元素必须保存并维护这个 props 对应的值，而更多时候，父容器只是想**一次性**的设置一个值，然后就走，让以后的事情交给子元素自己去维护。当然，你也可以通过 ref 来获取子元素的应用，然后调用其成员方法来达到**一次性**改变属性的目的，但又会带来其他问题，请看下一条。
    2. 在大多数客户端框架里，有一个视图类 `ComponentA`，`compA = new ComponentA()`， `compoA` 即是对这个可视组件的表示。而，在 React 世界里，`compA.render()` 返回的结果（结果的结构参见下图），才是这个可视组件的表示。这种不同带来的后果是，我们很难对已经渲染过的节点进行后续更改。当然 React 提供了 `React.cloneElement(element,props)` 接口来改变，但是这个接口怎么看都不像一种**正派**的接口，而像是。。。`hack`。因为这种改变，其他人时候很难看懂的。换句话说，我们不是通过一个明确定义的接口，而是“随性的”，“肆无忌惮”的修改了内存。![函数返回结果](/idx/react_render_result_object.png)
    3. React 用 `props & PropType` 的方式重新定义了一种接口规范，定义了一个组件的输入和输出。输入 `props` 一般是 `string`, `boolean`, `number`, `object`；输出参数一般是 `function`。`PropType` 定义了每个 `props` 的取值范围、 是否必填等信息。这整个一套机制是跟面向对象语言里的 `interface` 的功能是高度重合的，也就是说，React 重新发明了轮子。并且这个轮子有自己很大的局限性：他不是能自表达的。什么意思呢？在典型的面向对象语言里（以 java 为例）, 假设有一下代码：
    ```java
    inteface IRunable{
        run();
        getSpeed();
    }

    function clone(IRunable){}
    ```
    这段代码里的 `clone` 函数的第一个参数必须满足 `IRunable` 接口，否则编译器会抛出编译错误。但是，切换到 `props` 接口规范，假设我们定义了一个对象的接口: 
    ```js
    const SomePropType = PropType.shape({
        name: PropType.string.isRequired,
        uuid: PropType.stirng.isRequired,
        onChange: PropType.func.isRequired,
    });

    function clone(element){}. // element 是一个必须满足 SomePropType 接口的元素。
    
    const AnotherPropType = PropType.shape({
        elment: ???how??? // 一个必须满足 SomePropType 接口的元素。
    })
    ```
    你没办法定义一个 `clone` 方法，限制它的第一个参数必须是满足某个 `PropType` 定义的组件对象；甚至，你也没办法定义一个接口 `AnotherPropType`，使得它的某个属性满足 `SomePropType`。
2. **Controller 和 View 融为一谈**。React 本身是一个 View 层的工具，他最擅长的是把数据 `render()` 成 dom 元素。在 Android SDK 里，有一个东西跟它的功能很相似： layout xml。但是在 React 的设计理念里，没有对 view 和 controller 的明确的概念的区分。我们可以在 Component 里做 view 层该做的事情（根据给定数据显示界面），也可以做 controller 应该做的事情（处理交互动作、处理网络请求）。以至于有很多博客来“教” React 的开发者来明确分这两个概念，参考博客[Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)。当然你可以说，这是开发者自身素质和抽象能力的体现，但是作为一个使用量如此巨大的框架，官方居然没有为用户构造和区分这个概念，还要用户自己去**发现和总结**，不得不说，是一大黑点。
