---
title: js手写
date: 2022-05-13 12:21:10
tags: 面试
cover: /img/JS.png
categories: JavaScript
sticky: 4
---

# 手写题

## 手写 new

new 的大致原理:

```js
function _new(fn) {
  const obj = {};
  obj.__proto__ = fn.prototype;
  return fn.call(obj);
}
```

1. 创建一个对象，将来作为实例
1. 将实例的`__proto__`属性指向构造函数的`prototype`属性，即`let obj; obj.__proto__ = fn.prototype`
1. 调用构造函数并绑定 this 到 obj 上
1. 返回调用结果

因此手写 new 按照类似的原理创建：

```js
function Animal(name) {
  this.name = name;
}

function _new(fn, ...args) {
  const obj = Object.create(fn.prototype); //新建对象并将原型指向构造函数的prototype
  const ret = fn.apply(obj, args); //调用构造函数，并把上下文指定为当前对象（让构造函数内的this都指向当前对象）
  return ret instanceof Object ? ret : obj; //检查返回值，如果没有返回值就返回该对象，如果有就返回返回值
}
let dog = _new(Animal, "Puppy");
```

实际上关键的一步就是`fn.apply(obj, args)`；
如果函数没有显式的返回值（构造函数如果显式返回对象，new 会直接返回该对象而不是实例），实际上就是一个绑定对象，给对象添加属性并设置原型的过程。

```js
let obj = {};
obj.__proto = fn.prototype;
fn.call(obj, ...args);
let dog = obj;
```

## 实现一个 const

const 可以通过对对象属性的配置实现，即用 Object.defineProperty 设置某个对象下的值不能被修改即可。

假设存在一个作用域对象 scopeObj，所有通过 const 定义的变量都会存储为这个对象的属性。然后对这个属性配置 set 时报错并不能修改即可。

```js
var _const = function (name, val, scopeObj = window) {
  scopeObj[name] = val;
  Object.defineProperty(scopeObj, name, {
    enumerable: false,
    configurable: false,
    writable: false,
    get() {
      return val;
    },
    set(value) {
      if (value !== val)
        throw new TypeError("Assignment to constant variable.");
    },
  });
};
```

## 手写数组去重

原理:

- filter 会在返回为 false 时去除该元素
- indexOf 会返回第一个找到的该元素的索引; 如果有重复元素则显然 indexOf 只返回第一个该元素, 不会等于当前 index,就会返回 false 从而筛掉

**注意**：是indexOf和index比较，没有lastIndexOf的事

```js
function unique(arr) {
  let res = arr.filter((item, index, array) => {
    return arr.indexOf(item) === index;
  });
  return res;
}

// reduce也可以
function unique(arr) {
  return arr.reduce((pre, cur, index, arr) => {
    return arr.indexOf(cur) === index ? pre.concat(cur) : pre;
  }, []);
}
```

## 手写数组扁平

原理:

- `arr.some()` 方法, 参数传递一个测试函数, 对每个元素测试, 如果至少一个通过测试就会返回 true; 也就是判断这个数组中是否至少还有一个数组(至少还有一维没有展平)
- `Array.isArray` 判断是否是一个数组
- 核心方法是 `concat` ,这个方法会抹平二维数组(`[].concat(1,[2]) = [1,2]`), 所以只需要递归/循环就行

```js
function flatten(arr) {
  while (arr.some((item) => Array.isArray(item))) {
    arr = [].concat(...arr);
  }
  return arr;
}
//[].concat(1,[2]) = [1,2]
//
```

如果需要加上深度判断：

```js
function flatten(arr, depth = 1) {
  if (!Array.isArray(arr)) return arr;
  while (arr.some((item) => Array.isArray(item)) && depth > 0) {
    arr = [].concat(...arr);
    depth--;
  }
  return arr;
}
```

其他递归写法：

```js
function flatten(arr) {
  if (!Array.isArray(arr)) return arr;
  return arr.reduce((pre, curr, index, arr) => {
    return pre.concat(flatten(curr));
  }, []);
}

function flatten(arr) {
  if (!Array.isArray(arr)) return arr;
  let res = [];
  for (const val of arr) {
    res = res.concat(flatten(val));
  }
  return res;
}

// 不用concat也可以
// concat的主要作用就是可以接受一个值，也可以接受一个数组，把所有的值连接到新数组
// 不用concat的话，就区分一下是否是数组就可以了
function flatten(arr, depth = Infinity) {
  if (depth === 0) return arr;
  let newArr = [];
  for (const item of arr) {
    if (Array.isArray(item)) {
      const resArr = flatten(item, depth - 1);
      newArr.push(...resArr);
    } else newArr.push(item);
  }
  return newArr;
}
```

## 手写数组分组

实现一个函数，参数是数组 arr 和要分的每组大小 size，返回这个数组分成 n 个这么大的数组的二维数组

```
chunk([1,2,3,4],2)
=> [[1,2],[3,4]]

chunk([1,2,3,4],3)
=> [[1,2,3],[4]]
```

最基本写法：每 n 个放入一个数组

```js
const chunk = (arr, size) => {
  const res = [];
  while (arr.length) {
    const tmp = [];
    for (let i = 0; i < size; i++) {
      const top = arr.shift();
      if (top) tmp.push(top);
    }
    res.push([...tmp]);
  }
  return res;
};
```

还可以在一次循环中完成。即用 i 和 size 的商得到每个新数组的位置，显然当 i 是 size 的 n 倍时，这些数字会被放入同一个数组。
比如 size 是 3，那么 3 4 5 得到的 index 都是 1，就会被放入同一个数组。

```js
function chunk(arr, size) {
  const res = [];
  for (let i = 0; i < arr.length; i++) {
    const index = Math.floor(i / size);
    if (res[index] == undefined) res[index] = [];
    res[index].push(arr[i]);
  }
  return res;
}
```

## 手写 reduce

reduce 实际上是一个循环，依次把之前的数据和初始值（或第一次）一起调用回调
考虑到初始值的情况，如果有初始值就从初始值开始，并且从数组第二项开始调用

```js
function myReduce(callback, init) {
  if (!Array.isArray(this)) return; //要求是数组
  const arr = this;
  let total = init || arr[0]; //如果有初始值就取初始值
  for (let i = init ? 0 : 1; i < arr.length; i++) {
    //有初始值就从i=1开始，即从第二项开始
    total = callback(total, arr[i], i, arr); //调用callback，每次的total都是上次的返回值
  }
  return total;
}
```

## 手写深拷贝

### 基本递归

最基本的递归很简单，实际上就是一个回溯复制：
原理就是先创建一个新对象，然后依次把旧对象的值赋值过去；如果这个值是一个对象，就递归一次，否则就直接复制。

```js
function clone(obj) {
  let newObj = {};
  if (typeof obj !== "object") return obj;
  for (const key in obj) {
    newObj[key] = clone(obj[key]);
  }
  return newObj;
}
```

### 添加数组判断

但是要考虑到数组，就得稍微改进一下：
只需要添加类型判断，不同的初始化即可：

```js
function clone(target) {
  if (typeof obj !== "object") return target;
  let newTarget = Array.isArray(target) ? [] : {};
  for (const key in target) {
    newTarget[key] = clone(target[key]);
  }
  return newTarget;
}
```

### 避免循环调用

当对象的属性间接或直接的引用了自身，就会形成循环引用。上面的方法处理一个循环引用会导致栈溢出。

```js
let obj = {};
obj.obj = obj;
```

这种情况下，访问 `obj.obj` 依旧是一个 obj 类型，无论如何都不会退出递归，会形成死循环。
解决的方法是设定一个存储空间 `Map`，每次 `clone` 都向 `Map` 中添加，并且每次递归都先检查是否有克隆过的对象，如果有就直接返回
范例如下：

```js
function clone(target, map = new WeakMap()) {
  if (typeof target !== "object") return target;

  let newTarget = Array.isArray(target) ? [] : {};
  //如果有这个对象就返回一个空对象或数组，阻止循环调用
  //正常情况下都没有，只有重复引用时target和newTarget都是一样的对象
  if (map.has(target)) return map.get(target);

  map.set(target, newTarget); // 存键为当前对象，值为当前对象的复制对象。

  for (const key in target) {
    newTarget[key] = clone(target[key], map); //把当前的map传下去
  }
  return newTarget;
}
```

### BFS 深拷贝

基本思路是同步处理origin和target。origin每次取一个子节点，target也就相应的向下走一层；
queue内是当前的origin对象和target对象，他俩的层级应该是相同的。


```js
function deepClone(obj) {
  const res = Array.isArray(obj) ? [] : {};
  const queue = [];
  queue.push([obj, res]);
  const map = new WeakMap();
  map.set(obj, res);
  while (queue.length) {
    const [curr, target] = queue.shift();
    for (const key of Object.keys(curr)) {
      const val = curr[key];
      if (map.has(val)) {
        target[key] = map.get(val);
        continue;
      }
      if (typeof val !== "object") target[key] = val;
      else {
        target[key] = Array.isArray(val) ? [] : {};
        queue.push([curr[key], target[key]]);
        map.set(curr[key], target[key]);
      }
    }
  }
  return res;
}
```

### 检查更多类型

考虑到参数还可能是更多的类型，并且 typeof 判断类型不一定准确，因此可以考虑更改一下判断参数类型的函数：

考虑用 Object.prototype.toString 来判断类型，可以把类型分为可继续遍历和不可继续遍历两类：

```js
const mapTag = "[object Map]";
const setTag = "[object Set]";
const arrayTag = "[object Array]";
const objectTag = "[object Object]";

const boolTag = "[object Boolean]";
const dateTag = "[object Date]";
const errorTag = "[object Error]";
const numberTag = "[object Number]";
const regexpTag = "[object RegExp]";
const stringTag = "[object String]";
const symbolTag = "[object Symbol]";
```

如果 value 是可遍历的 5 种类型，那就需要进一步处理。比如 map 不能直接复制值，而是展开遍历 map 再复制到一个新的 map 中。

而对于不可遍历的类型，也要考虑其复制。比如 symbol、正则等需要重新构造

至于数组和对象的区分，我们可以考虑直接用它的 constructor 属性构造，这样就不用判断了。比如数组的 constructor 就是 Array 构造函数，直接调用就可以。

并且由于 forin 的性能很差，这里采用 while 循环的形式。对于数组直接遍历即可，而对于对象则遍历其`Object.keys(target)`。实际上这个循环函数和 forEach 的用法是完全一样的

```js
const mapTag = "[object Map]";
const setTag = "[object Set]";
const arrayTag = "[object Array]";
const objectTag = "[object Object]";

const boolTag = "[object Boolean]";
const dateTag = "[object Date]";
const errorTag = "[object Error]";
const numberTag = "[object Number]";
const regexpTag = "[object RegExp]";
const stringTag = "[object String]";
const symbolTag = "[object Symbol]";
const bigIntTag = "[object BigInt]";
const functionTag = "[object Function]";

const getType = (target) => Object.prototype.toString.call(target); // 获取类型
const getInit = (target) => new target.constructor(); // 用构造函数构造
const isObject = (
  target // 是否是对象或数组
) => getType(target) === arrayTag || getType(target) === objectTag;
const deepTag = [mapTag, setTag, arrayTag, objectTag]; // 可以深度遍历的类型
const whileLoop = (target, callback) => {
  // while循环代替forin
  let start = 0;
  while (start < target.length) {
    callback(target[start], start);
    start++;
  }
};
const cloneReg = (target) => {
  // 复制正则
  const reFlags = /\w*$/;
  const result = new target.constructor(target.source, reFlags.exec(target)[0]); // 第一个参数是原正则的字符串形式，第二个是范围符，比如'g'、'i'等
  result.lastIndex = target.lastIndex; // 指定下一次匹配的起始索引
  return result;
};
const cloneSymbol = (target) => Object(Symbol.prototype.valueOf.call(target)); // symbol的复制就是取其valueOf并转为对象
const cloneBigInt = (target) => BigInt(target.toString());

// 不可深度遍历类型的处理
const cloneBasicType = (target) => {
  const ctor = target.constructor;
  switch (getType(target)) {
    case boolTag:
    case numberTag:
    case stringTag:
    case errorTag:
    case dateTag:
      return new ctor(target);
    case regexpTag:
      return cloneReg(target);
    case symbolTag:
      return cloneSymbol(target);
    case bigIntTag:
      return cloneBigInt(target);
    default:
      return target;
  }
};

function deepClone(target, map = new WeakMap()) {
  if (!isObject(target)) {
    return target;
  }
  const type = getType(target);
  let cloneTarget; // 最终复制结果
  if (deepTag.includes(type)) cloneTarget = getInit(target);
  else cloneTarget = cloneBasicType(target);

  if (map.has(target)) return map.get(target);
  map.set(target, cloneTarget);

  // map的处理，遍历map并创建新的
  if (type === mapTag) {
    for (const [key, value] of target.entries()) {
      cloneTarget.set(key, deepClone(value, map));
    }
    return cloneTarget;
  }

  if (type === setTag) {
    for (const value of target.values()) {
      cloneTarget.add(deepClone(value, map));
    }
    return cloneTarget;
  }

  if (type === objectTag) {
    const keys = Object.keys(target);
    // 取keys遍历，key就是每个键，相当于forin
    whileLoop(keys, (key) => {
      cloneTarget[key] = deepClone(target[key], map);
    });
    return cloneTarget;
  }

  if (type === arrayTag) {
    // 直接取值遍历，用index作为索引
    whileLoop(target, (value, index) => {
      cloneTarget[index] = deepClone(value, map);
    });
    return cloneTarget;
  }
  return cloneTarget;
}
```

剩下更进阶的可以参考：https://juejin.cn/post/6844903929705136141

## 手写深比较

深比较即主要针对对象的比较，不能直接通过`===`判断。思路就是取出两个对象的键，分别遍历比较每个键对应的值是否相等；然后再通过递归的形式比较值是对象类型的键。

其实可以理解为多叉树的比较。二叉树的比较很简单，这里就是在二叉树的基础上改为多叉树就可以了

```js
// 二叉树的比较
function compareTree(root1, root2) {
  if (!root1 && !root2) return true;
  if ((!root1 && root2) || (root1 && !root2)) return false;
  return (
    compareTree(root1.left) &&
    compareTree(root2.left) &&
    root1.val === root2.val
  );
}

// 多叉树
function compareTree(root1, root2) {
  if (!root1 && !root2) return true;
  if ((!root1 && root2) || (root1 && !root2)) return false;
  const pairs = [];
  for (const child of root1.children) {
    // 这里把children换为Object.values(root1)就可以
    pairs.push([child]);
  }
  for (let i = 0; i < root2.children.length; i++) {
    pairs[i].push(root2.children[i]);
  }
  let res = true;
  for (const pair of pairs) {
    res = res && compareTree(...pair);
  }
  return res && root1.val === root2.val;
}
```

```js
function isEqual(x, y) {
  if (x === y) {
    return true;
  } else if (
    typeof x === "object" &&
    x !== null &&
    typeof y === "object" &&
    y !== null
  ) {
    const keysX = Object.keys(x);
    const keysY = Object.keys(y);
    if (keysX.length !== keysY.length) {
      return false;
    }
    for (const key of keysX) {
      if (!isEqual(x[key], y[key])) {
        return false;
      }
    }
    return true;
  } else {
    return false;
  }
}
```

另外一种写法，思路上有点像比较两棵树；但是由于对象的键可能大于 2，因此写出来更像回溯：

注意几个边界情况

1. 对象的 key 顺序可能不同，可以使用 sort 将其排个序
2. value 相同 key 不同的情况也要注意，要排序后再比较一下 key
3. `[0]`和`{"0":0}`表现是一样的，但不是同一类，要注意区分，可以判断一下两者类型。

```js
/**
 * @param {any} o1
 * @param {any} o2
 * @return {boolean}
 */
var areDeeplyEqual = function (o1, o2) {
  if (!o1 && !o2) return o1 == o2;
  if ((o1 && !o2) || (!o1 && o2)) return false;
  if (!isObject(o1) && !isObject(o2)) return o1 === o2;
  if (getType(o1) !== getType(o2)) return false;
  const keys1 = Object.keys(o1).sort();
  const keys2 = Object.keys(o2).sort();
  if (keys1.length !== keys2.length) return false;
  let flag = true;
  for (let i = 0; i < keys1.length; i++) {
    const key1 = keys1[i];
    const key2 = keys2[i];
    flag = flag && key1 === key2 && areDeeplyEqual(o1[key1], o2[key2]);
  }
  return flag;
};
function isObject(obj) {
  return typeof obj === "object" && obj != null;
}

function getType(val) {
  return Object.prototype.toString.call(val).slice(8, -1);
}
```

## 手写防抖/节流

基本实现见下，更加高级的实现可以参考：https://juejin.cn/post/7057053516849758222#heading-7

- 防抖

```js
function debounce(func, wait) {
  let timeout;
  return function () {
    if (timeout) clearTimeout(timeout);
    timeout = setTimeout(() => {
      func.apply(this, arguments);
    }, wait);
  };
}
```

上面的写法有个缺点，就是第一次触发时间时仍然会等待 wait 秒。可以增加一个 immediate，使得第一次执行不用等待

```js
function debounce(func, wait, isImmediate) {
  let timeout;
  let once = true;
  return function () {
    if (isImmediate && once) {
      func.apply(this, arguments);
      once = false;
      // 注意这里还是要设置一个timeout
      timeout = setTimeout(() => {
        timeout = null;
      }, wait);
      return;
    }
    if (timeout) clearTimeout(timeout);
    timeout = setTimeout(() => {
      func.apply(this, arguments);
    }, wait);
  };
}
```

