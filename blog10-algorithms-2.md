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

基本思路（dp）是以最后一列为基准，一共有四种情况。设`dp[i][j]`表示第 i 列的地板在第 j 种情况下，铺的方法有多少种
相当于以是 i-1 列为基础，（第 i-1 列可能有四种情况）然后每一步是以第 i-1 列的四种情况为基础，铺设新的第 i 列

因为是 2 \* n 的排列，因此四种情况分别为：（参考官方解的图理解）

- 第 i - 1 列的两行都没有瓷砖，j = 0
- 第 i - 1 列第一行有瓷砖，j = 1
- 第 i - 1 列第二行有瓷砖，j = 2
- 第 i - 1 列的两行都有瓷砖，j = 3

考虑第 i 列的铺设，其实就是从 dp[i-1][j]中推导出新的四个状态。还是按照上面的四个状态顺序：

> 注意：每种情况的关键点其实是铺完之后的形状，而并非铺的方法。因此里边会有很多情况是铺的方法不同，但最后得到的形状是相同的，这样只用算一次就好了

- 第 i 列的两行都没有瓷砖，那么相当于在上面第四种情况下什么都不做。即第 i-1 列是铺满的，第 i 列什么都不做就可以。
  - 这里还有一种方法是从 j = 0 的情况下再插一个竖着的瓷砖，也可以得到这种情况。但是有一个原则，**就是（如果要铺的话）铺的瓷砖应该至少有一格在第 i 列中**。即至少需要让第 i 列发生状态变化，不然 dp[i][0]从 dp[i-1][0]推出的话，相当于第 i-1 列始终铺不满了。（这块确实比较迷，可能得先记下）
- 第 i 列的第一行有瓷砖，画个图可知第 i-1 列的第一行肯定不能有瓷砖，不然第一行就放不下了，因此只能从第二行有瓷砖或两行都没瓷砖的情况下推导，即 j = 2 或 j = 0
- 同上，不过这里是第二行不能有瓷砖，即 j = 1 或 j = 0
- 第 i 列的两行都有瓷砖，相当于铺满了第 i 列和第 i-1 列。显然上面四种情况其实都可以得到铺满的结果，因此 j 的四个取值都可以：
  - j = 0，横着铺两个瓷砖就可以填满。注意这里其实竖着铺两个瓷砖的情况也可以，但是下面的 j = 3 也是两个竖着的，只用算一个就可以
  - j = 1 和 j = 2：比较抽象，参考官解的图片
  - j = 3，第 i-1 行已满，那么补一个竖着的在第 i 列即可。

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
    dp[i][3] =
      (dp[i - 1][0] + dp[i - 1][1] + dp[i - 1][2] + dp[i - 1][3]) % mod;
  }
  return dp[n][3];
};
```

# 贪心

## 分发糖果

https://leetcode.cn/problems/candy/description/

> n 个孩子站成一排。给你一个整数数组 ratings 表示每个孩子的评分。
> 
> 你需要按照以下要求，给这些孩子分发糖果：
> 
> 每个孩子至少分配到 1 个糖果。
> 相邻两个孩子评分更高的孩子会获得更多的糖果。
> 请你给每个孩子分发糖果，计算并返回需要准备的 最少糖果数目 。
> 
> 示例 1：
> 
> 输入：ratings = [1,0,2]
> 输出：5
> 解释：你可以分别给第一个、第二个、第三个孩子分发 2、1、2 颗糖果。

这道题虽然从分类上来说确实是贪心问题，但是实际上并没有什么固定的套路。
基本思路是这样：默认每个孩子都是1颗糖果。当某个孩子比他左边或右边分数高的时候，就应该多给这个孩子一颗糖。并且：

- 如果同时比左边和右边的高，那就应该是max(左边孩子的糖果数量, 右边孩子的糖果数量) + 1
- 左边和右边孩子的数量由更前面或更后面一个推出，所以应该是一种递推

因为对于每个孩子来说，可以看做左边孩子和右边孩子对它有不同的“贡献”。左边和右边的贡献其实是不冲突的
比如ratings = [1,3,2]，对于中间的孩子来说，左边和右边对它的贡献都是1，也就是说中间孩子需要比左边和右边孩子的糖果数量至少多一个。
注意这个思想，**当左边和右边都对某个结果的计算产生影响时，我们可以考虑遍历两次，从左向右和从右向左**，分别计算每个孩子的左边孩子对它的贡献（只满足左边情况）和右边孩子的贡献，最后取两者在同一个位置上的最大值+1即可。

解法如下：

```js
var candy = function (ratings) {
    const lefts = new Array(ratings.length).fill(1)
    const rights = new Array(ratings.length).fill(1)
    for (let i = 0; i < ratings.length; i++) {
        if (i > 0 && ratings[i] > ratings[i - 1]) {
            lefts[i] = lefts[i - 1] + 1
        }
    }
    for (let i = ratings.length - 1; i >= 0; i--) {
        if (i < ratings.length - 1 && ratings[i] > ratings[i + 1]) {
            rights[i] = rights[i + 1] + 1
        }
    }
    let res = 0
    for (let i = 0; i < lefts.length; i++) {
        res += Math.max(lefts[i], rights[i])
    }
    return res
};
```

代码和思路其实不难，关键在于这种考虑两个方向遍历的思想。
如果是一开始的思路，通过单向遍历多次的方式计算，就可能比较难想。
