---
title: '[iOS]Orientation 想怎么转就怎么转'
date: 2013-12-12
tags: 
- ios
- transform
- orientation
- screen
- ios6
- ios7
- interface
categories: 
- 技术
---

此博文主要针对IOS应用, 是屏幕旋转相关问题的一个总结. 主要内容有:  
  
1. IOS5,6,7不同版的适配.  
2. 强制旋转和自动旋转.


---

## 改变Orientation的三种途径
这里, 咱们主要理清一下: **到底有哪些设置可以改变屏幕旋转特性**. 这样:  

* 出现任何问题我们都可以从这几个途径中发现原因.
* 灵活应付产品经理的各种需求.  

首先我们得知道: 
 
1. 当手机的重力感应打开的时候, 如果用户旋转手机, 系统会抛发`UIDeviceOrientationDidChangeNotification` 事件.  
2. 您可以分别设置`Application`和`UIViewcontroller`支持的旋转方向.`Application`的设置会影响整个App, `UIViewcontroller`的设置仅仅会影响一个`viewController`(IOS5和IOS6有所不同,下面会详细解释).  
3. 当`UIKit`收到`UIDeviceOrientationDidChangeNotification`事件的时候, 会根据`Application`和`UIViewcontroller`的设置, 如果双方都支持此方向, 则会自动屏幕旋转到这个方向. 更code的表达就是, 会对两个设置求**与**,得到可以支持的方向. 如果求**与**之后,没有任何可支持的方向, 则会抛发`UIApplicationInvalidInterfaceOrientationException`异常.


<br/>
<br/>
<br/>
### Info.plist设置
在App的Info.plist里设置:

|key|xcode name|Summary|avilable value|
|:--|:--       |:--    |:--           |
|UIInterfaceOrientation|initial interface orientation|Specifies the initial orientation of the app’s user interface.|UIInterfaceOrientationPortrait,<br/>UIInterfaceOrientationPortraitUpsideDown,<br/>UIInterfaceOrientationLandscapeLeft,<br/>UIInterfaceOrientationLandscapeRight|
|UISupportedInterfaceOrientations|Supported interface orientations|Specifies the orientations that the app supports.|UIInterfaceOrientationPortrait,<br/>UIInterfaceOrientationPortraitUpsideDown,<br/>UIInterfaceOrientationLandscapeLeft,<br/>UIInterfaceOrientationLandscapeRight|

在Info.plist中设置之后,这个app里所有的`viewController`支持的自动旋转方向都只能是app支持的方向的子集.



<br/>
<br/>
<br/>
### UIViewController
#### IOS6 and above
##### supportedInterfaceOrientations

在IOS6及以上的版本中, 增添了方法`UIViewController.supportedInterfaceOrientations`. 此方法返回当前`viewController`支持的方向. 但是, 只有两种情况下此方法才会生效:

1. 当前`viewController`是`window`的`rootViewController`. 
2. 当前`viewController`是`modal`模式的. 即, 此`viewController`是被调用`presentModalViewController`而显示出来的.

在以上两种情况中,`UIViewController.supportedInterfaceOrientations`方法会作用于当前`viewController`和所有`childViewController`. 以上两种情况之外, `UIKit`并不会理会你的`supportedInterfaceOrientations`方法.
    
举个栗子: 

```
- (NSUInteger)supportedInterfaceOrientations
{
    return UIInterfaceOrientationMaskPortrait | UIInterfaceOrientationMaskLandscapeLeft;
}

```

如果某个`viewController`实现了以上方法. 则, 此`viewController`就支持竖方向和左旋转方向. 此`viewController`的所有`childViewController`也同时支持这两个方向, 不多不少. 

##### preferredInterfaceOrientationForPresentation
此方法也属于`UIViewController`. 影响当前`viewController`的初始显示方向.
此方法也仅有在当前`viewController`是`rootViewController`或者是`modal`模式时才生效.
##### shouldAutorotate
此方法,用于设置当前`viewController`是否支持自动旋转. 如果,你需要`viewController`暂停自动旋转一小会儿. 那么可以通过这个方法来实现.同样的, 此方法也仅有在当前`viewController`是`rootViewController`或者是`modal`模式时才生效.  
#### IOS5 and before
在IOS5和以前的版本中, 每个`viewController`都可以指定自己可自动旋转的方向.(~~~这样不是挺好么?苹果那帮工程师为啥要搞成这样...~~~).  
每当`UIkit`收到`UIDeviceOrientationDidChangeNotification`消息的时候, 就会用以下方法询问当前显示的`viewController`支不支持此方向:

```
- (BOOL)shouldAutorotateToInterfaceOrientation:(UIInterfaceOrientation)orientation
{
   if ((orientation == UIInterfaceOrientationPortrait) ||
       (orientation == UIInterfaceOrientationLandscapeLeft))
      return YES;
 
   return NO;
}
```

