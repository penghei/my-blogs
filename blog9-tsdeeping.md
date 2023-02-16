---
title: typescript深入学习
date: 2022-02-06 13:44:25
tags: 日常学习
categories: TypeScript
cover: /img/typescript.png
---

# 基本类型补充

### 元组

相当于类型的数组

```ts
type tuple = [string, number];
let x: tuple;
x = ["hello", 10]; // OK
x = [10, "hello"]; // Error
```

元组和 js 的数组类似，实际上通过一些骚操作，也可以实现类似数组的 pop/push/shift 等功能：

```ts
// 取T的序号，在S中取值。比如T中有两个元素，结果就是S中的前两个元素
type ReplaceValByOwnKey<T, S extends any> = { [P in keyof T]: S[P] };

// 利用函数剩余参数，把第一个参数提出来，剩余的给infer R，然后返回R就是除去第一个之后的数组
type ShiftAction<T extends any[]> = ((...args: T) => any) extends (
  arg1: any,
  ...rest: infer R
) => any
  ? R
  : never;

// 同样是利用参数，把要插入的放在第一个参数上，最后返回整个args
type UnshiftAction<T extends any[], A> = ((
  args1: A,
  ...rest: T
) => any) extends (...args: infer R) => any
  ? R
  : never;

// 和shift刚好相反，因此用shift之后的数量取；
// 比如shift之后是去掉首位的2个，那么pop就相当于去掉末尾的的2个，也就是只取0 1号
type PopAction<T extends any[]> = ReplaceValByOwnKey<ShiftAction<T>, T>;

// 看不懂呜呜呜
type PushAction<T extends any[], E> = ReplaceValByOwnKey<
  UnshiftAction<T, any>,
  T & { [k: string]: E }
>;

// test ...
type tuple = ["vue", "react", "angular"];

type resultWithShiftAction = ShiftAction<tuple>; // ["react", "angular"]
type resultWithUnshiftAction = UnshiftAction<tuple, "jquery">; // ["jquery", "vue", "react", "angular"]
type resultWithPopAction = PopAction<tuple>; // ["vue", "react"]
type resultWithPushAction = PushAction<tuple, "jquery">; // ["vue", "react", "angular", "jquery"]
```

元组还可以通过直接规定索引类型的形式一次取出所有类型成为一个联合类型；这点也适合对象的类型别名

```ts
type tuple = ["vue", "react", "angular"];
type tupleVal1 = tuple[number]; // "vue" | "react" | "angular"
type tupleVal2 = tuple[string]; //报错：tuple中没有string索引的值
type tupleVal3 = tuple[any]; // "vue" | "react" | "angular"
```

### never 类型

never 出现于：返回 error 或者死循环函数，或者其他一定不会被访问到的
比如最常见的死循环函数或者抛出错误的函数：

```ts
function infiniteLoop(): never {
  while (true) {}
}

function createError(mess: string): never {
  throw new Error(mess);
}
```

### 类型断言

类型断言除了 as，还可以用尖括号的形式

```ts
let someValue: any = "this is a string";
let strLength: number = (<string>someValue).length;
```

但是容易和 jsx 混淆， 因此不建议使用

# 接口

接口可以理解为一个对象的类型，当然他不仅可以给对象注解，还可以是函数、数组等，但不能用于基本类型的注解。

### 数组

接口可以对数组使用。当用接口约束数组时，其实是把数组看作是一个对象，因此还可以同时约束数组的其他属性和方法。
如果想要给数组添加某种方法或重写方法，接口约束的方式可能很有用

```ts
interface MyNums {
  [index: number]: number; // 这里的index可以是string或number
  length:number;
}

const nums: MyNums = [1, 2, 3];
```


### 方法和函数

注意接口中的方法和函数定义相似但不同

```ts
//方法，相当于一个对象内部
interface Animal{
    eat(food:string):string;
}
const animal:Animal = {
    eat(food){
        return food;
    }
}

//函数，一般只能有一个，除非是函数重载
interface IFunc{
    (arg1:string,...args:number[]):number;
}
const getSum:IFunc = (arg1,...args)=>{...}
```

### 任意索引签名

任意索引签名指的是这种形式：

```ts
interface IValue {
  name: string;
  password: number;
  [prop: string]: any; //可以添加任意属性
}
```

- `[prop: string]: any`表示键为字符串、值为任意的属性；当需要给对象添加原先不存在的属性时需要声明这种属性，表示一个任意的属性。

```ts
const userInfo: IValue = {
  name: "zzx",
  password: 123,
  id: "0123456", //原先不存在的属性
};
userInfo.age = 18; //添加属性
console.log(userInfo);
```

- 另外，该种属性的值类型必须包括所有值的类型，比如`[prop: string]: string`会报错， 因为没有包括 number，因此一般直接用 any。

```ts
interface IValue {
  name: string;
  password: number;
  [prop: string]: string; //报错
}
```

### 用索引取得接口中的属性

接口可以通过`[propsName]`的形式取单个接口，还可以嵌套；

- 接口中只有 symbol/number/string 三种类型键，因此取值也是这三种方式；
- number 类型不会按顺序索引，比如`[0]`表示键为数字 0 的值，而不是第一项。
- type 也是同理，与之类似的还有元组、联合类型

```ts
interface IProps {
  method: string;
  msg: IProps["method"];
  0: number;
  1: IProps["0"];
  2: IProps[0];
}

type TProps = {
  method: string;
  msg: TProps["method"];
  0: number;
  1: TProps["0"];
  2: TProps[0];
};

type UProps = "a" | "b" | "c";
type UProps1 = UProps["0"]; //type UProps1 = string
type UProps2 = UProps[0]; //type UProps1 = string
```

### 类和接口

类可以实现(implements)接口，类似 java 的接口；

```ts
interface ICar {
  speed: number;
  canRun(): any;
}

class Car implements ICar {
  speed: number;
  constructor(s: number) {
    this.speed = s;
  }
  canRun() {
    console.log(this.speed);
  }
}
```

注意类其实是分作**静态部分**和**实例部分**两个部分的，前者包括 static 定义的方法和变量以及构造函数，后者是其他部分；接口只能指定类的实例部分。

---

类本身也是一种类型，因此接口其实可以继承一个类

```ts
class Control {
  private state: any;
}

interface SelectableControl extends Control {
  select(): void;
}
```

接口继承类相当于是继承类的成员但不包括其实现；也就是只声明类的成员，但不实现。
并且如果接口继承的类中有 private 或 protected 类型，那么这个接口只能被原先类的子类实现（implements）

```ts
class Fruit {
  private weight: number;
}
interface IFruit extends Fruit {
  eat(): void;
}
class OtherClass implements IFruit {} //报错，因为OtherClass不是子类

class Apple extends Fruit implements IFruit {
  //这样才可以
  eat() {
    //...
  }
}
```

### 混合类型

可以利用接口，使得一个变量既可以是对象又可以作为函数调用

```ts
interface Counter {
  (start: number): string;
  interval: number;
  reset(): void;
}

function getCounter(): Counter {
  let counter = function (start: number) {} as Counter; //这里是关键，把函数断言为Counter接口，使得定义的counter是Counter类型的
  counter.interval = 123;
  counter.reset = function () {};
  return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```

# 类

### 类的 getter 和 setter

类也可以使用 getter 和 setter 来控制值的读写

```ts
class Person {
  private _fullName: string;
  get fullName(): string {
    return this._fullName;
  }
  set fullName(name: string) {
    if (name.length < 2) {
      console.log("wrong");
      return;
    }
    this._fullName = name;
  }
}
const xiaoming = new Person();
xiaoming.fullName = "aa"; //wrong
```

### 抽象类和方法

