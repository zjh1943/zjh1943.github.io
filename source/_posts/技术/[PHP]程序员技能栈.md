---
date: 2015-05-14 18:46
title: '[PHP]PHP程序员技能站'
categories: 
- 技术
tags:
- php
- mysql
- cache
- css
- xdebug
- composer
- pear
---


摘要：创业不息、折腾不止，今年跟朋友又折腾了一个新项目：[Color公寓](color520.com)。线上产品主要是一个在线网站，所以不得不把技术栈切换到**网站开发**模式。通过最近一段时间alpha版本的开发，对于PHP开发有一个全面而粗浅的了解。又因为面试需要，在这里做一下总结。


<a name="写在前面"></a>
## 写在前面

当我们谈到**PHP开发**，我们其实是在谈论`网站`，`数据库`，`缓存`，`session`，`负载均衡`等等等等一些列复杂技术的集合。所以，PHP的技能站也比一般得要长很多。

<a name="php基础"></a>
## PHP基础

在这里，你需要了解的是：

1. 在各个平台安装php并用php内置的web服务器成功的见到“Hello PHP”界面。
2. PHP函数式编程语法，面向对象语法，命名空间。
4. PHP标准库的熟悉。
5. 调试工具：起码要知道xdebug的使用方法。
6. 依赖包管理工具的安装、使用、原理，最常用的：composer,pear。
7. 语法性能及原理：
    8. 万能的array。用法、实现原理、使用注意事项。（关键字：Hash Table）。
    9. 字符串链接的效率。
    10. 弱类型的实现原理。（关键字：zval）
8. Http协议，Get/Post请求的不同。 

<a name="php原理"></a>
## PHP原理

<a name="设计理念及特点"></a>
### 设计理念及特点

1. 多线程模型，请求独立。
2. 弱类型语言。
3. 引擎（Zend）+组件（ext）的组合模式。

<a name="php四层体系"></a>
### PHP四层体系

1. **Zend引擎**：Zend整体用纯C实现，是PHP的内核部分，它将PHP代码翻译（词法、语法解析等一系列编译过程）为可执行opcode的处理并实现相应的处理方法、实现了基本的数据结构（如hashtable、oo）、内存分配及管理、提供了相应的api方法供外部调用，是一切的核心，所有的外围功能均围绕Zend实现。
2. **Extensions**：围绕着Zend引擎，extensions通过组件式的方式提供各种基础服务，我们常见的各种内置函数（如array系列）、标准库等都是通过extension来实现，用户也可以根据需要实现自己的extension以达到功能扩展、性能优化等目的（如贴吧正在使用的PHP中间层、富文本解析就是extension的典型应用）。
3. **Sapi**：Sapi全称是Server Application Programming Interface，也就是服务端应用编程接口，Sapi通过一系列钩子函数，使得PHP可以和外围交互数据，这是PHP非常优雅和成功的一个设计，通过sapi成功的将PHP本身和上层应用解耦隔离，PHP可以不再考虑如何针对不同应用进行兼容，而应用本身也可以针对自己的特点实现不同的处理方式。
4. **上层应用**：这就是我们平时编写的PHP程序，通过不同的sapi方式得到各种各样的应用模式，如通过webserver实现web应用、在命令行下以脚本方式运行等等。

<a name="mvc-框架"></a>
## MVC 框架

1. MVC理论。
1. PHP模板。
2. 常见的开源MVC框架，至少对其中一种有深入研究。 

<a name="oop、设计模式、重构"></a>
## OOP、设计模式、重构

1. 面向对象编程的基础：继承、封装、多态。
2. 常见的设计模式及应用场景。[参考](http://zh.wikipedia.org/wiki/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F_(%E8%AE%A1%E7%AE%97%E6%9C%BA))
3. 对代码的坏味道有敏锐的嗅觉，掌握基本的重构方法。


<a name="web前端技能"></a>
## Web前端技能

1. HTML/CSS/JavaScript 基础。
2. 缓存，Cookie，Session原理和使用。


<a name="mysql数据库技能"></a>
## MySQL数据库技能

1. MySQL安装和配置
1. MySQL增删改查基本语法
2. 数据库设计原则和常见的技巧
3. MySQL性能诊断和优化
4. 分布式数据库设计、数据库备份和恢复 

<a name="服务器架设"></a>
## 服务器架设

1. Linux常用命令。
2. dns,cdn,缓存,带宽等资源的合理利用。
3. nginx,apache安装和配置。
4. 图床的架设。（关键字：EvaCloudImage）

<a name="引用"></a>
## 引用

1. [PHP The Right Way](http://laravel-china.github.io/php-the-right-way/)
2. [PHP底层的运行机制与原理](http://nonfu.me/p/232.html)
3. [A Baseline for Front-End Developers](http://rmurphey.com/blog/2012/04/12/a-baseline-for-front-end-developers/)
4. [代码重构](http://zh.wikipedia.org/wiki/%E4%BB%A3%E7%A0%81%E9%87%8D%E6%9E%84)
5. [ 一次完整的HTTP事务是怎样一个过程？](http://linux5588.blog.51cto.com/65280/1351007)

<a name="广告"></a>
## 广告

好吧，下面是广告时间：

【Color国际青年公寓】是麦芒资产旗下的主打项目，一个面向租房市场的O2O项目。  简单来说：就是将现有出租房统一包装之后再分租给客户，统一规范管理；然后利用租客的天然流量优势，打造更加丰富的O2O生活闭环。面向群体是中等以上收入的年轻人。  
我们的创始团队来自于阿里巴巴、美国雅虎、完美世界、魅族等一线企业，线下也有10多年地产中介经验的人才加盟。核心创始人已多次创业。  
丰富的创业经验，千万级的天使融资，为您的未来提供了更有力的保证！  
只要您认同我们的文化，有舍我其谁的自信，就果断向我们砸简历吧：<hr@color520.com>，待遇绝对超过BAT。
详情见：[Color国际青年公寓](http://www.cnblogs.com/jhzhu/p/4377942.html)


