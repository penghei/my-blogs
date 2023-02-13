---
title: typescript+react基础部分
date: 2021-11-20 22:19:35
tags: 日常学习
categories: React
cover: /img/typescript.png
---

## typescript+react 基础部分

### 安装及相关包使用

1. ts 版脚手架安装

```powershell
create-react-app myapp --template typescript
```

2. v5 路由

```powershell
yarn add react-router-dom@5.2.1
yarn add -D @types/react-router-dom
```

3. 最新路由

```powershell
yarn add react-router-dom @types/react-router-dom
```

4. pubsub

```powershell
yarn add pubsub-js @types/pubsub-js
```

## 注意事项

1. 接口定义函数类型的形式为`func():void`或者`func()=>void`;还有一种写法就是`(price:number):number`,用于`const func = (price:number)=>{}`这种形式.
   如果有参数必须要写明参数;一般来说,库或者内部函数都会有 React.自带,可以查查或者鼠标放上去看类型
2. 泛型一般用于:

- 函数放在函数名后面 括号前,给参数及内部使用:注意使用时不带尖括号;
- 类
- 数组或对象等构造函数调用的后面,括号前

3. 断言是比较重要的,当 ts 告诉你 xxx 没有属性 xx 时,就可以考虑断言.一般函数或者类这种大数据类型没有断言
4. Utility Types： 可以理解为基于 ts 封装的工具类型,是很有用的方法,下面详细叙述
5. React.FC 函数声明不是必须的,也可以直接当做函数来写
6. 遍历对象属性,或者用方括号获取对象属性时不能直接实现,可以利用`keyof`和`typeof`,如下:

```ts
//遍历对象
let obj = {
  name: "zzx",
  age: 18,
};
let key: keyof typeof obj;
for (key in obj) {
  console.log(obj[key]);
}
```

- 首先 typeof 获取对象类型,如果对象提前用了接口,那么直接用接口就可以;
- 然后用 keyof 获取键属性,上面代码的 keyof 值应该是`'name'|'age'`
- 之后就可以用方括号访问属性,或者遍历属性

7. 对于数组或对象,当 setState 奇奇怪怪无法生效的时候,可以尝试解构

```ts
//...
setxxxState(arr1); //如果不生效
setxxxState([...arr1]); //展开赋值,对象同理
```

```ts
interface IProps {}
const App = (props: IProps) => {};
```

## 常用 ts Utility Types

### Partial<T>

将 T 中所有属性转换为可选属性。返回的类型可以是 T 的任意子集

```ts
export interface User {
  name: string;
  age?: number;
}

type NUser = Partial<Userl>;
// =
type NUser = {
  name?: string | undefined;
  age?: number | undefined;
};
```

### Required<T>

将 T 的所有属性设置为必选属性来构造一个新的类型。与 Partial 相反

### Readonly<T>

将 T 中所有属性设置为只读

```ts
type NUser = Readonly<UserModel>;

// =
type NUser = {
  readonly name: string;
  readonly age?: number | undefined;
};
```

### NonNullable<T>

去除 T 中的 null 和 undefined 类型

## react tsx 相关

### 基础格式

```ts
import React from "react";

interface IProps {}
const TodoList: React.FC<IProps> = (props) => {
  return <div></div>;
};
export default TodoList;
```

- React.FC 是 react 函数组件的类型
- IProps 规定 props 类型.**当需要 withrouter 等添加 props 属性的操作时,可以让 IProps 继承这些接口**

### hooks

#### useState/useRef

```ts
const [todos, setTodos] = React.useState<string[]>([]);
const inputs = React.useRef<HTMLInputElement>(null);
let todoCount = useRef<number>(0);
let inputValue = useRef<string>("");
```

- usestate 最好规定 state 的类型
- useRef 如果是 html 元素,就也要设置该元素的 html 类型(比如 HTMLInputElement,HTMLDivElement)
- 如果是一个值的 Ref,也要设置好类型(存储的数据的类型)

