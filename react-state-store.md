---
title: react状态管理
date: 2022-01-15 17:03:25
tags: 日常学习
cover: /img/recoil.png
categories: React
---

# Redux

## 基础概念

首先是 Redux 的一些结构：
![](https://pic.imgdb.cn/item/63b64b07be43e0d30e702dc8.jpg)

### store

用于保存 state 的一个对象。在一个 Redux app 中，通常只有一个 store，可以有多个 reducer 表示不同的状态的持有。就像一个 React 组件中可以有多个 useState，但是 fiber 只能有一个。

store 对象是一些函数的集合：

```ts
const store = {
  dispatch: dispatch as Dispatch<A>,
  subscribe,
  getState,
  replaceReducer,
  [$$observable]: observable,
};
```

store 对象内部并没有一个“state”属性，只有在调用函数时才会使用这个 state。
state 实际上是 createStore 函数内部的一个变量 currentState，而其他函数都是使用这个变量的闭包；

```ts
function createStore(reducer) {
  let currentState = preloadedState as S;
  // ...
  function getState(): S {
    return currentState as S;
  }
  // ...
  currentState = currentReducer(currentState, action);
}
```

### reducer

一个接收 state、返回 state 的函数。在 redux 中还有第二个参数 action，表示要更改的类型（type）和携带的数据（payload）。经过逻辑处理后，返回的 state 将作为新的 state 使用。

reducer 不能直接修改 state（state 是对象或数组），而是要复制 state，并对复制的值做修改。不能修改的主要原因是要符合 Redux“数据不可变”的理念。

> 其实纯 redux 并没有对 reducer 内部的操作做出限制，源码中也只是调用获取 state：
>
> ```ts
> try {
>   isDispatching = true;
>   currentState = currentReducer(currentState, action);
> } finally {
>   isDispatching = false;
> }
> ```
>
> 但维持这种形式，主要目的是维持其他功能的有效性，比如状态的回溯、测试等。

reducer 可以有多个，但是最终 redux 会将其合为一个，作为一个“rootReducer”。从 createStore 函数内部可以看出，参数只能是一个 reducer，调用也只调用了一次。
如果有多个 reducer，那就需要 combineReducer 合并了。

### action

action 是一个包含 type 和 payload?的对象。
action 本身比较简单，但是有个问题是，为什么 redux 一定需要一个 action？

显然直接调 dispatch 传入状态也是完全可以的：

```js
dispatch(someState);
```

主要原因有几个：

1. action 体现了 redux 对状态的可控性、复现性。也就是说，action 的存在使得 redux 可以更好地追踪状态的流向，可以很轻易地跟踪、捕获和预测状态变化。

如果只是一个简单的数据而不是 action，当通过 dispatch 调度时就很难确定这个状态到底是来自哪里、归属于哪个 reducer、哪个状态里的修改。
比如一个 action 是这样：

```js
{type: 'todos/todoAdded', payload: todoText}
```

这样很容易就能确认，这个 action 是为了 todos 状态服务的，且要修改的操作为 todoAdded。
当 redux 进行调试、测试时，也能很容易捕获状态以及其所指向的状态对象。

2. action 配合 reducer 内的条件语句，才能达成更好的状态修改，以保证状态不可变。

假如我们只是给 reducer 传一个状态

```js
reducer(someState);
```

那么这个状态是干什么的呢？它应该怎么创建新的 state？这个传入的参数只是一个变量，还是原来的整个 state？显然这些都很难确定。
reducer 的存在就是为了服务于 redux“状态不可变”的理念。redux 要求 reducer 必须传出一个新的状态用以替代原状态，那么 reducer 就需要知道具体对状态是怎么操作的。

### state

redux 的核心数据，也是数据的基本单位，即“状态”。状态不能手动直接改变，唯一改变的方式就是调用 dispatch，触发 reducer，并在 reducer 内部做出修改。

**Redux 的状态只能为对象或数组**，其他的类型都不可以，包括 map、set、一般数据类型。

state 是不可变的，每一次对 state 的更改都是以一个新的值替换旧的 state，而不是修改 state。

```ts
// 不能这样
const state = store.getState();
state.count = 2;

// reducer中也不能直接修改
function reducer(state, action) {
  state.count = 1;
  return state;
}
```

但是这两种操作本身的逻辑是没有问题的。也就是说，直接修改了 state 确实会使 state 改变，但 redux 限制这种做法。

### 其他

- subscriber：当 state 发生改变时，store 会通知所有的订阅者，将最新的 state 传给他。在 React-Redux 中，订阅者就是 React 组件
- selector：一个“取 state”的函数。因为 state 通常是对象，selector 的作用就是把 state 的某个属性提取出来，比如：

  ```js
  const selectCounterValue = (state) => state.value;
  const currentValue = selectCounterValue(store.getState());
  console.log(currentValue);
  ```

### middleWare

即“中间件”，类似于 express 这种中间件的形式。在 redux 中中间件主要是在调用 dispatch 方法时添加操作
比如：

```ts
const middlewareEnhancer = applyMiddleware(print1, print2, print3);
const store = createStore(rootReducer, middlewareEnhancer);
store.dispatch({ type: "todos/todoAdded", payload: "Learn about actions" });
// log: '1'
// log: '2'
// log: '3'
```

如果需要编写 middeware，redux 提供了一个范式，即所有的 middleware 其实都是这种形式的：

```ts
function exampleMiddleware(storeAPI) {
  return function wrapDispatch(next) {
    return function handleAction(action) {
      // 这里是主要逻辑
      return next(action);
    };
  };
}
```

- exampleMiddleware：即中间件本身。比如上面的 print1、print2 这几个函数就是这样。参数 storeAPI 是 store 的一部分，只包含 dispatch 和 getState 方法
- wrapDispatch：用于传递 dispatch。参数 next 可能是原本的 store.dispatch 函数，也可能是下一个中间件函数，调用 next 函数会将 action 传递给下一个中间件，或直接调用真实的 dispatch 结束中间件的调用。
- handleAction：中间件实际执行逻辑的地方，参数是本次调用的 action。

这三个函数没有明确的函数名，甚至可以这样写：

```ts
const anotherExampleMiddleware = (storeAPI) => (next) => (action) => {
  // 其他逻辑
  return next(action);
};
```

在编写 middleware 时，可以看作是一个函数被传入了三个参数，即 store、dispatch 和 action

举个例子：

```ts
const loggerMiddleware = (storeAPI) => (next) => (action) => {
  console.log("dispatching", action);
  let result = next(action);
  console.log("next state", storeAPI.getState());
  return result;
};
```

中间件的返回值

## api

### createStore

createStore redux 中通过 createStore 可以创建一个 Store ，使用者可以将这个 Store 保存传递给 React 应用，具体怎么传递那就是 React-Redux 做的事了。

```js
const Store = createStore(rootReducer, initialState, middleware);
```

createStore 函数实际上是几个函数的集合。在函数内部创建了 currentReducer、currentState 和 currentListeners 这三个关键对象，后面将作为其他函数的闭包使用。
这几个关键变量的含义：

- currentReducer：即传入的 reducer
- currentState：传给 reducer 的 state，同时也作为 reducer 的返回值，getState 返回的就是这个 state
- currentListeners：一个 listeners 数组，表示所有的 listeners。当调用 dispatch 时会遍历这个数组并执行每个 listener；
- nextListeners：currentListeners 的副本，调用 subscribe 时实际上是向 nextListeners 内放入 listener，最后会将 nextListeners 赋给 currentListeners
- isDispatching：一个类似于 react 中批量更新开关的变量。表示当前“正在执行 dispatch”，具体来说是正在执行 reducer。如果 isDispatching 为 true，就不能执行其他操作，比如 subscribe、getState 等。主要是保证 reducer 执行过程的稳定

然后就是几个关键的函数，下面会分章节说

```ts
export default function createStore(
  reducer: Reducer<S, A>,
  preloadedState?: PreloadedState<S> | StoreEnhancer<Ext, StateExt>,
  enhancer?: StoreEnhancer<Ext, StateExt>
): Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext {
  let currentReducer = reducer;
  let currentState = preloadedState as S;
  let currentListeners: (() => void)[] | null = [];
  let nextListeners = currentListeners;
  let isDispatching = false;

  function getState(): S {}

  function subscribe(listener: () => void) {}

  function dispatch(action: A) {}

  function replaceReducer(nextReducer): Store {}

  // 这个type的action会将所有的reducer初始化，将他们持有的state初始化
  dispatch({ type: ActionTypes.INIT } as A);

  const store = {
    dispatch: dispatch as Dispatch<A>,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable,
  } as unknown as Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext;
  return store;
}
```

#### getState

非常简单，就是返回 currentState
唯一需要注意的是，在 isDispatching 期间不能获取。

```ts
function getState(): S {
  if (isDispatching) {
    throw new Error(
      "You may not call store.getState() while the reducer is executing. " +
        "The reducer has already received the state as an argument. " +
        "Pass it down from the top reducer instead of reading it from the store."
    );
  }

  return currentState as S;
}
```

#### subscribe

核心内容就是添加 listener 到 nextListeners 数组中去，返回一个 unsubscribe 函数用于卸载本次添加的 listener

```ts
function subscribe(listener: () => void) {
  let isSubscribed = true;
  // 复制currListeners，主要目的是防止dispatch期间出现的问题
  ensureCanMutateNextListeners();
  nextListeners.push(listener);

  return function unsubscribe() {
    if (!isSubscribed) {
      return;
    }
    isSubscribed = false;
    ensureCanMutateNextListeners();
    const index = nextListeners.indexOf(listener);
    nextListeners.splice(index, 1);
    currentListeners = null;
  };
}
```

#### dispatch

传入一个 action，内部调用 reducer 获取最新 state，然后逐个调用 listener

```ts
function dispatch(action: A) {
  try {
    // 执行currentReducer的过程就是isDispatching的过程
    isDispatching = true;
    currentState = currentReducer(currentState, action);
  } finally {
    isDispatching = false;
  }
  // 执行listeners
  const listeners = (currentListeners = nextListeners);
  for (let i = 0; i < listeners.length; i++) {
    const listener = listeners[i];
    listener();
  }

  return action;
}
```

#### replaceReducer

替换当前的 reducer。
官方的注释说，这个函数主要用于动态加载或更新 reducer。他会将原本的 reducer 替换为传入的 reducer，然后返回一个 store

```ts
function replaceReducer<NewState, NewActions extends A>(
  nextReducer: Reducer<NewState, NewActions>
): Store<ExtendState<NewState, StateExt>, NewActions, StateExt, Ext> & Ext {
  currentReducer = nextReducer;
  // store初始化会做的操作
  dispatch({ type: ActionTypes.REPLACE } as A);
  return store;
}
```

比如官方示例的热重载功能，即当在开发环境下修改了 reducer 后，如果不添加热重载，就需要刷新去重新读取新的 reducer；而热重载可以监听 reducer 的变化，如果发生改变就调用 replaceReducer 更新 reducer。

其中，module.hot.accept 方法来自于 webpack 提供的 api，用以监听哪个模块发生了变动，应该被重载。

```ts
import { applyMiddleware, compose, createStore } from "redux";
import thunkMiddleware from "redux-thunk";

import monitorReducersEnhancer from "./enhancers/monitorReducers";
import loggerMiddleware from "./middleware/logger";
import rootReducer from "./reducers";

export default function configureStore(preloadedState) {
  const middlewares = [loggerMiddleware, thunkMiddleware];
  const middlewareEnhancer = applyMiddleware(...middlewares);

  const enhancers = [middlewareEnhancer, monitorReducersEnhancer];
  const composedEnhancers = compose(...enhancers);

  const store = createStore(rootReducer, preloadedState, composedEnhancers);

  if (process.env.NODE_ENV !== "production" && module.hot) {
    module.hot.accept("./reducers", () => store.replaceReducer(rootReducer));
  }

  return store;
}
```

#### preloadedState

可以作为 createStore 的第二个参数传入，表示 state 的初始值。
虽然 reducer 本身也有一个初始值，但由于 createStore 的创建时机，这时可以进行进行一些预操作，比如取 localStorage 等。

```ts
let preloadedState;
const persistedTodosString = localStorage.getItem("todos");

if (persistedTodosString) {
  preloadedState = {
    todos: JSON.parse(persistedTodosString),
  };
}

const store = createStore(rootReducer, preloadedState);
```

#### enhancer

作为创建的 store 的“增强剂”，enhancer 是一个接受原 createStore 函数，并返回一个“增强”之后的 createStore 函数的函数。
enhancer 函数接受旧的 reducer，会返回一个新的 reducer，并且新的 reducer 内部会调用旧的 reducer。这样，就相当于在旧 reducer 的基础上，添加了一些 case 用以强化原 reducer。
比如一个添加撤销效果的 enhancer，他可以接受任意一个普通的 reducer，在原本 reducer 基础上添加 undo 和 redo 两个 case，从而添加撤销和重做功能。原本的 case 和 state 依然存在

```ts
function undoable(reducer) {
  const initialState = {
    past: [],
    present: reducer(undefined, {}),
    future: [],
  };
  // 返回的新reducer
  return function (state = initialState, action) {
    const { past, present, future } = state;

    switch (action.type) {
      case "UNDO":
        const previous = past[past.length - 1];
        const newPast = past.slice(0, past.length - 1);
        return {
          past: newPast,
          present: previous,
          future: [present, ...future],
        };
      case "REDO":
        const next = future[0];
        const newFuture = future.slice(1);
        return {
          past: [...past, present],
          present: next,
          future: newFuture,
        };
      default:
        // 在内部调用旧reducer，保证兼容原本reducer的情况
        // 旧reducer返回值就是原本的state，原来的state也要和当前的state合并
        const newPresent = reducer(present, action);
        if (present === newPresent) {
          return state;
        }
        return {
          past: [...past, present],
          present: newPresent,
          future: [],
        };
    }
  };
}
```

enhancer 的调用，具体来说是这样：

```ts
if (typeof enhancer !== "undefined") {
  return enhancer(createStore)(reducer, preloadedState);
}
```

可以看到 enhancer 的返回值其实就是 createStore 函数的强化版

多个 enhancer 可以通过 compose 函数合并：

```ts
import {
  sayHiOnDispatch,
  includeMeaningOfLife,
} from "./exampleAddons/enhancers";

const composedEnhancer = compose(sayHiOnDispatch, includeMeaningOfLife);

const store = createStore(rootReducer, undefined, composedEnhancer);
```

### combineReducers

正常状态可以会有多个 reducer ，combineReducers 可以合并多个 reducer。

```js
/* 将 number 和 PersonalInfo 两个reducer合并   */
const rootReducer = combineReducers({
  number: numberReducer,
  info: InfoReducer,
});
```

combineReducers 接受一个对象，其中键名将成为根状态对象中的键。这个键很重要，在 combineReducers 内部就是通过键来分配 state 的。

combineReducers 函数代码简化如下：

```ts
export default function combineReducers(reducers: ReducersMapObject) {
  // 复制reducers对象到finalReducers
  const reducerKeys = Object.keys(reducers);
  const finalReducers: ReducersMapObject = {};
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i];
    if (typeof reducers[key] === "function") {
      finalReducers[key] = reducers[key];
    }
  }
  const finalReducerKeys = Object.keys(finalReducers);

  // 这个地方是检查每个reducer有没有问题，比如类型不是函数、返回undefined等
  let shapeAssertionError: unknown;
  try {
    assertReducerShape(finalReducers);
  } catch (e) {
    shapeAssertionError = e;
  }

  // 返回一个combination函数，即新的reducer，参数和单个reducer相同
  return function combination(
    // 这里的state被合并成了一个大的state。下面会说
    state: StateFromReducersMapObject<typeof reducers> = {},
    action: AnyAction
  ) {
    let hasChanged = false;
    // 新的state
    const nextState: StateFromReducersMapObject<typeof reducers> = {};
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i];
      const reducer = finalReducers[key];
      const previousStateForKey = state[key];
      // 遍历所有的reducer，把对应的state传进去
      const nextStateForKey = reducer(previousStateForKey, action);
      nextState[key] = nextStateForKey;
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey;
    }
    hasChanged =
      hasChanged || finalReducerKeys.length !== Object.keys(state).length;
    // 返回新的state，和原本的state相同结构
    return hasChanged ? nextState : state;
  };
}
```

combineReducers 的返回值 combine 函数实际上就是一个 reducer。这个 reducer 并不会被用户调用，而是和其他所有的 reducer 一样，在 dispatch 函数内部被调用。

如果使用了 combineReducers，会发现它返回的 state 并不是一个单一的 state，而是一个由 key 标记的 state 对象；
比如原本有 count 和 color 两个 state：

```ts
const countState = {
  count: 1,
};

const colorState = {
  color: "red",
};
```

合并之后就成了：

```ts
const state = {
  count: countState,
  color: colorState,
};
```

state 的合并也是在 combineReducers 中完成的，即 combineReducers 函数的返回值 nextState 本身就是一个合并的 state。
这也就是为什么说 combineReducers 传入的参数对象中的键很重要，因为这些键将会作为合并 state 的键；当调用`store.getState`时，返回的也是这样一个合并对象。

剩下的操作就是：遍历合并 reducer 对象中的每个 reducer，传入对应的 state，得到新的 state 单个，再插入到新的合并 state 对象上。
每个合并 reducer 对象上每个 reducer 的键和合并 state 对象上每个 state 的键是相同的。

### applyMiddleware

将中间件合并，返回一个 enhancer。
要说这个函数，主要要先讲一下中间件的概念。在 redux 中，中间件实际上是对 dispatch 方法的包装，即用一个函数包装 dispatch 方法，在调用 dispatch 之前先调用该中间件内的逻辑。

```ts
export default function applyMiddleware(
  ...middlewares: Middleware[]
): StoreEnhancer<any> {
  return (createStore: StoreEnhancerStoreCreator) =>
    (reducer, preloadedState) => {
      const store = createStore(reducer, preloadedState);
      let dispatch: Dispatch = () => {
        throw new Error(
          "Dispatching while constructing your middleware is not allowed. " +
            "Other middleware would not be applied to this dispatch."
        );
      };

      const middlewareAPI: MiddlewareAPI = {
        getState: store.getState,
        dispatch: (action, ...args) => dispatch(action, ...args),
      };
      const chain = middlewares.map((middleware) => middleware(middlewareAPI));
      dispatch = compose<typeof dispatch>(...chain)(store.dispatch);

      return {
        ...store,
        dispatch,
      };
    };
}
```

下面详细介绍一下这个函数。
首先，这个函数的返回值是一个 enhancer；上面说过 enhancer 实际上是接受 createStore 函数，返回一个新的 createStore 函数，新的 createStore 函数也要创建并返回一个 store 对象。因此这个函数的这些部分实际上是 enhancer 内容：

```ts
return (createStore: StoreEnhancerStoreCreator) =>
  (reducer, preloadedState) => {
    // ...实际的逻辑
    return {
      ...store,
      dispatch,
    };
  };

// 调用
enhancer(createStore)(currReducer, preloadedState);
```

抽出内部的内容，才是 applyMiddleware 真正的执行内容

```ts
const store = createStore(reducer, preloadedState);
let dispatch: Dispatch = () => {
  throw new Error(
    "Dispatching while constructing your middleware is not allowed. " +
      "Other middleware would not be applied to this dispatch."
  );
};

// 这个对象将作为中间件函数的外层函数的参数，即storeAPI
const middlewareAPI: MiddlewareAPI = {
  getState: store.getState,
  // 注意这个dispatch，后面的dispatch实际上是上面用let定义的dispatch，调用会报错
  dispatch: (action, ...args) => dispatch(action, ...args),
};
// chain是一个middleware数组，每一项都是middleware的第二层的那个函数（即接收next参数）
const chain = middlewares.map((middleware) => middleware(middlewareAPI));
// 关键，调用compose函数把chain内的函数合并起来，并传入真正的dispatch
dispatch = compose(...chain)(store.dispatch);
```

上面说过 middleware 的第二层实际上是这样：

```ts
function wrapDispatch(next) {
  return function handleAction(action) {
    return next(action);
  };
}
```

因此 chain 数组内的每一项，都是这样一个函数；

那么 compose 函数怎么做的呢？

```ts
export default function compose(...funcs: Function[]) {
  return funcs.reduce(
    (a, b) =>
      (...args: any) =>
        a(b(...args))
  );
}
```

比如说有两个函数 a 和 b，那么 compose 的返回就是一个这样的函数：

```ts
const a = (num) => num++
const b = (num) => num--

compose(a,b) === (num) => a(b(num)) === (num) => a(num--) === (num) => (num--)++
```

compose 的返回值是一个函数，这个函数接收的参数将会作为最内部的 compose 函数的参数调用，然后最内部的函数的返回值交给下一个函数，依次类推。

再举个栗子，下面这里用到了 compose 来合并两个 print 函数，合并的结果是是什么？

```js
const compose = (...fn) =>
  [...fn].reduce(
    (f, g) =>
      (...args) =>
        f(g(...args))
  );
const print1 = (next) => (action) => {
  const res = next(action); // 这个next实际上就是print2
  return res;
};
const print2 = (next) => (action) => {
  const res = next(action);
  return res;
};
const fnComposed = compose(print1, print2);
fnComposed();
```

应该是这样：

```js
const print1 = (next) => (action) => {
  const res = next(action);
  return res;
};
const print2Fn = (action) => {
  ...
}
fnComposed
= (...args) => print1(print2Fn)

// print1内部，next就是print2Fn
(print2Fn) => (action) => {
  const res = print2Fn(action);
  return res;
};

// 调用之后返回的就是这个
(action) => {
  const res = print2Fn(action);
  return res;
};

// 即
fnComposed = (action) => {
  const res = print2Fn(action);
  return res;
};
```

所以，这里 print1 的 next，就是下一个 middleWare 的内层函数，也就是接受 action 为参数的函数。
如果 print2 还有下一个，那么就是 print3 的内层函数，依次类推。最后一层的 next 实际上是传入的标准 dispatch 函数
标准 dispatch 函数会返回调用时传入的 action，也就是最后一层的 action。

```js
// compose(...chain) 的结果是一个 (next) => (action) => {...}

dispatch = compose(...chain)(store.dispatch); // 最后一个middleWare的next是标准dispatch
```

每个 middleWare 的返回值，都在上一层中表示为`next(action)`。每一层可以在前一层的返回值基础上返回，也可以单独返回

注意：合并的时候调用顺序是从后向前的，但是合并时候的函数是参数为 next 的函数，返回值是一个接受 action 的函数，并不是最终调用的结果。
在把所有的 middleWare 都合并之后，最后得到了一个接受 action，返回调用结果的函数，这个就是我们的最终 dispatch。
**当调用这个 dispatch 时，实际上是按照从前到后的顺序调用中间件**。比如就会按照 print1、print2、print3 的传入顺序调用。

## 基本使用

### 栗子

下面是官方文档上的一个例子：

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Redux basic example</title>
    <script src="https://unpkg.com/redux@latest/dist/redux.min.js"></script>
  </head>
  <body>
    <div>
      <p>
        Clicked: <span id="value">0</span> times
        <button id="increment">+</button>
        <button id="decrement">-</button>
      </p>
    </div>
    <script>
      const initialState = {
        value: 0,
      };

      function counterReducer(state = initialState, action) {
        switch (action.type) {
          case "counter/incremented":
            return { ...state, value: state.value + 1 };
          case "counter/decremented":
            return { ...state, value: state.value - 1 };
          default:
            return state;
        }
      }
      const store = Redux.createStore(counterReducer);

      const valueEl = document.getElementById("value");

      function render() {
        // store.getState可以在任何地方获取到最新的state
        const state = store.getState();
        valueEl.innerHTML = state.value.toString();
      }

      render();
      // render函数作为订阅者；当state发生改变时，render会被调用，内部的store.getState就会获得最新的state，从而更新ui
      store.subscribe(render);

      document
        .getElementById("increment")
        .addEventListener("click", function () {
          store.dispatch({ type: "counter/incremented" });
        });

      document
        .getElementById("decrement")
        .addEventListener("click", function () {
          store.dispatch({ type: "counter/decremented" });
        });
    </script>
  </body>
</html>
```

### 处理异步

由于 reducer 必须是纯函数，因此如果想异步地修改状态，就只能通过在 dispatch 周围操作了。
redux 提供了中间件的形式去处理异步。基本思路是，当时 dispatch 传入的是一个函数时，调用这个函数；这个函数内部是异步代码，并最终会调用一个新的 dispatch。随后 state 会在新的 dispatch 中进行更新

```ts
const asyncFunctionMiddleware = (storeAPI) => (next) => (action) => {
  if (typeof action === "function") {
    return action(storeAPI.dispatch, storeAPI.getState);
  }

  return next(action);
};
```

然后可以编写一个异步的 action 函数：

```ts
const fetchSomeData = (dispatch, getState) => {};

store.dispatch(fetchSomeData);
```

这样当执行 dispatch 时，实际上就是执行 fetchSomeData 这个函数。当异步请求完成后，调用同步的 dispatch。由于此时 action 不再是一个函数，因此会正常更新。

redux 存在这样一个中间件，叫做 redux-thunk。thunk 自动提供了一个异步中间件，开发者只需要编写对应的函数，在函数内执行请求并 dispatch，就可以实现异步效果。

标准的函数如下：

```js
const fetchSomeData = async (dispatch, getState) => {
  const res = await doSomeAsyncJob();
  dispatch({ type: "", payload: res });
};

store.dispatch(fetchSomeData);
```

如果需要一个传入值，比如做一个请求的时候需要请求参数，那么我们可以在外面包一层

```js
const sendData = (data) => {
  return const sendSomeData = async (dispatch, getState) => {
    const res = await doSomeAsyncJob(data)
    dispatch({type:'',payload:res})
  };
}

store.dispatch(sendData(someData));
```

示例如下：

```ts
import thunkMiddleware from "redux-thunk";
import rootReducer from "./reducer";

const composedEnhancer = applyMiddleware(thunkMiddleware);

const store = createStore(rootReducer, composedEnhancer);
export default store;

const fetchSomeData = async (dispatch, getState) => {
  const response = await client.get("/fakeApi/todos");
  log("before", getState());
  dispatch({ type: "todos/todosLoaded", payload: response.todos });
  log("after", getState());
};

store.dispatch(fetchSomeData);
```

其中，getState 方法可以获取实时的 state 值，用在 dispatch 前后就可以获取更新前后的值。

### thunk

redux-thunk 代码其实非常简单，就只是判断 action 的类型。如果是函数就直接调用 action，如果不是就通过 next 调用 action

```js
function createThunkMiddleware<
  State = any,
  BasicAction extends Action = AnyAction,
  ExtraThunkArg = undefined
>(extraArgument?: ExtraThunkArg) {
  const middleware: ThunkMiddleware<State, BasicAction, ExtraThunkArg> =
    ({ dispatch, getState }) =>
    next =>
    action => {
      if (typeof action === 'function') {
        return action(dispatch, getState, extraArgument)
      }

      return next(action)
    }
  return middleware
}

export const thunk = createThunkMiddleware()

```

### 请求状态处理

实际上通过 thunk 实现异步请求的过程不只是简单调用 dispatch 传入一个请求函数这么简单。还要考虑的其他方面有：

- 错误处理
- 加载状态处理
- 请求数据传入

对于前两种，其实我们需要在原本 state 的基础上增加 pending 和 error 处理的 state。当请求发出时触发 pending，返回时根据结果去触发 error 或 fulfilled 的 state。
比如最基础的版本是这样：

```js
const fetchSomeData = async (dispatch, getState) => {
  const res = await doSomeAsyncJob();
  dispatch({ type: "", payload: res });
};

store.dispatch(fetchSomeData);
```

我们加上对错误和 loading 状态的处理

```js
const fetchSomeData = async (dispatch, getState) => {
  dispatch({ type: "data/loading" });
  try {
    const res = await doSomeAsyncJob();
    dispatch({ type: "data/succeed", payload: res });
  } catch (err) {
    dispatch({ type: "data/failed", payload: err });
  }
};

store.dispatch(fetchSomeData);
```

在组件内部，需要对这几种状态处理。比如当 loading 状态时显示 loading 内容，其他情况处理结果和错误

```js
export const PostsList = () => {
  const dispatch = useDispatch()
  const posts = useSelector(state => state.data.dataList)
  // status也是个状态，包括idle/succeeded/failed/loading四种状态
  const postStatus = useSelector(state => state.data.status)
  const error = useSelector(state => state.data.error)

  useEffect(() => {
    // 空闲时在useEffect内请求
    if (postStatus === 'idle') {
      dispatch(fetchSomeData())
    }
  }, [postStatus, dispatch])

  let content

  if (postStatus === 'loading') {
    content = <Spinner text="Loading..." />
  } else if (postStatus === 'succeeded') {
    content = ... // 正常数据返回
  } else if (postStatus === 'failed') {
    content = <div>{error}</div>
  }

  return (
    <section className="posts-list">
      <h2>Posts</h2>
      {content}
    </section>
  )
}
```

redux 提供了一个 createAsyncThunk 可以方便创建上面所需的函数。配合 createSlice 的 extraReducers，就可以简化上述步骤。
在 slice 内部的 extraReducers 下，通过 builder 添加的 case 可以包含不同状态的 thunk 对应的 type。如下所示，即我们不再需要单独在 reducer 中自己处理关于这几种状态的的 case 了。

```js
export const fetchPosts = createAsyncThunk("posts/fetchPosts", async () => {
  const response = await client.get("/fakeApi/posts");
  return response.data;
});

const initialState = {
  posts: [],
  status: "idle",
  error: null,
};

const postsSlice = createSlice({
  name: "posts",
  initialState,
  reducers: {
    // 处理其他type
  },
  extraReducers(builder) {
    builder
      .addCase(fetchPosts.pending, (state, action) => {
        state.status = "loading";
      })
      .addCase(fetchPosts.fulfilled, (state, action) => {
        state.status = "succeeded";
        // Add any fetched posts to the array
        state.posts = state.posts.concat(action.payload);
      })
      .addCase(fetchPosts.rejected, (state, action) => {
        state.status = "failed";
        state.error = action.error.message;
      });
  },
});
```

createAsyncThunk 还可以接受参数，以实现发送数据的效果

```js
export const addNewPost = createAsyncThunk(
  "posts/addNewPost",
  async (initialPost) => {
    const response = await client.post("/fakeApi/posts", initialPost);
    return response.data;
  }
);
```

# React-Redux

React-Redux 实际上就是把 Redux 接入到 React 中。在历史版本中曾经还有 connect 这样的高阶函数来主动连接，不过现在则主要依赖 hooks 和 context 完成。

## 基本概念

### Provider

Provider 是一个封装了 Context.Provider 的组件，接受一个 store 作为 props：

```ts
function Provider<A extends Action = AnyAction, S = unknown>({
  store,
  context,
  children,
}: ProviderProps<A, S>) {
  // ContextValue是创建的context对象，包含store和subscription对象
  const contextValue = useMemo(() => {
    // subscription对象持有一个listeners链表，上面有订阅、触发等方法
    const subscription = createSubscription(store);
    return {
      store,
      subscription,
    };
  }, [store]);

  // 即持有的state
  const previousState = useMemo(() => store.getState(), [store]);

  useLayoutEffect(() => {
    const { subscription } = contextValue;
    subscription.onStateChange = subscription.notifyNestedSubs;
    subscription.trySubscribe();
    // 如果新旧状态不一致，就触发所有的listeners
    if (previousState !== store.getState()) {
      subscription.notifyNestedSubs();
    }
    return () => {
      subscription.tryUnsubscribe();
      subscription.onStateChange = undefined;
    };
  }, [contextValue, previousState]);

  const Context = context || ReactReduxContext;
  // 将contextValue对象放入context下发
  return <Context.Provider value={contextValue}>{children}</Context.Provider>;
}
```

contextValue 保存有创建的 store 和 subscription 对象。当 store 更新时，contextValue 就会被更新，从而触发 useLayoutEffect 内部的逻辑去触发 listeners。
subscription 上的 listeners 是在 useSelector 函数内部被加入的，在下面会说到。

可以看到，contextValue 的更新主要依赖 store 对象。

### selector 和 useSelector

#### 旧版 useSelector

参考：https://juejin.cn/post/6960838627945349151
旧版的 useSelector 其实逻辑比较简单，这里直接贴出源码：

```js
const refEquality = (a, b) => a === b

function useSelectorWithStoreAndSubscription(
  selector,
  equalityFn,
  store, // 通过context获取的store，context始终不会改变，改变的是store.getState
  contextSub
) {
  // 通过传入状态来强制更新
  const [, forceRender] = useReducer(s => s + 1, 0)

  // 订阅全局更新
  const subscription = useMemo(() => new Subscription(store, contextSub), [
    store,
    contextSub
  ])

  const latestSelector = useRef() // 上一个selector函数
  const latestSelectedState = useRef() // 上一个select的状态

  let selectedState

  try {
    // 如果本次的select状态和上一次不相等
    if (
      selector !== latestSelector.current
    ) {
      // 调用selector计算得到本次的select状态
      // 从这里可以看出，如果selector函数返回一个对象，那么selectedState永远都是最新的，永远不可能和上次状态相等
      selectedState = selector(store.getState())
    } else {
      selectedState = latestSelectedState.current
    }
  }

  useIsomorphicLayoutEffect(() => {
    function checkForUpdates() {
      try {
        const newSelectedState = latestSelector.current(store.getState())
        // 比较新旧select状态
        if (equalityFn(newSelectedState, latestSelectedState.current)) {
          return
        }

        latestSelectedState.current = newSelectedState
      }
      // 如果改变，执行更新
      forceRender({})
    }

    subscription.onStateChange = checkForUpdates
    subscription.trySubscribe()

    checkForUpdates()

    return () => subscription.tryUnsubscribe()
  }, [store, subscription])

  return selectedState
}

export function createSelectorHook(context = ReactReduxContext) {
  const useReduxContext =
    context === ReactReduxContext
      ? useDefaultReduxContext
      : () => useContext(context)
  return function useSelector(selector, equalityFn = refEquality) { // 这里就是useSelector函数
    const { store, subscription: contextSub } = useReduxContext()

    return useSelectorWithStoreAndSubscription(
      selector,
      equalityFn,
      store,
      contextSub
    )
  }
}
export const useSelector = /*#__PURE__*/ createSelectorHook()
```

核心内容其实就是下面几点：

1. 通过 selector 函数得到派生状态，也就是组件中真正需要的状态
2. 通过 useRef 记录上一次的派生状态
3. 每次订阅到 state 改变时，通过本次的 selector 计算得到本次的 select 状态，然后和上一次的状态进行比较。比较的函数是 equalityFn，默认就是索引比较。如果不相等，那么触发一次更新
4. 触发更新的方式是一个 setState

从这里可以很明显看出，当 selector 函数返回的是一个新建的对象时，那就永远不会和旧 state 相等，也就始终会导致组件更新。
同样的，如果 selector 取得的状态比较顶层，也会有这个问题。比如：

```js
const initState = {
  list:{
    listOffset:0,
    listData:[],
  }
}

const reducer = (state,{type,payload}) => {
  case 'list/updateOffset':
    return {...state,payload.listOffset}
}

// ...
const listData = useSelector(state => state.list)
```

上面的 useSelector 内部直接取的是整个 list 对象，那么当 listOffset 变化时，整个 list 都会被重新创建，这时比较结果肯定不同，就会引起不必要的渲染。

另一方面，如果在 useSelector 中使用 filter 等会返回新对象的方法时，也会有这个问题：

```js
const postsForUser = useSelector((state) => {
  const allPosts = selectAllPosts(state);
  return allPosts.filter((post) => post.user === userId);
});
```

同样的问题，当调用 selector 时返回的一定是一个新的状态。

这个问题的解决方式是使用 reselect 库，在 redux 中则是直接使用 createSelector 方法来创建一个“记忆化的”selector：

```js
export const selectPostsByUser = createSelector(
  [selectAllPosts, (state, userId) => userId],
  (posts, userId) => posts.filter((post) => post.user === userId)
);
const postsForUser = useSelector((state) => selectPostsByUser(state, userId));
```

经过包装后的 selector，只会在 posts 或 userId 发生变化时重新执行输出 selector
reselect 的原理可以参考：https://juejin.cn/post/7147294478074658823

基本原理可以总结为：reselect 使用闭包保存上一次的参数 lastArgs 与结果 lastResult ，只有当依赖中的某个 Redux state 发生了变化，导致前后参数比对不一致了，才会触发 selector 的再次计算。selector 重新计算，那么就和上面逻辑一样，根据比较判断是否需要更新。

#### 新版 useSelector

useSelector 简化如下：

```ts
function useSelector<TState, Selected extends unknown>(
  selector: (state: TState) => Selected,
  equalityFn: EqualityFn<NoInfer<Selected>> = refEquality
): Selected {
  // 这里就是通过context获取contextValue对象
  const { store, subscription } = useContext();

  const selectedState = useSyncExternalStoreWithSelector(
    subscription.addNestedSub,
    store.getState,
    getServerState || store.getState,
    selector,
    equalityFn
  );
  return selectedState;
}
```

可以看到核心的函数实际上是一个 useSyncExternalStoreWithSelector。这个函数和 useSyncExternalStore 一样都是 React18 提供的新 hook，前者增加了 selector 和 equalityFn 的接入，便于以“selector”的方式来订阅更新。

简单来说就是会在 state 发生变动时，通过 selector 得到 state 的派生状态，然后比较两次派生 state 是否相等（默认索引相等，比较函数就是 equalityFn），如果不相等就会更新。

#### useSyncExternalStore

useSyncExternalStore 接受一个 subscribe 函数和一个外部数据源，当数据源的值发生变化时，会触发 subscribe 绑定的 listeners，并计算最新的 state 并返回，同时还会强制调度更新一次。
具体来说，现在有一个简单 redux 的例子：

```jsx
function numberReducer(state = 1, action) {
  switch (action.type) {
    case "ADD":
      return state + 1;
    case "DEL":
      return state - 1;
    default:
      return state;
  }
}

const rootReducer = combineReducers({ number: numberReducer });
const store = createStore(rootReducer, { number: 1 });

function Index() {
  /* 订阅外部数据源 */
  const state = useSyncExternalStore(
    store.subscribe,
    () => store.getState().number
  );
  return (
    <div>
      {state}
      <button onClick={() => store.dispatch({ type: "ADD" })}>点击</button>
    </div>
  );
}
```

在 useSyncExternalStore 内部，第一次调用时会调用一个 subscribeToStore 函数，这个函数会调用 store.subscribe 并传入一个 handleStoreChange 函数。
handleStoreChange 的作用就是检查 state（即从第二个参数来的 state）是否发生变化。如果变化就触发更新。

而传入的 store.subscribe 函数会将 handleStoreChange 放入 store 的 listeners 上，在 dispatch 时调用。
因此后面调用 dispatch 时就会执行 handleStoreChange，得到新的 state、比较新旧 state、触发更新。

```ts
function mountSyncExternalStore(subscribe, getSnapshot) {
  const hook = mountWorkInProgressHook();
  // 第二个参数返回一个state，将作为后续判断的依据
  let nextSnapshot = getSnapshot();

  hook.memoizedState = nextSnapshot;
  const inst = {
    value: nextSnapshot,
    getSnapshot,
  };
  hook.queue = inst;

  mountEffect(subscribeToStore.bind(null, fiber, inst, subscribe), [subscribe]);

  return nextSnapshot;
}

// 调度一次更新
function forceStoreRerender(fiber) {
  scheduleUpdateOnFiber(fiber, SyncLane, NoTimestamp);
}

function subscribeToStore(fiber, inst, subscribe) {
  const handleStoreChange = () => {
    /* 检查 state 是否发生变化 */
    if (checkIfSnapshotChanged(inst)) {
      /* 触发更新 */
      forceStoreRerender(fiber);
    }
  };
  /* 发起订阅 */
  return subscribe(handleStoreChange);
}
```

这个 hooks 最重要的作用是，它的数据来源是一个外部的状态（store.state），但他返回一个能供 React 使用的 state，并且还依赖的是外部的 subscribe 函数。

### dispatch 和 useDispatch

React-Redux 的 dispatch 实际上仍然是 store.dispatch:

```ts
function useDispatch<
  AppDispatch extends Dispatch<A> = Dispatch<A>
>(): AppDispatch {
  const store = useStore();
  // @ts-ignore
  return store.dispatch;
}
```

这里的 store 就是在 Provider 中传入的那个 store。store 的创建则是在 toolkit 的 configureStore 中实现。

### store

store 的创建主要是通过 configureStore api，其内部本质上还是调用了 createStore

简化代码如下：

```ts
export function configureStore(options) {
  const {
    reducer = undefined, // reducer是一个对象，可以是combineReducer的结果，也可以是多个reducer
    middleware = curriedGetDefaultMiddleware(), // 添加默认middleware，包括thunk、检查错误的一些中间件
    devTools = true,
    preloadedState = undefined,
    enhancers = undefined, // 一个enhancers数组
  } = options || {};

  let rootReducer: Reducer<S, A>;

  // 调用combineReducers
  if (typeof reducer === "function") {
    rootReducer = reducer;
  } else if (isPlainObject(reducer)) {
    rootReducer = combineReducers(reducer) as unknown as Reducer<S, A>;
  }

  // 处理middleware，自动添加thunk中间件
  let finalMiddleware = middleware;
  if (typeof finalMiddleware === "function") {
    finalMiddleware = finalMiddleware(curriedGetDefaultMiddleware);
  }
  const middlewareEnhancer: StoreEnhancer = applyMiddleware(...finalMiddleware);

  let finalCompose = compose;

  // 处理enhancer，将多个enhancer合并
  let storeEnhancers: Enhancers = [middlewareEnhancer];
  if (Array.isArray(enhancers)) {
    storeEnhancers = [middlewareEnhancer, ...enhancers];
  } else if (typeof enhancers === "function") {
    storeEnhancers = enhancers(storeEnhancers);
  }
  const composedEnhancer = finalCompose(
    ...storeEnhancers
  ) as StoreEnhancer<any>;

  return createStore(rootReducer, preloadedState, composedEnhancer);
}
```

## 其他 api

### connect

connect 实际上是类组件时代的产物，它主要作用是将 redux 和 react 组件结合起来。
函数组件可以通过 hooks 直接获取 dispatch 函数和保持的 state，但是类组件只能通过向 props 注入的方式添加。connect 函数返回一个函数，其实是一个高阶组件，它接受一个组件，并返回一个强化后的组件。强化后的组件相对于原组件，添加了 state 和 dispatch。

connect 的两个参数，分别是 mapStateToProps 和 mapDispatchToProps。这两个函数的结构相似，都返回一个对象，将融合在被包裹组件的 props 中。

```ts
const mapStateToProps = (state, ownProps) => ({
  // ... computed data from state and optionally ownProps
});

const mapDispatchToProps = {
  // ... normally is an object full of action creators
};

// `connect` returns a new function that accepts the component to wrap:
const connectToStore = connect(mapStateToProps, mapDispatchToProps);
// and that function returns the connected, wrapper component:
const ConnectedComponent = connectToStore(Component);

// We normally do both in one step, like this:
connect(mapStateToProps, mapDispatchToProps)(Component);
```

### mapStateToProps

mapStateToProps 类似 useSelector，它接收一个来自 store 的 state，然后内部对 state 进行解构，最后返回一个对象合并到 props 中

```ts
const mapStateToProps = state => {
  const todos = state.map(...)
  return { todos };
};

// 组件内
class TodoComponent extends Component {
  render(){
    return (
      {
        this.props.todos.map((todo) => (...))
      }
    )

  }
}
```

### mapDispatchToProps

和 mapStateToProps 类似，不过注入对象是 dispatch 函数。
mapDispatchToProps 接受一个 dispatch 参数，即真实的 dispatch。

```tsx
const mapDispatchToProps = dispatch => {
  return {
    toggleTodo: todoId => dispatch(toggleTodo(todoId))
  }
}

// 组件内
render(){
  return (
    <div>
      <button onClick={() => this.props.toggleTodo()}></button>
    </div>
  )
}
```

# Redux-Toolkit

Toolkit 是一个工具包，并非必须的。Toolkit 可以简化 React-Redux 的使用，当然完全不用 toolkit，手动创建 store 也是可以的。

## 常用 api

### configureStore

这个在上面说过，主要作用有：

- 处理 reducers，调用 combineReducers 组合
- 创建 Redux store
- 自动添加了 thunk 中间件
- 自动添加了更多中间件来检查常见错误，例如意外改变状态
- 自动设置 Redux DevTools 扩展连接

通过 configureStore 传入的 reducer 内部如果直接修改 state，则会导致崩溃（中间件作用）。

### createSlice

Redux 定义了一个“slice”的概念，可以理解为一个 state 的专属的对象：

```ts
const postsSlice = createSlice({
  name: "posts",
  initialState: [],
  reducers: {
    createPost(state, action) {},
    updatePost(state, action) {},
    deletePost(state, action) {},
  },
});
```

每个 slice 只管理一个状态，可以配置他的 name、初始值，以及修改这个状态用到的函数。

其中，reducers 内部的这几个函数可以理解为原本的 reducer 的 switchcase 逻辑拆分。
在 createSlice 内部，这些 reducers 函数会组合为一个 reducer。name 属性的字符串会成为返回的 action 传的 key 的一部分（比如调用 createPost，绑定的 type 就是'posts/createPost'）
并且，在这些函数内部，可以直接修改 state，类似使用了 immer 的 reducer。

```ts
const todosSlice = createSlice({
  name: "todos",
  initialState,
  reducers: {
    todoAdded(state, action) {
      // ✅ This "mutating" code is okay inside of createSlice!
      state.entities.push(action.payload);
    },
    todoToggled(state, action) {
      const todo = state.entities.find((todo) => todo.id === action.payload);
      todo.completed = !todo.completed;
    },
    todosLoading(state, action) {
      return {
        ...state,
        status: "loading",
      };
    },
  },
});
```

createSlice 的返回值是一个对象，导出时需要分开导出：

```ts
return {
    name,
    reducer(state, action) {},
    actions: actionCreators
    caseReducers: sliceCaseReducersByName,
    getInitialState() {},
  }

// 大概是这样：
{
    name: 'posts',
    actions : {
        createPost,
        updatePost,
        deletePost,
    },
    reducer
}
```

其中，传入的 reducers 对象内的函数会被 createAction 转换成自动绑定了 type 的 action，因此调用时不需要传入 type，直接传 payload 即可。
当存在这些 action 时，就不必再调用 dispatch 函数了

```ts
createPost({ id: 123, title: "Hello World" });
// 这个函数会返回实际的action对象
// {type : "posts/createPost", payload : {id : 123, title : "Hello World"}}
```

而 reducer 对象则是 redux 要用的 reducer，需要导出

因此整体导出就是**分别导出各个 action，然后默认导出 reducer**即可。

```ts
const { actions, reducer } = postsSlice;
export const { createPost, updatePost, deletePost } = actions;
export default reducer;
```

导出的 reducer 将会作为 configureStore 的 reducer

```ts
const store = configureStore({
  reducer: {
    A: ReducerA,
    B: ReducerB,
  },
});
```

最后把 store 传给 Provider 组件。
完整的过程如下：

```ts
// 创建slice
const counterSliceA = createSlice({
  name: "counterA",
  initialState: 0,
  reducers: {
    increment(state, { payload, type }) {
      return state + payload.count;
    },
    addFive(state, action) {
      return state + 5;
    },
  },
});
// 导出reducer和action
const { actions, reducer } = counterSliceA;
export const { increment, addFive } = actions;
export default reducer;

// 创建store
const store = configureStore({
  reducer: {
    counterA: counterAReducer,
  },
});

// 使用store
document.getElementById("root").render(
  <Provider store={store}>
    <App />
  </Provider>,
  rootElement
);

// 组件内部调用action
increment({ count: 2 });

// 组件内部使用state
const count = useSelector((state) => state.counterA);
```

## 配合 TS

Redux 需要单独编写一部分类型代码
首先是在 configureStore 附近的，定义 store 的 state 和 dispatch 类型

```ts
const store = configureStore({
  reducer: {
    posts: postsReducer,
    comments: commentsReducer,
    users: usersReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

然后定义 useSelector 和 useDispatch。这里实际上是重新创建了这两个 hook。在组件内调用的是这两个重新创建的 hook 而不是原生的 useSelector 和 useDispatch

```ts
import { TypedUseSelectorHook, useDispatch, useSelector } from "react-redux";
import type { RootState, AppDispatch } from "./store"; // 上面的两个类型

export const useAppDispatch: () => AppDispatch = useDispatch;
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

如果没有用 slice，到这就结束了。反之还需要设置 slice
在 slice 对象中主要要设置 initiateState 的类型，将作为后面的 reducers 函数中 state 的类型。而 action 则可以由 PayloadAction 泛型指定。

```ts
import { createSlice, PayloadAction } from "@reduxjs/toolkit";
import type { RootState } from "../../app/store";

interface CounterState {
  value: number;
}

const initialState: CounterState = {
  value: 0,
};

export const counterSlice = createSlice({
  name: "counter",
  initialState,
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action: PayloadAction<number>) => {
      state.value += action.payload;
    },
  },
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;
export default counterSlice.reducer;
```

# Redux 哲学

Redux 官方文档除了介绍 Redux 的使用之外，还讲述了不少 Redux 的使用哲学，这些思维模式不仅仅可以用于 redux，也适用于平时的 react 项目。

## 三大原则

redux 的三大原则是

- 单一数据源：整个应用的 全局 state 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 store 中。
  - 维持数据的单一来源，便于状态的调试和监控，只需要在顶层检查即可
  - 在 ssr 或其他应用中，可以很方便实现数据持久化、序列化等，比如从服务端初始化后序列化发送给客户端并注入
- state 只读，是 immutable 的，改变 state 需要通过派发 action。主要原因在于
  - redux 中包含很多通过浅比较来判断是否需要更新的地方，比如 selector
  - 维持数据的可追踪性，任何时候只需要回溯不同的 action，就可以得到旧的或新的状态，也可以被方便地打印、储存、调试、测试等等
  - state 应该是可序列化的，即不能包含类、函数这样的类型
- 纯函数：用于改变 state 的 reducer 是一个纯函数
  - 直接更改 state，state 的值会改变，但 state 本身没有改变。在需要浅比较更新的地方，比较结果就会相同，导致无法触发更新。

## xxxManager

官方文档在介绍动态替换 Reducer 是，介绍了一个对象，即 reducerMangaer。manager 本质上是一个维护一些方法的对象，这些方法会对某个数据结构进行增删改等操作。该对象由一个 createXxxManager 创建，该对象内的方法将作为闭包，使用 createXxxManager 函数内的变量作为自己维护的数据。

举个例子，reducerManager 其实就是维护一个 reducers 对象，reducers 实际上是 createReducerManager 函数的一个内部变量；同时 createReducerManager 函数返回一个对象（即 manager 对象），对 reducers 的增删查操作则作为 manager 对象的属性。

```ts
export function createReducerManager(initialReducers) {
  const reducers = { ...initialReducers };
  let combinedReducer = combineReducers(reducers);
  let keysToRemove = [];

  return {
    getReducerMap: () => reducers,
    // 修改reducer时，顺带把state对象上该reducer对应的state也修改，并最后返回一个combine结果
    reduce: (state, action) => {
      if (keysToRemove.length > 0) {
        state = { ...state };
        for (let key of keysToRemove) {
          delete state[key];
        }
        keysToRemove = [];
      }
      return combinedReducer(state, action);
    },
    // 增加和移除reducer
    add: (key, reducer) => {
      if (!key || reducers[key]) {
        return;
      }
      reducers[key] = reducer;
      combinedReducer = combineReducers(reducers);
    },
    remove: (key) => {
      if (!key || !reducers[key]) {
        return;
      }
      delete reducers[key];
      keysToRemove.push(key);
      combinedReducer = combineReducers(reducers);
    },
  };
}

const staticReducers = {
  users: usersReducer,
  posts: postsReducer,
};

export function configureStore(initialState) {
  const reducerManager = createReducerManager(staticReducers);
  const store = createStore(reducerManager.reduce, initialState);
  store.reducerManager = reducerManager;
}
```

这是一种思路，即维护一种数据结构时，将其封装在函数内部，通过返回 manager 对象的方法来修改该数据结构。

```ts
function createXxxManager(initalValue) {
  const data = initalValue;
  return {
    add() {},
    remove() {},
    getData: () => data,
  };
}
```

其实 createStore 函数本质上也是这个思路，其函数内部有 getState、dispatch 等方法，最后返回一个对象包含了这些方法。

```ts
export default function createStore(
  reducer: Reducer<S, A>,
  preloadedState?: PreloadedState<S> | StoreEnhancer<Ext, StateExt>,
  enhancer?: StoreEnhancer<Ext, StateExt>
) {
  let currentReducer = reducer;
  let currentState = preloadedState as S;
  let currentListeners: (() => void)[] | null = [];
  let nextListeners = currentListeners;
  let isDispatching = false;

  function getState(): S {}

  function subscribe(listener: () => void) {}

  function dispatch(action: A) {}

  function replaceReducer(nextReducer): Store {}

  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
  };
}
```

## Reducer 的要求

官方通过 12 个章节来介绍怎么构建一个合适的 reducer。即使不用 redux，有时为了整理 state 也会创建 reducer，官方提供的思路就可以很好的优化 reducer 结构。

### reducer 的基本特点

1. `(previousState, action) => newState`，即接受旧状态和 action，返回新状态。至于内部怎么处理没有关系，但是外部一定是这样的
2. 纯函数，即 reducer 内部不能有任何副作用，比如

- 定时器、网络请求等异步任务，
- 修改外部变量（state 应该复制而不是直接修改）
- 调用 date.now 等非纯函数，因为每次调用显然值都不相同

### 拆分 reducer

按照通常 reducer 的写法，可能会导致 reducer 内部很臃肿，即，一个 switch case 下有很长的一段逻辑，多个 case 下的逻辑可能是重复的，也可能毫不相关。这时就可以把这些 case 下的逻辑抽出来单独形成一个函数。具体来说是两步：

1. 把 case 下相同的逻辑抽出来成一个函数。比如复制原对象修改新属性的函数
2. 把 case 下不同的逻辑每个单独创建函数，reducer 内部只是调用这些函数。

比如现在有个 reducer：

```ts
const initialState = {
  visibilityFilter: "SHOW_ALL",
  todos: [],
};

function appReducer(state = initialState, action) {
  switch (action.type) {
    case "SET_VISIBILITY_FILTER": {
      return Object.assign({}, state, {
        visibilityFilter: action.filter,
      });
    }
    case "ADD_TODO": {
      return Object.assign({}, state, {
        todos: state.todos.concat({
          id: action.id,
          text: action.text,
          completed: false,
        }),
      });
    }
    case "TOGGLE_TODO": {
      return Object.assign({}, state, {
        todos: state.todos.map((todo) => {
          if (todo.id !== action.id) {
            return todo;
          }

          return Object.assign({}, todo, {
            completed: !todo.completed,
          });
        }),
      });
    }
    case "EDIT_TODO": {
      return Object.assign({}, state, {
        todos: state.todos.map((todo) => {
          if (todo.id !== action.id) {
            return todo;
          }

          return Object.assign({}, todo, {
            text: action.text,
          });
        }),
      });
    }
    default:
      return state;
  }
}
```

Object.assign 方法频繁出现，因此我们可以抽象成一个函数。

```ts
function updateObject(oldObject, newValues) {
  return Object.assign({}, oldObject, newValues);
}
```

对 todos 数组的操作也可以抽象成一个函数，因为上面的基本逻辑都是在 todos 数找到对应的 id，然后修改这一项，复制其他项。

```ts
function updateItemInArray(array, itemId, updateItemCallback) {
  const updatedItems = array.map((item) => {
    if (item.id !== itemId) {
      return item;
    }
    const updatedItem = updateItemCallback(item);
    return updatedItem;
  });
  return updatedItems;
}
```

最后把每个 case 下面的内容分成不同的函数

```ts
// Omitted
function updateObject(oldObject, newValues) {}
function updateItemInArray(array, itemId, updateItemCallback) {}

function setVisibilityFilter(state, action) {
  return updateObject(state, { visibilityFilter: action.filter });
}

function addTodo(state, action) {
  const newTodos = state.todos.concat({
    id: action.id,
    text: action.text,
    completed: false,
  });
  return updateObject(state, { todos: newTodos });
}

function toggleTodo(state, action) {
  const newTodos = updateItemInArray(state.todos, action.id, (todo) => {
    return updateObject(todo, { completed: !todo.completed });
  });
  return updateObject(state, { todos: newTodos });
}

function editTodo(state, action) {
  const newTodos = updateItemInArray(state.todos, action.id, (todo) => {
    return updateObject(todo, { text: action.text });
  });
  return updateObject(state, { todos: newTodos });
}

function appReducer(state = initialState, action) {
  switch (action.type) {
    case "SET_VISIBILITY_FILTER":
      return setVisibilityFilter(state, action);
    case "ADD_TODO":
      return addTodo(state, action);
    case "TOGGLE_TODO":
      return toggleTodo(state, action);
    case "EDIT_TODO":
      return editTodo(state, action);
    default:
      return state;
  }
}
```

如果再进一步优化，可以把 state 中的 visibilityFilter 和其他对 todos 函数的操作分开，使一个 reducer 只维护一个逻辑上相同的 state。
这个思维不仅可以用于 reducer，对于项目中其他庞大的函数，也可以按照这个思维模式去优化（复用+拆分）

### 减少样板

所谓“样板”，其实是指一些相似结构的对象、函数重复出现。比如 action 对象的结构都是差不多的，那么这些 action 就是样板。

```ts
{ type: 'ADD_TODO', text: 'Use Redux' }
{ type: 'REMOVE_TODO', id: 42 }
{ type: 'LOAD_ARTICLE', response: { ... } }
```

因为他们都有 type 属性和一个其他属性，因此可以用一个 createAction 函数来创建他们
但是注意这里的方式，如果你使用这种形式，那么实际上并没有减少样板，只是把传对象改成了传参数而已

```ts
function createAction(type, payload) {
  return {
    type,
    payload,
  };
}
```

我们希望的肯定是减少样板，而不是少了对象样板，又出来一个函数样板。
官方提供的思路是，把每个 type 对应的 action 的创建都变成一个函数，这样调用这些函数就不需要传 type，只需要传后面的值就可以：

```ts
export function addTodo(text) {
  return {
    type: "ADD_TODO",
    text,
  };
}

export function editTodo(id, text) {
  return {
    type: "EDIT_TODO",
    id,
    text,
  };
}

export function removeTodo(id) {
  return {
    type: "REMOVE_TODO",
    id,
  };
}
```

因为这些函数结构相似，因此需要再抽象一层，即再创建一个创建这些函数的函数。这个函数就应该是返回一个函数的函数，返回的函数就是上面的这几个

```ts
function makeActionCreator(type, ...argNames) {
  return function (...args) {
    const action = { type };
    argNames.forEach((arg, index) => {
      action[argNames[index]] = args[index];
    });
    return action;
  };
}

export const addTodo = makeActionCreator("ADD_TODO", "text");
export const editTodo = makeActionCreator("EDIT_TODO", "id", "text");
export const removeTodo = makeActionCreator("REMOVE_TODO", "id");
```

因此基本思路是：

1. 把重复、相似的数据结构（对象、数组、字符串等）的创建改成函数的创建。注意这个过程最好是减少参数，但不需要减少数量，比如这里是减少了 type 参数的传入，但是实际上每种 type 还是对应一个函数
2. 再创建一个创建第一步函数的函数，即一个高阶函数，根据参数返回不同的函数，这些返回的函数就是第一步需要的这些函数。

### 格式化数据

通常后端传来的数据是这样的：

```ts
const blogPosts = [
  {
    id: "post1",
    author: { username: "user1", name: "User 1" },
    body: "......",
    comments: [
      {
        id: "comment1",
        author: { username: "user2", name: "User 2" },
        comment: ".....",
      },
      {
        id: "comment2",
        author: { username: "user3", name: "User 3" },
        comment: ".....",
      },
    ],
  },
  {
    id: "post2",
    author: { username: "user2", name: "User 2" },
    body: "......",
    comments: [
      {
        id: "comment3",
        author: { username: "user3", name: "User 3" },
        comment: ".....",
      },
      {
        id: "comment4",
        author: { username: "user1", name: "User 1" },
        comment: ".....",
      },
      {
        id: "comment5",
        author: { username: "user3", name: "User 3" },
        comment: ".....",
      },
    ],
  },
];
```

这种形式被称为是“嵌套的”。虽然里边的数据很复杂，但是本质上其实就是三个对象的数据（post、comment 和 author），而这个数据中 post 内嵌套了 author 和 comments，每个 comments 又嵌套了 author。
这种数据结构会有什么问题呢？比如说我想维护一个 comments 列表的增删改和渲染，那么我就需要很长的逻辑去取出来这些 comments

```ts
const comments = posts.reduce((comments, post) =>
  comments.concat(post.coomments)
);
```

如果是在 reducer 或者 useState 维护的状态里，那就更加麻烦了，比如想修改一个 comments，还得先找到对应的 post，并且修改 comments 还需要复制一大堆完全不相关的属性

```ts
function postReducer(state, { type, payload }) {
  const { commentId, commentValue } = payload;
  switch (type) {
    case "EDIT_COMMENT": {
      const post = state.find((post) =>
        post.comments.include((comment) => (comment.id = commentId))
      );
      return {
        ...post,
        comments: post.comments.map((comment) => {
          if (comment.id !== commentId) return comment;
          return {
            ...comment,
            comment: commentValue,
          };
        }),
      };
    }
  }
}
```

因此我们维护的数据应该是扁平的。
即，这个数据内有三个核心对象，即 comment、post 和 author，他们三个的共同特点是都有对应的 id；那么就可以把这三个对象分别列出来，以自己的 id 形成序列：

```ts
{
    posts : {
        byId : {
            "post1" : {
                id : "post1",
                author : "user1",
                body : "......",
                comments : ["comment1", "comment2"]
            },
            "post2" : {
                id : "post2",
                author : "user2",
                body : "......",
                comments : ["comment3", "comment4", "comment5"]
            }
        },
        allIds : ["post1", "post2"]
    },
    comments : {
        byId : {
            "comment1" : {
                id : "comment1",
                author : "user2",
                comment : ".....",
            },
            "comment2" : {
                id : "comment2",
                author : "user3",
                comment : ".....",
            },
            "comment3" : {
                id : "comment3",
                author : "user3",
                comment : ".....",
            },
            "comment4" : {
                id : "comment4",
                author : "user1",
                comment : ".....",
            },
            "comment5" : {
                id : "comment5",
                author : "user3",
                comment : ".....",
            },
        },
        allIds : ["comment1", "comment2", "comment3", "comment4", "comment5"]
    },
    users : {
        byId : {
            "user1" : {
                username : "user1",
                name : "User 1",
            },
            "user2" : {
                username : "user2",
                name : "User 2",
            },
            "user3" : {
                username : "user3",
                name : "User 3",
            }
        },
        allIds : ["user1", "user2", "user3"]
    }
}
```

这样的数据就可以直接存储在 state 中，作为 reducer 使用或直接给 useState 使用。
这种数据可以很方便的修改和查找，只需要他们各自的 id 即可。

因此这样格式化的核心就是找到数据中的关键数据，然后将他们单独以 id 形成列表，将数据扁平化。

这种方法实际上是**数据库**思维，即，这里的 users、comments、posts 都是一个个**表**，而每个 id 则是**主键**，其他属性则作为其他字段存在。

用于格式化的库有一个 normalizr，具体的会单独写一篇笔记。

## Undo-Redo

参考https://redux.js.org/usage/implementing-undo-history

这部分内容其实和 redux 没啥关系，但是文档提出了一种实现 undo-redo 的方式，可以学习一下。

## redux 的性能

关于 redux 的性能问题其实很大程度上是 react 本身的 state 性能问题，即当 state 改变后，如何最小程度上降低渲染消耗，同时还能保持组件正常渲染。

在 react-redux 中，可能造成重渲染的点主要有两个

1. useContext 和 Provider。当 context 改变时，所有使用该 context 的组件都会更新；但在 react-redux 中，context 初始化为{store, subscription}之后就不会再变动，后续状态更新都是通过 store.getState 获取，以发布订阅模式来通知更新。
2. useSelector。前面讲过 useSelector 的原理和引起更新的情况，因此针对 useSelector 导致的重渲染，解决方案就是考虑 selector 函数的合理性，以及使用 reselect 来保存 selector 函数执行结果。

剩下的重复渲染问题本质上都是 react 的问题。比如经典的 10k 列表问题（10k 个列表元素，点击一个改变状态），其实放在 useState 中也是一样的，解决方法都是要么给子组件加上 memo，要么不要触发父组件的重渲染。

还有一些其他的性能问题考虑，和 react 无关的一些，比如

- reducer 的调用消耗：因为每个 dispatch 都会执行所有的 reducer 内部逻辑。但是 reducer 内部其实真正执行的代码不多
- 拷贝状态的内存消耗：react 推崇的是状态的浅拷贝而非深拷贝。如果只需要修改其中几个属性，其他属性直接复用就可以了。当然更好的优化方式是直接使用 immutable 数据或 immer（redux 的状态可以是 immutable 类型的）。当然如果 state 嵌套过深，就需要考虑扁平化 state
- 批量更新：redux 的多个 dispatch 不会被 react 自动批处理，因为不在同一更新内。因此 redux 提供了一个 batch 函数用于合并多个同步 dispatch 的更新。不过 react18 内不再需要了

# Recoil

Recoil 的基本使用不再赘述。

相关原理文档：
https://cloud.tencent.com/developer/article/1961153

## 基本特点

### 原子化

即 recoil 的基本特点，Recoil 推崇 state 分散管理，我们可以单独定义应用中各个独立的子状态。一个状态可以放在一个 atom 里，每个状态独立，而不用像在 redux 中将其整合在一个 reducer 中。
原子化的 state 主要特点有：

1. 清晰易辨。由于状态都是以最小单位 atom 保存的，因此不同的 atom 之间相互独立
2. 状态之间不会互相影响，即下面说的 items 问题，如果不希望一个 item 的改变会导致其他 item 都被更新，那么将每个 item 单独管理就是很好的方法，再通过 atomFamily 等方式将其整合起来。这样每个状态都是独立的，修改状态并不会影响其他 item 的状态
3. （缺点）不容易获得全局状态。不像 reducer 本身就是在维护全局状态，atom 的缺点就是难以一眼看清全局有哪些状态，不过可以用 snapshot 做为弥补
4. 便于代码分割。类似 redux 的状态管理库有一个问题，store 被放置在顶层，如果希望某个组件被分离出去，那么对应的 state 和 reducer 也要一并被拆分和分割。如果是原生的 react 状态提升，那么就会出现矛盾情况，提升不够高可能会无法共享，提升太高则导致难以分离。

### 数据流

![](https://pic.imgdb.cn/item/63d9339ae90d1c00983cfbc5.jpg)

这张图很好展示了 recoil 的结构，可以看到基本单位是 atom，从 atom 派生的 selector，这两个都是提供状态的对象；他们将状态下发到 React 的组件树中。

本质上这种关系是一种双向图，从共享状态到组件，组件到共享状态这样一个闭环。组件订阅状态，则就形成了从状态到组件的一个数据流，组件更新状态，则形成了从组件到状态的一个数据流。
为什么叫图呢，就是因为 Atom 和 Selector 可以看作是图上的节点，Atom 和 Selector 通过有向图的连接方式连接，形成一个图；这个数据流图和 React 组件树并不相关，哪个节点用到了数据，哪个地方才会和图关联起来
![](https://pic.imgdb.cn/item/63da5aa4ac6ef860166cd018.jpg)

这个图路径上的函数可以是异步的（即 selector 函数可以是异步的）。比如 selector 的 get 函数中如果返回一个 Promise，那它就被视为是一个异步的数据，将会异步加载这个状态。

### SnapShot 全局状态管理

Snapshot 对象是 Recoil atoms 状态的一个不可改变的快照。它的目的是规范用于观察、检查和管理全局 Recoil 状态的 API。对于开发工具、全局状态同步、历史导航等大部分需求，它都是很有用的。
实际上是对 recoil 松散状态的一个补充

### 对 ConcurrentMode 的支持

ConcurrentMode 最大的特点是渲染中断。render 阶段可能会被多次打断、多次重复，这种情况对于状态管理来说，很可能的问题是状态的不一致。

![](https://pic.imgdb.cn/item/63d93c30e90d1c00986009da.jpg)

如上，在 react 中断渲染的这段时间，可能状态已经发生变化，这时回来重新渲染就会出现状态不一致的问题。这不仅仅是 recoil 的问题，所有的外部状态管理库都会有这个问题。
在 Recoil 的最新版本，已经提供了一些实验性的 api 来支持该模式。Recoil 在渲染时如果检测到状态已经发生了改变，则会重新渲染整个节点树来避免状态的不一致。目前该机制是高效的，之后会继续优化性能。

## 核心概念

### atom

atom 是一个原子状态。在 Recoil 中，核心理念是状态的原子化，即将状态细分，划分到足够细的粒度，成为一个“atom”。
比如有一个画布，这个画布上可能有很多的图案，每一个图案（items）都有它自己的状态。按照传统 redux 的管理方式，就用一个 items 数组存储各个 item

```ts
import { createStore } from "redux";
const initialState = { items: [] };
function itemReducer(state = initialState, action) {
  switch (action.type) {
    case "ADD":
      return { items: [...state.items, action.preload] };
    default:
      return state;
  }
}
export const store = createStore(itemReducer);
```

这种方式有一个致命的问题，就是一个 item 状态的改变，会导致整个 items 数组被更新。因为 redux 的理念不是修改 state，而是替换 state。
如果 item 的更新需要通过拖动这种的高频率事件触发，那么就会造成极大的更新消耗。我们希望的是每个 item 的更新独立，更新一个 item 不会导致其他 item 更新。

atom 就是这样一种方式。为每个 item 创建一个 atom，然后用一个整体的数据结构（atomFamily）来管理。
比如单个 item 的状态

```tsx
const item = atom({
  key: "item",
  defaultValue: {},
});

function Item() {
  const [item, setItem] = useRecoilState(itemState);
  // other
  return <div>{item.title}</div>;
}
```

我们创建一个这样的函数，为每一个 item 根据 id 创建一个 atom

```tsx
const itemsList = {};
export const getItemState = (id) => {
  if (!itemsList[id]) {
    itemsList[id] = atom({
      key: `item-${id}`,
      default: {},
    });
  }
  return itemsList[id];
};

function Item({ id }) {
  const [item, setItem] = useRecoilState(getItemState(id));
  // other
  return <div>{item.title}</div>;
}
```

通过这种方式，给 item 组件传入不同的 id，就可以得到独立的 item 对象，每个维护自己独立的状态。

Recoil 提供了这种方式的 api atomFamily。经过 atomFamily 创建的 family 对象可以看作是和普通的 atom 相同，但它会根据不同的参数创建不同的单个 atom，保证不同的 id 对应到不同的 atom，从而形成 item 的独立，一个 item 状态的更改不会影响其他 item

```tsx
export const itemFamily = atomFamily({
  key: 'itemFamily',
  default: {
    x: 0,
    y: 0
  }
})

function Item({ id }) {
  // 这里的item管理的是这个item独立的状态，setItem也是设置这个状态
  const [item, setItem] = useRecoilState(itemFamily(id));
  return (
    <>
      <div>
        position:
        x: {item.x}
        y: {item.y}
      </div>
      <button onClick={() => setItem(pos => ({...pos,x: pos.x + 1}))}>向下</button>
      <button onClick={() => setItem(pos => ({...pos,x: pos.y + 1}))}>向右</button>
    </>
  );
}

function ItemsList(...){
  //...
  return (
    {
      items.map(item => (<Item id={item.id} />))
    }
  )
}
```

这就是 Recoil 状态管理的核心理念，抽离独立的状态，独立管理，避免不必要的渲染，同时便于扩展和管理。而 Redux 是状态的集中管理，基本上整个应用只有一个全局的状态。

### selector

selector 是一种派生状态的方式，即根据一个原生状态派生出一个新的状态。
比如说（官方文档例子），以 todoList 项目为例，todos 数组是一个原始状态（atom），而 todos 列表的信息（比如 todos 数量、已完成和未完成数量），或者通过某种方式过滤的 todos，都属于基于 todos 的派生状态。
因此 selector 基于 atom 或者其他的 selector，它的参数需要传递一个 atom 或 selector。
这种派生方式可以避免冗余 state，即不需要为可以派生的状态单独创建一个 atom 或维护一个状态（比如 completeTodos 派生于 todos，如果不采用 selector，就需要单独维护一个 completeTodos 状态）

selector 结构如下：

```ts
const someState = selector({
  key: "someState",
  get: ({ get }) => {
    const otherState = get(otherAtom);
    const anotherState = get(otherSelector);
    return {
      otherState,
      anotherState,
    };
  },
  set: ({ get, set, reset }, newValue) => {},
});
```

get 方法通过其他的 atom 或 selector 的 state，返回一个新的 state 作为派生值。通常就是从原生状态 get 获取 state，然后返回一个派生的 state。比如基于 todos atom 派生的 filterTodos：

```ts
const completeSelector = selector({
  key: "completeTodos",
  get: ({ get }) => {
    const todos = get(todoAtom);
    return todos.filter((todo) => todo.isComplete);
  },
});

// 使用和atom一样

const completeTodos = useRecoilValue(completeSelector);
```

一旦通过 get 获取了某个 atom 的状态，该 selector 就会和 atom“连接”，源 atom 的状态改变会导致重新执行 get 方法，自动重新计算新值，并触发订阅组件的更新。

需要注意的是，如果 selector 只设置了 get 而没有设置 set，那么就只能取值而不是调用 setState 修改值。如果希望调用 setState 修改状态，那么还需要设置 set 方法
set 方法参数是一个对象，包含 get、set 和 reset 三个方法，不需要返回值，关键是调用 set 方法修改数据来源的 atom。

- get：和 get 里的那个 get 相同，可以从 atom 中获取状态。
- set：格式为`(atom, value) => void`，可以 value 设置为 atom 的 state 值。

比如：

```ts
const completeSelector = selector({
  key: 'completeTodos',
  get: ({get}) => {
    const todos = get(todoAtom)
    return todos.filter(todo => todo.isComplete)
  },
  set: ({get, set},newValue) => {
    // 调用set修改todoAtom，才能使得源状态更改，从而使派生状态更改
    set(todoAtom, newValue)
  }
})

// 组件内
const setCompleteTodosState = useSetRecoilState(completeSelector)
setCompleteTodosState([...])
```

### 异步数据流

Recoil 可以实现在正常的数据流图中直接添加异步函数来实现异步数据查询。
而 redux 的异步方案，其实是一种同步的流程中加入了异步函数，即通过传递 dispatch 回调，让异步任务完成后调用 dispatch。这实际上和 react 原生的异步方式没有区别
但是 recoil 的异步则是直接在数据流中的。通过 selector 设置一个异步函数，消费这个 selector 的组件，类似于 react 提出的 suspense 用法，会让渲染和请求同时启动，当异步任务完成后才渲染完成。
参考 React 对 suspense 组件的新用法：https://beta.reactjs.org/reference/react/Suspense ，可以看到第一和第二个例子中，Suspense 包裹的组件内部直接渲染的数据实际上是一个在 render 阶段执行的异步任务，这和普通 React 组件要求在 useEffect 内完成不同。Recoil 的异步实现也类似于这种方式。

需要注意的是，由于 recoil 的 selector 推崇的是一种“幂等”，也就是说多次调用 selector 的返回值应该是相等的，因为 recoil 可能会对 selector 进行追踪，并且 selector 的计算可能被缓存、重启或多次执行。所以 selector 内的异步要求必须也是幂等的，即对于请求来说，多次请求的返回值应该是一样的（错误不算）；而对于常规的副作用，比如定时器，则不能在 selector 中使用。

如果希望更通用，或者说能支持非幂等的请求，那可以用 atom 代替，然后再利用 useRecoilCallback 来强制更新 atom：

```js
const userInfoState = atomFamily({
  key: "UserInfo",
  default: (userID) => fetch(userInfoURL(userID)),
});

// 刷新查询的 React 组件
function RefreshUserInfo({ userID }) {
  // useRecoilCallback返回一个可以用于更新atom的函数
  const refreshUserInfo = useRecoilCallback(
    ({ set }) =>
      async (id) => {
        const userInfo = await request({ userID });
        set(userInfoState(userID), userInfo); // 强制设置atom
      },
    [userID]
  );

  // 每秒钟刷新一次用户信息
  useEffect(() => {
    const intervalID = setInterval(refreshUserInfo, 1000);
    return () => clearInterval(intervalID);
  }, [refreshUserInfo]);

  return null;
}
```

#### 简单请求

通常的同步数据查询方式，即从 atom 向某个 selector 派生一个状态出来，比如：

```ts
const currentUserIDState = atom({
  key: "CurrentUserID",
  default: 1,
});

const currentUserNameState = selector({
  key: "CurrentUserName",
  get: ({ get }) => {
    return tableOfUsers[get(currentUserIDState)].name;
  },
});

function CurrentUserInfo() {
  const userName = useRecoilValue(currentUserNameState);
  return <div>{userName}</div>;
}
```

那么异步数据，就只需要修改 selector 的 get 方法为异步函数，即返回一个 promise，或者改为 async 函数

```ts
const currentUserNameQuery = selector({
  key: "CurrentUserName",
  get: async ({ get }) => {
    const response = await myDBQuery({
      userID: get(currentUserIDState),
    });
    return response.name;
  },
});
```

这时对于组件来说，就不能再使用 useRecoilValue，因为显然这个状态是异步产生的。在 await 任务完成之前，状态都是空值；
因此应该采取一个特别的 hooks useRecoilValueLoadable。

```tsx
function UserInfo({ userID }) {
  const userNameLoadable = useRecoilValueLoadable(userNameQuery(userID));
  switch (userNameLoadable.state) {
    case "hasValue":
      return <div>{userNameLoadable.contents}</div>;
    case "loading":
      return <div>加载中……</div>;
    case "hasError":
      throw userNameLoadable.contents;
  }
}
```

注意 contents 是一个`T | Error | LoadablePromise<T>`的复合类型，会比较难用，所以可以封装一个小的 hooks 用于过滤 contents

```ts
function useLoadable(loadableSelector) {
  const { contents, state } = useRecoilValueLoadable(loadableSelector);
  const [loading, setLoading] = useState(true);
  switch (state) {
    case "hasValue":
      setLoading(false);
      return { contents, loading };
    case "loading":
      return { contents, loading };
    case "hasError":
      throw contents;
  }
}
```

还有一种方式是仍然正常使用 useRecoilValue，但是使用的组件外部应该包裹 Suspense 组件，并且还需要包裹 ErrorBoundary 去处理错误

```ts

<RecoilRoot>
  <ErrorBoundary>
    <React.Suspense fallback={<div>Loading...</div>}>
     <CurrentUserInfo />
    </React.Suspense>
  </ErrorBoundary>
</RecoilRoot>


const currentUserNameQuery = selector({
  key: 'CurrentUserName',
  get: async ({get}) => {
    const response = await myDBQuery({
      userID: get(currentUserIDState),
    }).catch(err => throw err)
    return response;
  },
});

function CurrentUserInfo() {
  const userName = useRecoilValue(currentUserNameState);
  return <div>{userName}</div>;
}
```

#### 带参请求

上面这种形式只适用于不需要额外参数的（请求参数只来自于其他 atom 的派生，比如上面的 id 来自于别的 atom，但没有调用时传入的参数）
如果需要额外参数，就需要使用 selectorFamily。selectorFamily 的 get 方法是一个高阶函数，类比 atomFamily 的使用

```ts
const userNameQuery = selectorFamily({
  key: "UserName",
  get: (userID) => async () => {
    const response = await myDBQuery({ userID });
    if (response.error) {
      throw response.error;
    }
    return response.name;
  },
});

function UserInfo({ userID }) {
  const userName = useRecoilValue(userNameQuery(userID));
  return <div>{userName}</div>;
}

function MyApp() {
  return (
    <RecoilRoot>
      <ErrorBoundary>
        <React.Suspense fallback={<div>加载中……</div>}>
          <UserInfo userID={1} />
          <UserInfo userID={2} />
          <UserInfo userID={3} />
        </React.Suspense>
      </ErrorBoundary>
    </RecoilRoot>
  );
}
```

#### 刷新请求

需要注意的是，如果我们通过上面的方式发起了请求，那就和在 React 中使用的 useEffect 形式完全不同。
在 React 中可以控制 effect 从而实现请求的刷新和初始化，把请求放在 useEffect 内部。但是 Recoil 则是把请求放在 selector 中，因此不能依靠 effect 刷新请求。

但是 selector 的一个特点是可以随着订阅的 atom 的更新而更新，因此我们可以让它订阅一个 atom，只需要刷新这个 atom，就能刷新请求。

```ts
import { selector, atom, DefaultValue } from "recoil";
import fetchData, { MockData } from "../mock/fetchData";

const requestIdState = atom({
  key: `requestId`,
  default: 0,
});
const dataQuery = selector({
  key: "dataQuery",
  get: async ({ get }) => {
    get(requestIdState);
    const { data } = await fetchData();
    return data;
  },
  set: ({ set }) => {
    set(requestIdState, (requestId) => requestId + 1);
  },
});

// 组件内

const [data, refresh] = useRecoilState(dataQuery);
useEffect(() => {
  refresh();
});
```

## 简单原理

Recoil 的原理要比上面两个都要复杂，他在底层做了更多事，同时也引入了更多的概念。
不过 recoil 大致可以看做，每个 atom 是一个数据源，都是一个单独的发布订阅的类；当我们在组件内调用 useRecoilValue 时，就相当于在这个类上订阅了一个更新。当 atom 更新时触发组件更新，可以通过 setState 或 useSyncExternalStore 等方法。

参考：https://cloud.tencent.com/developer/article/1961153

# 其他状态管理库

## Flux

flux 官方文档：https://facebook.github.io/flux/docs/

flux 是一种理念，redux 是基于 flux 的实现。
flux 的核心主要有四个：

![](https://pic.imgdb.cn/item/63da5ba9ac6ef860166eff72.jpg)

- View：视图层
- Action：动作，即数据改变的消息对象
  - Store 的改变只能通过 Action
  - 具体 Action 的处理逻辑一般放在 Store 里
  - Action 对象包含 type （类型）与 payload （传递参数）
- Dispatcher：派发器，接收 Actions ，发给所有的 Store
- Store：数据层，存放应用状态与更新状态的方法，一旦发生变动，就提醒 Views 更新页面

flux 的特点可以参考 redux 的特点（redux 是 flux 较为完善的实现），唯一不太一样的是，store 可以有多个，而 redux 中无论是 store 还是 reducer 都是一个

## MobX

![](https://pic.imgdb.cn/item/63da613dac6ef860167ceea4.jpg)

![](https://pic.imgdb.cn/item/63da6198ac6ef860167dd17d.jpg)

### 核心理念

有关 mobx 的原理性内容可以参考：https://juejin.cn/post/7083975275645501448

mobx 推崇一种响应式编程。 mobx 的实现和使用方式和 redux 差距很大。

- observable：即“被监听的”状态，通常用于定义一个状态。这个状态可以是任意类型，比如字符串、数字、对象、数组、map 等
- action：用于修改状态的函数，只有被标记为 action 的函数内部修改的状态才会生效，在其内部采用直接修改 state 值的形式，而非 redux 那种复制形式
- computed：一个 getter，相当于监听一个 observable 的函数，当 observable 变化时，computed 会执行并返回一个派生的状态。

上面三个是主要的形式，通过 observable 创建一个 state，然后用 action 修改，再配合 computed 配合派生状态，就完成了一个状态管理的基本结构。

下面是一个例子：
这里我们在一个类中使用 makeObservable 标记属性和方法的类型，把 todos 标记为 observable，用于修改 todo 的函数标记为 action，根据 todo 变化而变化的状态标记为 computed

```js
import { makeObservable, observable, action } from "mobx";

class TodoStore {
  todos = [];

  constructor() {
    makeObservable(this, {
      todos: observable,
      addTodo: action,
      removeTodo: action,
      todoLen: computed,
    });
  }

  addTodo(todo) {
    this.todos.push(todo);
  }

  removeTodo(index) {
    this.todos.splice(index, 1);
  }

  get todoLen() {
    return this.todos.length;
  }
}

// 注意这里返回的是store的实例，而非类本身
export default new TodoStore();
```

也可以使用 makeAutoObservable 自动将所有属性和方法变为 observable 或 action，把 getter 变为 computed

```js
class TodoStore {
  todos = [];

  constructor() {
    makeAutoObservable(this);
  }

  addTodo(todo) {
    this.todos.push(todo);
  }

  removeTodo(index) {
    this.todos.splice(index, 1);
  }

  get todoLen() {
    return this.todos.length;
  }
}
```

除了这三个之外，还有几个：

- autorun：类似 watch 的功能，它接受一个函数，函数内引用的状态（即上面的 store 的实例）变化时，这个函数会自动执行

```js
class OrderLine {
  price = 0;
  amount = 1;
  //...
}

const order = new OrderLine(0);

// 修改order的值，这个函数就会继续执行
const stop = autorun(() => {
  console.log("Total: " + order.total);
});
```

- reaction：类似于 autorun，但可以让你更加精细地控制要跟踪的可观察对象。

```js
class Animal {
  name;
  energyLevel;
  // ...
}

const giraffe = new Animal("Gary");

reaction(
  () => giraffe.isHungry,
  (isHungry) => {
    if (isHungry) {
      console.log("Now I'm hungry!");
    } else {
      console.log("I'm not hungry!");
    }
    console.log("Energy level:", giraffe.energyLevel);
  }
);
```

### 异步数据

在 mobx 内其实不用关心什么副作用，因为在 mobx 的类内部本身就是响应式的，不同于 react 或 redux 的设计。
需要注意的只是数据的更新应该有效。
最简单的方法是，在 action 内调用异步或副作用，然后通过 runInAction 来修改状态：

```js
class Store {
  data = [];
  state = "pending"; // "pending" / "done" / "error"
  constructor() {
    makeAutoObservable(this);
  }

  async fetchProjects() {
    this.data = [];
    this.state = "pending";
    try {
      const data = await fetchData();
      // await 之后，再次修改状态需要动作:
      runInAction(() => {
        this.state = "done";
        this.data = data;
      });
    } catch (error) {
      runInAction(() => {
        this.state = "error";
      });
    }
  }
}
```

### 性能

参考：https://www.v2think.com/mobx-redux-performance
通用的说法是，mobx 拥有比 redux 更好的性能。原因大致是，mobx 使用了更精准的状态更新方式，能使得状态更新时引起的订阅数量达到尽可能小。

mobx 的更新“精准”在哪里？

举个栗子，假设有一个状态，该状态是一个 user 对象，里边有 8 个不同的属性。B 组件只获取了其中的一个，比如说 user.name；当然对象内的具体属性是通过 computed 派生得到的

```js
class Store {
  user = {}
  constructor(){
    ...
  }
  get userName(){
    return user.name
  }
}
```

当 A 组件更新其他属性时，B 组件因为没有获取其他属性，因此不在触发的 listeners 中，也就不会更新；但当 A 组件修改 user.name 时，mobx 就能精确地调用 B 组件的更新，让 B 组件内的数据更新到最新。

相比之下，redux 天生就不具备这种能力，它只能修改整个状态，然后在 selector 函数内通过比较的方式来确定是否需要更新。虽然从渲染角度来说，两者都可以达到无关属性不影响渲染，但是由于 redux 本身需要调用 reducer、拷贝 state、执行每个 selector，这些消耗就导致性能要比 mobx 略差一些。

### 和 React 结合

上面的例子讲了如何创建一个 store。通过 class 的形式可以创建一个 store，然后导出类的实例，直接访问实例的属性就可以获得状态的输出：

```js
class TodoList{
  // ...
}

const todo = new TodoList()

todo.todos
todo.addTodo(...)
```

和 react 结合的话，最重要的是告知 react mobx 输出的状态是一个合法的 react state，从而使得 state 改变时，react 组件能够相应的更新。
在 redux 中采用的方法是通过 context 下发状态，然后再通过 useSelector 订阅状态并强制更新。
mobx 采用了一种“响应式”的方式，用 observer 将函数包裹起来，当函数内使用的状态发生变化时，就相应地更新函数：

```js
export default class useMobxStore {

  count: number = 0 // 初始化状态数据

  constructor() {
    // 对初始化数据进行响应式处理
    makeAutoObservable(this)
  }

  // 设置改变初始化数据方法
  addCount = () => {
    this.count++
    console.log(this.count)
  }
}

// 函数组件
import React from 'react'
import { observer } from 'mobx-react-lite' // 从mobx-react-lite内部引入observer让mobx与react进行关联
import UseMobxStore from '@/store'


const useMobxStoreState = new UseMobxStore()
const MobxDemo = () => {
  return (
    <div>
      <h2>{useMobxStoreState.count}</h2>
      <button onClick={useMobxStoreState.addCount}>+1</button>
    </div>
  )
}

export default observer(MobxDemo)
```

实际使用过程中，由于一个 store 对应一部分 state，项目中可能有很多 store，因此我们可以采用类似 combineReducer 的方式，将多个 store 合并到一个 RootStore 中去，然后用 context 将其下发，在组件中使用 useContext 获取 RootStore

```js
// store.ts

class Store1{
  constructor(rootStore:RootStore){
    this.rootStore = rootStore
    ...
  }
}

class Store2{
  constructor(rootStore:RootStore){
    this.rootStore = rootStore
    ...
  }
}

class RootStore{
  store1:Store1
  store2:Store2
  constructor(){
    this.store1 = new Store1(this)
    this.store2 = new Store2(this)
  }
}

const RootStateContext = createContext(new RootStore())

// some-component.tsx

const SomeComponent = observer(()=>{
  const {store1,store2} = useContext(RootStateContext)
})
```

可以参考：![](https://pic.imgdb.cn/item/640f5ef7f144a010075c3e09.jpg)

### 和 redux 比较

相同点：

- 两者都秉持视图的改变必须要通过 state 改变来实现的思想。而 state 也不能直接改变，需要通过 action 改变。如果直接修改 state，将不会导致视图的更新
- 单向数据流。虽然两者实现单向数据的方式不一样，并且 mobx 并没有很强调单向数据，即 action -> state -> view -> action 这种基本形式。

不同点：

1. Redux 是 FLUX 编程思想，单向数据流。Mobx 是 TFRP 编程思想，响应式编程。

redux 是每次返回一个全新的状态，一般搭配实现对象 immutable 的库来用。
mobx 每次都是修改的同一个状态对象，基于响应式代理，也就是 Object.defineProperty 代理 get、set 的处理，get 时把依赖收集起来，set 修改时通知所有的依赖做更新。

关于 mobx 的基本原理，可以这样理解：
当通过 makeObservable 处理类组件中的 state 之后，该 state 的 getter 和 setter 操作会被拦截。如果调用 setter，那就会触发执行保存的依赖，即类似 listeners。也就是说当更新 state 时，就会执行 listeners，从而执行一些更新，比如对 react 组件使用 forceUpdate。
当在组件中使用 state 的 getter 时（即获取 state），mobx 就会顺带收集该组件到全局中，这样到执行更新时就知道该更新哪个组件了。

2. redux 推崇纯函数的函数式编程，而 mobx 是面向对象的思想

3. mobx 的响应式能精准的通知依赖做更新，而 redux 只能全局通知，而且 mobx 只是修改同一个对象，不是每次创建新对象，性能会比 redux 更高。

mobx 的问题：

1. mobx 的编程思想和 react 不搭。mobx 是类似 vue 的响应式编程，和 react 推崇的单向数据、函数式编程矛盾。如果把大量状态都使用 mobx 这种形式去修改的话，肯定不符合 react 的 setState 修改原则
2. 调试困难，最大的问题是打印状态时打印的是一个 proxy
3. 对 hooks 支持一般

## zustand

zustand 是一个轻量级状态管理库，和 redux 一样都是基于不可变状态模型和单向数据流的，状态对象 state 不可被修改，只能被替换。

```js
import { create } from "zustand";

const useBearStore = create((set) => ({
  bears: 0,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
  removeAllBears: () => set({ bears: 0 }),
}));

function BearCounter() {
  const bears = useBearStore((state) => state.bears);
  return <h1>{bears} around here ...</h1>;
}

function Controls() {
  const increasePopulation = useBearStore((state) => state.increasePopulation);
  return <button onClick={increasePopulation}>one up</button>;
}
```

zustand 和其他状态管理库最大的一个差异在于它全局同例。也就是说它并没有采用 context，而是在全局仅保存一个 useStore，每次返回的是同一个实例。
不过 zustand 也对此做了优化，它可以保证组件内每个函数的引用都是固定的，类似 useMemo 的效果，可以保证减少重复渲染。

和其他状态管理库一样，zustand 也有类似的 action 和 selector，用于处理状态的派生和修改：

```js
import create from 'zustand';


// 添加第一个入参 set
export const useStore = create((set) => ({
  panelTabKey: 'antd',
  iconList: ...,
  antdIconList,

  selectIcon: (icon) => {
    set({ icon, open: false, filterKeywords: undefined });
	},
}))

// 展示用户会看到 icon list
export const displayListSelector = (s: typeof useStore) => {
  // ...
};
```

## Jotai

和 recoil 一样的追求原子性的库，使用起来也很简单，并且没有什么多余的 api，一个 useAtom 就够了。
和 recoil 相比还有一个特点是不需要一个独立的 key 值来用作表示 atom，每个 atom 都是自然独立的：

```js
import { atom } from 'jotai'

const countAtom = atom(0)
const countryAtom = atom('Japan')
const citiesAtom = atom(['Tokyo', 'Kyoto', 'Osaka'])
const mangaAtom = atom({ 'Dragon Ball': 1984, 'One Piece': 1997, Naruto: 1999 })

import { useAtom } from 'jotai'

function Counter() {
  const [count, setCount] = useAtom(countAtom)
  return (
    <h1>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>one up</button>
      ...
```

派生状态也很容易，是和 recoil selector 类似的写法，通过 get 函数捕获一个 atom，然后返回新的状态。只不过这里并不需要额外的 api，使用 atom 就好

```js
const doubledCountAtom = atom((get) => get(countAtom) * 2);

function DoubleCounter() {
  const [doubledCount] = useAtom(doubledCountAtom);
  return <h2>{doubledCount}</h2>;
}
```

缺点可能就是比较小众的库，维护上不一定比得上 recoil

# 总结

react 状态管理是一个比较麻烦的话题，因为 react 没有官方的解决方案。但是无论选择哪些状态管理库，都离不开几个话题：

1. 状态储存方案。redux 采用的是闭包，mobx 采用的是类，recoil 等原子化状态管理库其实也是采用全局对象（Map）

2. 状态流向。即一个状态如何从产生，到传递到存储结构内，再到具体的组件内部。绝大多数状态管理库都使用了发布订阅。

3. 接入 react 更新。即当状态发生修改时，如何让 react 组件更新？目前主要的方式有三种

- props 传入，比如 redux 的 connect 方法
- setState 更新，比如通过每次给 setState 传一个新的对象，保证更新
- useSyncExternalStore。这种方案理论上是最好的，不过对低版本的 react 没有兼容

4. 优化层面。对于状态管理库来说，最大的优化其实就是降低 react 组件的重复渲染。可以从以下几个方面考虑优化

参考：https://github.com/shaozj/blog/issues/36

- 批量更新，通过某些 batch 方法让频繁的触发更新合并为一次更新。比如 redux 提供了一个 batch 方法可以合并 dispatch：

```js
import { batch } from "react-redux";

function myThunk() {
  return (dispatch, getState) => {
    // 应该只导致一个组合重新渲染，而不是两个
    batch(() => {
      dispatch(increment());
      dispatch(increment());
    });
  };
}
```

- 状态的精准订阅。一个 state 有可能非常大，尤其是 redux 这种把整个应用的 state 都集合在一起的。那么当 reducer 执行时，整个 state 都改变，那么如何保证每个使用了 state 的组件不会发生渲染？换句话说，不同的子组件，需要的只是 store 上的一部分数据，如何在 store 发生变化后，仅仅影响那些用到 store 变化部分 state 的组件？

在 redux 中，不管是 connect 还是 useSelector 方法，都有一个重要逻辑，就是通过比较子组件真正所需数据的变化，来确定是否执行更新。如果组件内部依赖的状态（selector 函数执行结果）在两次 state 变动中没有改变，那么就不会执行组件更新。

其他状态管理库，mobx 采用的是响应式的更新，本身就是一个状态对应一个组件，因此不存在这种问题；recoil 同理
