---
title: 【LeetCode每日一题】162.寻找峰值.md
date: 2021-01-07 09:58:24
tags: [LeetCode,算法,二分查找]
categories:
- 技术博客
- LeetCode每日一题
---

[162.寻找峰值](https://leetcode-cn.com/problems/find-peak-element/description/)

<!-- more -->

>  峰值元素是指其值大于左右相邻值的元素。
> 
> 给定一个输入数组 nums，其中 nums[i] ≠ nums[i+1]，找到峰值元素并返回其索引。
>
> 数组可能包含多个峰值，在这种情况下，返回任何一个峰值所在位置即可。
>
> 你可以假设 nums[-1] = nums[n] = -∞。
>
> 示例 1:
> 输入: nums = [1,2,3,1]
> 输出: 2
> 解释: 3 是峰值元素，你的函数应该返回其索引 2。
>
> 示例 2:
> 输入: nums = [1,2,1,3,5,6,4]
> 输出: 1 或 5
> 解释: 你的函数可以返回索引 1，其峰值元素为 2；
> 或者返回索引 5， 其峰值元素为 6。
>
> 说明:
> 你的解法应该是 O(logN) 时间复杂度的。

如果一个题目，可以简单到用复杂度O(n)的方式解决，但题目要求复杂度为O(logN)复杂度，这种情况下基本上就直接考虑使用二分查找。

但原始二分查找要求数组是有序的，很显然，这个题目中数组并无序。所以我们要转化一下思路，二分就是折半嘛，如何才能折半查找呢？

**对于任何一个数组中的元素，如果它大于右边的元素，那么这个元素肯定处于一个递减区间里，所以对于这个递减区间的峰值，肯定在这个元素的左侧（包括它自己）。**

这个性质很隐蔽，但也是解决这个问题的最重要的性质，如果能想明白这个，问题就迎刃而解了。

我们采用折半查找的思想，取中间元素，如果中间元素大于右边元素，那么再比较和左边元素，如果也大于左边，那么它就是峰值元素。如果不大于左边元素，那么说明峰值在它左侧，我们再对它左侧区间进行查找，直到找到峰值即可。

```golang
func findPeakElement(nums []int) int {
    length := len(nums)
    if length == 1 {
        return 0
    }
    start, end := 0, length-1
    mid := 0
    for start <= end {
        mid = (start + end) / 2
        // 这里由于数组特殊性
        // 如果mid为0，那么还要判断mid位置和mid+1位置的大小
        // 只有nums[mid] >= nums[mid+1]时，位置0才是峰值元素索引
        // 比如nums=[1,2]，mid=0，此时峰值元素为2; nums=[1,1],mid=0，此时峰值元素为1,返回索引0或1均可.
        if (mid == 0 && nums[mid] >= nums[mid+1]) || mid == length-1 {
            return mid
        }
        if nums[mid] < nums[mid+1] {
            start = mid + 1
        } else if nums[mid] > nums[mid-1] {
            return mid
        } else {
            end = mid - 1
        }
    }

    return mid
}

```