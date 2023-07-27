---
title: JavaScript学习总结
date: 2021-12-05 21:21:10
tags: 日常学习
cover: /img/JS.png
categories: JavaScript
sticky: 10
---

# js 数据类型

## 数据类型概览

js 只有 8 种基本数据类型 

- number
- bigint
- string
- boolean
- null
- undefined
- symbol
- object

其中 Symbol 和 BigInt 是 ES6 中新增的数据类型：

- Symbol 代表创建后独一无二且不可变的数据类型，它主要是为了解决可能出现的全局变量冲突的问题。
- BigInt 是一种数字类型的数据，它可以表示任意精度格式的整数，使用 BigInt 可以安全地存储和操作大整数，即使这个数已经超出了 Number 能够表示的安全整数范围。创建一个 bigint 需要在数字后面加上 n：

```js
const bigInt = 1234567890123456790123456789012345690n;
```

> bigint 不能和正常的 number 类型做加减乘除
> bigint 只能和自己运算，结果也是自己，并且只可能是正整数
> bigint 可以通过`Number()`和 number 互相转化，但是不支持`+bigint`这样转换。

## js 中的数组

js 中没有专门的数组类型，数组会被当作是一个特殊的对象。

在其他大多数语言中，数组都是被连续划分的一大块空间，这样方便存储和读写；但是 js 中的数组有点特殊。根据**分配内存方式**和**访问速度**的区别，可以把 js 的数组分为两种：

- 快数组：连续内存空间，在创建一个连续块并且数组中类型相同时产生。特点是访问读写快，但是不能随意删除和改变索引。
  通过`new Array(LENGTH)`创建的数组都是快数组，当数组中元素相同时，并且对该数组的操作都是数组原生操作（push、pop 这类），就是一一个快数组。
- 慢数组：哈希表形式的存储，表现为一个键、值、描述符 key、value、descriptor）三元组。
  当对 length 属性更改、访问元素的上面三个属性、直接修改索引等操作都会引起数组转化为慢数组。如果数组的值类型不同（如果是对象数组或者二维数组就一定是慢数组），也会变成慢数组。

参考：https://zhuanlan.zhihu.com/p/371236424

## `0.1 + 0.2 !== 0.3`

js 中的数字是 IEEE754 标准下的 64 位双精度浮点数：

