---
title: "[剑指offer]03-数组中重复数字"
date: 2021-02-09 11:20:09
description: 剑指offer
draft: false

tags:
- "Easy"
series:
- 
categories:
- LeetCode
image: images/feature1/graph.png
---



### 题目:



[找出数组中重复的数字](https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)

在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，
但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。



> 示例 1：
>
> 输入：
> [2, 3, 1, 0, 2, 5, 3]
> 输出：2 或 3 



### 题解



#### 解法一:



* 使用一个Set, 出现过则记录该键值true, 遍历一遍得出结果

* go中使用Map[key]struct{}作为set



```go
func findRepeatNumber(nums []int) int {
	// 使用Map实现set,
	set := make(map[int]bool)

	for _,val := range nums {
		// set中存在, 则重复, 返回
		if set[val] == true{
			return val
		} else {
			set[val] = true
		}
	}
	return -1
}
```



#### 解法二:



由于数字范围在 0~n-1, 我们可以原地置换, 使得数字n落在数组的下标n上. 如果遍历到数字num时, 数组arr[num]的值已经为num, 则num为重复数字.



* 注意Go中while的写法



```go
func findRepeatNumber2(nums []int) int {
	// 遍历一遍, 置换下标
	for index := range nums {

		for nums[index] != index {

			if nums[index] == nums[nums[index]] {
				return nums[index]
			}

			// 交换
			nums[nums[index]], nums[index] = nums[index], nums[nums[index]]
		}
	}
	return -1
}
```

