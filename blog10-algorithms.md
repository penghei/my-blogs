---
title: 算法学习总结
date: 2022-04-05 16:01:22
tags: 日常学习
categories: 数据结构与算法
cover:
sticky: 4
---

# 算法复杂度

## 时间复杂度

**时间复杂度是一个函数，它定性描述该算法的运行时间**。

我们在软件开发中，时间复杂度就是用来方便开发者估算出程序运行的答题时间。

那么该如何估计程序运行时间呢，通常会估算算法的操作单元数量来代表程序消耗的时间，这里默认 CPU 的每个单元运行消耗的时间都是相同的。

假设算法的问题规模为 n，那么操作单元数量便用函数 f(n)来表示，随着数据规模 n 的增大，算法执行时间的增长率和 f(n)的增长率相同，这称作为算法的渐近时间复杂度，简称时间复杂度，记为 O(f(n))。大 O 就是数据量级突破一个点且数据量级非常大的情况下所表现出的时间复杂度，这个数据量也就是常数项系数已经不起决定性作用的数据量。所以我们说的时间复杂度都是省略常数项系数的，是因为一般情况下都是默认数据规模足够的大，基于这样的事实，给出的算法时间复杂的的一个排行如下所示：

> O(1)常数阶 < O(logn)对数阶 < O(n)线性阶 < O(n^2)平方阶 < O(n^3)立方阶 < O(2^n)指数阶

通常来说，O(n)表示的都是算法的最大复杂度；但是对于一些更常见的情况，往往表示的是最一般的情况。比如快排的复杂度最大是 $O(n^2)$ ，但我们通常都说是 $O(nlogn)$，因为后者是更普遍的情况。

## 迭代时间复杂度

时间复杂度最基本的形式就是迭代形式的。
比如一个最简单的迭代：

```js
for (let i = 0; i < n; i++) {
  console.log(i);
}
```

因为这个迭代一定会执行 n 次，所以时间复杂度就是 O(n)

如果 i 每次不是加 1 这样的线性增长，而是其他变化，时间复杂度就会有所不同：

```js
for (let i = 0; i < n; i += 2) {
  console.log(i);
}

for (let i = 0; i < n; i *= 2) {
  console.log(i);
}
```

第一个迭代实际上执行了`n/2`次，但是算法复杂度忽略参数，我们还是看作 O(n)
第二个迭代很明显每次执行都会 i\*2，这时就是一个对数复杂度，即 $O(log_{2}n)$ （通常忽略底数）

对于比较复杂的迭代问题，比如双指针、滑动窗口，虽然看起来有很多复杂的步骤，但是只用关注核心的指针移动就可以。
无论多复杂的逻辑，如果遍历过程中指针没有回退而是一直向前走，那么复杂度就是简单的 O(N)

## 递归的时间复杂度

递归算法的时间复杂度本质上是要看: **递归的次数 \* 递归函数本身的复杂度**。如果把递归看成是一棵树，那就是节点数\*每个节点的复杂度。
递归其实是另一种迭代，因此很多算法的递归形式其实和迭代形式的复杂度差不多。
比如计算 x 的 n 次方，用递归形式描述为：

```js
function fn1(x, n) {
  if (n === 0) return 1;
  return x * fn1(x, n - 1);
}
```

这个递归一共执行了 n 次（到 0 停止），每一次都是一个乘法操作（O(1)），因此总体时间复杂度就是 O(n)

再比如，递归形式的二分查找：

```js
function binarySearch(nums, target) {
  if (nums.length < 1) return;
  const mid = Math.floor(nums.length / 2);
  if (target === nums[mid]) flag = true;
  if (target > nums[mid]) {
    binarySearch(nums.slice(mid + 1), target);
  } else {
    binarySearch(nums.slice(0, mid - 1), target);
  }
}
```

因为每次实际上只会进入两个递归中的一个，并且 mid 一直在取数组长度 n 的一半，所以时间复杂度也是 O(logn)

---

但是不一定所有每次减半的递归复杂度都是 O(logn)；如果一个递归是几个 O(logn)一起出现，那就有可能回到 O(n)

比如还是上面的乘方题，如果写成这样：

```js
function fn2(x, n) {
  if (n === 0) return 1;
  if (x % 2 === 1) return fn2(x, n / 2) * fn2(x, n / 2) * x;
  return fn2(x, n / 2) * fn2(x, n / 2);
}
```

这个写法实际上就是把 2 的整数倍次方的情况单独列出来了；看起来好像是每次都只取 n/2，但是实际上每次递归都会执行两次同时的递归，所以时间复杂度还是 O(n)
![](https://pic.imgdb.cn/item/6291a6c309475431292a43c7.jpg)

如果提前把这个结果缓存，让递归只执行一次，那么就会是 O(logn)了

```js
function fn3(x, n) {
  if (n === 0) return 1;
  const val = fn2(x, n / 2);
  if (x % 2 === 1) return val * val * x;
  return val * val;
}
```

---

有些递归会展开成二叉树或多叉树的形式，比如回溯相关的题目，以及部分用递归来解的动态规划。
一棵深度（按根节点深度为 1）为 k 的 N 叉树最多可以有 $(N^k - 1)/(K - 1)$ 个节点，时间复杂度为 O(K^N)。（把 1/K - 1 看作常数）

以斐波那契数列为例，最基本的不做任何优化的算法复杂度为 $O(2^n)$ ，就是最大的指数时间复杂度
![](https://pic.imgdb.cn/item/6291bdb90947543129471d30.jpg)
产生这么大复杂度的主要原因是同时进行了两次递归，并且每个递归的次数其实并没有多少减少（还是 n 次）

```js
return fibonacci(i - 1) + fibonacci(i - 2);
```

# 数组

## 两数之和

> 给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

这道题是经典的解法简单、但是可以使用更好的方法优化的问题。
最简单的方式是通过双重循环遍历：

```js
function findTarget(nums, target) {
  for (let a = 0; a < nums.length; a++) {
    for (let b = 0; b < nums.length; b++) {
      if (nums[a] + nums[b] === target) return [a, b];
    }
  }
}
```

双指针也可以，但是要先对数组排序。

但是可以通过“空间换时间，加法转减法”的方法优化这道题目。
思路是求和问题转化为求差问题, 每遍历到一个新数字的时候，都回到 `Map` 里去查询 `targetNum` 与该数的差值是否已经在前面的数字中出现过了。如果出现过显然就是结果；如果没有出现就记录当前数字。

```js
function findTargetByMap(nums, target) {
  const map = new Map();
  map.set(nums[0], 0);
  for (let i = 1; i < nums.length; i++) {
    if (map.has(target - nums[i])) {
      // 如果目标数减去当前数在map中存在, 说明找到了
      return [i, map.get(target - nums[i])];
    } else {
      // 没有找到, 则把该数放到map中
      map.set(nums[i], i);
    }
  }
  return false;
}
```

两数之和还提供了一个重要的思路：既然两个数可以通过哈希表简化，那么三个数、四个数也可以（先计算两个数的和`sum1`并存到 map 中，再计算另外几个数的和并查找`target - sum1 === sum2`），本质上还是两个数的求和。

## 合并有序数组

> 给你两个有序整数数组 nums1 和 nums2，请你将 nums2 合并到 nums1 中，使 nums1 成为一个有序数组。

思路是使用两个指针（i、j）分别指向两个数组中的数字，然后两边同时遍历，比较两个指针所指的数字的大小关系，把较小的一个放入新数组，依次类推；
最终有一个数组会剩下一部分，再把这一部分截下来补在最后即可

> 注意：双指针法用在涉及求和、比大小类的数组题目里时，大前提往往是：该数组必须有序。否则双指针根本无法帮助我们缩小定位的范围，压根没有意义。

```js
function concatArray(arr1, arr2) {
  const newArr = [];
  let i = 0;
  let j = 0;
  while (i < arr1.length && j < arr2.length) {
    newArr.push(arr1[i] < arr2[j] ? arr1[i++] : arr2[j++]);
  }
  return [...newArr, ...(i === arr1.length ? arr2.slice(j) : arr1.slice(i))];
}
```

## 三数求和

> 给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有满足条件且不重复的三元组。

思路：

1. 首先先对数组排序；
2. 然后固定一个元素`k`,然后双指针分别指向其后后面元素的第一个`k+1`和最后一个`length - 1`,按照两元素和的方法移动指针
   - 若三个数相加之和大于 0，说明右侧的数偏大了，右指针左移
   - 若三个数相加之和小于 0，说明左侧的数偏小了，左指针右移

关键在于去重：

- 对于基准数 k 的去重，用一个数组记录重复数即可
- 对于 i 和 j 的去重，当和为 0 时就顺便把和当前 i 和 j 相同的数字都跳过。比如找到了一个组合为[-1,0,1]，那么就把数组中的 0 和 1 都跳过

```js
function triNumberSum(nums) {
  nums = nums.sort((a, b) => a - b);
  const cache = []; // 存储防止重复
  const newArr = [];
  for (let k = 0; k < nums.length - 2; k++) {
    let i = k + 1;
    let j = nums.length - 1;
    if (cache.includes(nums[k])) {
      continue;
    }
    while (i < j) {
      const sum = nums[i] + nums[j] + nums[k];
      if (sum === 0) {
        newArr.push([nums[i], nums[j], nums[k]]);
        cache.push(nums[k]);
        while (i < j && nums[j] === nums[j - 1]) j--;
        while (i < j && nums[i] === nums[i + 1]) i++;
        i++;
        j--;
      } else if (sum > 0) j--;
      else i++;
    }
  }
  return newArr;
}
```

> 左右指针一起从两边往中间位置相互迫近，这样的特殊双指针形态，被称为“对撞指针”。
> **有序**和**数组**这样的关键词出现，就要考虑使用对撞指针

---

从三数求和开始，后序的求和就可以递推得来。
比如四数求和（同一个数组的四数求和），就可以固定一个数 num，然后另外三个数用三数求和的函数计算得到 target-num 即可。有了四数求和的结果，五数、六数直到 n 数都是可行的。

## 有序数组的平方

> 给你一个按 非递减顺序 排序的整数数组 nums，返回 每个数字的平方 组成的新数组，要求也按 非递减顺序 排序。
> 示例 1： 输入：nums = [-4,-1,0,3,10] 输出：[0,1,9,16,100] 解释：平方后，数组变为 [16,1,0,9,100]，排序后，数组变为 [0,1,9,16,100]

这道题的解法不是先平方再排序，而是通过双指针的形式原地 O(1)实现。

思路是，新创建一个数组，然后新数组从后往前存值；接下来循环：
比较左指针和右指针的数的平方值大小；将较大的一个数放入新数组，然后较大数的那边指针移动。

之所以从后向前存值，是因为负数的平方有可能超过正数，如果从前向后没法确定最小值是哪个。

```js
/**
 * @param {number[]} nums
 * @return {number[]}
 */
var sortedSquares = function (nums) {
  let i = 0;
  let j = nums.length - 1;
  let k = 0;
  const res = new Array(nums.length);
  while (k < res.length) {
    if (nums[i] ** 2 < nums[j] ** 2) {
      res[k++] = nums[i] ** 2;
      i++;
    } else {
      res[k++] = nums[j] ** 2;
      j--;
    }
  }
  return res;
};
```

## 移除元素

> 给你一个数组 nums  和一个值 val，你需要**原地**移除**所有**数值等于  val  的元素，并返回移除后数组的新长度。
> 不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并 原地 修改输入数组。
> 元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。

这道题的思路是利用快慢指针；当走到目标元素时，让快指针先走到第一个不是目标元素的位置，然后将快指针所指的值替换到慢指针的位置即可。注意要保存两个指针之间的位置差

还有一点，遍历至少要完成所有元素的一次遍历；所以可以**让慢指针延迟，而不是让快指针先走**的方式；当快指针走到头时，遍历就完成了，此时慢指针恰好就是最后一个。

```js
/**
 * @param {number[]} nums
 * @param {number} val
 * @return {number}
 */
var removeElement = function (nums, val) {
  let k = 0;
  for (let i = 0; i < nums.length; i++) {
    if (nums[i] !== val) {
      nums[k] = nums[i];
      k++;
    }
  }
  return k;
};

/**
 * @param {number[]} nums
 * @param {number} val
 * @return {number}
 */
var removeElement = function (nums, val) {
  let i = 0;
  let j = i + 1;
  // 让j每次覆盖i+1也可以
  for (; j < nums.length; i++, j++) {
    while (nums[j] === val) j++;
    nums[i + 1] = nums[j];
  }
  return i;
};
```

## 原地删除相邻重复元素（重要）

https://leetcode.cn/problems/remove-duplicates-from-sorted-array/

> 给你一个 升序排列 的数组 nums ，请你 原地 删除重复出现的元素，使每个元素 只出现一次 ，返回删除后数组的新长度。元素的 相对顺序 应该保持 一致 。

这道题有一个更通用的题型，即，使每个元素最多出现k次。k可能是1、2甚至更多，每个数字可以少于k次，但不能多于k次。

通用的模板是下面这样：

```js
var removeDuplicates = function (nums, k) {
  let i = 0
  let j = 0
  while(j < nums.length){
      if(i < k || nums[i - k] !== nums[j]){
          nums[i] = nums[j]
          i++
      }
      j++
  }
  return i
};
```

下面解释一下。

首先，这个方法的核心是控制i什么时候移动；之前的方法总是在关注j什么时候移动，但是j移动到最后一个位置时通常i还没到边缘，这样就会导致最后的边界出问题。解决方法就是让j自然增长，而控制i什么时候移动，其他时刻i不动。

```js
if(i < k || nums[i - k] !== nums[j])
```

`i < k`表示前k个元素都不做任何赋值。也就是说前k个值一定是要保留的，不管是重复的k个还是不重复的k个；前面k个位置的i和j同步自增，直到i和j都为k为止。
当`i >= k`之后，每次判断的是i的前面k个元素和j的关系。比如k = 2时，判断i的上上一个元素是否和j相等。

```
举个例子
[1,1,1,2,2,3] k = 2

当i = 2，j = 2
比较nums[0]和nums[2]
发现重复，即nums[i - k] === nums[j]
这是已经有了两个1，那么当前i位置上的数就应该被替换，所以i不动，j继续走到下一个和i当前数不同的位置为止

当j = 3，nums[j]为2，此时nums[i - k] !== nums[j]
替换i所在的数，然后让i向前走
```

最后得到的i就是边界。

如果题目希望删除所有重复的元素，即如果一个元素重复，则一个不留；那么就不能使用这种方法，因为当k=0时每一步`nums[i] === nums[j]`都成立，就没法算了。
有一个思路是参考链表的移除全部相邻重复数字的题，和那道题的链表解法一样，这里做一个参考。如果有更好的方法后面补充：

```js
const removeDuplicate = (nums) => {
  nums.sort((a,b) => a - b)
  // 原地删除某个位置的数
  const deleteNum = (i) => {
    for(;i < nums.length;i++){
      nums[i] = nums[i+1]
    }
  }
  let i = -1
  while(nums[i+1]!=null && nums[i+2] != null ){
    if(nums[i+1] === nums[i+2]){
      let val = nums[i+1]
      while(nums[i+1]  === val) deleteNum(i+1)
    }else{
      i++
    }
  }
  return nums.slice(0,i+1)
}
```


## 移动零

https://leetcode.cn/problems/move-zeroes/

> 给定一个数组 nums，编写一个函数将所有 0 移动到数组的末尾，同时保持非零元素的相对顺序。
> 请注意 ，必须在不复制数组的情况下原地对数组进行操作。

主要是学习一下移动数组的方式，这里直接写代码了

```js
/**
 * @param {number[]} nums
 * @return {void} Do not return anything, modify nums in-place instead.
 */
var moveZeroes = function (nums) {
  function move(arr, index) {
    for (let i = index; i < arr.length; i++) {
      arr[i] = arr[i + 1];
    }
  }
  for (let i = 0; i < nums.length; i++) {
    while (nums[i] === 0) {
      move(nums, i);
    }
    if (nums[i] == undefined) nums[i] = 0;
  }
};
```

## 两个数组的交集

https://leetcode.cn/problems/intersection-of-two-arrays/

> 给定两个数组  nums1  和  nums2 ，返回 它们的交集  。输出结果中的每个元素一定是 唯一 的。我们可以 不考虑输出结果的顺序 。
> 示例 1：
> 输入：nums1 = [1,2,2,1], nums2 = [2,2]
> 输出：[2]

```js
/**
 * @param {number[]} nums1
 * @param {number[]} nums2
 * @return {number[]}
 */
var intersection = function (nums1, nums2) {
  const set = new Set();
  const res = new Set();
  for (let num of nums1) {
    set.add(num);
  }
  for (let num of nums2) {
    if (set.has(num)) res.add(num);
  }
  return [res.values()];
};
```

---

方法 2：排序+双指针
思路：给两个数组排序，然后用双指针（i，j）分别从两个数组的开头开始，向后移动。

- 如果`nums1[i] > nums2[j]`，就 j++
- 如果`nums1[i] < nums2[j]`，就 i++。总之就是移动较小的那个数字。
- 如果`nums1[i] === nums2[j]`，就记录，然后 i 和 j 同时++。

另外考虑到去掉重复的数字，还可以设置一个变量 pre 表示前一个作为交集的数字。当`nums1[i] === nums2[j]`时，就把 pre 设置为 nums1[i]；后面如果再遇见值为 pre 的 nums1[i]就跳过，否则就记录并更改 pre。

## 旋转数组（轮转数组）

> 给你一个数组，将数组中的元素向右轮转 k  个位置，其中  k  是非负数。
> 示例 1:
> 输入: nums = [1,2,3,4,5,6,7], k = 3
> 输出: [5,6,7,1,2,3,4]
> 解释:
> 向右轮转 1 步: [7,1,2,3,4,5,6]
> 向右轮转 2 步: [6,7,1,2,3,4,5]
> 向右轮转 3 步: [5,6,7,1,2,3,4]

这道题的暴力法可以把最后一个元素移到前面，然后执行 k 次就行。

但是还是有一个简单方法的：

1. 翻转整个数组
2. 翻转前 K 个值
3. 翻转后 K 个值

最后就可以得到结果

```js
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {void} Do not return anything, modify nums in-place instead.
 */
var rotate = function (nums, k) {
  if (nums.length === 1) return;
  if (k === nums.length) return;
  const swap = (arr, i, j) => ([arr[i], arr[j]] = [arr[j], arr[i]]);
  function partReverse(arr, i, j) {
    while (i < j) {
      swap(arr, i, j);
      i++;
      j--;
    }
  }
  const tmp = nums.reverse();

  if (k > nums.length) {
    // 注意处理K大于数组长度的情况
    k = k % nums.length;
  }
  partReverse(tmp, 0, k - 1);
  partReverse(tmp, k, nums.length - 1);
};
```

## 差分数组（前缀和升级版）

差分数组的主要适用场景是频繁对原始数组的某个区间的元素进行增减。
比如说，给输入一个数组 nums，然后又要求给区间 nums[2..6] 全部加 1，再给 nums[3..9] 全部减 3，再给 nums[0..4] 全部加 2，最后问 nums 数组的值是什么？

差分数组的核心是 diff 数组。diff 数组和原数组可以互相转换，之间最重要的关系：

```js
diff[i] = nums[i] - nums[i - 1];

nums[i] = nums[i - 1] + diff[i];

diff[0] = nums[0];
```

可以看到，diff 数组基本上特点是：

- 表示当前位置的数和前一个位置的差
- 和原数组首位相同

差分数组问题的解决方法，通常也就是对 diff 数组进行操作，再转回原来的数组。

> 这样构造差分数组 diff，就可以快速进行区间增减的操作，如果你想对区间 `nums[i..j]` 的元素全部加 3，那么只需要让 `diff[i] += 3`，然后再让 `diff[j+1] -= 3` 即可
> 注意后面的范围是 j+1，可以理解为前缀和求区间时，不应该减掉区间的最后一个数 j，而是从最后一个数的下一个 j+1 剪掉。但是下面有一道题恰好是要从 j 开始减去。

这个原理也比较好理解，手写推导一下就出来了。`diff[i] += 3` 意味着给 `nums[i..]` 所有的元素都加了 3，`diff[j+1] -= 3` 又意味着对于 `nums[j+1..]` 所有元素再减 3，综合起来就像前缀和一样，将 ij 之间的部分加了 3.

解决这类问题的通用模板：

```js
// 创建diff
const diff = [];

// 根据推导公式得到diff具体的值
diff[0] = nums[0];
for (let i = 1; i < length; i++) {
  diff[i] = nums[i] - nums[i - 1];
}

// 根据题目对某个范围的加减要求，对diff进行加减
for (let update of updates) {
  const [start, end, inc] = update;
  // 从start到end之间的数字增加inc，
  diff[start] += inc;
  if (end + 1 < length) diff[end + 1] -= inc; // 如果end+1都超长了，就不再需要减
}

// 再转换回基本数组即可
const res = [];
res[0] = diff[0];
for (let i = 1; i < length; i++) {
  res[i] = res[i - 1] + diff[i];
}
return res;
```

**注意：要记住当end+1超长时就不需要再减了，要记得这个条件判断**

### 区间加法

https://leetcode.cn/problems/range-addition/

> 假设你有一个长度为  n  的数组，初始情况下所有的数字均为  0，你将会被给出  k​​​​​​​ 个更新的操作。
> 其中，每个操作会被表示为一个三元组：[startIndex, endIndex, inc]，你需要将子数组  A[startIndex ... endIndex]（包括 startIndex 和 endIndex）增加  inc。
> 请你返回  k  次操作后的数组。

这是差分数组的最基本题型，这里直接上代码了：

```js
/**
 * @param {number} length
 * @param {number[][]} updates
 * @return {number[]}
 */
var getModifiedArray = function (length, updates) {
  const nums = new Array(length).fill(0);
  const diff = [];

  diff[0] = nums[0];
  for (let i = 1; i < length; i++) {
    diff[i] = nums[i] - nums[i - 1];
  }

  for (let update of updates) {
    const [start, end, inc] = update;
    diff[start] += inc;
    if (end + 1 < length) diff[end + 1] -= inc;
  }

  const res = [];
  res[0] = diff[0];
  for (let i = 1; i < length; i++) {
    res[i] = res[i - 1] + diff[i];
  }
  return res;
};
```

### 拼车

https://leetcode.cn/problems/car-pooling/

> 车上最初有  capacity  个空座位。车   只能   向一个方向行驶（也就是说，不允许掉头或改变方向）
> 给定整数  capacity  和一个数组 trips ,  trip[i] = [numPassengers, from, to]  表示第 i 次旅行有  numPassengers  乘客，接他们和放他们的位置分别是  from  和  to 。这些位置是从汽车的初始位置向东的公里数。
> 当且仅当你可以在所有给定的行程中接送所有乘客时，返回  true，否则请返回 false。

这道题有一点不太明显，但是如果考虑用一个数组 passengers[i]表示第 i 站时车上的人数，每次上下车都是对该数组一个区间内的增减，就很好理解了。

但是还是有不少坑：

- 题目没给站数，因此直接取范围上限 1000
- 乘客到第 i 站时下车，意味着此时会立刻减去，而不是到下一站才减去。一站内可以允许乘客同时上下车，即使此时最大乘客数已经超出 capacity。因此后面的部分就应该是`to`而不是`to+1`

```js
/**
 * @param {number[][]} trips
 * @param {number} capacity
 * @return {boolean}
 */
var carPooling = function (trips, capacity) {
  const passengers = new Array(1001).fill(0); // passengers[i]表示第i站时车上的人数

  const diff = [...passengers];

  for (let trip of trips) {
    const [num, from, to] = trip;
    diff[from] += num;
    if (to < 1001) diff[to] -= num;
  }

  const res = [];
  res[0] = diff[0];
  for (let i = 1; i < 1001; i++) {
    res[i] = res[i - 1] + diff[i];
    if (res[i - 1] > capacity) return false;
  }
  return true;
};
```

## 前缀和

基本的前缀和就不再赘述，这里说一个前缀和解题的常用套路，即和哈希表结合。

因为即使用前缀和求子数组和，如果分别遍历双指针前缀，最后的复杂度还是 O(n^2)，没有意义；如果和哈希表结合，就可以实现 O(n)的复杂度。

方法类似于两数之和：遍历一次数组，每次加上当前值，然后用哈希表存起来当前位置的前缀和。每次查找`当前前缀和 - 子数组目标和`在哈希表中是否出现即可。原理就是`当前前缀和 - 之前的前缀和 = 子数组和`。

至于哈希表的 value 存什么，根据情况可以是索引（求子数组长度）或者是次数（求子数组和出现次数）

```js
for(let i = 1;i < nums.length;i++){
    pre += nums[i]
    if(map.has(pre - k)){
        ...
    }
    if(!map.has(pre)) map.set(pre,...)
    else map.set(pre,...)
}

```

**注意，前缀和的起始应该是0，但这个0不应该在前缀和的数组里，而是直接放到哈希表里**。
啥意思呢？以数组`[1,2,3,4,5]`为例。如果以nums[0]作为preSum[0]的话，那么如果想得到和为10的子数组，就会计算不出来，因为preSum[0]是1。
因此我们应该在-1位置上增加一个0，即`preSum[-1] = 0`；实际解题过程中并不需要真的向preSum添加，因为会打乱其他元素的index。可以在map中增加一个，即`map.set(0,-1)`，然后其他位置正常设置即可。这样就不会出现子数组从头开始没算上的问题。

```js
const findSubArraySum = (nums,target) => {
  const preSum = [nums[0]]
  for(let i = 1;i < nums.length;i++){
    preSum[i] = preSum[i-1] + nums[i]
  }
  const map = new Map()

  map.set(0,-1) // 这里，如果不加的话，从第一位开始的子数组就算不出来
  
  for(let i = 0;i < preSum.length;i++){
    if(map.has(preSum[i] - target)) return [map.get(preSum[i] - target),i]
    map.set(preSum[i],i) 
  }
  return [-1,-1]
}
```


### 连续数组（前缀和）

https://leetcode.cn/problems/contiguous-array/

> 给定一个二进制数组 nums , 找到含有相同数量的 0 和 1 的最长连续子数组，并返回该子数组的长度
> 示例 1:
> 输入: nums = [0,1]
> 输出: 2
> 说明: [0, 1] 是具有相同数量 0 和 1 的最长连续子数组。

这道题需要转化一下思路：
我们把 0 变为-1，这样这道题就变成了：找到一个最长的子数组使得其内部的和为 0。
这显然就是**前缀和**问题。想到子数组求和问题就应该考虑到前缀和、差分数组等方法。
但是具体实现中也有特殊技巧。如果直接用前缀和（即双指针确定两个边界）遍历所有和为 0 的情况然后找出最大长度，就会超时。
题解的实现是并不直接采用前缀和数组，而是这样：

- 设置一个 map 用于记录子数组的各种可能的和

- 遍历数组，和每一项相加，然后把和存到 map 中

  - 如果 map 中没有，就存下这个和为键，值为当前的索引，表示从第一项开始到这一项的和
  - 如果 map 中有，就取出这个和对应的值（也就是第一次得到这个和的索引）。这时，**过了几项之后和依然不变，就说明这两个范围之间的所有值的和是 0**；也就是找到了一个和为 0 的子数组，记录长度即可

  > ```
  > 比如数组[1,1,-1,-1,1,1,1]
  > 第二项时得到和为2，记录；
  > 然后到第6项时发现和又为2，这就说明在第二项和第六项之间的这些数没有对总和产生影响，即这些数和为0
  > 可以看到中间的子数组为[-1,-1,1,1]，就是和为0的子数组。
  > ```

代码如下：

```js
var findMaxLength = function (nums) {
  let maxLen = 0;
  let sum = 0;
  const map = new Map();
  map.set(0, -1);
  for (let i = 0; i < nums.length; i++) {
    sum += nums[i] === 0 ? -1 : 1; // 加的还是-1和1
    if (map.has(sum)) {
      const len = i - map.get(sum);
      maxLen = Math.max(maxLen, len);
    } else map.set(sum, i);
  }
  return maxLen;
};
```

## 有效三角形的个数（特殊三指针）

https://leetcode.cn/problems/valid-triangle-number/

> 给定一个包含非负整数的数组  nums ，返回其中可以组成三角形三条边的三元组个数。
> 示例 1:
> 输入: nums = [2,2,3,4]
> 输出: 3
> 解释: 有效的组合是:
> 2,3,4 (使用第一个 2)
> 2,3,4 (使用第二个 2)
> 2,2,3

这道题的回溯解法超时了，就不考虑了。

三指针解法的基本思路很好想 就是固定一个指针改变另外两个指针，如果保证两边之和大于第三边，就记录下来。但是实际操作过程中有一些问题，这道题需要考虑换一种指针的遍历方式。

（首先数组应该从小到大排序）

最开始想到的是这种：

```
[1, 2, 3, 4, 5]
 i  j        k

j = i+1
k = len - 1
```

类似对撞指针，当 `i+j<k`，就把 k 向左移动。
这种方法相当于 j 和 k 之间的范围在从大到小变化。由于 j 不能和 k 相等，这样就会错过一些情况

比如示例 1：

```
[2, 2, 3, 4]
 i  j     k
 i  j  k
    i  j  k

 i     j  k  ?
```

这里就可以明显看到，少了一种情况，即最后一行的问号情况。如果想要考虑这种情况，就需要每当两边之和小于第三边时，分别把 k 向左和 j 向右两种情况都考虑一遍。但是这样太麻烦了

这个解法有问题的关键在于，k 是那个最大的边，但 k 一直在变动，不好比较。大多数题解采用的方式都是固定位置为最大边，即 i 为最右边的位置
比如 i 倒序遍历：

```
[1, 2, 3, 4, 5]
 j        k   i
 j     k  i
```

代码如下：

```js
var triangleNumber = function (nums) {
  nums.sort((a, b) => a - b);
  let res = 0;
  for (let i = nums.length - 1; i >= 2; i--) {
    let j = 0;
    let k = i - 1;
    while (j < k) {
      if (nums[j] + nums[k] > nums[i]) {
        res += k - j;
        k--;
      } else j++;
    }
  }
  return res;
};
```

或者 i 正向，l 和 r 位于 i 的后方这样的遍历：

```
[1, 2, 3, 4, 5]
 l     r  i
 l        r  i
```

总之就是要保证最大值位置固定（i），这样改变 j 和 k 的值就有了一个确切的依据

代码如下：

```js
var triangleNumber = function (nums) {
  nums.sort((a, b) => a - b);
  let res = 0;
  for (let i = 0; i < nums.length; i++) {
    let left = 0,
      right = i - 1;
    while (left < right) {
      if (nums[left] + nums[right] > nums[i]) {
        res += right - left;
        right--;
      } else {
        left++;
      }
    }
  }
  return res;
};
```

实际上很多双指针、三指针的问题，指针的移动方式都有很多种。如果一种常用的方式（比如两边向中间逼近）有问题，就可以尝试其他种
官方题解的移动方式甚至是这样：

```
[1, 2, 3, 4, 5]
 i  j  k
 i  j     k
 i     j     k
```

即 j 和 k 同向移动；j 从 i+1 移动到 len-1，每移动一步，k 就循环向右移动，直到找到一个边界位置使得`k >= i + j`；那么前一个位置就是 k 的最远位置，`res += k - 1 - j`即可。

代码：

```js
var triangleNumber = function(nums) {
    const n = nums.length;
    nums.sort((a, b) => a - b);
    let ans = 0;
    for (let i = 0; i < n; i++) {
        let k = i;
        for (let j = i + 1; j < n; j++) {
            while (k < n && nums[k] < nums[i] + nums[j]) k++;
            ans += Math.max((k - 1) - j, 0);
        }
    }
    return ans;
};。
```

## 寻找重复数（成环链表）

> 给定一个包含  n + 1 个整数的数组  nums ，其数字都在  [1, n]  范围内（包括 1 和 n），可知至少存在一个重复的整数。
> 假设 nums 只有 一个重复的整数 ，返回   这个重复的数 。
> 你设计的解决方案必须 不修改 数组 nums 且只用常量级 O(1) 的额外空间
> 示例 1：
> 输入：nums = [1,3,4,2,2]
> 输出：2

这道题如果不限制空间，最好的方法当然是哈希表。
但是限制额外空间，就需要一种特殊的方法。这个方法在判断成环链表中出现过，就是 floyd 判圈算法。

> 详细解释可以参考环形链表的两道题：
> https://leetcode.cn/problems/linked-list-cycle/ > https://leetcode.cn/problems/linked-list-cycle-ii/

这个算法大概是这样：
![](https://pic1.imgdb.cn/item/6346a72c16f2c2beb1b7ef82.jpg)

设定两个指针 fast 和 slow，fast 每次走两步，slow 每次走一步。由于链表有环，fast 和 slow 最终会相遇；此时 fast 一定在环中走了 n 圈；
在上面这张图中，我们假设在紫色点处两个指针相遇；这时 slow 指针走的长度为`a + b`，fast 可能走了 n 圈，因此为`n * (b + c) + a`。又因为 fast 走的长度一定是 slow 的两倍，因此可以得到下面这个关系：
![](https://pic1.imgdb.cn/item/6346a7fd16f2c2beb1b9ce7c.jpg)
即：从相遇点到入环点的距离加上 n-1 圈的环长，恰好等于从链表头部到入环点的距离。

所以这里就可以得到两个题的题解：

- 如果 slow 和 fast 相遇，就说明一定有环
- 入环点应该是可以计算的。计算方法见下：

计算入环点其实就是计算 a 值，起点加上 a 就是入环点。

- 设置一个 ptr 指针，从头开始走
- slow 指针从 fast 和 slow 相遇的位置开始
- 两个指针每次都走一步。当他们相遇时，就到了入环点的位置。

为什么呢？显然他们相遇时，ptr 指针一定只走了 a 长度（一旦入环就不会再相遇）。从上面的公式可以看出，长度 a 应该等于`c + (n-1)(b+c)`，即长度 c 和 n-1 圈。也就是说当 slow 和 ptr 相遇时，slow 指针可能会走 n-1 圈（相当于没走），然后再走一个距离 c，就恰好在入环点位置相遇。

说回这道题，这道题的关键是怎么理解成上面说的环形链表问题，也就是怎么找到一个“环”
在链表中，每一个节点都有相连的下一个节点。但是数组中没有明确的相连关系；因此我们定义一个映射`x -> f(x)`为`n -> nums[n]`，即对于每个 n，它的下一项应该是数组中的第 n 个数；
这里是最重要的一点，即`n -> nums[n]`这个映射

![](https://pic1.imgdb.cn/item/6346aa0916f2c2beb1be258e.jpg)

如果数组中有相同的数，就会导致不同的 n 对应同一个 nums[n]（不同位置上的数相同），转换成链表形式就是成环
![](https://pic1.imgdb.cn/item/6346aa6516f2c2beb1bef6ba.jpg)

所以问题就转化成了，找到这个链表的入环点，就是重复的那个数字。

同样设置两个指针 fast 和 slow；注意这里，链表中我们可以表示为 slow = slow.next，因为链表的映射为`p -> p.next`；这里我们的映射是`n -> nums[n]`，所以这个指针的“下一项”就是`slow = nums[slow]`。
fast 同理，即`fast = nums[nums[fast]]`，相当于两次映射

![](https://pic1.imgdb.cn/item/6346aaff16f2c2beb1c04f5c.jpg)

有了这个关系之后，后面的代码就和
链表基本一致了：

```js
var findDuplicate = function (nums) {
  let slow = 0;
  let fast = 0;
  slow = nums[slow]; // slow的下一个（走一步）
  fast = nums[nums[fast]]; // fast的下一个（走两步）
  while (slow !== fast) {
    slow = nums[slow];
    fast = nums[nums[fast]];
  }
  // pre指针找到入环点
  let pre = 0;
  let post = slow;
  while (pre !== post) {
    pre = nums[pre];
    post = nums[post];
  }
  return pre;
};
```

# 字符串

## 回文字符串

> 给定一个非空字符串 s，最多删除一个字符。判断是否能成为回文字符串。
> 示例：
> 输入: "abaca"
> 输出: True
> 解释: 你可以删除 c 字符。

思路：利用回文字符串的“对称性”，即字符串从两头分别遍历，每个字符都相等（直到两个遍历指针相碰）

```js
function isPalindrome(str) {
  const strArr = str.split("");
  let i = 0;
  let j = strArr.length - 1;
  while (i <= j) {
    if (strArr[i] === strArr[j]) {
      i++;
      j--;
    } else {
      return false;
    }
  }
  return true;
}
```

那么放到这个题就是，现进行一次上面的操作，直到找到第一个不相等的字符串为止；

然后“跳过”这个字符，分别让左指针跳过当前正在指的字符、右指针跳过当前正在指的字符，然后判断跳过之后左右指针中间的字符段是否回文。

原理其实是，由于判断出现不同时，不知道是左边还是右边的那个字符不对，因此就让左右分别跳过，测试跳过之后中间部分是不是还能正常回文。

```js
function palindrome(str) {
  const strArr = str.split("");
  let i = 0;
  let j = strArr.length - 1;
  while (i <= j) {
    if (strArr[i] === strArr[j]) {
      i++;
      j--;
    } else {
      return isPalindrome(str.splice(i, 1)) && isPalindrome(str.splice(j, 1));
    }
  }

  return true;
}

function isPalindrome(str) {
  const strArr = str.split("");
  let i = 0;
  let j = strArr.length - 1;
  while (i <= j) {
    if (strArr[i] === strArr[j]) {
      i++;
      j--;
    } else {
      return false;
    }
  }
  return true;
}
```

## KMP 算法

KMP 算法的讲解可以参考 https://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html
详细代码参考 https://programmercarl.com/0028.%E5%AE%9E%E7%8E%B0strStr.html#%E5%85%B6%E4%BB%96%E8%AF%AD%E8%A8%80%E7%89%88%E6%9C%AC

```js
/**
 *
 * @param {string} s1
 * @param {string} s2
 */
function KMP(s1, s2) {
  // 获取next数组，这里就是最大前缀数组
  function getNext(str) {
    const res = [0]; // 第一项一定是0
    for (let k = 1; k < str.length; k++) {
      //从第二个字母开始遍历
      let i = 0;
      let j = k;
      let start = "";
      let end = "";
      let max = 0;
      while (i < str.length - 1 && j > 0) {
        start += str[i++];
        end += str[j--]; // 因为end是从后向前拼的，应该倒转一下
        if (start === end.split("").reverse().join("")) {
          max = Math.max(max, start.length);
        }
      }
      res.push(max);
    }
    return res;
  }
  const str = s1.split("");
  const search = s2.split("");
  const next = getNext(search);
  let j = 0;
  for (let i = 0; i < str.length; i++) {
    // 这里用while而不是用if，因为如果出现不匹配，应当是按照前缀数组next一个一个
    while (j > 0 && str[i] !== search[j]) {
      // 如果j=0，说明子串第一项就不匹配，直接继续（只i++）
      j = next[j - 1]; // 回退的实际上是前一项的最大前后缀
    }
    // 当前匹配，继续
    if (str[i] === search[j]) {
      j++;
    }
    // 匹配完成
    if (j === search.length) {
      return i - search.length + 1;
    }
  }
  return -1;
}
```

next 的计算还有一种的写法，利用前缀和后缀的最后一个字母不同时，回退前一个 next 值。

```js
function getNext(s) {
  const next = [0]; // 第一项一定是0
  let j = 0; // j不仅表示前缀的最后一个位置,还表示前后缀相同长度
  for (let i = 1; i < s.length; i++) {
    while (j > 0 && s[i] !== s[j]) {
      // 这里j一直回退到0或者相同为止。“相同”即代表回退到了之前匹配到的最长前缀；如果之前都没有，那j就会一直到0，next[i]也是0
      // 为什么要用while一直回退而不是只回退一次呢
      // 因为只有s[i] === s[j]的时候, j才能表示之前的最长相同前后缀的长度,从而跳出循环并把j+1赋给next[i]
      // 当然有可能前面一直没有相同的,那就回退到0,表示这个位置没有相同前后缀,下次再从头开始
      j = next[j - 1];
    }
    if (s[i] === s[j]) {
      // 找到了相同的前后缀
      j++;
    }
    next[i] = j;
  }
  return next;
}
```

## 重复子字符串

> 给定一个非空的字符串 s ，检查是否可以通过由它的一个子串重复多次构成。
> 输入: s = "abcabcabcabc"
> 输出: true
> 解释: 可由子串 "abc" 重复四次构成。 (或子串 "abcabc" 重复两次构成。)

思路是利用 KMP 算法中的 next 数组。因为 next 数组判断的是最长公共前缀，那么 next 数组的最后一项就应该是前面的最长重复子串的长度的整数倍数。
即：

```
getNext("abcdabcdabcd")
(12) [0, 0, 0, 0, 1, 2, 3, 4, 5, 6, 7, 8]
```

8 这个值是`abcdabcd`的长度，这个长度重复两次就刚好是这个字符串。
然后让`12 - 8 = 4`，即除去公共串后，剩下还有多长；剩下的 4 可以整除 12，说明存在长度为 4 的重复子字符串。
因此公式为：

```js
next.length % (next.length - next[next.length - 1]) === 0 &&
  next[next.length - 1];
// 能整除，并且最后一项不为0，说明为true
```

代码如下：

```js
/**
 * @param {string} s
 * @return {boolean}
 */
var repeatedSubstringPattern = function (s) {
  function getNext(s) {
    const next = [0]; // 第一项一定是0
    let j = 0;
    for (let i = 1; i < s.length; i++) {
      while (j > 0 && s[i] !== s[j]) {
        j = next[j - 1];
      }
      if (s[i] === s[j]) {
        j++;
      }
      next[i] = j;
    }
    return next;
  }
  const next = getNext(s);
  if (s.length % (s.length - next[next.length - 1]) || !next[next.length - 1])
    return false;
  return true;
};
```

但是这道题其实有更简单的两个解法，可以参考官方解法 https://leetcode.cn/problems/repeated-substring-pattern/

简单写一下思路：

1. 解法 1：枚举，从第一个元素开始作为子串。假设子串长度为 n，那么把子串重复`length / n`次就可以和 s 相同；如果不同就继续增加子串长度，直到相同或遍历到`length / 2`为止。
2. 解法 2: 查找`s+s`去掉首位元素的中间部分有没有 s，这个方法的原理参考题解。最重要的在于提供了一个思路，就是`s+s`中间部分依次取 n 个字符可以得到 s 的多种“翻转”情况，比如：

```
s = acd
s+s = acdacd

排除首尾元素，中间部分用大小为 s.length 的滑动窗口取，就是s所有可能的翻转

a(cda)cd ac(dac)d

那为什么说找到s的翻转就能证明s是可以用一个短串重复得到的呢?
原因是对于s本身,如果s是由一个短串得来的,那么s翻转n次一定会得到原来的s
例如：abcabc

移位一次：cabcab
移位两次：bcabca
移位三次：abcabc

上面的移位n次都表示一种可能的翻转. 那么我们只要想办法找到所有可能的翻转,其中只要包含原本的,就说明可以
因此只要在s+s中找到一个s就可以证明
```

## 最长公共前缀

https://leetcode.cn/problems/longest-common-prefix/

> 编写一个函数来查找字符串数组中的最长公共前缀。
> 如果不存在公共前缀，返回空字符串  ""。
> 示例 1：
> 输入：strs = ["flower","flow","flight"]
> 输出："fl"

思路就是纵向比较同一个位置上的字符，如果出现不一样的就立刻返回结果，否则持续向结果数组中放入当前字符。

```js
/**
 * @param {string[]} strs
 * @return {string}
 */
var longestCommonPrefix = function (strs) {
  const res = [];
  const lengths = strs.map((str) => str.length);
  for (let i = 0; i < Math.min(...lengths); i++) {
    let tmp = "";
    for (let str of strs) {
      if (!tmp || tmp === str[i]) tmp = str[i];
      else return res.join("");
    }
    res.push(tmp);
  }
  return res.join("");
};
```

## 列表序号

https://leetcode.cn/problems/excel-sheet-column-number/submissions/

> 给你一个字符串  columnTitle ，表示 Excel 表格中的列名称。返回 该列名称对应的列序号  。
> 例如：
>
> A -> 1
> B -> 2
> C -> 3
> ...
> Z -> 26
> AA -> 27
> AB -> 28

```js
/**
 * @param {string} columnTitle
 * @return {number}
 */
var titleToNumber = function (columnTitle) {
  const letters = columnTitle.split("").reverse();
  let sum = 0;
  const getNum = (letters) => letters.charCodeAt() - "A".charCodeAt();
  for (let i = 0; i < letters.length; i++) {
    sum += getNum(letters[i]) * 26 ** i;
  }
  return sum;
};
```

## 列表名称

这道题就是上一道题的反向，也就是常见的进制转化题。

对于一般性的十进制转 n 进制题目，只需要不断地对十进制数 num 进行 % 运算取得最后一位，然后对 num 进行 / 运算，将已经取得的位数去掉，直到 num 为 0 即可。

一般性的进制转换题目无须进行额外操作，但是这道题的特点是进制是从 1 开始的（A 表示 1），而不是 0，所以必须要给十进制数字先-1，再 % 得到余数。

```js
/**
 * @param {number} columnNumber
 * @return {string}
 */
var convertToTitle = function (columnNumber) {
  let ans = [];
  while (columnNumber > 0) {
    const a0 = (columnNumber - 1) % 26; // 这里是因为该进制是从1开始的而不是0
    ans.unshift(String.fromCharCode(a0 + "A".charCodeAt()));
    columnNumber = Math.floor((columnNumber - a0) / 26);
  }
  return ans.join("");
};
```

## 验证 IP 地址

https://leetcode.cn/problems/validate-ip-address/

这道题本身比较简单，但是要学习一个方法：
怎么检验一个字符是合法字符？（数字、字母等）
最简单粗暴的方法：

```js
const all = "0123456789abcdef";
for (let s of str) {
  if (!all.includes(s)) return false;
}
```

完整代码：

```js
/**
 * @param {string} queryIP
 * @return {string}
 */
var validIPAddress = function (queryIP) {
  return queryIP.includes(".") ? IPv4(queryIP) : IPv6(queryIP);
};
function IPv4(str) {
  const arr = str.split(".");
  let flag = true;
  arr.forEach((num) => {
    if (isNaN(num)) flag = false;
    else if (Number(num).toString().length !== num.length) flag = false;
    else if (+num > 255) flag = false;
  });
  return flag ? "IPv4" : "Neither";
}
function IPv6(str) {
  const arr = str.split(":");
  let flag = true;
  const isLetter = (str) => {
    str = str.toLowerCase();
    const all = "0123456789abcdef";
    for (let s of str) {
      if (!all.includes(s)) return false;
    }
    return true;
  };
  arr.forEach((num) => {
    if (num.length > 4 || num.length < 1) flag = false;
    else if (!isLetter(num)) flag = false;
  });
  return flag ? "IPv6" : "Neither";
}
```

## 重构字符串

https://leetcode.cn/problems/reorganize-string/

> 给定一个字符串  s ，检查是否能重新排布其中的字母，使得两相邻的字符不同。
> 返回 s  的任意可能的重新排列。若不可行，返回空字符串  "" 。

这道题其实是贪心。重构字符串时，需要根据每个字母在字符串中出现的次数处理每个字母放置的位置。**如果出现次数最多的字母可以在重新排布之后不相邻，则可以重新排布字母使得相邻的字母都不相同**。如果出现次数最多的字母过多，则无法重新排布字母使得相邻的字母都不相同。

那么当 n 为偶数时，显然每个字母出现次数都不能超过 n/2；当 n 为奇数，则不能超过`(n+1)/2`。如果能保证这样，就一定可以实现不相邻排布。

思路：先用哈希表统计每个字符的出现次数，然后按照每个字符的出现次数排序，即先排列出现次数多的
然后按照序号先偶后奇的顺序，先把所有偶数位排布字符，再排布剩下的奇数位。在偶数位和奇数位循环内部，还是按照字符出现次序，优先排布出现次数多的字符。

这个思路的证明很麻烦，参考[官方题解](https://leetcode.cn/problems/reorganize-string/solution/zhong-gou-zi-fu-chuan-by-leetcode-solution/)

代码：

```js
var reorganizeString = function (s) {
  const map = {};
  for (const char of s.split("")) {
    // 统计次数
    if (!map[char]) map[char] = 1;
    else {
      // 如果某一个字符出现次数超过了 n/2 + 1，就一定不能
      // 这里同时考虑了n为奇数和偶数的情况
      // 实际上可以分开考虑，偶数不能超过n/2，奇数不能超过(n+1)/2
      if (++map[char] >= s.length / 2 + 1) return "";
    }
  }
  const res = [];
  const charEntries = Object.entries(map);
  charEntries.sort((a, b) => b[1] - a[1]); // 排序，出现次数最多的在最前面

  let resIndex = 0; // 偶数位
  let index = 0;
  while (resIndex < s.length) {
    if (charEntries[index][1] > 0) {
      // 选取出现次数最多的字符，如果用完了就index++到下一个字符
      res[resIndex] = charEntries[index][0]; // 偶数位放上
      resIndex += 2;
      charEntries[index][1]--;
    } else index++;
  }
  resIndex = 1; // 奇数位
  while (resIndex < s.length) {
    if (charEntries[index][1] > 0) {
      res[resIndex] = charEntries[index][0];
      resIndex += 2;
      charEntries[index][1]--;
    } else index++;
  }
  return res.join("");
};
```

## 两两删除相同字符并加入一个新字符

> 给定一个只包含小写字母字符串，每次可以选择两个相同的字符删除，并在字符串结尾新增任意一个小写字母。
> 请问最少多少次操作后，所有的字母都不相同？
>
> 示例 1
> "abab"
>
> 输出
> 2
>
> 说明
> 第一次操作将两个'a'变成一个'f'，字符串变成"bbf"。
> 第二次操作将两个'b'变成一个'b'，字符串变成"fb"。
> 操作方式不是唯一的，但可以证明，最少操作次数为 2。

首先通过一个长度为 26 的数组记录下每个字母出现的次数；对于出现次数>=2 的字符，将其删去，然后选择一个未被放入字符串的字符放入。

```
比如对于"abab"，数组为[2,2,0,0...,0]
删掉b，放入第一个个数为0的字母，即第一个没有出现过的字母，比如c，数组为[2,0,1]
删掉a，同上，这时从头开始遍历，b已经变为0，因此可以再放入b，数组为[0,1,1]，没有重复元素了。
```

但是这种方法有个问题，如果所有字母都出现一遍之后，仍有>=2 的字母怎么办？即比如所有位置上都为 1，然后有几个位置上的值>=2
这时其实放入哪个字母都可以。我们就放入那个多的字母，比如字母 c 有三个，那就删去两个，再放入 1 个，相当于删了一个。依次类推，最后就能保证全为 1.

总结一下，一共有两种情况：

1. 第一轮删除可以完成，即数组中所有值都为 0 或 1
2. 第一轮删除不能完成，即数组中所有值都为 1 或更大。

第二种情况，删除次数就应该等于`多出来的总字母数量 - 1 = 此时的字符串长度 - 26`。

综合两种情况，代码如下：

```js
function minOperations(str) {
  const letters = new Array(26).fill(0);
  const getIndex = (char) => char.charCodeAt(0) - "a".charCodeAt(0);
  const isAllOne = (arr) => {
    for (const num of arr) {
      if (num !== 1) return false;
    }
    return true;
  };
  for (const char of str) {
    letters[getIndex(char)]++;
  }
  let cnt = 0;
  for (let i = 0; i < letters.length; i++) {
    while (letters[i] >= 2) {
      if (isAllOne(letters)) break;
      letters[i] -= 2;
      for (let j = 0; j < letters.length; j++) {
        if (letters[j] === 0) {
          letters[j]++;
          break;
        }
      }
      cnt++;
    }
    if (i === letters.length - 1) i = 0;
  }
  // cnt是第一轮的操作次数，此时的字符串长度 - 26是第二轮需要的操作次数
  return cnt + letters.reduce((a, b) => a + b) - 26;
}
```

# 链表

## 合并有序链表

> 将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有结点组成的。

有序链表的合成和有序数组的合并类似，都是用双指针依次遍历节点。

通过 LinkNode 新创建一个头节点，然后依次把两个链表中的节点比较、附加在这个新的头节点后面，形成的新链表就是有序链表。最后再把没连接上的多余部分加上去即可。

```js
const mergeTwoLists = function (l1, l2) {
  const head = new LinkNode();
  let curr = head;
  while (l1 && l2) {
    if (l1.val < l2.val) {
      curr.next = l1;
      l1 = l1.next;
    } else {
      curr.next = l2;
      l2 = l2.next;
    }
    curr = curr.next;
  }

  curr.next = l1 ? l1 : l2;
  return curr;
};
```

## 合并 K 个升序链表

https://leetcode.cn/problems/merge-k-sorted-lists/

题目就是给你 K 个链表，把他们按照升序合并。

思路有好几种，最简单的依次合并就不写了。

首先是归并合并，即每次将数组中的链表两两合并，合并后的结果放入一个数组中，然后再把这个数组递归合并，直到只剩一个链表为止。

```js
var mergeKLists = function (lists) {
  function merge2Lists(l1, l2) {
    if (!l1 && !l2) return null;
    else if (!l1 && l2) return l2;
    else if (l1 && !l2) return l1;
    const dummy = new ListNode(-Infinity);
    let curr = dummy;
    while (l1 && l2) {
      if (l1.val < l2.val) {
        const next = l1.next;
        curr.next = l1;
        l1.next = null;
        l1 = next;
      } else {
        const next = l2.next;
        curr.next = l2;
        l2.next = null;
        l2 = next;
      }
      curr = curr.next;
    }
    if (l1 && !l2) {
      curr.next = l1;
    } else if (!l1 && l2) {
      curr.next = l2;
    }
    return dummy.next;
  }
  function mergeList(lists) {
    if (lists.length <= 1) return lists[0] || null;
    const mergedLists = [];
    for (let i = 0; i < lists.length; ) {
      if (lists[i + 1] !== undefined) {
        // 可能是空链表null，不能直接if(list[i+1])
        mergedLists.push(merge2Lists(lists[i], lists[i + 1]));
        i += 2;
      } else {
        mergedLists.push(lists[i]);
        i++;
      }
    }
    return mergeList(mergedLists);
  }
  return mergeList(lists);
};
```

第二种方法是用优先队列。合并链表的过程可以看作是每次取这些链表的头节点的最小值，然后连接到一个新链表上。因此关键就是取最小值；如果每次都比较得到最小值的话时间复杂度会比较高
因此可以维护一个优先队列，初始化放入所有头节点，每次出队一个节点就是这些节点的最小值，然后再把这个出队的节点的 next 放入，依次循环直到 null 为止。
优先队列的数据结构 js 没有，但是可以类比最小栈写一个最小队列当作优先队列用，也就是一个能在 O(1)时间内返回最小值的队列

```js
class PQueue {
  constructor(compareFn = (val1, val2) => val1 - val2) {
    this.queue = [];
    this.compareFn = compareFn;
    this.size = 0;
  }
  push(val) {
    this.size++;
    if (!this.queue.length) this.queue.push(val);
    else {
      let left = 0;
      let right = this.queue.length - 1;
      // 二分查找找第一个大于它的数的左边界，然后插入这个数
      while (left <= right) {
        const mid = Math.floor((left + right) / 2);
        if (this.compareFn(this.queue[mid], val) < 0) {
          left = mid + 1;
        } else {
          right = mid - 1;
        }
      }
      this.queue.splice(left, 0, val);
    }
  }
  shift() {
    this.size--;
    return this.queue.shift();
  }
}

/**
 * @param {ListNode[]} lists
 * @return {ListNode}
 */
var mergeKLists = function (lists) {
  const queue = new PQueue((node1, node2) => node1?.val - node2?.val);
  for (const node of lists) {
    queue.push(node);
  }
  const dummy = new ListNode();
  let curr = dummy;
  while (queue.size) {
    const minNode = queue.shift();
    if (minNode) {
      queue.push(minNode.next);
      curr.next = minNode;
      curr = curr.next;
    }
  }
  return dummy.next;
};
```

## 删除所有重复元素（dummy 节点）

> 给定一个排序链表，删除所有含有重复数字的结点，只保留原始链表中 没有重复出现的数字。

这道题乍看起来比较简单，可以用一个 set 存储前面遍历过的数字，如果在 set 中找到，就把这些删去。

问题在于，这里要求删去所有节点，那么重复的这一组节点中的头一个节点就无法删去，因为不能获取到它前面的节点，就不能删去它

为了解决这个问题，设置一个位于整个链表头部的 dummy 节点，从 dummy 开始遍历（假设当前指针是 curr）：如果`curr.next.val === curr.next.next.val`，就用循环依次遍历到不相等的节点，然后把`curr.next`指过去即可。

dummy 节点的出现在这里主要是为了解决第一个节点就是重复节点的问题

```js
const deleteDuplicates = function (head) {
  // 设定 dummy 指针，作为头节点的前一个虚拟节点
  let dummy = new ListNode();
  dummy.next = head;
  let cur = dummy; // 从dummy开始，这样就解决了第一个就重复的情况
  while (cur.next && cur.next.next) {
    if (cur.next.val === cur.next.next.val) {
      // 若值重复，则记下这个值
      let val = cur.next.val;
      // 反复地排查后面的元素是否存在多次重复该值的情况
      while (cur.next && cur.next.val === val) {
        // 若有，则删除
        cur.next = cur.next.next;
      }
    } else {
      // 若不重复，则正常遍历
      cur = cur.next;
    }
  }
  // 注意是dummy.next
  return dummy.next;
};
```

## 快慢指针-链表的倒数第 n 个结点

> 给定一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点。

快慢指针即设定两个指针，快指针在慢指针前面或者快指针一次走的步数比慢指针多；当快指针到达某个位置时，慢指针往往就是对应的解。

比如倒数 n 个节点，可以让快指针先走 n 步，然后两者一起往前走；当快指针走到最后一个节点时，慢指针所指向的就是倒数第 n 个

```js
const removeNthFromEnd = function (head, n) {
  const dummy = new ListNode();
  dummy.next = head;
  let fast = dummy;
  let slow = dummy;
  for (let i = 0; i < n; i++) {
    if (!fast.next) return;
    fast = fast.next;
  }
  while (fast.next) {
    fast = fast.next;
    slow = slow.next;
  }
  slow.next = slow.next.next;
  return dummy.next;
};
```

## 反转链表

> 定义一个函数，输入一个链表的头结点，反转该链表并输出反转后链表的头结点。

记住方法：

1. 使用 prev/curr/next 三个指针分别指向前一个、当前和后一个节点；其中 curr 是第一个节点，prev 可以是 null 或者 dummy
2. 从 curr 指向的第一个节点开始，让当前节点的 next 指向 pre；
3. 然后令`curr = next，prev = curr，next = next.next`，即依次向后移动，直到最后一个节点被反转
4. cur 节点是真正反转的节点，因此要遍历到最后一个；当 cur 成为 null 时，pre 就是最后一个节点

> 实际上并不需要 next 指针，因为直接用`curr.next`就可以，每次遍历都先获取`curr.next`并缓存一下，然后继续即可

```js
const reverseList = function (head) {
  if (!head) return null;
  let pre = null; // 前节点
  let cur = head; // 当前节点，从头节点开始

  while (cur) {
    let next = cur.next; // 先缓存next
    cur.next = pre;
    pre = cur;
    cur = next;
  }

  return pre;
};
```

## 部分反转一个链表

> 部分反转：反转从位置 m 到 n 的链表。请使用一趟扫描完成反转。

核心方法和正常的反转链表一样，关键在于反转之后把反转的部分和其他的连起来。
其实就是先把反转部分的前节点和后节点存起来，反转完成后再连起来。

![](https://pic.imgdb.cn/item/6266b3c3239250f7c528044f.jpg)

以这幅图为例，当 curr 指针遍历到 left 时，即开始遍历到区域时，就把节点 1 缓存下来（此时正好是 pre 指针指向的）；
随着遍历的进行，当结点 4 的指针反转后，此时 cur 指针就恰好指在结点 5 上，这时只需要把节点 2 指向 cur 即可。

另外，还需要记录反转区域内部的头尾节点（上面的 2、4 节点）；其中 4 节点恰好是遍历完的 pre，直接`leftHead.next = pre`即可；但是 2 节点需要提前缓存（下面代码中的 start），结束后让`start.next = cur`连接尾部。

```js
const reverseBetween = function (head, m, n) {
  // 定义pre、cur，用leftHead来承接整个区间的前驱结点
  let pre, cur, leftHead;
  const dummy = new ListNode();
  // dummy后继结点是头结点
  dummy.next = head;
  // p是一个游标，用于遍历，最初指向 dummy
  let p = dummy;
  // p往前走 m-1 步，走到整个区间的前驱结点处
  for (let i = 0; i < m - 1; i++) {
    p = p.next;
  }
  // 缓存这个前驱结点到 leftHead 里
  leftHead = p;
  // start 是反转区间的第一个结点
  let start = leftHead.next;
  // pre 指向start
  pre = start;
  // cur 指向 start 的下一个结点
  cur = pre.next;
  // 开始重复反转动作
  for (let i = m; i < n; i++) {
    let next = cur.next;
    cur.next = pre;
    pre = cur;
    cur = next;
  }
  //  leftHead 的后继结点此时为反转后的区间的第一个结点
  leftHead.next = pre;
  // 将区间内反转后的最后一个结点 next 指向 cur
  start.next = cur;
  // dummy.next 永远指向链表头结点
  return dummy.next;
};
```

## 递归反转一个链表

链表的递归实际上和树的递归很像。链表同样有自己的前序和后序遍历：

```js
function traverse(head) {
  if (!head) return head;
  // 前序（正向遍历）
  traverse(head.next);
  // 后序（反向遍历）
}
```

后序遍历实际上就是在倒着走一遍链表。如果能在反向遍历的过程中改变指针，就表现为反转链表了。

先上写法：

```js
const reverseList = (head) => {
  if (!head || !head.next) return head; // 递归边界返回当前节点
  const last = reverseList(head.next); // last相当于一直递归找到了最尾端的节点
  /* 这个时候的head实际上是倒数第二个节点 */
  head.next.next = head; // 让当前节点的next指向倒数第二个节点
  head.next = null; // 倒数第二个节点的next要去掉，不然会成环
  return last; // 理论上递归过程中不需要这个last，但是最后需要这个作为反转之后的头节点
};
```

可以看到这种写法有几个关键点：

1. last 节点，虽然并没参与递归，但是需要最后作为头节点；
2. 固定的两步

- `head.next.next = head;`：表示把后一个节点指向前一个（当前的）
- `head.next = null`：表示当前节点的后指针置空，因为链表已经反转了，原先的头节点变成了尾，因此就必须要置空。但是如果是部分反转，那么就不能设为空，因为还要和其他部分连起来。

递归反转链表常见于部分反转，部分反转实际上就是更改递归边界即可。
比如反转前 k 个节点：

```js
const reverseN = (head, k) => {
  const after = null;
  function reverse(head, k) {
    if (k === 1) {
      after = head.next; // 递归到位，然后将后驱节点存下来
      return head;
    }
    const last = reverse(head.next, k - 1);
    head.next.next = head;
    head.next = after;
    return last;
  }
};
```

最后，反转任意一个部分的链表，其实就是先递归到起始节点位置，然后以起始节点为第一个节点，执行上面的反转前 k 个节点，最后把返回值和前面的连起来就行

```js
const reverseMN = (head, m, n) => {
  if (m === 1) {
    return reverseN(head, n);
  }
  head.next = reverseMN(head.next, m - 1, n - 1); // 相当于往后走到了起始节点，然后m和n都对应-1（比如从2/4变为1/3这样）
  return head;
};
```

## 链表交叉

> 给你两个单链表的头节点 headA 和 headB ，请你找出并返回两个单链表相交的起始节点。如果两个链表没有交点，返回 null 。

思路是这样：因为相交之前，两个链表长度不一样；所以可以先调整起始位置，让较长的链表向前移动几位，和较短的链表头节点并齐之后再开始遍历。
判断相交节点的方式是两个**指针**相同，注意不是值相同。

```js
/**
 * @param {ListNode} headA
 * @param {ListNode} headB
 * @return {ListNode}
 */
var getIntersectionNode = function (headA, headB) {
  function getLen(head) {
    let len = 0;
    let curr = head;
    while (curr) {
      len++;
      curr = curr.next;
    }
    return len;
  }
  const lenA = getLen(headA);
  const lenB = getLen(headB);
  let startIndex = Math.abs(lenA - lenB);
  let startA = headA;
  let startB = headB;
  let flag = null;
  if (lenA > lenB) {
    startA = headA;
    for (let i = 0; i < startIndex; i++) {
      startA = startA.next;
    }
  } else if (lenA < lenB) {
    startB = headB;
    for (let i = 0; i < startIndex; i++) {
      startB = startB.next;
    }
  }

  while (startA && startB) {
    startA = startA.next;
    startB = startB.next;
    if (startA === startB) flag = startA;
  }
  return flag;
};
```

---

上面那个方法太麻烦了，实际上哈希表就可以。第一次遍历记录所有节点，第二次检查是否有相同节点即可。

另外一个方法：

- 设置两个指针 p1 和 p2，分别从两个链表开头开始走，一直向后走到其中一个为空为止
  - 当 p1 或 p2 有一个为空，比如 p1 为空，就将其设为 head2 然后继续。p2 则设为 head1
  - 当 p1 === p2 或 p1 和 p2 都为空，则说明存在交点/不存在交点。

证明参考官方题解:`https://leetcode.cn/problems/intersection-of-two-linked-lists/solution/xiang-jiao-lian-biao-by-leetcode-solutio-a8jn/`

## 判断回文链表

> 给你一个单链表的头节点 head ，请你判断该链表是否为回文链表。如果是，返回 true ；否则，返回 false 。

这道题有几个思路：

1. 最朴实的方法，先快慢指针找到中间节点，然后从中间节点开始反转后面的链表，再双指针比较两个链表是否相等。
2. 顺序输出转换成数组，然后判断数组是否回文。

```js
/**
 * @param {ListNode} head
 * @return {boolean}
 */
var isPalindrome = function (head) {
  let curr = head;
  const arr = [];
  while (curr) {
    arr.push(curr.val);
    curr = curr.next;
  }
  return arr.join("") === arr.reverse().join("");
};
```

3. 递归方法。先通过递归走到最后一个节点，在返回的途中和正向遍历一一比较。

关键在于怎么一边利用递归反向遍历，一边同时还能正向遍历；
其实就是再设置一个指针，每次反向遍历一个位置时，就将这个指针指向的节点向后移一个。最终这两个指针都会走遍整个链表，中间过程中始终保持两个链表的值相同。

```js
var isPalindrome = function (head) {
  let left = head;
  let res = true;
  function dfs(node) {
    if (!node) return;
    dfs(node.next);
    if (left.val !== node.val) res = false;
    left = left.next;
  }
  dfs(head);
  return res;
};
```

这个方法的时间复杂度并没有优化，但是提供了一种全新的思路，即如何同时从两个方向遍历同一个链表，期间还能互相比较。

## 拆分链表

https://leetcode.cn/problems/partition-list/

> 给你一个链表的头节点 head 和一个特定值 x ，请你对链表进行分隔，使得所有 小于 x 的节点都出现在 大于或等于 x 的节点之前。
> 你应当 保留 两个分区中每个节点的初始相对位置。
> 输入：head = [1,4,3,2,5,2], x = 3
> 输出：[1,2,2,4,3,5]

这道题其实就是把一个链表拆成两个部分，同时不改变相对顺序。
如果是数组的话就很简单，创建两个空数组，遍历时将比 x 小的放入一堆，比 x 大的放入另一队，最后再合并就行。

链表的原理类似，但是有几个注意点：

- 需要 dummy 节点。这个很好理解，创建连接的空链表最好都要有 dummy，这个和合并有序链表时创建新链表先创建一个 dummy 的原因相同
- **将原链表的节点拆出来后，一定注意要把这个节点的 next 置空**。这点很重要，不然在最后的连接时就会出现问题。
- 最后链接的时候，前面的链表可能是空的（节点都比 x 大），但后面的一定不会（至少包括一个 x 所在的节点）。所以必须要 dummy 节点，不然不能把第二个链表的头节点挂在 null 上。

```js
/**
 * @param {ListNode} head
 * @param {number} x
 * @return {ListNode}
 */
var partition = function (head, x) {
  if (!head) return null;
  const dummy1 = new ListNode();
  const dummy2 = new ListNode();
  let p = head;
  let p1 = dummy1;
  let p2 = dummy2;
  while (p) {
    if (p.val < x) {
      p1.next = p;
      p1 = p1.next;
    } else {
      p2.next = p;
      p2 = p2.next;
    }
    // 注意把拆出来的节点的next置空
    const tmp = p.next;
    p.next = null;
    p = tmp;
  }
  p1.next = dummy2.next;
  return dummy1.next;
};
```

## 排序链表

https://leetcode.cn/problems/sort-list

> 给你链表的头结点 head ，请将其按 升序 排列并返回 排序后的链表 。
> 必须在 O(n log n) 时间复杂度和常数级空间复杂度下，对链表进行排序

这道题本身不难，难点在于要求的时间和空间复杂度。为了达到 O(nlogn)，只能采用归并和快排两种方式，并且归并还应该是迭代归并的自底向上形式，不能是传统的递归归并（递归的空间复杂度是 O(logn)，n 为链表长度，不符合常数空间复杂度要求）

递归归并和数组的排序方法基本一样，但是需要一个分割链表的函数。有了这个函数之后其他部分几乎完全一样。

首先是迭代归并，迭代归并其实就是从小向大的合并。递归归并采用的是把数组逐个分成 1/2、1/4、1/8 这样，然后从下向上组装起来；而迭代归并则是一开始就拆成长度为 1 的数组，然后两两合并变成长度为 2 的数组；然后再次合并，以此类推最后得到完整的数组。
大概是这样：

![](https://pic1.imgdb.cn/item/634032e816f2c2beb12c99c2.jpg)

因此可以使用两层循环：

- 外层循环遍历 len，每次都是上一次的 2 倍，len 就是每次要分割的数组长度，每次有序合并两个相邻的长度为 len 的数组，直到 len >= n 为止
- 在两个循环中间，每次让 curr 指针重新到头部，并且还要确定一个尾指针 tail，保证下一步每次连接到 tail 指针上。tail 指针所指的位置前面是已经合并好的，后面是待合并的。
- 内层循环遍历链表，先通过切割的方式切割两个相邻的、紧挨着上一次切割合并结果的长度为 len 的部分，然后按照合并有序链表的形式合并，再接到 tail 指针的末尾。

图：
![](https://pic1.imgdb.cn/item/634034b616f2c2beb1302aea.jpg)
可以看到，step=1 时，每次切割两个长度为 1 的链表，有序合并之后再接到 tail 上；step=2 时每次切两个长度为 2 的，如果最后一个长度小于 step，就不用合并，直接连接即可。

代码如下：
首先是合并有序链表的公共代码：

```js
const mergeList = (p1, p2) => {
  const dummy = new ListNode();
  let p = dummy;
  while (p1 && p2) {
    if (p1.val < p2.val) {
      let next = p1.next;
      p.next = p1;
      p1.next = null;
      p1 = next;
    } else {
      let next = p2.next;
      p.next = p2;
      p2.next = null;
      p2 = next;
    }
    p = p.next;
  }
  if (p1) p.next = p1;
  else p.next = p2;
  return dummy.next;
};
```

迭代归并代码：

```js
var sortList = function (head) {
  if (!head) return head;
  let len = 0;
  let tail = head;
  while (tail.next) {
    // 计算长度
    tail = tail.next;
    len++;
  }
  const dummy = new ListNode();
  dummy.next = head;
  let pre = dummy;
  let curr = head;
  for (let subLen = 1; subLen <= len; subLen *= 2) {
    pre = dummy;
    curr = pre.next; // curr每次subLen改变时重置为链表的第一个节点
    while (curr) {
      const head1 = curr; // head1和head2是切割的两个链表的头节点
      for (let i = 1; i < subLen && curr.next; i++) {
        curr = curr.next; // 向前走sublen步
      }
      const tmpNext1 = curr.next; // 存一下下一个节点
      curr.next = null; // 切割
      curr = tmpNext1; // curr变为下一个节点
      const head2 = curr; // 第二个切割部分的头部
      for (let i = 1; i < subLen && curr && curr.next; i++) {
        curr = curr.next;
      }
      // 同理
      let next = null;
      if (curr) {
        next = curr.next;
        curr.next = null;
      }
      curr = next; // 这里curr走到了两个切割部分的下一个，下次切割也从这里开始

      const mergedList = mergeList(head1, head2);
      pre.next = mergedList; // pre就是上面说的tail指针，始终指向连接好的部分的尾节点
      while (pre.next) {
        // 走到尾节点。因为前半部分是断开的，所以直接走到最后就行
        pre = pre.next;
      }
    }
  }
  return dummy.next;
};
```

---

还有一种方式是快排。快排不一定要能写出来，但是要理解方法。
和普通数组快排相比，最大的问题是不能用双指针一个向左一个向右来找小于、大于中间节点的节点。因此可以创建三个链表分别连接小于、等于和大于中间元素的元素，然后把小于和大于的链表递归快排，再把三个链表连接起来返回即可。

```js
function qsort(head) {
  let head1 = new ListNode();
  let head2 = new ListNode();
  let head3 = new ListNode();
  let tail1 = head1;
  let tail2 = head2;
  let tail3 = head3;

  let curr = head;
  const pivot = getMid(head).val; // 用快慢指针找中点，就不写了
  while (curr) {
    const next = curr.next;
    if (curr.val < pivot) {
      curr.next = null;
      tail1.next = curr;
      tail1 = tail1.next;
    } else if (curr.val > pivot) {
      curr.next = null;
      tail2.next = curr;
      tail2 = tail2.next;
    } else {
      curr.next = null;
      tail3.next = curr;
      tail3 = tail3.next;
    }
    curr = next;
  }

  head1 = qsort(head1.next);
  head3 = qsort(head3.next);
  // h1链表不一定有元素 h2链表一定有元素 先把h2、h3连起来
  head2 = head2.next;
  tail2.next = head3;
  // 如果h1链表为空 直接返回h2即可
  // 否则找到h1链表的结尾，连上h2的头
  if (head1 == null) {
    return head2;
  } else {
    tail1 = head1;
    // 找到t1链表的结尾
    while (tail1.next != null) {
      tail1 = tail1.next;
    }
    tail1.next = head2;
    return head1;
  }
}
```

## LRU 缓存

https://leetcode.cn/problems/lru-cache/submissions/

LRU，即最近最少使用，它的缓存内的内容会根据最近访问的对象改变位置。每次 put 和 get 都会把最新访问的数据放到最顶层，即最近使用；当 put 放入的数据超出大小限制时，则删除最后一个数据，即最少使用的那个。

在实现上可以仅使用一个 map 或者一个数组，但是如果想要按照题目要求使 get 和 put 复杂度为 O(1)，就必须采用哈希表+双链表的形式，并且双链表需要自己实现。

双链表的实现：

- 初始化双链表应该有两个 dummy 节点 dummyhead 和 dummytail，这两个节点通过 next 和 pre 相连，方便后面的节点插入。
- 双链表的节点需要有一个 key 和 val,分别对应 put 放入的 key 和 val。设置 key 的目的是**方便从双链表中删除节点时也从哈希表中删除**，这点很重要。

哈希表的作用就是记录当前双链表内部的值，并且哈希表的 value 应该是**双链表的节点**，也就是可以通过 key 直接检索到对应的节点，进行删除或修改操作。这也是 put 和 get 方法能达到 O(1)的根本原因

基本思路：

- 插入节点：可能会超出大小，也可能已经存在值。如果超出大小就删掉最后一个元素；如果已经存在值就删除这个值，然后创建一个新节点插入到头部。
- 获取节点：根据哈希表查找到对应的 node，把 node 删掉并插入到头部，这样就相当于移到头部；

代码如下：

```js
class ListNode {
  constructor(val, key, pre, next) {
    this.val = val || 0;
    this.next = next || null;
    this.pre = pre || null;
    this.key = key || 0; // 这个key为了方便从双链表中删除节点时也从哈希表中删除
  }
}

class DBLinkList {
  constructor() {
    const dummyHead = new ListNode();
    const dummyTail = new ListNode();
    dummyHead.next = dummyTail;
    dummyTail.pre = dummyHead;
    this.dummyHead = dummyHead;
    this.dummyTail = dummyTail;
  }
  insertHead(node) {
    let next = this.dummyHead.next;
    node.pre = this.dummyHead;
    node.next = next;
    next.pre = node;
    this.dummyHead.next = node;
  }
  deleteNode(node) {
    node.pre.next = node.next;
    node.next.pre = node.pre;
    node.next = null;
    node.pre = null;
    return node.key; // 返回的key正好是哈希表对应节点的key，相当于从节点反向检索到了key值
  }
  deleteTail() {
    return this.deleteNode(this.dummyTail.pre);
  }
}

class LRUCache {
  constructor(limit) {
    this.limit = limit;
    this.map = new Map();
    this.dbLinkList = new DBLinkList();
  }
  put(key, val) {
    const node = new ListNode(val, key);
    // 这个地方注意，要考虑新建节点和修改节点两种情况
    if (!this.map.has(key)) {
      if (this.map.size >= this.limit) {
        this.map.delete(this.dbLinkList.deleteTail());
      }
      this.dbLinkList.insertHead(node);
    } else {
      const oldNode = this.map.get(key);
      this.dbLinkList.deleteNode(oldNode);
      this.dbLinkList.insertHead(node); // 注意这里插入的是新的node，不要传成oldNode了
    }
    this.map.set(key, node);
  }
  get(key) {
    if (!this.map.has(key)) {
      return -1;
    } else {
      const node = this.map.get(key);
      this.dbLinkList.deleteNode(node);
      this.dbLinkList.insertHead(node); // 相似的逻辑，这里传的是oldNode
      return node.val;
    }
  }
}
```

# 栈和队列

## 括号匹配问题

> 给定一个只包括 `'('`，`')'`，`'{'`，`'}'`，`'['`，`']'` 的字符串，判断字符串是否有效。
> 有效字符串需满足：
> 左括号必须用相同类型的右括号闭合。
> 左括号必须以正确的顺序闭合。
> 注意空字符串可被认为是有效字符串。

比较基础的栈问题，先把左括号入栈，然后当遇见右括号时，根据预设的匹配关系出栈左括号，如果不匹配就返回 false

```js
const isValid = function (s) {
  const stack = [];
  const str = s.split("");
  const map = new Map([
    ["{", "}"],
    ["[", "]"],
    ["(", ")"],
  ]);

  for (let s of str) {
    if (s === "(" || s === "[" || s === "{") {
      stack.push(s);
    } else if (s === ")" || s === "]" || s === "}") {
      let top = stack.pop();
      if (s !== map.get(top)) return false;
    }
  }
  if (!stack.length) return true;
  else return false;
};
```

## "对对碰"消除

> 给出由小写字母组成的字符串 S，重复项删除操作会选择两个相邻且相同的字母，并删除它们。
> 在 S 上反复执行重复项删除操作，直到无法继续删除。
> 在完成所有重复项删除操作后返回最终的字符串。答案保证唯一。
>
> 示例：
>
> 输入："abbaca"
> 输出："ca"
> 解释：例如，在 "abbaca" 中，我们可以删除 "bb" 由于两字母相邻且相同，这是此时唯一可以执行删除操作的重复项。之后我们得到字符串 "aaca"，其中又只有 "aa" 可以执行重复项删除操作，所以最后的字符串为 "ca"。

利用栈，把每一项一次入栈，如果有和栈顶元素相同的就把栈顶元素出栈

![](https://code-thinking.cdn.bcebos.com/gifs/1047.%E5%88%A0%E9%99%A4%E5%AD%97%E7%AC%A6%E4%B8%B2%E4%B8%AD%E7%9A%84%E6%89%80%E6%9C%89%E7%9B%B8%E9%82%BB%E9%87%8D%E5%A4%8D%E9%A1%B9.gif)

代码：

```js
/**
 * @param {string} s
 * @return {string}
 */
var removeDuplicates = function (s) {
  const stack = [];
  for (let i = 0; i < s.length; i++) {
    const top = stack[stack.length - 1] || "";
    if (s[i] === top) stack.pop();
    else stack.push(s[i]);
  }
  return stack.join("");
};
```

## 后缀表达式求值

> 根据 逆波兰表示法，求表达式的值。
> 有效的运算符包括 + , - , \* , / 。每个运算对象可以是整数，也可以是另一个逆波兰表达式。
> 说明：
> 整数除法只保留整数部分。 给定逆波兰表达式总是有效的。换句话说，表达式总会得出有效数值且不存在除数为 0 的情况。
>
> 示例 1：
> 输入: ["2", "1", "+", "3", " * "]
> 输出: 9
> 解释: 该算式转化为常见的中缀算术表达式为：((2 + 1) \* 3) = 9
> 示例 2：
> 输入: ["4", "13", "5", "/", "+"]
> 输出: 6
> 解释: 该算式转化为常见的中缀算术表达式为：(4 + (13 / 5)) = 6

后缀表达式是计算机常用的计算表达式的方法。
后缀表达式可以看作是一棵后序遍历的树，每个运算符都是中间节点，以运算符为根节点的子树是一个表达式的值；
因此可以通过这个思路引入栈，执行这样的操作：

1. 如果是数字，入栈
2. 如果是运算符，就把栈顶的两个元素出栈，并通过运算符计算，结果入栈

代码如下：

> 注意这里有个坑：除法应该使用`Math.trunc`去掉小数部分，模拟 java 等语言中的整形运算。不能是`Math.floor`，因为处理负数时会向下取整。

```js
/**
 * @param {string[]} tokens
 * @return {number}
 */
var evalRPN = function (tokens) {
  const s = new Map([
    ["+", (a, b) => a * 1 + b * 1],
    ["-", (a, b) => b - a],
    ["*", (a, b) => b * a],
    ["/", (a, b) => Math.trunc(b / a)],
  ]);
  const stack = [];
  for (const i of tokens) {
    if (!s.has(i)) {
      stack.push(i);
      continue;
    }
    stack.push(s.get(i)(stack.pop(), stack.pop()));
  }
  return stack.pop();
};
```

## 单调栈和单调队列

单调栈顾名思义就是栈内元素从小到大或从大到小排列的栈。通常操作是在遍历数组的时候，将每个元素依次放入栈中，如果出现不符合单调栈单调特征的元素，就考虑把前面所有的数出栈，直到继续符合特征为止。

单调栈有两种遍历方式，可以正向遍历数组，也可以反向。

- 正向遍历的结果输出通常在循环内部，因为只有出栈时才能得出计算值
- 反向遍历的结果输出在循环外部，因为不出栈时，栈顶元素一般就是满足条件的元素，可以和当前元素计算得出结果

> 反向遍历的主要意义在于循环内比较简单，只需要弹出就够了，弹出完成后栈顶就是正常的值；相比于正向遍历每次弹出都要比一下要简单不少

模板如下：

```js
// 正向
const stack = [];
for (let i = 0; i < nums.length; i++) {
  while (stack.length && nums[i] >= stack[stack.length - 1]) {
    // 如果是递增栈就是<=，看情况
    const val = stack.pop();
    // ... 结果处理
  }
  stack.push(nums[i]); // 或者是索引i，索引栈是一个很好用的思路
}

// 反向
const stack = [];
for (let i = nums.length - 1; i >= 0; i--) {
  while (stack.length && nums[i] >= stack[stack.length - 1]) {
    // 如果是递增栈就是<=，看情况
    stack.pop();
  }
  res[i] = stack.length ? stack[stack.length - 1] : null; // 依题意而定
  stack.push(nums[i]);
}
```

### 更高温度（单调栈）

> 给定一个整数数组  temperatures ，表示每天的温度，返回一个数组  answer ，其中`answer[i]`  是指在第 i 天之后，才会有更高的温度。如果气温在这之后都不会升高，请在该位置用  0 来代替。
> 例如，给定一个列表 temperatures = [73, 74, 75, 71, 69, 72, 76, 73]，你的输出应该是 [1, 1, 4, 2, 1, 1, 0, 0]。

思路：创建一个索引栈，用于记录索引。这个栈同时还是一个最小栈，即保持栈顶的元素始终是最小的。

确定一个**单调递减的趋势**，一旦有数字打破了这个趋势，也就说一个数字比它前面入栈的数字大，那就把它前面比他小的数字都出栈，并且用它的索引减去这些数字的索引，减去的值恰好就是对应的 answer。

> 演示视频：https://www.bilibili.com/video/BV12t4y1274o/

至于为什么栈中不直接放入温度而是存索引，是因为要得出的结果 ans 是每个元素的索引相加减得出的，并且也恰好是按照索引顺序摆放的。
索引栈是一个很好用的思维方式，可以尝试多用用

```js
const dailyTemperatures = function (temperatures) {
  const stack = [];
  stack[0] = 0;
  const ans = [];
  for (let i = 1; i < temperatures.length; i++) {
    while (
      stack.length &&
      temperatures[i] > temperatures[stack[stack.length - 1]]
    ) {
      const index = stack.pop();
      ans[index] = i - index;
    }
    stack.push(i);
  }
  return ans;
};

// 反向遍历
var dailyTemperatures = function (temperatures) {
  const stack = [];
  const res = [];
  for (let i = temperatures.length - 1; i >= 0; i--) {
    while (stack.length && temperatures[i] >= stack[stack.length - 1])
      stack.pop();
    res[i] = stack.length ? stack[stack.length - 1] - i : 0;
    stack.push(i);
  }
  return res[i];
};
```

### 移掉 K 位数字（单调栈）

https://leetcode.cn/problems/remove-k-digits/

> 给你一个以字符串表示的非负整数  num 和一个整数 k ，移除这个数中的 k 位数字，使得剩下的数字最小。请你以字符串形式返回这个最小的数字。
> 示例 1 ：
> 输入：num = "1432219", k = 3
> 输出："1219"
> 解释：移除掉三个数字 4, 3, 和 2 形成一个新的最小的数字 1219 。

首先要先考虑一个原理型的问题，什么情况下能保证数字更小？
比如两个数字 123a456 和 123b456，当位数相同时，显然若`a<b`则`num1<num2`。因此如果要保证一个数字尽可能小，就应该使每一位尽可能小。
将原数字挨个遍历，如果当前的数字比前面的数字小，就应该替换掉前面的。反之则不行，因为替换了一定会使整个数字变大。

从这里就可以看出，这道题实际上是一个单调栈问题，即保证单调栈内的数字始终要递增；但是也不完全是单调栈，因为还有一个重要的条件是删除的数字限制。
每一次从单调栈中 pop 值，就相当于一次删除，总删除次数就应该减 1。当删除次数用尽时，就不能再继续 pop 了。这时可能还剩下几个数字，也会一并放入栈中。最后的结果应该是`slice(0, n-k)`，即截取应该保留的部分。

最后还要注意全删掉和前导 0 的处理

```js
var removeKdigits = function (num, k) {
  let stack = [];
  let cnt = k;
  for (let n of num) {
    // 单调栈的通常处理，逐个循环和栈顶元素比较
    // 当前数比栈顶元素小
    // 还有删除次数
    // 栈非空
    while (n < stack[stack.length - 1] && cnt > 0 && stack.length > 0) {
      stack.pop();
      cnt--;
    }
    stack.push(n);
  }
  // 最后只取前面的部分，因为上面每个元素都会被遍历到，超出删除次数的部分不会删除，要在这里截掉
  stack = stack.slice(0, num.length - k);
  while (stack.length > 1 && stack[0] === "0") stack.shift();
  return stack.join("") || "0";
};
```

这道题可以借鉴思路的地方除了采用单调栈来比较大小，还有一点就是删除次数这个记录量。每一次从栈中 pop 就是一次删除

### 下一个更大元素 II（单调栈）

https://leetcode.cn/problems/next-greater-element-ii/

> 给定一个循环数组  nums （ nums[nums.length - 1]  的下一个元素是  nums[0] ），返回  nums  中每个元素的 下一个更大元素 。
> 数字 x  的 下一个更大的元素 是按数组遍历顺序，这个数字之后的第一个比它更大的数，这意味着你应该循环地搜索它的下一个更大的数。如果不存在，则输出 -1 。
> 示例 1:
> 输入: nums = [1,2,1]
> 输出: [2,-1,2]
> 解释: 第一个 1 的下一个更大的数是 2；
> 数字 2 找不到下一个更大的数；
> 第二个 1 的下一个最大的数需要循环搜索，结果也是 2。

考虑一个递减栈，如果当前元素比栈顶元素小就入栈，如果比栈顶元素大，就逐个出栈，出栈的每个元素的“下一个最大元素”正好就是这个比栈顶元素大的元素。

又因为要考虑循环搜索，实际上只要走两遍数组就可以。最后没有结果的位置就是没找到最大值，设为-1 即可。

```js
var nextGreaterElements = function (nums) {
  const stack = [];
  const res = new Array(nums.length);
  for (let i = 0; i < 2 * nums.length; i++) {
    let index = i % nums.length;
    while (stack.length && nums[stack[stack.length - 1]] < nums[index]) {
      const top = stack.pop();
      res[top] = nums[index];
    }
    stack.push(index);
  }
  for (let i = 0; i < res.length; i++) {
    if (res[i] == undefined) res[i] = -1;
  }
  return res;
};
```

### 去除重复字母

https://leetcode.cn/problems/remove-duplicate-letters/

> 给你一个字符串 s ，请你去除字符串中重复的字母，使得每个字母只出现一次。需保证 返回结果的字典序最小（要求不能打乱其他字符的相对位置）。
> 示例 1：
> 输入：s = "bcabc"
> 输出："abc"

这道题关键在于不仅要去重，还要考虑结果的字典序最小。

按照上一道题的思路，同样可以采用单调栈的形式，每次放入的字母字典序都应该尽可能的小，如果前面的字典序更大，就应该依次出栈。

而在控制删除次数方面，这道题有点不一样。上一道题有一个全局的最大删除次数 k，但是这道题必须考虑单个字母的情况，即每个字母都应该有自己的一个最大删除次数（即保证自己出现的次数最少要有一次）。

所以还需要一个 map，用来记录每个字母的出现次数作为最大删除次数，不能小于 1。每次出栈，都要把这个字母的删除次数减 1；如果删除次数为 1，即说明此时剩余串中只剩下一个该字母，那就不能删去，应该保留。

另外还需要一个 set 用于记录重复字母。当当前遍历到的字母在栈中已经有了，就应该跳过，并且仍需要从 map 中给这个字母删除次数-1。即使是跳过，也相当于是一个删除，因此还是需要对 map 处理。

这道题不需要最后的截取，因为去重的 visited 一定保证每个字母出现且仅出现一次，那么结果的长度一定是固定的。

那么这个“最小字典序”的实现，实际上和上一道题类似，依旧使用递增栈。每次遍历一个字母时，就先判断这个字母和栈顶字母的字典序关系，如果字典序小于栈顶字母，就考虑将栈顶字母出栈，并将其删除次数-1（如果删除次数不足就不删掉，但当前字母还是要放入）

代码如下：

```js
var removeDuplicateLetters = function (s) {
  let stack = [];
  const visited = new Set();
  const map = new Map();
  for (let letter of s) {
    // 先记录每个字母的出现次数作为各自的最大删除次数
    if (!map.has(letter)) map.set(letter, 1);
    else map.set(letter, map.get(letter) + 1);
  }
  for (let letter of s) {
    if (visited.has(letter)) {
      // 重复的跳过
      map.set(letter, map.get(letter) - 1); // 跳过也相当于是删除
      continue;
    }
    while (
      letter < stack[stack.length - 1] && // 当前字母字典序小于栈顶
      map.get(stack[stack.length - 1]) > 1 && // 栈顶字母还能被删除
      stack.length > 0 // 非空栈
    ) {
      // 给栈顶字母的删除次数-1
      map.set(stack[stack.length - 1], map.get(stack[stack.length - 1]) - 1);
      visited.delete(stack.pop()); // 删除栈顶，并且从重复记录中去掉
    }
    visited.add(letter);
    stack.push(letter);
  }
  return stack.join("");
};
```

### 滑动窗口最大值

> 给定一个数组 nums 和滑动窗口的大小 k，请找出所有滑动窗口里的最大值。
> 滑动窗口大概长这样：
> 1 [3 -1 -3] 5 3 6 7
> 1 3 [-1 -3 5] 3 6 7
> 1 3 -1 [-3 5 3] 6 7
> 1 3 -1 -3 [5 3 6] 7
> 1 3 -1 -3 5 [3 6 7]

基础思路是用一个双指针分别指向窗口的左和右，然后取得双指针之间的最大值放入结果.但是这个方法不好，因为实际上有很多已经判断过的数被重复比较

因此更好的思路是利用**单调递减**的**双端队列**，遍历把每个 nums 中的索引放入（还是索引，因为需要判断队列中的数字是否超出滑动窗口范围）

然后最重要的是和上面的最小栈类似，如果有比队尾数小的就放入；如果比队尾数大就依次出队，直到`队尾数>=当前数`为止。因此队头的元素一定是本次遍历的最大值

遍历时从第一个开始，当遍历数到达 k 时，就相当于产生了第一个窗口，开始记录最大值。

```js
const maxSlidingWindow = function (nums, k) {
  const deque = [];
  const answer = [];
  for (let i = 0; i < nums.length; i++) {
    // 如果当前数比队尾数大，就依次出队
    while (deque.length && nums[i] > nums[deque[deque.length - 1]]) {
      deque.pop();
    }
    deque.push(i); // 当前数的索引放入

    // 判断队头元素的位置是否超过了滑动窗口的前沿（即 i + 1 - k）
    if (deque.length && deque[0] < i + 1 - k) {
      deque.shift();
    }

    //当遍历数到达k时，开始记录最大值
    if (i >= k - 1) {
      answer.push(nums[deque[0]]);
    }
  }
  return answer;
};
```

### 子数组的最小值之和（单调栈）

https://leetcode.cn/problems/sum-of-subarray-minimums/

> 给定一个整数数组 arr，找到 min(b)  的总和，其中 b 的范围为 arr 的每个（连续）子数组。
> 由于答案可能很大，因此 返回答案模 10^9 + 7 。 
> 示例 1：
> 输入：arr = [3,1,2,4]
> 输出：17
> 解释：
> 子数组为 [3]，[1]，[2]，[4]，[3,1]，[1,2]，[2,4]，[3,1,2]，[1,2,4]，[3,1,2,4]。
> 最小值为 3，1，2，4，1，1，2，1，1，1，和为 17。

这道题其实是一道很好的题目，也提供了一个很好的思路；
关于子数组性质的题目有不少，类似的解决方法也有动态规划、单调栈、滑动窗口、前缀和、差分数组等几种常见的解法。
其中单调栈是一种不是很能看出来的方法，因为单调栈在这里并不是核心思路，只是更方便计算。

这道题的核心思路，在于**找出最小值的辐射范围**。什么是辐射范围？即一个数在多大范围的数组形成的子数组中都能作为最小值。
比如示例 1 的数字 1，任何子数组只要有它，它就是最小值，因此它的辐射范围就是区间`[0,3]`；其他几个数的辐射范围分别是：

```
3  [0,0]   即只有第一个数
1  [0,3]
2  [2,3]   即2和4两个数
4  [3,3]   即最后一个数
```

那么该数的值乘它的辐射范围，就得到了该值对于整个和的**贡献**。最后把每个数的贡献加在一起，就是最后的和。

所以关键就在于怎么这个辐射范围。最简单粗暴的方法当然是用双指针，从当前数的位置出发，向左向右找到**第一个小于当前数的位置**，就是该数的辐射范围。注意这里就出现了关键，即，找到每个数最近小于它的数（这里看起来就很像每日温度），有这样类似的问题，就可以采取单调栈。

我们设两个栈分别用于存储每个数的左边第一个小于的位置和右边第一个小于的位置，即每个数的左边界和右边界；

- 右边界的计算：正向遍历数组，设置一个递增栈；如果当前数比栈顶元素小，就依次出栈，出栈元素的“第一个最小元素位置”就是当前数的位置。
- 左边界的计算：反向遍历数组，同样是递增栈，操作和右边界一样。

另外，在单调栈中有一个等于的情况，即当前元素和栈顶元素相等；
例如 `5 8 5`这种情况出现时，在正序单增中，我们如果把`5 5`都放入的话，那么对于第一个 5 来说，第二个 5 不是边界，可以继续往后走找到下一个更小的。如果在倒序也这样，就会出现`5 8 5`这一个子序列的贡献次数被记录两次的情况。所以我们只需要在正序记录，倒叙不记录。

代码：

```js
var sumSubarrayMins = function (arr) {
  const stack = [];
  const left = [];
  const right = [];

  // 左边界
  for (let i = arr.length - 1; i >= 0; i--) {
    // 递增栈，如果当前元素比栈顶小或等于就出栈
    while (stack.length && arr[i] <= arr[stack[stack.length - 1]]) {
      const top = stack.pop();
      left[top] = i;
    }
    stack.push(i);
  }
  // 没有找到的就看作-1
  while (stack.length) {
    left[stack.pop()] = -1;
  }

  // 右边界
  for (let i = 0; i < arr.length; i++) {
    // 注意这里没有等于.其实左右边界有一个有等于就行，主要是为了考虑到等于的情况
    while (stack.length && arr[i] < arr[stack[stack.length - 1]]) {
      const top = stack.pop();
      right[top] = i;
    }
    stack.push(i);
  }
  while (stack.length) {
    right[stack.pop()] = arr.length;
  }

  console.log(left, right);
  let sum = 0;
  for (let i = 0; i < arr.length; i++) {
    sum += arr[i] * (i - left[i]) * (right[i] - i);
  }
  return sum % (10 ** 9 + 7);
};
```

## 字符串解码

https://leetcode.cn/problems/decode-string/

> 给定一个经过编码的字符串，返回它解码后的字符串。
> 编码规则为: k[encoded_string]，表示其中方括号内部的 encoded_string 正好重复 k 次。注意 k 保证为正整数。
> 你可以认为输入字符串总是有效的；输入字符串中没有额外的空格，且输入的方括号总是符合格式要求的。
> 此外，你可以认为原始数据不包含数字，所有的数字只表示重复的次数 k ，例如不会出现像  3a  或  2[4]  的输入。
> 示例 1：
> 输入：s = "3[a]2[bc]"
> 输出："aaabcbc"

思路：利用栈，逐个遍历字符串的每一项：

- 如果是数字、左括号或者字母，入栈
- 如果是右括号，就出栈直到左括号为止，然后把之间的字母按照左括号前的数字作为倍数放入结果。

```js
/**
 * @param {string} s
 * @return {string}
 */
var decodeString = function (s) {
  const stack = [];

  const isLetter = (s) =>
    s.charCodeAt() >= "a".charCodeAt() && s.charCodeAt() <= "z".charCodeAt();
  for (let i = 0; i < s.length; ) {
    const res = []; // 每次执行res都应该清零，防止影响上一次的结果
    if (!isNaN(s[i])) {
      // 如果是数字，处理不止一位数的情况
      let num = "";
      while (!isNaN(s[i])) num += s[i++];
      stack.push(+num);
    } else if (s[i] === "[" || isLetter(s[i])) {
      // 左括号和字母入栈
      stack.push(s[i]);
      i++;
    } else if (s[i] === "]") {
      let str = "";
      while (true) {
        // 出栈到右括号
        const top = stack.pop();
        if (top === "[") break;
        str = top + str;
      }
      const times = stack.pop();
      for (let j = 0; j < times; j++) {
        // 把中间的字符按照倍数放入
        res.push(str);
      }
      stack.push(res.join("")); // 注意，前面拼接好的还应该放回栈中，主要为了处理嵌套括号的情况。
      i++;
    }
  }
  if (stack.length) {
    // 剩余字母
    for (let i = 0; i < stack.length; i++) res.push(stack[i]);
  }
  return res.join("");
};
```

## 栈实现队列和队列实现栈

> 用栈实现队列，只能使用栈的 push 和 pop 方法。

思路是用两个栈存储，正常的入队就是 push 到 stack1；但是每次出队时，将 stack1 中的元素依次出栈再入栈到 stack2 中，这样栈底元素就被取出了；此后再次入队仍然是到 stack1 中

注意每次出队操作并不是都要把stack1全部移到stack2中，而是要先判断stack2是否为空，如果为空才会移动。如果stack2中还有值，那么出队操作就是直接pop stack2即可。

```js
class MyQueue {
  constructor() {
    this.stack1 = [];
    this.stack2 = [];
  }
  enqueue(val) {
    this.stack1.push(val);
  }
  dequeue() {
    // 如果stack2为空才把stack1全部放入stack2，否则就只是stack2.pop
    if (!this.stack2.length) {
      for (let i = 0; i < this.stack1.length - 1; i++) {
        this.stack2.push(this.stack1.pop());
      }
    }
    return this.stack2.pop();
  }
  peek() {
    if (!this.stack2.length) {
      for (let i = 0; i < this.stack1.length; i++) {
        this.stack2.push(this.stack1.pop());
      }
    }
    return this.stack2[this.stack2.length - 1];
  }
  size() {
    return this.stack1.length + this.stack2.length;
  }
}
```

---

用队列实现栈，也是类似的思路。要求队列只能使用 push、shift、top（返回栈顶元素）和 empty 四种方法，实现一个栈。整体思路不难，但是细节有不少

还是创建两个队列 q1 和 q2，每个操作的逻辑如下：

- push：直接向 q1 push。但是有一个特殊情况，就是下面的 pop 操作把 q1 清空后，即 q1 为空时就向 q2 push
- pop：根据 q1 和 q2 是否为空来判断：
  - 如果 q1 不空且 q2 为空，那就把所有 q1 的元素依次 shift 并 push 到 q2 中。这样最后一个元素就是“栈顶”元素，返回这个元素即可。这一步执行完后 q1 就变成空的
  - 如果 q1 为空且 q2 不空，和上一步同理，把 q2 依次 shift 并 push 到 q1 中。也就是说 pop 的过程就是两个队列相互倒值的过程，这样才能取出队列的最后一个数，也就说栈顶元素。这个过程中 q1 和 q2 必然有一个是空的，如果不是就说明栈中没有值或者出错了。
- top：返回非空的那个队列的最后一项就可以。如果必须用标准操作，就调用 pop 得到值，然后再放回非空的那个队列中就可以。

代码如下：

```js
class MyStack {
  constructor() {
    this.queue1 = [];
    this.queue2 = [];
  }
  push(val) {
    if (!this.queue1.length && this.queue2.length) this.queue2.push(val);
    else this.queue1.push(val);
  }
  pop() {
    if (this.queue1.length && !this.queue2.length) {
      if (this.queue1.length < 2) return this.queue1.shift();
      const len = this.queue1.length;
      for (let i = 0; i < len - 1; i++) {
        const top = this.queue1.shift();
        this.queue2.push(top);
      }
      return this.queue1.shift();
    } else if (this.queue2.length && !this.queue1.length) {
      if (this.queue2.length < 2) return this.queue2.shift();
      const len = this.queue2.length;
      for (let i = 0; i < len - 1; i++) {
        const top = this.queue2.shift();
        this.queue1.push(top);
      }
      return this.queue2.shift();
    } else return null;
  }
  top() {
    const res = this.pop();
    if (!this.queue1.length && this.queue2.length) this.queue2.push(res);
    else this.queue1.push(res);
    return res;
  }
  empty() {
    return this.queue1.length + this.queue2.length === 0;
  }
}
```

## 验证栈序列

https://leetcode.cn/problems/validate-stack-sequences/

> 给定  pushed  和  popped  两个序列，每个序列中的 值都不重复，只有当它们可能是在最初空栈上进行的推入 push 和弹出 pop 操作序列的结果时，返回 true；否则，返回 false 。
> 示例 1：
> 输入：pushed = [1,2,3,4,5], popped = [4,5,3,2,1]
> 输出：true
> 解释：我们可以按以下顺序执行：
> push(1), push(2), push(3), push(4), pop() -> 4,
> push(5), pop() -> 5, pop() -> 3, pop() -> 2, pop() -> 1

这道题主要思路是按照这两个序列去模拟栈的 push 和 pop。
首先遍历 pushed 数组，按照 pushed 数组的顺序去向栈中压入数字；在这个过程中不断检查栈顶元素和 poped 数组中元素的关系，如果相同就弹出栈顶元素，并且把 popped 数组向后一个（这个过程应该是个循环）。最后如果栈是空的，就说明可以满足。

```js
var validateStackSequences = function (pushed, popped) {
  const stack = [];
  let i = 0;
  let j = 0;
  while (i < pushed.length) {
    stack.push(pushed[i]);
    while (stack[stack.length - 1] === popped[j] && stack.length) {
      // 注意这里应该是个循环，所有和当前popped数组中对应的栈顶元素都该逐个出栈
      stack.pop();
      j++;
    }
    i++;
  }
  return stack.length === 0;
};
```

# 滑动窗口

## 基本思路

基本思路是这样的：

1. 循环，然后逐个将字符放入窗口中
2. **根据窗口本身的状态**，判断是否需要弹出窗口（缩小左边），然后循环弹出直到窗口本身满足状态
3. 结束循环的条件通常是一个指针走到了其中一个序列的末尾。

通常的格式：

```js
function slidingWindow(s,t){
  const queue = []
  for(let i = 0;i < s.length;i++){
    queue.push(s[i])// 增大窗口通常是在循环中直接插入的，不需要单独一个循环
    // 一些更新
    while(/*弹出的条件，通常是窗口内部的状态*/){
      const out = queue.shift()
      // 一些更新
    }
  }
}
```

可以看到其实问题的关键就是判断窗口里边部分的状态是否满足结束条件或者弹出条件。
以字符串为例，常用的判断方式有：

1. 用两个对象分别统计当前窗口中的字符数量 window 和目标的字符数量 need，如果匹配就记录，其他时候弹出直到不匹配为止；
2. 直接判断窗口内和目标是否相等（字符串或数组）
3. 记录一个 a-z 26 个字符的出现次数的数组，**如果两个字符串之间除了顺序之外完全相同，那么这两个数组也一定是完全相同的**，用于判断顺序不一致时的子序列。

## 字符串的排列

https://leetcode.cn/problems/permutation-in-string/

> 给你两个字符串 s1 和 s2 ，写一个函数来判断 s2 是否包含 s1 的排列。如果是，返回 true ；否则，返回 false 。
> 示例 1：
> 输入：s1 = "ab" s2 = "eidbaooo"
> 输出：true
> 解释：s2 包含 s1 的排列之一 ("ba").

这道题的思路是这样：首先确定一个滑动窗口，然后逐个将字符放入窗口中，（当滑动窗口大小够的时候）比较窗口的字符是否满足条件（即判断窗口内的字符串和目标字符串 s1 是否相等）。如果相等就直接返回 true，不相等就把窗口中的第一个字符弹出（保证窗口大小始终和 s1 大小相等），然后继续添加下一个并判断。

但是这个问题在于判断字符相等，因为求得是字符串的**排列**而非子序列，因此它是连续但并不按顺序的，所以不能直接比较判断字符串相等（太耗时）。
这里提供了一个思路是记录 a-z 的 26 个字符的出现次数的数组，**如果两个字符串之间除了顺序之外完全相同，那么这两个数组也一定是完全相同的**。因此就可以在每次遍历中当前字符++，并把超出窗口大小的字符--即可。

注意如果采用这个方法，就不需要再创建一个queue了。说到底滑动窗口其实是一种思想，队列并不是必须的结构。

```js
/**
 * @param {string} s1
 * @param {string} s2
 * @return {boolean}
 */
var checkInclusion = function (s1, s2) {
  const target = new Array(26).fill(0);
  const now = new Array(26).fill(0);
  for (let s of s1) {
    target[s.charCodeAt() - "a".charCodeAt()]++;
  }
  const isEqual = (a, b) => {
    for (let i = 0; i < a.length; i++) {
      if (a[i] !== b[i]) return false;
    }
    return true;
  };
  let i = 0;
  while (i < s2.length) {
    now[s2[i].charCodeAt() - "a".charCodeAt()]++;
    if (i >= s1.length - 1) {
      if (isEqual(now, target)) return true;
      else {
        now[s2[i - s1.length + 1].charCodeAt() - "a".charCodeAt()]--;
      }
    }
    i++;
  }
  return false;
};
```

这道题还有可能的几个变种：

1. s1 就是 s2 的子串，不会变顺序：在判断阶段直接比较窗口的字符串和目标串即可，比较简单.(当然这个实际上就是 KMP 算法了)
2. s1 是 s2 的子序列，即不一定连续：用一个 now 对象（表示当前字符数量，按照`{A:1,B:2,C:0}`这种形式）表示当前记录次数，用同样的 target 对象表示目标，窗口每次增大都判断是否满足条件，满足就为 true；
3. 判断 s1 在 s2 中出现的最短序列（https://leetcode.cn/problems/minimum-window-substring/）：最难的情况，要计算最短情况，建议直接看解析。

## 无重复字符的最长子串

> 给定一个字符串 s ，请你找出其中不含有重复字符的 最长子串 的长度。

同样的滑动窗口题，弹出条件是窗口有重复字符，把窗口左边的循环弹出直到不重复为止。

```js
/**
 * @param {string} s
 * @return {number}
 */
var lengthOfLongestSubstring = function (s) {
  let res = 0;
  const queue = [];
  const isRepeat = (arr) => {
    return [...new Set([...arr]).values()].length !== arr.length;
  };
  for (let i = 0; i < s.length; i++) {
    queue.push(s[i]);
    while (isRepeat(queue)) {
      queue.shift();
    }
    res = Math.max(res, queue.length);
  }
  return res;
};
```

## 找到字符串中所有字母异位词

https://leetcode.cn/problems/find-all-anagrams-in-a-string/

![](https://pic.imgdb.cn/item/628bc6f00947543129baa39b.jpg)

这个题的思路和前面的差不多：滑动窗口大小固定，然后每次增加一个并弹出一个，判断内部的字符是否满足

```js
/**
 * @param {string} s
 * @param {string} p
 * @return {number[]}
 */
var findAnagrams = function (s, p) {
  const index = [];
  const len = p.length;
  const A = "a".charCodeAt();
  const window = new Array(26).fill(0);
  const need = new Array(26).fill(0);

  for (let s of p) {
    need[s.charCodeAt() - A]++;
  }
  const isEqual = (arr1, arr2) => arr1.join("") === arr2.join("");

  for (let i = 0; i < s.length; i++) {
    window[s[i].charCodeAt() - A]++;
    if (i >= len - 1) {
      if (isEqual(window, need)) index.push(i - len + 1);

      window[s[i - len + 1].charCodeAt() - A]--;
    }
  }
  return index;
};
```

# 二分查找

## 基本思路

二分查找问题的基本框架：

```js
function binarySearch(nums,target){
  let left = 0
  let right = nums.length - 1

  while(.../*通常是left <= right*/){
    const mid = left + Math.floor((right - left) / 2)

    if(nums[mid] ===  target){
      return ...
    }else if(nums[mid] > target){
      right = ... // 这里通常是mid - 1，相当于把右边界变为中值的左边一个数
    }else if(nums[mid] < target){
      left = ... // 通常是mid + 1
    }
  }
}
```

二分查找也可以写成递归形式，但是一般不需要
二分查找的关键就是几个细节：

1. while 循环的边界。因为这个边界表示的是没有查找到之后的退出条件，因此要保证`right - left`区域内的空间为 0，即当 left>=right 时退出。

- `left <= right` ，对应`[left, right]`的左右都闭区间，因此 right 的初始值应该是`nums.length-1`（right 要取到具体的值）。这个时候的退出条件就是`left>right`，即完全没有交集
- `left < right`，对应`[left,right)`的左闭右开区间，这时是不取 right 的值的，因此 right 应该是`nums.length`。所以当`left===right`就可以退出了，终止的条件是 left == right，此时搜索区间 `[left, left)` 为空

大多数情况下都选择第一种，方便理解。

2. mid+1 和 mid-1，这两个情况都是因为 mid 已经排除了，因此自然不需要 mid 这一项。除非有一个特殊情况，就是需要左右边界的数，可以继续取 mid 表示边界位置。

## 二分查找的边界

对于有重复数字的二分查找，需要找到其最左边或者最右边的值；

> 给定一个按照升序排列的整数数组 nums，和一个目标值 target。找出给定目标值在数组中的开始位置和结束位置。
> 如果数组中不存在目标值 target，返回  [-1, -1]。

一个方法是，先找到目标数，然后循环找到其左右边界并返回。

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var searchRange = function (nums, target) {
  let left = 0;
  let right = nums.length - 1;

  const res = [-1, -1];
  while (left <= right) {
    const mid = left + Math.floor((right - left) / 2);
    if (nums[mid] === target) {
      let index = mid;
      while (nums[index] === target) {
        res[0] = index--;
      }
      index = mid;
      while (nums[index] === target) {
        res[1] = index++;
      }
      break;
    } else if (nums[mid] > target) {
      right = mid - 1;
    } else if (nums[mid] < target) {
      left = mid + 1;
    }
  }
  return res;
};
```

但是这种方法不能保证二分查找的对数时间复杂度。并且如果特殊的测试用例下，两个边界离得特别远，就会趋近于线性遍历的复杂度。

因此更合适的方法，还是通过控制左右边界实现。

基本思路：

```js
if (nums.length == 0) return -1;
let left = 0;
let right = nums.length - 1;

while (left <= right) {
  let mid = left + Math.floor((right - left) / 2);
  if (nums[mid] === target) {
    right = mid - 1;
    // 查找右边界的话
    // left = mid + 1
  } else if (nums[mid] < target) {
    left = mid + 1;
  } else if (nums[mid] > target) {
    right = mid - 1;
  }
}
return left;
```

找到 target 时不要立即返回，而是缩小「搜索区间」的上界 right，在区间 `[left, mid)` 中继续搜索。又因为左闭右开的特点，找到的位置 mid 实际上会被排除；那么剩下的查找无论怎么找都一定是小于等于 target 的，因此就会一直向左走直到左边界。向右的话同理

## 求 x 的平方根

https://leetcode.cn/problems/sqrtx/

> 给你一个非负整数 x ，计算并返回 x 的 算术平方根 。由于返回类型是整数，结果只保留 整数部分 ，小数部分将被 舍去 。

这道题是二分查找的最基本应用。本身过程不算难，但是要想到用二分查找就比较难。
其实对于这种**在一个范围内找一个符合条件的值**的这类题，都可以考虑用二分查找，关键就是在于改变 left 和 right 时的条件。比如这道题就是计算 mid 的平方和 target 的关系改变 left 和 right，下面的送包裹和吃香蕉的题则是计算花费时间和目标时间的关系。

```js
/**
 * @param {number} x
 * @return {number}
 */
var mySqrt = function (x) {
    if(x === 0 || x === 1) return x
    let left = 0;
    let right = x;
    // 本质上是在求第一个大于target的数，然后减一即可
    // target = floor(sqrt(x))
    while (left < right) {
        const mid = (left + right) >> 1;
        if (mid * mid <= x) left = mid + 1
        else right = mid
    }
    return left - 1;
};

// 还有一种方法
var mySqrt = function (x) {
  let left = 0;
  let right = x;
  // 下面不取left = mid - 1，所以当left和right重合时就应该退出，否则会死循环
  while (left < right) {
    // mid每次要加1，主要是解决范围内只有left和right两个数的的情况
    // 这种情况下，mid计算出来一定是left，然后下面又把left设为mid，这样就会死循环
    // 解决方法就是把计算出来的mid+1，这样只有两个数时就会取到right，跳出死循环
    const mid = left + Math.floor((right - left) / 2) + 1;
    if (mid ** 2 <= x) {
      // mid不能直接抛弃，因为mid ** 2 < x时也有可能是结果
      left = mid;
    } else {
      right = mid - 1;
    }
  }
  return left;
};
```

## 寻找峰值

https://leetcode.cn/problems/find-peak-element/

> 峰值元素是指其值严格大于左右相邻值的元素。
> 给你一个整数数组  nums，找到峰值元素并返回其索引。数组可能包含多个峰值，在这种情况下，返回 任何一个峰值 所在位置即可。
> 你可以假设  nums[-1] = nums[n] = -∞ 。
> 你必须实现时间复杂度为 O(log n) 的算法来解决此问题。
> 示例 1：
> 输入：nums = [1,2,3,1]
> 输出：2
> 解释：3 是峰值元素，你的函数应该返回其索引 2。

思路：基本解法很简单，但是这道题其实可以用二分查找优化。
因为只需要找出一个峰值就可以，所以可以只找一边。首先先计算一个中间值 mid

- 如果 左边 > 中间值 > 右边 ，那就在左边查找，即相当于取左边的值为新的 right
- 如果 左边 < 中间值 < 右边，在右边查找
- 如果 左边 >= 中间 <= 右边，即中间是“谷底”，那么向右向左都可以，这里就取向右
- 如果 左 < 中间 > 右，说明找到了。


```js
/**
 * @param {number[]} nums
 * @return {number}
 */
var findPeakElement = function (nums) {
  if (nums.length === 1) return 0;
  const len = nums.length;
  let left = 0;
  let right = nums.length - 1;
  while (left <= right) {
    const mid = left + Math.floor((right - left) / 2);
    if (mid === 0) nums[mid - 1] = -Infinity;
    if (mid === len - 1) nums[len] = -Infinity;
    if (nums[mid] > nums[mid - 1] && nums[mid] > nums[mid + 1]) {
      return mid;
    } else if (nums[mid] > nums[mid - 1] && nums[mid] < nums[mid + 1]) {
      left = mid + 1;
    } else if (nums[mid] < nums[mid - 1] && nums[mid] > nums[mid + 1]) {
      right = mid - 1;
    } else if (nums[mid] <= nums[mid - 1] && nums[mid] <= nums[mid + 1]) {
      left = mid + 1;
    }
  }
};
```

## 搜索旋转排序数组

> 整数数组 nums 按升序排列，数组中的值 互不相同 。
> 在传递给函数之前，nums 在预先未知的某个下标 k（0 <= k < nums.length）上进行了 旋转，使数组变为 `[nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]`（下标 从 0 开始 计数）。例如， [0,1,2,4,5,6,7] 在下标 3 处经旋转后可能变为  [4,5,6,7,0,1,2] 。
> 给你 旋转后 的数组 nums 和一个整数 target ，如果 nums 中存在这个目标值 target ，则返回它的下标，否则返回  -1 。
> 你必须设计一个时间复杂度为 O(log n) 的算法解决此问题。
> 示例 1：
> 输入：nums = [4,5,6,7,0,1,2], target = 0
> 输出：4

这道题 O(log n)肯定是二分查找了。但是这里的特点是，数组有一部分被改变位置了，并不是完全有序的数组。

二分查找的核心是有序数组，而这道题实际上是部分有序的。因此我们需要在这个部分有序的数组中找到有序的那部分查找，也就相当于是从这个数组中“截出来”一段再去查找。

思路如下：

首先还是计算 mid，计算出来的 mid 和两边有两种情况：

- 左边有序，右边无序，判断方式是右边的最后一个元素 nums[nums.length-1]是不是小于 mid，如果小于就说明一定是右边的部分乱序
  - 这种情况下，如果`target < mid && target > nums[0]`，即刚好在左边**并且不会比最左的元素小**，就直接`right = mid - 1`
    注意这里`target > nums[0]`很重要，因为如果一个数即使比 mid 小，但它比最左边的还要小，那就不能在左边找了。右边也是同理
  - 如果`target > mid`，说明在乱序的右边，则`left = mid+1`，然后再执行一遍上面的步骤
- 右边有序，左边无序，同理判断 nums[0]和 mid 大小，如果前者大于后者就说明无序
  - 处理和上面同理

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number}
 */
var search = function (nums, target) {
  const len = nums.length;
  if (len === 1 || len === 2) return nums.indexOf(target);
  let left = 0;
  let right = len - 1;
  while (left <= right) {
    const mid = Math.floor(left + (right - left) / 2);
    if (nums[mid] === target) return mid;
    if (nums[0] <= nums[mid]) {
      // 左边顺序
      if (target >= nums[0] && target < nums[mid]) {
        // 注意这里target可能会和第一项相等
        right = mid - 1;
      } else {
        left = mid + 1;
      }
    } else {
      // 左边乱序
      if (target <= nums[nums.length - 1] && target > nums[mid]) {
        // 同理也可能是最后一项
        left = mid + 1;
      } else {
        right = mid - 1;
      }
    }
  }
  return -1;
};
```

---

这道题有一个扩展，就是数组中包含重复数字的情况，如果有重复就需要返回最小索引

基础逻辑不变，在基础上增加一下新的判断：

- 比较nums[0] 和 nums[mid]，大于和小于的情况不变，但是如果等于，就让left++。这个原理和下面搜索最小值的重复情况的right++类似，因为相等时不能确定mid到底是在哪个部分，因此我们让left++，逐渐缩小范围，直到mid确定范围为止
- 每次循环时，如果`nums[left] === target`，就直接返回。这种主要是考虑到第一个元素就是target的情况，由于可能有重复数字，因此第一个元素不会被正常计算。

比如`[5,5,5,1,2,3,4,5]`这个数组，如果不增加这个逻辑，那么最后得出的5应该是最后一个位置的
这个比较难想到，但是如果类似的题目出现这个问题，可以尝试加上这个逻辑。

- 如果target和nums[mid]相等，那么循环找到左边界。这个就和查找重复数字一样

代码如下：

```js
var search = function (arr, target) {
    let left = 0
    let right = arr.length - 1
    while (left <= right) {
        const mid = (left + right) >> 1
        if(arr[left] === target) return left
        if (arr[mid] === target) {
            let i = mid
            while (arr[i] === target) {
                i--
            }
            return i + 1
        }
        if (arr[mid] > arr[0]) {
            if (target >= arr[0] && target < arr[mid]) {
                right = mid - 1
            } else {
                left = mid + 1
            }
        } else if (arr[mid] < arr[0]) {
            if (target <= arr[arr.length - 1] && target > arr[mid]) {
                left = mid + 1
            } else {
                right = mid - 1
            }
        } else {
            left++
        }
    }
    return -1
};
```



## 搜索旋转排序数组的最小值

这道题有一个基本版和一点点变种，后者和前者的区别只是数组中添加了重复元素：
https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array/
https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array-ii/

> 已知一个长度为 n 的数组，预先按照升序排列，经由 1 到 n 次 旋转 后，得到输入数组。例如，原数组 nums = [0,1,2,4,5,6,7] 在变化后可能得到：
> 若旋转 4 次，则可以得到 [4,5,6,7,0,1,2]
> 若旋转 7 次，则可以得到 [0,1,2,4,5,6,7]
> 注意，数组 [a[0], a[1], a[2], ..., a[n-1]] 旋转一次 的结果为数组 [a[n-1], a[0], a[1], a[2], ..., a[n-2]] 。
> 给你一个元素值 互不相同 的数组 nums ，它原来是一个升序排列的数组，并按上述情形进行了多次旋转。请你找出并返回数组中的 最小元素 。
> 你必须设计一个时间复杂度为  O(log n) 的算法解决此问题。

两个题基本思路差不多，先说第一道题的。
这道题和上面的搜索排序数组是一类，但是要求的是找最小值而不是目标值，所以有一点不一样就是不能根据 target 和 mid 的大小关系调整 mid，而是需要和左右边界的数比较。

具体来说，mid 和 left 和 right 可能会有四种情况：
![](https://pic1.imgdb.cn/item/63368b9c16f2c2beb1577753.jpg)

其中第二种应该算是找到了（同时小于左右一定就是刚好分界的那个数），第四种不可能，因此只有两种需要判断的情况。
而这两种情况恰好就是 mid 和 right 的关系。也就是说在这道题中只需要判断 mid 和 right 的关系再去调整 mid 即可。

> 为什么不需要 mid 和 left 判断？因为 mid 和 left 不能得出最小值
> 比如 nums[mid] > nums[left]，这时 left 和 mid 可能都在左半边或者右半边，不能确定最小值到底在左还是在右，还需要借助 mid 和 right 判断
> 如果需要借助 mid 和 right 的话，就会发现 left 在两种情况下取值都相同（即 nums[mid] > nums[left]），所以就可以无视掉直接判断右边就好。

有了这两种情况就可以得出应该缩小的范围：

- mid > right：最小值在右边，应该缩小 left
- mid < right：最小值在左边，缩小 right

另外这里还要注意第二种情况的 mid 取值，即`right = mid`而不是`right = mid - 1`。这个也是和这道题有关系的，假如这样一种情况：

```
[4, 5, 6, 0, 1, 2, 3]

nums[mid] = 0 < nums[right]
```

此时如果直接让 right = mid - 1，就显然会直接跳过已经找到了的最小值。因此正好可以采取左闭右开的区间`while(left < right)`，但是注意 right 初始化应该是`nums[nums.length - 1]`，因为每次比较都需要 right 位置有具体的值，但是又不需要这个位置的值做最终结果。

代码：

```js
var minArray = function (numbers) {
  let left = 0;
  let right = numbers.length - 1;

  while (left < right) {
    const mid = left + Math.floor((right - left) / 2);
    if (numbers[mid] > numbers[right]) {
      left = mid + 1;
    } else if (numbers[mid] < numbers[right]) {
      right = mid;
    }
  }
  return numbers[left];
};
```

---

第二题的解法上只是多了判断`numbers[mid] === numbers[right]`的情况。因为存在重复数字，因此当相等时不能判断最小值到底在 mid 的左边还是右边：
![](https://pic1.imgdb.cn/item/633690df16f2c2beb15ce878.jpg)
但是因为 mid 和 right 相等，也就是说至少有一个 right 左边、并且和 right 值相等的位置，因此给 right--，这样 `nums[right]`的值没有改变但是还是缩小了范围。

```js
var minArray = function (numbers) {
  let left = 0;
  let right = numbers.length - 1;

  while (left < right) {
    const mid = left + Math.floor((right - left) / 2);
    if (numbers[mid] > numbers[right]) {
      left = mid + 1;
    } else if (numbers[mid] < numbers[right]) {
      right = mid;
    } else {
      right--;
    }
  }
  return numbers[left];
};
```

## 按权重随机选择

https://leetcode.cn/problems/random-pick-with-weight/

> 给你一个 下标从 0 开始 的正整数数组  w ，其中  w[i] 代表第 i 个下标的权重。
> 请你实现一个函数  pickIndex ，它可以 随机地 从范围 [0, w.length - 1] 内（含 0 和 w.length - 1）选出并返回一个下标。选取下标 i  的 概率 为 w[i] / sum(w) 。
> 例如，对于 w = [1, 3]，挑选下标 0 的概率为 1 / (1 + 3) = 0.25 （即，25%），而选取下标 1 的概率为 3 / (1 + 3) = 0.75（即，75%）。

这道题本质上是一个前缀和+二分查找的问题，具体解析可以参考https://labuladong.github.io/algo/2/19/29/

最开始的思路是创建一个数组，按照每个元素的大小控制其出现次数。比如当前值是 5，那就在新数组中出现 5 次。总的数组大小就是所有数之和。
但是当和非常大时就会出问题。

因此可以秉持这种思路，但是不采用具体生成数组的方法，而是用前缀和模拟一个区间

![](https://pic.imgdb.cn/item/62d53f65f54cd3f93739f99e.jpg)

这样在区间内任意选取一个随机数，再通过**二分查找左边界**的形式找到最近的对应的前缀和（相当于落在了这个范围内），就可以对应到相应的数字上去。

其中有几个关键点：

- 随机数的选取范围：应该是`random * sum + 1`，即范围从`[0-n]`变为`[1-n+1]`。之所以要加 1，主要是选取时考虑具体的点而不是区间，选取的概率分布也是在点上而不是区间上。比如说 w=[1,3,2,1]，那么 7 = （1 + 3 + 2 + 1）恰好是 7 个数，即 1-7，如果是 0-7 多了个 0 就不对了。
- 二分查找的使用：不能直接使用二分查找找，因为随机数大概率不会匹配到具体的值，而只是匹配一个范围。因此需要的是返回左边界的二分查找。至于为什么是左边界，可以参考下面的解释：
  ![](https://pic.imgdb.cn/item/62d54130f54cd3f9373c4726.jpg)

代码如下：

```js
class Solution {
  constructor(w) {
    this.w = w;
  }

  pickIndex() {
    const prefix = [this.w[0]];
    for (let i = 1; i < this.w.length; i++) {
      prefix[i] = prefix[i - 1] + this.w[i];
    }
    // 注意随机数+1
    // 原因是这样：比如说w=[1,3,2,1]，那么7 = （1 + 3 + 2 + 1）恰好是7个数，即1-7，如果是0-7多了个0就不对了。
    // 也就是说应该考虑具体的点而不是区间，选取的概率分布也是在点上而不是区间上。
    const random = Math.floor(Math.random() * prefix[prefix.length - 1]) + 1;

    let left = 0;
    let right = prefix.length - 1;
    while (left <= right) {
      const mid = left + Math.floor((right - left) / 2);
      if (prefix[mid] < random) {
        left = mid + 1;
      } else if (prefix[mid] > random) {
        right = mid - 1;
      } else {
        return mid;
      }
      return left;
    }
  }
}
```

## 在 D 天内送达包裹的能力

https://leetcode.cn/problems/capacity-to-ship-packages-within-d-days/

> 传送带上的包裹必须在 days 天内从一个港口运送到另一个港口。
> 传送带上的第 i  个包裹的重量为  weights[i]。每一天，我们都会按给出重量（weights）的顺序往传送带上装载包裹。我们装载的重量不会超过船的最大运载重量。
> 返回能在 days 天内将传送带上的所有包裹送达的船的最低运载能力。
> 示例 1：
>
> 输入：weights = [1,2,3,4,5,6,7,8,9,10], days = 5
> 输出：15
> 解释：
> 船舶最低载重 15 就能够在 5 天内送达所有包裹，如下所示：
> 第 1 天：1, 2, 3, 4, 5
> 第 2 天：6, 7
> 第 3 天：8
> 第 4 天：9
> 第 5 天：10
>
> 请注意，货物必须按照给定的顺序装运，因此使用载重能力为 14 的船舶并将包装分成 (2, 3, 4, 5), (1, 6, 7), (8), (9), (10) 是不允许的。

这道题看起来和二分查找没有任何关系。但是这类问题经过分析可以发现，基本满足一个大概的趋势：能形成单调的、有平台的函数，即大概是这个样子：
![](https://pic.imgdb.cn/item/62d574c7f54cd3f9377a1b2c.jpg)

而对于这道题，我们设横坐标表示单次运载能力 x，纵坐标表示需要运送的天数 y，那显然这两者成反比例。并且，由于不是连续的，所以一定会出现多个 x 对应一个 y 的情况，也就是“平台”

![](https://pic.imgdb.cn/item/62d5753ff54cd3f9377ac173.jpg)

看到这个平台，就应该想到要用二分查找的左边界或右边界去求最大/最小值了。

换句话说，这道题其实是在“搜索”一个值（船的运载能力），这个值能够满足条件（即满足限定的时间内把所有货物运走），并且不只是能满足，还得是最小的。那么就可以先找到一个能满足的，然后按照二分查找的左边界，找到最小能满足的那个值，就是结果。

因此代码主要分为两部分：

1. 二分查找部分，先计算出一个值作为运载能力，然后计算这个值下的需要的天数。如果天数多于限定的，说明这个值小了，应该改变左边界；反之亦然
2. 计算所需天数部分，根据上面二分查找的值计算所需天数

其中，计算天数的逻辑如下：
遍历所有包裹的重量，依次相加。如果超重，就需要延后一天（超重的这个包裹不能算上），即天数+1；然后把当天的包裹重量和清零，再继续计算。

代码如下：

```js
var shipWithinDays = function (weights, days) {
  let left = Math.max(...weights);
  let right = 0;
  for (const w of weights) right += w;
  if (left === right) return left;
  while (left < right) {
    // 初始的左边界应该是最大的货物重量，右边界应该是总重量
    let mid = Math.floor((left + right) / 2);
    let result = getDay(weights, mid); // 这里把先计算出的mid传入，测试能不能保证在限制内完成
    if (result > days) {
      // 如果需要的天数大于限制，就应该增加运输量，即增大mid
      left = mid + 1;
    } else {
      // 反之就减小
      right = mid;
    }
  }
  return left;
};
const getDay = function (weights, limit) {
  let day = 1; // 需要的天数
  let cur = 0; // 当前运送的包裹重量和
  for (const weight of weights) {
    if (cur + weight > limit) {
      day++;
      cur = 0;
    }
    cur += weight;
  }
  return day;
};
```

因此二分查找相关的题目核心在于“查找”，而不是“二分”。如果一道题是在一个区间内查找最值，并且能满足上面所说的函数，就可以尝试用这种方法。

## 爱吃香蕉的珂珂

> 珂珂喜欢吃香蕉。这里有 n 堆香蕉，第 i 堆中有  piles[i]  根香蕉。警卫已经离开了，将在 h 小时后回来。
> 珂珂可以决定她吃香蕉的速度 k （单位：根/小时）。每个小时，她将会选择一堆香蕉，从中吃掉 k 根。如果这堆香蕉少于 k 根，她将吃掉这堆的所有香蕉，然后这一小时内不会再吃更多的香蕉。  
> 珂珂喜欢慢慢吃，但仍然想在警卫回来前吃掉所有的香蕉。
> 返回她可以在 h 小时内吃掉所有香蕉的最小速度 k（k 为整数）。

这道题和上面那道基本套路都是一样的。

思路还是先确定一个范围，然后利用二分查找的左边界，找到可以满足的最小值。由题意，k 和吃掉的时间 t 成反比。那么可以先从确定的范围内计算一个 mid 作为 k，计算这个 k 下吃掉所有香蕉所需的时间 t，和 h 比较，如果大了就改变左边界，反之亦然。

```js
var minEatingSpeed = function (piles, h) {
  let left = 1;
  let right = piles.reduce((a, b) => a + b);
  while (left < right) {
    const mid = Math.floor((right + left) / 2);
    const hours = piles.reduce((pre, cur) => pre + Math.ceil(cur / mid), 0);
    if (hours > h) {
      left = mid + 1;
    } else {
      right = mid;
    }
  }
  return left;
};
```

# 二叉树

## 二叉树问题的基本递归思路

二叉树问题基本上都离不开递归和遍历；
而递归实际上有两种，一种是单纯只递归，比如基本的递归遍历方式：

```js
function dfs(node) {
  if (!node) return;
  dfs(node.left);
  dfs(node.right);
}
```

这种递归只有在边界条件才会返回，因此一般常用于彻底的遍历，比如路径问题。
在思考方式上，这种递归可以看作是只有一个左右节点的树的遍历。常常把操作放置在两个递归的`dfs()`前面、中间或后面，对应不同的时刻：

- 前面：该节点已经遍历到，但其子节点还没开始遍历，可以进行记录该节点值、判断临界条件等操作
- 中间：只遍历了左子树，一般很少放在这里，除非是为了中序遍历
- 后面：该节点和其子树都被完全遍历。站在最顶层调用栈的角度上，这时整棵树已经遍历完毕；可以在这里显式返回一个值，这个值将会是上面每个 dfs 都会返回的值；也可以执行一些**回溯**相关的操作，比如路径问题中在这里让当前节点出栈、回溯问题中在这里返回上一层等等

---

另一种递归是显式的返回值：

```js
function dfs(node) {
  if (!node) return 0;
  let left = dfs(node.left);
  let right = dfs(node.right);
  return node.val;
}
```

这时前面两个递归的 dfs 也会产生有效的返回值，并且返回的值就是最末端的返回值。

注意要以“递归的方式思考问题”，不要把上面两个 dfs 看成复杂的调用栈，而是只把他们当作左节点和右节点的调用结果即可。

> 这种遍历方式能够生效的最重要一点是：**左右递归调用结果**（**当前节点**有时候也会）都会参与返回值的运算

```js
function dfs(node) {
  if (!node) return 0;
  let left = dfs(node.left);
  let right = dfs(node.right);
  return node.val + Math.max(left, right);
}
```

这里就会把当前节点的值和左右子节点中较大的值加起来。由于自底向上的每次返回都用到了 left 和 right，因此数值就是从下向上连续的，left 和 right 可以看作是左右子树的计算结果。

再比如，常见的方法之一是利用返回布尔值进行一些判断。比如基本的递归搜索一个值，找到了则返回 true

```js
// 和搜索相关的问题是少都要考虑这个写法，这是最基本的
function search(root, val) {
  if (!root) return false;
  return search(root.left, val) || search(root.right, val) || root.val === val;
}
```

这里返回值综合了两个遍历的结果和当前节点的值，只要有一个符合条件就返回 true，即找到一个就返回 true。

当函数参数不止有节点这一个参数时，当前节点值还可以放入参数中。比如`路径总和1`这个题，就是通过每次递减一个当前节点值来实现。

还可以有条件的返回不同的值；这种方法常见于偏运算，比如只计算左叶子和这种问题：

```js
function dfs(node) {
  if (!node) return 0;
  let left = dfs(node.left);
  dfs(node.right);
  sum += left;
  if (!node.left && !node.right) return node.val;
  return 0;
}
```

这里只有当节点为叶子节点时才会返回有效的值，相当于是在遍历之中设置了回溯条件。

## 二叉树的迭代遍历

### 先序遍历

1. 将根节点入栈
2. 出栈，输出该节点，然后将该节点的子节点入栈，先右后左；这样出栈的顺序就是先左后右，符合先序遍历的`左->右->中`顺序
3. 后续依次类推

```js
const preorder = (root) => {
  const stack = [];
  const res = [];
  stack.push(root);
  while (stack.length) { 
    let node = stack.pop();
    res.push(node.val);
    if (node.right) stack.push(node.right);
    if (node.left) stack.push(node.left);
  }
};
```

### 后序遍历

1. 同样是先入栈根节点
2. 出栈，将该节点的输出插入到结果队列的头部（unshift），这样结果数组相当于是反向；然后将子节点入栈，先左后右
3. 以此类推

```js
const postorderIterate = (root) => {
  const stack = [];
  const res = [];

  stack.push(root);
  while (stack.length) {
    const node = stack.pop();

    // 相当于把结果数组反向了
    res.unshift(node.val);
    // 因为结果数组反向,这里也要变一下
    if (node.left) stack.push(node.left);
    if (node.right) stack.push(node.right);
  }
};
```

### 中序遍历

中序遍历根另外两者都不一样，因为先序和后序都是通过出栈一个元素作为根节点，然后把它的左右子元素入栈。但是中序遍历的顺序是`左->中->右`，也就说必须有从`左`回到其父元素`中`的过程。

1. 从根节点开始迭代取左子元素，每迭代一次就入栈一个元素，直到最左端位置
2. 出栈一个元素，输出结果，然后取该节点的右节点再次迭代。因为这时的出栈元素如果有右节点，说明就是一个父元素，相当于`中`

```js
const inorderIterate = (root) => {
  const stack = [];
  const res = [];

  stack.push(root);
  let curr = root;
  while (stack.length) {
    while (curr) {
      stack.push(curr);
      curr = curr.left;
    }

    curr = stack.pop();
    res.push(curr.val);
    curr = curr.right;
  }
};
```

### 层序遍历

1. 设置一个队列，将根元素入队
2. 出队队头元素并输出，把这个元素的子元素全部入队。重复这个过程，直到把上一步的所有入队元素都出队并输出；这里需要先缓存上一步时队列的长度，然后遍历这个长度直到上一层的元素都依次执行该过程。
3. 依次类推

```js
const levelOrder = (root) => {
  const queue = [];
  const res = [];
  queue.push(root);

  while (queue.length) {
    const len = queue.length;
    // 相当于每次执行到这里就是新的一层
    for (let i = 0; i < len; i++) {
      const node = queue.shift();
      res.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
  }
  return res;
};
```

## 翻转二叉树

> ![](https://pic.imgdb.cn/item/62675aab239250f7c5499cac.jpg)

思路：以递归的方式，遍历树中的每一个结点，并将每一个结点的左右孩子进行交换。

```js
const invertTree = (root) => {
  if (!root) return;
  let left = invertTree(root.left);
  let right = invertTree(root.right);
  root.left = right;
  root.right = left;
  return root;
};
```

翻转二叉树是这种非尾递归思想的重要体现之一；通过递归先一直到边界（即叶子节点）为止，然后返回该节点给上一层的`left`；再同理把`right`交给上一层，在上一层中进行交换，就完成了自下而上的第一次交换，后续同理。

比如一个通常的遍历是这样：

```js
function dfs(node) {
  if (!node) return;
  dfs(node.left);
  dfs(node.right);
}
```

这样的遍历只会在遇到边界时返回。如果我们在最后显式返回某一个值（一般就是本次递归的参数传入的节点），每一个递归就相当于拿到了其上一层的返回值：

```js
function dfs(node) {
  if (!node) return node;
  let left = dfs(node.left); // left是当前node的左子树的根节点
  let right = dfs(node.right); // 右子树
  return node; // 返回本节点
}
```

得到了 left 和 right，从递归的思维来看，实际上就是拿到了任一个节点的左右子树（不用考虑反复递归的多层，只考虑一层即可）。所以我们可以进行一些操作，比如上面的交换节点。

如果最简单的二叉树（只有三个节点）可以执行，那么任何情况下的二叉树都可以执行。因此这类问题可以先从最简单的考虑。如果最简单的情况可以保证递归的完整，那么之后的逻辑就不用考虑递归的内部，而是只把递归当成一个“取值”的步骤，正常处理其他逻辑。

## 平衡二叉树

### 基础数据结构

js 平衡二叉树代码如下。

参考：https://zhuanlan.zhihu.com/p/438604092

平衡二叉树的实现关键是插入节点时的旋转，左旋和右旋。只有当插入和删除节点时才会出现不平衡的情况。

比如考虑插入的情况，会有下面四种：
1. LL：向根节点的左子树插入一个节点。这时根节点的左子树有可能会比右子树高，而且如果出现不平衡，那么根节点的平衡因子应该是+2，即左子树比右子树高2层
2. LR：向根节点的左子树插入一个右子节点，根节点平衡因子为+2
3. RR：右子树插入右子节点，平衡因子为-2
4. RL：右子树插入左子节点，-2

当出现上面四种情况时，就需要旋转树以保证平衡。注意这里的根节点不一定是整个树的根节点，也可以是某个子树的根节点。

以LL旋转为例，实际操作是：

![](https://pic.imgdb.cn/item/64bbffdb1ddac507cc08e5d9.jpg)

其他三种旋转可以参考那篇文章，大体的旋转方式类似，都是改变根节点，然后把原来的根节点插入改变之后的，这样就可以降低高度差。

```js
class AVLTree {
  constructor() {
    this.root = null;
  }

  // 获取节点的高度
  getHeight(node) {
    if (!node) return 0;
    return Math.max(this.getHeight(node.left), this.getHeight(node.right)) + 1;
  }

  // 获取节点的平衡因子，其实是左右节点高度的差值
  // 如果>1，则说明左子树比右子树高度高，且不平衡，如果<-1则是反过来
  getBalanceFactor(node) {
    if (!node) return 0;
    return this.getHeight(node.left) - this.getHeight(node.right);
  }

  // 左旋转
  leftRotate(node) {
    const temp = node.right;
    node.right = temp.left;
    temp.left = node;
    return temp;
  }

  // 右旋转
  rightRotate(node) {
    const temp = node.left;
    node.left = temp.right;
    temp.right = node;
    return temp;
  }

  // 插入节点
  insert(key) {
    const newNode = {
      key,
      left: null,
      right: null,
    };
    this.root = this._insertNode(this.root, newNode);
  }

  _insertNode(node, newNode) {
    if (!node) return newNode;
    if (newNode.key < node.key) {
      node.left = this._insertNode(node.left, newNode);
    } else if (newNode.key > node.key) {
      node.right = this._insertNode(node.right, newNode);
    } else {
      return node; // 已存在相同节点
    }

    // 插入完成之后，如果不平衡就需要进行旋转操作
    // 后序检查每一个node，如果不平衡就进行旋转
    const balanceFactor = this.getBalanceFactor(node);
    if (balanceFactor > 1 && newNode.key < node.left.key) {
      // LL情况，进行右旋转
      return this.rightRotate(node);
    }
    if (balanceFactor < -1 && newNode.key > node.right.key) {
      // RR情况，进行左旋转
      return this.leftRotate(node);
    }
    if (balanceFactor > 1 && newNode.key > node.left.key) {
      // LR情况，先对左子节点进行左旋转，再对当前节点进行右旋转
      node.left = this.leftRotate(node.left);
      return this.rightRotate(node);
    }
    if (balanceFactor < -1 && newNode.key < node.right.key) {
      // RL情况，先对右子节点进行右旋转，再对当前节点进行左旋转
      node.right = this.rightRotate(node.right);
      return this.leftRotate(node);
    }
    return node;
  }

  // 删除节点
  remove(key) {
    this.root = this._removeNode(this.root, key);
  }

  _removeNode(node, key) {
    if (!node) return null;
    if (key < node.key) {
      node.left = this._removeNode(node.left, key);
    } else if (key > node.key) {
      node.right = this._removeNode(node.right, key);
    } else {
      // 当前节点就是要删除的节点
      if (!node.left && !node.right) {
        // 情况1：叶子节点
        return null;
      } else if (!node.left || !node.right) {
        // 情况2：只有一个子节点
        return node.left || node.right;
      } else {
        // 情况3：有两个子节点
        // 这里的删除和二叉搜索树的方式一样，找到左子树的最大或者右子树的最小，替换根节点
        const minNode = this._findMinNode(node.right);
        node.key = minNode.key;
        node.right = this._removeNode(node.right, minNode.key);
      }
    }

    const balanceFactor = this.getBalanceFactor(node);
    if (balanceFactor > 1 && this.getBalanceFactor(node.left) >= 0) {
      // LL情况，进行右旋转
      return this.rightRotate(node);
    }
    if (balanceFactor < -1 && this.getBalanceFactor(node.right) <= 0) {
      // RR情况，进行左旋转
      return this.leftRotate(node);
    }
    if (balanceFactor > 1 && this.getBalanceFactor(node.left) < 0) {
      // LR情况，先对左子节点进行左旋转，再对当前节点进行右旋转
      node.left = this.leftRotate(node.left);
      return this.rightRotate(node);
    }
    if (balanceFactor < -1 && this.getBalanceFactor(node.right) > 0) {
      // RL情况，先对右子节点进行右旋转，再对当前节点进行左旋转
      node.right = this.rightRotate(node.right);
      return this.leftRotate(node);
    }
    return node;
  }

  // 查找最小节点
  _findMinNode(node) {
    while (node && node.left) {
      node = node.left;
    }
    return node;
  }
}
```

### 判断平衡二叉树

平衡二叉树，即一个节点的左右子树高度差不能大于 1。

思路就是递归遍历每一个节点，计算每隔节点的左右子树高度差，如果有一个节点高度差大于 1，整体就失去平衡。

第一种方法是计算和遍历分开，计算节点高度是单独的方法，而遍历采用先序遍历的方式。
获取节点高度的方式是分别递归左右子树，每返回一次高度都会+1，最终会以左右子树中较高的为结果

```js
const isBalanced = function (root) {
  let flag = true;
  dfs(root);
  function dfs(root) {
    if (!root) return;
    let left = getNodeHeight(root.left);
    let right = getNodeHeight(root.right);
    if (Math.abs(left - right) > 1) flag = false;
    dfs(root.left);
    dfs(root.right);
  }
  function getNodeHeight(node) {
    if (node == null) {
      return -1;
    }
    return Math.max(getNodeHeight(node.left), getNodeHeight(node.right)) + 1;
  }
  return flag;
};
```

更好的解决方法是遍历的同时就计算高度，在递归函数末尾返回上一步得到的左右高度中较大一个+1

```js
const isBalanced2 = function (root) {
  let flag = true;
  dfs(root);
  function dfs(root) {
    if (!root) return -1;
    const left = dfs(root.left);
    const right = dfs(root.right);
    if (Math.abs(left - right) > 1) {
      flag = false;
      return -1;
    }
    return Math.max(left, right) + 1;
  }

  return flag;
};

// 或者同时返回两个值也可
const isValidTree = (root) => {
  if (!root) return [true,0];
  const [left, leftHeight] = isValidTree(root.left);
  const [right, rightHeight] = isValidTree(root.right);
  return [
    left && right && Math.abs(leftHeight - rightHeight) <= 1,
    Math.max(leftHeight, rightHeight) + 1,
  ];
};
```

### 构造平衡二叉搜索树

> 构造平衡二叉树.给你一棵二叉搜索树，请你返回一棵平衡后的二叉搜索树，新生成的树应该与原来的树有着相同的节点值。

思路就是**中序遍历**获取数的数组形式,然后用上面的有序数组转为平衡二叉树方法即可

```js
const balanceBST = function (root) {
  const nums = [];

  inorder(root);
  function inorder(node) {
    if (!node) return;
    inorder(node.left);
    nums.push(node.val);
    inorder(node.right);
  }

  const tree = buildBST(nums);
  function buildBST(arr) {
    if (!arr.length) return null;
    let mid = Math.floor(arr.length / 2);
    const node = new TreeNode(arr[mid]);
    node.left = buildBST(arr.slice(0, mid));
    node.right = buildBST(arr.slice(mid + 1));
    return node;
  }
  return tree;
};
```

## BST（二叉搜索树）

### BST 的特点

- BST 的基本特点是一个根节点的左子树上的**所有值**一定比该节点小，右子树的**所有值**一定比该节点大
- BST 的中序遍历会返回一个从小到大的顺序排列，可以利用这一点做很多事，比如求最值等等。

二叉搜索树的主要作用是便于查找。类似二分查找，一颗比较平衡的二叉搜索树的查找的理想时间复杂度是 O(logn)；但在最坏的情况下，二叉搜索树可能会退化为链表，此时查找的时间复杂度将变为 O(n)。
因此比较好的方式是采用平衡的二叉搜索树。即，将二叉搜索树和二叉平衡树结合起来，保证二叉搜索树效率最大化。

### BST 基本操作

查找：

```js
function search(root, n) {
  if (!root) return null;
  if (root.val === n) return root;
  else if (root.val < n) search(root.left, n);
  else search(root.right, n);
}
```

增加：

```js
function insert(root, n) {
  if (!root) return;
  if (n < root.val) {
    if (!root.left) root.left = new TreeNode(n);
    else insert(root.left, n);
  } else {
    if (!root.right) root.right = new TreeNode(n);
    else insert(root.right, n);
  }
}
```

删除：

删除节点的函数要返回参数 root，因为执行函数的返回值实际上是“替换子树”的效果，也是上面说的非尾递归的形式。

```js
function removeNode(root, n) {
  if (!root) {
    return null;
  }
  if (root.val > key) {
    root.left = deleteNode(root.left, key);
    return root;
  }
  if (root.val < key) {
    root.right = deleteNode(root.right, key);
    return root;
  }
  if (root.val === key) {
    if (!root.left && !root.right) {
      return null;
    }
    if (!root.right) {
      return root.left;
    }
    if (!root.left) {
      return root.right;
    }
    const minNode = findMin(root.right); //先找到右子树的最小节点
    root.right = deleteNode(root.right, minNode.val); // 从当前节点的右子树中删去这个找到的最小节点
    minNode.right = root.right; // 这两步相当于把root替换为了minNode
    minNode.left = root.left;
    return minNode; // 返回的minNode将会连接到父节点上，相当于完全替换了原来的root
  }
  return root;
}

function findMin(node) {
  if (!node.left) return node;
  return findMin(node.left);
}
```

### 验证是否是有效 BST

即验证树是否保证`左 < 中 < 右`

注意不能单独把左子节点提出来判断是不是小于本节点，因为这样相当于只判断了一层（紧挨着的左子节点），而不是整颗左子树

```js
const isValidBST = function (root) {
  function dfs(root, minValue, maxValue) {
    // 若是空树，则合法
    if (!root) {
      return true;
    }
    if (root.val <= minValue || root.val >= maxValue) return false;
    // 接下来要遍历左子树，那就把当前节点的值作为最大值传入，保证左子树的值不大于当前节点。右子树同理
    return (
      dfs(root.left, minValue, root.val) && dfs(root.right, root.val, maxValue)
    );
  }
  // 初始化最小值和最大值为极小或极大
  return dfs(root, -Infinity, Infinity);
};

// 另一种方法
// 返回三个值，分别是[该子树是否合法，该子树的最小值，该子树的最大值]
const isValidSearchTree = (root) => {
    // 如果子树为null，那么最大值就应该是-Infinity。在返回的时候和root.val比较，就会选成root.val；相当于如果一个节点没有右子树，那么最大值就是他自己
    // 最小值也是同理，这里设置-Infinity和Infinity就是为了下面的比较替换
    if (!root) return [true, -Infinity, Infinity]
    if (!root.left && !root.right) return [true, root.val, root.val];
    // [左子树是否合法，左子树的最大值，左子树的最小值]
    const [left, leftMax, leftMin] = isValidSearchTree(root.left);
    const [right, rightMax, rightMin] = isValidSearchTree(root.right);
    return [
        left && right && root.val > leftMax && root.val < rightMin,
        Math.max(rightMax, root.val), // 这两个，主要考虑左右子数可能为空的情况。如果右子树为空，那么显然该子树的最大值应该是当前节点
        Math.min(leftMin, root.val), // 反过来同理
    ];
};
```

---

还有一种方法是利用二叉搜素树的中序遍历，如果遍历的过程中不是有序的（每次和前一个值做对比），就说明不是有效。

### 有序数组转 BST

> 把有序数组转为平衡二叉树

思路类似二分查找,把中间的数作为根节点"提起来",然后范围二分为`[low,mid)`和`(mid,high]`分别给左右子树

![](https://pic.imgdb.cn/item/62677250239250f7c582b679.jpg)

```js
const sortedArrayToBST = function (nums) {
  return buildBST(0, nums.length - 1);
  function buildBST(low, high) {
    if (low > high) return;
    let mid = Math.floor(low + (high - low) / 2);
    //注意这个地方需要low+...,因为不一定是从0开始的,需要low+计算值才是真正的中间值
    const curr = new TreeNode(nums[mid]);
    curr.left = buildBST(low, mid - 1);
    curr.right = buildBST(mid + 1, high);
    return curr;
  }
};
```

`buildBST`方法还可以接收一个分割好的数组:

```js
function buildBST(arr) {
  if (!arr.length) return null;
  // 这里取arr.length / 2,这样偶数元素中间值就会划分到偏后一位,防止arr.slice(0, mid - 1)在有元素的时候仍截取到空数组
  let mid = Math.floor(arr.length / 2);
  const node = new TreeNode(arr[mid]);
  node.left = buildBST(arr.slice(0, mid));
  node.right = buildBST(arr.slice(mid + 1));
  return node;
}
```

#### 构造最大二叉树

> 给定一个不含重复元素的整数数组。一个以此数组构建的最大二叉树定义如下：
> 二叉树的根是数组中的最大元素。
> 左子树是通过数组中最大值左边部分构造出的最大二叉树。
> 右子树是通过数组中最大值右边部分构造出的最大二叉树。
> 通过给定的数组构建最大二叉树，并且输出这个树的根节点。

构造最大二叉树的思路就是在构造二叉树时，把中间节点置为数组中的最大值即可。

```js
/**
 * @param {number[]} nums
 * @return {TreeNode}
 */
var constructMaximumBinaryTree = function (nums) {
  return buildBST(nums, 0, nums.length - 1);
  function buildBST(nums, low, high) {
    if (low > high) return null;

    let maxVal = -1;
    let maxIndex = -1;
    for (let i = low; i <= high; ++i) {
      if (nums[i] > maxVal) {
        maxVal = nums[i];
        maxIndex = i;
      }
    }
    let curr = new TreeNode(maxVal);
    console.log(maxVal);
    curr.left = buildBST(nums, low, maxIndex - 1);
    curr.right = buildBST(nums, maxIndex + 1, high);
    return curr;
  }
};
```

### 二叉树的双指针（前指针）

二叉树问题的解决（尤其是二叉搜素树）可以遍历成一个数组，然后用对数组的操作方式操作。因为数组通常可以比较方便的访问前后的值；
实际上不需要额外的内存空间，也可以直接在二叉树问题遍历时取得前一个值。
方法就是利用中序遍历：

```js
function dfs(node){
  if(!node) return
  dfs(node.left)
  if(!pre){
    ...
  }
  pre = node
  dfs(node.right)
}
```

因此可以利用这个方法解决（尤其是二叉搜索树）一些可能需要通过转成数组的问题。

### 二叉搜素树的众数

https://leetcode.cn/problems/find-mode-in-binary-search-tree/solution/

> 给你一个含重复值的二叉搜索树（BST）的根节点 root ，找出并返回 BST 中的所有 众数（即，出现频率最高的元素）。
> 如果树中有不止一个众数，可以按 任意顺序 返回。

利用上面的方法，因为二叉搜索树的中序遍历一定是从小到大的有序数组，因此可以完全按照数组的方式去做。
数组中怎么求众数？定义一个出现次数的变量，前一个数等于后一个，就给出现次数+1；如果出现次数大于最大出现次数，就把最大出现次数更新；如果等于最大出现次数，就把数字放入最大出现次数的数字的数组中。
二叉树同理

```js
/**
 * @param {TreeNode} root
 * @return {number[]}
 */
var findMode = function (root) {
  if (!root.left && !root.right) return [root.val];
  let cnt = 1;
  let maxTimes = 0;
  const maxNumbers = [];
  let pre = null;
  function dfs(node) {
    if (!node) return;
    dfs(node.left);
    if (pre && pre.val === node.val) {
      cnt++;
    } else {
      cnt = 1;
    }
    if (cnt > maxTimes) {
      maxTimes = cnt;
      maxNumbers.length = 0;
      maxNumbers.push(node.val);
    } else if (cnt === maxTimes) {
      maxNumbers.push(node.val);
    }
    pre = node;
    dfs(node.right);
  }
  dfs(root);
  return maxNumbers;
};
```

## 完全二叉树

### 二叉树的完全性校验（检查是否是完全二叉树）

https://leetcode.cn/problems/check-completeness-of-a-binary-tree/

给你一颗树，检查是否是完全二叉树。完全二叉树的最后一行可以不全吗，但必须靠左。

方法是通过记录索引，每次放入节点时也把索引放入。对于完全二叉树来说，一个节点的左节点和右节点的索引应该是确定的。

然后将所有索引打印输出，应该是一个从1开始连续的数。如果中间不连续则返回false

```js
var isCompleteTree = function (root) {
    const queue = [{ node: root, index: 1 }]
    const res = []
    while (queue.length) {
        const len = queue.length
        for (let i = 0; i < len; i++) {
            const { node, index } = queue.shift()
            res.push(index)
            if (node.left) {
                queue.push({ node: node.left, index: index * 2 })
            }
            if (node.right) {
                queue.push({ node: node.right, index: index * 2 + 1 })
            }
        }
    }
    console.log(res)
    for(let i = 1;i <= res.length;i++){
        if(res[i-1] !== i) return false
    }
    return true
};
```



## 比较两棵二叉树

> 给你两棵二叉树的根节点 p 和 q ，编写一个函数来检验这两棵树是否相同。如果两个树在结构上相同，并且节点具有相同的值，则认为它们是相同的。

思路：递归比较两棵树的左、右子树即可。如果两棵树完全相等，则 p 和 q 应当是完全相同的，只要有一个对不上就是 false

```js
var isSameTree = function (p, q) {
  if (!p && !q) return true;
  else if ((q && !p) || (p && !q) || p.val !== q.val) return false;

  return isSameTree(p.left, q.left) && isSameTree(p.right, q.right);
};
```

## 二叉树轴对称

> 给你一个二叉树的根节点 root，检查它是否轴对称。

这道题和翻转二叉树思路近似，从`root.left`和`root.right`分开，分别递归比较左子树的右节点和右子树的左节点

```js
function check(p, q) {
  if (!p && !q) return true;
  if ((!p && q) || (!q && p) || p.val !== q.val) return false;
  return check(p.left, q.right) && check(p.right, q.left);
}
```

## 二叉树的深度

> 给定一个二叉树，找出其最大深度。二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。

这道题就是求节点高度，递归遍历每一个节点，每次遍历都给返回值+1，并选择左右子树中较大的一个。

```js
var maxDepth = function (root) {
  if (!root) return 0;
  let left = maxDepth(root.left);
  let right = maxDepth(root.right);
  return Math.max(left, right) + 1;
};
```

这道题还有一个变种，就是求一个节点的最小深度。注意最小深度仍然是到叶子节点的，不能在空节点就停下，因此需要判断左右子节点的高度，不为 0 才返回：

```js
var minDepth = function (root) {
  if (!root) return 0;
  let left = minDepth(root.left);
  let right = minDepth(root.right);

  if (left === 0 && right > 0) return right + 1; // 左节点为空
  else if (left > 0 && right === 0) return left + 1; // 右节点为空
  else return Math.min(left, right) + 1;
};
```

---

关于深度问题，其实还有一个更合适和通用的解法，就是利用 BFS 层序遍历。因为层序可以保证一层一层的往下走，那么走到第一个叶子节点就可以得到最小深度，最后走完就可以得到最大深度。

```js
var minmaxDepth = function (root) {
  const queue = [];
  queue.push(root);
  let depth = 0;
  while (queue.length) {
    const len = queue.length;
    for (let i = 0; i < len; i++) {
      const top = queue.shift();
      if (!top.left && !top.right) return depth; // 最小深度就在这里返回
      if (top.right) queue.push(top.right);
      if (top.left) queue.push(top.left);
    }
    depth++;
  }
  return depth; // 最大深度就遍历完之后返回
};
```

## 二叉树的路径问题

这里可以参考：https://leetcode-cn.com/problems/path-sum-ii/solution/yi-pian-wen-zhang-jie-jue-suo-you-er-cha-oo63/

路径问题可以分为两大类

- 自顶向下，从某一个节点(不一定是根节点)，从上向下寻找路径，到某一个节点(不一定是叶节点)结束。方法类似，在每次遍历中记录路径，当遍历到合适条件时输出结果，再在每次递归的最尾端执行复原操作（比如出栈、总和减去当前值等等）。

> 其实这就是回溯，回溯问题本质也是在遍历一棵树并记录路径。回溯部分（状态的恢复）就位于后序遍历位置，状态改变则位于先序位置

```js
const path = [];
function dfs(node) {
  if (!node) return;
  if (临界条件) {
    res.push(path);
    return;
  }
  path.push(node.val);
  dfs(node.left);
  dfs(node.right);
  path.pop();
}
return res;
```

如果要求有路径和，一般是把目标值放在最顶端，然后自顶向下每次遍历都减去当前节点值，直到减为 0 或者其他临界条件为止。

```js
function dfs(node, sum) {
  if (!node) return;
  path.push(node.val);
  sum -= node.val;

  if (sum === 0) {
    res.push(path);
  }

  dfs(node.left, sum);
  dfs(node.right, sum);
  path.pop();
}
dfs(root, targetNum);
```

- 非自顶向下：就是从任意节点到任意节点的路径，不需要自顶向下。这种一般比较麻烦，除了从上到下的基本顺序，还有可能是横着的甚至倒着的路径。

这类题目一般解题思路如下：
设计一个辅助函数 maxpath，调用自身求出以一个节点为根节点的左侧最长路径 left 和右侧最长路径 right，那么经过该节点的最长路径就是 `left+right`
接着只需要从根节点开始 dfs,不断比较更新全局变量即可

```js
let res = -Infinity;

function maxPath(node) {
  if (!node) return 0;
  let left = maxPath(node.left);
  let right = maxPath(node.right);
  res = Math.max(res, node.val + left + right);
  return Math.max(left, right) + node.val;
}
```

1. left,right 代表的含义要根据题目所求设置，比如最长路径、最大路径和等等
2. 全局变量 res 的初值设置是 0 还是`-Infinity`要看题目节点是否存在负值，如果存在就用`-Infinity`，否则就是 0

### 路径问题 1（目标和）

> 给你二叉树的根节点 root 和一个表示目标和的整数  targetSum 。判断该树中是否存在 根节点到叶子节点 的路径，这条路径上所有节点值相加等于目标和  targetSum 。如果存在，返回 true ；否则，返回 false

路径问题最基本的一道题，好好背就完事了。
思路就是：把总和依次相减下来，每到一个节点就减去当前节点的值，直到叶子节点判断是否等于叶子节点的值，
如果等于说明一路上减去的值加起来正好就是 targetSum

```js
var hasPathSum = function (root, targetSum) {
  if (!root) return false;
  if (!root.left && !root.right) {
    return targetSum === root.val;
  }
  return (
    hasPathSum(root.left, targetSum - root.val) ||
    hasPathSum(root.right, targetSum - root.val)
  );
};
```

另外一个方法是先序遍历时判断，思路差不多：

```js
var hasPathSum = function (root, targetSum) {
  let flag = false;
  function dfs(node, num) {
    if (!node) return;
    if (!node.left && !node.right && num === node.val) flag = true;
    dfs(node.left, num - node.val);
    dfs(node.right, num - node.val);
  }
  dfs(root, targetSum);
  return flag;
};
```

### 路径问题 2（具体路径）

这道题是上一个的升级版，要求求出具体路径

思路在上面的解析中已经有过，就是记录路径，在递归结束时弹出当前节点，并在条件合适时记录路径。

```js
var pathSum = function (root, targetSum) {
  const path = [];
  const res = [];
  dfs(root, targetSum);
  function dfs(node, sum) {
    if (!node) return;
    if (!node.left && !node.right && sum === node.val) {
      res.push([...path, node.val]);
      return;
    }
    path.push(node.val);
    dfs(node.left, sum - node.val);
    dfs(node.right, sum - node.val);
    path.pop();
  }
  return res;
};
```

### 路径问题 3（任意节点开始）

https://leetcode.cn/problems/path-sum-iii/

> 给定一个二叉树的根节点 root ，和一个整数 targetSum ，求该二叉树里节点值之和等于 targetSum 的 路径 的数目。路径 不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。

这道题有两种解法，最好想的就是二重递归：

- 第一重递归遍历所有节点
- 然后对于每个节点，执行路径问题 1 的算法

方法如下：

```js
var pathSum = function (root, targetSum) {
  if (!root) return 0;
  let paths = dfs(root, targetSum);
  return paths + pathSum(root.left, targetSum) + pathSum(root.right, targetSum);
};

function dfs(node, targetSum) {
  let paths = 0; 
  if (!node) return 0;
  // 这里不能直接返回1，因为考虑到节点的值是有正负的，当前路径和达到了，并不代表之后的不会达到
  // 因此这里继续递归即可。实际上paths只会是1或0
  if (node.val === targetSum) paths++; 
  return (
    paths +
    dfs(node.left, targetSum - node.val) +
    dfs(node.right, targetSum - node.val)
  );
}
```

还有一种方法是使用**前缀和**，但是比较麻烦，没太看懂，参考https://leetcode-cn.com/problems/path-sum-iii/solution/lu-jing-zong-he-iii-by-leetcode-solution-z9td/

> 前缀和还是先搞清一维数组的前缀和把

### 最大路径和

> 给你一个二叉树的根节点 root ，返回其 最大路径和 。
> 路径和 是路径中各节点值的总和。
> 路径 被定义为一条从树中任意节点出发，沿父节点-子节点连接，达到任意节点的序列。
> 同一个节点在一条路径序列中 至多出现一次 。该路径 至少包含一个 节点，且不一定经过根节点

这个是上面说的非自顶向下的典型例题，方法也是参考那个的，但是稍有改动：

- 节点有负值，加上一个负值会对最大路径和起到反作用，因此如果左右路径和为负数就不选取（即取 0，和空节点的情况一样）
- `路径` 一定是先上升（ 0 ～ n 个）节点, 到达顶点, 后下降（ 0 ～ n 个）节点；因此表现出来就是`node.val + left + right`

代码：

```js
var maxPathSum = function (root) {
  let res = -Infinity;
  function maxPath(node) {
    if (!node) return 0;
    let left = Math.max(maxPath(node.left), 0);
    let right = Math.max(maxPath(node.right), 0);
    res = Math.max(res, node.val + left + right);
    // 返回的是左右子树的最大路径和的较大值，因为不可能把左右子树路径连起来
    // 只能选择左或右，否则不能构成路径
    return Math.max(left, right) + node.val; 
  }
  maxPath(root);
  return res;
};
```

## 二叉树左叶子之和

> 计算给定二叉树的所有左叶子之和。

注意是左叶子，因此不能包括树枝节点；所以可以利用遍历找到包含左叶子的节点，加上这个节点即可。

```js
/**
 * @param {TreeNode} root
 * @return {number}
 */
var sumOfLeftLeaves = function (root) {
  let sum = 0;
  function dfs(node) {
    if (!root) return;
    if (root.left && !root.left.left && !root.left.right) sum += root.left.val;
    dfs(node.left);
    dfs(node.right);
  }
  dfs(root);
  return sum;
};
```

## 二叉树最值

怎么求一个二叉树的最值？

对于一个普通的二叉树，求最大值的方法就是简单的递归：

```js
function dfs(node) {
  if (!node) return 0;
  return Math.max(dfs(node.left), dfs(node.right), node.val);
}
```

如果是 BST，BST 的一个特点的**中序排列的输出结果是一个递增序列**。即第 n 次到输出位置时，恰好是第 n 个最小的数字。
如果希望是一个倒序的输出，调换两个递归的位置即可。

```js
function dfs(node) {
  if (!node) return;
  dfs(node.left);
  res.push(node.val);
  dfs(node.right);
}
```

## 二叉树的最近公共祖先

> 给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/

思路：记住满足条件：

```js
const lson = dfs(root.left, p, q);
const rson = dfs(root.right, p, q);
(lson && rson) ||
  ((root.val === p.val || root.val === q.val) && (lson || rson));
```

解释：

0. 首先明确递归的返回值：指**在该树中是否存在 p 或者 q 的一个或多个**
1. 如果 p 和 q 分别位于左子树和右子树中，那么`lson && rson`一定为 true，说明两边至少包含 p 和 q 的一个；
2. 如果 p 或 q 的一个是当前节点本身，那么值肯定相等，并且自己的左子树和右子树至少要有一个包含另一个节点。

判断之后，最后返回的值应该是：

1. 判断自己和这个 p 和 q 之一是否相等，因为如果有一个相等就必然为 true（一定包含 q 或 p）
2. 判断自己的子树情况，即`lson || rson`，因为递归条件是只要有一个包含就行，因此两个有一个为 true 就行

最终代码：

```js
var lowestCommonAncestor = function (root, p, q) {
  let ans;
  const dfs = (root, p, q) => {
    if (root === null) return false;
    const lson = dfs(root.left, p, q);
    const rson = dfs(root.right, p, q);
    if (
      (lson && rson) ||
      ((root.val === p.val || root.val === q.val) && (lson || rson))
    ) {
      ans = root;
    }
    return lson || rson || root.val === p.val || root.val === q.val;
  };
  dfs(root, p, q);
  return ans;
};
```

## 从前序遍历和中序遍历构造二叉树

https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/

思路：

- 前序遍历的第一个节点是整个二叉树的根元素
- 中序遍历从根元素划分，左右两半数组恰好是左右子树的中序遍历

因此通过前序遍历划分左右子树的前序、先序遍历，再依次递归即可。

```js
/**
 * @param {number[]} preorder
 * @param {number[]} inorder
 * @return {TreeNode}
 */
var buildTree = function (preorder, inorder) {
  if (!inorder.length) return null;
  const root = new TreeNode(preorder[0]);
  const tmp = preorder[0]; // 前序的第一项，即根元素
  const mid = inorder.indexOf(tmp); // 获取根元素所在的位置
  // 因为前序和中序的左右子树长度一定一样，所以preorder也可以用mid+1划分
  root.left = buildTree(preorder.slice(1, mid + 1), inorder.slice(0, mid));
  root.right = buildTree(preorder.slice(mid + 1), inorder.slice(mid + 1));
  return root;
};
```

---

如果是从后序遍历和中序遍历构建二叉树，其实也差不多，核心其实就是找到每个子树的根节点的值的位置；因为后序遍历的最后一项是数组的根元素值，所以一直以最后一项划分即可，和先序遍历没太大区别。

```js
var buildTree = function (inorder, postorder) {
  if (!inorder.length || !postorder.length) return null;
  const rootVal = postorder[postorder.length - 1]; // 最后一项，上面的是第一项
  const root = new TreeNode(rootVal);
  const idx = inorder.indexOf(rootVal);
  root.left = buildTree(inorder.slice(0, idx), postorder.slice(0, idx));
  root.right = buildTree(
    inorder.slice(idx + 1),
    postorder.slice(idx, postorder.length - 1)
  );
  return root;
};
```

## 合并二叉树

https://leetcode.cn/problems/merge-two-binary-trees/

> 给你两棵二叉树： root1 和 root2 。
> 想象一下，当你将其中一棵覆盖到另一棵之上时，两棵树上的一些节点将会重叠（而另一些不会）。你需要将这两棵树合并成一棵新二叉树。合并的规则是：如果两个节点重叠，那么将这两个节点的值相加作为合并后节点的新值；否则，不为 null 的节点将直接作为新二叉树的节点。

（其实我也不知道是怎么过的）

```js
/**
 * @param {TreeNode} root1
 * @param {TreeNode} root2
 * @return {TreeNode}
 */
var mergeTrees = function (root1, root2) {
  function dfs(node1, node2) {
    if (!node1 && !node2) return null;
    else if (!node1 && node2) return node2;
    else if (node1 && !node2) return node1;

    node1.val += node2.val; // 注意这里不要提前return了
    node1.left = dfs(node1.left, node2.left);
    node1.right = dfs(node1.right, node2.right);
    return node1;
  }
  return dfs(root1, root2);
};
```

## 二叉树的序列化和反序列化

https://leetcode.cn/problems/serialize-and-deserialize-binary-tree/

这道题的题干比较复杂，简单来说就是把二叉树输出成一个字符串（序列化），然后再把这个字符串转成二叉树（反序列化）
序列化和反序列应该是对应的。如果序列化是先序遍历，那么反序列应该采取相同的识别方式。这里有几个注意点：

1. 序列化不能只输出有效值。对于 null 也需要输出，否则反序列化不能识别哪些地方应该是 null。输出 null 只需要在递归边界时记录一个`'null'`即可，这时不仅是只有一个单子树的另一边会变成 null，叶子节点也会多出两个为 null 的节点。

2. 反序列化的思路：不能采取之前的分治法，因为这里要求构建的二叉树应该完全和序列化的相同。可以有下面这个思路：
3. 设置一个指针，依次从前到后走遍序列化的每一项
4. 如果当前指向的值是 null，那么就返回 null，让指针+1
5. 如果不是 null，创建节点，左右子节点分别是继续递归的返回值。

这个思路中，节点的选择依赖于指针。每次递归都会使指针向前走一项，因此按照先序遍历的特点，按顺序一定是`根->左->右`的顺序，恰好可以构建

代码如下：

```js
// 序列化
var serialize = function (root) {
  const nodes = [];
  function dfs(node) {
    if (!node) {
      nodes.push("null"); // 注意这里每个null都要输出
      return;
    }
    nodes.push(node.val);
    dfs(node.left);
    dfs(node.right);
  }
  dfs(root);
  return nodes.join(",");
};

// 反序列化
var deserialize = function (data) {
  const nodes = data.split(",");
  let p = 0;
  // 不需要参数，每走一步p++，相当于取到下一个应该的位置上去
  function traverse() {
    if (nodes[p] === "null") {
      p++;
      return null;
    }

    const node = new TreeNode(nodes[p++]);
    node.left = traverse();
    node.right = traverse();
    return node;
  }
  return traverse();
};
```

这道题关键是反序列化的思路，如何从一个先序遍历得到的数组还原二叉树

## 不同的二叉搜索树 II

https://leetcode.cn/problems/unique-binary-search-trees-ii/

这道题的前置问题 不同的二叉搜索树 在动态规划部分，其中第二种解法稍微改一改就可以得到这道题的解。

一样的思路，设`dfs(left,right)`返回一个数组，数组的每一项都是节点数量为`right-left`时可能的子树（就是具体的子树）。在每个递归内部都设置一个 i 从 left 到 right 遍历，分别得到左右子树的结果数组，然后二重循环这两个数组，得到的左右子树的所有组合就是当前 left 和 right 下的结果。

```js
/**
 * @param {number} n
 * @return {TreeNode[]}
 */
var generateTrees = function (n) {
  function dfs(left, right) {
    if (left > right) return [null];
    const trees = [];
    for (let i = left; i <= right; i++) {
      const leftNodes = dfs(left, i - 1);
      const rightNodes = dfs(i + 1, right);
      for (let left of leftNodes) {
        for (let right of rightNodes) {
          const node = new TreeNode(i);
          node.left = left;
          node.right = right;
          trees.push(node);
        }
      }
    }
    return trees;
  }
  return dfs(1, n);
};
```

## 翻转等价二叉树

https://leetcode.cn/problems/flip-equivalent-binary-trees/

这道题其实就是比较二叉树的升级，在原版的比较二叉树代码的基础上，增加判断“翻转”一下即可。

即，当 A 树的左子节点和 B 树的左子节点不同时，就交叉过来，比较 A 的右子节点和 B 的左子节点。如果两个都不能相同，那就一定是 false。

代码：

```js
var flipEquiv = function (root1, root2) {
  function compareTree(node1, node2) {
    if (!node1 && !node2) return true;
    else if ((!node1 && node2) || (node1 && !node2)) return false;
    if (node1.val !== node2.val) {
      return false;
    }
    // 比较左左和右右
    if (
      node1.left?.val === node2.left?.val &&
      node1.right?.val === node2.right?.val
    ) {
      return (
        compareTree(node1.left, node2.left) &&
        compareTree(node1.right, node2.right)
      );
      // 比较左右和右左
    } else if (
      node1.left?.val === node2.right?.val &&
      node1.right?.val === node2.left?.val
    ) {
      return (
        compareTree(node1.left, node2.right) &&
        compareTree(node1.right, node2.left)
      );
    } else return false;
  }
  return compareTree(root1, root2);
};
```

## 二叉树展开为链表

https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/

题目是把一个二叉树原地变为一个只有右子树的长串链表，并且顺序应该是先序遍历的顺序。

思路就是，设 flatten 函数表示展平 node 节点的所有子节点为一条右子树链；
对于一个节点，我们分别用这个函数得到展平后的左右子树，然后：

1. 把左子树删掉
2. 把左子树连到该节点的右子树位置
3. 用一个函数找到左子树的最后一个节点
4. 把右子树连到左子树的最后一个节点上
5. 返回该节点

代码如下：

```js
var flatten = function (root) {
  if (!root) return null;
  const left = flatten(root.left);
  const right = flatten(root.right);
  root.left = null; // 注意这里要先删掉，不然会重复
  root.right = left; // 先连左子树再连右子树；如果要求是后序顺序就先连右子树
  const end = findLast(root);
  end.right = right;
  return root;
};

function findLast(node) {
  // 找到左子树的最后一个节点
  while (node.right) node = node.right;
  return node;
}
```


# 回溯

## 九种基本回溯

参考https://labuladong.github.io/algo/di-san-zha-24031/bao-li-sou-96f79/hui-su-sua-56e11/

模板：

### 组合/子集

组合和子集问题在剪枝上属于同一类，区别只是在于输出不同。

#### 组合/子集——元素不重复，不可重复选

```js
const dfs = (start) => {
  if(...){
    res.push([...path])
    return
  }
  for(let i = start;i < nums.length;i++){
    path.push(...)
    dfs(i+1)
    path.pop()
  }
}
```

#### 组合/子集——元素重复，不可重复选

```js
// 数组排序情况下
const dfs = (start) => {
  if(...){
    res.push([...path])
    return
  }
  for(let i = start;i < nums.length;i++){
    if(i > start && nums[i] === nums[i-1]) continue
    path.push(...)
    dfs(i+1)
    path.pop()
  }
}

// 数组不可排序的情况
const dfs = (start) => {
  if(...){
    res.push([...path])
    return
  }
  const set = new Set()
  for(let i = start;i < nums.length;i++){
    if(set.has(nums[i])) continue
    set.add(nums[i])
    path.push(...)
    dfs(i+1)
    path.pop()
  }
}
```

#### 组合/子集——元素不重复，可重复选

```js
const dfs = (start) => {
  if(...){
    res.push([...path])
    return
  }
  for(let i = start;i < nums.length;i++){
    path.push(...)
    dfs(i) // i不用+1
    path.pop()
  }
}
```


### 排列

#### 排列——元素不重复，不可重复选


```js
const visited = new Set()
const dfs = () => {
  if(...){
    res.push([...path])
    return
  }
  for(let i = 0;i < nums.length;i++){
    if(visited.has(i)) continue
    path.push(...)
    visited.add(i)
    dfs(i+1)
    path.pop()
    visited.delete(i)
  }
}
```

#### 排列——元素重复，不可重复选

最麻烦的一种，最好单独记忆这种情况。
注意这里的visited判断是i-1而不是i

```js
const visited = new Set()
const dfs = () => {
  if(...){
    res.push([...path])
    return
  }
  for(let i = 0;i < nums.length;i++){
    if(i > 0 && nums[i] === nums[i-1] && !visited.has(i-1)) continue
    path.push(...)
    visited.add(i)
    dfs(i+1)
    path.pop()
    visited.delete(i)
  }
}
```

#### 排列——元素不重复，可重复选

```js
const dfs = () => {
  if(...){
    res.push([...path])
    return
  }
  for(let i = 0;i < nums.length;i++){
    path.push(...)
    dfs(i+1)
    path.pop()
  }
}
```


## 回溯问题的基本思路

> 回溯问题常见于题目中这样要求：找出所有情况、找出所有排列组合、遍历所有情况、走过所有格子等等。

参考
https://programmercarl.com/%E5%9B%9E%E6%BA%AF%E7%AE%97%E6%B3%95%E7%90%86%E8%AE%BA%E5%9F%BA%E7%A1%80.html#%E9%A2%98%E7%9B%AE%E5%88%86%E7%B1%BB%E5%A4%A7%E7%BA%B2%E5%A6%82%E4%B8%8B

上面的回溯说到回溯问题的基本模板是这样的：

```js
function xxx(入参) {
  前期的变量定义、缓存等准备工作

  // 定义路径栈
  const path = []

  // 进入 dfs
  dfs(起点)

  // 定义 dfs
  dfs(递归参数) {
    if(到达了递归边界) {
      结合题意处理边界逻辑，往往和 path 内容有关
      return
    }

    // 注意这里也可能不是for，如果是两三个明确的情况 ，直接递归也有可能，视题意决定
    for(遍历坑位的可选值) {
      path.push(当前选中值)
      dfs(添加一些改变的参数)
      path.pop()// 这里让已经选中的值退出，相当于树回退一层
    }
  }
}
```

实际上这并不是一种生硬的模板，而是对应了回溯问题的基本形状：**迭代表示同一层的横向展开，递归表示纵向的遍历**

根据前面 dfs 的经验，我们把递归只看成是一个简单的访问语句：

```js
dfs(){
  递归边界
  for(i){
    path.push(当前选中值)
    访问第(i+1)个
    path.pop()
  }
}
```

可以看到，迭代的部分实际上就是将所有情况横向展开，也就是**树的分叉**；迭代几次也就对应根节点展开了几个枝；这也是通过限制迭代次数从而避免重复的原因。

> ![](https://pic.imgdb.cn/item/626a9a55239250f7c5e9286e.jpg)
> 迭代实际上就是这部分

而递归则是**纵向**的延伸；和二叉树问题类似，递归的每一项可以看作是一个子节点的产生，而递归语句旁边的`path.push`和`path.pop`操作也正是对应了递归路径的记录和回溯。回溯的关键并不是循环，而是结尾的`pop()`，也就是状态的恢复；

理解了这两个地方，就可以理解各种其他问题中**去重**的方式和原因。

### 去重的几种方法

1. (仅用于组合问题)所选序列是否重复按照出现次数区分，即不希望“回头”，即不能出现`[1,2]`和`[2,1]`这样“回头”的情况。
   这种通常限制迭代的**起始位置**。比如说组合问题 I，从数组`[1,2,3,4]`中选择 2 之后，就不希望再回头到 1 了，而是只从 3、4 以及之后的数中选。  
   ![](https://img-blog.csdnimg.cn/20201123195223940.png)
   如果数组中有重复元素，用这种方法只能避免同一位置的数被重用，而不同位置上相同的数还是会被重用（即组合之间可能重复，比如出现两个`[1,1,2]`）。不过这不一定是错误，还是要根据题意分析。

   **对于组合问题**（排列和子集问题有特殊情况），这种方式一般是针对**同一个集合**内部取值的情况。如果像电话号码的字母组合那道题这种不同集合中取值的，就不需要。

```js
function dfs(val, begin) {
  //递归边界

  for (let i = begin; i < xxx.length; i++) {
    path.push();
    dfs(val, i + 1); // i作为下一个的begin
    path.pop();
  }
}
```

2. (仅用于排列问题)纵向不能重复；限制同一结果中元素出现个数，比如全排列问题，不允许一个元素使用多次。
   每次迭代到一个元素时，如果该元素已经被记录访问过，就会跳到下一个元素。注意这种方法是去重**垂直**方向的，即 visited 数组横向独立、纵向统一。
   **这种方法只适用于排列问题**；

```js
function dfs(val) {
  //递归边界

  for (let i = 0; i < xxx.length; i++) {
    if (!visited[i]) {
      path.push();
      visited[i] = true;
      dfs(val); // i作为下一个的begin
      path.pop();
      visited[i] = false;
    }
  }
}
```

3. 横向不能重复，限制所有结果中相同值的重复，即不能有`[[1,2,2],[1,2,2]]`这样两个相同的元素，但是元素内部仍然可以重复。

判断方法条件为：`nums[i] === nums[i - 1]`和`i > 0`，以及`visited[nums[i - 1]] === false`，满足这三个条件时就应当跳过。

- `nums[i] === nums[i - 1]`：当前值和前一个不能相同，即横向规定不能有重复值
- `visited[nums[i - 1]] === false`：这个很重要，是判断上一个条件是横向重复还是纵向重复；因为上一个条件既有可能是横向重复，也可以是纵向的重复；因此当这个条件为 true 时，即上一个值已经被访问过了，说明是纵向重复；反之则是横向重复。**该条件===true 判断纵向重复，==false 判断横向重复**

> 横向（数层）是不同组合中的，纵向（一个树枝）是同一个组合内的。因此纵向不重复实际上就是组合内不重复，同理横向不重复就是组合内可以重复，但是组合间不能重复

![](https://pic.imgdb.cn/item/626ab4ee239250f7c5375bd0.jpg)

```js
nums.sort((a, b) => a - b); //！！！一定要先排序
function dfs(val) {
  //递归边界

  for (let i = 0; i < xxx.length; i++) {
    // 注意visited应该存的是序号！！！
    if (!visited[i - 1] && nums[i] === nums[i - 1] && i > 0) continue;
    if (!visited[i]) {
      // 如果是排列问题，就还需要这个判断
      visited[i] = true;
      dfs(val); // i作为下一个的begin
      path.pop();
      visited[i] = false;
    }
  }
}
```

或者综合第一种 begin 的方式；当`i > begin`时，相当于当前遍历的位置是其他同层元素，和`visited[nums[i - 1]] === false`效果相同。

```js
if (i > begin && nums[i] === nums[i - 1]) continue
```

> 注意第三种去重方法判断的前提是数组**必须有序**。如果无序的情况下，就不能用`nums[i] === nums[i - 1]`这种形式了，因为相同是数肯定不相邻
> 所以应该采用一个**每次递归（也就是每一层）都会更新的 set**，用 set 存储本层的重复数字，然后判断`set.has()`就可以。参考下面的递增子序列题目

### 去重方法的选择

首先，根据回溯问题的类型不同，去重的选择也不一样。大致分为四种类型的回溯：

1. **组合问题**。
   组合问题的特点是顺序不同的序列不能重复出现，即不能出现`[1,2]`和`[2,1]`这样“回头”的情况。因此，组合问题基本上都需要利用 begin 限制起始位置保证去重，即方法 1；
   具体来说，分为以下几种情况（都要求不能有重复组合）：

- 从一个集合中选组合，且集合没有重复元素：最基本的情况，begin 去重即可。
- 从一个集合中选组合，但集合有重复元素：给数组排序，并用第三种去重。
- 从一个集合中选组合，集合没有重复元素，但一个元素可以用多次：每次递归传 i 而不是 i+1（即`dfs(i)`而不是`dfs(i+1)`），这样就可以一直用一个数。
- 从多个集合中选组合：不能用 begin，起始必须是 0，每层选取的集合要重新计算（每层不一样）

2. **分割问题**。
   分割问题不需要去重，核心思想是分割出来的部分符合条件之后，把分割之后的再传下去继续分割。
3. **子集问题**。
   子集和组合是一类问题，去重方式是一样的，即，不能出现像`[1,2]`和`[2,1]`这样顺序不同的组合。因此需要用 begin 限制起始位置；
   子集问题和组合的主要区别在于子集取得是每个节点，而组合只会取叶子节点；不需要任何剪枝,因为子集就是要遍历整棵树
   子集问题的集合内部如果没有重复元素，那子集之间一定不会重复。反之，就需要第三种方法去重。
   另外子集问题有一类特殊的题，即子序列问题。这种要求数组不能排序，但是又要使用第三种方法去重；方法是每层创建一个 used 数组记录本层的重复情况，用 used 数组代替`visited[i-1]`即可。（参考递增子序列）
4. **排列问题**。
   排列问题元素顺序可以改变，因此不能通过 begin 限制起始，而要使用 visited 限制**同一颗树下**的元素不会纵向重复。如果排列问题的内部元素有重复但是总体不能有重复（比如允许`[1,2,2]`但不允许出现两个`[1,2,2]`），就也要考虑最后一种去重方法。
   排列问题的分类：

- 从一个集合中选排列，且集合没有重复元素：最基本的情况，用 visited 去重保证一个元素不被使用多次即可。
- 从一个集合中选排列，但集合有重复元素：给数组排序，并用第三种去重；注意要同时使用第三种和 visited
- 从一个集合中选排列，集合没有重复元素，但一个元素可以用多次：去掉 visited 数组限制即可

### 特殊回溯问题

回溯问题的模板不一定都是递归+迭代这种类型的。有些情况下并不需要迭代。
迭代的核心目标是**把所有情况都放入递归中执行一次**，以达到遍历所有情况的目的；当一个回溯问题只有少数几种（通常是两种）情况时，就不需要迭代来遍历了。同样，如果递归的情况不能通过迭代线性展开（比如岛屿问题），也不需要。
总之回溯问题的核心部分其实还是**递归**和**状态恢复**，其他的地方都是在为这两个核心提供帮助。

```js
function dfs(...){
  if(...) return

  // 状态改变
  if(...) dfs(...)
  else dfs(...)
  // 状态恢复
}
```

## 全排列问题

### 基本全排列

> 给定一个没有重复数字的序列，返回其所有可能的全排列。
> 示例：  
> 输入: [1,2,3]
> 输出: [
> [1,2,3],
> [1,3,2],
> [2,1,3],
> [2,3,1],
> [3,1,2],
> [3,2,1]
> ]

像全排列这样的穷举，或者有一定限制的穷举，都可以用 DFS、递归回溯的思想。

这类问题的特点都是，穷举之中有不变化的部分；比如全排列问题不变的是三个数字的位置。三个位置可能是 1/2/3 任意一个数字，这样就可以构建出一个树：
![](https://pic.imgdb.cn/item/6266c1cc239250f7c554635b.jpg)

然后按照树的先序遍历方式，就可以得到结果。当然并不需要构造一棵树。

代码如下：

```js
const permute = function (nums) {
  // 缓存数组的长度
  const len = nums.length;
  // curr 变量用来记录当前的排列内容
  const curr = [];
  // res 用来记录所有的排列顺序
  const res = [];
  // visited 用来避免重复使用同一个数字
  const visited = {};
  // 定义 dfs 函数，入参是坑位的索引（从 0 计数）
  function dfs(nth) {
    // 若遍历到了不存在的坑位（第 len+1 个），则触碰递归边界返回
    if (nth === len) {
      // 此时前 len 个坑位已经填满，将对应的排列记录下来
      res.push(curr.slice());
      return;
    }
    // 检查手里剩下的数字有哪些
    // 循环套递归是常见的形式,循环的作用就是让DFS过程能"返回"
    for (let i = 0; i < len; i++) {
      // 若 nums[i] 之前没被其它坑位用过，则可以理解为“这个数字剩下了”
      if (!visited[nums[i]]) {
        // 给 nums[i] 打个“已用过”的标
        visited[nums[i]] = 1;
        // 将nums[i]推入当前排列
        curr.push(nums[i]);
        // 基于这个排列继续往下一个坑走去
        dfs(nth + 1);
        // nums[i]让出当前坑位
        curr.pop();
        // 下掉“已用过”标识
        visited[nums[i]] = 0;
      }
    }
  }
  // 从索引为 0 的坑位（也就是第一个坑位）开始 dfs
  dfs(0);
  return res;
};
```

这段代码很奇怪的地方是有一个“循环套递归”，并且递归的前后恰好是入栈、出栈结果的时候。

其实这正是这类题的“模板解法”。在递归产生的不同函数上下文中，每个的遍历次数不同。
以这道题为例，当 dfs 执行到第三个时，前两个的上下文的遍历次数分别是 1、2；这时当第三个返回时，上一个的循环还会执行一次，就相当于从树的底部“退回”了一个位置，并把 3 这个数字让了出来。

因此这类穷举相关的题目有一个通用模板，遇到这种问题可以先试着套一下：

```js
function xxx(入参) {
  前期的变量定义、缓存等准备工作

  // 定义路径栈
  const path = []

  // 进入 dfs
  dfs(起点)

  // 定义 dfs
  dfs(递归参数) {
    if(到达了递归边界) {
      结合题意处理边界逻辑，往往和 path 内容有关
      return
    }

    // 注意这里也可能不是 for，视题意决定
    for(遍历坑位的可选值) {
      path.push(当前选中值)
      处理坑位本身的相关逻辑,一般就是递归
      path.pop()// 这里让已经选中的值退出，相当于树回退一层
    }
  }
}
```

### 全排列 2（剪枝）

> 给定一个可包含重复数字的序列 nums ，按任意顺序 返回所有不重复的全排列。
> 示例 1：
> 输入：nums = [1,1,2]
> 输出：
> [[1,1,2],
[1,2,1],
[2,1,1]]

这个题和全排列 1 的区别在于，每个结果允许数组内的数字重复，即给定的 nums 中有重复的数字。
因此这是一个横向剪枝的问题，即在保证一个数字不会纵向重复的情况下，进行横向剪枝去掉可能重复的组合。
![](https://pic.imgdb.cn/item/628108d109475431293e2fd6.png)

利用上面说的去重的第三种方法即可。

代码如下：

```js
/**
 * @param {number[]} nums
 * @return {number[][]}
 */
var permuteUnique = function (nums) {
  const curr = []; // 当前排列
  const res = []; // 结果数组
  const visit = {};
  nums.sort((a, b) => a - b);

  dfs(0);
  function dfs(index) {
    if (index === nums.length) {
      res.push([...curr]);
      return;
    }

    for (let i = 0; i < nums.length; i++) {
      if (i > 0 && nums[i] === nums[i - 1] && !visit[i - 1]) continue;
      if (!visit[i]) {
        visit[i] = true;
        curr.push(nums[i]);
        dfs(index + 1);
        curr.pop();
        visit[i] = false;
      }
    }
  }
  return res;
};
```

## 组合问题

### 基本组合问题

> 给定两个整数 n 和 k，返回 1 ... n 中所有可能的 k 个数的组合。
> 示例:
> 输入: n = 4, k = 2
> 输出:
> [
> [2,4],
> [3,4],
> [2,3],
> [1,2],
> [1,3],
> [1,4],
> ]

最基本的组合问题，当递归次数等于 k 时记录即可。
因为同一个位置上的数字不能用多次，因此需要纵向剪枝；并且同样开头的数字也只能统计一次，所以需要起始序号剪枝。

```js
/**
 * @param {number} n
 * @param {number} k
 * @return {number[][]}
 */
var combine = function (n, k) {
  const res = [];
  const curr = [];
  const nums = [];
  for (let i = 0; i < n; i++) {
    // 创建数组
    nums[i] = i + 1;
  }
  let times = 0;
  function dfs(nth, begin) {
    if (nth === k) {
      res.push([...curr]);
      return;
    }
    for (let i = begin; i < nums.length; i++) {
        curr.push(nums[i]);
        times++;
        dfs(times, i);
        times--;
        curr.pop();
      }
    }
  }
  dfs(0, 0);
  return res;
};
```

### 组合总和

> 给你一个 无重复元素 的整数数组  candidates 和一个目标整数  target ，找出 candidates  中可以使数字和为目标数  target 的 所有   不同组合 ，并以列表形式返回。你可以按 任意顺序 返回这些组合。
> candidates 中的 同一个 数字可以 无限制重复被选取 。如果至少一个数字的被选数量不同，则两种组合是不同的。

基本组合问题，设定一个 sum 变量每次传入，每次加上当前值，当 sum 值等于 target 时记录。

那怎么样保证每个数字可以被重复选取呢？关键就是每次递归后的`i`不会再增加，传入的参数是 i 而不是 i+1

```js
/**
 * @param {number[]} candidates
 * @param {number} target
 * @return {number[][]}
 */
var combinationSum = function (candidates, target) {
  const res = [];
  const curr = [];

  function dfs(sum, begin) {
    if (sum === target) {
      res.push([...curr]);
      return;
    } else if (sum > target) return;

    for (let i = begin; i < candidates.length; i++) {
      curr.push(candidates[i]);
      sum += candidates[i];
      dfs(sum, i); // 关键点,这里是i而不是i+1,这样每次还能选到同一个数字
      sum -= candidates[i];
      curr.pop();
    }
  }
  dfs(0, 0);

  return res;
};
```

### 组合总和 2

> 给定一个候选人编号的集合  candidates  和一个目标数  target ，找出  candidates  中所有可以使数字和为  target  的组合。
> candidates  中的每个数字在每个组合中只能使用 一次  。解集不能包含重复的组合。

显然和上一个的区别在于每个数字在每个组合中只能使用一次，但是每个组合中允许重复数字，最终结果不允许有重复组合；

因此使用第三种剪枝就可以。即，横向不允许重复，但是纵向可以（`visited[nums[i-1]] === false`时跳过）

这道题有一个特殊的解法，就是使用一个 prev 变量保存前一个的值；如果前一个值和当前`candidates[i]`相等，就直接跳过。

```js
/**
 * @param {number[]} candidates
 * @param {number} target
 * @return {number[][]}
 */
var combinationSum2 = function (candidates, target) {
  const res = [];
  const curr = [];
  candidates.sort((a, b) => a - b);

  function dfs(sum, begin) {
    if (sum > target) return;
    if (sum === target) {
      res.push([...curr]);
      return;
    }
    for (let i = begin; i < candidates.length; i++) {
      if (candidates[i] === candidates[i - 1] && i > begin) continue;
      curr.push(candidates[i]);
      dfs(sum + candidates[i], i + 1);
      curr.pop();
    }
  }
  dfs(0, 0);

  return res;
};
```

### 电话号码的字母组合

https://leetcode.cn/problems/letter-combinations-of-a-phone-number/

> 给定一个仅包含数字  2-9  的字符串，返回所有它能表示的字母组合。答案可以按 任意顺序 返回。
> 给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。
> 示例 1：
> 输入：digits = "23"
> 输出：["ad","ae","af","bd","be","bf","cd","ce","cf"]

这道题和前面的题有一点不一样的地方，主要在于每层取值取的是不同集合中的元素。
前面的大多数组合、排列问题都是在一个数组中进行的，因此取值只用考虑遍历这一个数组即可；但是这道题每次都应该是取下一个字符串中的任意值。所以思路需要变通一下：

1. 设定数字到字母的映射关系
2. 每次回溯根据数字的第 i 个，确定本次要遍历的字符串；比如示例的 23，相当于先遍历 abc，再遍历 def。

代码如下：

```js
/**
 * @param {string} digits
 * @return {string[]}
 */
var letterCombinations = function (digits) {
  if (digits === "") return [];
  const dic = [
    "",
    "",
    "abc",
    "def",
    "ghi",
    "jkl",
    "mno",
    "pqrs",
    "tuv",
    "wxyz",
  ];
  if (digits.length === 1) return dic[+digits].split("");
  const res = [];
  const path = [];
  function dfs(idx) {
    if (idx === digits.length) {
      res.push(path.join(""));
      return;
    }
    const str = dic[digits[idx]]; // 关键在这一步，每层的遍历对象不是同一个集合
    for (let s of str) {
      path.push(s);
      dfs(idx + 1);
      path.pop();
    }
  }
  dfs(0);
  return res;
};
```

### 连续差相同的数字

https://leetcode.cn/problems/numbers-with-same-consecutive-differences/

> 返回所有长度为 n 且满足其每两个连续位上的数字之间的差的绝对值为 k 的 非负整数 。
> 请注意，除了 数字 0 本身之外，答案中的每个数字都 不能 有前导零。例如，01 有一个前导零，所以是无效的；但 0  是有效的。
> 你可以按 任何顺序 返回答案。 
> 示例 1：
> 输入：n = 3, k = 7
> 输出：[181,292,707,818,929]
> 解释：注意，070 不是一个有效的数字，因为它有前导零。

思路：回溯，纵向走每一位的数字，横向走从 0-9 的数字；

- 如果前面的没有数字，就放入当前数
- 如果前面有数字，就计算当前数和前面的差，如果等于 k 就放入，否则就 continue

代码：

```js
var numsSameConsecDiff = function (n, k) {
  const res = [];
  const nums = [];
  function dfs() {
    if (nums.length === n) {
      res.push(+nums.join(""));
      return;
    }
    for (let i = 0; i < 10; i++) {
      if (i === 0 && nums.length === 0) continue; // 跳过前导0
      if (nums.length === 0 || Math.abs(i - nums[nums.length - 1]) === k) {
        nums.push(i);
        dfs();
        nums.pop();
      }
    }
  }
  dfs();
  return res;
};
```

## 子集问题

### 基本子集问题

> 给定一组不含重复元素的整数数组 nums，返回该数组所有可能的不重复子集

首先先按照树形思维构建；这道题中不变的部分是是否选中某个数字。

比如选中 1、不选 2、选中 3，那么结果就是[1,3]，依次类推就是所有选择。树的两个叶子分别代表选或不选该层对应的数字，每一层恰好就是这个数字。

![](https://pic.imgdb.cn/item/6266c73d239250f7c5647e11.jpg)

按照上面的模板，可以尝试构建如下：

```js
const subsets = function (nums) {
  const ans = [];
  const curr = [];

  dfs(0);
  function dfs(index) {
    // 每次进入，都意味着组合内容更新了一次，故直接推入结果数组
    ans.push(curr.slice());
    // 从当前数字的索引开始，遍历 nums
    for (let i = index; i < nums.length; i++) {
      // 这是当前数字存在于组合中的情况
      curr.push(nums[i]);
      // 基于当前数字存在于组合中的情况，进一步 dfs
      dfs(i + 1);
      // 这是当前数字不存在与组合中的情况
      curr.pop();
    }
  }
  return ans;
};
```

注意这道题虽然有显式的递归边界（遍历三层），但是其实并不需要，因为循环直接从当前的 index（即树的层数）开始，跳过了前面的情况。

如果我们再给这个解法显式加上一个递归条件，比如规定`curr.length < k`，这就是一种“回溯”，即到达某种情况后就提前返回，并不是每次都遍历完。

```js
const combine = function (n, k) {
  // 初始化结果数组
  const res = [];
  // 初始化组合数组
  const subset = [];
  // 进入 dfs，起始数字是1
  dfs(1);

  // 定义 dfs 函数，入参是当前遍历到的数字
  function dfs(index) {
    if (subset.length === k) {
      res.push(subset.slice());
      return;
    }
    // 从当前数字的值开始，遍历 index-n 之间的所有数字
    for (let i = index; i <= n; i++) {
      // 这是当前数字存在于组合中的情况
      subset.push(i);
      // 基于当前数字存在于组合中的情况，进一步 dfs
      dfs(i + 1);
      // 这是当前数字不存在与组合中的情况
      subset.pop();
    }
  }
  // 返回结果数组
  return res;
};
```

### 递增子序列

> 给定一个整型数组, 你的任务是找到所有该数组的递增子序列，递增子序列的长度至少是 2。
> 示例:
>
> 输入: [4, 6, 7, 7]
> 输出: [[4, 6], [4, 7], [4, 6, 7], [4, 6, 7, 7], [6, 7], [6, 7, 7], [7,7], [4,7,7]]

这道题有两个关键点：

1. 数组不能排序，所以不能直接使用第三种去重，需要用一个每层都会更新的 set 去重；
2. 要求必须是递增序列，所以可以让元素大于 path 的最后一个元素时才继续，否则之间跳过。

```js
/**
 * @param {number[]} nums
 * @return {number[][]}
 */
var findSubsequences = function (nums) {
  const res = [];
  const path = [];
  function dfs(nth) {
    if (path.length >= 2) {
      res.push([...path]);
    } else if (nth >= nums.length) {
      return;
    }
    const set = new Set();
    for (let i = nth; i < nums.length; i++) {
      if (i > 0 && set.has(nums[i])) continue;
      if (path.length >= 1 && nums[i] < path[path.length - 1]) continue;
      set.add(nums[i]);
      path.push(nums[i]);
      dfs(i + 1);
      path.pop();
    }
  }
  dfs(0);
  console.log(res);
};
```

### 划分 K 个相等的子集

https://leetcode.cn/problems/partition-to-k-equal-sum-subsets/

> 给定一个整数数组   nums 和一个正整数 k，找出是否有可能把这个数组分成 k 个非空子集，其总和都相等。
> 示例 1：
> 输入： nums = [4, 3, 2, 3, 5, 2, 1], k = 4
> 输出： True
> 说明： 有可能将其分成 4 个子集（5），（1,4），（2,3），（2,3）等于总和。

这道题可以站在两个角度思考，即站在每个数字上，或者每个要放数字的桶上（即 k 个桶，要放进去数字）。

- 对于每个数字，可能放进任何一个桶中。那么就应该遍历数组中的数字，尝试放到每个桶中；
  - 如果当前这个数字加上桶内的值超过了 sum/k，就跳过这个桶找下一个
  - 如果没超过，就加上这个数字，然后遍历下一个数字
    最后当所有数字都遍历完之后，检查是否每个桶的和都是 sum/k 即可。
    因此这种思路下应该在外层递归遍历数字，内层循环遍历每个桶。因为每个数字都需要从第一个桶开始尝试，所以不存在数字重复的问题，即不需要去重等操作
- 对于每个桶，需要遍历 nums 中的所有数字，决定是否把当前数字装进桶中；当装满一个桶之后，还要装下一个桶，直到所有桶都装满为止。
  - 如果当前数字加上桶内和小于 sum/k，就加上该数字，然后递归下一个桶
  - 如果超过 sum/k 就继续找下一个数字
    即递归遍历每个桶，循环遍历每个数字。这时就要考虑数字的重复情况了。对于数组中的数字（某个位置），一旦放入一个桶就一定不能再放入另一个桶，因此每次放入一个数字都要记录是否被选中。

这里给出第一种思路的解法：

```js
var canPartitionKSubsets = function (nums, k) {
  const sum = nums.reduce((a, b) => a + b);
  const target = sum / k;
  const buckets = new Array(k).fill(0);
  nums.sort((a, b) => b - a); // 排序能减少时间复杂度，尽可能让大的数先进入桶，这样会加快总体进度

  let flag = false;
  function dfs(index) {
    // 这个index就是第几个数字
    if (index === nums.length) {
      // 遍历完所有数字
      flag = buckets.reduce(
        // 检查每个桶的值是否都到了target
        (pre, cur) => pre && cur === target,
        buckets[0] === target
      );
      return;
    }

    for (let i = 0; i < buckets.length; i++) {
      // 对于每个数字都要从第一个桶开始，因此这里都是从0开始
      if (buckets[i] + nums[index] > target) continue;
      buckets[i] += nums[index];
      dfs(index + 1); // 如果某个桶能放入当前数字，就取 下一个
      buckets[i] -= nums[index];
    }
  }
  dfs(0);
  return flag;
};
```

## 分割问题

分割问题是变种的回溯问题；常见回溯问题一般是记录路径，累加求值；但是分割问题通常是把字符串分割递归下去。
具体来说，分割问题的通常思路如下：

1. 在迭代中，从某一项开始取前 i 个字符
2. 判断该字符是否符合要求

- 不符合，继续迭代
- 符合，把分割后剩下的串传递下去。

```js
function dfs(str, times) {
  if (临界条件) {
    return;
  }

  for (let i = 0; i < str.length; i++) {
    let part = str.slice(0, i);
    if (isValid(part)) {
      curr.push(part);
      dfs(str.slice(i + 1)); // 把分割后剩下的继续递归分割
      curr.pop();
    }
  }
}
```

> 关于回溯分割和动态规划分割的区别：
> 回溯分割可以针对绝大多数分割情况，但是动态规划分割一般只能用于给定分割结果的统计。
> 比如给定一个字符串和要分割成的几个子串，可以把要分割的字串作为物品，每个字符作为背包容量计算，每次分割一部分字符串和子串匹配。

### 分割回文串

> 给定一个字符串 s，将 s 分割成一些子串，使每个子串都是回文串。
> 返回 s 所有可能的分割方案。
> 示例: 输入: "aab" 输出: [ ["aa","b"], ["a","a","b"] ]

思路：先切割子串，对每个子串判断回文，如果回文就记录。

关键是切割字串的方法。切割子串思路和回溯记录路径类似：

1. 遍历每一项，把整个子串先从头到最后一个切割
2. 如果子串是回文串，就记录，并且把**切割之后剩下的部分**递归下去。
   思路可以参考这张图：
   ![](https://pic.imgdb.cn/item/6281198609475431297e2c6e.jpg)

代码：

```js
/**
 * @param {string} s
 * @return {string[][]}
 */
var partition = function (s) {
  const curr = [];
  const res = [];
  const visit = {};

  function dfs(str) {
    if (str === "") {
      res.push([...curr]);
      return;
    }
    for (let i = 0; i < str.length; i++) {
      const part = str.slice(0, i + 1);
      if (isPalindrome(part)) {
        curr.push(part);
        dfs(str.slice(i + 1));
        curr.pop();
      }
    }
  }

  dfs(s);
  console.log(res);
  function isPalindrome(str) {
    const strArr = str.split("");
    let i = 0;
    let j = strArr.length - 1;
    while (i <= j) {
      if (strArr[i] === strArr[j]) {
        i++;
        j--;
      } else {
        return false;
      }
    }
    return true;
  }
};
```

### 有效 ip 地址

> 给定一个只包含数字的字符串，复原它并返回所有可能的 IP 地址格式。
> 有效的 IP 地址 正好由四个整数（每个整数位于 0 到 255 之间组成，且不能含有前导 0），整数之间用 '.' 分隔。
> 例如："0.1.2.201" 和 "192.168.1.1" 是 有效的 IP 地址，但是 "0.011.255.245"、"192.168.1.312" 和 "192.168@1.1" 是 无效的 IP 地址。
> 示例 1：
> 输入：s = "25525511135"
> 输出：["255.255.11.135","255.255.111.35"]

思路和分割回文串类似，先从头开始分割，如果分割到符合单个 IP 的情况，就存下来并把后面的继续递归。

```js
/**
 * @param {string} s
 * @return {string[]}
 */
var restoreIpAddresses = function (s) {
  const curr = [];
  const res = [];

  function dfs(str, times) {
    if (str === "" && times === 4) {
      res.push(curr.join("."));
      return;
    }

    for (let i = 0; i < str.length; i++) {
      let part = str.slice(0, i + 1);
      if (times >= 4) part = str.slice(i + 1);
      if (isValid(part)) {
        curr.push(part);
        times++;
        dfs(str.slice(i + 1), times);
        times--;
        curr.pop();
      }
    }
  }
  dfs(s, 0);
  return res;
  function isValid(str) {
    return (
      str !== "" &&
      Number(str) <= 255 &&
      Number(str).toString().length === str.length
    );
  }
};
```

### 把数字翻译成字符串

> 给定一个数字，我们按照如下规则把它翻译为字符串：0 翻译成 “a” ，1 翻译成 “b”，……，11 翻译成 “l”，……，25 翻译成 “z”。一个数字可能有多个翻译。请编程实现一个函数，用来计算一个数字有多少种不同的翻译方法。
> 示例 1:
> 输入: 12258
> 输出: 5
> 解释: 12258 有 5 种不同的翻译，分别是"bccfi", "bwfi", "bczi", "mcfi"和"mzi"

一个分割问题，分割方法和回文串一样；关键在于分割之后的数字要小于 26 才能记录。
因为只需要知道次数，所以只需要记录次数就行

```js
/**
 * @param {number} num
 * @return {number}
 */
var translateNum = function (num) {
  const nums = num.toString();
  if (nums.length === 1) return 1;
  let cnt = 0;
  function dfs(str) {
    if (str.length === 0) {
      cnt++;
      return;
    }
    for (let i = 0; i < str.length; i++) {
      const part = str.slice(0, i + 1);
      if ((+part).toString().length !== part.length) continue;
      if (+part < 26 && +part >= 0) {
        dfs(str.slice(i + 1));
      }
    }
  }
  dfs(nums);
  return cnt;
};
```

## 岛屿问题

岛屿问题有个有趣的思考方式，即对题目反过来分析。
比如求封闭岛屿数量，那就先求不封闭的数量，然后把不封闭的淹掉；求子岛屿的数量，那就先求不是子岛屿的数量，然后把不是子岛屿的淹掉。
总之题干如果条件不好直接用，就可以试着反过来，把反过来的去掉，就可以得到正的。

### 岛屿数量

> 给你一个由  '1'（陆地）和 '0'（水）组成的的二维网格，请你计算网格中岛屿的数量。
> 岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。
> 此外，你可以假设该网格的四条边均被水包围。

```js
/**
 * @param {character[][]} grid
 * @return {number}
 */
var numIslands = function (grid) {
  let islands = 0;
  const m = grid[0].length;
  const n = grid.length;

  function dfs(i, j) {
    if (i < 0 || j < 0 || i >= n || j >= m) return; // 超出边界
    else if (grid[i][j] === "0") {
      // 遍历到的是0就回溯
      return;
    }
    // 这道题的关键就在这里,遍历之后直接就把这一个变为0
    // 这样既完成了去重,同时下面统计岛屿数量时,遍历过的一整块都会变成'0'
    grid[i][j] = "0";

    dfs(i + 1, j);
    dfs(i, j + 1);
    dfs(i - 1, j);
    dfs(i, j - 1);
  }

  for (let i = 0; i < n; i++) {
    for (let j = 0; j < m; j++) {
      // 从每个点开始执行一次递归
      // 因为遍历过的地方会置0，因此实际上只会调用 岛屿数量 次dfs
      if (grid[i][j] === "1") {
        dfs(i, j);
        islands++;
      }
    }
  }
  return islands;
};
```

### 最大岛屿面积

https://leetcode.cn/problems/max-area-of-island/

和上一道题类似

```js
/**
 * @param {number[][]} grid
 * @return {number}
 */
var maxAreaOfIsland = function (grid) {
  const m = grid[0].length;
  const n = grid.length;

  let max = 0;

  function dfs(i, j) {
    if (i < 0 || j < 0 || i >= n || j >= m) return 0;
    else if (grid[i][j] === 0) {
      return 0;
    }

    grid[i][j] = 0;

    // 这里+1的方法类似二叉树计算深度的方法；遍历该点四周，并加上当前点的面积（就是1）
    return dfs(i + 1, j) + dfs(i, j + 1) + dfs(i - 1, j) + dfs(i, j - 1) + 1;
  }

  for (let i = 0; i < n; i++) {
    for (let j = 0; j < m; j++) {
      if (grid[i][j] === 1) {
        let area = dfs(i, j);
        max = max > area ? max : area;
      }
    }
  }
  return max;
};
```

### 封闭岛屿数量

> 二维矩阵 grid  由 0 （土地）和 1 （水）组成。岛是由最大的 4 个方向连通的 0  组成的群，封闭岛是一个   完全 由 1 包围（左、上、右、下）的岛。
> 请返回 封闭岛屿 的数目。

这道题其实就是岛屿数量的改动。封闭岛屿的数量 = 岛屿数量 - 靠边的岛屿数量，因此只要减去靠边岛屿，得到的剩下岛屿数量就是总岛屿数量。
实际上计算岛屿数量就是在“淹没”一个独立的岛屿，一次 dfs 执行完成之后，一块岛屿就被完全淹没了。因此可以选择把所有靠边的岛屿（`grid[0][j]`、`grid[i][0]`等）都淹没掉，再计算剩下的即可。

```js
/**
 * @param {number[][]} grid
 * @return {number}
 */
var closedIsland = function (grid) {
  let islands = 0;
  const m = grid[0].length;
  const n = grid.length;

  function dfs(i, j) {
    if (i < 0 || j < 0 || i >= n || j >= m) return;
    else if (grid[i][j] === 1) {
      return;
    }
    grid[i][j] = 1;
    dfs(i + 1, j);
    dfs(i, j + 1);
    dfs(i - 1, j);
    dfs(i, j - 1);
  }

  for (let j = 0; j < m; j++) {
    // 淹掉上下边界
    dfs(0, j);
    dfs(n - 1, j);
  }
  for (let i = 0; i < n; i++) {
    // 淹掉左右边界
    dfs(i, 0);
    dfs(i, m - 1);
  }
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < m; j++) {
      if (grid[i][j] === 0) {
        dfs(i, j);
        islands++;
      }
    }
  }
  return islands;
};
```

### 被围绕的区域

https://leetcode.cn/problems/surrounded-regions/

这道题和上面的封闭岛屿数量很像。唯一的区别是，这里填充的时候不能填充和边界相连的部分，因此关键点就是怎么找到这些“和边界相连的'O'”

题解的思路是，从边界的每个位置出发，正常回溯，把所有 O 都替换为不同于 X 的其他字符（比如说'A'），表示这些 O 是特殊的边界区域。然后在后序的填充时，跳过这些字符（'A'）即可。

```js
// fill函数就是上面常规的dfs填充函数
for (let i = 0; i < n; i++) {
  // 填充和左右边界相连的区域
  fillA(i, 0);
  fillA(i, m - 1);
}
for (let j = 0; j < m; j++) {
  // 填充和上下边界相连的区域
  fillA(0, j);
  fillA(n - 1, j);
}
```

代码太长就不放了，但是这个思路一定要记好。

## 括号生成

https://leetcode.cn/problems/generate-parentheses/

> 数字 n  代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 有效的 括号组合。
> 示例 1：
> 输入：n = 3
> 输出：["((()))","(()())","(())()","()(())","()()()"]

> 这道题第一眼看上去没想到用回溯，而是想到判断括号有效用到的栈。但是这种需要找到**所有组合**的题目，一般都是回溯。

思路和上面的回溯有点不一样，递归的时候需要判断当前放入括号是否合理。关键就在于，怎么判断当前应该放入哪个括号。

- 首先左括号和右括号各自的数量都不能大于 n
- 其次，为了右括号能和左括号匹配，右括号的数量不能大于左括号。

所以在回溯过程中，要记录左右括号的数量，满足上面的条件再加入。

```js
/**
 * @param {number} n
 * @return {string[]}
 */
var generateParenthesis = function (n) {
  const res = [];
  function dfs(str, left, right) {
    if (str.length === 2 * n) {
      res.push(str);
      return;
    }
    if (left < n) dfs(str + "(", left + 1, right);
    if (right < left) dfs(str + ")", left, right + 1);
  }
  dfs("", 0, 0);
  return res;
};
```

## 24 点问题

给出 4 个 1-10 的数字，通过加减乘除运算，得到数字为 24 就算胜利,除法指实数除法运算,运算符仅允许出现在两个数字之间,本题对数字选取顺序无要求，但每个数字仅允许使用一次，且需考虑括号运算
此题允许数字重复，如 3 3 4 4 为合法输入，此输入一共有两个 3，但是每个数字只允许使用一次，则运算过程中两个 3 都被选取并进行对应的计算操作。
输入描述：
读入 4 个[1,10]的整数，数字允许重复，测试用例保证无异常数字。

这道题有两个关键：

1. 一开始想的回溯方向是回溯遍历所有可能的运算符组合，即三个位置的运算符，然后根据运算符计算具体的值。但是这种方法显然没考虑到括号的情况。

正确的思考路线是以数字为回溯对象，每次遍历四个数字，然后对当前的已有的运算结果执行四种运算（加减乘除）。

2. 这道题其实是一个排列问题，即，这四个数的顺序是完全可以乱的，而且顺序不同结果也不同。比如`7 2 1 10`这四个数，下面这几种情况的运算结果是不一样的：

```
7 + 2 * 1 + 10
7 + 2 * 10 + 1
```

这也就说明这道题其实是排列问题，因为即使选择的运算符相同，数字顺序不同结果也不同。

```js
function solution(nums) {
  let flag = false;
  const visited = new Array(nums.length).fill(false);
  function dfs(level, sum) {
    if (level === 3) {
      if (sum === 24) {
        flag = true;
      }
      return;
    }
    // 这里的循环原因是这四个数是可以任意修改位置的，即排列问题
    for (let i = 0; i < 4; i++) {
      if (visited[i]) continue;
      visited[i] = true;
      dfs(level + 1, sum + nums[i]);
      dfs(level + 1, sum - nums[i]);
      dfs(level + 1, sum * nums[i]);
      dfs(level + 1, sum / nums[i]);
      visited[i] = false;
    }
  }
  dfs(0, nums[0]);
  return flag;
}
```

# 排序算法

这些算法没什么好说的，记就完事了。
<img src="https://pic1.imgdb.cn/item/635a55fa16f2c2beb1158fe7.jpg"  />

## 冒泡排序

冒泡排序的原理是比较相邻的两个元素，如果不符合顺序（前大于后）就交换这两个的位置。
因为每次交换都会把最大的一个元素放到最后的位置，因此实际上每次内层遍历的结束都是把最大的元素排好了序。所以内层遍历的结尾位置可以每次向前一个。

```js
const bubbleSort = (nums) => {
  for (let i = 0; i < nums.length; i++) {
    for (let j = 0; j < nums.length - i - 1; j++) {
      if (nums[j + 1] < nums[j]) swap(nums, j, j + 1);
    }
  }
};
```

冒泡排序的最优解,即时间复杂度 O(n)；
对应完全有序的情况，这时只需要遍历一次

```js
const betterBubbleSort = (nums) => {
  for (let i = 0; i < nums.length; i++) {
    let flag = false;
    for (let j = 0; j < nums.length - i - 1; j++) {
      if (nums[j + 1] < nums[j]) swap(nums, j, j + 1);
      // 如果发生一次交换就破功了
      flag = true;
    }
    // 如果一次都没有交换,说明数组有序,直接返回,这时只遍历了一次
    if (!flag) return nums;
  }
  return nums;
};
```

冒泡排序的复杂度：

- 最好时间复杂度：它对应的是数组本身有序这种情况。在这种情况下，我们只需要作比较（n-1 次），而不需要做交换。时间复杂度为 O(n)
- 最坏时间复杂度： 它对应的是数组完全逆序这种情况。在这种情况下，每一轮内层循环都要执行，重复的总次数是 n(n-1)/2 次，因此时间复杂度是 O(n^2)
- 平均时间复杂度： O(n^2)

## 选择排序

循环遍历数组，每次都找出当前范围内的最小值，把它放在当前范围的头部；然后缩小排序范围，继续重复以上操作，直至数组完全有序为止。

```js
const selectionSort = (nums) => {
  for (let i = 0; i < nums.length; i++) {
    let minIndex = i;
    for (let j = i; j < arr.length; j++) {
      if (arr[j] < arr[minIndex]) {
        minIndex = j;
      }
    }
    swap(nums, minIndex, i);
  }
  return nums;
};
```

选择排序的复杂度都是 O(n^2)

## 插入排序

插入排序所有的操作都基于一个这样的前提：当前元素前面的序列是有序的。基于这个前提，从后往前去寻找当前元素在前面那个序列里的正确位置；

即把当前选中的元素和前面的依次比较，如果小于前面的元素，就把前面的元素往后移一位，直到大于前面元素为止。

实际操作上，插入排序假设第一个数是有序的，从第二个数开始排序。

```js
const insertionSort = (nums) => {
  let tmp = null;
  for (let i = 1; i < nums.length; i++) {
    let j = i;
    tmp = nums[i];
    while (tmp < nums[j - 1] && j > 0) {
      nums[j] = nums[j - 1];
      j--;
    }
    nums[j] = tmp;
  }
  return nums;
};
```

上面的是移动法的插入排序，即将元素一个一个移动到后一个位置。
既然是将当前选择的元素移动到前面，那么直接和前一个交换也是可以的。

```js
const insertionSort = (nums) => {
  for (let i = 1; i < nums.length; i++) {
    let j = i;
    while (nums[j] < nums[j - 1] && j > 0) {
      swap(nums, j, j - 1);
      j--;
    }
  }
  return nums;
};
```

- 最好时间复杂度：它对应的数组本身就有序这种情况。此时内层循环只走一次，整体复杂度取决于外层循环，时间复杂度就是一层循环对应的 O(n)。
- 最坏时间复杂度：它对应的是数组完全逆序这种情况。此时内层循环每次都要移动有序序列里的所有元素，因此时间复杂度对应的就是两层循环的 O(n^2)
- 平均时间复杂度：O(n^2)

## 希尔排序

希尔排序是插入排序的改进。简单插入排序很循规蹈矩，不管数组分布是怎么样的，依然一步一步的对元素进行比较，移动，插入。

希尔排序采用的是“分组”的形式，每次选择一个增量 gap，根据 gap 将数组分组。
gap 初始化为 length/2，即每个数和它相隔 gap 的那个数分为一组。初始显然是两个数一组，并且一共有 length/2 组。分组之后，在组内进行插入排序。
而后每次将 gap/2，即 gap=gap/2；这时 gap 缩小了一倍，同样在组内进行插入排序
最后直到 gap=1，即整个数组为 1 组，再进行一次插入排序。虽然这次是对整个数组的插入排序，但是由于数组已经基本有序，所以基本上只需要微调几个位置即可，时间复杂度会很低。

![](https://pic1.imgdb.cn/item/635e111c16f2c2beb1389b20.jpg)

代码如下：

```js
function shellSort(nums) {
  for (
    let gap = Math.floor(nums.length / 2);
    gap > 0;
    gap = Math.floor(gap / 2)
  ) {
    // 里边就是插入排序
    // 普通插入排序是从1开始的，希尔排序就是从每次的gap（即第一个分组的第二个数的位置）开始，然后i++，表示下一个分组的第二个gap，依次类推
    for (let i = gap; i < nums.length; i++) {
      let j = i;
      // 普通插入排序是j和j-1比较，这里就是j和j-gap比较，即j和组内的前一个元素比较
      while (j >= gap && nums[j] < nums[j - gap]) {
        swap(nums, j, j - gap);
        j -= gap;
      }
    }
  }
  return nums;
}
```

希尔排序是第一批冲破 O(n^2)的排序算法之一，它的平均时间复杂度是 O(nlogn)，空间复杂度和插入排序一样都是 O(1)。

## 归并排序

将数组拆分成两个部分，然后分别单独排序，最后合并。因此拆分过程实际上是类似二叉树的递归遍历，递归边界是数组只有一个元素的时候，直接返回该元素。

```js
// 合并两个有序数组
function mergeArr(arr1, arr2) {
  let i = 0;
  let j = 0;
  const newArr = [];
  while (i < arr1.length && j < arr2.length) {
    arr1[i] < arr2[j] ? newArr.push(arr1[i++]) : newArr.push(arr2[j++]);
  }
  return [...newArr, i === arr1.length - 1 ? arr2.slice(j) : arr1.slice(i)];
}

const mergeSort = (nums) => {
  if (nums.length <= 1) return nums;
  const mid = Math.floor(nums.length / 2);
  const left = mergeSort(nums.slice(0, mid));
  const right = mergeSort(nums.slice(mid));
  return mergeArr(left, right);
};
```

归并排序的时间复杂度是 `O(nlog(n))`；由于每次合并都是创建一个新数组，所以空间复杂度也是`O(nlogn)`。

另外还有一种归并是迭代归并，可以保证 O(1)的空间复杂度，详情参考上面的链表排序部分

## 快排

快速排序会将原始的数组筛选成较小和较大的两个子数组，然后递归地排序两个子数组。

```js
const quickSort = (nums, left = 0, right = nums.length - 1) => {
  if (nums.length <= 1) return nums;
  // ！！！！！！！！！！！！！！
  // 注意这里不能只求一个序号！！！！！！！！！
  // 必须是得到具体的中间值，因为后面数组会变化，中间值如果是个序号会变，排序会出错！！！！！
  // 即必须是nums[Math.floor(left + (right - left) / 2)]
  // 不能只是Math.floor(left + (right - left) / 2)
  let divider = nums[Math.floor(left + (right - left) / 2)];
  let i = left;
  let j = right;
  while (i <= j) {
    while (nums[i] < divider) i++;
    while (nums[j] > divider) j--;
    if (i <= j) {
      swap(nums, i, j);
      i++;
      j--;
    }
  }

  if (left < i - 1) quickSort(nums, left, i - 1);
  if (i < right) quickSort(nums, i, right);
  return nums;
};
```

拆分一下：

```js
function quickSort(arr, left = 0, right = arr.length - 1) {
  if (arr.length > 1) {
    const lineIndex = partition(arr, left, right);
    if (left < lineIndex - 1) quickSort(arr, left, lineIndex - 1);
    if (lineIndex < right) quickSort(arr, lineIndex, right);
  }
  return arr;
}
function partition(arr, left, right) {
  let pivotValue = arr[Math.floor(left + (right - left) / 2)];
  let i = left;
  let j = right;
  while (i <= j) {
    while (arr[i] < pivotValue) i++;
    while (arr[j] > pivotValue) j--;
    if (i <= j) {
      swap(arr, i, j);
      i++;
      j--;
    }
  }
  return i;
}

function swap(arr, i, j) {
  [arr[i], arr[j]] = [arr[j], arr[i]];
}
```

partition 函数还有一种写法，个人感觉第二种写法更合适一点，对于下面的快速选择算法也能更好兼容：

这里 partition 函数先选择了最左边的数为基准（而非中间数），然后排序这个数右边的所有元素。最后再把这个元素放进顺序正确的位置即可。

```js
/**
 * @param {number[]} nums
 * @return {number[]}
 */
var sortArray = function (nums) {
  function quickSort(arr, left = 0, right = arr.length - 1) {
    if (left < right) {
      const p = partition(arr, left, right);
      quickSort(arr, left, p - 1);
      quickSort(arr, p + 1, right);
    }
  }
  function partition(arr, left, right) {
    let pivotValue = arr[left];
    let i = left + 1;
    let j = right;
    while (i <= j) {
      while (arr[i] <= pivotValue && i < right) i++;
      while (arr[j] > pivotValue && j > left) j--;
      // 这里为啥要判断一次i和j，直接在循环判断不行吗？
      // 原因是，当i=j，且上面两个循环不能进入时，i和j一直不能自增，如果不加这个判断就无法退出
      if (i >= j) {
        break;
      }
      swap(arr, i, j);
    }
    swap(arr, left, j);
    // 这里返回j是因为j会停在排好序的位置上，而i则会走过一位
    return j;
  }

  function swap(arr, i, j) {
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
  quickSort(nums);
  return nums;
};
```

---

假设数组元素个数为 N，那么二叉树每一层的元素个数之和就是 O(N)；分界点分布均匀的理想情况下，树的层数为 O(logN)，所以理想的总时间复杂度为 O(NlogN)。
![](https://pic.imgdb.cn/item/62dbac38f54cd3f937aa09b4.jpg)

如果数组极端情况（已经正序或倒序排序），也就是说每次选择的这个分区点无法将数组一分为二，每次得到的 p 都是 left+1，那么构成的二叉树就会很极端的单向延伸，即一共要走 N 层，并且每层都需要对 N、N-1、N-2...个数排序，即

```
N + (N - 1) + (N - 2) + ... + 1 = O(N^2)
```

![](https://pic.imgdb.cn/item/62dbac43f54cd3f937aa5108.jpg)

为了防止这种极端情况，应该考虑排序之前用洗牌算法把数组随机化一下，或者在 partition 开始选择 pivotValue 时选择随机的一个数。

因为 partition 函数内每次选择的是第一个元素，那就可以在每次排序之前，先从数组中随机选一个数跟第一个交换即可。

```js
function partition(arr, left, right) {
  // 随机一个位置和第一个元素交换
  const rIndex = Math.floor(Math.random() * arr.length);
  swap(arr, left, rIndex);

  let pivotValue = arr[left];
  let i = left + 1;
  let j = right;
  while (i <= j) {
    while (arr[i] <= pivotValue && i < right) i++;
    while (arr[j] > pivotValue && j > left) j--;
    if (i >= j) {
      break;
    }
    swap(arr, i, j);
  }
  swap(arr, left, j);
  // 这里返回j是因为j会停在排好序的位置上，而i则会走过一位
  return j;
}
```

## 计数排序

计数排序是利用数组进行排序的方式，它通过将原数组的每一项的值作为新数组的 index，新数组的 index 对应的值表示该数的出现次数，最后遍历一遍新数组；由于遍历数组是按照从小到大的顺序来的，因此就相当于对原数组进行了排序。

```js
function countingSort(nums) {
  const bucket = new Array(Math.max(...nums) + 1).fill(0);
  const res = [];
  for (const num of nums) {
    bucket[num]++;
  }
  for (let i = 0; i < bucket.length; i++) {
    while (bucket[i] > 0) {
      res.push(i);
      bucket[i]--;
    }
  }
  return res;
}
```

这种情况会造成极大的空间复杂度，比如一组数据[100,102,104,101,99]，显然没必要创建前 100 个空间。所以还需要找到数组的最小值，以最小值为基准处理数据：

同时这种方法也解决了负数的问题。比如[-5,-4,1,2,3]，以最小值-5 为基准，变成[0,1,6,7,8]的排序，最后结果再加-5 即可。

```js
function countingSort(nums) {
  const max = Math.max(...nums);
  const min = Math.min(...nums);
  const bucket = new Array(max - min + 1).fill(0);
  const res = [];
  for (const num of nums) {
    bucket[num - min]++;
  }
  for (let i = 0; i < bucket.length; i++) {
    while (bucket[i] > 0) {
      res.push(i + min);
      bucket[i]--;
    }
  }
  return res;
}
```

计数排序限制很多，只能处理整数的排序，并且当数据跨度过大时将会占用很大空间。但它是一个 O(n)时间复杂度的排序方法，比前面的归并、快速都要更快。

## 桶排序

桶排序是计数排序的升级版，它的基本原理是根据某个映射关系，将数字分到不同的桶中。
桶之间有一定顺序，比如第一个桶是 0-9，第二个桶是 10-19，依次类推；
然后对桶内元素排序，这个排序可以选择上面的各种方法，通常选择插入排序；
最后将桶内元素按顺序输出即可。

其中，桶的数据之间的关系如下：
桶的数量 = (最大值 - 最小值) / 一个桶的大小
每个数字应该放入的桶的序号 = (该数字 - 最小值) / 一个桶的大小

代码如下：

```js
function bucketsSort(nums, bucketSize = 5) {
  const max = Math.max(...nums);
  const min = Math.min(...nums);
  const res = [];
  // 公式1
  const buckets = Array.from(
    new Array(Math.floor((max - min) / bucketSize)),
    () => []
  );
  for (const num of nums) {
    // 公式2
    buckets[Math.floor((num - min) / bucketSize)].push(num);
  }
  for (const bucket of buckets) {
    insertionSort(bucket);
    for (const num of bucket) {
      res.push(num);
    }
  }
  return res;
}
```

## 基数排序

基数排序和上面两个排序一样，都用了桶的概念。
基数排序的“桶”是每个数字的某一个位上的数，即先按照个位数排序，再按照十位数排序，依次类推直到最高位。
那么就需要创建 0-9 这十个桶，分别用于收集某一位的值为 0-9 的数字。

- 第一次排序遍历每个数的个位数，按照个位数上的数字分别放入桶。
- 按照队列的方式（shift）将桶内元素弹出，依次排成新的数组
- 对新的数组的十位上操作，再次重复第一步和第二步，最后直到最高位

![](https://www.runoob.com/wp-content/uploads/2019/03/radixSort.gif)

```js
unction radixSort(nums){
  let tmpNums = [...nums]
  const min = Math.min(...nums)
  if(min < 0){ // 处理负数
    tmpNums = nums.map((num)=>num-min)
  }
  const buckets = Array.from(new Array(10),()=>[])
  const maxRadix = Math.max(...nums).toString().length - 1

  for(let radix = 0;radix <= maxRadix;radix++){
    for(const num of tmpNums){
      const radixNum = Math.floor(num / 10 ** radix) % 10
      buckets[radixNum].push(num)
    }
    tmpNums.length = 0 // 临时数组要清空
    for(const bucket of buckets){
      tmpNums.push(...bucket)
      bucket.length = 0 // 注意每次桶要清空
    }

  }
  if(min < 0){
    tmpNums = tmpNums.map((num)=>num+min)
  }
  return tmpNums
}
```

## TOP K 问题

> 给你一个数组，求这个数组中的第 K 个最大元素。
> 需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。

这道题的思路有好几种：

1. 利用大根堆，只要把堆的数据结构建立好，依次取 K 次就可以。但是问题在于 js 很难用大根堆

实际上就是一个数组，详细解释可以参考 https://leetcode.cn/problems/kth-largest-element-in-an-array/solution/xie-gei-qian-duan-tong-xue-de-ti-jie-yi-kt5p2/

堆的时间复杂度为 O(nlogk)，其中 k 是取的次数。空间复杂度是 O(k)，因为堆里最多就是 k 个数

```js
const swap = (arr, i, j) => ([arr[i], arr[j]] = [arr[j], arr[i]]);
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number}
 */
var findKthLargest = function (nums, k) {
  let size = nums.length;
  function buildMaxHeap(nums, size) {
    // 每次从第一个非叶子节点开始，自下而上递归堆化
    for (let i = Math.floor(size / 2) - 1; i >= 0; i--) {
      heapify(nums, i, size);
    }
  }
  function heapify(nums, i, size) {
    let left = 2 * i + 1;
    let right = 2 * i + 2;
    let mid = i;
    if (left < size && nums[left] > nums[mid]) {
      mid = left;
    }
    // 如果左右都大于当前，那就选左右两个更大的一个
    if (right < size && nums[right] > nums[mid] && nums[right] > nums[left]) {
      mid = right;
    }
    if (mid !== i) {
      // 如果发生了交换
      swap(nums, mid, i);
      heapify(nums, mid, size);
    }
  }
  buildMaxHeap(nums, size);
  for (let i = nums.length - 1; i >= nums.length - k + 1; i--) {
    swap(nums, 0, i); // 每次把堆顶元素和堆底元素交换
    size--;
    heapify(nums, 0, size);
  }
  // 到这里就是完整的堆排序

  return nums[0];
};
```

2. 快速选择算法，即“快排的二分查找版本”。利用快排的 partition 函数，这个函数会确定一个元素的确定位置，并返回这个元素的正确位置。那么我们就可以这样：
1. 先通过 partition 计算出一个位置 p，然后判断 p 和要找的 TopK 的 k 的大小关系。比如现在有 10 个元素，计算的 p=5，说明从小到大第 5 个元素已经排好位置；如果选择第二大的元素，那么 k=10-2=8，即从小到大第 8 个元素。这时`p<k`，说明应该在 p 的右边继续查找
1. 设置新的边界，类似二分查找一样，即`left=p+1`，或`right=p-1`，直到`p=n-k`为止。这个过程可以是递归，也可以是二分查找那样的循环

快速选择算法的平均时间复杂度为 O(n)，最坏情况是 O(n^2)

代码如下：

> 注意这里的 partition 应该选择第二种，第一种会在边界出现奇奇怪怪的 bug，不建议选

```js
var findKthLargest = function (nums, k) {
  const swap = (arr, i, j) => {
    [arr[i], arr[j]] = [arr[j], arr[i]];
  };
  function partition(arr, left, right) {
    let pivotValue = arr[left];
    let i = left + 1;
    let j = right;
    while (i <= j) {
      while (arr[i] <= pivotValue && i < right) i++;
      while (arr[j] > pivotValue && j > left) j--;
      if (i >= j) {
        break;
      }
      swap(arr, i, j);
    }
    swap(arr, left, j);
    return j;
  }
  let left = 0;
  let right = nums.length - 1;
  k = nums.length - k;
  while (left <= right) {
    const p = partition(nums, left, right);
    if (p === k) {
      return nums[p];
    } else if (p < k) {
      left = p + 1;
    } else {
      right = p - 1;
    }
  }
};
```

# 动态规划

> 动态规划（dynamic programming，DP）是一种将复杂问题分解成更小的子问题来解决的优化技术。
> 用动态规划解决问题时，要遵循三个重要步骤：
>
> 1. 定义子问题；
> 2. 实现要反复执行来解决子问题的部分；
> 3. 识别并求解出基线条件。

动态规划的题目有几个关键的特征：

1. 要求你给出达成某个目的的解法个数
2. 不要求你给出每一种解法对应的具体路径
3. 题目要求中有“最值”，即最优解；
4. 一个解的得出依赖于前一个解，依次依赖迭代

这样的问题，往往可以用动态规划进行求解

动态规划的题目可能很难理解，如果有实在理解不了的可以先记下来

## 爬楼梯问题

> 假设你正在爬楼梯。需要 n 阶你才能到达楼顶。每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？

倒着思考问题,如果要到达 n 阶的方法为`f(n)`种,从第 n 阶退一层的方法就是`f(n-1)`或`f(n-2)`种,即`f(n) = f(n-1) + f(n-2)` ,然后依次类推,直到`f(1)`和`f(2)`返回

```js
const climbStairs = (n) => {
  if (n === 1) return 1;
  if (n === 2) return 2;

  // 树形思考问题,这里相当于两个子节点
  return climbStairs(n - 1) + climbStairs(n - 2);
};
```

上面这个解法会有很多重复的,比如`f(n-2)`的下一层会包含`f(n-3)`,这个值可能被多次计算
可以考虑 Map 缓存计算过的值

```js
const climbStairsCached = (n) => {
  const map = new Map();
  return climbStairs(n);
  function climbStairs(n) {
    if (map.has(n)) return map.get(n);
    if (n === 1) return 1;
    if (n === 2) return 2;
    const res = climbStairs(n - 1) + climbStairs(n - 2);
    map.set(n, res);
    return res;
  }
};
```

以上这种在递归的过程中，不断保存已经计算出的结果，从而避免重复计算的手法，叫做记忆化搜索。

真正的动态规划，是一个自底向上的过程。它要求我们站在已知的角度，通过定位已知和未知之间的关系，一步一步向前推导，进而求解出未知的值。

由上面的关系，可以推出状态转移方程为：

```
f(n) = f(n-1) + f(n-2)
且
f(1) = 1
f(2) = 2
```

把这个方程放到循环中，就可以解出来了：

```js
const climbStairsDynamic = (n) => {
  const f = [];
  f[1] = 1;
  f[2] = 2;
  for (let i = 3; i < n; i++) {
    f[i] = f[i - 1] + f[i - 2];
  }
  return f[n];
};
```

因此动态规划的关键就是推出状态转移方程，只要方程能得出，丢到迭代里边就可以计算了。

## 跳跃问题

### 跳跃游戏 I

> 给定一个非负整数数组 nums ，你最初位于数组的 第一个下标 。
> 数组中的每个元素代表你在该位置可以跳跃的最大长度。
> 判断你是否能够到达最后一个下标。

这道题目本质上应该算是贪心解法，但是这里其实用动态规划也能解。
思路是这样：设`dp[i]`表示表示当前位置`i`能跳到的最远位置，dp[i]的变化有几种情况：

0. 首先计算当前位置的序号+nums[i]值`maxLen`，表示从当前位置起跳，能到达的最远位置。但是该位置不一定可达，因此需要一些方法判断是否可达。
1. 如果`maxLen`小于 dp[i-1]，说明当前位置能跳到的最远距离包括在前面的范围之内，不用更新。同理如果`maxLen`大于 dp[i-1]就应该更新
2. 如果当前位置的序号 i 比 dp[i-1]大，说明该位置无法到达，直接返回 false
3. 如果遍历完都没返回，就说明可达，返回 true

```js
/**
 * @param {number[]} nums
 * @return {boolean}
 */
var canJump = function (nums) {
  const dp = new Array(nums.length).fill(0); // dp[i]表示当前位置能跳到的最远位置
  dp[0] = nums[0];
  for (let i = 1; i < nums.length; i++) {
    const maxLen = i + nums[i];
    if (i > dp[i - 1]) {
      return false;
    }
    dp[i] = Math.max(dp[i - 1], maxLen);
  }
  console.log(dp);
  return true;
};
```

### 跳跃游戏 II

在上一题的基础上，假设一定可以达到最后位置，求最少跳跃次数。
我们可以利用上一题的结果。上一题其实告知了我们数组中的哪些元素可以到达某个位置的元素，即从开始向这个元素的位置遍历，当 dp[i] >= i 时，其后的所有位置都可以经过一次跳跃到达该元素。因此在这些元素中遍历找出一个之前跳跃次数最小的，加 1 即可。
设 dp2[i]表示能到达 i 位置的最小跳跃次数，dp2[i]应该等于前面所有能一次跳到该位置的元素的 dp2 的最小值+1

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
var jump = function (nums) {
  const dp = new Array(nums.length).fill(0);
  dp[0] = nums[0];
  for (let i = 1; i < nums.length; i++) {
    const maxLen = i + nums[i];
    dp[i] = Math.max(dp[i - 1], maxLen);
  }
  let start = 0;
  dp.forEach((num, i) => {
    if (num >= nums.length) start = i;
  });

  /* line */

  const dp2 = new Array(nums.length).fill(0); // dp2[i]表示到达i的最小跳跃次数
  dp2[0] = 0;
  for (let i = 1; i < nums.length; i++) {
    let mindp2 = Infinity;
    for (let j = 0; j < i; j++) {
      if (dp[j] < i) continue; // j位置的元素到达不了当前元素，不考虑
      // 注意上面是dp，即判断是否能到达；下面是真正迭代的dp2
      mindp2 = Math.min(mindp2, dp2[j]); // 找到dp2的最小值，即跳跃次数的最小值
    }
    dp2[i] = mindp2 + 1; // 最少跳跃次数+1
  }
  console.log(dp2);
  return dp2[dp2.length - 1];
};
```

---

还有一种贪心的思路，即每次跳跃的是当前能跳跃的范围内的那部分中，能到达最远的距离。
解释一下就是这样：

```
[2,3,1,1,4]

1. 从第一个数2开始，能到达的范围是后两个位置3和1。
2. 计算每个位置的nums[j]+j，得到最大值。比如这里分别得到3+1=4 1+2=3，并且4>3>0+2（即比当前已经覆盖的范围远），选择最大值4，也就是第二个位置作为下一个起跳点。
3. 再从3跳，这时发现3 + 1 = 4 >= arr.length-1，所以直接返回跳跃次数+1就可以到了

第二步中最重要的是计算跳跃最大值，不只是nums[j]的值，还应该算上本来的位置。可能会有特殊情况即：
范围内的每个位置的nums[j]+j都小于本来就已经覆盖的范围，那就直接跳到最后一个位置。比如[5,1,2,3,4]这个数组，显然第一步能覆盖的范围比后面的都远，那就直接跳到最后一个就可以了
其他情况就是计算出最远能覆盖的范围的那个位置，再跳到那个位置即可
```

代码如下：

```js
var jump = function (nums) {
  if (nums.length === 1) return 0;
  let jumps = 0;
  for (let i = 0; i < nums.length; ) {
    // i就表示当前的位置
    const end = i + nums[i];
    if (end >= nums.length - 1) return jumps + 1;
    let maxIndex = 0;
    let maxEnd = 0;
    for (let j = i + 1; j <= end; j++) {
      if (nums[j] + j >= maxEnd) {
        maxEnd = nums[j] + j;
        maxIndex = j;
      }
    }
    if (maxEnd > end) i = maxIndex; // 如果更新最远距离，就更新当前位置
    else i = end; // 没有比一开始更长的，就从最后一个位置跳
    jumps++;
  }
  return jumps;
};
```


## 整数拆分

> 给定一个正整数 n，将其拆分为至少两个正整数的和，并使这些整数的乘积最大化。 返回你可以获得的最大乘积。
> 示例 :
> 输入: 10
> 输出: 36
> 解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36。
> 说明: 你可以假设 n 不小于 2 且不大于 58

思路：对于一个数 i，可能的拆分方式有两种：

1. 从 j 到 i 遍历，即`j * (i - j)`
2. 因为`i - j`可能也能拆分（比如 6 可以再拆成 3*3），因此可能是`i - j`拆分的`最大乘积 * j`

状态转移方程：
设`dp[i]`表示数字 i 拆分得到的最大乘积

```
dp[i] = Math.max(dp[i - j] * j, j * (i - j))
```

实际实现应该取 j 从 1 到 i 遍历得到的`dp[i]`最大值

```js
/**
 * @param {number} n
 * @return {number}
 */
var integerBreak = function (n) {
  const dp = [];
  dp[0] = 1;
  dp[1] = 1;
  dp[2] = 1;
  // dp[i]有两种情况,一种是拆分成两个数的,最大值为max(...[j * (i - j)]),即从1到i遍历j,计算拆分成两个数的最大值
  // 另一种情况是可能拆成多个数,即j * dp[i - j],即拆分i-j.
  // 这里不能拆分j,即不能dp[j] * dp[i - j],因为这样最少都是四个数相乘
  // dp[i] = max(dp[i], max((i - j) * j, dp[i - j] * j));
  for (let i = 3; i <= n; i++) {
    dp[i] = 0;
    for (let j = 1; j < i - 1; j++) {
      dp[i] = Math.max(dp[i], (i - j) * j, dp[i - j] * j);
    }
  }
  return dp[n];
};
```

## 不同的二叉搜索树

https://leetcode.cn/problems/unique-binary-search-trees/

> 给你一个整数 n ，求恰由 n 个节点组成且节点值从 1 到 n 互不相同的 二叉搜索树 有多少种？返回满足题意的二叉搜索树的种数。

这道题看着像是二叉树问题，但实际上是一道动态规划。写一个式子就懂了
设 dp[i]表示 n=i 时的二叉树个数，那么 dp[1]和 dp[2]应该很容易得出。
接下来是 dp[3]，dp[3]的二叉树可以**根据头节点是哪个数**分为这几种情况：

- 元素 1 为头结点搜索树的数量 = 右子树有 2 个元素的搜索树数量 \* 左子树有 0 个元素的搜索树数量
- 元素 2 为头结点搜索树的数量 = 右子树有 1 个元素的搜索树数量 \* 左子树有 1 个元素的搜索树数量
- 元素 3 为头结点搜索树的数量 = 右子树有 0 个元素的搜索树数量 \* 左子树有 2 个元素的搜索树数量

![](https://img-blog.csdnimg.cn/20210107093129889.png)

所以得到了递推关系：

```js
dp[3] = dp[0] * dp[2] + dp[1] * dp[1] + dp[2] * dp[0];

dp[n] = dp[0]*dp[n-1] + dp[1]*dp[n-2] +...+dp[n-1]*dp[0]

// 那么对于每个n都是
let sum = 0
for(let i = 1;i < n;i++){
  sum += dp[i-1] * dp[n-i]
}
dp[n] = sum

// 还需要得到前面的每个n才能推出现在的n，因此要从dp[2]开始一直算到dp[n]
for(let i = 2;i <= n;i++){
  let sum = 0
  for(let j = 1;j < i;j++){
    sum += dp[j-1] * dp[i-j]
  }
  dp[n] = sum
}
```

完整代码：

```js
/**
 * @param {number} n
 * @return {number}
 */
var numTrees = function (n) {
  const dp = new Array(n).fill(1);
  dp[0] = 1;
  dp[1] = 1;
  for (let i = 2; i <= n; i++) {
    let sum = 0;
    for (let j = 1; j <= i; j++) {
      sum += dp[j - 1] * dp[i - j];
    }
    dp[i] = sum;
  }
  return dp[n];
};
```

---

还有一种思路，我认为是更适合二叉树问题的一种思路，就是用递归解决。并且这个思路也适用于这道题的 II 版。

还是秉持上面的思路，如果有一个五个节点的数组，那么定中间一个节点为根节点，剩下的部分有几种组合？

![](https://pic.imgdb.cn/item/62db865ff54cd3f937b95c7d.jpg)

其实也就是[1,2]和[4,5]这两个组合的数量的积。那么就可以从第一个节点开始，分别选择每个节点作为中间节点，然后递归计算剩下的左右部分的组合数量，相乘即可得到。

设递归函数`dfs(left, right)`表示左子树大小为 left，右子树大小为 right 的树的种数。那么对于整个数组的一部分来说，显然以其中一个节点为根节点的树的数量应该等于`dfs(left, i - 1) * dfs(i + 1, right)`，其中 i 表示从 left 到 right 的一个数，即从 left 到 right 的区间中依次选一个作为根节点，然后全部加起来。

另外，这种思路不会出现重复导致数量增多的情况，因为同样长度的数组，left 和 right 不可能重复；而 left 和 right 重复的时候，数组长度又肯定不会一样。
但是仍然需要记录，否则会导致复杂度过高。显然当数组长度固定时，对应的树的数量应该是确定的。因此我们可以把该数组长度对应的树的数量记录下来，减少计算量。

> 这么来想其实这道题和爬楼梯问题很像，只是爬楼梯只能走一步或两步；而这道题的两个子树数量可以任意组合。

代码如下：

```js
/**
 * @param {number} n
 * @return {number}
 */
var numTrees = function (n) {
  const visited = Array.from(new Array(n), () => new Array(n).fill(null));
  function dfs(left, right) {
    if (left > right) return 1;
    let trees = 0;
    if (visited[left - 1][right - 1]) return visited[left - 1][right - 1];
    for (let i = left; i <= right; i++) {
      trees += dfs(left, i - 1) * dfs(i + 1, right);
    }
    visited[left - 1][right - 1] = trees;
    return trees;
  }
  return dfs(1, n);
};
```

## 不同路径问题

> 一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为 “Start” ）。机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。问总共有多少条不同的路径？
> ![](https://img-blog.csdnimg.cn/20210110174033215.png)

机器人移动经过的是二维数组，因此对于每一个节点来说，必然会有水平和竖直两种方向上的两组路径，即只需要记录左边和上边来的路径数（因为是从左向右遍历，因此不需要统计右和下）
![](https://pic.imgdb.cn/item/6273a6e40947543129d857e0.jpg)

因此设`dp[i][j]`为经过点`(i,j)`的路径最大值，并且该值一定是由其水平和竖直推出的，即`dp[i][j] = dp[i - 1][j] + dp[i][j - 1]`

![](https://img-blog.csdnimg.cn/20201209113631392.png)

代码如下：

```js
var uniquePaths = function (m, n) {
  const dp = Array.from(new Array(n), () => new Array(m));
  for (let i = 0; i < n - 1; i++) {
    dp[i][0] = 1;
  }
  for (let j = 0; j < m - 1; j++) {
    dp[0][j] = 1;
  }
  for (let i = 1; i < n; i++) {
    for (let j = 1; j < m; j++) {
      dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
    }
  }
  return dp[n - 1][m - 1];
};
```

## 轰炸敌人

https://leetcode.cn/problems/bomb-enemy/

这道题可以用暴力法解并且不会超时，即每一个空格都计算能轰炸的人数，最后找出最大值即可。

但是观察可以发现，同一行的某些位置在本行的轰炸人数应该是相同的，同一列也是同理；对于一行来说，一个位置的轰炸数量应该是可以由它前面的一个位置得出的：

- 0，当前面一个是墙时
- dp[i-1] + 1，当前面是敌人时
- dp[i-1]，当前面是空格时

但是一个位置上的值不仅由其所在的行决定，还有它所在的列的值。也就是说如果我们设`dp[i][j]`表示(i,j)位置能轰炸的人数，这个值不能仅从本行或本列得出，也不能给本行或本列的下一个位置使用，更不能从斜向（比如`dp[i-1][j-1]`）得出。
因此，可以分开计算每行和每列的值，然后把值加到 dp 数组中。这样 dp 的每一项不是直接由其他 dp 推导来，而是相当于很多个数组的和；

![](https://pic1.imgdb.cn/item/6346bc5616f2c2beb1ec303f.jpg)

比如上图的第 2 行。我们只看第 2 行，设第 2 行的第 i 个位置上能击杀的敌人数为 dp[i]，则

```js
if (grid[i - 1] === "E") {
  dp[i] = dp[i - 1] + 1;
} else if (grid[i - 1] === "0") {
  dp[i] = dp[i - 1];
} else {
  dp[i] = 0;
}
```

> 注意这里有敌人的位置和有墙的位置也是正常计算的，但是并不影响最终结果；最后只需要跳过这些非空位即可。

但是这还没完，这里其实只是从左向右计算的结果（即只考虑左边对右边的影响），还需要从右向左计算一次。对于每个位置来说，两次计算的和才是这一行的最终结果。

```js
if (grid[i + 1] === "E") {
  dp[i] = dp[i + 1] + 1;
} else if (grid[i + 1] === "0") {
  dp[i] = dp[i + 1];
} else {
  dp[i] = 0;
}
```

那么对于其他行、其他列也是同理。得出这个位置的 dp 值后，把他加到一个最终的结果数组中；当所有的行和列都遍历完之后，得到的结果数组的每一项都是最后的结果。

在题解中，并不需要这个确切的 dp 数组，因为数组的值最后还是会一一赋给结果数组。所以题解设置了一个变量；当向一个方向遍历时，如果遇到了敌人就++，如果遇到了墙就清零，如果遇到空位就不操作。

```js
let cnt = 0;
for (let j = 0; j < m; j++) {
  if (grid[i][j] === "E") cnt++;
  else if (grid[i][j] === "W") cnt = 0;
  dp[i][j] += cnt;
}
//再从右向左计算一次
// 本行计算完后再计算下一行

// 行计算完之后再计算列
```

代码：

```js
var maxKilledEnemies = function (grid) {
  const n = grid.length;
  const m = grid[0].length;
  const dp = Array.from(new Array(n), () => new Array(m).fill(0));
  for (let i = 0; i < n; i++) {
    let cnt = 0; // cnt是根据前一项得出的，相当于一个只保留当前位置值的dp数组
    for (let j = 0; j < m; j++) {
      // 本行从左往右
      if (grid[i][j] === "E") cnt++;
      else if (grid[i][j] === "W") cnt = 0;
      dp[i][j] += cnt; // 每个位置都加最新的cnt
    }
    cnt = 0;
    for (let j = m - 1; j >= 0; j--) {
      // 本行从右往左
      if (grid[i][j] === "E") cnt++;
      else if (grid[i][j] === "W") cnt = 0;
      dp[i][j] += cnt;
    }
  }
  for (let j = 0; j < m; j++) {
    // 列
    let cnt = 0;
    for (let i = 0; i < n; i++) {
      // 本列从上到下
      if (grid[i][j] === "E") cnt++;
      else if (grid[i][j] === "W") cnt = 0;
      dp[i][j] += cnt;
    }
    cnt = 0;
    for (let i = n - 1; i >= 0; i--) {
      // 本列从下到上
      if (grid[i][j] === "E") cnt++;
      else if (grid[i][j] === "W") cnt = 0;
      dp[i][j] += cnt;
    }
  }
  console.log(dp);
  let max = 0;
  for (let i = 0; i < dp.length; i++) {
    // 只统计空位的结果
    for (let j = 0; j < dp[0].length; j++) {
      if (grid[i][j] === "0") {
        max = Math.max(max, dp[i][j]);
      }
    }
  }
  return max;
};
```

## 下降路径最小和

https://leetcode.cn/problems/minimum-falling-path-sum/submissions/

> 给你一个 n x n 的 方形 整数数组  matrix ，请你找出并返回通过 matrix 的下降路径 的 最小和 。
> 下降路径 可以从第一行中的任何元素开始，并从每一行中选择一个元素。在下一行选择的元素和当前行所选元素最多相隔一列（即位于正下方或者沿对角线向左或者向右的第一个元素）。具体来说，位置 (row, col) 的下一个元素应当是 (row + 1, col - 1)、(row + 1, col) 或者 (row + 1, col + 1) 。

设 dp[i][j]表示位于[i][j]的元素最小下降路径和，显然等于上面一层相邻的三个元素之间的最小值的递推。

```js
/**
 * @param {number[][]} matrix
 * @return {number}
 */
var minFallingPathSum = function (matrix) {
  const dp = Array.from(
    new Array(matrix.length),
    () => new Array(matrix[0].length)
  );
  dp[0] = [...matrix[0]];
  for (let i = 1; i < matrix.length; i++) {
    for (let j = 0; j < matrix[i].length; j++) {
      dp[i][j] =
        matrix[i][j] +
        Math.min(
          dp[i - 1][j - 1] || Infinity,
          dp[i - 1][j] || Infinity,
          dp[i - 1][j + 1] || Infinity
        );
    }
  }
  return Math.min(...dp[dp.length - 1]);
};
```

## 杨辉三角

> 给定一个非负整数 numRows，生成「杨辉三角」的前 numRows 行。
> 在「杨辉三角」中，每个数是它左上方和右上方的数的和。

![](https://pic.leetcode-cn.com/1626927345-DZmfxB-PascalTriangleAnimated2.gif)

这道题不是标准的动态规划题，但是可以用动态规划解。
思路是通过二维数组，设`dp[i][j]`为第 i 行、第 j 个数字的值，那么显然

```
dp[i][j] = dp[i-1][j] + dp[i - 1][j-1]
```

然后每行计算的时候，都初始化行首和行尾的元素值为 1

代码如下：

```js
/**
 * @param {number} numRows
 * @return {number[][]}
 */
var generate = function (numRows) {
  const dp = [[1]];
  const res = [...dp];
  for (let i = 1; i < numRows; i++) {
    dp[i] = [];
    dp[i][0] = 1;
    dp[i][i] = 1;
    for (let j = 1; j < i; j++) {
      dp[i][j] = dp[i - 1][j] + dp[i - 1][j - 1];
    }
    res.push(dp[i]);
  }
  return res;
};
```

## 背包类问题

### 背包问题基本思路

背包类问题的特点是暴力解法一般都是常见的回溯，但是回溯的复杂度又太高，也不好剪枝优化，用动态规划能代替这种回溯。
并不是所有的回溯问题都可以用背包的思路来解。通常背包问题的特点是：

1. 一个数组或一些序列，要从中**求和**或者找组合。
2. 对于数组中的元素，有“选”或者“不选”两种情况，即符合回溯问题的思路；

如果不是这类问题，通常就要进行一些转化。比如分目标和和最后一块石头这两道题，看起来都不太能看出来是符合这类问题的，需要对问题转化一下，变成“几个数求和得到某个数”这样的问题就可以用背包解决。

01 背包问题解决的通用思路：

1. 每行设为数组中的每个元素，即从`[0,i]`中任意选；一般初始化的时候会把第一行设置好，所以实际上是从前两个数开始的。（有时候也可以定义成前 i 个，则对应的值应该是`arr[i-1]`，这时 i=1 表示的就是第一个数而不是[0,1]这样第一和第二个数）
2. 每列设为从 0 到目标值的连续自然数。比如想计算凑出值 m，就从 0 开始到 m 每一个连续的整数都是一列。
3. 计算时每个 dp 大多数情况都是`dp[i-1][j]`和`dp[i-1][j-arr[i]]`推出来的，可能会有：

- 计算值，比如确保不大于 j 的值或者最大值，就取 j>=nums[i]时，`dp[i][j] = Math.max(dp[i-1][j],dp[i-1][j-nums[i]])`，表示选或不选当前数 nums[i]两种情况；如果`j<nums[i]`，一般就保持值不变
- 计算组合次数或多少种，一般是`dp[i][j] = dp[i-1][j]+dp[i-1][j-nums[i]]`，因为求组合的一般是两种情况都可以，取一个和
- 计算能不能确切满足的，比如刚好等于 j，一般是`dp[i][j] = dp[i-1][j] || dp[i-1][j-nums[i]]`，选或不选只要有一个能成立就行。或者还是用第一种计算最值，如果最值等于当前值，就是可以刚好相等。

4. 最后的结果基本上都是最右下角的。

因此，背包类问题的最难的地方在于怎么把问题转化为类似背包的“选或不选”

关于初始化数组大小和范围问题：

- 列的大小一般是目标数+1，比如要求和为 5，那么就应该是`new Array(5+1)`
- 行的大小一般是遍历的总元素个数 len+1
- j 一般表示的是从 0 到 j，所以一般初始化就定义好第一列，遍历从第二列(j = 1)开始。特殊情况不能确定第一列的也可以从第一列开始。
- i 一般表示从 0 到 i 的数，i=0 一般表示的是第一个数；
  - 如果能确定第一个行的情况就初始化，遍历过程中的元素为 nums[i]，相当于跳过了第一个数。
  - 如果不能确定第一行就不初始化，遍历过程中的元素为 nums[i-1]，相当于考虑第一个数。

### 0-1 背包问题

> 有 n 件物品，物品体积用一个名为 w 的数组存起来，物品的价值用一个名为 value 的数组存起来；
> 每件物品的体积用 w[i] 来表示，每件物品的价值用 value[i] 来表示。现在有一个容量为 c 的背包
> 问你如何选取物品放入背包，才能使得背包内的物品总价值最大？

首先先考虑一个问题，假如不用动态规划，这道题怎么解？
其实背包问题本质上就是一个回溯问题，给定了一个空间，我们可以遍历任意物品的组合（单个物品不能重复），如果超重了就剔除最后一个记录并返回。
背包问题动态规划的目的其实也是在实现这个回溯。尤其是数组中的 i，表示从`[0,i]`中任选几个物品，这个看起来就和一般动态规划不太一样，根本原因就是它在模拟一种“组合”。

设`dp[i][j]` 表示从下标为`[0-i]`的物品里任意取，放进容量为 j 的背包，价值**总和最大**是多少。(记住 i 和 j 的含义)

我们假设每个物品重量和价值关系如下:

|        | weight | value |
| ------ | ------ | ----- |
| 物品 0 | 1      | 15    |
| 物品 1 | 3      | 20    |
| 物品 2 | 4      | 30    |

那么得到的 dp 数组就是这样的：
![](https://img-blog.csdnimg.cn/20210110103003361.png)

然后需要确定递推公式。我们可以这样理解，对于每一个即将要放入背包的物品 i ，可能有两种情况：

1. 当前背包容量不够（没有空余或者空余小于当前物品重量），放不下（背包不是一开始就是最大的，而是从 0 开始一个一个遍历上来的）
2. 当前背包容量够，并且空余大于等于当前物品重量

这两种情况恰恰是对应两个公式：

1. 不够放下该物品 i，所以直接无视掉，依旧取之前的值。这个“之前的值”，就是**背包容量不变，并且不放物品 i**的值，体现为`dp[i - 1][j]`。`dp[i-1]`表示的含义是，当前物品没有放入，那么就还是上一个或上面几个物品放入的情况。
2. 能放下物品 i，这时候就选择放入物品 i，并且加上物品 i 的价值。但是注意这里并不是直接在`dp[i - 1][j]`的基础上加，而是取`dp[i - 1][j - weight[i]]`；原因是这样：

比如计算`dp[1][3]`，如下图
![](https://img-blog.csdnimg.cn/20210110103244701.png)

这时`dp[i - 1][j] = dp[0][3]`，即不放入物品 1，那么背包中仍旧只有一个物品 0，价值为 15
如果放入物品 1，我们就需要知道一个特殊的情况，即**空余空间足够，但并未放入**的情况。即`dp[i - 1][j - weight[i]] = dp[0][0]`：这时什么都没有放入，背包空余大小为 3，足够放入重量为 3 的物品 1。最后加上物品 1 的价值 20，得到最终值

如果不找到有足够空间的情况，那么本次的`j`之前可能会有其他情况，比如一个比物品 i 重量小的已经放入，但是没有填满，并且去掉这个小物品是可以放入该物品 i 的。应当避免的就是这种情况。

结合上面两种情况，其实就是在比较“选不选物品 i”；选和不选之间取出一个最大值，成为当前的结果。

状态转移方程为：

```
dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i]);
```

核心遍历代码如下：

```js
// weight数组的大小 就是物品个数
for (let i = 1; i < weight.length; i++) {
  // 遍历物品
  for (let j = 0; j <= size; j++) {
    // 遍历背包容量
    if (j < weight[i])
      dp[i][j] = dp[i - 1][j]; // 如果当前容量都小于物品i的重量，那肯定不用放入
    else dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i]);
  }
}
```

完整代码如下：

```js
function testweightbagproblem(wight, value, size) {
  const len = wight.length;
  const dp = array.from(new Array(len + 1), () => new Array(size).fill(0));

  for (let i = 1; i <= len; i++) {
    for (let j = 0; j <= size; j++) {
      if (j < weight[i])
        // 如果当前容量都小于物品i的重量，那肯定不用放入
        dp[i][j] = dp[i - 1][j];
      else dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i]);
    }
  }

  return dp[len][size];
}
```

> 背包问题是一类问题的代替解法，即部分求组合、子集的**回溯**问题。实际上背包问题的暴力解法就是回溯求所有组合，满足大小要求的计算价值取最大即可。
> 典型案例可以参考下一道题

#### 滚动数组

在状态转移过程中，每一行的 dp 状态值都只与其正上方和左方的状态值有关，因此可对状态空间 dp 进一步优化

实际上，滚动数组是将原先的二维数组拆成了一层一层的数组，实际的遍历量没有改变。每次都是对这个数组进行更新；

**dp 定义**：设 dp[j]表示：容量为 j 的背包，所背的物品价值可以最大为 dp[j]

![](https://img-blog.csdnimg.cn/20210110103614769.png)

**原理**：比如上面这个图中的红框，当物品 2 遍历的时候（还没更新），从后向前遍历的红框位置的 dp[j]还是物品 1 对应的最后一个元素，也就是相当于 dp[i-1][j]；同理 dp[j-w[i]]也是物品 1 的那列的元素，即相当于 dp[i-1]j-w[i]]。这才是能压缩的根本原因，也是为什么每次更新都需要 j 从大到小遍历。

j 的遍历下限是 w[i]，也就是最远能更新的位置。这时因为在二维数组中如果 j 小于 w[i]，那必然取得是 dp[i-1][j]，相当于 dp[j]，也就是不改变；因此在一维数组中根本就不需要更新 j 小于 w[i]的部分。

**初始化**：常见的初始化是全部初始化为 0，然后对 dp[0]单独处理。一般计算次数、组合数的题目 dp[0]都是 1，如果是取最大值的一般 dp[0]是 0.

**遍历**：刚刚说过每个数组的 j 都必须从后向前，所以内层对 j 的遍历应该是从最后一个元素到 w[i]。

```js
const len = weight.length;
const dp = Array(size + 1).fill(0);

// 一维形式下i不一定必须从1开始，可以看情况，如果dp[0]赋值之后能正常得出结果就不用从1开始
for (let i = 0; i <= len; i++) {
  for (let j = size; j >= weight[i]; j--) {
    dp[j] = Math.max(dp[j], value[i] + dp[j - weight[i]]);
  }
}
return dp[size];
```

> 另外，倒序遍历是为了保证物品 i 只被放入一次。如果正序遍历，那么物品 0 就会被重复加入多次。
> 这句话怎么理解呢？
> 刚刚说过，倒序遍历的时候前面的值还没被更新，相当于是上一个数组的结果。但是正序遍历的时候每次取得的 dp[j-w[i]]就是本数组更新的结果。
> 比如数组第一项是放入一个物品，第二项的 dp[j-w[i]]刚好是第一项，那么再加一个 value[i]，就相当于 dp[j]+2\*value[i]，也就是把物品 0 放入了两次。

### 分割等和子集

> 给你一个 只包含正整数 的 非空 数组  nums 。请你判断是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。
> 示例 1：
> 输入：nums = [1,5,11,5]
> 输出：true
> 解释：数组可以分割成 [1, 5, 5] 和 [11] 。

这道题乍一看像回溯问题，即找到一个组合，使得和恰好为综合的一半即可。
但是回溯问题做出来会超时，要么就需要很复杂的剪枝。因此可以考虑动态规划。
上面的背包问题说过，背包问题是一类问题的解法，就是这种**组合相关的回溯问题**；注意不是所有的回溯都可以这样解，一般来说只有组合等不限制个数的才可以。

还有一点不一样的是，背包问题的 j 只是一个限制，即最大空间不能大于 j；但是这里要求是值必须等于 j，**所以 dp 内部的值建议用布尔值表示能不能满足（能不能等于 j），而不是最大和为多少。**

设 dp[i][j]表示从 0 到 i 的数字任意选，和能不能等于 j。因此这里 dp 应该是一个布尔值数组。注意 j 和 i 都是序号，但是 j 同时充当数字和的效果，而 i 需要 nums[i]取值。
和背包问题类似，dp 也有两个递推方向：

- 如果`j >= nums[i]`，说明当前数字至少可以放入一个 nums[i]，那么只要`dp[i - 1][j]`和` dp[i - 1][j - nums[i]]`有一个为 true 就行。这个地方在背包问题中是求较大值，这里其实原理一样，即选择放入数字 nums[i]或者不放
- else，说明当前 j 完全不能放入数字 nums[i]，那它必然还是等于`dp[i - 1][j]`

初始化：

- 首先`j = 0`时，说明要选数字使和为 0，那肯定都能满足（一个都不选），所以第一数列都为 true
- 对于第一行，意思是只选第一个数字能使得和等于哪个 j，显然就是找到`j === nums[0]`那一项，这时只选一个第一个数就可以满足。

```js
/**
 * @param {number[]} nums
 * @return {boolean}
 */
var canPartition = function (nums) {
  if (nums.length < 2) return false;
  const _sum = nums.reduce((a, b) => a + b);
  if (_sum % 2 !== 0) return false;
  const target = _sum / 2;

  // dp[i][j]表示从0到i的数选几个是否能使和等于j
  const dp = Array.from(new Array(nums.length), () =>
    new Array(target).fill(false)
  );

  for (let i = 0; i < dp.length; i++) {
    dp[i][0] = true;
  }
  dp[0][nums[0]] = true;
  for (let i = 1; i < dp.length; i++) {
    for (let j = 1; j <= target; j++) {
      if (j >= nums[i]) dp[i][j] = dp[i - 1][j] || dp[i - 1][j - nums[i]];
      else dp[i][j] = dp[i - 1][j];
    }
  }
  return dp[dp.length - 1][target];
};
```

还有一种设置 dp 的方法，更好理解一些。
设 dp[j]表示[0,i]的元素组合，得到的和的**不大于 j 的最大值**；如果 dp[j] === j，就说明不大于 j 的最大值刚好可以是 j，也就是满足了题目的“相等要求”。
最后只需要判断最后一项的值等不等于 target 即可。

这种方法实际上还是在算组合最大值，只是判断最大值是不是恰好等于 j。从逻辑上来说更好理解一些

```js
/**
 * @param {number[]} nums
 * @return {boolean}
 */
var canPartition = function (nums) {
  if (nums.length < 2) return false;
  const _sum = nums.reduce((a, b) => a + b);
  if (_sum % 2 !== 0) return false;
  const target = _sum / 2;

  const dp = new Array(target + 1).fill(0);
  dp[0] = 0;
  for (let i = 0; i < nums.length; i++) {
    for (let j = target; j >= 0; j--) {
      if (j >= nums[i]) dp[j] = Math.max(dp[j], dp[j - nums[i]] + nums[i]);
    }
  }
  return dp[dp.length - 1] === target;
};
```

### 最后一块石头的重量 II

https://leetcode.cn/problems/last-stone-weight-ii/

> 有一堆石头，用整数数组  stones 表示。其中  stones[i] 表示第 i 块石头的重量。
> 每一回合，从中选出任意两块石头，然后将它们一起粉碎。假设石头的重量分别为  x 和  y，且  x <= y。那么粉碎的可能结果如下：
>
> 如果  x == y，那么两块石头都会被完全粉碎；
> 如果  x != y，那么重量为  x  的石头将会完全粉碎，而重量为  y  的石头新重量为  y-x。
> 最后，最多只会剩下一块 石头。返回此石头 最小的可能重量 。如果没有石头剩下，就返回 0。

这道题目关键在于怎么能转化成背包问题，实际上解决起来比较简单。
一堆数字每一次都从中取两个，取出来的求差再放入数组，最后反复求得最小值。实际上每组取出来的石头并没有消失，而是变成差的形式放入了数组。
比如：

```
[31,26,33,21,40]
1: 40-21   [19,26,31,33]
2: 31-(40-21)   [12,26,33]
3: 33-(31-(40-21))   [21,26]
4: 26-(33-(31-(40-21)))   [5]
总： (26+31+21) - (40+33)
```

可以看出，最后的结果实际上就是**把石头分成重量相近的两堆，求出他们的差的最小值**。这样就和上面的分割等和子集基本一样了

我们设 dp[i][j]表示从 0 到 i 的石头中选，石头重量不超过 j 时的最大重量。由于要尽可能达到差最小，所以以综合的一半为基准，j 的上限就是 sum/2；这里重量不需要完全相等于 j，因此其实更像背包原始问题，只需要找到小于等于 j 时的最大重量即可。

```js
/**
 * @param {number[]} stones
 * @return {number}
 */
var lastStoneWeightII = function (stones) {
  const _sum = stones.reduce((a, b) => a + b);
  const tar = Math.floor(_sum / 2);
  console.log(tar);
  const dp = Array.from(new Array(stones.length), () =>
    new Array(tar + 1).fill(0)
  );
  // dp[i][j]表示从0到i石头中选，石头重量不超过j时的最大重量。
  for (let j = stones[0]; j < dp[0].length; j++) {
    dp[0][j] = stones[0];
  }
  for (let i = 1; i < dp.length; i++) {
    for (let j = 1; j < dp[0].length; j++) {
      dp[i][j] = Math.max(
        dp[i - 1][j] || 0,
        dp[i - 1][j - stones[i]] + stones[i] || 0
      );
    }
  }
  return Math.abs(_sum - 2 * dp[dp.length - 1][dp[0].length - 1]);
};
```

### 目标和

> 给你一个整数数组 nums 和一个整数 target 。
> 向数组中的每个整数前添加  '+' 或 '-' ，然后串联起所有整数，可以构造一个 表达式 ：
> 例如，nums = [2, 1] ，可以在 2 之前添加 '+' ，在 1 之前添加 '-' ，然后串联起来得到表达式 "+2-1" 。
> 返回可以通过上述方法构造的、运算结果等于 target 的不同 表达式 的数目。

这道题用回溯做很简单，甚至不需要任何剪枝就可以完成。

动态规划解法有点特殊，不是很好想。因为这个回溯并不是组合问题，而是排列问题，需要转成背包问题适合的组合问题。
需要推导一下：

```
假设 a + b + c + d = sum
a + b - c - d = sum - 2(c + d) = target

(c+d) = (sum - target) / 2   这就是负数部分的值
(a+b) = (sum + target) / 2   这是正数部分的值
```

所以其实还是一个分组问题，和上面的分石头类似。
这个问题可以转化成：从数组中的前 i 个数任意取，能凑成`(a+b)`（`c+d`也可以）的**方案数量**。

> 还要注意这道题是求方案。最大差别在于 dp 的意义，所以初始化和求每个 dp 值都要改变。
> 前面几个题的 dp 值都是最大值或者最接近的值（最大价值、最接近的重量、确切的数），并且由于要求的是最接近 j（或者等于 j）的值，所以一半都是求较大值 max 的形式。
> 但是求方案的题目和路径类似，需要的是几种方案的叠加，即中间的计算应该是加法而不是求较大者。

设 dp[i][j]表示从数组中的前 i 个数任意取，能凑成和为 j 的方案数量。
由于是方案数量，所以`dp[0][0]`应该为 1（第一个数要凑成 0，只有一种方案就是不选），其他的全部初始化为 0（实际上整个第一列都应该是 1）

> 注意这道题还有一个很大的不同：j=0 这一项必须有，并且 j 要从 0 开始遍历。
> 因为和是有可能等于 0 的，并且如果有多个 0，[i][0]还可可能有很多种，所以必须从 0 开始遍历。因此 j 的范围应该是[j, tar]
> 另外，这里表示的是“前 i 个数”，而不是`[0,i]`中选。后者通常从 i=1 开始时，就隐式把第一个数抛弃了，然后在初始化的时候考虑只选第一个数的情况。上面三道题都是这种方式。但是这道题我们不能在初始化的时候就确定第一行，还是因为有 0 的原因。所以取具体的数需要 nums[i-1]

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number}
 */
var findTargetSumWays = function (nums, target) {
  const len = nums.length;
  const sum = nums.reduce((a, b) => a + b);
  const tar = (1 / 2) * (sum + target); // 正数那部分的和
  if (Math.floor(tar) !== tar || tar < 0) return 0;

  // dp[i][j]表示从前i个数中选，凑成和为j的数字的**方案数量**
  const dp = Array.from(new Array(len + 1), () => new Array(tar + 1).fill(0));
  dp[0][0] = 1;
  for (let i = 1; i < dp.length; i++) {
    // 注意这里i是从1开始的，但是实际上选数字应该是从0号开始，所以下面都需要i-1
    for (let j = 0; j < dp[0].length; j++) {
      // 如果要凑的数字比当前可选的数字大，说明才可以选，否则一定不能选
      if (j >= nums[i - 1]) {
        dp[i][j] = dp[i - 1][j] + dp[i - 1][j - nums[i - 1]]; // 方案 = 不选当前数字 + 选当前数字。注意关键在于这里是加法而不是取较大
      } else dp[i][j] = dp[i - 1][j];
    }
  }
  console.log(dp);
  return dp[dp.length - 1][dp[0].length - 1];
};
```

当然如果你用一维数组的话其实更简单，只用考虑`dp[0] = 1`，其他全为 0，然后从 i=0 开始推就完了。原因和上面说的一样，i=0 时的情况没办法确定，不能初始化，需要在创建的过程中得出。

```js
var findTargetSumWays = function (nums, target) {
  const sum = nums.reduce((a, b) => a + b);
  const tar = (1 / 2) * (sum + target); // 正数那部分的和
  if (Math.floor(tar) !== tar || tar < 0) return 0;

  const dp = new Array(tar + 1).fill(0); // dp[j]表示凑成目标和为j有多少种方法（从前i个中选数字）
  dp[0] = 1; // 必须为1，不然后面就没法推了
  for (let i = 0; i < nums.length; i++) {
    // i从0开始，因为第一行不能初始化
    for (let j = dp.length - 1; j >= nums[i]; j--) {
      dp[j] += dp[j - nums[i]];
    }
  }
  return dp[dp.length - 1];
};
```

### 完全背包问题

> 有 n 件物品，物品体积用一个名为 w 的数组存起来，物品的价值用一个名为 value 的数组存起来；
> 每件物品的体积用 w[i] 来表示，每件物品的价值用 value[i] 来表示，**每个物品数量无限**。现在有一个容量为 c 的背包
> 问你如何选取物品放入背包，才能使得背包内的物品总价值最大？

完全背包和 01 背包最大的区别在于每个物品的数量没有限制，也就是说一个物品可以被取多次。
完全背包和 01 背包的定义没有区别,dp[i][j]仍然表示从[0,i]中选择物品，放入到容量为 j 的背包中的最大价值
但是完全背包中的每个值都可以选择多次，**每个物品不是“选或不选”，而是“选几个”**；相当于增加了一个选择数量的变量
比如 01 背包中选择一个物品时：

```
dp[i][j] = Math.max(dp[i-1][j], dp[i][j - w[i]] + value[i])
```

这里其实默认了这个物品最多只能选一个
而完全背包就是考虑多个：

```
dp[i][j] = Math.max(dp[i-1][j],
                    dp[i][j - 1*w[i]] + 1*value[i],
                    dp[i][j - 2*w[i]] + 2*value[i],
                    ...,
                    dp[i][j - k*w[i]] + k*value[i]
                    )
```

如果想直接解决完全背包，就需要再加入一个循环遍历 k，这样就成了三重循环，复杂度太高。

在 01 背包中提到过，如果内部数组从小到大遍历（“小”是指 w[i]而不是从 0 开始），相当于每个物品被添加了多次。所以可以考虑使用完全背包的一维表达式。

![](https://img-blog.csdnimg.cn/20210126104510106.jpg)

完全背包的遍历有两种:

- 如果求组合数就是外层 for 循环遍历物品，内层 for 遍历背包
- 如果求排列数就是外层 for 遍历背包，内层 for 循环遍历物品

```js
// 求组合：
for (let i = 0; i <= weight.length; i++) {
  for (let j = weight[i]; j <= bagWeight; j++) {
    dp[j] = Math.max(dp[j], dp[j - weight[i]] + value[i]);
  }
}

// 求排列：
for (let j = 0; j <= bagWeight; j++) {
  for (let i = 0; i < weight.length; i++) {
    if (j >= weight[i]) {
      dp[j] = Math.max(dp[j], dp[j - weight[i]] + value[i]);
    }
  }
}
```

这两个遍历顺序有什么区别呢？
完全背包的 dp[j]依赖于前面的计算出的本数组的 dp[j]，所以只需要计算某个 dp[j]时保证前面是有效元素就可以。

先遍历物品的话，背包里的物品是依次改变的。
而对于物品来说，一定是有序的，因为物品 2 不可能出现在物品 1 之前，只可能有[1,2]这样的组合而不可能有[2,1]这样的。
![](https://img-blog.csdnimg.cn/20210126104529605.jpg)

如果是先遍历背包，相当于是竖着一列一列更新，比如红框 45 接下来就要更新的是另一个物品 2 中的下一个元素。
这时
![](https://code-thinking-1253855093.file.myqcloud.com/pics/20210729234011.png)

> 其他问题的内部迭代：
>
> - 组合、排列问题：
>
> ```js
> dp[j] += dp[j - nums[i]];
> ```
>
> - 存在问题：
>
> ```js
> dp[j] = dp[j] || dp[j - nums[i]];
> ```
>
> - 最值问题：
>
> ```js
> dp[j] = min(dp[j], dp[j - nums[i]] + 1);
> dp[j] = max(dp[j], dp[j - nums[i]] + 1);
> ```

### 零钱兑换

#### 零钱兑换1——最少数量

https://leetcode.cn/problems/coin-change/

> 给定不同面额的硬币 coins 和一个总金额 amount。
> 编写一个函数来计算可以凑成总金额所需的 最少 的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 -1。


这道题实际上是一个完全背包问题。我们把 coins 看作物品价值，要凑成的 amount 看作是重量，即求 coins 的组合，使得凑得 amount 的数量最小。

设dp[i][j]表示选择前i个硬币，当amount为j时的最小硬币数量，那么这里的value应该是1，相当于每次增加1个硬币

```js
dp[i][j] = min(dp[i-1][j-coins[i]] + 1,dp[i-1][j])
```

用一维数组表示的递推式就应该是：

```js
dp[j] = min(dp[j], dp[j - coins[i]] + 1);
```

不过其实这道题经常拿出来不是当做完全背包考的，而是普通的一维dp。我们设dp[i]表示凑成金额为i时的最小硬币数量，每次遍历所有硬币，可以选择不选当前硬币(dp[i])或者选当前硬币(dp[i-coins[j]] + 1)，取两个最小值即可。


如果给这道题换一个问题，即问你要凑出 amount 的硬币有多少种，其实就是对上面式子的小改进

```js
dp[j] += dp[j - coins[i]];
```

放入循环：

```js
const getCoinsDynamic = (coins, amount) => {
  const f = [];
  f[0] = 0;
  for (let i = 1; i <= amount; i++) {
    f[i] = Infinity;
    // 这个循环相当于是 f[n] = Math.min(f(n-coin1),f(n-coin2),...,f(n-coinn))
    // 由于coin数量不确定,因此两两比较每个f(n)和f(n-coin)
    for (let coin of coins) {
      if (i >= coin) f[i] = Math.min(f[i], f[i - coin] + 1);
    }
  }
  if (f[amount] === Infinity) return -1;
  return f[amount];
};
```



#### 零钱兑换2——组合总数

https://leetcode.cn/problems/coin-change-ii/

> 给你一个整数数组 coins 表示不同面额的硬币，另给一个整数 amount 表示总金额。
> 请你计算并返回可以凑成总金额的硬币组合数。如果任何硬币组合都无法凑出总金额，返回 0 。
> 假设每一种面额的硬币有无限个。

这道题是上面的零钱兑换的变种，实际上就是把求最少硬币个数变成求组合个数。
因为一个零钱可以取无数次，所以是一个典型的完全背包问题。套入完全背包的`dp[i][j]= dp[i−1][j] + dp[i][j−w[i]]`即可。

设 dp[i][j]表示要凑成总面额为 j 的硬币组合种数，显然第一列（凑成 0）全都是 1，其他的都是 0.

```js
/**
 * @param {number} amount
 * @param {number[]} coins
 * @return {number}
 */
var change = function (amount, coins) {
  const len = coins.length;
  const dp = Array.from(new Array(len + 1), () =>
    new Array(amount + 1).fill(0)
  );
  for (let i = 0; i < dp.length; i++) {
    dp[i][0] = 1;
  }
  for (let i = 1; i < dp.length; i++) {
    for (let j = 1; j < dp[0].length; j++) {
      if (j >= coins[i - 1]) {
        dp[i][j] = dp[i - 1][j] + dp[i][j - coins[i - 1]];
      } else {
        dp[i][j] = dp[i - 1][j];
      }
    }
  }
  console.log(dp);
  return dp[dp.length - 1][dp[0].length - 1];
};
```

这道题的一维数组形式：

```js
const change = (amount, coins) => {
  let dp = Array(amount + 1).fill(0);
  dp[0] = 1;
  // 一维形式下i不一定必须从1开始，一般给dp[0]设好默认值就可以
  for (let i = 0; i < coins.length; i++) {
    for (let j = coins[i]; j <= amount; j++) {
      dp[j] += dp[j - coins[i]];
    }
  }

  return dp[amount];
};
```


### 组合总和 IV

https://leetcode.cn/problems/combination-sum-iv/

> 给你一个由 不同 整数组成的数组 nums ，和一个目标整数 target 。请你从 nums 中找出并返回总和为 target 的元素组合的个数。
> 示例:
> nums = [1, 2, 3] target = 4
> 所有可能的组合为： (1, 1, 1, 1) (1, 1, 2) (1, 2, 1) (1, 3) (2, 1, 1) (2, 2) (3, 1)
> 请注意，顺序不同的序列被视作不同的组合。

这道题每个元素可重复选择，因此是一个完全背包的问题，而且是一个排列问题（因为这道题允许相同数字不同排列的出现）

> 组合不强调顺序，(1,5)和(5,1)是同一个组合；排列强调顺序，(1,5)和(5,1)是两个不同的排列。
> 如果求组合数就是外层 for 循环遍历物品（i，行），内层 for 遍历背包（j，列），**并且内部应该是反向的**
> 如果求排列数就是外层 for 遍历背包（j，列），内层 for 循环遍历物品（i，行），内部正向

所以需要把物品 nums 放在内部遍历。
注意这种排列的问题，用二维数组已经解不了了，必须用一维数组。

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number}
 */
var combinationSum4 = function (nums, target) {
  const dp = new Array(target + 1).fill(0);

  dp[0] = 1;

  for (let j = 1; j <= target; j++) {
    // j表示目标和，即背包，列，在外部遍历
    for (let i = 0; i < nums.length; i++) {
      // i表示每一项数字
      if (j >= nums[i]) {
        dp[j] += dp[j - nums[i]];
      }
    }
  }
  console.log(dp);
};
```

### 完全平方数

https://leetcode.cn/problems/perfect-squares/

> 给你一个整数 n ，返回 和为 n 的完全平方数的最少数量 。
> 完全平方数 是一个整数，其值等于另一个整数的平方；换句话说，其值等于一个整数自乘的积。例如，1、4、9 和 16 都是完全平方数，而 3 和 11 不是
> 示例  1：
> 输入：n = 12
> 输出：3
> 解释：12 = 4 + 4 + 4

这道题也是一个典型的完全背包，让你在[1,4,9....10000]这些数中选可重复的数凑成目标数 n，求凑成的最少数字数量（类似最少零钱问题）

按照完全背包的方法，组合题外层遍历物品（数字），内层遍历背包容量（n）。因为是求最小值，所以初始化 dp 全为 Infinity，且 dp[0] = 0 即可。

```js
var numSquares = function (n) {
  if (n === 1) return 1;
  const nums = new Array(100).fill(0).map((val, index) => (index + 1) ** 2);
  if (nums.includes(n)) return 1;
  let end = 0;
  while (nums[end] <= n) end++;

  const dp = new Array(n + 1).fill(Infinity);
  dp[0] = 0;

  for (const num of nums) {
    for (let j = num; j <= n; j++) {
      dp[j] = Math.min(dp[j], dp[j - num] + 1);
    }
  }
  return dp[n];
};
```

这道题如果用暴力回溯则会超时

### 单词拆分

> 给你一个字符串 s 和一个字符串列表 wordDict 作为字典。请你判断是否可以利用字典中出现的单词拼接出 s 。
> 注意：不要求字典中出现的单词全部都使用，并且字典中的单词可以重复使用。
> 示例 1：
> 输入: s = "leetcode", wordDict = ["leet", "code"]
> 输出: true
> 解释: 返回 true 因为 "leetcode" 可以由 "leet" 和 "code" 拼接成。

这道题和回溯中的分割回文串很像，只不过题干说的是“组合”，实际上也可以对字符串进行分解。
因此这道题其实有好几个思路：

1. 组合 wordDict 中的单词，找到匹配的。计算量最大，基本上会超时
2. 分割 s，每次分割的结果在 wordDict 中查验，如果有就记录并把剩余字符串继续回溯检查。参考上面分割回文串的方法
3. 动态规划，完全背包，把字符串看作是列，每列刚好对应一个字符；每行对应一个 wordDict 中的单词。每次迭代取前面一部分，然后判断是不是符合 wordDict[i]即可。

设 dp[j]表示截止到位置 j 的字符串，能不能匹配到 wordDict 中的单词。显然这是一个 boolean 数组，并且 dp[0]表示截止到位置 0，也就是空串，对应不选，所以为 true.（这里有点强行解释了，但是第一项一般是不为 0 或者 false 的，要不然后面就不好算了）

至于遍历顺序，其实两种都可以。但是题解上建议是外层遍历字符，内层遍历单词，因为一个串的匹配应该是从前到后一个个查找的，想多来说更合理更好想一点。

下面是动态规划的代码。内部实际上就是常见的匹配 dp 的形式（`dp[i] || dp[i - weight[i]]`）

```js
/**
 * @param {string} s
 * @param {string[]} wordDict
 * @return {boolean}
 */
var wordBreak = function (s, wordDict) {
  const dp = new Array(s.length + 1).fill(false);
  dp[0] = true;
  for (let j = 1; j <= s.length; j++) {
    // 外层遍历字符
    for (let i = 0; i < wordDict.length; i++) {
      // 内层遍历单词
      if (
        j >= wordDict[i].length &&
        s.slice(j - wordDict[i].length, j) === wordDict[i] // j位置前面的wordDict[i].length长度的单词是不是恰好匹配到了当前单词
      ) {
        dp[j] = dp[j] || dp[j - wordDict[i].length];
      }
    }
  }
  return dp[dp.length - 1];
};
```

## 买卖股票 I

> 给定一个数组 prices ，它的第 i 个元素  prices[i] 表示一支给定股票第 i 天的价格。
> 你只能选择 某一天 买入这只股票，并选择在 未来的某一个不同的日子 卖出该股票。设计一个算法来计算你所能获取的最大利润。
> 返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 0 。
> 示例 1：
> 输入：[7,1,5,3,6,4]
> 输出：5
> 解释：在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。

设 dp[i]表示第 i 天持有股票的现金。
为了区分在某一天购入和售出，设置两个状态表示在改天持有和不持有股票：

- dp[i][0]表示在第 i 天持有股票。**持有**股票表示两种可能：
  - 今天购买了股票，那么现在剩余的现金就是`-price[i]`
  - 今天没有购买，保持昨天的状态，即`dp[i-1][0]`
- dp[i][1]表示在第 i 天不持有股票，同理：
  - 今天售出了股票，那么剩余现金就是 当天的股票价格 price[i]和前一天的持有金钱的差值；又因为 dp[i][0]一定是小于等于 0，因此实际上是`price[i] + dp[i-1][0]`。
    并且，`dp[i-1][0]`本身是由前面推出来的，因此一定在这天之前是购买成本最小的情况。
  - 今天没有售出，即保持`dp[i - 1][1]`

因此实际上 dp 是两个动态规划的组合。

```
dp[i][0] = Math.max(dp[i-1][0],-price[i])

dp[i][1] = Math.max(dp[i-1][1],price[i] + dp[i-1][0])
```

最终结果取`dp[price.length - 1][1]`就是最大值。

![](https://pic.imgdb.cn/item/6281dbeb09475431290d7d2e.jpg)

代码如下：

```js
/**
 * @param {number[]} prices
 * @return {number}
 */
var maxProfit = function (prices) {
  const dp = [[-prices[0], 0]];
  for (let i = 1; i < prices.length; i++) {
    dp[i] = [];
    dp[i][0] = Math.max(dp[i - 1][0], -prices[i]);
    dp[i][1] = Math.max(dp[i - 1][1], prices[i] + dp[i - 1][0]);
  }
  return dp[prices.length - 1][1];
};
```

## 买卖股票 II

https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/

> 给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。
> 设计一个算法来计算你所能获取的最大利润。你可以尽可能地完成更多的交易（多次买卖一支股票）。
> 注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

这道题和上一道题的唯一区别在于：dp[i][0]的推导有一点点不一样。
因为上一道题购买的时候一定是不持有现金的，因此必然是`-prices[i]`；但是本题购买的时候可能是已经持有现金了，所以就需要把值变为`dp[i][1] - prices[i]`。
即可

```js
dp[i][0] = Math.max(dp[i - 1][0], dp[i][1] - prices[i]);

dp[i][1] = Math.max(dp[i - 1][1], price[i] + dp[i - 1][0]);
```

带入代码得：

```js
/**
 * @param {number[]} prices
 * @return {number}
 */
var maxProfit = function (prices) {
  const dp = [[-prices[0], 0]];
  for (let i = 1; i < prices.length; i++) {
    dp[i] = [];
    dp[i][1] = Math.max(dp[i - 1][1], prices[i] + dp[i - 1][0]);
    // 注意顺序稍微改了一下
    dp[i][0] = Math.max(dp[i - 1][0], dp[i][1] - prices[i]);
  }
  return dp[prices.length - 1][1];
};
```

## 买卖股票含冷冻期

> 给定一个整数数组，其中第 i 个元素代表了第 i 天的股票价格 。
>
> 设计一个算法计算出最大利润。在满足以下约束条件下，你可以尽可能地完成更多的交易（多次买卖一支股票）:
>
> 你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。
> 卖出股票后，你无法在第二天买入股票 (即冷冻期为 1 天)。
> 示例: 输入: [1,2,3,0,2] 输出: 3 解释: 对应的交易状态为: [买入, 卖出, 冷冻期, 买入, 卖出]

包含冷冻期实际上就是在前两个的基础上添加了“冷冻”状态（实际上是两个状态）。
上面两个基本的买卖股票只有卖出和买入两个状态，卖出状态的 dp 值只能从自己或买入推出来，买入同理；
这里其实就是添加了两个和冷冻期的状态，然后分别推出四个状态的 dp 方程。每个状态下的 dp 都和其他一些状态有关系，综合起来就可以得到每个状态的每个值，最终结果就可以得到。

一共设置四个状态 j：

0. 买入状态，即持有股票状态
1. 卖出但不在冷冻期状态，即前两天就已经卖出了，现在可以随时买入而不受冷冻期的限制
2. 今天刚卖出状态，即今是冷冻期，明天就转换成 1
3. 冷冻期状态

注意后两个状态都是不可持续的，即`dp[i][2]`和`dp[i][3]`都不能从`dp[i-1][2]`或`dp[i-1][3]`推出来，只能从其他状态得到

接下来就是分别得到这四个状态的转移方程：

买入有可能是三种情况：

1. 状态 1 转来的，即前两天卖出今天买入
2. 保持买入状态
3. 状态 3 转来的，即前一天是状态 3 冷冻期

卖出可能是两种情况：

5. 保持卖出状态
6. 前一天是状态 3 冷冻期，即前一天刚卖出，过了冷冻期但是还没有任何操作。

刚卖出状态只有一种可能，就是前一天是状态 0 刚买入今天就卖出
冷冻期也只有一种可能，就是前一天是状态 2 刚卖出

因此得到四个状态转移方程：

```js
dp[i][0] = Math.max(
  dp[i - 1][1] - prices[i],
  dp[i - 1][3] - prices[i],
  dp[i - 1][0]
);
dp[i][1] = Math.max(dp[i - 1][1], dp[i - 1][3]);
dp[i][2] = dp[i - 1][0] + prices[i];
dp[i][3] = dp[i - 1][2];
```

完整代码：

```js
/**
 * @param {number[]} prices
 * @return {number}
 */
var maxProfit = function (prices) {
  const dp = Array.from(new Array(prices.length), () => new Array(4).fill(0));
  dp[0][0] = -prices[0];
  let max = 0;
  for (let i = 1; i < prices.length; i++) {
    dp[i][0] = Math.max(
      dp[i - 1][1] - prices[i],
      dp[i - 1][3] - prices[i],
      dp[i - 1][0]
    );
    dp[i][1] = Math.max(dp[i - 1][1], dp[i - 1][3]);
    dp[i][2] = dp[i - 1][0] + prices[i];
    dp[i][3] = dp[i - 1][2];
    max = Math.max(max, dp[i][0], dp[i][1], dp[i][2], dp[i][3]);
  }
  console.log(dp);
  return max;
};
```

---

实际上“卖出且不在冷冻期”，和“卖出并在冷冻期”、“买入”这三个状态就可以。

这三个状态的推导：
1. 当天要买入，那么前一天必须是冷冻期，即必须从冷冻期状态推导，不能直接从卖出推导。其实就是买卖股票II里边的推导方式，只是这里要从冷冻期状态下推导
2. 当天卖出，那么后一天就必须是冷冻期。对卖出来说，推导方式和买卖股票II完全相同
3. 当天处在冷冻期，那么后一天可以买入，而前一天应该是刚好卖出的。



设:

- `dp[i][0]`表示今天买入
- `dp[i][1]`表示今天卖出，那么下一天就是冷冻期，买入不能从这一天继承值；
- `dp[i][2]`表示今天是冷冻期，只能从前一天卖出的状态推导来。

那么方程为：

```js
dp[i][1] = Math.max(dp[i - 1][1], dp[i - 1][0] + prices[i])
dp[i][2] = dp[i - 1][1]
dp[i][0] = Math.max(dp[i - 1][2] - prices[i], dp[i - 1][0])
```

## 买卖股票III和IV

III和IV其实本质上是同一道题，都是限制了交易次数。我们把每次买入看做是一次交易，在原来的dp基础上增加一个k维度，当每次买入时从k-1推导就可以。

有几个关键点：
1. 三维数组的创建方法：其实就是在最后一个数组map一下就可以，创建一个`n * (k + 1) * 2`的数组
2. 初始化：这个的初始化比前几道麻烦一些，因为是三维数组。具体来说，在迭代过程中不会得出k = 0的情况，但却需要k = 0的结果，因此我们需要在每个i内，都把k=0的情况赋值为0。
另外当i = 0时，交易次数大于0的`dp[i][k][0]`应该为`-price[i]`，相当于总共交易次数为k次时，第一次买入的利润。其他的都初始为0即可。

具体实现：

```js
var maxProfit = function (K, prices) {
    const dp = Array.from(new Array(prices.length), () => new Array(K + 1).fill(0).map(() => new Array(2)))
    for (let i = 0; i < prices.length; i++) {
        for (let k = K; k >= 1; k--) {
            if (i === 0) {
                dp[i][k][0] = -prices[i]
                dp[i][k][1] = 0
                continue
            }
            dp[i][k][0] = Math.max(dp[i - 1][k][0], dp[i - 1][k - 1][1] - prices[i])
            dp[i][k][1] = Math.max(dp[i - 1][k][1], dp[i - 1][k][0] + prices[i])
        }
        dp[i][0][0] = 0
        dp[i][0][1] = 0
    }
    return dp[dp.length - 1][K][1]
};
```

## 打家劫舍 I

> 你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。
> 给定一个代表每个房屋存放金额的非负整数数组，计算你 不触动警报装置的情况下 ，一夜之内能够偷窃到的最高金额。
> 示例 1： 输入：[1,2,3,1] 输出：4 解释：偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。 偷窃到的最高金额 = 1 + 3 = 4 。
> 示例 2： 输入：[2,7,9,3,1] 输出：12 解释：偷窃 1 号房屋 (金额 = 2), 偷窃 3 号房屋 (金额 = 9)，接着偷窃 5 号房屋 (金额 = 1)。 偷窃到的最高金额 = 2 + 9 + 1 = 12 。

这道题的关键在于怎么考虑“不偷相邻家”的问题。
设 dp[i]表示偷到第 i 家时的最高金额，那么就会有两种情况：

1. 选择偷第 i 家，那么前一个就不能被偷，也就是说只能偷更前面的。至于具体是哪个不用管，就直接用`dp[i-2]`代替就可以（因为这个值一定代表 i-2 及之前的最大值）
2. 选择不偷第 i 家，那么就保持 dp[i-1]即可。

当数组长度小于等于 2 时，可以直接取最大值，所以数组至少要有三项才能开始推；
初始化全 0，开始偷东西时至少从第一项或第二项开始，所以应该初始化 dp[0]和 dp[1];

- dp[0]表示偷到第一家时的金额，应该是 nums[0]；
- dp[1]应该取 max(dp[0],dp[1])，即如果第二项比第一项大，那就不偷第一项了，直接从第二项开始。

代码如下：

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
var rob = function (nums) {
  // 设dp[i]表示截止到第i个房屋偷窃到的最大金额
  if (nums.length <= 2) return Math.max(...nums);
  const dp = new Array(nums.length).fill(0);
  dp[0] = nums[0];
  dp[1] = Math.max(nums[0], nums[1]);

  for (let i = 2; i < dp.length; i++) {
    dp[i] = Math.max(dp[i - 1], dp[i - 2] + nums[i]);
  }
  return dp[dp.length - 1];
};
```

回到一开始的问题，怎么去判断邻家的问题？
其实就是在偷的时候做出改变：如果选择要偷，那一定不考虑临近的一个。不偷的时候则不需要刻意避开。
也就是说，遍历给偷指定了一个“范围”，即考虑偷这个范围内的，但是具体偷哪个则需要选出最大值决定。
其实这道题也就是最长上升子序列的“不可相邻”版。

---

另外一种思路是利用递归，其实更类似于暴力回溯的方式。
设置一个 rob 的递归函数，rob 传入开始偷的位置，返回从这个房间开始偷得到的最大值：

```js
var rob = function (nums) {
    const memo = new Map()
    const robRoom = (index) => {
        if (index >= nums.length) return 0
        if (memo.has(index)) return memo.get(index)
        const val = Math.max(robRoom(index + 1), robRoom(index + 2) + nums[index])
        memo.set(index, val)
        return val
    }
    return robRoom(0)
};
```

这个方式在普通线性中不如 dp 数组好理解，但是在打家劫舍 III 的树形结构中，只需要把 dfs(i+1)/dfs(i+2)的判断换为 dfs(left)、dfs(left.left/right)即可

## 打家劫舍 II

> 你是一个专业的小偷，计划偷窃沿街的房屋，每间房内都藏有一定的现金。这个地方所有的房屋都 围成一圈 ，这意味着第一个房屋和最后一个房屋是紧挨着的。同时，相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警 。
> 给定一个代表每个房屋存放金额的非负整数数组，计算你 在不触动警报装置的情况下 ，能够偷窃到的最高金额。
> 示例 1：
> 输入：nums = [2,3,2] 输出：3 解释：你不能先偷窃 1 号房屋（金额 = 2），然后偷窃 3 号房屋（金额 = 2）, 因为他们是相邻的。

这道题不一样之处在于头尾相邻，但是也只是头尾相邻，其他情况和第一种完全一样

之前思考过可以偷过一遍之后再转一圈再偷一遍，但是自己想想是不可能的，第二圈再偷任何一家都是至少有一个偷过的相邻的，如果有没偷过相邻的那一定在第一遍就偷过了。因此只需要考虑一次遍历就够了。

如果第一家偷过的话，最后一家就一定不偷；如果从第二家开始，就一定要包含最后一家。
那怎么判断第一家还是第二家偷了呢？之前考虑判断`dp[1] === nums[0]`，如果是就说明偷了第一家；但是这个方法在第一家和第二家一样的时候就不能用了。
因此应该从序号入手，计算两次，第一次是直接从 nums[1]开始；第二次从 nums[0]开始，到了倒数第二项就结束即可。最后取两个的较大值

**注意：这道题的细节很多，要关注细节！！！！**

主要细节在注释里


```js
/**
 * @param {number[]} nums
 * @return {number}
 */
var rob = function (nums) {
  const n = nums.length;
  if (n === 0) return 0;
  if (n === 1) return nums[0];
  const result1 = robRange(nums, 0, n - 2);
  const result2 = robRange(nums, 1, n - 1);
  return Math.max(result1, result2);
};

const robRange = (nums, start, end) => {
  if (end === start) return nums[start];
  // 这里dp正常创建就可，因为下面是从start开始的，可能是1或者0
  const dp = Array(nums.length).fill(0);
  // 注意这里，不要设置dp[0]或dp[1]，而是直接设置dp[start]，相当于跳过了0和1的讨论，直接根据start确定起始元素
  dp[start] = nums[start];
  // 这个地方需要取第一位和第二位的最大值，不要忘了
  dp[start + 1] = Math.max(nums[start], nums[start + 1]);
  for (let i = start + 2; i <= end; i++) {
    dp[i] = Math.max(dp[i - 2] + nums[i], dp[i - 1]);
  }
  // 这里应该直接返回end，和dp计算的下界一致，不要dp[dp.length - 1]，因为当start=1时，最后一项和start=0的不同
  return dp[end];
};
```

## 打家劫舍 III

https://leetcode.cn/problems/house-robber-iii/

打家劫舍 III 和前两个题不同的在于变成了树形遍历。
如果直接按照前两个的思路使用 dp 就会出问题，因为树形的遍历和数组的线性结构不一致。
因此更好的方式是类回溯递归。对于一个节点：

- 如果偷这个节点，那就不偷紧挨着的两个（root.left 和 root.right），而是直接偷更后面的四个（root.left.left/root.left.right/root.right.left/root.right.right）
- 如果不偷这个节点，那就偷紧挨的两个，直接递归 root.left 和 root.right 即可，表示从这两个节点开始计算

另外，还需要一个 map 存储节点，如果下次遍历到这个节点就直接返回偷的值而不用递归。因为每个点对应的偷的值应该是固定的，不用重复计算。

```js
var rob = function (root) {
    const memo = new Map()
    const robRoom = (node) => {
        if (!node) return 0
        if (memo.has(node)) return memo.get(node)
        let amount1 = robRoom(node.left) + robRoom(node.right)
        let amount2 = node.val
        if (node.left) amount2 += robRoom(node.left.left) + robRoom(node.left.right)
        if (node.right) amount2 += robRoom(node.right.left) + robRoom(node.right.right)
        const res = Math.max(amount1,amount2)
        memo.set(node, res)
        return res
    }
    return robRoom(root)
};
```

## 最大子数组和（最大子序和）

> 给你一个整数数组 nums ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

用 `f(i)`代表以第 i 个数结尾的「连续子数组的最大和」,可得方程为

```
f(i)=max{f(i−1)+nums[i],nums[i]}
```

也就是以第 i 个数结尾的最大和，有可能是加上第 i 个数的结果，也有可能是只有第 i 个数的结果。因为`f(i)`代表以第 i 个数结尾的,因此必须要包含当前数(即`nums[i]`)；

```js
const maxChildArr = (nums) => {
  const f = [];
  f[0] = nums[0];
  for (let i = 1; i < nums.length; i++) {
    f[i] = Math.max(f[i - 1] + nums[i], nums[i]);
  }
  return Math.max(...f);
};
```

---

如果要求是一个最大子序列呢？即不要求连续数组

其实就是改一下方程为：

```js
dp[i] = Math.max(dp[i - 1], dp[i - 1] + nums[i]);
```

因为这样每一项都可以选择其前面的某一项和，当自己为正时就加上自己即可。
而如果要求必须是子数组，那么一旦不合适就应该抛弃前面的子数组，重新以自己为起点创建数组。

## 最大子数组积（乘积最大子数组）

https://leetcode.cn/problems/maximum-product-subarray

这道题和上一道题很像，上一个题是最大子数组和，这道题是子数组的积。

区别在于，由于积有“负负得正”的性质，所以对于一个位置上的数来说，如果这个数是负数，并不一定代表它只能单独起一个子数组，它还可以和前面某个乘积为负数的结果相乘得到正数

因此，可以考虑维护两个 dp 数组，dp1 表示以当前数结尾的子数组的积的最**大**值，dp2 表示当前数结尾的子数组的积的最**小**值。
每遍历到一个数

- 如果这个数是负数，就考虑和 dp2[i-1]相乘，得到的乘积还要和 dp1[i-1]比较，如果由于“负负得正”使得乘积更大了，那就替换 dp1[i]。
- 如果这个数是正数，dp1[i]正常由`dp1[i-1]*nums[i]`得到，dp2[i]也要考虑`dp2[i-1]*nums[i]`的结果得到新的更小的值。

因此实际代码中，dp1 和 dp2 每次递推都是同时和另外一个数组的`nums[i] * dpn[i-1]`比较，一起比较取得最大/最小值即可：

```js
var maxProduct = function (nums) {
  const dp1 = new Array(nums.length); // 以i为最后一个位置的子数组的积的最大值
  const dp2 = new Array(nums.length); // 以i为最后一个位置的子数组的积的最小值
  dp1[0] = nums[0];
  dp2[0] = nums[0];
  for (let i = 1; i < nums.length; i++) {
    // 最大值可能有两个来源，一个是从dp2“负负得正”得来，一个是从dp1“正正得正”得来
    // 最小值同理
    dp1[i] = Math.max(nums[i], nums[i] * dp1[i - 1], nums[i] * dp2[i - 1]);
    dp2[i] = Math.min(nums[i], nums[i] * dp2[i - 1], nums[i] * dp1[i - 1]);
  }
  return Math.max(...dp1);
};
```

## 最长上升子序列

> 给定一个无序的整数数组，找到其中最长上升子序列的长度。

假设最长序列长度为`f(i)`,这道题不适合倒推得出方程,而是通过正向思考
对于每一个数字,如果前面还有比它小的一个数字或一组数字,就可以组合成一个序列;
那么可以执行两次遍历,遍历每个数字,每次迭代都遍历**这个数字前面的所有数字**,如果有比它小的就加入序列长度，直到找到该项之前的、比该项小的数字个数

```js
const lengthOfLIS = (nums) => {
  const f = [];
  f[0] = 1;
  for (let i = 1; i < nums.length; i++) {
    f[i] = 1;
    for (let j = 0; j < i; j++) {
      if (nums[i] > nums[j]) {
        //注意这里不是要f[i] 与 f[j] + 1进行比较，而是取f[j] + 1的最大值。
        f[i] = Math.max(f[i], f[j] + 1);
      }
    }
  }
  return Math.max(...f);
};
```

核心代码写成这样也可以：

```js
for (let i = 1; i < nums.length; i++) {
  f[i] = 1;
  let tmp = 0;
  for (let j = 0; j < i; j++) {
    if (nums[i] > nums[j]) {
      // 取f[j] + 1的最大值。
      tmp = tmp + 1;
    }
  }
  f[i] = tmp;
}
```

## 最长递增子序列的个数

https://leetcode.cn/problems/number-of-longest-increasing-subsequence/

> 给定一个未排序的整数数组  nums ，  返回最长递增子序列的个数  。
> 注意   这个数列必须是 严格 递增的。
> 示例 1:
> 输入: [1,3,5,4,7]
> 输出: 2
> 解释: 有两个最长递增子序列，分别是 [1, 3, 4, 7] 和[1, 3, 5, 7]。

思路：用两个数组 dp 和 cnt 分别记录最长递增子序列的长度和数量。

- dp 比较好算，dp[i]就是从 0 到 i 的最大值+1
- cnt 的变化依赖 dp；由于 cnt 表示的是第 i 个位置上的最长递增子序列数量，那么 cnt 就会有两种情况：
  - 当当前 dp 被更新，即此时 0 到 i 中有一个位置的长度比当前 dp 大，说明最长递增子序列的长度增加了；如果只是增加长度，那么数量不会发生改变，即 cnt[i] = cnt[j]。
  - 当 dp 不变（dp[i] = dp[j] + 1），即找到了两个相同长度的递增子序列，那么此时就应该把这个位置上的 cnt[j]和当前的 cnt[i]加起来。
- 最后，找到 dp 中最大的值，然后在 dp 中遍历出现最大值的位置，对应到 cnt 的出现次数，就是这个最大序列值的出现次数，全部加起来即可。

这里的关键就是这个 cnt 数组的动态规划计算，什么时候 cnt 应该变化
以上面的[1,3,5,4,7]为例：

1. 当 i = 4，j = 2，j 走到 5 这个位置上，dp[i]被更新为 4
2. 当 j = 3，也就是走到 4 这个位置上时，这时发现之前计算的最长子序列长度又出现了一次（4），说明 5 和 4 都可以和 7 构成最长序列。那么显然就应该加上这个位置的 cnt。

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
var findNumberOfLIS = function (nums) {
  let len = nums.length;
  let dp = Array(len).fill(1);
  let count = Array(len).fill(1);
  let res = 0;
  for (let i = 0; i < len; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[i] > nums[j]) {
        if (dp[j] + 1 > dp[i]) {
          dp[i] = dp[j] + 1;
          count[i] = count[j];
        } else if (dp[j] + 1 === dp[i]) {
          count[i] += count[j];
        }
      }
    }
  }
  let max = Math.max(...dp);
  for (let i = 0; i < len; i++) {
    if (dp[i] === max) {
      // 在 dp 中遍历出现最大值的位置
      res += count[i]; // 对应到 cnt 的出现次数
    }
  }
  return res;
};
```

## 最长连续递增子序列

> 给定一个未经排序的整数数组，找到最长且 连续递增的子序列，并返回该序列的长度。连续递增的子序列 可以由两个下标 l 和 r（l < r）确定，如果对于每个 l <= i < r，都有 nums[i] < nums[i + 1] ，那么子序列 [nums[l], nums[l + 1], ..., nums[r - 1], nums[r]] 就是连续递增子序列。

这道题和上面的最大区别在于“连续”，即序列是连续的。
因此同样设`dp[i]`表示以 i 结尾的最长连续递增序列的长度，但是`dp[i]`此时只能由`dp[i - 1]`推出，即`dp[i] = dp[i - 1] + 1`。

遍历整个数组，如果当前数`nums[i]`比前面的数大，就更新`dp[i]`，代码如下：

```js
var findLengthOfLCIS = function (nums) {
  const dp = [];
  let res = 0;
  dp[0] = 1;
  for (let i = 1; i <= nums.length; i++) {
    dp[i] = 1;
    if (nums[i] > nums[i - 1]) dp[i] = dp[i - 1] + 1;
    res = res > dp[i] ? res : dp[i];
  }
  return res;
};
```

## 最长公共子数组

> 给两个整数数组 A 和 B ，返回两个数组中公共的、长度最长的子数组的长度。
> 示例：
> 输入： A: [1,2,3,2,1] B: [3,2,1,4,7] 输出：3 解释：长度最长的公共子数组是 [3, 2, 1] 。

类似这种的公共问题，一定是取二维数组`dp[i][j]`，并且两个维度分别对应一个数组。
这里`dp[i][j]`表示：以下标`i - 1`为结尾的 A，和以下标`j - 1`为结尾的 B，最长重复子数组长度为`dp[i][j]`。

- 当`A[i - 1] === B[j - 1]`相等的时候，`dp[i][j] = dp[i - 1][j - 1] + 1`
- 其他情况下（i 和 j 不相等），`dp[i][j]`为 0。归零的目的主要是相当于“从头开始”

> 比如说序列[3,5,4,2,1]和[3,2,1,4,5]，在 i-1 = 1 和 j-1 = 1 时发现对应的数字不相等，这时如果不设 dp 为 0，那么后面的就可能会继续在 1 的基础上计算；但是很显然 3 5 4 2 和 3 2 这两个序列的子数组长度不能算作是 2，因为不连续；所以就必须在对应的值不相同的时候直接把 dp 归为 0，类似于最长递增子数组的那种“重新开始数”的方式

> 这里取`i-1`和`j-1`主要是为了方便计算，直接取 i、j 也可以，但是要提前初始化`dp[i][0]`和 dp`[0][j]`

dp 数组的结构如下，状态的更新是斜向的，即 `dp[i][j]`总是由左上方的 `dp[i - 1][j - 1]`更新而来

![](https://img-blog.csdnimg.cn/2021011215282060.jpg)

代码如下：

```js
const findLength = (A, B) => {
  // A、B数组的长度
  const [m, n] = [A.length, B.length];
  // dp数组初始化，都初始化为0
  const dp = new Array(m + 1).fill(0).map(() => new Array(n + 1).fill(0));
  // 初始化最大长度为0
  let res = 0;
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      // 遇到A[i - 1] === B[j - 1]，则更新dp数组
      if (A[i - 1] === B[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1] + 1;
      }
      // 更新res
      res = dp[i][j] > res ? dp[i][j] : res;
    }
  }
  // 遍历完成，返回res
  return res;
};
```

## 最长公共子序列

> 给定两个字符串 text1 和 text2，返回这两个字符串的最长公共子序列的长度。
> 一个字符串的 子序列 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。
> 例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。两个字符串的「公共子序列」是这两个字符串所共同拥有的子序列。
> 若这两个字符串没有公共子序列，则返回 0。
> 示例 1:
> 输入：text1 = "abcde", text2 = "ace" 输出：3 解释：最长公共子序列是 "ace"，它的长度为 3。
> 示例 2: 输入：text1 = "abc", text2 = "abc" 输出：3 解释：最长公共子序列是 "abc"，它的长度为 3。
> 示例 3: 输入：text1 = "abc", text2 = "def" 输出：0 解释：两个字符串没有公共子序列，返回 0。

这道题是上一个公共子数组的序列版本。
还是设 dp[i][j]表示 text1[i-1]和 text2[j-1]之间的最长公共子序列长度。

- 当 text1[i - 1] 与 text2[j - 1]相同，那么找到了一个公共元素，所以 dp[i][j] = dp[i - 1][j - 1] + 1;
- 如果 text1[i - 1] 与 text2[j - 1]不相同，注意这里和子数组的处理不一样。因为子数组要求必须连续，所以如果有不同就之间抛弃就行；但是子序列可以不连续，所以需要在前一个的结果上“继承”。对于双序列来说，就不能只考虑一个，要同时考虑两个 text 都缩一个的情况，即 dp[i-1][j]和 dp[i][j-1]的较大值

![](https://img-blog.csdnimg.cn/20210204115139616.jpg)

> 为什么要取 dp[i-1][j]和 dp[i][j-1]？
> 实际上这里最原始的做法应该是从头开始取，比如 abcde 和 ace 这两个串，到'abc'和'ac'匹配的位置时，这是发现'ac'的下一项'e'和'abc'的下一项'd'不一样，这时就不能直接+1；那应该怎么做呢？就是从头开始找一个之前已经匹配上的最长的。
> 即查找 'abc'和'a'，'abc'和'ac'，等等，固定一个字符串然后另一个字符串从头开始找一个最长的匹配结果作为当前值。
> 但是这里不需要，因为这个 dp 数组本来就是自增的，dp[i-1][j]就代表了前面所有项里最长的匹配的长度，没必要再去一个一个比较
> 这一点也适用于其他问题，包括一维数组的一些序列问题，即，什么时候 dp 的确定需要遍历前面所有项，什么时候只需要前一项就够。其实关键就是搞清楚 dp 的定义，如果 dp 是一个自增的数组，每一项都是前面所有项的最大值，那就不需要遍历，比如打家劫舍 I；反之如果 dp 不确定，每一项都并非自增，那就可能需要向前遍历所有项找最大值。

整体结构如下：

![](https://img-blog.csdnimg.cn/20210210150215918.jpg)

可以看到，对于这种双数组的动态规划，如果理解不来就可以画表来看看。通常子序列长度问题都要考虑两边分别“缩一个”然后继承。

```js
/**
 * @param {string} text1
 * @param {string} text2
 * @return {number}
 */
var longestCommonSubsequence = function (text1, text2) {
  const [len1, len2] = [text1.length, text2.length];
  // dp[i][j]表示text1[i-1]和text[j-1]之间的最长公共子序列长度
  const dp = Array.from(new Array(len1 + 1), () => new Array(len2 + 1).fill(0));

  for (let i = 1; i < dp.length; i++) {
    for (let j = 1; j < dp[0].length; j++) {
      if (text1[i - 1] === text2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1] + 1;
      } else {
        dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
      }
    }
  }
  return dp[dp.length - 1][dp[0].length - 1];
};
```

## 判断子序列

> 给定字符串 s 和 t ，判断 s 是否为 t 的子序列。
> 字符串的一个子序列是原始字符串删除一些（也可以不删除）字符而不改变剩余字符相对位置形成的新字符串。（例如，"ace"是"abcde"的一个子序列，而"aec"不是）。
> 示例 1： 输入：s = "abc", t = "ahbgdc" 输出：true

这道题用双指针就能很快解决，但是用动态规划也可以，并且可以引出下一道题的类似思路。

用动态规划的话思路和*最长公共子数组*这道题类似，同样取二维数组 `dp[i][j]`，表示以下标`i - 1`结尾的字符串 s 和下标`j - 1`结尾的字符串 t 之间的子序列长度。

那么就会有两种情况：

1. 如果`s[i-1]===t[j-1]`，那么子序列长度加 1，即`dp[i][j] = dp[i - 1][j - 1] + 1`；（即下标 i-2 和 j-2 的串的子序列长度+1）
2. 如果`s[i-1]!==t[j-1]`，那么子序列长度不变，即`dp[i][j] = dp[i][j - 1]`。注意这里相当于 s 串不变，但 t 串向前走了，应该删去当前元素，所以是下标 i-1 的串和下标 j-2 的串子序列长度。

因此可以得到方程：

```js
// s[i-1]===t[j-1]
dp[i][j] = dp[i - 1][j - 1] + 1;

// s[i-1]!==t[j-1]
dp[i][j] = dp[i][j - 1];
```

最终结果只需要返回最右下角的元素即可。

代码：

```js
const isSubsequence = (s, t) => {
  // s、t的长度
  const [m, n] = [s.length, t.length];
  // dp全初始化为0
  const dp = new Array(m + 1).fill(0).map((x) => new Array(n + 1).fill(0));
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      // 更新dp[i][j]，两种情况
      if (s[i - 1] === t[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1] + 1;
      } else {
        dp[i][j] = dp[i][j - 1];
      }
    }
  }
  // 遍历结束，判断dp右下角的数是否等于s的长度
  return dp[m][n] === m ? true : false;
};
```

> 对于这种两个序列判断公共的问题，通常的解法都是利用二维数组；
> 而递推关系一般是两种情况得出的，可以类比双指针解法

## 不同子序列

> 给定一个字符串 s 和一个字符串 t ，计算在 s 的子序列中 t 出现的个数。
> 字符串的一个 子序列 是指，通过删除一些（也可以不删除）字符且不干扰剩余字符相对位置所组成的新字符串。（例如，"ACE" 是 "ABCDE" 的一个子序列，而 "AEC" 不是）

这个题和上面的题思路大致不变，因为都是在找公共子序列；
但是这个题是让你找到 s 有几种方法删除元素可以得到 t

同样设 `dp[i][j]`为 对于 i-1 结尾的 s 和 j-1 结尾的 t，t 在 s 中出现的个数。

那么会有同样的两种情况：

1. 如果`s[i-1]===t[j-1]`，此时`dp[i][j] = dp[i - 1][j - 1] + dp[i-1][j]`。这个的关键在于，后者表示 s 串中除去当前元素的判断结果，是因为对于如果已经匹配上的两个串，**s 串向后走不管多少位都不会减少之前 t 串已经出现的次数**。因此不应该是+1，而是加上之前匹配的次数。

2. 如果`s[i-1]!==t[j-1]`，`dp[i][j] = dp[i-1][j]`，和上面原因一样，只有之前匹配的次数结果。

得到方程为：

```js
dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j];

dp[i][j] = dp[i - 1][j];
```

注意初始化：

- dp[i][0]表示从以 i 结尾的串 s 中找到空字串。那么选取方法就只有一种，即一个都不选，或者说是全部删除
- dp[0][i]表示从空串中找到非空串，显然不可能，都为 0
- dp[0][0]表示空串找空串，为 1

代码：

```js
/**
 * @param {string} s
 * @param {string} t
 * @return {number}
 */
var numDistinct = function (s, t) {
  let dp = Array.from(Array(s.length + 1), () => Array(t.length + 1).fill(0));

  for (let i = 0; i <= s.length; i++) {
    dp[i][0] = 1;
  }

  for (let i = 1; i <= s.length; i++) {
    for (let j = 1; j <= t.length; j++) {
      if (s[i - 1] === t[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j];
      } else {
        dp[i][j] = dp[i - 1][j];
      }
    }
  }

  return dp[s.length][t.length];
};
```

## 两个字符串的删除操作

> 给定两个单词 word1 和 word2，找到使得 word1 和 word2 相同所需的最小步数，每步可以删除任意一个字符串中的一个字符。
>
> 示例：
>
> 输入: "sea", "eat"
> 输出: 2
> 解释: 第一步将"sea"变为"ea"，第二步将"eat"变为"ea"

这道题其实是编辑距离问题的一个情况，即删除情况。

这道题有两个思路：

1. 设`dp[i][j]`为以 i-1 为结尾的 s 和以 j-1 为结尾的 t，两个串要相等需要删除的字符数量。
2. 计算两个串的最长公共子序列长度，然后任意一个串的长度减去公共子序列的长度就可以

以第一个思路为例，还是比较`s[i-1]`和`t[i-1]`

- 如果相等，那么说明不用删除这个位置，那删除的数量就继承前面的一个的`dp[i-1][j-1]`
- 如果不等，那么有三种情况：
  - 删除`s[i-1]`，然后步数+1，即`dp[i-1][j] + 1`
  - 删除`t[j-1]`，同理
  - 两个都删，这时相当于两步，即`dp[i-1][j-1] + 2`
    然后取这三种情况的最小值即可

注意 dp 的大小和遍历上限：
由于元素可能被完全删除，即删成空串，因此 dp 的长宽应该比单词长度大一个单位，第一行表示 s 的空串和 t 的比较，第一列表示 t 的空串和 s 的比较。

![](https://pic.imgdb.cn/item/6412d64debf10e5d53d26df0.jpg)

然后是初始化。这个需要初始化第一行和第一列，

- `dp[i][0]`: 表示字符串 s 需要删除多少个字符，才能到达空串（即以索引-1 为结尾的 t 串，就是空串），即有多少删多少
- `dp[0][j]`：同理。t 需要删除所有，才能到达空串（以-1 为结尾的 s 串）

代码如下：

```js
/**
 * @param {string} word1
 * @param {string} word2
 * @return {number}
 */
var minDistance = function (word1, word2) {
  const dp = Array.from(new Array(word1.length + 1), () =>
    new Array(word2.length + 1).fill(0)
  );
  for (let i = 0; i <= word1.length; i++) dp[i][0] = i;
  for (let j = 0; j <= word2.length; j++) dp[0][j] = j;
  for (let i = 1; i <= word1.length; i++) {
    for (let j = 1; j <= word2.length; j++) {
      if (word1[i - 1] === word2[j - 1]) dp[i][j] = dp[i - 1][j - 1];
      else
        dp[i][j] = Math.min(
          dp[i - 1][j] + 1,
          dp[i][j - 1] + 1,
          dp[i - 1][j - 1] + 2
        );
    }
  }
  return dp[dp.length - 1][dp[0].length - 1];
};
```

## 编辑距离

> 给你两个单词 word1 和 word2，请你计算出将 word1 转换成 word2 所使用的最少操作数 。
> 你可以对一个单词进行如下三种操作：
> 插入一个字符
> 删除一个字符
> 替换一个字符
>
> 示例 1：
> 输入：word1 = "horse", word2 = "ros"
> 输出：3
> 解释： horse -> rorse (将 'h' 替换为 'r') rorse -> rose (删除 'r') rose -> ros (删除 'e')

上面那道题其实是编辑距离的一种情况，即删除情况。现在新增了两种情况，也就是在上面那道题的基础上，增加插入字符、替换字符的操作。

- 插入字符：不用新增判断，因为插入字符和删除字符相当于是一样的。

word2 添加一个元素，相当于 word1 删除一个元素
例如 word1 = "ad" ，word2 = "a"，word1 删除元素'd' 和 word2 添加一个元素'd'，变成 word1="a", word2="ad"，最终的操作数是一样的。

- 删除字符：上面那道题的三种情况
- 替换字符：也是按照替换 word1、替换 word2 的思路。相当于是在之前匹配的基础上，把 word1 的位置上的字符替换成 word2 上的，或者反过来也行。`dp[i][j] = dp[i-1][j-1] + 1`

因此代码就是在上面那道题基础上增加一个替换字符的情况：

```js
var minDistance = function (word1, word2) {
  let dp = Array.from(Array(word1.length + 1), () =>
    Array(word2.length + 1).fill(0)
  );

  for (let i = 1; i <= word1.length; i++) {
    dp[i][0] = i;
  }

  for (let j = 1; j <= word2.length; j++) {
    dp[0][j] = j;
  }

  for (let i = 1; i <= word1.length; i++) {
    for (let j = 1; j <= word2.length; j++) {
      if (word1[i - 1] === word2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1];
      } else {
        dp[i][j] = Math.min(
          dp[i - 1][j] + 1,
          dp[i][j - 1] + 1,
          dp[i - 1][j - 1] + 1
        );
      }
    }
  }

  return dp[word1.length][word2.length];
};
```

## 回文子串数量

> 给定一个字符串，你的任务是计算这个字符串中有多少个回文子串。
> 具有不同开始位置或结束位置的子串，即使是由相同的字符组成，也会被视作不同的子串。
> 示例 1：
> 输入："abc" 输出：3 解释：三个回文子串: "a", "b", "c"

回文子串类型的动态规划数组 dp 一般都是这样设置的：
设 dp[i][j]表示字符串 i 和 j 之间的串**是不是回文串**，如果是就是 ture，不是就是 false。注意这里的值是布尔值，算是一个特例。（回文子串相关的都可以这么取）
因此主要判断两端的字符是不是相等，因此有几种情况：

1. s[i] === s[j]，说明两端字符相等

- `i === j`，即同一个字符，为 true
- `|i - j| === 1`，说明是两个相邻的字符，即回文串的中心字符是偶数个，这种当然也是 true
- 其他情况，即 i 和 j 之间还包含其他字符串。因此需要判断两者之间的部分是不是，即判断`dp[i + 1][j - 1]`是否为 true

2. s[i] !== s[j]，肯定是 false

由此可以得到公式：

```js
if (s[i] === s[j]) {
  if (i === j || Math.abs(i - j) <= 1) dp[i][j] = true;
  else dp[i][j] = dp[i + 1][j - 1];
} else dp[i][j] = false;
```

这里注意因为 `dp[i][j]`是从 `dp[i + 1][j - 1]`推出来的，因此一定要从下向上、从左向右遍历。
即从 `dp[s.length - 1][0]`开始，i 递减。从字符串角度来说，就是从最右边的元素作为中间元素，一直到最左边元素为止，相当于每个字符都作为中心统计了一次回文子串数量。
回文子串的题基本都是这么遍历的，即 i 从第一列最后一项开始到第一项；初始化也只需要全部初始化一次即可。

并且对于回文子串问题，模拟出来的矩阵只有右上角才是有意义的，即 j >= i，右指针在左指针的右边才有意义。
![](https://img-blog.csdnimg.cn/20210121171059951.jpg)
上图中，对角线表示同一个字符，也就是 i=j 的情况；右上角都是 j>i，表示 i 和 j 之间的部分是不是回文串。

代码：

```js
/**
 * @param {string} s
 * @return {number}
 */
var countSubstrings = function (s) {
  const dp = Array.from(new Array(s.length), () =>
    new Array(s.length).fill(false)
  );
  let cnt = 0;
  for (let i = s.length - 1; i >= 0; i--) {
    for (let j = i; j <= s.length; j++) {
      if (s[i] === s[j]) {
        if (i === j || j - i <= 1) dp[i][j] = true;
        else dp[i][j] = dp[i + 1][j - 1];
      }
      if (dp[i][j]) cnt++;
    }
  }
  return cnt;
};
```

---

当然这道题这么解其实和动态规划没什么关系了，更好的方法是用双指针。
思路就是以每个字母都作为中点试一下，从中点向两边扩散，记录以该字母为中点的回文串数量。因为是从一个中心扩散，所以双指针同时左右移动，遇到所指相同字母就是回文串。
另外有个小问题是回文串的中心可能是一个或两个字母，因此最好的方式是分开算。

```js
var countSubstrings = function (s) {
  let res = 0;
  for (let i = 0; i < s.length; i++) {
    res += getCount(i, i);
    res += getCount(i, i + 1);
  }
  return res;

  function getCount(i, j) {
    let count = 0;
    while (i >= 0 && j < s.length) {
      if (s[i--] === s[j++]) count++;
      else break;
    }
    return count;
  }
};
```

## 最长回文子串

> 给你一个字符串 s，找到 s 中最长的回文子串。

这道题的思路其实和找到回文子串数量的方式差不多，先判断是否存在回文子串，再在每一次遍历时更新最长字串即可。

```js
/**
 * @param {string} s
 * @return {string}
 */
var longestPalindrome = function (s) {
  const dp = [];
  let res = "";
  let max = -Infinity;
  for (let i = s.length; i >= 0; i--) {
    dp[i] = [];
    for (let j = i; j < s.length; j++) {
      if (s[i] === s[j]) {
        if (i === j || Math.abs(i - j) === 1) dp[i][j] = true;
        else dp[i][j] = dp[i + 1][j - 1];
      } else {
        dp[i][j] = false;
      }
      if (dp[i][j] && j - i + 1 > max) {
        max = j - i + 1;
        res = s.slice(i, j + 1);
      }
    }
  }
  return res;
};
```

---

当然这道题本质上没有这么麻烦，不用动态规划，直接用双指针也能解决，并且耗时更短，更有效

思路其实和双指针近似。我们考虑并计算每个位置上，以该位置为中心、向两边扩散的最长回文串长度，再遍历每个位置，求得最长长度即可。

因此求任一个位置的最长回文串长度时，就需要考虑两个指针的起始位置。如果相同就是奇数长度回文串，如果不同且相邻就是偶数长度。

```js
function palindromeLen(str, left, right) {
  let len = 0;
  while (left >= 0 && right < str.length && str[left] === str[right]) {
    len++;
  }
  return len;
}
```

对于整个字符串的任意位置 i，要考虑`left=right=i`和`left=i,right=i+1`两种情况下的值。

> 注意这道题要求的是具体的回文串，而不是长度

```js
/**
 * @param {string} s
 * @return {string}
 */
var longestPalindrome = function (s) {
  function palindromeLen(str, left, right) {
    while (left >= 0 && right < str.length && str[left] === str[right]) {
      left--;
      right++;
    }
    // 此时left和right都是超出了一个，也就是恰好不相同的位置，向内各走一个就是回文串
    // slice 留前不留后，因此right就是向内走一个位置，但left不是，还需要+1
    return str.slice(left + 1, right);
  }
  let str = "";
  for (let i = 0; i < s.length; i++) {
    const s1 = palindromeLen(s, i, i);
    const s2 = palindromeLen(s, i, i + 1);
    str = str.length > s1.length ? str : s1;
    str = str.length > s2.length ? str : s2;
  }
  return str;
};
```

## 最长回文子序列

https://leetcode.cn/problems/longest-palindromic-subsequence/

> 给定一个字符串 s ，找到其中最长的回文子序列，并返回该序列的长度。可以假设 s 的最大长度为 1000 。
> 示例 1: 输入: "bbbab" 输出: 4 一个可能的最长回文子序列为 "bbbb"。
> 示例 2: 输入:"cbbd" 输出: 2 一个可能的最长回文子序列为 "bb"。

思路和上面的回文子串差不多，分开判断`s[i] === s[j]`;

- 如果相等，`dp[i][j] = dp[i + 1][j - 1] + 2;`，这里可以不考虑 i、j 重合的情况
- 如果不相等，不能直接缩小范围，要测试一下只加上左边和只加上右边能不能构成回文；

注意这里和上一道题不一样，因为上一道题只用考虑中间的是不是回文串，是就记录，不是就不记录，如果单独考虑反而会重复。

> 比如字符串"bbbab"，当 i 指向第一个 b，j 指向 a 时，两者不相等，如果缩小右边的一个位置就有"bbb"；但是其实本来 j 就会走过这一串，这样统计反而重复了

而这里需要的是前一个的最长回文子序列长度，除了复用两边都减去的情况，还必须要考虑两边各减一个的情况

> 比如字符串"bbbab"，当 i 指向第一个 b，j 指向 a 时，两者不相等，同理缩小左右分别得到"bbb"和"bba"，因为是统计长度，因此不存在重复的可能，dp[i][j]一定取得是范围内设法得到的的最大值。

模拟：
![](https://img-blog.csdnimg.cn/20210127151452993.jpg)

```js
/**
 * @param {string} s
 * @return {number}
 */
var longestPalindromeSubseq = function (s) {
  const dp = Array.from(new Array(s.length), () => new Array(s.length).fill(0));
  for (let i = s.length; i >= 0; i--) {
    for (let j = i; j < s.length; j++) {
      if (s[i] === s[j]) {
        if (i === j) dp[i][j] = 1;
        else if (Math.abs(i - j) <= 1) dp[i][j] = 2;
        else dp[i][j] = dp[i + 1][j - 1] + 2;
      } else {
        dp[i][j] = Math.max(dp[i][j - 1], dp[i + 1][j]);
      }
    }
  }
  console.log(dp);
  return dp[0][dp[0].length - 1];
};
```

## 最大正方形

https://leetcode.cn/problems/maximal-square/

思路：设 dp[i][j]表示以(i,j)为右下角的正方形大小。那么显然 dp[i][j]可以从其左、上、斜上三个方向推出来。

- 首先三个位置必须都是 1，否则当前 dp[i][j]最多只能是 1
- 满足上一条之后，dp[i][j]应该是三个中 `(根号下最小值+1)^2`。比如三个值分别是 4、9、16，那么最多只能和 4 所在的那个位置凑成一个 3\*3 的正方形。

```js
/**
 * @param {character[][]} matrix
 * @return {number}
 */
var maximalSquare = function (matrix) {
  const [m, n] = [matrix.length, matrix[0].length];
  const dp = Array.from(new Array(m), () => new Array(n).fill(0));
  for (let i = 0; i < m; i++) {
    for (let j = 0; j < n; j++) {
      dp[0][j] = +matrix[0][j];
      dp[i][0] = +matrix[i][0];
    }
  }
  let max = 0;
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (matrix[i] && matrix[i][j] !== "0") {
        if (dp[i - 1][j] && dp[i][j - 1] && dp[i - 1][j - 1]) {
          dp[i][j] =
            (Math.sqrt(
              Math.min(+dp[i - 1][j], +dp[i][j - 1], +dp[i - 1][j - 1])
            ) +
              1) **
            2;
        } else dp[i][j] = 1;
      }
      max = Math.max(max, dp[i - 1][j - 1]);
    }
  }
  return max;
};
```

# 哈希表

## 欢乐数

https://leetcode.cn/problems/happy-number/

> 编写一个算法来判断一个数 n 是不是快乐数。
> 「快乐数」  定义为：
>
> 对于一个正整数，每一次将该数替换为它每个位置上的数字的平方和。
> 然后重复这个过程直到这个数变为 1，也可能是 无限循环 但始终变不到 1。
> 如果这个过程 结果为  1，那么这个数就是快乐数。

这道题的关键是，不能化为 1 的数会一直重复其计算结果，所以如果发现重复，就说明不是欢乐数。

```js
/**
 * @param {number} n
 * @return {boolean}
 */
var isHappy = function (n) {
  const nums = [0, 1, 4, 9, 16, 25, 36, 49, 64, 81];
  const set = new Set();
  function getHappy(num) {
    let sum = 0;
    while (num > 0) {
      sum += nums[num % 10];
      num = Math.floor(num / 10);
    }
    return sum;
  }
  let init = getHappy(n);
  while (init !== 1) {
    if (set.has(init)) return false;
    else set.add(init);
    init = getHappy(init);
  }
  return true;
};
```

## 四数之和 II

> 给你四个整数数组 nums1、nums2、nums3 和 nums4 ，数组长度都是 n ，请你计算有多少个元组 (i, j, k, l) 能满足：
> 0 <= i, j, k, l < n
> nums1[i] + nums2[j] + nums3[k] + nums4[l] == 0

https://leetcode.cn/problems/4sum-ii/

这道题可以这样想：

1. 首先，先考虑两个数组的情况。怎么在两个数组中计算相加为 0 的数的个数？其中一个方法是，用一个 map 存储一个数组的数字出现个数（键为数字，值为出现次数），然后再遍历另一个数组，检查其中是否有当前数字的相反数。如果有，就让总个数加上这个数字的值（即出现次数）。
2. 那么对于四个数组也是一样的：我们把前两个看作一个数组，计算他们每一项的和，再计算出上面说的这个 map；然后遍历后两个数组，后两个的和如果有 map 中的相反数，就取出来统计即可。

```js
const map = new Map();
const len = nums1.length;
let cnt = 0;

for (let i = 0; i < len; i++) {
  for (let j = 0; j < len; j++) {
    let sum = nums1[i] + nums2[j];
    if (!map.has(sum)) map.set(sum, 1);
    else map.set(sum, map.get(sum) + 1);
  }
}

for (let i = 0; i < len; i++) {
  for (let j = 0; j < len; j++) {
    const sum = nums3[i] + nums4[j];
    if (map.has(-sum) && map.get(-sum) > 0) {
      cnt += map.get(sum);
      map.set(sum, 0);
    }
  }
}

return cnt;
```

# 贪心

贪心的核心思维是“局部最优推整体最优”，和动态规划的核心思维一致。因此大多数贪心能解的题，动态规划也能解。
但是贪心还有一个思维，就是“最多”，或者“最大”这种思维。这个是动态规划不具备的。

比如说我们从一堆钱中限制取的次数，求能去到的最多的钱数，那肯定是每次都取这堆钱中最大面额的，才能在限定次数中取得总值最大的。

## 最大子序和

> 给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。
> 示例: 输入: [-2,1,-3,4,-1,2,1,-5,4] 输出: 6 解释: 连续子数组 [4,-1,2,1] 的和最大，为 6

这道题用动态规划能解，但是这里要考虑的是贪心解法。

贪心贪的是什么呢？实际上就是保证和始终要增大；如果一组连续的和为负数，就应该立即抛弃，从下一个数开始。
具体来说，就是一个数字`nums[i]+sum<0`，就直接记录 sum 并令 sum=0 然后继续从下一个遍历，否则持续累加 sum

```js
var maxSubArray = function (nums) {
  let result = -Infinity;
  let count = 0;
  for (let i = 0; i < nums.length; i++) {
    count += nums[i];
    if (count > result) {
      result = count;
    }
    if (count < 0) {
      count = 0;
    }
  }
  return result;
};
```

## K 次取反后最大化的数组和

> 给定一个整数数组 A，我们只能用以下方法修改该数组：我们选择某个索引 i 并将 A[i] 替换为 -A[i]，然后总共重复这个过程 K 次。（我们可以多次选择同一个索引 i。）
>
> 以这种方式修改数组后，返回数组可能的最大和。
>
> 示例 1：
>
> 输入：A = [4,2,3], K = 1
> 输出：5
> 解释：选择索引 (1,) ，然后 A 变为 [4,-2,3]。

这道题的思路就是：

- 如果有负数，那就优先把绝对值最大的负数取反
- 如果没有负数，就把绝对值最小的正数取反

具体实现，其实就是给数组排序，然后每次把第一个元素取反即可。因为每次排序之后第一个元素一定是最应该取反的（要么是最小的负数，要么是最小的正数）

```js
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number}
 */
var largestSumAfterKNegations = function (nums, k) {
  for (let i = 0; i < k; i++) {
    nums.sort((a, b) => a - b);
    nums[0] = -nums[0];
  }
  return nums.reduce((pre, cur) => pre + cur);
};
```

## 无重叠区间

https://leetcode.cn/problems/non-overlapping-intervals/

> 给定一个区间的集合  intervals ，其中 intervals[i] = [starti, endi] 。返回 需要移除区间的最小数量，使剩余区间互不重叠  。
> 示例 1:
> 输入: intervals = [[1,2],[2,3],[3,4],[1,3]]
> 输出: 1
> 解释: 移除 [1,3] 后，剩下的区间没有重叠。

这道题有一种解法是之前说过的，就是先按照 start 排序，再依次遍历，把每个 start < 前一个 end 的数组删掉，最后剩下的就是无重叠区间的个数。

这里提供另外一种解法，更像贪心的思路，即按照 end 排序，大概是这样：
![](https://labuladong.github.io/algo/images/interval/1.gif)

也就是说，每次删除的是 start 小于最小 end 的区间，逐个更新最小 end，最后就可以得到结果。

```js
var eraseOverlapIntervals = function (intervals) {
  if (intervals.length === 1) return 0;
  intervals.sort((a, b) => a[1] - b[1]); // 按照end排序
  let minEnd = intervals[0][1];
  let cnt = 0;
  for (let i = 1; i < intervals.length; i++) {
    const [start, end] = intervals[i];
    if (start < minEnd) cnt++; // 如果一个区间的start小于minEnd，就删掉这个区间
    else minEnd = end; // 更新minEnd
  }
  return cnt;
};
```

这个思路还可以用于解决`https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/`这道题，其实都是一个思路的问题。

## 不含 AAA 或 BBB 的字符串

https://leetcode.cn/problems/string-without-aaa-or-bbb/

> 给定两个整数 a  和 b ，返回   任意   字符串 s ，要求满足：
> s  的长度为 a + b，且正好包含 a  个 'a'  字母与  b  个 'b'  字母；
> 子串  'aaa'  没有出现在 s  中；
> 子串  'bbb' 没有出现在 s  中。
> 示例 1：
> 输入：a = 1, b = 2
> 输出："abb"
> 解释："abb", "bab" 和 "bba" 都是正确答案。

这个题有两种解法，一种是贪心，还有一个是取巧的解法。

先说贪心的解法。既然 a 和 b 可以最多有两个相连，那么就应该考虑选取 a 和 b 中个数更多的一个优先放入，如果放入连续两个在考虑放入其他的。即：

1. 循环放入 a 和 b，每次放入之前，先计算 a 和 b 哪个多
2. 如果总字符长度大于 2，并且前两个字符相同，那就放另一个；

- 其他情况下（前两个字符不同，或长度小于 2）就放入第一步计算出的更多的那个。

代码如下：

```js
var strWithout3a3b = function (a, b) {
  let res = "";
  let index = 0;
  while (a > 0 || b > 0) {
    let aorb = a > b; // 计算a多还是b多
    if (index >= 2 && res[index - 1] === res[index - 2]) {
      // 前两个重复就放另一个
      if (res[index - 1] === "b") {
        a--;
        res += "a";
      } else if (res[index - 1] === "a") {
        b--;
        res += "b";
      }
    } else {
      // 其他情况就放更多的那个
      if (aorb) {
        res += "a";
        a--;
      } else {
        res += "b";
        b--;
      }
    }
    index++;
  }
  return res;
};
```

---

第二种方法比较取巧。每次放入的不是'a'或'b'，而是'ab'、'aab'和'abb'。

- 如果 a 和 b 一样多，就放入 ab
- 如果 a 比 b 多，就放入 aab
- 如果 b 比 a 多，就放入 abb

最后直到放完为止

```js
var strWithout3a3b = function (a, b) {
  let res = "";
  while (a > 0 || b > 0) {
    if (a > b) {
      res += "aab";
      a -= 2;
      b--;
    } else if (a < b) {
      res += "abb";
      b -= 2;
      a--;
    } else {
      res += "ab";
      a--;
      b--;
    }
  }
  return res;
};
```

# 分治法

## 为运算表达式设计优先级

> 给你一个由数字和运算符组成的字符串  expression ，按不同优先级组合数字和运算符，计算并返回所有可能组合的结果。你可以 按任意顺序 返回答案。
> 生成的测试用例满足其对应输出值符合 32 位整数范围，不同结果的数量不超过 104 。
> 示例 1：
> 输入：expression = "2-1-1"
> 输出：[0,2]
> 解释：
> ((2-1)-1) = 0
> (2-(1-1)) = 2

这道题其实是一个加括号的问题。
比如说，`2-1-1`可能由几种加括号的方法，根据括号的层数：

- 一层括号，也就是没有嵌套，可以是`(2-1)-1`或者`2-(1-1)`
- 两层括号，`((2-1)-1)`或`(2-(1-1))`

后面还有多层的括号。
括号的层数可能有很多，但是我们不需要考虑多个括号的情况，只需要考虑一层括号的划分，然后把划分出来的括号内部再进行划分，最后直到没有运算符只有数字为止。这就是分治的思想。

一层括号的划分可以根据运算符，每个运算符都可以作为一个分界线，两边作为不同的表达式，分别计算这两个表达式，再求值即可。这个思路有点像二叉树的处理，递归计算两个表达式的值，返回值，然后只考虑本层的计算结果即可。
因此整体思路如下：

1. 遍历字符串，每个运算符都进行一次分割成左表达式和右表达式
2. 递归计算左右表达式的值。这里本来的函数就是接收一个表达式并计算可能的结果，那么递归得到的左右表达式也是一个结果数组，表示所有可能的值
3. 对两个数组的值依次遍历，再根据本层的运算符计算得到值，放入 res 数组。

> 注意这里应该是交叉计算。
> 举个例子 `2*3-4*5`
> 如果分割成 `2 * (3-4*5)`这种形式，那么右半部分应该有两个计算结果[-17, -5]
> 所以这里实际上是两种情况，`2*(3-(4*5))`和`2*((3-4)*5))`，因此要交叉计算[2]和[-17, -5]，得到所有情况

4. 返回 res 数组

递归边界是表达式没有找到运算符，说明只是一个数字，那么直接返回包含这个数字的 res 数组就行。

```js
/**
 * @param {string} expression
 * @return {number[]}
 */
var diffWaysToCompute = function (expression) {
  const res = []; // 每一层都不一样，每一层都是本层表达式的可能计算结果
  for (let i = 0; i < expression.length; i++) {
    const char = expression[i];
    if (char === "+" || char === "*" || char === "-") {
      // 根据运算符分割左右表达式
      /****** 分 ******/
      const leftExp = expression.split("").splice(0, i).join("");
      const rightExp = expression
        .split("")
        .splice(i + 1)
        .join("");
      /* 这里我们不用考虑下一层是怎么计算表达式的，只需要把本层处理好，即分割和合并两个步骤 */
      const left = diffWaysToCompute(leftExp); // 递归计算左表达式，返回是一个res数组，即表达式的可能计算结果
      const right = diffWaysToCompute(rightExp); // 同理
      /****** 治 ******/
      for (let a of left) {
        // 因为left和right都是结果数组，可以按照运算符拼接
        for (let b of right) {
          if (char === "+") res.push(a + b);
          else if (char === "-") res.push(a - b);
          else if (char === "*") res.push(a * b);
        }
      }
    }
  }
  // 如果只有一个数字，直接返回
  if (res.length === 0) res.push(+expression);
  return res;
};
```

# BFS

BFS 的其实就是树的层序遍历，在树那一章也有讲到过。
BFS 的基本框架：

```js
function BFS() {
  const queue = [];
  queue.push(第一项);
  while (queue.length) {
    const len = queue.length;
    for (let i = 0; i < len; i++) {
      const top = queue.shift();
      处理top相关的操作;
      queue.push(top的子节点);
    }
    level++; // 层数++
  }
}
```

这样的 BFS 有明显的“层”，每次弹出是以层为单位的，一层一层弹出。

还有一种是图遍历中的 BFS，每次以单个节点为单位，会对队列本次弹出的部分做拼接再放回队列。这种广泛用于图的遍历、迷宫路径等问题，详细算法参考下面的图的遍历的章节。

---

BFS 解决问题求出的一定是**最短路径**，也就是到达目标的最短方法。
因此，如果看到题目要求是最短，并且是组合、排列之类的可以用回溯解决的，就可以考虑一下 BFS。

BFS 的关键在于找出 top 的子节点，也就是理顺父子节点的关系。
在解决大多数组合和排列问题时，DFS 通常是一个一个的拼接，但是 BFS 一般是先得到一个序列，然后依次替换每个值，再把新值做类似的替换。因为 BFS 实际上是在横向遍历一棵树，因此通常是一开始就确定了。
BFS 的其他类似用途：走迷宫，迷宫最短路径，替换单词变成另外一个单词等，本质就是让你在一幅「图」中找到从起点 start 到终点 target 的最近距离，相当于是在“数次数”

## 打开转盘锁

> 你有一个带有四个圆形拨轮的转盘锁。每个拨轮都有 10 个数字： '0', '1', '2', '3', '4', '5', '6', '7', '8', '9' 。每个拨轮可以自由旋转：例如把 '9' 变为  '0'，'0' 变为 '9' 。每次旋转都只能旋转一个拨轮的一位数字。
> 锁的初始数字为 '0000' ，一个代表四个拨轮的数字的字符串。
> 列表 deadends 包含了一组死亡数字，一旦拨轮的数字和列表里的任何一个元素相同，这个锁将会被永久锁定，无法再被旋转。
> 字符串 target 代表可以解锁的数字，你需要给出解锁需要的最小旋转次数，如果无论如何不能解锁，返回 -1 。
> 示例 1:
> 输入：deadends = ["0201","0101","0102","1212","2002"], target = "0202"
> 输出：6
> 解释：
> 可能的移动序列为 "0000" -> "1000" -> "1100" -> "1200" -> "1201" -> "1202" -> "0202"。
> 注意 "0000" -> "0001" -> "0002" -> "0102" -> "0202" 这样的序列是不能解锁的，
> 因为当拨动到 "0102" 时这个锁就会被锁定。

这道题其实可以先考虑怎么遍历所有组合。
有 DFS 和 BFS 两种方式：

- DFS 法：迭代 0-9，组合到 4 个数字就记录；并且每次新的数都应该是上一个连续的
- BFS 法：直接对'0000'计算，一个'0000'拨动之后可能有 8 种情况，即 "1000", "9000", "0100", "0900"... 共 8 种密码。按照这个分支进行 BFS。

因为这道题要考虑最短距离，也就是最少拨动次数，因此应该采用 BFS。

思路：

1. 首先将'0000'放入队列，然后出队，把 0000 的子元素放入；这里考虑的子元素实际上就是**拨动一次**的可能情况，也就是`"1000", "9000", "0100", "0900"...`这样的 8 种情况。
2. 按照 BFS 的方法，依次放入并处理重复的情况（比如'1000'不能再走回'0000'，会死循环）；如果遇到 deadends 中的元素就跳过本次循环，相当于这个树枝下面的路就堵死了。

```js
/**
 * @param {string[]} deadends
 * @param {string} target
 * @return {number}
 */
var openLock = function (deadends, target) {
  const minus = (str, idx) => {
    // 选一位+1
    const tmp = str.split("");
    if (tmp[idx] === "0") {
      tmp[idx] = "9";
    } else {
      tmp[idx] = (+tmp[idx] - 1).toString();
    }
    return tmp.join("");
  };

  const add = (str, idx) => {
    const tmp = str.split("");
    if (tmp[idx] === "9") {
      tmp[idx] = "0";
    } else {
      tmp[idx] = (+tmp[idx] + 1).toString();
    }
    return tmp.join("");
  };

  const queue = [];
  let times = 0;
  const visited = new Set(deadends); // 一开始就把deadends放入visited，就不用后续过滤了
  queue.push("0000");
  if (deadends.includes("0000")) return -1;
  while (queue.length) {
    const len = queue.length;
    for (let i = 0; i < len; i++) {
      const top = queue.shift();
      if (target === top) return times;
      for (let j = 0; j < top.length; j++) {
        // 遍历字符串的每一位，分别+1或-1
        const pre = minus(top, j);
        if (!visited.has(pre)) {
          visited.add(pre);
          queue.push(pre);
        }
        const next = add(top, j);
        if (!visited.has(next)) {
          visited.add(next);
          queue.push(next);
        }
      }
    }
    times++;
  }
  return -1;
};
```

## 到家的最少跳跃次数

> 有一只跳蚤的家在数轴上的位置  x  处。请你帮助它从位置  0  出发，到达它的家。
> 跳蚤跳跃的规则如下：
> 它可以 往前 跳恰好 a  个位置（即往右跳）。
> 它可以 往后   跳恰好 b  个位置（即往左跳）。
> 它不能 连续 往后跳 2 次。
> 它不能跳到任何  forbidden  数组中的位置。
> 跳蚤可以往前跳 超过   它的家的位置，但是它 不能跳到负整数   的位置。
> 给你一个整数数组  forbidden ，其中  forbidden[i]  是跳蚤不能跳到的位置，同时给你整数  a， b  和  x ，请你返回跳蚤到家的最少跳跃次数。如果没有恰好到达 x  的可行方案，请你返回 -1 。
> 示例 1：
> 输入：forbidden = [14,4,18,1,15], a = 3, b = 15, x = 9
> 输出：3
> 解释：往前跳 3 次（0 -> 3 -> 6 -> 9），跳蚤就到家了。

这道题和上面那个转密码锁的很像，同样都是从同一个起点开始，经过一些步骤走到目标，中间包含一些不能走的位置，要求最少次数。
因此这道题也是 bfs，方法类似。

但是有一些细节：

1. 不能连续向后两次：因为是 bfs，所以不能直接确定上一次是向后还是向前，因此每一次向队列中放入的元素应该是一个对象，包含位置和上一次走的方向。如果上一次走的是向后，那这次就不行
2. 上限：不能一直向前跳，向前的位置应该有个上限。题解给出的上限是`Math.max(...forbidden) + a + b + x`，这个值是尽可能大的上限了。之前考虑过上限应该是`x+2b`（因为不能连续向后跳两次）但是不太对

代码：

```js
var minimumJumps = function (forbidden, a, b, x) {
  const queue = [{ pos: 0, dire: "right" }];
  let times = 0;
  const max = Math.max(...forbidden) + a + b + x;
  const visited = new Set(forbidden);
  while (queue.length) {
    const len = queue.length;
    for (let i = 0; i < len; i++) {
      const curr = queue.shift();
      if (visited.has(curr.pos) || curr.pos < 0 || curr.pos >= max) continue;
      visited.add(curr.pos);
      if (curr.pos === x) return times;
      if (curr.dire !== "left") queue.push({ pos: curr.pos - b, dire: "left" });
      else queue.push({ pos: curr.pos + a, dire: "right" });
    }
    times++;
  }
  return -1;
};
```

## 迷宫

### 迷宫 I

https://leetcode.cn/problems/the-maze/

> 由空地（用 0 表示）和墙（用 1 表示）组成的迷宫 maze 中有一个球。球可以途经空地向 上、下、左、右 四个方向滚动，且在遇到墙壁前不会停止滚动。当球停下时，可以选择向下一个方向滚动。
> 给你一个大小为 m x n 的迷宫 maze ，以及球的初始位置 start 和目的地 destination ，其中 start = [startrow, startcol] 且 destination = [destinationrow, destinationcol] 。请你判断球能否在目的地停下：如果可以，返回 true ；否则，返回 false 。

（具体的图可以看原题目）

这道题和普通的寻路不太一样，小球不能停在“中段”，必须只能停在一个特殊的位置，即“撞墙”。
并且题目只问了能不能到达，所以 bfs 和 dfs 都可以。这里采用 bfs。
和常规 bfs 最大的区别在于，每次向 queue 中放入的下一个坐标不是简单的 x+1/y-1 这种，而是基于当前的 x 和 y，分别向四个方向循环走到墙壁的位置。这样得出的四个方向的坐标，就一定是撞墙情况下的坐标。

代码如下：

```js
var hasPath = function (maze, start, destination) {
  const [m, n] = [maze[0].length, maze.length];
  const visited = Array.from(new Array(n), () => new Array(m).fill(false));
  const [desX, desY] = destination;
  const queue = [];
  queue.push(start);
  while (queue.length) {
    const curr = queue.shift();
    let [x, y] = curr;
    if (maze[x][y] === 1 || visited[x][y]) continue;
    if (x === desX && y === desY) return true;
    visited[x][y] = true;
    let topx = (bottomx = x);
    let lefty = (righty = y);
    while (topx >= 0 && maze[topx][y] !== 1) topx--; // 向上走到撞墙为止
    while (bottomx < n && maze[bottomx][y] !== 1) bottomx++; // 向下走到撞墙
    while (lefty >= 0 && maze[x][lefty] !== 1) lefty--; // 向左撞墙
    while (righty < m && maze[x][righty] !== 1) righty++; // 向右
    queue.push(
      [topx + 1, y], // 因为while循环会多走一次，所以每个坐标其实还要反向走一格
      [bottomx - 1, y],
      [x, lefty + 1],
      [x, righty - 1]
    );
  }
  return false;
};
```

如果是 dfs，其实也是这个思路。每次递归的 dfs 参数还是按照这个方式来的（即撞墙），其他的部分和常规 dfs 一样

### 迷宫 II

https://leetcode.cn/problems/the-maze-ii/

这道题和上面那个题的唯一区别，在于问你能到达的最短路径长度是多少。
同样是 bfs，但是方法完全不一样。因为这里有一个致命问题，即，撞墙次数最少（也就是 bfs 的层数）并不代表最终路径最短，即这张图：

![](https://pic1.imgdb.cn/item/6342c10116f2c2beb1f369e4.jpg)

所以这道题需要另辟蹊径。
按照题解的方法，设置一个 distances 数组。这是一个二维数组，`distance[i,j]`表示从起点到`[i,j]`的最短距离（最小步数）（**一定要始终记住这点**），初始化全为 infinity，只有起点的值为 0。

对于每次到达的位置，执行以下几步：

1. 从[x,y]向四个方向循环走，直到撞墙为止
2. 在上一步的循环中，记录一个 cnt 变量。每走一步 cnt 都会++，即撞墙时**cnt 表示本次移动走的步数**。走完之后的坐标记为[nextX,nextY]
3. 比较`cnt + distance[x,y]`和`distance[nextX,nextY]`，会有几种情况：
4. 之前没走到过[nextX,nextY]点，那么值就是 Infinity，显然小于，就替换 distance[nextX,nextY]为 cnt+distance[x,y]
5. 如果之前走到过，那么 distance[nextX,nextY]点表示从原点到该点的最小距离（目前的最小距离）。如果本次计算得到的`cnt+distance[x,y]`比这个距离小，那就更新，然后把[nextX,nextY]点放入队列，下一次从这个点走；反之就不更新，重新从起点向另外一个方向走

注意这个流程中是没有 visited 这种记录走过路径的数组的，而是依靠 distance 数组；如果某次移动时回头走到了之前的位置，那么`cnt + distance[x][y]` 必然会大于`distance[nextX][nextY]`（因为回头走肯定是会重复之前的路径，当前位置的 distance 值肯定都大于之前位置，加上 cnt 就更大了），因此就会自然而然地跳过，起到了防回头的效果。

代码如下：

```js
var shortestDistance = function (maze, start, destination) {
  const [m, n] = [maze[0].length, maze.length];

  const [desX, desY] = destination;
  const [startX, startY] = start;

  const distance = Array.from(new Array(n), () => new Array(m).fill(Infinity));
  distance[startX][startY] = 0;
  const dires = [
    [-1, 0],
    [1, 0],
    [0, -1],
    [0, 1],
  ]; // 四个方向

  const queue = [];
  queue.push(start);
  while (queue.length) {
    // 注意这里不再需要循环len了，因为是图的bfs
    const curr = queue.shift();
    const [x, y] = curr;
    for (const [direX, direY] of dires) {
      let cnt = 0;
      let nextX = x;
      let nextY = y;
      // 这个循环的方式和迷宫I的那种方式同理，但是只用写一个循环
      while (
        nextX >= 0 &&
        nextX < n &&
        nextY >= 0 &&
        nextY < m &&
        maze[nextX][nextY] !== 1
      ) {
        cnt++;
        nextX += direX;
        nextY += direY;
      }
      // while循环会多走一步，往回走一步
      cnt--;
      nextX -= direX;
      nextY -= direY;
      // 比较，这里上面说过了
      if (cnt + distance[x][y] < distance[nextX][nextY]) {
        distance[nextX][nextY] = cnt + distance[x][y];
        queue.push([nextX, nextY]);
      }
    }
  }
  // 如果最后走不到，那distance值一定就是infinity
  return distance[desX][desY] === Infinity ? -1 : distance[desX][desY];
};
```

# 图相关算法

## 课程表问题（有向图的成环判断）

https://leetcode.cn/problems/course-schedule/

> 你这个学期必须选修 numCourses 门课程，记为  0  到  numCourses - 1 。
> 在选修某些课程之前需要一些先修课程。 先修课程按数组  prerequisites 给出，其中  prerequisites[i] = [ai, bi] ，表示如果要学习课程  ai 则 必须 先学习课程   bi 。
> 例如，先修课程对  [0, 1] 表示：想要学习课程 0 ，你需要先完成课程 1 。
> 请你判断是否可能完成所有课程的学习？如果可以，返回 true ；否则，返回 false 。
> 输入：numCourses = 2, prerequisites = [[1,0]]
> 输出：true
> 解释：总共有 2 门课程。学习课程 1 之前，你需要完成课程 0 。这是可能的。

这道题其实就是在判断是否存在循环依赖的情况，如果存在就不能正常修完课程。
数组 prerequisites 给出的关系，实际上就是这样一个有向图：

![](https://pic.imgdb.cn/item/62dc0e8cf54cd3f937153d79.jpg)

我们需要做的，就是先构建这个图，然后判断这个图中有没有循环依赖，即有没有成环。

那么最简单的思路，就是通过 DFS 遍历整个图，如果有一个地方成环，就返回 false

当然首先还是要构建一个图，绝大多数情况下都采用邻接表的形式。设一个 map，**key 为节点，value 为依赖于这个节点的其他节点**，即该节点的后续节点。遍历的过程中通过`map[key]`就可以得到该节点连接的后续节点（因为是有向图，所以一定是后驱的节点）

```js
var canFinish = function (numCourses, prerequisites) {
  const map = new Map();
  for (let prerequisite of prerequisites) {
    const [cur, pre] = prerequisite; // cur表示当前课，pre表示当前课依赖的课
    if (!map.has(pre)) {
      map.set(pre, [cur]);
    } else {
      map.set(pre, [...map.get(pre), cur]);
    }
  }

  const visited = new Set(); // visited记录的是整个遍历过程中走过的节点，只要记录一次后面就不会删除，主要用来防止走走过的节点导致死循环
  const onPath = new Set(); // onPath记录的是本轮递归（某一个路径）走过的节点，走完后会回溯地删除，path才是判断成环的主要依据

  let hasCycle = false;
  function dfs(node) {
    if (onPath.has(node)) {
      hasCycle = true; // 如果本次路径path包含走过的节点，就说明成环了
      return;
    }
    // 注意这里的条件还有visited.has(node)
    if (node == undefined || hasCycle || visited.has(node)) return;

    visited.add(node);
    onPath.add(node);
    const neighbours = map.get(node); // neighbours是后继节点数组
    if (neighbours && neighbours.length) {
      for (let neighbour of neighbours) {
        dfs(neighbour);
      }
    }
    onPath.delete(node);
  }
  // 从每个节点开始都走一遍
  for (let i = 0; i < numCourses; i++) {
    dfs(i);
  }
  return !hasCycle;
};
```

第二种方法是 BFS，稍微麻烦一点，需要利用到有向图的特征，即入度和出度的概念。
对于每一个节点，依赖于它的课程可以被看作是由它指向外部的箭头，被它依赖的课程可以看作是指向它的箭头。一个课程如果入度不为 0，说明要修该课程就必须先修其他某个课，因此不能直接修这个课。同理，如果入度为 0 就可以立即修这门课。
一门课的入度可以减少，当它依赖的课被修完之后，这门课的入度就会-1，直到减为 0 就可以立即修这门课了。
整体来说就是这样一个思路：

- 构建一个图
- 让入度为 0 的课入列，它们是能直接选的课。
- 然后逐个出列，出列代表着课被选，需要减小相关课的入度。“相关课”就是这门课的后继节点，也就是从邻接表`map[node]`得到的
- 如果相关课的入度新变为 0，安排它入列、再出列……直到没有入度为 0 的课可入列。

另外，还需要一个数组记录每个节点的入度。每次增减入度都会对该数组操作，如果入度减为 0，就可以放入队列中。
每出队一门课，就相当于修了这门课，我们可以得到这门课的具体节点。而成环的课不会被修到。用一个 count 记录修课的数量，如果最终数量达到 numCourses，就说明可以修完。

```js
var canFinish = function (numCourses, prerequisites) {
  const map = new Map();
  let count = 0;
  const inDegrees = new Array(numCourses).fill(0);
  for (let prerequisite of prerequisites) {
    const [cur, pre] = prerequisite; // cur表示当前课，pre表示依赖的课
    inDegrees[cur]++;
    if (!map.has(pre)) {
      map.set(pre, [cur]);
    } else {
      map.set(pre, [...map.get(pre), cur]);
    }
  }
  // 队列中始终都是入度为0的节点，出队表示修了该门课
  const queue = [];
  // 先把所有入度为0的入队
  for (let i = 0; i < numCourses; i++) {
    if (inDegrees[i] === 0) queue.push(i);
  }

  while (queue.length) {
    const top = queue.shift();
    // 获取到该门课的后继课
    const lessons = map.get(top);
    count++; // 这里其实可以得到完整的修课顺序
    if (lessons && lessons.length) {
      for (let lesson of lessons) {
        // 只要遍历到某门课，就给这门课的入度-1，减为0就可以入队
        if (--inDegrees[lesson] === 0) queue.push(lesson);
      }
    }
  }
  return count === numCourses;
};
```

---

这道题的第二题，即课程表 II，就是在这个基础上，返回合适的修课顺序即可。这种把相互依赖的有向图转化为一个正确遍历顺序的算法叫做**拓扑排序**

下面就是第二题的代码，其实就是记录了路径而已

```js
var findOrder = function (numCourses, prerequisites) {
  const map = new Map();
  const res = [];
  let count = 0;
  const inDegrees = new Array(numCourses).fill(0);
  for (let prerequisite of prerequisites) {
    const [cur, pre] = prerequisite;
    inDegrees[cur]++;
    if (!map.has(pre)) {
      map.set(pre, [cur]);
    } else {
      map.set(pre, [...map.get(pre), cur]);
    }
  }
  const queue = [];
  for (let i = 0; i < numCourses; i++) {
    if (inDegrees[i] === 0) queue.push(i);
  }

  while (queue.length) {
    const top = queue.shift();
    const lessons = map.get(top);
    res.push(top); // 这里记录路径
    count++;
    if (lessons && lessons.length) {
      for (let lesson of lessons) {
        if (--inDegrees[lesson] === 0) queue.push(lesson);
      }
    }
  }
  return count === numCourses ? res : [];
};
```

当然第二题也有 dfs 解法，其实也是在第一题的基础上记录路径、判断成环。

实际上 dfs 每次递归，只要不在 onPath 和 visited 中，就是该路径的合理经过节点。因此每次在判断完 onPath 和 visited 通过之后就可以记录下来，最后会形成走过的路径。
**DFS 实现的拓扑排序，是在后序位置记录路径，再将路径反转**。至于原因可以参考这个树：

![](https://pic.imgdb.cn/item/62dc1d0bf54cd3f937721930.jpg)

一个节点只有依赖的所有节点遍历过，才能遍历该节点。对于这个数来说，从下向上看，每个节点只有等到其子节点都遍历过，才能遍历该节点。因此后序遍历的结果其实就是把这个图中的箭头全部反过来的合理排序；那么如果把后序遍历的结果 reverse，就是合理的拓扑排序结果

> 不一定需要翻转，如果建图时，邻接表中每个节点 key 的 value 是该节点的前驱节点而不是后继节点，即表示依赖于而不是被依赖的节点数组，就不需要翻转

代码如下：

```js
var findOrder = function (numCourses, prerequisites) {
  const map = new Map();
  for (let prerequisite of prerequisites) {
    const [cur, pre] = prerequisite; // cur表示当前课，pre表示依赖的课
    if (!map.has(pre)) {
      map.set(pre, [cur]);
    } else {
      map.set(pre, [...map.get(pre), cur]);
    }
  }
  const visited = new Set();
  const onPath = new Set();
  const path = [];

  let hasCycle = false;
  function dfs(node) {
    // 注意这里应该先改变hasCycle，否则可能导致visited中有该节点，而还没来得及更改成环就直接跳过
    if (onPath.has(node)) {
      hasCycle = true;
    }
    if (hasCycle || visited.has(node)) return;

    visited.add(node);
    onPath.add(node);
    const neighbours = map.get(node) || [];
    for (let neighbour of neighbours) {
      dfs(neighbour);
    }
    path.push(node); // 后序位置记录节点
    onPath.delete(node);
  }
  for (let i = 0; i < numCourses; i++) {
    dfs(i);
  }
  return hasCycle ? [] : path.reverse();
};
```

## 无向图的成环判断

无向图的成环判断和有向图的核心差别在于：需要在搜索时记录每个节点的父节点 parent，防止返回自己的父节点。
因为无向图本质是双向的有向图，因此每个节点都需要保证不能返回自己的父节点。其他方面和有向图完全相同。

```js
function hasCycle(n, edges) {
  const map = new Map();
  for (let edge of edges) {
    const [u, v] = edge;
    if (!map.has(u)) {
      map.set(u, [v]);
    } else {
      map.set(u, [...map.get(u), v]);
    }
    if (!map.has(v)) {
      map.set(v, [u]);
    } else {
      map.set(v, [...map.get(v), u]);
    }
  }

  const visited = new Set();
  const onPath = new Set();

  let hasCycle = false;
  function dfs(node, parent) {
    if (onPath.has(node)) {
      hasCycle = true;
      return;
    }
    if (node == undefined || hasCycle || visited.has(node)) return;

    visited.add(node);
    onPath.add(node);
    const neighbours = map.get(node);
    if (neighbours && neighbours.length) {
      for (let neighbour of neighbours) {
        // 这里，必须判断不能走到父节点
        if (neighbour !== parent) {
          dfs(neighbour, node);
        }
      }
    }
    onPath.delete(node);
  }

  for (let i = 0; i < n; i++) {
    dfs(i, undefined);
  }
  return hasCycle;
}
```

## 二分图

二分图的定义：二分图的顶点集可分割为两个互不相交的子集，图中每条边依附的两个顶点都分属于这两个子集，且两个子集内的顶点不相邻。
换句话说就是，二分图的每个边都可以做到该边的左右两个顶点颜色不同。如果不能做到，就不是二分图

![](https://pic.imgdb.cn/item/62dccf2af54cd3f937e052e0.jpg)

二分图的问题主要就是判断是否是二分图，以及判断可能的二分法。
先来说二分图的判定，主要思路就是遍历一遍图，一边遍历一边染色，看看能不能用两种颜色给所有节点染色，且相邻节点的颜色都不相同。
在遍历过程中（DFS），每次先判断某个顶点的邻居是否被遍历过。

- 如果没遍历过，就给邻居染上不同颜色。可以用一个布尔值数组表示某个顶点的颜色，这样就可以直接令`colors[neighbors] = !colors[node]`即可
- 如果遍历过，就判断邻居颜色和自己是否相同。一旦相同就说明不是二分图

初始化可以考虑一个全为 true 的 colors 数组，用于表示每个元素的颜色；

代码如下：

```js
var isBipartite = function (graph) {
  // 邻接表图
  const map = new Map();
  for (let i = 0; i < graph.length; i++) {
    map.set(i, [...graph[i]]);
  }
  const visited = new Set();
  // color为true和false的区别
  const colors = new Array(graph.length).fill(true);
  let isBipartite = true;

  function dfs(node) {
    if (!isBipartite || visited.has(node)) return;
    visited.add(node);
    const neighbours = map.get(node) || [];
    for (let neighbour of neighbours) {
      if (!visited.has(neighbour)) {
        // 给邻居染上不同颜色
        colors[neighbour] = !colors[node];
        dfs(neighbour);
      } else {
        // 如果颜色相同就说明不是二分
        if (colors[neighbour] === colors[node]) isBipartite = false;
      }
    }
  }
  for (let i = 0; i < graph.length; i++) {
    dfs(i);
  }
  return isBipartite;
};
```

# 特殊数据结构

## 哈希表

哈希表是一类可以很快查找数据的顺序存储结构。对于数组来说查找一个定值的最快速度也是 O(logn)，但是哈希表可以做到 O(1).

哈希表的实现原理就是通过一个 key 来快速确定存储的 value 的位置。通常数组只有一个顺序索引，而哈希表会用生成的唯一的哈希值作为索引。当查找一个数据时，通过转换为哈希值的形式就可以快速找到对应的值。

转换哈希值的函数叫做哈希函数。

举个例子，比如要存储学校里所有学生的姓名到一个哈希表中：

![](https://pic.imgdb.cn/item/62d3b2f0f54cd3f9373aaf38.jpg)

通过哈希函数转换之后，得到的结果就可以存在表中。

### 哈希函数

常见的哈希函数有：（x 表示原本的值）

- 除留取余法：`H(x) = x % p`，p 一般取不超过 x 的最大质数
- 直接定址法：`H(x) = a * x + b`，线性确定地址，比较简单的定址方法，但是可能比较浪费空间
- 折叠法、平方取中法、数字分析法：都是对数字型的键或哈希值操作，得到地址。

### 哈希冲突（哈希碰撞）

当哈希表的大小没那么大，或者哈希函数的特点决定的，会导致计算出的地址相同。这时就会产生哈希冲突。

比如 H(x) = x % 5 这种算法，6 和 11 都会计算出 1,此时就会产生冲突

哈希碰撞有两种解决方法：

- 拉链法（链接地址法）：就是将重复的元素按照链表拉成一链，放在冲突的地址后面。
  ![](https://pic.imgdb.cn/item/62d3b6b1f54cd3f937403ed5.jpg)
- 线性探测法：向后找一个地址，放下冲突的数据。前提是哈希表的大小要大于总数据量

### js 实现哈希表

基本哈希表的实现类似于手动实现一个类似 Map 的数据结构

```js
class HashMap {
  constructor(size = 10) {
    this.table = new Array(size); // 类似map的二维数组
    this.maxLen = size;
    this.size = 0;
  }
  // 哈希函数
  hashFunction(str) {
    let code = 0;
    console.log(str);
    for (let s of str) {
      code += s.charCodeAt();
    }
    // 这里哈希值的计算只是一种方式
    //  除留取余法
    return code % this.maxLen;
  }
  set(key, val) {
    let hashKey = this.hashFunction(key);
    // 考虑哈希冲突
    // 如果当前位置已经有值，就线性查找下一个有空的地方
    while (this.table[hashKey] != undefined && this.table[hashKey][0] !== key) {
      hashKey++;
      if (hashKey >= this.maxLen) throw new Error("无可用空间");
    }
    this.table[hashKey] = [key, val];
    this.size++;
  }
  get(key) {
    let hashKey = this.hashFunction(key);
    // get时同理，如果当前位置的值对应的键（table[hashKey][0]）不是要查找的key，就顺序向后找
    while (this.table[hashKey] != undefined && this.table[hashKey][0] !== key) {
      hashKey++;
      if (hashKey >= this.maxLen) return undefined;
    }
    return this.table[hashKey][1];
  }
  has(key) {
    return this.get(key) == undefined;
  }
  delete(key) {
    if (!this.has(key)) return false;
    let hashKey = this.hashFunction(key);
    while (this.table[hashKey] != undefined && this.table[hashKey][0] !== key) {
      hashKey++;
      if (hashKey >= this.maxLen) return false;
    }
    delete this.table[hashKey];
    this.size--;
    return true;
  }
}
```

## 堆

常见的堆结构实际上是最小/最大二叉堆的形式。
二叉堆本质是一个完全二叉树，在具体数据结构上，我们用一个数组表示：

![](https://pic.imgdb.cn/item/62d3d2bef54cd3f9376dbc36.jpg)

而最小/最大二叉堆就是对这个堆的排序。
最小二叉堆的定义是：每个节点都不大于其左子和右子节点的完全二叉树。因此实现上主要考虑把数组调整达到这个目的。

最小二叉堆的实现：

1. 首先以一个数组为基础结构。对于任意一个位置上的节点，可以得到其左子节点和右子节点以及父节点的位置

- 左子节点：2i-1
- 右子节点：2i+1
- 父节点：(i-1)/2

2. 当插入一个值时，首先先将其 push 到数组最尾端。然后从这个位置开始自下向上调整：

- 如果当前值比其父节点值大，就保持不动
- 如果比父元素小，就和父元素交换位置，然后再从父元素位置开始，继续向上调整，直到确保最小堆的结构特点

3. 对于调整完毕的二叉堆，堆顶元素一定是最小值。如果要找第二小的值，可以将堆顶值 shift，然后将堆尾元素 unshift 到堆顶，再从上向下进行调整。从上向下调整称作“堆化”，是二叉堆最重要的实现。在堆排序，初始化完成数组后，每一次堆化都会得到当前数组中最小的值。

```js
class MinHeap {
  constructor() {
    this.heap = [];
  }
  getLeftChildIndex(index) {
    return 2 * index + 1 >= this.heap.length ? null : 2 * index + 1;
  }
  getRightChildIndex(index) {
    return 2 * index + 2 >= this.heap.length ? null : 2 * index + 2;
  }
  getParentIndex(index) {
    return index === 0 ? null : Math.floor((index - 1) / 2);
  }
  swap(a, b) {
    [this.heap[a], this.heap[b]] = [this.heap[b], this.heap[a]];
  }
  insert(value) {
    if (this.heap.length === 0) {
      this.heap.push(value);
      return;
    }
    this.heap.unshift(value);
    this.shiftDown(0);
    return true;
  }
  min() {
    return this.heap.length === 0 ? null : this.heap[0];
  }

  pop() {
    if (this.heap.length) {
      const removedValue = this.heap.shift();
      this.heap.unshift(this.heap.pop()); // 把底部元素插入根
      this.heap.length > 1 && this.shiftDown(0); // 从上向下调整
      return removedValue;
    }
    return null;
  }

  // 向下调整
  shiftDown(currIndex) {
    const leftIndex = this.getLeftChildIndex(currIndex); // 获取左右子元素索引
    const rightIndex = this.getRightChildIndex(currIndex);
    // 如果当前元素比左右子元素大，选择左右元素较小的一个交换并递归
    // 如果是最小堆，这里就是小于，取左右两个元素较小的
    if (
      this.heap[currIndex] > this.heap[leftIndex] ||
      this.heap[currIndex] > this.heap[rightIndex]
    ) {
      const tmpIndex =
        this.heap[leftIndex] > this.heap[rightIndex] ? rightIndex : leftIndex;
      this.swap(currIndex, tmpIndex);
      this.shiftDown(tmpIndex);
    }
  }
}
```

### 堆排序

堆排序实际上是利用了堆的原理，但是并不需要一个完整的堆数据结构。

堆排序主要的步骤有两个：

1. 构建大顶堆。
2. 依次把大顶堆的最大的数和最后一个数交换，然后继续处理前面的数，直到全部有序为止。如图所示：

![](https://pic.imgdb.cn/item/62d3e6d5f54cd3f9378e981a.jpg)

每次得到堆顶的最大数，把这个数放到数组后面，然后再对前面的数排序，最后就可以得到一个升序数组。

代码如下：

```js
const swap = (arr, a, b) => ([arr[a], arr[b]] = [arr[b], arr[a]]);

function buildMaxHeap(arr) {
  for (let i = Math.floor(arr.length / 2); i >= 0; i--) {
    heapify(arr, i, arr.length);
  }
}

function heapify(arr, i, size) {
  const left = 2 * i + 1;
  const right = 2 * i + 2;
  let mid = i;
  // 关键点2：当left和right超出size时，放弃这个值
  if (left < size && arr[left] > arr[mid]) {
    mid = left;
  }

  if (right < size && arr[right] > arr[mid]) {
    mid = right;
  }

  if (mid !== i) {
    swap(arr, i, mid);
    heapify(arr, mid, size);
  }
}

function heapSort(arr) {
  buildMaxHeap(arr);
  let len = arr.length; // 关键点0：注意size和len是数组长度，而不是最后一项索引，因此相当于是以len为边界+1，left和right都不能等于size或len
  for (let i = len - 1; i > 0; i--) { // 关键点1：这里i>0终止
    swap(arr, 0, i);
    len--;
    heapify(arr, 0, len);
  }
  return arr;
}
```

### Top K

有了上一步堆排序，其实 Top K 问题也就很好解决了。

如果要求前 K 个最大的数，只需要在上面的基础上限制循环次数为 k 即可：

```js
// 其他代码不变
const TopK = (arr, k) => {
  let len = arr.length - 1;
  const res = [];
  for (let i = Math.floor(arr.length / 2); i >= 0; i--) {
    heapify(arr, i);
  }

  for (let i = 1; i <= k; i++) {
    swap(arr, 0, len);
    len--;
    heapify(arr, 0, len);
    res.push(arr[arr.length - i]);
  }
  return res;
};
```

如果是 K 个最小的数，就需要反过来，从小顶堆得到。
小顶堆和大顶堆的唯一区别：把 heapify 函数中的大于号和小于号统一换个方向就行：

```js
const heapify = (heap, index, heapSize = heap.length) => {
  const left = 2 * index + 1 > heapSize ? null : 2 * index + 1;
  const right = 2 * index + 2 > heapSize ? null : 2 * index + 2;

  // 从这里后面的全换成小于号
  if (heap[left] < heap[index] || heap[right] < heap[index]) {
    const tmp = heap[left] < heap[right] ? left : right;
    swap(heap, tmp, index);
    heapify(heap, tmp, heapSize);
  }
};
```

## 图

图的表示方法：

- 邻接矩阵：最直观，但是比较浪费空间
  ![](https://pic.imgdb.cn/item/62d3ff7df54cd3f937b234a0.jpg)

- 邻接表：省空间，常用方法

![](https://pic.imgdb.cn/item/62d3ff89f54cd3f937b2464c.jpg)

- 关联矩阵：矩阵的行表示顶点，列表示边，如果一个顶点和两个边连接则表示为 1，否则是 0。
  常用于于边的数量比顶点多的情况，节省内存

![](https://pic.imgdb.cn/item/62d40024f54cd3f937b32634.jpg)

### 邻接表

js 实现邻接表表示图主要通过 map，map 中的每一项的键是一个顶点，值是相连的顶点组成的链表（这里用有序数组）
同时还需要一个记录所有节点的数组。

```js
class Graph {
  constructor(isDirected = false) {
    this.isDirected = isDirected; // 有向图
    this.vertices = []; // 所有顶点
    this.adjList = new Map(); // 邻接表
  }

  // 添加顶点
  addVertex(v) {
    if (!this.vertices.includes(v)) {
      this.vertices.push(v);
      this.adjList.set(v, []); // 邻接表的每一项的值都是一个数组,包含和该节点相邻的其他节点
    }
  }

  //添加边,参数是两个顶点
  addEdge(v, w) {
    if (!this.adjList.has(v)) this.addVertex(v);
    if (!this.adjList.has(w)) this.addVertex(w);
    this.adjList.get(v).push(w); // v顶点的邻接数组加入w
    if (!this.isDirected) this.adjList.get(w).push(v); // 如果是无向图,w也需要
  }
}
```

邻接表在大多数题目对应的都是有向图，其中每个节点 key 表示有出度的节点，而只有入度的节点一般不会在 key 中；每个节点 key 对应的的 value 都是该节点的后继节点，即依赖于该节点的节点。
比如这样一幅图，邻接表应该这样表示：

```js
// 数组的序号就是key，即节点0-3
const graph = [[1, 3, 4], [2, 3, 4], [3], [4]];
```

![](https://pic.imgdb.cn/item/62dcd7cff54cd3f937140a44.jpg)

如果是无向图，就可以看作是双向图，这时连接两边的两个节点都应该存在，即节点 key 应当包含每个节点，并且 value 是自己相连的全部节点，而不只是单向的。

```js
const graph = [
  [1, 3, 4],
  [0, 2, 3, 4],
  [1, 3],
  [0, 1, 2, 4],
  [0, 1, 3],
];
```

### 图的遍历

图的遍历和树基本相似，主要的两种方法同样是 DFS 和 BFS。

但是注意的是，图是有可能包含环的。因此必须要考虑避免环的情况，已经访问过的节点不能再次访问。

图的 BFS 遍历：

> 注意图的遍历和树的层序遍历的区别：即图的遍历不一定需要次数为 len 的循环，即这个东西
>
> ```js
> while(queue.length){
>   const len = queue.length
>   for(let i = 0;i < len; i++){ // 图不一定需要这个循环
>     .....
>   }
> }
> ```

```js
function BFS(graph, start) {
  const adjList = graph.adjList; // 邻接表
  const queue = [];
  queue.push(start);

  const visited = new Set();
  const res = [];

  while (queue.length) {
    const u = queue.shift();
    if (visited.has(u)) continue;
    visited.add(u);
    const neighbors = adjList.get(u);
    for (let w of neighbors) {
      queue.push(w);
    }
    res.push(u);
  }
  console.log(res);
}
```

如果希望通过 BFS 记录路径，就不能这样写。queue 中不应该只是每个节点，而是**到该节点为止前面走过的路径**，也就应该是一个数组。每次 shift 得到一组路径，这组路径的最后一项就是当前走到的位置，以这一项为基准再获取其相邻的节点，拼接放入数组。

代码如下：

```js
function BFS(graph){
  const res = [];
  const n = graph.length;
  // 队列的每一项表示到该节点为止走过的路径数组
  const queue = [[0]];

  while (queue.length) {
    const top = queue.shift();
    // 每一项是一个数组，这个数组的最后一项就是当前走到的那个节点last
    const last = top[top.length - 1];
    // 获取last的相邻节点，依次遍历
    for (let neighbour of graph[last]) {
      if (/*走到终点*/){
        res.push([...top, neighbour]);
      }else{
        // 没走到终点，继续添加当前项的相邻节点到queue中
        // 这里push的是一个新数组，也就是在当前路径的基础上又添加了新的一步
        // 比如原先的路径是[[0,1]]，1和2、3相邻，那么这里就相当于把[0,1]弹出，然后再追加为[[0,1,2],[0,1,3]]。
        queue.push([...top, neighbour]);
      }
    }
  }

  return res;
}
```

BFS 的成环判断也是这个思路。如果当前节点在之前存储的走过的节点中出现过，就说明成环了。

---

DFS 遍历：

```js
function DFS(graph, start) {
  const adjList = graph.adjList;
  const res = [];
  // visited记录的是总共走过的节点，即不论是从哪个节点开始的，只要该节点走过就进入visited
  const visited = new Set();
  // path记录的是当前这个路径下走过的节点，即本次走过的路径，判断成环时是必要的
  // 如果要记录走过的路径，path也会记录
  const path = new Set();

  function dfs(node) {
    if (!node) return;
    if (visited.has(node)) return;
    visited.add(node);
    path.add(node);

    // 可以在这里得到走过的路径
    // 先序位置记录路径
    // if(...){
    //   paths.push([...path])
    // }

    // 输出节点
    res.push(node);

    const neis = adjList.get(node);
    for (let neighbor of neighbors) {
      dfs(neighbor);
    }
    // path在遍历完之后需要删除，相当于本次走过了，下次重新统计，类似回溯
    // 这里是拓扑排序记录路径的地方（后序位置记录路径）
    path.delete(node);
  }
  dfs(start);
  console.log(res);
}
```

另外还有一点，很多题的 DFS 都需要从每个顶点作为起点遍历图，即类似这样：

```js
for(...){
  dfs(node[i])
}
```

如果题目明确指明起点，显然就从起点开始就行。否则其他大多数需要走遍完整图的算法（拓扑排序、二分图、并查集等）都需要这样遍历

## 并查集

并查集是解决图论中「动态连通性」问题的。即，图可能有多个节点，这些节点之间的连接可以看作是一种连通。图的每个节点之间可能相互连通，也可能孤立成小岛

![](https://pic.imgdb.cn/item/62dce9c1f54cd3f937811caa.jpg)

比如在这个图中，有两个独立的“小岛”，那么我们就可以称该图的连通分量数目为 2。如果把 2 和 3 连接起来变成一整个图，连通分量就会变为 1。处理连通分量动态变化的数据结构就叫做并查集

并查集的结构本质是一个森林，即若干个独立的图的结合。在实际的数据结构实现上，可以通过数组实现并查集。基本数据结构：

```js
class UF {
  constructor(n) {
    this.count = n;
    this.parent = [];
  }
  // 连通p和q
  union(p, q) {}
  // 检查p和q的连通性
  connected(p, q) {}
  // 找到p的根节点
  find(p) {}
}
```

主要特点如下：

- 设定树的每个节点有一个指针指向其父节点，如果是根节点的话，这个指针指向自己。具体来说，通过一个 parent 数组用来记录每个节点的父节点。初始化每个节点的父节点都指向自己

![](https://pic.imgdb.cn/item/62dceafef54cd3f937890fac.jpg)

- 如果某两个节点被连通，则让其中的（任意）一个节点的根节点接到另一个节点的根节点上。这样，如果节点 p 和 q 连通的话，它们一定拥有相同的根节点

![](https://pic.imgdb.cn/item/62dceb0ff54cd3f937897cd5.jpg)

- 优化层面，连接和检查连接的算法都依赖于 find，因此 find 的时间复杂度会影响整个的复杂度。find 函数的基本操作是逐个向上遍历查找父节点，显然如果树高越小，find 就查找的越快。因此应该尽可能减少树的高度

![](https://pic.imgdb.cn/item/62dcecc4f54cd3f937946fce.jpg)

并查集代码如下：

```js
class UnionFind {
  constructor(n) {
    this.count = n;
    this.parent = []; // parent[i]表示节点i的父节点
    for (let i = 0; i < n; i++) {
      this.parent[i] = i; // 每个节点初始化是自己的父节点
    }
  }

  union(p, q) {
    // 获取p和q的根节点
    const [rootP, rootQ] = [this.find(p), this.find(q)];
    if (rootP === rootQ) return;
    // 连通两棵树：一个的根节点的父节点设置为另一个的根节点即可
    this.parent[rootP] = rootQ;
    this.count--;
  }
  connected(p, q) {
    // 判断两个节点的根节点是否相同
    return this.find(p) === this.find(q);
  }
  find(node) {
    // 基本算法是找到根节点
    if (this.parent[node] !== node) {
      // 如果当前节点不是自己的父节点，说明不是根节点，就继续递归
      // 每次递归都把当前节点向上挂载一层，直到根节点
      // 因此所有最后的节点都会直接连在根节点上，通过parent[node]就可以直接得到根节点
      this.parent[node] = this.find(this.parent[node]);
    }
    return this.parent[node];
  }
}
```

## 前缀树

前缀树是类似多叉树的一种结构，但是区别在于它的每一个节点都包含一个长度为 26 的数组，用于表示 26 个字母。如果某一个位置上的值不为 null，就说明包含这个字符，如图所示：

![](https://pic1.imgdb.cn/item/6340437316f2c2beb1552693.jpg)

前缀树的方法主要有插入、查找和前缀查找三种。

- 插入：从根节点的 next 数组开始，遍历字符，查找 next 数组中是否包含当前字符。如果有就向下，没有就在当前字符对应的 next 数组位置创建一个 TrieNode 节点并向下。注意当所有字符都遍历完之后，停到的最后一个节点的 isEnd 属性要为 true。
- 查找：同样是从根节点开始，查找每一个字符对应的 next 数组的位置。如果该字符对应在 next 数组中的值为 null，就说明当前这条路径对应的单词不匹配，返回 false。并且，即使该单词遍历完成，但是最后到达的节点的 isEnd 不为 true，同样属于 false
- 查找前缀：和查找唯一的区别就是最后的 isEnd 可以不为 true。

代码如下：

```js
class TrieNode {
  constructor(isEnd = false, next = new Array(26)) {
    this.isEnd = isEnd; // 用于标识当前节点是不是叶节点，即是不是一个字符的终点
    this.next = next;
  }
}
class Trie {
  constructor() {
    this.root = new TrieNode(); // 根节点不表示任何一个字符
    this.baseCode = "a".charCodeAt(0);
  }
  insert(word) {
    let node = this.root;
    for (const char of word) {
      const index = char.charCodeAt(0) - this.baseCode;
      if (!node.next[index]) {
        // 如果next数组还没有当前字符
        node.next[index] = new TrieNode(); // 为这个字符在next数组的位置创建一个节点，相当于子节点
      }
      node = node.next[index]; // 向子节点走一个
    }
    node.isEnd = true; // 最后一个节点的isEnd应该是true，表示到这个节点为止是一个完整单词
  }
  search(word) {
    let node = this.root;
    for (const char of word) {
      const index = char.charCodeAt(0) - this.baseCode;
      if (!node.next[index]) return false; // 一旦下一个字符不存在，就必然是false
      node = node.next[index];
    }
    return node.isEnd; // 最后一个字符要是end；如果只是查找前缀的话就不需要
  }
  searchPrefix(word) {
    let node = this.root;
    for (const char of word) {
      const index = char.charCodeAt(0) - this.baseCode;
      if (!node.next[index]) return false;
      node = node.next[index];
    }
    return true;
  }
}
```

这里的搜索操作都是迭代，实际上用递归也可以。但是前缀树由于每次只会匹配一条链，因此其实更类似链表，所以迭代看起来要直观的多。

### 添加与搜索单词

> 请你设计一个数据结构，支持 添加新单词 和 查找字符串是否与任何先前添加的字符串匹配 。
> 实现词典类 WordDictionary ：
> WordDictionary() 初始化词典对象
> void addWord(word) 将 word 添加到数据结构中，之后可以对它进行匹配
> bool search(word) 如果数据结构中存在字符串与  word 匹配，则返回 true ；否则，返回   false 。word 中可能包含一些 '.' ，每个  . 都可以表示任何一个字母。

这道题的方法就是前缀树。addword 就是向前缀树中添加单词，但是 search 有一点不一样，要考虑`'.'`的情况。
因为`'.'`可以表示任意一个字符，相当于对于一个节点来说，如果是确定的字符 char，下一个就应该是 next[char]；而对于点号节点，就应该遍历 next 的所有有效字符，然后一一检查，只要有一个找到就可以。
由于需要对点号字符这样的操作，所以用递归遍历搜索更好一点。

```js
// 前缀树的代码就是前面的
class TrieNode {}
class Trie {}

class WordDictionary {
  constructor() {
    this.trie = new Trie();
  }
  addWord(word) {
    this.trie.insert(word);
  }
  search(word) {
    // dfs函数接收一个index和node，返回字符串从index位置开始，节点从node开始的搜索结果。
    function dfs(index, node) {
      if (index === word.length) {
        // 如果index走到最后一位，节点就应该是末尾
        return node.isEnd;
      }
      const char = word[index];
      if (char !== ".") {
        const code = char.charCodeAt(0) - "a".charCodeAt(0);
        const child = node.next[code]; // 取当前节点的对应子节点
        if (child) {
          return dfs(index + 1, child);
        }
      } else {
        const res = [];
        for (const child of node.next) {
          // 遍历所有子节点
          if (child) {
            res.push(dfs(index + 1, child));
          }
        }
        return res.length === 0 ? false : res.reduce((pre, cur) => pre || cur); // 子节点只要有一个返回true就可以
      }
      return false; // 如果没有子节点就返回false
    }
    return dfs(0, this.trie.root);
  }
}
```

### 搜索推荐系统

https://leetcode.cn/problems/search-suggestions-system/

> 给你一个产品数组 products 和一个字符串 searchWord ，products 数组中每个产品都是一个字符串。
> 请你设计一个推荐系统，在依次输入单词 searchWord 的每一个字母后，推荐 products 数组中前缀与 searchWord 相同的最多三个产品。如果前缀相同的可推荐产品超过三个，请按字典序返回最小的三个。
> 请你以二维列表的形式，返回在输入 searchWord 每个字母后相应的推荐产品的列表。
>
> 示例 1：
>
> 输入：products = ["mobile","mouse","moneypot","monitor","mousepad"], searchWord = "mouse"
> 输出：[
> ["mobile","moneypot","monitor"],
> ["mobile","moneypot","monitor"],
> ["mouse","mousepad"],
> ["mouse","mousepad"],
> ["mouse","mousepad"]
> ]
> 解释：按字典序排序后的产品列表是 ["mobile","moneypot","monitor","mouse","mousepad"]
> 输入 m 和 mo，由于所有产品的前缀都相同，所以系统返回字典序最小的三个产品 ["mobile","moneypot","monitor"]
> 输入 mou， mous 和 mouse 后系统都返回 ["mouse","mousepad"]

这道题虽然看着就像是用前缀树的样子，但是实际上可以不使用前缀树，直接暴力法也可以过。
有一个比较有技巧的方法，即逐渐缩小搜索范围；每次 searchWord 向后一个字符，就缩小搜索的 products 的范围，然后在缩小之后的 products 内部进行筛选即可。
这个方法比较简单，这里不再写了。
说说前缀树写法：之前的前缀树的 search 只是判断 true/false，即只是判断是否包含这个单词；但这里需要的不是判断是否包含，而是包含的具体是哪些。
因此我们需要改造一下前缀树，让其能够存储匹配到的字符串。具体来说，在每个 TrieNode 上增加一个 words 数组，用于存储匹配到当前 node 时符合条件的单词有哪些。
举个例子：

```
products = ["abc","abe","a","abd"], searchWord ="abe"

trie = {
  root: {
    child: {
      a: {
        child: {
          b: {
            child: {
              c: { child: {}, words: ['abc'] },
              d: { child: {}, words: ['abd'] },
              e: { child: {}, words: ['abe'] }
            },
            words: ['abc', 'abd', 'abe']
          }
        },
        words: ['a', 'abc', 'abd', 'abde']
      }
    },
    words: []
  }
}
```

每个节点的 words 存储的实际上是这个节点对应的前缀匹配到的单词。比如 a 节点的 words 就是所有以 a 开头的字符串组成的数字，b 节点就是以 ab 为开头的，依次类推。
修改一下原来的 insert 代码，其实就是增加一个数组：

```js
insert(word) {
  let node = this.root;
  for (const char of word) {
    const index = char.charCodeAt(0) - this.baseCode;
    if (!node.next[index]) {
      // 如果next数组还没有当前字符
      node.next[index] = new TrieNode();
    }
    node = node.next[index];
    node.words.push(word) // 注意顺序，应该是先走到子节点上再创建，不然root上就会有words了
  }
  node.isEnd = true;
}
```

在查找时，如果在该节点就能匹配上（node[index]存在），说明当前节点的 words 内的所有单词都包含要查找的部分。比如上面的例子中`['abc', 'abd', 'abe']`数组，如果查找的词是`'ab'`，显然这里所有的词都是包含查找词的。由此就可以得到匹配结果。

需要注意的是，这里需要将整个查找词匹配完才能记录 words 数组，即在遍历完成后再输出。

```js
searchPrefix(word) {
  let node = this.root;
  for (const char of word) {
    const index = char.charCodeAt(0) - this.baseCode;
    if (!node.next[index]) return [];
    node = node.next[index];
  }
  return node.words;
}
```

全部代码：

```js
class TrieNode {
  constructor(isEnd = false, next = new Array(26)) {
    this.next = next;
    this.words = [];
  }
}
class Trie {
  constructor() {
    this.root = new TrieNode(); // 根节点不表示任何一个字符
    this.baseCode = "a".charCodeAt(0);
  }
  insert(word) {
    let node = this.root;
    for (const char of word) {
      const index = char.charCodeAt(0) - this.baseCode;
      if (!node.next[index]) {
        node.next[index] = new TrieNode();
      }
      node = node.next[index];
      // 这里的判断是这道题需要前三个匹配结果，其他时候可以去掉
      if (node.words.length < 3) node.words.push(word);
    }
  }
  search(word) {
    let node = this.root;
    for (const char of word) {
      const index = char.charCodeAt(0) - this.baseCode;
      if (!node.next[index]) return [];
      node = node.next[index];
    }
    return [...node.words];
  }
}
/**
 * @param {string[]} products
 * @param {string} searchWord
 * @return {string[][]}
 */
var suggestedProducts = function (products, searchWord) {
  products.sort();
  const trie = new Trie();
  for (const pro of products) {
    trie.insert(pro);
  }
  const res = [];
  for (let i = 0; i < searchWord.length; i++) {
    res.push(trie.search(searchWord.slice(0, i + 1)));
  }
  return res;
};
```

# 其他类型题目

## 圆圈中最后剩下的数字

https://leetcode.cn/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/

这道题的最佳解法是数学解法，但是用循环链表暴力计算也可以得出，只是会超时。
这里先放上会超时的代码，如果不考虑超时的话这个写法是没问题的：

```js
function ListNode(val, next) {
  this.val = val || 0;
  this.next = next || null;
}
const createLinkList = (n) => {
  const head = new ListNode(0);
  let p = head;
  for (let i = 1; i < n; i++) {
    const listNode = new ListNode(i);
    p.next = listNode;
    p = p.next;
  }
  p.next = head;
  return { last: p, head };
};
/**
 * @param {number} n
 * @param {number} m
 * @return {number}
 */
var lastRemaining = function (n, m) {
  const { head, last } = createLinkList(n);
  let p = head;
  let prev = last;
  let cnt = n;
  while (true) {
    for (let i = 1; i < (m > cnt ? m % cnt : m); i++) {
      prev = p;
      p = p.next;
    }
    prev.next = prev.next.next;
    p = prev.next;
    cnt--;
    if (p === prev) return p.val;
  }
};
```

数学解法：约瑟夫环，采用倒推的思路。

因为最后数组中只剩下一个数，因此这个数的序号一定是 0。

然后从 0 开始倒推，每次的新序号就是`(上一次的序号+去掉的第m个数的m)%当前数组大小`

参考https://leetcode.cn/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/solution/javajie-jue-yue-se-fu-huan-wen-ti-gao-su-ni-wei-sh/

```js
var lastRemaining = function (n, m) {
  let res = 0;
  for (let i = 2; i <= n; i++) {
    res = (res + m) % i;
  }
  return res;
};
```

## 汉诺塔问题

https://leetcode.cn/problems/hanota-lcci/

> 在经典汉诺塔问题中，有 3 根柱子及 N 个不同大小的穿孔圆盘，盘子可以滑入任意一根柱子。一开始，所有盘子自上而下按升序依次套在第一根柱子上(即每一个盘子只能放在更大的盘子上面)。移动圆盘时受到以下限制:
> (1) 每次只能移动一个盘子;
> (2) 盘子只能从柱子顶端滑出移到下一根柱子;
> (3) 盘子只能叠在比它大的盘子上。
> 请编写程序，用栈将所有盘子从第一根柱子移到最后一根柱子。
> 你需要原地修改栈。

两个思路：

1. 递归，即每次考虑相当于两个盘子的移动。比如 n=2 的最基本情况；当 n>2 时，把上面多于一个的盘子看作一个整体，即一部分是 1 个最底端的盘子，另一部分是上面的 n-1 个盘子，然后按照移动两个的方法移动这两部分。我们不用考虑上面 n-1 个是如何移动的，递归会自然做到。
   具体来说，就是：

- n = 1 时，直接把盘子从 A 移到 C；
- n > 1 时，
  1. 先把上面 n - 1 个盘子从 A 移到 B（子问题，递归）；
  2. 再将最大的盘子从 A 移到 C；
  3. 再将 B 上 n - 1 个盘子从 B 移到 C（子问题，递归）。

上面这个逻辑就和代码几乎完全对应。代码如下：

```js
var hanota = function (A, B, C) {
  const n = A.length;
  function move(n, A, B, C) {
    // move函数表示把n个盘子按照合法顺序从A移到C
    if (n === 1) {
      C.push(A.pop());
    }
    move(n - 1, A, C, B);
    C.push(A.pop());
    move(n - 1, B, A, C);
  }
  move(n, A, B, C);
};
```

递归解法几乎隐藏了确切的移动方式，只是通过对最基本的 n=2 的形式的扩充。而迭代解法则使用具体的移动策略完成

迭代的讲解具体参考：https://leetcode.cn/problems/hanota-lcci/solution/by-1105389168-tijv/

代码：

```js
var hanota = function (A, B, C) {
  if (A.length === 1) {
    C.push(A.pop());
    return;
  }
  const lists = [];
  const n = A.length;
  if (n % 2 === 0) lists.push(A, B, C);
  else lists.push(A, C, B);

  let curr = 0;
  while (C.length < n) {
    const next = (curr + 1) % 3;
    const prev = (curr + 2) % 3;
    lists[next].push(lists[curr].pop()); // 把当前的最小的移动到下一个
    if (lists[prev].length === 0) {
      lists[prev].push(lists[curr].pop()); // 如果有空的，就移动到空的上
    } else {
      // 否则另外两个里顶端较小的一个移到较大的一个上
      if (
        lists[prev][lists[prev].length - 1] >
        lists[curr][lists[curr].length - 1]
      ) {
        lists[prev].push(lists[curr].pop());
      } else {
        lists[curr].push(lists[prev].pop());
      }
    }
    curr = next;
  }
};
```

## 字符串乘法

> 给定两个以字符串形式表示的非负整数  num1  和  num2，返回  num1  和  num2  的乘积，它们的乘积也表示为字符串形式。
> 注意：不能使用任何内置的 BigInteger 库或直接将输入转换为整数。

这个题不是很难，但是很麻烦。思路就是把乘积拆成先个位数相乘，然后末尾加上若干个 0（表示十位数百位数这样），最后再加起来

```js
if (num1 == 0 || num2 == 0) return "0";
const addZero = (str, amount, dire = "back") => {
  const arr = str.split("");
  for (let i = 0; i < amount; i++) {
    if (dire === "back") arr.push(0);
    else arr.unshift(0);
  }
  return arr.join("");
};

const mult1 = (str, num) => {
  let acc = 0;
  const res = [];
  for (let i = str.length - 1; i >= 0; i--) {
    let val = Math.floor((acc + str[i] * num) % 10);
    acc = Math.floor((acc + str[i] * num) / 10);
    res.unshift(val);
  }
  if (acc !== 0) res.unshift(acc);
  return res.join("");
};
const add = (str1, str2) => {
  const [len1, len2] = [str1.length, str2.length];
  const len = Math.max(len1, len2);
  if (len1 > len2) {
    str2 = addZero(str2, len1 - len2, "front");
  } else if (len1 < len2) {
    str1 = addZero(str1, len2 - len1, "front");
  }
  let acc = 0;
  const res = [];
  for (let i = len - 1; i >= 0; i--) {
    let val = Math.floor(
      (acc + Number(str1[i] || 0) + Number(str2[i] || 0)) % 10
    );
    acc = Math.floor((acc + Number(str1[i] || 0) + Number(str2[i] || 0)) / 10);
    res.unshift(val);
  }
  if (acc !== 0) res.unshift(acc);
  return res.join("");
};

let sum = "0";
const longerNum = num1.length > num2.length ? num1 : num2;
const shoterNum = num1.length <= num2.length ? num1 : num2;
for (let j = shoterNum.length - 1, i = 0; j >= 0; j--, i++) {
  let res = mult1(longerNum, shoterNum[j]);
  res = addZero(res, i);
  console.log(res);
  sum = add(sum, res);
}
return sum;
```

## 旋转一个矩阵

https://leetcode.cn/problems/rotate-image/

> 给定一个 n × n 的二维矩阵  matrix 表示一个图像。请你将图像顺时针旋转 90 度。
> 你必须在 原地 旋转图像，这意味着你需要直接修改输入的二维矩阵。请不要 使用另一个矩阵来旋转图像。

这个题其实关键在于思路，思路有了代码很好写。
思路就是：一个矩阵顺时针旋转 90°，就相当于先按照从左上到右下的对角线两两交换值，然后再两两交换对应的列即可（比如第一列和第四列，第二列和第三列这样）。
如果是逆时针，就把对角线变成右上到左下即可。

```js
/**
 * @param {number[][]} matrix
 * @return {void} Do not return anything, modify matrix in-place instead.
 */
var rotate = function (matrix) {
  const m = matrix.length;
  const n = matrix[0].length;
  for (let i = 0; i < m; i++) {
    for (let j = i; j < n; j++) {
      // 注意这里j从i开始，否则就会又转回去了
      [matrix[i][j], matrix[j][i]] = [matrix[j][i], matrix[i][j]];
    }
  }
  for (let i = 0; i < m; i++) {
    for (let j = 0; j < n / 2; j++) {
      [matrix[i][j], matrix[i][n - j - 1]] = [
        matrix[i][n - j - 1],
        matrix[i][j],
      ];
    }
  }
};
```

## 简化路径

https://leetcode.cn/problems/simplify-path/

> 给你一个字符串 path ，表示指向某一文件或目录的 Unix 风格 绝对路径 （以 '/' 开头），请你将其转化为更加简洁的规范路径。
> 示例 1：
>
> 输入：path = "/home/"
> 输出："/home"
> 解释：注意，最后一个目录名后面没有斜杠。

思路其实不是很复杂，主要是通过栈。

```js
/**
 * @param {string} path
 * @return {string}
 */
var simplifyPath = function (path) {
  if (path[path.length - 1] === "/") path = path.slice(0, path.length - 1); // 去掉尾部多余的/
  if (path[0] !== "/") path = "/" + path; // 给头部加上'/'
  let tmp = "";
  for (let i = 0; i < path.length; i++) {
    if (path[i] === "/" && path[i + 1] === "/") continue; // 删掉双斜杠
    tmp += path[i];
  }
  path = tmp;

  const files = path.split("/");
  files.shift();
  const stack = [];
  for (let file of files) {
    if (file === ".") continue; // 表示本级目录
    else if (file === "..") stack.pop(); // 表示回到上一级目录，这时候栈顶对应的路径就应该弹出，表示返回到了栈顶路径的上一层
    else stack.push(file);
  }
  return "/" + stack.filter((str) => str !== "").join("/");
};
```

## 简单计算器

https://leetcode.cn/problems/basic-calculator-ii/

> 给你一个字符串表达式 s ，请你实现一个基本计算器来计算并返回它的值。
> 整数除法仅保留整数部分。
> 你可以假设给定的表达式总是有效的。所有中间结果将在  [-2^31, 2^31 - 1] 的范围内。
> 示例 1：
> 输入：s = "3+2\*2"
> 输出：7

这道题因为没有括号，因此不用考虑转为后缀表达式求值；
思路还是利用栈，每个数字前都有一个符号（第一个数字视为+）

- 如果这个符号是+/-，就入栈数字
- 如果这个符号是乘或除，就把当前数字和栈顶数字取出来做相乘、相除，并把结果入栈
- 最后栈中剩下的数字全部相加即可。

```js
/**
 * @param {string} s
 * @return {number}
 */
var calculate = function (s) {
  const stack = [];

  if (s[0] !== "-") s = "+" + s;
  for (let i = 0; i < s.length; ) {
    let num = "";
    if (s[i++] === "+") {
      while (!isNaN(s[i])) num += s[i++];
      stack.push(+num);
    } else if (s[i++] === "-") {
      while (!isNaN(s[i])) num += s[i++];
      stack.push(-Number(num));
    } else if (s[i++] === "*") {
      const top = stack.pop();
      while (!isNaN(s[i])) num += s[i++];
      stack.push(top * +num);
    } else if (s[i++] === "/") {
      const top = stack.pop();
      while (!isNaN(s[i])) num += s[i++];
      stack.push(Math.trunc(top / +num));
    }
  }
  return stack.reduce((pre, cur) => pre + cur);
};
```

## 完全计算器

完全计算器就是在简单计算器的基础上，增加了括号。基本思路还是简单计算器的方法，但对括号要做递归处理。
我们把括号内的算式视为一个运算的结果，这个结果由递归计算得出；然后将剩下的部分当做和简单计算器一样的方法去计算即可。

代码如下：

```js
var calculate = function (s) {
  var helper = function (s) {
    var stack = [];
    var sign = "+";
    var num = 0;

    while (s.length > 0) {
      var c = s.shift();
      if (!isNaN(c)) {
        num = +c;
      }
      // 遇到左括号开始递归计算 num
      // 把括号内的计算结果看做是一个值，即helper的返回值
      if (c === "(") {
        num = helper(s);
      }
      // 然后下面就和简单计算器一样的思路
      if ((!parseInt(c) && c !== " ") || s.length === 0) {
        if (sign === "+") {
          stack.push(num);
        } else if (sign === "-") {
          stack.push(-num);
        } else if (sign === "*") {
          stack[stack.length - 1] = stack[stack.length - 1] * num;
        } else if (sign === "/") {
          // javascript 除法向 0 取整的写法
          stack[stack.length - 1] = parseInt(stack[stack.length - 1] / num);
        }
        num = 0;
        sign = c;
      }
      // 遇到右括号返回递归结果
      if (c === ")") break;
    }

    return stack.reduce(function (sum, current) {
      return sum + current;
    }, 0);
  };

  return helper(s.split(""));
};
```

另一种通过index移动的写法，原理相同，只是改用index

```js
var calculate = function (s) {
  let idx = 0;
  const helper = function (s) {
    const stack = [];
    let sign = "+";
    let num = 0;

    while (idx < s.length) {
      const char = s[idx++];
      if (!isNaN(char)) {
        num = +char;
      }
      if (char === "(") {
        num = helper(s);
      }
      // 然后下面就和简单计算器一样的思路
      if ((!parseInt(char) && char !== " ") || idx === s.length) {
        if (sign === "+") {
          stack.push(num);
        } else if (sign === "-") {
          stack.push(-num);
        } else if (sign === "*") {
          stack[stack.length - 1] = stack[stack.length - 1] * num;
        } else if (sign === "/") {
          stack[stack.length - 1] = parseInt(stack[stack.length - 1] / num);
        }
        num = 0;
        sign = char;
      }
      // 遇到右括号返回递归结果
      if (char === ")") break;
    }

    return stack.reduce(function (sum, current) {
      return sum + current;
    }, 0);
  };

  return helper(s);
};
```

## 对角线遍历矩阵

https://leetcode.cn/problems/diagonal-traverse/

> 给你一个大小为 m x n 的矩阵 mat ，请以对角线遍历的顺序，用一个数组返回这个矩阵中的所有元素。

思路：首先确定遍历方式：始终从右上向左下遍历。因此以第一行为基准，每一行都从右上向左下遍历。
注意要确定遍历次数，遍历次数经过一点数学计算，应该是`长边+短边-1`，表示一共需要来回走多少次。最后把偶数位置上的数组翻转展平即可。

```js
/**
 * @param {number[][]} mat
 * @return {number[]}
 */
var findDiagonalOrder = function (mat) {
  // 右上角mat[i-1][j+1]
  // 左下角mat[i+1][i-1]
  const res = [];
  const m = mat.length;
  const n = mat[0].length;
  const mostTimes = m + n - 1;

  for (let k = 0; k < mostTimes; k++) {
    let i = 0;
    let j = k;
    const tmp = [];
    while (j >= 0 && i < m) {
      mat[i][j] != undefined && tmp.push(mat[i][j]);
      i++;
      j--;
    }
    res.push([...tmp]);
  }
  for (let i = 0; i < res.length; i++) {
    if (i % 2 === 0) {
      res[i] = res[i].reverse();
    }
  }
  return res.flat(Infinity);
};
```

## 整数转罗马数字

https://leetcode.cn/problems/integer-to-roman/

这道题有一个很重要的思路：数字的拆分。
因为罗马数字实际上只是表示不同，位数的顺序和阿拉伯数字并没有区别；因此可以将罗马数字对应的阿拉伯数字从高到低位依次从数字中减去，按照这个顺序就恰好是结果。
比如，1586 这个数字：

```
1586 -- 1000 -- M
586 -- 500 -- D
86 -- 50 -- L
36 -- 10 -- X
26 -- 10 -- X
16 -- 10 -- X
6 -- 5 -- V
1 -- 1 -- I
```

可以看到，随着数字的减小，对应的罗马数字也是依次减小的；因此只需要**从大到小遍历一次**就可以。每次遍历执行这个步骤：

1. 找到第一个比当前数 num 小的数
2. num - 该数，然后把结果放入数组。注意这里应该是一个**循环**，如果减去之后还是大于这个数，那就继续减，对应的是叠加的情况。
3. 当减到 num 比该数小的时候，继续向后遍历，直到再找到一个比 num 小的数，执行上面步骤。

代码如下：

```js
/**
 * @param {number} num
 * @return {string}
 */
var intToRoman = function (num) {
  const res = [];
  // map还要包含4和9的情况
  const map = new Map([
    [1, "I"],
    [4, "IV"],
    [5, "V"],
    [9, "IX"],
    [10, "X"],
    [40, "XL"],
    [50, "L"],
    [90, "XC"],
    [100, "C"],
    [400, "CD"],
    [500, "D"],
    [900, "CM"],
    [1000, "M"],
  ]);
  for (let [key, val] of [...map.entries()].reverse()) {
    while (num > key) {
      // 找到第一个比当前数num小的数
      num -= key;
      res.push(val);
    }
    if (num === 0) return res.join("");
  }
};
```

## 第 N 个数

这道题是一个找规律的题目，主要是前期的规律分析，找出规律之后很简单

按照题意，可以得到下面这个表格：

| 序号 | 范围      | 位数 | 总位数大小         | 起始序号 n    |
| ---- | --------- | ---- | ------------------ | ------------- |
| 1    | 0-9       | 1    | `1*9*10^0 = 9`     | 0             |
| 2    | 10-99     | 2    | `2*9*10^1 = 180`   | 0+9=9         |
| 3    | 100-999   | 3    | `3*9*10^2 = 2700`  | 180+9=189     |
| 4    | 1000-9999 | 4    | `4*9*10^3 = 36000` | 189+2700=2889 |

思路如下：

1. 首先计算 n 应该是上面哪个范围的，把 n 依次减去 $(9 \times 10^{n-1})\times n$ 得到 n 应该是哪个位数哪个范围的
2. 计算 n 应该在这个范围的多少位，`n/位数`可以得到是这一组数中的第几个数字，余数则是找到该数字之后的位数。

```js
/**
 * @param {number} n
 * @return {number}
 */
var findNthDigit = function (n) {
  if (n < 10) return n;
  let cnt = 0;
  let num = n;
  while (num > 9 * 10 ** cnt * (cnt + 1)) {
    num -= 9 * 10 ** cnt * (cnt + 1);
    cnt++;
  }
  cnt++;
  let rest = num - 1; // 这里注意要-1，因为num是这一组数的第num个数，但是每组数都是从0开始的，-1才得到真正的数字值
  let idx = Math.floor(rest / cnt); // 这一组数中的第idx个数
  let extra = rest % cnt; // 余数，相当于idx的第extra位数
  return (idx + 10 ** (cnt - 1)).toString().charAt(extra);
};
```

## 和为 k 的子数组 （前缀和）

> 给你一个整数数组 nums 和一个整数  k ，请你统计并返回 该数组中和为  k  的子数组的个数  。
> 示例 1：
> 输入：nums = [1,1,1], k = 2
> 输出：2

这道题用暴力法很好解决，固定左边界，伸缩右边界计算和即可。

最开始想到了背包问题和回溯解法，但是这两个方法都有问题：

- 01 背包要求 k>=0，这里 k 可以是负数，肯定不行。并且 01 背包得到的一般都是子序列而不是子数组
- 回溯对于不重复的数组可以处理，考虑子数组的话，设置一个 path 每次存储上一个的序号，如果本次不是相邻就跳过。但是对于有重复数字的数组好像 8 太行，反正我是没整出来

因此这道题标准解法是前缀和+map。其实看到子数组的和就应该考虑前缀和，并且前缀和应该也只能在子数组中用，子序列是不行的。

如果有一个子数组的和为 k，那么就必然会有`pre[j] - pre[i] === k`，也就是说它两边的两个前缀和的差恰好是 k。由此可以类似两数之和的思路：

- 设定一个 map，每次遍历计算 pre 并存储 pre 为键，出现次数为值；如果 pre 存在，就给次数+1
- 每次遍历查找是否有 pre-k 存在，有就给总数 cnt 加上 pre-k 对应的值（次数）

```js
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number}
 */
var subarraySum = function (nums, k) {
  let pre = 0;
  let cnt = 0;
  const map = new Map();
  map.set(0, 1);

  for (let num of nums) {
    pre += num;
    if (map.has(pre - k)) {
      cnt += map.get(pre - k);
    }
    if (map.has(pre)) {
      map.set(pre, map.get(pre) + 1);
    } else {
      map.set(pre, 1);
    }
  }
  return cnt;
};
```

## 最大数

> 给定一组非负整数 nums，重新排列每个数的顺序（每个数不可拆分）使之组成一个最大的整数。
> 注意：输出结果可能非常大，所以你需要返回一个字符串而不是整数。
> 示例 1：
> 输入：nums = [10,2]
> 输出："210"

这道题的思路其实是根据字典序排序数组。
因为字典序的排序规则是先比较最高位，相同再依次比较后面的位。正好符合题意，优先把最高位移到前面。
但是这仅限于位数相同的情况；当位数不同时，就会出现问题
比如`[3, 30]`，按照字典序是`[30,3]`，但是显然`303 < 330`，所以需要对这种情况做出调整。
调整的方式就是比较这两种排列的大小，即比较\`${b}${a}\`和\`${a}${b}\`的大小。实际上这里还需要比较严密的数学证明，就不详述了。
最后还要考虑全 0 时的处理，直接返回 0 就可以。

```js
/**
 * @param {number[]} nums
 * @return {string}
 */
var largestNumber = function (nums) {
  if (+nums.join("") === 0) return "0";
  nums.sort((a, b) => {
    return +`${b}${a}` - +`${a}${b}`;
  });
  return nums.join("");
};
```

## 优势洗牌

https://leetcode.cn/problems/advantage-shuffle

> 给定两个大小相等的数组  nums1  和  nums2，nums1  相对于 nums  的优势可以用满足  nums1[i] > nums2[i]  的索引 i  的数目来描述。
> 返回 nums1  的任意排列，使其相对于 nums2  的优势最大化。
> 示例 1：
> 输入：nums1 = [2,7,11,15], nums2 = [1,10,4,11]
> 输出：[2,11,7,15]

这道题其实就是田忌赛马的加强版，即调整 nums1 的顺序，使得比较的总次数中 nums1[i] > nums2[i]  的次数占更多。

那么就可以按照田忌赛马的思路：

1. 首先将两个数组排序，然后依次比较。这里采用倒序排序，正序也可以。但是结果不能直接按照排序的结果来，因为本来 nums1 的顺序就应该是对应 nums2 的每个元素的，因此不能直接对 nums2 排序，具体操作下面解释
2. 依次比较两个数组的对应值：

- 如果 nums1[i] > nums2[i]，即比得过，那么就比，即 res[i] = nums1[i]
- 如果比不过，那么就选一个送人头的。为了使比不过选择最小的造成的损失最小，应该每次都选最小的那个数，即倒序，开始选

有了思路，接下来就是很多细节问题。这道题真正的难点就是细节：

1. 上面说不能直接对 nums2 排序，因此可以不对 nums2 直接操作，而是取它的序号，然后将序号按照原数组从大到小排序。

```
就是这个意思：
比如说原数组  [20,32,8,12]
排序之后  [32,20,12,8]
把每个数在原数组中的序号提出来 [1,0,3,2]
接下来就用这个数组去比。比如 nums2[tmp[1]] 就表示从大到小第二个数，和直接排序一个效果
```

2. 依次比较应该采用双指针的形式，一边从左边取值，一边从右边取最小值，直到两个指针相遇为止。
3. 结果数组不能直接 push！！这也是很重要的一点。因为结果数组的顺序应该对应于 nums2，因此应该是`res[nums2[i]] = nums1[i]`这样的形式，而不是直接通过 push、unshift 的形式放入值。

代码如下：

```js
var advantageCount = function (nums1, nums2) {
  const res = [];
  nums1.sort((a, b) => b - a);

  const tmpNums2 = new Array(nums2.length).fill(0).map((num, index) => index);
  tmpNums2.sort((a, b) => nums2[b] - nums2[a]); // 给nums2的序号数组排序

  let right = nums1.length - 1;
  let left = 0;
  let j = 0;
  while (left <= right) {
    // 注意nums1的数组取数和nums2的应该是独立的，不管比不比的过，nums2应该始终向前走
    // 因此再设置一个j，j始终自增
    // left和right只在nums1中生效
    const index2 = tmpNums2[j++];
    if (nums1[left] > nums2[index2]) {
      // 比得过，就选这个数，放到结果数组对应nums2的位置上
      res[index2] = nums1[left++];
    } else {
      // 比不过，选最小的
      res[index2] = nums1[right--];
    }
  }
  return res;
};
```

## O(1) 时间插入、删除和获取随机元素

https://leetcode.cn/problems/insert-delete-getrandom-o1/

> 实现一个类，能在 O(1) 时间插入、删除和获取随机元素

这道题的关键是在 O(1)时间同时完成插入删除和随机获取。
如果只是前两个，可以采取哈希表的形式，因为哈希表的查找、插入、删除、修改都是 o(1)的。但是问题在于哈希表不连续，并且可能会有哈希冲突带来的更复杂的数据结构，因此不能保证在获取随即元素时的真随机。因此仍然需要一个紧凑的数组，同时关键在于如何实现删除的 o(1)。思路是这样：

- 如果我们想在 O(1) 的时间删除数组中的某一个元素 val，可以先把这个元素交换到数组的尾部，然后再 pop 掉，具体流程为：
  1. 首先通过 valToIndex 找到要删除元素 val 对应的 index
  2. 通过交换函数，把该元素和最后一个元素交换位置。
  3. 改变刚刚最后一个元素在 valToIndex 中的 index 值，即相当于把最后一个元素的 index 改为删掉的这个元素
  4. pop
- 设置一个 Map valToIndex 来记录每个元素值对应的索引，用于删除元素时“找到”对应元素的位置，进行交换等步骤。因为只有通过索引直接取数的方式才能在 O(1)时间内找到要删除的数字，否则通过 indexOf 等查找方法肯定是不能满足时间要求的。

代码细节比较多，如下：

```js
class RandomizedSet {
  constructor() {
    this.numsSet = [];
    this.valToIndex = new Map();
    this.len = 0;
  }

  swap(i, j) {
    [this.numsSet[i], this.numsSet[j]] = [this.numsSet[j], this.numsSet[i]];
  }

  insert(val) {
    if (this.valToIndex.has(val)) return false;
    this.numsSet.push(val);
    this.valToIndex.set(val, this.len++);
    return true;
  }

  remove(val) {
    if (!this.valToIndex.has(val)) return false;
    const index = this.valToIndex.get(val); // 找到对应的值的索引
    if (index !== this.numsSet.length - 1) {
      const lastVal = this.numsSet[this.numsSet.length - 1]; // 最后一个值，用于找最后一个值对应的索引
      this.swap(index, this.numsSet.length - 1); // 交换最后一个和当前
      this.valToIndex.set(lastVal, index); // 交换后，把原先最后一个值对应的索引改成该值的索引
    }
    this.valToIndex.delete(val); // 删除该值
    this.numsSet.pop();
    this.len--;
    return true;
  }

  getRandom() {
    return this.numsSet[Math.floor(Math.random() * this.len)];
  }
}
```

## 分割数组为连续子序列

https://leetcode.cn/problems/split-array-into-consecutive-subsequences/

> 给你一个按升序排序的整数数组 num（可能包含重复数字），请你将它们分割成一个或多个长度至少为 3 的子序列，其中每个子序列都由连续整数组成。
> 如果可以完成上述分割，则返回 true ；否则，返回 false 。
> 示例 1：
> 输入: [1,2,3,3,4,5]
> 输出: True
> 解释:
> 你可以分割出这样两个连续子序列 :
> 1, 2, 3
> 3, 4, 5

这道题其实不是常规的分割数组的题目，因为预先并不知道能分成几组，也不能通过上面讲的“装桶”算法解决。

这道题的思路其实可以借鉴到其他子序列问题的，即，通过一个 map 统计序列中的数字数量，当需要一个数字时直接到 map 中查找，而不是遍历数组。
比如说现在有一个数字 3，希望构成一个序列[3,4,5]，那么就可以查找 map[3]、map[4]、map[5]是否都大于 0（即还有几个能用的数字），如果能用就取出来用，不行就考虑其他方法。这种方式就不用考虑数字具体接到了哪个序列后面或者形成了什么样的序列，只需要考虑整个数组的需求关系

现在来说这道题的思路。对于每一个数字 num，有两种情况：

1. 当前元素 num 自成一派，「以自己开头」构成一个长度至少为 3 的序列。
   比如输入 nums = [1,2,3,6,7,8]，遍历到元素 6 时，它只能自己开头形成一个符合条件的子序列 [6,7,8]。
2. 当前元素 num 接到已经存在的子序列后面。
   比如输入 nums = [1,2,3,4,5]，遍历到元素 4 时，它只能接到已经存在的子序列 [1,2,3] 后面。它没办法自成开头形成新的子序列，因为少了个 6。

并且每个元素都应该优先判断自己是否能够接到其他序列后面。

我们可以设置两个 map，

- appear 用于记录当前序列中的每个数字的剩余数量，即这个数字还有几个没有被合并到其他序列中去，初始就是每个数字出现次数
- need 用于记录已经成型的子序列还需要哪个元素接到后面，比如现在已经有两个子序列 [1,2,3,4] 和 [2,3,4]，那么 need[5] 的值就应该是 2，说明对元素 5 的总需求为 2。

代码如下：

```js
var isPossible = function (nums) {
  const appear = [];
  const need = [];

  // appear初始就是每个数字出现次数
  nums.forEach((num) => {
    appear[num] = appear[num] ? appear[num] + 1 : 1;
  });

  for (let num of nums) {
    if (appear[num] === 0) continue;
    if (need[num] > 0 && need.length) {
      // 先判断能不能接在后面
      // 如果当前数字恰好被其他序列需要，就用上
      appear[num]--;
      need[num]--;
      need[num + 1] = need[num + 1] ? need[num + 1] + 1 : 1; // 把当前序列的需要向后移动一个
    } else if (appear[num] > 0 && appear[num + 1] > 0 && appear[num + 2] > 0) {
      // 如果要新形成一个序列，就把后面两个数都用上
      appear[num]--;
      appear[num + 1]--;
      appear[num + 2]--;
      need[num + 3] = need[num + 3] ? need[num + 3] + 1 : 1; // 需要num+2后面的数
    } else {
      return false;
    }
  }
  return true;
};
```

## 煎饼排序

https://leetcode.cn/problems/pancake-sorting/

（题目太长了这里就不写了）
其实就是这样一件事，和汉诺塔很像：
![](https://pic.imgdb.cn/item/62e8e62216f2c2beb1ee7c2d.jpg)

这个题的思路就是：
每次选择最大的一个元素，先把它翻到最上面，再翻到最下面去，也就是相当于把数组从大到小排序。
但是这个“最大的元素”并不能每次都是整个数组最大的，显然第一次放好最大元素之后，后面就应该选取第二大的，然后是第三大，以此类推。
这个思路其实可以用一个倒序的循环，这里用递归更清晰一些，其实就是在每次缩小排序的范围。

代码如下：

```js
var pancakeSort = function (arr) {
  const findMax = (nums) => nums.indexOf(Math.max(...nums));
  const reversePart = (nums, start, end) => {
    // 翻转函数
    let i = start;
    let j = end;
    while (i <= j) {
      [nums[i], nums[j]] = [nums[j], nums[i]];
      i++;
      j--;
    }
  };
  const kVals = [];
  function dfs(end) {
    if (end === 0) {
      return;
    }
    const maxIndex = findMax(arr.slice(0, end + 1)); // 先找到范围内最大元素的位置
    if (maxIndex !== end) {
      reversePart(arr, 0, maxIndex); // 翻转到顶层
      kVals.push(maxIndex + 1); // 翻转一次，记录翻转的位置
      reversePart(arr, 0, end); // 反转到底层，即全部翻转一次
      kVals.push(end + 1);
    }
    dfs(end - 1); // 范围缩小一个，相当于最后一个已经排好了
  }
  dfs(arr.length - 1); // 从整个数组开始
  return kVals;
};
```

## 括号问题

括号问题的解决无非就是两种方式：栈和数量统计。

栈解决就是“左括号入栈，右括号出栈”的思路。这里说一个新的思路，就是数量统计。

比如这一类问题的最简单问题<a href="https://leetcode.cn/problems/valid-parentheses/">有效的括号</a>，可以设置一个变量 need，用于表示需要的右括号数量；每遍历一个左括号，就 need++。如果最后 need 不为 0，就说明不能完全匹配。当然设置变量表示还未匹配的左括号数量也可以。

```js
let left = 0; // 未匹配的左括号数量
for (let i = 0; i < str.length; i++) {
  if (s[i] == "(") {
    left++;
  } else {
    // 遇到右括号
    left--;
    // 右括号太多
    if (left === -1) return false;
  }
}
// 是否所有的左括号都被匹配了
return left === 0;
```

这个方法就可以解决[使括号有效的最少添加](https://leetcode.cn/problems/minimum-add-to-make-parentheses-valid/)。
这道题就是统计匹配完成后还剩几个，那么就可以直接套用上面的方法。
但是还有一点要注意的，就是右括号数量可能多于左括号，并且这题不是到这一步就结束的，需要遍历完所有括号，得出没匹配的还有几个。
因此需要两个变量，分别统计需要的左括号和右括号数量；如果需要的右括号小于 0 了，那就还需要一个左括号；最后的结果应该是两者之和

```js
var minAddToMakeValid = function (s) {
  let needLeft = 0;
  let needRight = 0;
  for (let str of s.split("")) {
    if (str === "(") {
      needRight++;
      // 这里不需要needLeft--，因为left是主动增加的，这一步不会出现-1
    } else {
      needRight--;
      if (needRight === -1) {
        needLeft++;
        needRight = 0;
      }
    }
  }
  return needLeft + needRight;
};
```

再改进一下就可以解决另一道题[平衡括号字符串的最少插入次数](https://leetcode.cn/problems/minimum-insertions-to-balance-a-parentheses-string/)。
这道题无非就是一个左括号对应两个 needRight，**并且右括号的数量（即 needRight）应该是偶数**，如果不是就需要再添加一个左括号（不能删除，因为题目要求是插入次数）

```js
var minInsertions = function (s) {
  let needLeft = 0; // 需要的左括号数量
  let needRight = 0; // 需要的右括号数量
  for (let i = 0; i < s.length; i++) {
    if (s[i] === "(") {
      needRight += 2;
      if (needRight % 2) {
        needLeft++;
        needRight--;
      }
    } else {
      needRight--;
      if (needRight === -1) {
        needLeft++;
        needRight = 1;
      }
    }
  }
  return needLeft + needRight;
};
```

## 区间问题（区间的重叠等）

### 会议室 II（扫描线）

https://leetcode.cn/problems/meeting-rooms-ii/

> 给你一个会议时间安排的数组 intervals ，每个会议时间都会包括开始和结束的时间 intervals[i] = [starti, endi] ，返回 所需会议室的最小数量 。
> 示例 1：
> 输入：intervals = [[0,30],[5,10],[15,20]]
> 输出：2

这道题看着和前面的区间类问题很像，但是这道题其实是求最大的重叠数量，也就是从开始到结束的最多重叠区间数量。
暴力法很容易想到，就是遍历每一个整数时间，检查这个时间经过了多少个区间（`i >= start && i <= end`）。
还可以采用之前的区间问题的解决方法，即先按照 end 排序，每次更新 minEnd 时计算穿过的数量（即重叠了应该删除的数量），最后得到最大值即可。

但是这里有一个更好的方法，即扫描线，可以参考这张图：

![](https://pic.imgdb.cn/item/62e8fd6016f2c2beb10ae149.jpg)

红色的点代表每个会议的开始时间点，绿色的点代表每个会议的结束时间点。

现在假想有一条带着计数器的线，在时间线上从左至右进行扫描，每遇到红色的点，计数器 count 加一，每遇到绿色的点，计数器 count 减一：
![](https://pic.imgdb.cn/item/62e8fd6a16f2c2beb10aea83.jpg)

这样一来，每个时刻有多少个会议在同时进行，就是计数器 count 的值，count 的最大值，就是需要申请的会议室数量。

具体的代码实现，需要先对开始和结束的时间进行记录和排序，然后用双指针比较每一项，如果`begin<end`，就给 count++，反之就 count--。

```
比如数组[[0,30],[5,10],[15,20]]
得到 begins = [0,5,15]
     ends = [10,20,30]

我们模拟一个从小到大走的扫描线，那肯定是先走两个数组中较小的，即按照0、5、10、15、20、30的顺序。
```

代码：

```js
var minMeetingRooms = function (intervals) {
  const begins = [];
  const ends = [];
  for (let [begin, end] of intervals) {
    begins.push(begin);
    ends.push(end);
  }
  begins.sort((a, b) => a - b);
  ends.sort((a, b) => a - b);
  let i = 0;
  let j = 0;
  let count = 0;
  let res = 0;
  while (i < begins.length && j < ends.length) {
    if (begins[i] < ends[j]) {
      i++;
      count++;
    } else {
      count--;
      j++;
    }
    res = Math.max(res, count);
  }

  return res;
};
```

### 合并区间

https://leetcode.cn/problems/merge-intervals/

> 以数组 intervals 表示若干个区间的集合，其中单个区间为 intervals[i] = [starti, endi] 。请你合并所有重叠的区间，并返回   一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间  。
> 示例 1：
> 输入：intervals = [[1,3],[2,6],[8,10],[15,18]]
> 输出：[[1,6],[8,10],[15,18]]
> 解释：区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].

双指针比较，然后原地替换并删除多于的。

```js
/**
 * @param {number[][]} intervals
 * @return {number[][]}
 */
var merge = function (intervals) {
  intervals.sort((a, b) => a[0] - b[0]);
  const res = [];
  for (let i = 0; i < intervals.length; i++) {
    while (intervals[i + 1] && intervals[i][1] >= intervals[i + 1][0]) {
      intervals[i] = [
        intervals[i][0],
        Math.min(intervals[i + 1][1], intervals[i][1]),
      ];
      intervals.splice(i + 1, 1);
    }
  }
  return intervals;
};
```

### 区间列表的交集

https://leetcode.cn/problems/interval-list-intersections/

> 给定两个由一些 闭区间 组成的列表，firstList 和 secondList ，其中 firstList[i] = [starti, endi] 而  secondList[j] = [startj, endj] 。每个区间列表都是成对 不相交 的，并且 已经排序 。
> 返回这 两个区间列表的交集 。
> 形式上，闭区间  [a, b]（其中  a <= b）表示实数  x  的集合，而  a <= x <= b 。
> 两个闭区间的 交集 是一组实数，要么为空集，要么为闭区间。例如，[1, 3] 和 [2, 4] 的交集为 [2, 3] 。

这道题的解法和会议室 II 的思路很像。
考虑把所有的 start 和所有的 end 单独放入一个数组并排序，然后用两个指针分别在两个数组中遍历；

- 设置一个统计重叠个数的变量 cnt。每经过一个 start，cnt 就++，相应的经过一个 end 就给 cnt--。如果 cnt 加了 1 之后等于 2，就说明已经是重叠区域了，考虑记录开头结尾。
- 如果 start[i] < end[j]，那就给 i++，并给 cnt++；如果 cnt 等于 2，就把当前 start[i]作为一个重叠区间的起始。
- 如果 start[i] > end[j]，那就给 j++，同时 cnt--。如果 cnt 等于 1，就结束当前正在记录的重叠区间，把 end[j]作为该重叠区间的结尾。

还要注意，这道题还有一个特殊的地方，即存在开头结尾相同的区间（长度为 0）.那么当 start[i] === end[j]时，记录下来即可。

代码如下：

```js
var intervalIntersection = function (firstList, secondList) {
  const intervalsBegin = [];
  const intervalsEnd = [];
  for (const [start, end] of firstList) {
    intervalsBegin.push(start);
    intervalsEnd.push(end);
  }
  for (const [start, end] of secondList) {
    intervalsBegin.push(start);
    intervalsEnd.push(end);
  }
  intervalsBegin.sort((a, b) => a - b);
  intervalsEnd.sort((a, b) => a - b);
  const res = [];
  const curr = [];
  let cnt = 0; // 记录当前有几个重叠，可能为0 1 2
  let i = 0;
  let j = 0;
  while (j < intervalsEnd.length) {
    if (intervalsBegin[i] < intervalsEnd[j] && i < intervalsBegin.length) {
      if (++cnt === 2) {
        curr[0] = intervalsBegin[i];
      }
      i++;
    } else if (
      intervalsBegin[i] > intervalsEnd[j] ||
      i >= intervalsBegin.length
    ) {
      if (--cnt === 1) {
        curr[1] = intervalsEnd[j];
        res.push([...curr]);
        curr.length = 0;
      }
      j++;
    } else if (intervalsBegin[i] === intervalsEnd[j]) {
      res.push([intervalsBegin[i], intervalsEnd[j]]);
      i++;
      j++;
    }
  }
  return res;
};
```

### 无重叠区间

https://leetcode.cn/problems/non-overlapping-intervals/

> 给定一个区间的集合  intervals ，其中 intervals[i] = [starti, endi] 。返回 需要移除区间的最小数量，使剩余区间互不重叠  。
> 示例 1:
> 输入: intervals = [[1,2],[2,3],[3,4],[1,3]]
> 输出: 1
> 解释: 移除 [1,3] 后，剩下的区间没有重叠。

这道题有一种解法是之前说过的，就是先按照 start 排序，再依次遍历，把每个 start < 前一个 end 的数组删掉，最后剩下的就是无重叠区间的个数。

这里提供另外一种解法，更像贪心的思路，即按照 end 排序，大概是这样：
![](https://labuladong.github.io/algo/images/interval/1.gif)

也就是说，每次删除的是 start 小于最小 end 的区间，逐个更新最小 end，最后就可以得到结果。

```js
var eraseOverlapIntervals = function (intervals) {
  if (intervals.length === 1) return 0;
  intervals.sort((a, b) => a[1] - b[1]); // 按照end排序
  let minEnd = intervals[0][1];
  let cnt = 0;
  for (let i = 1; i < intervals.length; i++) {
    const [start, end] = intervals[i];
    if (start < minEnd) cnt++; // 如果一个区间的start小于minEnd，就删掉这个区间
    else minEnd = end; // 更新minEnd
  }
  return cnt;
};
```

这个思路还可以用于解决`https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/`这道题，其实都是一个思路的问题。

## 最小栈

https://leetcode.cn/problems/min-stack/

核心思想是创建一个辅助栈，每次 push 时也向辅助栈中放入值，有两种情况：

1. push 的参数（要放入的值）比当前辅助栈栈顶（即当前的最小值）小，那就放入 push 的参数
2. 要放入的值大于等于辅助栈顶，那就**再放入一次栈顶元素**。注意这一步是最重要的

最核心的思想就是第二步里再放入一次栈顶元素。这样每次 pop 主栈时，把辅助栈也 pop，就会出现两种情况：

1. 弹出的元素是最小值，也就意味着主栈和辅助栈栈顶元素都是该最小值，这时候都弹出即可
2. 弹出元素不是最小值，那么此时辅助栈中对应位置的元素就还是当前的最小值的重复（上面第二步重复放入的），弹出之后并不影响最小值。

```js
class MinStack {
  constructor() {
    this.stack = [];
    this.minHistory = [Infinity];
  }
  push(val) {
    this.minHistory.push(
      Math.min(val, this.minHistory[this.minHistory.length - 1])
    );
    this.stack.push(val);
  }
  pop() {
    this.minHistory.pop();
    return this.stack.pop();
  }
  top() {
    return this.stack[this.stack.length - 1];
  }
  getMin() {
    return this.minHistory[this.minHistory.length - 1];
  }
}
```

## 搜索二维矩阵（Z 字形查找）

https://leetcode.cn/problems/search-a-2d-matrix-ii/

> 编写一个高效的算法来搜索  m x n  矩阵 matrix 中的一个目标值 target 。该矩阵具有以下特性：
> 每行的元素从左到右升序排列。
> 每列的元素从上到下升序排列。

这道题是一个技巧题，排除暴力法还有两种方法：

1. 二分查找。即遍历每个行，对每行的数组进行二分查找
2. Z 字形查找。初始化取数组右上角的元素`[x,y]`为起点，然后和 target 比较，有三种情况：

- 该元素 === target，找到
- 该元素 > target，说明这一列所有的**下面的**元素都大于 target（因为从上向下单调递增），因此不用考虑这一列剩下的元素了，直接向内挪动一列，即 y - 1
- 该元素 < target，说明这一行所有的**左边的**元素都小于 target（因为从右向左单调递减），因此不用考虑这一行剩下的元素了，直接向下挪动一行，即 x + 1。（注意这里不用考虑该元素右边的部分，因为已经被列过滤掉了。上一步同理）

```js
/**
 * @param {number[][]} matrix
 * @param {number} target
 * @return {boolean}
 */
var searchMatrix = function (matrix, target) {
  const [m, n] = [matrix[0].length, matrix.length];
  let [x, y] = [0, m - 1];
  while (y >= 0 && x < n) {
    const num = matrix[x][y];
    if (num === target) return true;
    if (num > target) y--;
    else if (num < target) x++;
  }
  return false;
};
```

## 下一个排列 & 下一个更大元素 III

这两道题的方法几乎一样，题目要求也只有一点差别

https://leetcode.cn/problems/next-greater-element-iii/
https://leetcode.cn/problems/next-permutation/

下一个更大元素的前两道都是单调栈问题，但是这道其实和单调栈没啥关系，而是采用下一个排列这道题的方法。

题干基本都是这样：

> 整数数组的一个 排列   就是将其所有成员以序列或线性顺序排列。
> 例如，arr = [1,2,3] ，以下这些都可以视作 arr 的排列：[1,2,3]、[1,3,2]、[3,1,2]、[2,3,1] 。
> 整数数组的 下一个排列 是指其整数的下一个字典序更大的排列。更正式地，如果数组的所有排列根据其字典顺序从小到大排列在一个容器中，那么数组的 下一个排列 就是在这个有序容器中排在它后面的那个排列。如果不存在下一个更大的排列，那么这个数组必须重排为字典序最小的排列（即，其元素按升序排列）。
> 例如，arr = [1,2,3] 的下一个排列是 [1,3,2] 。
> 类似地，arr = [2,3,1] 的下一个排列是 [3,1,2] 。
> 而 arr = [3,2,1] 的下一个排列是 [1,2,3] ，因为 [3,2,1] 不存在一个字典序更大的排列。
> 给你一个整数数组 nums ，找出 nums 的下一个排列。

> 给你一个正整数 n ，请你找出符合条件的最小整数，其由重新排列 n 中存在的每位数字组成，并且其值大于 n 。如果不存在这样的正整数，则返回 -1 。

也就是给一个数字或者数字数组，让你找出这个数组经过重新排列后，新的数组组成的数字刚好大于当前数字，并且是大于当前数字的值中的最小值。
比如说 123 的下一个数应该是 132，然后是 213、231、312、321，依次类推。

> 唯一的区别是下一个排列这道题有一个循环，即 321 下一个是 123，而下一个更大元素没有这种要求

方法是这样：

1. 首先从后（len-2）向前遍历 i，找到一个前小后大的位置（即 nums[i] < nums[i+1]）
2. 在从后（len-1）向前找 j，找到第一个大于 nums[i]的位置
3. 交换 i 和 j，然后令`left = i + 1`，`right = len - 1`，两个指针同时向中间走，每次交换两个指针对应的值。这一步也就是在排序 i 右边的部分，因为上一步交换了 ij 之后，i 右边部分一定是降序；通过交换的方式，就可以重新转为升序

```
以排列 [4,5,2,6,3,1]为例：

我们能找到的符合条件的一对「较小数」与「较大数」的组合为 2 与 3，满足「较小数」尽量靠右，而「较大数」尽可能小。

当我们完成交换后排列变为 [4,5,3,6,2,1]，此时我们可以重排「较小数」右边的序列，序列变为 [4,5,3,1,2,6]
```

代码如下：

```js
var nextPermutation = function (nums) {
  let i = nums.length - 2;
  while (i >= 0 && nums[i] >= nums[i + 1]) {
    i--;
  }
  if (i >= 0) {
    let j = nums.length - 1;
    while (j >= 0 && nums[j] <= nums[i]) {
      j--;
    }
    [nums[i], nums[j]] = [nums[j], nums[i]];
  }
  let l = i + 1;
  let r = nums.length - 1;
  while (l < r) {
    [nums[l], nums[r]] = [nums[r], nums[l]];
    l++;
    r--;
  }
};
```

如果是下一个更大元素，其实就是当没找到一个前小后大的元素（说明此时已经是最大值）时直接返回-1 即可

```js
var nextGreaterElement = function (n) {
  const nums = n.toString().split("");
  let i = nums.length - 2;
  while (i >= 0 && nums[i] >= nums[i + 1]) {
    i--;
  }
  if (i < 0) return -1; // 这里直接返回
  let j = nums.length - 1;
  while (j >= 0 && nums[j] <= nums[i]) {
    j--;
  }
  [nums[i], nums[j]] = [nums[j], nums[i]];
  let left = i + 1;
  let right = nums.length - 1;
  while (left < right) {
    [nums[left], nums[right]] = [nums[right], nums[left]];
    left++;
    right--;
  }
  return +nums.join("") > 2 ** 31 - 1 ? -1 : +nums.join("");
};
```

## 根据身高重建队列

https://leetcode.cn/problems/queue-reconstruction-by-height/

> 假设有打乱顺序的一群人站成一个队列，数组 people 表示队列中一些人的属性（不一定按顺序）。每个 people[i] = [hi, ki] 表示第 i 个人的身高为 hi ，前面 正好 有 ki 个身高大于或等于 hi 的人。
> 请你重新构造并返回输入数组  people 所表示的队列。返回的队列应该格式化为数组 queue ，其中 queue[j] = [hj, kj] 是队列中第 j 个人的属性（queue[0] 是排在队列前面的人）。
> 示例 1：
> 输入：people = [[7,0],[4,4],[7,1],[5,0],[6,1],[5,2]]
> 输出：[[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]]

> 套路）：一般这种数对，还涉及排序的，根据第一个元素正向排序，根据第二个元素反向排序，或者根据第一个元素反向排序，根据第二个元素正向排序，往往能够简化解题过程。

这道题比较特殊，之前没有做过类似的题目。看了题解才恍然大悟。其实题解的方法还是有不小的技巧性的

基本思路是创建一个新的 res 数组，然后往这个数组中填值；填的依据有两个：

1. 如果当前数组长度小于当前的 person[1]（取 person 为 people 数组中的每项），那就 push 放入
2. 如果数组长度够长，即大于 person[1]，就放到 person[1]对应的位置。比如[5,2]就放在序号为 2 的位置上。

但是这样填有一个先前条件，就是先给数组排序，具体来说就是**根据第一个元素正向排序，根据第二个元素反向排序**。
原因是，按照 person[0] 进行降序排序，对于每个元素，在其之前的元素的个数，就是大于等于他的元素的数量；
而按照 person[1]升序，也是为了保证正确性。

> 举个例子，在身高一样，k 不一样的时候，譬如[5,2]和[5,3], 对于最后排完的数组，[5,2]必然在[5,3]的前面。所以如果遍历的时候[5,3]在前面，等它先插入完，这个时候它前面会有 3 个大于等于它的数组对，遍历到[5,2]的时候，它必然又会插入[5,3]前面（因为它会插入链表索引为 2 的地方），这个时候[5,3]前面就会有 4 个大于等于它的数组对了，这样就会出错。因此必须按照 person[1]从小到大的顺序插入

代码很简单，就是创建数组并插入，难点就在于上面的思路。

```js
var reconstructQueue = function (people) {
  people.sort((a, b) => {
    if (a[0] === b[0]) {
      return a[1] - b[1];
    } else return b[0] - a[0];
  });
  const res = [];
  for (const person of people) {
    if (person[1] >= res.length) {
      res.push(person);
    } else {
      res.splice(person[1], 0, person);
    }
  }
  return res;
};
```

# 数学题

## 位运算

常用的位运算操作就是与`&`、或`|`、异或`^`这三个。

- 与：同为 1 结果为 1，其他都为 0。比如`110 & 010 = 010`
- 或：只要有一个 1 结果就为 1，只有同为 0 结果才为 0。`110 | 010 = 110`
- 异或：相同为 0，**不同为 1**，`110 ^ 010 = 100`

> js 可以通过前缀 0b 或 0B 表示二进制数字。但是只能作为字面量形式存在，如果作为变量，实际上还是在以十进制的形式计算。
> 但是这并不影响左移右移、与或异或等操作。两个十进制数进行与操作，本质上还是对他们的二进制形式操作，只是得到的结果还是十进制而已。因此大多数位运算的题目还是可以做的。

常见的位运算操作有：

1. 根据数字的二进制值判断性质。

比如判断一个数是不是 2 的幂，一个数如果是 2 的指数，那么它的二进制表示一定只含有一个 1。因此把这个数转为 2 进制数 1 的个数即可。

2. `n & (n-1)`，这个操作是算法中常见的，作用是消除数字 n 的二进制表示中的最后一个 1

![](https://pic.imgdb.cn/item/62e78ce38c61dc3b8edfce13.jpg)

这个操作主要用于二进制“数 1”，即从后向前依次去掉最后一个 1（全部去掉这个数就成为 0 了），顺便数出来 1 的个数。当然直接数也是可以的，只是这种方法相较于字符串要快的多。
比如力扣第 136 题<a href="https://leetcode.cn/problems/number-of-1-bits/">位 1 的个数</a>，就可以逐个去掉最后一个 1，直到该数变成 0 为止。

3. `a ^ a = 0`，即异或，一个数和它本身做异或运算结果为 0，即 `a ^ a = 0`；一个数和 0 做异或运算的结果为它本身，即 `a ^ 0 = a`。并且异或操作遵循结合律、交换律，可以无视顺序。

两个数字异或的结果是保留两个数字的位中不同的位。比如`10010110 ^ 11001010 = 10100011`，结果的这个数字的每一位1都是两个数不同的位。比如[汉明距离](https://leetcode.cn/problems/hamming-distance)这道题，要统计两个数的每个位不同的，即我们将两个数异或之后数1的个数就好了。

还有一种题型是把一组数字异或起来，往往剩下的那个数字就是一个特殊值。比如下面的这道题就是。
还有一道下面的题的变种，即<a href="https://leetcode.cn/problems/missing-number/">丢失的数字</a>这道题

### 只出现一次的数字

https://leetcode.cn/problems/single-number/

> 给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

这道题的暴力解法很好想到，给数组排序，如果一个数既不和前面的相等又不和后面的相等，就说明是唯一的数字。

但是这道题有一个更 nb 的解法，就是利用异或。

**一个数和它本身做异或运算结果为 0，即 `a ^ a = 0`；一个数和 0 做异或运算的结果为它本身，即` a ^ 0 = a`**

所以我们就可以把数组内的所有数都异或起来（初始值为 0），相同的数无论顺序，最后都会会互相异或成为 0；因此最后剩下的数，就是只出现一次的数。

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
var singleNumber = function (nums) {
  return nums.reduce((a, b) => a ^ b, 0);
};
```

## 阶乘后的 0

https://leetcode.cn/problems/factorial-trailing-zeroes/

> 给定一个整数 n ，返回 n! 结果中尾随零的数量。

这道题肯定不能硬算，解法有且只有一个，只能记住：即，寻找小于等于 n 的 5 的因子数量。

> 首先，两个数相乘结果末尾有 0，一定是因为两个数中有因子 2 和 5，因为 10 = 2 x 5。
> 也就是说，问题转化为：n! 最多可以分解出多少个因子 2 和 5？
> 比如说 n = 25，那么 25! 最多可以分解出几个 2 和 5 相乘？这个主要取决于能分解出几个因子 5，因为每个偶数都能分解出因子 2，因子 2 肯定比因子 5 多得多。
> 25! 中 5 可以提供一个，10 可以提供一个，15 可以提供一个，20 可以提供一个，25 可以提供两个，总共有 6 个因子 5，所以 25! 的结果末尾就有 6 个 0。
> 现在，问题转化为：n! 最多可以分解出多少个因子 5？
> 难点在于像 25，50，125 这样的数，可以提供不止一个因子 5，怎么才能不漏掉呢？
> 这样，我们假设 n = 125，来算一算 125! 的结果末尾有几个 0：
> 首先，125 / 5 = 25，这一步就是计算有多少个像 5，15，20，25 这些 5 的倍数，它们一定可以提供一个因子 5。
> 但是，这些足够吗？刚才说了，像 25，50，75 这些 25 的倍数，可以提供两个因子 5，那么我们再计算出 125! 中有 125 / 25 = 5 个 25 的倍数，它们每人可以额外再提供一个因子 5。
> 够了吗？我们发现 125 = 5 x 5 x 5，像 125，250 这些 125 的倍数，可以提供 3 个因子 5，那么我们还得再计算出 125! 中有 125 / 125 = 1 个 125 的倍数，它还可以额外再提供一个因子 5。
> 这下应该够了，125! 最多可以分解出 25 + 5 + 1 = 31 个因子 5，也就是说阶乘结果的末尾有 31 个 0。

依照这个思路代码如下：

```js
/**
 * @param {number} n
 * @return {number}
 */
var trailingZeroes = function (n) {
  let sum = 0;
  let i = 1;
  while (n >= 5 ** i) {
    // 分别计算，n里有多少个5、多少个25、多少个125....，一直到大于了n为止
    // 25的倍数相当于两个5，因为之前5的倍数已经算过一次，所以25的倍数只需要再加一次就可以，后面的同理
    sum += Math.floor(n / 5 ** i);
    i++;
  }
  return sum;
};
```

---

这个题有一个衍生出来的问题。假如数字都是随机的，即任意的 n 个数相乘，怎么计算他们的乘积 0 的数量？

有一个结论：**乘积末尾 0 的数量取决于乘积中因子 2 和因子 5 的数量的较小值。**
其中因子指的是最多能除几次，比如 4 的 2 的因子数量为 2，因为它可以除两次 2；100 的 5 的因子为 2，因为它可以除两次 5。

```
举个例子：数组[5,2,3,50,4]
他们的每一项的因子5和因子2的数量分别为：
因子2：[0,1,0,1,2] 总数：4
因子5：[1,0,0,2,0] 总数：3

如果把他们全部乘起来，那么总的0的个数就为 min(2的因子数量, 5的因子数量) = 3
```

## （乘积中 0 的个数大于 x 的）连续子数组数量

> 给定一个数组，请你编写一个函数，返回元素乘积末尾零数量大于等于 x 的连续子数组数量。
> 数组元素均为不超过 10^9 的正整数
>
> 示例 1
>
> 输入例子：
> [5,2,3,50,4],2
>
> 输出例子：
> 6
>
> 例子说明：
> 共有以下 6 个合法连续子数组：
> [5,2,3,50]，乘积为 1500，末尾有 2 个零。
> [5,2,3,50,4]，乘积为 6000，末尾有 3 个零。
> [2,3,50]，乘积为 300，末尾有 2 个零。
> [2,3,50,4]，乘积为 1200，末尾有 2 个零。
> [3,50,4]，乘积为 600，末尾有 2 个零。
> [50,4]，乘积为 200，末尾有 2 个零。

这道题应该分为两个方面去考虑，即：

1. 怎么确定一组数的乘积中 0 的个数
2. 当找到一个符合条件的连续子数组之后，怎么找下一个或者计算剩下的还有多少个？

对于第一个方面，可以看上一道题，结论为一组数中**乘积末尾 0 的数量取决于乘积中因子 2 和因子 5 的数量的较小值**。
第二个方面，可以采取滑动窗口的方法。首先先增大右边界，当满足条件时计算数量；然后缩小左边界直到 0 的数量不足，再增大右边界，依次循环。

当一个数组满足条件时，对它任意的增大都满足条件（比如乘积 100，再乘多少个正整数都至少有 2 个 0）。因此一个子数组满足条件后，`数组长度 - 右边界的索引`就是其他子数组的数量。可以理解为是多乘了右边的数。
比如`[2,3,50]`满足条件，那么它右边还有一个数 4，如果乘上肯定满足条件。加上他自己，一共就是`5 - 3 = 2`种子数组数量。其他的类似。
不用考虑左边的情况，因为可能会算重复

代码如下：

```js
function get2And5Amount(a) {
  const arr = new Array(a.length)(2);
  for (let i = 0; i < this.n; ++i) {
    let temp = a[i];
    while (temp > 0 && temp % 2 === 0) {
      arr[i][0]++;
      temp /= 2;
    }
    temp = a[i];
    while (temp > 0 && temp % 5 === 0) {
      arr[i][1]++;
      temp /= 5;
    }
  }
  return arr;
}

function solution(a, x) {
  let ans = 0;
  let two_num = 0;
  let five_num = 0;
  let l = 0;
  let r = 0;
  // 滑动窗口
  while (r < a.length) {
    // 每次分别计算这个位置上2或5的因子数量
    two_num += arr[r][0];
    five_num += arr[r][1];
    while (Math.min(two_num, five_num) >= x) {
      // a.length - r表示包括当前子数组和当前子数组右边的所有子数组的数量
      ans += a.length - r;
      // 缩左边窗口
      two_num -= arr[l][0];
      five_num -= arr[l][1];
      ++l;
    }
    ++r;
  }
  return ans;
}
```

## 颠倒二进制位

https://leetcode.cn/problems/reverse-bits/

> 颠倒给定的 32 位无符号整数的二进制位。
>
> 示例 1：
>
> 输入：n = 00000010100101000001111010011100
> 输出：964176192 (00111001011110000010100101000000)
> 解释：输入的二进制串 00000010100101000001111010011100 表示无符号整数 43261596，
> 因此返回 964176192，其二进制表示形式为 00111001011110000010100101000000。

基本思路：把 n 循环和 1 与，每次把 n&1 的结果左移 31-i 位即可

即，比如 n 的从右往左第一位为 1，那么结果 res 的从右往左第 32 位（左移 31-0）就是 1；然后把这个值和下一个结果或即可。

需要注意的是，由于 js 的数字采用补码表示有符号整数，因此将左移右移操作分为了有符号左右移（`<<`和`>>`）和无符号左右移（`<<<`和`>>>`）。对于正数来说，两者没有区别；但是对于负数来说，有符号右移会保留符号位的值，而无符号右移则会将符号位一块右移，会使得原来的负数变成一个值非常大的正数。具体参考https://www.jianshu.com/p/588eb74b5a03

如果把一个负数无符号右移 0 位（即`n>>>0`），还会有特殊的情况。虽然位数没有变化，但是作为负数的补码被看作是作为正数的补码，这时数值就会发生巨大变化。

比如:

```js
-1 >>> 0; // 4294967295
// -1的补码
11111111111111111111111111111111;
// 无符号右移0位，位数不变，但是被看作是正数补码，即当做原码去计算
11111111111111111111111111111111; // 4294967295
```

所以这道题的要求是无符号整数的颠倒，即不能出现正数颠倒成负数的情况，也就是说对于最终的结果，应该在>>>0 一次；同理在循环过程中 n 的移位也应该是>>>1 而不是>>1，因为如果是后者，当 n 是负数时，有符号右移则会导致负数变成一个很大的正数。

```js
var reverseBits = function (n) {
  let res = 0;
  for (let i = 0; i < 32 && n > 0; i++) {
    res = res | ((n & 1) << (31 - i));
    n = n >>> 1;
  }
  return res >>> 0;
};
```

## IP 地址与 int 整数的转换

> 移位运算不需要考虑其的二进制结果。十进制数移位之后还是十进制数，虽然底层是根据二进制的逻辑来的，但是通过对十进制数的直接移位或者与或等操作，可以省去转为二进制字符串再操作的麻烦。

https://www.nowcoder.com/questionTerminal/66ca0e28f90c42a196afd78cc9c496ea

> ip 地址的每段可以看成是一个 0-255 的整数，把每段拆分成一个二进制形式组合起来，然后把这个二进制数转变成一个长整数。
> 举例：一个 ip 地址为 10.0.3.193
> 每段数字 相对应的二进制数
> 10 00001010
> 0 00000000
> 3 00000011
> 193 11000001
> 最终结果是 00001010 00000000 00000011 11000001,转换为 10 进制数就是：167773121
> 要求实现从十进制 ip 地址转到十进制数，和从十进制数转到 ip 地址两个功能。

这道题的笨办法就是利用 toString 的二进制字符串直接算。由于二进制转十进制比较简单，因此虽然逻辑麻烦一些但是整体不难：

```js
// 二进制转十进制数
const biToInt = (num) => {
  let sum = 0;
  for (let i = num.length - 1; i >= 0; i--) {
    sum += +num[i] * 2 ** (num.length - i - 1);
  }
  return sum;
};

const ipToInt = (ip) => {
  const ipVal = ip.split(".");
  const partlyArr = [];
  for (let ipStr of ipVal) {
    let biIpStr = (+ipStr).toString(2);
    while (biIpStr.length < 8) biIpStr = "0" + biIpStr;
    partlyArr.push(biIpStr);
  }
  const biIpNum = partlyArr.join("");
  console.log(biIpNum);
  return biToInt(biIpNum);
};

const intToIp = (num) => {
  let ipStr = (+num).toString(2);
  while (ipStr.length < 32) ipStr = "0" + ipStr;
  const ipValArr = [];
  for (let i = 0; i < 32; i += 8) {
    const n = biToInt(ipStr.slice(i, i + 8));
    ipValArr.push(n);
  }
  return ipValArr.join(".");
};
```

还有一个方法更合适，就是利用移位操作直接得到数字对应的二进制数表示的十进制数。这么说可能有点绕，其实就是直接让 IP 的四个数字分开进行左移；因为左移的结果还是十进制数，相当于将原数字转为位置合适的二进制数后再转回来，因此节省很多步骤。

```
10  ---> 00001010 ---> 167772160
```

具体来说，处在第一位的数字后面应该有 32-8=24 位，因此把第一个数左移 24 位就可以得到。比如把 10 左移 24 位就是 `00001010 00000000 00000000 00000000`，这个值正好处在它二进制表示时正确的位置。
相应的，后面三个数分别左移 16、8、0 位。最后四个值通过或（`|`）拼接在一起就可以得到结果。

![](https://pic1.imgdb.cn/item/633ad46116f2c2beb13cd29f.jpg)

```js
const ipToInt2 = (ip) => {
  const ipList = ip.split(".");
  return (
    (+ipList[0] << 24) | (+ipList[1] << 16) | (+ipList[2] << 8) | +ipList[3]
  );
};
```

如果从整数转回来，则需要通过和 255 与（`&`）的形式拆分每个部分。
原理大概是这样：

![](https://pic1.imgdb.cn/item/633ad49716f2c2beb13d2c07.jpg)

即，每次和 255 相与并右移 8 位，然后再相与，最后得到四个位置上的十进制数。原理就是 255 是后八位全为 1，和原数相与的话，相当于只留下原数的后八位，其他的都变为 0；这样就会分开得到四个位置上的值，再转化成十进制就是对应的 ip 了。

```js
const intToIp2 = (num) => {
  const res = [];
  for (let i = 0; i < 4; i++) {
    res.unshift(num & 255);
    num = num >> 8;
  }
  return res.join(".");
};
```

## 计数质数

https://leetcode.cn/problems/count-primes/

> 给定整数 n ，返回 所有小于非负整数 n 的质数的数量 。

首先先考虑怎么判断一个数 n 是不是质数：

```js
for (let i = 2; i < n; i++) {
  if (n % i === 0) return false;
}
```

即，从 2 开始到 n 的数，如果这之间有一个数能整除 n，那么 n 一定就不是质数。
但是这个明显有个问题，即因子会重复。比如：

```
n    i  n/i
12 = 2 * 6
12 = 3 * 4
...
12 = 4 * 3
12 = 6 * 2
```

显然这里 i 重复了，当 i=2 时已经得到了另一个因子为 6 的结果，那么 i 就不应该再遍历到 6。这个的分界线就是根号下 n，即`i < Math.sqrt(n)`

```js
for (let i = 2; i * i < n; i++) {
  if (n % i === 0) return false;
}
```

---

接下来就是判断 n 以内的数是不是正数：方法是采取一个数学方法，即`素数筛选法`。每次从`[2,n)`中去掉每个数的倍数，剩下的数就一定是质数。
比如先去掉 2 的倍数，再去掉 3 的，然后是 5、7、11 等等

![](https://labuladong.github.io/algo/images/prime/1.gif)

代码如下：

```js
var countPrimes = function (n) {
  if (n === 0 || n === 1) return 0;
  let sum = 0;
  const primes = new Array(n).fill(true);

  // 循环：从2开始一直到根号n的每个数的倍数都一定不是质数，把这些数都改为false
  // 外层循环：遍历从2开始的每个数
  for (let i = 2; i * i < n; i++) {
    // 这里和上面一样的逻辑，i只需要到根号n就够了
    if (primes[i]) {
      // 内层循环：找到这个数i的所有倍数，变成非质数
      // j从i的i倍开始，接下来是i的i+1倍、i+2倍等等一直到n
      for (let j = i * i; j < n; j += i) {
        // 这里j是从i*i开始的，因为：
        // 比如 n = 25，i = 5 时算法会标记 5 × 2 = 10，5 × 3 = 15 等等数字，但是这两个数字已经被 i = 2 和 i = 3 的 2 × 5 和 3 × 5 标记了。
        // 因此原本是2*i，改进称i*i更好一些
        primes[j] = false;
      }
    }
  }

  for (let prime of primes) {
    if (prime) sum++;
  }
  return sum;
};
```

## 水塘抽样算法

这个算法的题目参考https://leetcode.cn/problems/linked-list-random-node/，就是这个算法的实现。

概括一下其实就是解决这样一个问题：给你一个未知长度的链表，请你设计一个算法，只能遍历一次，随机地返回链表中的一个节点。

**结论是，当你遇到第 i 个元素时，应该有 1/i 的概率选择该元素，1 - 1/i 的概率保持原有的选择**。这样就可以一边遍历，一边保持随机性了。
举个例子，比如遍历到第二个元素，那就是 1/2 的概率选该元素，1-1/2 概率保持选第一个元素。
这个结论的证明需要数学方法，参考https://labuladong.github.io/algo/4/32/119/

代码实现上，为了实现`1/i`的概率选择，其实可以通过`Math.floor(Math.random() * i) === 0`的形式得到 1/i 的概率。
比如上面的例题，代码就是这个样子：

```js
class Solution {
  constructor(head) {
    this.head = head;
  }
  getRandom() {
    let p = this.head;
    let i = 0;
    let res = null;
    while (p) {
      i++;
      if (Math.floor(Math.random() * i) === 0) {
        // 1/i的概率选择新遍历到的元素
        res = p.val;
      }
      p = p.next;
    }
    return res;
  }
}
```

## 求 x 的平方根

> 给你一个非负整数 x ，计算并返回  x  的 算术平方根 。
> 由于返回类型是整数，结果只保留 整数部分 ，小数部分将被 舍去 。
> 注意：不允许使用任何内置指数函数和算符，例如 pow(x, 0.5) 或者 x \*\* 0.5 。

暴力法就不讲了，这里记一下官方题解的几种方法：

1. 二分查找：即找一个最大的 k，使得 $k^2 ≤ x$。左右边界分别设为 0 和 x/2，在这个范围中比较 mid 的平方 和 x，最后取到最大的 mid 即可
   这个思路有点类似于二分查找的应用的那几道题，即“平台”最大值，也就是找平台最右边的那个值

```js
var mySqrt = function (x) {
  let left = 0;
  let right = x;
  while (left <= right) {
    const mid = left + Math.floor((right - left) / 2);
    if (mid * mid === x) {
      return mid;
    } else if (mid ** 2 < x) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  return left - 1;
};
```

2. 牛顿迭代法：数学方法：

![](https://pic1.imgdb.cn/item/6333f16c16f2c2beb102ce93.jpg)
![](https://pic1.imgdb.cn/item/6333f1e816f2c2beb1035bad.jpg)

所以核心就是第二个图片的最后一个方程。迭代计算这个方程，直到`xi - xi+1`的值小于 $ 10 ^ {-7} $ 为止

```js
var mySqrt = function (x) {
  if (x === 0) return 0;
  let x0 = x;
  let xi = 0;
  while (true) {
    xi = (x0 + x / x0) / 2;
    if (x0 - xi < 0.0000001) return Math.floor(xi);
    x0 = xi;
  }
};
```

## 用 Rand7()实现 Rand10()

https://leetcode.cn/problems/implement-rand10-using-rand7/

题目很简单，就是给你一个`[1,7]`范围内生成随机数的函数 rand7，你需要利用这个函数实现一个从`[1,10]`范围内生成随机数的函数 rand10。

也就是说，要利用 rand7 的概率实现 rand10 要求的概率。

方法：获取两个 rand7 的数 a 和 b，那么可以生成 [1, 49] 之间的随机整数，们只用到其中的前 40 个用来实现 rand10，而拒绝剩下的 9 个数，即：

![](https://pic1.imgdb.cn/item/633adfe516f2c2beb155175a.jpg)

这时候每个数的概率就是 4/49，总范围为 40/49，即 1/10

从上图也可以看到 a 和 b 和 index 的关系，即假设 a 为行，b 为列，那么 a 行 b 列的那个数就是第`7*(a-1)+b`个数。我们只需要保证整个数的 index 小于 40（如果超过 40 就返回去继续选，因此是一个循环），然后将取到的`index % 10`即可。注意因为是取`[1,10]`的范围，因此最后的值还需要+1

```js
var rand10 = function () {
  let a, b, index;
  do {
    a = rand7();
    b = rand7();
    index = 7 * (a - 1) + b;
  } while (index > 40); // 注意这里是index > 40 ，不能只判断a*b > 40
  return (index % 10) + 1;
};
```

推广一下，可以应用到任意 randA 实现 randB，只要保证 `A ^ 2 > B`就可以。

## 丑数

丑数系列的题目有好几道，这里主要是丑数 I 和丑数 II

https://leetcode.cn/problems/ugly-number/
https://leetcode.cn/problems/ugly-number-ii/

> 丑数 就是只包含质因数 2、3 和 5 的正整数。给你一个整数 n ，请你判断 n 是否为 丑数 。如果是，返回 true ；否则，返回 false 。

判断一个数是不是丑数比较简单，即不断用这三个数去除这个数，最后得到的值是 1 就说明是丑数，否则就不是

```js
var isUgly = function (n) {
  if (n === 1) return true;
  if (n <= 0) return false;
  const res = [];
  const vals = [2, 3, 5];
  while (n > 1) {
    let tmp = n;
    for (const val of vals) {
      if (n % val === 0) {
        res.push(val);
        n /= val;
      }
    }
    if (n === tmp) return false;
  }
  return true;
};
```

然后是丑数 II，这道题其实和丑数 I 的解法完全不一样

> 给你一个整数 n ，请你找出并返回第 n 个 丑数 。
>
> 丑数 就是只包含质因数  2、3 和/或  5  的正整数。
> 示例 1：
> 输入：n = 10
> 输出：12
> 解释：[1, 2, 3, 4, 5, 6, 8, 9, 10, 12] 是由前 10 个丑数组成的序列。

显然 n 的值可能会很大，因此不能遍历判断一个数是不是丑数，只能从第一个丑数 1 开始逐个生成丑数，生成到 n 为止。

那怎么生成丑数呢？因为丑数是只包含质因数  2、3 和/或  5 的数，所以一个丑数的 2、3、5 倍也一定是丑数。即对于 2、3、5 三个基本数，他们的 n 倍（n>=1）也都是丑数；如果把这三个数的 n 倍按顺序组合起来，就是一个完整的丑数链

```
每一项都是一个丑数的2、3、5倍

1*2 -> 2*2 -> 3*2 -> 4*2 -> 5*2 -> 6*2 -> 8*2 ->... // 丑数的2倍

1*3 -> 2*3 -> 3*3 -> 4*3 -> 5*3 -> 6*3 -> 8*3 ->... // 丑数的3倍

1*5 -> 2*5 -> 3*5 -> 4*5 -> 5*5 -> 6*5 -> 8*5 ->... // 丑数的5倍

// 合并三个有序数组
```

但是这里还有一个问题，除了 1、2、3、5，其他的丑数预先都不知道，也就不能先计算一条链上的丑数，而是应该一起算三条链上的丑数，本链上计算的丑数将会给其他链用。

所以我们可以定义三个指针，这三个指针每个都是独立的，都是从 1 开始的连续增长，分别表示三个链内的倍数值（也就是丑数值）。这三个指针都是在最后的丑数链表上移动的，即他们指向的值就是丑数；然后，在每一个链的计算中，每条链的下一个值就应该是`n * ugly[pn++]`（n=2 或 3 或 5），这样就可以依次推导出新的丑数值。

在上面三个的基础上，每次选取计算出的最小的值作为当前位置上的丑数值。选的是哪条链上的计算结果，哪条链上的指针 pn 就向前走一步。

代码：

```js
var nthUglyNumber = function (n) {
  let p2 = 1; // 这里没有采取真正的链表或数组，只是一个指针
  let p3 = 1;
  let p5 = 1;
  let head2 = 1; // head表示的是本条链上的最末段的值，其实不一定需要
  let head3 = 1;
  let head5 = 1;
  const ugly = [];
  for (let i = 1; i <= n; i++) {
    const min = Math.min(head2, head3, head5); // 三个链的尾端最小值。这三个值一定都是前面某个丑数的2、3、5倍
    ugly[i] = min; // 选取这个值作为新的丑数
    if (min === head2) {
      // 选取的是哪个链上的值，这个链的指针就向前一个，计算出新的值赋给headn
      head2 = 2 * ugly[p2++];
    }
    if (min === head3) {
      head3 = 3 * ugly[p3++];
    }
    if (min === head5) {
      head5 = 5 * ugly[p5++];
    }
  }
  return ugly[n];
};
```

## 众数（多数元素）I II

https://leetcode.cn/problems/majority-element/
https://leetcode.cn/problems/majority-element-ii/

> 众数 I：
> 给定一个大小为 n 的数组  nums ，返回其中的多数元素。多数元素是指在数组中出现次数 大于  ⌊ n/2 ⌋  的元素。
> 你可以假设数组是非空的，并且给定的数组总是存在多数元素。
> 示例  1：
> 输入：nums = [3,2,3]
> 输出：3

众数 I 的解法比较多，题解给出了 5 种解法，但是比较有意义的其实是三种

1. 首先是最基本的哈希表，遍历一次统计次数，返回次数大于 n/2 的即可。这个方法适用于各种众数问题。
2. 给数组排序，然后直接返回序号为 n/2 上的元素；证明可以参考[题解的方法二](https://leetcode.cn/problems/majority-element/solution/duo-shu-yuan-su-by-leetcode-solution/)
3. 摩尔投票算法，这个才是关键。

所谓摩尔投票，其实是一种“抵消”算法。
基本思路是如果我们把众数记为 +1，把其他数记为 -1，将它们全部加起来，显然和大于 0，从结果本身我们可以看出众数比其他数多。

![](https://pic1.imgdb.cn/item/6342b29716f2c2beb1d1b4b7.jpg)

也就是说遍历整个数组，选取一个数为基准；如果当前的数字等于基准数，就给 cnt++，不等于就--；如果这个基准数是正数，那么最后的 cnt 一定大于 0。其实相当于两个不同的数在这个过程中“抵消”了，这样最后剩下的数就是那个众数，因为众数的数量一定大于非众数的数量总和。这就相当于每个“多数元素”和其他元素 两两相互抵消，抵消到最后肯定还剩余**至少 1 个“多数元素”**。

具体写法上，肯定不能选取每个数字都试一遍，这样就是 O(n^2)的复杂度了；正确的方法是先从第一个数开始（以第一个数作为基准数），然后依次遍历并和后面数比较。同样的，如果相同就给 cnt++，不同就 cnt--，但当 cnt===0 时，说明这个数已经被抵消完了；这时我们就需要重新选取正在遍历的 num 作为新的基准数，后面也是同样的步骤。

代码如下：

```js
var majorityElement = function (nums) {
  let cand = nums[0]; // 选第一个数作为候选数
  let cnt = 1; // 次数
  for (let num of nums) {
    if (num === cand) cnt++; // 如果当前数就是候选数，那就cnt++
    else cnt--; // 否则就--
    if (cnt === 0) {
      // 当cnt === 0时，说明这个数已经被全部抵消完了。众数不可能被抵消完，因此必然不是众数，直接替换
      cand = num;
      cnt = 1; // 重置cnt为1
    }
  }
  return cand;
};
```

---

然后是多数元素 II 这道题。这道题让你找到出现次数大于 n/3 的数字，这个数字可能会有多个（最多 2 个）。
首先用哈希表也可以做，第二种方法就还是摩尔投票法。
和上一道题不一样，这道题要统计的数应该是两个（当然也有可能是 0、1 个，这里先考虑两个），也就是候选数应该是两个。基本思路还是“抵消”，只不过上一道题是两个数抵消，这道题是三个数抵消。

![](https://pic1.imgdb.cn/item/6342b36d16f2c2beb1d3bcbf.jpg)

我们考虑设置两个阵营 A 和 B，这两个阵营分别可以放一类数字，即阵营内的数字应该相同。每次遍历到一个数字，就检查它是不是和 A 或 B 中的数字相同

- 如果和 A 或 B 的数相同，就放入 A 或 B
- 如果不同，就从 A 和 B 中各取一个数，和这个数抵消

类比上一道题，其实上一道题就相当于只有一个阵营，相同就放入，不相同就抵消。

> 其实还可以扩展到 n/k 的情况，按照众数 II 的这个思路来就好，k 为多少就设置 k-1 个阵营，然后按照相同的方法即可。

代码：

```js
var majorityElement = function (nums) {
  /**
   * 假设现在有两个阵营a和b，那么对于每一个元素，只有三种情况：
   * 1. 进入A阵营，前提是它的值和A阵营的核心数字（其实也就是elementA自己）相同
   * 2. 进入B阵营，前提是和elementB相同
   * 3. 两个都不进入，即这个数和A和B都不同。
   * 对于这三种情况，分别有三种处理：
   * 1. 给A阵营的数量+1，即cntA+1，用于表示A阵营中数字的数量
   * 2. cntB+1
   * 3. 从A阵营和B阵营中各选出一个数，然后把这三个数抵消掉。也就是说给cntA--和cntB--，然后跳到下一个数字。
   * 另外，如果经过第三步之后A阵营和B阵营的人数又为0，即cntA === 0或cntB === 0，就选择下一次遍历时的数字作为新的阵营核心数字
   */
  let elementA = 0; // A阵营
  let elementB = 0; // B阵营
  let cntA = 0;
  let cntB = 0;
  const n = nums.length;

  for (let num of nums) {
    if (num === elementA) {
      cntA++;
    } else if (num === elementB) {
      cntB++;
    } else if (cntA === 0) {
      // 当cntA为0时选择当前数作为阵营A的数
      cntA++;
      elementA = num;
    } else if (cntB === 0) {
      // 同理
      cntB++;
      elementB = num;
    } else {
      // 抵消
      cntA--;
      cntB--;
    }
  }
  // 统计两个阵营的数字数量
  // 上一步只是得到了两个数字，并不知道两个数字具体的出现次数
  let voteA = 0;
  let voteB = 0;
  for (let num of nums) {
    if (cntA > 0 && num === elementA) voteA++;
    if (cntB > 0 && num === elementB) voteB++;
  }

  const ans = [];
  if (voteA > Math.floor(n / 3)) ans.push(elementA);
  if (voteB > Math.floor(n / 3)) ans.push(elementB);
  return ans;
};
```

## 只出现一次的数字

https://leetcode.cn/problems/single-number/
https://leetcode.cn/problems/single-number-ii/
https://leetcode.cn/problems/single-number-iii/

这个系列一共有三道题，三道题都可以用哈希表解，但是最佳的方法都是位运算。

### 只出现一次的数字 I

> 给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。
> 说明：
> 你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？
> 示例 1:
> 输入: [2,2,1]
> 输出: 1

这道题在上面位运算里讲到过。因为两个相同的数无论出现次序，只要相互异或，最终结果一定为 0，而 0 和其他数异或还是原来的数，因此可以把数组中所有的数都异或在一起，最后得到的数就是那个剩下的单个数。

```js
var singleNumber = function (nums) {
  return nums.reduce((a, b) => a ^ b, 0);
};
```

### 只出现一次的数字 II

> 给你一个整数数组  nums ，除某个元素仅出现 一次 外，其余每个元素都恰出现 三次 。请你找出并返回那个只出现了一次的元素。
> 你必须设计并实现线性时间复杂度的算法且不使用额外空间来解决此问题
> 示例 1：
> 输入：nums = [2,2,3,2]
> 输出：3

这个题和上一道唯一的区别在于每个数出现了三次，所以单纯的异或也就不可能了。
它的操作具体是这样：

![](https://pic1.imgdb.cn/item/6346811e16f2c2beb17351cf.jpg)

我们依次把每个数的二进制位加在一起并放入一个数组中。那么对于这个数组的每一位，都表示所有数字在该位的二进制值的和；又因为除了一个数之外，其他都是三的倍数出现，因此把最后结果%3 就可以得到只出现一次的数字的二进制形式。

在具体实现上，可以外层循环遍历结果数组的每一位，内层循环每个数并计算它们二进制形式在这一位上的和，当求出和时对 3 取余得到结果，作为该位的最后结果。

计算二进制形式的某位值：

```js
num >> i && 1;
// 其中i表示从小到大第i位
```

计算和以及对 3 取余时，就当作正常的十进制数处理即可。最后的结果通过“二进制加法”（即或运算）加在一起。

```js
res = res | (2 ** i);
// 因为res表示的就是结果的二进制形式中的第i位的值，这里就相当于给原来的res在第i位上放上一个1
// 比如res = 1001，i = 5，那么1001 | 10000 = 11001
```

注意最好不要把结果放入数组然后手动得到，因为数组中有负数。在上面的过程中最后得到的数负号会被忽略，如果手动计算十进制形式就会失去负号，而且你事先无法知道结果到底是正数还是负数。

代码：

```js
var singleNumber = function (nums) {
  let res = 0;
  for (let i = 0; i < 32; i++) {
    let sum = 0;
    for (let num of nums) {
      sum += (num >> i) & 1; // 注意括号
    }
    // 如果不是3的倍数就说明这一位上有额外的值（即单独的数）加的结果
    // 即说明这一位上的值应该是单独的数拥有的
    if (sum % 3 !== 0) {
      res = res | (1 << i);
    }
  }
  return res;
};
```

### 只出现一次的数字 III

> 给你一个整数数组  nums，其中恰好有两个元素只出现一次，其余所有元素均出现两次。 找出只出现一次的那两个元素。你可以按 任意顺序 返回答案。
> 你必须设计并实现线性时间复杂度的算法且仅使用常量额外空间来解决此问题。
> 示例 1：
> 输入：nums = [1,2,1,3,2,5]
> 输出：[3,5]
> 解释：[5, 3] 也是有效的答案。

这道题核心思路还是利用异或，也就是和第一道题一样的思路。

我们把所有数字都异或在一起，相同的数会被消去，**因此最后得到的异或结果一定是这两个单独的数异或的结果**。比如示例 1 中，所有数异或的结果就是`3^5`的结果
所以下一步就是考虑怎么把这两个数分离出来。异或的特点是，相同的位为 0，不同的位为 1，也就是说这个异或的结果中为 1 的位就是这两个数不同的位。

```
15 = 1111
11 = 1011

1111 ^ 1011 = 0100
说明15和11的第三位不同
```

得出哪个位不同之后，显然数组中的其他数就可以根据这个位分成两部分：

- 一部分是这一位为 0 的，这些数的这一位和 11（1011）的异或结果都是 0
- 另一部分是这一位为 1 的，这些数就恰相反，在这一位上和 11 的异或结果都是 1

又因为数组中的其他数都是两两成对的，所以我们把这两部分的内部分别异或，相同的数消去，最后得到的就是这个数。
其实也就是把这一位为 1 和为 0 的数分开，然后分别异或即可。

![](https://pic1.imgdb.cn/item/6346896016f2c2beb1805980.jpg)

具体代码实现：

1. 遍历所有数并异或起来，从后向前找到第一个位为 1 的位置，将这个位置设为 index
   获取从后向前第一个位为 1 的位置可以这样：

```js
while ((val & 1) === 0 && index < 32) {
  val = val >> 1;
  index++;
}
```

即每次比较最后一位是不是 1，不是就右移。

2. 遍历数组中的每个数，把每个数按照第 index 位区分，为 1 的放入一个数组，为 0 的放入另一个数组
   比较的方法和上面类似，将每个数先右移 index，然后和 1 与即可

```js
((num >> index) & 1) === 0;
```

3. 分别异或两个数组内部，得到的值就是两个数。

代码：

```js
var singleNumber = function (nums) {
  let val = 0;
  for (const num of nums) {
    val ^= num;
  }
  let index = 0;
  while ((val & 1) === 0 && index < 32) {
    val = val >> 1;
    index++;
  }
  let num1 = 0;
  let num2 = 0;
  for (const num of nums) {
    // 注意括号，位运算最好每个式子都带括号
    if (((num >> index) & 1) === 0) num1 ^= num;
    else num2 ^= num;
  }
  return [num1, num2];
};
```

## Pow(x,n) （快速幂和快速积算法）

https://leetcode.cn/problems/powx-n/

题目很简单，就是让你在不使用库函数的情况下，实现计算 x 的 n 次方的函数。

首先最简单的方法就是直接利用递归，每次把 n 砍半，分别计算两个部分的幂，然后乘在一起：

```js
var myPow = function (x, n) {
  if (n === -1) return 1 / x;
  if (n === 0) return 1;
  if (n === 1) return x;
  if (n === 2) return x * x;

  return myPow(x, Math.floor(n / 2)) * myPow(x, n - Math.floor(n / 2));
};
```

这个方法会超时，显然不是最好的方法，最佳的方法是快速幂算法。算法解释如下：

![](https://pic1.imgdb.cn/item/63468e2616f2c2beb187fcfc.jpg)

大概解释一下，其实和上面的解法很像，还是通过把 n 砍半的方式分治计算；

- 如果 n 为奇数，这里的处理就是在原来的积的基础上，再乘上一个原数 x。
- 如果 n 为偶数，那就直接返回`x^(n/2) * x^(n/2)`的结果即可

```
比如x = 5,n = 7
那么 7 % 2 = 1，这时结果就应该是 x^3 * x^3 * x。
x^3通过递归计算得到
```

代码：

```js
var myPow = function (x, n) {
  function quickMul(x, n) {
    if (n === 0) return 1;
    const y = quickMul(x, Math.floor(n / 2));
    return n % 2 === 0 ? y * y : y * y * x;
  }
  return n > 0 ? quickMul(x, n) : 1 / quickMul(x, -n);
};
```

---

推广一下，如果把乘法换成加法，就可以得到快速积算法。快速积可以用于一些不允许使用乘法的题目用于替代乘法：

```js
function quickAdd(x, n) {
  if (n === 0) return 0;
  if (n === 1) return x;
  const y = quickAdd(x, Math.floor(n / 2));
  return n % 2 === 0 ? y + y : y + y + x;
}
```

（这个方法在后面一道题目中有很大作用）

## 两数相除

https://leetcode.cn/problems/divide-two-integers

> 给定两个整数，被除数  dividend  和除数  divisor。将两数相除，要求不使用乘法、除法和 mod 运算符。
> 返回被除数  dividend  除以除数  divisor  得到的商。
> 整数除法的结果应当截去（truncate）其小数部分，例如：truncate(8.345) = 8 以及 truncate(-2.7335) = -2

如果要手动实现加、减、乘都是逻辑上比较简单，具体操作稍微麻烦一些的。但是这道题的除法，从逻辑上就不简单了。

主要方法有两个，一个是通常对于计算类型题目常用的**二分查找**，另一个是递归骚操作
先说二分查找，基本思路就是由乘法倒推；因为我们直到除数，显然`除数 * 商 = 被除数`，那么如果能找到一个商使得和除数的积刚好小于或等于被除数，就可以得到结果。
那么怎么样去找这个商呢？从一个范围遍历当然是一种方法，不过对于这种**从一个范围中找一个结果**的类型，并且结果还是某个最大或最小值，首先就应该想到二分查找。
把二分查找的上限设为 `2**31-1`，下限设为 1，在这个范围中找一个数 mid，使得`mid * divisor <= dividend`即可。因为不一定能找到刚好等于的数，因此不用设置相等的情况（参考上面二分查找的题）

核心代码就是这样：
需要注意的是还需要快速积代替乘法，用移位运算代替除以 2

```js
let left = 1;
let right = MAX; // max就是上限数2**31-1
while (left <= right) {
  const mid = left + Math.floor((right - left) >> 1);
  if (mid === MAX) break;
  // 不能用乘法，用上面的快速积代替乘法
  if (quickAdd(divisor, mid) <= dividend) {
    left = mid + 1;
  } else right = mid - 1;
}
return right; // 最后的right就是结果
// 因为当小于等于时left还会移动，有可能会使left越过结果并大于right。所以要以right为准
```

剩下的代码，也就是这个题最麻烦的地方，就是处理各种边界情况。边界情况大概有以下几种：（我们把上限`2**31-1`设为 MAX，下限`-2**31`设为 MIN）

- 被除数是 MAX
  - 除数是-1，结果是 MIN
  - 除数是 MIN，结果是-1
- 被除数是 0，结果是 0
- 被除数是 MIN
  - 除数是 MAX，结果是-1
  - 除数是-1，结果是 MAX
- 除数是 MIN
  - 被除数如果不是 MIN，就都是 0
  - 被除数是 MIN，结果是 1
- 除数是 MAX
  - 被除数如果不是 MAX，都是 0

当然实际上这些边界情况并不用全写进去，只需要一部分就可以通过全部用例。代码如下：

```js
var divide = function (dividend, divisor) {
  const MAX = 2 ** 31 - 1;
  const MIN = -(2 ** 31);
  let isNeg = false;
  if (dividend === 0) return 0;
  if (dividend === MIN) {
    if (divisor === 1) return MIN;
    if (divisor === -1) return MAX;
  }

  if (divisor === MIN) {
    if (dividend === MIN) return 1;
    else return 0;
  }
  // 计算正负
  // 这里也是个很好的思路，用同一个布尔值，如果数取反那么布尔值也取反
  if (dividend < 0) {
    dividend = -dividend;
    isNeg = !isNeg;
  }
  if (divisor < 0) {
    divisor = -divisor;
    isNeg = !isNeg;
  }

  const quickAdd = (x, n) => {
    if (n === 0) return 0;
    if (n === 1) return x;
    const y = quickAdd(x, Math.floor(n / 2));
    return n % 2 === 0 ? y + y : y + y + x;
  };

  let left = 1;
  let right = MAX;
  while (left <= right) {
    const mid = left + Math.floor((right - left) / 2);
    if (mid === MAX) break;
    if (quickAdd(divisor, mid) <= dividend) {
      left = mid + 1;
    } else right = mid - 1;
  }
  return isNeg ? -right : right;
};
```

## 两数相加

> 给你两个数 a 和 b，求两个数的和，但不能用加法

位运算：

1. 两个数的异或可以看作是两个数的无进位加法
2. 两个数的与再左移一位（`(a & b) << 1`）可以看作是两个数的总的进位值。

把这两个值相加，就可以得到结果。
但是这两个值可能还有进位，因此转成一个循环，除了第一次的值是 a 和 b，后面每次的和都是`两个数的异或值+两个数的进位值`，直到进位为 0 为止。

```js
while (b !== 0) {
  // 注意这里是b!==0不是b > 0；考虑到a或b是负数的情况，extra也有可能是补码负数，只有完全为0才说明没有进位
  const extra = (a & b) << 1;
  const sum = a ^ b;
  a = sum;
  b = extra;
}

// 递归
function sum(a, b) {
  if (a === 0) return b;
  return sum((a & b) << 1, a ^ b);
}
```
