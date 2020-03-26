---
title: " LeetCode 239. Sliding Window Maximum "
date: 2020-03-14T16:34:51+08:00
draft: false
toc: false
images:
comments: true
tags:
  - LeetCode
  - 算法
---


![W1GXbu](https://static.sailfishc.cn/Blog/W1GXbu.png)

这道题最开始看其实思路比较简单，但是做的时候发现实现还是挺麻烦的，题目的难度也是Hard，先说说自己做这道题目最开始的一个思路：

- 为了方便描述，把输入定义为`[1,3,-1,-3,5,3,6,7]`, k = 3
- 滑动窗口的移动有两种情况，一种是从窗口从索引0 -> 2移动，另外一种就是窗口长度已经是3之后的窗口移动

> 下面是自己最开始的思考过程，自己第一遍没有做出来，但是思路是比较接近答案的，这也就是算法中如果没有好的数据结构代码会写的又长又臭的原因  

- 第一步：对于`[1,3,-1]`的处理
- 第二步：对于最大值的处理，最开始想到的是定义一个`max`变量，然后使用`Math.max(max, nums[i])`这种方式去处理
- 第三步：移动窗口，用新进入窗口和上一个窗口最大值作比较，但是这一步有一种特殊情况，那就是得处理上一个窗口移出的值是不是最大值，比如`[3,-1,-3`中最大值是3，但是移动后，3已经被移除了，当前窗口的最大值是-1，为了解决这给问题自己最开始的思路是添加一个变量去记录是不是上一个窗口的第一个元素


## 解法一

```
public static int[] maxSlidingWindow(int[] nums, int k) {
    if (nums == null || nums.length == 0 || nums.length < k) {
        return new int[0];
    }

    Deque<Integer> deque = new LinkedList<>();
    int[] res = new int[nums.length + 1 - k];
    for (int I = 0; I < nums.length; I++) {
        if (!deque.isEmpty() && deque.peekFirst() == I - k) {
            deque.poll();
        }

        while (!deque.isEmpty() && nums[deque.peekLast()] < nums[I]) {
            deque.removeLast();
        }
        deque.offerLast(i);

        if (I + 1 >= k) {
            res[I + 1 - k] = nums[deque.peek()];
        }
    }
    return res;
}
```

> 这是自己写的第一篇LeetCode题解，所以主要是把自己在做题的过程中的问题全部记录下来  

### 问题一：数组长度为什么是nums.length + 1 - k

比如例子中的数组长度是8，窗口的长度是3，在前3个元素中有一个最大值，之后每移动一个，会产生一个最大值，所以长度回事`nums.length + 1 - k`

### 问题二：使用到的Deque的API	

[LinkedList (Java Platform SE 8 )](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html#poll--)

- peek
- peekFirst
- peekLast
- poll
- removeLast
- removeFirst
- offerLast

> 对于peek类的方法都是不删除元素的，对应的API用法可以参考Java Docs  

### 问题三：代码讲解

`Deque<Integer> deque = new LinkedList<>();`

- Deque的作用是方便的操作第一个元素和最后一个元素，**存储的是一个单调递减的数组下标**

```
if (!deque.isEmpty() && deque.peekFirst() == I - k) {
            deque.poll();
        }
```

- 这段代码的用途是用来删除在上一个窗口中队首元素是最大值得操作

```
while (!deque.isEmpty() && nums[deque.peekLast()] < nums[I]) {
            deque.removeLast();
        }
        deque.offerLast(i);
```

这段代码是这个算法的核心，它做得事情是维护队列的单调递减性，从`[1,3,-1,-3,5,3,6,7]`例子中可以分解下它是如何操作的：

- 1加入deque中
- 加入3是发现3大于1，将1删除
- 加入-1时-1<3，不删除3，直接加入-1
- 加入-3时-3<-1，不删除-1，直接加入-3
- 加入5时，5>-3,也大于-1，会将-3和-1都删除，加入5
- 因为后边给res赋值的时候使用的是peek，取的是队首元素，所以[3,-1,-3]会永远取3，直到5加入后会将-1和-3删除，下一个最大的数就是5了

## 总结
这道题还是最开始看的时候很简单，后来做的时候发现有很多处理比较难处理，并且Deque不一定可以直接想出来在这里运用，还是做不出来看看答案，然后再去理解，之后会写一篇我目前在用的算法学习方法。


## 参考资源
- [Find Maximum in Sliding Window - Coderust: Hacking the Coding Interview](https://www.educative.io/courses/coderust-hacking-the-coding-interview/k5llE)
- LeetCode
- [以练代学，学习算法和数据结构 :: 橙小张的博客  — 讨论技术，聊聊生活](https://blog.sailfishc.cn/posts/2019/10/%E4%BB%A5%E7%BB%83%E4%BB%A3%E5%AD%A6%E5%AD%A6%E4%B9%A0%E7%AE%97%E6%B3%95%E5%92%8C%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/)