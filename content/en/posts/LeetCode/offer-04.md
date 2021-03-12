---
title: "[剑指offer]04-二维数组中查找"
date: 2021-02-17 10:41:13
description: 剑指offer
draft: false

tags:
- "Medium"
series:
- 
categories:
- LeetCode
image: images/feature1/graph.png
---





### 题目:



[二维数组中查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)



> 在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个高效的函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。
>



**示例 1：**

现有矩阵 matrix 如下：

> [
>   [1,   4,  7, 11, 15],
>   [2,   5,  8, 12, 19],
>   [3,   6,  9, 16, 22],
>   [10, 13, 14, 17, 24],
>   [18, 21, 23, 26, 30]
> ]



### 题解

我们可以线性查找, 从矩阵左下角开始, 若target比当前值大,则向右, 否则向上

#### 解法一:

```go
package main

func findNumberIn2DArray(matrix [][]int, target int) bool {
	// 数组边界
	row := len(matrix)-1	
	if row == -1{
		return false
	}
	col := len(matrix[0])-1


	var tmp int

	for i, j := row, 0 ; i>=0 && j <= col ;{
		tmp = matrix[i][j]
		if tmp == target {
			return true
		} else {
			if  tmp < target {
				j++
			} else {
				i--
			}
		}
	}
	
	return false
}
```