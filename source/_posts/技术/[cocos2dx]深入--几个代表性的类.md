---
title: "[cocos2dx]深入了解几个代表性的类"
date: 2014-07-22
tags: 
- cocos2dx
- openGL
- GLView
- Application
-  Director
-  class
-  Renderer
-  Node
categories: 
- 技术
---



摘要: 此文对`cocos2d-x`引擎中最具代表性,最能体现框架结构的几个类做了简单的介绍, 包括`Director`,`Application`, `Renderer`, `EventDispatcher`, `Scheduler`. 对于这些类, 也只对关系主要流程的方法做了介绍, 略过了容错代码和其它细节. 主要目的是让大家快速的对`cocos2d-x`引擎有一个全面笼统的认识, 也方便快速定位问题.


---


## GLView
`cocos2d-x`对`openGL`的封装. 不同平台下, `openGL`有一些差别.

### openGL

#### 一段简单的例子

以下内容引用自[Introduction to OpenGL](http://www.glprogramming.com/red/chapter01.html). 需要更具体的介绍也可参考这个链接.
```  
# include <whateverYouNeed.h>
main() {
   InitializeAWindowPlease();
   
   glClearColor (0.0, 0.0, 0.0, 0.0);
   glClear (GL_COLOR_BUFFER_BIT);
   
   glColor3f (1.0, 1.0, 1.0);
   glOrtho(0.0, 1.0, 0.0, 1.0, -1.0, 1.0);
   glBegin(GL_POLYGON);
      glVertex3f (0.25, 0.25, 0.0);
      glVertex3f (0.75, 0.25, 0.0);
      glVertex3f (0.75, 0.75, 0.0);
      glVertex3f (0.25, 0.75, 0.0);
   glEnd();
   glFlush();
   
   UpdateTheWindowAndCheckForEvents();
}
```
#### OpenGL Command Syntax

* OpenGL commands use the prefix `gl` and initial capital letters for each word making up the command name 
* some seemingly extraneous letters appended to some command names (for example, the `3f` in `glColor3f()` and `glVertex3f()`)

#### OpenGL as a State Machine
OpenGL is a state machine. You put it into various states (or modes) that then remain in effect until you change them.





## Application

### 主要方法:
```
virtual const char * getCurrentLanguage();
virtual Platform getTargetPlatform();
virtual void setAnimationInterval(double interval);
int run();//启动主循环
```
### run()函数

```
int Application::run()
{

    ...

    while(!glview->windowShouldClose())
    {
        QueryPerformanceCounter(&nNow);
        if (nNow.QuadPart - nLast.QuadPart > _animationInterval.QuadPart)
        {
            nLast.QuadPart = nNow.QuadPart;
            
            director->mainLoop();       //Director进行这一帧的渲染
            glview->pollEvents();       // This function processes only those events that have already been received and then returns immediately.
        }
        else
        {
            Sleep(0);
        }
    }

    ...
    
    return true;
}
```


## Director

### 主要函数预览
```
//openGL Matrix Operate
    void pushMatrix(MATRIX_STACK_TYPE type);
    void popMatrix(MATRIX_STACK_TYPE type);
    void loadIdentityMatrix(MATRIX_STACK_TYPE type);
    void loadMatrix(MATRIX_STACK_TYPE type, const Mat4& mat);
    void multiplyMatrix(MATRIX_STACK_TYPE type, const Mat4& mat);
    Mat4 getMatrix(MATRIX_STACK_TYPE type);
    void resetMatrixStack();
    
//View Data
    inline double getAnimationInterval();
    inline bool isDisplayStats();
    inline GLView* getOpenGLView();
    inline Projection getProjection();
    Size getVisibleSize() const;
    
    Vec2 getVisibleOrigin() const;
    Vec2 convertToGL(const Vec2& point);
    Vec2 convertToUI(const Vec2& point);
    float getZEye() const;

// Scene 场景管理
    inline Scene* getRunningScene();
    void runWithScene(Scene *scene);
    void pushScene(Scene *scene);
    
    
// 控制绘制的暂停和恢复
    void end();
    void pause();
    void resume();
    
//绘制图形(界面展示最重要的函数)
    void drawScene();
    
//Getter and Setter
    Scheduler* getScheduler() const { return _scheduler; }
    void setScheduler(Scheduler* scheduler);

    ActionManager* getActionManager() const { return _actionManager; }
    void setActionManager(ActionManager* actionManager);
    
    EventDispatcher* getEventDispatcher() const { return _eventDispatcher; }
    void setEventDispatcher(EventDispatcher* dispatcher);

    Renderer* getRenderer() const { return _renderer; }
```

### drawScene(): 主要绘制函数

```
// Draw the Scene
void Director::drawScene()
{
    ...
    
    if (! _paused)
    {
        _scheduler->update(_deltaTime);                         //Scheduler 定时器 更新
        _eventDispatcher->dispatchEvent(_eventAfterUpdate);     //Dispatcher 抛发事件.
    }

    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);         //glClear

    if (_nextScene)                                             //取得下一个将要显示的Scene.
    {
        setNextScene();
    }

    pushMatrix(MATRIX_STACK_TYPE::MATRIX_STACK_MODELVIEW);      //将上一次绘制的Context放到堆栈

    // draw the scene
    if (_runningScene)
    {
        _runningScene->visit(_renderer, Mat4::IDENTITY, false);
        _eventDispatcher->dispatchEvent(_eventAfterVisit);
    }

    _renderer->render();                                        //渲染
    _eventDispatcher->dispatchEvent(_eventAfterDraw);

    popMatrix(MATRIX_STACK_TYPE::MATRIX_STACK_MODELVIEW);       //返回到上一次绘制时的状态.


    // swap buffers
    if (_openGLView)
    {
        _openGLView->swapBuffers();                             //把上面渲染的结果显示到屏幕
    }
    
    ...
}
```

## Node::visit() 函数

### 预览
Node::visit() 的主要功能就是

1. 调用所有孩子的`visit`函数
2. 调用`self->draw()`函数
```
void Node::visit(Renderer* renderer, const Mat4 &parentTransform, uint32_t parentFlags)
{
    // quick return if not visible. children won't be drawn.
    if (!_visible)
    {
        return;
    }

    uint32_t flags = processParentFlags(parentTransform, parentFlags);

    // IMPORTANT:
    // To ease the migration to v3.0, we still support the Mat4 stack,
    // but it is deprecated and your code should not rely on it
    Director* director = Director::getInstance();
    director->pushMatrix(MATRIX_STACK_TYPE::MATRIX_STACK_MODELVIEW);
    director->loadMatrix(MATRIX_STACK_TYPE::MATRIX_STACK_MODELVIEW, _modelViewTransform);

    int i = 0;

   
    if(!_children.empty())
    {
        sortAllChildren();
        // draw children zOrder < 0
        for( ; i < _children.size(); i++ )
        {
            auto node = _children.at(i);

            if ( node && node->_localZOrder < 0 )
                node->visit(renderer, _modelViewTransform, flags);
            else
                break;
        }
        // self draw
        this->draw(renderer, _modelViewTransform, flags);

        for(auto it=_children.cbegin()+i; it != _children.cend(); ++it)
            (*it)->visit(renderer, _modelViewTransform, flags);
    }
    else
    {
        this->draw(renderer, _modelViewTransform, flags);
    }

    director->popMatrix(MATRIX_STACK_TYPE::MATRIX_STACK_MODELVIEW);
}
```

### Node::draw()
因为`Node`是所有可显示对象的父类, 没有任何显示内容, 所以`draw`函数为空.
这里我们以`Sprite::draw`函数为例简单介绍下`draw`的作用.

```
void Sprite::draw(Renderer *renderer, const Mat4 &transform, uint32_t flags)
{
    // Don't do calculate the culling if the transform was not updated
    _insideBounds = (flags & FLAGS_TRANSFORM_DIRTY) ? renderer->checkVisibility(transform, _contentSize) : _insideBounds;

    if(_insideBounds)
    {
        _quadCommand.init(_globalZOrder, _texture->getName(), getGLProgramState(), _blendFunc, &_quad, 1, transform);
        renderer->addCommand(&_quadCommand);
    }
}
```
我们看到, `Sprite::draw`函数主要实现了[添加一个`QuadCommand`到`Render`中去]的功能.
再看看`Label`的绘制函数.

### Label::draw

```
void Label::draw(Renderer *renderer, const Mat4 &transform, uint32_t flags)
{
    // Don't do calculate the culling if the transform was not updated
    _insideBounds = (flags & FLAGS_TRANSFORM_DIRTY) ? renderer->checkVisibility(transform, _contentSize) : _insideBounds;

    if(_insideBounds) {
        _customCommand.init(_globalZOrder);
        _customCommand.func = CC_CALLBACK_0(Label::onDraw, this, transform, flags);
        renderer->addCommand(&_customCommand);
    }
}
```

其实, 跟`Sprite::draw`也差不多. 关键在于这个`RenderCommand`怎么构造和执行的.


## Renderer 渲染器
### 主要函数预览


```
    void initGLView();

    /** Adds a `RenderComamnd` into the renderer */
    void addCommand(RenderCommand* command);

    /** Adds a `RenderComamnd` into the renderer specifying a particular render queue ID */
    void addCommand(RenderCommand* command, int renderQueue);

    /** Pushes a group into the render queue */
    void pushGroup(int renderQueueID);

    /** Pops a group from the render queue */
    void popGroup();

    /** Creates a render queue and returns its Id */
    int createRenderQueue();

    /** Renders into the GLView all the queued `RenderCommand` objects */
    void render();
```
可见它主要由两个功能:

1. 对`ReanderCommand`进行排序和分类管理
2. 进行渲染:`render()`


### 渲染函数Renderer::render()

```
void Renderer::render()
{
   ...
    
    if (_glViewAssigned)
    {
       ...
        //排列渲染队列
        for (auto &renderqueue : _renderGroups)
        {
            renderqueue.sort();
        } 
        //进行渲染
        visitRenderQueue(_renderGroups[0]);
        ...
    }
    ...
}
```
### Renderer::visitRenderQueue

按照顺序执行所有的 `RenderCommand`
```
void Renderer::visitRenderQueue(const RenderQueue& queue)
{
    ssize_t size = queue.size();
    
    for (ssize_t index = 0; index < size; ++index)
    {
        auto command = queue[index];
        auto commandType = command->getType();
        if(RenderCommand::Type::QUAD_COMMAND == commandType)
        {
            auto cmd = static_cast<QuadCommand*>(command);
            //Batch quads
            if(_numQuads + cmd->getQuadCount() > VBO_SIZE)
            {
                drawBatchedQuads();
            }
            
            _batchedQuadCommands.push_back(cmd);
            
            memcpy(_quads + _numQuads, cmd->getQuads(), sizeof(V3F_C4B_T2F_Quad) * cmd->getQuadCount());
            convertToWorldCoordinates(_quads + _numQuads, cmd->getQuadCount(), cmd->getModelView());
            
            _numQuads += cmd->getQuadCount();

        }
        else if(RenderCommand::Type::GROUP_COMMAND == commandType)
        {
            flush();
            int renderQueueID = ((GroupCommand*) command)->getRenderQueueID();
            visitRenderQueue(_renderGroups[renderQueueID]);
        }
        else if(RenderCommand::Type::CUSTOM_COMMAND == commandType)
        {
            ...
        }
        ...
    }
}
```

### openGL VAO, VBO 介绍.

[GLSL渲染语言入门与VBO、VAO使用：绘制一个三角形](http://blog.csdn.net/xiajun07061225/article/details/7628146)
[OpenGL 4.0 VAO VBO 理解](http://blog.csdn.net/zhuyingqingfen/article/details/19238651)


## Schelduler介绍
`Schelduler`是`cocos2d-x`中实现延迟调用,定时调用时最重要的功能. 类似于其他语言中的`Timer`
他最核心的函数就是:

```
void schedule(const ccSchedulerFunc& callback, void *target, float interval, unsigned int repeat, float delay, bool paused, const std::string& key);
```

用来启动一个定时操作: 在延迟`delay`时间后, 每隔`repeat`时间, 调用一次`callback`. `target`用来标记这个操作属于谁, 方便管理, 比如在析构的时候调用`void unschedule(void *target)`即可移除当前对象的所有`定时操作`.

`Schelduler`的其它大部分方法, 要么是它的衍生, 为了减少调用参数; 要么是对`定时操作`的控制, 比如暂停, 恢复, 移除等. 如果只对想对框架的各个模块有大概的了解, 可以不做深入.


## EventDispatcher
(后续添加)





