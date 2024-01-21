---
title: "滑动窗口"
description: "滑动窗口"
keywords: "滑动窗口"

date: 2024-01-21T11:07:45+08:00
lastmod: 2024-01-21T11:07:45+08:00

categories:
  - 算法
tags:
  - 滑动窗口
  - 数组

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
#url: "sliding_window.html"
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

# 滑动窗口

## 算法思想
滑动窗口是一种思路，不断滑动滑动窗口的左右边界，使得窗口中所包含元素符合预期结果，不断滑动窗口寻找到最合适的结果。一般需要思考三个问题
- 如何移动右边界，扩展元素
- 如果移动左窗口，收缩元素
- 如何验证左边界到右边界元素符合预期

注意定义好left right区间，这样好判断元素操作顺序，数据长度以及终止条件  
`滑动窗口最大特性是连续性，它一般聚焦在数组一块连续区域内`
## 算法代码


## 算法实例

### 209.长度最小的子数组
[209.长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/description/)
给定一个含有 n 个正整数的数组和一个正整数 target 。
找出该数组中满足其总和大于等于 target 的长度最小的 连续子数组 [numsl, numsl+1, ..., numsr-1, numsr] ，并返回其长度。如果不存在符合条件的子数组，返回 0

#### 解题思路
移动左右窗口，使其左右窗口之和大于指定target
- 移动右边界，如果发现sum值小于预期值，继续加入更多元素
- 移动左边界，如果发现sum值大于等于预期值，减少元素加入，看是否能保持当前条件
- 通过sum字段将区间指向的值求和，对于移动右边界则加入数组，移动左边界则减少sum的和求值。

注意如果right已经超过右边界了，总和还小于预期值，那么这条件，left和right都没必要继续移动了，接下来不会有总和满足的条件

#### 代码
```go
func minSubArrayLen(target int, nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    left := 0;
    right := 0;
    maxLen := math.MaxInt
    length := 0
    sum := nums[0]
    for ;left<len(nums)&&right<len(nums); {
        if sum >= target {
            length = right-left +1
            if length < maxLen {
                maxLen =  length
            }
            sum -= nums[left]
            left += 1
        } else {
            right += 1
            if right < len(nums) {
                sum += nums[right]
            } else {
                break; //总和小于目标值 并且已经到右边端点了，此时只能退出循环，找不到其他解了
            }
        }
        
    }
    if maxLen != math.MaxInt {
        return maxLen
    }
    return 0
}
```

