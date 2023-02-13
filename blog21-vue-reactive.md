---
title: vue3响应式
date: 2021-11-28 19:40:40
tags: 日常学习
categories: Vue
cover: /img/vue3.png
---

# 原理（根据文档个人理解）

首先,响应式是需要解决这样一种问题:

```js
let a = 1;
let b = 2;
let sum = a + b;
sum; //3
b = 3;
sum; //还是3
```

在原生 js 里边,由于 sum 实际上是 a+b 的索引,所以改变 a,b 并不会改变 sum
而响应式就是希望 a,b 改变时,sum 也会相应地改变
当然,扩展到视图层的响应式不只是改变值,而是在值变化的时候去更新页面

要实现响应式更新,vue 采用了这样一种思路

> 1. 当一个值被读取时进行追踪
> 2. 当某个值改变时进行检测
> 3. 重新运行代码来读取原始值

这是官方文档上面的内容,而具体实现上,可以理解为这样:

1. 读取一个值时,正常返回这个值,并且执行相应操作（比如视图渲染）
2. 当值改变时,查找哪些依赖于这个值的方法并执行（比如更新视图）,并且执行更改操作
   所以我们就需要在读写值之前对值进行操作,能实现这种操作的就是`Proxy`,也就是代理
   看一个代理的简单例子:

```js
const data = {
  name: "xiaoming",
};
const handler = {
  get(target, property) {
    //target:获取的目标;property:获取值
    track(target, property);
    return target[property];
  },
  set(target, property, value) {
    //target:获取的目标;property:获取值;value:要更改的值
    track(target, property);
    target[property] = value;
  },
};
const dataProxy = Proxy(data, handler);
```

这里的 Proxy 拦截了对于 data 属性的 get 和 set 方法（当然也可以有 delete,没写出来）,当对 dataProxy 操作时 data 也会正常改变.vue 会在真正更改值之前做一些事,比如更新视图（官网上叫做**执行副作用**）.  
但是,如果直接更改属性,在属性不存在或者不能被更改的时候会报错;对于封装框架来说报错意味着可能会崩溃;如果使用 trycatch 又会导致代码量增加  
所以 vue 采用了 Reflect,一种独特的更新方式

```js
const data = {
  name: "xiaoming",
};
const handler = {
  get(target, property) {
    track(target, property);
    return Reflect.get(target, property);
  },
  set(target, property, value) {
    track(target, property);
    return Reflect.set(target, property, value);
  },
};
const dataProxy = Proxy(data, handler);
```

reflect 主要意义在于:**当 get 或 set 失败时,会通过返回值 false 报错,而不是直接崩溃**,这样就要比满篇的 trycatch 好多了  
由此完成响应式,当响应式数据更新时视图会被更新（类似 react 中的 state）

# ref & reactive

| 区别点   | ref                                            | reactive                |
| -------- | ---------------------------------------------- | ----------------------- |
| 数据类型 | 基本数据类型                                   | 对象&数组               |
| 原理     | 通过 Object.defineProperty()的 get 和 set 实现 | 通过 Proxy&Reflect 实现 |
| 使用     | 需要.value                                     | 不需要.value            |

# 数组和对象

一般数组和对象都需要 reactive 包裹;在包裹下,setup 中的该对象和数组实际上并不是原值,而是一个代理对象
如果把普通的对象或者经过函数操作过后的对象直接赋给原数组,就会导致响应式消失,从而不能改变
举个例子:

```js
const todoList = reactive(['wake up','have breakfast'])
(()=>{
  todoList = todoList.filter(todo=>todo!=='wake up')
}();)
```

这种是会改变 todoList 的,但是由于 filter 返回的是一个纯数组,所以不会触发响应式,而且执行之后 todoList 也变成了普通数组
所以改变数组并更新渲染的方法就是使用一些改变数组的操作:
`push()`
`pop()`
`shift()`
`unshift()`
`splice()`
`sort()`
`reverse()`
`直接改变数组元素和map()方法`
但是如果用 ref 包裹数组或对象,就可以通过直接赋值的方法改变,同理相应的 push,pop 等方法也可以实现;**不过不建议使用 ref 包裹对象和数组**

# 其他（常用）API

| api                           | 描述                                                                                                                 | 用法（括号表示`xxx()`）                                            |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| readonly                      | 把响应式对象改成只读                                                                                                 | `()`                                                               |
| isProxy/isReactive/isReadonly | 返回是否是这几个的类型                                                                                               | `()`                                                               |
| toRaw                         | 把响应式数据改成普通数据                                                                                             | `()`                                                               |
| makeRaw                       | 标记一个对象，使其永远不会转换为 proxy                                                                               | `()`                                                               |
| unRef                         | 参数是 ref 就返回 ref 的值,不是就返回参数本身                                                                        | `()`                                                               |
| toRef                         | 把**对象**的选定的属性改成单个 ref，参数是**响应式对象**和要转换的键名，返回这个值的 ref                             | `toRef({name:'zzx'},'name')`                                       |
| toRefs                        | 把**对象**的所有键转成单个 ref,参数是**响应式对象**，返回普通对象，内部是具有响应式的 ref                            | `toRefs({name:'zzx',age:18})`                                      |
| computed                      | 把参数函数的返回值变成不可变的 ref 对象，一般可以用于把普通值转为响应式值                                            | `computed(()=>{})`                                                 |
| watchEffect                   | 类似`useEffect`，但是会自动跟踪所有内部变量                                                                          | `watch(()=>{})`                                                    |
| watch                         | 监听一个**响应式对象**，并执行操作；监听是深的，对象内部不管嵌套多深改变都会被监听；也可以监听一个数组，里边有多个值 | `watch(xxx,(nv,ov)=>{})`或`watch([a,b],([anv,aov],[bnv,bov])=>{})` |
| provide&inject                | 类似 react 的 Context                                                                                                | 见下                                                               |
| setup                         | **在组件被创建之前**，props 被解析之后执行                                                                           | `setup(props,context){}`                                           |
