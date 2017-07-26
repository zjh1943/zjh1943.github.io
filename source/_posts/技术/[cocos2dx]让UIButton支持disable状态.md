---
date:  2014-03-23 22:16
title: "[cocos2dx]让UIButton支持disable状态"
tags: 
- cocos2dx
- uibutton
- button
- disable
- uiwidget
categories: 
- 技术
description: 在cocostudio中添加一个UIButton组件, 我们可以看到通常以一下按钮的三态:normal,pressed,disable. 但是,当我们设置了disable状态之后, 在我们的游戏项目中, 对某个按钮执行button->setEnable(false)后, 按钮居然完全不见了?!
---


摘要: 主要解决cocos2dx-2.2.2版本中, UIButton显示不了disable状态图的问题. 顺便, 理解了一下cocos2dx中UIWidget的渲染原理.

### 发现问题

![enter image description here][1]

在cocostudio中添加一个UIButton组件, 我们可以看到通常以一下按钮的三态:`normal`,`pressed`,`disable`. 但是,当我们设置了`disable`状态之后, 在我们的游戏项目中, 对某个按钮执行`button->setEnable(false)`后, 按钮居然完全不见了?!  

### 解决方法
1. 修改`Widget::visit()`方法, 改为:   
    
    ~~~cpp
    void Widget::visit()
    {
        CCNodeRGBA::visit();
    } 
    ~~~

    即, 删除`if`判断.
2. 修改`Widget::setEnable()`方法, 改为:   

    ~~~cpp 
    void Widget::setEnabled(bool enabled)
    {
        _enabled = enabled;
        if(_widgetChildren && _widgetChildren->count() > 0)
        {
            CCObject* child;
            CCARRAY_FOREACH(_widgetChildren, child)
            {
                ((Widget*)child)->setEnabled(enabled);
            }
        }
    	setBright( enabled );//增加此行
    }
    ~~~ 

    即, 增加`setBright( enabled );`行.


如果想知其所以然的, 下面是解释.
### 解释

#### Widget的诡异渲染
`Widget::visit()`的实现如下:  

```cpp
void Widget::visit()
{
    if (_enabled)
    {
        CCNodeRGBA::visit();
    }    
}
```

如果您没时间仔细看过cocos2dx的源码, 这里可以有一个简单的解释:  

1. `visit()`方法最初是从`CCNode`继承过来的. 也就是说***任何显示对象***都有这个方法. 
2. `CCNode::visit()`就是CCNode的渲染函数, 是视图渲染中最重要的一个函数, 跟IOS开发中的`UIView::draw()`功能类似. 框架在每一帧都会调用这个方法, 而这个方法负责当前组件的`openGL`绘制. 我们可以看下`CCNode::visit()`的实现来作为验证:

    ```cpp
    void CCNode::visit()
    {
        ...
        this->draw();
        ...
    }
    //算了, 这段代码有点长, 有些人时间比较宝贵可能没兴趣看. 我就放在***附录***了.(*^__^*)
    ```

看到这里, 你肯定一经发现问题了: 为什么只有在`_enabled`为`true`的时候才渲染?难道不是坑爹么?`disable`状态就不理了么?  
可能cocos2dx另有打算, 我没理解清楚. 不过这个功能既然不符合我的要求, 我就果断的把那个`if`去掉了. (谁让咱们是程序员呢?).  
  
但是, 问题并没有完全结束. 还是没有显示`disable`状态.  

#### Widget的enable?
`UIButton::setEnable()`方法比较短, 贴出来看看. 

```cpp
void Widget::setEnabled(bool enabled)
{
    _enabled = enabled;
    if(_widgetChildren && _widgetChildren->count() > 0)
    {
        CCObject* child;
        CCARRAY_FOREACH(_widgetChildren, child)
        {
            ((Widget*)child)->setEnabled(enabled);
        }
    }
}
```
发现这段代码除了把`_enable`和所有孩子的`_enable`状态设为`false`, 没做任何事情. 通过查看`widget`的代码, 很容易看到, `widget`的三态是通过下面的是三个函数实现的:

```cpp
void Widget::onPressStateChangedToNormal(){}
void Widget::onPressStateChangedToPressed(){}
void Widget::onPressStateChangedToDisabled(){}
```
不错,他们都是空函数.子类根据需要来实现这三个函数.   
下面是`setBright`的实现:

```cpp
void Widget::setBright(bool bright)
{
    _bright = bright;
    if (_bright)
    {
        _brightStyle = BRIGHT_NONE;
        setBrightStyle(BRIGHT_NORMAL);
    }
    else
    {
        onPressStateChangedToDisabled();
    }
}
```

由上代码可以看出. 而我们调用`setBright(false)`, 即可根据`widget`当前的状态, 渲染不同的图像.


### 附录
#### 代码片段一`CCNode::visit()`

```cpp
void CCNode::visit()
{
    // quick return if not visible. children won't be drawn.
    if (!m_bVisible)
    {
        return;
    }
    kmGLPushMatrix();

     if (m_pGrid && m_pGrid->isActive())
     {
         m_pGrid->beforeDraw();
     }

    this->transform();

    CCNode* pNode = NULL;
    unsigned int i = 0;

    if(m_pChildren && m_pChildren->count() > 0)
    {
        sortAllChildren();
        // draw children zOrder < 0
        ccArray *arrayData = m_pChildren->data;
        for( ; i < arrayData->num; i++ )
        {
            pNode = (CCNode*) arrayData->arr[i];

            if ( pNode && pNode->m_nZOrder < 0 ) 
            {
                pNode->visit();
            }
            else
            {
                break;
            }
        }
        // self draw
        this->draw();

        for( ; i < arrayData->num; i++ )
        {
            pNode = (CCNode*) arrayData->arr[i];
            if (pNode)
            {
                pNode->visit();
            }
        }        
    }
    else
    {
        this->draw();
    }

    // reset for next frame
    m_uOrderOfArrival = 0;

     if (m_pGrid && m_pGrid->isActive())
     {
         m_pGrid->afterDraw(this);
    }
 
    kmGLPopMatrix();
}
```


  [1]: https://lh6.googleusercontent.com/-bwcB18CZFyo/Uy7Ywf_ew9I/AAAAAAAACSo/G1QRO-a4z9Y/s0/QQ%E6%88%AA%E5%9B%BE20140323205043.png "cocostudio-capture-button-disable.png"