---
date: 2016-03-01
title: '[RESTful]最优雅的RESTful设计'
tags: 
- restful
- rest
- api
- http
- get
- post
- put
categories: 
- 技术
---

摘要：RESTful设计，着重点在于理解四个概念：资源（resources)、资源的命名(naming)、资源的表达方式(representations)、资源间的链接（Link)。一个设计是否足够RESTful，取决于四个方面的属性：addressability, statelessness,connectedness and the uniform interface。


## Web is Simple

REST的全称是Representational State Transfer，翻译过来是“代表性的状态转换”，虽然不知道在说啥，但这并不重要，不必过分关注它晦涩的名字。

REST的初衷，是想从Method和URI两个角度统一Web世界。Method，就是HTTP的那几个固定的方法，GET、PUT、DELETE、POST几兄弟，他们的意义在正常人眼里都是固定的。URI就不多说了，就是资源的门牌号码。我们假设，Web世界里所有的服务提供商都用这一套不仅人类可阅读，而且机器可识别的接口，该有多美好。

Web是基于HTTP构建的，它其实是一个最简单的协议。想当初HTTP 还是0.9版本的时候，就是一个静态文件读取协议，那么的单纯：

| Client Request | Server Response |
|----------------|-----------------|
| GET /hello.txt | Hello, world!   |

在这里，我们看到了HTTP最重要的两个品质：adressability and statelessness。基于他们，HTTP得飞黄腾达、光宗耀祖。


## Big Web Services Are Not Simple

常见的 RPC-style 服务，就是把Method、地址、参数都统统塞进HTTP的body里，用envelope封装起来。人们很难看到，这个方法是什么意思，参数是什么意思；也不知道这个方法有没有 side effect, 是否数据安全（safe）；是否有状态（stateness)，在什么样的前置条件下才能得到正确的结果。。。

这一切，都让Web变得不单纯鸟。


## How To Design And Implement RESTful Web Service

## What Is Means To Be RESTful


## Why Web Service Should be More RESTful


