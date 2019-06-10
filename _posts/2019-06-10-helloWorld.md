---
layout: post
title: 欢迎来到我的新博客
---

## 入坑jekyll
  以前把博客部属在用vps搭建的网站上,然而,那个编辑界面很不友好.  
  于是我投向了github Page + jekyll 的怀抱.  
  jekyll 是什么呢?  
  是用ruby写的静态博客生成器,使用它可以快速搭建自己的播客.  
  而且不用花域名和服务器的冤枉钱,也节省了网站的的维护时间和精力.  
  岂不美哉?  
## 快速上手
  jekyll的官网提供了非常详细的[官方手册](https://jekyllcn.com/docs/home/)  
  你可以阅读这份手册搭建出属于你自己风格的网站.  
  稍微介绍一下目录结构吧.  
*  _posts  是存放你的文章的目录.
*  _drafts是存放草稿的目录.
*  _site   是最后生成的站点的目录
*  _config.yml 是配置文件
*  _layouts 是布局的目录  
如果你懒得折腾,那么直接fork一份别人的配置是最快的.  
比如我用的是[这个](https://github.com/barryclark/jekyll-now)  
fork的时候注意一下版权.  
记得把_posts目录下的文章删掉.  
  
## 使用markdown写作
之所以使用markdown而不是 org-mode 来写作,因为我搞不定它的配置.~~其实是是我太菜了~~  
用org如果不折腾一番,导出的html文件样式就很难看.  
markdown语法简单易学,通用性也很强.  
而jekyll默认支持md格式的文件,感觉用起来就相当方便.  

