---
title: "双指针"
description: "双指针"
keywords: "算法，双指针，左右指针，快慢指针"

date: 2024-01-07T23:35:39+08:00
lastmod: 2024-01-07T23:35:39+08:00

categories:
  - 算法
tags:
  - 数组
  - 快慢指针,左右指针,回文串

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
#url: "array.html"
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

双指针就是通过一个快指针和慢指针在一个for循环下完成两个for循环的工作
- 快指针：寻找新数组的元素 ，新数组就是不含有目标元素的数组
- 慢指针：指向更新 新数组下标的位置

  双指针法（快慢指针法）在数组和链表的操作中是非常常见的，很多考察数组、链表、字符串等操作的面试题，都使用双指针法。
  https://programmercarl.com/0027.%E7%A7%BB%E9%99%A4%E5%85%83%E7%B4%A0.html#%E6%80%9D%E8%B7%AF
- https://www.hello-algo.com/chapter_array_and_linkedlist/array/#412
- https://labuladong.github.io/algo/di-ling-zh-bfe1b/shuang-zhi-fa4bd/#%E4%BA%8C%E3%80%81%E5%B7%A6%E5%8F%B3%E6%8C%87%E9%92%88%E7%9A%84%E5%B8%B8%E7%94%A8%E7%AE%97%E6%B3%95

双指针技巧是经常用到的，双指针技巧主要分为两类：左右指针和快慢指针。
所谓左右指针，就是两个指针相向而行或者相背而行；而所谓快慢指针，就是两个指针同向而行，一快一慢。
对于单链表来说，大部分技巧都属于快慢指针
数组问题中比较常见的快慢指针技巧，是让你原地修改数组。

## 算法实现细节
1. 快慢指针开始均指向0
2. 当快慢指针相同时候，快指针往前移动
3. 当快慢指针不同时候，快慢指针均往前移动一步 慢指针要先加1 才是新元素位置
```go
func removeDuplicates(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    slow,fast :=0,0 //初始均指向0
    for fast<len(nums) {
        if nums[fast] != nums[slow] {
            slow++
             // 维护 nums[0..slow] 无重复
            nums[slow] = nums[fast]
        }
        fast++
    }
    // 数组长度为索引 + 1
    return slow + 1
}
```

## 算法应用场景

###  左右指针
[两数之和 II - 输入有序数组](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/description/)  
`一个有序的非递减顺序数组，找到两数之和等于目标值的元素`
解题思路: 使用左右指针，左指针位于0 右指针位于n-1, 计算两数之和，如果大于目标值，那么移动右指针，如果小于目标值，移动左指针。
左指针朝着数据增大方向，右指针朝着数据减少方向。注意，如果题目是递减顺序则相反。如果两数之和等于目标值则返回该元素。
依据元素是否重复以及求解的唯一性，判断是否终止或者继续。
```go
func twoSum(numbers []int, target int) []int {
    left := 0
    right := len(numbers)-1
    for ;left<right; {
        sum := numbers[left] + numbers[right]
        if sum > target {
            right = right -1;
        } else if sum < target {
            left = left + 1;
        } else {
            return []int{left+1,right+1}
        }
    }
    return []int{}
}
```

### 快慢指针
快慢指针在数组，链表中使用得比较频繁
[26. 删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/description/)  
快慢指针区别于左右指针在于，快慢指针是同向运动，快指针寻找目标值，慢指针停留在原地。
初始条件下，快慢指针指向初始位置，当条件满足时候快慢指针一起移动，同时做一些元素移动操作，如果不满足，则只有快指针移动，慢指针不移动
```go
func removeDuplicates(nums []int) int {
	if len(nums) == 0 {
	    return 0	
    }
    slow := 0
    fast := 0
    for ;fast<len(nums); {
        if nums[slow] != nums[fast] {
            slow+=1
            nums[slow] = nums[fast]
        }
        fast+=1
    }
    return slow+1;
}
```
删除slow指向的是上一个元素，因此要先加上1，然后再复制,fast指向是快指针

