---
title: react-router使用及原理浅析
date: 2023-02-16 18:06:32
tags: 日常学习
categories: React
cover: /img/react.png
---

# 路由原理

## history、React-router、React-router-dom 三者关系

![](https://pic.imgdb.cn/item/624a829e239250f7c5cdeae5.jpg)

- `history`： history 是整个 React-router 的核心，里面包括两种路由模式下改变路由的方法，和监听路由变化方法等。
- `react-router`：既然有了 history 路由监听/改变的核心，那么需要调度组件负责派发这些路由的更新，也需要容器组件通过路由更新，来渲染视图。所以说 React-router 在 history 核心基础上，增加了 Router ，Switch ，Route 等组件来处理视图渲染。
- `react-router-dom`： 在 react-router 基础上，增加了一些 UI 层面的拓展比如 Link ，NavLink 。以及两种模式的根部路由 BrowserRouter ，HashRouter 。

## 两种路由方式

路由主要分为两种方式，一种是 history 模式，另一种是 Hash 模式。History 库对于两种模式下的监听和处理方法不同：

- history 模式下：`http://www.xxx.com/home`
- hash 模式下： `http://www.xxx.com/#/home`

两种模式分别被成为`BrowserRouter` 或者是 `HashRouter`，实际上就是 React-Router-dom 根据 history 提供的 `createBrowserHistory` 或者 `createHashHistory` 创建出不同的 history 对象。

```js
import { createBrowserHistory as createHistory } from "history";
class BrowserRouter extends React.Component {
  history = createHistory(this.props);
  render() {
    return <Router history={this.history} children={this.props.children} />;
  }
}
```

### BrowserHistory (BrowserRouter)

BrowserRouter 详解参考：https://zhuanlan.zhihu.com/p/350145707

1. **改变路由**

改变路由，指的是通过调用 api 实现的路由跳转，比如开发者在 React 应用中调用 `history.push` 改变路由，本质上是调用 `window.history.pushState` 方法。

```js
window.history.pushState(state, title, path);
```

1. `state`：一个与指定网址相关的状态对象， popstate 事件触发时，该对象会传入回调函数。如果不需要可填 null。
2. `title`：新页面的标题，但是所有浏览器目前都忽略这个值，可填 null 。
3. `path`：新的网址，必须与当前页面处在同一个域。浏览器的地址栏将显示这个地址。

当调用这个方法时，不会触发页面刷新，只是导致 History 对象发生变化，地址栏也会改变。然后后面可以通过`History.state`属性读出曾经 push 的值。

---

另一种方法是 replace，不会记录前一次的历史，当访问历史时只能访问到最前面一个用`pushState`传入的路径。

```js
window.history.replaceState(state, title, path);
```

参数和 pushState 一样，这个方法会修改当前的 history 对象记录， 但是 `history.length` 的长度不会改变。

2. **监听路由**

```js
window.addEventListener("popstate", function (e) {
  /* 监听改变 */
});
```

同一个文档的 history 对象出现变化时，就会触发 popstate 事件
`history.pushState` 可以使浏览器地址改变，但是不会刷新页面。

> 注意 ⚠️：用 `window.history.pushState()` 或者` window.history.replaceState()` 不会触发 popstate 事件。 popstate 事件只会在浏览器某些行为下触发, 比如点击后退、前进按钮或者调用 `window.history.back()`、`window.history.forward()`、`window.history.go()`方法。
> 虽然直接调用`window.history.pushState()`并不会触发 popstate 事件，但是 React-Router 会记录这次 push 的值，新增一条历史记录，然后触发更新`location`的 setState 并更新

具体来说是这样：

- 首先，虽然 BrowserRouter 用到了 h5 history api，但它并非直接监听 popstate 事件，而是通过自己内部的控制修改状态、更新页面
- push 和 replace 方法可以修改地址栏的值，同时改变路由。他们的代码内部通过调用`history.pushState`方法修改地址栏，然后再调用 setState 方法；这个方法作用是根据当前地址信息（location）更新history，然后再取出原先设置的 listener 函数依次执行，这些 listener 都包含了修改 history 对象和路由组件状态的代码，这样就实现了改变路由的显示。
  push 函数简化如下：

```js
function push(path, state) {
  const action = "PUSH";
  // 创建一个location对象
  const location = createLocation(path, state, createKey(), history.location);
  // 进行路由切换
  const href = createHref(location);
  const { key, state } = location;

  if (canUseHistory) {
    // 调用HTML5 history pushState history.pushState(state, title[, url])
    // pushState可以改变url但是不会刷新页面、也不会发起请求
    globalHistory.pushState({ key, state }, null, href);
    setState({ action, location });
  }
}

function setState(history) {
  history.length = globalHistory.length;
  // transitionManager就是一个切换路由的管理器，上面保存着一个listeners数组，以及添加、触发监听器的方法
  transitionManager.notifyListeners(history.location, history.action); // 这一步是执行之前注册的listeners
}
```

- push 和 replace 操作并不会触发 popstate 事件，而对于会触发 popstate 的操作，比如调用`history.go`，或者点击浏览器的前进、后退，还有一套额外的监听。通过监听这些会触发 popstate 的事件，再创建新的 location 对象，然后通过 setState 实现页面的更新。
  简化后的代码如下：

```js
// 其实就是更改state，只不过这里是监听popstate做出的更改
function handlePop(location) {
  const action = "POP";
  setState({ action, location });
}

// 记录是否有添加listener
let listenerCount = 0;

// 监听历史条目记录的改变
function checkDOMListeners(delta) {
  listenerCount += delta;
  if (listenerCount === 1 && delta === 1) {
    // 监听popstate事件
    // PopStateEvent就是history对象上的popstate事件
    window.addEventListener(PopStateEvent, handlePop);
  } else if (listenerCount === 0) {
    // 移除popstate事件
    window.removeEventListener(PopStateEvent, handlePop);
  }
}
```

另外，路由的改变离不开 location 对象的更新。每次通过 push、replace 方法更改路由时，都会创建新的 location 对象，包括 path、search、hash 等 h5 location 对象自带的，以及 h5 history 对象上的 state 等：
location 对象将会影响 Router 组件的状态，后续还会影响其他 Route 组件的展示。在各个 listener 中，location 也是传递的核心，包含了路由整体的状态变化。

简化代码如下

```js
function createLocation(path, state, key, currentLocation) {
  let location;
  location = { ...path };
  location.state = state; // 这个state实际上是history里边的那个state，即可以通过history.pushState改变的那个state
  location.pathname = decodeURI(location.pathname);
  location.search = "?" + location.search;
  location.hash = "#" + location.hash;
  location.state = state;
  location.key = key;
  // currentLocation指的是window.history.location
  if (currentLocation) {
    if (!location.pathname) {
      location.pathname = currentLocation.pathname;
    } else if (location.pathname.charAt(0) !== "/") {
      location.pathname = resolvePathname(
        location.pathname,
        currentLocation.pathname
      );
    }
  } else {
    if (!location.pathname) {
      location.pathname = "/";
    }
  }
  return location;
}
```

3. **服务端要求**

`BrowserRouter` 要求服务端对发送的不同的 URL 都要返回对应的 HTML。
造成的影响是，如果只是在不刷新的情况下，只在前端进行跳转一般没有任何问题，但是在直接在此路由下进行页面的刷新，就会得到一个 404。
解决方法主要有两种，一种是在后端做对应的配置，不管请求路径是什么，都始终返回单个 html。比如在 express 中设置：

```js
app.get("/*", function (req, res) {
  res.sendFile(path.join(__dirname, "build", "index.html"));
});
```

在 nginx 中也可以

还有一种方法是在 webpack 的 devserver 配置项添加一个属性：

```js
devServer: {
  historyApiFallback: true;
}
```

#### HashHistory (HashRouter)

> 本质上是利用`window.location`对象，这是一个包含浏览器 URL 相关的信息和操作方法的对象，参考 https://wangdoc.com/javascript/bom/location.html
> 其中主要利用`Location.hash`属性。这个属性会返回一个 hash 值，一般情况下是 URL 标识中的 `#` 和 后面 URL 片段标识符。比如很多页面的锚点，就可以通过此 api 访问。

1.  改变路由 `window.location.hash`

通过 `window.location.hash` 属性获取和设置 hash 值。开发者在哈希路由模式下的应用中，切换路由，本质上是改变 `window.location.hash` 。

2. 监听路由

```js
window.addEventListener("hashchange", function (e) {
  /* 监听改变 */
});
```

hash 路由模式下，监听路由变化用的是 `hashchange`。这也是和`BrowserRouter`的主要区别，即改变和监听路由的方式不一样。

---

hashrouter 的实现就比 broswerRouter 简单的多，其实就是监听 hashchange 事件，根据改变的 hash 更新 location 对象，然后触发 Router 组件的 listener 并更新。

```js
export default class HashRouter extends React.Component {
  state = {
    location: {
      pathname: window.location.hash.slice(1), // #/user
      state: window.history.state,
    },
  };
  //组件挂载完成之后，根据hash改变pathname的值
  componentDidMount() {
    window.addEventListener("hashchange", (event) => {
      this.setState({
        ...this.state,
        location: {
          ...this.state.location,
          pathname: window.location.hash.slice(1),
        },
      });
    });
    window.location.hash = window.location.hash || "/"; //如果没有hash值就给一个默认值
  }
  render() {
    //渲染子组件 如果子组件里面嵌套着二级三级路由需要通过上下文Context取
    let routerValue = {
      location: this.state.location,
    };
    return (
      // this.props.children
      <RouterContext.Provider value={routerValue}>
        {this.props.children}
      </RouterContext.Provider>
    );
  }
}
```

# 基本构成

## 三个基本对象：history，location，match

在路由页面中，开发者通过访问 props，发现路由页面中 props 被加入了这几个对象：

- `history` 对象：history 对象保存改变路由方法 push ，replace，和监听路由方法 listen 等。大概属性如下：

```js
{
  length:6,// history长度
  action:'PUSH'|'REPLACE'|'POP',// 当前操作
  location:{
    pathname: '/xxx',// 当前路径
    search: "name='xxx'",// url参数，即`ww.xx.com?name=xxx`问号后面的参数
    hash: '#aaaa', // 哈希值
    state: { // 传递的state参数
      name: 'zzx',
      age: 18
    }
  },
  push(path,state){},
  replace(path,state){},
  ...
}
```

- `location` 对象：可以理解为当前状态下的路由信息，包括 pathname ，state 等。大概看起来像这样：

```js
{
  key: 'ac3df4', // HashHistory 没有
  pathname: '/somewhere',
  search: '?some=search-string',
  hash: '#howdy',
  state: {
    [userDefined]: true
  }
}
```

- `match` 对象：这个用来证明当前路由的匹配信息的对象。存放当前路由 path 等信息。
  - `params` - (object) 从对应于路径动态段的 URL 解析的键/值对
  - `isExact` - (boolean) 如果整个 URL 匹配则为真（无尾随字符）
  - `path` - (string) 用于匹配的路径模式。
  - `url` - (string) URL 的匹配部分。

## 路由组件

### Router

Router 组件不等同于`BrowserRouter` 或者 `HashRouter` ，它是一个单独的组件，可以直接取出来用，但是一般不会主动使用它。
`BrowserRouter` 或 `HashRouter` 是不同模式下向容器 Router 中注入不同的 history 对象后的结果，也就是相当于封装了自己构造的`history`对象之后的组件，因此一般不会使用原生 Router。

但是这个组件仍然十分重要，是监听、修改路由变化并分发状态的核心组件：

```js
class Router extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      location: props.history.location,
    };
    this.listen = props.history.listen((location) => {
      /* 当路由发生变化，派发更新 */
      this.setState({ location });
    });
  }
  /* .... */
  componentWillUnmount() {
    if (this.listen) this.unlisten();
  } // 取消监听
  render() {
    return (
      <RouterContext.Provider
        children={this.props.children || null}
        value={{
          history: this.props.history,
          location: this.state.location,
          match: Router.computeRootMatch(this.state.location.pathname),
          staticContext: this.props.staticContext,
        }}
      />
    );
  }
}
```

React-Router 是通过 context 上下文方式传递的路由信息，包括 history 对象、location 等关键信息；

当通过 push、replace 等方法，或者点击浏览器的后退、前进按钮时，根据上面介绍 BrowserHistory 的部分，其实本质上就是触发了 listener 并传入一个新的 location 对象。
这里就是注册 listener 的地方。可以看到当传入新的 location 时，Router 组件的状态会被改变，下发的 context 的 value 也会改变，那么他之下持有该 value 的 Route 组件的状态也会改变，从而导致页面的更新。

所以改变路由，本质上是 location 改变带来的更新作用。

### Route

核心的匹配组件，主要功能是： 匹配路由，路由匹配，渲染组件。

Route 可以通过 `RouterContext.Consumer` 来获取上一级传递来的 context value 进行路由匹配（包含 history 对象、最新的 location 对象等）；如果匹配，渲染子代路由。并利用 context 逐层传递的特点，将自己的路由信息，向子代路由传递下去。这样也就能轻松实现了嵌套路由。

Route 的写法主要有四种：

```js
{
  /* Route Component形式 */
  /*无法传递父组件中的信息*/
}
<Route path="/router/component" component={RouteComponent} />;
{
  /* Render形式 */
}
<Route
  path="/router/render"
  render={(props) => <RouterRender {...props} />}
  {...mes}
/>;
{
  /* chilren形式 
    子组件不能获取到路由相关的props
  */
}
<Route path="/router/children">
  <RouterChildren {...mes} />
</Route>;
{
  /* renderProps形式 */
}
<Route path="/router/renderProps">
  {(props) => <RouterRenderProps {...props} {...mes} />}
</Route>;
```

Route 组件的属性：

1. `path`：路径，即`history.location`值，是一个字符串，用于匹配路径。当 Route 是嵌套子集路由时，要写完整父级路由。

> 官方文档：
> `<Route path='/xxx'>` 匹配 URL 的**开头**，而不是整个内容。因此 `<Route path="/">` 将和任何 url 匹配，应当放在 `<Switch>` 的最后。

2. `component`：要渲染的组件
3. `exact`：精准匹配，即路径必须是完整和 path 的字符串相等。**如果是嵌套路由的父路由，千万不要加 exact=true 属性。换句话只要当前路由下有嵌套子路由，就不要加 exact。**

配置可以使用`react-router-config`库做更清晰的配置，详见 https://juejin.cn/post/6911497890822029326

### Switch

> 官方文档：
> 当 `<Switch>` 被渲染时，它会搜索其子 `<Route>` 元素以找到其路径与当前 URL 匹配的元素。当它找到一个时，它会渲染那个 `<Route>` 并忽略所有其他的。这意味着您应该将具有更具体（通常更长）路径的 `<Route>` 放在不太具体的路径之前。

Switch 作用是先通过匹配选出一个正确路由 Route 进行渲染，而不是一次性渲染所有组件。
再由文档描述的一样，switch 会只渲染其匹配到的第一个，因此最好把更长的 path 放到短的前面。

```js
<Switch>
  <Route path="/home" component={Home} />
  <Route path="/list" component={List} />
  <Route path="/my" component={My} />
</Switch>
```

### Redirect

`Redirect`体现一种“强制导航”，即当路由匹配到这一级时，将会被强制改为`to='/xxx'`指向的路径。
因此该组件应该被放在路由匹配的最后，作为找不到路径的“兜底”选项；

另外一种情况是用于页面鉴权，比如：

```js
{
  noPermission ? (
    <Redirect from={"/router/list"} to={"/router/home"} />
  ) : (
    <Route path="/router/list" component={List} />
  );
}
```

如果 `/router/list` 页面没有权限，那么会渲染 Redirect 就会重定向跳转到 `/router/home`，反之有权限就会正常渲染 `/router/list`。

### 从路由改变到页面跳转流程图

![](https://pic.imgdb.cn/item/624ab386239250f7c53abc4a.jpg)

# 路由功能

## 传参

### url 拼接

```js
const name = "alien";
const mes = "let us learn React!";
history.push(`/home?name=${name}&mes=${mes}`);
```

这种方式通过 url 拼接，比如想要传递的参数，会直接暴露在 url 上，而且需要对 url 参数，进行解析处理。
拼接完成后，可以通过路由的`location.search`属性直接获取

### state 路由状态

```js
const name = "alien";
const mes = "let us learn React!";
history.push({
  pathname: "/home",
  state: {
    name,
    mes,
  },
});
```

可以在 location 对象上获取上个页面传入的 state 。

```js
const { name, mes } = location.state;
```

**缺点是刷新页面值会丢失**

## 动态路径参数路由

路由中参数可以作为路径。比如像一些有大量要具体展示的商品信息、文章等，就是通过路由路径带参数（文章 ID ）来实现精确的定位。在绑定路由的时候需要做如下处理。

```js
<Route path="/post/:id" />
```

`:id` 就是动态的路径参数，

路由跳转：

```js
history.push("/post/" + id); // id为动态的文章id
```

在跳转的组件内部，可以通过`params.id`获取。

```js
let { id } = useParams();
```

## `<Link>`和`<NavLink>`

创建一个可以用于跳转路由的链接，相当于特殊的`<a>`标签。

```js
<Link to="/about">About</Link>
```

`<Link>`的主要属性 to，其值可以是字符串、携带 params 参数的字符串、返回路径字符串的函数，还可以是一个对象

```js
<Link to="/courses?sort=name" />

<Link
  to={{
    pathname: "/courses",
    search: "?sort=name",
    hash: "#the-hash",
    state: { fromDashboard: true }
  }}
/>

<Link to={location => ({ ...location, pathname: "/courses" })} />
<Link to={location => `${location.pathname}?sort=name`} />
```

跳转默认采用`pushState`方法，可以添加属性`replace`变成`replaceState`

```js
<Link to="/courses" replace />
```

---

`<NavLink>`可以看作是一个可以动态改变样式的 link，对于正在触发的 link，可以通过设置类名、样式等形式改变该 link 的样式。
主要方法有四种：

```js
// 动态设置className
<NavLink
  to="/faq"
  className={isActive =>
    "nav-link" + (!isActive ? " unselected" : "")
  }
>
  FAQs
</NavLink>

// 使用activeClassName属性
<NavLink to="/faq" activeClassName="selected">
  FAQs
</NavLink>

// 改变style也可以，和上面类似
<NavLink
  to="/faq"
  style={isActive => ({
    color: isActive ? "green" : "blue"
  })}
>
  FAQs
</NavLink>

// activeStyle
<NavLink
  to="/faq"
  activeStyle={{
    fontWeight: "bold",
    color: "red"
  }}
>
  FAQs
</NavLink>
```
