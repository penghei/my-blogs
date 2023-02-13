---
title: typescript+react:自定义hooks代替redux
date: 2021-11-20 22:19:35
tags: 日常学习
categories: React
cover: /img/typescript.png
---

## react 自定义 hooks 代替 redux

简单介绍:利用 react 自带的 useReducer 产生对应的 state 和 dispatch,然后再通过 Context 分发到每一层,就可以实现
jsx 版本就不再叙述了,详情来自于这篇文章:https://juejin.cn/post/6844903854807482382

### 配置

1. 首先创建 redux 文件夹,创建 xxxReducer.tsx 文件.文件主要结构如下,其他 reducer 类似:(注意是 tsx)

```ts
//colorReducer.tsx
import React, { createContext, useReducer } from "react";
interface IContext {
  //创建设置context类型,就是下面provider的value属性类型,要有color和dispatch
  color: string;
  dispatchColor?: React.Dispatch<IAction>; //dispatch是react自带的Dispatch属性,注意要加一个IAction泛型表示dispatch的参数
}

interface IAction {
  //action的类型
  type: string;
  data: string | null;
}
interface IProps {
  //一般是固定的,props.children类型,不写不能用props.children
  children:
    | boolean
    | React.ReactChild
    | React.ReactFragment
    | React.ReactPortal
    | null
    | undefined;
}
export const ColorContext = createContext<IContext>({} as any); //创建context,需要有context泛型规定参数类型;这里因为不设初值所以直接as any

const reducer = (state: string, action: IAction): string => {
  //reducer,state和返回值类型相同,复杂的话最好定义接口;action就是上面的接口
  switch (action.type) {
    case "CHANGECOLOR":
      return action.data as string; //这里有可能为null
    default:
      return state;
  }
};
//注意多个reducer区别好导出名:
//xxx是state
//xxxcontext是useContext使用的参数
//xxxReducer是包裹在App外面的Context
//dispatchxxx是对应的dispatch
export const ColorReducer = (props: IProps) => {
  //导出reducer到context,props类型
  const [color, dispatchColor] = useReducer(reducer, "blue"); //这块基本正常
  return (
    <ColorContext.Provider value={{ color, dispatchColor }}>
      {/*上面规定好的话,这块基本没啥问题的*/}
      {props.children}
    </ColorContext.Provider>
  );
};
```

主要解释:

- 需要三个主要接口:IContext IAction IProps,如果 state 类型复杂还需要 IState
  - IContext 用于`createContext<IContext>`设定 context 参数类型和初值类型,以及下方 Context.Provider 的 value 类型,也就是子组件接收到的类型,**很重要**
    - 第一个一般是 reducer 的 state 数据类型
    - 第二个是 React 自带的 Dispatch 类型,`注意加上IAction泛型`才能确定 dispatch 内容
  - IAction 用于设定 Action 对象的属性,一般是 type:string 和 data:any 两个属性,也会影响 dispatch 类型
  - IProps 固定,props.children 类型,不写不能用 props.children
  - IState 确定 reducer 的返回类型和 state 参数类型
- 创建 context,需要有 context 泛型规定参数类型;这里因为不设初值所以直接 as any
- reducer 的返回值有可能为 null,所以需要断言;如果确实要删除返回 null,可以设定好 IAction 中 data 类型为`xxx | null`
- 多个 reducer 的命名规范:
  - xxx 是 state
  - xxxContext 是 useContext 使用的参数
  - xxxReducer 是包裹在 App 外面的 Context
  - dispatchxxx 是对应的 dispatch

2. 在 App 的外面套上导出的`xxxReducer`,多个可以嵌套;如果有路由也可套在外面或者 index 里

```ts
//App.tsx
function App() {
  return (
    <div className="App">
      <ColorReducer>
        <CounterReducer>{/*子组件*/}</CounterReducer>
      </ColorReducer>
    </div>
  );
}
```

3. 子组件使用,需要先 useContext 导入 xxxContext,具体使用如下:

```ts
//ChangeColor.tsx
const ChangeColor: React.FC<IProps> = (props) => {
  const { dispatchColor } = useContext(ColorContext);
  const changeColor: React.MouseEventHandler<HTMLDivElement> = (e) => {
    if (dispatchColor)
      dispatchColor({
        type: "CHANGECOLOR",
        data: (e.target as HTMLButtonElement).textContent,
      });
  };
  return (
    <div onClick={changeColor}>
      <button>red</button>
      <button>yellow</button>
    </div>
  );
};
```

主要解释:

- 先 useContext 导入 ColorContext,注意这里解构只获取了 dispatch 方法,同理也可以在展示组件中获取 state
- 第 5 行对 dispatch 的判断:如果不判断 dispatch 有可能是 undefined,只能通过这种方式,以后有更好的方法会更新

### 总结及原理概述和缺陷

利用 react 自带的 useReducer 产生对应的 state 和 dispatch,然后再通过 Context 分发到每一层,就可以实现.
在 reducer 文件中,除了主要 reducer 之外就是 context,通过`props.children`的方法使得嵌套的子组件可以使用 reducer
而对于在 ts 中的使用,基本上全靠自己摸索;最主要的还是类型的确定:比如 useReducer 产生的 dispatch 是 React 自带的属性,createContext 需要 IContext 说明等.只要这些类型通过,理论上任何 js 的使用都可以用 ts 代替
缺陷:state 其中一个改变,即使不持有 state 的组价也会被迫更新;目前还没有见到具体问题,后续如果有问题或者有更好的解决办法还会更新

## Recoil
2022/1/15更新：Recoil库可以做到类似的效果，并且封装的很好
Recoil官方文档：https://recoiljs.org/zh-hans/docs/introduction/getting-started