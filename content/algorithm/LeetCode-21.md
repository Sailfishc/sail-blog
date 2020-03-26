---
title: "LeetCode 21. Merge Two Sorted Lists"
date: 2020-03-26T23:00:40+08:00
draft: false
toc: false
images:
comments: true
tags:
  - LeetCode
  - 算法
---

## 题目解读

```
Merge two sorted linked lists and return it as a new list. The new list should be made by splicing together the nodes of the first two lists.

Example:

Input: 1->2->4, 1->3->4
Output: 1->1->2->3->4->4
```

> 注意事项  

- 两个链表都是有序的

## 思考路径
### 第一遍思路

> 我是在Educative上做的题目，题目的原始代码结构对我有一定的误区，因为他的题目结构是这样的：  

```
public static LinkedListNode mergeSorted(LinkedListNode head1, LinkedListNode head2) {    
    //TODO: Write - Your - Code
    return head1;
  }
```

已经指定了必须返回`head1`,基于这个设定，我的思路如下：

- 比较head1和head2的第一个值，如果head1.val小于head2.val，那么head1和head2交换
- 设置node1和node2，分别赋值为head1和head2
- 这个时候我思考了几个case：
	- 当node1 <= node2 <= node.next时，node2才能插入到node1之后
	- 如果head1或者head2遍历不相等，需要考虑对剩下的元素进行处理，例如必须返回head1，那么head1长度如果比head2短，那么head2都添加到了head1之后还有其他元素如何处理
	- xxxxxx

反正就是考虑了很多case，然后觉得好复杂，这也是我在做算法题的时候一个问题，就是出发点不对，就会衍生出来很多复杂的case，觉得这个题目很难，但是其实是一道easy的题目。

### 正确的思路

- 设置一个dummy节点作为头节点
- 再设置一个cur节点作为连接L1和L2的节点（L1指的是head1，L2指的是Head2）
- 遍历L1和L2，直到L1或者L2有一个为空跳出循环
- 如果L1的值比L2的值小，则`cur.next = new ListNode(L1.val)`,将L1移动到下一个节点
- 反之操作L2
- 移动cur节点
- 遍历结束
- 由于L1或者L2元素不是定长的，所以需要判断L1和L2如果不为空，将其追加到cur节点
- 返回dummy.next即可

### 递归思想

> 递归真是的给人力量  

只要可以把问题集缩小，就可以使用递归，这个总结虽然不严谨，但是也可以做一个参考，这个问题中，两个链表都是有序的，所以可以抽象这一个过程：

- 比较L1和L2的值，如果L1的值小于L2，那么下一个小的问题集就是L1当前的元素，以及L1.next和L2。伪代码:   L1,  L1.next =  缩小的链表集, 缩小的链表集 = (L1.next, L2)

## 代码

```
public static ListNode mergeSorted(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode cur = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val < l2.val) {
            cur.next = new ListNode(l1.val);
            l1 = l1.next;
        } else {
            cur.next = new ListNode(l2.val);
            l2 = l2.next;
        }
        cur = cur.next;
    }

    if (l1 != null) {
        cur.next = l1;
    }
    if (l2 != null) {
        cur.next = l2;
    }
    return dummy.next;
}
```


### 递归代码

```
public static ListNode mergeSorted2(ListNode l1, ListNode l2) {
    if (l1 == null ) {
        return l2;
    }
    if (l2 == null) {
        return l1;
    }

    if (l1.val < l2.val) {
        l1.next = *mergeSorted2*(l1.next, l2);
        return l1;
    } else {
        l2.next = *mergeSorted2*(l1, l2.next);
        return l2;
    }
}
```