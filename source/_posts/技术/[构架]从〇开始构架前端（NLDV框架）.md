---
date: 2014-08-21
title: '[构架]从〇开始构架前端（NLDV框架）'
tags: 
- 框架
- 设计模式
- NLDV
categories: 
- 技术
---


摘要：一个普通应用，大到微信， 小到豆瓣FM，必不可少的都包括四部分：Network、Logic、Data、View（NLDV）。如何把他们组合起来，**结构清晰**、又**协作便利**，是前端主程的基本修养。本文用通（有）俗（点）易（啰）懂（嗦）的语言，界定了这四个模块的职能范围，同时提供了一种简单易用的组织方式。知者可互动，不知者可参考。



# 先唠点嗑
说说自己吧：毕业三年，经历4个项目，前两个主做功能开发，后两个全面负责。因为大学对UI交互深感兴趣，梦想成为优秀的交互设计师，所以毕业就做了前端，想着至少也离得近点。不料一入代码深似海，以前看过的十几本交互书也随风而去，脑子里剩下的只有：几行干巴巴的代码、几个平台的API、和一点不成熟的总结。借此机会，整理一下。

我把他名为**NLDV**，也就图一叫着方便（首字母的组合），并不是想标新立异。你或许会想到MVC，其实本质上跟MVC没啥区别。只是与时俱进，现在App几乎都是连着网的，也就把Network提出来了。

从第三个项目开始，这个NLDV的结构在我意识里渐渐清晰。第四个项目（游戏），因为是从零开始，我把NLDV的结构应用到了项目中。经历了渲染引擎更换和网络引擎更换，过程中其它模块儿做得改动极少，切身体会到它的妙处，忍不住前来分享。

# 从账号登陆谈起

我们从一个最简单的登陆请求谈起。按照`NLDV`框架的思想，步骤如下：

1. Logic告诉Network：以后你收到来自远方服务器的消息都跟我打声招呼，不然我告诉程序员整死你丫。(Logic向Network注册网络监听函数。)
2. 用户点击登陆按钮，此时View调用Logic的登录方法。Logic赶紧写封信，告诉Network把这封信送到服务器。（Logic根据传入的参数，构造相应的消息体，Network负责把消息发出去）
3. 漫长的等待。。。
4. Network终于等到服务器发来的消息，就急急忙忙递给Logic。Logic打开信封一看：“尼玛，居然能一次成功了！32个赞！”
    1. 可是信上还有性别，年龄，头像，邮箱等等等信息，头都大了，不记下来恐怕是隔夜就忘啊。于是，找来Logic的御用秘书Data（只有Logic能写入），这些信息就交给你了，临时存储还是永久存储我不管，反正我和View来取的时候，你要能给我。
    2. 好消息要和大家分享，于是Logic全应用广播：“我们已经出色的完成了登陆任务，大家再接再厉”
3. View收到此广播消息，用界面告诉用户“亲爱的，你登陆成功了”。到此，完成一次登陆。
    （*PS：对于大多数界面实例，Logic刚刚发的这条广播是可以当耳边风的，因为跟自己业务无关。但，对于登陆界面来说，他必须时刻竖起耳朵监听这个消息，要不然就是失职。所以一般情况下登陆View会在发送登录请求之前就向系统注册监听这个消息的函数，以确保万无一失。这是后话，没看懂也没关系*）

# 职责分配

首先，看一幅图：  
![NLDV Diagram][1]  


通过上面的例子和图示，可以来总结一下NLDV四大家族各自的职能范围了：
## Network
Network的是数据交流的基础。在这里并不单指socket，并且不暴露任何底层网络的实现。而是一个更加完整、稳定，有纠错功能的职能单位。主要功能：