特别要注意的是:你必须至少要对一个方向返回`YES`.(为难系统总不会有啥好事儿,你懂得).

<br/>
<br/>
<br/>
### UIView.transform
最后一个方法是设置`UIView`的`transform`属性来强制旋转.   
见下代码: 
 
```
//设置statusBar
[[UIApplication sharedApplication] setStatusBarOrientation:orientation];

//计算旋转角度
float arch;
if (orientation == UIInterfaceOrientationLandscapeLeft)
	arch = -M_PI_2;
else if (orientation == UIInterfaceOrientationLandscapeRight)
	arch = M_PI_2;
else
	arch = 0;

//对navigationController.view 进行强制旋转
self.navigationController.view.transform = CGAffineTransformMakeRotation(arch);
self.navigationController.view.bounds = UIInterfaceOrientationIsLandscape(orientation) ? CGRectMake(0, 0, SCREEN_HEIGHT, SCREEN_WIDTH) : initialBounds;
```
需要注意的是: 

1. 当然我们可以对当前`viewController`进行旋转, 对任何`view`旋转都可以.但是, 你会发现`navigationBar`还横在那里. 所以, 我们最好对一个占满全屏的`view`进行旋转. 在这里我们旋转的对象是`self.navigationController.view`, 当然`self.window`也可以, help yourself~
2. 我们需要显式的设置`bounds`. `UIKit`并不知道你偷偷摸摸干了这些事情, 所以没法帮你自动设置.


<br/>
<br/>
## 如何应付产品经理的需求
有了以上三把武器, 我想基本可以应付BT产品经理所有的需求了. 但是这里还有一些小技巧.
### 直接锁死
(略)
### 随系统旋转
#### IOS5及之前
对于IOS5及之前的版本, 只要在对每个`viewController`重写`shouldAutorotateToInterfaceOrientation`方法, 即可方便的控制每个`viewController`的方向.  
  
#### IOS6及以后
对于IOS6及以后的版本, 如果想方便的单独控制每个`viewController`的方向. 则可以使用这样:

* 对于非`modal`模式的`viewController`: 

	* 如果不是`rootViewController`,则重写`supportedInterfaceOrientations`,`preferredInterfaceOrientationForPresentation`以及`shouldAutorotate`方法, 按照当前`viewController`的需要返回响应的值.
	* 如果是`rootViewController`,则如下重写方法:

```
-(NSUInteger)supportedInterfaceOrientations
{
	return self.topMostViewController.supportedInterfaceOrientations;
}
-(BOOL)shouldAutorotate
{
    return [self.topMostViewController shouldAutorotate];
}
- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation
{
    return [self.topMostViewController preferredInterfaceOrientationForPresentation];
}
-(UIViewController*)topMostViewController
{
	//找到当前正在显示的viewController并返回.
}
```
显而易见, 我们巧妙的绕开了`UIKit`只调用`rootViewController`的方法的规则. 把决定权交给了当前正在显示的`viewController`.	

* 对于`modal`模式的`viewController`. 则按照需要重写`supportedInterfaceOrientations`,`preferredInterfaceOrientationForPresentation`以及`shouldAutorotate`方法即可.

### 强制旋转
有时候, 需要不随系统旋转, 而是强制旋转到某一个角度. 最典型的场景就是视频播放器, 当点击了全屏按钮的时候, 需要横过来显示.  

* 对于IOS5及以前的版本, 可以用下面的方法:  

```
if ([[UIDevice currentDevice] respondsToSelector:@selector(setOrientation:)]) {
	SEL selector = NSSelectorFromString(@"setOrientation:");
	NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:[UIDevice instanceMethodSignatureForSelector:selector]];
	[invocation setSelector:selector];
	[invocation setTarget:[UIDevice currentDevice]];
	int val = UIInterfaceOrientationLandscapeRight;
	[invocation setArgument:&val atIndex:2];
	[invocation invoke];
}
```

* 对于IOS6及以后的版本. `UIDevice.setOrientation`从隐藏变为移除.只能通过设置`UIView.transform`的方法来实现.

<br/>
<br/>
<br/>
## 参考资料

* [iOS两个强制旋转屏幕的方法](http://blog.csdn.net/volcan1987/article/details/11563741)
* [Supporting Multiple Interface Orientations](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/RespondingtoDeviceOrientationChanges/RespondingtoDeviceOrientationChanges.html)

[apple_ref_1]: https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIDevice_Class/Reference/UIDevice.html#//apple_ref/c/data/UIDeviceOrientationDidChangeNotification]
[apple_ref_2]: https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIApplication_Class/Reference/Reference.html#//apple_ref/c/data/UIApplicationInvalidInterfaceOrientationException