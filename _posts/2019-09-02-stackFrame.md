---
layout: post
title: 理解栈帧
---
## 前言

最近看书的时候讲到了栈帧，下面将会通过调试的手段对栈帧有一个进一步的认识.  

## 栈帧的产生

何谓栈帧呢?

我们都知道,调用函数的时候就会开辟一个新的栈空间。

比如进入Main函数,就会开辟一块栈空间

其中，ebp到esp这段空间就被称为栈帧。

每次调用新的函数时，保存当前栈基地址，创建新的栈帧。


## 栈帧的调试

先来看一下程序的源代码.  
这个一个很简单的程序,  
源码如下:  
```c++
#include "stdio.h"
long add(long a, long b)
{
    long x = a, y = b;
    return (x + y);
}

int main(int argc, char* argv[])
{
    long a = 1, b = 2;
    add(a,b);
    return 0;
}
```

经vs2013编译后,  
把程序拖到OD中进行调试.  
经过启动函数,来到main函数入口处.  
ebp的值为0018FF80,esp的值为0018FF40.  

![image](/images/stackframe1.PNG)  

调用main函数以后:  

![image](/images/stackframe2.PNG)  

ebp的值没有变化，而esp的值减少为0018FF3C.  
减少了4个字节,也就是指向了上面一个单元.  
然后执行:  
* push ebp ;保存上一个栈的栈帧
* mov ebp,esp ;创建新的栈帧  

可以很惊奇的发现，新创建的栈的基地址其实就在原来栈的顶部,  
也就是说,  
两个栈是相邻的,  
这就是esp减少了4个字节的原因.  
这样做应该是为了节省内存.  
而且,  
进行参数的复制和传递时,  
也不需要移动太大距离,  
大大节省了时间.  

* mov dword ptr ss:[ebp-8],1
* mov dword ptr ss:[ebp-4],2  

紧接着是将参数逆序压栈,  
执行完以后栈中的值:  

![image](/images/stackframe3.PNG)  

* mov eax,dword ptr ss:[ebp-4]
* push eax
* mob ecx,dword ptr ss:[ebp-8]
* push ecx  

执行后寄存器的值如下:  

![image](/images/stackframe4.PNG)  

这就是将两个参数的值存到寄存器中,  
准备执行add函数.  
add函数的分析大同小异,这里就步过了.  
继续往下走.  

* add esp 8  

这一步就是将函数内的参数清空.  
x和y总共占了8个字节，所以esp加8  

* xor eax,eax   

函数返回值为0，这个没什么好讲的  

* mob esp,ebp ;清空栈
* pop ebp ;弹出栈帧,返回到原来的栈中去  
* retn ；返回函数

这样,栈帧的调用流程就大体结束了  
## 栈帧的思考
其实,还有一点要补充的,  
esp的变化只是简单的限制了栈的范围,  
并不是真正意义上增删栈中的数据.  
数据还在,只不过访问不到而已，  
这样设计的原因是为了减少清除栈中数据的开销.  
反正数据会被覆盖掉.  

