---
date:  2013-11-03 00:25
title: '[iOS]UIView.frame的骗局'
tags:  
- ios
- transform
- anchorPoint
- frame
- center
- bounds
- position
- UIView
categories: 
- 技术
---


摘要：如果你刚刚开始接触IOS编程, 刚刚接触UIKit, 肯定会被 `frame, bounds, center, layer.anchorPoint, layer.position` 这些乱七八糟得属性折腾得心烦意乱. 并且,聪明的你肯定早就发现,这些属性并不是独立的, 比如`frame`和`bounds`, 你改变一个必然会影响另一个, 这就更加大了理解难度. 我想通过这篇**浅显的日志,和一个简单的Demo**来表达出我对这些变量的理解. 难免有偏差之处, 欢迎拍砖. 但是我能保证的是这些理解方式是**实用**的. 我个人也是看过网上很多日志对其有些微理解, 然后又通过写一个Demo来证明自己的想法. 

--- 

其实, 受过10几年教育的你, 必然知道, 一个二维矩形, 只要有了`{x,y,width,height}`, 也就唯一确定了它的几何属性. 没错, 其实`UIView`里面也就这几个变量. 其他变量, 比如`frame,bounds`都是这些变量通过基本变量导出的.那么`UIView`拥有的真正意义上的属性有哪些呢?



## UIView 真正意义上的属性:
- **bounds**:   
`bounds`是一个`CGRect`. 他的size部分决定了`UIView`的大小,也就是,`bounds.width`和`bounds.height`决定了`UIView`的大小.你也可以说`bounds.width`和`bounds.height`就是`UIView`的`width`和`height`. `bounds.x`和`bounds.y`决定了`UIView`的`subView`的原点坐标.如果你更改了`bounds.x`或者`bounds.y`,`UIView`的位置和大小完全不为所动, 但是`UIView`的所有`subView`都会平移一段距离`(-bounds.x,-bounds.y)`(这一点我们会在下文做详细陈述).
- **center**:  
望文生义(注意,这是个带贬义的词),他就是`UIView`的中心,也就是坐标点`(view.width/2,view.height/2)`.但是,可恶的但是, 上句话仅仅在在一个`UIView`刚被创建的时候成立. 也就是,在刚刚创建`UIView`的时候,他**恰好**成立. **其实, `center`有它更重要的角色: 就是决定了`UIView`的位置.** (但是,这个位置并不是我们常规意义上理解的`(x,y)`. 在这里你先知道它来决定我们UIView的位置就好了.)

下面来看我们的第一个公式.


## 揭开frame的本质
我想,对于程序员的你,没有比比代码更直接的方式了吧?  
下面就是`UIView`的属性`frame`的实现:

    
```
//代码1
-(CGRect) frame  
{  
     float x = center.x - 1/2 * bounds.width;   
     float y = center.y - 1/2 * bounds.height;  
     float width = bounds.width;
     float height = bounds.height;  
     return CGRectMake(x, y, width, height);
}

-(void) setFrame:(CGRect) rect
{
	center.x = rect.x + 1/2 * rect.width;
	center.y = rect.y + 1/2 * rect.height;
	bounds.width = rect.width;
	bounds.height = rect.height;
}
```
  
下面来到实战演习:  

```
//代码2
- (void)viewDidLoad
{
    [super viewDidLoad];
	testView = [[UIView alloc] initWithFrame:CGRectMake(100, 100, 50, 50)];
	testView.backgroundColor = [UIColor blueColor];
	[self.view insertSubview:testView atIndex:0];
	
	UIView* innerView = [[UIView alloc] initWithFrame:CGRectMake(23,23, 4, 4)];
	innerView.backgroundColor = [UIColor redColor];
	[testView addSubview:innerView];
}


```
初始状态,我们新建一个`UIView`,设置`frame`为`(100,100,50,50)`:  
图1  
![Resize icon][1]  

然后,我们改变`frame`  

```
testView.frame = CGRectMake(0,0,40,40); //代码3
```
我们再看看各个属性的变化:  
图2  
![Resize icon][2]  


