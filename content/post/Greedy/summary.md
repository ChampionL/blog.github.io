---
title: "贪心算法总结"
description: "贪心算法"
keywords: "贪心算法"

date: 2023-12-21T22:58:43+08:00
lastmod: 2023-12-21T22:58:43+08:00

categories:
  - 算法
tags:
  - 贪心

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
expand: false
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
#url: "summary.html"
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

# 贪心算法笔记

## 贪心算法定义

贪心算法是求解**最优解**问题常用的算法，最优解问题有许多解法，今天我们学习第一种最优解贪心算法。
一般将求解过程分成若干个步骤，但每个步骤都应用贪心原则，选取当前状态下最优的选择（局部最有利的选择）， 
并以此希望堆叠局部最优解赖达到全局最优解。
举一个最简单的例子：小明和小王喜欢吃苹果，小明可以吃五个，小王可以吃三个。已知苹
果园里有吃不完的苹果，求小明和小王一共最多吃多少个苹果。在这个例子中，我们可以选用的
贪心策略为，每个人吃自己能吃的最多数量的苹果，这在每个人身上都是局部最优的。又因为全
局结果是局部结果的简单求和，且**局部结果互不相干**，因此局部最优的策略也同样是全局最优的
策略

***只有在局部最优解互不相干情况下，贪心算法所得的解才是全局最优解***


## 贪心算法步骤

***贪心算法一般将求解过程分为几个步骤***
1. 建立模型描述问题
2. 将问题拆解成若干个子问题
3. 对每个问题求解，得到局部最优解
4. 把几个子问题的局部最优解合成为全局最优解
----
- 充分条件：有A就一定有B，则A是B的充分条件；
- 必要条件：无A就一定无B，则A是B的必要条件；
- 充要条件：有A就一定有B，无A就一定无B，则A是B的充要条件
----
- 线性时间  O(n)
- 多项式时间 n nlogn n^10
- 指数时间  2^n

贪心算法可以认为是动态规划算法的一个特例，相比动态规划，使用贪心算法需要满足更多的条件（贪心选择性质），但是效率比动态规划要高

无论是贪心还是动态规划都必须有最优子结构

如果问题的最优解包含两个（或更多）子问题的最优解，且子问题多有**重叠**，我们考虑使用动态规划算法。
而如果问题经过贪心选择后，只剩下一个子问题，且具有优化子结构，那么可以使用贪心算法。

如何一样看出重叠子问题  
首先，最简单粗暴的方式就是画图，把递归树画出来，看看有没有重复的节点，重复节点就是我们的状态方程

sort 升序还是降序

贪心和动态规划的区别

1.贪心：每一步的最优解一定包含上一步的最优解，上一步之前的最优解则不作保留；

动态规划：全局最优解中一定包含某个局部最优解，但不一定包含前一个局部最优解，因此需要记录之前的所有的局部最优解

2.贪心：如果把所有的子问题看成一棵树的话，贪心从根出发，每次向下遍历最优子树即可（通常这个“最优”都是基于当前情况下显而易见的“最优”）；这样的话，就不需要知道一个节点的所有子树情况，于是构不成一棵完整的树；

动态规划：动态规划则自底向上，从叶子向根，构造子问题的解，对每一个子树的根，求出下面每一个叶子的值，最后得到一棵完整的树，并且最终选择其中的最优值作为自身的值，得到答案

3.根据以上两条可以知道，贪心不能保证求得的最后解是最佳的，一般复杂度低；

而动态规划本质是穷举法，可以保证结果是最佳的，复杂度高。


/*
    两种动态规划解题思路总结
*/
https://doc.vercel.app/ds/article/75702.html#%E4%B8%A4%E7%A7%8D%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E8%A7%A3%E9%A2%98%E6%80%9D%E8%B7%AF%E6%80%BB%E7%BB%93


注意，SearchInts() 的使用条件为：切片 a 已经升序排序 以下是一个错误使用的例子：

s := []int{5, 2, 6, 3, 1, 4} // 未排序的切片数据
fmt.Println(sort.SearchInts(s, 2)) // 将会输出 0 而不是

<!--more-->
