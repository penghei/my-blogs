---
title: 算法学习总结-2
date: 2022-04-05 16:01:22
tags: 日常学习
categories: 数据结构与算法
cover:
sticky: 4
---

# 排序算法

## 逆序对的个数（归并排序）

https://leetcode.cn/problems/shu-zu-zhong-de-ni-xu-dui-lcof/

> 在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组，求出这个数组中的逆序对的总数。
> 示例 1:
> 输入: [7,5,6,4]
> 输出: 5

这道题的暴力解法会超时，就不多说了。不超时的解法主要有两个：

1. 基于归并排序的方法，在归并排序的基础上增加对逆序对的计算
2. 树状数组

先说归并。基本题解可以参考：https://leetcode.cn/problems/shu-zu-zhong-de-ni-xu-dui-lcof/solution/bao-li-jie-fa-fen-zhi-si-xiang-shu-zhuang-shu-zu-b/
这里说一下要注意的地方。

为什么数组都排序了，还能正常求出逆序对？
以`[7,5,6,4]`为例，我们把数组分为`[7,5] [6,4]`两个，排完序后为`[5,7] [4,6]`；虽然两个数组分别的顺序改变了，但两个数组之间的逆序对关系不会改变，逆序对的数量也不会改变。
而且两个数组本身内部的逆序对，则在递归的上一层已经计算完了。在代码中体现为 leftPairs 和 rightPairs 是从递归中得出的，而 crossPairs 的计算不受子数组的排序影响。因此最后的效果是，随着计算的完成，归并排序也相应完成了。

代码：

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
var reversePairs = function (nums) {
  const combineSort = (nums) => {
    if (nums.length <= 1) return [nums, 0];
    const mid = Math.floor(nums.length / 2);
    const [nums1, leftPairs] = combineSort(nums.slice(0, mid));
    const [nums2, rightPairs] = combineSort(nums.slice(mid));
    const [numsCombined, crossPairs] = combineAndCompute(nums1, nums2);
    return [numsCombined, leftPairs + rightPairs + crossPairs];
  };

  const combineAndCompute = (nums1, nums2) => {
    const newArr = [];
    let i = 0;
    let j = 0;
    let val = 0;
    while (i < nums1.length && j < nums2.length) {
      if (nums1[i] <= nums2[j]) {
        newArr.push(nums1[i++]);
      } else {
        newArr.push(nums2[j]);
        // 只在这里增加一个计算
        val += nums1.length - i;
        j++;
      }
    }
    if (i === nums1.length && j < nums2.length) {
      newArr.push(...nums2.slice(j));
    } else if (i < nums1.length && j === nums2.length) {
      newArr.push(...nums1.slice(i));
    }
    return [newArr, val];
  };
  return combineSort(nums)[1];
};
```

类似的题目还有几道，都可以参考：https://labuladong.github.io/algo/di-yi-zhan-da78c/shou-ba-sh-66994/gui-bing-p-1387f/
这些题目的基本方法都是一样的，即都是在基本归并的基础上加一些操作。这个归并的方法掌握好即可。


