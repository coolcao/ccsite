---
title: 链表反转.md
date: 2020-12-30 15:52:50
tags: [算法与数据结构, 链表]
categories:
  - 技术博客
  - 原创
---

链表是一个很简单的结构，虽然简单，但对于链表节点的操作，以及对于边界细节的处理，是面试中经常问到的。而且由于链表的定义符合递归定义，因此有时候考察递归思想时，也用链表来考察。

<!-- more -->

## 链表反转

给定一个单链表，写一个方法，将其反转，返回反转后的链表。
例如，给定一个链表 `1->2->3->4->5` 反转后 `5->4->3->2->1`

这是最基础的链表反转的题目，在 leetcode 上题目难度为简单。

对于链表的题目，可以使用递归思想，也可以使用迭代去解决。

### 递归

由于链表的定义就是递归的，因此链表的很多题目都可以使用递归。
使用递归的好处是，思考模型更直观，也更容易理解。

```golang
func reverseList(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}
	next := head.Next
	reversed := reverseList(next)
	next.Next = head
	head.Next = nil
	return reversed
}
```

### 迭代

```golang
func reverseList(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}
	pre, current, next := head, head.Next, head.Next.Next
	pre.Next = nil
	for current != nil {
		next = current.Next
		current.Next = pre

		pre = current
		current = next
	}

	return pre
}
```

## 链表区间反转

给定一个链表，反转其从 m 到 n 这一区间的节点。只允许一次遍历。

例如，`1->2->3->4->5`，m=2,n=4 反转后，`1->4->3->2->5`

这个题目在 leetcode 上难度为中等。

比上面单纯的反转整个链表相比，难点在于，要反转的是链表的某个区间，所以这里要精确控制其边界。

这里我们使用迭代的方式，其思想就是，遍历链表，等遍历到区间 m 到 n 时，反转这区间节点即可。

```golang
func reverseBetween(head *ListNode, m int, n int) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}
	if m >= n {
		return head
	}
	idx := 1
	var pre, current, next *ListNode
	var nodeM, mPre *ListNode

	current = head

	// 先找到第m个节点
	for current != nil && idx < m {
		pre = current
		current = current.Next
		idx++
	}

	nodeM = current
	mPre = pre

	pre = current
	current = current.Next
	next = current.Next
	pre.Next = nil

	// 开始反转区间链表
	for current != nil && idx < n {
		idx++
		next = current.Next
		current.Next = pre

		pre = current
		current = next
	}

	if mPre != nil {
		mPre.Next = pre
	} else {
		head = pre
	}
	nodeM.Next = current

	return head
}
```

## 链表间隔反转

给定一个链表，每 2 个节点为一组，进行反转。
例如，给定链表 `1->2->3->4->5`，反转后的链表为`2->1->4->3->5`。

```golang
func reverse(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}

	next := head.Next
	head.Next = reverse(next.Next)
	next.Next = head

	return next
}
```

## 链表间隔 k 反转

给定一个链表，每 k 个节点一组，进行反转。
例如，给定链表 `1->2->3->4->5->6->7`，k=3,反转后链表为 `3->2->1->6->5->4->7`。
这个题目是上面题目的一个延伸，上面是每 2 个节点为一组，这个题目是每 k 个节点，k 是变量，由调用方决定。

```golang
func reverseK(head *ListNode, k int) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}

	var pre, current, next *ListNode

	count := 0
	current = head
	for current != nil {
		count++
		current = current.Next
		if count == k {
			break
		}
	}
	// 如果链表长度小于k则不进行反转
	if count < k {
		return head
	}

	pre, current, next = head, head.Next, head.Next.Next
	pre.Next = nil
	idx := 2
	for current != nil && idx <= k {
		idx++
		next = current.Next
		current.Next = pre

		pre = current
		current = next
	}

	head.Next = reverseK(next, k)

	return pre

}
```