- 节流

时间戳实现：在外部函数留一个闭包表示上次调用时间，函数内部每次请求当前时间，如果时间戳小于延迟就不执行；反之执行之后修改上次时间为当前

```js
function throttle(fn, delay) {
  let pre = 0;
  return function () {
    let now = new Date().getTime();
    if (now - pre < delay) return;
    fn.apply(this, arguments);
    pre = now;
  };
}
```

`setTimeout`实现：几乎和防抖一样，只是如果 timeout 存在就不进入；并且执行之后把 timeout 设为 null，相当于 timeout 是一个“大门”。在定时器执行期间不会进入定时器，只有当定时器内的代码执行之后才会运行下一个定时器

```js
function throttle(fn, delay) {
  let timeout;
  return function () {
    if (timeout) return;
    timeout = setTimeout(() => {
      timeout = null;
      fn.apply(this, arguments);
    }, delay);
  };
}
```

两种写法可以结合一下。使用时间戳记录时间差值，然后和设定的时间间隔相减作为 remain；如果 remain<=0，就立即执行；反之就是开启一个定时器，等到 remain 时间结束后再执行

```js
function throttle(fn, delay) {
  let pre = 0;
  return function () {
    let now = new Date().getTime();
    const remain = delay - (now - pre);
    if (remain <= 0) {
      fn.apply(this, arguments);
      pre = now;
    } else {
      timer = setTimeout(fn, remain);
    }
  };
}
```

## 手写柯里化

柯里化本质是对一个函数的反复调用和返回，让原先直接返回结果的函数，变成返回一个函数，再次或 n 次调用之后才返回结果。在这 n 次之内的返回值都是函数

比如，想要对 add 函数实现柯里化，可能有很多层：

```js
// 两层
add(1)(2, 3);

function add(...args1) {
  return function (...args2) {
    return [...args1, ...args2].reduce((a, b) => a + b);
  };
}

// 三层
add(1)(2, 3)(4, 5);

function add(...args1) {
  return function (...args2) {
    return function (...args3) {
      return [...args1, ...args2, ...args3].reduce((a, b) => a + b);
    };
  };
}
```

可以看到，随着需要的层数增加，本质上就是函数的嵌套，逐层收集参数，在某个终止条件下得到结果。

因此对于不确定的层数，最佳的实现方式就是递归：

```js
function add(...args1) {
  return function add2(...args2) {
    // 某个条件下，比如参数达到限制，就合并参数计算
    return add(...args1, ...args2);
  };
  // 给函数安插某些方法
}
```

当层数超过 2 时，每增加一层都相当于递归调用了一次 add，得到的是 add2，也就相当于每次都是在执行 add2。
注意递归的应该是外层的 add，而不是里层的 add2。如果调用 add2 就会发现参数不对：因为 args1 始终没有改变，一直都是第一次的

如果是要求函数接收一个函数并转化为柯里化的函数，就在外层套一个函数就可以。本质上递归还是在调用第一次调用的函数

```js
function curry(func) {
  // 这里可以记录调用层数
  return function curried(...args) {
    if (args.length === func.length) {
      //这次调用传入的参数个数和函数总参数个数比较，如果等于说明是完整调用，直接返回；
      //如果小于，比如add(1)(2,3)，原本本来是三个参数，现在只传入一个，因此进入柯里化步骤
      return func(args);
    } else {
      return function (...args2) {
        //这里把柯里化的前部分和后部分拆开，这个函数的参数是后部分的
        //即，add(1)(2,3) 中，...agrs是1，...args2是2和3，然后把这两组参数合并调用；
        //如果还有n组参数会放在下一次递归
        return curried(...args, ...args2);
      };
    }
  };
}
```

### 柯里化占位符

在 lodash 的柯里化实现中有一个功能，可以通过传入一个占位符让某个位置上的参数暂时“占位”，然后在后续的调用中把后面的参数放到这个占位符上。比如

```js
const _ = {};
curryFn(1)(2, 3)(4);
// 相当于
curryFn(1)(_, 3)(2)(4); // 2被填充到了占位符上
```

这个要怎么实现呢？基本思路是，还是按照基本柯里化的方式收集参数，同时收集占位符。对于占位符和参数有不同处理：

- 如果当前元素是占位符，就放入占位符数组，放入的值是当前参数在所有参数的索引
  - 如果当前是占位符且前面有占位符，那么就把前面的占位符删掉，以后面的为准。
- 如果当前元素是正常值并且前面有一个占位符，那么通过索引删除前面的占位符，然后将参数插入占位符所在的 index

实现如下，关键点是收集参数和占位符数组：

```js
function curried(fn, holder) {
  return curry(fn, fn.length, holder, [], []);
}

function curry(fn, argsLen, holder, holderList, argsList) {
  return function (...args) {
    const _argsList = [...argsList];
    const _holderList = [...holderList];
    args.forEach((arg, idx) => {
      if (arg === holder) {
        if (_holderList.length) {
          _holderList.shift();
        }
        _holderList.push(_argsList.length);
      } else {
        if (_holderList.length) {
          const lastHolder = _holderList.shift();
          _argsList.splice(lastHolder, 1, arg); // 删掉占位符，并用参数替换
        } else {
          _argsList.push(arg);
        }
      }
    });
    if (_argsList.length >= argsLen) {
      if (holderList.length === 0) return fn.apply(this, _argsList);
    } else {
      return curry(fn, argsLen, holder, _holderList, _argsList);
    }
  };
}
```

## 手写懒计算函数

> 实现一个无限累加的 sum 函数如下所示：
>
> ```js
> sum(1, 2, 3).valueOf(); //6
> sum(2, 3)(2).valueOf(); //7
> sum(1)(2)(3)(4).valueOf(); //10
> sum(2)(4, 1)(2).valueOf(); //9
> sum(1)(2)(3)(4)(5)(6).valueOf(); // 21
> ```

这道题开始的思路是用上一道的柯里化函数包裹 sum 函数解决。但是问题在于 sum 的参数不确定，不能直接用。

因此实际的思路应该是先收集所有调用的参数，然后给函数一个 valueOf 方法，调用这个方法时计算参数只和即可。

收集参数的方法和柯里化类似：

```js
function sum(...args1) {
  return (...args2) => sum(...args1, ...args2);
}
```

每一次额外调用，都会产生一个递归，合并之前调用的所有参数。

但是这样没有终止条件。在柯里化中终止条件通常是总参数个数达到函数最大参数个数。而这里我们就需要一个 valueOf 方法，当调用这个方法时就终止，并返回计算结果。

```js
function sum(...args1) {
  const f = function (...args2) {
    return sum(...args1, ...args2);
  };
  f.valueOf = () => args1.reduce((x, y) => x + y, 0);
  return f;
}
```

## 手写缓存函数

> 实现一个缓存函数，返回一个处理之后的函数，这个函数多次调用相同的参数时只会调用一次，第一次之后都返回缓存结果。
> 具体参考：https://leetcode.cn/problems/memoize-ii/

缓存的实现是这道题的核心，具体来说方式有两种：

1. 计算 key。不过这里要用到两个 map，一个 map 用于记录具体的 args，然后对应一个 id 值作为 key（其他任何不重复的也行）；另一个 map 就是 id 和函数运行结果的映射。

之所以要两个，是因为题目需要的其实是两个地方的缓存判断，一个是比较各个参数，在参数全部相同的情况下才会取缓存；另一个是存具体的缓存值。

```
map1  args --> id ---> key
map2  id --> value
```

举个栗子，sum 函数接受两个参数。比如第一次调用的是`sum(1,2)`那么就会有`map{1 => 0, 2 => 1}`，每次调用通过 item 获取到 id 并拼接就可以得到专门的 key。如果这两个参数乱序或者不同，那么 key 也肯定不同，将会重新计算缓存。

代码如下：

```js
function memoize(fn) {
  const argToIdMap = new Map();
  const idToValueMap = new Map();
  let id = 0;
  return function (...args) {
    let key = "";
    for (let item of args) {
      // 注意这里的映射关系是item对应id，而不是反过来
      if (!argToIdMap.has(item)) argToIdMap.set(item, id++);
      key += argToIdMap.get(item);
    }
    if (idToValueMap.has(key)) {
      return idToValueMap.get(key);
    } else {
      const res = fn(...args);
      idToValueMap.set(key, res);
      return res;
    }
  };
}
```

2. 利用前缀树，把每个参数看作是一个树节点，整个函数的参数列表就形成了一条查询路径，在查询逻辑的末尾和其他前缀树一样，设置一个值用于表示是否匹配到相同的参数列表，以及值是多少

这种方法的好处在在于节省内存，因为相同的参数只会复用同一个 map，每个 map 存的并不多。

更进一步的方法是，如果参数的某一项是对象类型的，那就使用 weakMap 以节省内存。

```js
class DictNode {
  res = undefined; // 如果这个节点是某个函数参数列表匹配的位置，这里存的就是函数返回值
  save = false; // 类似前缀树的isEnd
  primitive = new Map(); // 存储基本类型参数
  object = new WeakMap(); // 存储引用类型参数
  setResult(res) {
    this.res = res;
    this.save = true;
    return res;
  }
}

function isObject(arg) {
  return typeof arg === "function" || (typeof arg === "object" && arg !== null);
}

function memoize(fn) {
  const root = new DictNode();
  return function (...args) {
    let node = root;
    for (const item of args) {
      const map = isObject(item) ? node.object : node.primitive;
      if (!map.has(item)) map.set(item, new DictNode());
      node = map.get(item);
    }
    if (node.save) return node.res;
    else return node.setResult(fn(...args));
  };
}
```

在 React 中也有类似的实现：https://github.com/facebook/react/blob/ee4233bdbc71a7e09395a613c7dde01194d2a830/packages/react/src/ReactCache.js#L50

## 手写 Promise

### 基础 Promise

原理:<a href="https://juejin.cn/post/6844903625769091079">具体原理可以看这里</a>

1. 首先构造一个类, promise 对象是需要传一个函数的，这里直接作为 constructor 的参数；在 constructor 中做如下配置：

- 设置状态 state，有三种情况：`pending`、`fulfilled`、`rejected`
- 设置 value 用于维持一个值，在后面 resolve 到 then 中，then 也需要这个 value
- 设置 `resolve` 函数，原理就是：
  1. 调用时更改 state 状态
  2. 存储调用时传入的值
- 设置 `rejected`，原理是：
  1. 更改 `state` 为 `rejected`
  2. 设置错误原因
- 设置完上面之后通过传入的 `executor` 传入 `resolve` 和 `rejected`；在外部使用 `resolve` 和 `rejected` 都会直接触发类中的这两个函数

2. 设计 then；then 实际上是类的一个方法，参数为 `onFulfilled` 和 `onRejected`，分别表示成功回调和失败回调

- 调用时先判断 state；如果为 fulfilled，就调用外部自定义的 `onFulfilled` 函数，参数为 `this.value`，也就相当于我们常写的"res"；如果为 `rejected` 同理

3. 添加定时器异步

- 首先在构造函数添加成功存放的数组和失败存放的数组，这两个存的分别是后续外部定义的 `onFulfilled` 函数和 `onRejected` 函数，在“完成的时刻”再从数组中取出调用他们。使用数组的原因是多个 `then`（不是 `then` 链，还没实现）
- 在 `resolve` 和 `rejected` 函数中添加调用上面两个数组中所有函数；这步最为关键，也就是说必须直到状态改变，后续的 `then` 中函数才会被调用，是一种异步效果
- 在 `then` 中设置 `pending` 状态的操作：如果在 `pending` 状态说明 `executor` 函数中有延时执行的任务，这时候就把所有的回调放在成功/失败数组中，等待完成后执行

```js
class Promise {
  constructor(executor) {
    // 初始化state为等待态
    this.state = "pending";
    // 成功的值
    this.value = undefined;
    // 失败的原因
    this.reason = undefined;
    // 成功存放的数组，主要是应对异步任务
    this.onResolvedCallbacks = [];
    // 失败存放的数组
    this.onRejectedCallbacks = [];
    let resolve = (value) => {
      // state改变,resolve调用就会失败
      if (this.state === "pending") {
        // resolve调用后，state转化为成功态
        this.state = "fulfilled";
        // 储存成功的值
        this.value = value;
        // 一旦resolve执行，调用成功数组的函数
        this.onResolvedCallbacks.forEach((fn) => fn());
      }
    };
    let reject = (reason) => {
      // state改变,reject调用就会失败
      if (this.state === "pending") {
        // reject调用后，state转化为失败态
        this.state = "rejected";
        // 储存失败的原因
        this.reason = reason;
        // 一旦reject执行，调用失败数组的函数
        this.onRejectedCallbacks.forEach((fn) => fn());
      }
    };
    // 如果executor执行报错，直接执行reject
    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }
  then(onFulfilled, onRejected) {
    // 状态为fulfilled，执行onFulfilled，传入成功的值
    if (this.state === "fulfilled") {
      onFulfilled(this.value);
    }
    // 状态为rejected，执行onRejected，传入失败的原因
    if (this.state === "rejected") {
      onRejected(this.reason);
    }
    // 当状态state为pending时，说明任务是个异步，需要把任务放在数组中，等到resolve/reject执行再执行
    if (this.state === "pending") {
      // onFulfilled传入到成功数组
      this.onResolvedCallbacks.push(() => {
        onFulfilled(this.value);
      });
      // onRejected传入到失败数组
      this.onRejectedCallbacks.push(() => {
        onRejected(this.reason);
      });
    }
  }
  // catch可以简单实现为执行then的第二个参数
  catch(onReject) {
    this.then(() => {}, onReject);
  }
}
```

如果希望 then 内部通过微任务的方式执行，可以使用 queueMicrotask：

```js
then(onResolve, onReject) {
  if (this.state === "fulfilled" && onResolve) {
    queueMicrotask(() => onResolve(this.value));
  }
  if (this.state === "rejected" && onReject) {
    queueMicrotask(() => onReject(error));
  }
  if (this.state === "pending") {
    onResolve &&
      this.onResolvedCallbacks.push(() => {
        queueMicrotask(() => onResolve(this.value));
      });
    onReject &&
      this.onRejectCallbacks.push(() => {
        queueMicrotask(() => onReject(this.error));
      });
  }
}
```

### then 穿透

`Promise`还有一个很重要的效果就是 then 的穿透，即 then 返回一个`Promise`给下一个 then 继续链式调用。
这个过程的实现主要有以下几点：

- then 可以没有参数，如果没有参数，`onFulfilled`默认向下继续传递，`onRejected`向外冒泡错误
- then 需要返回一个`Promise`，而不是上面的只是进行`onFulfilled`操作。这个`Promise`如果`resolve`就相当于向下穿透到了下一个 then，而其中任何一个`reject`都会被直接冒泡到最外层。
- 这个`Promise`需要判断 then 的返回是不是一个`Promise`；也就是说，由于 then 可以显式返回一个`新的Promise`，源码中`原本要返回的Promise`要和`新的自定义Promise`同步状态：`新的自定义Promise`如果`resolve`，原本的也要`resolve`，反之亦然；
- 上述的同步过程主要通过捕获 then 实现，也就是说根据`新Promise`中 then 的调用判断是哪种情况，然后在`原本的Promise`中进行`resolve`或`reject`。
- 最后，因为上面说过的 Promise 延迟绑定回调的特性，每个修改状态等操作都应该是异步的，保证 then 先绑定完整再进行操作。本来 then 应当是微任务实现，原教程使用了 setTimeout 宏任务代替，但`queueMicrotask`是更好的选择。（微任务：https://developer.mozilla.org/zh-CN/docs/Web/API/HTML_DOM_API/Microtask_guide）

下面是具体实现，详细解释在注释中：