没错,你看到了我们刚刚提到的`center`属性. 我们刚刚说过, 这个属性主要决定了`UIView`的位置. 所以当我们在`setFrame:`的时候会改变`UIView`的位置.  
好吧, 既然我们见过`center`先生, 那就给您介绍一下吧, 也要给点面子是不?


## center如何搞定了位置?
`center`是`UIView`的相关属性中主要决定`UIView`位置(跟大小相对), 我们在图2的基础上, 改变center:  

```
testView.center = CGPointMake(125,125);//代码4
```
我们来看一下效果:  
图3  
![Resize icon][3]  
看到了吧? `testView`的位置向左下移动了`(125-20, 125-20)`距离.
根据*代码1*, 我们可以看到, `center`和`bounds`属性是相互独立的. 也就是他们中间某一个发生了变化, 不会影响另一个. 这说明了什么? 说明我们改变`center`的时候, 仅仅会改变`testView`的位置, 而它的大小不会有任何改变.

  
好了, 到这里, `testView`的**位置**和**大小**问题,我们已经彻底解决了. 
但是对于`bounds`小伙儿, 我们只关注了它的`size`部分, 忽略了他的`origin`部分. 不好意思, `bounds`小伙, 现在才想起你.


## bounds的另一半
没图没真相, 我们首先来点料吧:  
图4,图5   
![Resize icon][4] ![Resize icon][5]

`bounds.origin`初始情况下为`(0,0)`. 我们设置  

```
testView.bounds = CGRectMak(25,25,40,40); //代码5
```
得到左图.
我们没有改变`bounds.size`(仍然是40,40), 只是修改了`bounds.origin`: 从`(0,0)`改变成`(25,25)`.我们发现`testView`内部的小红点移动了`(0-25,0-25)`距离.(至于为设么这里是-25而不是25, 我也还没理解, 望高人指点.) 反正, 知道当你改变`bounds.origin`的时候, `testView`内部所有的`subView`都要想做相反方向的位移就对了. 

在左图的基础上,再设置

```
testView.bounds = CGRectMak(-5,-5,40,40); //代码6

```
我们得到右图.

  
需要注意的是, 这里所有`subView`的`frame`是不会跟着改的, 还是原来的值. 我们可以这么理解: 设置`testView.origin`, 会改变所有孩子节点位置的基准点. 就比如, 我们把一辆车平移了, 我们站在路边发现车里的方向盘和发动机等子组件的位置都改变了, 而方向盘发动机等"子组件"相对于**汽车的坐标**没有改变.  

关于`bounds`, 还有一点要说: **当你的`subView`的某些部分落在了`bounds`定义的矩形之外, 那么这些落在矩形之外的部分, 便不能接受任何点击,踩踏,横扫等事件了....**

```
# ifdefine 欺骗 明明知道某个事实,却故意隐瞒或窜改并加以传播  
写到这里, 我其实要跟大家道一个歉, 因为我在上面对frame的定义欺骗了大家.   
# endif  
```
`frame`的定义并没有这么简单, 因为还搀插着第三者的关系: `testView.layer.anchorPoint`.  
有请 anchorPoint 出场!  

## 在墙上钉个钉子,就是anchorPoint
好了,我们重新定义`frame`的`getter/setter`函数(其实就是把代码1的定义中所有的`1/2`改为`view.layer.anchorPoint`):

```
//代码7
-(CGRect) frame  
{  
     float x = center.x - layer.anchorPoint.x * bounds.width;   
     float y = center.y - layer.anchorPoint.y * bounds.height;  
     float width = bounds.width;
     float height = bounds.height;  
     return CGRectMake(x, y, width, height);
}

-(void) setFrame:(CGRect) rect
{
	center.x = rect.x + layer.anchorPoint.x * rect.width;
	center.y = rect.y + layer.anchorPoint.y * rect.height;
	bounds.width = rect.width;
	bounds.height = rect.height;
}
```

