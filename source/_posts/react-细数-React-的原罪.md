---
title: '[react] 细数 React 的原罪'
date: 2017-08-09 09:27:36
tags:
---

1. **Props & onChange 的原罪** 。「props & onChange 接口规范」它不是一个典型的「程序接口规范」。当你拿到一个可视组件的 ref，却没有类似 `setProps()` 这样的方法来改变其 `props`，你只能在 render() 方法中，通过 jsx 语言来设置其 `props`。这个让人感觉非常别扭（除非你这辈子接触的第一个客户端框架就是 React）。这是因为，在大多数客户端框架里，有一个视图类 `ComponentA`，`compA = new ComponentA()`， `compoA` 即是对这个可视组件的表示。而，在 React 世界里，compA.render() 返回的结果，才是这个可视组件的表示。
2. **Controller 和 View 融为一谈**。没有 Component 的概念，Component with state 很多时候提代替了 Controller 的角色。
3. 