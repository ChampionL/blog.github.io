---
title: "Nike"
description: "nike"
keywords: "nike"

date: 2024-03-05T23:46:14+08:00
lastmod: 2024-03-05T23:46:14+08:00

categories:
  -
tags:
  -
  -

# 原文作者
# Post's origin author name
#author:
# 原文链接
# Post's origin link URL
#link:
# 图片链接，用在open graph和twitter卡片上
# Image source link that will use in open graph and twitter card
#imgs:
# 在首页展开内容
# Expand content on the home page
#expand: true
# 外部链接地址，访问时直接跳转
# It's means that will redirecting to external links
#extlink:
# 在当前页面关闭评论功能
# Disabled comment plugins in this post
#comment:
#  enable: false
# 关闭文章目录功能
# Disable table of content
#toc: false
# 绝对访问路径
# Absolute link for visit
#url: "nike.html"
# 开启文章置顶，数字越小越靠前
# Sticky post set-top in home page and the smaller nubmer will more forward.
#weight: 1
# 开启数学公式渲染，可选值： mathjax, katex
# Support Math Formulas render, options: mathjax, katex
#math: mathjax
# 开启各种图渲染，如流程图、时序图、类图等
# Enable chart render, such as: flow, sequence, classes etc
#mermaid: true
---

## nike Hero 页面布局
其采用两个div布局来完成，如果页面超过xl(最小1024),那么采用flex的行布局 flex-row 如果小于这个尺寸则采用列布局，上下分布着两个块，flex-column
- 如果要使用z-index 值越大，越靠近屏幕前的你，所以你要想将一张图片展示在你前面，你需要将其z-index设置为很大， 
- bottom-[5%] 父元素bottom减去子元素bottom的值，正值表示 子元素 在父元素边界之上  负值表示子元素边界在边界之下。
  bottom-[5%] 计算出来就是负值，子元素边界距离父元素边界差距5% 子元素在下面。 
- 
flex里面子元素 会平均瓜分掉宽度 
<!--more-->
