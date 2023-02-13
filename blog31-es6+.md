---
title: es6一些疏漏的知识点
date: 2021-12-09 18:57:20
tags: 日常学习
cover: /img/ES6.png
categories: JavaScript
---

# 解构赋值

## 数组解构赋值

- 嵌套数组(多维数组)也可以解构赋值
  比如:

```js
let [a, [b, [c]]] = [1, [2, [3]]];
```

- 数组可以用空值代替不想要的来跳过

```js
let [, , c] = [1, 2, 3];
let [x, , y] = [1, 2, 3, 4];
let [x, ...others] = [1, 2, 3, 4, 5];
```

- 数组也可以使用默认值

```js
let [a = 1, b, c = 2] = [2, 3];
```

- 解构赋值的右边必须是可迭代对象, 不然会报错

## 对象解构赋值

- 对象解构没有次序,所以赋值和被赋值的对象中的属性名必须相同;
  - 被赋值的对象实际上是对象简写形式, 所以需要名字和对象中的相同
  - 也可以不相同,自己声明值,但是属性仍要相同

```js
let { name, age } = { name: "xiaoming", age: 18 };
//实际上是
let { name: name, age: age } = { name: "xiaoming", age: 18 };
//或者可以
let { name: username, age: userage } = { name: "xiaoming", age: 18 };
username === "xiaoming" && userage === 18; //true
```

- 对象解构的嵌套:

```js
let {
    name:[
        fname,
        lname
    ],
    age
} = let {
    name:[
        "Ming",
        "Xiao"
    ],
    age:18
}
```

注意这个 name 不会被赋值,因为实际上是对象的一个属性名;如果属性名也需要赋值可以这样:

```js
let {
  name,
  name: {
    /*...*/
  },
} = {
  xiaoming: {
    /*...*/
  },
};
```

- 对象的默认值

```js
let { x = 3, y } = { y: 10 };
//注意区分
let { x: y = 5 } = { x: 3 }; //这个意思是, x是属性名, y是要赋的值, 默认是5
```

## 函数参数解构

- 函数参数可以直接写成一个解构赋值的对象或数组

```js
function add([a, b]) {
  return a + b;
}
add([1, 2]);

function concats({ fname, lname }) {
  return `${fname} ${lname}`;
}
concats({ fname: "Ming", lname: "Xiao" });
```

- 这种写法的主要用途在于多个参数设置默认值,但是使用函数的时候可以只传不为默认值的参数.
  比如这个例子, 只用设置 a 和 e 的值,依旧可以确定多个默认值

```js
function add({ a = 5, b = 4, c = 3, d = 2, e = 1 }) {
  return a + b + c + d + e;
}
add({ a: 6, e: 2 });
```

## 其他

注意要解构的数组/对象一般没有名字,而是可以直接拿出来变量使用;但是如果要分行写要加小括号

```js
let {x};
({x} = {x:5});//注意这个分号不要少
```

# 字符串扩展

## 模板字符串的直接调用

在某些方法后面直接加上模板字符串而不用加括号
比如:

```js
alert`hello`;
//等同于
alert(["hello"]);
```

这种也可以用在自定义函数里

```js
function tag(s) {
  console.log(s[0]);
}
tag`Hello`; //hello
```

而如果模板字符串有值, 甚至可以传递值,比如

```js
let a = 5;
let b = 10;
tag`Hello${a},world${b}`;
// 等同于
tag(["Hello", ",world", ""], 5, 10);
function tag(arr, num1, num2) {
  console.log(...arguments);
}
```

## 字符串实例方法

- `includes()`：返回布尔值，表示是否找到了参数字符串。
- `startsWith()`：返回布尔值，表示参数字符串是否在原字符串的头部。
- `endsWith()`：返回布尔值，表示参数字符串是否在原字符串的尾部。
  另外这三个方法都支持第二个参数，表示开始搜索的位置。

```js
let str = "hello world";
str.includes("h", 2); //true
str.startWith("h", 3); //true
str.endsWith("o", 5); //false
```

- `padStart()`/`padEnd()`,用于在字符串不足长度时补字符,参数是目标长度和不足时的补充字符；（如果没有第二个参数会用空格补全）

