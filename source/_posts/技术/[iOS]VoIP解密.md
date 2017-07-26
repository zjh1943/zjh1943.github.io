---
title: '[iOS]VoIP解密'
date: 2013-11-10
tags: 
- ios
- voip
- socket
- background
- iphone
categories: 
- 技术
---

摘要：一般来说, IOS很少给App后台运行的权限. 仅有的方式就是 VoIP. IOS少有的为VoIP应用提供了**后台socket连接,定期唤醒并且随开机启动**的权限.而这些就是IOS上实现VoIP App的关键. 苹果官方文档对于的描述就短短的一页([点击这里][apple_reference_voip]),很多细节没有提及. 这篇微博通过具体实现和查阅资料,补充了这些细节.并且列举出了在实现过程中可能遇到的问题, 作为参考.

---

# 官方文档描述如是:

**PS:**此节纯用来占座.如果你你E文不好或者想直接切入正题, 请看下一标题.  
  
  
> There are several requirements for implementing a VoIP app:

>1. Add the UIBackgroundModes key to your app’s Info.plist file. Set the value of this key to an array that includes the voip string.  
1. Configure one of the app’s sockets for VoIP usage.  
S
2. Before moving to the background, call the setKeepAliveTimeout:handler: method to install a handler to be executed periodically. Your app can use this handler to maintain its service connection.  
3. Configure your audio session to handle transitions to and from active use.  
4. To ensure a better user experience on iPhone, use the Core Telephony framework to adjust your behavior in relation to cell-based phone calls; see Core Telephony Framework Reference.  
5. To ensure good performance for your VoIP app, use the System Configuration framework to detect network changes and allow your app to sleep as much as possible.



# 我的翻译:  
关于IOS为VoIP应用提供的特殊权限和实现方法,我的描述如下. 我尽可能的涉及到voip实现的各种细节, 这样你能对这个运作机制有一个更好的**理解**,我觉得这远比单单贴几行代码有意义. 因为一个开发者在实际实现过程中遇到的千难险阻很少会体现在最终代码上, 就如你永远不知道台上的角儿在台下的挫折.  
 
1. **IOS允许App的一个Socket在App切换到后台后仍然保持连接**. 这样,当有通话请求的时候,App能及时处理. 这个`socket`需要在应用第一次启动的时候创建, 并标记为"此`socket`用于VoIP服务". 这样当App切换到后台的时候,IOS会**接管**这个标记为"用于VoIP服务"的`socket`. 这个`socket`的响应函数(比如,一个`delegate`,或者是个`block`)会正常的响应, 就像App还在前台一样.  
2. **10s魔咒**. 当`socket`有任何数据从服务端传来, 你在app里为`socket`写的响应函数都会做处理.但是, 你只有最多10s的时间来干你想干的事情. 也就意味着你在响应函数里新建一个大于10s的`timer`是没有意义的. 并且IOS并不保证给你足够10s的时间,视系统情况而定. 
3. 在`socket`的响应函数里, 你能通过`NSLocalNotification`来通知用户"电话来了". 除此之外, 你没法做其他任何视觉上的动作来提醒用户, 因为你的app还处于某个不知道的次元, 甚至连`window`都还没创建. 
4. 你永远也没有办法知道或者决定`NSLocalNotification`的样式是`banner`还是`alert`. 你也许钟爱后者, 但是决定权在用户手里.
3. **允许在后台定期执行一段代码**. 你可以设定一个大于等于10分钟的时间`t`, 和一个定期执行的`handler`, IOS系统会在每次经过`t`时间的时候调用一次这个`handler`. 但是IOS不保证这个`handler`会准时运行, 只保证在时间`t`范围内的某个点会执行一次.   
4. 我们通常用楼上的`handler`处理socket的断线重连操作. 因为网络不稳定, 或者用户开启飞行模式等原因, 我们用于voip服务的`socket`会断开连接. 在这个`handler`里,如果发现连接断开,我们只需要跟条目1一样的创建`socket`,设置一样的`socket`响应函数,一切又会恢复正常.
5. 不建议这个`handler`做太多事情, 因为它也有**10s魔咒**.(据不完全统计,苹果所有的后台处理都有这个10s限制. 不保证绝对正确哈, 仅供参考)
6. **自启服务**. 当IOS重新启动, 或者你的app因为其他原因退出时, IOS会马上启动你注册为voip的app, 你可以很迅速的恢复`socket`连接. 但是, 从底部多任务栏中手动关闭应用除外.更"code"的说明是:当程序退出的`exitcode != 0`,IOS会重启你的app.经试验发现,从底部多任务栏关闭的时候,程序的`exitcode == 0`.
7. 如果你亲爱的用户是一个典型的"app终结者",那么你还剩最后一条路来通知来电提醒:`NSRemoteNotification`.  你也许会被`NSRemoteNotification`的可靠性和实时性折腾的抓狂, 但是, 谁让你选了IOS? 你享受了封闭带来的传世体验, 也得承受封闭的限制.  
7. 当条目8描述的情况发生之后,app的`application:didFinishLaunchingWithOptions:`会被调用. 但是,请时刻提醒自己我们的app仍然处于后台. 我们以前总在这里创建`window`创建`rootController`, 但现在不必了. 现在我们需要的就是把可爱的`socket`连上, 像在条目1里一样,让一切回归正常(我不去写歌词真浪费了^_^).
8. 在`application:didFinishLaunchingWithOptions:`里你能通过`[application applicationState] == UIApplicationStateBackground`来判断是正常启动应用还是系统自动启动, 然后决定该创建`window`还是创建voip的`socket`.
9. 如果你看完上面一头雾水. 请回炉重造, 传送门:[Programming with Objective-C][apple_refrence_objective_c_program], [iOS Develop Library][apple_refrence_ios_develop].
  




[apple_reference_voip]: https://developer.apple.com/library/ios/documentation/iphone/conceptual/iphoneosprogrammingguide/AdvancedAppTricks/AdvancedAppTricks.html#//apple_ref/doc/uid/TP40007072-CH7-SW12
[apple_refrence_objective_c_program]: https://developer.apple.com/library/mac/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/Introduction/Introduction.html
[apple_refrence_ios_develop]: https://developer.apple.com/library/ios/navigation/