```js
const PENDING = "PENDING";
const FULFILLED = "FULFILLED";
const REJECTED = "REJECTED";

class MyPromise {
  constructor(executor) {
    this.status = PENDING;
    this.value = undefined;
    this.reason = undefined;
    this.onResolvedCallback = [];
    this.onRejectedCallback = [];

    const resolve = (value) => {
      if (this.status !== PENDING) return;
      this.status = FULFILLED;
      this.value = value;

      this.onResolvedCallback.forEach((fn) => fn());
    };

    const reject = (reason) => {
      if (this.status !== PENDING) return;
      this.status = REJECTED;
      this.reason = reason;
      this.onRejectedCallback.forEach((fn) => fn());
    };

    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }

  then(onFulfilled, onRejected) {
    //判断参数是不是函数，如果不是就直接resolve或者直接reject
    onFulfilled =
      typeof onFulfilled === "function" ? onFulfilled : (val) => val;
    onRejected =
      typeof onRejected === "function"
        ? onRejected
        : (reason) => {
            throw reason;
          };

    //返回一个Promise
    const thenPromise = new MyPromise((resolve, reject) => {
      if (this.status === FULFILLED) {
        //创建一个微任务
        queueMicrotask(() => {
          try {
            //先执行一次onFulfilled，取得当前then的调用结果
            let returnValueFromThen = onFulfilled(this.value); //这个值是当前then的返回值，有可能是promise（then返回一个promise供下个then使用）
            //下面这个函数可以看作是一系列判断和同步后进行具体的resolve和reject操作
            resolvePromise(thenPromise, returnValueFromThen, resolve, reject);
          } catch (err) {
            reject(err);
          }
        });
      }
      if (this.status === REJECTED) {
        queueMicrotask(() => {
          try {
            let returnValueFromThen = onRejected(this.reason);
            resolvePromise(thenPromise, returnValueFromThen, resolve, reject);
          } catch (err) {
            reject(err);
          }
        });
      }
      if (this.status === PENDING) {
        this.onResolvedCallback.push(() => {
          queueMicrotask(() => {
            try {
              let returnValueFromThen = onFulfilled(this.value);
              resolvePromise(thenPromise, returnValueFromThen, resolve, reject);
            } catch (err) {
              reject(err);
            }
          });
        });
        this.onRejectedCallback.push(() => {
          queueMicrotask(() => {
            try {
              let returnValueFromThen = onRejected(this.reason);
              resolvePromise(thenPromise, returnValueFromThen, resolve, reject);
            } catch (err) {
              reject(err);
            }
          };
        });
      }
    });

    return thenPromise;
  }
}

const resolvePromise = (thenPromise, returnValueFromThen, resolve, reject) => {
  if (returnValueFromThen === thenPromise) {
    /*不能return自己，即不能
      const p1 = promise.then(value => {
        return p1
      })
    */
    return reject(
      new TypeError("Chaining cycle detected for promise #<Promise>")
    );
  }

  /**理论上这时直接resolve(returnValueFromThen)就可以了，但是还需要考虑then返回一个Promise的情况，要确保这个新Promise状态和现在的同步
    *也就是说，如果这个新Promise失败，那么当前这个Promise应该直接reject，反之亦然
    */
  let settledCalled; //是否调用过resolve或reject，如果有后续都无视
  if (returnValueFromThen instanceof Promise) {
    //如果then的返回值是个Promise
    try {
      const thenFromThen = returnValueFromThen.then; //这里是then返回的那个Promise中的then，后面叫做套娃then（multiThen）
      if (typeof thenFromThen === "function") {
        /**正常调用这个套娃then（"multiThen"），绑定原先的Promise上下文（也就是then返回的那个Promise）
          *根据这个套娃then的结果决定当前then的结果
          *如果套娃then resolve，当前就resolve，反之亦然
          *因此给这个套娃then传两个回调，触发哪个就执行哪个
          */
        thenFromThen.call(
          returnValueFromThen,
          (multiThenFulfilled) => {
            //套娃then resolve的回调，如果套娃then resolve，当前的then也就resolve
            if (settledCalled) return;
            settledCalled = true; //如果之前被调用过就返回，否则就执行并设置为已调用过
            resolve(multiThenFulfilled); //这个地方更好的方法是递归调用，因为套娃then 还可能再返回一个promise；但是不考虑那么复杂的情况，直接resolve是可以的
          },
          (multiThenRejected) => {
            //如果套娃then被rejected，当前then也rejected
            if (settledCalled) return;
            settledCalled = true;
            reject(multiThenRejected);
          }
        );
      } else {
        //如果then的Promise返回一个普通值，就直接resolve
        resolve(returnValueFromThen);
      }
    } catch (err) {
      //这个trycatch是捕获访问套娃then中出现的任何错误的，会向上冒泡
      if (settledCalled) return;
      settledCalled = true;
      reject(err);
    }
  } else {
    //如果then的返回值并不是一个Promise就直接resolve
    resolve(returnValueFromThen);
  }
};
```

### Promise Api

手写 Promise Api 类型的题目一定要注意：
注意什么时候是需要创建一个新的 promise，什么时候是一直只用原来的 promise。
在循环、递归等方法中，要注意**每次需要调用函数创建一个新的 promise**，而不是一直在一个 promise 上操作。一个 promise 操作之后状态就会改变，后续的 then、await 等都不会再生效。

尤其注意的题目：

- Promise.retry 的递归调用，应该是每次创建新的 promise 进行重试，而不是重试旧的
- 红绿灯，同理，每次 await 之后下一次应该是一个新的 promise，否则旧的 promise await 不会生效，递归就会死循环。

#### `Promise.prototype.finally`

无论当前 Promise 是成功还是失败，调用 finally 之后都会执行 finally 中传入的函数，并且将值原封不动的往下传。

因此可以看作 finally 内部实际上是执行了一个当前 Promise 的 then，在 then 中执行回调函数，然后把该 then 接收到的参数（即上一层传下来的值）继续传下去

```js
Promise.prototype.finally = function (callback) {
  this.then(
    (res) => Promise.resolve(callback()).then(() => res),
    (error) =>
      Promise.resolve(callback()).then(() => {
        throw error;
      })
  );
};
```

考虑到 callback 可能是个 promise，就需要 Promise.resolve 包裹一下，但不用获取回调的返回值，而是返回 then 从上一层接到的返回值。

#### `Promise.all`

主要逻辑是把一个数组中的 promise 遍历依次执行`.then`，并把返回值（resolve 值）按顺序放入数组中，等待全部完成后再统一`resolve`。
因此内部实际上返回了一个 Promise，当全部完成时进行 resolve 操作，如果有一个发生 reject 则全部 reject。

> 注意执行多个异步任务是并行执行的，并行执行完成后会一起 resolve，消耗时间是最长的那个任务。

```js
MyPromise.all = function (values) {
  if (!Array.isArray(values)) {
    //首先要求参数必须可迭代
    const type = typeof values;
    return new TypeError(`TypeError: ${type} ${values} is not iterable`);
  }

  return new MyPromise((resolve, reject) => {
    let resultArr = [];
    let orderIndex = 0;
    const processResultByKey = (result, index) => {
      resultArr[index] = result; //按照顺序把刚刚的resolve值放到数组
      if (++orderIndex === values.length) {
        //如果满了就全部resolve掉
        resolve(resultArr);
      }
    };
    for (let i = 0; i < values.length; i++) {
      //遍历参数，对每个参数执行then；
      // 由于for循环是同步的，因此所有任务都是同步启动、并行执行的
      let value = values[i];
      if (value && typeof value.then === "function") {
        value.then((res) => {
          processResultByKey(res, i);
        }, reject); //如果有一个触发reject，相当于触发了整个的reject，直接退出
      } else {
        //如果参数有不是Promise的，不用then直接传入
        processResultByKey(value, i);
      }
    }
  });
};
```

稍微简化一点

```js
function pAll(_promises) {
  return new Promise((resolve, reject) => {
    // Iterable => Array
    const promises = Array.from(_promises);
    // 结果用一个数组维护
    const r = [];
    const len = promises.length;
    let count = 0;
    for (let i = 0; i < len; i++) {
      // Promise.resolve 确保把所有数据都转化为 Promise
      // 为什么不直接 promise[i].then, 因为promise[i]可能不是一个promise
      Promise.resolve(promises[i])
        .then((res) => {
          // 因为 promise 是异步的，保持数组一一对应
          r[i] = res;

          // 如果数组中所有 promise 都完成，则返回结果数组
          if (++count === len) {
            resolve(r);
          }
          // 当发生异常时，直接 reject
        })
        .catch(reject);
    }
  });
}
```

关键在于“依次执行”，可以看到上面的循环中，每个小 Promise 都在 then 之后把结果放入数组，待数组放满后才 resolve。

#### `Promise.allSettled`

和 Promise.all 的唯一区别，就是错误并不直接 reject，而是像收集正常结果一样收集错误，最后等到装满时返回。

```js
function promiseAllSettled(_promises) {
  return new Promise((resolve, reject) => {
    // Iterable => Array
    const promises = Array.from(_promises);
    // 结果用一个数组维护
    const r = [];
    const len = promises.length;
    let count = 0;
    for (let i = 0; i < len; i++) {
      // Promise.resolve 确保把所有数据都转化为 Promise
      // 为什么不直接 promise[i].then, 因为promise[i]可能不是一个promise
      Promise.resolve(promises[i]).then(
        (res) => {
          // 因为 promise 是异步的，保持数组一一对应
          r[i] = res;
          // 如果数组中所有 promise 都完成，则返回结果数组
          if (++count === len) {
            resolve(r);
          }
        },
        (err) => {
          r[i] = err;
          if (++count === len) {
            resolve(r);
          }
        }
      );
    }
  });
}
```

#### `Promise.race`

如果让 Promise 在 then 之后立刻就进行 resolve，相当于让每个小 Promise 中最快的返回，就可以实现`Promise.race`：

```js
MyPromise.race = function (promises) {
  return new Promise((resolve, reject) => {
    for (let i = 0; i < promises.length; i++) {
      let val = promises[i];
      if (val && typeof val.then === "function") {
        //谁先执行完谁直接resolve
        val.then(resolve, reject);
      } else {
        // 普通值
        resolve(val);
      }
    }
  });
};
```

#### `Promise.any`

和 Promise.race 唯一的区别在于，它会 resolve 第一个达到成功状态的的 promise，而不是只要有一个成功或失败就会 resolve。
如果没有成功的 promise，就会 reject 一个`AggregateError`，还会包含原因。

```js
MyPromise.any = function (promises) {
  return new Promise((resolve, reject) => {
    let errs = 0;
    for (let i = 0; i < promises.length; i++) {
      let val = promises[i];
      if (val && typeof val.then === "function") {
        //谁先执行完谁直接resolve
        val.then(resolve, (err) => {
          if (++errs === promises.length)
            reject(
              new AggregateError(`No Promise in Promise.any was resolved`)
            );
        });
      } else {
        // 普通值
        resolve(val);
      }
    }
  });
};
```

#### `Promise.map`限制并发

通过传入 limit 参数限制最多同时并发的 promise 数量，每一个任务完成之后及时替换。

思路：

1. 把执行任务和并发启动分开，通过 for 循环的形式一次同时启动 limit 个任务，每个任务自行递归选择下一个
2. 和 promise.all 一样的顺序判断，通过设定 index 使得输出按顺序，并通过设定 count 判断是否全部运行完成。

代码如下：

```js
const MyPromiseMap = (promises, limit) => {
  return new Promise((resolve, reject) => {
    const results = [];
    let count = 0;
    let index = 0; // 正在执行的序号
    for (let i = 0; i < Math.min(limit, promises.length); i++) {
      // 同时启动limit个任务
      next();
    }
    function next() {
      const currIndex = index; // 存一下当前index
      index++; //每次递归都先自增
      if (promises[currIndex]) {
        // 递归边界
        // 注意递归边界一定是在next的同步代码里的
        promises[currIndex].then((res) => {
          results[currIndex] = res;
          if (++count === promises.length) resolve(results);
          else next(); // 每个任务执行完后，单独递归调用下一个
        }, reject);
      }
    }
  });
};
```

#### `Promise.retry`自动重试

`Promise.retry(promiseFn, retryTimes)`，接受一个创建 promise 的函数，如果该 promise 正常返回就正常 resolve，如果 reject 就重试，最多重试 retryTimes 次。

实现方式有两种，一种是递归，一种是 await 循环：

```js
// 递归内部函数
Promise.retry = function (promise, retryTimes) {
  let index = 0;
  return new Promise((resolve, reject) => {
    function promiseFn(promise, retryTimes, err) {
      if (retryTimes === 0 && err != null) {
        reject(err);
        return;
      }
      promise().then(
        (res) => {
          resolve(res);
          return;
        },
        (err) => {
          console.log(index++);
          promiseFn(promise, retryTimes - 1, err);
        }
      );
    }
    promiseFn(promise, retryTimes, null);
  });
};

// 直接调用Promise.retry也行
Promise.retry = function (promise, retryTimes) {
  return new Promise((resolve, reject) => {
    if (retryTimes === 0 && err != null) {
      reject(err);
      return;
    }
    promise().then(
      (res) => {
        resolve(res);
        return;
      },
      (err) => {
        console.log(index++);
        Promise.retry(promise, retryTimes - 1);
      }
    );
  });
};

// while写法
Promise.retry = function (promise, times) {
  return new Promise(async (resolve, reject) => {
    while (times > 0) {
      try {
        const res = await promise();
        resolve(res);
      } catch (e) {
        times--;
        if (times === 0) reject(e);
      }
    }
  });
};
```

测试函数：

```js
const getMyPromise = () =>
  new Promise((res, rej) => {
    setTimeout(() => {
      rej("error");
    }, 1000);
  });

Promise.retry(getMyPromise, 3).catch(console.log);
```

**注意，函数里调用的 promiseFn 应该是一个创建 promise 的函数，而不是同一个 promise**。同一个 promise 状态改变之后就不能再 then 了。记住这种情况，一定要重新调用创建 promise 的函数创建一个新的 promise！

```js
getPromiseFn().then(...);

// 不能，因为promise不能多次调用
promise.then(...)
```

## 手写 async pool

async pool 和 promise map 的功能类似，即限制 promise 任务的并发数量，但是实现方式不同，下面是一种实现方式：

```js
// async pool一般不直接接收promise数组，而是通过函数生成
async function asyncPool(poolLimit, array, iteratorFn) {
  const ret = []; // 存储所有的异步任务
  const executing = []; // 存储正在执行的异步任务
  for (const item of array) {
    // 调用iteratorFn函数创建promise
    const p = Promise.resolve().then(() => iteratorFn(item, array));
    ret.push(p); // 同步任务，保存新的异步任务

    // 当poolLimit值小于或等于总任务个数时，进行并发控制
    if (poolLimit <= array.length) {
      // 创建e，e在p执行完后从正在执行的任务数组里删掉
      const e = p.then(() => executing.splice(executing.indexOf(e), 1));
      executing.push(e); // 保存正在执行的异步任务
      if (executing.length >= poolLimit) {
        await Promise.race(executing); // 等待较快的任务执行完成
      }
    }
  }
  return Promise.all(ret);
}
```

这个的实现逻辑是这样的：

1. 遍历数组，为每个数组中的项创建 promise 对象。需要注意的是，这个遍历在这里很重要，下面会说到
2. 创建 p，然后创建 e。e 就相当于是执行完具体的任务 p 之后，再在 executing 数组中把 e 删掉；也就是说在这个 pool 内，谁执行完谁就把自己顺便清除掉了
3. 使用 Promise.race 执行任务队列。相当于让 pool 中的任务竞争完成。下面举个例子

```
比如现在有三个任务，executing数组内 = [a,b,c]
调用Promise.race执行三个任务，假设a先执行完
a就是上面的e，因此会从executing数组内删掉自己，变成[b,c]
然后，跳出await，进入下一次循环，取数组中的下一项（比如e）再继续，executing数组变成[b,c,e]
重复上面几步，直到executing.length < poolLimit，最后剩下的几个任务并发执行。
```

核心有两个：

- 每个任务执行完成后会从数组中删掉自己，腾出空间；
- 利用 `await Promise.race(executing)`，等待正在执行任务列表中较快的任务执行完成之后，才会继续执行下一次循环。

## 实现异步 sum/add

> 请实现以下 sum 函数，只能调用 add 进行实现
>
> ```js
> /*
>   请实现一个 sum 函数，接收一个数组 arr 进行累加，并且只能使用add异步方法
>   add 函数已实现，模拟异步请求后端返回一个相加后的值
> */
> function add(a, b) {
>   return Promise.resolve(a + b);
> }
>
> function sum(arr) {}
> ```
>
> 追加问题：如何控制 add 异步请求的并发次数

这道题实际上是考察异步并行和串行。add 函数模拟的是一个返回 promise 的函数，要求对这个函数传入数组中的参数，依次计算。

基本的思路是串行实现，即 for await。

```js
async function sum(arr) {
  let res = arr[0];
  for (let i = 1; i < arr.length; i++) {
    res = await add(res, arr[i]);
  }
  console.log(res);
}
```

如果不用 await，就可以这样：

```js
function sum(arr) {
  let res = Promise.resolve(0);
  for (let i = 0; i < arr.length; i++) {
    res = res.then((val) => add(val, arr[i]));
  }
  return res;
}

// reduce也可
function sum(arr) {
  return arr.reduce(
    (pre, cur) => pre.then((res) => add(res, cur)),
    Promise.resolve(0)
  );
}
```

如果考虑并行，既然是把数组变为 promise 形式，那么首先想到的肯定是`Promise.all`。我们把数组的每两项都变成 add 函数，再按照归并的方式，两两合并计算即可。
关键就在这个“归并”，到底要怎么把计算的结果两两合并？
首先`Promise.all`会返回一个结果数组。第一步中会返回所有的两两计算的结果数组，并且这个数组是一个`number[]`。因此我们可以**递归**再把这个数组传入 sum，最后直到数组只有一项。

```js
function sum(arr) {
  if (arr.length === 1) return arr[0];
  // 分割数组用到了上面的chunk，将数组分为n个两份
  // 这里注意，因为有可能两两分组还剩下一个，所以需要判断一下y是否存在
  const promises = chunk(arr, 2).map(([x, y]) => (y ? add(x, y) : x));
  return Promise.all(promises).then((resArr) => sum(resArr));
}
```

但是这个方法仍然有问题。比如有 10000 个数据，那第一次就会发送 5000 个请求，网络拥堵了，控制成只能同时发送 10 个请求怎么办？
这就需要借用 Promise.map。只需要把 Promise.all 改成 map 即可。
（假设 Promise.map 代码已有）

```js
function sum(arr, limit) {
  if (arr.length === 1) return arr[0];
  const promises = chunk(arr, 2).map(([x, y]) => (y ? add(x, y) : x));
  return promiseMap(promises, limit).then((resArr) => sum(resArr, limit));
}
```

---