类似面向对象的语言，抽象类不能直接实例化, 必须要先继承；抽象方法没有具体方法体,必须继承后才能写。

```ts
abstract class Car {
  //抽象类不能直接实例化, 必须要先继承
  speed: number;
  abstract move(): void; //抽象方法没有具体方法体,必须继承后才能写
  constructor(s: number) {
    this.speed = s;
  }
}

class BMW extends Car {
  constructor(s: number) {
    super(s);
  }
  move(): void {
    console.log(`move with ${this.speed}kph`);
  }
}
```

### 类当作接口

> 类定义会创建两个东西：类的实例类型和一个构造函数。 因为类可以创建出类型，所以你能够在允许使用接口的地方使用类。

也就是说，类可以充当一个类似接口的效果，可以像上面接口中所述的给接口继承，或者直接给实例做注解

```ts
class Point {
  x: number;
  y: number;
}

interface Point3d extends Point {
  z: number;
}

let point3d: Point3d = { x: 1, y: 2, z: 3 };
```

# 函数

### 函数类型声明

函数声明有两种

- 一种是冒号型,声明被视作一个属性,需要在 interface 或者 type 的对象中

```ts
interface Ifunc {
  (num: number): number;
}
type Tfunc = {
  (num: number): number;
};
```

- 另一种是箭头型,被视作函数,必须是 type,可以理解为函数类型变量

```ts
type TFunc = (num: number) => number;
```

接口中含有名字的函数会被认作是对象中的方法；由于接口可以合并，因此同名不同参数和返回值的函数会被当作一个函数的重载。

```ts
interface Cloner {
  clone(animal: Sheep): Sheep;
}

interface Cloner {
  clone(animal: Dog): Dog;
  clone(animal: Cat): Cat;
}

const myCloners:Cloner{
  clone(animal:Sheep|Dog|Cat){
    //...
  }
}
```

### 可选参数

可选参数会被自动设置类型为`xxx|undifined`

```ts
function buildName(fname: string, lname?: string): string {
  if (lname) return `${fname} ${lname}`;
  else return fname;
}

buildName("wang");
buildName("ming", "xiao");
```

使用 es6 的默认值语法也可起到相应的效果

---

除了使用可选参数，使得函数有任意个参数的方法还可以用剩余参数的方法

```ts
function addAll(num: number, ...restNumber: number[]): number {
  if (!restNumber.length) {
    let sum = 0;
    for (let num of restNumber) {
      sum += num;
    }
    sum += num;
    return sum;
  } else {
    return num;
  }
}

addAll(1, 2, 3, 3, 4, 5, 6);
```

另外，如果为了对不同参数执行不同操作，就需要函数的**重载**

### 函数重载

重载相当于是给一个函数多种参数情况,根据你调用时的参数情况,在最后的实现中具体操作；
如果函数的返回值类型相同，那么就不需要使用函数重载。
最后函数的实现实际上并不算一个重载,但必须要把精确的定义放在前面，需要使用 `|`操作符或者`?:`操作符，把所有可能的输入类型全部包含进去。
比如下面这个例子：

```ts
function padding(all: number): void;
function padding(topAndBottom: number, leftAndRight: number): void;
function padding(
  top: number,
  right: number,
  bottom: number,
  left: number
): void;
function padding(a: number, b?: number, c?: number, d?: number) {
  if (!b && !c && !d) {
    b = c = d = a;
  } else if (!c && !d) {
    c = a;
    d = b;
  }
  //...
}
padding(1);
padding(1, 2);
padding(1, 2, 3, 4);
```

在这个例子中，padding 可以有 1、2、4 个参数，不同参数对应的类型是不同的，比如一个参数表示全部，两个参数表示上下和左右，以此类推；  
但是单独的函数重载不能实现执行效果，实际的代码还应该在最后一个函数中写明，并且要考虑之前各种参数的情况；最后一个**不是重载**，是函数的具体实现  
比如上面代码中最后一个函数，需要有以下几点：

1. 至少有一个参数为必须，其他为`?:`表示的可选参数；也可以都是不必须，主要依赖于重载的制定；
2. 要考虑到不同参数数量的情况；上面函数分别处理了参数为 1 个（全赋值为 a）和参数为 2 个（c=a,d=b）的情况
3. 执行具体代码

另外，最后函数实现必须要指明返回值，重载部分可以不需要

### ts 中的 this

ts 会自动指出可能会丢失 js 指向的问题，一般的解决方案是箭头函数。
比如下面一个经典的例子：

```ts
const userInfo = {
  name: "zzx",
  age: 19,
  createUserCard() {
    return function () {
      return `${this.name} and ${this.age}`;
    };
  },
};
const cardPicker = userInfo.createUserCard();
console.log(cardPicker());
```

如果在 js 中上面的例子会返回 undefined，因为单独调用`cardPicker`时 this 没有具体指向，会为 undefined；  
但是 ts 会在 this 上报错（需要 tsconfig.json 的`--noImplicitThis`配置），告知你这里的 this 会被该函数隐藏，也就是相当于单独调用该 this 指向 window（严格模式下是 undefined）  
因此可以这么写：

```ts
const userInfo = {
  name: "zzx",
  age: 19,
  createUserCard() {
    return () => {
      return `${this.name} and ${this.age}`;
    };
  },
};
```

因为箭头函数中的 this 默认指向上一层，也就是这个对象，因此不再会出错；

---

但是此时 this 仍然是 any 类型，没有语法提示；ts 对于 this 提供了一个强大的功能：把 this 当作一个参数，指明 this 的类型

```ts
interface IUser {
  name: string;
  age: number;
  createUserCard(): () => string; //返回值为返回string的函数
}

const userInfo: IUser = {
  name: "zzx",
  age: 19,
  createUserCard(this: IUser) {
    //规定this的类型,把this视作一个参数,类型就是this本来所指的当前这个对象
    return function () {
      //这里不用箭头函数的话this会报错,因为如果调用该函数会使this失去指向
      return `${this.name} and ${this.age}`;
    };
  },
};
```

这样 this 就被说明清楚了，是指该对象本身。

# 泛型

ts 泛型使得函数可以在调用时明确指定类型, 而不是一个 any 了事。

### 泛型和函数

最基本的泛型函数：

```ts
function echo<T>(arg: T): T {
  //这里的T实际上是一种"类型变量"
  return arg;
}

let echo1 = echo<string>("hello");
let echo2 = echo<number>(11);
let echo3 = echo(() => {
  console.log("hello");
}); //可以自己推断
```

泛型可以理解为一种“类型变量”或者“类型参数”，类似于函数中的形参，调用时传入实参（具体类型），应用于各个类型注解上。

泛型函数的声明和函数类型声明类似，表示泛型的简括号被置于表示函数的括号前面，其他的相同：

```ts
//泛型可以用type声明, 类似函数声明, 但是没有名字
type GenericsType = <T>(arg: T[]) => T[];
type GenericsType<T> = (arg: T[]) => T[];

type GenericsTypeObj = {
  <T>(arg: T[]): T[];
};
interface IGenerics {
  <T>(arg: T[]): T[];
}
```

### 泛型接口

可以在接口上定义泛型，对函数来说类型别名也可以

```ts
interface IGenericsFunc<T> {
  (arg: T[]): T;
}
//or
type genericsType<T> = (arg: T[]) => T;

const echoOneFromList: IGenericsFunc<number> = (args) => args[0];
```

通过接口泛型可以更灵活使用接口。
除了泛型接口，类似的还有泛型类，但是没有泛型枚举和泛型命名空间

### 泛型类

```ts
class Queue<T> {
  private data: T[] = [];
  push = (item: T) => this.data.push(item);
  pop = (): T | undefined => this.data.shift();
}
const queue = new Queue<number>();
queue.push(1);
let x = queue.pop();
```