* 响应Logic的调用，将构造好的消息体转换成服务器能识别的字节串，发送出去。
* 接收服务器发过来的字节串，转换成前端可识别的结构体，通知Logic。
    * 前后端在定义消息的时候，一般会用一个消息号(/或 `主消息号-子消息号`对）来唯一标识一种消息。
    * 而Logic也不止一个，账号系统，业务系统等，每个系统有对应的Logic，分别处理相关的业务逻辑。
    * 一个Logic仅仅会对某一些消息感兴趣，所以，它只向Network注册自己感兴趣的消息号。
* 容错处理。比如有限次的自动重发，需要的时候自动重连，网络彻底不可用时通知Logic等。

我们用最少的接口来定义它：

```java
// Real msg struct will implement this interface, adding some getter/setters.
interface IMessage
{
    uint getMsgId();// unique id for a type of msg
    byte[] toBytes();// serialize to bytes.
    void parseBytes( byte[] bytes); //deserialize to useful info.
}

//Generally, "Logic" will implement this interface for recieving data
interface INetworkHandler
{
    void onMessageRecv( IMessage msg);
}

interface INetwork
{
    void send( IMessage msg );
    void registHandler( uint msgId, INetworkHandler handler );
    void unregistHandler( uint msgId, INetworkHandler handler );
}
```

## Logic
Logic是一个应用中核心实现业务逻辑的部分。主要功能有： 

* 响应来自用户（View）的功能请求。或通过写入Data来改变状态，或构造消息体发送到服务器。
* 收到网络消息后，做相应的逻辑处理，将数据的改变写入Data，最后广播本地事件。
    * 这里的本地事件通常用一个 字符串或者整数指代，称为本地事件ID。
    * 这个过程通常会用到[观察者模式](http://baike.baidu.com/view/1854779.htm)。有一个本地消息中心`LocalMessageCenter`，监听者（通常是View）用本地事件ID来向`LocalMessageCenter`注册，而Logic调用`LocalMessageCenter.dispatch( localEventId ) ` 即可广播此本地事件。
    * 是的，Logic直接调用View的方法也能达到同样的目的。但是，为了减小Logic和View之间的耦合性，还是选用`LocalMessageCenter`作为中间层。随着需求的改变，View类可能面目全非，但`LocalMessageCenter`的接口却可以长久不变。

Logic的大致结构如下：

```java
class SomeLogic implemets INetworkHandler
{
    SomeLogic()
    {
        Network.getInstance().registHandler(1, this);
        Network.getInstance().registHandler(2, this);
    }

    
    void onMessageRecv( IMessage msg)
    {
        switch( msg.getMsgId() )
        {
            case 1:
                logicHandler1(IMessage msg);
                break;
            case 2:
                logicHandler2(IMessage msg);
                break;
            ...
        }
    }

    //请求操作
    void logicReq1(...);
    void logicReq2(...);
    ...

    //消息处理
    void logicHandler1(IMessage msg)
    {
        //TODO: 收到消息，逻辑处理
        
        //TODO: 向Data写入数据
        
        //TODO: 广播本地事件
        LocalMessageCenter.getInstance().dispatch( "Logic1Compelete" );
    };
    void logicHandler2(IMessage msg);
    ...
}



ILocalMessageHandler
{
    void onLocalMessage( String msg );
}

LocalMessageCenter
{
    static LocalMessageCenter getInstance(); // Singleton

    void dispatch( String msg );
    void regist( String msg, ILocalMessageHandler handler );
    void unregist( String msg, ILocalMessageHandler handler );
}


```
    
## Data
这个模块是全应用的数据中心。提供两个功能：

* **存储**。前面已经提到，基本只有Logic对Data有写入权限。由于没有想到好的办法，这个规范暂时只能通过编码习惯来约定，没有做框架级的约定，如果大家有好的办法，欢迎补充。Logic 不关心数据的存储方式：同步OR异步，临时OR永久；这些都由Data自己决定。
* **读取**。Logic和View都会使用到Data中的数据。但是**需要注意异步读取的问题**。一般App要求View对操作的响应速度要在0.1s级别。所以，若涉及大量的数据存储或读取，便需要借助异步处理。对于存储，我们可能不是那么关心异步存储什么时候结束，只需要知道它成功了既可。但对于读取，经常遇到的情况是：页面上加个菊花，数据完全读取成功之后，移除菊花。这就需要一个通知机制。此时，我们也会用`LocalMessageCenter`作为通信的桥梁。

## View
NLDV框架对View的限制，相比以上3个模块，非常少。因为，对于NLDV框架来说，View跟特定的平台无关，即它是对各种不同平台显示框架的抽象。在`IOS`里它是`UIFramework`,在`Android`里它是`xxx`，在游戏里它是`openGL`，甚至，在下面会讲到的机器人模拟器里它是一堆测试代码。

在这个框架里，View只在两个过程中出现：

1. 调用Logic的方法，发送逻辑请求。
2. 监听本地事件，读取Data，显示内容。

只补充一点：在View显示过程中，需要用到很多二级数据（二级数据，就是跟原始数据相对，对原始数据进行整合或者筛选后得到的数据），这些二级数据的处理过程最好在View中处理。因为这些代码大多跟特定的界面有关，而跟App的主要逻辑关系不大，为了以后更改方便，最好写在View里。

# WHY NLDV?
看到这里，读者大概能隐约的感受到NLDV的一些优点，但是又不那么清晰，要不要看下去呢？下个里程碑就到了看这个人扯淡有用么？再看下去两盘Dota的时间可就没了丫。。。

别急，举几个栗子提提神。

## 引擎更换
最早，因为兼容PC版本的斗地主，我们采用了原始的字节对齐的方式进行网络传输。又因为设计失误，网络层字节的pack和unpack以及异步读取的处理，都各种出问题。（因为需要跨平台，所以我们用了C++语言，采用了最基础的BSD socket进行socket连接。）

结果是，游戏在网络不稳定的情况下各种闪退。这下产品经理不高兴了。又因为当初开发网络层的同学接手了其他事情，只剩下我各种修修补补，最终也没能彻底解决问题。

于是，我决定重写网络层。（妈蛋，早看这段代码不爽了）。于是我花了两天封装了一个带自动重连和容错功能，支持异步接受和发送的`GameSocket`，简单测试可以发送和接受字节。5分钟替换游戏中的旧网络层，你猜，怎么着？一次Run就登陆成功了！点了几下，所有功能完好如初！尼玛，世界上有比这还幸福的事情么？

其实，别听我说的挺牛逼的，其实替换过程就改了不到10行代码。因为实在是跟Logic、Data、View没啥耦合的地方。

## 制作机器人

一般在线游戏，都会有一两个用户没问题，大量用户就有问题的时候。所以，机器人测试总是必要的。

当我把前端代码交给后端，简单介绍了一下结构之后，后端的小伙伴儿们都惊呆了。不是因为他看到这框架有多么优秀，而是：“这样，我只要写一个while循环，500行代码就能完成一个机器人了啊。我还申请了一个星期来做这个事情呢！”（这是他的原话）。

说的更具体点，制作一个机器人就这么几步：

1. 把所有的View代码文件删掉。
2. 写一个`AndroidLoop`类。在这个类里，监听斗地主主流程里必须处理的`LocalMessage`，在适当的时候发送主流程中的请求。（对于斗地主来说，主流程包括登陆、选房间、抢地主、出牌、退房间。每个应用有所不同，灵活自便。）
3. NLD（NLDV去掉V）部分都不变，几乎不用改一行代码。



## 逻辑清晰
框架的作用，理论层面上规范了整个软件的结构；而在实现层面，通俗一点，它就规范了**什么代码该写在哪里，不要随地乱放**。

作为一个针对性很强的框架，在上述过程中，NLDV约束了很多可能不需要约束的规范。其中大部分是项目中的干货经验。我知道他并不总是好的，我考略了很久要不要把他们加进来。最终，我还是写下来了，考虑到刚开始从零开始写应用的读者来说，这些可能避免他们走很多弯路；而对有经验的读者来说，可能他们有判断的能力，可以取舍自如。

我想，如果严格按照NLDV框架来编写程序，显而易见的好处就是：

1. 层次清晰，出现问题容易定位。
2. 主程再也不用担心同事们把代码写得到处都是了。。。


# 框架之外（下回分解）
# NLDV的适用场景（下回分解）

因为各种原因，这篇博客断断续续写了两个星期了，再不发布就要胎死腹中了。所以，最后两节放在这篇日志的续集中写。如果您感兴趣，请私信我，我会尽快补上。





[1]: How%20to%20use%20the%20Public%20folder.png "NLDV Dirgram"