[27. 移除元素](https://leetcode.cn/problems/remove-element/description/)
给你一个数组 nums 和一个值 val，你需要 原地 移除所有数值等于 val 的元素，并返回移除后数组的新长度.
这道题目还是双指针解法，slow指向没有val值的终点，fast快速寻找值。
```go
func removeElement(nums []int, val int) int {
  slow := 0
  fast := 0
  for fast < len(nums) {
      if nums[fast] != val {
        nums[slow] = nums[fast]
        slow += 1
      }
      fast += 1
  }
  return slow
}
```
输入数组注意判空
这次慢指针指向不重复元素末尾，定义好元素，首尾一致就行。
[283. 移动零](https://leetcode.cn/problems/move-zeroes/description/)
给定一个数组 nums，编写一个函数将所有 0 移动到数组的末尾，同时保持非零元素的相对顺序。
这到题目和移除val相似，多一步把后面元素设置为0
```go
    func moveZeroes(nums []int)  {
      slowIndex := 0
      for i:=0;i<len(nums);i++ {
        if nums[i] != 0 {
              nums[slowIndex] = nums[i]
              slowIndex += 1
        }
      }
      for i:=slowIndex;i<len(nums);i++{
            nums[i] = 0
      }
}
```
[344. 反转字符串](https://leetcode.cn/problems/reverse-string/description/)
编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 s 的形式给出。
```go
func reverseString(s []byte)  {
    left := 0
    right := len(s) -1
    var tmp byte
    for ;left<right; {
        tmp = s[left]
        s[left] = s[right]
        s[right] = tmp
        left +=1
        right -=1 
    }
}
```

[5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/description/)
给你一个字符串 s，找到 s 中最长的回文子串。如果字符串的反序与原始字符串相同，则该字符串称为回文字符串
这道题目难点在于回文串存在基数或者偶数的回文串。对于这种情况一种巧妙方法是使用使用区间来表示需要开始的区间，然后从中间往左右两边扩散。
回文串比较特殊的双指针，从中间往左右两边扩散。目前仅仅在回文串题目中会用到。
```go
// 以l,r为中心，看是否为回文子串
func palindrome(s string,l,r int) string{
    for ;l>=0&&r<len(s)&&s[l]==s[r]; {
        l--;
        r++;
    }
    return s[l+1:r]
}
```
以区间l和r为起始点，从中间往两边扩展，当且仅当s[l]=s[r]时候;循环结束后 l和r需要各自回1 l+1 r-1 但是因为slice是左闭右开区间。
所以是s[l+1:r]
接着主逻辑层判断,横向遍历每个元素，以该元素为中点往两边扩展，以该元素以及元素+1往两边扩展，返回的回文子串判读是否是最长的会问子串
```go
func longestPalindrome(s string) string {
    var result string
    for i:=0;i<len(s);i++{
            s1 := palindrome(s,i,i)
            s2 := palindrome(s,i,i+1)
            if len(s1) > len(result) {
                result = s1
            }
            if len(s2) > len(result) {
                result = s2
            }
    }
    return result
}
```

[83. 删除排序链表中的重复元素](https://leetcode.cn/problems/remove-duplicates-from-sorted-list/description/)
给定一个已排序的链表的头 head ， 删除所有重复的元素，使每个元素只出现一次 。返回 已排序的链表  
本质也是快慢指针的应用，不过这次换位链表，链表一般head指针是带有元素值的，元素指针为空nil,注意指针数组类型题目注意输入条件检查。
```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func deleteDuplicates(head *ListNode) *ListNode {
    if head == nil { //指针注意做输入防护
        return nil
    }
    fast := head
    slow := head
    for ;fast != nil ; {
        if fast.Val != slow.Val {
            slow.Next = fast
            slow = fast
        }
        fast = fast.Next
    }
    slow.Next = nil
    return head
}
```

[977.有序数组的平方](https://leetcode.cn/problems/squares-of-a-sorted-array/description/)
给你一个按 非递减顺序 排序的整数数组 nums，返回 每个数字的平方 组成的新数组，要求也按 非递减顺序 排序。
一种是非常朴素做法，找到-1和0的分界点，可以用二分查找发查找0，如果不存在则返回第一个小于0的元素，可以返回右区间。然后以此为左右指针出发点
从中间往两边走，这个和回文字符串有点相似，然后判断数值平方谁大，移动谁的乘积到新数组，就数组无法继续使用。
另外一种方法是一种巧思，因为数组有序，绝对值最大的肯定是在左右两边，可以省去寻找0的过程，然后指针从左右元素开始移动

解法1：
```go
func findInsertPos(nums []int, target int) int {
	left := 0
	right := len(nums) - 1
	middle := 0
	for left <= right {
		middle = (right-left)>>1 + left
		if nums[middle] < target {
			left = middle + 1
		} else if nums[middle] >= target {
			right = middle - 1
		}
	}
	return right
}

func sortedSquares(nums []int) []int {
	negativePos := findInsertPos(nums, 0) //找到0左边第一个负数位置
	p1 := negativePos
	p2 := negativePos + 1
	result := make([]int, 0)
	for {
		if p1 >= 0 && p2 < len(nums) {
			if nums[p1]*nums[p1] < nums[p2]*nums[p2] {
				result = append(result, nums[p1]*nums[p1])
				p1 -= 1
			} else {
				result = append(result, nums[p2]*nums[p2])
				p2 += 1
			}
		} else if p1 >= 0 {
			result = append(result, nums[p1]*nums[p1])
			p1 -= 1
		} else if p2 < len(nums) {
			result = append(result, nums[p2]*nums[p2])
			p2 += 1
		} else {
			break
		}
	}
	return result
}
```


解法2：
```go
func sortedSquares(nums []int) []int {
	p1 := 0
	p2 := len(nums)-1
	result := make([]int, len(nums))
	index := len(nums)-1
	for ;p1<=p2;{
            if nums[p1]*nums[p1] < nums[p2]*nums[p2] {
                result[index] = nums[p2]*nums[p2]
				p2 -= 1 
			} else {
                result[index] = nums[p1]*nums[p1]
				p1 += 1
			}
            index -= 1
    }
	return result
}
```

## 总结
  双指针经常用于数组或者链表题目中，而链表又分为快慢指针和左右指针，左右指针又有相向而行和相背而行，向背而行目前比较多的是在判断回文字串题目。
回文字串类题目难点在于奇数和偶数长度不好判断，一个巧妙方式是采用函数区间[l,r]形式。
<!--more-->