### 泛型约束

让泛型继承自某个接口来约束泛型, 相当于给泛型设置了默认值。
比如这个例子：

```ts
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}
loggingIdentity([1, 2, 3, 4]);
loggingIdentity({ value: 1, length: 2 });
```

这里规定泛型必须有 length 属性, 可以是一个数组, 或有 length 属性的对象。
也可以让泛型继承自一个类型别名，核心思路不变，就是把泛型必须的属性写在类型中。

---

甚至还可以规定默认值，就和真正的函数一样：

```ts
type foo<T extends string = "hello world"> = T;
//相当于
function foo(T: string = "hello world") {
  return T;
}

let bar: foo;
//bar === 'hello world'
```

# 类型兼容

ts 类型兼容是指，两个变量相互赋值、或者在函数参数中传递时会检查其兼容性。

> TypeScript 里的类型兼容性是基于结构子类型的。 结构类型是一种只使用其成员来描述类型的方式。 它正好与名义（nominal）类型形成对比。（在基于名义类型的类型系统中，数据类型的兼容性或等价性是通过明确的声明和/或类型的名称来决定的。这与结构性类型系统不同，它是基于类型的组成结构，且不要求明确地声明。）

比如 java 或 c 语言中，类型是明确指明的，不同类型不能赋值；但是 ts 是依据结构的，尤其是对于对象、数组这类非简单变量，只要结构有兼容性（比如属性互相包含）就有可能被互相赋值。

### 变体

#### 类型系统的父子关系

了解变体之前应该首先了解类型系统的父子关系。
子类型可以通过继承后继续添加新的属性，因此子类型的属性比父类型更多，更具体；在类型定义中**属性更多的类型是子类型**。

- 在类型系统中，属性更多的类型是子类型。
- 在集合论中，属性更少的集合是子集。

**子类型比父类型更加具体**，这点很关键。注意是**具体**，而不一定是多的。

#### 赋值

先看一个例子：

```ts
type Animal = {
  eat: boolean;
  sleep: boolean;
};

type Dog = {
  eat: boolean;
  sleep: boolean;
  weight: number;
};

let animal: Animal;
let dog: Dog;

animal = dog; // ✅ok
dog = animal; // ❌error!
```

animal 是一个「更宽泛」的类型，它的属性比较少，所以更「具体」的子类型是可以赋值给它的，dog 上拥有 animal 所拥有的一切类型，赋值给 animal 是不会出现类型安全问题的。

也就是说，多的属性类型可以赋值给少的，但是少的不能赋值给多的；从可赋值性角度来说，子类型是可以赋值给父类型的，也就是 `父类型变量 = 子类型变量` 是安全的，因为子类型上涵盖了父类型所拥有的的一切属性。

这个特点叫做**协变**，即多可以赋给少，但是反过来不行。

> 一种常见的泛型写法`<T extends {}>`也就是这个意思，这个约束了传入的参数一定是`{}`的子类型，也就是比`{}`要多任意属性的类型，比空多一个属性都算，即任何类型都可以。

但是对于联合类型，这个情况恰恰相反：

```ts
type Parent = "a" | "b" | "c";
type Son = "a" | "b";

let parent: Parent;
let son: Son;

parent = son; // ✅ok
son = parent; // ❌error! parent 有可能是 'c'
```

当 parent 取 c 时，son 的类型 Son 并没有包含"c"，因此不能赋值。

这种情况和上面恰恰相反，少的能赋值给多的，但是反过来不行。这种情况叫做**逆变**

#### 变体

变体有四种形式：

- 协变（Covariant）：子类型比父类型更具体、属性更多，子类型可以赋值给父类型，父类型可以转换为子类型（扩展）。ts 中对象、数组的相互赋值都是协变的。
- 逆变（Contravariant）：和协变相反，子类型变为父类型（收缩）。
- 双向协变（Bivariant）：父子类型相互转化，ts 函数的参数就是双向协变的。
- 不变（Invariant）：父子类型不能转化

这里先规定一种表示方法：

> A ≼ B 意味着 A 是 B 的子类型。
> A → B 指的是以 A 为参数类型，以 B 为返回值类型的函数类型。
> x : A 意味着 x 的类型为 A

- ts 的对象和数组是协变的，因为一个包含另一个对象的对象可以赋给该对象
- ts 的函数**返回值类型是协变的，而参数类型是逆变的**。注意这里的逆变协变并**不是发生在函数调用**，而是发生在函数间的比较和类型兼容时。

  - 返回值类型是协变的，意思是 `A ≼ B` 就意味着 `(T → A) ≼ (T → B) `，也就是返回值类型可以扩大；如果规定返回值为一个类型，但是传入一个更大的类型，是可以的。
  - 参数类型是逆变的，意思是 `A ≼ B `就意味着 `(B → T) ≼ (A → T)` 。
    参考这个例子：

  ```ts
  let func1 = (a: number) => a;
  let func2 = (a: number, b: boolean) => a;

  func1 = func2; //error
  func2 = func1;
  ```

### 比较对象

如果 x 要兼容 y，那么 y 至少具有与 x 相同的属性；也就是多可以赋给少，但是少不能赋给多。

```ts
interface AName {
  name: string;
}
interface BName {
  name: string;
  age: number;
}

let aname: AName;
let bname: BName;
aname = bname;
bname = aname; //error，因为bname中的age属性aname没有
```

aname 中的全部属性在 bname 中都有并且不冲突，因此可以把 bname 赋值给 aname。

从类型角度来说，BName 是 AName 的子类型，子类型可以转化为父类型，这就是协变。

对于函数参数也是这样的，参数如果被指明类型 A，B 类型是 A 类型的扩充，那么传入 B 类型并不会报错。当然这里指的是传参，并不是函数之间参数的比较；

```ts
function greet(n: IName) {
  //...
}
greet(bname); // OK
```

### 比较函数

函数的依据是参数列表，被赋值的 func2 必须有 func1 的所有参数且不冲突, 反过来就不行；

参数少的可以被赋给参数多的，反之不行。即参数只能变多不能变少；

从类型角度来说，func2 的参数是 func1 的子类，func2 可以变成 func1，即子类变成父类，是逆变；

注意这里对比的只是类型，和参数名无关。下面这个例子a和b名字不同，但是都是number类型。

```ts
let func1 = (a: number) => a;
let func2 = (b: number, c: boolean) => b;

func1 = func2; //error
func2 = func1; // func2的参数是func1的子类型，相当于func2可以“接住”func1的所有可能参数类型
```
如果有返回值那就还要考虑返回值，即参数和返回值都要满足才能赋值。返回值的依据是源函数（右边的）的返回值类型必须是目标函数（左边的）返回值类型的子类型（即右边的返回值应该是左边的子类型，属性多的）。

```js
let x = () => ({name: 'Alice'});
let y = () => ({name: 'Alice', location: 'NewYork'});

x = y; // y返回值是x返回值的的子类型
y = x; // error
```


可以参考这个例子：

```ts
let visitAnimal = (animal: Animal) => {
  animal.age;
};

let visitDog = (dog: Dog) => {
  dog.age;
  dog.bark();
};

visitAnimal = visitDog;

let animal = { age: 5 };

visitAnimal(animal); // ❌
```

如果我们把`visitAnimal = visitDog`，现在 visitAnimal 是和 visitDog 一样的具体代码，即会访问参数的 bark 方法；这时传入一个没有 bark 方法的对象就会报错。

而反过来就不会，函数内部不会访问除了 age 之外的任何属性，因此是一定安全的。

# 高级类型

### 交叉类型`&` 和 联合类型`|`

交叉类型是将多个类型合并为一个类型，包含了所需的所有类型的特性。
表示两个类型都具有, 对于对象型类型表示所有属性都拥有

