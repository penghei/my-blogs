---
title: react-router的使用
date: 2021-10-27 22:17:20
tags: 日常学习
categories: React
cover: /img/reactrouter.png
---

# 配置代码

## index.js

```js
import { BrowserRouter } from "react-router-dom";
ReactDOM.render(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
  document.getElementById("root")
);
```

BrowserRouter 和 HashRouter 区别:

- hash 路径后面有#,就是 vue 的那个;hash 不支持 state.location 传值
- hash 不需要服务器配合,而 browser 实际上是真的请求了
- 用默认的 BrowserRouter 就完事了(bushi

## App.js

如果需要整体页面跳转,App 组件就不需要放其他东西,只需要放需要整页面跳转的 route

```js
function App() {
  return (
    <div className="App">
      <Route path="/login" component={SignInPage}></Route>
      <Route path="/main" component={MainPage}></Route>
      <Redirect to="/login" />
    </div>
  );
}
```

这里只放了两个 route 和 redirect,目的就是 main 组件和 login 组件都会改变整体页面,至于各自的子路由,在各自组件中配置就行
另外一点就是引入的组件是页面组件.注意哪怕页面组件简单到只是引入组件都需要,route 中一定是页面组件

## pages 文件夹

用于放页面组件,下面会说

## 其他页面中的

### 子路由

components/Login/LoginBlock.jsx

```js
//...
<Route path="/login/signin" component={Login}></Route>
<Route path="/login/register" component={Register}></Route>
<Redirect to="/login/register" />
//...
export default withRouter(LoginBlock);
```

子路由的配置,需要在哪展示就在哪配置路由就行.注意 react 中子路由的判断是依赖组件的,如果 A 和 B 都设定了路由并且 A 是 B 的父组件,那么 B 中就是 A 的子组件.另外也可以通过 path 设置
withRouter 是组件使用编程式路由导航的基础,只有用了它才会在 props 中加入路由选项

### 编程式跳转

```js
props.history.push({
  pathname: "/login/signin",
  state: userObj,
});
```

需要跳转的组件也需要 withRouter 包装;
如果需要参数,最好用 state,就像这样传递,在接收组件中用`props.location.state`接收

# react 路由的(个人感觉)

react 路由没有专门的封装,晚上查找的封装多多少少不完全也有 bug.所以 react-router 的 Route 组件相当于同时完成了<router-view>的展示和路由的注册.也就是说其实不必单独注册.这当然简化了代码,但是增加了项目大了之后的寻找路由难度也会增大
所以这是我个人的想法:

- 在大项目中,最好单独写个路由配置文件,不需要使用只要自己看清楚就行,主要是防止搞混;
- 一定要分好页面组件和一般组件,route 中引入的尽可能都是页面组件而不是一般组件,防止乱引入

