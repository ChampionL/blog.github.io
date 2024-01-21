---
title: "螺旋矩阵"
description: "螺旋矩阵"
keywords: "螺旋矩阵"

date: 2024-01-21T19:30:52+08:00
lastmod: 2024-01-21T19:30:52+08:00

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
#url: "spiral-matrix.html"
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
#螺旋矩阵

[59.螺旋矩阵II](https://leetcode.cn/problems/spiral-matrix-ii/)
给你一个正整数 n ，生成一个包含 1 到 n2 所有元素，且元素按顺时针顺序螺旋排列的 n x n 正方形矩阵 matrix 。
该模型其实画出两个螺旋矩阵，奇数和偶数
通过step步长，从0开始到step/2 闭区间
注意如果是奇数情况，从左到右和从右到左会相互覆盖，因此在奇数情况并且是最后一轮情况下，需要禁止覆盖。
同时每个循环，最后从下往上和从上往下都是中间的短，从左往右或者从右往左都是迭代湾，注意交叉点的覆盖情况。结合实例给出每次循环填数的起始和终止位置条件
<!--more-->
```go
func generateMatrix(n int) [][]int {
	result := make([][]int, n)
	for i := 0; i < n; i++ {
		result[i] = make([]int, n)
	}
	step := 0
	number := 1
	for step <= n/2 { //TODO 修改条件
		for i1 := 0 + step; i1 < n-step; i1++ {
			result[step][i1] = number
			number += 1
		}
		for i2 := step + 1; i2 <= n-step-1-1; i2++ {
			result[i2][n-step-1] = number
			number += 1
		}
		if step == n/2 && n%2 == 1 {
			//none 如果奇数，且是最后一轮，从左往右和从右往左回重叠
		} else {
			for i3 := n - step - 1; i3 >= step; i3-- {
				result[n-step-1][i3] = number
				number += 1
			}
		}
		for i4 := n - step - 1 - 1; i4 >= step+1; i4-- {
			result[i4][step] = number
			number += 1
		}
		step += 1
	}
	return result
}

```

