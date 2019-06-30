---
layout: post
title: 浅谈cocos2d-x的内存管理
---
## 前言
和Java不同,c++中需要手动释放分配的内存.  
否则,会造成内存泄漏.  
而在cocos中并不需要我们主动释放.  
它借鉴了objective-c的内存管理的机制,采用了自动释放池来进行内存管理.  
## 分析
在对象的初始化中,经常可以看到如下的代码:  
{% highlight cpp %}
Scene *ret = new (std::nothrow) Scene();
    if (ret && ret->init())
    {
        ret->autorelease();
        return ret;
    }
{% endhighlight %}
autorelease函数就是自动管理内存的关键.  
通过查看内部代码,我们可以看到它的构造如下.  
{% highlight cpp %}
Ref* Ref::autorelease()
{
    PoolManager::getInstance()->getCurrentPool()->addObject(this);
    return this;
}
{% endhighlight %}
不急着往下看,先来了解一下Ref这个类.  

### Ref
引用计数类.  
>  Ref is used for reference count management. If a class inherits from Ref,
>  then it is easy to be shared in different places.

从官方的注释可以知道,如果一个类继承了Ref,就可以实现自动内存管理.  
#### 主要方法  
* void retain(); //引用计数加一  
* void release(); //引用计数减一,如果为0,就被释放  
* Ref* autorelease(); //自动释放  
* unsigned int getReferenceCount() const; //得到引用计数的值  
#### 成员变量  
* unsigned int _referenceCount; //引用计数的值  
* friend class AutoreleasePool; //自动释放池  
由此得知,对象通过自动释放池来管理,  
引用计数是否为0来判断是否需要释放对象.  
我们也同样看一下自动释放池的方法.  
### AutoreleasePool
自动释放池.  
#### 主要方法  
* void addObject(Ref *object); //把对象加到自动释放池中  
* void clear(); //清理自动释放池   
#### 成员变量  
* std::vector<Ref*> _managedObjectArray;  
* std::string _name;   
可以看到,自动释放池使用了一个动态数组来管理内存中的对象.  
自动释放池可以用名字来区分.  
### PoolManager
一个单例实现的池子的管理类.  
#### 主要方法  
* void push(AutoreleasePool *pool); //入栈  
* void pop(); //出栈  
#### 成员变量  
* static PoolManager* s_singleInstance;  //管理器的单例  
* std::vector<AutoreleasePool*> _releasePoolStack; //用来存放池子的栈   
* AutoreleasePool *getCurrentPool() const;  //得到当前池子的对象  
通过查看源代码,我们知道autorelase这个函数其实就是把对象加到自动释放池中.  
那么池子是在何时释放对象的呢?  
通过打断点调试可以知道,在mainloop这个函数调用了这样一句话.  
>  PoolManager::getInstance()->getCurrentPool()->clear();

走进这个函数:  
{% highlight cpp %}
void AutoreleasePool::clear()
{
#if defined(COCOS2D_DEBUG) && (COCOS2D_DEBUG > 0)
    _isClearing = true;
#endif
    std::vector<Ref*> releasings;
    releasings.swap(_managedObjectArray);
    for (const auto &obj : releasings)
    {
        obj->release();
    }
#if defined(COCOS2D_DEBUG) && (COCOS2D_DEBUG > 0)
    _isClearing = false;
#endif
}
{% endhighlight %}
可以得知,最终真正释放对象的是obj->release()这一步.  
## 总结
cocos把对象放到自动释放池中来实现引用计数.  
在一开始创建的时候,引用计数为1.  
如果进行了addchild或者runAction等操作的时候,引用计数加一.  
在游戏的主循环中,每一帧都会调用clear,这时候引用计数减为一.  
如果引用计数变成0,就在下一帧释放.  
所以,引用计数并不是立即释放,在有些情况下还是需要手动释放.  
内存管理是一个非常复杂而且让人头疼的问题,需要花费很多时间和教训才能掌握它.  
整体的机制差不多就到这里了,如果有问题或者错误,还请指出.  