并行还有一种写法更直观，本质上也是每次放入相邻两个值的 add 调用结果，然后用 promise.all 执行，再把结果（原先是 n 个 promise，结果就是 n/2 个 promise）递归

```js
async function sum(...args) {
  if (args.length === 1) return args[0];
  const tasks = [];
  for (let i = 0; i < args.length; i += 2) {
    tasks.push(add(args[i], args[i + 1] || 0));
  }
  const results = await Promise.all(tasks);
  return sum(...results);
}
```

## 手写 call apply 和 bind

call 和 apply 的实现思路类似，都是作用在函数上，参数是要设置的上下文和调用时传入的参数。

先来说 call：
call 方法应该在`Function.prototype`上，并且 call 内部的 this 应该是点号前面的函数。因此可以让这个函数在传入参数（context）下调用，再返回结果即可。
关键在于，怎么实现“函数在 context 下调用”：可以给这个 context 添加一个当前函数属性（即 this）并调用，然后删去并返回返回值即可

```js
Function.prototype.mycall = function (context, ...args) {
  if (typeof this !== "function") return; //this就是call前面的函数，比如fn.call(...)的this就是fn
  context = context || window; //如果没有指定参数就是window
  context.fn = this; //给context添加一个属性，这个属性就是当前的函数
  let res = context.fn(...args); //在context上调用函数
  delete context.fn; //删去这个属性
  return res;
};
```

apply 和 call 只差在参数传递方式，其实改一点就好了。

```js
Function.prototype.myapply = function (context, args) {
  //...
};
```

但是这两个方法都有漏洞：如果 fn 和 context 上某个属性同名，就会删去原先的方法。
因此可以把 fn 改为 Symbol，保证不会影响旧属性：

```js
Function.prototype.mycall = function (context, ...args) {
  if (typeof this !== "function") return;
  context = context || window;
  const fnSymbol = Symbol();
  context[fnSymbol] = this;
  let res = context[fnSymbol](...args);
  delete context[fnSymbol];
  return res;
};
```

---

对于 bind，其实主要是返回值不同，bind 会返回绑定好的函数，因此把调用封装在要返回的函数内部即可：
最简单的 bind：

```js
Function.prototype.myBind = function (context, ...args) {
  // 箭头函数保证返回的this是外层的
  return (...rest) => this.apply(context, [...args, ...rest]);
};
```

当然还要亿些细节，详见https://github.com/sisterAn/JavaScript-Algorithms/issues/81

```js
Function.prototype.mybind = function (context, ...args) {
  if (typeof this !== "function") return;
  context = context || window;
  const fn = this;
  function Fn() {
    //这里是考虑到返回的函数有可能作为构造函数的情况
    //两个this不一样；作为调用函数的this现在是fn；现在的this是fn中的this，即如果返回的函数用作构造函数，这个this应该指向实例
    //因此需要判定一下fn内部的this是不是返回函数的实例，如果是的话就不改变指向
    return fn.apply(
      this instanceof Fn ? this : context,
      args.concat(arguments)
    );
  }
  Fn.prototype = Object.create(fn.prototype);
  return Fn;
};
```

bind 的链式调用会无视掉后面的绑定，多次调用只会以第一个为准。也就是说 bind 返回的函数不能再用 bind 绑定为其他的上下文和参数。原因是第一次绑定之后的`context`不能再改变了

```js
return func.call(
  context, // 这个context是死绑的
  ...args,
  ...args1
);
```

如果能够改变这个死绑，就可以实现软绑定
为此可以单独实现一个`softBind`，多次调用以最后一个为准。其原理是`!this || this === window ? context : this`，即如果 this 为空就把新的 context 绑定，相当于覆盖了前面的绑定。

```js
Function.prototype.softBind = function (context, ...args) {
  const fn = this;
  const bound = function () {
    return fn.apply(!this || this === window ? context : this, [
      ...arguments,
      ...args,
    ]);
  };
  return bound;
};
```

## 手写 instanceof

原理很简单，参数传入父子，获取父对象（函数）的`prototype`和子对象的`__proto__`比较，如果没有就沿着子对象的`__proto__`循环向上找，直到为 null 或找到为止

```js
function Instanceof(parent, child) {
  let cp = Object.getPrototypeOf(child);
  const pp = parent.prototype;
  while (cp) {
    if (pp === cp) return true;
    cp = Object.getPrototypeOf(cp);
  }
  return false;
}
```

## 手写 sleep/delay

sleep 函数使后面代码阻塞：

```js
function sleep(delay){
  return new Promise(resolve=>{
    setTimeout(resolve,delay)
  })
}

async testFn(){
  //...
  await sleep(1000)
  //后续代码
}

//或者
sleep(1000).then(()=>{
    //后续代码
})
```

delay 使被传入的部分延迟执行，并阻塞后面的代码：

```js
function delay(func, second, ...args) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(func(...args));
    }, seconds);
  });
}

async function testFn() {
  //...
  const res = await delay((name) => `hello,${name}`, 1000, "jack");
  res; // 1秒后返回 hello,jack
}
```

## 手写 compose 函数

compose 函数实现的效果类似数学中的多次函数。比如

```
f(x) = x + 10
g(x) = x * 5
f(g(x)) = x*5 + 10
```

要实现的 compose 也要满足这个效果：

```js
const fx = (x) => x + 10;
const gx = (x) => x * 5;
const fgx = compose(fx, gx);
fgx(5); // 5 * 5 + 10 = 35
```

思路如下：

compose 为了符合数学上的定义，应当是从内向外计算的。因此应该按照其传入的参数从右向左计算。最简单的实现是利用 reduce：

```js
const compose = (...fns) => {
  return fns.reduce((f, g) => {
    return (...args) => f(g(...args));
  });
};
```

reduce 内部每次返回的都是一个函数。当 reduce 叠加时，相当于不断叠加**外部**包裹的函数。

```js
// 第一次
total = (...args) => f(g(...args))
// 后续
f = (...args) => f(g(...args))
g = h(x)
total = (...args) => f(h(...args)) = f(g(h(..args)))
// 依次类推
```

## 手写 immerjs 的 produce 函数效果

参考：https://leetcode.cn/problems/immutability-helper/solutions/2287059/proxychao-shi-by-yuan-zhi-b-e667/

## 手写对象扁平化和逆扁平化

### 扁平化

要求：

```js
const obj = {
  a: {
    b: 1,
    c: 2,
    d: { e: 5 },
  },
  b: [1, 3, { a: 2, b: 3 }],
  c: 3,
};

//flatten(obj) 结果返回如下
// {
//  'a.b': 1,
//  'a.c': 2,
//  'a.d.e': 5,
//  'b[0]': 1,
//  'b[1]': 3,
//  'b[2].a': 2,
//  'b[2].b': 3
//   c: 3
// }
```

是一个不需要任何剪枝的回溯问题。从根对象向下遍历到值为非对象为止，把沿途的路径记录下来，插入到新对象中。

```js
const obj = {
  a: {
    b: 1,
    c: 2,
    d: { e: 5 },
  },
  b: [1, 3, { a: 2, b: 3 }],
  c: 3,
};

const isObject = (val) => typeof val === "object" && val !== null;

function flatten(obj) {
  if (!isObject(obj)) return;
  const res = {};
  const dfs = (cur, prefix) => {
    if (!isObject(cur)) {
      const key = prefix.join(".");
      res[key] = cur;
    }
    for (let key in cur) {
      if (Array.isArray(cur)) prefix.push(`[${key}]`);
      else prefix.push(key);
      dfs(cur[key], prefix);
      prefix.pop();
    }
  };
  dfs(obj, []);
  return res;
}
```

### 逆扁平化

> 写一个函数，将 entry 格式转化成 output 的格式：
>
> ```js
> var entry = {
>   "a.b.c.dd": "value",
>   "a.d.xx": "val",
>   "a.e": "v",
> };
> // 要求转换成如下对象
> var output = {
>   a: {
>     b: {
>       c: {
>         dd: "value",
>       },
>     },
>     d: {
>       xx: "val",
>     },
>     e: "v",
>   },
> };
> ```

实现思路
递归。每次处理一个属性的键，有点像前缀树的思路

- 每次递归的 level 表示“层级”，即键数组的序号。比如键为`a.b.c.d`，拆成数组`[a,b,c,d]`，那么每次递归就是从第一项开始，每递归一次向后走一个，然后把上一层创建的对象传下去，本层递归在上一层创建的对象上添加属性
  - 如果属性存在，那就继续往下走
  - 如果属性不存在，就创建属性，初始化为空对象
- 当`level === keysArr.length - 1`，表示走到了最后一个属性，这时就把值赋给这个属性即可。

这是基础的版本，暂时没有处理数组的情况。

```js
const deFlat = (object) => {
  if (!object) return null;
  const entries = [];
  for (const [key, value] of Object.entries(object)) {
    const keysArr = key.split(".");
    entries.push([keysArr, value]);
  }
  const res = {};
  for (let i = 0; i < entries.length; i++) {
    dfs(0, res, entries[i][0], entries[i][1]);
  }

  function dfs(level, parentObj, keysArr, value) {
    if (level >= keysArr.length) return;
    const key = keysArr[level];
    if (keysArr[level] != null && level < keysArr.length - 1) {
      if (parentObj[key] != null && typeof parentObj[key] === "object") {
        dfs(level + 1, parentObj[key], keysArr, value);
      } else {
        parentObj[key] = {};
        dfs(level + 1, parentObj[key], keysArr, value);
      }
    } else if (level === keysArr.length - 1) {
      parentObj[key] = value;
    }
  }
  return res;
};
```

数组情况：数组的处理其实主要要在解析键上增加逻辑。之前解析键的方式是直接 split，现在需要遍历 str，根据得到的字符不同解析键。具体来说：

- 如果当前字符不是`.`/`[`/`]`的一个，那么就继续向后，直到统计完标准键为止
- 如果是`.`，那么把前面的 key 视作是一个对象的 key，在返回结果中加入一个`type: 'object'`字段，表示这个键应该创建一个对象
- 如果是`[`，同理，这时加入的是`type:'array'`表示这个键应该创建一个数组

```js
const parseKey = (str) => {
  if (!str || !str?.length) return [];
  const keys = [];
  const isValidKey = (char) =>
    char != null && char !== "." && char !== "[" && char !== "]";
  let index = 0;
  while (index < str.length) {
    let word = "";
    while (isValidKey(str[index]) && index < str.length) {
      word += str[index++];
    }
    if (index < str.length) {
      if (str[index] === ".") {
        keys.push({
          key: word,
          type: "object",
        });
      } else if (str[index] === "[") {
        keys.push({
          key: word,
          type: "array",
        });
      } else if (str[index] === "]") {
        if (str[index + 1] === ".") {
          keys.push({
            key: word,
            type: "object",
          });
          index++; // 跳过右括号
        }
      }
      index++;
    }
    // 处理最后一个属性是点的情况，比如`a[0].c`，最后一个c
    if (index >= str.length && word.length > 0) {
      keys.push({
        key: word,
        type: "object",
      });
    }
  }
  return keys;
};
```

然后在代码中遍历这个数组，根据 type 创建不同的对象即可。

```js
// 和上面的代码方法略有出入，逻辑是一样的
const decodeObject = (object) => {
  const res = {};
  for (const key of Object.keys(object)) {
    const keysArr = parseKey(key);
    dfs(0, keysArr, res, object[key]);
  }

  function dfs(level, keys, obj, value) {
    if (level === keys.length - 1) {
      obj[keys[level].key] = value;
      return;
    }
    const { key, type } = keys[level];
    if (Object.keys(obj).includes(key)) {
      dfs(level + 1, keys, obj[key], value);
    } else {
      obj[key] = type === "object" ? {} : []; //这里
      dfs(level + 1, keys, obj[key], value);
    }
  }
  return res;
};
```

## 手写虚拟 dom 转化为真实 dom

虚拟 dom 的大致结构如下：