### 904. 水果成篮
[904. 水果成篮](https://leetcode.cn/problems/fruit-into-baskets/description/)
你正在探访一家农场，农场从左到右种植了一排果树。这些树用一个整数数组 fruits 表示，其中 fruits[i] 是第 i 棵树上的水果 种类 。
你想要尽可能多地收集水果。然而，农场的主人设定了一些严格的规矩，你必须按照要求采摘水果：
你只有 两个 篮子，并且每个篮子只能装 单一类型 的水果。每个篮子能够装的水果总量没有限制。
你可以选择任意一棵树开始采摘，你必须从 每棵 树（包括开始采摘的树）上 恰好摘一个水果 。采摘的水果应当符合篮子中的水果类型。每采摘一次，你将会向右移动到下一棵树，并继续采摘。
一旦你走到某棵树前，但水果不符合篮子的水果类型，那么就必须停止采摘。
给你一个整数数组 fruits ，返回你可以收集的水果的 最大 数目

#### 解题思路
每个fruits[i]代表水果的种类，这里求解的是连续区间内，水果种类只有两种，并且长度最长的子数组。
滑动窗口的特征很明显，连续性数组.

- 移动右边界，如果right指针指向元素已经存在，那么可以继续扩展右边界或者元素不存在，但是整体元素类型不超过2
- 移动左边界，元素类型超过2种类型，减去一些元素，如果元素的数量等于0，则删除整个元素，为新种类提供空间
- 将区间设置为左闭合右开区间，[left,right) right-left即是最大连续子区间


#### 代码
```go
func totalFruit(fruits []int) int {
	if len(fruits) == 0 {
		return 0
	}
	left := 0
	right := 1  //left right区间定义变成左闭右开区间 写出来就顺畅很多
	fruitClassMap := make(map[int]int)
	fruitClassMap[fruits[0]] = 1
	maxClassCount := 1
	for right < len(fruits) {
		if _, ok := fruitClassMap[fruits[right]]; ok {
			fruitClassMap[fruits[right]] = fruitClassMap[fruits[right]] + 1
			right += 1
			if maxClassCount <= right-left {
				maxClassCount = right - left
			}
		} else {
			if len(fruitClassMap) < 2 {
				fruitClassMap[fruits[right]] = 1
				right += 1
				if maxClassCount <= right-left {
					maxClassCount = right - left
				}
			} else {
				fruitClassMap[fruits[left]] = fruitClassMap[fruits[left]] - 1
				if fruitClassMap[fruits[left]] == 0 {
					delete(fruitClassMap, fruits[left])
				}
				left += 1
			}
		}
	}
	return maxClassCount
}
```

### 76.最小覆盖子串
[76.最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/description/)
给你一个字符串 s 、一个字符串 t 。返回 s 中涵盖 t 所有字符的最小子串。如果 s 中不存在涵盖 t 所有字符的子串，则返回空字符串 "" 。
注意：
对于 t 中重复字符，我们寻找的子字符串中该字符数量必须不少于 t 中该字符数量。
如果 s 中存在这样的子串，我们保证它是唯一的答案。

#### 解题思路
该题目也有很明显的连续子数组特征，要求找到连续子数组中最优方案。
移动右区间: 如果区间内元素没有包含所有t,那么移动右区间
移动左区间: 移除左边区间，判断剩余元素是否还是满足区间，如果满足则继续移动左区间，如果不满足，那么移动右区间
判断区间内元素是否符合预期: 给t建立map统计每个字符数据量，子区间要满足需要map字符统计数目大于等于指定元素
#### 代码
```go
// 检查数组是否ok
func checkOk(mappingCost map[byte]int, mappingOrigin map[byte]int) bool {
	for b, expectedCount := range mappingOrigin {
		if rc, ok := mappingCost[b]; ok {
			if rc < expectedCount {
				return false
			}
		} else {
			return false
		}
	}
	return true
}

func minWindow(s string, t string) string {
	if len(s) == 0 {
		return ""
	}
	mappingCost := make(map[byte]int)
	mappingOrigin := make(map[byte]int)
	tbytes := []byte(t)
	for _, val := range tbytes {
		if v, ok := mappingOrigin[val]; ok {
			mappingOrigin[val] = v + 1
		} else {
			mappingOrigin[val] = 1
		}
	}
	sbytes := []byte(s)
	left := 0
	right := 0
	maxLength := math.MaxInt
	maxString := ""
	l := 0
	for right < len(sbytes) {
		//fmt.Println(left, right)
		v, ok := mappingCost[sbytes[right]]
		if ok {
			mappingCost[sbytes[right]] = v + 1
		} else {
			mappingCost[sbytes[right]] = 1
		}
		if checkOk(mappingCost, mappingOrigin) {
			//fmt.Println("check ok1", left, right)
			l = right - left + 1
			if l < maxLength {
				maxLength = l
				maxString = string(sbytes[left : right+1])
			}
			for left <= right {
				//fmt.Println("delete ", sbytes[left])
				mappingCost[sbytes[left]] -= 1
				left += 1
				if checkOk(mappingCost, mappingOrigin) {
					//fmt.Println("check ok2 ", left, right, mappingCost, mappingOrigin)
					l = right - left + 1
					if l < maxLength {
						maxLength = l
						maxString = string(sbytes[left : right+1])
					}
				} else {
					break
				}
			}
		}
		right += 1
	}
	return maxString
}
```

##  总结
滑动窗口通过移动左右区间使其左右区间内元素满足指定条件，该条件一般是连续子区间满足指定条件，如果子数组包含所有预期字符，子数组包含所有数之和大于预期值，
子数组包长度最大等条件。滑动窗口最明显特征就是一般求解子连续区间内元素是否满足某一条件，想好如何移动左右区间，如何维护好子区间状态以及判断状态是否为所需要状态。
<!--more-->