```js
let str = "aaa";
str.padStart(5, "a"); //aaaaa
str.padEnd(6, "b"); //aaabbb
```

- `replaceAll()`:一次性替换所有匹配。
  - 他的第二个参数可以是一些特殊值, 比如(注意外面要用反引号, 比如)
    - $&：匹配的字符串。
    - $`：匹配结果前面的文本。
    - $'：匹配结果后面的文本。
    - $n：匹配成功的第 n 组内容，n 是从 1 开始的自然数。这个参数生效的前提是，第一个参数必须是正则表达式。
    - $$：指代美元符号$
  ```js
  "aaabbbbbbbccc".replaceAll("b", "");
  "aaabbbbbbbccc".replaceAll("b", `$'`);
  ```

# 数字对象方法

- isNaN:会把参数用 Number()处理, 如果返回不是 NaN, 就返回 true;否则返回 false. 所以像`'15'`这样的字符串返回是 false(不是 NaN)
- Math.trunc()可以去除一个数的小数部分, 是直接去除而不是取整

```js
Math.trunc(4.1); // 4
Math.trunc(4.9); // 4
Math.trunc(-0.1234); // -0
```

- Math.sign 方法用来判断一个数到底是正数、负数、还是零

```js
Math.sign(-5); // -1
Math.sign(5); // +1
Math.sign(0); // +0
Math.sign(-0); // -0
Math.sign(NaN); // NaN
```

# 函数

## 默认值

默认值的写法不再赘述, 注意在用对象解构当做默认值的时候调用空参数函数需要内部一个{};但是可以通过一种方法不写这个{},比如:

```js
function add({ a = 1, b = 0 }) {
  return a + b;
}
add({}); //不传参数但是需要写空对象

function add2({ a = 1, b = 0 } = {}) {
  return a + b;
}
add(); //可以直接调用
```

同理, 这两种默认值的方法也有区别:

```js
function add1({ a = 1, b = 2 } = {}) {
  return [a, b];
}
function add2({ a, b } = { a: 1, b: 2 }) {
  return [a, b];
}
add1({ a: 2 }); //[2,2]
add2({ a: 2 }); //[2,undefined]
add1({}); //[1,2]
add2({}); //[undefined,undefined]
```

区别在于:

- 写法一函数参数的默认值是空对象，但是设置了对象解构赋值的默认值；
- 写法二函数参数的默认值是一个有具体属性的对象，但是没有设置对象解构赋值的默认值。

另外, 参数除了直接设置值之外还可以是变量或有返回值的函数;当默认值是变量的时候需要提前声明, 函数进行声明初始化时，参数会形成一个单独的作用域（context）。等到初始化结束，这个作用域就会消失。

```js
let x = 1;
function f1(y = x) {
  console.log(y);
  let x = 2;
  console.log(x);
}
f1(); //1,2
```

## 箭头函数

这里说明箭头函数几个注意点:

- 箭头函数没有自己的 this
- 不可以当作构造函数
- 不可以使用 arguments 对象
- 不可以使用 yield 命令
- 不适用与定义对象的方法

## 尾调用和尾递归

尾调用说起来就是一句话：在函数的最后一句返回另一个函数的调用：

```js
function f(n) {
  return g(n);
}
```

尾调用的限制很多，任何不满足这个情况的都不是尾调用。比如先取得返回值、返回的不是纯调用、不返回都不可

```js
// 以下都不是
function f(x) {
  let y = g(x);
  return y;
}

function f(x) {
  return g(x) + 1;
}