- 1 位数符
- 11 位阶码，是移码，移码的计算就是原数的规范浮点数表示的指数值+1023
- 52 位尾数
  ![](https://pic.imgdb.cn/item/622f1c815baa1a80abcfa2fe.jpg)

该数字能表达的最大的数：`2^53 - 1`，使用`Number.MAX_SAFE_INTEGER`可以获取
同理最小数就是改变数符，即`-(2^53 - 1)`，用`Number.MIN_SAFE_INTEGER`获取
超过最大数的数都会被视为相等：
![](https://pic.imgdb.cn/item/622f1dd45baa1a80abd014a0.jpg)

0.1 和 0.2 的 IEEE754 表示为

```
0 01111111100 0.1100110011001100110011001100110011001100110011001101
0 01111111100 1.1001100110011001100110011001100110011001100110011010
```

相加得

```
0 01111111100 10.0110011001100110011001100110011001100110011001100111
```

小数点向左移动一位，阶码加 1，尾数超出的部分则 0 舍 1 入
结果就是

```
0 01111111101 0011001100110011001100110011001100110011001100110100
```

这个值和 0.3 的表示有差距。其核心原因是上一步的舍去使得相加出现了精度问题。

### 避免方法

使用 `Number.EPSILON` 误差范围。

```js
function isEqual(a, b) {
  return Math.abs(a - b) < Number.EPSILON;
}

console.log(isEqual(0.1 + 0.2, 0.3)); // true
```

`Number.EPSILON`的实质是一个可以接受的最小误差范围，一般来说为 `Math.pow(2, -52)` 。
​
如果不追求精度，使用`toFixed(2)`保留两位小数即可。

## 值的比较

js 比较两个变量的符号通常有`>`、`<`、`!=`、`==`、`===`、`!==`。

除了`===`和`!==`，其他比较方式都会适当转换两个值；如果两个值类型不同，则会转换成相同类型再比较。

> `==` 和其他比较大小的几个运算符转换的方式并不相同，因此很多时候会出现`>=`成立但是`==`不成立的情况。

### `==`的转换规则

判断流程：

1. 首先会判断两者类型是否相同，相同的话就比较两者的大小；类型不相同的话，就会进行类型转换；
1. 会先判断是否在对比 null 和 undefined，是的话就会返回 true
1. 判断两者类型是否为 string 和 number，是的话就会将字符串转换为 number
1. 判断其中一方是否为 boolean，是的话就会把 boolean 转为 number 再进行判断
1. 判断其中一方是否为 object 且另一方为 string、number 或者 symbol，是的话就会把 object 转为原始类型再进行判断。
   比如`'1' == {}`，相当于`'1' == '[object Object]'`

流程图：

![](https://pic.imgdb.cn/item/6231cd8d5baa1a80ab1b76e2.jpg)

因此大多数情况下，都是将不同类型转换为 number 再进行比较。

### 相同类型比较

- `boolean`：
  ![](https://pic.imgdb.cn/item/62272d635baa1a80ab510506.jpg)

- `string`
  字符串比较默认按照字典序和 Unicode 编码顺序综合排序，序号在前面的比较小，因此小于序号在后面的字符；多个字符按照字典序规则比较。

> 注意，只有不同类型比较才会把字符串转为数字；
> `'2' > '12'`结果为 true，因为两个都是字符串，直接按照字符串的比较方式比较。

- `object`
  对象永不相等，但是`{} >= or <= {}`结果为 true

- `number`和`bigint`都是正常比较大小
- `null`、`undefined`和自己都相同，同时他们互相也相等（`null == undefined`）

### 不同类型比较

![](https://pic.imgdb.cn/item/627b447c09475431290e3f9e.jpg)

#### 其他值到数字的转换规则

- `Undefined` 类型的值转换为 NaN。
- `Null` 类型的值转换为 0。
- `Boolean` 类型的值，`true` 转换为 1，`false` 转换为 0。
- `String` 类型的值转换如同使用 `Number()` 函数进行转换，如果包含非数字值则转换为 NaN，空字符串为 0。
- `Symbol` 类型的值不能转换为数字，会报错。
- 对象（包括数组）会首先被转换为相应的基本类型值，如果返回的是非数字的基本类型值，则再遵循以上规则将其强制转换为数字。

为了将值转换为相应的基本类型值，抽象操作 `ToPrimitive` 会首先（通过内部操作 `DefaultValue`）检查该值是否有`valueOf()`方法。如果有并且返回基本类型值，就使用该值进行强制类型转换。如果没有就使用 `toString()` 的返回值（如果存在）来进行强制类型转换。
如果 `valueOf()` 和 `toString()` 均不返回基本类型值，会产生 `TypeError` 错误。

> 如何让 if(a == 1 && a == 2)条件成立？
> 其实就是 valueOf 的应用，代码如下：
>
> ```js
> var a = {
>   value: 0,
>   valueOf: function () {
>     this.value++;
>     return this.value;
>   },
> };
> console.log(a == 1 && a == 2); //true
> ```

#### 其他值到布尔类型的值的转换规则

以下这些是假值：

- `undefined`
- `null`
- `false`
- +0、-0
- `NaN`
- `""`

假值的布尔强制类型转换结果为 false。
假值列表以外的都是真值。

#### `Object.is()`

是一种更精确的判断方式.
使用 `Object.is` 来进行相等判断时，一般情况下和三等号的判断相同，它处理了一些特殊的情况，比如 -0 和 +0 不再相等，两个 NaN 是相等的。

## null 和 undefined

1. `==`比较时结果为 true，也就是`null == undefinded`；
2. `null/undefined` 会被转化为数字：`null` 被转化为 0，`undefined` 被转化为 NaN。
   ![](https://pic.imgdb.cn/item/622731015baa1a80ab543ae7.jpg)

### null

null 代表的含义是空对象，并不是指一个值完全不存在；主要用于赋值给一些可能会返回对象的变量，作为初始化。
null 和其他值比较时相当于 0，但是和 0 比较时只有`>=`和`<=`成立。这是因为相等性检查`==`和普通比较符 `> < >= <=` 的代码逻辑是相互独立的。进行值的比较时，null 会被转化为数字，因此它被转化为了 0

> null 只和 undefined 非严格相等

![](https://pic.imgdb.cn/item/622731855baa1a80ab54998c.jpg)

---

`typeof null` 的结果是`'object'`，和`typeof Object`的结果一样

### undefined

undefined 代表的含义是未定义，一般变量声明了但还没有定义的时候会返回 undefined

undefined 不应该被与其他有效值进行比较，因为它并不会和任何值相等；唯一特例是与 null 和自己非严格相等：
![](https://pic.imgdb.cn/item/6227334b5baa1a80ab55ba5d.jpg)

## NaN 和 Infinity

NaN 指“不是一个数字”（not a number），NaN 是一个“警戒值”（sentinel value，有特殊用途的常规值），用于指出数字类型中的错误情况

```js
typeof NaN; //number
NaN !== NaN; //true
```

NaN 是一个特殊值，它和自身不相等，是唯一一个非自反的值。即`NaN !== NaN` 为 true。

---

Infinity 表示“无穷”，是一个数值。当一个数过大超出 js 最大数值很多时，也会被认作 infinity，不一定是数学上的无穷

```js
typeof Infinity; //'number'
Infinity === Infinity; //true
Infinity > 1; //true
```

### isNaN 和 isFinite

NaN 的判断主要是是通过`isNaN`方法。当一个变量类型转换出错或者计算出错时就会出现 NaN，这时可以用 isNaN 判断

```js
isNaN(Number("a")); //true
isNaN(1 / 0); //true
isNaN("a"); //true
```

`isNaN`在参数为 NaN 的时候会返回 true。它会先**尝试将参数转化为`Number`类型**，任何不能被转换为数值的的值都会返回 true，因此非数字值传入也会返回 true ，会影响 NaN 的判断（即，不是 NaN 但也不是数字类型时仍会返回 true）
更好的方法是使用`Number.isNaN`，相对于 isNaN 增加了对于数字的判断，只有是 Number 类型才会判断

```js
Number.isNaN("a"); //false
Number.isNaN(NaN); //true
```

---

infinity 的判定通过`isFinite`方法，但是注意这是判断“是有限的”，即正常数值会返回 true，无穷会返回 false。
和 isNaN 一样，参数会先被转为 Number 类型，然后进行判断；因此有时候会出现一些奇怪的判断结果
![](https://pic.imgdb.cn/item/6231ccc75baa1a80ab19f546.jpg)
同理也有它的`Number.isFinite`版本，也是类似的更加准确。

## 对象转化原始数据类型

一个变量类型为对象的变量可以通过一些类型转换的方式变为原始数据类型的变量；
主要有三种转换，类型转换有三种变体（variant），发生在各种情况下，被称为 `hint`（这个词找不到具体含义，大概意思是对象的转化目标）

### 1. 转化为 string

通过 alert 或者其他方式可以让对象变成字符串。

```js
alert(obj);

// 将对象作为属性键
anotherObj[obj] = 123;
```

这种方式隐式调用了`Object.prototype.toString`方法.
直接调用对象本身的 toString()方法也可以；
![](https://pic.imgdb.cn/item/6226ec605baa1a80ab1488be.jpg)

### 2. 转化为 number

绝大多数对象或者实例转化为数字相关是都会变成 NaN，对象间的相加减也是。
![](https://pic.imgdb.cn/item/6226ed4a5baa1a80ab153c99.jpg)

但是有几个特例，比如 Date 对象的实例是可以相加减的。
![](https://pic.imgdb.cn/item/6226edca5baa1a80ab159013.jpg)
date 实例直接运算相当于是时间戳的运算。

### 3. `default`

当运算符“不确定”期望值的类型时，会把对象转化为`default`类型。
比如，二元加法 + 可用于字符串（连接），也可以用于数字（相加），所以字符串和数字这两种类型都可以。因此，当二元加法得到对象类型的参数时，它将依据 `"default" hint` 来对其进行转换。
此外，如果对象被用于与字符串、数字或 `symbol` 进行`==`比较，这时到底应该进行哪种转换也不是很明确，因此使用`"default" hint`。

```js
// 二元加法使用默认 hint
let total = obj1 + obj2;

// obj == number 使用默认 hint
if (user == 1) { ... };
```

### 转化方法

在对象进行类型转换时，JavaScript 尝试查找并调用三个对象方法，每一个不存在则调用下一个方法：

1. 调用 `obj[Symbol.toPrimitive](hint)`，即自定义的 `Symbol.toPrimitive`方法；
1. 如果 hint 是 `"string"` ，优先调用 `obj.toString()` ，如果不存在则调用 `obj.valueOf()`
1. 如果 hint 是 `"number"` 或 `"default"` ，优先调用 `obj.valueOf()` 如果不存在则调用 `obj.toString()`。

### `Symbol.toPrimitive`

因此可以手动实现一个`Symbol.toPrimitive`方法，从而达到自定义转化的效果。

```js
let user = {
  name: "aaa",
  age: 18,

  [Symbol.toPrimitive](hint) {
    console.log(`target: ${hint}`);
    return hint === "string" ? `${this.name}` : this.age;
  },
};

alert(user); //aaa
console.log(+user); //18
```

### `toString/valueOf`

默认情况下，普通对象具有 toString 和 valueOf 方法：

- `toString` 方法返回一个字符串 `"[object Object]"`。
- `valueOf` 方法返回对象自身

同样的，可以自定义这两个函数：

```js
let user = {
  name: "John",
  money: 1000,

  // 对于 hint="string"
  toString() {
    return `name: "${this.name}"`;
  },

  // 对于 hint="number" 或 "default"
  valueOf() {
    return this.money;
  },
};
alert(user); // "name: John"
console.log(+user); // 1000
```

## 原始数据类型和引用数据类型

### 对象包装器

js 中的数据可以分为**原始数据类型**和**引用数据类型**。  
有 7 种原始类型：`string`，`number`，`bigint`，`boolean`，`symbol`，`null` ，`undefined`。  
JavaScript 允许访问这些原始类型的方法和属性，但是理论上原始类型并没有办法提供任何方法和属性。  
因此 js 会创建提供额外功能的特殊“**对象包装器**”，使用后即被销毁。  
“对象包装器”对于每种原始类型都是不同的，有 String、Number、Boolean、Symbol 和 BigInt，各自提供了不同的方法。

> null/undefined 没有任何方法，没有对应的“对象包装器”

因为包装器即用即毁的特征，我们不能向原始数据类型上添加任何自定义的方法和属性，不能存储额外的数据。

```js
let str = "str";
str.test = 1;
clg(str.test); //undefined
```

### 存储结构

- 栈：原始数据类型（Undefined、Null、Boolean、Number、String、Symbol）。这个栈实际上就是 js 的调用栈，变量存储在函数的执行上下文中
- 堆：引用数据类型（对象、数组和函数）

两种类型的区别在于存储位置的不同：

- 原始数据类型直接存储在栈（stack）中的简单数据段，占据空间小、大小固定，属于被频繁使用数据，所以放入栈中存储；
- 引用数据类型存储在堆（heap）中的对象，占据空间大、大小不固定。**如果存储在栈中，将会影响程序运行的性能**；引用数据类型在栈中存储了指针，该指针指向堆中该实体的起始地址。当解释器寻找引用值时，会首先检索其在栈中的地址，取得地址后从堆中获得实体。

堆和栈的概念存在于数据结构和操作系统内存中，在数据结构中：

- 在数据结构中，栈中数据的存取方式为先进后出。
- 堆是一个优先队列，是按优先级来进行排序的，优先级可以按照大小来规定。

在操作系统中，内存被分为栈区和堆区：

- 栈区内存由编译器自动分配释放，存放函数的参数值，局部变量的值等。其操作方式类似于数据结构中的栈。
- 堆区内存一般由开发着分配释放，若开发者不释放，程序结束时可能由垃圾回收机制回收。

> 闭包保存的变量是存在堆中的

## 检测数据类型的方法

### typeof

```js
console.log(typeof 2); // number
console.log(typeof true); // boolean
console.log(typeof "str"); // string
console.log(typeof undefined); // undefined
console.log(typeof function () {}); // function
//这些都是object
console.log(typeof []); // object
console.log(typeof {}); // object
console.log(typeof null); // object
```

typeof 的结果是一个全小写**字符串**；null 和 array 类型会被判断为 object，其他的正常；因此数组和 null 类型还需要其他方法区分

### instanceof

> instanceof 运算符可以用来测试一个对象在其原型链中是否存在一个构造函数的 prototype 属性。
> 说人话就是：用来判断一个构造函数是否是某个对象的原型，或者说这个对象的原型链上有没有这个构造函数。当然根据原型链的相关知识就知道，对象原型链上的不应该是构造函数本身，应该是构造函数的 prototype

`instanceof`本来适用于判断一个实例的原型；由于数据类型实际上就是其对应构造函数的实例，因此也可以借此来判断类型；但是只能用于引用类型的判断，基础类型不可以。

```js
console.log(2 instanceof Number); // false
console.log(true instanceof Boolean); // false
console.log("str" instanceof String); // false

console.log([] instanceof Array); // true
console.log([] instanceof Object); // true
console.log(function () {} instanceof Function); // true
console.log({} instanceof Object); // true
```

上面例子中的 Number、Boolean 等实际上是一个构造函数
![](https://pic.imgdb.cn/item/62260d3e5baa1a80ab8d563f.jpg)

> 原始数据类型是由这些构造函数构造出来的，但是这些并不是对象，**如果我们试图访问它们的属性，那么临时包装器对象将会通过内建的构造器 String、Number 和 Boolean 被创建。它们提供给我们操作字符串、数字和布尔值的方法然后消失。**这些对象的方法也驻留在它们的 `prototype` 中，可以通过 `String.prototype`、`Number.prototype` 和 `Boolean.prototype` 进行获取。
> 具体来说，主要有以下特点：
>
> 1. 和正常的构造函数类似，也可以像 prototype 上添加方法和属性，并且可以通过原始值访问。
> 2. 不能通过 new 的形式创建。通过 new 形式返回的是一个对象，和对应的原始数据类型不严格相等；但可以直接调用，比如`String('aaa')返回的就是'aaa'这个string类型的数据` > ![](https://pic.imgdb.cn/item/640b97f9f144a01007efce3c.jpg)
>    另外，null 和 undefined 是特例，不能访问其构造函数的 prototype。

### constructor

> constructor 有两个作用，一是判断数据的类型，二是对象实例通过 constrcutor 对象访问它的构造函数。

```js
console.log((2).constructor === Number); // true
console.log(true.constructor === Boolean); // true
console.log("str".constructor === String); // true
console.log([].constructor === Array); // true
console.log(function () {}.constructor === Function); // true
console.log({}.constructor === Object); // true
```

上面说过原始数据类型不能通过原型链判断，但是可以通过`constrcutor`找到其对应的构造函数。
但是相应的，如果手动改变一个对象的原型，在使用判断就会出错

```js
let num = 1;

num = Object.create(new Array());
console.log(num.constructor); //[Function: Array]
```

### `Object.prototype.toString`

调用 Object 对象的原型方法 `toString()`可以让每种数据类型有不同的表现：

![](https://pic.imgdb.cn/item/622612555baa1a80ab907a7a.jpg)

> 每个对象都有一个 toString() 方法，当该对象被表示为一个文本值时，或者一个对象以预期的字符串方式引用时自动调用。默认情况下，toString() 方法被每个 Object 对象继承。如果此方法在自定义对象中未被覆盖，toString() 返回 "[object type]"，其中 type 是对象的类型。

一些操作中会隐式调用`Object.prototype.toString.call()`，使得本来正常的输出变成这种形式。常见的就是 alert 方法

```js
const obj = {};
alert(obj); //[object Object]
```

`Object.prototype.toString.call()` 使用 Object 对象的原型方法 toString 来判断数据类型。

> 注意不能直接调用`toString()`并在括号中传入参数，应当使用 call 或者 apply 方法调用。
> 因为这个函数内部含有 this，应当使用 call 明确指定 this，否则可能是 undefined

```js
const a = Object.prototype.toString;

console.log(a.call(2));
console.log(a.call(true));
console.log(a.call("str"));
console.log(a.call([]));
console.log(a.call(function () {}));
console.log(a.call({}));
console.log(a.call(undefined));
console.log(a.call(null));

/*
[object Number]
[object Boolean]
[object String]
[object Array]
[object Function]
[object Object]
[object Undefined]
[object Null]
*/
```

---

`obj.toString()`的结果和`Object.prototype.toString.call(obj)`的结果不一样，是因为 toString 是 Object 的原型方法，而 Array、function 等类型作为 Object 的实例，都重写了 toString 方法。不同的对象类型调用 toString 方法时，根据原型链的知识，调用的是对应的重写之后的 toString 方法（function 类型返回内容为函数体的字符串，Array 类型返回元素组成的字符串…），而不会去调用 Object 上原型 toString 方法（返回对象的具体类型），所以采用 obj.toString()不能得到其对象类型，只能将 obj 转换为字符串类型；因此，在想要得到对象的具体类型时，应该调用 Object 原型上的 toString 方法。

同理，如果重写一个变量原型上的 `toString()`方法，也可以更改输出：

```js
let arr = [];
arr.__proto__.toString = function () {
  return Object.prototype.toString.call(this).slice(8, -1);
};
console.log(arr.toString()); //'Array'

let dog = new Dog();
dog.prototype.toString = function () {
  return `${this.name}`;
};
console.log(dog.toString()); //'taddy'
```

### 判断数组的方法

1. 通过 `Object.prototype.toString.call()`做判断

```js
Object.prototype.toString.call(obj).slice(8, -1) === "Array";
```

2. 通过原型链做判断

```js
obj.__proto__ === Array.prototype;
```

3. 通过 ES6 的 `Array.isArray()`做判断

```js
Array.isArray(obj);
```

4. 通过 `instanceof` 做判断

```js
obj instanceof Array;
```

5. 通过 `Array.prototype.isPrototypeOf`

这个属性实际上是 `Object.prototype` 上的，通过原型链访问到

```js
Array.prototype.isPrototypeOf(obj);
```

# 对象拷贝

比如现在有一个对象：

```js
let obj1 = {
  name: "zzx",
  age: 19,
  info: {
    height: 175,
    weight: 80,
  },
};
```

首先要了解,js 中普通的数据类型在复制时是复制值,而对象和数组等引用类型是复制地址
所以当我们直接`let obj2 = obj1`并不能复制,只是复制了 obj1 的引用
所以我们需要有一个拷贝的方法,可以保证新对象与源对象不一样

## 浅拷贝

### `Object.assign`

```js
let obj2 = Object.assign({}, obj1);
```

`Object.assign`实际上是把一个对象所有的可枚举属性和自有属性(hasOwnProperty)复制到另一个对象中去，并返回复制的新对象。
如果通过 difineProperty 设置了属性不可枚举（不可遍历），则不能复制。
同时只会复制对象自己身上的属性，不会复制原型链上的属性。

### 展开运算符

```js
let obj2 = { ...obj1 };
```

而如果数组/对象互相嵌套,就需要进行深拷贝,方法有:

## 深拷贝

### JSON

原理是 `JSON.parse` 转换的对象是 `JSON.stringify` 生成的字符串，而不是原先的 obj

```js
/* JSON先转字符串再转回来的方法可以实现 */
let obj2 = JSON.parse(JSON.stringify(obj1));
```

这个方法有很大的问题。比如如果某个值是一个函数，json 就会忽略掉，这样再转回来的时候就会缺少属性

### 递归

详见下方手写深拷贝

### lodash 库函数 cloneDeep

```js
let obj2 = _.cloneDeep(obj1);
```

# Symbol

Symbol 可以理解为一个对象的**隐藏**属性, 而在构造函数或者类中还可以作为方法使用, 在内建类中起到很重要作用。

## 创建方法

```js
const obj = {
  name: "xiaoming",
};
const userId = Symbol("myId"); //创建一个symbol, myId是userId的description
obj[userId] = "0001"; //使用创建symbol时的变量值引用
console.log(obj[userId]);
```

- Symbol() 的参数是它的 description, 可以方便在后续使用。
- Symbol 只能用方括号访问, 访问的结果值是定义 Symbol 时的赋值，而不是 description（即不是调用 Symbol 方法传入的参数，而是具体用中括号放入对象时赋的值，比如上面的`userId`）

也可以通过对象字面量的方式直接定义：(注意要先声明 Symbol)

```js
let userId = Symbol();
const obj2 = {
  name: "xiaohong",
  [userId]: "0002",
};
```

## Symbol 的特点

1. 不会被 `forin`/`for` 循环等遍历,用 `Object.keys` 也不会访问,即对外部方法不可见。

> 但是其实 Symbol 并不是完全隐形的，有一个内建方法 `Object.getOwnPropertySymbols(obj)` 允许获取所有的 Symbol。还有一个名为 `Reflect.ownKeys(obj)` 的方法可以返回一个对象的**所有**键，包括 Symbol。
> 同时，`Object.assign` 会同时复制字符串和 symbol 属性，通过展开运算符等拷贝方式（深浅拷贝都会）也会复制 Symbol 属性；但是继承的对象并不能访问到父原型的 Symbol
>
> ```js
> const id = Symbol("id");
> let userObj = {
>   name: "aaa",
> };
> userObj[id] = "001";
> console.log(Object.getOwnPropertySymbols(userObj));
>
> let anotherUserObj = { ...userObj };
> console.log(Object.getOwnPropertySymbols(anotherUserObj));
>
> let childUserObj = Object.create(userObj);
> console.log(Object.getOwnPropertySymbols(childUserObj));
> ```
>
> ![](https://pic.imgdb.cn/item/6226e4795baa1a80ab0f181d.jpg)

2. 任何两个 symbol 都不相等,即使 description 相同。
   但是`Symbol.for()`可以创建一个被全局识别的 Symbol。也就是说由`Symbol.for()`创建的 Symbol 是全局唯一的，相同 description 的 symbol 会被判定为相同 Symbol。
   这个全局 Symbol 并不是某个对象的隐藏属性，而是一个**全局变量**。

   ```js
   let id = Symbol.for("id"); // 如果该 Symbol 不存在，则创建它
   // 再次读取（可能是在代码中的另一个位置）
   let idAgain = Symbol.for("id");
   // 相同的 Symbol
   clg(id === idAgain); // true
   ```

3. JavaScript 使用了许多系统 Symbol，比如后面的 `Symbol.iterator` 迭代器对象、 `Symbol.hasInstance` 用于更改 instanceof 访问对象等等。这些系统 Symbol 并不是某个对象的属性，更像 Symbol 原型上的一些属性和方法，通过在对象或类中添加并合适赋值，可以改变一些对象的性质，供一些内置方法使用。

4. Symbol 不能被默认字符串化，也就是不能调用`Object.prototype.toString.call(Symbol)`；因此 alert 一个 Symbol 会报错。显示一个 Symbol 可以直接调用`toString()`方法

```js
const id = Symbol("id");
let userObj = {
  name: "aaa",
};
userObj[id] = "001";
console.log(id.toString()); //'Symbol(id)'
```

## Symbol 相关方法

`Symbol.for()`:获取或创建一个**全局的**symbol 对象, 比如

```js
let id = Symbol.for("id");
let userId = Symbol.for("id");
id === userId; //true
```

他和直接 `Symbol()`的区别在于:

- 会被登记在全局环境中供搜索，也就是相当于声明了一个全局变量，可以被`Symbol.keyFor()`获取到。
- 可以确保每次访问相同名字的 Symbol 时，返回的都是相同的 Symbol。

---

`Symbol.keyFor()`：`Symbol.for()`的反向调用, 参数是定义**全局 Symbol** 时的`description`，返回这个全局 Symbol 对象。不适用于非全局 Symbol，如果 Symbol 不是全局的会返回 undefined。

```js
let symid = Symbol.for("id");
Symbol.keyFor(id); //symid
```

# 迭代器和可迭代

## forof 和 forin

- `for...of` 循环可以使用的范围包括
  - 数组
  - Set 和 Map
  - 某些类似数组的对象（比如 arguments/DOM NodeList）
  - Generator
  - 字符串
- `for...in` 以任意顺序遍历一个**对象**的除 Symbol 以外的可枚举**属性**
  - 常用于对象属性的遍历,比如`for(let key in obj){}`
  - 属性特征值被设置不可枚举的不能被遍历
  - **会遍历对象的整个原型链**，性能非常差不推荐使用
  - Symbol 不能被遍历到

## 可迭代对象

一般我们称可以被按一定顺序遍历的对象为可迭代对象, 更具体的概念是可以使用 for...of 的对象(注意不是 for...in)
可迭代对象主要有以下两类:

- 数组等本身可遍历的对象
- 添加了迭代器的对象

## 迭代器

正常来说, 对象是不能使用 forof 遍历的(不是可迭代对象), 但是我们可以手动添加一个迭代器
迭代器特点:

- 以`[Symbol.iterator]`作为方法名
- 返回一个迭代器对象, 内部有:
  - 一些数据, 用于限制迭代次数(不是必须)
  - 有一个 `next()`函数,每次迭代都会调用这个函数,这个函数将会返回 `{done:.., value :...}` 格式的对象(必须)
- 另外, 可迭代对象也可以用展开运算符展开

```js
let range = {
  from: 1,
  to: 5,
};

range[Symbol.iterator] = function () {
  return {
    current: this.from,
    last: this.to,
    //next() 在 for..of 的每一轮循环迭代中被调用
    next() {
      if (this.current <= this.last) {
        return { done: false, value: this.current++ };
      } else {
        return { done: true };
      }
    },
  };
};
//效果:
for (let i of range) {
  console.log(i);
} //12345
console.log([...range]);
```

比如封装一个 range 函数:

```js
function createRange(from, to) {
  return {
    from,
    to,
    [Symbol.iterator]() {
      return {
        current: this.from,
        last: this.to,
        next() {
          if (this.current <= this.last) {
            return { done: false, value: this.current++ };
          } else {
            return { done: true };
          }
        },
      };
    },
  };
}
for (let i of createRange(1, 10)) {
  console.log("hello world");
} //10遍hello world
```

## arrayLike

对于一些像数组的对象, 我们可以通过`Array.from()`变成真正的数组
像数组的对象有以下特点:

- 键名为数字, 或者数字型的字符(比如'1')
- 有 length 属性
- 本身没有数组相关方法, 比如 push pop

> 对于 Array.from()的参数来说，只有一个要求，就是必须有 length 属性即可。

举个栗子:

```js
let arrLike = {
  //声明一个类数组对象,特点是键名为数字,并且有length属性
  //可以是任意数字
  2: "aaa",
  1: "bbb",
  length: 3, //不声明长度将会是空
};
let arrFrom = Array.from(arrLike); //[undefined,'bbb','aaa']
```

**from 方法也可以用于可迭代对象的转化,结果就是迭代范围的一个数组**

```js
let arrRange = Array.from(range); //[1,2,3,4,5]
```

除了`Array.from`，还有其他方式也可以把类数组转化为数组：

1. 展开运算符

```js
function foo() {
  const arrArgs = [...arguments];
  arrArgs.forEach((a) => console.log(a));
}
```

2. `slice(arr)`/`splice(arr,0)`/`concat([],arr)`
   这三个方法都不能直接调用，因为不是一个数组；
   需要通过`Array.prototype.xxx.call()`的方式调用，相当于借用了 Array 上的方法

```js
Array.prototype.concat.apply([], arrayLike); //和一个空数组合并
Array.prototype.slice.call(arrayLike); //不切割数组，返回一个原本完整的
Array.prototype.splice.call(arrayLike, 0); //从第0个开始，但不删除也不插入元素
```

# Map 和 Set

Map 和 Set 都是一种类对象的数据类型, 都是通过构造函数方法创建

## Map

### Map 的特点

- Map 是一种键可以为任意值的对象, 对象的键一般只能是字符串, map 的键可以是数字/布尔值等任何类型
- 本身可迭代
- 值和键只能通过 get 和 set 等方法获取, 不能像对象一样用`.`获取;**注意 set 和 get 都返回 map 本身,所以可以链式调用**
- 创建 map 可以用一组每个为一个键值对的数组(如下)传入

比如:

```js
const map = new Map();
map.set(true, "hello").set(false, 20).set(12, "12");
console.log(map.get(true)); //'hello'

//或者
//实际上map就是这样的数据形式，即一个二维数组
const mapByArr = new Map([
  ["apple", 10],
  [false, 20],
  [12, "12"],
]);
```

### Map 迭代

如果要在 map 里使用循环，可以使用以下三个方法：

- `map.keys()` —— 遍历并返回所有的键
- `map.values()` —— 遍历并返回所有的值
- `map.entries()` —— 遍历并返回所有的键值对
- `map.forEach((value,key,map)=>{})`：Map 中也有类似数组的 forEach 方法，操作基本一致。

```js
for (let [key, value] of mapByArr) {
  console.log(key, value);
}
for (let key of mapByArr.keys()) {
  //遍历map对象需要用keys()等方法,返回一个可迭代的包含键值信息的数组,
  console.log(key);
}
for (let value of mapByArr.values()) {
  console.log(value);
}
for (let entry of mapByArr.entries()) {
  console.log(entry);
}
```

Map 和 Set 本身也是可以被直接用 forof 迭代的。迭代的值和对其调用 foreach 的参数相同，map 为一个[key,value]数组，set 为具体值

```js
const map = new Map([
  ["name", "aaa"],
  ["age", 18],
]);
for (const arr of map) {
  console.log(arr); // ['name','aaa'] ['age',18]
}
```

### Map 和对象相互转化

- `Object.entries()` 可以把已有对象转成 map 对象
- `Object.fromEntries()`**给定一个 [key, value] 键值对的数组**，它会根据给定数组创建一个对象. 注意不是一个直接的 Map 对象.这个创建方法和上面的 Map 构造函数参数很像

```js
const user = {
  name: "xiaoming",
  age: 18,
};
const userMap = new Map(Object.entries(user)); //这种方法可以把已有对象转成map对象
const objFromMap = Object.fromEntries([
  ["banana", 1],
  ["orange", 2],
  ["meat", 4],
]);
```

## Set

### Set 的特点

- Set 只有值没有键, 并且不可重复
- 跟 Map 一样,可迭代,不可直接访问,要借助 Set 自带的方法

### Set 的主要方法

- `new Set(iterable)` : 创建一个 set，**如果提供了一个 iterable 对象（通常是数组）,将会从数组里面复制值到 set 中**
- `set.add(value)` : 添加值,也是返回 set 本身,可以链式调用. 创建 set 还可以使用一个数组
- `set.delete(value)` : 删除值
- `set.has(value)`:是否存在值
- `set.clear()`:清空
- `set.size`:返回元素个数。

set 也有像 map 一样的 keys()/values()/entries()方法, **虽然没有键**

- `set.keys()` —— 遍历并返回所有的值
- `set.values()` —— 与 `set.keys()`作用相同，这是为了兼容 Map，
- `set.entries()` —— 遍历并返回所有的实体[value, value]，它的存在也是为了兼容 Map。
- `set.forEach((value,value,set)=>{})`：set 上的 foEach 方法，前两个参数相同都是当前值，是为了兼容 Map 所以重复两次。

```js
const set = new Set();
let xiaoming = { name: "xiaoming" };
let xiaohong = { name: "xiaohong" };
set.add(xiaoming).add(xiaoming).add(xiaohong);
console.log(set.size); //2
```

这里添加了两次 xiaoming，但是因为重复所以值加入了一个。
注意,**当添加的是引用对象时不会自动去重**,比如:

```js
const set = new Set();
set
  .add({ name: "xiaoming" })
  .add({ name: "xiaoming" })
  .add({ name: "xiaohong" });
console.log(set.size); //3
```

原因是引用对象的两个`{name:'xiaoming'}`本质上不算一个值(两种地址), 所以不会被去重；上面一个例子的两个 xiaoming 是同一个地址，所以会被去重

### Set 迭代

可以使用 for..of 或 forEach 来遍历 Set：

```js
const setFromArr = new Set(["hello", "world", 100, true]);
setFromArr.forEach((val) => console.log(val));
```

或者用 set.keys/set.values/set.entries 等方法

### Set 的转换

set 在类型上更像数组而不是对象, 所以 Set 可以用 Array.from 生成一个数组

```js
const arr = [1, 2, 3];
let set = new Set(arr); //生成set
const arr1 = Array.from(set); //返回数组
```

### Set 用于数组去重

```js
const arrRepeated = [1, 2, 2, 2, 3, 4, 5, 5];
const setNoRepeat = new Set(arrRepeated);
console.log(Array.from(setNoRepeat));
```

## WeakMap 和 WeakSet

这两者的出现都是为了解决 Map 和 Set 的垃圾回收问题。
由 js 的垃圾回收机制可以直到，如果一个对象的索引被置为 null，这个对象就相当于不可访问，因此就会被垃圾回收机制回收；
但是如果对象作为 Map 的键，那么当 Map 存在时，该对象也将存在。它会占用内存，并且应该不会被（垃圾回收机制）回收。
WeakMap 在这方面有着根本上的不同。它不会阻止垃圾回收机制对作为键的对象的回收。

### 特点

1. WeakMap 的键必须是对象，其他的都不可以

```js
let weakMap = new WeakMap();

let obj = {};

weakMap.set(obj, "ok"); // 正常工作（以对象作为键）

// 不能使用字符串作为键
weakMap.set("test", "Whoops"); // Error，因为 "test" 不是一个对象
```

2. WeakMap 没有任何迭代方法，只有基础的四个方法：

- `weakMap.get(key)`
- `weakMap.set(key, value)`
- `weakMap.delete(key)`
- `weakMap.has(key)`

3. WeakSet 和 WeakMap 类似，只能向 WeakSet 添加对象；并且也不能迭代只有基础的四个方法。

4. WeakMap 中，如果一个作为键的对象的索引被置 null，这个键和值也会被垃圾回收；WeakSet 同理。

### 使用场景

- WeakMap 的主要应用场景是 **额外数据的存储**，可以理解为缓存或者用于存储一些核心之外的边角数据。这些数据就应该与主要的对象共存亡，当对象被清除，这些数据不应该被保留。
- WeakSet 可以作为额外的存储空间。但并非针对任意数据，而是针对“是/否”的事实。WeakSet 的元素可能代表着有关该对象的某些信息。WeakSet 更多执行一个`has`的操作，只用判断是否“有过”

# 对象属性配置

## 对象的默认属性

对象的每个属性都有对应的四个默认属性,分别为

- `value`: 本身的值
- `writable`:默认 true, false 为只读
- `enumerable`: 是否可枚举,false 即不会出现在 forin/object.keys 这种遍历中,直接打印也不会出现
- `configurable`: 是否可配置。只有该属性的 configurable 为 true 时，writable 和 enumerable 才可以改变

设置或更改属性的属性值使用 `Object.defineProperty` 方法

```js
const user = {
  name: "xiaoming",
};

Object.defineProperty(user, "name", {
  value: "xiaohong",
  writable: false, //只读
  enumerable: false, //不可枚举,即不会出现在forin/object.keys这种遍历中,直接打印也不会出现
  configurable: false, //不可配置,即不可删除/覆盖,也不能修改只读属性等,但是可以改属性值
});
```

当属性是对象定义时就拥有，则后面三个配置项都是 true；
如果属性是通过 defineProperty 创建（对象定义时没有），则后面的配置项默认是 false

```js
const user = {
  name: "xiaoming",
};

Object.defineProperty(user, "age", {
  value: 18,
});
Object.getOwnPropertyDescriptor(user, "age"); //{value: 18, writable: false, enumerable: false, configurable: false}
```

获取属性可以用 getOwnPropertyDescriptor 方法

```js
const descriptor = Object.getOwnPropertyDescriptor(user, "name");
/* 属性描述符：
{
  "value": "xiaoming",
  "writable": true,
  "enumerable": true,
  "configurable": true
}
*/
```

## getter 和 setter

> getter 和 setter 是对象的访问器属性。它们本质上是用于获取和设置值的**函数**，但**从外部代码来看就像常规属性**

意思就是, getter 和 setter 定义为函数, 但访问是用属性的方式访问
getter 和 setter 的主要特点为:

- getter 和 setter 是属性不是方法,得用属性的形式访问
- 定义一个 getter 的方法是 `get funcname(){}`,相当于 get 后面一个正常的方法, get 只是一个修饰符;
- setter 需要一个参数, 作为设置的值;getter 不设返回值就无法获取访问结果(下面的 student.name 会是 undefined)

比如这个例子中,get 和 set 是属性不是方法,得用属性的形式访问而不是 name('xx')

```js
const student = {
  get name() {
    return this._name;
  },
  set name(val) {
    if (val.length < 3) return;
    //长度短不予设置
    else this._name = val;
  },
};
//注意,
student.name = "aa";
console.log(student.name); //undefined
student.name = "xiaoming";
console.log(student.name); //设置成功
```

同理我们也可以用 `defineProperty` 给对象加 getter 和 setter

```js
const user = {
  name: "xiaoming",
  age: 18,
};
Object.defineProperty(user, "birthday", {
  //通过define给对象加属性
  get() {
    const date = new Date();
    return date.getFullYear() - this.age;
  },
  set(val) {
    const date = new Date();
    this.age = date.getFullYear() - val;
  },
});
user.birthday = 2002; //触发set,改动了age
console.log(user.birthday, user.age); //birthday的访问通过get
```

这里有一个 this 指向的问题, 也适用于后面的类和构造函数:
**this 的指向始终只和`.`之前的值有关,跟内部引用和定义没关系**
另外, 类和构造函数也可以加 getter, 在其实例的访问时会调用该 getter

```js
function User(birthday) {
  this.birthday = birthday;
  Object.defineProperty(this, "age", {
    get() {
      let todayYear = new Date().getFullYear();
      return todayYear - this.birthday.getFullYear();
    },
  });
}
let john = new User(new Date(2002, 6, 1));
join.age; //触发getter, 返回计算值19
```

# 函数声明

函数有两种创建方式：函数声明和函数表达式。
无论函数是如何创建的，函数都是一个值。

```js
/*函数声明*/
function hello() {
  //...
}

/*函数表达式*/
let hello = function () {
  //...
};
```

函数声明和函数表达式的区别在于：JavaScript 引擎会在 **什么时候** 创建函数。

- 函数表达式是在代码执行到达时被创建，并且仅从那一刻起可用；
- 函数声明被定义之前，它就可以被调用，无论声明被写在何处；

并且，当一个函数声明在一个**代码块内**时，它在该代码块内的任何位置都是可见的。但在代码块外不可见。

```js
if (age < 18) {
  function welcome() {
    alert("Hello!");
  }
} else {
  function welcome() {
    alert("Greetings!");
  }
}

welcome(); // Greetings!
// 声明式的函数会变量提升。类似var变量，没有块级作用域
```

# 函数对象

在 js 中, 函数本质也是个对象, 因此函数也有对象的一些属性
一个空函数:
![](https://pic.imgdb.cn/item/61b0c2282ab3f51d915ba4b5.png)

- prototype 是函数固有的原型对象(原型链会讲到)
- name:函数名
- length:函数参数数量
- arguments:函数参数(返回一个数组，注意结尾的 s)

以及一些方法

## 函数属性

因为函数本质也是个对象, 所以我们可以给函数本身添加一些属性,比如:

```js
function sayHello(val) {
  console.log(val);
  sayHello.counter++; //使用函数名自定义属性
}
sayHello.counter = 0;
```

每调用一次 sayHello, 内部的 counter 就会加一.注意这里 counter 相当于一个全局存储的变量, 通过函数对象直接访问,并存储在全局。

> 函数属性可以用于在一些情况下替代闭包，同时这个闭包可以直接在外部访问
>
> ```js
> function makeCounter() {
>   function counter() {
>     return counter.count++;
>   }
>   counter.count = 0;
>   return counter;
> }
>
> let counter = makeCounter();
> counter.count = 10;
> alert(counter()); // 10
> ```
>
> 不仅是属性，方法也可以
>
> ```js
> function makeCounter() {
>   let count = 0;
>   function counter() {
>     return count++;
>   }
>   counter.set = (value) => (count = value);
>   counter.decrease = () => count--;
>   return counter;
> }
> ```

## IFE

定义式的函数可以额外拥有一个内部的名字, 该函数名可以用于递归, 外部不可见

```js
const sayHi = function _sayHi(who) {
  if (who) {
    alert(`Hello, ${who}`);
  } else {
    _sayHi("Guest");
  }
};
```

# 函数柯里化

> 柯里化是一种函数的转换，它是指将一个函数从可调用的 f(a, b, c) 转换为可调用的 f(a)(b)(c)。

## 原理

### 基本原理

举个栗子,两个参数的柯里化:

```js
function curry(f) {
  return function (a) {
    return function (b) {
      return f(a, b);
    };
  };
}
function sum(a, b) {
  return a + b;
}
let add = curry(sum);
add(1)(2); //3
```

一步步分析原理:

1. 首先调用 curry(sum),会返回第一层函数,即 add 函数

```js
let add = function (a) {
  return function (b) {
    return f(a, b);
  };
};
```

2. 然后 add(1),触发第一层,相当于参数 a = 1;进入第二层

```js
add(1) = function(b){
    return f(1,b)
}
```

3. 再调用 add(1)(2),到最后一层,把参数 b 传入

```js
add(1)(2) = f(1,2)
```

4. 最后返回 sum(1,2)的值 3

因此柯里化的过程实际上就是拆分参数的过程;把多个参数按照一定分组分开调用;

### 真正实现

还有更高级的写法,比如递归:

```js
function curry(func) {
  return function curried(...args) {
    if (args.length >= func.length) {
      return func(args);
    } else {
      return function (...args2) {
        return curried(...args, ...args2);
      };
    }
  };
}
function sum(a, b, c) {
  return a + b + c;
}
let add = curry(sum);
add(1, 2, 3);
add(1)(2, 3);
add(1)(2)(3);
```

同样原理如下:(以 `add(1)(2,3)`为例)

1. add 函数相当于这个:

```js
function curried(...args) {
  if (args.length >= func.length) {
    return func(args);
  } else {
    return function (...args2) {
      return curried(...args, ...args2);
    };
  }
}
```

2. 然后产生 add(1),即`args = [1]`;因为参数数量少于函数最大参数数量,所以进入 else,即 add(1)又是这个函数:

```js
function (...args2) {
    return curried(1, ...args2);
};
```

3. 调用 add(1)(2,3),后面的(2,3)相当于 add(1)这个函数的参数,被传入...args2 中

```js
function (2,3) {
    return curried(1, 2,3);
};
```

4. 递归,再次进入 1 中的函数;但是这次参数为三个,所以直接调用 sum(1,2,3)

```js
add(1)(2, 3) === sum(1, 2, 3); //true
```

5. 多个参数同理,这就是柯里化的根本原理

另外,lodash 库中有`_.curry` 可以用做柯里化一个函数,其源码如下

```js
function curry(func) {
  return function curried(...args) {
    if (args.length >= func.length) {
      return func.apply(this, args);
    } else {
      return function (...args2) {
        return curried.apply(this, args.concat(args2));
      };
    }
  };
}
```

## 使用

柯里化可以适用于函数多个参数的分开调用/延时执行等地方,并且可以创建偏函数(即原函数中只有一个参数的新函数)
比如:

```js
function getData(method, url, type) {
  //...
}
```

我们可以把它柯里化,然后转成一个固定 method 的偏函数

```js
getData = _.curry(getData)
let getDataByPost = getData("POST")
getDataByPost("/...",...)
```

（当然这个更方便的实现是通过 bind

# 箭头函数

JavaScript 的精髓在于创建一个函数并将其传递到某个地方。在诸如`forEach`、`map`这样的函数中，通常不想离开当前上下文。这就是箭头函数的主战场。

箭头函数的主要特点如下：

### 没有 this

箭头函数内部的 this，实际上是其外层执行上下文的 this。之前说 this 实际上就是函数**执行上下文**的指代，因此也可以理解为箭头函数的 this 指代的仍是其外层的。
注意这个外层是**调用**时的外层，就像执行上下文只有在执行时才能确定；

```js
let obj = {
  name: "aaa",
  sayName: () => {
    console.log(this.name);
  },
};
obj.sayName(); //undefined
```

这个例子中，sayName 是一个箭头函数，自身代码中的 this 实际上是调用时的外层`window`，因此不存在该属性。
如果是一个正常的函数，前面有对象和点访问符的情况，this 将会指向该对象，而不是直接指向外层。

从 Babel 翻译 this 也可以看出来：

```js
// ES6
const obj = {
  getArrow() {
    return () => {
      console.log(this === obj);
    };
  },
};

// ES5
var obj = {
  getArrow: function getArrow() {
    var _this = this;
    return function () {
      console.log(_this === obj);
    };
  },
};
```

可以看到箭头函数的 this 实际上是外层 this 的一个闭包。

### 不存在 prototype

箭头函数不存在自己的 prototype

```js
const arrow = () => {};
arrow.prototype; //undefined
```

因此箭头函数也不会拥有普通函数中的一些属性和方法，同样也不能访问函数对象，给函数本身添加属性

### 没有 arguments 属性

但是仍可以这样访问参数：

```js
const arrow = (...args) => {
  //...
};
```

但是在一些函数包装器中就不合适，因此函数装饰器返回的函数尽量采用普通函数。

> 箭头函数仍有`name`和`length`属性。

### 不能 new

由于没有自己的 this 和 prototype，箭头函数也不能作为构造函数去 new

### 没有 super

在类中的箭头函数上调用 super 会和 this 一样的效果，即从外层访问

### `call()`、`apply()`、`bind()`等方法不能改变箭头函数中 this 的指向

```js
var id = "Global";
let fun1 = () => {
  console.log(this.id);
};
fun1(); // 'Global'
fun1.call({ id: "Obj" }); // 'Global'
fun1.apply({ id: "Obj" }); // 'Global'
fun1.bind({ id: "Obj" })(); // 'Global'
```

# 闭包

## 基本概念

### 执行上下文

参考：https://blog.poetries.top/browser-working-principle/guide/part2/lesson07.html

执行上下文，里边有变量环境、词法环境、outer 和 this 四部分
![](http://blog.poetries.top/img-repo/2019/11/1.png)
![](https://static001.geekbang.org/resource/image/06/13/0655d18ec347a95dfbf843969a921a13.png)

#### 执行上下文的伪代码表示

词法环境的伪代码如下：

```js
GlobalExectionContext = {  // 全局执行上下文
  LexicalEnvironment: {       // 词法环境
    EnvironmentRecord: {     // 环境记录
      Type: "Object",           // 全局环境
      // 变量绑定在这里
      outer: <null>           // 对外部环境的引用
    }
  },
  FunctionExectionContext: { // 函数执行上下文
    LexicalEnvironment: {     // 词法环境
      EnvironmentRecord: {    // 环境记录
        Type: "Declarative",      // 函数环境
        // 变量绑定在这里      // 对外部环境的引用
        outer: <Global or outer function environment reference>
    }
  }
}
```

变量环境是存储 var 定义的变量的另一个词法环境

```js
 VariableEnvironment: {  // 变量环境
    EnvironmentRecord: {
      Type: "Object",
      // 标识符绑定在这里
      c: undefined,
    }
    outer: <null>
  }
}
```

---

举一个实际的例子，比如下面这段代码：

```js
let a = 20;
const b = 30;
var c;

function multiply(e, f) {
  var g = 20;
  return e * f * g;
}

c = multiply(20, 30);
```

对应的伪代码执行上下文如下：

```js
GlobalExectionContext = {//全局上下文，包裹着全局定义的函数、变量

  ThisBinding: <Global Object>,// this的绑定，这里是全局上下文中的this，即全局对象(window)

  LexicalEnvironment: {  // 词法环境
    EnvironmentRecord: {
      Type: "Object",
      // 标识符绑定在这里
      a: < uninitialized >,  // a被定义但是未被初始化，只有执行到特定的行才会被初始化，并且这一行之前也不能使用
      b: < uninitialized >,
      multiply: < func >  // 函数被定义但未被初始化
    }
    outer: <null>
  },

  VariableEnvironment: {  // 变量环境
    EnvironmentRecord: {
      Type: "Object",
      // 标识符绑定在这里
      c: undefined,  // var定义的变量会被初始化为undefined，可以使用但是没有意义
    }
    outer: <null>
  }
}

FunctionExectionContext = {  // 函数的执行上下文

  ThisBinding: <Global Object>,// 函数内部的this

  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // 标识符绑定在这里
      Arguments: {0: 20, 1: 30, length: 2},  // 参数被保存在词法环境，可以看到参数实际上是一个单独的块作用域
    },
    outer: <GlobalLexicalEnvironment>  // 函数的outer指向全局
  },

  VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // 标识符绑定在这里
      g: undefined
    },
    outer: <GlobalLexicalEnvironment>
  }
}
```

当形成这样的执行上下文后，就会放入 js 调用栈中执行。当 Javascript 引擎开始执行第一行脚本代码的时候，它就会创建一个全局执行上下文然后将它压到执行栈中；每当引擎碰到一个函数的时候，它就会创建一个函数执行上下文，然后将这个执行上下文压到执行栈中。也就是说，执行上下文是在开始执行的时候创建的，创建后立即被压入执行栈。
执行过程会进行赋值、代码执行等操作，执行完成后会回收出栈的执行上下文并销毁。

#### 词法环境(Lexical Environments)

- 词法环境是一个用于"登记"执行上下文中的变量/函数等的结构
- 词法环境创建的 5 种类型:
  - 函数
  - with 代码块
  - catch 代码块(注意不是 try)
  - 全局
  - eval
    ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/12/3/1677429807aea76d~tplv-t2oaga2asx-watermark.awebp)
- 词法环境对象由两部分组成：
  - 环境记录(`Environment Records`) :一个存储**所有局部变量**作为其属性（包括一些其他信息，例如 this 的值）的对象。就是登记环境内部变量的地方
  - 外部引用(`outer`):对外部词法环境的**引用**，与外部代码相关联。就是包含本自己词法环境的父外部词法环境,是作用域链连起来的根本
- 词法环境在 js 执行代码之前就已经创建了,也就是说在词法环境内的变量都会被先创建,但是不能引用;必须等待 let/const/函数声明来"登记"了之后才可以
- 我们常说的变量只是环境记录的一个属性
- 词法环境是嵌套的。内部的词法环境会通过 outer 和外部词法环境相连;当代码要访问一个变量时 —— 首先会搜索内部词法环境，然后搜索外部环境，然后搜索更外部的环境，以此类推，直到全局词法环境。

举个例子,比如下面这段代码:

```js
function makeCounter() {
  let count = 0;
  return function () {
    return count++;
  };
}
let counter = makeCounter();
```

会创建这样的嵌套词法环境:
![](https://pic.imgdb.cn/item/61b1d3072ab3f51d91da2eb8.png)
当我们调用 counter 函数,

1. counter 创建自己的词法环境
2. counter 函数查找自己词法环境内部的 count 变量;
3. 上一步没找到,就会根据 outer 到外部词法环境,也就是 makeCounter 函数的词法环境查找;
4. 如果再找不到就会到全局,在哪找到就会在哪修改

这是执行之后的:
<img src="https://pic.imgdb.cn/item/61b1d4162ab3f51d91dac8ca.png" style="zoom: 200%;" />

#### 变量环境(Variable Environment)

相对于词法环境:

- 两者本质上差不多,变量环境是一种特殊的词法环境
- 变量环境用来存储 var 声明和函数声明
- 词法环境存储 let/const 以及各种块声明

#### 执行上下文中的 this

https://blog.poetries.top/browser-working-principle/guide/part2/lesson11.html

### 作用域和作用域链

- 作用域：规定了如何查找变量，也就是确定当前执行代码对变量的访问权限。换句话说，作用域决定了代码区块中变量和其他资源的可见性。（全局作用域、函数作用域、块级作用域）
- 作用域链：从当前作用域开始一层层往上找某个变量，如果找到全局作用域还没找到，就放弃寻找 。这种层级关系就是作用域链。

https://juejin.cn/post/6844903797135769614

> 对于 for 循环的块级作用域问题也是常考点。比如这个例子：
>
> ```js
> for (var i = 0; i < 3; i++) {
>   setTimeout(() => console.log(i), 1000);
> }
>
> for (let i = 0; i < 3; i++) {
>   setTimeout(() => console.log(i), 1000);
> }
> ```
>
> 这两个的输出分别为：
>
> - 1 秒后同时输出 3 个 3，因为 var 不具有块级作用域，每次循环都会重新赋值，最终值为 3
> - 1 秒后同时输出 0 1 2
>
> 其实只要这样看待就很好理解：
>
> ```js
> {
>   var i = 0;
>   {
>     var i = 1;
>     setTimeout(() => console.log(i), 1000);
>   }
>   {
>     var i = 2;
>     setTimeout(() => console.log(i), 1000);
>   }
>   {
>     var i = 3; //每次赋值都会覆盖上面的，因此最终是向延迟队列中放入3个3
>     setTimeout(() => console.log(i), 1000);
>   }
> }
> ```
>
> ```js
> {
>   let i = 0;
>   {
>     let i = 1;
>     setTimeout(() => console.log(i), 1000);
>   }
>   {
>     let i = 2;
>     setTimeout(() => console.log(i), 1000);
>   }
>   {
>     let i = 3; //每个i不一样
>     setTimeout(() => console.log(i), 1000);
>   }
> }
> ```

#### 作用域链

先看这样一段代码:

```js
function bar() {
  console.log(myName);
}
function foo() {
  var myName = "BBB";
  bar();
}
var myName = "AAA";
foo();
```

这个代码最终打印的是 AAA 而不是 BBB
作用域链实际上就是上面所说的词法环境中的 outer 连续指向形成的链。在上面这段代码中，`bar` 和 `foo` 的 `outer` 都指向全局，所以没有的变量去全局查找，因此最终 `bar` 中的 `myName` 应该是 AAA
因此作用域链，或者说一个函数的 `outer` **只和定义时的位置相关**，跟怎么调用无关
如果是这样，就可以打印 BBB 了，因为`myName`是`bar`函数的一个闭包

```js
function foo() {
  var myName = "BBB";
  function bar() {
    console.log(myName);
  }
  bar();
}
var myName = "AAA";
foo();
```

![](https://pic.imgdb.cn/item/61b1d8f42ab3f51d91dda91b.png)
所以可以总结相关几点:

- 作用域链通过执行上下文中词法环境的 outer 属性设置
- 通常一个执行上下文的 outer 指向它声明的时候的直接外部执行上下文
- 一个函数的 outer 只和定义时的位置相关，跟怎么调用无关

### 作用域和执行上下文

JavaScript 属于解释型语言，JavaScript 的执行分为解释和执行两个阶段,这两个阶段所做的事并不一样：

**解释阶段**：

- 词法分析
- 语法分析
- 作用域规则确定

**执行阶段**：

- 创建执行上下文，并创建词法环境、变量环境、变量初始化为 undefined，函数确定但不执行
- 执行函数代码
- 垃圾回收

JavaScript 解释阶段便会确定作用域规则，因此**作用域在函数定义时就已经确定了**，而不是在函数调用时确定。但是**执行上下文是解释之后、执行之前创建的**。执行上下文最明显的就是 this 的指向是执行时确定的。而作用域访问的变量是编写代码的结构确定的。

作用域和执行上下文之间最大的区别是：
**执行上下文在运行时确定，随时可能改变；作用域在定义时就确定，并且不会改变。**这也就是上面说的，一个作用域的 outer 只和定义时的位置相关，和调用的位置无关。

一个作用域下可能包含若干个上下文环境。有可能从来没有过上下文环境（函数从来就没有被调用过）；有可能有过，现在函数被调用完毕后，上下文环境被销毁了；有可能同时存在一个或多个（闭包）。同一个作用域下，不同的调用会产生不同的执行上下文环境，继而产生不同的变量的值。

## 闭包

根据上面作用域链的相关概念,可以更清楚了解闭包:
先看一段代码:

```js
function foo() {
  var myName = " AAA ";
  let test1 = 1;
  const test2 = 2;
  var innerBar = {
    getName() {
      console.log(test1);
      return myName;
    },
    setName(newName) {
      myName = newName;
    },
  };
  return innerBar;
}
var bar = foo();
bar.setName(" BBB ");
bar.getName();
console.log(bar.getName());
```

1. 首先调用执行 foo 函数, 这个过程中会创建全局执行上下文和 foo 函数的执行上下文;在 foo 函数的执行上下文中有 myName,test1,test2,innerBar 这些变量的词法环境
2. 这时候函数继续执行到 return 为止, 赋值给 bar;bar 调用 setName,这时候虽然 foo 已经返回,但是因为 setName 函数中引用了 foo 函数的 myName 变量,因此就会导致 foo 函数的执行上下文出栈,但还留着一个"小包",里边就是 myName 和 test1 变量.
3. bar 函数在全局作用域下调用,这时候这个小包直接在全局上下文上,全局执行上下文走到哪都会带着这个"背包",我们把这个背包叫**闭包**
   ![](https://pic.imgdb.cn/item/61b1deab2ab3f51d91e09e76.png)

所以接下来就可以给闭包一个正式定义:
**在 JavaScript 中，根据词法作用域的规则，内部函数总是可以访问其外部函数中声明的变量，当通过调用一个外部函数返回一个内部函数后，即使该外部函数已经执行结束了，但是内部函数引用外部函数的变量依然保存在内存中，我们就把这些变量的集合称为闭包。**

这句话有几个关键点:

- 词法作用域的规则:就是我们上面一直讲的作用域链;一个变量查找不到就会到其上级的词法环境查找
- 外部函数已经执行结束:这也是 js 闭包的特点,在函数 return 之后依然可以使用这个函数里边的值
- 这些变量的集合称为闭包:闭包实际上是个变量的集合

这里还有一个例子:

```js
var bar = {
  myName: "CCC",
  printName: function () {
    console.log(myName);
  },
};
function foo() {
  let myName = " BBB ";
  return bar.printName;
}
let myName = " AAA ";
let _printName = foo();
_printName();
bar.printName();
```

结果是两次都打印 AAA。
解析：

1. 对象和其他类型的数据一样，只是一种映射关系而已，并不会单独创建一个上下文；访问对象的某个属性或方法，就是确定的那个属性或方法，和对象中的其他任何都没有关系（除非使用 this）
   所以上面这段代码调用`foo()`时相当于拿到了 bar 对象中的`printName`方法的索引（还没有编译和调用）；然后 foo 函数返回，即使`printName`内部有一个同名变量，但它还没有执行，自然不会有任何 foo 函数的闭包在它身上。
2. 调用`_printName()`时才为`printName`函数创建一个上下文；其中的 myName 变量按照 `词法环境 -> 变量环境 -> outer -> 全局变量环境 -> 全局词法环境`的顺序搜索,找到了全局下的 myName 并打印。因此，没有涉及到 bar 中的任何内容，最终结果只能是全局的。

从函数“在哪定义在哪确定 outer 关系”的角度说，`printName`只在全局环境下调用了一次，因此它的 outer 只指向全局上下文。如果在 foo 函数内部就已经调用，那么结果就会是`BBB`。

---

- 闭包就是能够读取其他函数内部变量的函数
- **闭包是指有权访问另一个函数作用域中变量的函数**，创建闭包的最常见的方式就是在一个函数内创建另一个函数，通过另一个函数访问这个函数的局部变量,利用闭包可以突破作用链域

### 闭包和内存泄漏

不再用到的内存，没有及时释放，就叫做内存泄漏（memory leak）。 内存泄漏是指程序执行时，一些变量没有及时释放，一直占用着内存 而这种占用内存的行为就叫做内存泄漏。

js 中常见的内存泄漏有三种情况：

1. 全局变量；由于非严格模式下全局变量会被挂载到 window 上，因此全局变量一直不使用但不清除就会造成这种问题。另外在非严格模式下未声明直接使用和 this 调用也会出现这种问题

```js
function bar() {
  say = "hehe";
}
bar();

function foo() {
  this.name = "hehe";
}
foo();
```

2. 定时器；当一个节点被删除或者被销毁时，其上挂载的定时器没有被清除，可能会导致定时器仍在运作

比如：

```js
const element = document.getElementById("element");

setInterval(() => {
  console.log(element);
}, 1000);
```

这里的定时器内部的回调函数就会一直引用这个变量，从而导致了内存泄漏的问题。解决方法就是清除定时器，或者把 element 的声明移到定时器内部。

3. 引用但已经被销毁的 dom 元素；即引用一个 dom，但这个 dom 被从 dom 树中移除了：

```js
const refA = document.getElementById("refA");
document.body.removeChild(refA); // dom删除了
console.log(refA, "refA"); // 但是还存在引用能console出整个div 没有被回收
refA = null;
console.log(refA, "refA"); // 解除引用
```

4. 闭包；即一个函数引用的数据成为一个闭包后，后面如果再次调用可能还会生成一个新闭包，这样累计就会造成极大的问题：

比如下面这段代码，每秒钟都会创建一个新的`Array(1000000)`，并用于`innerFunc`函数的闭包一直被保留，只要这个函数还在被引用，产生的闭包就不会被回收。

```js
const bigArray = null;
const scope = () => {
  const temp = bigArray;
  bigArray = new Array(100000);
  return function innerFunc() {
    if (temp) console.log("hello,world");
  };
};

setInterval(scope(), 1000);
```

如果不是某些特定任务需要使用闭包，尽量避免在函数中创建函数，因为闭包在处理速度和内存消耗方面对脚本性能具有负面影响。
比如构造函数的方法应该挂载到构造函数的`prototype`上，而不是构造函数内部。ES6 的类就是这样做的。

```js
// 函数中的两个方法会引起闭包，每一次构造都会导致方法被改变（重新赋值）
function MyObject(name, message) {
  this.name = name.toString();
  this.message = message.toString();
  this.getName = function () {
    return this.name;
  };

  this.getMessage = function () {
    return this.message;
  };
}

// 应该是
function MyObject(name, message) {
  this.name = name.toString();
  this.message = message.toString();
}
MyObject.prototype.getName = function () {
  return this.name;
};
MyObject.prototype.getMessage = function () {
  return this.message;
};
```

#### 解决内存泄漏问题

对应上述的内存泄漏场景，可以对症下药解决内存泄漏问题：

1. 不要设置全局变量，设置了也应该及时清除
2. 定时器引用的变量最好定义在内部，及时销毁定时器
3. 闭包产生的泄露问题：
4. 减少使用闭包
5. 如果是嵌套函数闭包：
   - 在闭包内部，当闭包执行完成后将引用的变量置空
   - 当闭包被使用完毕时，应该清除闭包的引用。比如：
   ```js
   const cacheFn = (fn) => {
     const cache = new Map();
     return function (...args) {
       console.log(cache);
       //...
     };
   };
   let cachedAdd = cacheFn(add);
   // ...调用cachedAdd
   cacheAdd = null; // 清除闭包
   ```
6. 如果是回调函数闭包，比如 addEventListener，就要及时清除监听器。如果是自己编写的回调函数，则可以通过置空的方式。

### 闭包的出现场景

1. 嵌套函数
   最基本的出现场景，不再赘述

2. 回调函数；在定时器、事件监听、Ajax 请求、跨窗口通信、Web Workers 或者任何异步中，只要使用了回调函数，实际上就是在使用闭包。

比如下面这个例子中，callback 函数就是一个闭包，因为他引用了 getUserInput 函数内部的 input 变量

```js
function getUserInput(callback) {
  const input = document.getElementById("input").value;
  callback(input);
}

getUserInput((value) => {
  console.log(value);
});
```

其他类型的回调函数也是同理，比如 addEventListener 的回调函数：

```js
domElement.addEventListener("click", (e) => {
  //...
});
```

3. IIFE 立即执行函数

```js
(function () {
  var name = "Mozilla";
  function displayName() {
    alert(name);
  }
})();
```

### 闭包的使用场景

闭包的作用主要作用体现在两个方面：保存和保护。

保存：保存闭包引用的外部变量。通过闭包可以实现很多高阶函数。

保护：保护私有变量；即函数内部定义一个变量，然后通过闭包访问并返回，但并不能直接修改该变量。

```js
function private() {
  let times = 0;
  return function () {
    // 一些操作
    return times;
  };
}
times; //未定义
const timesVal = private()();
```

# 高阶函数

接受一个函数，并返回包含新功能的函数被称为装饰器。装饰器可以给一个基本功能的函数添加一些“装饰”，但是不在函数本身做改变，而是在装饰器函数中做一些工作。
比如一个简单的函数：

```js
function hello(user) {
  console.log(`hello ${user}!`);
}
```

可以外包一个装饰器，让这个函数可以记录每次 hello 的参数

```js
function helloMemory(f, arr) {
  return function () {
    arr.push(...arguments);
    f.apply(this, arguments);
  };
}

let users = [];
hellos = helloMemory(hello, users);
hellos("xiaoming");
hellos("xiaohong");
```

装饰器利用了函数的闭包以及函数作为对象时的 arguments 属性。由于 js 中函数可以像变量一样任意传递，因此提供了极大的灵活性。

## 方法借用

当被装饰的函数参数>1 时，可以考虑使用`arguments`来取到所有的参数；
但是实际上，arguments 对象既是可迭代对象又是类数组对象，并不是真正的数组，因此如果对 arguments 调用 join、push 等方法都会出错。
因此我们可以从数组对象中“借用”这些方法，然后用 call 或 apply 的方式调用到 arguments 上。

```js
function hash() {
  return [].join.call(arguments);
  //或
  return Array.prototype.join.call(arguments);
  //相当于 arguments.join()
}

hash(1, 2, 3);
```

类似的方法还有很多，js 中许多内置方法并不是针对完全的数组或字符串，实际上只要相似的可迭代对象或`ArrayLike`就可以应用。
数组的大多数方法都可以借用。

## apply 和 call 的使用

注意到装饰器中调用作为参数的函数都是通过`call`或者`apply`的方式，主要原因是保证 this 指向的正确性。
在一个正常函数中不会有影响，但是如果函数是某个对象内部的一个方法，如果直接调用而不是用`.`访问符调用（在装饰器函数内部），this 可能会缺失指向，这时候需要手动指定 this

```js
let worker = {
  amount: 5,
  slow(x) {
    return x * this.name;
  },
};

function cachingDecorator(func) {
  let cache = new Map();
  return function (x) {
    if (cache.has(x)) {
      return cache.get(x);
    }
    let result = func(x); //这个地方直接调用worker中的slow方法，前面没有点访问符，this没有指向
    cache.set(x, result);
    return result;
  };
}

worker.slow(1); // 原始方法有效

worker.slow = cachingDecorator(worker.slow);
worker.slow(2); //由于上面的原因，执行中this为undefined，会报错
```

所以装饰器内部调用原函数一定要使用 call 或 apply。
这里 apply 参数的 this 就是后续`*`行的 worker

> `worker.slow` 现在是包装器 `function (x) { ... }`，这是一个普通函数，它的 this 来自外部点访问符指明的对象（即`worker`）

```js
function cachingDecorator(func) {
  let cache = new Map();
  return function (x) {
    if (cache.has(x)) {
      return cache.get(x);
    }
    let result = func.apply(this, arguments);
    cache.set(x, result);
    return result;
  };
}

worker.slow = cachingDecorator(worker.slow);
worker.slow(2); //（*）
```

## 常见的装饰器

### 延时

```js
function delay(f, ms) {
  return function () {
    setTimeout(() => {
      f.apply(this, arguments);
    }, ms);
  };
}
```

### 防抖和节流

参考下面的防抖和节流

### 缓存

```js
function caching(f) {
  const cache = new Map();
  return function (arg) {
    if (cache.has(arg)) return cache.get(arg);
    let res = f.call(this, arg);
    cache.set(arg, res);
    return res;
  };
}
```

# 原型链

原型链参考图：

```js
function Foo() {}

let f1 = new Foo();
let f2 = new Foo();
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a61ca07672a45d3aecf382100cc9719~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

分析如下：

每个对象的`__proto__`都是指向它的构造函数的原型对象 prototype 的

```js
person1.__proto__ === Person.prototype;
```

构造函数是一个函数对象，是通过 Function 构造器产生的。
实际上任何一个函数的`__proto__`都指向`Function.prototype`对象，包括`Object()`和`Function()`

```js
Person.__proto__ === Function.prototype;
```

原型对象本身是一个普通对象，而普通对象的构造函数都是 Object

```js
Person.prototype.__proto__ === Object.prototype;
```

刚刚上面说了，所有的构造器都是函数对象，函数对象都是 Function 构造产生的

```js
Object.__proto__ === Function.prototype;
```

Object 的原型对象也有`__proto__`属性指向 null，null 是原型链的顶端

```js
Object.prototype.__proto__ === null;
```

- 一切对象都是继承自`Object.prototype`对象，`Object.prototype` 对象直接继承根源对象 `null`
- 一切的函数对象（包括 `Object` 对象），都是继承自 `Function` 对象
- `Object` 对象直接继承自 `Function` 对象
- `Function`对象的`__proto__`会指向自己的原型对象，最终还是继承自`Object`对象

## `[[Prototype]]`和`__proto__`

> JavaScript 中，对象有一个特殊的隐藏**属性** ：`[[Prototype]]`。它要么为 null，要么就是对另一个对象的引用。该对象被称为“原型”
> 当我们从 object 中读取一个缺失的属性时，JavaScript 会自动从原型中获取该属性。在编程中，这种行为被称为“原型继承”

`[[Prototype]]`和`__proto__`:

- `[[Prototype]]`不等于`__proto__`;
- `__proto__`是访问`[[Prototype]]`的一种方法, 也就是`[[Prototype]]`的 getter 和 setter
- `__proto__`是一种"旧时的访问方法",应该用一些新方法代替

## 对象原型

### 原型和继承

一个对象内部可以设置其`__proto__`,或者在外部访问来设置这个对象的原型

```js
const animal = {
  eats: true,
  walk() {
    console.log(`i can walk`);
  },
};
const rabbit = {
  jump: true,
  __proto__: animal, //__proto__是[[Prototype]]的getter/setter,真正访问的是[[Prototype]]
};
rabbit.walk(); //i can walk
```

这里我们就说 rabbit 继承了 animal,或者说 animal 是 rabbit 的原型
![](https://pic.imgdb.cn/item/61b0c20c2ab3f51d915b97ba.png)
如果再继承一个对象,就会像上面两个一样形成原型关系
继承的属性有几个特点:

- 继承来的会被当做是"自己的", 也就是说用 forin 可以遍历到（但`Object.keys` 只返回自己的 key）。如果想分辨哪些是自己本身的属性, 可以使用 `hasOwnProperty`

- 原型仅用于读取属性；对于写入/删除操作直接在对象上进行，并不会影响其原型；但是 getter 和 setter 例外。因为写一个 setter 相当于是调用原型上的 setter 函数，而不是修改一般属性。
- 如果父类有 getter/setter, 子类会继承并维护自己独立的 getter 值,即触发子类的 setter 不会影响父类

```js
let user = {
  name: "John",
  surname: "Smith",

  set fullName(value) {
    [this.name, this.surname] = value.split(" ");
  },

  get fullName() {
    return `${this.name} ${this.surname}`;
  },
};

let admin = {
  __proto__: user,
  isAdmin: true,
};

alert(admin.fullName); // John Smith

// setter triggers!
admin.fullName = "Alice Cooper";

alert(admin.fullName); // Alice Cooper，admin 的内容被修改了
alert(user.fullName); // John Smith，user 的内容被保护了
```

### 原型链的限制：

1. 引用不能形成闭环，即原型链不能开头指向结尾
2. `__proto__`值只能是对象或 null
3. 一个对象不能从其他两个对象获得继承，一个对象只有一个原型。

### 原型和 this

另外，原型链上的对象中的 this 根本不受原型的影响。this 始终是`.`前面的对象。

```js
let animal = {
  walk() {
    if (!this.isSleeping) {
      alert(`I wake`);
    }
  },
  sleep() {
    this.isSleeping = true;
  },
};

let rabbit = {
  name: "White Rabbit",
  __proto__: animal,
};

// 修改 rabbit.isSleeping
rabbit.sleep();

alert(rabbit.isSleeping); // true，说明sleep方法中的this指的是rabbit而不是animal
alert(animal.isSleeping); // undefined（animal中没有此属性）
```

## 函数属性 prototype

prototype 是**函数**的一个固有属性，通过 `function.prototype` 读写；它被理解为一个名为 "prototype" 的常规属性，但是它表达的意思是函数的原型。

### prototype 在函数上的表现

> 如果函数是一个构造函数，那么他构造出的实例的**proto**属性将会指向该函数的 prototype 属性。注意实例并不是直接指向该构造函数的。

把**构造函数**的 prototype 指向某个原型, 则其 new 出来的实例的`[[Prototype]]`也会指向该对象.
`F.prototype` 属性**仅**在 `new F` 被调用时使用，它为新对象的 `[[Prototype]]` 赋值

比如:

```js
const animal = {
  eat: true,
};
function Rabbit(name) {
  this.name = name;
}
Rabbit.prototype = animal;
//不是把函数挂到animal,而是把new的实例的[[Prototype]]指向animal
//效果就是 rabbit.__proto__ === Rabbit.prototype
```

![](https://pic.imgdb.cn/item/622620495baa1a80ab998667.jpg)

> 每个函数都有 "prototype" 属性

一般函数的 prototype 是一个只有属性 constructor 的对象，属性 constructor 指向函数自身。

```js
function Rabbit() {}
Rabbit.prototype === { constructor: Rabbit };
```

![](https://pic.imgdb.cn/item/622625515baa1a80ab9cbc98.jpg)

但是 constructor 有时候会丢失指向
如果替换了函数的 prototype ,就会丢失指向,所以需要重定向 constructor.这个在组合继承中很重要

```js
function Rabbit() {}
Rabbit.prototype = {
  jumps: true,
};

let rabbit = new Rabbit();
alert(rabbit.constructor === Rabbit); // false
```

---

扩展：

1. 如果创建一个构造函数的实例，在它之后修改这个函数的 prototype 属性不会影响当前实例，但是会影响之后的实例

```js
function Rabbit() {}
Rabbit.prototype = {
  eats: true,
};

let rabbit = new Rabbit();
Rabbit.prototype = {};
clg(rabbit.eats); // true

let rabbit2 = new Rabbit();
clg(rabbit.eats); //undefined
```

2. 由于每个函数的 prototype 上都有`constructor`属性，它们的实例也可以访问这个属性，因此可以通过实例上调用`constructor`访问到自己的构造函数。
   这也是可以通过 constructor 属性判断变量类型的原理。
   但是这种方法不可取，因为 constructor 可能随着 prototype 的改变直接丢失或者改变，即它是不可信的。

```js
const obj = new SomeObject();

let obj2 = new obj.constructor();
```

## 用来代替`__proto__`的函数

- `Object.create()`：参数是一个用作设置为新对象的原型的对象，还可以有一个额外参数用于给返回对象添加属性；返回一个原型指向参数的子对象
- `getPrototypeOf()`/`setPrototypeOf()`：参数都是对象，用于获取/设置对象的原型

```js
const animal = {
  eats: true,
  walk() {
    console.log(`i can walk`);
  },
};
let snake = Object.create(animal, {
  slide: true,
}); //创建以animal为原型的对象,第二个参数是这个对象的额外属性
snake.walk();

//...

console.log(Object.getPrototypeOf(snake)); //获取原型
let bear = {};
Object.setPrototypeOf(bear, animal); //设置原型
bear.walk();
```

# js 五种继承方式

这里的继承是指一个构造函数继承另一个构造函数，使得 new 出来的实例拥有父构造函数的方法。
简单来说就是要达到这个目的：

```js
function Parent(name) {
  this.name = name;
  this.age = 18;
}
function Child(grade) {
  this.grade = grade;
}

//...操作

const child = new Child("aaa");
child.age; //18
```

由构造函数的`prototype`属性原理可知，如果我们把构造函数的`prototype`指向另一个对象，就可以继承这个对象

```js
function Animal(eats) {
  this.eats = eats;
}
const animal = new Animal(true);
/* 上面这段相当于生成了下面这个对象：
let animal = {
  eats: true
};
*/
function Rabbit(name) {
  this.name = name;
}
Rabbit.prototype = animal;

let rabbit = new Rabbit("White Rabbit"); //  rabbit.__proto__ == animal
rabbit.eats; // true
```

这就是一切的基础（其实就是第一种原型链继承），后面几个方法都是对此的改进。

## 1. 原型链继承

原理是通过原型链把子构造函数的原型(f.prototype)指向父构造函数的实例, 这样子构造函数创建的实例就有父类的方法。
原理参考：https://zh.javascript.info/function-prototype 的第一个例子
即改变构造函数的`prototype`相当于改变了其构造出来的实例的`__proto__`；如果我们把构造函数的`prototype`指向另一个构造函数的实例（实际上就是个拥有父构造函数属性的对象而已），就相当于子构造函数继承了父构造函数。

几个注意点:

- 不同的子实例共享父类属性, 比如下面的 stu1 更改 play 数组,stu2 的数组也会改变
- 子类可以调用父类的原型上的方法, 换句话说就是可以多层继承

缺点：

- 多个子实例共享一个父实例，并不是单独的
- 创建子类实例时，无法向父类构造函数传参
- 子类添加方法，必须要在`Student.prototype = new Person() `之后执行（因为这句会覆盖前面添加方法），不能放到构造器中

```js
//父
function Person(name) {
  this.name = name;
  this.play = [1, 2, 3];
}
Person.prototype.setAge = function () {
  console.log("i am 20 years old");
};
//子
function Student(grade) {
  this.grade = grade;
}
Student.prototype = new Person("zzx"); //子类的原型指向父类实例
let stu1 = new Student(100); //stu1上有Person的方法
let stu2 = new Student(90);
//父类Person上有一个引用数据类型(数组)play, 如果更改stu1继承的数组会导致stu2的也改变
//就说明不同实例实际上共同共享了父类(原型)的属性,对于引用对象来说地址也是相同的
stu1.play.push(4);
stu1.setAge(); //子类可以调用父类的原型的方法
console.log(stu2.play);
```

## 2. 构造函数继承

原理是在子构造函数中用 call 调用父构造函数 ,使得父的实例被转移到子构造函数中

- `Person.call(this, name);` 相当于`this.Person(name)`，显然这里的 this 是 Student 即将指向实例的，相当于把 Person 中的构造部分移到了 Student 中；但是由于只是把 person 构造了一遍，只拥有了 Person 内部的构造效果，本质上并没有继承 Person 原型。

缺点：

- 只能继承父类的实例属性和方法，不能继承原型属性和方法

```js
function Person(name) {
  this.name = name;
  this.play = [1, 2, 3];
}
Person.prototype.setAge = function () {
  //父类原型的方法
  console.log("i am 20 years old");
};
function Student(grade, name) {
  Person.call(this, name); //相当于this.Person(name)
  this.grade = grade;
}
let stu3 = new Student(100, "xiaohong");
//由于调用了call,Person的this指向stu3,也就相当于stu3.Person('xiaohong')
//又因为Person本身是个构造函数,这里就相当于stu3 = new Person('xiaohong'),加上外面的new,等于stu3同时有父类的属性
console.log(stu3);
```

## 3. 组合继承

就是上面两种方法的结合；由于改变了子类原型, 需要修复构造器指向。
相比于上面两种方法，组合继承的特点是：

- 可以向父构造函数传参，这点源于构造函数继承方法
- 子类可以继承完整的父类，这点源于基本的原型链继承方法
- 因为每个实例都相当于单独调用了一次父构造函数，所以不同实例之间也不会共享父元素数据，不会冲突。

缺点：

- 调用了两次父类构造函数，生成了两份实例（下一个的优化就是只调用一次构造函数）

```js
function Person(name) {
  this.name = name;
  this.play = [1, 2, 3];
}
Person.prototype.setAge = function () {
  //父类原型的方法
  console.log("i am 20 years old");
};

function Student(grade, name) {
  Person.call(this, name);
  this.grade = grade;
}

/*相比于构造函数继承加上了原型链指向和修复构造器,这样就可以访问到父类的原型上的方法setAge*/

Student.prototype = new Person(); //这里有没有参数都可以，最终参数来自于Student构造函数的调用
//修复构造器指向,因为正常情况下构造器应该是指向自己的,这里设置了student的原型需要修复回来,不然就会指向Person
Student.prototype.constructor = Student;

let stu4 = new Student(120, "jack");
let stu5 = new Student(80, "mike");
```

---

既然为了避免调用两次父构造函数，可以让子构造函数的 prototype 和父构造函数的 prototype 相等，比如这样：

```js
//...
Student.prototype = Person.prototype;
//...
```

但是显然这不是个最好的方案，因为这样实际上两者的实例就没有区别了，new 的时候实例的`__proto__`都指向相同，无法分辨

## 4. 组合寄生继承(构造函数中最好的继承方法)

用 `Object.create` 代替`new Person() `，解决了调用两次父类的问题。

```js
function Person(name) {
  this.name = name;
  this.play = [1, 2, 3];
}
Person.prototype.setAge = function () {
  //父类原型的方法
  console.log("i am 20 years old");
};
function Student(grade, name) {
  Person.call(this, name);
  this.grade = grade;
}
//Object.create返回一个以参数为原型的新对象,相当于 stu.__proto__ = Person.prototype
Student.prototype = Object.create(Person.prototype);
//恢复constructor指向
Student.prototype.constructor = Student;
//所以new出来的实例,__proto__会指向Person.prototype,继而相当于完整的原型链
let stu6 = new Student(120, "jack");
let stu7 = new Student(80, "mike");
console.log(stu6, stu7);
```

## 5. 类继承

利用 es6 类的特性完成比较完整的继承

- 构造函数中向构造函数原型上添加方法的方式相当于在类中直接声明
- 子类需要 constructor 使用 super 方法实现父类的构造,也就是说子类 new 的时候如果也想传入姓名, 就需要 super.(具体在类中解释)

```js
class Person {
  constructor(name) {
    this.name = name;
    this.play = [1, 2, 3];
  }
  //注意在构造函数中向构造函数原型上添加方法的方式相当于在类中直接声明
  setAge() {
    console.log("i am 20 years old");
  }
}
class Student extends Person {
  constructor(grade, name) {
    super(name); //super(...)相当于 Person.call(this,...arguments)
    this.grade = grade;
  }
}
let stu8 = new Student(150, "zzx");
console.log(stu8);
```

从这里其实也能看出来 js 中类实际上就是构造函数。

# 类

关于 js 基本类的操作和写法就不再赘述了,这里讲一些之前没用到的知识点

## 类、构造函数、this

js 中的类其实就是一种构造函数。
下面这两段代码效果是相同的：

```js
class Person {
  constructor(age) {
    this.age = age;
    this.name = "zzx";
  }

  getName() {
    console.log(this.name, this.age);
  }
}

function Person(age) {
  this.age = age;
  this.name = "zzx";
}
Person.prototype = {
  getName() {
    console.log(this.name, this.age);
  },
};
```

### 1. 类和类字段

如果直接写**类字段**，则是类字段方法/属性。类字段方法/属性会被直接放在类构造出的对象本身，而不是类的`prototype`上。
我们把前两种函数定义方法也可以称为**类字段**，第三个函数才是类真正的方法，安插在构造函数的`prototype`上

```js
class Person {
  constructor(age) {
    this.age = age;
  }
  name = "zzx";
  getName = () => {
    console.log(this.name);
  };
  getAge = function () {
    console.log(this.age);
  };
  getAll() {
    console.log(this.name, this.age);
  }
}
const person = new Person(19);
person.name; //zzx
Person.prototype.name; //undefined
person.getName(); //zzx
```

![](https://pic.imgdb.cn/item/622883965baa1a80ab3295ae.jpg)

### 2. 类和 this

关于类中方法的 this，正常使用指向应该是**实例**，不管是哪种方式定义的函数

```js
class Person {
  constructor(age) {
    this.age = age;
  }
  name = "zzx";
  getName() {
    console.log(this.name, this.age); //这里的this是下面的person
  }
}

const person = new Person(18);
console.log(person);
person.getName() === person.__proto__.getName();
```

但是类内部按照类字段定义的**箭头函数**，会携带类的闭包，相当于绑定了类作为 this。
比如下面这段箭头函数输出不会丢失 this：

```js
class Animal {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  sayName = function () {
    console.log(this.name);
  };

  sayAge = () => {
    console.log(this.age);
  };
}

let tom = new Animal("tom", 19);
let [sayName, sayAge] = [tom.sayName, tom.sayAge];
sayAge(); // 正常输出19
sayName(); // this为undefined
```

原理是这样：

```js
var Animal = function Animal(name, age) {
  var _this = this; //_this会作为闭包

  _classCallCheck(this, Animal);

  _defineProperty(this, "sayName", function () {
    console.log(this.name);
  });

  _defineProperty(this, "sayAge", function () {
    console.log(_this.age);
  });

  this.name = name;
  this.age = age;
};
```

注意两个`_defineProperty` 的回调函数内对于 this 的使用方式。一个是直接用了 this，一个用了构造实例的闭包，所以一个 this 会丢失（成为 undefined），而另一个不会。
所以：**类中的箭头函数会携带类的闭包，从而对外展现出不会丢失 this 指向的效果。**
详细可以参看这个 issue：https://github.com/dwqs/blog/issues/67

---

所以这也是在 React 的类组件中，方法要么写成箭头函数，要么明确绑定 this 的原因。
如果我们把上面代码的`sayName`bind 一下，同样也不会丢失 this 了（绑定的 this 显然应当是该类的实例，或者说是类本身）：

```js
class Animal {
  constructor(name, age) {
    this.name = name;
    this.age = age;
    this.getAge = this.getAge.bind(this);
  }

  sayName = function () {
    console.log(this.name);
  };

  sayAge = () => {
    console.log(this.age);
  };
}
```

## 类继承

类使用 extends 关键词继承。
类的继承相当于原型链的延伸，extends 执行的动作就是将子类的`prototype`和父类的`prototype`用`[[Prototype]]`连接起来：
![](https://pic.imgdb.cn/item/6228c7da5baa1a80ab8ab1b2.jpg)

类继承和 ES5 继承方式的区别：

子类可以直接通过 `__proto__`寻址到父类。

```js
function Super() {}
function Sub() {}

Sub.prototype = new Super();
Sub.prototype.constructor = Sub;

var sub = new Sub();

Sub.__proto__ === Function.prototype;
```

而通过 ES5 的方式，`Sub.__proto__ === Function.prototype`

### super 的使用

从前面的继承可以看出来，`super(...)`实际上就是`ParentClass.call(this,...)`，而 super 对象本身就相当于`Parent.prototype`

- 直接用`super.`可以在子类 `constructor` 外部使用父类的方法
- 在 `constructor` 使用 `super` 接受参数用于重写 `constructor`
- 在 `constructor` 使用父类属性需要`super.`,不然不能访问到
- **继承类的 `constructor` 必须调用` super(...)`，并且一定要在使用 `this` 之前调用。**原因是继承的 `constructor` 执行时，它不会执行此操作。它期望父类的 `constructor` 来完成这项工作。
- **如果继承类没有`constructor`，相当于只有 `super(...args)` 的空构造器，会执行一次父类`constructor`**

```js
class Benz {
  constructor(name, maxSpeed) {
    this.name = name;
    this.maxSpeed = maxSpeed;
  }
  run = () => {
    console.log(`${this.name} run with speed ${this.maxSpeed}`);
  };
}
class Track extends Benz {
  constructor(type) {
    this.maxSpeed = super.maxSpeed - 100; //在constructor中使用super调用父类属性
    this.type = type;
  }
  stop = () => {
    super.stop(); //使用super调用父类函数
    console.log(super.maxSpeed);
  };
}

class AnotherTrack extends Benz {
  constructor(type, ...args) {
    super(...args); //如果要重写constructor构造的属性(比如子类实例也想设置name),就需要调用super函数
    this.type = type;
  }
}
const anotherTrack = new AnotherTrack("AnotherTrack", "myAnotherTrack"); //这里重写了父类的constructor构造的属性name和maxSpeed
```

### super 的实现原理

super 使用了类中函数的一个`[[HomeObject]]`属性(当然每个函数都有),
当一个函数被定义为类或者对象方法时，它的 `[[HomeObject]]` 属性就成为了该对象或类(也就是一种绑定)
因为这个原理, super 可以准确沿着原型链找到父类上的方法和属性

## instanceof

`a instanceof B`返回 a 是否是 B 的实例,返回一个 boolean 值

instanceof 会先检查类中的`[Symbol.hasInstance]`静态方法,如果有会根据方法返回

```js
class Fruit {
  static [Symbol.hasInstance](obj) {
    if (obj.type === "fruit") return true;
  }
}
let apple = {
  type: "fruit",
};
apple instanceof Fruit; //true
```

如果没有 `Symbol.hasInstance`，会使用` obj instanceOf Class` 检查 `Class.prototype` 是否等于 `obj` 的原型链中的原型之一。
也就是一种遍历的方式一个一个比较：

```js
obj.__proto__ === Class.prototype?
obj.__proto__.__proto__ === Class.prototype?
obj.__proto__.__proto__.__proto__ === Class.prototype?
...
//或者
Class.prototype.isPrototypeOf(obj)
Class.prototype.prototype.isPrototypeOf(obj)
//...
```

## 类字段

直接在类中不声明而赋值的属性或者方法被称为类字段。

```js
class User {
  name = "aaa";
  getName = () => {
    return this.name;
  };
}
```

类字段本身直接保存在实例中而不是构造函数的 prototype 上。

### 重写类字段

不仅可以重写方法，还可以重写类字段。
但是：**父类构造器总是会使用它自己字段的值，而不是被重写的那一个。**
举个栗子：

```js
class Animal {
  name = "animal";

  constructor() {
    alert(this.name);
  }
}

class Rabbit extends Animal {
  /*
    隐式调用一次：
    alert(this.name)
  */
  name = "rabbit";
}

new Animal(); // animal
new Rabbit(); // animal
```

> `new Rabbit()`会 alert aminal 是因为没有构造器的子类会隐式调用一次父类的构造器。

这个例子很能说明问题，按理来说第二个输出应该是 rabbit 才对。
原因如下：

- 对于基类（还未继承任何东西的那种），在构造函数调用前初始化。
- 对于派生类，在 `super()` 后立刻初始化。

所以，`new Rabbit()` 调用了 `super()`，因此它执行了父类构造器，并且（根据派生类规则）只有在此之后，它的类字段才被初始化。在父类构造器被执行的时候，**`Rabbit` 还没有自己的类字段**，所以只能使用父类中的类字段。

## 扩展内建类

之前有过向内建对象的构造函数的 prototype 上添加属性和方法的方式添加自己的属性。实际上类也可以，而且更简单：

```js
Array.prototype.isEmpty = function () {
  return this.length === 0;
};
let arr = [];
console.log(arr.isEmpty());

//上面和下面相同
class MyArray extends Array {
  isEmpty() {
    return this.length === 0;
  }
}

let arr = new MyArray(1, 2, 5, 10, 50);
console.log(arr.isEmpty());
```

原理肥肠简单，前面说过类的方法实际上是放在类的 prototype 上的，这里`MyArray`继承了 Array 的所有方法和属性，并给自己添加了一个方法。

# Promise

> Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。Promise 提供统一的 API，各种异步操作都可以用同样的方法进行处理。

## Promise 解决的问题

Promise 的出现主要是为了解决回调地狱问题，即一个回调需要上一个回调的结果才能执行，就会依次嵌套，最后导致代码向右延伸，称为回调地狱。

```js
ajax(url, () => {
  // 处理逻辑
  ajax(url1, () => {
    // 处理逻辑
    ajax(url2, () => {
      // 处理逻辑
    });
  });
});
```

Promise 采用以下三个方式解决这个问题：

1. Promise 实现了回调函数的**延时绑定**。回调函数的延时绑定在代码上体现就是先创建 Promise ，通过 Promise 的构造函数 executor 来执行业务逻辑；创建好 Promise 对象 ，再使用 `.then` 来设置回调函数。也就是说相对于直接绑定回调，Promise 把主要执行逻辑和回调用 executor 和 then 分离了。
2. 回调函数 `resolve` 的**返回值穿透**到最外层，也就是说 executor 内部执行 resolve 可以在外部访问；而一个 then 又可以继续返回一个 Promise，因此实现垂直的链式调用
   ![](http://blog.poetries.top/img-repo/2019/11/45.png)
3. **错误冒泡**，即 Promise 中任何一个地方的冒泡都会向下，直到找到第一个 catch 或 reject 为止。

## 特点

Promise 对象有以下两个特点。

- Promise 对象代表一个异步操作，有三种状态：`pending`（进行中）、`fulfilled`（已成功）和 `rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。
- 一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise 对象的状态改变，只有两种可能：从 pending 变为 fulfilled 和从 pending 变为 rejected。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为 resolved（已定型）。
- 错误处理：Promise的错误不能被trycatch捕获，包括reject的错误、手动throw的错误。也不能被window.onerror捕获。唯一可以用trycatch捕获的情况是async函数内的await语句之后的promise。

## 基本用法

通过 new Promise()生成 Promise 实例,参数是一个 executor 函数(执行函数)
特点:

- executor 函数被创建会**立即执行**(不是一个异步任务)
- executor 函数中只能调用一个 resolve 或 reject(条件判断算一个),多个的后面会被无视
  - resolve:将 **Promise 对象的状态**从“未完成”变为“成功”（即从 pending 变为 resolved），在异步操作成功时调用，并将**异步操作的结果**，作为**参数**传递出去；
  - reject:将 Promise 对象的状态从“未完成”变为“失败”（即从 pending 变为 rejected），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

这两个方法都是向外传递一个变量,后续的 then 需要参数来接受

- 使用 resolve 之后,就不应该再在 promise 中操作了,剩下的操作应该在 then 中完成
- then 函数有**两个参数**,第二个函数将在 promise rejected 后运行并接收 error。

```js
promise.then(
  function(result){}
  function(error){}
)
```

- 还有一个 `finally` ,和 `trycatch` 中的 `finally` 类似
  - `.finally(f)` 调用与 `.then(f, f)` 类似，在某种意义上，`f` 总是在 `promise` 被 `settled` 时运行：即 `promise` 被 `resolve` 或 `reject`。
  - `finally` 没有参数。在 `finally` 中不知道 `promise` 是否成功，但无论是否成功都会向下传递
  - `finally` 处理程序将结果和 `error` 传递给下一个处理程序。

```js
let promise = new Promise((resolve, reject) => {
  setTimeout(() => resolve("done"), 2000);
})
  .finally(() => console.log("Promise ready"))
  .then((result) => {
    console.log(result);
  })
  .catch((error) => {
    console.log(error);
  });
```

通常使用都需要封装在一个函数中, 函数会返回一个 promise 对象
比如:

```js
function loadScript(src) {
  return new Promise(function (resolve, reject) {
    let script = document.createElement("script");
    script.src = src;

    script.onload = () => resolve(script);
    script.onerror = () => reject(new Error(`Script load error for ${src}`));

    document.head.append(script);
  });
}
loadScript("/src")
  .then((res) => {
    //...
  })
  .catch((err) => {
    console.err(err);
  });
```

## Promise 链

每个 then 还可以有一个返回值,会被下一个 then 接收到作为参数
举个例子,axios 的连续调用
当下一次请求需要上一次的返回值时,不需要写在上一次的回调中;可以返回请求 Promise 对象再进行一次 then。
实际上，每个对 `.then` 的调用都会返回了一个新的 promise，因此我们可以在其之上调用下一个 `.then`。

注意:

- 链必须始终是同一个 Promise 对象下的, 给多个 promise 分开 then 不是链，这种就不是：

```js
let promise = new Promise(function (resolve, reject) {
  setTimeout(() => resolve(1), 1000);
});

promise.then(function (result) {
  alert(result); // 1
  return result * 2;
});

promise.then(function (result) {
  alert(result); // 1
  return result * 2;
});
```

- 即使不刻意返回 Promise 对象,then 也会默认返回一个**新的**Promise 对象

```js
axios
  .get("/api")
  .then((res) => {
    //...
    return axios.get(`/api/${res}`);
  })
  .then((res) => {
    //...
    return axios.get(`/api/${res}`);
  })
  .catch((err) => console.log(err));
```

```js
new Promise(function (resolve, reject) {
  setTimeout(() => resolve(1), 1000);
})
  .then(function (result) {
    return result * 2;
  })
  .then(function (result) {
    return result * 2;
  })
  .then(function (result) {
    return result * 2;
  });
```

## Promise Api

### `Promise.all`

并行执行多个 promise，并等待所有 promise 都准备就绪后再执行一个 then
参数是一个数组或可迭代对象,每个元素都是 promise 对象,返回是一个每个 promise 结果组成的数组

> 如果任意一个 promise 被 reject，由 `Promise.all` 返回的 promise 就会立即 reject，并且带有的就是这个 error。

```js
const promises = [2, 3, 5, 7, 11, 13].map((id)=>{
  return fetch(`/.../${id}`);
});
Promise.all(promises)
  .then((posts) {
  // ...
}).catch((err){
  // ...
});
```

> 如何让 Promise.all 在抛出异常后依然有效？
> 其实原理很简单，就是给每个 Promise 单独设置 catch 即可；因为自己拥有自己的 catch 方法时，即使报错也会被捕获，并不会继续抛出错误让`promise.all`的 catch 去捕获从而导致 reject。

### `Promise.race`

与 `Promise.all` 类似，但只等待第一个 `settled` 的 `promise` 并获取其结果

```js
Promise.race([
  new Promise((resolve) => {
    setTimeout(() => resolve(1), 1000);
  }),
  new Promise((resolve) => {
    setTimeout(() => resolve(2), 2000);
  }),
  new Promise((resolve) => {
    setTimeout(() => resolve(3), 3000);
  }),
]).then((res) => console.log(res));
```

上面这段代码第一个 Promise 会首先`settled `，因此只返回第一个的结果，剩余不再返回。

如果有一个 promise 在其他 promise resolve 之前就 reject，则会只返回这个 reject 结果。
为了避免这种情况，有一个`Promise.any()` api，不会因为某个 Promise 变成 rejected 状态而结束，必须等到所有参数 Promise 变成 rejected 状态才会结束。

### `Promise.allSettled`

和 all 类似，但是会保证每个 promise 任务全都被处理完成，不管是 fulfilled 还是 rejected。
返回一个对象数组，数组的每一项包含对应序号的 promise 状态和结果

- 如果 resolve，返回 value 属性，值是 resolve 的值
- 如果 reject，返回 reason 属性，值的抛出的错误

```js
const promise1 = Promise.resolve(3);
const promise2 = new Promise((resolve, reject) =>
  setTimeout(reject, 100, "foo")
);
const promises = [promise1, promise2];

Promise.allSettled(promises).then((results) =>
  results.forEach((result) => console.log(result))
);

// { status: "fulfilled", value: 3 }
// { status: "rejected", reason: "foo" }
```

## 微任务

用于处理 Promise 程序的函数是一个微任务
也就是说, 每个`.then/.catch/.finally` 内的函数都是一个微任务

- 微任务队列会在上一个宏任务完成后,下一个宏任务开始前调用**微任务队列中的所有的微任务**完成(所以 promise2 会和 promise1 一起先后一起完成)

```js
setTimeout(() => console.log("timeout"));
Promise.resolve()
  .then((res) => {
    console.log("promise1");
  })
  .then((res) => {
    console.log("promise2");
  });
console.log("clg");
```

这一段代码会返回这样一个结果

```
clg
promise1
promise2
timeout
```

Promise 中微任务的主要体现在于上面说的“回调函数延时绑定”：
下面这段代码中，设置`this.then`就是绑定回调函数的过程。

- 首先有一个内部的`onFulFilled_`方法，即 then 内的回调函数，没有确定 then 之前为 null
- `onFulFilled`是`onFulFilled_`在 then 中的参数，在 then 中调用前者相当于调用后者。

但是这里有个问题：在调用到`onFulFilled_`函数的时候，`Bromise.then` 还没有执行，因此`onFulFilled_`实际上仍然是 null；
因此我们需要让执行在绑定之后，让 resolve 延迟调用 `onFulFilled_`。方法就是把`onFulFilled_`添加到微任务去，保证绑定的完成才去执行回调。

```js
function Bromise(executor) {
  var onFulFilled_ = null;
  // 模拟实现 resolve 和 then
  this.then = function (onFulFilled) {
    onFulFilled_ = onFulFilled;
  };
  function resolve(value) {
    //setTimeout(()=>{
    onFulFilled_(value);
    // },0)
  }
  executor(resolve, null);
}
```

## Promisify

这个单词指可以将一些函数 promisify 的函数，也就是一个装饰器。
举个栗子，图片转 base64 的基本代码如下：

```js
function imgToBase(img, callback) {
  const fr = new FileReader();
  fr.readAsDataURL(img);
  fr.onload = (e) => {
    callback(e.target.result);
  };
}
```

这里的 onload 实际上是一个回调形式的异步过程。我们可以把这个函数放到 executor 中：

```js
function promisifyImgToBase(img) {
  return new Promise((resolve, reject) => {
    const fr = new FileReader();
    fr.readAsDataURL(img);
    fr.onload = (e) => {
      resolve(e.target.result);
    };
    fr.onerror = (err) => {
      reject(err);
    };
  });
}

promisifyImgToBase(file).then((res) => {
  console.log(res);
});
```

## async/await

### async 和 await 的原理

async 在函数前面时会创建一个特殊的函数,这个函数默认返回 Promise 对象

```js
async function getNum() {
  return 2;
}
let num = getNum();
num.then((res) => clg(res));
```

这个例子中 num 实际上是这样:

```js
new Promise((resolve, reject) => {
  resolve(2);
});
//简写
Promise.resolve(2);
```

当然也可以显式返回一个 Promise 对象。
所以实际上 async 相当于一个 promisify，把一般的函数变成返回 promise 的函数。
因此，如果在一个地方需要拿到 async 函数的返回值，把它当作返回 promise 的函数就行：

```js
async function wait(){
  await new Promise(resolve=>setTimeout(()=>reslove(1),1000))
  return 0;
}

function getResult(){
  wait().then(res=>{...})
}
```

---

await 是一个执行比较复杂的语句, 按照 await 的原理,可以把 await 的语句看成是一个 Promise
先看一个例子，下面的输出顺序应该是什么？

```js
async function getUrl(url) {
  console.log(1);
  let text = await fetch(`/${url}`);
  //...
  console.log(2);
}
console.log(0);
getUrl();
console.log(3);
```

最终顺序为:`0 1 3 2`
原理:

1. 0 同步执行;然后进入 getUrl 函数直接执行 1
2. await 看成这样:

```js
new Promise((resolve, reject) => {
  resolve(fetch("/"));
}).then((res) => {
  let text = res;
  console.log(2);
});
```

因此 await 是:

- 把 await 紧挨其后的表达式当做 Promise 对象
- 把 await 语句**之后的**都当成下一个 then 内的语句
- 如果下面还有一个 await,就看做再下一个 then

> await 会导致函数暂停执行，直到被 await 等待的 Promise 状态变为 settled 才会继续。但是由于 await 执行在微任务队列，因此并不会阻塞主线程，但会阻塞后续微任务的执行。await 暂停的原理就是 Generator 函数中的 yield
> await 内部实现了 generator。参考 generator 可以得知 generator 可以保留函数内部的上下文而让函数外部继续执行（通过 yield 使函数保存在某个状态下），await 差不多也是这个效果。当 await 后面的是异步代码（Promise）时，它会立刻返回一个 pending 状态的 promise，然后让外部的同步代码继续执行。等到异步代码执行完，才会恢复函数后面部分的执行。

它还可以链式调用:

```js
async function getUrl(url) {
  console.log(1);
  let text = await fetch(`/${url}`);
  let text2 = await fetch(`/${text}`);
  console.log(2);
}

//相当于
let text, text2;
new Promise((resolve) => {
  fetch(`/${url}`);
})
  .then((res) => {
    text = res;
    return fetch(`/${text}`);
  })
  .then((res) => {
    text2 = res;
  });
```

其中`fetch(`/${text}`)`会在 text 获取之后调用(就像 then 的工作方式) 3. 由于 await 把后面任务变成了 promise 微任务,所以会先输出 3,然后在 3 的末尾调用 promise 微任务输出 2。

> 关于具体实现细节，可以参考 https://blog.poetries.top/browser-working-principle/guide/part4/lesson20.html#%E7%94%9F%E6%88%90%E5%99%A8-vs-%E5%8D%8F%E7%A8%8B

### 处理错误

await 处理的 Promise 语句如果被 reject，将 throw 这个 error。

```js
async function f() {
  await Promise.reject(new Error("Whoops!"));
}
//等同于
async function f() {
  throw new Error("Whoops!");
}
```

因此如果需要处理错误,就需要在 async 函数调用的外部加上 catch,或者直接在 await 附近加 trycatch

```js
//...
getUrl().catch((err) => {});

try{
  await ...
}catch(err){
  ...
}
```

### 使用注意

1. `forEach`中不能使用 await，因为`forEach` 只是简单的执行了下回调函数而已，并不会去处理异步的情况。因此如果要遍历数组执行，可以考虑`for of`或者`Promise.all/Promise.allSettled/Promise.race`等

# 模块

详细参考：https://juejin.cn/post/6844903744518389768

## 模块的发展历程

1. 全局函数调用形式
   将不同的功能封装成不同的全局函数
   问题: 污染全局命名空间, 容易引起命名冲突或数据不安全，而且模块成员之间看不出直接关系

2. namespace：将一组变量和函数封装在一个对象。
   外部可以直接修改模块内部的数据，不安全

3. 立即调用函数 IIFE：数据是私有的, 外部只能通过暴露的方法操作
   但是如果当前这个模块依赖另一个模块就没办法处理
   多个模块需要引入多个 script，增大加载时间

4. AMD: 主要应用于浏览器端；因为浏览器端不应该同步加载所有模块，因此 AMD 支持异步加载，通过回调的形式在加载完成后才执行代码。
   AMD 依赖于 require.js，基本思想是通过 define 方法，将代码定义为模块；通过 require 方法，实现代码的模块加载。

5. commonjs，详见下

6. CMD：整合了 commonjs 和 amd，模块的加载是异步的，模块使用时才会加载执行。CMD 的实现是 sea.js

7. UMD：可以根据导出类型的不同实现不同的导入方法，可以通过运行时或者编译时让同一个代码模块在使用 Commonjs、CMD 甚至是 AMD 的项目中运行

8. ES6 Moudle：设计思想是尽量的**静态**化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。
   ES6 模块输出的是值的引用。因此修改原文件中的值也会影响导入的值，但是导出的值不能在被导入的文件中修改
   ES6 模块不是对象，它的对外接口只是一种静态定义，不像 commonjs 一样导入的是一个对象

## 模块类型

### CommonJS

一种适用于 node 的模块化,主要特点:

- 每个文件就是一个模块，有自己的作用域。在一个文件里面定义的变量、函数、类，都是私有的，对其他文件不可见。
- 在服务器端，模块的加载是运行时同步加载的；在浏览器端，模块需要提前编译打包处理。
- 导入对象实际上是导出的**浅拷贝**
- 模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就被缓存了，以后再加载，就直接读取缓存结果。要想让模块再次运行，必须清除缓存。
- 使用 module、exports、require、global 为基本操作;其中 module 为模块内部的一个对象,常用属性有
  - `module.id` 模块的识别符，通常是带有绝对路径的模块文件名。
  - `module.filename` 模块的文件名，带有绝对路径。
  - `module.loaded` 返回一个布尔值，表示模块是否已经完成加载。
  - `module.parent` 返回一个对象，表示调用该模块的模块。
  - `module.children` 返回一个数组，表示该模块要用到的其他模块。
  - `module.exports` 表示模块对外输出的值。

> 浏览器不兼容 CommonJS 的根本原因，在于缺少四个 Node.js 环境的全局变量。
>
> - `module`
> - `exports`
> - `require`
> - `global`
>
> 只要能够提供这四个变量，浏览器就能加载 CommonJS 模块。

这四个全局变量的主要功能如下：

#### 1. module

module 是定义在每个模块中的对象；module 的常用属性上面已经列举过。
其中最核心的是`module.exports`对象，用于存储导出的变量，以便 require 读取。

#### 2. exports

为了方便，Node 为每个模块提供一个 exports 变量，指向 module.exports。因此这两者其实是完全相同的

这等同在每个模块头部，有一行这样的命令：

```js
var exports = module.exports;（commonJS隐式做了这个赋值）、
```

这样就可以通过`exports.xxx = xxx`来导出变量。

#### 3. require

require 命令的基本功能是，读入并执行一个 JavaScript 文件，然后返回该模块的 exports 对象。如果没有发现指定模块，会报错。(说白了就是将另一个文件中暴露的值，引用到本文件中。)

根据参数的不同格式，require 命令去不同路径寻找模块文件:

1. 缓存的模块优先级最高
1. 如果是内置模块，则直接返回，优先级仅次缓存的模块；
1. 如果是绝对路径 / 开头，则从根目录找
1. 如果是相对路径 ./开头，则从当前 require 文件相对位置找
1. 如果文件没有携带后缀，先从 js、json、node 按顺序查找
1. 如果是目录，则根据 `package.json`的 main 属性值决定目录下入口文件，默认情况为 index.js
1. 如果文件为第三方模块，则会引入 `node_modules` 文件，如果不在当前仓库文件中，则自动从上级递归查找，直到根目录

> 最后一步的查找优先级：
> 如果在`/home/ry/projects/foo.js`文件里调用了 `require('bar.js')`，则 Node.js 会按以下顺序查找：
>
> 1. `/home/ry/projects/node_modules/bar.js`（当前项目的目录下的 node_modules）
> 1. `/home/ry/node_modules/bar.js`（向上一级，如果有 node_modules 的话）
> 1. `/home/node_modules/bar.js`
> 1. `/home/node_modules/bar/index.js`；（如果上面不存在，说明可能是模块名错误，就会从默认按照 index.js 查找）
> 1. `/node_modules/bar.js`，即总根目录下的模块

#### 4. global

表示一个全局对象，可以被所有模块看见并共享。
其上有一些方法和属性，当在 nodejs 种直接调用某些方法是，就会被看作是从 global 对象上访问的
比如：setTimeout、clearTimeout、setImmediate 等。可以理解为类似浏览器中的 window 对象

同时，在非严格模式下的顶层 this（globalThis）就是这个 global 对象

### AMD

异步版的 Commonjs,依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行
AMD 常采用 require.js 实现,用 `require.config()`指定引用路径等，用 `define()`定义模块，用 `require()`加载模块
主要步骤为:

1. 在 main 中用 `require.config` 指定各模块路径, 参数是一个有基本路径和具体路径构成的模块整体路径,比如下面的`add: "add.js",`,"add"是模块名,后面是相对路径;

```js
require.config({
  baseUrl: "js/",
  paths: {
    add: "add.js",
  },
});
```

2. 引用模块, 参数是一个模块名的数组和一个回调函数,异步的关键

```js
require(["add", "minus"], function (add, minus) {
  add(1, 2);
  minus(3, 4);
  //...
});
```

3. 定义一个模块用 define

```js
//add.js
define(function () {
  var basicNum = 0;
  var add = function (x, y) {
    return x + y;
  };
  return {
    add,
    basicNum,
  };
});
```

### CMD

一种 AMD 的变体;
AMD 推崇依赖前置、提前执行，CMD 推崇依赖就近、延迟执行;
意思就是,AMD 会使导入的模块全部执行(即使没有调用); 而 CMD 是一种按需调用
CMD 也常和 sea.js 一同使用

```js
define(function (require, exports, module) {
  var a = require("./a");
  a.doSomething();

  var b = require("./b");
  b.doSomething();
});

/** sea.js **/
// 定义模块 math.js
define(function (require, exports, module) {
  var $ = require("jquery.js");
  var add = function (a, b) {
    return a + b;
  };
  exports.add = add;
});
// 加载模块
seajs.use(["math.js"], function (math) {
  var sum = math.add(1 + 2);
});
```

### UMD

> 它可以通过运行时或者编译时让同一个代码模块在使用 CommonJs、CMD 甚至是 AMD 的项目中运行。

意思就是, 他可以根据导出类型的不同实现不同的导入方法,比如上下文是 AMD 的方式,就可以用 AMD 导入导出
简单 UMD:

```js
((global, factory) => {
  //如果 当前的上下文有define函数，并且AMD  说明处于AMD 环境下
  if (typeof define === "function" && define.amd) {
    define(["moduleA"], factory);
  } else if (typeof exports === "object") {
    //commonjs
    let moduleA = require("moduleA");
    modules.exports = factory(moduleA);
  } else {
    global.moduleA = factory(global.moduleA); //直接挂载成 windows 全局变量
  }
})(this, (moduleA) => {
  //本模块的定义
  return {};
});
```

### ES6 Module

使用 import/export 作为导入导出的方法(下面会详细叙述)
**模块只通过 HTTP(s) 工作，在本地文件则不行**

### ES6 Module 与 CommonJS 异同

ES6 Module 和 CommonJS 模块的区别：

- CommonJS 是对模块的**浅拷⻉**，ES6 Module 是对模块的引⽤，即 ES6 Module 只存只读，不能改变其值，也就是指针指向不能变，类似 const；
- import 的接⼝是 read-only（只读状态），不能修改其变量值。 即不能修改其变量的指针指向，但可以改变变量内部指针指向。可以对 commonJS 对重新赋值（改变指针指向），但是对 ES6 Module 赋值会编译报错。
- ESM 是静态导入，即并不需要执行某个具体的函数（比如 require），而是在 js 的解释阶段就可以得到导入结果，并且仅在第一次导入时解析，后续导入都是第一次的复制。这种情况下就可以实现一些解释阶段的优化，比如提前得到依赖关系进行代码精简（treeshaking）等。

ES6 Module 和 CommonJS 模块的共同点：

- CommonJS 和 ES6 Module 都可以对引⼊的对象进⾏赋值，即对对象内部属性的值进⾏改变。

## ES6 Module

一个模块实际上就是一个文件,特点如下:

- 模块内部 js 始终使用"use strict"
- 模块代码**仅在第一次导入时被解析**,生成导出，然后它被复制给所有对其的导入,也就是说:
  - 导入语句`import`在编译阶段就被执行，不管在代码的什么位置，都会被提到最前执行。被导入的模块虽然不会立即运行，但它也会在执行阶段先于当前模块执行
  - 导入时模块中的代码就已经被解析（执行）,如果有错误会在此时报出;
  - **只解析一次**,同一个模块在其他文件中再次导入不会再解析
  - 如果导出一个引用类型的值(对象/数组),则一处数据更改也会影响另一处的导入的数据,但是不会影响源文件.所以尽量不要更改模块,而是只读或者使用拷贝
- `<script type="module">`下的 this 是 undefined 而不是 window(这个跟严格模式有关系)
- `<script type="module">`的脚本是异步的,会等待 html 完全就绪才会运行

### ESM 的工作原理

参考：https://juejin.cn/post/7098192216229117959

首先，ESM 可以在浏览器中通过`<script type="module">`直接使用，也可以是通过 webpack 等打包工具辅助解析。但不管是哪种方式，它的原理和流程基本都是一样的。
如果是浏览器使用，该 script 则会放在渲染之后执行，相当于加了 defer；

ESM 的工作流程，大致可以分为三步：

1. 创建依赖关系图。在解析阶段通过解析各个模块之间的依赖关系，生成一个模块的依赖关系图（树），不同依赖项之间通过 export\import 语句来进行关联。
2. 创建模块记录 Module Record。和 commonjs 不同的是，由于 ESM 是在编译阶段完成的，因此并不会生成具体的模块对象（类似 cjs 中的 module 对象），而是一个 Module Record，包含了当前模块的相关信息，比如该模块代码的 AST、导入关系、导出值等等。
   ![](https://pic.imgdb.cn/item/637d0dbb16f2c2beb10860aa.jpg)
3. 模块记录的实例化，模块记录转化为模块实例，浏览器最终能够读取也就是 Module Instance。

具体来说，主要有三个过程：构造、实例化和求值：

1. 构造：查找、下载并解析所有文件到模块记录中。这个阶段中，每个模块都会执行这三个步骤

- 查找：找出从哪里得到这个文件。从入口文件开始，查找并形成一棵依赖关系树
- 下载：获取具体的文件，可以是从 URL 下载，或者从文件系统获取
- 解析：对模块文件生成一个模块记录，包含了当前模块的 AST，引用了哪些模块的变量，以及一些特定属性和方法。**一旦 Module Record 被创建，它会记录在模块映射 Module Map 中**。被记录后，如果再有对相同 URL 的请求，Loader 将直接采用 Module Map 中 URL 对应的 Module Record。
  在构造过程结束时，从主入口文件变成了一堆模块记录 Module Record：
  ![](https://pic.imgdb.cn/item/637d102316f2c2beb10ca94f.jpg)

2. 实例化。实例化阶段的主要作用是保证在静态阶段就把内存和导入导出链接起来，也就是 ESM 采取的一种叫做**动态链接**的方式。

导入希望获取到对应的导出的值，那么导入就需要找到导出变量在内存中存放的位置，读取它并为自己所用。但是这时代码还没有被执行，内存现在都是空的，并且现阶段也不能知道内存中哪些地方会是哪个模块的导出值。因此就需要一种方式，使得提前“预定”好对应的内存位置，使对应的导出和导入都指向这个位置。等到执行阶段，执行代码并向内存中填值，导入和导出就都可以获取到具体的值了。

具体来说，JS 引擎创建一个**模块环境记录(Module Enviroment Record)**来管理每个 Module Record 的变量。然后它在内存中找到所有导出内容对应的位置，并跟踪该位置；这一步实际上就是在链接内存和导出，可确保所有导入都可以链接到匹配的导出。拥有这样的动态绑定可以使我们在不运行任何代码的情况下连接所有模块。

> 导入和导出都是指向内存的同一个位置，这也就解释了为什么 ESM 导出值不能更改（不能修改内存的指向），但可以修改对象属性（可以修改内存本身）。

3. 求值。上一步已经“预定”好了内存的位置，这一步就是执行代码，并向内存中填值。注意这一步执行的是顶层代码，也就是函数之外的代码，主要目的是向导入和导出的变量中填值。

全过程：
![](https://pic.imgdb.cn/item/637d169e16f2c2beb112a747.jpg)

### import

`import * as xxx from path`将所有内容导入为一个对象,相当于所有的导入都在 xxx 这个对象里

```js
import * as say from "./say.js";
say.sayHi("John");
say.sayBye("John");
```

导出到 as 后的这个对象，包含默认导出和基本导出的所有变量，都作为该对象的一个属性。具名导出的属性名就是导出的变量名，而默认导出则会被放在一个叫做“default”的属性里

如果导出的是个匿名函数或匿名变量，则这个变量/函数就会被命名为 default，可以通过`obj.xxx`访问

这也是为什么默认导出只能有一个，多的则会相互覆盖。

```js
// test.js
export const hello = "hello";
export default () => "world";

// index.js
import * as test from "./test.js";

test.default(); // 'world' // 相当于执行了那个匿名导出的函数，这个函数还是匿名的，test.default是它的引用方式
test.hello; // 'hello'
```

另外,借助 import(module) 表达式可以实现动态导入
import(module) 表达式加载模块并返回一个 promise，该 promise resolve 为一个包含其所有导出的模块对象。比如:

```js
//...
import('./test.js')
.then(obj=>console.log(obj))
//or
async getImport(){
    let obj = await import('./test.js')
    return obj
}
```

resolve 的 obj 就是模块的导出对象;可以对对象使用其 default 属性来访问默认导出
注意这种导入就不能使用`type="module"`了,而是看做一个正常的表达式

# Proxy 和 Reflect

## Proxy

一个 Proxy 对象包装另一个对象并拦截诸如读取/写入属性和其他操作，可以选择自行处理它们，或者透明地允许该对象处理它们。

```js
let proxy = new Proxy(target, handler);

let proxy = new Proxy({ name: "aaa" }, {});
proxy.age = 19;
console.log(proxy.name);
```

- target 是要代理的对象, handler 是"捕捉器",是一个包含 getter/setter 等属性的对象
- 直接对 Proxy 的实例读写相当于对代理对象读写(假设没有 handler),也可以迭代
- Proxy 本身没有任何属性
- 常见的 handler 方法:
  |内部方法| Handler 方法| 何时触发|
  |---|---|---|
  |`[[Get]]` |get| 读取属性|
  |`[[Set]]` |set| 写入属性|
  |`[[HasProperty]]`| has| in 操作符|
  |`[[Delete]]` |deleteProperty| delete 操作符|
  |`[[Call]]` |apply| 函数调用|
  |`[[Construct]]` |construct| new 操作符|

handler 的使用：

```js
let target = {};
proxy = new Proxy(target, {
  get(target, prop) {
    //...
  },
  set(target, prop, value) {
    //...
  },
  deleteProperty(target, prop) {
    //...
  },
});

proxy.xxx; //get
proxy.xxx = yyy; //set
delete proxy.xxx; //deleteProperty
```

**注意,这些 handler 方法都是访问代理对象而不是源对象时触发的**。
直接访问源对象属性并不会触发代理的拦截,这点和 `defineProperty` 加上 getter/setter 不同
所以为了可以直接访问对象代理,可以把代理对象赋给源对象

### get

get 方法一般如下:

```js
let numDict = {
  zero: 0,
  one: 1,
  two: 2,
};
let proxy = new Proxy(numDict, {
  get(target, property, receiver) {
    if (property in target) {
      return target[property];
    } else {
      return property;
    }
  },
});
```

参数:

- target:就是代理的对象,参数第一个
- property :访问的属性名，比如访问`proxy.xxx`,prop 就是 xxx
- receiver :如果目标属性是一个 getter 访问器属性，则 receiver 就是本次读取属性所在的 this 对象。**一般都是 Proxy 实例**

### set

set 方法如下:

```js
let numbers = [];
numbers = new Proxy(numbers, {
  set(target, prop, val) {
    if (typeof val == "number") {
      target[prop] = val;
      return true;
    } else {
      return false;
    }
  },
});
numbers.push(1); // 添加成功
numbers.push(2); // 添加成功
numbers.push("test"); // TypeError（proxy 的 'set' 返回 false）
```

- 参数:
  - target: 代理对象
  - property: 访问属性名，
  - value: 设置的属性的值，比如`proxy.xxx = aaa`,value 就是 aaa
  - receiver: 与 get 捕捉器类似，仅与 setter 访问器属性相关。
- set 必须返回 true 或者 false
- 对于数组对象等的内建方法(比如 push/shift/pop),也会触发 setter,因此也可以作为内建方法的监听

### ownKeys

对于`forin`和`Object.keys/entries/values`等遍历属性方法都会用到`[[OwnPropertyKeys]]`方法,通过 ownKeys 捕捉器拦截
参数只有 target 作为代理对象

```js
let user = {
  name: "John",
  age: 30,
  _password: "***",
};

user = new Proxy(user, {
  ownKeys(target) {
    return Object.keys(target).filter((key) => !key.startsWith("_"));
  },
});

for (let key in user) alert(key); // name age
```

注意:ownKeys 返回的键**只少不多**;也就是说这些键只能是原本就有的,不能返回原对象没有的键;同理,也不会返回不可枚举的键

### deleteProperty

用于对删除进行拦截
参数和 get 相同

```js
let user = {
  name: "John",
  age: 30,
  _password: "***",
};
user = new Proxy(user, {
  deleteProperty(tar, prop) {
    if (prop.startsWith("_")) return false;
    else {
      delete tar[prop];
      return true;
    }
  },
});
```

### has

has 会拦截`in`,比如:

```js
let range = {
  from: 1,
  to: 10,
};
let proxy = new Proxy(range, {
  has(tar, prop) {
    return prop >= tar.from && prop <= tar.to ? true : false;
  },
});

console.log(5 in proxy); //true
```

### 其他方法

其他一些方法和上面类似,列举如下:
|方法|参数|返回|效果|
|---|---|---|---|
|`construct (target, args, newTarget)` |`args` 是创建实例时传入的参数数组;`newTarget` 是 new 命令作用的构造函数|必须是个对象,用于设置属性|用于拦截 new 命令|
|`defineProperty (target, key, descriptor)`|`key` 和 `descriptor` 都是 `Object.defineProperty()`传入的参数|true/false|拦截了 `Object.defineProperty()`操作|
|`getPrototypeOf(target)`/`setPrototypeOf (target, proto)`|`proto` 是设置的父原型|返回 proto/true 或 false|拦截获取/设置原型|

## Reflect

> Reflect 是一个内建对象，可简化 Proxy 的创建。

也就是说, reflect 是一种"映射",可以把我们对 reflect 对象的操作"映射到"源对象上
每个 Proxy 对应的方法都是 reflect 对应的, 比如 get 中使用的应该是 `Reflect.get()`,has 使用 `Reflect.has()`,以此类推
举个栗子:

```js
let user = {
  name: "John",
};

user = new Proxy(user, {
  get(target, prop, receiver) {
    console.log(`GET ${prop}`);
    return Reflect.get(target, prop, receiver);
  },
  set(target, prop, val, receiver) {
    console.log(`SET ${prop}=${val}`);
    return Reflect.set(target, prop, val, receiver);
  },
});
```

这里的 Reflect 代替的直接的读取, **成功返回 true,失败返回 false**
注意这里用到了之前没用到的 receiver;这个属性的作用与 this 相关,比如:

```js
let user = {
  _name: "John",
  get name() {
    console.log(`hello,${this._name}!`);
  },
};

let proxy = new Proxy(user, {
  get(target, prop, receiver) {
    console.log(`GET ${prop}`);
    return Reflect.get(...arguments);
  },
  set(target, prop, val, receiver) {
    console.log(`SET ${prop}=${val}`);
    return Reflect.set(...arguments);
  },
});

let stu = {
  __proto__: proxy,
  _name: "Mike",
};
stu.sayHi();
```

这个例子中希望 stu 的调用 this 能指向 stu 而不是 user,就需要 reflect
如果没有 reflect:

- stu 访问 name, 在原型 proxy 上寻找
- proxy 通过 get 拦截, 并默认把上下文看做 target,调用 user 的 getter,返回一个 user 上下文
- 产生的是"John"
  如果有 reflect ,就可以把正常的上下文"receiver"保留,从而使得 this 的指向依旧是"点前面的对象"

---

Reflect 的其他作用：

1. `Object`对象的一些明显属于语言内部的方法（比如`Object.defineProperty`），放到`Reflect`对象上。现阶段，某些方法同时在`Object`和`Reflect`对象上部署，未来的新方法将只部署在`Reflect`对象上。也就是说，从`Reflect`对象上可以拿到语言内部的方法。

2. 修改某些`Object`方法的返回结果，返回 false、true 等比较清晰的信息。比如，`Object.defineProperty(obj, name, desc)`在无法定义属性时，会抛出一个错误，而`Reflect.defineProperty(obj, name, desc)`则会返回`false`。
3. `Reflect`对象的方法与`Proxy`对象的方法一一对应，只要是`Proxy`对象的方法，就能在`Reflect`对象上找到对应的方法。这就让`Proxy`对象可以方便地调用对应的`Reflect`方法，完成默认行为，作为修改行为的基础。

# 事件循环、消息队列、同步异步、宏任务微任务、调用栈、垃圾回收等各种原理性杂项

https://blog.poetries.top/browser-working-principle/

# 内存管理

对于 JavaScript 来说，会在创建变量（对象，字符串等）时分配内存，并且在不再使用它们时“自动”释放内存，这个自动释放内存的过程称为垃圾回收。
JS 环境中分配的内存有如下声明周期：

1. 内存分配：当我们申明变量、函数、对象的时候，系统会自动为他们分配内存
1. 内存使用：即读写内存，也就是使用变量、函数等
1. 内存回收：使用完毕，由垃圾回收机制自动回收不再使用的内存

## 垃圾回收

js 的垃圾回收的对象应该分为基本类型数据和引用类型数据两种方式。

对于基本类型数据，当函数的执行上下文从调用栈中弹出时就相当于销毁了这些变量。但是对于引用数据类型，只是销毁了它的引用，而实际的数据还在堆中占用着空间，这时就需要对存储这些对象的空间做垃圾回收，把没有用的对象回收掉。

---

js 的垃圾回收区分为新生代和老生代两个部分。

对于新生代，通常采用 Scavenge 算法。

即 将空间一分为二，From 部分表示正在使用的内存，To 是目前闲置的内存。当进行垃圾回收时，V8 将 From 部分的对象检查一遍，如果是存活对象那么复制到 To 内存中（复制期间即进行内存整理），如果是非存活对象直接回收即可。

当所有的 From 中的存活对象按照顺序进入到 To 内存之后，From 和 To 两者的角色对调，From 现在被闲置，To 为正在使用，如此循环。

当新生代的某部分内存经过好几次回收仍存在，就会晋升到老生代中。

老生代的回收方法主要是以下两种（老生代才是标记清除）：

### 引用计数垃圾收集

这是最初级的垃圾回收算法。引用计数算法定义“内存不再使用”的标准很简单，就是看一个对象是否有指向它的引用。 如果没有其他对象指向它了，说明该对象已经不再需了。

但是如果两个对象相互引用，尽管他们已不再使用，垃圾回收不会进行回收，导致内存泄露。
比如这样：

```js
function f() {
  var obj1 = {};
  var obj2 = {};
  obj1.a = obj2;
  obj2.a = obj1;
}

f();
```

为了解决循环引用造成的问题，现代浏览器通过使用标记清除算法来实现垃圾回收。

### 标记--清除

标记清除算法将“不再使用的对象”定义为“无法达到的对象”。 简单来说，就是从根部（在 JS 中就是全局对象）出发定时扫描内存中的对象。 凡是能从根部到达的对象，都是还需要使用的。 那些无法由根部出发触及到的对象被标记为不再使用，稍后进行回收。

实际上标记清除算法也包括了上面的引用计数方法；当一个对象不能被标记搜查到时，自然也就不能被引用。

工作流程：

1. 垃圾收集器会在运行的时候会给存储在内存中的所有变量都加上标记。
1. 从根部出发将能触及到的对象的标记清除。
1. 那些还存在标记的变量被视为准备删除的变量。
1. 最后垃圾收集器会执行最后一步内存清除的工作，销毁那些带标记的值并回收它们所占用的内存空间。

# event loop

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e15fc609aa84eac973c5b8ff163c11c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

详细讲解可以看这个视频：https://www.bilibili.com/video/BV1oV411k7XY/?spm_id_from=333.788.recommend_more_video.-1
从 12 分钟开始看

纠正几个误区：

0. 整个结构主要有三个部分：

- js 调用栈，即所有同步任务直接进入的地方，所有任务要执行必须经过调用栈
- webapis，或者可以是浏览器其他线程（对 js 来说就是一些 api），执行诸如定时器、网络请求等；当条件合适，会把回调放到任务队列中
- task queue，即任务队列，上一步完成后会把回调放入在此处排队，等待 eventloop 依次取任务执行。

1. event loop 并不是一直在工作，它的作用是把任务队列（消息队列）中的任务（消息）依次给 js 调用栈，让 js 执行这些任务；如果一段代码全部都是同步任务，或者异步任务还没有触发任何回调，event loop 就不工作。
2. event loop 只有在**调用栈为空**的时候才会从 task queue 中取任务放到调用栈中执行。
3. 不是所有任务都会经过 task queue，或者叫做消息队列；只有异步回调会经过，其他任务直接在 js 调用栈中运行！
4. 上面视频最后讲到了渲染，实际上渲染是在和 task queue 争夺调用栈（即 GUI 渲染线程和 JS 执行线程是互斥的），同样只有栈空才能渲染；这也是不建议让大量代码堆积阻塞的原因。

# 宏任务微任务

宏任务：`setTimeout`，`setInterval`，`Ajax`，DOM 事件，同步代码算作一整个宏任务
微任务：`Promise.then/.catch/.finally`，`async/await`，`queueMicrotask`。
两者区别：

- 宏任务：DOM 渲染后触发
- 微任务：DOM 渲染前触发。即**微任务是有可能阻塞渲染的**

纠正误区：

1. 宏任务和微任务是在两个队列中的，并不是微任务在宏任务中；
2. 同步代码算作一整个宏任务，因此同步代码执行完后会先调用微任务，然后才是剩下的异步宏任务。因此在表现上就是微任务先执行；实际上微任务一定是在宏任务之后执行。
3. `new Promise`本身是同步任务，包括其内部的`executor`都是同步的，只有`.then/.catch/.finally`或者 await 才是异步微任务
4. 对于上面的 api，一个函数就是一个宏任务；比如一个`setTimeout`就是一个宏任务，这个**宏任务执行完后会立刻把微任务队列中的所有微任务都执行**，才会进入下一个宏任务（下一个 setTimeout）

微任务会在执行任何其他事件处理，或渲染，或执行任何其他宏任务之前完成。eventLoop通常按照`同步任务 -> 微任务 -> 渲染 -> 宏任务 -> 微任务 -> ....`这样的顺序执行。类似下图：

![](https://pic.imgdb.cn/item/642470d6a682492fcc1d7eb6.jpg)

# DOM

## 概述

DOM 是文档对象模型，实际上是一种规范；根据这个规范，文档（document）中的标签、文本、属性、注释等都是对象，js 都可以访问他们并操作他们。

### 节点

DOM 的最小组成单位叫做节点（node），节点的类型有七种。

- `Document`：整个文档树的顶层节点
- `DocumentType`：doctype 标签（比如`<!DOCTYPE html>`）
- `Element`：网页的各种 HTML 标签（比如`<body>`、`<a>`等）
- `Attr`：网页元素的属性（比如`class="right"`）
- `Text`：标签之间或标签包含的文本
- `Comment`：注释
- `DocumentFragment`：文档的片段

### DOM Tree

一个文档的所有节点，按照所在的层级，可以抽象成一种树状结构。这种树状结构就是 DOM 树。它有一个顶层节点，下一层都是顶层节点的子节点，然后子节点又有自己的子节点，就这样层层衍生出一个金字塔结构，又像一棵树。

最根本的对象是`document`，代表整个文档对象；对于文档所有节点的操作都要在它之下。

## 遍历 DOM

DOM 主要关系图：
![](https://pic.imgdb.cn/item/622ac3fd5baa1a80abadacdf.jpg)

### 顶层节点

关于具体节点的 DOM 不再赘述，主要看看这几个顶层节点：

- `<html>` = `document.documentElement`
  最顶层的 document 节点是 `document.documentElement`。这是对应 `<html>` 标签的 DOM 节点。html 标签是一个可以完全可操控的对象，常见的比如操作`scrollTop`属性用于滚动。
- `<body>` = `document.body`
  另一个被广泛使用的 DOM 节点是 `<body>` 元素 — `document.body`。
- `<head>` = `document.head`
  `<head>` 标签可以通过 `document.head` 访问。

### 关系节点

给定一个 DOM 节点，我们可以使用导航（navigation）属性访问其直接的邻居。

这些属性主要分为两组：

- 对于所有节点：`parentNode`，`childNodes`，`firstChild`，`lastChild`，`previousSibling`，`nextSibling`。
  所有节点是指包括文本、元素、注释等全部节点（上面说的 7 个节点）
  ![](https://pic.imgdb.cn/item/622ac5c15baa1a80abaed0a6.jpg)

- 仅对于元素节点：`parentElement`，`children`，`firstElementChild`，`lastElementChild`，`previousElementSibling`，`nextElementSibling`。
  这里父、子节点都是单数，因为默认指向都是直接父元素和直接子元素。
  ![](https://pic.imgdb.cn/item/622ac5b05baa1a80abaec72e.jpg)

## `getElement*`和`querySelector*`

区别：

- 前者只能选择特定的选择器，比如`getElementById`只能选择 id；但是后者可以选择任意 css 选择器（字符串形式），直接的标签名当然也可以，还可以是伪类
- `getElementsBy*`方法都会返回一个 实时的集合，当 dom 变动时会相应变化；但是后者是**静态的**。

## DOM 节点类

每个 DOM 节点都属于相应的内建类；所谓内建类，即包含元素特有属性和方法的类，比如`HTMLInputElement`是`<input>`元素的内建类，其上有输入相关的属性。

内建类之间的大致关系如下：
![](https://pic.imgdb.cn/item/622acfaa5baa1a80abb50e82.jpg)

类如下所示：

- `EventTarget` — 是根的**抽象类**。该类的对象从未被创建（没有实例）。它作为一个基础，以便让所有 DOM 节点都支持所谓的“事件（event）”
- `Node` — 也是一个抽象类，充当 DOM 节点的基础。它提供了树的核心功能：parentNode，nextSibling，childNodes 等（它们都是 getter）。Node 类的对象从未被创建。但是有一些继承自它的具体的节点类，例如：文本节点的 Text，元素节点的 Element 等。
- `Element` — 是 DOM 元素的基本类。它提供了元素级的导航（navigation），例如 nextElementSibling，children，以及像 getElementsByTagName 和 querySelector 这样的搜索方法。浏览器中不仅有 HTML，还会有 XML 和 SVG。Element 类充当更多特定类的基本类：SVGElement，XMLElement 和 HTMLElement。
  - `HTMLElement` — 最终是所有 HTML 元素的基本类。各种 HTML 元素均继承自它：
  - `HTMLInputElement` — `<input>` 元素的类，
  - `HTMLBodyElement` — `<body>` 元素的类，
  - `HTMLAnchorElement` — `<a>` 元素的类，
  - h5 新增的一些语义化块级元素很多没有特定属性，都直接属于`HTMLElement`

当获取某个元素上的方法或属性时，会按照如上的层次顺序依次查找：
例如 `<input>` 元素的 DOM 对象。它属于 `HTMLInputElement` 类。

它获取属性和方法，并将其作为下列类（按继承顺序列出）的叠加：

1. `HTMLInputElement` — 该类提供特定于输入的属性，
1. `HTMLElement` — 它提供了通用的 HTML 元素方法
1. `Element` — 提供通用元素方法，
1. `Node` — 提供通用 DOM 节点属性，
1. `EventTarget` — 为事件（包括事件本身）提供支持，
1. `Object`，因为 `hasOwnProperty` 这样的“普通对象”方法也是可用的。

因此 input 元素继承了上面每个父类的属性和方法，并自己重写了一部分。

> 可以通过`console.dir(elem)` 将元素显示为 DOM 对象在控制台查看

### 元素属性

下面这些属性是基本每个元素通用的、常用的属性：

#### `innerHTML`

输出该元素**内部**的 HTML（即子元素），以字符串形式输出；注意不会输出该元素本身，会把其下各级子元素都输出
![](https://pic.imgdb.cn/item/622ad4c95baa1a80abb87d46.jpg)
可以被写入，直接赋值相当于完全替换；可以使用`elem.innerHTML += "...";`添加 HTML 元素

> `innerHTML`和`innerText`区别：后者只会输出各级子元素中的文本节点值，包括换行符等特殊字符；前者输出的是 HTML

#### `nodeValue`/`data`/`textContent`

文本节点内容，即上面所说 7 种节点类型之一的文本节点。
前提是访问的必须是文本节点，甚至不可以是包含文本节点的元素节点（比如`<p>`元素内部包含的文本不能通过对`<p>`本身获取，结果只能是 null）

---

更好的方式是使用`textContent`，会返回当前元素下所有文本节点的文本（不管嵌套多深）

## `Attributes` & `properties`

### 特性`attributes`

`attributes`指 html 标签上注明的属性，比如 class/id/style/name 等等

```html
<div class="main-block">...</div>
```

`attributes`会在浏览器解析时作为`properties`插入到元素对象中

```js
div = {
  //...
  class: "main-block",
  //...
};
```

只有符合标准的`attributes`才会被添加，比如特定的几个 id/name/class 等；其余的并不会；

```html
<body id="test" something="no-standard">
  <script>
    console.log(document.body.id); // test
    // 非标准的特性没有获得对应的属性
    console.log(document.body.something); // undefined
  </script>
</body>
```

自定义`attributes`可通过`data-`的形式添加
所有以 `data-` 开头的特性均被保留，可在 dataset 属性中使用，通过 dataset 对象可以获取：

```html
<body data-about-me="Elephants">
  <script>
    console.log(document.body.dataset.aboutMe); // Elephants
    //注意连字符形式转为小驼峰获取
  </script>
</body>
```

所有属性（包括不标准的）都可以通过以下方法进行访问：

- `elem.hasAttribute(name)` — 检查特性是否存在。
- `elem.getAttribute(name)` — 获取这个特性值。
- `elem.setAttribute(name, value)` — 设置这个特性值。
- `elem.removeAttribute(name)` — 移除这个特性。

HTML 特性有以下几个特征：

- 名字是大小写不敏感的（id 与 ID 相同）。
- 值总是字符串类型的。

### 属性`properties`

由于元素本质上是一个 DOM 对象，因此可以向这个对象添加属性或方法，甚至向原型上添加：

```js
document.body.sayTagName = function () {
  alert(this.tagName);
};

Element.prototype.sayHi = function () {
  alert(`Hello, I'm ${this.tagName}`);
};
document.documentElement.sayHi(); // Hello, I'm HTML
document.body.sayHi(); // Hello, I'm BODY
```

不同于`attributes`，而`properties`是多类型的；比如 style 是一个对象。
实际上`properties`就是相当于对象上的属性，其性质和操作等都可以直接认为是对象上的属性。

## `classList`和`style`

通常修改样式的方法不应该是直接操作 style 属性，而是添加或删除一个 class

- `className`会替换整个 class 字符串
- `classList` 的方法：
  - `elem.classList.add/remove(class)` — 添加/移除类。
  - `elem.classList.toggle(class)` — 如果类不存在就添加类，存在就移除它。
  - `elem.classList.contains(class)` — 检查给定类，返回 true/false。

---

直接操作 style 相当于操作元素的 style `attributes`，因此不能通过 style 属性获取元素的样式：

```html
<head>
  <style>
    body {
      color: red;
      margin: 5px;
    }
  </style>
</head>
<body>
  The red text
  <script>
    alert(document.body.style.color); // 空的
    alert(document.body.style.marginTop); // 空的
  </script>
</body>
```

但是可以通过`getComputedStyle`获取，参数第一个是元素 dom 对象，第二个是伪元素（可以选择该元素的`::after`等伪元素）

```js
getComputedStyle(element, [pseudo]);

let computedStyle = getComputedStyle(document.body);
console.log(computedStyle.marginTop); // 5px
console.log(computedStyle.color); // rgb(255, 0, 0)
```

getComputedStyle 实际上返回的是属性的解析值（resolved）：

- 计算值：即确切的 css 属性值
- 解析值：把相对单位解析为确切数值的绝对单位值

> 尽量不要从`getComputedStyle`中获取元素宽高；因为这样获取的是 css 中设定的值，如果 css 中并没有显式指定宽高就有可能是 auto。
> 宽高等具体尺寸最好用`clientWidth/clientHeight`等获取；这部分的详解在 css 中。

## Window 大小和滚动

为了获取窗口（window）的宽度和高度，我们可以使用 `document.documentElement.clientWidth/clientHeight`

> `window.innerWidth/innerHeight` 包括了滚动条，如果有滚动条的情况下实际上的可视空间会偏小。尽量使用 client

### 滚动

DOM 元素的当前滚动状态在其 scrollLeft/scrollTop 属性中；
但是对于整个 html 页面，通常使用这两个 api：（只读）

```js
console.log(window.pageYOffset); //表示当前可视区域顶部到整个文档顶部的距离，相当于整个html元素的scrollTop
console.log(window.pageXOffset);
```

这几个属性是相同的：

```js
// pageYOffset 就是 scrollY 的别名
(window.pageYOffset === window.scrollY) === document.documentElement.scrollTop;
```

如果要更改，可以使用 api 手动滚动：

- `window.scrollBy(x,y)`：移动相对距离
- `window.scrollTo(pageX,pageY)`：移动到绝对位置，单位是像素值，值可以从`window.pageYOffset`获取
- `elem.scrollIntoView(true/false)`：移动到当前元素，参数 true 为使 elem 出现在窗口顶部。元素的上边缘将与窗口顶部对齐；false 则为底部。

> 要使文档不可滚动，只需要设置 `document.body.style.overflow = "hidden"`。该页面将“冻结”在其当前滚动位置上。
> 再设置一次空值则可以取消

## `Mutation Observer API`

`Mutation Observer API` 用来监视 DOM 变动。DOM 的任何变动，比如节点的增减、属性的变动、文本内容的变动，这个 API 都可以得到通知。
`Mutation Observer`是一个典型的**微任务**，通过这个 api 插入的处理程序会在当前所有 dom 变动之后执行。
详见：
https://wangdoc.com/javascript/dom/mutationobserver.html

# 事件

## `EventTarget`

事件的本质是程序各个组成部分之间的一种通信方式，也是异步编程的一种实现。DOM 节点的事件操作（监听和触发），都定义在`EventTarget`接口。
所有节点对象都部署了这个接口，`Element`/`document`/`window` 是最常见的 `event targets`，这些 targets 可以被安插监听事件，可以被触发事件，也支持通过 `onevent` 设置事件处理程序。
该接口主要提供三个实例方法。

- `addEventListener()`：绑定事件的监听函数
- `removeEventListener()`：移除事件的监听函数
- `dispatchEvent()`：触发事件

从结构上来说，`EventTarget`实际上是一个维护各种监听处理回调、并按照事件类型分类的对象，通过`addEventListener()`等方法向上面添加、移除或者触发回调。
简单实现：

```js
class EventTarget {
  constructor() {
    this.listeners = {};
  }
  addEventListener(type, callback) {
    if (!(type in this.listeners)) {
      //如果之前没有这个类型的事件就初始化为空数组
      this.listeners[type] = [];
    }
    this.listeners[type].push(callback);
  }
  removeEventListener(type, callback) {
    if (!(type in this.listeners)) return;
    this.listeners = this.listeners.filter((list) => list !== callback);
  }
  dispatchEvent(event) {
    if (!(type in this.listeners)) return;
    event.target = this; //绑定e.target为当前EventTarget
    this.listeners[event.type].forEach((list) => {
      list.call(this, event); //依次触发该事件类型对应的每个事件
    });
  }
}
```

## 事件的异步和同步

因此关于事件的监听、触发的顺序大致是这样：

1. 解析到 js 代码中有`addEventListener()`部分，将对应的处理程序或监听安插在该`EventTarget`上，但并不会执行
2. 用户触发事件，这个过程可能由浏览器进程捕捉到，通过 IO 线程发送消息；
3. 该`EventTarget`维护一个事件池（事件侦听器列表），收到事件触发的消息后，将处理程序封装成消息，放入消息队列中，后续执行。

因此对于原生事件来说，可以理解为：

- 事件的触发是异步（随时可能触发）
- 事件的处理是同步（一触发立即放入消息队列）

### 嵌套事件的立即触发

> 通常事件是在队列中处理的。也就是说：如果浏览器正在处理 `onclick`，这时发生了一个新的事件，例如鼠标移动了，那么它的处理程序会被排入队列，相应的 `mousemove` 处理程序将在 `onclick` 事件处理完成后被调用。

上面的引用说到，如果一个事件处理中触发了另一个事件，不会打断当前事件回调的执行，而是会放在当前事件之后执行。因为事件的触发是异步的，只有触发时事件对象才能知道已经触发，并把事件处理程序插入到消息队列中。

---

但是特殊情况在于嵌套触发事件，会使得处理程序被“立即触发”，相当于同步触发。
比如下面这个例子，在一个事件的处理程序中触发了另一个事件：

```html
<body>
  <main id="main">
    <button id="btn">clickme</button>
    <button id="btn2">clickme2</button>
  </main>
</body>
<script>
  btn.addEventListener("click", (e) => {
    console.log("btn1 has toggled");
    btn2.click();
    console.log("btn1 finished");
  });
  btn2.addEventListener("click", (e) => {
    console.log("btn2 has toggled");
  });
</script>
```

点击 btn1 时的输出顺序：
![](https://pic.imgdb.cn/item/622cb2a85baa1a80abc02079.jpg)
可以看到先去执行了另一个事件的处理程序，然后才返回继续执行自己的。
因此**嵌套事件的传播和处理先被完成，然后处理过程才会返回到外部代码**；但通常事件都会**依次执行**，并不会在一个处理程序中插入另一个。

## `addEventListener`

> `addEventListener()`的工作原理是将实现`EventListener`的函数或对象添加到调用它的`EventTarget`上的指定事件类型的事件侦听器列表中。

使用：

```js
element.addEventListener(event, handler, { options });
```

- `event`：事件名，例如："click"。
- `handler`处理程序。handler 不一定是函数，可以是一个对象或者一个类，比如这样：

```html
<button id="elem">Click me</button>

<script>
  class Menu {
    handleEvent(event) {
      // mousedown -> onMousedown
      let method = "on" + event.type[0].toUpperCase() + event.type.slice(1);
      this[method](event);
    }

    onMousedown() {
      elem.innerHTML = "Mouse button pressed";
    }

    onMouseup() {
      elem.innerHTML += "...and released.";
    }
  }

  let menu = new Menu();
  elem.addEventListener("mousedown", menu);
  elem.addEventListener("mouseup", menu);
</script>
```

- `options`:一个对象，属性有这三种：
  - `once`：如果为 true，事件只触发一次；
  - `capture`：表示是否在事件捕获阶段触发。即“事件捕获”，从外向内触发事件；为 false 就是默认的“事件冒泡”
  - `passive`：如果为 true，`preventDefault()`不会生效，也就是说你无法阻止该事件的默认事件的发生。比如页面的滚动事件，大多数浏览器都将其设置为passive:true，为的就是防止某些事件处理程序阻塞渲染事件的执行。

## 事件对象`event`

> 当事件发生时，浏览器会创建一个 event 对象，将详细信息放入其中，并将其作为参数传递给处理程序。

### `event.target` 和 `event.currentTarget` 和 `this`

`this=event.currentTarget`;
这两个和 target 的区别在于:target 是指具体触发的元素,this 是指事件被插入的位置那个元素
比如:

```js
<div class="main">
  <div class="inner">clickme</div>
</div>;
//js
//如果点击的是inner
document
  .getElementsByClassName("main")[0]
  .addEventListener("click", (event) => {
    clg(event.target.className); //inner
    clg(this.className); //main
  });
```

这个时候如果点击里边的 inner,this 就还是 main,而 target 则是点击的具体元素:inner

## 事件冒泡

> 当一个事件发生在一个元素上，它会首先运行在该元素上的处理程序，然后运行其父元素上的处理程序，然后一直向上到其他祖先上的处理程序。

也就是说,触发一个元素的事件会从下向上连着触发它父元素和祖元素的事件。
注意这里是连续触发**事件**，而不是单纯的向上传递事件处理对象。
比如在子元素 input 点击一下，会触发 focus 事件，同时会触发其父元素的 click 事件：

```html
<body>
  <main onclick="handleClick()">
    <input id="my-input" onfocus="handleFocus()" />
  </main>
</body>
```

### 阻止冒泡

冒泡事件从目标元素开始向上冒泡。通常，它会一直上升到 `<html>`，然后再到 document 对象，有些事件甚至会到达 window，它们会调用路径上所有的处理程序。
阻止传播有两个方法：（注意不只是阻止冒泡，捕获也会被阻止）

- `event.stopPropagation()`可以阻止冒泡和捕获；安插在任意想要阻止冒泡的处理函数上，但并不会阻止当前元素的事件触发，只会阻止向上的。
- `event.stopImmediatePropagation()`阻止冒泡和捕获，并顺便阻止当前元素事件的执行

```html
<body>
  <main>
    <button>clickme</button>
  </main>
</body>
<script>
  document.querySelector("main").addEventListener("click", (e) => {
    console.log(e); //不会触发，因为下面冒泡被拦截了
  });
  document.querySelector("button").addEventListener("click", (e) => {
    console.log(e);
    e.stopPropagation();
  });
</script>
```

## 事件捕获

DOM 事件标准描述了事件传播的 3 个阶段：

- 捕获阶段（Capturing phase）—— 事件（从 Window）向下走近元素。
- 目标阶段（Target phase）—— 事件到达目标元素。
- 冒泡阶段（Bubbling phase）—— 事件从元素上开始冒泡。

事件首先通过祖先链向下到达元素（捕获阶段），然后到达目标（目标阶段），最后上升（冒泡阶段），**在途中调用处理程序**。
![](https://pic.imgdb.cn/item/622afec75baa1a80abd69535.jpg)

上面说的`addEventListener`如果设置了参数 capture 为 true，则事件会在捕获阶段被触发。

## 事件委托

> 如果我们有许多以类似方式处理的元素，那么就不必为每个元素分配一个处理程序 —— 而是将单个处理程序放在它们的共同祖先上。

举个栗子:比如说现在有一个 3\*3 的单元格,现在想实现点击哪个单元格就让哪个变色

```js
let table = document.getElementById("tab");
table.addEventListener("click", (e) => {
  if (e.target.tagName !== "td") return;
  e.target.style.backgroundColor = "red";
});
```

另外，更高级的写法是通过一个类或对象创建一组事件处理，然后传入事件后在内部匹配并触发：

```html
<body>
  <div id="menu">
    <button data-action="save">Save</button>
    <button data-action="load">Load</button>
    <button data-action="search">Search</button>
  </div>
</body>
<script>
  class Menu {
    constructor(elem) {
      this._elem = elem;
      elem.onclick = this.onClick; //插入事件
    }
    save() {
      console.log("saving");
    }
    load() {
      console.log("loading");
    }
    search() {
      console.log("searching");
    }
    onClick = (event) => {
      let action = event.target.dataset.action;
      if (action) {
        this[action]();
      }
    };
  }
  new Menu(document.getElementById("menu")); //只需要new即可，在constructor中会插入事件。
</script>
```

## 浏览器默认事件

浏览器主要的默认行为有:

- 点击`<a href="#"></a>`会自动跳转,也就是改变 location 的 Url 信息
- form 表单的`submit`事件会触发服务器提交,路径会带上 form 的信息跳转到新页面
- 在文本上按下鼠标按钮并移动会选中文本
- 在 `<input type="checkbox">` 上的 click 会选中/取消选中的 input

想阻止默认事件有这几种方法:

- 在事件的 event 对象使用`e.preventDefault()`方法。利用实践委托也可以
- `return false`,但是只能用于`onxxx`事件
- `addEventListener` 的可选项 `passive: true`

### 阻止默认行为

阻止默认行为的两个方法：

1. 如果通过`on<event>`的形式触发，通过 return false 可以阻止。

> React 中的事件虽然也是这种形式，但是并不相同，只能通过`e.preventDefault()`来阻止。

注意这里 return 不是调用函数中在内部 return，而是直接在属性上 return
这样是无效的：

```html
<body>
  <form onsubmit="handleSubmit()">
    <button>submit</button>
  </form>
</body>
<script>
  function handleSubmit() {
    return false;
  }
</script>
```

> 无效的原因是，浏览器会根据其内容创建对应的处理程序，也就是说绑定的这个函数实际上外面又包了一层：
>
> ```js
> //onclick="handleSubmit()"其实是
> elem.onclick = function (event) {
>   handleSubmit();
> };
> ```
>
> 同时也可以看到，安插在 html 元素上的 event 应该写完整，只写一个`e`或者不写都不行。
>
> ```html
> <form onsubmit="handleSubmit(event)">
>   <!-- <form onsubmit="handleSubmit(e)"> 不可以-->
> </form>
> ```

应该这样：

```html
<form onsubmit="return handleSubmit()"></form>
```

或者直接：

```html
<form onsubmit="return false"></form>
```

或者这样也可：

```html
<body>
  <form>
    <button>submit</button>
  </form>
</body>
<script>
  document.querySelector("form").onsubmit = (e) => {
    return false;
  };
</script>
```

2. `addEventListener`只要拿到 event 对象，可以通过`e.preventDefault()`来阻止，而返回 false 无效。

## 自定义事件

https://zh.javascript.info/dispatch-events

有关传递数据的自定义事件（customEvent）：https://developer.mozilla.org/zh-CN/docs/Web/Events/Creating_and_triggering_events

# 网络请求

## fetch

### 基本语法

> fetch 不是 ajax 的进一步封装，而是原生 js，没有使用 XMLHttpRequest 对象。

```js
let promise = fetch(url, [options]);
```

- `url` —— 要访问的 URL。
- `options` —— 可选参数，是一个对象：method，header 等。

`options`是 fetch 的主要配置选项，用于设置请求方法、放置请求体、设置请求头等
比如：

- 设置 headers：

```js
let response = fetch(protectedUrl, {
  headers: {
    Authentication: "secret",
  },
});
```

- 设置 Post

```js
let response = await fetch("/article/fetch/post/user", {
  method: "POST",
  headers: {
    "Content-Type": "application/json;charset=utf-8",
  },
  body: JSON.stringify(user),
});
```

浏览器立即启动请求，并返回一个该调用代码应该用来获取结果的 promise；获取响应通常需要经过两个阶段：

1. **服务器发送了响应头，fetch 返回的 promise 就使用内建的 `Response class` 对象来对响应头进行解析**。

fetch 会返回一个 promise 对象，resolve 值是一个 response，主要属性有：

- `status`：HTTP 状态码，例如 200。
- `ok`：布尔值，如果 HTTP 状态码为 200-299，则为 true。
- `headers`：一个类似 Map 的 header 对象，可以迭代和使用 get 方法获取值：
- `body`：一个 `ReadableStream`，是一个读取流，可以用于访问读取进度

```js
let response = await fetch(
  "https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits"
);

// 获取一个 header
alert(response.headers.get("Content-Type")); // application/json; charset=utf-8

// 迭代所有 header
for (let [key, value] of response.headers) {
  console.log(`${key} = ${value}`);
}
```

如果请求失败，会 reject 一个异常状态码以及错误信息

2. **使用对 response 的处理方法获取`response.body`**

主要方法有：

- `response.text()` : 读取 response，并以文本形式返回 response，
- `response.json()` : 将 response 解析为 JSON
- `response.formData()` : 以 FormData 对象的形式返回 response，
- `response.blob()` : 以 Blob 形式返回 response，
- `response.arrayBuffer()` : 以 ArrayBuffer 形式返回 response，

这些方法都不会直接返回结果，而是返回一个 Promise 对象；因此仍然需要 await 或 then 调用获取结果。
方法只能调用一次，一次处理之后后续再解析也无效。

---

fetch 等常见的请求方法都会返回一个 Promise 对象等待后续处理。因此如果针对“**一组**”请求，更好的方式是使用`Promise.all`。
比如，一个请求一组用户信息的函数，可以这样写：

```js
let infos = [];
for (let name of names) {
  let res = await fetch(`/api/${name}`);
  let parse = await res.json();
  infos.push(parse);
}
```

这种方法相当于对每个 name 依次执行一个请求，当一个 name 的请求完成之前下一个请求并不会开始（上一个还在 await）。这样会导致请求速度很慢；
更高级的方式是把 fetch 请求的返回值放在一个数组中，然后对这个数组调用`Promise.all`方法。

```js
let infos = [];
names.forEach((name) => {
  let info = fetch(`/api/${name}`).then(
    (success) => {
      return success.status === 200 ? success.json() : null;
    },
    (fail) => {
      return null;
    }
  );
  infos.push(info);
});

let results = await Promise.all(infos);
```

通过将 `.json() `直接添加到每个 fetch 中，就能确保每个 fetch 在收到响应时都会立即开始以 JSON 格式读取数据，而不会彼此等待。
不仅是 fetch，其他方法也是同理。

### FormData

之前讲过用 FormData 对象捕获表单数据的方法。fetch 可以接受一个 FormData 对象作为 body，会自动设置`Content-Type: multipart/form-data`

```html
<body>
  <form>
    <input name="username" id="username" />
    <input name="password" id="password" />
    <button>submit</button>
  </form>
</body>
<script>
  const form = document.forms[0];
  form.addEventListener("submit", async (e) => {
    e.preventDefault();
    let formData = new FormData(form);
    let res = await fetch("", {
      method: "POST",
      body: formData,
    });
  });
</script>
```

formData 不能被显式查看，但可以通过一些迭代方法访问，详见 BOM 章。

## XHR

https://zh.javascript.info/xmlhttprequest

常用的封装 xhr：

1. xhr 需要先通过`open()`传入方法和 url
2. xhr 需要指定`responseType`和`setRequestHeader`，即指明响应格式和请求头
3. 准备好后通过 send 发送请求
4. 不需要`onreadystatechange`监听，因为现在已经拥有 onload 方法了

```js
function request(url, method = "GET") {
  const xhr = new XMLHttpRequest();
  return new Promise((resolve, reject) => {
    xhr.open(method, url);
    xhr.responseType = "JSON";
    xhr.setRequestHeader("Content-type", "application/json");
    xhr.send();
    xhr.onload = (e) => {
      resolve(xhr.response);
    };
    xhr.onerror = (err) => {
      reject(err);
    };
  });
}
```

### 主要事件

#### ProgressEvent

> ProgressEvent 接口是测量如 HTTP 请求（一个 XMLHttpRequest，或者一个 `<img>`，`<audio>`，`<video>`，`<style>` 或 `<link>` 等底层资源的加载）等底层流程进度的事件。

它常出现于 xhr 中 onload、onprogress 等事件的回调函数参数中，常用属性有：

- `ProgressEvent.type`：返回当前时间的
- `ProgressEvent.lengthComputable` 只读
  是一个 布尔值，表示底层流程将需要完成的总工作量和已经完成的工作量是否可以计算。换句话说，它告诉我们进度是否可以被测量。这一项为 true 后就可在 onprogress 事件中记录进度等信息。
- `ProgressEvent.loaded` 只读
  是一个 unsigned long long 类型数据，表示底层流程已经执行的工作总量。可以用这个属性和 ProgressEvent.total 计算工作完成比例。当使用 HTTP 下载资源，它只表示内容本身的部分，不包括首部和其它开销。
- `ProgressEvent.total` 只读
  是一个 unsigned long long 类型数据，表示正在执行的底层流程的工作总量。当使用 HTTP 下载资源，它只表示内容本身的部分，不包括首部和其它开销。

#### onload

onload 事件 —— 当请求完成（即使 HTTP 状态为 400 或 500 等），并且响应已完全下载。

onload 事件触发后，可以在 xhr.response 种获取到响应信息。这个值它的具体类型取决于 responseType 的值，常见的有 arraybuffer、blob、text、json、null 这几种。

onload 的触发不一定是响应成功，只是表示收到了响应，但是依旧可能存在 http 错误，比如 404、500 等错误的响应体可能为 null 或者错误信息。因此如果需要处理 http 错误，就需要在 onload 内部处理：

```js
xhr.onload = function () {
  if (xhr.status != 200) {
    // 分析响应的 HTTP 状态
    alert(`Error ${xhr.status}: ${xhr.statusText}`); // 例如 404: Not Found
  } else {
    // 显示结果
    alert(`Done, got ${xhr.response.length} bytes`); // response 是服务器响应
  }
};
```

onload 事件的回调有一个 event 参数，是 xhr 事件的 ProgressEvent 对象。在常见的几个 api 中都有这个对象的出现；

#### onerror

用于处理错误。
这里的错误是指请求期间出现的非 HTTP 错误，即无法发出请求，例如网络中断或者无效的 URL。
触发这个事件时并没有响应，比如跨域也会导致触发 onerror。也是因为没有响应，因此不能在其中使用 response、status 等参数。一般情况下，也只能在内部打印错误信息，或者在 promise 封装时 reject。

#### onprogress

这个事件在下载响应期间定期触发，报告已经下载了多少。
下载进度可以通过`ProgressEvent.loaded`/`ProgressEvent.total`得到

#### 其他

1. onreadystatechange：这个是旧代码中常用的事件，当它触发时，表明`xhr.readyState`发生了改变。这个值一共有 0、1、2、3、4 五种取值，分别表示：

```
UNSENT = 0; // 初始状态
OPENED = 1; // open 被调用
HEADERS_RECEIVED = 2; // 接收到 response header
LOADING = 3; // 响应正在被加载（接收到一个数据包）
DONE = 4; // 请求完成
```

实际上通过`XMLHttpRequest.DONE`的写法也可以表示这四个值。 2. onloadend：在一个资源的加载进度停止之后被触发 (例如，在已经触发“error”，“abort”或“load”事件之后) 3. onloadstart：当程序开始加载时，loadstart 事件将被触发。 4. ontimeout：如果设置了 xhr.timeout 的具体值，那么超时时则会触发这个事件。

### 主要属性

1. readyState：上面说过
2. response：XMLHttpRequest 的 response 属性返回响应的正文。返回的类型为 ArrayBuffer、Blob、Document、JavaScript Object 或字符串中的一个。这取决于请求的 responseType 属性。response 可以通过 responseType 改变类型；但也可以通过访问 responseText/responseXML 等形式明确获取希望的返回值类型。
3. status：一个数字，表示 http 状态码。如果 XMLHttpRequest 出错，浏览器返回的 status 也为 0
4. upload：用于检测上传事件的对象。绑定在 xhr 本身的事件都可以出现在 upload 上，通过这些事件就可以获取上传进度等信息。

- loadstart —— 上传开始。
- progress —— 上传期间定期触发。
- abort —— 上传中止。
- error —— 非 HTTP 错误。
- load —— 上传成功完成。
- timeout —— 上传超时（如果设置了 timeout 属性）。
- loadend —— 上传完成，无论成功还是 error。
  跟踪上传进度可以这样：

```js
function upload(file) {
  let xhr = new XMLHttpRequest();

  // 跟踪上传进度
  xhr.upload.onprogress = function (event) {
    console.log(`Uploaded ${event.loaded} of ${event.total}`);
  };

  // 跟踪完成：无论成功与否
  xhr.upload.onloadend = function () {
    if (xhr.status == 200) {
      console.log("success");
    } else {
      console.log("error " + this.status);
    }
  };

  xhr.open("POST", "/article/xmlhttprequest/post/upload");
  xhr.send(file);
}
```

5. withCredentials：用于设置跨域请求下是否发送 cookie。

### 主要方法

1. `xhr.setRequestHeader(header,value)`：用于设置响应头部，按照键值对的形式传入。如果要设置多个头就需要多次调用这个函数。一旦设置了 header，就无法撤销了。其他调用会向 header 中添加信息，但不会覆盖它。

```js
xhr.setRequestHeader("X-Auth", "123");
xhr.setRequestHeader("X-Auth", "456");

// header 将是：
// X-Auth: 123, 456
```

2. `getResponseHeader()/getAllResponseHeaders()`：获取指定的响应头和全部响应头，返回都是字符串。
3. `abort()`：中断请求

## WebSocket

> WebSockets 是一种先进的技术。它可以在用户的浏览器和服务器之间打开交互式通信会话。使用此 API 可以向服务器发送消息并接收事件驱动的响应，而无需通过轮询服务器的方式以获得响应。数据可以作为“数据包”在两个方向上传递，而无需中断连接也无需额外的 HTTP 请求。对于需要连续数据交换的服务，例如网络游戏，实时交易系统等，WebSocket 尤其有用。
> WebSocket 没有跨源限制。
> 基本方法：`socket.send(data)`/`socket.close([code], [reason])`
> 基本事件：`onopen`、`onmessage`、`onerror`、`onclose`

### Websocket 握手过程

https://developer.mozilla.org/zh-CN/docs/Web/API/WebSockets_API/Writing_WebSocket_servers

1. 客户端发起握手请求：客户端将发送一个相当标准的 HTTP 请求（HTTP 版本必须是 1.1 或更高，方法必须是 GET）：

```
GET /chat HTTP/1.1
Host: example.com:8000
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

在请求头中设置 upgrade 字段用于升级协议。`Sec-WebSocket-Key`将在下面用于服务端的` Sec-WebSocket-Accept`生成；

2. 服务端响应握手请求：当服务器收到握手请求时，它应该发回一个特殊的响应，表明协议将从 HTTP 变为 WebSocket。

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

Sec-WebSocket-Accept 参数需要服务器通过客户端发送的 Sec-WebSocket-Key 计算出来。把客户发送的 Sec-WebSocket-Key 和 "258EAFA5-E914-47DA-95CA-C5AB0DC85B11" (这个叫做 "魔法值") 连接起来，把结果用 SHA-1 编码，再用 base64 编码一次，就可以了。
这看起来繁复的处理使得客户端**明确服务端是否支持 WebSocket**。这是十分重要的，如果服务端接收到一个 WebSocket 连接但是把数据作为 HTTP 请求理解可能会导致安全问题。
一旦服务器发送正确 Accept 的请求头，握手就完成了，你可以开始交换数据！

3. 跟踪客户端：服务器将不得不跟踪客户的套接字，所以你不会再和已经完成握手的客户握手。同一个客户端 IP 地址可以尝试连接多次（但是如果客户端尝试过多的连接，服务器可以拒绝它们以免遭拒绝服务攻击

4. 交换数据帧：连接建立后，客户端或服务端都可以在任何时间点发送数据，数据的基本格式是二进制加密帧

websocket 有自己独特的状态码，在关闭连接时可以传递这些状态码，用于表示关闭原因等信息，详见https://zhuanlan.zhihu.com/p/145628937

### 基本使用

首先创建一个 WebSocket：

```js
let socket = new WebSocket("ws://xxx");
```

这个过程会尝试和该 url 建立连接，并返回一个 socket 对象。
一共有 4 个事件：

- `socket.onopen`：连接已建立，
- `socket.onmessage`：接收到数据，
- `socket.onerror`：WebSocket 错误，
- `socket.onclose`：连接已关闭。

使用`socket.send()`发送消息

```js
let socket = new WebSocket(
  "wss://javascript.info/article/websocket/demo/hello"
);

socket.onopen = (e) => {
  socket.send("My name is John");
};

socket.onmessage = (event) => {
  console.log(`[message] Data received from server: ${event.data}`);
};

socket.onclose = (event) => {
  if (event.wasClean) {
    console.log(
      `[close] Connection closed cleanly, code=${event.code} reason=${event.reason}`
    );
  } else {
    // 例如服务器进程被杀死或网络中断
    // 在这种情况下，event.code 通常为 1006
    console.log("[close] Connection died");
  }
};

socket.onerror = (error) => {
  console.log(`[error] ${error.message}`);
};
```

### WebSocket 连接

当 `new WebSocket(url)` 被创建后，它将立即开始连接。
首先会先发起一个 http 请求，确知服务器支持 ws 后会建立 ws 连接：
![](https://pic.imgdb.cn/item/622dce4d5baa1a80ab4ab5b7.jpg)

### 数据传输

WebSocket 通信由 “frames”（即数据片段，帧）组成，可以从任何一方发送，并且有以下几种类型：

- `text frames` —— 包含各方发送给彼此的文本数据。
- `binary data frames` —— 包含各方发送给彼此的二进制数据。
- `ping/pong frames` 心跳保活，浏览器会自动响应。当断线时可以通过心跳机制进行断线重连。
- `connection close frame` 以及其他服务 frames

send 方法不用设置数据类型，文本或二进制文件都可以

> 当我们收到数据时，文本总是以字符串形式呈现。而对于二进制数据，我们可以在 Blob 和 ArrayBuffer 格式之间进行选择。
> `socket.binaryType` 属性可以设置，默认为 blob，可以设置为`"arraybuffer"`

# 其他 js 基本知识点

## 变量

JavaScript 的变量命名有两个限制：

- 变量名称必须仅包含字母，数字，符号 `$` 和 `_`。
- 首字符必须非数字。

单独的一个`$`或`_`也可以作为变量名。

---

js 中的变量可以被任意互相赋值，变量间的赋值是一种拷贝；简单数据类型直接复制值，对象等复杂数据类型则复制引用

## `||`和`&&`和`??`

- `||` 可以在一排中找到第一个真值:`a || b` 表示 a 为假则判断 b;a 为真则直接用 a;
- `&&`可以找到第一个假值:`a && b` 表示 a 为假则返回 a,a 为真则返回 b;都为真返回最后一个;
- `??`和`||`差不多,但是`||`是判断真值,`??`是判断已定义的值，即 null 或 undefined ; 比如 0 虽然为假但是已定义就可以正常返回

三者的优先级：`&&` > `||` = `??`
但是 js 禁止同时使用`??`和另外两个，除非明确带上括号

```js
let x = 1 && 2 ?? 3; //error
let x = (1 && 2) ?? 3;
```

## this

### this 原理

> this 的设计目的就是在函数体内部，指代函数当前的运行环境。
> this 是和执行上下文绑定的，也就是说每个执行上下文中都有一个 this

https://www.ruanyifeng.com/blog/2018/06/javascript-this.html

### this 基本

- this:指向最近的 obj 对象
  - 直接用或者函数里用,表示 window
  - class 里表示这个类
  - 构造函数里表示实例对象(new 了以后)
  - 在事件中表示触发事件的对象,比如 click 事件的 this 就会是 button 元素
  - 对象中的方法:就是指这个对象
- this 的 bind apply call
  - bind:一般给函数使用,bind 一个变量,让函数内部的 this 指向这个变量
  ```javascript
  function f1() {
    return this;
  }
  let obj1 = {
    name: "zzx",
  };
  f1.bind(obj1); //调用的时候就是obj作为this的指向了
  ```
  关于 this 更详细的讲解参考这里：
  https://blog.poetries.top/browser-working-principle/guide/part2/lesson11.html#javascript-%E4%B8%AD%E7%9A%84-this-%E6%98%AF%E4%BB%80%E4%B9%88

### this 的绑定

this 主要有几种绑定方式：

1. 默认绑定，即函数内直接使用 this 或者全局状态下使用 this，得到的是 window 或 undefined
2. 显式绑定，即通过 call/bind/apply 方式显式地将函数内的 this 绑定到一个对象上
3. 隐式绑定，即对象调用形式`obj.func()`。当调用对象内的方法时，就相当于把函数内的 this 隐式绑定到了点符号前的对象上
4. new 绑定：即构造函数内部的 this 在 new 的过程中自动绑定到了构造出的实例对象上

这四种绑定的优先级：new 绑定 > 显式绑定 > 隐式绑定 > 默认绑定

其中后面三个的比较就不说了，这里说一下前两个的比较

在 bind 的实现中有一个这样的部分，就是判断返回的 fn 中的 this 是不是 fn 的实例，如果是就说明返回的 fn 是作为构造函数调用，因此应该绑定当前 this 而不是传入的 context：

```js
Function.prototype.mybind = function (context, ...args) {
  if (typeof this !== "function") return;
  context = context || window;
  const fn = this;
  function Fn() {
    return fn.apply(
      this instanceof Fn ? this : context, // 就是这一句
      args.concat(arguments)
    );
  }
  Fn.prototype = Object.create(fn.prototype);
  return Fn;
};
```

这里就可以得出 new 绑定优先级高于显示绑定的结论，因为显然如果通过 new 调用一个 bind 绑定后的函数，依然会以构造函数内部的 this 优先，最终绑定结果应该是实例，而不是 bind 传入的显示绑定对象。

```js
const obj1 = {};
function stu(age) {
  this.age = age;
}
const setStu = stu.bind(obj);
const newStu = new setStu(20);
setStu(18); // {age:18}
newStu; // {age:20}
```

### this 特点

0. this 本质上是运行环境的指代，或者说是和执行上下文绑定。它的设计目的就是在函数体内部，指代函数当前的运行环境。
1. this 只有在**执行中**才能确定。因此判断 this 指向应该是看在哪里执行，而不是在哪里定义。
2. 箭头函数没有自己的 this，由上一条，它内部的 this 是**运行时**的父环境的 this。注意是运行时才确定的。

### 从调用栈看 this

https://blog.poetries.top/browser-working-principle/guide/part2/lesson11.html#javascript-%E4%B8%AD%E7%9A%84-this-%E6%98%AF%E4%BB%80%E4%B9%88

### 常见的 this 丢失指向场景：

1. 对象或类内部定义一个含有 this 的方法，但是使用时没有点访问符

```js
let user = {
  name: "foo",
  getName() {
    return this.name;
  },
};

let getname = user.getName();
getname(); //this丢失

class User {
  //...
  getName() {
    return this.name;
  }
}

const getname = new User().getName;
getname(); //this丢失
```

2. setTimeout、setInterval 中的第一个参数，由于触发环境和正常的 js 调用栈完全不同（异步任务），因此内部 this 默认为 window。即使是严格模式，内部的 this 还是 window；
   当然有一种特殊情况，就是这个函数是箭头函数。这种情况下 this 仍然是外层的

```js
const testObj = {
  name: "test",
  hello() {
    console.log(this); // testObj
    setTimeout(() => {
      console.log(this); // testObj ，这里是外层的this。
    }, 0);
  },
  hi() {
    console.log(this); // testObj
    setTimeout(function () {
      console.log(this); // window，虽然是严格模式但还是window
    });
  },
};

testObj.hello();
testObj.hi();
```

3. 参数传递，因为传入一个函数时相当于赋值给了形参，可能会导致 this 丢失。
   最常见的情况就是装饰器：

```js
function foo() {
  console.log(this.name);
}
const obj = {
  name: "aaa",
  foo,
};
function decorateFoo(fn) {
  return function () {
    fn(arguments); //应当为fn.apply(this,arguments)
  };
}

const fooFoo = decorateFoo(obj.foo);
fooFoo(); //丢失指向
```

4. 箭头函数情况。如果对象中方法使用箭头函数、或者一个函数返回一个箭头函数，会使得 this 指向其父层，有可能并不存在 this。

```js
let obj = {
  name: "aaa",
  getName: () => {
    console.log(this);
  },
};
obj.getName();
```

这里使用箭头函数，由于没有自己的 this 上下文，**调用时**又没有外层代码块，因此直接指向全局（window 或 undefined）。

```js
function foo() {
  return () => {
    console.log(this.name);
  };
}
let obj2 = {
  name: "bbb",
};
const bar = foo();
bar.call(obj2); //undefined
```

由于 foo 函数中的 this 也是 window，因此 bar 函数即使绑定了 obj2 也没用。
箭头函数内部的 this 是指向外层代码块的 this（最近的 this，上例的 foo 函数），所以可以通过改变外层代码块的 this 的指向从而改变箭头函数中 this 的指向。
可以先给 foo 进行绑定 obj2，这样其内部的箭头函数中的 this 就会变成 obj2.

```js
const bar = foo.call(obj2);
bar(); //bbb
```

## bind apply call

### 解释

例子 1:

```js
let user = {
  name: "zzx",
  myFun: function () {
    return this.name;
  },
};
user.myFun(); //'zzx'
let admin = {
  name: "xiaoming",
};
user.myFun.apply(admin); //'xiaoming'
user.myFun.call(admin); //'xiaoming'
user.myFun.bind(admin)(); //'xiaoming'
```

这个过程发生了什么?
首先,正常调用 `user.myFun()`,this 的指向就是 user,也就是调用该方法的对象.但是我们新建了一个对象 admin,通过 `apply(admin)`把 admin 绑定到 myFun 的 this 上,让这个 this 指向不是 user 而是 admin

接下来例子 2 说明一下三者区别

```js
let user = {
  name: "zzx",
  myFun: function (a, b) {
    return a + " " + this.name + " " + b;
  },
};
user.myFun("hello", "welcome");
let admin = {
  name: "xiaoming",
};
user.myFun.call(admin, "hello", "welcome"); //'hello xiaoming welcome'
user.myFun.apply(admin, ["hello", "welcome"]); //'hello xiaoming welcome'
user.myFun.bind(admin, "hello", "welcome")(); //'hello xiaoming welcome'
```

由此区别就出来了:

- bind 使用后返回的是一个函数,因此需要加个括号调用;
- apply 和 call 在不传参时完全没区别;但是需要传递参数时,apply 传参是数组,而 call 是由逗号分隔的一个个参数
  总结一下:
- 这三个都是**对函数使用的**,用于绑定函数内部 this 指向的方法
- call 和 apply 只在传参上有区别,使用时相当于直接执行了函数;而 bind 是返回一个函数,可以 bind 之后用变量接住再调用

### 用法

例子 3:

```js
let user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  },
};
setTimeout(user.sayHi, 1000); // Hello, undefined!
```

这是一种常见的 this 指向丢失,原因是浏览器中的 setTimeout 方法有些特殊：它为函数调用设定了 `this=window`;使用时函数和对象分离开了(一般直接调用不会出现这个问题)
所以可以这样:

```js
let f = user.sayHi.bind(user);
setTimeout(f, 1000); //hello,John!
```

### bind 的特点

bind 可以绑定一个上下文和若干个参数，返回绑定后的结果，并不会直接调用。
bind 有几个特点：

1. bind 是“硬绑定”，一旦 bind 了一个对象，后续调用无论`.`前面是谁都不会变。

```js
function f() {
  alert(this); // ?
}

let user = {
  g: f.bind(null),
};

user.g(); //null，即bind绑定的
```

2. 一个函数不能被重绑定，一次 bind 之后不能再次 bind 给其他上下文

```js
function f() {
  clg(this.name);
}

f = f.bind({ name: "John" }).bind({ name: "Pete" });

f(); // John
```

#### 偏函数

bind 不仅可以绑定 this，还可以绑定参数。所谓绑定参数大概是这个意思：

```js
function mul(a, b) {
  return a * b;
}

const double = mul.bind(null, 2);
const triple = mul.bind(null, 3);
double(2); //4
triple(3); //6
```

可以看到通过 bind 把两个参数其中之一绑定为 2 或 3，后续只需要一个参数就可以；

> 偏函数应用程序（partial function application） —— 通过绑定先有函数的一些参数来创建一个新函数。

但是 bind 必须传入一个上下文；这里用不到 this，所以直接传入 null。

## new

当一个函数被使用 new 操作符执行时，它按照以下步骤：

1. 创建一个新的空对象，并让 this 指向这个新对象。
1. 函数体执行。通常它会修改 this，为其添加新的属性。
1. 返回 this 的值。

```js
function User(name) {
  // this = {};（隐式创建）

  // 添加属性到 this
  this.name = name;
  this.isAdmin = false;

  // return this;（隐式返回）
}
```

所以 `new User("Jack")` 的结果是相同的对象：

```js
let user = {
  name: "Jack",
  isAdmin: false,
};
```

### 构造函数和 new

默认情况下，所有函数都有 `F.prototype = {constructor：F}`，即每个函数都有一个指向自身的`constructor`。

如果一个函数类似这样：

```js
function myConst(name) {
  this.name = name;
}
```

可以让他绑定一个对象调用：

```js
function myConst(name) {
  this.name = name;
}
const obj = {};
myConst.call(obj, "zzx");
console.log(obj); //{name:'zzx'}
```

因此，如果有一个方法可以给这个函数传入一个空对象，并且返回，就可以实现所谓的“构造”
这个方法就是 new：

```js
function myConst(name /*,context={}*/) {
  //this = context
  this.name = name;
  //return context
}

const user = new myConst("xiaoMing");
```

> 实际上，因为每个函数都有 constructor，所以每个函数都可以被构造
> new 调用的不是函数本身，是函数的`prototype`属性上的`constructor`，而`constructor`一般指向自身，因此最后还是调用自己，但是前提是必须得有`constructor`（这也是箭头函数不能 new 的原因之一）
> new 在调用过程中传入一个空对象、进行绑定、调用该函数并最后返回值
> 因此构造函数能构造的关键不是函数，而是 new

### `new.target`

在一个函数内部，可以使用 `new.target` 属性来检查它是否被使用 new 进行调用了。
对于常规调用，它为 undefined，对于使用 new 的调用，则等于该函数：

```js
function User() {
  clg(new.target);
}

// 不带 "new"：
User(); // undefined

// 带 "new"：
new User(); // function User { ... }
```

### 构造函数的 return

- 如果 return 返回的是一个对象，则返回这个对象，而不是 this。
- 如果 return 返回的是一个原始类型，则忽略。

比如这个例子中由于 A 和 B 函数都返回 obj，因此两者 new 出来的实例是相同的，都是 obj 变量

```js
let obj = {};

function A() {
  return obj;
}
function B() {
  return obj;
}

clg(new A() == new B()); // true
```

这也就是在手写 new 的时候，需要判断构造函数是否显式返回了一个值，如果返回就采用这个返回值而不是创建的对象。

## 防抖和节流

### 防抖

> 函数防抖，就是指触发事件后，在 n 秒后只能执行一次，如果在 n 秒内又触发了事件，则会重新计算函数的执行时间。当持续触发事件时，一定时间段内没有再触发事件，事件处理函数才会执行一次，如果设定的时间到来之前，又一次触发了事件，就重新开始延时.

也就是说,一个动作连续触发但是只执行最后一次
上代码:

```js
let timeout = null;
function debounce(fn, wait) {
  clearTimeout(timeout);
  timeout = setTimeout(() => {
    fn;
  }, wait);
}
// 处理函数
function handle() {
  console.log("事件触发");
}
// 滚动事件
window.addEventListener("scroll", debounce(handle, 1000));
```

这段代码意思是:滚动停止后 1000 毫秒才会触发 handle.思路就是:在 debounce 函数里边设置一个定时器,当触发函数时不立即执行 fn,而是等 wait 秒后再执行;而如果调用时不停下来,**就会反复清除定时**,直到停下来为止再开始算
更好的方法是写成装饰器形式：

```js
function debounce(fn, wait) {
  let timeout;
  return function () {
    clearTimeout(timeout);
    timeout = setTimeout(() => {
      fn.apply(this, arguments);
    }, wait);
  };
}
//下面相同
```

### 节流

> 每隔一段时间只执行一次函数

就是说,判断两次操作间隔大于一定时间再执行(比如说一直拖着进度条就每一秒触发一次),所以可以有 两次时间戳相减 或 定时器 这两种实现方案

```js
//时间戳实现
function throttle(fn, wait) {
  let pre = 0;
  return function () {
    let now = Date.now();
    if (now - pre > wait) {
      fn.apply(this, arguments);
      pre = now;
    }
  };
}
function handle() {
  console.log("事件触发");
}
window.addEventListener("scroll", throttle(handle, 1000));
```

```js
//定时器实现
function throttle(fn, wait) {
  let isWait = false;
  return function () {
    if (isWait) return;
    else {
      isWait = true; //在间隔期把入口卡住
      setTimeout(() => {
        fn.apply(this, arguments);
        isWait = false;
      }, wait);
    }
  };
}
//剩下一样
```

解释一下定时器实现:首先 isWait 函数可以进入第一次触发;一旦进入 else 就把 isWait 改成 true,这样就是在定时器计时期间不再有任何可以进入 else 分支;定时器计时完毕,isWait 改为 false,又可以进入了.所以如果一直触发节流,相当于每个 wait 间隔期间就不会触发,而过了间隔期就允许触发

## requestAnimationFrame

### 原理

执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画。该方法需要传入一个回调函数作为参数，该回调函数会在浏览器下一次**重绘**之前执行。
`requestAnimationFrame()` 运行在后台标签页或者隐藏的`<iframe>` 里时会被暂停调用。
回调函数会被传入`DOMHighResTimeStamp`参数，也就是通过`performance.now()`获取的参数，表示 `requestAnimationFrame()` **开始去执行回调函数的时刻**。
这是一个比`Date.now()`更精确的时间戳（微秒级）；同一个帧的多个回调函数会被赋予相同的时间戳

### 使用

`requestAnimationFrame`（后面简称 rAF）的使用和`setTimeout`的递归调用类似。
比如这是一个使用 setTimeout 的简单动画：

```js
let times = 0;
function animation() {
  block.style.transform = `translateX(${times++}px)`;

  if (times < 500) {
    setTimeout(() => {
      animation();
    }, 16);
  }
}
setTimeout(animation, 0);
```

rAF 和它的原理类似，在函数内部递归调用该函数；但是 rAF 的回调会自动传入一个表示开始运行时间戳的变量（即`performance.now()`），这个值每次递归调用都会发生变化；因此可以利用该值（timeStamp）和最开始的时间（start）的差值，来确定渲染时间（`interVal < 2000`）

```js
let start = performance.now();
function animate(timeStamp) {
  let interVal = timeStamp - start;
  block.style.transform = `translateX(${interVal * 0.1}px)`;
  if (interVal < 2000) requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
```

常用的方法是，把这个函数封装成更通用的函数，参数主要有三个：

- timing 函数，即动画时序函数，是一个数学意义上的“函数”，可以修改这个函数来改变动画的运行方式
- duration：持续时间
- draw：渲染函数，一般是修改 style

范例如下：

```js
function animate({ timing, draw, duration }) {
  let start = performance.now(); //初始时间
  function animation(timeStamp) {
    let timeFraction = (timeStamp - start) / duration; //(当前时间 - 初始时间)/总时间，相当于进度

    if (timeFraction > 1) timeFraction = 1; //进度不能大于100%
    let progress = timing(timeFraction); //时序函数处理进度

    draw(progress); //根据进度绘制函数

    if (timeFraction < 1) requestAnimationFrame(animation); //进度<1，递归调用
  }
  requestAnimationFrame(animation); //注意这里调用的是内部这个函数，外部的函数只起到传递参数的作用。
}

const timing = (timeFraction) => timeFraction;
const draw = (progress) => {
  block.style.transform = `translateX(${progress * 500}px)`;
};
const duration = 5000;
animate({ timing, draw, duration });
```

### 时序函数

时序函数实际上就是对进度的特殊处理。
上面的例子原封不动返回，相当于是个正比例函数，运动是匀速的。
如果改成返回一个 2 次幂，就是一个初速度为 0 的匀加速运动：
![](https://pic.imgdb.cn/item/622ec4a35baa1a80aba5ba14.jpg)

```js
const timing = (time) => time ** 2;
```

其他时序函数：https://zh.javascript.info/js-animation

## 函数参数更改影响原变量的问题

如果参数是基本数据类型，函数内的参数实际上是该数据的复制，并不会影响原变量；
如果参数是引用数据类型，参数实际上是该变量的地址，因此修改该变量会影响原变量

```js
let a = 1;
let obj = {
    b : 0;
}

function change(a,obj){
    a = 2;
    obj.b = 1;
}

change(a,obj)
console.log(a,obj)// 1  {b:1}
```

换句话说，基本数据类型内部作用域下的改变不会影响外部作用域，但是引用数据类型由于持有的是同一个地址，因此会影响。

## 原始数据不可变

> 所有基本类型的值都是**不可改变的**。但需要注意的是，基本类型本身和一个赋值为基本类型的变量的区别。变量会被赋予一个新值，而原值不能像数组、对象以及函数那样被改变。

MDN 上的描述，所有的原始类型数据都是不可改变的。但是不可改变并不意味着持有该数据的变量不可改变，只是说这个数据本身不能改，当改动变量时实际上是给这个变量重新赋值。也就是说，**基本类型值可以被替换，但不能被改变**；改变的只是变量持有的值，而不是这个值本身

```js
1 = 2 // 无效，原始数据不能改动
'hello' = 'world' //同理
'hello'[0] = 'w' // 同理

let a = 1;
a + 1
console.log(a) // 1 ,直接改变原数据无效

a = a + 1 // 2 , 改变的是变量的值，而不是 1 这个数字本身

```

对应的，引用数据类型（Object）是可以改变的。可以把`Object`想象成一个盒子，我们可以向盒子内部装各种东西，改变盒子内部就会导致改变这个盒子的状态（虽然盒子本身并没有改变）

## || 和 && 的返回值

当符号两边是`true`或`false`时，返回值也是 boolean 值，会按照`与`和`或`返回

```js
true || false; //true
true && false; //false

1 > 2 || 2 == "2"; //true
```

当符号两边有一个是其他类型的值，就会返回两个值的中的一个。
原则是，`||`会在前值为假时返回后值，`&&`会在前值为真时返回后值，其余都是返回前值，**这种情况下返回的是具体的值，而不是布尔值**

```js
"" || 1; // 1
"" && 1; // ''
true || 1; // true
```

如果比较的一方是`undefined`或者`null`，则虽然这两者都为`falsy`，但是仍不会返回布尔值，而是可能返回`undefined`或者`null`

```js
null || undefined; // undefined
null && undefined; // null
undefined && undefined; //undefined
```

因此通过这种方式给变量赋值时，一定要保证符号两边都是确定的值，而不要是`undefined`，否则可能导致结果并不是想要的布尔值

```js
const getRequest = () => {
  let truthy;
  let falsy;
  setTimeout(() => {
    // 模拟异步请求
    truthy = true;
    falsy = false;
  }, 0);
  display(truthy, falsy);
};

const display = (truthy, falsy) => {
  let result = truthy && falsy;
  console.log(result);
};

getRequest(); // undefined
```

## for 循环和 let、var

### 同步情况

看一个最简单的例子：

```js
for (let i = 0; i < 2; i++) {}
console.log(i); // ReferenceError: i is not defined

for (var j = 0; j < 2; j++) {}
console.log(j); // 2
```

for 循环会产生多个块级作用域，每一次循环都会产生一个块；另外 for 循环的括号中是高于循环产生的块、但低于外部的作用域。

let 存在块作用域，因此每个`i`单独保存，每个块中的 i 独立，不会影响其他块中的 i。

```js
//let
{
  let i = 0;
  {
    let i = 1;
  }
  {
    let i = 2;
  }
}
console.log(i); // i is not defined
```

var 不存在块作用域，因此所有的`i`都是在一个 for 内部

```js
/* var i = undefined; */
{
  i = 0; // i会被变量提升到外层
  i++; // 没有块级作用域
  i++;
}
console.log(i); // 2
```

另外，for 循环是“不满足退出”，因此最后的值一定是不满足条件的值。比如条件是`i < 2`，那么输出值就会是第一个不满足条件的（即`i`为 2），而不是`i`等于 1

### 异步情况

一个很经典的题目是在 for 循环中启动定时器打印`i`

```js
for (let i = 0; i < 2; i++) {
  setTimeout(() => console.log(i), 1000);
}

for (var i = 0; i < 2; i++) {
  setTimeout(() => console.log(i), 1000);
}
```

输出为：一秒后同时输出 `0 1 2 2`

setTimeout 会被放入定时器线程，而同步线程的 for 循环则会被先执行完。因此当共计 4 个回调函数开始执行时，for 循环已经被执行完成了

按照块分解如下：

let 定义的`i`会被依次放入每个任务中，因为循环产生的块会持有自己的`i`；每个`i`是块中自己维护的，初始值为 0，不会影响其他块中的`i`

```js
{
  let i = 0;
  {
    setTimeout(() => console.log(i), 1000);
    i++; //这个i是这个块中自己维护的，初始值为0，不会影响其他块中的i
  }
  {
    setTimeout(() => console.log(i), 1000);
    i++; //这个i是这个块中自己维护的，初始值为1，不会影响其他块中的i
  }
}
```

相当于：

```js
{
  let i = 0;
  {
    let i_first = i; // 0
    setTimeout(() => console.log(i_first), 1000);
    i++;
  }
  {
    let i_second = i; // 1
    setTimeout(() => console.log(i_second), 1000);
    i++;
  }
}
```

var 定义的变量在`setTimeout`内部引用时相当于外部 `var i` 的闭包；因此在 for 内部更改 i，会导致`setTimeout`的`i`闭包值改变

```js
var i = undefined;
{
  i = 0;
  i++;
  setTimeout(() => console.log(i), 1000); //这里的i是外部 `var i` 的闭包
  i++;
  setTimeout(() => console.log(i), 1000);
}
```

如果在 for 执行之后再对`i`进行操作，可以看到定时器中的`i`也改变了

```js
for (var i = 0; i < 2; i++) {
  setTimeout(() => console.log(i), 1000);
}
i++;

// 1秒后输出2次3
```

---

对于 var 导致的问题，解决方法除了使用 let 之外还有几种：

1. 利用 IIFE(立即执行函数表达式)，每次 for 循环都是创建一个立即调用函数并传入当前 i 作为参数。这样每个调用就独立，外部的 i 改变不会影响传入的 i

```js
for (var i = 1; i <= 5; i++) {
  (function (j) {
    setTimeout(function timer() {
      console.log(j);
    }, 0);
  })(i);
}

// 相当于
var i = undefined;
{
  i = 0;
  i++;
  (function (j) {
    setTimeout(function timer() {
      console.log(j);
    }, 0);
  })(i); // 1
  i++;
  (function (j) {
    setTimeout(function timer() {
      console.log(j);
    }, 0);
  })(i); // 2
}
```

2. 利用 setTimeout 第三个参数，把变量传入。原理和上面差不多

```js
for (var i = 1; i <= 5; i++) {
  setTimeout(
    function timer(j) {
      console.log(j);
    },
    0,
    i
  );
}
```

## `valueOf`

这个方法很少会用到；
当把一个对象转化为原始数据类型时，会调用对象上的该方法。默认调用的是`Object.prototype.valueOf()`方法，但是如果对象内部重写了该方法，会返回重写方法的返回值。

```js
let obj = {
  valueOf() {
    return 9;
  },
};

console.log(1 + obj); // 10
```

调用对象 valueOf 方法的情况有：

1. 原始类型和该对象进行运算（`+`、`-`、`*`、`/`、`%`、`**`）
2. 用`==` / `>=` / `<=` / `>` / `<`把对象和原始数据类型进行比较

## 类的静态方法

静态方法属于类本身，因此 class 的方法里，前面有 static，那么方法里的 this 是**类本身**；前面没有 static，那么方法里的 this 是类的实例化对象；
因此静态方法只能通过类本身调用，不能通过实例调用
另外静态方法允许和一般方法重名，因为他们所挂载的对象完全不一样。

## let 和 const 的暂时性死区(TDZ)

let 和 const 存在暂时性死区，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。
并且，一个块中如果已经有了 let 或 const 声明的变量，就不会再理会外部的同名变量：

```js
var a = 1;
if (true) {
  a = 2; // ReferenceError
  /* 这一行之前的块内部都是暂时性死区 */
  let a = 3;
}
```

> 不可引用不止是赋值操作，`typeof`/`instanceof`以及任何其他相关作为参数的操作都不可以，即只要出现就是错误的。

原理在于，let 和 const 虽然不会像 var 一样变量提升，但是仍会在 js 解释阶段（即创建作用域阶段）先创建出该变量，然后在执行到正式声明那一行才可以使用。

## html 页面生命周期

基本生命周期顺序如下：

1. `script`：解析同步 js 脚本；如果 js 放在页面最下部，这一步将延迟到`DOMContentLoaded`之后执行
2. `readyState`：允许通过此事件监听 dom 加载状态，有三种情况；此时可能是`interactive`或`loading`
   - `interactive`：准备加载
   - `loading`：加载中
   - `complete`：加载完成，会在后面触发
3. `DOMContentLoaded`：DOM 加载事件，触发时浏览器已完全加载 HTML，并构建了 DOM 树，但像 `<img>` 和样式表之类的外部资源可能尚未加载完成。该事件只能通过`addEventListener`捕获。
4. `image onload`/`iframe onload`/`style onload`：加载上一步没有加载出来的外部资源，触发比如安置在`<img>`元素上的`onload`事件
5. `readyState`: complete，即此时所有文档和资源都已经加载完成
6. `window onload`：加载完成，此时已经完成了所有的加载工作，即启动的生命周期已经结束
7. `beforeunload`：用户正在离开：我们可以检查用户是否保存了更改，并询问他是否真的要离开。
8. `unload`：用户几乎已经离开了，但是我们仍然可以启动一些操作，例如发送统计数据。

解释如下：

1. script 同步的 JavaScript 脚本最先执行，它先于`DOMContentLoaded`事件执行。
2. 此时可以通过`onreadystatechange`事件监听 document 的状态，会有三种状态；因此它在 Dom 准备前、js 同步脚本执行后就触发
3. 当 DOM 准备就绪时，`DOMContentLoaded`事件在 document 上触发。 我们可以在这个阶段利用 JavaScript 来操作 DOM 元素。

4. 所有脚本都执行完毕，除了那些外部使用异步（async）或延迟（defer）加载的脚本，图片和其他资源可能仍在载入过程中。
5. window 上的 onload 事件，在页面加载完所有资源后触发。 我们很少使用它，因为通常的操作不用等到最后才执行。

`document.readyState`表示文档的当前状态，可以在`onreadystatechange`事件中跟踪文档状态的变更。

- `loading` – 文档正在载入。
- `interactive` – document 已经解析完毕时触发，几乎与`DOMContentLoaded`同时发生，但在`DOMContentLoaded`事件之前触发。
- `complete` – 文档和资源加载完成时触发，几乎与`window.onload`同时发生，但在`onload`事件之前触发。

一般来说，大多数的操作我们都应该放在 DOMContentLoaded 事件中执行，而不要放在 window.onload 中执行。

当用户离开页面时，会触发后两个事件：

- window 上的`beforeunload`事件，该事件在用户准备离开页面，在`unload`事件之前触发。 如果`beforeunload`返回一个字符串，浏览器会给出用户是否真的想离开的提示。
- window 上的`unload`事件，当用户最终离开时会触发该事件。在`unload`的事件处理程序中，我们只能做简单的事情，不涉及延迟或询问用户。由于这个限制，它很少使用。

## js 全局对象

js 中全局对象有两个概念：

1. 是指用户定义在**全局作用域**下的对象，即`var`定义的变量或者通过`function xxx()`定义的函数，会被自动挂载到 window 上；挂载之后可以通过`window.xx`或者`this`访问到。
2. js 的**标准内置对象**，并不是用户定义的，而是相当于 js 自带的一些对象和方法。实际上全局作用域包含内置对象

标准内置对象很多，可以参考https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects
这里说几个主要的：

1. 内置方法，可以直接调用，不需要在调用时指定所属对象，执行结束后会将结果直接返回给调用者。

- `eval()`
- `uneval()`
- `isFinite()`
- `isNaN()`
- `parseFloat()`
- `parseInt()`
- `decodeURI()`
- `decodeURIComponent()`
- `encodeURI()`
- `encodeURIComponent()`
- 以及已废弃的`escape()`和`unescape()`

2. 基本值属性，即：

- `Infinity`
- `NaN`
- `undefined`
- `globalThis`

3. 大多数不用定义直接引用的内置对象，比如所有数据类型的包装对象（`Number`、`Array`等），错误对象，ES6 新增的对象（`Promise`、`Reflect`、`Proxy`、`Map`、`Set`等）等。

> 注意，还有很多能直接使用的但是并非 js 内置对象，而是被挂载到了`window`上的。详见https://developer.mozilla.org/zh-CN/docs/Web/API/Window
> 常用的 window 上对象主要有：
>
> - `setTimeout/setInterval`
> - `console`
> - `alert`
> - `requestAnimationFrame`
> - `xhr`和`fetch`
> - 所有窗口大小相关的属性，比如`innerHeight`等
>
> 基本上可以理解为，直接引用的对象除了 js 内置对象，基本都在 window 上。

## delete

delete 操作符用于删除对象的某个属性，不能被删除的特例有：

- **`var`, `let`以及`const`创建的变量**
- 设置了`configurable:false`的属性
- 内置对象，不管是 window 提供的还是 js 内置的

会被删除的除了对象属性，还有几个特例：

- 非严格模式下未声明直接使用的变量，由于会被挂载到 window 上，因此会被直接删除
- `eval`中声明的变量
- 数组中的元素，删除后数组`length`并不改变，被删除的一项仍保留索引，但是值为`undefined`；

> 关于 delete 的返回值，对于所有情况都是 true，除非属性是一个自身的 不可配置的属性，在这种情况下，非严格模式返回 false，严格模式会报错

## 三种函数定义方式区别

1. 函数声明，又叫函数语句；创建的函数是一个 Function 对象，具有 Function 对象的所有属性、方法和行为。
   被声明的函数具有变量提升，可以在函数声明之前使用该函数；

```js
a();
function a() {}
```

这种函数没有块级作用域，因此在条件或循环等块中声明依旧会被提升，但是不能使用。

并且，函数声明的提升优先级大于变量提升；也就是说如果前面还有一个同名的变量，函数声明将会覆盖它（let 和 const 不行，因为实际上 a 已经被声明到了前面，不能重复声明）

```js
var a;
a(); // 'hello'
function a() {
  console.log("hello");
}

let a;
a(); // a is already been declared
function a() {
  console.log("hello");
}
```

> MDN：
> 这种声明方式在不同的浏览器里可能有不同的效果。因此，不应该在生产环境代码中使用这种声明方式，应该使用函数表达式来代替。

```js
foo(); // foo is not a function
console.log(foo); // undefined

if (false) {
  function foo() {
    return 1;
  }
}

// 改成true是完全一样的
if (true) {
  function foo() {
    return 1;
  }
}
```

2. 函数表达式，即声明一个变量是函数；函数表达式不会提升，所以不能在定义之前调用。
   从声明角度，函数表达式可以用`var`或`let/const`定义；这两种方式的主要区别仍是在块级作用域上，但在执行到该语句之前变量都是`undefined`，都不能使用。

```js
a(); //a is not a function
console.log(a); // undefined
var a = function () {};

a(); //a is not a function
console.log(a); // undefined
const a = function () {};

console.log(a); // undefined
if (1) {
  var a = function () {};
}

console.log(a); // a is not defined
if (1) {
  const a = function () {};
}
```

另外，上面说到函数语句方式定义的函数，无论条件语句是 true 还是 false 都一样，都不会被提升；
这点使用函数表达式也是一样，只是不会有浏览器差异，结果一律都是 undefined

```js
console.log(a); // undefined
if (true) {
  var a = function () {
    console.log("hello");
  };
}

console.log(a); // undefined
if (0) {
  var a = function () {
    console.log("hello");
  };
}

console.log(a); // Error: a is not defined
if (true) {
  let a = function () {
    console.log("hello");
  };
}
```

另外，这种定义方式都会把函数名直接和变量名统一，即不在`function`后再加名称；
加上也可以，被加上的函数名只能用于递归引用，不能在外调用：

```js
const Myfunc = function YourFunc() {
  //...
  YourFunc();
};

YourFunc(); // error
```

3. 构造函数创建，即`new Function()`，语法：

```js
let func = new Function([arg1, arg2, ...argN], functionBody);

let sum = new Function("a", "b", "return a + b");
//相当于
let sum = function (a, b) {
  return a + b;
};
```

该方法可以从字符串变为函数，几乎用不到；
这种方法创建的函数不能作为闭包，它内部的 outer 一定指向全局：

```js
function getFunc() {
  let value = "test";

  let func = new Function("alert(value)");

  return func;
}

getFunc()(); // error: value is not defined
```

## 函数参数作用域

函数有形参，形参会被添加到函数的作用域中，并且形参不会被重新定义（用 var 声明与形参同名的变量会被忽略，用 let 声明会报错），如果函数内声明一个和形参同名的函数，则内部声明的函数会覆盖参数

```js
function fun(arg1, arg2, arg3) {
  var arg1; // 声明被忽略
  var arg2 = "hello"; // var arg2 声明被忽略，arg2 = "hello" 被执行
  function arg3() {} // 函数声明会覆盖参数
  console.log(arg1, arg2, arg3);
}
fun(1, 2, 3); // 1 "hello" [ƒ arg3() {}]
```

实际上函数的参数是一个高于函数内部、低于外部的独立作用域，在函数被执行（传入具体参数）时确立其具体值

如果参数有默认值，则参数还会形成一个单独的作用域；

```js
var x = 1;
function fun(x, y = x) {
  // 这里的y默认值x会先从参数作用域中查找，即x是另一个参数，因此是2
  console.log(y); // y === 2
}
fun(2); // 2

var x = 1;
function fun(a, y = x) {
  // 内部作用域没有x，从外部找到x = 1
  console.log(y); // y === 1
}
fun(2); // 1

/* 默认参数和普通的参数一样，都是值的引用 */
var obj = { a: 1 };
function fun(x, y = x) {
  y.a = 3;
  console.log(x); // {a: 3}
}
fun(obj);
console.log(obj); // {a: 3}
```

如果默认参数是函数，则这个函数相当于在参数作用域内部，和变量的情况是一致的。

```js
let foo = "outer";
// 默认参数函数中的foo会从参数作用域先寻找，找到了foo变量为'other'，并且现在它是一个闭包
function bar(foo, func = () => foo) {
  console.log(func());
}
bar("other"); // other
```

## 会改变原数组和不改变的数组方法

https://juejin.cn/post/6844904192671219719

## `import *` 和 `export default`

当使用`import *`导入另一个模块中的全部导出变量时，会将所有导出的变量都合并为一个对象。比如下面的导出：

```js
// info.js
export const name = "Lydia";
export const age = 21;
export default "I love JavaScript";
// index.js
import * as info from "./info";
console.log(info);
```

输出是这样的：
![](https://pic.imgdb.cn/item/624a58e3239250f7c57ac84a.jpg)

因此可以通过`info.xxx`来获取具体的导出值，甚至可以通过`info.default`获取默认导出值。
还可以通过在具名导入中用`as`获取默认导出的值：
实际上`default`只是导出对象的一个特殊属性，只是不能显式使用而已。

```js
// info.js
export const name = "Lydia";
export const age = 21;
export default "I love JavaScript";
// index.js
import { default as string, name, age } from "./info";
console.log(string); // "I love JavaScript"
```

---

并且默认导出和具名导出也是可以共存的。
还是上面的例子：

```js
// info.js
export const name = "Lydia";
export const age = 21;
export default "I love JavaScript";
// index.js
import string from "./info"; // "I love JavaScript"
import { name, age } from "./info"; // "Lydia" 21
```

唯一的限制是，一个模块只能有一个默认导出

## 细碎知识点

1. `var a = b = xxx`形式的赋值，会导致除了第一个之外其他变量都变成全局变量（比如这里的 b）
2. `let`在全局作用域下声明的变量不会挂载到 window 上，但是`var`会
3. 函数的`arguments`属性是一个**可迭代对象**，并不是一个数组，因此没有数组方法，但是拥有`length`属性；此外可以通过`Array.from`把他变成一个真正的数组，或者直接使用`[...arguments]`展开
4. this 绑定优先级：`new`绑定优先级 > 显式绑定（`call` `bind` `apply`）优先级 > 隐式绑定（对象调用）优先级 > 默认绑定优先级
5. `let`和`const`不能对变量重复声明，这一点不仅体现在自己不能重复声明自己，也体现在不能声明`var`声明过的变量和同名函数

```js
var a = 1;
let a = 2; // 报错

function a() {}
let a = 1; // 报错
```

6. `call(null)`或`call(undefined)`都会将 this 指向 window（严格模式是 undefined）；`apply`和`bind`也一样
7. 如果使用`clearTimeout`清除定时器，已放入宏任务队列或者还在等待的回调都会被清除不再执行。
8. 在`==`中，`null` 和 `undefined` 除了和彼此比较以外，与其他任何类型操作数进行相等性测试都为 false
9. `class`和`let/const`一样都有暂时性死区和块级作用域，都不能提升。
10. `import`引入的变量是只读的，对于基本类型来说，不能修改值，修改会报只读的错误（相当于`const`）；但是引用类型可以修改，并且修改会引起源对象的值也发生改变。
11. 类中可以通过`#xxx`添加私有变量，在外部不可访问；注意这是确实存在的语法，而不是一种约定，当试图在外修改时会报错。同理 static 也是确实可以添加的静态字段和，而不只是一个约定
12. `Object.freeze()`可以对对象进行**浅**冻结（不能对值为对象的属性生效），不能对属性进行添加，修改，删除
13. 可选链操作符`?.`不仅可以访问普通的对象属性，还可以通过`arr?.[i]`形式访问数组或变量形式值；或者通过`obj.someFunc?.()`调用可能不存在的函数。

## 关于对象中定时器不会被回收的问题

这个问题来源于一个例子：

```js
let config = {
  alert: setInterval(() => {
    console.log("aaa");
  }, 1000),
};
config = null;
```

当 config 为 null 时，看起来 config 已经被回收了，但是定时器依旧每秒输出 1。
这个例子的原文下说的是“`setInterval`的参数是一个箭头函数（所以上下文绑定到对象 config 了），回调函数仍然保留着对 config 的引用”
但是这显然不对，因为替换成非箭头函数也是一样的：

```js
let config = {
  alert: setInterval(function () {
    console.log("aaa");
  }, 1000),
};
config = null;
```

甚至，根本就不是引用和垃圾回收的问题。当直接在对象的属性中调用一个函数时，即使只是在声明，都会直接调用该函数，并把返回值赋给这个属性。

```js
let config = {
  alert: setInterval(() => {
    console.log("aaa");
  }, 1000),
};

console.log(config) // {alert: 158} 这个值是Interval的id，显然是返回值


let config = {
  alert: console.log("aaa"); // aaa
};
// 即使没有config.alert，在定义时会直接执行 console.log
```

这时把 config 设为 null 确实回收了 config，但是并没有清除定时器，依旧在运行。

## node 查找模块顺序

![](https://pic.imgdb.cn/item/6248f91c27f86abb2a36aade.jpg)

查找优先级如下：

1. 缓存的模块优先级最高
1. 如果是内置模块，则直接返回，优先级仅次缓存的模块；
1. 如果是绝对路径 / 开头，则从根目录找
1. 如果是相对路径 ./开头，则从当前 require 文件相对位置找
1. 如果文件没有携带后缀，先从 js、json、node 按顺序查找
1. 如果是目录，则根据 `package.json`的 main 属性值决定目录下入口文件，默认情况为 index.js
1. 如果文件为第三方模块，则会引入 `node_modules` 文件，如果不在当前仓库文件中，则自动从上级递归查找，直到根目录

> 最后一步的查找优先级：
> 如果在`/home/ry/projects/foo.js`文件里调用了 `require('bar.js')`，则 Node.js 会按以下顺序查找：
>
> 1. `/home/ry/projects/node_modules/bar.js`
> 1. `/home/ry/node_modules/bar.js`
> 1. `/home/node_modules/bar.js`
> 1. `/home/node_modules/bar/index.js`；（如果上面不存在，说明可能是模块名错误，就会从默认按照 index.js 查找）
> 1. `/node_modules/bar.js`，即总根目录下的模块

## shadowDOM

shadowDOM 是指浏览器内置的样式构建，通常对开发者是隐藏的；想要查看需要特别在浏览器的开发者工具中开启查看 shadowdom

比如一个`<input type="range">`的内部样式实际上长这样：
![](https://pic.imgdb.cn/item/628798110947543129722127.jpg)

这时浏览器为其内部定制的样式，不能通过 js 获取。类似的例子还有滚动条等

shadowDOM 的特点：

- 有自己独立的样式和 dom，正常的 css 和 js 不能访问
- 只能通过在某个节点之下用`attachShadow`创建，并且可以选择是否挂载到 shadow tree 上。

一个 DOM 元素可以有以下两类 DOM 子树：

- Light tree（光明树）：一个常规 DOM 子树，由 HTML 子元素组成。我们在之前章节看到的所有子树都是「光明的」。
- Shadow tree（影子树）：一个隐藏的 DOM 子树，不在 HTML 中反映，无法被察觉。

调用 `elem.attachShadow({mode: 'open' | 'close'})` 可以创建一个 shadow tree；参数为 open 则可以通过`elem.shadowRoot`访问。
这个 api 有几个限制：

1. 每个元素中，只能创建一个 shadow root。
2. elem 必须是自定义元素，或者是以下元素的其中一个：「article」、「aside」、「blockquote」、「body」、「div」、「footer」、「h1…h6」、「header」、「main」、「nav」、「p」、「section」或者「span」。其他元素，比如`<img>`，不能容纳 shadow tree。

```html
<style>
  p {
    color: red;
  }
</style>

<div id="elem"></div>

<script>
  elem.attachShadow({ mode: "open" });
  elem.shadowRoot.innerHTML = `
    <style> p { font-weight: bold; } </style>
    <p>Hello, John!</p>
  `;

  // <p> 只对 shadow tree 里面的查询可见 (3)
  clg(document.querySelectorAll("p").length); // 0
  clg(elem.shadowRoot.querySelectorAll("p").length); // 1
</script>
```

> 如果想要更改浏览器自带的某些 shadowdom 的样式，比如滚动条，可以通过点开并查看这个元素上的`pseudo`属性，它的值是一个伪元素；通过给这个元素添加上伪元素限制，就可以更改这个样式了。
> ![](https://pic.imgdb.cn/item/628798110947543129722127.jpg)

## WebComponents

WebComponents 就是浏览器自己支持的组件化开发，用户可以自己创建组件，也可以继承一部分原组件并改进。
用户可以定义两种 custom element：

1. 全新定义，通常是一个类，继承自浏览器内置的 DOM 对象

```js
class MyElement extends HTMLElement {
  constructor() {
    super(); /* ... */
  }
  connectedCallback() {
    /* ... */
  } // 在元素被添加到文档之后，浏览器会调用这个方法（挂载）
  disconnectedCallback() {
    /* ... */
  } // 在元素被添加到文档之后，浏览器会调用这个方法（卸载）
  static get observedAttributes() {
    return [
      /* 属性数组，这些属性的变化会被监视 */
    ];
  } // watch
  attributeChangedCallback(name, oldValue, newValue) {
    /* ... */
  } // 当元素定义的属性发生变化，并且属于上面数组监听的范畴时候，这个方法会被调用
  adoptedCallback() {
    /* ... */
  } // 在元素被移动到新的文档的时候，这个方法会被调用
}
customElements.define("my-element", MyElement);
```

2. 已有元素扩展，通常表现为仍是原有元素，但是属性上有一个`is`，可以把一般元素变为自定义元素。

```js
class MyButton extends HTMLButtonElement {
  /*...*/
}
customElements.define("my-button", MyElement, { extends: "button" });
/* <button is="my-button"> */
```

## template

可以使用`<template>`封装一部分 HTML 元素，浏览器会保存其内部的 HTML 元素排列。但是，样式不会被应用，脚本也不会被执行， `<video autoplay>` 也不会运行，相当于只是在单纯的保存。
它的一个重要应用是，可以把其中的内容完整的插入到一个元素中，而不只能通过字符串拼接的形式：

```html
<template id="tmpl">
  <script>
    alert("Hello");
  </script>
  <div class="message">Hello, world!</div>
</template>

<script>
  let elem = document.createElement("div");

  elem.append(tmpl.content.cloneNode(true));

  document.body.append(elem);
</script>
```

当把 template 元素插入节点时，实际上插入的是其内部的子节点（`template.content`值）；插入之后将会类似正常的 HTML 一样去解析。

可以把一些 shadowdom 的样式在 template 中封装好，然后插入到元素的`elem.shadowRoot`中

## Math.random 的安全问题

`Math.random()` 函数返回一个浮点数, **伪随机数**在范围(0, 1), 其生成的不能提供像密码一样安全的随机数字（黑客可以计算出客户端生成的的随机数）。
当程序在需要不可预测性的上下文中生成可预测的值时，攻击者可能会猜测将要生成的下一个值，并使用该猜测来冒充另一个用户或访问敏感信息。

可以使用 Web Crypto API 来代替, 和更精确的`window.crypto.getRandomValues()`

```js
// crypto需要考虑浏览器兼容
const crypto =
  window.crypto ||
  window.webkitCrypto ||
  window.mozCrypto ||
  window.oCrypto ||
  window.msCrypto;
crypto.getRandomValues(new Uint32Array(1))[0];
```

## 数组的空值

JavaScript中数组空位指的是数组中的empty，其表示的是在该位置没有任何值，而且empty是区别于undefined的，同样empty也不属于Js的任何数据类型。

在JavaScript的数组是以稀疏数组的形式存在的，所以当在某些位置没有值时，就需要使用某个值去填充。当然对于稀疏数组在各种浏览器中会存在优化的操作，例如在V8引擎中就存在快数组与慢数组的转化，此外在V8中对于empty的描述是一个空对象的引用。在Js中使用Array构造器创建出的存在空位的问题，默认并不会以undefined填充，而是以empty作为值

```js
console.log([,,,]); // (3) [empty × 3]
console.log(new Array(3)); // (3) [empty × 3]
console.log([undefined, undefined, undefined]); // (3) [undefined, undefined, undefined]
console.log(0 in [undefined, undefined, undefined]); // true
console.log(0 in [,,,]); // false // in 是检查索引 此处表示 0 位置是没有值的
```

一些方法对数组空值的处理：

- 跳过空位：forEach、for in、filter、every、some。map会跳过对空位的函数执行，但会保留空属性的占位
- 字符串：join和toString会将空位与undefined以及null处理成空字符串
- 转换：Array.form和扩展运算符会把空位转换成undefined，includes、entries、keys、values、find和findIndex等会将空位上的值视为undefined，for of和for let i循环也会遍历空位并将值作为undefined。



## MouseEvent上的各种和鼠标相关的值

1. clientX、clientY：鼠标位置相对于整个页面的左上角，不会受到页面滚动影响。即以页面左上角为原点，鼠标点击时的坐标。
2. offsetX、offsetY：鼠标位置和元素的padding在x、y方向上的偏移量。具体来说就是，一个元素监听鼠标事件，然后如果鼠标在其内部，那么这个值就是以当前元素的左上角为原点的位置。相当于使用这个值可以直接判断类似canvas的鼠标点击坐标。

```js
canvas.addEventListener("mousedown", (e) => {
  const mouseX = e.offsetX;
  const mouseY = e.offsetY;
  ctx.moveTo(mouseX, mouseY);
});
```

3. pageX、pageY：也是基于文档左上角，但是如果有滚动时，要考虑到滚动的位置。比如一个页面高度为1000px，实际显示500px，滚动了500px。那么这时如果点击可视区域的左上角，pageY的值是500，因为它基于原本的左上角计算的。

# IntersectionObserver




# 滚动smooth动画

通过scrollTo方法滚动时，实现类似平滑滚动的效果。
如果浏览器支持，那么直接使用api即可：

```js
window.scrollTo({
  top: pos,
  behavior: 'smooth'
})
```

如果不支持，那么需要手动实现。基本方法就是通过rAF控制，每次通过rAF传入的currentTime和起始时间计算出progress，然后移动progress段距离，直到移动到目标坐标。

```js
let start = null
time = time || 500
window.requestAnimationFrame(function step (currentTime) {
  start = !start ? currentTime : start
  if (currentPos < pos) {
    const progress = currentTime - start
    window.scrollTo(0, ((pos - currentPos) * progress / time) + currentPos)
    if (progress < time) {
      window.requestAnimationFrame(step)
    } else {
      window.scrollTo(0, pos)
    }
  } else {
    const progress = currentTime - start
    window.scrollTo(0, currentPos - ((currentPos - pos) * progress / time))
    if (progress < time) {
      window.requestAnimationFrame(step)
    } else {
      window.scrollTo(0, pos)
    }
  }
})
```
