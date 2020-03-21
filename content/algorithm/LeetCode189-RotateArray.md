---
title: "LeetCode189 RotateArray"
date: 2020-03-21T19:32:55+08:00
draft: false
toc: false
images:
tags:
  - LeetCode
  - 算法
---

```
Given an array, rotate the array to the right by k steps, where k is non-negative.

Example 1:

Input: [1,2,3,4,5,6,7] and k = 3
Output: [5,6,7,1,2,3,4]
Explanation:
rotate 1 steps to the right: [7,1,2,3,4,5,6]
rotate 2 steps to the right: [6,7,1,2,3,4,5]
rotate 3 steps to the right: [5,6,7,1,2,3,4]
Example 2:

Input: [-1,-100,3,99] and k = 2
Output: [3,99,-1,-100]
Explanation: 
rotate 1 steps to the right: [99,-1,-100,3]
rotate 2 steps to the right: [3,99,-1,-100]
Note:

Try to come up as many solutions as you can, there are at least 3 different ways to solve this problem.
Could you do it in-place with O(1) extra space?
```


## 思路
### 自己思路

> 自己最开始的思路是没有考虑空间复杂度的，如下  

- 通过规律发现可以将input数据分为两部分，一部分是[0, arr.length-k)，一部分是[arr.length-k, arr.length)
- 新定义一个临时数组存储，最后将临时数组的值再赋值给原数组

但是题目中要求的空间复杂度是O(1)，所以这种方案是不行的。

### 正解

利用反转，可以不用新开辟一个新的空间，规律如下：

- [1,2,3,4,5]，k =2
- 第一次翻转：[5,4,3,2,1]
- 第二次翻转，翻转[5,4]为[4,5]
- 第二次翻转，翻转[3,2,1]为[1,2,3]
- 最后结果是[4,5,1,2,3]
- yes


## 代码实现

```
public static void rotateArray(int[] arr, int k) {
    k %= arr.length;
    reverse(arr, 0, arr.length - 1);
    reverse(arr, 0, k - 1);
    reverse(arr, k, arr.length - 1);
}


/***
* * 翻转[start, end]的数组*
* ****@param***arr*
* ****@param***start*
* ****@param***end*
* */
private static void reverse(int[] arr, int start, int end) {
    //
    while (start < end) {
        int temp = arr[start];
        arr[start] = arr[end];
        arr[end] = temp;
        start++;
        end--;
    }
}
```

这里有几个点需要说明下：

### k取模

K取模是因为题目中没有说明k必须是小于等于arr.length的，如果是小于等于arr.length的话，k不需要取模

### start和end的定义

 因为要对数组中某一段元素进行互换，那么start和end就是这里的循环不变量，定义为互换的数组下标（以数组中间对称），这个定义中为啥`while(start < end)`而不是`<=`就有了解释，如果数组的个数是奇数，那么最中间的数是没必要交互的，所以是`<`，如果个数是偶数，那么刚好`start+1=end`时是需要交互的元素，也没必要相等。

## 总结

这道题不算难题，但是取模这个操作可能不太好想，并且利用3次翻转来完成空间复杂度是O(1）的算法，也不太好想。