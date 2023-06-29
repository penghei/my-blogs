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



# 动态规划

## 多米诺和托米诺平铺（类股票问题）

https://leetcode.cn/problems/domino-and-tromino-tiling

这道题比较抽象，可能需要辅助很多图才能理解，因此主要参考官方题解。

这里主要说下基本思路：

基本思路（dp）是以最后一列为基准，一共有四种情况。设`dp[i][j]`表示第i列的地板在第j种情况下，铺的方法有多少种
相当于以是i-1列为基础，（第i-1列可能有四种情况）然后每一步是以第i-1列的四种情况为基础，铺设新的第i列

因为是2 * n的排列，因此四种情况分别为：（参考官方解的图理解）

- 第i - 1列的两行都没有瓷砖，j = 0
- 第i - 1列第一行有瓷砖，j = 1
- 第i - 1列第二行有瓷砖，j = 2
- 第i - 1列的两行都有瓷砖，j = 3

考虑第i列的铺设，其实就是从dp[i-1][j]中推导出新的四个状态。还是按照上面的四个状态顺序：

> 注意：每种情况的关键点其实是铺完之后的形状，而并非铺的方法。因此里边会有很多情况是铺的方法不同，但最后得到的形状是相同的，这样只用算一次就好了

- 第i列的两行都没有瓷砖，那么相当于在上面第四种情况下什么都不做。即第i-1列是铺满的，第i列什么都不做就可以。
  - 这里还有一种方法是从j = 0的情况下再插一个竖着的瓷砖，也可以得到这种情况。但是有一个原则，**就是（如果要铺的话）铺的瓷砖应该至少有一格在第i列中**。即至少需要让第i列发生状态变化，不然dp[i][0]从dp[i-1][0]推出的话，相当于第i-1列始终铺不满了。（这块确实比较迷，可能得先记下）
- 第i列的第一行有瓷砖，画个图可知第i-1列的第一行肯定不能有瓷砖，不然第一行就放不下了，因此只能从第二行有瓷砖或两行都没瓷砖的情况下推导，即j = 2或j = 0
- 同上，不过这里是第二行不能有瓷砖，即j = 1或j = 0
- 第i列的两行都有瓷砖，相当于铺满了第i列和第i-1列。显然上面四种情况其实都可以得到铺满的结果，因此j的四个取值都可以：
  - j = 0，横着铺两个瓷砖就可以填满。注意这里其实竖着铺两个瓷砖的情况也可以，但是下面的j = 3也是两个竖着的，只用算一个就可以
  - j = 1和j = 2：比较抽象，参考官解的图片
  - j = 3，第i-1行已满，那么补一个竖着的在第i列即可。

最后的代码如下：
```js
/**
 * @param {number} n
 * @return {number}
 */
var numTilings = function (n) {
    const mod = 1e9 + 7;
    // 参考题解，dp[i][j]表示第i列的地板在第j种情况下，铺的方法有多少种
    // 注意这里相当于以是i-1列为基础，第i-1列可能有四种情况，然后每一步是以第i-1列的四种情况为基础，铺设新的第i列
    const dp = new Array(n + 1).fill(0).map(() => new Array(4).fill(0));
    dp[0][3] = 1;
    for (let i = 1; i <= n; i++) {
        dp[i][0] = dp[i - 1][3];
        dp[i][1] = (dp[i - 1][0] + dp[i - 1][2]) % mod;
        dp[i][2] = (dp[i - 1][0] + dp[i - 1][1]) % mod;
        // dp[i - 1][0]是第i-1和第i列都为空的情况，理论上应该有两种，即两个横或两个竖
        // 但是其实不需要两个竖，因为dp[i - 1][3]就已经表示一个竖，再加一个竖就可以了，这种情况已经算过了
        dp[i][3] = (dp[i - 1][0] + dp[i - 1][1] + dp[i - 1][2] + dp[i - 1][3]) % mod;
    }
    return dp[n][3];
};
```