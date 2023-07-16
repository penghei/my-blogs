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

# 二分查找

## 模板

基础二分：

```js
function binarySearch(nums, target) {
  let left = 0;
  let right = nums.length - 1;
  while (left <= right) {
    const mid = left + Math.floor((right - left) / 2);
    if (nums[mid] === target) {
      return mid;
    } else if (nums[mid] > target) {
      right = mid - 1;
    } else if (nums[mid] < target) {
      left = mid + 1;
    }
  }
}
```

查找大于 target 的第一个值，target 可能不在数组中
如果 target 在数组中，那么实际是求 target 的左边界

```js
function binarySearch(nums, target) {
  let left = 0;
  let right = nums.length;
  while (left < right) {
    const mid = (left + right) >> 1;
    if (target > nums[mid]) left = mid + 1;
    // 如果这里是大于等于，那么当target存在时，依然可以求得大于target的第一个值
    // 两种情况应用不同。如果是要找到target的正确位置，那么不需要等于；如果是一定要确定第一个大于target的数，就需要大于等于
    // if(target >= nums[mid]) left = mid + 1
    else right = mid;
  }
  return left;
}
```

同理，上面的写法可以换成求小于 target 的第一个值，也是 target 存在时的右边界

```js
function binarySearch(nums, target) {
  let left = 0;
  let right = nums.length;
  while (left < right) {
    const mid = (left + right) >> 1;
    if (target < nums[mid]) right = mid;
    // 同理  if(target <= nums[mid]) right = mid
    else left = mid + 1;
  }
  return left;
}
```

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

## 单串问题-递增子序列

递增子序列系列问题的核心类型就是最长递增子序列这道题，也叫 LIS。
这类问题一般比较明显，都是在一个数组中选择子序列，保证子序列尽可能长。
LIS 问题有一个很经典的二分优化，需要掌握。
当然更多时候，需要将类似的题目转化成合理的 LIS 问题，在求某些最长序列，且序列有单调性要求时一般可能可以转换。

### 最长递增子序列

https://leetcode.cn/problems/longest-increasing-subsequence/

最长递增子序列的基本解法就不再赘述，这里讲一下另一种通过二分查找降低复杂度的方法。
在原来的解法中，对于每个 dp[i]，都要从头开始挨个查找一遍能构成上升子序列的最后一个数。这个过程由于从 0 到 i 之间的数没有顺序，因此每次遍历都是 O(n)的消耗。
优化的方向是，把前 i 项设法改成顺序的，然后每次通过二分查找在前面查找。还有一点是，因为我们需要的是最长的递增序列，那么递增的速度应该越慢越好，即序列中的每一项应该尽可能选择递增最小的。

设数组 tails，每个元素 tails[k]的值代表 长度为 k+1 的子序列尾部元素的值。并且在更新的过程中，tails[k]应该是保证最小的。

比如序列`[1,5,3,7,8]`，tails 的值应该是`[1,3,7,8]`。第二项，即长度为 2 的子序列可以是`[1,5]`或`[1,3]`，但是显然`3 < 5`，应该选择更小的 3 作为长度为 2 的递增子序列的最后一个元素的值，而不是 5。

这个算法的过程就是，逐个遍历每一项，然后在已有的序列中查找当前项应该放在的位置。由于这个“已有的序列”一定是单增的（因为从第二个元素开始都是按顺序放的），因此可以使用二分查找确定每一项应该存放的位置。

具体流程为：

1. 初始化 tails 数组。tails[k]的值代表 长度为 k+1 的子序列尾部元素的值，那么 tails 的最后一项就表示当前最长递增子序列的最后一个元素，tails 的最终 length 就是最长递增子序列的长度。
2. 遍历 nums 数组。对于每一项 num，通过二分查找，找到 num 在 next 数组中应该放置的位置。这时候会有两种情况：

