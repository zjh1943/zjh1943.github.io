---
title: '[react] React 新手必须知道的 N 件事'
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

1. 尽量用 props & onChange，不要用 ref 获取引用然后调用方法。详情参考博客：[一等公民 props & onChange][first_class_people_props]
2. React 只是一个视图框架，请尽量在 Component 里只做他擅长的事情。尽量写无状态的 Component。视图以外的事情，比如控制层、数据层、网络层，需要借助其他框架来完成。详情参考博客：[React 仅仅是一个 view 框架][react_just_view]
4. 避免重复造轮子。react 有丰富的第三方 Component & Utils & everything。写任何组件前请先看看这里：[awesome-react-components](https://github.com/brillout/awesome-react-components)
5. Less state，more PureComponent. 深刻理解和区分 Presentational Component 和 Container Component。前者决定组件如何显示，更关心对已知数据的展示，大量操作 dom，很少有 state；后者更关心数据的获取和更新，关心交互操作，很少直接操作 dom，可能包括很多 state。详解请参考：[Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)
6. 理解 React 的单向数据流，了解他的优势和局限。详情：[React 组件数据流 && 组件间沟通](https://segmentfault.com/a/1190000006831820)
7. 如果是中大型项目，请添加静态类型检查。TypeScript 或者其他类似的解决方案。因为 js 太自由了，很容易对一个对象增加和删除一个字段。如果，恰好其他人需要看这段代码，可能需要追溯好几条街，阅读7、8个代码文件，才知道某个对象的一个对象是从哪里来，结构如何。在多人配合的项目中，这种「自由」带来的便利，远远抵不上代码可读性降低带来的阻碍。


[first_class_people_props]: /2017/08/23/react-React%E6%B7%B1%E5%85%A5-%E4%B8%80%E7%AD%89%E5%85%AC%E6%B0%91-props-onChange/ "一等公民 props & onChange"
[react_just_view]: /2017/08/23/react-%E6%B7%B1%E5%85%A5-React-%E4%BB%85%E4%BB%85%E6%98%AF%E4%B8%80%E4%B8%AA-view-%E6%A1%86%E6%9E%B6/
