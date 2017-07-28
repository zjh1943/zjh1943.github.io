---
date: 2014-03-29
title: "[cocos2dx]让CCScrollView支持分页"
tags: 
- cocos2dx
- CCScrollView
- page
- UIScrollView
categories: 
- 技术
---

简介： 做过`IOS`开发的朋友, 肯定知道`UIScrollView`有一个`isPaged`属性. 当设置其为`true`的时候, 滑动会自动分页. 即, 每次滑动之后, 会停止在整页的位置. 当开始介入`cocos2dx`开发的时候, 却发现跟`UIScrollView`接口十分相似的`CCScrollView`却没有这个分页属性. 于是手动实现了一个. 


## 基础知识
在常见的图形引擎中, 滑动组件的定义里有两个重要的概念

* `viewSize`: 这个大小值得是组件占用屏幕的大小. 即实际大小.
* `contentSize`: 这个大小是一个虚拟的大小. 我们之所以要滚动, 必然是因为需要展示的内容比现实的屏幕空间大. 我们需要滚动屏幕, 才能浏览到所有的显示内容. 这个`contentSize`即是我们虚拟出来的, 需要展示的所有内容加起来的大小.
* 分页: 最典型的例子就是iPhone的主界面. 我们不能任意指定一个位置, 让滑动固定在那里. 它要么停留在第一页, 要么停留在第二页, 不会是第0.5页. 每一页的大小是`viewSize`决定的. 那么总页数就是`total_page_count = ceil(viewSize / contentSize )`


## CCScrollView源码查看

我们知道, `cocos2dx`的触摸都是通过`CCTouchDelegate`来实现的. 如果对`cocos2dx`的touch机制不熟悉的, 可以参考博客.  

### 简单介绍ccTouchBegan方法的功能:
`ccTouchBegan`是`cocos2dx`touch机制的第一个方法. 这个方法的接口如下:
```
bool CCScrollView::ccTouchBegan(CCTouch* touch, CCEvent* event)
```
返回的`bool`值, 告诉`cocos2dx`中touch事件的管理者`CCTouchDispatcher`, 当前组件是否处理这一次触摸:  

* 如果返回`true`, 则`CCTouchDispatcher`会根据用户的touch动作, 在后续调用本组件的`ccTouchMoved`, `ccTouchEnded`, `ccTouchCancelled`方法.
* 如果返回`false`, 则表示当前组件不处理此次touch事件. 后续的三个方法不会被调用.

### CCScrollView::ccTouchBegan()解析
无码无真相, 先贴一张代码图:

```
bool CCScrollView::ccTouchBegan(CCTouch* touch, CCEvent* event)
{
    //tag-1
    if (!this->isVisible())
    {
        return false;
    }
    
    //tag-2
    CCRect frame = getViewRect();
    //dispatcher does not know about clipping. reject touches outside visible bounds.
    if (m_pTouches->count() > 2 ||
        m_bTouchMoved          ||
        !frame.containsPoint(m_pContainer->convertToWorldSpace(m_pContainer->convertTouchToNodeSpace(touch))))
    {
        return false;
    }
    ...

    //tag-3
    if (m_pTouches->count() == 1)
    { // scrolling
        m_tTouchPoint     = this->convertTouchToNodeSpace(touch);
        m_bTouchMoved     = false;
        m_bDragging     = true; //dragging started
        m_tScrollDistance = ccp(0.0f, 0.0f);
        m_fTouchLength    = 0.0f;
    }
    //tag-4
    else if (m_pTouches->count() == 2)
    {
        ...
        m_bDragging  = false;
    } 
    return true;
}
```

我把跟本博文主题关系不大的代码用省略号(...)代替了. 下面是相关代码解释:  

* tag-1 当此组件隐藏的时候, 不予处理.
* tag-2 当此触摸点数大于2或者触摸点不在当前组件的显示范围内的时候, 不予处理.
* tag-3 这是我们的重点. 当触摸点数等于1的时候, 处理滑动操作. 
* tag-4 触摸点等于2的时候, 响应缩放处理.



## 源代码的修改

### 变量申明及方法增加
在`CCScrollView.h`文件中增加以下成员变量和方法:

```
//CCScrollView.h
public:
	bool isPaged(){ return m_bPaged; };
	void setPaged( bool value ){ m_bPaged = value; };
protected:
    clock_t m_touchBeganTime;
	int     m_touchBeganOffset;
	int     m_targetPage;
	int     m_currPage;
	float   m_pageAccSpeed;
	float   m_distanceRatioOfTurn;
	bool    m_bPaged;
	
	void __pageTouchBegan();    //在CCScrollView的滑动被触发的时候调用
	bool __pageTouchEnd();      //在ScrollView的滑动停止的时候调用
	void __pageTouchCancel();   //在滑动被取消的时候调用
	void __pageClearTouch();    //在一次滑动结束的时候调用
```

