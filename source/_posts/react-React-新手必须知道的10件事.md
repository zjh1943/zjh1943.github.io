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

尽量用 props & onChange，不要用 ref 接口。[更多][first_class_people_props]

## 2. React 仅仅是一个 view 框架

React 只是一个视图框架，请尽量在 Component 里只做他擅长的事情。尽量写无状态的 Component。视图以外的事情，比如控制层、数据层、网络层，需要借助其他框架来完成。[更多][react_just_view]

## 3. 多用 PureComponent

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


[first_class_people_props]: /2017/08/23/react-React%E6%B7%B1%E5%85%A5-%E4%B8%80%E7%AD%89%E5%85%AC%E6%B0%91-props-onChange/ "一等公民 props & onChange"
[react_just_view]: /2017/08/23/react-%E6%B7%B1%E5%85%A5-React-%E4%BB%85%E4%BB%85%E6%98%AF%E4%B8%80%E4%B8%AA-view-%E6%A1%86%E6%9E%B6/
