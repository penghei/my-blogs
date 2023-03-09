---
title: 设计模式学习总结
date: 2022-07-24 16:21:22
tags: 面试
categories: 原理
cover:
---

# 设计模式概览

![](https://pic.imgdb.cn/item/62dd6956f54cd3f9377ba2e7.jpg)

设计模式有五个基本设计原则，设计原则是设计模式的指导理论，它可以帮助我们规避不良的软件设计。这五个基本原则称为`SOLID`，指代五个基本原则（前端最常用的是前两个）：

- 单一功能原则（Single Responsibility Principle）：一个类或函数的职责应该尽可能单一，如果有多个职责，那就再创建一个类
- 开放封闭原则（Opened Closed Principle）：对拓展开放，对修改封闭。即软件实体（类、模块、函数）**可以扩展，但是不可修改**
- 里式替换原则（Liskov Substitution Principle）
- 接口隔离原则（Interface Segregation Principle）
- 依赖反转原则（Dependency Inversion Principle）

设计模式一共有 23 种，可以划分为 3 个基本类型：

![](https://pic1.imgdb.cn/item/6354e4ef16f2c2beb106ef62.jpg)

- 创建型：即创建对象过程中的变化，是构造相关的设计模式，包括：
  - 单例模式
  - 原型模式
  - 工厂模式（抽象工厂、简单工厂）
- 结构型：即对象之间组合方式的变化，目的在于灵活地表达对象间的配合与依赖关系，包括：
  - 组合模式
  - 装饰器模式
  - 适配器模式
  - 代理模式
- 行为型：将对象千变万化的行为进行抽离，确保我们能够更安全、更方便地对行为进行更改（优化对象的处理），包括：
  - 迭代器模式
  - 观察者和发布订阅模式
  - 状态模式
  - 策略模式

# 工厂模式

## 简单工厂模式

定义：工厂模式就是**将创建对象的过程单独封装**。

这里的封装是彻底的封装，可以通过函数或者类的形式，但是封装的结果一定是可以**无脑传参**的。也就是说我们完全不用关心内部如何实现，以及参数的相互依赖关系，而只用传参就好，工厂模式封装的内部负责这些。

工厂模式依赖于基本的构造器模式，即构造函数创建对象的形式。构造器模式很简单就不再赘述。工厂模式是对构造函数的进一步封装，以满足一些更复杂的情况。

比如现在有一个 user 类：

```js
class User {
  constructor(name, age, career) {
    this.name = name;
    this.age = age;
    this.career = career;
  }
}
```

如果现在还要添加一个字段，即 work 字段，要求这个字段根据 career 的不同做出变化。比如 career 是'保安'，那 work 就应该是'看大门'。
那么就可以这样：

```js
function getUser(name,age,career){
  let work
  switch(career) {
    case 'coder':
        work =  ['写代码','写系分', '修Bug']
        break
    case 'product manager':
        work = ['订会议室', '写PRD', '催更']
        break
    case 'boss':
        work = ['喝茶', '看报', '见客户']
    case 'xxx':
        // 其它工种的职责分配
        ...
    return new User(name, age, career, work)
  }
}
```

或者在 work 字段设置一个函数，根据 career 的不同得到不同的结果：

```js
class User {
  constructor(name, age, career) {
    this.name = name;
    this.age = age;
    this.career = career;
    this.work = getWork(career);
  }
}

function getWork(career) {
  let work = null;
  switch (career) {
    case "coder":
      work = ["写代码", "写系分", "修Bug"];
      break;
    case "product manager":
      work = ["订会议室", "写PRD", "催更"];
      break;
    case "boss":
      work = ["喝茶", "看报", "见客户"];
    case "xxx":
      work = [];
    // 其它工种的职责分配
  }
  return work;
}
```

以上两种都是对整个创建过程做了封装。不管内部如何实现，我们只需要调用函数并传入参数即可。

## 抽象工厂

- 在简单工厂的使用场景里，处理的对象是类，并且是一些非常好对付的类——它们的共性容易抽离，同时因为逻辑本身比较简单，故而不苛求代码可扩展性。
- 抽象工厂本质上处理的其实也是类，但是是一帮非常棘手、繁杂的类，这些类中不仅能划分出门派，还能划分出等级，同时存在着千变万化的扩展可能性。

工厂模式中的关键角色：

- 抽象工厂（抽象类，它不能被用于生成具体实例）： 用于声明最终目标产品的**共性**。在一个系统里，抽象工厂可以有多个，每一个抽象工厂对应的这一类的产品，被称为“产品族”。
- 具体工厂（用于生成产品族里的一个具体的产品）： **继承自抽象工厂**、实现了抽象工厂里声明的那些方法，用于创建具体的产品的类。

抽象工厂其实就是抽象类的应用。js 中没有抽象类，但是 ts 有，可以用 ts 来实现。
抽象类提供的是一种“规范”，更多提供的是具体产品的**基本**组成结构，而不是确切的产品。这种“基本”是最关键的地方。**在实现具体的类时，通常是继承并拓展新的功能，而不是直接修改基本类**。

# 单例模式

> **保证一个类仅有一个实例，并提供一个访问它的全局访问点**，这样的模式就叫做单例模式。

单例模式的使用很常见，比如操作缓存的 LocalStorage 对象，为了保证缓存操作始终操作在一个对象上，要通过单例模式保证对象的单一。
类似的还有 Vuex、Redux 等，以及全局唯一的 Modal 框等实现。

在实现上，可以通过给类定义一个静态方法，每次构建实例时应该调用这个静态方法，而不是直接 new。
具体的操作就是在这个方法内部，在这个类的 instance 属性上挂载一个实例，第一次创建时挂载，后续访问直接返回。

```js
class SingleDog {
  show() {
    console.log("我是一个单例对象");
  }
  static getInstance() {
    // 判断是否已经new过1个实例
    if (!SingleDog.instance) {
      // 若这个唯一的实例不存在，那么先创建它
      SingleDog.instance = new SingleDog();
    }
    // 如果这个唯一的实例已经存在，则直接返回
    return SingleDog.instance;
  }
}

const s1 = SingleDog.getInstance();
const s2 = SingleDog.getInstance();

// true
s1 === s2;
```

还有一种实现是依赖闭包，通常用于不用类的情况。

比如要实现一个全局唯一的弹框 Modal，不管在哪里创建 Moda 的实例，都是对同一个 Modal 操作。
（这里只有 js 部分）
在这种情境下，类通常不太好操作，因为创建的 modal 对象实际上是一个`document.createElement`对象，而非从 Modal 类中获得的。所以用闭包模拟一个类，直接返回创建的`document.createElement`对象，即 new 的结果，就相当于始终对全局创建的唯一 element 对象操作了。

```js
const Modal = (function () {
  let modal = null;
  return function () {
    if (!modal) {
      modal = document.createElement("div");
      modal.innerHTML = "我是一个全局唯一的Modal";
      modal.id = "modal";
      modal.style.display = "none";
      document.body.appendChild(modal);
    }
    return modal;
  };
})();

// 点击打开按钮展示模态框
document.getElementById("open").addEventListener("click", function () {
  // 未点击则不创建modal实例，避免不必要的内存占用;此处不用 new Modal 的形式调用也可以，和 Storage 同理
  const modal = new Modal();
  modal.style.display = "block";
});

// 点击关闭按钮隐藏模态框
document.getElementById("close").addEventListener("click", function () {
  const modal = new Modal();
  if (modal) {
    modal.style.display = "none";
  }
});
```

# 装饰器模式

所谓装饰器模式，实际上就是对现有的类的再次封装，对现有的类增加一些功能，添加一些“装饰”。

举个栗子，比如有一个基本类 Car，可以再创建一个 Decorate 类，传入原先的 Car 类，为其添加一些功能：

```js
class Car {
  constructor(speed) {
    this.speed = speed;
  }
  run() {
    console.log(`i can run with ${this.speed}`);
  }
}

class Decorator {
  constructor(baseClass) {
    this.baseClass = baseClass;
  }
  doubleRun() {
    console.log(`i can run with ${this.baseClass.speed * 2}`);
  }
  engine() {
    console.log("i have a strong engine");
  }
}

const car = new Car(100);
const decoratoredCar = new Decorator(car);
decoratoredCar.doubleRun(); // i can run with 200
```

这个 Decorator 就是一个装饰类。这个装饰类相当于在基本类外面套了一层，添加了一些新的功能。而相比于继承，他又不会访问到原生类的属性。

## js 中的装饰器模式

在 ES7 的实验性语法中有一个装饰器模式，即可以通过`@xxxx`的形式，实现装饰器：
这里的装饰器本质就是一个在编译阶段执行的函数，它的目标可以是类、类的方法、类的属性和属性的存取器（getter 和 setter）

```js
// 装饰器函数，它的第一个参数是目标类
function classDecorator(target) {
  target.hasDecorator = true;
  return target;
}

// 将装饰器“安装”到Button类上
@classDecorator
class Button {
  // Button类的相关逻辑
}

// 验证装饰器是否生效
console.log("Button 是否被装饰了：", Button.hasDecorator); // true
```

这里把`@classDecorator`放在 Button 类前面，就相当于在外面套了一层 classDecorator 函数。这个函数会有一些参数传入，可以进行操作，也就相当于实现了上面的 Decorator 类。

> 注意这个语法不能直接使用，必须通过 babel 编译。方法是安装下面的插件：
>
> ```
> yarn add @babel/plugin-proposal-decorators
> ```
>
> 然后在 babel.config.json 中配置：
>
> ```json
> {
>   "presets": ["@babel/preset-env"],
>   "plugins": [["@babel/plugin-proposal-decorators", { "legacy": true }]]
> }
> ```
>
> 在 packages.json 中配置一行命令：
>
> ```json
> "scripts": {
>     "....": "babel test.js --out-file babel_test.js && node babel_test.js",
> },
> ```
>
> 这样就可以运行了

装饰器的具体使用可以参考https://wangdoc.com/es6/decorator.html 。由于是一个实验性的语法，因此不需要太深入了解，看看基本装饰类和装饰类内方法即可。

装饰器的实际使用，还可以应用于 React HOC：

```js
const BorderHoc = (WrappedComponent) =>
  class extends Component {
    render() {
      return (
        <div style={{ border: "solid 1px red" }}>
          <WrappedComponent />
        </div>
      );
    }
  };

// 可以写成这样：
@BorderHoc
class TargetComponent extends React.Component {
  render() {
    // 目标组件具体的业务逻辑
  }
}
```

# 原型模式

原型模式不仅是一种设计模式，它还是一种编程范式（programming paradigm），是 JavaScript 面向对象系统实现的根基。

在原型模式下，当我们想要创建一个对象时，会先找到一个对象作为原型，然后通过克隆原型的方式来创建出一个与原型一样（共享一套数据/方法）的对象。在 JavaScript 里，Object.create 方法就是原型模式的天然实现——准确地说，只要我们还在借助 Prototype 来实现对象的创建和原型的继承，那么我们就是在应用原型模式。

原型模式并不等同于对实例的完全深拷贝。在 JavaScript 中，我们使用原型模式，并不是为了得到一个副本，而是为了得到与构造函数（类）相对应的类型的实例、实现数据/方法的共享。克隆是实现这个目的的方法，但克隆本身并不是我们的目的。换句话说就是 js 中的原型模式的主要体现就在于继承父类的方法，而不一定是完全克隆

因此原型编程范式的体现就是**基于原型链的继承**，即 js 的原型、原型链机制。

如果要实现类似 Java 等强类型语言中的原型模式，比如“模拟 JAVA 中的克隆接口”、“JavaScript 实现原型模式”之类的，那可能就是在问深拷贝

# 适配器模式

适配器本质上就是一个函数，用于适配功能相同，但调用方式和参数不同的几种 api。
比如有一个老的库 A，想要转成更新一点的库 B。但是这两个库的调用方式完全不一样。这时就需要一个适配器，是一个参数和调用方式与 A 完全相同的函数，将参数进行一些处理，转成 B 库调用的方式达成最终目的。

最常见的是 ajax 和 fetch 之间的转换。比如现在有一个封装了 xhr 的简单 ajax：

```js
function Ajax(type, url, data, success, failed)
```

还有一个新的 fetch 封装的库:

```js
class HttpUtils {
  // get方法
  static get(url) {
    return new Promise((resolve, reject) => {
      // 调用fetch
    });
  }
  static post(url, data) {
    return new Promise((resolve, reject) => {
      // 调用fetch
    });
  }
  // ...其他方法
}
```

现在就需要一个适配器，能够按照第一种定义的传参方式，调用第二个类完成任务：

```js
async function AjaxAdapter(type, url, data, success, failed) {
  let result;
  if (type === "GET") {
    result = (await HttpUtils.get(url)) || {};
  } else if (type === "POST") {
    result = (await HttpUtils.post(url, data)) || {};
  }
  // ... 其他api
}

// 用适配器适配旧的Ajax方法
async function Ajax(type, url, data, success, failed) {
  await AjaxAdapter(type, url, data, success, failed);
}
```

# 代理模式

> 出于种种考虑/限制，一个对象不能直接访问另一个对象，需要一个第三者（代理）牵线搭桥从而间接达到访问目的，这样的模式就是代理模式。

开发中最常见的四种代理类型：

- 事件代理：即事件冒泡带来的事件委托，安插在父元素上用于捕获委托的事件处理函数就可以称为代理器
- 虚拟代理：下面详述
- 缓存代理：这里的缓存一般是指函数内部对相同计算结果的缓存，当参数相同时不再重复计算，直接返回计算值即可。
- 保护代理：即“拦截”，通过代理对原数据映射操作，而不是直接操作源数据，相当于保护了原数据。Proxy 就是一种保护代理

## 虚拟代理例

以图片预加载为例。图片预加载不等同于懒加载，预加载主要是为了避免网络不好、或者图片太大时，页面长时间给用户留白的尴尬。常见的操作是先让这个 img 标签展示一个占位图，然后创建一个 Image 实例，让这个 Image 实例的 src 指向真实的目标图片地址、观察该 Image 实例的加载情况 —— 当其对应的真实图片加载完毕后，即已经有了该图片的缓存内容，再将 DOM 上的 img 元素的 src 指向真实的目标图片地址。此时我们直接去取了目标图片的缓存，所以展示速度会非常快

预加载的示例，这里用函数实现，也可以用类：

```js
function preloadImg(url) {
  const virtualImage = new Image();
  virtualImage.src = url;
  virtualImage.onload = () => setImg(url);
}
function setImg(url) {
  document.getElementById("img").src = url;
}
```

> virtualImage 这个对象是一个“幕后英雄”，它始终存在于 JavaScript 世界中、代替真实 DOM 发起了图片加载请求、完成了图片加载工作，却从未在渲染层面抛头露面。因此这种模式被称为“虚拟代理”模式。

## 缓存代理例

其实就是一个缓存函数，可以是原地直接实现：

```js
const addAll = function () {
  console.log("进行了一次新计算");
  let result = 0;
  const len = arguments.length;
  for (let i = 0; i < len; i++) {
    result += arguments[i];
  }
  return result;
};

// 为求和方法创建代理
const proxyAddAll = (function () {
  // 求和结果的缓存池
  const resultCache = {};
  return function () {
    // 将入参转化为一个唯一的入参字符串
    const args = [...arguments].join(",");

    // 检查本次入参是否有对应的计算结果
    if (args in resultCache) {
      // 如果有，则返回缓存池里现成的结果
      return resultCache[args];
    }
    return (resultCache[args] = addAll(...arguments));
  };
})();
```

> 注意这里立即调用函数的使用，其实是一个新思路；如果不想要一个专门的高阶函数，就可以通过这种方式直接获得一个能用的函数，同时还能保留闭包的功能。

也可以用一个高阶函数：

```js
function cache(fn) {
  const cache = new Map();
  return function () {
    const key = [...arguments].join(",");
    if (cache.has(key)) {
      console.log("cache!");
      return cache.get(key);
    }
    const value = fn();
    cache.set(key, value);
    return value;
  };
}
const sum = (a, b) => a + b;
const sumCached = cache(sum);
sumCached(3, 4);
```

# 策略模式

> 策略模式的定义：定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换。

简单来说，就是为了解决一大长串的 if-else。

比如有这样一个函数：

```js
function move(dire) {
  if (dire === "up") {
    console.log("up");
  }
  if (dire === "down") {
    console.log("down");
  }
  if (dire === "left") {
    console.log("left");
  }
  if (dire === "right") {
    console.log("right");
  }
}
```

我们可以变成这样：

```js
const moveMent = {
  up() {
    console.log("up");
  },
  down() {
    console.log("down");
  },
  left() {
    console.log("left");
  },
  right() {
    console.log("right");
  },
};

function move(dire) {
  moveMent[dire]();
}
```

通过函数名来区分不同的决策，将决策封装，增删改都很容易。
当然，这是最简单的一种情况。现实一般是 if 内的判断逻辑很复杂，这时就可以考虑这样做：

```js
// 比如
function move(dir,speed,weight,currentDir,...){
    if(currentDir !== dir && speed > 50 && weight < 100){...}
}
// 这时不能通过直接在对象中取函数名的形式调用
// 可以这样：
const getActionType = (dir,speed,weight,currentDir) => {
    if(currentDir !== dir && speed > 50 && weight < 100) return 'type1'
    if(...) return 'type2'
    ...
}

function move(dir,speed,weight,currentDir,...){
    const moveMent = {
        type1:fn1(),
        type2:fn2(),
        ...
    }
    return moveMent[getActionType(...arguments)]()
}
```

# 观察者模式

观察者模式有一个“别名”，叫发布 - 订阅模式。但两者不完全等同，具体差距在于：

- 观察者模式中，发布者直接触及到订阅者。下面的例子中可以看到，订阅者可以得到发布者的实例
- 发布订阅模式中，发布者不直接触及到订阅者、而是由统一的第三方来完成实际的通信的操作。这个第三方有可能是一个对象，或者是一个“总线”，发布和订阅都在这个“总线”上完成，两者不互相接触。

> 观察者模式定义了一种一对多的依赖关系，让多个观察者对象同时监听某一个目标对象，当这个目标对象的状态发生变化时，会通知所有观察者对象，使它们能够自动更新。

在观察者模式里，至少应该有两个关键角色是一定要出现的——发布者和订阅者。用面向对象的方式表达的话，那就是要有两个类。

示例：

```js
// 发布者类
class Publisher {
  constructor() {
    this.observers = [];
    this.state = 'init'
  }
  // 增加订阅者
  add(observer) {
    console.log("Publisher.add invoked");
    this.observers.push(observer);
  }
  // 移除订阅者
  remove(observer) {
    console.log("Publisher.remove invoked");
    this.observers.forEach((item, i) => {
      if (item === observer) {
        this.observers.splice(i, 1);
      }
    });
  }
  // 通知所有订阅者
  notify() {
    console.log("Publisher.notify invoked");
    this.observers.forEach((observer) => {
      // 这里订阅者可以得到发布者的实例
      observer.update(this);
    });
  }
  changeState(...){
    //... 修改数据的代码
  }
  getState(){
    return this.state
  }
}

// 定义订阅者类
class Observer {
  constructor(name) {
    this.name = name
    console.log("Observer created");
  }

  update(publisher) {
    console.log("Observer.update invoked");
    publisher.changeState(...)
    console.log(publisher.getState())
  }
}

const pb = new Publisher()
pb.add(new Observer('aaa'))
pb.add(new Observer('bbb'))
pb.notify()
```

---

而发布订阅模式，最常见的实现就是实现一个可以注册、触发事件的 EventEmitter 类。这个类在手写题那里有实现，不再赘述。
另外一个实现是类似 PubSub 库，实现起来和 EventEmitter 差不多，最基本 subscribe 和 publish 功能的可以这样实现：

```js
class PubSub {
  constructor() {
    this.subscribes = new Map();
  }
  subscribe(eventName, callback) {
    if (!this.subscribes.has(eventName)) {
      this.subscribes.set(eventName, [callback]);
    } else {
      this.subscribes.set(
        eventName,
        this.subscribes.get(eventName).push(callback)
      );
    }
  }
  publish(eventName, payload) {
    for (let [event, callbacks] of this.subscribes.entries()) {
      if (event === eventName) {
        callbacks.forEach((callback) => {
          callback(payload);
        });
      }
    }
  }
}

const pubsub = new PubSub();
pubsub.subscribe("getValue", (val) => console.log(val));
pubsub.publish("getValue", 111); // 111
```

# 其他设计模式

## 迭代器模式

> 迭代器模式提供一种方法顺序访问一个聚合对象中的各个元素，而又不暴露该对象的内部表示。
> 迭代器模式是设计模式中少有的目的性极强的模式。所谓“目的性极强”就是说它不操心别的，它就解决这一个问题——遍历。

在 js 中，迭代器的实现可以参考 Symbol.iterator 相关知识。迭代器模式的实现实际上就是迭代器的应用，让一个普通对象可以通过迭代器的添加，变得可以迭代。

## 状态模式

在类中维持一个状态，根据状态的不同，类内的方法会做出不同的操作；
状态模式的最佳实践就是 Promise，当 Promise 中的状态改变时，相应的方法的触发也会不同。
