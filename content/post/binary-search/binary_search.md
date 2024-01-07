---
title: "二分查找算法"
description: "二分查找"
keywords: "二分查找,二分法，Binary Search"

date: 2024-01-06T10:08:37+08:00
lastmod: 2024-01-06T10:08:37+08:00

categories:
  - 算法
tags:
  - 二分查找

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
#url: "binary_search.html"
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

# 二分查找算法总结

## 算法描述
二分查找算法是一种基于分治策略的高效搜索算法，利用数据的有序性，每轮缩减一半查找范围，直到找到目标元素或者区间为空位置。

## 算法实现细节

算法的判断条件和区间有关，一般有全闭区间，左闭右开区间，左开右闭区间，不同区间会导致循环判断条件，以及左右区间left right更新数值不一样。
注意区间更新大于 小于时候更新的是left还是right,不要搞混
### 左闭右闭区间
```go
func search(nums []int, target int) int {
    high := len(nums)-1
    low := 0
    for low <= high { // notice here
        mid := low + (high-low)/2
        if nums[mid] == target {
            return mid
        } else if nums[mid] > target {
            high = mid-1 // notice here
        } else {
            low = mid+1
        }
    }
    return -1
}
```
### 左闭右开区间
```go
func search(nums []int, target int) int {
    high := len(nums)
    low := 0
    for low < high {
        mid := low + (high-low)/2
        if nums[mid] == target {
            return mid
        } else if nums[mid] > target {
            high = mid
        } else {
            low = mid+1
        }
    }
    return -1
}
```

两种写法区别在于
- left right 这个区间是否为有效区间，是否要继续查找
- 左右边界更新时候形成区间是否是没查找过区间，如果确定没有该元素区间则不进行查找

### 优点和局限性
`优点`
- 二分查找算法效率比较高，log(n)时间复杂度 比如数据量为2^10 线性算法需要寻找1000次，二分查找只需寻找10次
- 常数复杂度空间

`缺点`
- 二分查找算法仅适用于有序数据，部分场景还需要不存在重复元素，如果数据本身无序，为了使用二分查找算法，进行排序再使用二分查找算法通常是复杂度比较高
  比如通常排序时间复杂度在nlog(n)，在使用二分查找，总体时间复杂度还是nlog(n),直接查找还是o(n),本末倒置了。
- 二分查找仅仅适用于连续数组空间
- 数据量少时候，直接查找反而更快，单次搜索需要 一次加法 + 一次除法 + 最多三次比较 + 一次加法，需要4 到6次操作。如果数据量少可能计算次数还会超过
  直接数据查找，直接数据查找只需要一次比较

## 算法应用场景

