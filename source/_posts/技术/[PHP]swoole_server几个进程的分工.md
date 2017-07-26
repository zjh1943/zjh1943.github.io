---
date: 2015-08-26 15:33
title: '[PHP]swoole_server进程的分工'
categories: 
- 技术
tags: 
- php
- swoole
- swoole_server
- worker
- manager
- process
- 进程
- 线程
---



摘要：Swoole是一个PHP语言的高性能网络通信框架，提供了PHP语言的异步多线程服务器，异步TCP/UDP网络客户端，异步MySQL，数据库连接池，AsyncTask，消息队列，毫秒定时器，异步文件读写，异步DNS查询。强大的功能，由背后若干个分工明确的进程来实现，这里详细介绍下几个进程的分工，以便入门者更快速的理解Swoole框架。

## 目录
[TOC]

## Swoole简介 

[Swoole官网](http://www.swoole.com/)

### Swoole：重新定义PHP
Swoole：PHP语言的高性能网络通信框架，提供了PHP语言的异步多线程服务器，异步TCP/UDP网络客户端，异步MySQL，数据库连接池，AsyncTask，消息队列，毫秒定时器，异步文件读写，异步DNS查询。
Swoole虽然是标准的PHP扩展，实际上与普通的扩展不同。普通的扩展只是提供一个库函数。而swoole扩展在运行后会接管PHP的控制权，进入事件循环。当IO事件发生后，swoole会自动回调指定的PHP函数。


### 功能展示代码片段

#### TCP Server

```php
$serv = new swoole_server("127.0.0.1", 9501);
$serv->set(array(
    'worker_num' => 8,   //工作进程数量
    'daemonize' => true, //是否作为守护进程
));
$serv->on('connect', function ($serv, $fd){
    echo "Client:Connect.\n";
});
$serv->on('receive', function ($serv, $fd, $from_id, $data) {
    $serv->send($fd, 'Swoole: '.$data);
    $serv->close($fd);
});
$serv->on('close', function ($serv, $fd) {
    echo "Client: Close.\n";
});
$serv->start();
```



#### TCP Client

```php
$client = new swoole_client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_ASYNC);
//设置事件回调函数
$client->on("connect", function($cli) {
    $cli->send("hello world\n");
});
$client->on("receive", function($cli, $data){
    echo "Received: ".$data."\n";
});
$client->on("error", function($cli){
    echo "Connect failed\n";
});
$client->on("close", function($cli){
    echo "Connection close\n";
});
//发起网络连接
$client->connect('127.0.0.1', 9501, 0.5);
```
更多代码片段请见[swoole官网](http://www.swoole.com/)。


## 主要进程分析

### Master进程

Master进程主要用来保证Swoole框架机制的运行。它会创建几个功能性的线程：

* Reactor线程:就是真正处理TCP连接，收发数据的线程。swoole的主线程在Accept新的连接后，会将这个连接分配给一个固定的Reactor线程，并由这个线程负责监听此socket。在socket可读时读取数据，并进行协议解析，将请求投递到Worker进程。在socket可写时将数据发送给TCP客户端。
* Master线程(主线程）: 负责：Accept新的连接、UNIX PROXI信号处理、定时器任务。
* 心跳包检测线程：（略）
* UDP收包线程：（略）

### Manager进程

swoole中Worker/Task进程都是由Manager进程Fork并管理的。

* 子进程结束运行时，manager进程负责回收此子进程，避免成为僵尸进程。并创建新的子进程
* 服务器关闭时，manager进程将发送信号给所有子进程，通知子进程关闭服务
* 服务器reload时，manager进程会逐个关闭/重启子进程

>为什么不是Master进程呢，主要原因是Master进程是多线程的，不能安全的执行fork操作。

### Worker进程

* 接受由Reactor线程投递的请求数据包，并执行PHP回调函数处理数据
* 生成响应数据并发给Reactor线程，由Reactor线程发送给TCP客户端
* 可以是异步非阻塞模式，也可以是同步阻塞模式
* Worker以多进程的方式运行

>Swoole提供了完善的进程管理机制，当Worker进程异常退出，如发生PHP的致命错误、被其他程序误杀，或达到max_request次数之后正常退出。主进程会重新拉起新的Worker进程。 Worker进程内可以像普通的apache+php或者php-fpm中写代码。不需要像Node.js那样写异步回调的代码。


### Task进程

* 接受由Worker进程通过swoole_server->task/taskwait方法投递的任务
* 处理任务，并将结果数据返回给Worker进程
* 完全是同步阻塞模式
* Task以多进程的方式运行

Task进程的全称是task_worker进程，是一种特殊的worker进程。所以`onWorkerStart`在task进程中也会被调用。当`$worker_id >= $serv->setting['worker_num']`时表示这个进程是task_worker，否则，代表此进程是worker进程。

## 进程与事件回调的对应关系

### Master进程内的回调函数

```php
onStart
onShutdown
onMasterConnect
onMasterClose
onTimer
```

### Worker进程内的回调函数

```php
onWorkerStart
onWorkerStop
onConnect
onClose
onReceive
onTimer
onFinish
```

### Task进程内的回调函数

```php
onTask
onWorkerStart
```

### Manager进程内的回调函数

```php
onManagerStart
onManagerStop
```