- next 数组中没有比 num 更大的数，这时表示当前每个最长递增子序列的最后一个元素都比 num 小，那么 num 就可以接在他们后边。即`next.push(num)`，表示新的更长的子序列产生了。
  比如上面的 `[1,5,3,7,8]`，当遍历到数字 7 时，next 数组为`[1,3]`，没有比 7 更大的，那么就把 7 放在最后，表示长度为 3 的递增子序列的最后一个元素为 7
- 如果 next 数组中有比 num 更大的数，相当于 num 应该插入到 next 数组中间；这时应该找到插入的位置，然后把第一个大于 num 的数替换成 num。
  比如当遍历到数字 3 时，next 数组为`[1,5]`，这时发现第一个大于 3 的数是 5，那么就应该把 5 替换为 3，因为 3 和 5 都能构成长度为 2 的递增子序列，但是 3 更小，更有可能构成更长的。

3. 最后 next'的长度就是结果。

至于二分查找的写法，其实就是在一个数组中查找某个值的位置，且数组中可能不包含这个值。

```js
const binarySearch = (target) => {
  let left = 0;
  let right = tails.length;
  while (left < right) {
    const mid = Math.floor((left + right) / 2);
    if (tails[mid] < target) left = mid + 1;
    else right = mid;
  }
  return left;
};
```

全部代码：

```js
var lengthOfLIS = function (nums) {
  const tails = new Array(nums.length).fill(0);
  let res = 0; // res就是tails的最后一项的index
  const binarySearch = (target) => {
    let left = 0;
    let right = res;
    while (left < right) {
      const mid = Math.floor((left + right) / 2);
      if (tails[mid] < target) left = mid + 1;
      else right = mid;
    }
    return left;
  };

  for (let i = 0; i < nums.length; i++) {
    const found = binarySearch(nums[i]);
    if (found >= res) {
      tails[res++] = nums[i];
    } else {
      tails[found] = nums[i];
    }
  }
  return res;
};
```

这个方法的时间复杂度比原先的方法小了很多，因此如果基本方法在某些题目中超时，就可以用该方法尝试替换。

### 俄罗斯套娃信封问题

https://leetcode.cn/problems/russian-doll-envelopes/

> 给你一个二维整数数组 envelopes ，其中 envelopes[i] = [wi, hi] ，表示第 i 个信封的宽度和高度。
> 当另一个信封的宽度和高度都比这个信封大的时候，这个信封就可以放进另一个信封里，如同俄罗斯套娃一样。
> 请计算 最多能有多少个 信封能组成一组“俄罗斯套娃”信封（即可以把一个信封放到另一个信封里面）。
> 注意：不允许旋转信封。
> 示例 1：
> 输入：envelopes = [[5,4],[6,4],[6,7],[2,3]]
> 输出：3
> 解释：最多信封的个数为 3, 组合为: [2,3] => [5,4] => [6,7]。

这道题看起来和最长递增子序列差不多，直接的思路就是先按第一项从小到大，再按第二项从小到大排，然后按照递增子序列的方法算（二分查找法）。
具体来说，因为套娃的信封一定要求是 w 单增的，因此我们按照 w 排序之后，只用找 h 的最长递增子序列即可。
比如示例的数组，按照 w 排序之后为`[[2,3],[5,4],[6,4],[6,7]]`，然后忽视 w，只对 h 进行最长递增子序列的计算，相当于计算`[3,4,4,7]`的最长子序列。

但有一个区别是，其实排序的时候，h 应该从大到小排。为啥要这样呢？因为宽高需要都比上一个信封大（不能相等）
考虑输入 `[[1，3]，[1，4]，[1，5]，[2，3]]`，如果直接对 h 进行 LIS 算法，将会得到 `[3，4，5]`，显然这不是想要的答案，因为 w 相同的信封是不能够套娃的。但我们需要排除 w，因此必须想办法去掉 w 相同时的影响
如果 h 按照从大到小排序，那么多个 w 相同时只会取第一个了（最大的 h）比如输入为 `[[1，5]，[1，4]，[1，2]，[2，3]]`。则提取 h 为 `[5，4，2，3]`。我们对 h 进行 LIS 算法将得到 `[2，3]`，就是正确答案了。这时 w 为 1 的三项一定不会连续取到。