```ts
function mixin<T, U>(f: T, l: U): T & U {
  let res = <T & U>{};
  for (let key in f) {
    (res as any)[key] = (f as any)[key];
  }
  for (let key in l) {
    (res as any)[key] = (l as any)[key];
  }
  return res;
}

let mixins = mixin<{ name: string }, { age: number }>(
  { name: "zzx" },
  { age: 19 }
);
```

---

联合类型表示“或”，一般是数种类型的其中一种。
对象类型的联合类型只能确定共有的属性，比如下面这个例子：

```ts
interface union1 {
  run: boolean;
  walk: boolean;
}
interface union2 {
  run: boolean;
  speak: boolean;
}

function getUnion(): union1 | union2 {
  return {
    run: false,
    walk: true,
  };
}
let res = getUnion();
res.walk; //报错, 只能访问共有的run
```

虽然返回的对象中有 walk，但是由于只共有 run 这个属性，因此只能确定有该属性，其他的都不能直接取。
这种问题的解决方案有两种：

- 类型断言，即令要取属性的变量为某种类型，然后取值即可
- 类型保护，即通过判断是哪种类型，采取不同的处理。类型保护其实就是一个判断是哪种联合类型的函数, 返回值是一个类型谓词

比如下面这个例子是一个自定义的类型保护，原理就是判断某个类型是否含有某个属性：

```js
interface Dog {
  name: string;
  canBite: boolean;
}
interface Cat {
  name: string;
  canCatch: boolean;
}

function isDog(animal: Dog | Cat): animal is Dog {
  return (animal as Dog).canBite !== undefined;
}

function clg(arg: Cat | Dog) {
  isDog(arg) ? console.log("Dog") : console.log("Cat");
}

clg({ name: "miaomiao", canCatch: true });

```

写法：

- 一个判断是哪种类型的函数，返回是一个**布尔值**，可以通过类型断言返回确定是否含有一个对象的某个键；
- 返回类型可以直接定义为 boolean，但是更好的方式是用 is；用 `arg is typename` 的形式；is 运算符相当于缩小了类型范围，其中 `typename` 是参数类型之一，并且应该是当前判定的类型。比如判定是否为 Dog，那么就应该是 `animal is Dog`

除了自定义类型保护，也可以用 typeof 或 instanceof 来判断类型，后者主要用于类类型的判断。

### 可选参数

使用了 --strictNullChecks，可选参数会被自动地加上 | undefined:

```ts
function getValue(username: string, userId?: string) {
  return `${username} ${userId!.length}`; //通过 ! 去除undefined
}
```

添加`!`后缀可以去除了 null 和 undefined，或者用 || 或 if 判断也可以

### 类型别名

类型别名会给一个类型起个新名字。 类型别名有时和接口很像，但是可以作用于原始值，联合类型，元组以及其它任何需要手写的类型。

```ts
type str = string;
type myFn<T> = () => T;
type myObj = {
  name: string;
};
```

类型别名可以在类型内部调用自己，这是一个很神奇的、类似递归的效果：

```ts
type Tree<T> = {
  value: T;
  left: Tree<T>;
  right: Tree<T>;
};
```

但是类型别名不能作为一个变量，不能被任何情况赋值、不能被继承或复制给其他类型别名。

#### 接口和类型别名的区别

基本的写法区别不再赘述，其他的区别还有：

- 类型别名可以给基本类型、部分高级类型（联合交叉）重新取名，接口不行
- 接口通过 extends 扩展，类型别名可以通过`&`扩展；两者也可以相互扩展

```ts
interface PartialPointX {
  x: number;
}
interface Point extends PartialPointX {
  y: number;
}

type PartialPointX = {
  x: number;
};
type Point = PartialPointX & { y: number };

type PartialPointX = {
  x: number;
};
interface Point extends PartialPointX {
  y: number;
}

interface PartialPointX {
  x: number;
}
type Point = PartialPointX & { y: number };
```

- 接口可以定义多次，并将被视为单个接口，类型别名不可以
- type 能使用 in 关键字生成映射类型（比如 forin 循环，或者`[key in Keys]`遍历），但 interface 不行。

### 索引类型

#### 索引签名

首先要理解索引签名是什么

索引，就是根据一定的指向返回相应的值，比如数组的索引就是下标 0,1,2;
typescript 里的索引签名有两种：数字索引和字符串索引；
如下的`[xxx:xxx]:xxx`就是索引签名形式

```ts
interface numberIndex {
  [index: number]: any;
}

interface objIndex {
  [index: string]: any;
}

interface strIndex {
  [index: string]: string;
}

const foo: {
  [index: string]: { message: string };
} = {};
```

最后一种可以是自定义的索引签名类型，也就是规定每一个值都应当是`{message:xxx}`形式的。

##### 有限的索引

使用`in`可以从一组联合或者对象类型别名中取得有限个索引类型

```ts
type index = "a" | "b" | "c";
type indexType = {
  a: string;
  b: string;
  c: number;
};

type FormIndex = {
  [key in index]: any;
};
type FormIndexType = {
  //这里可以自动确定值的类型，核心是keyof取得所指对象的key序列，因此后面可以取到；没有keyof则不行
  [key in keyof indexType]: indexType[key];
};
```

可以使用泛型规定来源的类型，也可以利用泛型自动确定值的类型。

```ts
type FromGenericsIndex<K extends string> = {
  [key in K]: any;
};

type IndexFormGenerics<T> = {
  [key in keyof T]: T[key];
};
```

#### 索引操作符

ts 中直接访问对象属性可能会报没有属性的错误，因此可以借助 keyof 取得对象索引
比如这样一个 js 函数

```js
function pluck(o, names) {
  return names.map((n) => o[n]);
}
```

在 ts 中是这样的

```ts
function pluck<T, K extends keyof T>(o: T, names: K[]): T[K][] {
  return names.map((n) => o[n]);
}

interface Person {
  name: string;
  age: number;
}
let person: Person = {
  name: "Jarid",
  age: 35,
};
let strings: string[] = pluck(person, ["name"]);
```

这里的 K 依赖于 T，不需要明确指明，调用时根据传入的参数确定
这里有两个操作符：

- 索引类型查询操作符，即`keyof`，对于任何类型 T， keyof T 的结果为 T 上已知的公共属性名的联合，比如：

```ts
let personProps: keyof Person; // 'name' | 'age'
```

这一点也可用于对象的 forin 遍历

```ts
const myTestObj = {
  prop1: "prop1",
  prop2: "prop2",
};
type myKey = keyof typeof myTestObj;
for (let key in myTestObj) {
  myTestObj[key as myKey];
}

//或者
let key: keyof typeof myTestObj;
for (key in myTestObj) {
  console.log(obj[key]);
}
```

- 索引访问操作符，即`T[K]`，表示 T 中的 K 属性的类型；比如这个例子中，当 K 取 name 时，`T[k]`就是 string，后面的[]表示数组类型（注意不是二维数组）

### 映射类型

> TypeScript 提供了从旧类型中创建新类型的一种方式 — 映射类型。 在映射类型里，新类型以相同的形式去转换旧类型里每个属性。

其实就是一种工具类，用于转换一个接口或者类型别名
常见的映射类型：

- Readonly，把所有属性转为可读
  实现原理：

```ts
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Person = {
  name: string;
  age: number;
  [propsName: string]: any;
};
type ReadonlyPerson = Readonly<Person>;
```

readonly 之后的部分相当于对原对象的遍历

- Partial，把所有类型转换为可选，原理和使用方法相同

```ts
type Partial<T> = {
  [P in keyof T]?: T[P];
};
type PersonPartial = Partial<Person>;
```