### 查找指定元素是否存在
这类场景直接使用上述算法即可，不需要做修改
### 二分查找插入点
[35.搜索插入位置](https://leetcode.cn/problems/search-insert-position/description/)

可能存在该元素，那么直接返回pos,如果不存在该元素，则pos是第一个大于target的元素，因此pos也是其插入位置
nums[pos−1]<target≤nums[pos]

`最后抽象出，第一个找到第一个大于等于该元素位置` 
```go
func searchInsert(nums []int, target int) int {
    left := 0
    right := len(nums) -1
    ans := len(nums)
    for ;left <= right; {
        middle := left + (right-left)/2
        if nums[middle] > target {
            right = middle -1  //找到第一个大于target的下标，也就是最后一个大于的下标
            ans = middle
        } else if nums[middle] < target {
            left = middle + 1
        } else {
            return middle
        }
    }
    return ans
}
```
`note`: 注意ans初始化为数组长度，存在一种情况，所有元素都小于该元素，那么ans不会进入到大于分支，因此最后位置为元素长度即为插入位置

### 查找区间
[34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/description/)
这道题目寻找的双闭合区间，如果是开区间，会有所调整  
查找区间本质上还是在寻找插入点,左区间点和二分查找插入位置点是一样的  
`查找最左一个target的索引`  
左区间: 找到第一个大于等于该元素值  
右区间: 右区间如何寻找呢? 这道题目规定的是整数，我们可以查找target+1 如果有，那么返回的是最左边位置，如果不存在返回的是该位置插入点，该位置减去1  
就是右区间闭合区间终点 

`当数组不包含 target 时，最终left和right会分别指向首个大于、小于 target 的元素。`

查找target最左插入点
i := searchInsert(nums,tartget)
此时i会返回两种情况，第一种情况，越界，返回数组长度，第二种不存在该元素，返回指向第一个大于该元素的位置
if i == len(nums) || nums[i] != target {
   return -1
}
二分查找返回最左一个区间
```go
//查找插入位置
func findInsertPosition(nums []int, target int) int {
	left := 0
	right := len(nums) - 1
	middle := 0
	for left <= right {
		middle = left + (right-left)>>2
		if nums[middle] >= target {
			right = middle - 1
		} else if nums[middle] < target {
			left = middle + 1
		}
	}
	return left
}
```
二分查找返回最左区间
```go
//查找最左边界
func searchLeft(nums []int, target int) int {
    insPos := findInsertPosition(nums, target)
    if insPos == len(nums) || nums[insPos] != target {
        return -1
    }
    return insPos
}
```
二分查找返回最右区间
```go
/* 二分查找最右一个 target */
func searchRight(nums []int, target int) int {
    insPos := findInsertPosition(nums, target+1)
    //如果所有数据都比target大，那么insPos返回0 
    //如果所有数据都比它大，那么返回len(nums) 
    //如果不存在该元素，返回第一个大于该元素的位置
    insPos = insPos -1
    if insPos==-1 || nums[insPos] != target {
         return -1
    }
    return insPos
}
```
```go
func searchRange(nums []int, target int) []int {
	// 找到第一个等于该值位置 找到
	begin := searchLeft(nums, target)
	end := searchRight(nums, target)
	return []int{begin, end}
}
```

查找最左区间，其实就是在寻找插入位置点，利用left结束后指向第一个大于target元素或者等于target元素，返回left,为什么等于target时候指向的是最左边的
实际上再元素相等时候更新的是right区间，整体搜索区间在不断往左边缩减，最后达到效果就是指向最左边一个元素

搜索完成后的插入位置，存在两种情况是不存在该元素的
- 元素最小 返回0 
- 元素最大 返回n
- 元素位置所在元素不等于target,未找到该元素，此时指向的是第一个大于该元素的位置
代码中元素最小没有写出来，因为可以取到元素，该元素取出来不会等于target,所以最后变成 
`要么元素长度为n，此时取不到元素，要么可以取到该元素，元素不相等`

### 求解平方根
[69. x的平方根](https://leetcode.cn/problems/sqrtx/description/)  
求解x平方根，可以转化成1-x区间求解 n*n  乘积最接近x，本质上再二分搜索过程中找到第一个小于x的数据，返回right
```go
func mySqrt(x int) int {
	left := 0
	right := x
	middle := 0
	for left <= right {
		middle = (right-left)>>1 + left
		total := middle * middle
		if x == total {
			return middle
		} else if x < total {
			right = middle - 1 //注意大于小于时候left赋值 还是 right赋值
		} else if x > total {
			left = middle + 1
		}
	}
	return right
}
```

[367. 有效的完全平方数](https://leetcode.cn/problems/valid-perfect-square/description/)
如果找到数乘积为n，那么返回true，如果没有找到对应的解，那么返回false
```go
func isPerfectSquare(num int) bool {
    left := 0 
    right := num
    for ;left <= right; {
        middle := (right - left ) >> 1  + left
        acc := middle * middle
        if acc > num {
            right = middle - 1
        } else if acc < num  {
            left = middle + 1
        } else {
            return true
        }
    }
    return false
}
```
## 总结
二分查找法的用途
- 查找元素是否存在
- 查找元素插入点位置
- 查找元素的区间，本质上还是查找插入点位置
- 求解x的平方根
<!--more-->
