---
tags: 
- yii2 
- component 
- action 
- filter
title: '[PHP]深入理解Yii2的基础类'
date: 2016-03-04 09:12
categories: 
- 技术
---

摘要：了解`Yii2`框架的基础支撑类。


# Component

`Component`是`Yii2`框架中非常重要的一个类。它主要实现了三个方面的功能：
1. `property`: 重写了`__get()`和`__set()`方法，可以直接调用`getter/setter`方法来读写属性，或者读写`behaviors`中的同名属性。
2. `event`:允许其他实例在`Component`上注册监听事件，当`Component::trigger()`被调用时，对应的回调函数会被调用。
3. `hehavior`:允许动态的为`Component`添加方法/接口。实现方式是：重写`__call()`方法，如果在`_behaviors`有同名的方法，也可以被调用。相当于扩展了`Component`的方法和属性。


## Component::behaviors()
```php
class Component
    + __call():
        $this->ensureBehaivors():
        then call;
    + ensureBehaviors():
        $this->attachBehaviorInternal();
        $this->behaviors();
```

# Behavior

两个功能：
1. 把自己的方法注入的`Component`中去，增加`Component`的功能。
2. 响应`$owner`抛发的事件，做对应的处理。

```php
class Behavior
    + attach($owner);
    + detach();
    + events();
```


# ActionFilter

`ActionFilter`可以添加在`Module/Controller`上，监听`Module/Controller`的`beforeAction`和`afterAction`事件，做相应的处理。

```php
class ActionFilter extends Behavior:
    + attach($owner): //当被添加到$owner上时，绑定$owner的'beforeAction'事件
        $owner->on('beforeAction',[$this, 'beforeFilter']);
    + beforeFilter($event):
        $event->isValid = $this->beforeAction($event->action);
        if $event->$isValid then
            $this->owner->on('afterAction',[$this,'afterFilter']);
        else
            $event->handled = true;
    + afterFilter($event);
```


# SercieLocator

ServiceLocator implements a [service locator](http://en.wikipedia.org/wiki/Service_locator_pattern).

 * To use ServiceLocator, you first need to register component IDs with the corresponding component definitions with the locator by calling `set()` or `setComponents()`.
 * You can then call `get()` to retrieve a component with the specified ID. The locator will automatically instantiate and configure the component according to the definition.

`ServiceLocator`重要的方法：

```php
class ServiceLocator:
    + set($id,$defination); //设置某个组件的定义
    + get($id); //如果未实例化，则根据定义新建。
```


# Controller

MVC的重要组成部分，相当于一个路由，没啥核心功能：

```php
class Controller:
    + run($route,$params); //执行Action
    + renderXXX(); //渲染输出结果，赋给Response
```

