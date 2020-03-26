---
layout: post
title: 定制自己的GDB界面 
---

## 前言

以前都是OD用的比较多，也就是最近才开始用GDB，因为需要调试Linux上的程序。

GDB默认不好用，所以需要Peda，Gef，Pwndbg这些插件。

虽然使用了插件以后调试信息更加丰富，但是使用体验非常不好。

对比OD，有如下几个缺点。

* 1.布局非常不合理，浪费空间的同时关键的反汇编窗口只能看到寥寥几行，分析起来非常难受

* 2.因为是竖着排列的，所以要看上一条命令的输出就很难。

* 3.有时候命令输多了，其他窗口就被挤占了空间，调试起来更加难受

想起来Emacs里的GDB默认有个图形界面，于是想着针对它改造一下

## 效果图

![](/images/gdb_custom.jpg)

根据我自己的需求，使反汇编窗口最大化，右上角是寄存器窗口，左下用来输入和查看GDB命令，有下角是IO窗口。

这些只是初步的设置，只能达到最基本的使用需求。

堆栈窗口的显示，反汇编窗口的语法高亮，调试信息的增加这些都需要好好研究下。

### 配置

在*user-config*中加入如下代码即可。

快捷键设置上我模仿了OD的快捷键。

```c

(defun dotspacemacs/user-config ()
  (set-frame-parameter (selected-frame) 'alpha '(90 92)) ;设置窗口透明度90%

  (global-set-key (kbd "<f2>") 'gdb) ;run gdb 
  (global-set-key (kbd "<f4>") 'gud-cont) ;continue
  (global-set-key (kbd "<f5>") 'gud-go) ;run program
  (global-set-key (kbd "<f7>") 'gud-stepi) ; si
  (global-set-key (kbd "<f8>") 'gud-nexti) ; ni
  
  (setq gdb-many-windows t) ; 默认开启多窗口模式
  
  (defadvice gdb-setup-windows (after activate)
    (gdb-setup-my-windows) ; 重新修改布局
    )

  (defun gdb-setup-my-windows ()
    (set-window-dedicated-p (selected-window) nil)
    (switch-to-buffer gud-comint-buffer)
    (gdb-display-disassembly-buffer) ; 打开反汇编窗口
    (gdb-display-io-buffer) ; 打开输入输出窗口 
    (delete-other-windows)
    (let
        ((win0 (selected-window))             
         (win1 (split-window-horizontally
                (floor (* 0.67 (window-width)))))   
         (win2 (split-window-vertically
                (floor (* 0.8 (window-body-height))))) 
         )
      (gdb-set-window-buffer (gdb-get-buffer-create 'gdb-disassembly-buffer))
      (select-window win1)
      (split-window-vertically (floor (* 0.6 (window-body-height))))
      (gdb-set-window-buffer (gdb-get-buffer-create 'gdb-registers-buffer))
      (other-window 1)
      (gdb-set-window-buffer (gdb-get-buffer-create 'gdb-inferior-io))
      (select-window win2)
      )
    )
  )
```

