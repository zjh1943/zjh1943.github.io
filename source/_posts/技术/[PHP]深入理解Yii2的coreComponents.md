---
title: '[PHP]深入理解Yii2的核心组件'
date: 2016-03-03 11:12
categories: 
- 技术
tags: 
- yii2 
- coreComponents 
- php  
---

摘要：[上一篇文章](http://blog.bookbook.in/ji-zhu/-php-shen-ru-li-jie-yii2de-lifecycle)中，我们了解了`Appliction`的生命周期大概做了哪些事情，在其中我们提到核心组件`Appliction::coreComponents`但未深入讲解。我将在此文中挑选一些重要的组件详细讲解。


# coreComponents

`Application::coreComponents()` 是一个`Application`运行过程中必须的`component`，不管用户有没有在`config`文件中设置，这些组件都会创建和使用。  
所有在`config`文件中设置或者在`coreComponents()`中的组件，都不需要手动创建，通过`Application->get($componentId)`都可以自动创建。

`\yii\base\Application`的核心组件：

```php
public function coreComponents()
{
    return [
        'log' => ['class' => 'yii\log\Dispatcher'],
        'view' => ['class' => 'yii\web\View'],
        'formatter' => ['class' => 'yii\i18n\Formatter'],
        'i18n' => ['class' => 'yii\i18n\I18N'],
        'mailer' => ['class' => 'yii\swiftmailer\Mailer'],
        'urlManager' => ['class' => 'yii\web\UrlManager'],
        'assetManager' => ['class' => 'yii\web\AssetManager'],
        'security' => ['class' => 'yii\base\Security'],
    ];
}
```


`\yii\web\Application`的必要组件：

```php
public function coreComponents()
{
    return array_merge(parent::coreComponents(), [
        'request' => ['class' => 'yii\web\Request'],
        'response' => ['class' => 'yii\web\Response'],
        'session' => ['class' => 'yii\web\Session'],
        'user' => ['class' => 'yii\web\User'],
        'errorHandler' => ['class' => 'yii\web\ErrorHandler'],
    ]);
}
```



下文，我们将对`view,urlManager,assetManager,security,request,response,session,user`这几个组件做详细讲解。

# yii\web\Request

`Request`代表着一次HTTP请求，它封装了`$_SERVER`的大多数变量，并且对不同平台的浏览器做了兼容处理。同时，它还封装了一些方便的访问接口来访问诸如`$_POST`，`$_GET`，`$_COOKIES`，`_REST`的参数。



```php
class Request
{
    validateCsrfToken($token); //验证csrf码是否合法。
}
```

## 关于CSRF验证

服务器在接受一次请求，都会向前端发送两个字符串：一个在返回的HTML的`head`中；另一个在cookie中。这两个字符串是互相匹配的，可通过某种算法得到另一个。当用户发送非安全的请求时，需要在请求中把head中的那个字符串附带上，这样，服务器会判断这两个字符串是否匹配。



## Component::behaviors()

```php
class Component
    + __call() / 
        + $this->ensureBehaivors();
            + $this->attachBehaviorInternal();
            + $this->behaviors();


```

## ActionFilter

`ActionFilter`可以添加在`Module/Controller`上，监听`Module/Controller`的`beforeAction`和`afterAction`事件，做相应的处理。

```php
class ActionFilter:
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


# yii\web\Session




# \yii\web\User

`\yii\web\User`是用来处理用户验证和用户登录状态的类。

```php
//定义：
class User:
    + login(); //调用switchIdentify()，抛发一系列事件。
    + loginByAccessToken($token,$type); //如果findIdentityByAccessToken()成功，则login()
    + loginByCookie(); //如果Cookie中，id+authKey+duration都能对的上，则登陆成功，调用switchIdentify()
    + switchIdentify(); //setIdentify()，然后设置session和cookie

    + renewAuthStatus(); //根据SessionId获得Identify。如果已经过期，则logout()；如果enableAutoLogin，则loginByCookie()；如果autoRenewCookie，则renewIdentifyCookie()。
    + renewIdentityCookie(); //更新Cookie的过期时间为从当前开始。
    + getIdentify($autoRenew = true); //if $enableSession && $autoRenew then renewAuthStatus(). 最终返回Identify.

```

