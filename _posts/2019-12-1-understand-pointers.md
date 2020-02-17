---
layout: post
title: 理解指针
---

## 什么是指针

指针是一种数据结构，它指向的是数据的地址。



## 指针类型


指针的类型有 $$void*，char* ，short*，int*$$  这些。

类型只是影响了取出元素的字节数，并不影响元素本身的大小。

看如下代码，定义了三个不同类型的指针

```c
#include<stdio.h>
int main()
{
    int a=0x12345678;
    printf("%x %x %x \n",*(int*)&a,*(short*)&a,*(char*)&a);
    getchar();
    return 0;
}

```

运行结果如下：

![](/images/points3.png)



不管指针的数据类型是什么，它所指向的地址都是不变的。

### 指针和函数

函数名保存了函数的首地址，所以函数也可以用指针来寻址。

#### 函数指针

用来回调函数，或者使用函数。

下面举了一个简单的例子来说明它的用法。

```c
#include <stdio.h>
void hello()
{
	printf("hello\n");

}
int hello_arg(int a,int b)
{
	printf("%x\n",a+b);

}	


int main()
{
	void (*p)(void)=&hello;
	p();
	int (*q)(int,int)=&hello_arg;
	q(0x1,0x1);
	return 0;
}

```


#### 指针函数

返回值类型为指针的函数，没什么好讲的。

### 指针和数组

数组名保存的是数组的首地址，而且数组的元素在内存中是从低到高排列的，所以数组可以用指针来寻址。

## 指针的符号运算

### 引用

 & 取地址

 \* 间接引用，取出地址中的值 

为什么我不直接说取值呢？
因为内存中存放的不一定是值，也有可能是地址。

### 加法和减法

指针的运算本质上来说是地址的运算。

指针相减，含义是两个地址的相减。
在数组中，可以用来算出元素相差的个数

指针的加法，可以用来索引数组

可以看一下如下代码：

```c
#include<stdio.h>
int main()
{
	int a[2][3][4]={0};	
	printf("%p %p %p\n",&a,a+2,*a+1,(*(*a+1)+2)+3);
	printf("0x%x 0x%x 0x%x\n",*(a+2),**a+1,*(*(a+1)+2)+4);
	getchar();
		
	return 0;
}
```

运行结果如下：

![](/images/points.png)

打开OD，我们来分析一下这个代码。

首先，我们定义了一个 $$96(2*3*4*sizeof(int))$$ 个字节大小的数组。

汇编代码如下:
```
0040D75E  |.  C745 A0 00000>mov     dword ptr ss:[ebp-60], 0   //首地址赋值为0
0040D765  |.  B9 17000000   mov     ecx, 17 //取剩下的23个数，注意是16进制
0040D76A  |.  33C0          xor     eax, eax
0040D76C  |.  8D7D A4       lea     edi, dword ptr ss:[ebp-5C] 
0040D76F  |.  F3:AB         rep     stos dword ptr es:[edi] //剩下23个数赋值为0
```

查看堆栈 

```
0012FF20   00000000
0012FF24   00000000
0012FF28   00000000
0012FF2C   00000000
0012FF30   00000000
0012FF34   00000000
0012FF38   00000000
0012FF3C   00000000
0012FF40   00000000
0012FF44   00000000
0012FF48   00000000
0012FF4C   00000000
0012FF50   00000000
0012FF54   00000000
0012FF58   00000000
0012FF5C   00000000
0012FF60   00000000
0012FF64   00000000
0012FF68   00000000
0012FF6C   00000000
0012FF70   00000000
0012FF74   00000000
0012FF78   00000000
0012FF7C   00000000
0012FF80  /0012FFC0
0012FF84  |00401259  返回到 ponints.00401259 来自 ponints.00401005
```

0012FF20 ~ 0012FF7c 里面存放的是数组的元素，24个从低到高排列

0012FF80 里面是放的是上个函数也就是main函数的ebp

0012FF84 里面放的是 ***返回地址***

然后来解释一下后面printf中表达式的值。

指针是有宽度的。

指针和常量相加时，应该保证数据宽度是一样的,所以常量会被转成指针相同的长度。

在三维数组中，a+1 等价于 $$a +1 * 3 * 4 * sizeof(int) $$ 

在二维数组中，*a+1 等价于$$ a + 4 * sizeof(int)$$

在一维数组中，**a+1 等价于$$ a + 1 * sizeof(int)$$
     
根据上面可以得出规律，n维指针的大小为后面n-1位相乘，并且每一维指针都可以当作一个新的数据类型。

虽然说，有些表达式中指针已经访问到数组之外的元素了，但是它是合法的。

c语言并不会对数组越界做检查，这也是常常引起错误的原因。

同理，上面的式子就可以很好的被理解了。感兴趣的可以自己算一下。

### 扩展

既然我们可以用指针访问和修改任意元素，能否修改返回地址呢？

当然是可以的，看下面这个程序。

```c
#include<stdio.h>
void hello()
{
  printf("The Function will be run ...");
  getchar();
}
int main()
{
  int a[2][3][4]={0};
  printf("%p %p %p %p\n",&a,a+1,*a+1,(*(*a+1)+2)+3);
  printf("0x%x 0x%x 0x%x\n",a+2,**a+1,*(*(a+1)+2)+4);
  *(*(*(a+1)+2)+5)=&hello; // a[1][2][5]
  return 0;
}

```
运行结果：

![](/images/points2.png)

观察上面的堆栈或者计算一下就可以知道，a[1][2][5]里面放的是 ebp+4 也就是返回地址。

利用指针可以很轻松的修改函数的返回地址。

当然这样明显是存在安全隐患的，所以高版本的编译器禁止了这个操作。

## 指针和引用

c++ 中，推荐使用引用来代替指针。

从本质上来说，引用和指针没有任何区别。

从语法上来讲,可以把引用看成阉割版的指针。

为什么这么说呢？

看代码
```c
#include<stdio.h>
int main()
{

    char a=0xff;
    char* p=&a;
    char& q=a;

    return 0;
}

```

```
00401028  |.  C645 FC FF    mov     byte ptr ss:[ebp-4], 0FF
0040102C  |.  8D45 FC       lea     eax, dword ptr ss:[ebp-4]
0040102F  |.  8945 F8       mov     dword ptr ss:[ebp-8], eax
00401032  |.  8D4D FC       lea     ecx, dword ptr ss:[ebp-4]
00401035  |.  894D F4       mov     dword ptr ss:[ebp-C], ecx
```

观察可以发现，使用指针和引用生成的汇编代码是完全一样的。

虽然说 c++ 尝试从语法上使指针的使用更加安全，但是提高了复杂性。