![](https://pic.imgdb.cn/item/637cf98116f2c2beb1f0c0fc.jpg)

需要转化为真实 dom，其实也就是递归遍历虚拟 dom 对象，然后利用几个创建 dom 对象和插入 dom 的 api 实现。
要用到的 api 包括：

- `document.createElement(type)`，其中 type 设置为 tag 即可，但对于 tag 为非 dom 元素节点需要特殊处理
- `document.createTextNode(value)`，创建文本节点，对于类型为非对象而是字符串的元素可以创建文本节点
- `node.setAttribute(attr,value)`，为节点设置属性
- `node.appendChild(node)`，插入创建的 dom 节点

代码如下：

```js
function render(vnode, container) {
  let node = null;
  if (typeof vnode === "string") {
    node = document.createTextNode(vnode);
  } else if (typeof vnode === "object") {
    const { tag, key, children, attrs } = vnode;
    node = document.createElement(tag);
    if (Object.keys(attrs).length !== 0) {
      for (const attr of Object.keys(attrs)) {
        node.setAttribute(attr, attrs[attr]);
      }
    }
    if (Object.keys(children).length !== 0) {
      for (const child of children) {
        render(child, node);
      }
    }
  }
  container.appendChild(node);
}
```

vnode 的类型可能有多种。在 React 中一共有七种 vnode 类型；除去不好处理的 Fragment 类型，其他的类型还包括：

- 数组
- 组件，包括函数组件和类组件
- 三元运算
- 函数执行

后两个其实在 jsx 解析过程中就转化为了具体的变量，因此需要额外考虑的也只有组件和数组两种情况。
如果 vnode 是数组，那就再递归一次，但 container 不改变
如果 vnode 是函数，就执行，得到的返回值作为新的 vnode 操作即可。

```js
function render(vnode, container) {
  let node = null;
  const type = Object.prototype.toString.call(vnode).slice(8, -1);
  if (type === "String") {
    node = document.createTextNode(vnode);
  } else if (type === "Object") {
    const { tag, key, children, attrs } = vnode;
    node = document.createElement(tag);
    if (Object.keys(attrs).length !== 0) {
      for (const attr of Object.keys(attrs)) {
        node.setAttribute(attr, attrs[attr]);
      }
    }
    if (Object.keys(children).length !== 0) {
      for (const child of children) {
        render(child, node);
      }
    }
  } else if (type === "Array") {
    console.log(container);
    for (const val of vnode) {
      render(val, container);
    }
    return;
  } else if (type === "Function") {
    const res = vnode();
    render(res, container);
    return;
  }
  container.appendChild(node);
}
```

还有一种方法，可以让 render 函数返回一个创建好的对象。如下代码：
具体可以根据虚拟 dom 结构调整，但是整体思路就是这样。

```js
const getType = (val) => Object.prototype.toString.call(val).slice(8, -1);

const render = (node) => {
  if (!node) return null;
  const nodeType = getType(node);
  if (nodeType === "String") {
    return document.createTextNode(node);
  } else if (nodeType === "Object") {
    const element = document.createElement(node.tag);
    // 对象类型的虚拟dom，如果有children就遍历
    for (const child of node.children) {
      // 依次渲染子元素并插入到element上
      const childElement = render(child);
      // 这里是针对children内的某一项可能是数组的情况，比如列表渲染
      // 如果是数组就会返回一个数组，里边是创建好的元素，这里遍历并插入element
      if (Array.isArray(childElement)) {
        for (const cElem of childElement) {
          element.appendChild(cElem);
        }
      } else element.appendChild(childElement);
    }
    return element;
  } else if (nodeType === "Array") {
    // 处理node是数组的情况。返回一个数组
    const elements = [];
    for (const elem of node) {
      elements.push(render(elem));
    }
    return elements;
  }
  return null;
};

// 虚拟dom结构
// const virtualNode = {
//   type: "div",
//   children: [
//     {
//       type: "div",
//       children: ["aaa"],
//     },
//     {
//       type: "p",
//       children: ["bbb"],
//     },
//     {
//       type: "ul",
//       children: [
//         [
//           { type: "li", children: ["a"] },
//           { type: "li", children: ["b"] },
//           { type: "li", children: ["c"] },
//         ],
//       ],
//     },
//   ],
// };
```

## 手写类似Vue模板字符串转化

即实现类似效果：

```js
const html = parse(
  `<div>{{content}}, I am {{ user.name }}, here is my age: {{ages[0].val }}</div>`,
  {
    content: "hello",
    user: {
      name: "zzx",
    },
    ages: [
      {
        val: 18,
      },
      27,
      36,
    ],
  }
);

html;
// <div>hello, I am zzx, here is my age: 18</div>
```

这道题其实是两道题的结合：即一方面通过正则表达式取出双括号内部的值，一方面通过类似lodash.get方法来获取对象的具体值。

正则表达式应该匹配的是
- 双括号内部
- 双括号两边包含0-n个空格
- 可能有点运算符或中括号

因此：
```js
const parseRegExp = /\{\{(\s?[a-zA-Z0-9\.\[\]]+\s?)\}\}/;
```

然后通过exec方法获取到括号内的匹配值。
**注意**：关键点来了，exec方法只能取到其中的第一个匹配结果，如果想匹配完全就需要用循环
每次去通过replace修改原字符串，然后把修改后的字符串再做exec，直到返回null为止

```js
while (parseRegExp.exec(str) != null) {
  const content = parseRegExp.exec(str)[1];
  let val = getDataByKeys(data, content);
  str = str.replace(`{{${content}}}`, val);
}
```

替换的方式可以直接用拼接字符串替换。这里的content就是之前匹配取出来的内容，也包括空白、点、中括号等各个部分，因此可以完全替换掉。

最后就是实现一个类似lodash.get的效果。
全部代码如下：

```js
const getDataByKeys = (obj, keyStr) => {
  // 把数组索引替换为点访问索引的方法
  keyStr = keyStr.trim().replace(/\[(\w+)\]/, ".$1");
  const keys = keyStr.split(".");
  let res = obj;
  for (let i = 0; i < keys.length; i++) {
    if (res[keys[i]] != null) res = res[keys[i]];
    else return null;
  }
  return res;
};

const parse = (str, data) => {
  const parseRegExp = /\{\{(\s?[a-zA-Z0-9\.\[\]]+\s?)\}\}/;
  while (parseRegExp.exec(str) != null) {
    const content = parseRegExp.exec(str)[1]; // 这里要取第二项
    let val = getDataByKeys(data, content);
    if (
      typeof val !== "string" &&
      typeof val !== "number" &&
      typeof val !== "boolean"
    )
      val = "";
    str = str.replace(`{{${content}}}`, val);
  }
  return str;
};
```

## 手写 JSON.stringify

利用递归逐层解析并转化为 json 格式。
注意转化过程中有一些特殊值会被忽略或改写：

- 对象中的 function、symbol、undefined 会被忽略，即直接没有这个属性；boolean、number、string 会保持原样
- 数组中的 function、symbol、undefined 会被改为 null

具体实现上，对于某一层对象，设置一个数组用于存储当前对象的每个值的遍历结果。对于每个值都递归一次，非对象会返回原值或忽略掉，而对象会返回递归之后的结果（该对象的 stringify 结果），最后将数组中的值用`join(',')`连接起来，再根据数组或是对象在外部加上`{}`或`[]`就可以。

代码：

```js
const isObject = (object) => typeof object == "object" && object != null;
const isArray = Array.isArray;
const toString = (val) => {
  if (typeof val === "number" || typeof val === "boolean") return val;
  if (typeof val === "string") return `"${val}"`;
  if (val == null) return "null";
};
var jsonStringify = function (object) {
  if (object === null || typeof object !== "object") return toString(object);
  let str = isArray(object) ? "[" : "{";
  const keys = [...Object.entries(object)];
  for (let i = 0; i < keys.length; i++) {
    const [key, value] = keys[i];
    if (isArray(object)) {
      str += jsonStringify(value);
    } else {
      str += `"${key}":${jsonStringify(value)}`;
    }
    str += i === keys.length - 1 ? "" : ",";
  }
  str += isArray(object) ? "]" : "}";
  return str;
};
```

## 手写 JSON.parse

看到一个非常好的解法，很工整：

https://leetcode.cn/problems/convert-json-string-to-object/solutions/2329596/shou-xie-jsonparse-fang-fa-by-alex-pang-toii/

parse 的解法其实可以分解成几个部分，即从一个嵌套结构开始，再扩展到多种类型

其实类似的解析类算法都可以采取这个思路。
从简单的例子来说，如何把这个字符串解析成数组？

```js
const str = "[[1],2,3,[4,[5]]]";
```

方法如下：

1. 设置一个全局的 index，始终递增
2. 每次遍历到`[`，就进入一层递归，同时 idx++跳过左括号
3. 如果 str[idx]是数字，那就放到本层递归创建的数组里，然后跳过逗号
4. 如果遍历到`]`，就返回一层递归即可

```js
let index = 0;
const parseArray = (str) => {
  if (index >= str.length) return [];
  const res = [];
  while (index < str.length && str[index] !== "]") {
    if (str[index] === ",") index++; // 跳过逗号
    if (str[index] === "[") {
      index++;
      res.push(parseArray(str)); // 如果是左括号就递归一层
    } else if (!isNaN(str[index])) {
      res.push(+str[index]); // 这里只处理1位数，多位数、小数、负数都要到单独处理
      index++;
    }
  }
  index++; // 跳过右括号
  return res;
};
```

这时一个最简单的括号的 parse。那么对象其实也是类似的，不过需要把引号内的 key 取出来。
对于一个 json 来说，可能有的数据就是 number/string/boolean/null/Array/Object 这几种。对于每个类型我们都可以单独写一个 parse 函数，这些函数之间还可以互相调用，比如对象在获取 key 时就需要调 parseString 来得到具体的 key。

代码如下：

```js
var jsonParse = function (str) {
  const n = str.length;
  let index = 0;
  // 解析字符串
  function parseString() {
    let curStr = "";
    // ++index 是为了直接跳过首个 `"` 字符
    while (++index < n && str.charAt(index) !== '"') {
      curStr += str.charAt(index);
    }
    // 跳过结束双引号 `"`
    index++;
    return curStr;
  }
  // 解析数值
  function parseNumber() {
    let num = "";
    while (index < n) {
      cur = str.charAt(index);
      // 判断是否是数值，包含小数、负数
      if (!/[\d.\-]/.test(cur)) break;
      num += cur;
      index++;
    }
    return parseFloat(num);
  }
  // 解析数组
  function parseArray() {
    let arr = [];
    // 跳过数组开始括号 `[`
    index++;
    while (index < n && str.charAt(index) !== "]") {
      arr.push(parseValue());
      // 跳过数组中的逗号
      if (str.charAt(index) === ",") {
        index++;
      }
    }
    // 跳过数组结束括号 `]`
    index++;
    return arr;
  }
  // 解析对象
  function parseObject() {
    let obj = {};
    // 跳过对象开始括号 `{`
    index++;
    while (index < n && str.charAt(index) !== "}") {
      const key = parseString();
      index++;
      const value = parseValue();
      obj[key] = value;
      // 跳过对象中的逗号
      if (str.charAt(index) === ",") {
        index++;
      }
    }
    // 跳过对象结束括号 `}`
    index++;
    return obj;
  }
  // 解析值
  function parseValue() {
    const first = str.charAt(index);
    if (first === '"') {
      return parseString();
    } else if (first === "[") {
      return parseArray();
    } else if (first === "{") {
      return parseObject();
    } else if (first === "f") {
      // 这里是处理false、true和null
      index += 5;
      return false;
    } else if (first === "t") {
      index += 4;
      return true;
    } else if (first === "n") {
      index += 4;
      return null;
    } else {
      return parseNumber();
    }
  }

  return parseValue();
};
```

## 手写 LazyMan 类

```
实现一个LazyMan，可以按照以下方式调用:
LazyMan(“Hank”)输出:
Hi! This is Hank!

LazyMan(“Hank”).sleep(10).eat(“dinner”)输出
Hi! This is Hank!
//等待10秒..
Wake up after 10
Eat dinner~

LazyMan(“Hank”).eat(“dinner”).eat(“supper”)输出
Hi This is Hank!
Eat dinner~
Eat supper~

LazyMan(“Hank”).sleepFirst(5).eat(“supper”)输出
//等待5秒
Wake up after 5
Hi This is Hank!
Eat supper
```

这道题有两个思路

1. 最直观的，通过阻塞执行线程实现 sleep；再利用微任务会在宏任务之前执行的特点，sleepFirst 写成微任务，其他的用 setTimeout 包裹，保证 sleepFirst 最先执行。
2. 利用任务队列；每个链式调用会放入一个任务，eat 放入同步任务，两个 sleep 放入定时器任务；每个任务调用时，会调用其后的任务执行（`next()`），这样依次链接执行。而 sleepFirst 放入队列最前即可。

关于链式调用的问题：
链式调用的关键的返回 this，并且应该是在同步部分返回。那为什么依然还可以依次执行呢？
关键在于“调用”和“执行”的区分。return this 的部分会在同步任务中一次性完成，也就是一次把所有任务都先放进队列；然后再进入异步任务阶段，按照一定顺序执行。

阻塞写法：不能用 await 阻塞，因为 async 会强制返回 promise，导致链式调用失效。
方法是用 while 阻塞同步任务，但是会导致卡死，不是很好。

```js
class LazyMan {
  constructor(name) {
    this.name = name;
    this.say();
  }
  say() {
    setTimeout(() => console.log(`hello ${this.name}`), 0);
  }
  eat(food) {
    setTimeout(() => {
      console.log(`Eat ${food}`);
    });
    return this;
  }
  sleep(delay) {
    const now = Date.now();
    while (Date.now() - now < delay * 1000) {
      // wait
    }
    setTimeout(() => {
      console.log(`Wake up after ${delay}`);
    }, 0);
    return this;
  }
  sleepFirst(delay) {
    new Promise((resolve) => {
      const now = Date.now();
      while (Date.now() - now < delay * 1000) {
        // wait
      }
      resolve();
    }).then(() => {
      console.log(`Wake up after ${delay}`);
    });
    return this;
  }
}
```

任务队列写法：核心思想就是先把所有任务按照顺序放入 tasks 数组，再依次取出来执行。任务有同步异步之分，执行定时器任务时，就相当于 sleep 了。

```js
class _LazyMan {
  constructor(name) {
    this.tasks = [];
    this.tasks.push(() => {
      console.log(`hello ${name}`);
      this.next(); // 每个task都必须调用next，不然会卡住
    });
    // 这里一定是异步，因为必须需要先同步收集所有任务，再开始依次执行。
    // 这个setTimeout是第一个执行的异步任务，相当于“开始执行”
    setTimeout(() => {
      this.next();
    });
  }
  next() {
    const task = this.tasks.shift(); // 取第一个任务执行即可，每个任务会自己调用下一个
    if (task) task();
  }
  eat(food) {
    // eat放入同步函数
    this.tasks.push(() => {
      console.log(`Eat ${food}`);
      this.next();
    });
    return this;
  }
  sleep(delay) {
    // sleep放入定时器异步函数，delay之后调用next执行下一个
    this.tasks.push(() => {
      setTimeout(() => {
        console.log(`Wake up after ${delay}`);
        this.next();
      }, delay * 1000);
    });
    return this;
  }
  sleepFirst(delay) {
    // 注意unshift，将会第一个执行
    this.tasks.unshift(() => {
      setTimeout(() => {
        console.log(`Wake up First after ${delay}`);
        this.next();
      }, delay * 1000);
    });
    return this;
  }
}
```

## 手写 LRUCache 类

LRU 就是操作系统中的那个 LRU。
实现可以借助 map，关键是怎么在 map 中删除元素和移动位置：

```js
// 删除map中的元素
// 直接用delete传入键名就行
map.delete(map.keys().next().value); // map中的第一个值

//改变位置
// 删除再插入即可
const value = map.get(key);
map.delete(key);
map.set(key, value);
```

整体代码：

```js
class LRUCache {
  constructor(limit) {
    this.limit = limit;
    this.cache = new Map();
  }

  get(key) {
    if (!this.cache.has(key)) return -1;
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
    return value;
  }

  put(key, value) {
    if (this.cache.has(key)) this.cache.delete(key);
    else if (this.cache.size >= this.limit) {
      this.cache.delete(this.cache.keys().next().value);
    }
    this.cache.set(key, value);
  }
}
```

## 手写发布/订阅类 Event

> 使用 JS 实现一个发布订阅器，Event，示例如下:
>
> ```js
> const e = new Event();
>
> e.on("click", (x) => console.log(x.id));
>
> e.once("click", (x) => console.log(id));
>
> //=> 3
> e.emit("click", { id: 3 });
>
> //=> 4
> e.emit("click", { id: 4 });
> ```
>
> API 如下：
>
> ```js
> class Event {
>   emit(type, ...args) {}
>
>   on(type, listener) {}
>
>   once(type, listener) {}
>
>   off(type, listener) {}
> }
> ```

基本思路比较简单，可以用一个数组存储注册的事件；在 emit 时，寻找该事件并执行对应的监听函数即可。

但是这道题有其他特点：同一个事件可能会注册多个处理函数。同时，on 和 once 的注册应该分开，即使回调相同。

因此存储注册事件的数据结构就不能选取一个单数组，应该这样：

```js
const events = {
  click: [
    {
      once: true,
      listener: callback,
    },
    {
      listener: callback,
    },
  ],
};
```

对 once 的处理上，只需要在触发时检查 once 的值，如果为 true 就用 off 取消注册即可。

具体代码如下：

```js
class _Event {
  constructor() {
    this.events = {};
  }
  emit(type, ...args) {
    const events = this.events[type];
    if (events) {
      for (let e of events) {
        e.listener(...args);
        if (e.once) this.off(type, e.listener);
      }
    }
  }

  on(type, listener) {
    const events = this.events[type];
    if (events) {
      events.push({
        listener,
      });
    } else {
      this.events[type] = [{ listener }];
    }
  }

  once(type, listener) {
    const events = this.events[type];
    if (events) {
      events.push({
        listener,
        once: true,
      });
    } else {
      this.events[type] = [{ listener, once: true }];
    }
  }

  // 这个off取消监听是依靠listener查找的
  off(type, listener) {
    const events = this.events[type];
    if (events)
      this.events[type] = events.filter((ev) => ev.listener !== listener);
  }
}
```

## 手写 batcher 实现异步任务队列

要求实现一个函数，像这样：

```js
let executeCount = 0;
const fn = (nums) => {
  // 要求就是这个函数只能被执行一次
  executeCount++;
  return nums.map((x) => x * 2);
};

const batcher = (f) => {
  // todo 实现 batcher 函数
};

const batchedFn = batcher(fn);

const main = async () => {
  const [r1, r2, r3] = await Promise.all([
    batchedFn([1, 2, 3]),
    batchedFn([4, 5]),
    batchedFn([7, 8, 9]),
  ]);

  //满足以下 test case
  assert(r1).tobe([2, 4, 6]); // 即r1r2r3的值分别为tobe里边的
  assert(r2).tobe([8, 10]);
  assert(r3).tobe([14, 16, 18]);
  assert(executeCount).tobe(1); // executeCount必须是1，也就是说fn只能执行一次
};
```

这道题的关键在于要获取三个数组的 fn 结果，但是要求 fn 只能执行一次。
因此最基本是思路是，想办法把这三个调用的数组组合成一个数组，然后用 fn 处理一次数组，这三个函数再分别从中 slice 就可以。

这个思路关键就在于怎么先收集所有的值，并且收集完成之后还能继续交给这三个函数。可以想到的方法就是利用异步；如果把收集值的过程放到同步任务中，把 fn 的调用和最后的 slice 放到异步中，就可以实现这个效果。

具体来说，因为有一个 Promise.all，所以 batchedFn 应该是可以返回一个 Promise 的。
我们在 batcher 中存一个 Promise，里边是异步执行 fn 的操作。然后在 batcher 的返回值（即 batchedFn）中先用同步代码收集所有的参数数字，再在返回值中对上面的 Promise 执行一个 then。这个 then 就会被 Promiseall 并行执行，此时因为 fn 已经执行过，再从中 slice 值即可。

代码如下：

```js
const batcher = f => {
  const nums = []
  const p = new Promise((resolve) => setTimeout(() => resolve(f(nums)))
  return function (arr) {
    let start = nums.length
    nums.push(...arr)
    return p.then((res)=>res.slice(start,start + arr.length))
  }
}
```

基本思路是这样，还有其他写法，比上面这个更容易理解一点

```js
const batcher = (f) => {
  const nums = [];
  let count = 0;
  let res = [];
  return function (arr) {
    nums.push(...arr);
    const start = count++;
    return new Promise((resolve) => {
      setTimeout(() => {
        if (start === 0) res = f(nums);
        resolve(nums.splice(0, arr.length));
      });
    });
  };
};
```

## 手写超时取消请求

即实现 axios 中配置的 timeout 效果，当请求超时时报错并终止请求。也就是说，相当于实现了一个 Promise 的“中断”；
Promise 一旦改变状态就不能再更改。利用这一点，可以采用“抢跑”的方法，通过在异步任务之前 reject，阻止后面的 resolve 或 reject 的执行。

就像这样：

```js
let cancelFn = null;
const uploadFn = (val) => {
  return new Promise((resolve, reject) => {
    cancelFn = () => reject("cancel");
    setTimeout(() => {
      resolve("success");
      console.log(val);
    }, 5000);
  });
};
```

如果在外部执行 cancelFn，这个 Promise 就会被“中断”，resolve 的值就不会传出来。
利用这个方法，我们可以同时开启这个 upload 任务和一个定时器。当定时器到时时执行 cancel，就可以中断任务的执行

```js
let cancelFn = null;
const uploadFn = (val) => {
  return new Promise((resolve, reject) => {
    cancelFn = () => reject("cancel");
    setTimeout(() => {
      resolve("success");
      console.log(val);
    }, 5000);
  });
};

const countDown = (time) => {
  setTimeout(cancelFn, time);
};

uploadFn().then(console.log);
countDown(5000);
```

更好的方式是利用 Promise.race。因为 race 在一个 Promise 状态改变之后，另一个 Promise 的返回值会被忽略，即使另一个已经完成也不会再接受；因此可以让 upload 和倒计时任务在 Promise.race 中开启，这样如果倒计时完成并 reject，upload 就相当于被“中断”。

```js
let cancelFn = null;
const uploadFn = (val) => {
  return new Promise((resolve, reject) => {
    cancelFn = () => reject("timeout");
    setTimeout(() => {
      resolve("success");
    }, 5000);
  });
};

const uploadTimeLimit = (limit) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject("timeout");
      if (cancelFn) cancelFn();
    }, limit);
  });
};

const upload = (val, timout) => {
  Promise.race([uploadFn(val), uploadTimeLimit(timout)])
    .then((res) => console.log(res))
    .catch((err) => console.error(err));
};