- `Pick<T,P>`，给出两个泛型，相当于提取取前一个泛型的 P 属性，实现如下：

```ts
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

interface IBus {
  price: number;
  weight: number;
}
type price = Pick<IBus, "price">; //type price = {price:number}
```

要求后者必须是前者的键之一

- `Record<T,P>`，把 T 的属性值遍历转化为 P 类型，返回一个键为 T 值，值为 P 类型的对象类型。因此 T 只能是 string/number/symbol，P 可以是任意类型（对象的值可以是任意类型），但是一般用作对象或者接口，相当于创建了一个每一项都是该接口的类型。

实现如下：

```ts
type Record<K extends string | number | symbol, T> = {
  [P in K]: T;
};

interface ICar {
  price: number;
  name: string;
}
type carList = "bmw" | "benz";
type cars = Record<carList, ICar>;
/*type cars = {
    bmw:ICar;
    benz:ICar;
}*/
```

还有一些其他常用的类型：

- `Exclude<T, U>` -- 从 T 中剔除和 U 交集的类型，返回去除后的 T 类型；如果 T 是 U 的子集，即 T 全部被剔除，就返回 never；

```ts
//实现
type Exclude<T, U> = T extends U ? never : T;

type T00 = Exclude<"a" | "b" | "c" | "d", "a" | "c" | "f">; // "b" | "d"
```

- `Extract<T, U>` -- 保留 T 和 U 的交集，没有交集返回 never

```ts
//实现
type Extract<T, U> = T extends U ? T : never;

type T01 = Extract<"a" | "b" | "c" | "d", "a" | "c" | "f">; // "a" | "c"
```

注意这两个在判断上恰恰相反。

- `NonNullable<T>` -- 从 T 中剔除 null 和 undefined。

```ts
//实现
type NonNullable<T> = T extends null | undefined ? never : T;
```

- `ReturnType<T>` -- 获取函数返回值类型。

```ts
//实现
type ReturnType<T extends (...args: any) => any> = T extends (
  ...args: any
) => infer R
  ? R
  : any;
```

这个实现稍微有点复杂。

1. 首先`<T extends (...args: any) => any>`限制 T 是一个任意函数；
2. 传入的类型 T 若能够赋值给 `(...args: any) => R` 则返回类型 R，也就是说如果 T 是函数就返回该函数的返回值；但是这个返回值恰恰是要去获得的，因此需要“推断”一下。
   infer 表示一个“推断”；没有 infer 的话，ReturnType 实际上是要实现这样的效果：

```ts
type ReturnType<T> = T extends (...args: any[]) => R ? R : any;
```

也就是判断 T 是否可以赋值给 R，如果可以就返回 R（也就是要获取的返回值类型），不可以就返回 any；但是 R 我们事先不知道，也不能给出，就给 R 前面加上 infer，表示 R 由 T 的返回值推出。**infer R 被放在 (...args: any) => infer R 函数类型的返回值处，因此推断出了 返回值 类型并赋值给 R 。**

> 关于 infer，还常用作推断参数；既然我们已知 infer 推断和其位置有关，那么把 infer 放在参数中也可以表示推断参数类型，然后像推断返回值一样把获取的值存在一个泛型中（R）并返回回去。

```ts
type ParamType<T extends (arg: any) => any> = T extends (arg: infer R) => any
  ? R
  : any;
```

另外，这种写法中等号右边的 extends 其实相当于程序中的`===`，也就是整体相当于返回 R 的三元运算符。
依照这个原理，可以用类型写出一些判断

```ts
type IsNumber<N> = N extends number ? "yes, is a number" : "no, not a number";
```

由于 infer 的类型推断，综合起来就可以把 infer 放在想要获取的地方并在运算后返回。
比如一个对元组的 shift 操作，等号右边就是利用函数剩余参数的性质，将第一个参数提取出来，剩下的部分作为 R 返回；

> 其中 extends 在右边并不是单单相当于等号，与 infer 共用时可以理解为一个“转化”，就是把 extends 左边转化为右边的形式；

```ts
type ShiftAction<T extends any[]> = ((...args: T) => any) extends (
  arg1: any,
  ...rest: infer R
) => any
  ? R
  : never;
```

# 命名空间

ts 的命名空间可以理解为一个模块，但不同于 ES6 的模块是以文件为单位的，namespace 更多的是封装一些小的模块进行调用
命名空间的基本使用：

```ts
namespace UserModule {
  export interface IUserInfo {
    name: string;
    id: string;
    age: number;
  }

  const usernameRegExp = /^[a-zA-Z][0-9a-zA-Z]+$/;
  const userList: IUserInfo[] = [];

  export const addUser = (userObj: IUserInfo) => {
    if (usernameRegExp.test(userObj.name)) {
      userList.push(userObj);
    } else {
      console.log("error");
    }
  };
}

UserModule.addUser({
  name: "zzx",
  age: 19,
  id: "001",
});
```

上面代码主要有这几个注意点：

- namespace 声明并包裹一堆代码，需要在外部引用的方法、变量、接口、类型等需要 export，其他的内部则不需要；
- namespace 可以使内部的声明独立，也就是让外部其他变量或函数拥有和内部一样的命名；这样就可以减少重名的冲突。
- 使用时采用`name.xxx`的形式

另外，命名空间还可以在文件间相互调用；调用的方式是专属的 reference 语法：在文件头部用`/// <reference path="xxx.ts" />`引用，其中 path 就是该文件的相对路径。引入后该文件定义的命名空间会和当前文件**合并**。

```ts
// xxx.ts
namespace Food {
  export interface Fruits {
    taste: string;
    hardness: number;
  }
}

// yyy.ts
<reference path="xxx.ts" />;

let meat: Food.Meat;
let fruits: Food.Fruits;
```

当涉及到多文件时，我们必须确保所有编译后的代码都被加载，也就是让 tsc 编译多个文件；这点在 webpack 等打包工具中不存在，但是仍要考虑

1. 第一种方式，把所有的输入文件编译为一个输出文件，需要使用--outFile 标记：

```
tsc --outFile sample.js Test.ts
```

编译器会根据源码里的引用标签自动地对输出进行排序 

2. 编译多个 ts 文件，然后用多个 script 标签引入（好单纯的方法）。

# 声明合并

> “声明合并”是指编译器将针对同一个名字的两个独立声明合并为单一声明。 合并后的声明同时拥有原先两个声明的特性。 任何数量的声明都可被合并；不局限于两个声明。

### 接口合并

接口合并有两个主要规则：

1. 同名接口中**非函数成员唯一**；如果不唯一，**类型也必须相同**，否则会报错

```ts
interface Box {
  height: number;
  width: number;
}

interface Box {
  scale: number;
}
//等同于
interface Box {
  height: number;
  width: number;
  scale: number;
}
```

2. 没有名称的函数成员不会引起冲突，即使完全一样；有名称的函数被视作对象中的方法，每个同名函数声明都会被当成这个函数的一个重载。当接口 A 与后来的接口 A 合并时，**后面的接口具有更高的优先级**。

```ts
interface Cloner {
  clone(animal: Animal): Animal;
}

interface Cloner {
  clone(animal: Sheep): Sheep;
}
//等同于
interface Cloner {
  clone(animal: Sheep): Sheep;
  clone(animal: Animal): Animal; //先声明的在后面
}
```

**类型别名不可以被合并，同名的类型别名会冲突**。

### 命名空间合并

与接口相似，同名的命名空间也会合并其成员。其合并的规则如下：

- 命名空间导出的同名接口进行合并，构成单一命名空间内含合并后的接口。
- 命名空间导出的非接口成员会被加入到之前存在的命名空间中
  - 非导出成员不合并，只在原先的命名空间中可见
  - 合并之后的成员当作统一个命名空间内，因此 const 和 let 声明的变量不能重复；函数同理