总结下来关键点有两个：

1. 按照 w 排序，然后只取 h，计算 h 的 LIS
2. 相同的 w，h 应该从大到小排序，防止 w 相同的多个信封形成 LIS（w 相同是不能套的）

代码如下，可以看到其实和 LIS 差不多的代码。

```js
var maxEnvelopes = function (envelopes) {
  if (envelopes.length === 1) return 1;
  envelopes.sort((a, b) => {
    if (a[0] !== b[0]) return a[0] - b[0];
    else return b[1] - a[1]; // h要降序排列
  });
  const tails = new Array(envelopes.length).fill(0);
  let res = 0;
  const binarySearch = (target) => {
    let left = 0;
    let right = res;
    while (left < right) {
      const mid = Math.floor((left + right) / 2);
      if (tails[mid] < target) left = mid + 1;
      else right = mid;
    }
    return left;
  };

  for (let i = 0; i < envelopes.length; i++) {
    const found = binarySearch(envelopes[i][1]); // 提取h求LIS
    if (found >= res) {
      tails[res++] = envelopes[i][1];
    } else {
      tails[found] = envelopes[i][1];
    }
  }
  return res;
};
```

## 单串问题-子数组和

### 环形子数组的最大和

https://leetcode.cn/problems/maximum-sum-circular-subarray

题目很简单，其实就是在最大子数组和的基础上，现在可以首尾相连判断了，但是子数组中每个位置上的元素只能用一次，比如你不能在子数组中包含两个 nums[0]

这个题比较麻烦的地方在于，在迭代过程中我们不能得知这个子数组从哪里起始的，只能得到当前的和。这样有可能导致元素重复。

解决方法有两个方向，一个是不用动态规划，比如用前缀和；前缀和可以保证子数组内部不会有重复位置的元素，但是复杂度比较高。
如果要用动态规划，那么解法就是这样：https://leetcode.cn/problems/maximum-sum-circular-subarray/solutions/1152143/wo-hua-yi-bian-jiu-kan-dong-de-ti-jie-ni-892u/

参考链接内的图解，一眼就能看出来。方法就是求最大子数组和最小子数组和，最终结果是`max(最大子数组和, 总和 - 最小子数组和)`

注意：最小子数组不能占全了整个数组，即至少要留一个位置表示最大子数组。也就是说，最小子数组的和不能等于总和，否则就直接取最大子数组和即可。

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
var maxSubarraySumCircular = function (nums) {
  if (nums.length === 1) return nums[0];
  const dp = new Array(nums.length).fill(0);
  const dp2 = new Array(nums.length).fill(0);
  dp[0] = nums[0];
  dp2[0] = nums[0];
  let max = -Infinity;
  let min = Infinity;
  for (let i = 1; i < nums.length; i++) {
    dp[i] = Math.max(nums[i], dp[i - 1] + nums[i]);
    max = Math.max(max, dp[i]);
    dp2[i] = Math.min(nums[i], dp2[i - 1] + nums[i]);
    min = Math.min(min, dp2[i]);
  }
  const sum = nums.reduce((a, b) => a + b);
  if (sum === min) return max;
  return Math.max(max, sum - min);
};
```

### 最大子矩阵（和）

https://leetcode.cn/problems/max-submatrix-lcci

> 给定一个正整数、负整数和 0 组成的 N × M 矩阵，编写代码找出元素总和最大的子矩阵。
> 返回一个数组 [r1, c1, r2, c2]，其中 r1, c1 分别代表子矩阵左上角的行号和列号，r2, c2 分别代表右下角的行号和列号。若有多个满足条件的子矩阵，返回任意一个均可。
> 注意：本题相对书上原题稍作改动
> 示例：
> 输入：
> [
> [-1,0],
> [0,-1]
> ]
> 输出：[0,1,0,1]
> 解释：输入中标粗的元素即为输出所表示的矩阵

这道题的核心思路是：**降维**
即，把二维子数组按行计算，压缩成一维之后再对一维的数组求最大子数组和即可。

举个例子，我们把从第 i 行到第 j 行的所有数按列加起来，压缩成一个一维数组，那么数组的每一项 k 都表示这一列的从第 i 行到第 j 行的和。

然后我们对数组进行最大子数组的计算，并记录范围。比如计算得到这个数组的最大子数组和的范围为[a,b]，那么矩阵的坐标就能得到了（因为 i 和 j 就相当于这个矩阵的纵坐标），即左上角为[a,i]，右下角为[b,j]

代码如下：

```js
/**
 * @param {number[][]} matrix
 * @return {number[]}
 */
