---
layout: post
title: 欢迎来到我的新博客
---

## 入坑jekyll
  
  以前把博客部署在用vps搭建的网站上,然而那个编辑界面很不友好,于是我投向了Github Page + Jekyll 的怀抱 
  
  Jekyll 是什么呢? 
  
  是用Ruby写的静态博客生成器,使用它可以快速搭建自己的播客 
  
  而且不用花域名和服务器的冤枉钱,也节省了网站的的维护时间和精力 
  
  虽然后来事实证明是不存在的,因为只要网站存在就一定要花时间维护和更新
  
## 快速上手
  
  Jekyll的官网提供了非常详细的[官方手册](https://jekyllcn.com/docs/home/) 
  
  还有一份教程也不错 [链接](https://gist.github.com/biezhi/f88be58ef4ae0f3741bb36ab8daa53c5)
  
  你可以阅读这些教程再加上自己的构思和喜好,搭建出属于你自己风格的网站 
  
  下面介绍一下我自己博客的目录结构吧
  
  主要页面
  
--- 
* index.html    主页面
* tags.html    博客分类(还没加)
* feed.xml    RSS订阅
* about.html    自我介绍
* 404.html    错误链接的跳转

---

文件夹

---
* _includes/ 存放模版头文件
* _posts/ 文章目录
* images/ 引用图片目录
* _layouts/ 布局模版
* styles/ 样式文件

---
如果你懒得折腾,那么直接*fork*一份别人的配置是最快的

比如我一开始用的是[这个](https://github.com/barryclark/jekyll-now) 

*fork*的时候记得注意一下版权,把原来的文章和图片删掉
  
## 使用markdown写作

之所以使用markdown而不是 org-mode 来写作,因为我搞不定它的配置

~~其实是是我太菜了~~

用org-mode如果不折腾一番,导出的html文件样式就很难看

markdown语法简单易学,通用性也很强,很适合用来写文章