# 声明文件

## 什么是声明文件？

广义上说，声明文件就是我们在使用一些 js 库时的`@types/`文件，其中包含了 js 库的类型定义，使得其在 ts 文件中也可以使用。现在主流的 js 库都有发布到 npm 的@types 文件，但有些仍需要使用时去手动编写，或者自己开发的 js 模块也需要声明文件兼容 ts。
文件中只包含与类型相关的代码，不包含逻辑代码，它们的作用旨在为开发者提供类型信息，所以它们只在开发阶段起作用。

举个例子，现在在一个ts-react项目中，绝大多数模块都是ts类型的，但是这时创建了一个js的文件：
```js
// someInfo.js
export const userInfo = {
  username:'aaa';
  age:18
  ...
}
```
如果我希望引入someInfo.js文件到ts文件中，就会提示找不到someInfo.js的声明文件，因为他是一个js模块，ts不能得知它的类型。
为了让ts得知它的类型，我们不需要把他改编成ts文件，而是创建一个声明文件，把其中**导出的变量、函数等**声明一个类型，让ts能识别它。
我们在同目录下创建一个someInfo.d.ts文件：
```ts
export const userInfo: {
  username:string;
  age:number
  ...
}
```
这样我们在ts文件中导入userInfo时，ts会自动解析someInfo.d.ts文件，并把其中的类型作为导入的类型。

对于js编写的库，也是类似的操作。但是不同的库的模块化方式有所不同，因此为库创建类型文件时有所不同。
比如jQuery在全局声明了一个`$`变量，因此我们就需要创建一个全局的global.d.ts文件，给`$`变量扩充类型
```ts
declare var $ : {
  ...
}
```

## 声明文件类型

根据模块的不同，声明类型有不同，主要分为三类：

- 全局变量：通过 `<script>` 标签引入第三方库，注入全局变量，常见的比如 jquery
- npm 包：通过 `import foo from 'foo'` 导入，符合 ES6 模块规范，即当前绝大多数包的方式
- UMD 库：既可以通过 `<script>` 标签引入，又可以通过 import 导入，UMD 现在比较少用。

这三类模块对应的声明文件存放位置、声明形式等都有所不同

### 全局变量

全局变量是最简单的一种场景。直接在声明文件中声明的变量、函数、接口、类型别名以及命名空间都可以被任意一个文件使用，相当于在所有文件顶部声明了全局变量，不需要任何导入导出。
全局变量声明文件存放位置应当直接在当前项目中，比如可以直接放在 src/@types 文件夹下。
主要语法：
​

- `declare var` 声明全局变量
- `declare function` 声明全局方法
- `declare class` 声明全局类
- `declare enum` 声明全局枚举类型
- `declare namespace` 声明全局对象类型；如果对象是嵌套的，就在内部再声明一个命名空间，以此类推
- `interface` 和 `type` 声明全局类型

举个栗子，比如我们的模块其中有一个文件中有这样的几个函数和变量：

```js
function helloWorld(arg) {
  console.log(arg);
}

const HELLO = "hello world";

const hello = {
  version: "1.0.0",
  params: "hello world",
};
```

根据上面的规则，我们就可以书写这样的声明文件

```ts
//index.d.ts
declare const HELLO: string; //常量声明，只需要标明类型
declare function helloWorld(arg: any): void; //函数

declare namespace hello {
  //表示对象
  const version: string;
  const params: string;
}
```

这样就可以在任意 ts 文件中使用如上几个变量。其中 namespace 中的变量需要`hello.xxx`来访问

### npm 模块

如果需要自己书写模块的声明文件，采用 export 的形式编写

```ts
// types/foo/index.d.ts
export const name: string;
export function getName(): string;
export class Animal {
  constructor(name: string);
  sayHi(): string;
}
export enum Directions {
  Up,
  Down,
  Left,
  Right,
}
export interface Options {
  data: any;
}
export namespace foo {
  const name: string;
  namespace bar {
    function baz(): string;
  }
}
```

当然也可以先声明再导出，声明的方式和全局一样

```ts
// types/foo/index.d.ts
declare const name: string;
declare function getName(): string;
declare class Animal {
  constructor(name: string);
  sayHi(): string;
}
declare enum Directions {
  Up,
  Down,
  Left,
  Right,
}
interface Options {
  data: any;
}

export { name, getName, Animal, Directions, Options };
```

还可以使用`export default`默认导出；
注意，只有 function、class 和 interface 可以直接默认导出，其他的变量需要先定义（delcare）出来，再默认导出

```ts
export default function foo(): string;

declare enum Directions {
  Up,
  Down,
  Left,
  Right,
}
export default Directions;
```

## 扩展全局变量

利用 ts 中接口、命名空间可合并的特征，可以扩展全局变量。有两种方法扩展：

### 1. 直接扩展

在`.d.ts文件`中直接使用接口，比如扩展 String 类型

```ts
interface String {
  prependHello(): string;
}

"foo".prependHello();
```

也可以使用 `declare namespace` 给已有的命名空间添加类型声明，也是在对应.d.ts 文件中

```ts
declare namespace JQuery {
  interface CustomOptions {
    bar: string;
  }
}

interface JQueryStatic {
  foo(options: JQuery.CustomOptions): string;
}
```

### 2. npm 包扩展

使用 `declare global` 可以在 npm 包或者 UMD 库的声明文件中扩展全局变量的类型；注意不能直接像全局直接扩展一样，因为 es6 模块下只有 export 导出的才能被导入

```ts
declare global {
  interface String {
    prependHello(): string;
  }
}

export {};
```

结尾的`export {}`用来告诉编译器这是一个模块的声明文件，而不是一个全局变量的声明文件，很重要！

## 声明文件位置

1. 目录 src/@types/，在 src 目录新建 @types 目录，在其中编写 .d.ts 声明文件，声明文件会自动被识别，可以在此为一些没有声明文件的模块编写自己的声明文件。实际上在 tsconfig.json 中 include 字段包含的范围内编写 .d.ts，都将被自动识别；
2. 与被声明的 js 文件同级目录内，创建相同名称的 .d.ts 文件，这样也会被自动识别；
3. 设置 package.json 中的 typings 属性值，如 ./index.d.ts. 这样系统会识别该地址的声明文件。同样当我们把自己的 js 库发布到 npm 上时，按照该方法绑定声明文件。
4. 同过 npm 模块安装，如 @type/react ，它存放在 node_modules/@types/ 路径下

## 三斜线

类似命名空间的相互引用，如果想使用另一个文件，而这个文件不支持 import 语法就要使用三斜线语法。一般来说 xxx.d.ts 就是不支持 import 语法的，因此多个声明文件的模块化就依赖于此。

```ts
// node_modules/@types/jquery/index.d.ts
/// <reference types="sizzle" />
/// <reference path="JQueryStatic.d.ts" />
/// <reference path="JQuery.d.ts" />
/// <reference path="misc.d.ts" />
/// <reference path="legacy.d.ts" />
export = jQuery;
```

## 自动生成声明文件

ts 文件可以通过规定 tsc 的编译模式来自动生成声明文件
在 tsconfig.json 里面进行配置：

- `declaration` 为每个 TS 文件生成 xx.d.ts 和 x.js 文件。
- `declarationDir` 设置生成 .d.ts 文件的目录
- `declarationMap` 对每个 .d.ts 文件，都生成对应的 .d.ts.map（sourcemap）文件
- `emitDeclarationOnly` 仅生成 .d.ts 文件，不生成 .js 文件

# 编译选项（tsconfig.json）