### 方法的实现:
四个方法的代码可能要占用一些篇幅, 所以在这里, 先简要讲一下原理:

1.  在touch开始的时候, 记录一下当时时间`m_touchBeganTime`和开始滑动的位置`m_touchBeganOffset`.
2.  在touch结束的时候, 获取结束时刻的时间和位置. 我们有两个标准来判断应该翻页还是停留在上一页:
    * 如果滑动距离超过一页距离的一半(或者是其他阈值),那么判断为用户希望翻到下一页(或上一页)
    * 如果滑动速度超过一个阈值, 那么判断为用户希望翻到下一页(或上一页)  
3. 当做了判断之后, 即可滑动到对应的位置.

下面是四个方法的实现, 实现了上述原理.


```
void CCScrollView::__pageTouchBegan()
{
	//仅在设置了分页属性, 并且只有一个滑动方向的时候, 才支持分页.
	if( !m_bPaged || ( m_eDirection != kCCScrollViewDirectionHorizontal && m_eDirection != kCCScrollViewDirectionVertical )) return ;

	//记录初试时间和位置
	m_touchBeganTime = clock();
	m_touchBeganOffset = m_eDirection == kCCScrollViewDirectionHorizontal ? getContentOffset().x : getContentOffset().y;

}
bool CCScrollView::__pageTouchEnd()
{
	if( !m_bPaged || ( m_eDirection != kCCScrollViewDirectionHorizontal && m_eDirection != kCCScrollViewDirectionVertical )) return false ;

	//constant
	const float PAGE_DISTENCE = m_eDirection == kCCScrollViewDirectionHorizontal ? getViewSize().width : getViewSize().height ;
	if( PAGE_DISTENCE <= 0 ) return false;

	const float MAX_PAGE = ( m_eDirection == kCCScrollViewDirectionHorizontal ? getContentSize().width : getContentSize().height ) / PAGE_DISTENCE;
	const float MIN_PAGE = 0;

	float currOffset = m_eDirection == kCCScrollViewDirectionHorizontal ? getContentOffset().x : getContentOffset().y;
	float deltaOffset = -(currOffset - m_touchBeganOffset);
	clock_t currTime = clock();
	float speed =  currTime != m_touchBeganTime ? deltaOffset / ( currTime - m_touchBeganTime ) : 0;


	m_targetPage = m_currPage;
	if( abs(deltaOffset) >= TURN_PAGE_MIN_OFFSET_RATIO*PAGE_DISTENCE )
	{//滑动距离大于某一阈值.
		
		if( deltaOffset > 0 )
		{
			m_targetPage = m_currPage + 1;
		}
		else if( deltaOffset < 0 )
		{
			m_targetPage = m_currPage - 1;
		}
	}
	else if( abs(speed) >= TURN_PAGE_SPEED )
	{//速度大于某一阈值.
		if( speed > 0 )
		{
			m_targetPage = m_currPage + 1;
		}
		else if( speed < 0 )
		{
			m_targetPage = m_currPage - 1;
		}
	}
	
	if( m_targetPage > MAX_PAGE ) m_targetPage = MAX_PAGE;
	else if( m_targetPage < MIN_PAGE ) m_targetPage = MIN_PAGE;

	float targetOffset = -m_targetPage*( m_eDirection == kCCScrollViewDirectionHorizontal ? getViewSize().width : getViewSize().height );
	float pageDurateion = 0.5;
	CCPoint targetPointOffset = m_eDirection == kCCScrollViewDirectionHorizontal ? ccp( targetOffset, getContentOffset().y ) : ccp(getContentOffset().x, targetOffset );
	setContentOffsetInDuration(targetPointOffset, pageDurateion);

	m_currPage = m_targetPage;
	
	return true;
}
void CCScrollView::__pageTouchCancel()
{
	if( !m_bPaged || ( m_eDirection != kCCScrollViewDirectionHorizontal && m_eDirection != kCCScrollViewDirectionVertical )) return ;

	__pageClearTouch();

}
void CCScrollView::__pageClearTouch()
{
	//clear所有状态
	m_touchBeganOffset = 0;
	m_touchBeganTime = 0;
	m_targetPage = m_currPage;
}
```


## 修改后的CCScrollView.h, CCScrollView.cpp
这里是修改后的文件, 可以直接下载覆盖.   
[CCScrollView.zip][sourceFileZipBaiduCloud]  
***注意***:上述代码仅在`cocos2dx-2.2.2`和`cocos2dx-2.2.1`版本上验证通过. 其他版本请根据上述原理做适当的修改~~~

[sourceFileZipBaiduCloud]: http://pan.baidu.com/...
> Written with [StackEdit](https://stackedit.io/).