upload("hello world", 3000);
```

## 计时器相关

### setTimeout 递归调用校准

js 的 setTimeout、setInterval 的执行是不准确的。由于 js 单线程的缘故，有些情况可能会导致实际的 timeout 比设定的长，或实际的 interval 比指定的 interval 长。

gui 渲染、同步代码执行、微任务执行都会造成 timeout 的不准确。这些部分我们无法获取，只能通过时间差值来知道误差是多少，然后再减去误差执行。

我们设定一个全局变量 count，以及一个开始时间 startTime（默认计时器开始的时间）。每次递归 count 自增；
如果时间没有误差，那么 count 表示经历过的倒计时次数，即`count * interval + startTime = now`。存在误差时，实际的 now 会更大，因此通过减去的方式计算误差，在下一次递归中去掉误差。

方法参考来源：https://segmentfault.com/a/1190000043829997

```js
let count = 0;
const loop = () => {
  count++;
  callback(); // 可能会有的callback
  const now = Date.now();
  const gap = now - (count * interval + startTime);
  setTimeout(loop, interval - gap);
};
setTimeout(loop, interval);
```


### 用 setTimeout 实现 setInterval

原理：和上面的调用校准类似。我们递归调用 myInertval 函数，每次检查调用的时间差值：

- 如果差值和 interval 的差值<=0，说明定时时间到了，立即执行
- 如果差值>0，说明还没到，那就用 setTimeout 延迟剩余时间执行。

```js
let pre = Date.now();
function myInterval(fn, interval) {
  let now = Date.now();
  const remain = interval - (now - pre);
  if (remain <= 0) {
    fn();
    pre = now;
    myInterval(fn, interval);
  } else {
    setTimeout(() => {
      myInterval(fn, interval);
    }, remain);
  }
}
myInterval(() => console.log(1), 1000);
```

除了 interval 本身，还需要一个 clearInterval 来让定时器停下来。实现 clearInterval 的思路有两种：

- interval 函数直接返回一个 clear 函数，调用这个函数就可以清除
- 维护一个全局变量，interval 每次递归时修改这个全局变量，当 clear 调用时清除

下面是两种实现：

```js
const myInterval = (callback, interval) => {
  let timeout = null;
  const loop = () => {
    callback();
    timeout = setTimeout(loop, interval);
  };
  timeout = setTimeout(loop, interval);
  return () => clearTimeout(timeout);
};

const clear = myInterval(() => console.log(1), 1000);
```

全局变量
用对象保存的原因是考虑多个 interval 的情况。每个 interval 持有一个 id，通过 id 在 timer 对象中获取具体的 timeout，再在 clear 中清除。

```js
const timer = {};
let id = 0;

const myInterval = (callback, interval) => {
  const timerKey = id++;
  const loop = () => {
    callback();
    timer[timerKey] = setTimeout(loop, interval);
  };
  timer[timerKey] = setTimeout(loop, interval);
  return timerKey;
};

const clearMyInterval = (timerKey) => {
  const timeout = timer[timerKey];
  clearTimeout(timeout);
};

const timeout = myInterval(() => console.log(1), 1000);

setTimeout(() => {
  clearMyInterval(timeout);
}, 3500);
```

### 实现 requestAnimationFrame

最简单实现 rAF 的方法就是简单的调用 setTimeout：

```js
const requestAnimationFrame = function (callback, lastTime) {
  var id = setTimeout(function () {
    var currTime = new Date().getTime();
    callback(currTime);
  }, 16.6);
  return id;
};
```

但是这里有个问题，如果 callback 内部执行下一个 rAF 之前花了一些时间，那么就不一定能保证 raf 的调用时机准确了。

比如 callback 内部的同步代码花费了 4ms，那么在 4ms 之后才能调用下一个 raf，这样两个 raf 之间的时间就是 20ms 了。

为了避免这种情况，我们用 lastTime 表示上一次调用 raf 的时间，然后每次 raf 内部计算上一次和本次调用的时间差，再用 16.6 减去就可以得到本次应该延迟的时间。这点和 setTimeout 实现 interval 中的校准其实是一个原理。

```js
const requestAnimationFrame = function (callback) {
  const currTime = new Date().getTime();
  const timeToCall = Math.max(0, 16.6 - currTime);
  const lastTime = currTime + timeToCall;
  const id = setTimeout(function () {
    callback(lastTime);
  }, timeToCall);
  return id;
};

const cancelAnimationFrame = function (id) {
  clearTimeout(id);
};
```

## 手写 dayjs 时间格式化功能

原理：参考 dayjs 源码。核心其实是这个正则的匹配

其他功能考虑：

- 增加更多模式字符串，比如 M、MMMM 等，以及对星期的支持
- 类似 dayjs 函数的功能，函数本身可以接收一个时间，将这个时间进行格式化

```js
function formatDate(formateStr) {
  const dateMatchRegexp = /Y{2,4}|M{2}|D{2}|H{2}|m{2}|s{2}|S{2}/g;
  const date = new Date();
  const dateInfo = {
    second: date.getSeconds().toString(),
    minute: date.getMinutes().toString(),
    hour: date.getHours().toString(),
    day: date.getDate().toString(),
    month: date.getMonth().toString(),
    year: date.getFullYear().toString(),
    week: date.getDay().toString(),
    ms: date.getMilliseconds().toString(),
  };
  const addZero = (str, len) => {
    let newStr = str;
    while (newStr.length < len) {
      newStr = "0" + newStr;
    }
    return newStr;
  };
  const matches = {
    YYYY: dateInfo.year,
    YY: dateInfo.year.slice(2),
    MM: addZero(dateInfo.month, 2),
    DD: addZero(dateInfo.day, 2),
    HH: addZero(dateInfo.hour, 2),
    mm: addZero(dateInfo.minute, 2),
    ss: addZero(dateInfo.second, 2),
    SS: addZero(dateInfo.ms, 4),
  };
  return formateStr.replace(dateMatchRegexp, (match) => matches[match]);
}
console.log(formatDate("YYYY-MM-DD HH:mm:ss:SS"));
```

如果类似 dayjs 可以接受一个时间，其实也很简单，就是用 new Date 初始化这个时间就好。
最好改成类的形式，方便初始化时间

```js
function dayjs(dateStr) {
  return new Dayjs(dateStr);
}
class Dayjs {
  constructor(dateStr) {
    if (this.isValid(dateStr)) {
      this.date = new Date(dateStr);
    } else {
      this.date = new Date();
    }
  }
  isValid(str) {
    return new Date(str).toString() !== "Invalid Date";
  }
  formatDate(formateStr) {
   ...
  }
}
console.log(dayjs('2002-08-08 08:08:08').formatDate('YYYY-MM-DD'));
```

## 手写大文件分片上传

分片上传的核心是使用 Blob.slice 方法对文件对象进行分割，然后把分割的文件块整理上传。
文件块需要使用 formdata 封装，可以在请求头或 formdata 的数据中告知后端当前文件块是第几块，是否分片完毕。
如果需要确定进度，需要通过 xhr，还需要单独统计当前上传成功/总共的片数。

简单实现如下：

```js
function setProgress(chunkList) {
  const eachChunkPercent = new Array(chunkList.length);
  return function (percent, index) {
    eachChunkPercent[index] = percent;
    const fullPercent =
      eachChunkPercent.reduce((a, b) => a + b) / eachChunkPercent.length;
    console.log(fullPercent);
  };
}

async function sliceFileUpload(file) {
  const fileChunkList = getFileChunkList(file);
  const requestFileChunkList = fileChunkList.map((chunk, index, list) =>
    request(createFormData(chunk), (e) => {
      setProgress(list)(e.loaded / e.total, index);
    })
  );
  await MyPromiseMap(requestFileChunkList);
}

function createFormData(fileObj) {
  const fd = new FormData();
  for (const [key, value] of Object.entries(fileObj)) {
    fd.append(key, value);
  }
  return fd;
}

function getFileChunkList(file) {
  const chunkSize = 1024 * 1024 * 5;
  const chunkAmount = Math.floor(file.size / chunkSize) + 1;
  let start = 0;
  const chunkedFileList = [];
  for (let i = 0; i < chunkAmount; i++) {
    const end = start + chunkSize > file.size ? file.size : start + chunkSize;
    chunkedFileList.push({
      file: file.slice(start, end),
      index: i,
      isEnd: false,
      filename: file.name,
    });
  }
  chunkedFileList[chunkedFileList.length - 1].isEnd = true;
  return chunkedFileList;
}

function request(data, url, onProgress) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open("post", url);
    xhr.send(data);
    xhr.setRequestHeader("Content-Type", "mutiply/form-data");
    xhr.onload = () => {
      resolve(xhr.response);
    };
    xhr.upload.onprogress = onProgress;
    xhr.onerror = (e) => {
      reject(e);
    };
  });
```

如果需要错误处理、断点续传的话就比较麻烦。
错误处理思路：把 promiseMap 内部的 promiseAll 实现替换成类似 promiseAllSettled 的形式，这样可以等把所有文件上传完之后再获取哪些成功、哪些失败。
然后再通过递归重传失败的文件列表即可。

```js
async function sliceFileUpload(file) {
  if(!file.length) return
  const fileChunkList = getFileChunkList(file);
  const requestFileChunkList = fileChunkList.map((chunk, index, list) =>
    request(createFormData(chunk), (e) => {
      setProgress(list)(e.loaded / e.total, index);
    })
  );
  const res = await MyPromiseMapAllSettled(requestFileChunkList);
  const failed = res.filter((item) => ...)
  await sliceFileUpload(filed)
}
```

# 随机数相关问题

随机数的核心是利用`Math.random()`。
如果要取`[0, n)`上的数，就可以利用`Math.floor(Math.random() * n)`。注意 random 生成的范围不包括 1，所以很多情况下要取[1,n]，应该是`Math.floor(Math.random() * n) + 1`
如果要取`[0,n]`，那就是`Math.floor(Math.random() * (n + 1))`

## 从数组中随机取一个元素

主要思路有两个：

1. 利用`Math.random()`从随机位置上取数。即现有一个从 0-9 的数组，然后用 random 随机一个位置取。
2. 直接用`Math.random()*n`取 0-n 的数

基本代码：

```js
function sample(arr) {
  return arr[Math.floor(Math.random() * arr.length)];
}
```

## 随机字符串

> random 接收一个整数作为随机数的个数，最多生成 8 个随机数
>
> ```
> // 'a839ac'
> random(6);
>
> // '8abc'
> random(4);
> ```

思路：

1. （个人思路）一个从 a-z0-9 的数组，从中随机位置取数。即随机数决定位置

2. 利用 ASCII 码。字母、数字的 ascii 码范围为[65,90] && [97,122] && [48,51]，因此可以取 0-62 的随机数（这三个范围大小和为 26+26+10=62），如果结果在 0-25，那就+65 成为大写字母；如果结果在 26-52，就成为小写字母，剩下的为数字，依次类推

```js
// ASCII：
// 大写字母：65~90
// 小写字母：97~122
// 数字：48-57
function random_v1(num: number) {
  let b = new Array(num)
    .fill(0)
    .map(() => String.fromCharCode(generateAcsii()))
    .join("");
  console.log(b);
}

function generateAcsii() {
  // 生成 [65,90] && [97,122] && [48,51]
  let a = Math.floor(Math.random() * 62); // [0,62]
  if (a < 26) {
    //  返回 大写字母 的 ASCII
    return a + "A".charCodeAt();
  } else if (a >= 26 && a < 52) {
    //  返回 小写字母 的 ASCII
    return a - 26 + "a".charCodeAt();
  } else {
    //  返回 数字 的 ASCII
    return a - 52 + "0".charCodeAt();
  }
}
```

3. 一个神奇的方法。随机取一个数，然后用`toString(36)`转为一个 36 进制的数即可。因为 36 进制的数一定会包含所有的字母和数字（进制的字母不区分大小写，因此如果考虑大小写就不行）

```js
const random = (n) =>
  Math.random()
    .toString(36)
    .slice(2, 2 + n);

random();
// => "c1gdm2"
```

## 随机生成六位数的手机验证码(重复/不可重复)

思路：

- 重复的很好生成，按照取随机数的方法连续取 6 个就行。
- 不重复，方法可以是每次取一个数之后，就在可选数的数组中把这个数去掉，这样就一定不会重复。

代码如下：

```js
function randomCode1() {
  return [0, 0, 0, 0, 0, 0].map(() => random(9));
}

const randomCode2 = () => {
  const numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
  const res = [];
  for (let i = 0; i < 6; i++) {
    const selectIdx = Math.floor(Math.random() * numbers.length);
    res.push(numbers[selectIdx]);
    numbers.splice(selectIdx, 1);
  }
  console.log(res);
};
```

## 洗牌函数 shuffle

洗牌函数的关键是利用随机数的概率，让数组中的每个数随机排列。
让每个数随机排列的方法有很多，比如利用`sort`，或者让随机两个数两两交换

> 有一种洗牌算法如下：
> 第 N 项数字与前 N 项数字随机选一相互交换
> 第 N-1 项数字与前 N-1 项数字随机选一相互交换
> ...
> 第 2 项数字与前 2 项数字随机选一相互交换

利用这种算法，代码如下：

```js
const shuffle = (arr) => {
  for (let i = arr.length - 1; i > 0; i--) {
    // 从[0, i]中随机取一个数
    const tmp = Math.floor(Math.random() * (i + 1));
    [arr[tmp], arr[i]] = [arr[i], arr[tmp]];
  }
  console.log(arr);
};
```

还有一种更简单的，直接利用 sort 函数：

```js
const shuffle = (arr) => arr.sort((a, b) => Math.random() - 0.5);
```

## 用 Rand5() 实现 Rand7()

参考来源是 leetcode
https://leetcode.cn/problems/implement-rand10-using-rand7/

> 给定方法  rand7  可生成 [1,7] 范围内的均匀随机整数，试写一个方法  rand10  生成 [1,10] 范围内的均匀随机整数。
> 你只能调用  rand7()  且不能调用其他方法。请不要使用系统的  Math.random()  方法。

题目和这个要求不太一样，不过方法是一样的。

思路：random5 生成 [0, 5]每个数的概率是 $\frac {1}{6}$， 使用两次 random 函数，相乘，找到等概率出现的 8 个数就可以，不满足的数据排除掉

![](https://pic.imgdb.cn/item/62d16a42f54cd3f9378fe4bb.jpg)

从表中分析可以看到，恰好有 8 个数的出现次数都为 2，因此选这 8 个数就可以。

具体算法上，我们取两个随机数分为为行和列，可以得到具体的位置为`col + (row - 1) * 5`（即把上面的二维数组拉成一维）
然后我们只用取前 21 个数。因为这样，每个数取得的概率为 $\frac {3}{25}$，取 0-7 时就恰好为 $\frac {21}{25}$
（具体解析参考力扣）

```js
var rand7 = function () {
  let row, col, idx;
  do {
    row = rand5();
    col = rand5();
    idx = col + (row - 1) * 5;
  } while (idx > 21);
  return 1 + ((idx - 1) % 7);
};
```

# lodash api

## lodash.get

描述：实现这样的一个功能：

```js
const object = { a: [{ b: { c: 3 } }] };

//=> 3
get(object, "a[0].b.c");

//=> 3
get(object, 'a[0]["b"]["c"]');

