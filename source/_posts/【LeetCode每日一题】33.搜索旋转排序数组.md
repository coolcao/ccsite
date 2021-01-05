---
title: 【LeetCode每日一题】33.搜索旋转排序数组.md
date: 2021-01-05 13:38:57
tags: [搜索旋转排序数组]
categories:
- 技术博客
- 搜索旋转排序数组
---

 [LeetCode 33.搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

<!-- more -->

> 升序排列的整数数组 nums 在预先未知的某个点上进行了旋转（例如， [0,1,2,4,5,6,7] 经旋转后可能变为 [4,5,6,7,0,1,2] ）。
> 请你在数组中搜索 target ，如果数组中存在这个目标值，则返回它的索引，否则返回 -1 。
>
> 示例 1：
> 输入：nums = [4,5,6,7,0,1,2], target = 0
> 输出：4
>
> 示例 2：
> 输入：nums = [4,5,6,7,0,1,2], target = 3
> 输出：-1
>
> 示例 3：
> 输入：nums = [1], target = 0
> 输出：-1
>
> 提示：
> 1. 1 <= nums.length <= 5000
> 2. -10^4 <= nums[i] <= 10^4
> 3. nums 中的每个值都 独一无二
> 4. nums 肯定会在某个点上旋转
> 5. -10^4 <= target <= 10^4


这个题目本身题意并不难理解，就是如何在一个数组中查找一个元素。但难度定义为中等难度，所以，如果写一个复杂度为O(n)的顺序查找肯定是不ok的了。我们应该考虑使用一个复杂度比O(n)还低的查找算法，比如，二分查找。

二分查找，是一种在有序数组上查找元素的方法，其要求就是，数组是有序的。

那对于上面这个题目，本来是有序数组，做了旋转后，整个数组就不是有序的了。那怎么使用二分查找呢？

原数组是有序的，从某个元素旋转后，其中肯定有一半是有序的，比如[4,5,6,7,8,0,1,2]，总共7个元素，我们将数组一分为二，[4,5,6,7]和[8,0,1,2]，其中前半部分[4,5,6,7]是有序的，这样我们就可以在这部分有序的数组上进行二分查找。

那后半部分还是无序的，如果要查找的元素在后半部分怎么办呢？仔细看，后半部分虽然是无序的，但和整个数组性质是一样的，也算是一个有序数组的旋转数组，我们再将后半部分一分为二，其中有一半肯定是有序的。

所以，到这里问题就明朗了，我们的大致解题步骤如下：
1. 先找到升序区间
2. 如果元素在升序区间内，直接在升序区间对其进行二分查找
3. 如果元素不在升序区间内，那么我们再对剩下的非升序区间进行步骤1操作。
4. 找到元素或区间缩小至只有一个元素时结束查找。

```golang
func binarySearch(nums []int, start, end int, target int) int {
    if start > end {
        return -1
    }
    mid := (start + end) / 2
    if nums[mid] == target {
        return mid
    }
    if nums[mid] > target {
        return binarySearch(nums, start, mid-1, target)
    }
    if nums[mid] < target {
        return binarySearch(nums, mid+1, end, target)
    }
    return -1
}

func toSearch(nums []int, start, end int, target int) int {
	if start >= end {
		if nums[start] == target {
			return start
		}
		return -1
	}
	mid := (start + end) / 2

	// 升序区间start, end
	ascStart, ascEnd := start, mid
	// 剩余区间start, end
	nStart, nEnd := mid+1, end

	if nums[nStart] <= nums[nEnd] {
		ascStart = nStart
		ascEnd = nEnd
		nStart = start
		nEnd = mid
	}

	// 判断target是否在升序区间内，如果在，直接二分
	if target >= nums[ascStart] && target <= nums[ascEnd] {
		return binarySearch(nums, ascStart, ascEnd, target)
	}
	// 如果不在，则继续在剩余非升序区间查找
	return toSearch(nums, nStart, nEnd, target)

}
func search(nums []int, target int) int {
	length := len(nums)
	if length == 0 {
		return -1
	}
	return toSearch(nums, 0, length-1, target)
}

```