var getMaxMatrix = function (matrix) {
  // 求一维数组的最大子数组和，以及范围
  const getMaxRangeAndValue = (nums) => {
    const dp = new Array(nums.length).fill(0);
    const range = [0, 0];
    let resRange = [0, 0];
    let max = nums[0];
    dp[0] = nums[0];
    for (let i = 1; i < nums.length; i++) {
      if (dp[i - 1] + nums[i] > nums[i]) {
        // 没有发生替换，当前最大子数组和向后一个
        dp[i] = dp[i - 1] + nums[i];
        range[1] = i;
      } else {
        // 发生替换，重新记录子数组起始
        dp[i] = nums[i];
        range[0] = i;
        range[1] = i;
      }
      if (dp[i] > max) {
        // 最大值更新，同时记录下来范围
        max = dp[i];
        resRange = [...range];
      }
    }
    return [resRange, max];
  };
  const [n, m] = [matrix.length, matrix[0].length];
  let maxRes = -Infinity;
  let result = [];
  for (let i = 0; i < n; i++) {
    const sum = [...matrix[i]];
    for (let j = i; j < n; j++) {
      // j从i开始从上向下走
      for (let k = 0; k < m; k++) {
        // sum数组就是这个一维数组，每次累加第j行的数
        if (j !== i) sum[k] += matrix[j][k];
      }
      const [[c1, c2], maxSum] = getMaxRangeAndValue(sum);
      if (maxSum > maxRes) {
        result = [i, c1, j, c2];
        maxRes = maxSum;
      }
    }
  }
  return result;
};
```

类似的题目还有一道：https://leetcode.cn/problems/max-sum-of-rectangle-no-larger-than-k/
不过这道题只需要用到压缩就可以了，并不需要动态规划。



## 单串问题-打家劫舍



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
基本思路是这样：默认每个孩子都是 1 颗糖果。当某个孩子比他左边或右边分数高的时候，就应该多给这个孩子一颗糖。并且：

- 如果同时比左边和右边的高，那就应该是 max(左边孩子的糖果数量, 右边孩子的糖果数量) + 1
- 左边和右边孩子的数量由更前面或更后面一个推出，所以应该是一种递推

因为对于每个孩子来说，可以看做左边孩子和右边孩子对它有不同的“贡献”。左边和右边的贡献其实是不冲突的
比如 ratings = [1,3,2]，对于中间的孩子来说，左边和右边对它的贡献都是 1，也就是说中间孩子需要比左边和右边孩子的糖果数量至少多一个。
注意这个思想，**当左边和右边都对某个结果的计算产生影响时，我们可以考虑遍历两次，从左向右和从右向左**，分别计算每个孩子的左边孩子对它的贡献（只满足左边情况）和右边孩子的贡献，最后取两者在同一个位置上的最大值+1 即可。

解法如下：

```js
var candy = function (ratings) {
  const lefts = new Array(ratings.length).fill(1);
  const rights = new Array(ratings.length).fill(1);
  for (let i = 0; i < ratings.length; i++) {
    if (i > 0 && ratings[i] > ratings[i - 1]) {
      lefts[i] = lefts[i - 1] + 1;
    }
  }
  for (let i = ratings.length - 1; i >= 0; i--) {
    if (i < ratings.length - 1 && ratings[i] > ratings[i + 1]) {
      rights[i] = rights[i + 1] + 1;
    }
  }
  let res = 0;
  for (let i = 0; i < lefts.length; i++) {
    res += Math.max(lefts[i], rights[i]);
  }
  return res;
};
```

代码和思路其实不难，关键在于这种考虑两个方向遍历的思想。
如果是一开始的思路，通过单向遍历多次的方式计算，就可能比较难想。

# 位运算

## Brian Kernighan 算法

Brian Kernighan 算法其实是一个数 n，`n & (n - 1)`的运算结果可以看作是把 n 的二进制位的从右向左数第一个 1 给去掉的结果。
比如数字`1100`，使用该算法之后为`1100 & 1011 = 1000`，相当于去掉了从右向左第一个 1

### 汉明距离

https://leetcode.cn/problems/hamming-distance/

> 两个整数之间的 汉明距离 指的是这两个数字对应二进制位不同的位置的数目。
> 给你两个整数 x 和 y，计算并返回它们之间的汉明距离。
> 示例 1：
> 输入：x = 1, y = 4
> 输出：2
> 解释：
> 1 (0 0 0 1)
> 4 (0 1 0 0)
> ↑ ↑
> 上面的箭头指出了对应二进制位不同的位置。

这道题有两个技巧，一个是利用异或得到两个数的不同位置，还有一个是通过 Brian Kernighan 算法快速进行 1 的数量的统计

具体来说，`x ^ y`的每一位 1 都刚好是两个数的每一个不同的位，因此就是统计`x ^ y`的 1 的个数。
可以用 Brian Kernighan 算法每次消掉最右边的一个 1，不断循环直到数变为 0，这样比逐个按位遍历要快一些。

```js
var hammingDistance = function (x, y) {
  let n = x ^ y;
  let cnt = 0;
  while (n > 0) {
    n &= n - 1;
    cnt++;
  }
  return cnt;
};
```

### 数字范围按位与

https://leetcode.cn/problems/bitwise-and-of-numbers-range/

> 给你两个整数 left 和 right ，表示区间 [left, right] ，返回此区间内所有数字 按位与 的结果（包含 left 、right 端点）。
> 示例 1：
>
> 输入：left = 5, right = 7
> 输出：4

这道题肯定不能使用迭代的方式暴力。核心是与运算的特点，如果与运算某一次变为了 0，那么之后这一位就一定全是 0。也就是说这些数字如果某一个数的某一位是 0，那么最终结果的这一位也一定是 0.

然后推导一下，可以得出对所有数字执行按位与运算的结果是所有对应二进制字符串的公共前缀再用零补上后面的剩余位。具体可以参考https://leetcode.cn/leetbook/read/bit-manipulation-and-math/onmvcj/，大概意思就是从前向后取相同的前缀部分，后面不同的位置一定都是0（有1有0肯定最终都是0）

因此解法就是，计算 left 和 right 的公共前缀。计算的方法是通过 Brian Kernighan 算法，逐个清除 right 右边的 1，直到 right 比 left 小为止。

```js
var rangeBitwiseAnd = function (left, right) {
  while (right > left) {
    right = right & (right - 1);
  }
  return right;
};
```

当然还有一种比较直观的方式，就是把 right 和 left 同时右移，直到 left===right 就说明到了公共前缀，最后再把 left 或 right 右移回去即可。

## 比特位计数

https://leetcode.cn/problems/counting-bits/

> 给你一个整数 n ，对于 0 <= i <= n 中的每个 i ，计算其二进制表示中 1 的个数 ，返回一个长度为 n + 1 的数组 ans 作为答案。
> 示例 1：
> 输入：n = 2
> 输出：[0,1,1]
> 解释：
> 0 --> 0
> 1 --> 1
> 2 --> 10

这道题的非递推做法很简单，遍历每一个数求出 1 的个数即可。

递推做法，对于每个数有这样的性质：

- 如果当前数 n 是偶数，那么 n 的最后一位一定是 0，因此 n 中 1 的个数和 n / 2 的相同（相当于把 n 的最后一个 0 去掉，1 的个数不变）
- 如果当前 n 是奇数，那么 n 的前一项的最后一位一定是 0，因此 n 中 1 的个数就是前一项的 1 的个数 + 1（相当于填上了最后一个 0）

```js
var countBits = function (n) {
  const res = [0];
  for (let i = 1; i < n; i++) {
    if (i % 2) res[i] = res[i - 1] + 1;
    else res[i] = res[i / 2];
  }
  return res;
};
```

# 数学题

## 几何题

### 矩形重叠和面积

https://leetcode.cn/problems/rectangle-overlap/
https://leetcode.cn/problems/rectangle-area/

这两个题都是求矩形的重叠的。
关于矩形是否重叠的判断，其实主要如下：

我们设:
第一个矩形的

- 左下角坐标是(A,B)
- 右上角坐标是 (C,D)

第二个矩形的

- 左下角坐标是 (E,F)
- 右上角坐标是 (G,H)。

那么重叠部分的判断就是

`min(C,G) < max(A,E) && min(G,H) < max(B,F)`

同理重叠部分矩形面积计算也是

`min(C,G) * max(A,E) + min(G,H) * max(B,F)`

具体代码就不放了，了解这个方法即可。

### 有效的正方形

https://leetcode.cn/problems/valid-square

题目大概是，给你四个点的坐标，判断是否能构成正方形。

思路参考:https://leetcode.cn/leetbook/read/bit-manipulation-and-math/ovhgkc/

这道题的解法是，计算四个点两两之间的距离（一共是 3 \* 4 / 2 = 6 对），然后依据正方形的特性判断这些距离之间的关系。
比如把这 6 个距离排序，那么前 4 个应该是相等的，并且后两个应该和前四个有勾股定理的关系。

这个思路也可沿用于长方形、特殊三角形。因为四个点不好判断具体位置，因此求距离一定是最不会漏掉的。

代码如下：

```js
var validSquare = function (p1, p2, p3, p4) {
  const getLen = (x1, x2, y1, y2) => (x1 - x2) ** 2 + (y1 - y2) ** 2;
  const lens = [
    getLen(p1[0], p2[0], p1[1], p2[1]),
    getLen(p1[0], p3[0], p1[1], p3[1]),
    getLen(p1[0], p4[0], p1[1], p4[1]),
    getLen(p4[0], p2[0], p4[1], p2[1]),
    getLen(p4[0], p3[0], p4[1], p3[1]),
    getLen(p3[0], p2[0], p3[1], p2[1]),
  ];
  lens.sort((a, b) => a - b);
  return (
    lens[4] === lens[5] &&
    lens[0] + lens[1] === lens[4] &&
    lens[0] > 0 &&
    lens[4] > 0
  );
};
```

# 关键题目及概括

这里放一些比较重要的题的链接和简单类型，某些题目可能有些比较有代表性的解法，或者刚好是不知道的类型。但是都写题解就太长了，这里简要总结下。

- 原地哈希：将数组作为哈希表，通常是索引和元素的值有函数关系

缺失的第一个正数：https://leetcode.cn/problems/first-missing-positive
找到所有数组中消失的数字：https://leetcode.cn/problems/find-all-numbers-disappeared-in-an-array/
数组中重复的数据：https://leetcode.cn/problems/find-all-duplicates-in-an-array/

- 多源bfs：即从多个起点开始的bfs

腐烂的橘子：https://leetcode.cn/problems/rotting-oranges/

- 类接雨水（单调栈）：双向单调栈

https://leetcode.cn/problems/largest-rectangle-in-histogram/

- 堆，双堆：同时维护最大和最小堆，两个堆可能会相互交换

摆动排序：https://leetcode.cn/problems/wiggle-sort/
中位数：https://leetcode.cn/problems/find-median-from-data-stream/

- 鸡蛋问题（分治）

https://leetcode.cn/problems/egg-drop-with-2-eggs-and-n-floors/
https://leetcode.cn/problems/super-egg-drop/

- 