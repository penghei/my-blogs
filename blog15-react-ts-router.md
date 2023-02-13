---
title: typescript+react:react-routerv6和旧版v5的使用
date: 2021-11-20 22:19:35
tags: 日常学习
categories: React
cover: /img/typescript.png
---

## react-router v5(最常见版本)

### 安装

```powershell
yarn add react-router-dom@5.2.1
yarn add -D @types/react-router-dom
```

### 在 ts 中使用注意

接下来是一些与 jsx 使用不太一样的地方,一样的就不再写了

#### withrouter 的使用

```ts
interface IProps extends RouteComponentProps {
  //解决的办法:让接口继承RouteComponentProps即可,这样props就会多出react-router自带的方法
}
//...
<button
  onClick={() => {
    props.history.push({
      pathname: "/home/display",
      state: valueToDis,
    });
  }}
>
  display
</button>;
//...
export default withRouter(Home);
```

解决的办法:让接口继承 RouteComponentProps 即可,这样 props 就会多出 react-router 自带的属性,比如 history,location 等
这个东西是从 react-router-dom 获取的

```ts
import { RouteComponentProps } from "react-router-dom";
```

state 参数的属性需要如下规定

#### state 参数

```ts
interface IRouterState {//注意最好规定一下路由state参数类型
    name:string,
    age:number
}
//...
const IRouterState = (props.location.state as IRouterState)//断言参数类型,不然会报错
//...
<div>
    <h1>Display:{IRouterState.name}:{IRouterState.age}</h1>
</div>
```

需要断言 state 参数类型,不然会报错

## react-router v6(最新)

react-router v6 更改了很多 api,用法也大改.但是现在很新,网上也没什么教程和问题解决.主要更新点如下:

### 更新点

- 集成化路由:
  - 所有路由注册都在 App 下,配置一般在 App 或 index 中完成,使用`<Outlet/>`在组件任何需要的位置放置并显示(类似 vue 的`<router-view/>`)
  - 也可以分开展示,但是路径要完整;
- 取消了 Redirect,改变为上面 10 行的形式:`path="*"`代表没匹配到的路径,`{<Navigate to="/home" />}`表示导向
- 取消了 switch,用 Routes 代替
- 取消了 component 属性,现在是 element,并且括号里边是完整的组件(要带尖括号)
- 嵌套路由,嵌套路由可以将子路由的注册放在父路由之下,不用再写父路由的路径;另外如果希望传递 parmas 参数,可以把之前的`path="home/:id"`改为子路由的形式
- 传参展示不能传以前的 state 参数,正在了解中
- 取消了 history/location 等对象,改变为 useNavigate(),useLocation()

### tsx 配置

`<BrowserRouter>`不变,仍然套在 App 外面

```ts
//App.tsx
<div className="App">
  <Routes>
    <Route path="home" element={<Home />}>
      <Route path="counter" element={<Counter />} />
      <Route path="input" element={<Input />} />
    </Route>
    <Route path="msg" element={<Message />}>
      <Route path=":msgvalue" element={<Message />}></Route>
    </Route>
    <Route path="*" element={<Navigate to="/home/counter" />}></Route>
  </Routes>
</div>
```

### 手动跳转 useNavigate()

不再使用 props.history 的形式,使用 useNavigate()获取操作方法

```ts
const nav = useNavigate();
//...
<button onClick={()=>{nav('/home')}}>skip</button>//
<button onClick={()=>{nav(`/home/${value}`)}}>skip</button>//传参
<button onClick={()=>{nav('/home',{replace:true})}}>skip</button>//replace方法
```

### 获取路由对象 useLocation()

同理,location 改为这个

```ts
const loc = useLocation();
console.log(loc);
```

loc 中一般含有路径信息/参数等

### 获取 params 参数 useParams()

```ts
const params = useParams(); //固定为string
return (
  <div>
    <h1>This is params:{params.value}</h1>
    {/*这个.value是配置中:后面写的参数名*/}
  </div>
);
```

### 其他

暂时还在学习,如果有新的再写下来
