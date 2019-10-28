---
layout: post
title: 8086汇编环境的搭建 下 
---
## 回顾一下
通过上一篇文章,我们对8086汇编有了一个初步的认识.  
接下来,就真正的来写一个程序,以此加深对编译运行过程的理解.  
## 怎么又是Hello,World!
众所周知,学习一门语言都是从HelloWorld开始入门的.  
(这是来自c语言之父的一个很有名的梗.  
我们也不例外.  
为了方便,我们使用notepad++来编写汇编代码.  
这是我提前花了几分钟写的一个demo.  

#### 功能:在屏幕中间显示HelloWorld
#### 文件名:hello.asm(之所以不起名HelloWorld.asm是因为文件名不能超过8个字符) 
代码如下:  
{% highlight m68k %}
assume cs:code 
data segment
	db 'Hello,World!'
data ends
code segment
start:mov ax,data
      mov ds,ax
      mov ax,0b800h ;显存缓存区地址
      mov es,ax
      mov cx,12 ;字符串长度
      mov si,0
      mov di,160*12+40*2-12 ;显示到屏幕正中间,屏幕大小25*80		
s:    mov bl,[si]
      mov al,bl
      mov ah,2 ;绿色
      mov es:[di],ax
      add di,2
      inc si
      loop s
      mov ax,4c00h
      int 21h
code ends
end start
{% endhighlight %}
**如果利用系统的中断这个程序会更短**  
## 编译->链接->运行
通过masm.exe对整个代码进行编译,  
```shell
 masm hello;  (加分号是为了省去中间步骤,节约时间)  
```

通过link.exe对代码进行链接

```shell
 link hello;
```

最后运行代码 
```shell
 hello.exe
```

整个过程可以表示为:  
hello.asm->hello.obj->hello.exe   
当然编译之前我们需要添加临时环境变量.  
(全局的也行  
如图:  

![图片](/images/options_2.PNG)

最后运行的效果:  

![gif](/images/result.gif)