function f(x) {
  g(x);
}
```

尾调用的作用：
因为函数在最后一步返回时才调用其他函数，因此相比于直接调用，外层函数的调用栈会被正常销毁而不会因为正在调用其他函数而导致保留。如果所有函数都是尾调用，那么完全可以做到每次执行时，调用帧只有一项，这将大大节省内存。
注意，如果尾调用的函数是一个闭包，就不能实现优化。

---

尾递归：因为递归是一个很需要调用栈的方式，因此如果利用尾调用优化，让递归在最后返回执行，就可以大大减少调用栈的消耗。对于尾递归来说，由于只存在一个调用帧，所以永远不会发生“栈溢出”错误。

比如斐波那契数列：

```js
function Fibonacci(n) {
  if (n <= 1) {
    return 1;
  }

  return Fibonacci(n - 1) + Fibonacci(n - 2);
}
```

用尾递归优化一下：

```js
function Fibonacci2(n, ac1 = 1, ac2 = 1) {
  if (n <= 1) {
    return ac2;
  }

  return Fibonacci2(n - 1, ac2, ac1 + ac2);
}
```

# 数组

## 展平数组

flat 方法会默认展开一层数组, 如果希望展平全部可以设置参数表示阶数

```js
[1, [2, [3]]].flat(Infinity);
```

## 排序

sort 方法需要一个函数作为排序器, 函数根据条件判断返回正数或负数

```js
[{ id: 1 }, { id: 4 }, { id: 3 }, { id: 2 }].sort((a, b) => {
  if (a.id > b.id) return 1;
  return -1;
});
```

判断依据:

- 如果比较函数(a,b)返回大于 0, 则 a 会排在 b 后
- 如果比较函数(a,b)返回等于 0, 则不排序
- 如果比较函数(a,b)返回小于 0, 则 a 会排在 b 前

所以如果单纯比较数字可以直接 `a-b`, 一般会升序排列, 比如`arr.sort((a,b)=>a-b)`
注意, 比较中尽量不要带有`>=`或者`<=`, 否则就会不准确

## 数组的空位

当通过new Array创建数组但并不填值，或者通过Array.from一个只有length属性的对象，都会在数组中创建空值。
```js
Array(3) // [, , ,]
```
空值有以下几个特点：
- 不等于`null`或`undefined`。空值就是没有值，undefined算做有值；
- 将含有空值的类数组转化为真数组时，会将空值变为undefined
- forEach(), filter(), reduce(), every() 和some()都会跳过空位。

## 一些数组的方法

### `reduce`

接受两个参数, 第一个参数相当于给每个元素都执行一遍的函数, 一般是一个累加器; 第二个参数是传递给函数的初始值, 也就是初始的 total 值, 默认为 0
其中第一个参数函数有四个参数

- `total` 这个值继承自上一次遍历的返回值，同时本次的返回值也会作为下一次的该参数
- `currentValue` 必需。当前元素
- `currentIndex` 可选。当前元素的索引
- `arr` 可选。调用 reduce 的数组, 就是前面的那个数组对象

比如:

```js
let sum = [1, 2, 3, 4, 5].reduce((total, num) => {
  return (total += num);
}, 10);
sum === 25; //true
```

更高级的应用, 可以用于对象数组的累加

```js
let arr = [
  {
    //...
    price: 100,
  },
  {
    price: 200,
  },
];
arr.reduce((initObj, currentObj) => {
  return (initObj.price += currentObj.price);
});
```

### `forEach`/`map`/`find`/`filter`

这里主要想说这些函数共有的一个容易疏忽的点:额外参数

```js
let arr = [1, 2, 3];
arr.forEach((num, index, arr) => {
  //...
}, thisArg);
```

- `index`: 当前序号
- `arr`: 当前数组
- `thisArg`：当执行回调函数 callback 时用作 this 的值。

另外, `map` 和 `forEach` 的区别在于 `map` 会返回一个操作过后的新数组, 但 `forEach` 会修改原数组，其实是 forin 循环的一种简化。

### 其他的小方法

- `Array.of`：可以用于部分替代`new Array()`，主要特点是只有一个参数时看作的是数组的值而不是大小
```js
Array.of(3) // [3]
Array(3) // [, , ,]
```
- `Array.at`：索引数组的方法，**支持负索引**

# 运算符

## ??

行为类似||，但是只有运算符左侧的值为 null 或 undefined 时，才会返回右侧的值。
和||最大区别是, ||会在左边是 0 或 false 或''生效, 但??不会
所以??也称作空值合并, 比如判断表单元素:

```js
let inputValue = e.target.value ?? "";
```

这样可以避免空值或 0 也被删去
注意, 多个逻辑运算符同时使用需要括号指明顺序

```js
(lhs && middle) ?? rhs;
lhs && (middle ?? rhs);

(lhs ?? middle) && rhs;
lhs ?? (middle && rhs);

(lhs || middle) ?? rhs;
lhs || (middle ?? rhs);

(lhs ?? middle) || rhs;
lhs ?? (middle || rhs);
```
