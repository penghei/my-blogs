---
title: redux 和 react-redux的使用经验
date: 2021-10-27 00:15:44
tags: 日常学习
categories: React
cover: /img/redux.png
---

# 写在前面

这两天被 redux 和 react-redux 真的算是折腾的够呛,总算是勉强搞明白了一些.这么比下来 vuex 简直像个天使.看来说这两个东西能不用就不用是对的,因为实在是太太太麻烦了
这篇博客可能不是很标准,但是是我自己用的最习惯并且是优化到最佳的,在这里记录一下

# 纯 redux

## 原理

![redux原理图.png](https://i.loli.net/2021/10/27/3I1pfGW6iLdqv2y.png)

## 项目结构

![redux.png](https://i.loli.net/2021/10/27/6D9AJ5ySiFx2jL1.png)
与正常组件不同的地方:

- redux 这个文件夹下的全部
- 在 index.js 中需要有一些改动(新增在后面)

```js
store.subscribe(() => {
  ReactDOM.render(<App />, document.getElementById("root"));
});
```

相当于是监听 store 的变化,如果变化就引入.
接下来介绍 redux 中的文件

## redux 新增

### store.js

目的是创建一个连接组件和 reducer 的中间件,是 redux 的核心,并且一个项目只能有一个 store

```js
import { createStore } from "redux";
import MyReducer from "./reducer/MyReducer";

export default createStore(MyReducer);
```

引入 reducer 并创建 store
如果有多种不相关的数据需要存储,就会有多个 reducer,可以用`combineReducers`来合成

```js
import { combineReducers, createStore } from "redux";
import todoReducer from "./reducer_todos";
import inputReducer from "./reducer_inputs";

const Reducers = combineReducers({
  todoReducer,
  inputReducer,
});
export default createStore(Reducers);
```

注意合成之后的特点:

- 原先的 dispatch 不会受影响,相当于是同时在多个 reducer 中寻找 type,因此不能相同;
- 原先的 getState()不再是一个直接的返回值,而是一个对象如下:

```js
{
    todoReducer:[],
    inputReducer:""
}
```

即按照 combine 的索引加了一层包裹,每个值是原来的返回值

### reducer

reducer 是存储数据的文件,大致写法如下:

```js
export default function todoList(pre = [], act) {
  const { type, data } = act;
  switch (type) {
    case "addToList":
      return [...pre, data];
    default:
      return pre;
  }
}
```

这里用了一个 todolist 的 list 举例子.大致是

- 一个参数为(pre,action)的函数
- 利用结构赋值从 action 中获取 type 和传的数据(数据的键名不一定是 data,可以自己定),也就是 dispatch 写的对象
- switch 判断 type 并返回值
  然后接下来是几个重点:
- **pre 一定要有**,并且要有初值.如果是数组最好把增删操作直接在 reducer 完成,如果只是想单纯存值可以只返回 data;但是注意一定不能少 pre,因为**pre 相当于每次存储调用 reducer 后的数据,也就是说没有 reducer 就不会真正的存储数据**,所以一定要有,并且要确定好和 data 的交互
- return 返回的值有两个去向,一个是更新了 pre,一个是返回给 getState(),如果调用 getState()获得的就是这个返回值
- reducer 可以有多个,命名要清晰,大致结构都差不多,注意 type 名不要重复

### dispatch

这个严格来说不是 redux 文件夹要写的,而是写在正文的组件中的;但是和上面几个关系密切,于是写在这里

```js
store.dispatch({ type: "", data: [] });
```

参数是一个包含 type 和自定义数值的对象,效果是给 store 发送一个数据,根据 type 在 reducer 中进行判断,获取返回值
另外,官方推荐用 action 封装 dispatch 的写法,个人认为直接写相对会简单很多

## 共享数据

比如有 A 组件和 B 组件希望获取 C 组件的一个数组,并且已经有了 store 和 c 的 reducer,那么流程如下:

- C 组件 dispatch 该数组 ==> C 组件的 store 收到并执行操作(单纯存储的话直接 return 就行) ==> A 和 B 组件调用 getState()(如果有 combine 就是上面说的对象调用) ==> 使用
- 如果 A 和 B 希望修改该数组影响 C,则应该把 c 中的数组直接放在 redux 上而 C 中不再留该数组,调用的时候 getState 即可
- 这是需要理解清楚的最重要一点:**因为单纯存数据会可能产生奇奇怪怪的 bug 并且很麻烦,所以应该把共享高的数据直接放在 reducer 中,组件都是直接通过 getState 和 dispatch 使用**
- 比如之前的 vue 购物车案例,如果需要 redux+react 写的话,购物车/选择的商品/首页商品列表/用户信息这些都需要直接放在 redux 中

# react-redux

react-redux 比 redux 多了一层容器组件,其他的地方变化并不大.总的来说是更加麻烦了,但是该会还是得会

## 原理

![react-redux模型图.png](https://i.loli.net/2021/10/27/IjBOEA6LPthfGMg.png)
个人解释:在原先 ui 组件外面加了一层容器组件(通过 connect 方法而不是直接编写组件),所有与 redux 的操作都直接和容器组件交互,ui 组件中没有 redux 的方法;容器组件通过 props 把 redux 的数据和操作数据的方法传递给 ui 组件使用

## 项目结构

![react-redux.png](https://i.loli.net/2021/10/27/fNZAoUX95BGzROT.png)
跟 redux 相比只多了一个 container,就是容器组件.当然经过最优化后是不需要单独写一个 container 的,接下来会说

## react-redux 新增

### store,reducer,(actions)

全部不变,即原先 redux 中的都不变

### index.js

不再需要通过订阅 store 来监听渲染了,而是通过 Provider 包裹 app 组件,使得每一个组件都可以接收到 store

```js
import { Provider } from "react-redux";
import store from "./redux/store";

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("root")
);
```

解释一下就是:Provider 给根组件传了一个 store,意味着所有项目中的组件都可以调用 store 相关的方法

### container

这个是重点:包裹 ui 组件的容器组件是怎么生成的

```js
import CountUI from "../components/Count"; //引入ui组件
import { connect } from "react-redux";

function mapStateToProps(state) {
  //给ui组件的props加一个属性,可以用props.count获取这个值,相当于原来的getState()
  return { count: state };
}
function mapDispatchToProps(dispatch) {
  //同理,相当于原来的dispatch
  return {
    setNumber: (value) => dispatch({ type: "SETNUMBER", data: value }),
  };
}

export default connect(mapStateToProps, mapDispatchToProps)(CountUI);
```

- 首先,引入原来写的 Count 组件和一个 connect
- connect 接受两个参数:一个返回值对象的函数,一个返回操作值方法的对象的函数.说白了就是上面注释里写的
- 两个参数都是函数,每个函数返回一个对象,一共会返回一个值和一个函数,会放在后面 ui 组件的 props 中.所以这个 pros 大概长这样:

```
    {
        store:{...},
        count:'',
        setNumber:f
    }
```

    所以键名就是获取值的键名,即props.count;方法就是props.setNumber()

- 注意的是这个两个函数的参数:state 就是 reducer 的 getState 值,而 dispatch 也是 redux 操作的方法.两者都是通过参数引入进来的,是 react-redux 的语法糖
- 后面的括号是调用并传入原来的 ui 组件,没啥说的
  当然可以把函数直接放进去,就可以写成这样

```js
import CountUI from "../components/Count"; //引入ui组件
import { connect } from "react-redux";

export default connect(
  (state) => ({ count: state }), //state是传给ui组件的getState值
  (dispatch) => ({
    setNumber: (value) => dispatch({ type: "SETNUMBER", data: value }),
  })
)(CountUI); //创建容器组件
```

然后既然要引入 ui 组件,那最简单的方式不就是直接放在原来的 Ui 组件里吗
所以更直接的方法,不需要 container 单独写出来,直接放在需要包裹的组件下面.
完整示例:

```js
import React, { useState, useEffect } from "react";
import { connect } from "react-redux";
function Count(props) {
  const [sum, setSum] = useState(0);
  function setNumber() {
    setSum((sum) => sum + 1);
  }
  useEffect(() => {
    props.setNumber(sum);
  });
  return (
    <div>
      <button onClick={setNumber}>setNumber</button>
      {props.value.listReducer === [] ? (
        <h1>empty</h1>
      ) : (
        props.value.map((obj) => (
          <h1>
            {obj.name}:{obj.age}
          </h1>
        ))
      )}
    </div>
  );
}
export default connect(
  (state) => ({ value: state }),
  (dispatch) => ({
    setNumber: (value) => dispatch({ type: "SETNUMBER", data: value }),
  })
)(Count);
```

可以很清晰看出这个示例中,通过 props.value 获取 state 值,通过 setNumber 触发 dispatch,在 ui 组件中不需要直接操作 redux 的方法

## 共享数据

由于相对 redux 只是加了一层组件,所以其实并没有太大变化.
如果一个组件只需要访问值,就不需要 dispatch,同理也可以不需要 state

# 总结

react-redux 真的算是复杂到了极致,和 react 的原生路由有的一拼.个人感觉日常开发纯 redux 已经够用,react-redux 只是锦上添花而已.但是使用 redux 还是有几个注意点

1. 改变之前"只是存一个数据"的想法.redux 希望的更多是把数据放在上面而不是原来的组件里边,所有增删改查都直接在 reducer 中完成,尤其是数组和对象
2. pre 值很重要!!!它是初始值和每一次 return 后的新值,不能随便不要.并且 return 的值也要谨慎
3. react-redux 的 dispatch 钩子最好不要在生命周期调用.同理 redux 的 dispatch 也最好不要在声明周期或者 useEffect 使用,不然会出奇奇怪怪的 bug