> 如果一个目录下存在一个 tsconfig.json 文件，那么它意味着这个目录是 TypeScript 项目的根目录。 tsconfig.json 文件中指定了用来编译这个项目的根文件和编译选项。

### 使用

初始化操作有 2 种方式：

1. 手动在项目根目录（或其他）创建 tsconfig.json 文件并填写配置；
2. 通过 tsc --init 初始化 tsconfig.json 文件。

另外也可以为 tsc 命令指定参数 --project 或 -p 指定需要编译的目录，该目录需要包含一个 tsconfig.json 文件。指定后其他位置的 tsconfig 不再生效

### 结构

tsconfig 主要由 8 个顶层属性组成，其中 compilerOptions 内还有几十个相关配置属性

![](https://pic.imgdb.cn/item/627a15dd09475431290c29d8.png)
![](https://pic.imgdb.cn/item/627a15f409475431290c62f2.png)

常用的顶层属性是如下四个：

```json
{
  "compilerOptions": {},
  "files": [],
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

- `compilerOptions`是最大、最多配置项的部分，其中有几十个配置选项，后面会详述
- `files`指定要编译的 ts 文件，使用根目录下的相对路径表示
- `include`和`exclude`分别表示包括和排除的文件和目录。两者具有类似的 glob 通配符：
  - `*` 匹配 0 或多个字符（不包括目录分隔符）
  - `?` 匹配一个任意字符（不包括目录分隔符）
  - `**/` 递归匹配任意子目录

### compilerOptions 配置编译选项

编译选项配置非常繁杂，有很多配置，这里只列出常用的配置。

```json
{
  // ...
  "compilerOptions": {
    "incremental": true, // TS编译器在第一次编译之后会生成一个存储编译信息的文件，第二次编译会在第一次的基础上进行增量编译，可以提高编译的速度
    "tsBuildInfoFile": "./buildFile", // 增量编译文件的存储位置
    "diagnostics": true, // 打印诊断信息
    "target": "ES5", // 目标语言的版本
    "module": "CommonJS", // 生成代码的模块标准，如果要使用import/export需要写成ES2015或es6；在webpack相关配置中必须是es6
    "outFile": "./app.js", // 将多个相互依赖的文件生成一个文件
    "lib": ["DOM", "ES2015", "ScriptHost", "ES2019.Array"], // TS需要引用的库，即声明文件，es5 默认引用dom、es5、scripthost,如需要使用es的高级版本特性，通常都需要配置，如es8的数组新特性需要引入"ES2019.Array",
    "allowJS": true, // 允许编译器编译JS，JSX文件
    "checkJs": true, // 允许在JS文件中报错，通常与allowJS一起使用
    "outDir": "./dist", // 指定输出目录
    "rootDir": "./", // 指定输出文件目录(用于输出)，用于控制输出目录结构
    "declaration": true, // 生成声明文件，开启后会自动生成声明文件
    "declarationDir": "./file", // 指定生成声明文件存放目录
    "emitDeclarationOnly": true, // 只生成声明文件，而不会生成js文件
    "sourceMap": true, // 生成目标文件的sourceMap文件
    "inlineSourceMap": true, // 生成目标文件的inline SourceMap，inline SourceMap会包含在生成的js文件中
    "declarationMap": true, // 为声明文件生成sourceMap
    "typeRoots": [], // 声明文件目录，默认时node_modules/@types
    "types": [], // 加载的声明文件包
    "removeComments": true, // 删除注释
    "noEmit": true, // 不输出文件,即编译后不会生成任何js文件
    "noEmitOnError": true, // 发送错误时不输出任何文件
    "noEmitHelpers": true, // 不生成helper函数，减小体积，需要额外安装，常配合importHelpers一起使用
    "importHelpers": true, // 通过tslib引入helper函数，文件必须是模块
    "downlevelIteration": true, // 降级遍历器实现，如果目标源是es3/5，那么遍历器会有降级的实现
    "strict": true, // 开启所有严格的类型检查
    "alwaysStrict": true, // 在代码中注入'use strict'
    "noImplicitAny": true, // 不允许隐式的any类型
    "strictNullChecks": true, // 不允许把null、undefined赋值给其他类型的变量
    "strictFunctionTypes": true, // 不允许函数参数双向协变
    "strictPropertyInitialization": true, // 类的实例属性必须初始化
    "strictBindCallApply": true, // 严格的bind/call/apply检查
    "noImplicitThis": true, // 不允许this有隐式的any类型
    "noUnusedLocals": true, // 检查只声明、未使用的局部变量(只提示不报错)
    "noUnusedParameters": true, // 检查未使用的函数参数(只提示不报错)
    "noFallthroughCasesInSwitch": true, // 防止switch语句贯穿(即如果没有break语句后面不会执行)
    "noImplicitReturns": true, //每个分支都会有返回值
    "esModuleInterop": true, // 允许export=导出，由import from 导入
    "allowUmdGlobalAccess": true, // 允许在模块中全局变量的方式访问umd模块
    "moduleResolution": "node", // 模块解析策略，ts默认用node的解析策略，即相对的方式导入
    "baseUrl": "./", // 解析非相对模块的基地址，默认是当前目录
    "paths": {
      // 路径映射，相对于baseUrl
      // 如使用jq时不想使用默认版本，而需要手动指定版本，可进行如下配置
      "jquery": ["node_modules/jquery/dist/jquery.min.js"]
    },
    "rootDirs": ["src", "out"], // 将多个目录放在一个虚拟目录下，用于运行时，即编译后引入文件的位置可能发生变化，这也设置可以虚拟src和out在同一个目录下，不用再去改变路径也不会报错
    "listEmittedFiles": true, // 打印输出文件
    "listFiles": true // 打印编译的文件(包括引用的声明文件)
  }
}
```

# TIPs

## 关于索引签名的区别

`[P in T]`和`[P in keyof T]`有什么区别呢？什么时候该用 in，什么时候该用 in keyof？

- `[P in T]`表示 T 一定是`string|number|symbol`或者他们的联合；因此 in 表示 T 是联合或者经过 keyof 取值的对象（其实还是联合）
- `[P in keyof T]`表示 T 是一个有索引的类型，比如对象、数组。keyof 取值之后就成为了字符串的联合

```ts
interface IUser {
  name: string;
  age: number;
}

type union = "a" | "b" | "c";

type getIn<T> = {
  [P in T]: P;
};
getIn<union>
/*
type getIn = {
    a: "a";
    b: "b";
    c: "c";
}
*/
getKey<IUser>
type getKey<T> = {
  [P in keyof T]: P;
};
/*
type getKey = {
    name: "name";
    age: "age";
}
*/
```

---

那索引之后的`P`和`T[P]`又分别表示什么呢？（按照上面的形式）

- `P`一定是**键**，并且 P 应当是`number|string|symbol`之一。
- `T[P]`表示 T 中取 P 对应的值，也就要求 T 一定是**接口或对象型的类型别名**。如果 P 满足是键的话，T 就是该键对应的类型

```ts
type getKey<T> = {
  [P in keyof T]: T[P];
};

getKey<IUser>
/*
type getKey = {
    name: string;
    age: number;
}
*/
```

## 强大的类型语法

ts 中的类型是一个完整的体系，以类型别名定义的类型几乎可以作为一个完整的语言来运作

### extends

extends 位于类型别名等号右边时，就会产生不同于“约束”“继承”的功能。

- 三元运算，让类型也可以有 if-else

  ```ts
  type IsNumber<N> = N extends number ? "yes, is a number" : "no, not a number";
  ```

  三元运算中的 extends 相当于`===`，并会返回符合条件的结果。

- 和 infer 一起时表示“转换”
  比如在元组中说过的 shift 代码：

  ```ts
  type ShiftAction<T extends any[]> = ((...args: T) => any) extends (
    arg1: any,
    ...rest: infer R
  ) => any
    ? R
    : never;
  ```

  这里的 extends 就不是“等于”，而更多表示把一个`((...args: T) => any)`的函数转化为第一个参数提取出来的形式，并配合`infer R`把要求的 R 返回。

- 用于映射，通过`extends any`让任意类型都可以转化为想要的类型

  ```ts
  // 这里的 placeholder 可以键入任何你所希望映射成为的类型
  type UnionTypesMap<T> = T extends any ? "placeholder" : never;
  type UnionTypesMap2Func<T> = T extends any ? () => T : never;
  type UnionTypesMap3Func<T> = T extends any ? { [k: string]: T } : never;
  ```

  `extends any`相当于抹平的对 T 的所有限制，然后就可以返回任何类型，也就是把 T 转化为任何想要的类型（placeholder 的类型仍需要含有 T，不能直接是 string 这样的简单类型）。

### 类型语言

#### 类型支持作用域

泛型类型可以当作 js 中函数来看待，尖括号内是参数，等号后面是具体的函数体；结合之前说的三元运算、extends 就可以实现很多类型相关的处理，比如自定义工具类等。
既然相当于函数，类型泛型其实也像 js 函数一样有“作用域”，比如闭包：

```ts
function Foo<T>() {
  return function (param: T) {
    return param;
  };
}

const myFooStr = Foo<string>();
// const myFooStr: (param: string) => string
// myFooStr的类型和Foo函数中定义的相同，说明类型作为一个闭包跟在myFooStr上
const myFooNum = Foo<number>();
// const myFooNum: (param: number) => number
// 由于闭包，类型也会保持相互独立，互不干涉
```

#### 类型递归

类型语法虽然不支持循环，但是因为类型别名可以自己引用自己的原因，使用递归可以相当于循环：
比如

```ts
// shift在前面元组中讲过
type combineTupleTypeWithTecursion<T extends any[], E = {}> = {
  1: E; //键的1和0也可以是任何键，主要用于引用E方便返回
  0: combineTupleTypeWithTecursion<ShiftAction<T>, E & T[0]>; //递归调用类型，相当于把第一个剔除之后放到E中然后继续
}[T extends [] ? 1 : 0]; //判断T是否为空数组，为空则终止

type test = [{ a: string }, { b: number }];
type testResult = combineTupleTypeWithTecursion<test>; // { a: string; } & { b: number; }

//换成正常的ts代码
function combineTupleTypeWithTecursion(T: object[], E: object = {}): object {
  return T.length
    ? combineTupleTypeWithTecursion(T.slice(1), { ...E, ...T[0] })
    : E;
}

const testData = [{ a: "hello world" }, { b: 100 }];
// 此时函数的返回值为 { a: 'hello world', b: 100 }
combineTupleTypeWithTecursion(testData);
```

类似的写法还可以完成 Concat(拼接元组):

```ts
type Concat<T extends any[], S extends any[]> = {
  0: T;
  1: Concat<Push<T, S[0]>, Shift<S>>; //把S第一项插入T，并把S第一项去掉；Push是之前提到的方法
}[S extends [] ? 0 : 1];

//正常ts代码
function Concat(T: any[], S: any[]): any[] {
  return S.length ? Concat(T.push(S[0]), S.slice(1)) : T;
}
```

## es6 相关内容

### Promise

Promise 的具体类型定义肥肠复杂，这里只是简单说一下使用相关

- Promise 直接链式调用可以自动判断 resolve 的值类型

```ts
Promise.resolve(123)
  .then((res) => res.toString())
  .then((res) => res.length)
  .then((res) => console.log(res));
```

- new Promise 调用需要手动规定类型，方法有两种，类型注解泛型规定和 Promise 本身的泛型规定

```ts
function getValue(): Promise<string> {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve("123");
    }, 1000);
  });
}

