---
title: 【LeetCode每日一题】74.搜索二维矩阵.md
date: 2021-01-07 10:49:31
tags: [算法,LeetCode,二分查找]
categories:
- 技术博客
- LeetCode每日一题
---

[74.搜索二维矩阵](https://leetcode-cn.com/problems/search-a-2d-matrix/description/)

> 编写一个高效的算法来判断 m x n 矩阵中，是否存在一个目标值。该矩阵具有如下特性：
>
> 每行中的整数从左到右按升序排列。
> 每行的第一个整数大于前一行的最后一个整数。
>
> 示例 1：
> ![mat](_v_images/20210107110407129_444538278.jpg)
> 输入：matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 3
> 输出：true
>
> 示例 2：
> ![mat2](_v_images/20210107110345182_226670950.jpg)
> 输入：matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 13
> 输出：false
>
> 提示：
> * m == matrix.length
> * n == matrix[i].length
> * 1 <= m, n <= 100
> * -10^4 <= matrix[i][j], target <= 10^4


题目中给的是一个二维矩阵，把这个矩阵拍平后，我们就得到了一个有序的数组。所以，这里可以使用二分查找的方式来进行搜索。


拍平后在数组中的索引idx和原先矩阵中的二维索引row,col的关系，不难发现，row=idx/n， col=idx%n，有了这个转换关系，我们的查找就完全当成一个标准二分查找即可。

```golang
func searchMatrix(matrix [][]int, target int) bool {
    m, n := len(matrix), len(matrix[0])
    length := m * n
    start, end := 0, length-1

    for start <= end {
        mid := (start + end) / 2
        // 转换为矩阵的二维索引
        row, col := mid/n, mid%n
        if matrix[row][col] == target {
            return true
        } else if matrix[row][col] > target {
            end = mid - 1
        } else if matrix[row][col] < target {
            start = mid + 1
        }
    }
    return false
}

```
