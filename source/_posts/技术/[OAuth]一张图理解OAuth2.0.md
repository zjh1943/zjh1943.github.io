---
title: '[OAuth]一张图理解OAuth2.0'
date: 2016-02-29 14:13 
description: 用一张详细解剖图阐述了OAuth的验证流程，希望可以帮到大家理解。
tags: 
- OAuth
- PHP
- Server
categories: 
- 技术
---


如果你想全面的了解OAuth，请先参考阮一峰的博客[理解OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)。阮先生的博客全面精炼、条理清楚，但OAuth的整个流程涉及到多端，所以理解起来需要很专注。此篇博文只是针对阮先生的博文中提到的OAuth协议的实现方式中的“授权码模式”进行了傻瓜似的图解。

图中的名词解释：

* 浏览器（User Agent）：表示用户的浏览器。
* Client（应用服务器）：表示用户当前访问的服务器。假设我们正在访问client.com，有一个功能需要注册才能用。在注册界面，我发现，可以用auth.com的账号来登录，这样，client.com就能获取到auth.com中关于我的很多信息了，比如头像、邮箱、ID等。而恰好我以前注册过auth.com网站的账号。此时，client.com所在的服务器就代表应用服务器。
* Auth Server：在上个例子中，表示client.com的服务器中负责OAuth授权的部分。
* Resource Server：在上个例子中，表示client.com服务器中负责提供用户信息的部分。


<iframe id="embed_dom" name="embed_dom" frameborder="0" style="border:1px solid #000;display:block;width:100%; height:1000px;" src="https://www.processon.com/embed/56d3d11fe4b0d06b7057a9ff"></iframe>