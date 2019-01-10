---
title: 基础排序算法总结
date: 2018-10-29 19:39:02
tags: [排序算法]
categories:
- 博客
- 原创
---

排序算法分为内部排序和外部排序，而我们经常说的基础排序算法，都是内部排序算法。包括冒泡排序，选择排序，插入排序，快速排序，并归排序，希尔排序，堆排序等。

这里总结一下这几种排序算法，以备不时之需。

<!-- more -->

## 冒泡排序
----
> 它重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误就把他们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。这个算法的名字由来是因为越小的元素会经由交换慢慢“浮”到数列的顶端。
--- [冒泡排序-维基百科](https://zh.wikipedia.org/wiki/%E5%86%92%E6%B3%A1%E6%8E%92%E5%BA%8F)

上面是摘自维基百科上冒泡排序的说明，之所以起名冒泡排序，是因为整个过程非常形象，每次经过比较交换后，较大的值都会“浮”到最后面（或者较小的值“浮”到最前面）。

动画示意图：
![冒泡排序](https://img001-10042971.cos.ap-shanghai.myqcloud.com/blog/bubble-sort.gif)

它的过程如下：

1. 比较相邻的元素。如果第一个比第二个大，就交换他们两个。
2. 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。这步做完后，最后的元素会是最大的数。
3. 针对所有的元素重复以上的步骤，除了最后一个。
4. 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。

|||
|----|----|
|最坏时间复杂度|О(n²)|
|最优时间复杂度|O(n)|
|平均时间复杂度|О(n²)|
|最坏空间复杂度|总共O(n)|
|需要辅助空间|O(1)|

### 代码实现
```c
#include <stdio.h>
#define LEN 5

void sort(int a[], int length) {
  for (int i = length - 1; i > 0; i--) {
    for (int j = 0; j < i; j++) {
      if (a[i] < a[j]) {
        swapArray(a, i, j); // 交换数组的i，j元素
      }
    }
  }
}

int main(int argc, char const *argv[]) {
  int a[LEN] = {10, 5, 2, 4, 7};
  sort(a, LEN);
  printf("最终结果：%d %d %d %d %d\n", a[0], a[1], a[2], a[3], a[4]);
  return 0;
}

```

## 选择排序
----
> 选择排序（Selection sort）是一种简单直观的排序算法。它的工作原理如下。首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。
[选择排序-维基百科](https://zh.wikipedia.org/wiki/%E9%80%89%E6%8B%A9%E6%8E%92%E5%BA%8F)

动画示意图：

![选择排序](https://img001-10042971.cos.ap-shanghai.myqcloud.com/blog/Selection-Sort-Animation.gif)

|||
|----|----|
|最坏时间复杂度|	О(n²)|
|最优时间复杂度|	О(n²)|
|平均时间复杂度|	О(n²)|
|最坏空间复杂度|	О(n) total, O(1) auxiliary|

### 代码实现

```c
#include <stdio.h>
#define LEN 5

void sort(int a[], int length) {
  for (int i = 0; i < length - 1; i++) {
    int minIdx = i;
    for (int j = i + 1; j < length; j++) {
      if (a[j] < a[minIdx]) {
        minIdx = j;
      }
    }
    swapArray(a, minIdx, i);
  }
}

int main(int argc, char const *argv[]) {
  int a[LEN] = {10, 5, 2, 4, 7};
  sort(a, LEN);
  return 0;
}

```


## 插入排序
----
> 它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。插入排序在实现上，通常采用in-place排序（即只需用到O(1)的额外空间的排序），因而在从后向前扫描过程中，需要反复把已排序元素逐步向后挪位，为最新元素提供插入空间。
[插入排序--维基百科](https://zh.wikipedia.org/wiki/%E6%8F%92%E5%85%A5%E6%8E%92%E5%BA%8F)


动画示意图：

![插入排序](https://img001-10042971.cos.ap-shanghai.myqcloud.com/blog/220px-Insertion-sort-example-300px.gif)

|||
|----|----|
|最坏时间复杂度|О(n²)|
|最优时间复杂度|	O(n)|
|平均时间复杂度|	О(n²)|
|最坏空间复杂度|	总共 O(n) ，需要辅助空间 O(1)|

### 代码实现

```c
#include <stdio.h>
#include "../common/utils.h"
#define LENGTH 5

void sort(int array[], int length) {
  for (int i = 1; i < length; i++) {
    for (int j = 0; j < i; j++) {
      if (array[i] < array[j]) {
        swapArray(array, i, j);
      }
    }
  }
}

int main(int argc, char const* argv[]) {
  int array[LENGTH] = {10, 5, 2, 4, 7};
  sort(array, LENGTH);
  printf("最终结果：");
  printArray(array, LENGTH);
  return 0;
}

```


## 快速排序
----
快速排序采用分而治之的策略，找一个基准值，将所有小于基准值的元素放到该基准值前面，所有大于基准值的元素放到基准值的后面。
然后递归的将基准值的左右两部分子序列进行快速排序。

步骤：
1. 从数列中挑出一个元素，称为“基准”（pivot），
2. 重新排序数列，所有比基准值小的元素摆放在基准前面，所有比基准值大的元素摆在基准后面（相同的数可以到任何一边）。在这个分割结束之后，该基准就处于数列的中间位置。这个称为分割（partition）操作。
3. 递归地（recursively）把小于基准值元素的子数列和大于基准值元素的子数列排序。

递归到最底部时，数列的大小是零或一，也就是已经排序好了。这个算法一定会结束，因为在每次的迭代（iteration）中，它至少会把一个元素摆到它最后的位置去。

从上面可以看出，该算法采用递归思想。

动画示意图：

![快速排序](https://img001-10042971.cos.ap-shanghai.myqcloud.com/blog/Sorting_quicksort_anim.gif)

|||
|----|----|
|最坏时间复杂度|О(n²)|
|最优时间复杂度|O(nlogn)|
|平均时间复杂度|O(nlogn)|

### 代码实现
```c
#include <stdio.h>
#define LEN 5

/*
 * 快速排序
 * 先选定一个元素，将所有小于该元素的元素放到该元素的左边，所有大于该元素的元素放到该元素的右边
 * 分别对左右两部分进行快速排序
 * 直到每次排序的元素只有一个元素为止
 */

void sort(int a[], int length, int start, int end) {
  printf("排序区间：a[%d]-a[%d]\n", start, end);
  if (start >= end) {
    return;
  }
  int idx = start;
  for (int i = start + 1; i <= end; i++) {
    // 将所有小于基准的值放到该基准值的前面
    if (a[idx] > a[i]) {
      int tmp = a[i];
      int sidx = i;
      while (sidx > idx) {
        a[sidx] = a[sidx - 1];
        sidx -= 1;
      }
      a[idx] = tmp;
      idx += 1;
    }
  }
  for (int i = 0; i < length; i++) {
    printf("%d ", a[i]);
  }
  printf("\n-----------------------\n");
  if (idx > 0) {
    sort(a, length, start, idx - 1);
  }
  if (idx < length - 1) {
    sort(a, length, idx + 1, end);
  }
}

int main(int argc, char const *argv[]) {
  int a[LEN] = {10, 5, 2, 4, 7};
  sort(a, LEN, 0, LEN - 1);
  printf("最终结果：%d %d %d %d %d\n", a[0], a[1], a[2], a[3], a[4]);
  return 0;
}

```

## 归并排序
----
归并排序和上面的快速排序相似，也是采用分而治之的策略，同样也是用递归的思想。

不同点在于，归并排序依赖于归并操作，即将两个已经排好序的子序列合并成一个大的有序序列。

步骤：
1. 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列
2. 设定两个指针，最初位置分别为两个已经排序序列的起始位置
3. 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置
4. 重复步骤3直到某一指针到达序列尾
5. 将另一序列剩下的所有元素直接复制到合并序列尾

动画示意图：
![归并排序](https://img001-10042971.cos.ap-shanghai.myqcloud.com/blog/Merge-sort-example-300px.gif)

|||
|----|----|
|最坏时间复杂度|O(nlogn)|
|最优时间复杂度|O(nlogn)|
|平均时间复杂度|O(nlogn)|
|最坏空间复杂度|O(n)|

### 代码实现
```c
#include <stdio.h>
#include "../common/utils.h"

#define LEN 12

/**
 * 对两个已经拍好序的子序列进行合并操作
 *
 */
void merge(int a[], int start, int mid, int end) {
  // 左侧序列的范围是 [start, mid]
  int leftLenght = mid - start + 1;
  // 右侧序列的范围是 [mid+1, end]
  int rightLenght = end - mid;
  int left[leftLenght], right[rightLenght];
  // 将左右两侧数据分别放到两个临时数组中
  for (int i = 0; i < leftLenght; i++) {
    left[i] = a[start + i];
  }
  for (int i = 0; i < rightLenght; i++) {
    right[i] = a[mid + 1 + i];
  }

  printf("left:");
  printArray(left, leftLenght);
  printf("right:");
  printArray(right, rightLenght);

  // 左右两部分进行合并
  int start1 = 0, start2 = 0;
  int i = start;
  while (start1 < leftLenght && start2 < rightLenght) {
    if (left[start1] < right[start2]) {
      a[i] = left[start1];
      start1 += 1;
    } else {
      a[i] = right[start2];
      start2 += 1;
    }
    i += 1;
  }

  while (start1 < leftLenght) {
    a[i++] = left[start1++];
  }

  while (start2 < rightLenght) {
    a[i++] = right[start2++];
  }

  printf("merge: ");
  printArray(a, leftLenght + rightLenght);
  printf("----------\n");
}

void sort(int a[], int start, int end) {
  int mid = (start + end) / 2;
  if (start != mid) {
    sort(a, start, mid);
  }
  if (mid + 1 != end) {
    sort(a, mid + 1, end);
  }
  merge(a, start, mid, end);
}

int main(int argc, char const *argv[]) {
  int a[LEN] = {3, 5, 6, 4, 7, 9, 8, 0, 1, 2, 19, 15};

  sort(a, 0, LEN - 1);
  printf("排序后：");
  printArray(a, LEN);
  return 0;
}

```

## 希尔排序
----
希尔排序，也称递减增量排序算法，是插入排序的一种更高效的改进版本。

希尔排序通过将比较的全部元素分为几个区域来提升插入排序的性能。这样可以让一个元素可以一次性地朝最终位置前进一大步。然后算法再取越来越小的步长进行排序，算法的最后一步就是普通的插入排序，但是到了这步，需排序的数据几乎是已排好的了（此时插入排序较快）。

例如，假设有这样一组数[ 13 14 94 33 82 25 59 94 65 23 45 27 73 25 39 10 ]，如果我们以步长为5开始进行排序，我们可以通过将这列表放在有5列的表中来更好地描述算法，这样他们就应该看起来是这样：

```
13 14 94 33 82
25 59 94 65 23
45 27 73 25 39
10
```

然后我们对每列进行排序：

```
10 14 73 25 23
13 27 94 33 39
25 59 94 65 82
45
```

将上述四行数字，依序接在一起时我们得到：[ 10 14 73 25 23 13 27 94 33 39 25 59 94 65 82 45 ].这时10已经移至正确位置了，然后再以3为步长进行排序：

```
10 14 73
25 23 13
27 94 33
39 25 59
94 65 82
45
```

排序之后变为：

```
10 14 13
25 23 33
27 25 59
39 65 73
45 94 82
94
```

最后以1步长进行排序（此时就是简单的插入排序了）。

### 代码实现
```c
#include <stdio.h>

#define LEN 10

void sort(int a[], int length) {
  for (int gap = length / 2; gap > 0; gap /= 2) {
    printf("gap: %d\n", gap);
    for (int g = 0; g < gap; g++) {
      int tmplength = length / gap;

      // 针对每个子间隔序列进行插入排序
      for (int i = g + gap; i < tmplength * gap + g; i += gap) {
        for (int j = g; j < i; j += gap) {
          if (a[i] < a[j]) {
            swapArray(a, i, j);
          }
        }
      }
      printArray(a, LEN);
    }
  }
}


int main(int argc, char const *argv[]) {
  int a[LEN] = {10, 5, 2, 4, 7, 3, 1, 9, 8, 6};
  sort(a, LEN);
  printArray(a, LEN);
  return 0;
}

```

## 堆排序
----
堆排序（英语：Heapsort）是指利用堆这种数据结构所设计的一种排序算法。堆是一个近似完全二叉树的结构，并同时满足堆积的性质：即子结点的键值或索引总是小于（或者大于）它的父节点。

```

          1                         11
        /   \                      /  \
       2     3                   9     10
     /  \   /  \                / \    /  \
    4    5  6  7               5   6   7   8
   / \  / \                   /\   /\
  8  9 10 11                 1  2 3  4

   小顶堆                         大顶堆
```

如上图所示，就是小顶堆和大顶堆的示意图。

堆一般用数组来表示，有如下特点：
* 索引为i的左孩子的索引是 `2*i`
* 索引为i的右孩子的索引是 `2*i+1`
* 索引为i的父节点的索引是 `floor(i/2)`

堆排序即是用堆的这种特性，先将数组构造成小顶堆（或大顶堆），堆的根元素即为最小值（或最大值）。
然后再一次将剩下的元素再构造成小顶堆（或大顶堆），直至最后的堆只有一个元素为止。

步骤：
1. 数组调整为小顶堆
2. 取小顶堆的根节点，放到第一个元素
3. 递归调整剩下的元素为小顶堆，并取堆的根节点

|||
|----|----|
|最坏时间复杂度|O(nlogn)|
|最优时间复杂度|O(nlogn)|
|平均时间复杂度|O(nlogn)|


### 代码实现

```c
#include <stdio.h>

#define LEN 15

/**
 * arr: 传入的数组
 * start: 开始位置
 * end: 结束位置
 * offset: 偏移量，即从哪个位置开始做最小堆
 */
void swapMin(int arr[], int start, int end, int offset) {
  int dad = start;
  int son = dad * 2 + 1 - offset;
  if (son >= end) {
    return;
  }

  // 先比较两个子节点，选择最小的
  if (son + 1 < end && arr[son] > arr[son + 1]) {
    son += 1;
  }
  // 如果父节点小于子节点，交换父子节点
  if (arr[dad] >= arr[son]) {
    swapArray(arr, dad, son);
  }
}

/**
 * 构建最小堆
 * arr: 数组
 * start: 起始元素。从start元素往后的所有元素构建最小堆。start之前的表示已排好序
 * length: 数组的长度
 *
 */
void minHeapify(int arr[], int start, int length) {
  for (int i = length / 2 - 1 + start; i >= start; i--) {
    swapMin(arr, i, length, start);
  }
}

/**
 * 排序方法
 * arr: 数组
 * length: 数组长度
 *
 */
void sort(int arr[], int length) {
  for (int i = 0; i < length; i++) {
    minHeapify(arr, i, length);
  }
}

int main(int argc, char const *argv[]) {
  int a[LEN] = {10, 5, 3, 4, 7, 3, 2, 9, 7, 6, 12, 8, 3, 15, 20 };
  sort(a, LEN);
  printArray(a, LEN);
  return 0;
}

```
