---
title: 一道Easy的LeetCode题目引发的血案
date: 2019-10-11 11:31:59
tags: [动态规划, 算法与数据结构]
categories:
  - 技术博客
  - 原创
---

## LeetCode 题目

一直觉得，程序员应该持续的修炼内功，训练编程思维。最近也是不间断的在做 LeetCode 算法题，来锻炼思维。可是，今天在一道难度为 Easy 的题目上，栽了，受打击了，于是整理成此博文，来记录一下吧。

<!--more-->

先看看题目：

```
Say you have an array for which the ith element is the price of a given stock on day i.

If you were only permitted to complete at most one transaction (i.e., buy one and sell one share of the stock), design an algorithm to find the maximum profit.

Note that you cannot sell a stock before you buy one.

Example 1:

Input: [7,1,5,3,6,4]
Output: 5
Explanation: Buy on day 2 (price = 1) and sell on day 5 (price = 6), profit = 6-1 = 5.
             Not 7-1 = 6, as selling price needs to be larger than buying price.
Example 2:

Input: [7,6,4,3,1]
Output: 0
Explanation: In this case, no transaction is done, i.e. max profit = 0.
```

简单点说，就是给定一个数组，代表未来几天的股价价格预测，让你设计一个算法，来算出最大收益是多少。
比如给定的是[7,1,5,3,6,4]，我们应该输出 5，因为我们应该在第二天（价格为 1）买进，第五天（价格为 6）卖出，此时收益最大，为 5。

这个题目看上去就简单，真简单，而且看完题后都感觉 Low，随手一写不就出来了吗？

```golang
func maxProfit(prices []int) int {
    length := len(prices)
    max := 0
    for i := 0; i < length-1; i++ {
        for j := i + 1; j < length; j++ {
            diff := prices[j] - prices[i]
            if max < diff {
                max = diff
            }
        }
    }
    return max
}
```

两层遍历，分别代指买进和卖出时的价格，然后拿出最大的价格差不就 OK 了嘛。

确实，没毛病，可是一提交，过是过了，一看结果：

