---
title: "[node.js] LeanCloud 云引擎的学习和部署"
date: 2016-09-26 23:14:24
tags:
- nodejs
- npm
- express
- lean
- avoscloud
categories: 
- 技术
---

摘要：本文是一个流水文，主要记录我在 LeanCloud 云平台上使用 node.js 搭建web服务器的过程。其中遇到的困难和需要记忆的知识点，在此做一个汇总。


# 安装node.js

一定要是最新版本，否则各种 warn 和 err。
安装的时候 warn 一定要处理，否则会导致意想不到的 err。

# 安装 avoscloud 工具

一定要使用最新版本，因为 leanengine 2.0 和 3.0 版本的项目结构发生了变化，老版本的命令行工具不能正确找到对应的文件，无法启动。

# node.js 学习

node.js 是一个异步事件驱动的 web 框架。

```js
const http = require('http');

const hostname = '127.0.0.1';
const port = 1337;

var server = http.createServer();
server.on('request', (req, res) => {
    var method = request.method;
    var url = request.url;

    req.on('data', function(chunk) {
        body.push(chunk);
    }).on('end', function() {
        body = Buffer.concat(body).toString();
        // at this point, `body` has the entire request body stored in it as a string
    }).on('error', function(err) {
        // This prints the error message and stack trace to `stderr`.
        console.error(err.stack);
    });

    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello World\n');
});
server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

# express 学习

## Express的文件结构

```js
.
├── app.js
├── bin
│   └── www
├── package.json
├── public
│   ├── images
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── routes
│   ├── index.js
│   └── users.js
└── views
    ├── error.jade
    ├── index.jade
    └── layout.jade
```

## routing

Route definition takes the following structure:
```js
app.METHOD(PATH, HANDLER)
```

Where:

* `app` is an instance of express.
* `METHOD` is an HTTP request method.
* `PATH` is a path on the server.
* `HANDLER` is the function executed when the route is matched.


### PATH 还支持正则表达式：

This route path will match abcd, abxcd, abRANDOMcd, ab123cd, and so on.

```js
app.get('/ab*cd', function(req, res) {
  res.send('ab*cd');
});
```

This route path will match /abe and /abcde.

```js
app.get('/ab(cd)?e', function(req, res) {
 res.send('ab(cd)?e');
});
```


### express.Router

利用 express.Router 中间件可以创建一个子应用来响应特定的请求。
以下的代码，表示把所有 `/birds` 路径下的请求都托付给 `birds.js` 文件处理。

```js
// birds.js

var express = require('express');
var router = express.Router();

// middleware that is specific to this router
router.use(function timeLog(req, res, next) {
  console.log('Time: ', Date.now());
  next();
});
// define the home page route
router.get('/', function(req, res) {
  res.send('Birds home page');
});
// define the about route
router.get('/about', function(req, res) {
  res.send('About birds');
});

module.exports = router;

```

app 中：

```js
//app.js
var birds = require('./birds');
app.use('/birds', birds);
```


## middleware

<http://expressjs.com/en/guide/using-middleware.html>

中间件可以应用在 app 上，也可以应用在 router 上，可以使所有的请求都必须先经过中间件，最后到达具体的请求处理函数。下面是一个记录请求时间的中间件，可以记录任何请求的请求时间：

```js
var express = require('express');
var app = express();

var requestTime = function (req, res, next) {
  req.requestTime = Date.now();
  next();
};

app.use(requestTime);

app.get('/', function (req, res) {
  var responseText = 'Hello World!';
  responseText += 'Requested at: ' + req.requestTime + '';
  res.send(responseText);
});

app.listen(3000);
```

express有一些自带的中间件，也有很多扩展中间件，最常用的是 static 中间件，用来访问静态资源：

```js
var options = {
  dotfiles: 'ignore',
  etag: false,
  extensions: ['htm', 'html'],
  index: false,
  maxAge: '1d',
  redirect: false,
  setHeaders: function (res, path, stat) {
    res.set('x-timestamp', Date.now());
  }
}

app.use(express.static('public', options));
app.use(express.static('uploads'));
app.use(express.static('files'));

```

## template

### jade

```js
//app.js
app.set('view engine', 'jade');
app.get('/', function (req, res) {
  res.render('index', { title: 'Hey', message: 'Hello there!'});
});
```

```js
//index.jade
html
  head
    title= title
  body
    h1= message
```

jade 语法非常简单：<http://jade-lang.com>

### ejs

略

