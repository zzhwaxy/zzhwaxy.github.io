---
layout: post
title: ActionManager的分析
---
## 前言
上一期讲计时器的时候提到了动作管理器,这篇文章就来好好谈一下它. 

{% highlight cpp %}
typedef struct _hashElement
{
    struct _ccArray     *actions;
    Node                *target;
    int                 actionIndex;
    Action              *currentAction;
    bool                currentActionSalvaged;
    bool                paused;
    UT_hash_handle      hh;
} tHashElement;
{% endhighlight %}

### 动作的创建

先来看一下这个结构体,里面封装了对象拥有的动作的集合和其他一些动作的属性. 
动作创建的时候是走了管理器的addAction函数的,如下:

{% highlight cpp %}
void ActionManager::addAction(Action *action, Node *target, bool paused)
{
    CCASSERT(action != nullptr, "action can't be nullptr!");
    CCASSERT(target != nullptr, "target can't be nullptr!");
    if(action == nullptr || target == nullptr)
        return;
    tHashElement *element = nullptr;
    // we should convert it to Ref*, because we save it as Ref*
    Ref *tmp = target;
    HASH_FIND_PTR(_targets, &tmp, element);
    if (! element)//封装动作并放入哈希表
    {
        element = (tHashElement*)calloc(sizeof(*element), 1);
        element->paused = paused;
        target->retain();//引用计数加一
        element->target = target;
        HASH_ADD_PTR(_targets, target, element);
    }
     actionAllocWithHashElement(element);
     CCASSERT(! ccArrayContainsObject(element->actions, action), "action already be added!");
     ccArrayAppendObject(element->actions, action);
     action->startWithTarget(target);//开始动作
}
{% endhighlight %}

action是要执行的动作,target是动作执行者. 
可以发现,动作执行的对象是Ref类型的. 
那么它就被放到内存管理中. 
### 动作的更新

先看一下管理器的更新: 

{% highlight cpp %}
 void ActionManager::update(float dt)
{
    for (tHashElement *elt = _targets; elt != nullptr; )
    {
        _currentTarget = elt;
        _currentTargetSalvaged = false;
        if (! _currentTarget->paused)
        {
            // The 'actions' MutableArray may change while inside this loop.
            for (_currentTarget->actionIndex = 0; _currentTarget->actionIndex < _currentTarget->actions->num;
                _currentTarget->actionIndex++)
            {
                _currentTarget->currentAction = static_cast<Action*>(_currentTarget->actions->arr[_currentTarget->actionIndex]);
                if (_currentTarget->currentAction == nullptr)
                {
                    continue;
                }
                _currentTarget->currentActionSalvaged = false;
                _currentTarget->currentAction->step(dt);
                if (_currentTarget->currentActionSalvaged)
                {
                    // The currentAction told the node to remove it. To prevent the action from
                    // accidentally deallocating itself before finishing its step, we retained
                    // it. Now that step is done, it's safe to release it.
                    _currentTarget->currentAction->release();
                } else
                if (_currentTarget->currentAction->isDone())
                {
                    _currentTarget->currentAction->stop();
                    Action *action = _currentTarget->currentAction;
                    // Make currentAction nil to prevent removeAction from salvaging it.
                    _currentTarget->currentAction = nullptr;
                    removeAction(action);
                }
                _currentTarget->currentAction = nullptr;
            }
        }
        // elt, at this moment, is still valid
        // so it is safe to ask this here (issue #490)
        elt = (tHashElement*)(elt->hh.next);
        // only delete currentTarget if no actions were scheduled during the cycle (issue #481)
        if (_currentTargetSalvaged && _currentTarget->actions->num == 0)
        {
            deleteHashElement(_currentTarget);
        }
        //if some node reference 'target', it's reference count >= 2 (issues #14050)
        else if (_currentTarget->target->getReferenceCount() == 1)
        {
            deleteHashElement(_currentTarget);
        }
    }
    // issue #635
    _currentTarget = nullptr;
} 
{% endhighlight %}

可以看到,每一次更新,管理器遍历了存放动作的集合, 
如果当前动作不为空, 
就调用step函数来实现具体的更新逻辑. 
如果动作结束了,就把它停止并且从管理器中移除. 
可能有人就要问了,管理器是怎么知道动作是否结束呢? 
别着急,下面就来讲一下. 
### 动作的结束
通过查看继承关系可以知道,动作有actionInterval(持续动作)和 
actionInstant(瞬时动作). 
这两个都继承了FiniteTimeAction(加了时间的动作类). 
FiniteTimeAction又继承了Action(最基本的动作类). 
瞬时动作没什么好说的,直接看持续动作的更新类:

{% highlight cpp %}

void ActionInterval::step(float dt)
{
    if (_firstTick)//第一次触发
    {
        _firstTick = false;
        _elapsed = MATH_EPSILON;
    }
    else
    {
        _elapsed += dt;//持续时间增加
    }
    float updateDt = std::max(0.0f,                                  // needed for rewind. elapsed could be negative
                              std::min(1.0f, _elapsed / _duration)
                              );//更新的时间,范围0到1
    if (sendUpdateEventToScript(updateDt, this)) return;
    this->update(updateDt);
    _done = _elapsed >= _duration;
}
{% endhighlight %}

最后一行代码是重点,如果持续的时间大于设定的总时间, 
那么这个动作被标记完成,会被管理器干掉. 
### 疑问

可以发现一个细节: 
判断是放在更新之后的, 
也就是说动作是在下一帧才真正移除的. 
具体是什么原因我也不清楚, 
有知道的可以告诉下我哈. 