//=> 10086
get(object, "a[100].b.c", 10086);
```

其实就是分解字符串，然后从一个具体的对象中取值。因此思路也是先分解字符串，然后循环逐次取值即可。
具体处理上，最好先把括号全部去掉，全部转为`a.1.b.c`这样的形式，方便分解

```js
const _get = (obj, str, val) => {
  str = str.replace(/\[("|')?(\w+)("|')?\]/g, ".$2"); // 用 $2 来代表捕获到的内容
  const props = str.split(".");
  let res = obj;
  for (let prop of props) {
    if (typeof res !== "object") break;
    res = res[prop];
  }
  return res;
};
```

## lodash.maxBy

> 类似 loadash 如：
>
> ```js
> const data = [{ value: 6 }, { value: 2 }, { value: 4 }];
>
> //=> { value: 6 }
> maxBy(data, (x) => x.value);
> ```
>
> 如果最大的项有多个，则多个都返回，如下所示
>
> ```js
> const data = [{ value: 6 }, { value: 2 }, { value: 4 }, { value: 6 }];
>
> //=> [{ value: 6 }, { value: 6 }]
> maxBy(data, (x) => x.value);
> ```

lodash 的 xxxBy 函数基本都是类似的套路。当第二个参数是函数时，其实相当于对数组进行了一个 map。因此可以先把数组 map 一下，然后不改变序号的情况下，找到对应的值并返回。

```js
const maxBy = (data, fn) => {
  if (!Array.isArray(data)) return data;
  const pureData = data.map(fn);
  let maxIndex = [];
  let max = pureData.reduce((pre, cur) => (pre > cur ? pre : cur), -Infinity);
  pureData.forEach((num, i) => {
    if (num === max) maxIndex.push(i);
  });

  if (maxIndex.length === 1) return data[maxIndex[0]];
  else {
    const res = [];
    for (let idx of maxIndex) {
      res.push(data[idx]);
    }
    return res;
  }
};
```

但是这种方法比较低效。相对更好一点的方法是把数组直接当作`number[]`处理，只是对每个项用函数处理一下即可。
另外，类似这种相互比较的算法都可以考虑使用 reduce，尤其是第二个函数里，既然要得到数组，那就可以一开始是一个数组，不断返回数组的 concat 即可。

```js
const maxBy = (list, keyBy) =>
  list.reduce((x, y) => (keyBy(x) > keyBy(y) ? x : y)); // 对x和y处理一下即可

// 返回多项
const maxBy = (list, keyBy) => {
  return list.slice(1).reduce(
    (acc, x) => {
      if (keyBy(x) > keyBy(acc[0])) {
        return [x];
      }
      if (keyBy(x) === keyBy(acc[0])) {
        return [...acc, x];
      }
      return acc;
    },
    [list[0]]
  );
};
```

## lodash.intersection

求数组交集。

```js
//=> [2]
intersection([2, 1], [2, 3]);

//=> [1, 2]
intersection([1, 2, 2], [1, 2, 2]);

//=> [1, 2]
intersection([1, 2, 2], [1, 2, 2], [1, 2]);
```

注意这题有可能要求多个数组的交集，所以要多考虑一层。

基本方法：先随便选一个数组，去重后作为基准，然后遍历剩下的数组，如果一个数字在剩下的数组中都出现过，就说明是交集。

```js
const intersection = (...arrays) => {
  const res = [];
  const basic = [...new Set([...arrays][0]).values()];

  for (let num of basic) {
    let times = 0;
    for (let array of arrays) {
      if (array.includes(num)) times++;
    }
    if (times === arrays.length) res.push(num);
  }
  console.log(res);
};
```

简化方法：利用 reduce，每次比较相邻的两个，求得交集后再传递给下一个继续求。

两个数组求交集的方法：

```js
// 只保留在a中包含的b中的元素
// 要去重，a中相同的数会被选多次
[...new Set(a.filter((item) => b.includes(item)))];
```

结合 reduce：

```js
const intersection = (...list) =>
  list.reduce((a, b) => [...new Set(a.filter((item) => b.includes(item)))]);
```

## lodash.keyBy

示例：

```js
var array = [
  { dir: "left", code: 97 },
  { dir: "right", code: 100 },
];

_.keyBy(array, (o) => String.fromCharCode(o.code));
// => { 'a': { 'dir': 'left', 'code': 97 }, 'd': { 'dir': 'right', 'code': 100 } }
```

即传入一个函数，对每个对象进行函数的操作，得到的返回值作为新生成对象的键

```js
const keyBy = (arr, fn) => {
  const vals = arr.map(fn);
  const res = {};
  for (let val of vals) {
    res[val] = arr.find((obj) => obj.id === val);
  }
};
```

## lodash.groupBy

和 keyBy 类似，也是以某个值为标准，但是会把该键的相同值放入一个数组

```js
_.groupBy([6.1, 4.2, 6.3], Math.floor);
// => { '4': [4.2], '6': [6.1, 6.3] }
```

实现：把数组当成正常数组处理即可，只需要在处理具体值时调用 fn

```js
const groupBy = (arr, fn) => {
  const res = {};
  arr.forEach((obj) => {
    if (res[fn(obj)]) res[fn(obj)].push(obj);
    else res[fn(obj)] = [obj];
  });
  return res;
};
```

# 类算法题

## 压缩字符串

Input: 'aaaabbbccd'
Output: 'a4b3c2d1'
代表 a 连续出现四次，b 连续出现三次，c 连续出现两次，d 连续出现一次

```js
//=> a4b3c2
encode("aaaabbbcc");

//=> a4b3a4
encode("aaaabbbaaaa");

//=> a2b2c2
encode("aabbcc");
```

进阶：

- 如果只出现一次，不编码数字，如 aaab -> a3b
- 如果只出现两次，不进行编码，如 aabbb -> aab3

思路：用栈，每次向内放入字母时，如果和栈顶字母相同就统计次数但不放入，不同就把次数数字入栈，再放入新的字母。

如果考虑到只出现一次或两次不编码，只需要限制次数大于 1 或 2 才开始计数即可。或者还可以在正常生成之后，单独对出现的 1 或 2 进行处理。比如删掉所有的 1，把数字 2 替换成前面的一个字母等。

```js
const encode = (str) => {
  const stack = [];
  stack.push(str[0]);
  let times = 1;
  for (let i = 1; i <= str.length; i++) {
    if (stack[stack.length - 1] !== str[i]) {
      // if(times > 1)  // 这里控制出现一次不编码
      stack.push(times);
      str[i] && stack.push(str[i]);
      times = 1;
    } else {
      times++;
    }
  }
  return stack;
};
```

## 给数字添加千分符

> ```js
> //=> '123'
> numberThousands(123);
>
> //=> '1,234,567'
> numberThousands(1234567);
> ```

基本思路比较简单，从后向前每三位加一个逗号，再把可能出现的首位去掉就行。

```js
const numberThousands = (num) => {
  num = num.toString();
  const nums = num.split("");
  let cnt = 0;
  const res = [];
  for (let i = nums.length - 1; i >= 0; i--) {
    if (cnt === 3) {
      cnt = 0;
      res.unshift(",");
    }
    res.unshift(nums[i]);
    cnt++;
  }
  if (res[0] === ",") res.shift();
  console.log(res.join(""));
};
```

## 字符串解码

> 实现 countOfLetters
>
> ```js
> countOfLetters("A2B3"); // { A: 2, B: 3 }
> countOfLetters("A(A3B)2"); // { A: 7, B: 2}
> countOfLetters("C4(A(A3B)2)2"); // { A: 14, B: 4, C: 4 }
> ```

这道题在 leetcode 上有类似题目，不过那道题是方括号。
但是基本思路没有区别，同样是利用栈，入栈所有的数字、字母和左括号。如果遇到右括号，就出栈到前一个左括号，然后根据重复次数再次放入。

另外这道题还可以添加一个小步骤，就是如果发现数字和字母紧挨（没有括号的情况下发现数字），就把前面的字母重复数字遍即可。

最后整理字符，统计出现次数

```js
const countOfLetters = (str) => {
  const stack = [];
  const valid = "QWERTYUIOPASDFGHJKLZXCVBNM(";
  for (let i = 0; i < str.length; i++) {
    const s = str[i];
    if (valid.includes(s)) {
      stack.push(s);
    } else if (!isNaN(s)) {
      // 数字紧挨字母 直接把字母重复n遍
      for (let i = 0; i < Number(s) - 1; i++) {
        stack.push(stack[stack.length - 1]);
      }
    } else {
      // 对右括号的处理
      const times = +str[i + 1];
      const part = [];
      while (stack[stack.length - 1] !== "(") {
        part.unshift(stack.pop());
      }
      stack.pop();
      for (let i = 0; i < times; i++) {
        stack.push(part.join(""));
      }
      i++;
    }
  }
  // 整理并统计次数
  const resTmp = stack.join("");
  const obj = {};
  for (let s of resTmp) {
    if (!obj[s]) obj[s] = 1;
    else obj[s]++;
  }
  console.log(obj);
};
```

# Promise/异步 相关题目

## 1.

```js
setTimeout(() => {
  console.log("timer1");
  Promise.resolve().then(() => {
    console.log("promise");
  });
}, 0);
setTimeout(() => {
  console.log("timer2");
}, 0);
console.log("start");
```

输出顺序：

```
'start'
'timer1'
'promise'
'timer2'
```

注意`promise`在`timer2`之前，因为`Promise.resolve().then`内的是微任务，和`timer2`的`setTimeout`宏任务相比要更优先。
即会先执行完第一个`setTimeout`内的代码，才会执行第二个，并且第一个内的代码执行期间，第二个`setTimeout`仍然是异步宏任务，并不会和第一个一起同时把回调放入宏任务队列。

**注意**：Promise.resolve/reject的参数如果是个函数，不会执行这个函数，它的作用只是将参数Promise化。

## 2.

```js
Promise.reject(1)
  .then((res) => {
    console.log(res);
    return 2;
  })
  .catch((err) => {
    console.log(err);
    return 3;
  })
  .then((res) => {
    console.log(res);
  });
```

输出：

```
1
3
```

注意`reject`会直接跳到 `catch`，忽略过程中的 `then`；而 `catch` 如果有返回值，则还会递交给下一个 `then`，并不会彻底结束。

如果没有抛出错误，则 catch 中的代码不会被执行，catch 的返回值不会传递给后序的 then

```js
Promise.resolve(1)
  .then((res) => {
    console.log(res); // 1
    return 2; // 这个2传递给下一个then
  })
  .catch((err) => {
    return 3; // 这部分代码不执行
  })
  .then((res) => {
    console.log(res); // 2
  });
```

---

另外，Promise 对于错误的验证机制是由 throw 手动或自动抛出的语句。如果直接在 then 中返回自己 new 的一个 Error，或者 resolve 一个，都被视作是普通对象而非错误。

```js
Promise.resolve()
  .then(() => {
    return new Error("error!!!");
  })
  .then((res) => {
    console.log("then: ", res); // "then: " "Error: error!!!"
  })
  .catch((err) => {
    console.log("catch: ", err); // 不会输出
  });
```

## 3.

```js
const promise = Promise.resolve().then(() => {
  return promise;
});
promise.catch(console.err);
```

输出：

```
Uncaught (in promise) TypeError: Chaining cycle detected for promise #<Promise>
```

`Promise` 的 `then` 中不能返回他自己，否则会报错。这点在手写 promise 中也体现过，进入 `then` 之后需要有一个判断，如果返回值是自己就会报错。

## 4.

```js
Promise.resolve(1).then(2).then(Promise.resolve(3)).then(console.log);
```

输出：

```
1
```

`.then` 或者 `.catch` 的参数期望是函数，传入非函数则会发生值透传。
透传是会将 `resolve` 的结果透传到第一个能接受的 `then` 里边，而并不会理会其他 `then` 或 `catch` 中的参数。如果一直没有，就会直接返回一个 `Promise` 对象，中间都不会处理。  
从手写 promise 中可以看到，如果参数不是函数，就变成一个直接返回参数值的函数`(val)=>val`，然后在下面的新`Promise`中直接返回上一个`resolve`值（即`this.value`）

```js
then(onFulfilled, onRejected) {
    //判断参数是不是函数
    onFulfilled = typeof onFulfilled === "function" ? onFulfilled : (val) => val;
// ...
    let returnValueFromThen = onFulfilled(this.value);
// ...
}
```

---

另外，即使发生返回值穿透，这些 then、catch 等还会被算作是一个微任务。也就是说这道题里，resolve 的 1 还需要经历 2 个微任务，在第三个时才会传入最后一个 then

```js
new Promise((resolve) => resolve(1))
  .then(console.log) // 1
  .then(console.log) // 3
  .then(console.log) // 5
  .then(console.log); // 7
Promise.resolve(1)
  .then(2) // 2
  .then(Promise.resolve(3)) // 4
  .catch(4) // 6
  .then((res) => console.log(res)); // 8
```

上面的打印顺序是：

```
1
undefined
undefined
undefined
2
```

注意看上面的序号，表示第几个被放入微任务队列的任务

## 5.

```js
Promise.resolve("1")
  .then((res) => {
    console.log(res);
  })
  .finally(() => {
    console.log("finally");
  });
Promise.resolve("2")
  .finally(() => {
    console.log("finally2");
    return "我是finally2返回的值";
  })
  .then((res) => {
    console.log("finally2后面的then函数", res);
  });
```

输出：

```
'1'
'finally2'
'finally'
'finally2后面的then函数' '2'
```

`finally`的特点：

- 任何情况下都会调用，不管`fulfilled`还是`rejected`
- 回调不接受参数，因此不能收到任何内部的值，也没法知道`Promise`最终的状态是`resolved`还是`rejected`
- 返回的默认会是一个上一次的`Promise`对象的返回值，如果抛出的是一个异常则返回异常的`Promise`对象，也就是说如果`finally`位于中间，则他的返回值不会对其他`then`或`catch`有任何影响，相当于原封不动的传递。
- `finally`依然是一个微任务，和`then`和`catch`一样，按照微任务顺序执行

## 6.

```js
function runAsync(x) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(x);
      console.log(x);
    }, 1000);
  });
}
function runReject(x) {
  return new Promise((resolve, reject) =>
    setTimeout(() => {
      reject(`Error: ${x}`);
      console.log(x);
    }, 1000 * x)
  );
}
Promise.all([runAsync(1), runReject(4), runAsync(3), runReject(2)])
  .then((res) => console.log(res))
  .catch((err) => console.log(err));
```

输出：

```
// 1s后输出
1
3
// 2s后输出
2
Error: 2
// 4s后输出
4
```

`Promise.all`的执行原则：

1. 所有任务并行执行，最终输出事件以最长任务为准；每个任务内部的同步代码会独立并行执行。
2. 如果有一个任务`reject`或者抛出错误，就会触发`Promise.all`后面的`catch`，这时整个`Promise.all`状态变为`rejected`，不会再触发`then`；但是并不代表所有任务都不再执行，任务**仍会继续**执行，但不会有有效的返回结果。像上面的例子中虽然 3 秒时已经`rejected`，但是最后一个任务还是会执行，输出自己内部的`4`，并且不会再触发`then`或`catch`

## 7.

```js
async function async1() {
  console.log("async1 start");
  await new Promise((resolve) => {
    console.log("promise1");
  });
  console.log("async1 success");
  return "async1 end";
}
console.log("srcipt start");
async1().then((res) => console.log(res));
console.log("srcipt end");
```

输出：

```
'script start'
'async1 start'
'promise1'
'script end'
```

> `await`紧挨的内容如果是普通代码，相当于`new Promise`内部的，其后的代码相当于`.then`；如果是`Promise`对象，则会等待这个`Promise resolve`并阻塞后面代码，如果该 Promise 没有 resolve 或 reject，就会一直在`pending`状态，一直阻塞。

在`async1`中`await`后面的`Promise`是没有返回值的（没有 resolve），也就是它的状态始终是`pending`状态，因此相当于一直在 await，await，await 却始终没有响应...
所以在`await`之后的内容是不会执行的，也包括`async1`后面的 `.then`。

## 8.

```js
async function async1() {
  await async2();
  console.log("async1");
  return "async1 success";
}
async function async2() {
  return new Promise((resolve, reject) => {
    console.log("async2");
    reject("error");
  });
}
async1().then((res) => console.log(res));
```

输出：

```
'async2'
Uncaught (in promise) error
```

`await`之后的代码相当于被包裹在了一个`then`中，因此执行条件是`await`的`promise`被`resolve`；对应的，如果这个`promise`被`reject`，那么结果就是其后的代码都不会执行了，而是直接跳到 catch（如果没有 catch 就直接抛出错误）。
上面的代码相当于：

```js
new Promise(resolve=>{
  new Promise((,reject)=>{
    console.log('async2')
    reject('error')
  }).then(res=>{
    console.log('async1');
    resolve('async1 success')
  })
}).then(res=>{
  console.log(res)
})
```

显然后面的两个`then`中代码都不会执行。

---

但是如果使用`try catch`包裹并捕获错误，那么将会在 catch 中将错误立即处理，并且继续执行之后的代码。
所以这段代码会正常执行：

```js
async function async1() {
  try {
    await Promise.reject("error!!!");
  } catch (e) {
    console.log(e);
  }
  console.log("async1");
  return Promise.resolve("async1 success");
}
async1().then((res) => console.log(res));
console.log("script start");

// 输出：
("script start");
("error!!!");
("async1");
("async1 success");
```

## 9.

```js
const p1 = new Promise((resolve) => {
  setTimeout(() => {
    resolve("resolve3");
    console.log("timer1");
  }, 0);
  resolve("resovle1");
  resolve("resolve2");
})
  .then((res) => {
    console.log(res);
    setTimeout(() => {
      console.log(p1);
    }, 1000);
  })
  .finally((res) => {
    console.log("finally", res);
  });
```

输出：

```
'resolve1'
'finally' undefined
'timer1'
Promise{<resolved>: undefined}
```

首先要说一下，`then`中执行完成后会紧接着把后面的下一个`then/finally/catch`放入微任务队列，如果连续接着很多 then 就会依次放入；这个过程中相当于微任务队列一直有任务，定时器等宏任务不会执行。
就比如本题中，虽然第一个`setTimeout`在同步阶段都已经被放入宏任务队列，但是仍会先执行`then`和`finally`中的代码，等这两个中的代码都被执行完后才会执行剩下的宏任务。

最后一个定时器打印出的 p1 其实是`.finally`的返回值，我们知道`.finally`的返回值如果在没有抛出错误的情况下默认会是上一个`Promise`的返回值，而这道题中.`finally`上一个`Promise`是`.then()`，但是这个`.then()`并没有返回值，所以 p1 打印出来的`Promise`的值会是`undefined`，如果你在定时器的下面加上一个`return 1`，则值就会变成 1。

## 使用 Promise 实现红绿灯交替重复亮

红灯 3 秒亮一次，黄灯 2 秒亮一次，绿灯 1 秒亮一次；如何让三个灯不断交替重复亮灯？

要求：用 Promise 实现

三个亮灯函数已经存在：

```js
function red() {
  console.log("red");
}
function green() {
  console.log("green");
}
function yellow() {
  console.log("yellow");
}
```

延迟亮灯比较好实现，就是用 promise 包裹定时器，到时间就`resolve`并执行亮灯函数；
要让这三个不断亮灯，可以采用无限递归的方式。

```js
function red() {
  console.log("red");
}
function green() {
  console.log("green");
}
function yellow() {
  console.log("yellow");
}

// 注意注意：light函数应该每次都返回一个新的promise，而不是之前的promise
// 之前在这里出过错，不能提前调用light获取promise，那样的话相当于是把一个promise用了多次，第一次执行之后就不会再执行了
const light = (timer, cb) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      cb();
      resolve();
    }, timer);
  });
};
const step = async () => {
  await light(3000, red);
  await light(2000, yellow);
  await light(1000, green);
  await step();
};

