---
title: 【LeetCode每日一题】11.盛最多水的容器.md
date: 2021-01-04 17:15:29
tags: [算法,LeetCode,二分查找]
categories:
- 技术博客
- LeetCode每日一题
---

 [LeetCode 11.盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/description/)


 > 给你 n 个非负整数 a1，a2，...，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为(i, ai) 和 (i, 0) 。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。
 > 说明：你不能倾斜容器。
 > 示例 1：
 > 输入：[1,8,6,2,5,4,8,3,7]
 > 输出：49
 > 解释：图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。
 > 示例 2：
 > 输入：height = [1,1]
 > 输出：1
 > 示例 3：
 > 输入：height = [4,3,2,1,4]
 > 输出：16
 > 示例 4：
 > 输入：height = [1,2,1]
 > 输出：2
 > 提示：
 > * n = height.length
 > * 2 <= n <= 3 * 10^4
 > * 0 <= height[i] <= 3 * 10^4

<!-- more -->
这个题目要求计算能够盛放的最多的水，这里其实是求的面积。

我们知道面积公式为： 面积=长*高

而在该题目中，长度和高度都是变量，所以，最直观，最简单的想法是，将长度和高度作为两个变量，穷举出所有可能的容器，计算所有容器能够盛的水，取其最大的即可。
这样的话，我们就需要双层循环遍历，时间复杂度为 O(n^2)。这个题目被设计成中等难度，显然这个穷举法不是题目要考察的。

我们该怎么把这由于两个变量导致的两层循环给优化一下呢？

如果我们能定住其中一个变量，或者将另外一个变量的变化引起的作用，不能实质改变问题的求解，让问题的解只和其中一个变量发生作用，就可以减掉一层循环，将复杂度降到O(n)。

由于面积=长度*高度，而在本题中，长度和高度，都不是固定的，所以上面说的定住一个变量，恐怕难以实现。

那我们能不能改变一下思路，让其中一个变量“失去”作用，最终的最大盛水量只和其中一个变量有关呢？

面积=长度*高度。其中高度就是给的一个数组，其中每个元素都是一个高度，而长度就是两个高度之间的距离。

如果我们将长度不断缩小，那么要求最大面积，最大面积就只和高度相关了，因为长度不断缩小，要想面积增大，那么只能是高度不断增大。

我们取长度最大的两个高度作为容器的边界，即height[0]和height[length-1]，然后不断缩小长度。
那缩小长度时，怎么确定是移动左边界还是右边界呢？依据就是上面说的，由于长度缩小，要想面积增大，那只能是高度增加，我们应该移动左右边界中，小的那一个边界，这样才有可能遇到更大的边界，面积才有可能变大。

其操作过程如下：

![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1609755698_20210104181147641_416578513.png)

取height[0]和height[8]为左右边界，这时面积为8.

![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1609755699_20210104181325218_638378771.png)
由于height[0]<height[8]，因此移动左边界，这时面积为7*7=49.
根据此逻辑，不断移动左右边界，计算出相应的面积，选取最大面积即可。
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1609755700_20210104181603597_403703785.png)
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1609755700_20210104181734230_1148673622.png)
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1609755701_20210104181746888_2027062071.png)
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1609755702_20210104181758478_1151651289.png)
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1609755702_20210104181809496_533103812.png)
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1609755703_20210104181819061_573993000.png)

这样我们只需要O(n)复杂度即可完成实现。

```golang
func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}
func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}
func maxArea(height []int) int {
	length := len(height)
	start, end := 0, length-1
	maxArea := 0
	for start < end {
		area := (end - start) * min(height[start], height[end])
		maxArea = max(area, maxArea)
		if height[start] < height[end] {
			start++
		} else {
			end--
		}
	}
	return maxArea
}
```

