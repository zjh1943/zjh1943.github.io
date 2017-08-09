---
title: '[react] React 新手必须知道的10件事'
date: 2017-08-08 19:58:24
tags:
- react
- reactjs
- props
- onChange
- 框架
- redux
categories:
- 技术
---

## 1. 一等公民：props & onChange 

React 最推荐的数据交互方式是：props & onChnage。在这种交互方式里：对于一个可视组件 `ComponentA`，用 `props` 来向它发送信息，而用 `onChange` 回调函数来接收 `ComponentA` 发送的信息。在程序世界里，我们更喜欢把上述「交互方式」称为「接口」，虽然这个「接口」不是我们在面向对象语言里的 `interface`，但是跟 `interface` 有着类似的功能。 我们暂且把这个「接口规范」取名为 「props & onChange 接口规范」。

React 还是给了另外一种方法来进行数据交互：ref & method。在这种交互方式里，我们通过 `<ComponentA ref={ r => this.refOfComponentA = r }` 的方式来获得 `ComponentA` 对象的引用，然后用 `this.refOfComponentA.someMethod()` 来向它发送信息。我们把这交互方式称为 「ref & method 接口规范」。在典型的客户端开发环境里（iOS、Android、Windows PC等），这种方式更为常见，并且对函数调用更加友好，更「像」程序语言。**但是，对于 React 新手，我们强烈不建议使用这种借口规范**，除非你对 React 整个机制非常了解，仍然想用它。因为它**严重破坏了 React 组件的一致性**。原因有：
1. React 的可视组件的层级结一般是在 jsx 文件中以一种类似于 html 的语言来表示的，这种表示方式既方便又直观，表达力很强。在这种特殊的 jsx 语言里，「props & onChange接口规范」很容易且自然的被遵守。而如果用 「ref & method接口规范」，你不得不跳转到很多行以外，才能明白信息的传递过程，既不利于代码编写，也不利于阅读。
2. 我们避免不了用 props 方式来进行数据传递。我们说「避免不了」，因为很多原因，在此仅列举两个：一、jsx 文件中，Html 内置元素只能通过 props 来传递参数；二、很多第三方库（如果我们在开发一个大型项目，必定有很多「轮子」不用自己造），也必须通过 props 来传递参数。所以，props 不可避免；而同时存在两种接口规范，是没有意义切容易出错的。
3. 第三个原因可能比较「经验化」。如果现在不能理解和认同，你听听就好；反正，当你使用过的优秀开源框架足够多，你肯定会明白的：当你新接触一个框架时，暂时抛弃自己以往的习惯，转而遵守它的语言规范，是最好的选择。原因很简单：
    1. 一个框架从出生到出名，一定有自己与众不同的框架思想，才能从其他同类型框架中脱引而出。时间的验证。是有意义的。
    2. 过于轻率的使用其他的编程思想，会多处碰壁；也不利于你真正了解此框架的优势和瓶颈。

## 2. React 仅仅是一个 view 框架

React 最擅长的事情是「把可视组件的描述高效的转换为HTML」。理解这句话我们需要理
    - redux
    - Rx

## 3. 多用 PureComponent。

## 4. 避免重复造轮子。

    - 原因：
        - 学习思想。
        - 高效。
        - react，state 和 props 的理解，有误导性。
    - 参考：
        - antd
        - bootstrap

## 5. Less state，more PureComponent. 

( container component VS show component)

## 6. 理解 React 的单向数据流


## 请添加用静态类型语言

如果是中大型项目，添加静态类型检查。

