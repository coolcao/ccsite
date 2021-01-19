---
title: 【LeetCode每日一题】300. 最长递增子序列.md
date: 2021-01-19 13:55:08
tags: [算法,LeetCode,动态规划,二分查找]
categories:
- 技术博客
- LeetCode每日一题
---

[300.最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/description/)

<!-- more -->

>给你一个整数数组 nums ，找到其中最长严格递增子序列的长度。
>子序列是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，[3,6,2,7] 是数组 [0,3,1,6,2,2,7] 的子序列。
>
>示例 1：
>
>输入：nums = [10,9,2,5,3,7,101,18]
>输出：4
>解释：最长递增子序列是 [2,3,7,101]，因此长度为 4 。
>
>示例 2：
>输入：nums = [0,1,0,3,2,3]
>输出：4
>
>示例 3：
>输入：nums = [7,7,7,7,7,7,7]
>输出：1
>
>提示：
>1 <= nums.length <= 2500
>-10^4 <= nums[i] <= 10^4
>
>进阶：
>你可以设计时间复杂度为 O(n2) 的解决方案吗？
>你能将算法的时间复杂度降低到 O(n log(n)) 吗?


## 动态规划

```go
func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
func lengthOfLIS(nums []int) int {
    length := len(nums)
    if length == 0 {
        return 0
    }

    maxLis := 1
    dp := make([]int, length)
    dp[0] = 1
    for i := 1; i < length; i++ {
        dp[i] = 1
        for j := 0; j < i; j++ {
            if nums[i] > nums[j] {
                dp[i] = max(dp[i], dp[j]+1)
            }
        }
        maxLis = max(maxLis, dp[i])
    }

    return maxLis
}
```

## 贪心算法+二分查找

```go
func lengthOfLIS(nums []int) int {
    length := len(nums)
    if length == 1 {
        return 1
    }
    tmp := []int{nums[0]}
    for i := 1; i < length; i++ {
        if nums[i] > tmp[len(tmp)-1] {
            // 如果nums[i]大于tmp的最后一个元素，直接追加到tmp
            tmp = append(tmp, nums[i])
        } else {
            // 否则替换掉tmp中第一个大于nums[i]的元素
            start, end := 0, len(tmp)-1
            for start < end {
                mid := (start + end) / 2
                if tmp[mid] < nums[i] {
                    start++
                } else {
                    end--
                }
            }
            tmp[start] = nums[i]
        }
    }
    return len(tmp)
}
```