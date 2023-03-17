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

```js
function unique(arr) {
  let res = arr.filter((item, index, array) => {
    return array.indexOf(item) === index;
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

或者不使用 concat 函数直接抹平，而是利用 reduce 逐个拼接数组中的元素，如果遇到数组类型就递归一次直到深度为 0

```js
function flatten(arr) {
  if (!Array.isArray(arr)) return arr;
  return arr.reduce((pre, curr, index, arr) => {
    return pre.concat(flatten(curr));
  }, []);
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

```js
const isObj = (obj) => typeof obj === "object";
function isEqual(obj1, obj2) {
  if (!isObj(obj1) && !isObj(obj2)) return obj1 === obj2;
  else if ((isObj(obj1) && !isObj(obj2)) || (!isObj(obj1) && isObj(obj2)))
    return false;
  const keysX = Object.keys(obj1);
  const keysY = Object.keys(obj2);
  if (keysX.length !== keysY.length) {
    return false;
  }
  let flag = true;
  for (let key in obj1) {
    flag = flag && isEqual(obj1[key], obj2[key]);
  }
  return flag;
}
```

## 手写防抖/节流

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
遍历对象，如果键名称含有 `.`， 将最后一个子键拿出来，构成对象，如 `{'a.b.c.dd': 'abcdd'}` 变为 `{'a.b.c': { dd: 'abcdd' }}` , 如果变换后的新父键名中仍还有点，递归进行以上操作。

```js
function nested(obj) {
  Object.keys(obj).map((k) => {
    //一次传递一个键，这一组键拆分完后再处理第二组
    getNested(k);
  });

  return obj;

  function getNested(key) {
    const idx = key.lastIndexOf(".");
    const value = obj[key];
    if (idx !== -1) {
      delete obj[key]; //删掉原对象的这个键
      //取最后一个点前面和后面的部分，分别作为主键和副键（要被放入内部的键）
      const mainKey = key.substring(0, idx);
      const subKey = key.substr(idx + 1);
      if (!obj[mainKey]) {
        //如果原对象没有主键，创建一个新键，值是一个新对象
        obj[mainKey] = { [subKey]: value };
      } else {
        //这种情况是源对象已经有一个同名的键，把新值直接放入原先有的键下面
        obj[mainKey][subKey] = value;
      }
      if (mainKey.includes(".")) {
        //
        getNested(mainKey);
      }
    }
  }
}
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

## 手写 JSON.stringify

利用递归逐层解析并转化为 json 格式。
注意转化过程中有一些特殊值会被忽略或改写：

- 对象中的 function、symbol、undefined 会被忽略，即直接没有这个属性；boolean、number、string 会保持原样
- 数组中的 function、symbol、undefined 会被改为 null

具体实现上，对于某一层对象，设置一个数组用于存储当前对象的每个值的遍历结果。对于每个值都递归一次，非对象会返回原值或忽略掉，而对象会返回递归之后的结果（该对象的 stringify 结果），最后将数组中的值用`join(',')`连接起来，再根据数组或是对象在外部加上`{}`或`[]`就可以。

代码：

```js
function stringify(obj) {
  if (obj === null) return obj;
  if (typeof obj !== "object") {
    const type = typeof obj;
    if (type === "number" || type === "boolean") return obj;
    if (type === "string") return `"${obj}"`;
    else return undefined;
  }

  const res = [];
  const isArray = Array.isArray(obj);
  for (const key in obj) {
    let val = obj[key];
    val = stringify(val);
    if (val === undefined) {
      if (!isArray) continue;
      else val = null;
    }
    // 如果当前obj是对象，那么每个值都是`key:value`形式，否则就只是value
    res.push((isArray ? "" : `"${key}":`) + val);
  }
  return isArray ? `[${res.join(",")}]` : `{${res.join(",")}}`;
}
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

# 随机数相关问题

随机数的核心是利用`Math.random()`。
如果要取`[0, n)`上的数，就可以利用`Math.floor(Math.random() * n)`。注意 random 生成的范围不包括 1，所以很多情况下要取[1,n]，应该是`Math.floor(Math.random() * n +)`

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
    const tmp = Math.floor(Math.random() * i + 1);
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
  str = str.replaceAll("[", ".").replaceAll(/\'|\]|\"/g, ""); // 把左括号全部改为 "." ，其他字符去掉
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

# 计算机基础题

## 设计模式

1. 对象间存在一对多关系，当一个对象被修改时，则会自动通知它的依赖对象：这是观察者模式，不是代理模式

## 计网

各协议端口、作用
![](https://pic.imgdb.cn/item/64142ddfa682492fcc78e926.jpg)

网络接口卡是局域网组网需要的基本部件。网卡一端连接局域网中的计算机，另一端连接局域网的**传输介质**

SMTP 是简单邮件传输协议，没有认证功能，并且只传输 7 位的 ACII 码，不能传送二进制文件。SMTP 协议只能支持从用户代理向邮件服务器发送邮件，反过来不行
pop3 和 IMAP 协议是用于客户端接收邮件。
pop3 协议从服务器上单向下载邮件，不会有邮件读取反馈。
IMAP 协议是双向传输信息，客户端信息会反馈到服务器端。

常用的信道复用技术：频分复用 FDM、时分复用 TDM、码分复用 CDM、波分复用 WDM。

传输延迟与分组长度 L 和链路带宽 R 有关，T=L/R
传播延迟与物理链路长度 D 和信号传播速度 S 有关，T=D/S

全双工是指交换机在发送数据的同时也能够接收数据，两者同步进行。TCP 连接、UDP 连接、电话通信都是全双工的
半双工数据传输允许数据在两个方向上传输，但是，在某一时刻，只允许数据在一个方向上传输，它实际上是一种切换方向的单工通信。
单工数据传输只支持数据在一个方向上传输。

IEEE（美国电子电气工程师协会）制定了以 802 开头的标准，目前共有 11 个与局域网有关的标准。
IEEE 802.1—概述、体系结构和网络互连，以及网络管理和性能测量。  
IEEE 802.2—逻辑链路控制 LLC。最高层协议与任何一种局域网 MAC 子层的接口。  
IEEE 802.3—CSMA/CD 网络，定义 CSMA/CD 总线网的 MAC 子层和物理层的规范，**以太网**的技术原形
IEEE 802.4—令牌总线网。定义令牌传递总线网的 MAC 子层和物理层的规范。  
IEEE 802.5—令牌环形网。定义令牌传递环形网的 MAC 子层和物理层的规范。  
IEEE 802.6—城域网。  
IEEE 802.7—宽带技术。  
IEEE 802.8—光纤技术。  
IEEE 802.9—综合话音数据局域网。  
IEEE 802.10—可互操作的局域网的安全。  
IEEE 802.11—无线局域网。  
IEEE 802.12—优先高速局域网(100Mb/s)。  
IEEE 802.13—有线电视(Cable-TV)。

链路层的多路划分协议：
目的：解决多个接受节点和多个发送节点在一个共享广播信道上的访问问题。（即多个占一条线，怎么协调）
三个基本方式
信道划分协议
TDM、FDM、CDMA（码分多址）
随机接入协议
时隙 ALOHA
ALOHA
CSMA
CSMA/CD
发送前空闲检测，只有信道空闲才发送数据
发送过程中冲突检测，若有冲突发生则需避让
冲突发生后，立即停止发送，随机避让。
不存在优先级
轮流协议

承载信息量的基本信号单位是**码元**
