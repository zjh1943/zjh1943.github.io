---
title: "[cocos2dx]深入了解几个代表性的类 (续)"
date: 2014-07-22
tags: 
- cocos2dx
- EventDispatcher
- Event
- EventListener
- fixedPriority
categories: 
- 技术
---


摘要: 此文对`cocos2d-x`引擎中最具代表性,最能体现框架结构的几个类做了简单的介绍, 包括`Director`,`Application`, `Renderer`, `EventDispatcher`, `Scheduler`. 对于这些类, 也只对关系主要流程的方法做了介绍, 略过了容错代码和其它细节. 主要目的是让大家快速的对`cocos2d-x`引擎有一个全面笼统的认识, 也方便快速定位问题.


---

# EventDispatcher

## EventDispatcher,EventListener,Event之间的关系

* `EventDispatcher`: 事件分发器, 相当于所有事件的中控中心. 管理着`EventListener`，当一个`Event`到来的时候决定`CallBack`的调用顺序。
* `Event` ( `EventTouch`, `EventKeyboard` 等), 具体的事件数据, 
* `EventListener` ( `EventListenerTouch`, `EventListenerKeyboard` 等 ): 建立了`Event`到`CallBack`的映射关系, `EventDispatcher` 根据这种映射关系调用对应的 `CallBack`.


## Event

`Event`有以下几种类型:

```
    enum class Type
    {
        TOUCH,
        KEYBOARD,
        ACCELERATION,
        MOUSE,
        FOCUS,
        CUSTOM
    };
```
`Event`最重要的属性就是`type`, 标识了它是那种类型的事件, 也决定了由哪个`EventListner`来处理它.


## EventListener
`EventListner`有以下几种类型:
```
    enum class Type
    {
        UNKNOWN,
        TOUCH_ONE_BY_ONE,
        TOUCH_ALL_AT_ONCE,
        KEYBOARD,
        MOUSE,
        ACCELERATION,
        FOCUS,
        CUSTOM
    };
```
除了`UNKNOWN`, 跟`Event::Type`相比,`Event::Type::TOUCH`会**同时**被两种类型的`EventListener`处理: `TOUCH_ONE_BY_ONE`和`TOUCH_ALL_AT_ONCE`. 这两种`EventListener`分别处理单点触摸事件和多点触摸事件. 多说几句: 假如一个`TouchEvent`事件中有多个触摸点, 那么类型为 `EventListener::Type::TOUCH_ONE_BY_ONE` 的 `EventListener` 会把这个事件分解成若干个单点触摸事件来处理. 而类型为 `EventListener::Type::TOUCH_ALL_AT_ONCE` 的 `EventListener` 就是来处理多点触摸的, 会一次处理它.
其它几种类型都是一一对应的, 即一种`Event::Type`的`Event`会被对应类型的`EventListener`接受.

## 存放 EventListener 的地方

在`EventDispatcher`中, 它把以上7种 `EventListener::Type` 类型的 `EventListner` 放到7个队列中. 也就是在这样一个字段中:

```
std::unordered_map<EventListener::ListenerID, EventListenerVector*> _listenerMap;
```
* `EventListener::ListenerID` : 每一种`EventListener::Type`有唯一的 `EventListener::ListenerID`. 其实通过这段代码 `typedef std::string ListenerID;` 可知: `EventListener::ListenerID` 就是简单 `string`, 就是一个名称而已.
* `EventListenerVector`: 顾名思义, 就是一个 `EventListener` 的向量容器. 相对于普通的向量容器, 它增加了`priority`管理功能. 

## EventListener的fixedPriority
简单来说, 每个 `EventListener` 有自己的 `fixedPriority` 属性, 它是一个整数. 

### EventListener的遍历顺序<a name="priority_of_event_listener"></a>
`EventDispatcher` 在抛发事件的时候, 会先处理 `Event` 的时候, 会优先遍历 `fixedPriority` 低的 `EventListener`, 调用它的 `CallBack`. 在某些条件下, 一个 `Event` 被一个 `EventListener` 处理之后, 会停止遍历其它的 `EventListener`. 反映到实战中就是: 你监听了某种事件, 这种事件也出发了, 但是对应的回调函数并没有被调用, 也就是被优先级更高的 `EventListener` 截获了. 

如果 `fixedPriority` 一样呢? 按照什么顺序? 

1. `fixedPriority` 为0, 这个值是专门为 Scene Object 预留的. 即, 默认情况下, 绝大多数继承自 `Node` 的对象添加的普通事件监听器, 其 `fixedPriority` 都为0. 此时, `Node` 的 `globalZOrder`决定了优先级, 值越大, 越先被遍历到, 即在显示层中层级越高, 越先接受事件. 这在ui响应逻辑中也是合理的.
2. `fixedPriority` 不为0, 那就按添加顺序. 
### Event在什么条件下会被优先级更高的EventListener截获?

1. 对于 `EventListenerTouchOneByOne`, 它有一个字段: `_needSwallow`, 当它为 `true` 的时候, 如果它接受了某个 `Event`, 优先级更低的 `EventListener` 就接受不到了. 可以用 `EventListenerTouchOneByOne::setSwallowTouches(bool needSwallow)` 来改变它.
2. 对于其它类型的 `EventLIstener`, 只有在显示调用了 `Event::stopPropagation()` 的时候, 才会中断遍历.

## 核心函数: EventDispatcher::dispatchEvent()
下面我们看看`EventDispatcher`最核心的函数: 
`void EventDispatcher::dispatchEvent(Event* event)`: 当有响应的事件到来的时候, 都会调用这个函数来通知监听了此事件的对象.


其实, 上面的介绍, 已经把这个函数里绝大部分逻辑都描述了,这里做一个最后的总结
事件抛发的简要流程如下:

1. 检查 `_listenerMap` 中所有的`EventListnerVector`, 如果哪个容器的 `EventListener` 优先级顺序需要更新, 则重新排序
2. 对于类型为 `Event::Type::TOUCH` 的事件, 则按照[EventListener的遍历顺序](#priority_of_event_listener)遍历所有的 `EventListener`. 只有接受了 `EventTouch::EventCode::BEGAN` 事件的 `EventListener`, 才会收到其他类型的 `EventTouch` 事件.
3. 对于其他类型的事件, 也按照[EventListener的遍历顺序](#priority_of_event_listener)的顺序遍历对应的`EventListener`.

## 总结
`Eventdispatcher` 中的其它函数, 主要功能都是 添加`EventListener`, 删除`EventListener`等, 不做详细介绍. 

总的来说, `Eventdispatcher` 是一个中转器:

1. 事件的产生模块儿, 只关心自己构造正确的 `Event`, 调用 `EventDispatcher::dispatchEvent(Event* event)` 交给 `EventDispatcher`.
2. 需要监听事件的模块儿, 只需调用 ` EventDispatcher::addEventListener(EventListener* listener)` (或者它的其它变种)来注册自己作为监听者.
3. 而 `EventDispatcher` 的作用是: 
    4. 把特定类型的 `Event` 送给对应类型的 `EventListener`.
    5. 对于同一种 `Event`, 规定了事件送达的优先级.

> Written with [StackEdit](https://stackedit.io/).