#### useEffect/useLayoutEffect

```ts
useEffect(() => {
  //useEffect没有特别大区别
  todoCount.current = todos.length;
}, [todos]);
```

useEffect 没有特别大区别

#### useContext

```ts
import { createContext, useContext } from "react";

interface IContext {
  backgroundColor: string;
  color: string;
}

const ThemeContext = createContext<IContext>({
  backgroundColor: "black",
  color: "white",
});

const themeContext = useContext<IContext>(ThemeContext);
```

- 需要规定 context 类型,也就是 useContext 和 createContext 的参数类型;同时也是 Context.Provider 的 value 类型

#### useReducer

```ts
const reducer = (state: IState, action: IAction): IState => {
  switch (action.type) {
    case "PLUS":
      return state + (action.data as number); //这里有可能为null
    case "MINUS":
      return state - (action.data as number);
    default:
      return state;
  }
};

const [count, dispatch] = useReducer(reducer, 0);
```

- reducer,state 和返回值类型相同,复杂的话最好定义接口;IAction 是定义的 action 类型,一般需要`type:string`和`payload?:any`
- 注意 dispatch 有 react 提供的类型:`React.Dispatch<IAction>`,也就是要用到上面的 action 确定的类型

#### 其他 hooks

可以参考这篇文章:https://zhuanlan.zhihu.com/p/99341354

### React 与类型相关的属性或 API

#### 事件处理(event 对象)

ts 中的 event 事件处理比较麻烦,需要规定好事件类型和触发事件的 HTML 元素类型,才能不报错的获取到 e 的相关属性
举个栗子,常用的按钮 click 事件:

```ts
<button onClick={getValue}>clickme</button>;
//...
function getValue(e: React.MouseEvent<HTMLButtonElement>) {
  const value = (e.target as HTMLButtonElement).textContent;
}

//另一种写法，handler是作用于事件处理函数的，event直接作用于事件对象本身
const getValue: React.MouseEventHandler<HTMLButtonElement> = (e) => {
  //...
};
```

- 函数的 e 形参后面的类型:
  - React 自带了一些事件 handler,可以通过将鼠标放在 onClick 上看到对应类型;这里是鼠标事件,常见的还有`KeyboardEvent(onKeyPress)` `ChangeEvent(onChange)`
  - 后面的泛型规定了 HTML 元素类型,这里点击的按钮所以是按钮类型
- 底下获取 e.target 属性的时候还需要断言,主要原因是 e 有可能是 undefinded,这里断言之后才可以拿到一些元素特有属性,比如 value,textContext 等
- 实在麻烦还可以用通用类型:`React.ReactEvent<HTMLElement>`

#### 事件处理函数（EventHandler）

注意EventHandler实际上是事件处理函数的类型，而非事件对象event本身的类型。
以onclick为例，其他事件类似：
```ts
type CompProps = {
  onClick: React.MouseEventHandler<HTMLDivElement>
}
```
而onclick的回调函数内部的event，才是上面的event类型。


#### React 元素/结点/组件

React 元素对应属性:`React.ReactElement<P> 或 JSX.Element`
比如可以自定义一段 tsx

```ts
const elementOnly: React.ReactElement = <div /> || <MyComponent />;

const info:React.ReactElement = (
    <div>
        <h2>hello world</h2>
    </div>
)
//...
<div>{info}</div>

```

#### React CSS

一般用于标识 tsx 文件中的 style 对象

```ts
React.CSSProperties;
//举个栗子:
const styles: React.CSSProperties = { display: "flex" };
const element = <div style={styles}></div>;
```

## 其他 TIPs

在 map 循环列表渲染中获取对应元素的方法

```ts
{
  todos.map((obj, index) => {
    return (
      <li key={index}>
        {obj.value}
        <button onClick={(e) => deleteTodos(obj.value, e)}>delete</button>
      </li>
    );
  });
}
//...
function deleteTodos(value: string, e: React.MouseEvent<HTMLElement>) {
  console.log(value);
}
```