getValue().then((res) => {
  if (res.length < 2) {
    //...
  }
});

//或者
const promise = new Promise<number>((resolve) => {
  resolve(123);
});
promise.then((res) => {
  //...
});
```

### Map 和 Set

#### Map 的创建

和 js 一样，ts 中的 map 和 set 也有两中创建方式

```ts
const map = new Map();
map.set(true, false).set(1, "1").set("1", 1);

// 或者
const map = new Map<string | number, string | number>([
  ["hello", "world"],
  [0, 1],
]);

const map = new Map([//error:没有规定类型
  [...]
])
```

> 注意：第二种也就是初始化 map 的方式必须在泛型中注明类型否则会报错

观察 Map 的类型定义可得 Map 有三种重载：

```ts
Map<K, V>(iterable: Iterable<readonly [K, V]>): Map<K, V>

Map(): Map<any, any>

Map<K, V>(entries?: readonly (readonly [K, V])[] | null | undefined): Map<K, V>
```

其中第二种是最基础的类型，可以先用空值创建后续用 set 添加值，这种是不能有参数的；
第一种和第三种都强制要求注明类型，并返回一个该类型的 map；第三种就是初始化形式创建，第一种则是一个可迭代对象，比如设置了迭代器的 Object 或者另一个 map（是的，可以用一个 map 再创建一个 map）

```ts
const map = new Map();
const mapFromMap = new Map(map);
```

有一个缺陷是：map 只能大致规定键值的类型，不能像对象一样用接口来约束具体值；因此在遍历过程中可能需要类型断言或守卫。

#### Map 相关操作

Map 的遍历和在 js 中没有太多区别：

```ts
const map = new Map();
map.set(true, false).set(1, "1").set("1", 1);

for (let key of map.keys()) {
  //...
}
for (let value of map.values()) {
  console.log(value);
}
for (let entry of map) {
  let [key, value] = entry;
}
map.forEach((value, key, map) => {
  console.log(`${key}:${value}`);
});
```

如果通过 set 形式添加值，key 和 value 都会被认作`any`，除非再创建时用泛型规定值。

还可以利用相关 API 在 object 和 Map 间相互转换
注意要求`tsconfig.ts`中`"lib"`配置至少有`ES2019`及以上

```ts
let mapObj = {
  name: "zzx",
  age: 18,
};

//对象变map
const mapFromObj = new Map(Object.entries(mapObj));
console.log(mapFromObj.entries());

//map变对象
const objFromMap = Object.fromEntries(map);
console.log(objFromMap);
```

#### Set

Set 和 Map 类似，有两种创建的方式；但是 Set 可以初始化时不指明类型

```ts
const set = new Set([0, 1, 2, "3"]); //不会报错

const set = new Set<number | string>([1, "2", 3, "4"]); //这种更好

set.add(5).add("6");
for (let key of set.keys()) {
}
for (let value of set.values()) {
}
for (let entry of set) {
}
console.log(set.has(7));
console.log(set.size);
set.delete("5");
```

Set 的重载有两个，第一个是和 Map 中一样的可迭代对象，第二个则是一个数组

```ts
Set(iterable?: Iterable<unknown> | null | undefined): Set<unknown>

Set<T = any>(values?: readonly T[] | null | undefined): Set<T>
```

可以看到 set 中对类型有一个默认值 any，因此初始化不指明类型也不会报错

# TS 代码哲学

1. 减少不必要的显式类型定义，尽可能多地使用类型推导，让类型的流动像呼吸一样自然。
1. 尽可能少地使用 `any` 或 `as any`，注意这里并不是说不能用，而是你判断出目前情况下使用 `any` 是最优解。
1. 如果确定要使用 `any` 作为类型，优先考虑一下是否可以使用 `unknown` 类型替代，毕竟 `any` 会破坏类型的流动。
1. 尽可能少地使用 `as xxx`，如果大量使用这种方式纠正类型，那么大概率你对 `类型流动 `理解的还不够透彻。