step();
```

最基本的 setTimeout 回调实现：

```js
const light = (timer, cb) => {
  setTimeout(()=>cb();timer)
};
const step = () => {
  light(1000,()=>{
    red()
    light(1000,()=>{
      yellow()
      light(1000,()=>{
        green()
        step()
      })
    })
  })
};

step();
```

# 输出题

## 1. args 和 arguments 的类型

```js
function getAge(...args) {
  console.log(typeof args);
}

getAge(21);
```

输出："object"

当`...arr`作为收集数组的值的表达式时，arr 的类型就是一个数组，比如

```js
[1,...arr] = [1,2,3]
// arr就是[2,3]这个数组
```

作为参数也是一样，args 是一个参数组成的数组

另外，函数里有一个对象`arguments`，这个值是一个特殊的`Arguments`类型

```js
function test() {
  console.log(Object.prototype.toString.call(arguments));
}
test(111); // [object Arguments]
```

Arguments 类型可以类比类数组对象。它的构造函数并不是 Array，因此不能直接调用数组方法，但是可以转为数组后使用。

## 2. 关于对象中嵌套函数的 this 绑定

### 普通对象

代码如下，求输出：

```js
var name = "window";
const person1 = {
  name: "person1",
  foo() {
    return function () {
      console.log(this.name);
    };
  },
  bar() {
    return () => {
      console.log(this.name);
    };
  },
};

const person2 = { name: "person2" };

person1.foo()();
person1.foo().call(person2);
person1.foo.call(person2)();

person1.bar().call(person2);
person1.bar.call(person2)();
```

输出：

```
window
person2
window

person1
person2
```

解释：
首先，如果闭包是非箭头函数，则闭包和外部函数的执行上下文是不同的。也就是说我们可以通过分别对这两个函数调用 call，可以得到内外不同的 this 环境。
上面的输出按顺序 1-5 标号：

1. 输出 window，因为相当于直接调用的内部的这个匿名函数，这个函数是一个非箭头函数，并且没有显式绑定，因此默认绑定为 window
2. 输出 person2，因为给内部的这个闭包绑定到了 person2 对象上
3. 输出 window，因为没有给闭包指定 this，但是把 foo 函数的上下文设置到了 person2 上；类比下面的输出 5，箭头函数用到的 this 是外部的，这个“外部”恰好就是外层函数；而外层函数 bar 又被绑定到了 person2，因此这个闭包里的 this 就是 person2.
4. 输出 person1，即 bar 内部的 this，指代 person1，因为对箭头函数调用 call 无效，不能改变其内部 this 指向
5. 输出 person2，原因上面输出 2 解释过

### 构造函数

![](https://pic1.imgdb.cn/item/63625c5116f2c2beb1f1a20b.jpg)

输出：

```
person1
person2

window
window

window
window
person2

person1
person2
person1
```

对构造函数来说，当构造函数完成构造之后，就和 new 绑定什么关系了。也就是说 person1 就相当于是一个普通的对象，后面的操作就是在 person1 这个对象上执行的，上下文都是 person1

```js
const person1 = {
  name:'person1',
  foo1(){
    clg(this.name)
  },
  ...
}
```

之前说过 new 绑定优先级高于显式绑定，但是这句话的前提是针对构造函数的；也就是说，只有在 new 作用于构造函数时，并且构造函数用于构造时才会出现 new 优先级压过显式绑定。如果构造完成，new 就不会对实例中的 this 产生影响
即，即使有一个通过 bind 绑定的函数用 new 构造，那么函数中的 this 还是指向实例，而非绑定的值。

```js
const obj = { name: "obj0" };
const Person1 = Person.bind(obj);
Person1("obj1"); // Person1的操作是把this.name修改，如果显式绑定上，那么this就是obj，调用这个函数就是修改obj.name
console.log(obj); // obj1  说明Person1中的this是obj，改变了obj的name属性
const person1 = new Person1("obj2");
person1.name; // obj2
console.log(obj); // obj1  new的调用并没有改变this，说明Person1中的this根本就不是之前绑定的obj，而是新实例person1
```

如果 new 影响的不是构造函数，就不存在 new 绑定高于显式绑定的情况了。
因此在这个输出中，实际上就是对 person1 这个对象操作，和上一道题是一样的，**person1 的这几个方法中的 this 并不会被强制绑定到 person1 上，仍然可以被 call 改变指向**，只有 Person 构造函数里的几个 this 才是始终指向 person1

### 构造函数创建对象

![](https://pic1.imgdb.cn/item/6362611e16f2c2beb1fba29d.jpg)

这道题其实和第一题是一样的。其实就是创建了一个对象，是 person 的一个属性；直接调用 obj 和 person.obj 是一样的

## 输出题错题记录

### 1. 嵌套对象的引用问题：单独取出来嵌套对象的引用和原本的引用是冲突的

```js
var obj = { a: { b: 2 } };
var obj2 = obj;
var obj3 = obj.a;
obj2.a = 3;
console.log(obj3);
// 复原
obj2.a = { b: 3 };
console.log(obj3);
// 复原
obj2.a.b = 3;
console.log(obj3);
```

输出：

```
{b:2}
{b:2}
{b:3}
```

解释：

obj3 和 obj.a 都是对对象`{b:2}`的索引。假如覆盖 obj.a 的值，那么相当于 obj.a 不再保持该对象的索引，只有 obj3 才保留；
但是由于修改的是 obj.a 的属性，并不会影响这个 obj.a 这个对象本身（即修改的是 obj 内的一个属性，这时已经和`{b:2}`这个对象没有关系了）。
如果是通过 obj.a.b 来修改`{b:2}`这个对象，那么会对 obj3 造成影响。反过来，如果修改 obj3，也会修改 obj.a 的值。因为 obj.a 和 obj3 这时都是`{b:2}`这个对象的索引。

注意好一点：对于嵌套的引用类型结构，如果取的是内部的嵌套对象，并不是复制，只是对这个子对象取了一个索引。如果通过新的索引或源对象的方式修改该子对象，都会修改对象本身。
如果是直接替换，那就相当于丢失了这个索引。

---

### 2. var 定义问题：函数定义在哪就从那哪查找，但实际的值是执行阶段确定的

```js
const fn = () => console.log(i);
for (var i = 0; i < 3; i++) {
  setTimeout(fn, i * 1000);
}

// 如果把var替换成let呢
```

输出：

```
3 // 第0秒
3 // 第1秒
3 // 第2秒

// 如果换成let
报错：i未定义
```

这道题其实和那个 var 对于 setTimeout 的输出是一样的。即使把 fn 提取到外面，也是一样的。
因为 var 的变量提升，导致 i 每次执行都会更新顶层的值

```js
var i = undefined;
const fn = () => console.log(i);
for (i = 0; i < 3; i++) {
  setTimeout(fn, i * 1000);
}
```

注意：这里输出**不是**undefined。因为 i 在下面循环中被赋值；当 fn 执行时，i 已经被赋值为 3 了。这和 fn 在内部定义是完全一样的：

```js
// 和上面是一样的
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), i * 1000);
}
```

这里是之前错的地方。i 如果定义在 for 循环内部，那么它查找的时候会按照`for的块级作用域 -> 外部作用域`这个顺序查找的，而直接定义在外部则是直接在`外部作用域`内查找。但是不管在哪里查找，对于 var 定义的变量来说，都是声明在最顶层的，并且当执行 fn 的时候 i 都是已经赋值的了。并不会出现 undefined 的情况。

如果是 let，那么只有 for 内部才有 i 这个变量。如果是把 fn 定义在外面，就会报错找不到 i，因为 i 不在外部作用域内（注意不是 TDZ，因为 i 根本不在外部作用域，不是 i 在声明之前被使用）

---

### 3. function 和 var 声明的覆盖问题

```js
var a;
function a() {}
console.log(a);
a = 10;
console.log(a);
```

输出：

```
f a()
10
```

函数声明不会被 var 声明覆盖。因此这里函数声明可以声明成功，第一个 console.log 打印的是函数 a；
而后面 a=10 的语句相当于覆盖了函数 a 的声明。注意不是 var 定义变量的声明。

如果把 var 替换成 let，这里就会报错 a 已经被声明过了，不能再声明一个名字叫 a 的变量。

```js
let a; // 报错
function a() {}
```

---

### 4. 函数内参数重命名、赋值，以及函数参数默认值

```js
function f(x) {
  console.log(x);
  var x = 200;
  console.log(x);
}
f(100);
```

输出

```
100
200
```

函数内的变量理论上是不能重新声明的。如果使用 let、const 重新声明，则会直接报错；而如果使用 var，则不会显式报错，但声明会失败。
这里`var x = 200`相当于只执行了`x = 200`，`var x`这个并没有生效。因此第一个 log 打印的是参数的值而不是 var 的声明；第二个 log 打印的是赋值之后的 x

---

### 5. 对象内箭头函数中 this 的指向

```js
function global() {
  const obj = {
    name: "inner",
    f1() {
      console.log(this.name);
    },
    f2: function () {
      console.log(this.name);
    },
    f3: () => {
      console.log(this.name);
    },
    f4() {
      return function () {
        console.log(this.name);
      };
    },
    f5() {
      return () => console.log(this.name);
    },
  };
  return obj;
}
const outer = {
  name: "outer",
};
const obj = global.bind(outer)();
obj.f1();
obj.f2();
obj.f3();
obj.f4()();
obj.f5()();
```

输出：

```
inner
inner
outer
undefined
inner
```

解释：

1. f1 和 f2 是相同的方式的不同写法，由于调用采用了隐式绑定，因此输出值为 obj 的 name
2. f3 是箭头函数。箭头函数是将 this 设置为外层的 this，注意这个“外层”，指的是上层执行上下文。也就是说，这个外层只能是函数或全局，而不是一个对象。

什么意思呢，就是说箭头函数取的 this 应该是其外层函数内的 this，而不是外层对象内的 this。因此上面的 f3 取的是 global 函数内的 this，而不是 obj 对象的 this。
同理也适用于 f5，f5 的返回值这个箭头函数的“外层”指的是 f5 这个函数，因此执行这个箭头函数会从 f5 函数内取 this，即 obj（因为是通过`obj.f5`调用的，f5 内的 this 被设置为 obj）

3. f4 打印 undefined，因为这个函数相当于单独执行，但它有又没有去取外层的 this。

### 6. 构造函数内的箭头函数

```js
function Test() {
  this.flag = false;
  this.change = () => {
    this.flag = true;
  };
}
function Test1() {
  this.flag = false;
}
Test1.prototype.change = function () {
  this.flag = true;
};
const test1 = new Test1();
const test = new Test();
test.change();
test1.change();
console.log(test.flag, test1.flag);
```

输出：

```
true
true
```

解释：

这两个定义的效果是相同的：

```js
function Test() {
  this.change = () => {
    this.flag = true;
  };
}

function Test() {
  this.flag = false;
  this.change = function () {
    this.flag = true;
  };
}
```

执行 change，都可以修改 flag 的值。原因在于，第一个因为是箭头函数，所以内部的 this 就是 Test 函数的 this。当构造时，this 被设置为实例，因此构造函数内的 this 也是构造出的实例。

```js
test = {
  flag: false,
  change: () => {
    this.flag = true;
  },
};
```

如果是上面这个对象内的 change，显然不能实现。因为这里的 this 是指外层函数的 this，不会指向 test 对象，也不能绑定到 test 对象。
但是为什么在构造函数内就可以呢？还是因为箭头函数使用的是 Test 函数的 this，而构造时这个 this 被绑定为构造出的实例。
再具体一点，像下面这样：

```js
function Test() {
  this.flag = false;
  this.change = () => {
    // 这里的this就是Test函数的this，是同一个值，因此改变这里的this就是改变实例
    this.flag = true;
  };
}

const test = new Test();
const testChange = test.change;
testChange();
console.log(test.flag); // true，修改成功
```

可以看到即使把 change 方法从 test 中取出来，依旧可以让函数内部的 this 指向构造出的对象。这也就证明 new 绑定的优先级高于隐式绑定，并且 new 绑定之后即使不使用隐式绑定的调用方式依然可以保证 this 的指向。

再说回这个函数：

```js
function Test() {
  this.flag = false;
  this.change = function () {
    // Test内的this不会影响到这里，这里的this是change函数自己的
    this.flag = true;
  };
}

test = {
  flag: false,
  change() {
    this.flag = true;
  },
};
```

这个就很好理解了，change 方法的调用就是和对象内调用一样。注意这里的 this 是不会像箭头函数那样和 Test 内的 this 绑定的，因此如果不使用隐式绑定，就会出错。

```js
const test = new Test();
const testChange = test.change;
testChange(); // window，即this没有绑定到test对象上
```

总结来说，就是：

- new 会把构造函数内的 this 绑定到实例上，因此如果构造函数内有箭头函数，由于他们共享 this（箭头函数内的 this 本来就是构造函数内的），所以这个 this 一定是实例，不会绑定到其他对象上。
- 对于构造函数内定义的普通函数，函数内的 this 依然是自己的，不会被 new 执行影响；因此这些函数调用就需要进行绑定，否则会缺少指向。

### 7. 作用域问题

还是上面那个例子：

```js
function test() {
  this.flag = false;
  this.change = () => {
    console.log(button.flag);
  };
}
const button = new test();
button.change(); // 输出：false
```

这里似乎有个很奇怪的地方，为什么看起来 change 函数内的 button 是在其声明前调用的，但是依旧能正常输出 button？

这里纠正一个之前错误的点：作用域的查找单位是“层”，而不是“行”。也就是说，变量查找是以层为单位的，在本层查找不到，就到外层查找，而不用管这个变量到底在外层的哪里。
比如说：

```js
function getName() {
  console.log(name);
}
const name = "aaa";
getName(); // aaa
```

当执行 getName 时，打印出的 name 就是外层的 name。**注意这个“外层”，指的是整个外层的变量，和具体位置没关系，和先后顺序也没关系。**
即使 const 是在函数声明之后定义的，但依然不影响变量的查找。
只要 getName 函数的调用在 name 声明和赋值之后，就能取到正常的值。

另外，TDZ 也指的是执行时才会出现的作用域问题，而不是在作用域查找时才会出现。比如：

```js
getName(); // Cannot access 'name' before initialization
const name = "aaa";
function getName() {
  console.log(name);
}
```

很显然 TDZ 和函数定义和变量定义的先后顺序没有关系，只和函数执行的前后有关系。

### 8. 函数的构造调用和直接调用的 this

```js
function Foo() {
  console.log(this);
  return this;
}
Foo.prototype.getName = function () {
  console.log(this);
};
console.log(Foo());
console.log(new Foo());
new Foo().getName();
```

输出：

```
window
window
{}
{}
```

解释：

当构造函数直接返回一个对象时，new 构造函数的结果就是这个对象。不过有一个例外，就是返回 this 的情况。
按理来说 Foo 函数内的 this 指向 window，new Foo()应该返回 window 才对；但是 new Foo 同时也会把 this 指向创建的实例对象。因此 Foo 函数内的 this 实际是构造出的实例对象，返回的也是这个，而不是默认情况下的 window 或 undefined

```js
function Foo() {
  // const this = {}
  return this;
}
```

既然 new Foo 能成功创建，因此 getName 方法也能正常执行。

# 其他乱七八糟的题目

## 通过 js 修改全局样式

可以通过`document.querySelectorAll("*")`获取所有元素，然后通过 getComputeStyle 获取每个元素样式，再通过`element.style`修改样式。

比如，把所有的背景白色修改为红色：

```js
document.querySelectorAll("*").forEach((element) => {
  if (getComputedStyle(element).backgroundColor === "rgba(0, 0, 0, 0)") {
    element.style.backgroundColor = "red";
  }
});
```

注意最好先查看一下具体的值再做匹配。

## 颜色单位互相转换

### 16 进制转 rgb

方法就是把 16 进制的每 2 位通过 parseInt 转化为 10 进制的数即可。

代码如下：

```js
function hexToRgb(color) {
  const colorStrArr = color.split("");
  colorStrArr.shift(); // 去掉开头的#
  let colors = colorStrArr.join("");
  // 如果是简写，每个位置都要加倍一次
  if (colors.length === 3) {
    const newColors = new Array(6);
    for (let i = 0; i < colorStrArr.length; i++) {
      newColors[i * 2] = colorStrArr[i];
      newColors[i * 2 + 1] = colorStrArr[i];
    }
    colors = newColors.join("");
  }
  const r = parseInt(colors.slice(0, 2), 16);
  const g = parseInt(colors.slice(2, 4), 16);
  const b = parseInt(colors.slice(4, 6), 16);
  console.log(`rgb(${r}, ${g}, ${b})`);
}
```

### rgb 转 16 进制

如下：

```js
function rgbToHex(color) {
  const regex = /\d+/g;
  const matches = [];

  let match;
  while ((match = regex.exec(color))) {
    matches.push(parseInt(match[0]));
  }
  return `#${matches[0].toString(16).padEnd(2, "0").toUpperCase()}${matches[1]
    .toString(16)
    .padEnd(2, "0")
    .toUpperCase()}${matches[2].toString(16).padEnd(2, "0").toUpperCase()}`;
}
```

这里的关键其实是用 exec 循环提取数字。然后转成 16 进制数即可。padEnd 可以很方便地填充字符以保证 2 位数。
