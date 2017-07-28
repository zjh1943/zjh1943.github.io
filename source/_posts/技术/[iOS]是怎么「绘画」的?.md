---
title: "[iOS]是怎么「绘画」的?"
date: 2013-12-31
tags: 
- ios
- graphic
- anmation
- CGContextRef
- drawRect
- drawAtPoint
categories: 
- 技术
---

摘要：如果您以前从事其它平台的图形/界面开发或者游戏开发,一定知道, 不管上层UI怎么呈现和响应, 底层必须有一个绘图引擎. iOS也不例外. 本文详细介绍了iOS Graphics的用法和相关知识, 希望对您的Coding有帮助.  


^此博客需要对`CALayer`和`UIView`有基本的了解. 可参考博客[谈谈iOS Animation](http://geeklu.com/2012/09/animation-in-ios/)


# 什么是绘图引擎
绘图引擎, 通俗来说就好比给你一张纸一支笔和若干颜色的颜料, 你可以用它来做**最基本**的图形绘制.   
一个**最基本的绘图引擎**包括一下接口:  

```objc
//Code-1
I1. drawLine()		//绘制任意线条,并支持对线条上色.
I2. drawPath()		//根据路径绘制形状,并支持填充颜色.
I3. drawImage()		//绘制图像(e.g. xxx.jpg, xxx.png)
I4. drawGradient()	//绘制渐变填充
I5. transform()		//矩阵映射变换.
I6. drawText()		//绘制文字
```

不难想象, 有了以上接口, 我们就可以**方便**的绘制任意想要的图像.  
这里强调的是**方便**, 有些接口并不是**必须**的. 比如说`drawImage()`,我们总可以调用有限次`drawLine()`和`drawShape()`来绘制任意给定的Image. 但是复杂程度可想而知.   
一个绘图引擎设计的目的就是为了方便上层调用, 所以它会封装一些**最常用**和**最基本**的接口. 以上5个接口就满足这两个条件之一. 所谓**最常用**和**最基本**并没有一个明确的定义, 所以不同的绘图引擎可能会多一些常用接口,但都大同小异.


<br/>
<br/>
<br/>
# iOS的绘图引擎
下面我们就`Code-1`里提到的接口在iOS平台上做一个介绍.

## 在哪里绘制?
如果我们在XCode里新建一个`UIView`类, 我们会得到以下代码:

```objc
//Code-2
# import "GraphicsViewControllerView.h"
@implementation GraphicsViewControllerView
- (id)initWithFrame:(CGRect)frame
{ 
	self = [super initWithFrame:frame]; 
	if (self) {
		// Initialization code }
	return self; 
}
- (void)drawRect:(CGRect)rect
{
	// Drawing code 
}
@end
```

通常,`drawRect()`都会被注释起来. 因为, 如果你向`UIView`添加`subView`或者设置`UIView`的显示相关的属性(e.g. `backgroundCrolor`)的时候, `UIKit`会自动的把这些参数代表的含义绘制到`CALayer`上去. 也就是说, 一般情况我们并不需要自己来绘制, `UIKit`会自动帮我们完成绘制工作.   
但是, 当不添加`subView`, 不设置`UIView`的显示相关的属性时, 我们就可以通过重载`drawRect()`来手动绘制图像了.   

## Context
>A graphical context can be thought of as a canvas, offering an enormous number of properties such as pen color, pen thickness, etc. Given the context, you can start painting straight away inside the drawRect: method, and Cocoa Touch will make sure that the attributes and properties of the context are applied to your drawings. We will talk about this more later, but now, let’s move on to more interesting subjects.


## drawText
我们新建一个`UIView`类`CustomUIView`,如下重载`drawRect()`方法.  
新建`CustomUIView`的对象,不设置任何属性,添加到显示列表. 

```objc
//Code-3
- (void)drawRect:(CGRect)rect{ 
	UIFont *font = [UIFont systemFontWithSize:40.f]; 
	NSString *myString = @"Some String";
	[myString drawAtPoint:CGPointMake(40, 180) withFont:font];
}
```

运行, 就可以看到我们没有添加任何`UITextField`却显示了文字~  

#### more:
关于文字绘制的方法还有 `drawInRect:withFont:`等几个方法, 可参考官方文档.

## setColor
我们把`Code-3`中的代码添加两行, 变成:  

```objc code-4
//Code-4
- (void)drawRect:(CGRect)rect{ 
	UIColor* color = [UIColor blueColor];		//create color
	[color set];								//set color
	UIFont *font = [UIFont systemFontWithSize:40.f]; 
	NSString *myString = @"Some String";
	[myString drawAtPoint:CGPointMake(40, 180) withFont:font];
}
```

就可以看到,文字由黑色变成了蓝色.  
#### more:
`UIColor`还有两个方法`setStroke`和`setFill`分别设置线条颜色和填充颜色. 在后面的章节会用到.`set`方法影响所有前景色.

## drawImage
同样的, 如下重写`drawRect`方法:

```objc
//Code-5
- (void)drawRect:(CGRect)rect{ 
	/* Assuming the image is in your app bundle and we can load it */
	UIImage *xcodeIcon = [UIImage imageNamed:@"filename.png"];
	[xcodeIcon drawAtPoint:CGPointMake(0.0f, 20.0f)];
}
```

我们看到图像被绘制出来了.

#### more:
`UIImage`还有其他绘制方法:

```objc
drawInRect:
drawAsPatternInRect:
drawAtPoint:blendMode:alpha:
drawInRect:blendMode:alpha:
```

方法名已经很清楚的说明了方法的用途. 具体可参考官方文档

## drawLine
这两个是绘图引擎里最基本的, 所以放在一起讲述.

```
//Code-6: drawLine
- (void)drawRect:(CGRect)rect{ 
	
	/* Step1 设置绘图颜色 */ 
	[[UIColor brownColor] set];
	
	/* Step2 获取当期的画布: Graphic Context */ 
	CGContextRef currentContext = UIGraphicsGetCurrentContext();
	
	/* Step3 设置线条宽度 */ 
	CGContextSetLineWidth(currentContext,5.0f);
	
	/* Step4 把画笔移动到起始点 */ 
	CGContextMoveToPoint(currentContext,50.0f, 10.0f);
	/* Step5 从起始点绘制线条到终点 */ 
	CGContextAddLineToPoint(currentContext,100.0f, 200.0f);
	
	/* Step6 提交绘制 */
	CGContextStrokePath(currentContext); 
}
```
如果想连续绘制多条线, 可以再`Code-6`中的`Step5`和`Step6`之间多次调用`CGContextAddLineToPoint()`.
#### more:

`CGContextSetLineJoin`可以改变线条交叉点的样式.

## drawPath
如果我们想快速绘制一条折线, 调用`drawLine`就显得有些臃肿. 所以有了`drawPath`, 它是`drawLine`的加强版.  
`drawPath`的一般步骤如下: 

```
//Code-7: drawPath
- (void)drawRect:(CGRect)rect{ 
	
	/* Step1 获取当期的画布: Graphic Context */ 
	CGContextRef currentContext = UIGraphicsGetCurrentContext();
	
	/* Step2 创建 path */ 
	CGMutablePathRef path = CGPathCreateMutable();
	
	/* Step3 移动到起始点 */
	CGPathMoveToPoint(path,NULL, screenBounds.size.width, screenBounds.origin.y);
	
	/* Step4 绘制一个椭圆 */
	CGPathAddEllipseInRect(path, &CGAffineTransformIdentity, CGRectMake(0, 320, 320, 160));
	
	/* Step5 再添加一条直线 */
	CGPathAddLineToPoint(path,NULL, screenBounds.origin.x, screenBounds.size.height);
	
	/* Step6 向画布添加path */
	CGContextAddPath(currentcontext, path);
	
	/* Step7 设置绘制类型: kCGPathStroke(绘制边缘), kCGPathFill(填充path内区域), kCGPathFillStroke(包含前面两项)*/
	CGContextDrawPath(currentcontext, kCGPathFillStroke);
	
	/* Step8 提交绘制 */
	CGContextStrokePath(currentContext); 
	
	/* Step9 release path */
	CGPathRelease(path);
}
```
#### more:
常见几何图形的绘制接口: `CGPathAddCurveToPoint`,`CGPathAddArcToPoint`,`CGPathAddRect`等等...


## transform
iOS中的`transform`使用`CGAffineTransform`表示的. 你可以用矩阵方式构造任意二维变换:  

```
/*
a: The value at position [1,1] in the matrix.
b: The value at position [1,2] in the matrix.
c: The value at position [2,1] in the matrix.
d: The value at position [2,2] in the matrix.
tx: The value at position [3,1] in the matrix.
ty: The value at position [3,2] in the matrix.
*/
CGAffineTransformMake(CGFloat a, CGFloat b, CGFloat c, CGFloat d, CGFloat tx, CGFloat ty)
```
矩阵表示为:  
![](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CGAffineTransform/Art/equation01_2x.png)

iOS也提供了常见变换的快速构建方式:   

```
CGAffineTransformIdentity			//单位矩阵, 不做任何变换
CGAffineTransformMakeRotation		//旋转
CGAffineTransformMakeScale			//缩放
CGAffineTransformMakeTranslation	//平移
```

在`Code-7 Step4`中, 绘制`path`时用到了单位矩阵`CGAffineTransformIdentity`, 表示不做任何变换. 通常情况下, 都是在绘制阶段把`transform`作为参数传入. 上面提到的`CGPathAddCurveToPoint`,`CGPathAddArcToPoint`,`CGPathAddRect`函数都有一个`transform`参数.

<br/>
<br/>
<br/>
# iOS绘图在项目中的应用
通常, 我们只需要随心所欲的对`UIView`增加`subView`, `UIKit`会自动帮我们绘制. 但是下列情况下可能需要手动绘制:  

1. 优化`UITableViewCell`的时候. 如果我们的`cell`很复杂, 有很多`subView`, 就会变得很卡顿. 就需要手动绘制了. 可参考博客: [优化UITableView性能](http://www.keakon.net/2011/08/03/%E4%BC%98%E5%8C%96UITableView%E6%80%A7%E8%83%BD)或者[IOS详解TableView——性能优化及手工绘制UITableViewCell](http://blog.csdn.net/cocoarannie/article/details/11183067)
2. 暂未想到, 以后想到再说.
