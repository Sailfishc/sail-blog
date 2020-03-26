---
title: "LeetCode 283. Move Zeroes"
date: 2020-03-26T13:55:37+08:00
draft: false
toc: false
images:
comments: true
tags:
  - LeetCode
  - 算法
  - 快慢指针
---

## 题目解读

```
Given an array nums, write a function to move all 0's to the end of it while maintaining the relative order of the non-zero elements.

Example:

Input: [0,1,0,3,12]
Output: [1,3,12,0,0]
Note:

1. You must do this in-place without making a copy of the array.
2. Minimize the total number of operations.

```


> 注意点：  

- 维持数组的相对顺序，也就是不能改变非0元素的相对顺序
- 不使用新的数组
- 最少的操作数（操作数指的是对数组的赋值操作）

## 思路
> 这道题难度是easy，思路也比较简单，有点类似快慢指针的做法  

- 定义一个索引nonZero（慢指针），定义为在[0, nonZero]的元素都是符合条件的
- 遍历数组，索引下标为index（快指针）
- 如果index对应的值不等于0，则将index的值赋值给nonZero索引，将index和nonZero索引向后移动
- 如果index对应的值等于0，则只移动index的值
- 直到index遍历结束
- 这个时候nonZero的之前的元素已经是数组中全部不为0的数了，只需要把(nonZero, 数组长度)的值赋值为0即可

## 代码

```
public void moveZeroes(int[] nums) {
    if (nums == null || nums.length == 0){
        return;
    }
    int nonZero = 0;

    for (int index = 0; index < nums.length; index++) {
        if (nums[index] != 0) {
            nums[nonZero++] = nums[index];
        }
    }

    while (nonZero < nums.length) {
        nums[nonZero++] = 0;
    }
}
```