![提交结果](https://img001-10042971.cos.ap-shanghai.myqcloud.com/blog/f70e0459-db24-421a-9778-d7c35c623565.png)

只打败了 7.6% 的 golang 提交者，瞬时感觉不对头啊，应该还有更好的方式，不然这么一个简单的问题不会有 3231 个 👍🏻。

看了大家在评论区的评论，得到如下一种方法：

```golang
func maxProfit(prices []int) int {
    length := len(prices)
    if length == 0 {
        return 0
    }

    minPrice, maxProfit := prices[0], -1

    for _, p := range prices {
        if p < minPrice {
            minPrice = p
        }
        profit := p - minPrice
        if profit > maxProfit {
            maxProfit = profit
        }
    }

    return maxProfit
}
```

这次提交的结果明显好很多：
![结果](https://img001-10042971.cos.ap-shanghai.myqcloud.com/blog/fd01e30b-a1e1-4d8a-b40b-65af23b00f41.png)

这种方法，定义两个变量 minPrice,maxProfit 分别代指遍历到当前位置时的最小价格和最大收益。在遍历数组的时候，动态的改变最小价格和最大收益，这样只需要 O(n)的复杂度即可，相比第一个方法，提升了一个量级。

感觉上面第二种方法的代码有点眼熟，但又觉得，怎么才会想到这种方法呢？

👆🏻 第二种方法呢，使用的就是动态规划，这也是为什么觉得眼熟，但却很难想到的原因，因为我之前对动态规划就没理解好，老感觉动态规划是一很神奇的算法，那么今天就借这个题目机会，来再巩固一下动态规划的知识。

## 动态规划

动态规划(dynamic programming)是求解决策过程(decision process)最优化的数学方法，首先说，动态规划这一数学概念，还是挺“高深”的，包含各种数学公式，还有各种著作，先看一大神在知乎上的回答吧，感受一下：[什么是动态规划（Dynamic Programming）？动态规划的意义是什么？](https://www.zhihu.com/question/23995189/answer/723475721)。
这里我就不去研究动态规划底层的数理逻辑了，因为太高深了，我还不够格。那么，作为程序员，怎么将动态规划应用到编程上面来，才是重点。我就简单梳理一下在做类似的算法题时，如何应用动态规划算法，怎么去建模。

> 和分治算法一样，动态规划是通过组合子问题的解而解决整个问题的。分治算法是指将问题分成一些独立的子问题，递归的求解各子问题，然后合并子问题的解而得到原问题的解。与此不同，动态规划适用于子问题不是独立的情况，也就是各子问题包含公共子问题。动态规划算法对每个子子问题只求解一次，将其结果保存在一张表中，从而避免每次遇到子问题时重新计算答案。

👆🏻 一段是摘自《算法导论》中的描述，其中提到了一个分治算法，具体分治算法怎么样的，我们再找时间讨论，大概举一个例子就是有序数组的二分查找，还有一个例子就是归并排序，他们都是将一个大问题，拆解成若干个小问题，递归求解小问题的解，然后再合并为原问题的解。

动态规划是将一个大问题，拆分为若干小问题，依次求解这若干子问题，最后根据这些子问题的解来求解最终问题的解。这里面和分治算法的区别在于，这若干子问题不是独立的，可能会相互影响，而分治算法的子问题是相互独立的。

适合于用动态规划法求解的问题，经分解后得到的子问题往往不是互相独立的（即下一个子阶段的求解是建立在上一个子阶段的解的基础上，进行进一步的求解）。

动态规划的核心是**定义状态**和**设计状态转移公式**。

这里引入两个概念，状态和状态转移公式。

状态，可以理解为拆分成的子问题的解，定义状态即是定义子问题解的表示方法。状态转移公式，刚才说过了，拆分的这些子问题，可能是相互关联的，这个状态转移公式即是子问题的状态之间的关系公式。

## 例题解析

好了，上面说了那么一堆定义，概念，非常的枯燥无味，相信看到这里的观众也是不明白到底说了个啥。那我们就把上面那个 LeetCode 里的题作为例子来说一下这里面对应的所谓的状态，以及状态转移公式，该如何解决那个问题。

题目中，给定一个数组，表示未来几天股票的预测价格，让我们求解在这几天的预测价格，怎样买卖才能获得最大收益。

这个问题可以这么去描述：

> 给定一个数组 prices，长度为 N
>
> 设 F(k)为数组中从第一项到第 k 项的子数组的最大利润,minPrice(k)为第 k 项元素之前所有元素的最小值。
>
> 求 F(1)...F(n)中的最大值。

这里 F(k)以及 minPrice(k)即是状态，F(k)表示数组中从第一项到第 k 项的子数组的最大利润,minPrice(k)表示第 k 项之前所有元素的最小值。

接下来就是设计状态转移公式。就是 F(k)与 F(k-1)甚至 F(k-2)...F(1)之前的关系，我们来分析一下这个题目，要求 F(k)，和 F(k-1)是不是有关系呢？

F(k)表示从第一项到第 k 项的子数组的最大利润。这个最大利润和 F(k-1)的最大利润有关系么？

求 F(k)时，我们已经知道了 Price[k]，利润等于卖出价格减去买进价格，这里隐含了一个条件，那就是，买进日期肯定在卖出日前之前对吧，如果说 Price[k]为卖出价格，那么，买进价格一定在 F(k-1)这个状态之内，此时我们只需要对比如果是 Price[k]-minPrice 和 F(k-1)，取其中最大即可。

那么，状态转移公式也就出来了：

```
F(k) = max(F(k-1), Price[k]-minPrice(k))
```

有了这个状态转移公式，我们就根据这个公式去求解就可以了。最后写出的代码也就是 👆🏻 第二段代码。

代码中 minPrice 变量记录的就是从第一项到第 k 项的最小价格，maxProfit 记录的就是最大利润，对应着上面公式的 minPrice(k)和 F(k-1)。

这样只需要执行一次遍历，复杂度为 O(n)。

## 再理解

解析完了 👆🏻 例题，我们再回过头来看看动态规划的一些概念，加深理解。

动态规划有那么几个特性：

### 无后效性

简单来说，无后效性指的是，当前状态确定后，之后的状态转移与之前的状态就无关了。

上面的例子中，F(k)的状态与 F(k-1)状态有关，但是 F(k-1)是如何算出来的，对 F(k)这一状态来说是不关心的，也是没有影响的。如果这么说还不理解，再举一个简单的例子，下围棋就是无后效性，当前的棋局，不管是怎么来的，可能是随机下的，也可能深思熟虑下成这样，对于以后的决策，即下一步该怎么下是没影响的，影响下一步决策的，只有当前的棋局（即当前的状态）。

### 最优子结构

看 👆🏻 例子里，我们定义的状态，F(k)表示什么，F(k)表示从第一项到第 k 项的子数组的最大利润，这里 F(k)在定义时就已经是“最优”的了，即子问题的最优。

**大问题的最优解可以由小问题的最优解推出，这个性质就是最优子结构。**

上面例子中，F(k)是和 F(k-1)有关系，也是由 F(k-1)推出来的，满足最优子结构。

所以，如何判断一个问题能不能用动态规划算法去求解呢？

**能将大问题拆成几个小问题，且满足无后效性、最优子结构性质。**这个问题就可以使用动态规划算法求解。

对于一个动态规划问题，如何拆分问题，定义状态以及状态转移公式才是解决问题的难点。

## 再来两个 🌰️

### 数组最大子数组和

上面 LeetCode 的题目，其实稍加变形就是另外一个例子。

上面给定数组表示股票价格，比如[7,1,5,3,6,4]代表未来 6 天的股票预测价格，那么我们是不是可以由这个股票价格得出股票的涨跌走势，得到一个新的数组[-6,4,-2,3,-2]，这个数组每个数表示股票价格的涨跌。这时，要求最大的利润，是不是就变成了求这个涨跌数组的最大子数组，因为这个数组里的每一项可表示为每天的利润。现在问题就转变成了求解一个数组的最大子数组问题了。

同样，这个问题能不能用动态规划呢，我们来分析一下。

> 给定一个数组 profits，长度为 N
>
> 我们定义 F(k)为以第 k 项结尾的子数组的最大和。
> 那么，F(k)就只与 profits[i-1]和 F(k-1)相关，即
>
> 状态转移公式： `F(k)=max(F(k-1)+Profits(k), Profits(k))`
>
> 求 F(1)...F(n)中的最大值
>
> 公式说明：F(k)的定义是，以第 k 项结尾的子数组的最大和，这里结尾元素一定是 Profits(k)，而起始元素，不管是从哪个开始的(肯定是 1~(k-1)中的某一个)，这里不关心。
>
> 最终要求的问题的解为 F(1)...F(n) 的最大值

这样定义满足刚才说的无后效性和最优子结构么，首先，F(k)定义的是以第 k 项(Profits(k))结尾的子数组的最大和，至于这个子数组的起始位置是哪一个，我们不关心，当 F(k)定了，后续的状态只与 F(k)有关，所以满足无后效性。本身定义的 F(k)就是最大子数组之和，满足最优子结构。

好，实现一下代码试试：

```golang
func maxProfit4(prices []int) int {
    length := len(prices)
    if length == 0 || length == 1 {
        return 0
    }

    // profits为价格差，从第二天开始，每天相对于前一天的利润
    profits := make([]int, length-1)
    for i := 1; i < length; i++ {
        profits[i-1] = prices[i] - prices[i-1]
    }

    // maxProfits[i]表示从profits[0]到profits[i]的连续子数组的最大和
    maxProfits := make([]int, length-1)
    maxProfits[0] = profits[0]

    // 记录maxProfits中的最大值
    maxProfit := maxProfits[0]

    for i := 1; i < length-1; i++ {
        maxProfits[i] = max(profits[i], maxProfits[i-1]+profits[i])
        maxProfit = max(maxProfits[i], maxProfit)
    }
    return max(maxProfit, 0)
}
```

我们定义了一个数组 maxProfits，maxProfits[k]表示以 profits[k]为结尾的子数组的最大和，即，对应者状态转移公式中的 F(k)。
`maxProfits[i] = max(profits[i], maxProfits[i-1]+profits[i])`即是状态转移公式。
maxProfit 用以记录 maxProfits 数组中的最大值，即这个题目中我们要求的最终解。

> 注：其实 maxProfits 并不需要定义成一个数组，因为最后我们需要的状态只和前一个状态有关，也就是说，每次递推，我们只用到了 maxProfits[i]，可以使用一个变量即可，只不过这里为了和刚才状态转移公式对应，定义成了一个数组。

![提交结果](https://img001-10042971.cos.ap-shanghai.myqcloud.com/blog/af255533-dded-43ee-8808-0bb7244e85a2.png)
从最终的结果，我们可以看出，效率并不如上面第二种方式高，因为这里我们先将原股价数组转换成了一个利润数组，使用的这个中间结果利润数组求的，而且在求解过程中，还定义了 maxProfits 数组来存放中间结果，因此使用的内存也比第二种方式多。不过请注意，这种方式的时间复杂度也是 O(n)。

### 爬楼梯问题

```
一个人爬楼梯，每次只能爬1个或2个台阶，假设有n个台阶，那么这个人有多少种不同的爬楼梯方法。
```

首先，这个问题放到这里，肯定是可以使用动态规划算法去解决的，那么应该怎么拆分子问题，定义状态以及状态转移公式呢？

设想一下，我们要爬到第 k 阶台阶，我们怎么爬上去呢？因为每次只能爬一个或两个台阶，所以，要爬到第 k 阶台阶，有两种方法，从第 k-1 阶台阶爬 1 阶台阶上去，或者从第 k-2 阶台阶爬 2 阶台阶上去。

> 定义 F(k)为爬到 k 阶台阶的总方法数
>
> F(k)=F(k-1)+F(k-2)
>
> 👆🏻 即状态转移公式，F(k) 的状态与 F(k-1)与 F(k-2)相关

有了状态转移公式，代码就好写了：

```golang
func climbStairs(n int) int {
    if n == 1 || n == 2 {
        return n
    }

    return climbStairs(n-1) + climbStairs(n-2)
}
```

在解决这个问题时，要到了递归，递归一定要有终结条件，这里的终结条件就是 n==1 或 n==2 的时候，因为 F(k)=F(k-1)+F(k-2)有关，因此终结条件有两个。

👆🏻 这直接使用递归，这里面重复算了很多，我们可以优化一下，将之前的计算结果缓存起来，以供后面的状态使用。

```golang
func climbStairs(n int) int {
    steps := make([]int, n)
    steps[0], steps[1] = 1, 2
    for i := 2; i < n; i++ {
        steps[i] = steps[i-1] + steps[i-2]
    }
    return steps[n-1]
}
```

### 最长上升子序列

```
最长上升子序列（LIS）问题：给定长度为n的序列nums，从nums中抽取出一个子序列，这个子序列需要单调递增。问最长的上升子序列（LIS）的长度。
例如，给定序列 [1,5,3,4,6,9,7,8]的LIS为[1,3,4,6,7,8]，长度为6。
```

这个问题和 👆🏻 第一个问题最大子数组有点类似，只不过最大子数组问题要求子数组是连续的，而最长子序列中元素并不一定是连续的。
即便如此，思考的角度是类似的。最大子数组问题，我们定义的状态是以第 k 个元素结尾的子数组长度，这个最长上升子序列，我们用同样的思路，定义如下状态：

> 给定一个序列 nums，长度为 n
>
> 定义 F(k)为以 nums[k]为结尾的最长子序列长度
>
> 对于以 nums[0]到 nums[k]的子序列中，可能会存在一个 i，使得`nums[i]<nums[k]`，即：`i<k && nums[i]<nums[k]`
> 注：此时 nums[i]和 nums[k]可能并不连续
>
> 那么，F(k) = max(F(i)+1, F(k))

因此，这里我们只需要两层循环，分别标注 i 和 k，然后比较 nums[i]和 nums[k]即可。

```golang
func lis(nums []int) {
    length := len(nums)
    if length == 0 {
        return
    }
    lis := make([]int, length)
    lis[0] = 1
    maxLen := lis[0]
    for i := 0; i < length-1; i++ {
        for j := i + 1; j < length; j++ {
            if nums[j] > nums[i] {
                lis[j] = max(lis[i]+1, lis[j])
                maxLen = max(maxLen, lis[j])
            }
        }
    }
    return maxLen
}

```

### 不同路径问题

```
一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为“Start” ）。
机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为“Finish”）。
现在考虑网格中有障碍物。那么从左上角到右下角将会有多少条不同的路径？

```

![](https://img001-10042971.cos.ap-shanghai.myqcloud.com/blog/robot_maze.png)

```
网格中的障碍物和空位置分别用 1 和 0 来表示。
说明：m 和 n 的值均不超过 100。
示例 1:
输入:
[
  [0,0,0],
  [0,1,0],
  [0,0,0]
]
输出: 2
解释:
3x3 网格的正中间有一个障碍物。
从左上角到右下角一共有 2 条不同的路径：
1. 向右 -> 向右 -> 向下 -> 向下
2. 向下 -> 向下 -> 向右 -> 向右

注：此题为LeetCode第63个题目，难度为中等。

```

这个问题和 👆🏻 那个爬楼梯那个分析方法类似。
最右下角 Finish 位置，怎么样才能走过去呢？因为机器人每次只能 👇🏻 或 👉🏻 移动一步，因此，走像 Finish 位置有两种方法，从其上面那个位置 👇🏻 走一步，或从其左边的位置 👉🏻 走一步即可。

比如实例 1 中，finish 处的座标是 [2,2]，走到这个位置有两种方式，从[1,2]向下移动一步或从[2,1]位置向右移动一步。那么，总方法数就是这两个位置的方法数之和。

> 给定一个 m\*n 的二维数组，表示机器人要走的座标，其中 0 的位置可走，1 的位置为障碍物
>
> 定义 F(i,j)表示从开始位置[0,0]走到[i,j]位置的总路径数
>
> 那么，F(i,j) = F(i-1,j) + F(i,j-1)

好了，状态定义以及状态转移公式都出来了，代码就好写了。

写代码时，要注意边界问题，因为 F(i,j)是有其“上面”位置或“左边”位置移动一步而来，因此注意“上边”或“左边”位置不存在的情况。

```golang
func uniquePathsWithObstacles(obstacleGrid [][]int) int {
    rows := len(obstacleGrid)
    if rows == 0 {
        return 0
    }
    cols := len(obstacleGrid[0])

    steps := make([][]int, rows)
    for i := 0; i < rows; i++ {
        steps[i] = make([]int, cols)
    }

    // 设置初始位置
    if cols > 0 && obstacleGrid[0][0] == 0 {
        steps[0][0] = 1
    }

    for i := 0; i < rows; i++ {
        for j := 0; j < cols; j++ {
            current := obstacleGrid[i][j]
            // 初始位置或遇到障碍物
            if current == 1 || (i == 0 && j == 0) {
                continue
            }
            if i == 0 {
                steps[i][j] = steps[0][j] + steps[i][j-1]
            } else if j == 0 {
                steps[i][j] = steps[i-1][j] + steps[i][0]
            } else {
                steps[i][j] = steps[i-1][j] + steps[i][j-1]
            }
        }
    }

    return steps[rows-1][cols-1]
}

```

定义一个二维数组 steps，来标记[i,j]从起始位置到[i,j]位置的总路径数，那么，初始状态即为[0,0]位置的路径数，其余的后续状态根据状态转移公式即可逐步推算。

### 最少时间问题

```
小明家住A城市，他有事要到B城市办事，从A城市到B城市中间有n个城市，
每个城市到下一个城市都会有两种方式，乘坐火车或者乘坐汽车，
输入一个二维数组，其中每个元素三个值，分别表示该城市到下一城市乘坐火车的时间，乘坐汽车的时间，以及从火车站到汽车站的时间。
假设，小明家到A城市的火车站以及到汽车站的时间都是30分钟，那么，求小明从家到B城市所需的最少时间。

输入是一个二维数组：
[
    [30,40,10],
    [35,20,15],
    [38,32,8]
]
表示从小明所在的A城市到B城市有3个中间城市，
从A到C1乘坐火车需30分钟，乘坐汽车需40分钟，A城市从火车站到汽车站需要10分钟（反过来从汽车站到火车站也是10分钟），
从C1到C2乘坐火车需要35分钟，乘坐汽车需要20分钟，C1城市从火车站到汽车站需要15分钟，
从C2到B城市乘坐火车需要38分钟，乘坐汽车需要32分钟，C2城市火车站到汽车站需要8分中。
```

这个问题比上面几个问题稍复杂点，这里交通方式有火车，有汽车，还涉及火车与汽车的换乘。

首先分析一下题目中给出的例子，画如下示意图：

![图片](https://img.coolcao.site/file/84b6d61a446b0a3081dcb.png)

题目中给出的时间数组表示上一个城市与下一个城市的交通时间花费，因此总的城市是数组长度加1。

对于中间任意城市而言，有两种方式到达：1. 乘坐火车 2. 乘坐汽车
我们定义 `train[i]` 为到达第i个城市时乘坐火车所需的最小时间，定义 `bus[i]` 为到达第i个城市乘坐汽车所需的最小时间，那么到达该城市所需的最小时间两者取小即可。

对于 `train[i]` 而言：
1. 可以由上一个城市的火车站乘坐火车而来，此时 `train[i]=train[i-1]+times[i-1][0]`
2. 也可以由上一个城市的汽车站换乘火车而来，此时 `train[i]=bus[i-1]+times[i-1][2]+times[i-1][0]`
3. 两者取最小即可

同理，对于 `bus[i]` 而言：
1. 可以由上一个城市的汽车站乘坐汽车而来，此时 `bus[i]=bus[i-1]+times[i-1][1]`
2. 也可以由上一个城市的火车站换乘汽车而来，此时 `bus[i]=train[i-1]+times[i-1][2]+times[i-1][1]`
3. 两者取最小即可

对于初始状态 `train[0]` 和 `bus[0]` ，由于题目中指出小明家到A城市的火车站和汽车站所需时间都是30分钟，因此两者都初始化为30，即：

`train[0]=30, bus[0]=30`

有了初始状态以及状态转移公式，代码就好写了。

```golang
func minTime(times [][]int) int {
    n := len(times)

    train := make([]int, n+1)
    bus := make([]int, n+1)

    train[0] = 30
    bus[0] = 30

    for i := 1; i <= n; i++ {
        train[i] = min(
            train[i-1]+times[i-1][0],
            bus[i-1]+times[i-1][2]+times[i-1][0],
        )

        bus[i] = min(
            bus[i-1]+times[i-1][1],
            train[i-1]+times[i-1][2]+times[i-1][1],
        )
    }

    return min(train[n], bus[n])
}
```

## 陈词总结

通过 👆🏻 几个例子，我们可以发现，其实动态规划并不难理解。

如果一个问题由交叠的子问题构成，我们都可以使用动态规划的方式来解决它。一般来说，这样的子问题出现在对给定问题求解的递推关系中，这个递推关系中包含了相同问题的更小子问题的解。动态规划建议，与其对交叠子问题一次又一次的求解，还不如对每个子问题只求解一次，并把结果记录在表中，这样我们就可以得从表中得出原始问题的解。👆🏻 爬楼梯那个问题就是，我们使用了一个数组来缓存子问题的解，然后后面的问题，只需要从这个缓存数组中即可得到，不用再计算。

动态规划类的题目，真正难点在于，如何去拆分子问题，定义状态以及状态转移公式。如果能够拆分出状态，定义好状态，写出状态转移公式，那么，写代码只是分分钟的事。这类问题，流程是固定的，拆分问题，定义状态，设计状态转移公式。但是，题目类型确实千变万化，要想真正搞定动态规划，还是要不断的练习，不断的总结。
