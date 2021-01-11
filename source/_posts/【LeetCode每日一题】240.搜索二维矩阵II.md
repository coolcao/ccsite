---
title: 【LeetCode每日一题】240.搜索二维矩阵II.md
date: 2021-01-11 11:43:05
tags: [算法,LeetCode,二分查找]
categories:
- 技术博客
- LeetCode每日一题
---

[240. 搜索二维矩阵 II](https://leetcode-cn.com/problems/search-a-2d-matrix-ii/)

>编写一个高效的算法来搜索 m x n 矩阵 matrix 中的一个目标值 target 。该矩阵具有以下特性：
>
>每行的元素从左到右升序排列。
>每列的元素从上到下升序排列。
> 
>**示例 1：**
>![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1610337100_20210111114634992_1297285897.png)
>输入：matrix = [[1,4,7,11,15],[2,5,8,12,19],[3,6,9,16,22],[10,13,14,17,24],[18,21,23,26,30]], target = 5
>输出：true
>**示例 2：**
>![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1610337101_20210111114648787_82602888.png)
>输入：matrix = [[1,4,7,11,15],[2,5,8,12,19],[3,6,9,16,22],[10,13,14,17,24],[18,21,23,26,30]], target = 20
>输出：false
>**提示：**
>* m == matrix.length
>* n == matrix[i].length
>* 1 <= n, m <= 300
>* -10^9 <= matix[i][j] <= 10^9
>* 每行的所有元素从左到右升序排列
>* 每列的所有元素从上到下升序排列
>* -10^9 <= target <= 10^9

<!-- more-->

这个题目是[74.搜索二维矩阵](https://leetcode-cn.com/problems/search-a-2d-matrix/description/)的升级版，升级的地方在于，这个二维矩阵的元素，每一列从上到下是升序的，但每一行的第一个元素并不比上一行的最后一个元素大，这样就不能和74题一样，我们拍平整个二维矩阵后能得到一个升序的数组。

这里我们仅仅能够得到，每一行是一个升序序列， 每一列是一个升序序列。

我们可以选取左下角或者右上角的元素作为初始位置。这里选取左下角。
如果元素大于target，那么上移，即row--。
如果元素小于target，那么右移，即col++。

因为左下角的元素，在行方向上是最小的，在列方向上是最大的。同样，右上角元素在行方向上是最大的，列方向上是最小的。

比如，我们要在下面这个二维矩阵中查找元素8，可以按照下面的查找路径：

![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1610344415_20210111135329815_332134507.png)

```golang
func searchMatrix(matrix [][]int, target int) bool {
    m, n := len(matrix), len(matrix[0])
    if m == 0 {
        return false
    }
    row, col := m-1, 0
    for row >= 0 && col <= n-1 {
        if matrix[row][col] == target {
            return true
        }
        if matrix[row][col] > target {
            row--
        } else {
            col++
        }
    }
    return false
}

```