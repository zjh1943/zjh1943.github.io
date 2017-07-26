---
date: 2016-03-02 18:12
title: '[PHP]深入理解Yii2的生命周期'
categories: 
- 技术
tags:
- yii2 
- php 
- application 
- run
---


摘要：此文章为自己阅读Yii2框架代码的笔记，故不成为一篇完整的文章。如果你也在读Yii2的源码，或对Yii2很熟悉，可以作为参考。否则，可能很难懂。


# \yii\web\Application的生命周期

在Yii2中，Application的生命分两步：__contruct()和run()。下面为这两步过程的调用层级关系。每一次缩进代表函数调用栈的增进异步。

```php
class Application :
    + __contruct():
        preInit($config); //把coreComponents()添加到$config中去
        registerErrorHandler($config); 
        Component::__coustruct($config):
            Yii::configure($this,$config):
                ServiceLocator::set($id,$definition); //设置每个组件的defination
            Application::init():
                Application::bootstrap(); //每个extension,每个bootstrap,createObject() AND bootstrap().
    + run(): //处理request, 抛发一系列事件。
        handleRequest();
        reponse->send();

```


```php
class Application extends \yii\base\Application:
    + handleRequest():
        Request::resolve(): //根据URL解析出 [$route, params]
            UrlManager::parseRequest($this); //根据URL解析出 [$route, params]
        $response = runAction($route,$params); // inherate from Module::runAction():
            $controller=Module::createController($route, $params); //根据$route找到并创建Controller
            $response=$controller->runAction($actionId, $params):
                $action=Controller::createAction($id):
                    return new InlineAction($this,$methodName);
                $action->runWithParams($params):
                    $controller->bindActionParams(); //为method准备参数
                    call the method;
                return $result;
            return $response;
        return $response;
```



# coreComponents

Application::coreComponents() 是一个Application必须的component，不管用户有没有在config文件中设置，这些组件都会创建和使用。  
所有在config文件中设置或者在coreComponents()中的组件，都不需要手动创建，通过Application->get($componentId)都可以自动创建。

\yii\base\Application的核心组件：

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


\yii\web\Application的必要组件：coreComponents()

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




# \yii\BaseYii

`BaseYii`重要的方法：

```php
clss BaseYii:
    + createObject($type, array $params = []); //根据配置信息创建Object
    + configure($object, $properties); // copy properties
```





到此，我们大概了解了Application的生命周期里，主要做了哪些事情。当然，本文只介绍了概要流程。因为，涉及到具体的`coreComponents()`我们涉及的很少，而这些才是真正做事情的类，我们将在[下一篇文章](http://blog.bookbook.in/ji-zhu/-php-shen-ru-li-jie-yii2de-corecomponents)中介绍。
