---
title: "[cocos2dx]2.2到3.1(3.0)升级帮助"
date: 2014-07-22
tags: 
- cocos2dx
- cocos2dx-3.x
- anchorPoint
- vector
- Sprite
- GLProgram
- Armature
- UILayer
categories: 
- 技术
---


摘要: cocos2dx 是一款**优秀的多平台,专为2D游戏设计的引擎**. 在活跃的开源社区的推进下, 越发稳定和强大. 2.x -> 3.x的更新幅度很大, 性能的提升和功能的丰富也非常明显. 但在享受**进步**的同时,也要承受**迁徙**之苦. 本文主要是总结自己迁徙的经历, 以防大家走弯路.

---

## 基础准备

1. 我一直把VS当作主开发环境, eclipse和xcode作为特定机型的调试环境. 所以, 偷个懒, **假设你也在用VS开发**. 另外两个平台也也都有[正则表达式](http://zh.wikipedia.org/zh-hant/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)替换功能, 大同小异.
2. 在这里, **假设你已经搭好了3.x的VS开发环境**, 能正常运行`HelloWorld`演示.
3. **假设你有一定的正则表达式基础**. 如果没有的话, 可参考这篇速成教程:[正则表达式30分钟入门教程](http://deerchao.net/tutorials/regex/regex.htm)


## 进入正题

cocos2dx 3.1的简要feature更新介绍可参考[这里](https://github.com/cocos2d/cocos2d-x#main-features), 详细的`changelog`可参考[这里](https://github.com/cocos2d/cocos2d-x/blob/v3/CHANGELOG).在升级之前, 建议扫一遍`changelog`( 否则都不知道要做啥...).

接下来, 进入正题啦:

### 语法更新
本文将有非常多字符串替换的步骤, 都在vs2012中进行. vs2012中字符串替换窗口如下:
![此处是vs窗口截图][1]
本文将上图代表的替换表示为:( 以后将不特殊说明 )

```
//regex-replace
\bUIImageView\b ==> ui::ImageView
```
#### <a id="objc2cpp"></a>obj-c式命名 ==> c++命名空间式命名
以下是常见的替换:
```
//regex-replace
//Widget:
\bUIWidget\b                                ==> ui::Widget
\bUIImageView\b                             ==> ui::ImageView
\bUIButton\b                                ==> ui::Button
...                                     
\bUILayout\b                                ==> ui::Layout
\bUILabel\b                                 ==> ui::Text
\bUILayer\b                                 ==> Layer
\bCCObject\b                                ==> Ref
...

\baddTouchEventListener\s*\(\s*(.+)\s*,\s*toucheventselector\s*\((.*)\)\s*\)  ==> addTouchEventListener( CC_CALLBACK_2( $2, $1 ))
\bTouchEventType\b                          ==> ui::Widget::TouchEventType
\bTOUCH_EVENT_([A-Z]+)\b                    ==> ui::Widget::TouchEventType::$1
\baddWidget\b\s*\(                          ==> addChild(
```

注释: `\b`代表单词的分割符. `()`代表被标记的内容, `$1` 代表原始字符串中被标记的内容中的第一段. 具体请参考[在 Visual Studio 中使用正则表达式](http://msdn.microsoft.com/query/dev11.query?appId=Dev11IDEF1&l=ZH-CN&k=k(vs.netregularexpressionhelp)&rd=true).
#### include文件变更
因为包结构的变化, 所以有些组件的定义会未被include.
主要用到的head文件有:

|head|描述|
|--|--|
|`cocos2d.h`|cocos2dx的基本数据类和Node类都包含在里面|
|`ui/CocosGUI.h`| cocos2dx 绝大部分的UI类都包含在内. 2.x版本中, UI类都包含在`cocos-ext.h`中. 所以绝大部分原来引用`cocos-ext.h`的地方都需要引用此文件 |
|`cocostudio/CocoStudio.h`| cocostudio功能相关的类都包含在里面. 最主要的是各种读取json文件的Reader. 其次, 是`Armature`动画. |
|`cocos-ext.h`|相比2.x的`cocos-ext.h`, 此文件做了非常大的精简. 现在主要包括`CCScrollView`及其子类, 另外还有`EditBox`|

#### std::function作为监听函数
虽然, 3.1 的版本仍然支持绝大多数老版本的回调函数方式, 比如: `m_btnSubmit->addTouchEventListener (this,toucheventselector(RewardItemCell::onBtnSubmitClick));`仍然能工作. 但是, 不能保证在将来的3.x版本中仍然如此. 所以, 尽量一次搞定吧. 在[obj-c式命名 ==> c++命名空间式命名](#objc2cpp)章节中, 有批量替换的正则表达式,可作为参考.

### 功能更新
#### 中间层`UILayer`的去除
在2.x版本, `Widget`需要作为`UILayer`的孩子节点才能响应触摸事件; 而在3.x版本中取消了这个限制, `Widget`的响应机制变为跟`Menu`类似. 目前, 我也没有大量的去掉之前的`UILayer`层, 大部分功能仍然正常工作.
#### 键盘响应方式变更
在2.x版本中, 键盘响应的监听是用`CCDirector::sharedDirector()->getKeypadDispatcher()->addDelegate(this);`实现的. 在3.x版本中是这样:

```
auto keyListener = EventListenerKeyboard::create();
keyListener->onKeyReleased = CC_CALLBACK_2(MainScene::keyBackClicked, this);
_eventDispatcher->addEventListenerWithSceneGraphPriority(keyListener, this);
```
监听的取消, 也要做响应的替换.

#### Armature的陷阱
在2.x版本中, 一般通过下面的方式来监听`Armature`的事件. 在监听到`COMPLETE`的时候, 可以将此`Armature`移出场景.
```
CCArmature* swf = CCArmature::create(swfName);
swf->getAnimation()->setMovementEventCallFunc(this, movementEvent_selector(OutCardEffectHelper::onAnimationEnd));
```

但是在3.x中, 我们不能在监听事件的回调函数中做任何可能会导致`release`此对象的操作. 否则, 会导野指针错误.
是不是有点鸡肋? 我们来看看这种情况是怎么发生的:

```sequence
Scheduler->Armature:update(t)
Armature->ArmatureAnimation:update(t)
ArmatureAnimation->this:告诉监听者
Note right of this:假设此时触发了COMPLETE事件
Note right of this: this->remove Armature, release Armature
Armature->Bone: update(t)
Note right of Armature: set `_armatureTransformDirty = false`
Note right of Armature: but now, Armature has been RELEASED!!!
```

我临时用定时器来移除`Armature`的. 具体过程如下:
1. 为`ArmatureAnimation`增加一个公开函数:  `virtual MovementData *getMovementData() const { return _movementData; }`
2. 通过下面的方式来移除: (PS: Lmada 表达式真是好用啊啊啊啊啊 )

```
auto data = swf->getAnimation()->getMovementData();
float speed = data->scale;
float frames = data->duration;
float delay = frames/60/speed;
string id = FORMAT_TEXT( "%p", swf );
Director::getInstance()->getScheduler()->schedule( [swf](float t){swf->removeFromParent();},this,0,0,delay,false, id);
```
#### 更加优雅的anchorPoint
如果你加载了cocostudio生成的ui json文件, 并改动过内部`Widget`的位置的话, 你会发现, 代码中`x=20`跟cocostudio ui编辑器中`x=20`效果是不一样的.
问题出在哪里呢?
简单来说, 2.x版本中的`anchorPoint`对自己在父亲节点的位置和孩子节点在自己中的位置都有影响; 3.x版本中的`anchorPoint`只对自己在父亲节点中的位置有影响.

举个例子. 假设A是父亲节点,B是A的孩子节点. A和B同为10*10的正方形.

```
A.anchorPoint = Point(0.5,0.5);
A.setSize(Size(10,10))
B.anchorPoint = Point(0.5,0.5);
B.setSize(Size(10,10))
B.setPosition( Point::ZERO );
A.addChild(B);
```
在2.x版本中: A和B的位置完全重合
在3.x版本中: B的中心点和A的左下角的点重合.

![此处是位置示意图][2]



#### Sprite的默认GLProgram
什么?你没听过`GLProgram`, 那恭喜你, 可以跳过这小节了. 因为你肯定不会出现下面的恼人情况.

`SHADER_NAME_POSITION_TEXTURE_COLOR`是2.x版本中的默认`GLProgram`. 它对应的vect和frag分别是:`ccPositionTextureColor_vert`和`ccPositionTextureColor_frag`. vect用来确定位置,frag用来确定色彩.  

然而, 在3.x版本中,默认的`GLProgram`是`SHADER_NAME_POSITION_TEXTURE_COLOR_NO_MVP`, 对应的vect和frag分别是:`ccPositionTextureColor_noMVP_vert`和`ccPositionTextureColor_noMVP_frag`.

如果你恰好用过`ccPositionTextureColor_vert`的话, 建议改为`ccPositionTextureColor_noMVP_vert`. 否则会出现莫名的位置偏移问题.


#### stl::vector的不稳定排序导致的层级问题

2.x版本中, 如果没有对孩子设置过`zOrder`的话, 孩子节点的覆盖顺序为: 后加的节点在上层, 先加的节点在下层.

3.x版本中, win32环境下, 孩子节点的覆盖层级还能保续. 但是在android平台下, `zOrder`相同的孩子, 层级顺序是随机的. 关键在下面的代码:

```
void Node::sortAllChildren()
{
    if( _reorderChildDirty ) {
        std::sort( std::begin(_children), std::end(_children), nodeComparisonLess );
        _reorderChildDirty = false;
    }
}
```

win32和android平台对`stl::sort()`的实现不同, android下的排序算法不是稳定的. 解决方法:

1. 修改上述代码, 实现稳定排序
2. 添加孩子节点的时候手动设置`zOrder`来保证层级顺序.


#### umeng等第三方库的更新

第三方库, 也有很多跟cocos2dx版本相关. 注意升级, 否则会闪退.




  [1]: o_Image.png
  [2]: o_Untitled%20Diagram.png