因为`anchorPoint`的默认值是`(0.5,0.5)`,所以如果你不改变`anchorPoint`,那么*代码1*就是正确的. 之所以在*代码1*里撒了一个谎, 是因为不想那么早把`anchorPoint`引出来.  
现在,你既然知道了`anchorPoint`跟`frame`之间的关系, 必然想知道它到底有什么用:  

  
先给你一个直观印象: `anchorPoint`就是一个钉子,把一幅画钉在墙上. 以后你想做什么转动也好, 把相框拉伸也好, 这个点是绝对不会动的.  
`anchorPoint`的默认值是`(0.5,0.5)`. 也就是说默认情况下,你对`testView`作旋转和缩放, 都会以`(bounds.size.width/2,bounds.size.height/2)`为基准点.下面我们接着图5来看一个转化:

```
testView.transform = CGAffineTransformMakeScale(2, 2);//代码8
```
视图如下:  
图6  
![Resize icon][6]   
我们看到,`testView`以中心点位固定点,等比例扩大了一倍.`frame.origin`也移动了`(-20,-20)`. 跟我们的预期一样.  
~~这里需要特别提醒各位的是:** 当对`testView`进行了`transform`之后,我们再去设置`frame`, `frame`已经完全不理咱们了. 也就是说`setFrame`函数完全不工作了.** 我们这个时候调用`frame`的`getter`函数, 得到的是transform后的大小.如上图所示,变成了`(85,85,80,80)`.~~
如果, 我们把`scale`恢复为1, 再改变`anchorPoint`的位置为`(0,0)`, 然后再把`testView`放大一倍(`scale=2`). 看看会发现什么:  

```
//代码9
testView.transform = CGAffineTransformMakeScale(1, 1);
testView.layer.anchorPoint = CGPointMake(0,0)
testView.transform = CGAffineTransformMakeScale(2, 2);
```
下面三个图分别对应上面三行代码执行后的状态:  
图7,图8,图9  
![Resize icon][7] ![Resize icon][8]![Resize icon][9]

对于上面的运行结果, 我有几点说明:   
 
* 当我们改变`anchorPoint`的时候,`center`没有改变,那么根据*代码7*, `frame`也会随着发生改变. 您可以在图8中观察到这一变化.  
* 观察图8和图9的变化,你可以发现,这次缩放的中心点在左上角. 因为我们设置了`anchorPoint = CGPointMake(0,0)`.  
* 如果您是对图像做旋转, `anchorPoint`也是旋转的中心.


到此为止, 文章开始提到的属性都基本讲完了. 只剩下了`layer.position`. 如果你细心, 你会发现上面所有的图片中, `layer.position === center`, 没错, 任何时候他们都是相等的.


## 结尾
这里有博客中Demo的代码下载, 本博客中的所有截图都来自于Demo的截屏.如果我在博客中没有说明白, 您可以下载Demo仔细把玩. 相信你可以在实际操作中有更深刻的理解  
Demo底部的输入框支持的的语法有:

```
scale = number
center = ( number, number )
frame = (number, number, number, number )
bounds = (number, number, number, number)
center = (number, number)
position = (number, number) //layer.position
anchorPoint = (number, number) //layer.anchorPoint
```
比如您输入`frame =(0,0,40,40)`,就相当于执行代码`testView.frame = CGRectMake(0,0,40,40)`. 输入`center = (50,50)`就相当于执行代码`testView.center = CGPointMake(50,50)`.

[点此下载Demo][demo_download]


[1]: frame_images/IMG_0012.PNG "初始状态图(1)"
[2]: frame_images/IMG_0013.PNG "第一次设置frame(2)"
[3]: frame_images/IMG_0014.PNG "第一次设置center(3)"
[4]: frame_images/IMG_0015.PNG "第一次设置bounds.origin(4)"
[5]: frame_images/IMG_0016.PNG "第二次设置bounds.origin(5)"
[6]: frame_images/IMG_0020.PNG "第一次设置scale=2(14)"
[7]: frame_images/IMG_0017.PNG "恢复scale=1(9)"
[8]: frame_images/IMG_0019.PNG "设置anchorPoint=(0,0)(10)"
[9]: frame_images/IMG_0022.PNG "设置scale=2(18)"

[demo_download]: http://pan.baidu.com/s/1eiKZ0 "downlaod demo code"