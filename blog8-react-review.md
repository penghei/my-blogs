---
title: react“高级指引”学习
date: 2022-03-29 19:20:32
tags: 日常学习
categories: React
cover: /img/react.png
---

# 组件懒加载

`React.lazy`可以包裹一个动态导入的组件，实现懒加载

```js
const lazyComponents = React.lazy(() => import("./component"));
```

`lazy`的参数必须是一个返回`Promise`对象的函数， 该`Promise`需要 `resolve` 一个 `export default` 的 React 组件。（必须是默认导出的组件）

然后使用`<Suspense>`组件包裹这个要加载的组件，参数可以是任意等待加载中的组件

```js
import React, { Suspense } from "react";

const OtherComponent = React.lazy(() => import("./OtherComponent"));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

通常用于路由页面的加载，可以把`<Suspense>`组件包裹住所有的`<Route>`组件

```js
const App = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Switch>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Switch>
    </Suspense>
  </Router>
);
```

# Context

> Context 设计目的是为了共享那些对于一个组件树而言是“全局”的数据

一般通过`React.createContext()`创建，参数是该 context 的初始值。

```js
export const MyContext = React.createContext(defaultValue);
```

然后在创建的组件中用 `provider` 包裹。
当 `Provider` 的 value 值发生变化时，使用这个`Provider`的组件都会被重新渲染

```js
<MyContext.Provider value={someValue}>
  <Component />
</MyContext.Provider>
```

在需要接收的组件使用`useContext()`,括号是`React.createContext()`创建的 context 对象。
组件会从离自己**最近的**匹配的 `Provider` 中读取到当前的 context 值

```js
import MyContext from "./App";

const value = useContext(MyContext);
//value
```

在类组件中只需要定义一个静态属性，值为创建的 context 值，就可以在组件内部用 this.context 访问

```js
class MyClass extends React.Component {
  static contextType = MyContext;
  render() {
    let value = this.context;
  }
}
// or
MyClass.contextType = MyContext;
```

注意在分组件使用的时候,把创建单独写一个文件,比如这样
通常还会把修改该 context 的方法传递下去

```js
import { createContext } from "react";
export default createContext();
//...
//其他组件中
import MyContext from "./context";
<MyContext.Provider value={someValue}>
  <Component />
</MyContext.Provider>;
//使用的组件
import MyContext from "./context";
const value = useContext(MyContext);
```

---

Context 还有一些其他的常用 api，详细使用可参考官方文档：https://zh-hans.reactjs.org/docs/context.html

1. `Class.contextType`：只能用于类组件，相当于给类组件添加了一个属性，在组件内部可以用`this.context`访问到 context 的 value 值
2. `Context.Consumer`：已经被 hooks 代替，用于在函数组件中访问 context

# 错误边界

> 错误边界是一种 React 组件，这种组件可以捕获发生在其子组件树任何位置的 JavaScript 错误，并打印这些错误，同时展示降级 UI，而并不会渲染那些发生崩溃的子组件树。
> 错误边界无法捕获以下场景中产生的错误：
>
> - 事件处理
> - 异步代码
> - 服务端渲染
> - 它自身抛出来的错误

实现如下：

```js
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 更新 state 使下一次渲染能够显示降级后的 UI
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // 你同样可以将错误日志上报给服务器
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // 你可以自定义降级后的 UI 并渲染
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}
```

> 自 React 16 起，任何未被错误边界捕获的错误将会导致整个 React 组件树被卸载。

根据 React，出错的组件未捕获会导致整个页面瘫痪；因此可以让错误边界包裹单独的可能会出错的组件，或者包裹在整个路由组件外面；当某个组件或者某部分出错时，依然可以使用其他组件。

---

错误边界的实现原理，是两个关键的生命周期：`getDerivedStateFromError`和`componentDidCatch`。

- `getDerivedStateFromError`：会在后代组件抛出错误后被调用。它将抛出的错误作为参数，并返回一个值以更新 state。这个生命周期和`getDerivedStateFromProps`不一样，它是在**渲染**阶段调用的，因此不允许有副作用
- `componentDidCatch`：在后代组件抛出错误后被调用，接受两个参数：
  - error：抛出的错误。
  - info：带有 componentStack key 的对象，其中包含有关组件引发错误的栈信息。（类似控制台输出错误的效果）
    这个生命周期在提交阶段调用

# Refs 转发

React 组件不能直接通过 ref 获取其实例；理论上说，ref 只能应用于明确定义的 dom 元素（虽然实际上是虚拟 dom）

但是通过 Refs 转发，可以在父组件获取子组件中某个 dom 元素的实例。
方法是用`React.forwardRef`包裹子组件（只能是函数组件），然后参数中添加一个 ref 参数：

```js
// 子组件
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// 父组件
const childBtnRef = useRef(null); // 传递后能拿到子组件button的ref实例
<FancyButton ref={childBtnRef}>Click me!</FancyButton>;
```

useImperativeHandle hook可以指定子组件要暴露给父组件什么值，需要配合forwardRef

useImperativeHandle的第二个参数是一个函数，这个函数可以返回一个对象，用于表示父元素通过ref获取到的值。比如下面这个例子中，返回的这个对象就成为了父元素中的fancyRef值，包含一个focus方法，可以调用子组件FancyInput中的代码。

```js
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);

// 父组件
const fancyRef = useRef(null)
<FancyInput ref={fancyRef} />

// fancyRef = {
//   focus:...
// }
```

通过这种方式，可以向父组件提供指定的某些数据，比如某个状态、某个方法等。相对于callback props更简单和灵活


# Fragment

关于这个只有一个特别的点，就是`Fragment`可以是显式的，可以向上面添加 key，用于包裹列表渲染的组件

```js
function Glossary(props) {
  return (
    <dl>
      {props.items.map((item) => (
        <React.Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </React.Fragment>
      ))}
    </dl>
  );
}
```

# 高阶组件

## 概述

> 高阶组件是参数为组件，返回值为新组件的函数。

高阶组件在官方文档上表现得很复杂，其实可以理解成类似高阶函数一样的东西，即接收一个组件，经过包装后返回一个组件，中间可能会插入一些参数。

最基本的高阶组件就是这个样子：

```js
function HOC(SomeComponents) {
  return class extends React.Component {
    // 这里的props来自于新组件传入的props，即下方DisplayHOC传入的props
    constructor(props) {
      super(props);
    }
    render() {
      return <SomeComponents {...this.props} />;
    }
  };
}

const DisplayHOC = HOC(Display);

// 使用
<DisplayHOC value="hello world" />;
```

`Display`组件经过高阶组件的包装成为`DisplayHOC`，新组件和原组件使用上没有太多区别；

高阶组件也常用于解决组件之间的复用问题。比如两个组件十分相似，只有部分函数或者数据源不同，但是大部分是相同的。这时就可以通过高阶函数创建这两个组件，通过给函数传入不同的参数从而形成不同的组件。这个例子可以参考官方文档的高阶组件章节

然后可以给高阶组件传入一些额外的设置，比如一个修改 display 值的函数：

```js
function Hoc(SomeComponents, handleChange) {
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.state = {
        value: "",
      };
    }
    componentDidMount() {
      this.setState({
        value: handleChange(this.props.value),
      });
    }
    render() {
      return <SomeComponents value={this.state.value} {...this.props} />;
    }
  };
}

const Display = ({ value }) => <h1>{value}</h1>;

const DisplayHOC = Hoc(
  Display,
  (value) => `${value}${parseInt(Math.random() * 1000)}`
);
```

在这个例子中，向高阶组件额外传了一个函数；这个函数很简单，只是在字符串末尾加一个数字。
然后在高阶组件返回的组件中设置状态，简单粗暴的在挂载时就改变 state，可以看到最后显示的值也改变了。

因此可以考虑传入不同的函数，来获取类似但关键部分不一样的组件：

```js
//上面的不改变

const DisplayHOCHi = Hoc(Display,(value)=>`hi, ${value}!`);
const DisplayHOCHello = Hoc(Display,(value)=>`hello, ${value}!`);

// 使用
<DisplayHOCHello value='Xiao Ming'/>
<DisplayHOCHi value='Xiao Ming'/>

```

其实本质上和高阶函数很类似，只是返回的是一个类组件。

---

既然可以返回类组件，同理也可以返回函数组件，方法更简单：

```js
function HocFC(SomeComponents, methods) {
  return function ({ value, ...otherProps }) {
    const [displayValue, setValue] = useState(value);

    useEffect(() => {
      setValue(methods(value));
    }, []);

    // 注意其他的props要透传，高阶组件内部只处理个别的props
    return <SomeComponents value={displayValue} {...otherProps} />;
  };
}
```

> React 很多库的实现都依赖高阶组件。比如`withRouter()`、`connect()()`等。

---

上一节讲到的 refs 转发也适用于高阶组件和其内部返回的组件之间传递 refs。详见https://zh-hans.reactjs.org/docs/forwarding-refs.html

## 应用

高阶组件的主要用处可以由下列出：

1. 强化 props ，可以通过 HOC 向原始组件添加一些 props。常见的就是 withRouter 向组件添加了路由的 history 等对象

下面是 withRouter 的简单实现：

```js
function withRouter(Component) {
  return (props) => {
    // remainingProps 是组件原始的props
    const { wrappedComponentRef, ...remainingProps } = props;
    return (
      // react router中的history等对象就是通过context传递的
      <RouterContext.Consumer>
        {(context) => {
          return (
            <Component
              {...remainingProps} // 组件原始的props
              {...context} // 存在路由对象的上下文，history  location 等
              ref={wrappedComponentRef}
            />
          );
        }}
      </RouterContext.Consumer>
    );
  };
}
```

2. 渲染劫持，可以利用 HOC ，动态挂载原始组件，还可以先获取原始组件的渲染树，进行可控性修改。在 HOC 中可以通过`super.render()` 得到 render 之后的内容（即 React 创建的 element 对象），再进行一些修改操作。

这个用处不是很大。劫持渲染其实可以通过 props 和子元素配合实现。

```js
const HOC = (WrapComponent) =>
  class Index extends WrapComponent {
    render() {
      if (this.props.visible) {
        return super.render(); // 拦截到的渲染结果
      } else {
        return <div>暂无数据</div>;
      }
    }
  };
```

3. 动态加载。可以配合 import 等 api ，实现动态加载组件，加入 loading 效果。

下面是一个例子。

```js
export default function AsyncRouter(loadRouter) {
  return class Content extends React.Component {
    constructor(props) {
      super(props);
      this.state = { Component: null }; // 这里的state是要动态渲染的组件
    }

    componentDidMount() {
      if (this.state.Component) return;
      loadRouter() // 执行动态import的函数
        .then((module) => module.default) // 动态加载 component 组件
        .then((Component) => this.setState({ Component }));
    }
    render() {
      const { Component } = this.state;
      const { Loading } = this.props.loading;
      return Component ? <Component {...this.props} /> : <Loading />; // 渲染state中的Component
    }
  };
}
// 类似React.lazy
const LazyIndex = AsyncRouter(() => import("../pages/index"));

// 使用
<LazyIndex loading={<Loading />} />;
```

4. 可以对原始组件做一些事件监听，错误监控等。

这类高阶组件就是像添加 props 一样在组件上添加一些监听。但是只能基于整个组件监听，实际上是安插在组件外部的一个 div 或其他元素上的。
比如下面这个例子，可以实现点击内部组件时触发事件。

```js
function ClickHoc(Component) {
  return function Wrap(props) {
    const dom = useRef(null);
    useEffect(() => {
      const handerClick = () => console.log("发生点击事件");
      dom.current.addEventListener("click", handerClick);
      return () => dom.current.removeEventListener("click", handerClick);
    }, []);
    return (
      // 事件实际上是安插在组件外部的一个div上的。
      // 如果需要之间安插在组件上，可能需要forwardRef配合
      <div ref={dom}>
        <Component {...props} />
      </div>
    );
  };
}
```

# JSX

> JSX 是 `React.createElement(component, props, ...children)` 函数的语法糖

## jsx 中使用点语法

基本的不再赘述，有一个小技巧时在 JSX 类型中使用点语法，即组件是`<Components.xxx>`的形式
方法非常简单：

```js
const Layouts = {
  Header: (props) => {
    return <header>{props.children}</header>;
  },
  Container: (props) => {
    return <main>{props.children}</main>;
  },
  Footer: (props) => {
    return <footer>{props.children}</footer>;
  },
};

export default Layouts;
```

原理就是，jsx 的标签名实际上就是一个函数或者类；因此可以用对象包装；

```js
const MyButton = ({ value }) => <button>{value}</button>;

React.createElement(
  MyButton, // 就是上面那个函数
  { color: "blue", shadowSize: 2 },
  "Click Me"
);
```

## jsx 作为一个变量

jsx 标签名甚至可以是动态的，即可以是一个（以大写字母开头的）变量：

```js
import React from "react";
import { PhotoStory, VideoStory } from "./stories";

const components = {
  photo: PhotoStory,
  video: VideoStory,
};

function Story(props) {
  // JSX 类型可以是大写字母开头的变量。
  const SpecificStory = components[props.storyType];
  return <SpecificStory story={props.story} />;
}
```

## jsx 中的布尔值

> false, null, undefined, true 是合法的子元素。但它们并不会被渲染

也就是说下面这几种情况渲染相同，即都不会渲染 jsx 括号中的值

```js
<div />

<div></div>

<div>{false}</div>

<div>{null}</div>

<div>{undefined}</div>

<div>{true}</div>
```

jsx 中使用`&&`、`||`和`??`时，尽可能保证之前的表达式总是布尔值，比如`0`虽然是 falsy，但是仍会渲染`0`。

> 经测试，导致这种情况的 falsy 有`0`、`+0`、`-0`（这三个都会渲染 0）、`NaN`

```js
<div>
  {props.messages.length &&
    <MessageList messages={props.messages} />
  }
</div>

// 应该是
<div>
  {props.messages.length > 0 &&
    <MessageList messages={props.messages} />
  }
</div>
```

# 跳过渲染

> 可以通过覆盖生命周期方法 `shouldComponentUpdate` 来进行提速。该方法会在重新渲染前被触发。其默认实现总是返回 true，让 React 执行更新；可以在 `shouldComponentUpdate` 中返回 false 来跳过整个渲染过程。其包括该组件的 `render` 调用以及之后的操作。
> 也可以继承 `React.PureComponent` 以代替手写 `shouldComponentUpdate()`。它用当前与之前 props 和 state 的**浅比较**覆写了 `shouldComponentUpdate()` 的实现。

如果跳过当前组件的重新渲染，也会跳过其后所有子组件的重新渲染，除非子组件内部状态被单独改变。
![](https://pic.imgdb.cn/item/6243343927f86abb2a529023.jpg)

在 hooks 中，可以通过`React.memo`代替`React.PureComponent`；
但它**只比较 props**，只有 props 和之前的浅比较结果不同才会触发重渲染

```js
const Button = React.memo((props) => {
  // ...
});
```

这个函数还可以接受第二个参数，是一个布尔值或者返回布尔值的函数（这个函数会有当前 props 和下一个 props 两个参数）；如果为 true 就一定会跳过这次渲染。

## 应用场景

子组件持有父组件的 props，但是并没有用到所有的 props 上的属性；如果只用了一部分，改变其他属性时也会导致该组件渲染。因此可以指定第二个参数比较当前 props 和下一次渲染的 props，只有某些属性改变才重新渲染。

```js
import React, { memo } from "react";

const isEqual = (prevProps, nextProps) => {
  if (prevProps.number !== nextProps.number) {
    return false;
  }
  return true;
};

export default memo((props = {}) => {
  console.log(`--- memo re-render ---`);
  return (
    <div>
      {/* <p>step is : {props.step}</p> */}
      {/* <p>count is : {props.count}</p> */}
      <p>number is : {props.number}</p>
    </div>
  );
}, isEqual);
```

## useMemo

某些场景下，我们只是希望 component 的部分不要进行 re-render，而不是整个 component 不要 re-render，也就是要实现 局部 Pure 功能。

`useMemo()` 基本用法如下：

```js
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

返回的是一个 memoized 值，实际上就是原值的一个包装，并没有太多改动。因此可以返回一个组件，然后正常渲染：
只有当依赖项改变才会重新计算 memoized，而 memoized 值不变的情况下，不会重新触发渲染逻辑。因此实际上就是取决于依赖项数组。

```js
export default ({ a, b }) => {
  const Header1 = useMemo(() => <h2>hello,{a}</h2>, [a]);
  const Header2 = useMemo(() => <h2>hello,{b}</h2>, [b]);
  return (
    <div>
      {Header1}
      {Header2}
    </div>
  );
};
```

注意`useMemo`不能用于副作用，`useMemo()` 是在 render 期间执行的，不能用于异步任务等有副作用的任务。

# Portals

> Portal 提供了一种将子节点渲染到存在于父组件以外的 DOM 节点的优秀的方案。
>
> ```js
> ReactDOM.createPortal(child, container);
> ```
>
> 第一个参数（child）是任何可渲染的 React 子元素，例如一个元素，字符串或 fragment。第二个参数（container）是一个 DOM 元素。

具体使用：

```js
import reactDom from "react-dom";

export default (props) => {
  return reactDom.createPortal(
    <div style={{ position: "fixed", top: "10px", left: "50%", right: "50%" }}>
      {props.children}
    </div>,
    document.getElementsByTagName("body")[0]
  );
};
```

利用这个方法可以方便的实现对话框、悬浮卡、提示框等内容。

> 另外，即使该元素已经被安插在指定的 dom 节点上，仍然可以通过该组件实际放置位置的父组件获取组件上事件的冒泡。

# Render Props

> 具有 render prop 的组件接受一个返回 React 元素的函数，并在组件内部通过调用此函数来实现自己的渲染逻辑。
>
> ```js
> <DataProvider render={(data) => <h1>Hello {data.target}</h1>} />
> ```

即，一个组件接收一个特殊的 props，这个 props 会被渲染到组件内部。
比如这样：

```js
<Mouse render={(mouse) => <Cat mouse={mouse} />} />
```

实现起来很简单，其实本质上就是向父组件传递了一个返回子组件的函数；父组件调用它，并传入自己的 state，就可以实现动态影响子组件。
Cat 组件就是一个很普通的组件

```js
const Cat = ({ mouse }) => {
  const catStyle = {
    position: "absolute",
    left: mouse.x,
    top: mouse.y,
  };
  return <div style={catStyle}>{mouse.name}</div>;
};
```

它的父组件 Mouse，在 jsx 内部渲染 render props：

```js
<Mouse render={(mouse) => <Cat mouse={mouse} />} />

const Mouse = ({ render }) => {
    const [mouse, setMouse] = useState(...)
  //... 亿些东西
  return (
    <div>
      {render(mouse)}
      相当于
      <Cat mouse={mouse} />
    </div>
  );
};

```

可以看到，子组件 Cat 的 props，实际上是父组件动态调用`render`方法传入的参数，即自己的 state 或任意值

这样，相比于直接在 Mouse 组件中写死 Cat 组件的位置，可以更灵活改变 Mouse 内部渲染的东西。

# 函数组件和类组件

> 函数组件和类组件本质的区别是什么呢？
> 对于类组件来说，底层只需要实例化一次，实例中保存了组件的 state 等状态。对于每一次更新只需要调用 render 方法以及对应的生命周期就可以了。但是在函数组件中，每一次更新都是一次新的函数执行，一次函数组件的更新，里面的变量会重新声明。

函数组件会直接执行函数，然后拿到函数返回的`React.element`对象

```js
function renderWithHooks(
  current, // 当前函数组件对应的 `fiber`， 初始化
  workInProgress, // 当前正在工作的 fiber 对象
  Component, // 我们函数组件
  props, // 函数组件第一个参数 props
  secondArg, // 函数组件其他参数
  nextRenderExpirationTime //下次渲染过期时间
) {
  /* 执行我们的函数组件，得到 return 返回的 React.element对象 */
  let children = Component(props, secondArg);
}
```

类组件会实例化类，得到类的实例

```js
function constructClassInstance(
  workInProgress, // 当前正在工作的 fiber 对象
  ctor, // 我们的类组件
  props // props
) {
  /* 实例化组件，得到组件实例 instance */
  const instance = new ctor(props, context);
}
```

因此在 hooks 诞生之前，函数组件就像普通函数一样，只能执行，但是不能保存任何数据；但是类组件由于是实例化的，因此可以保存 state 等组件相关的状态。

# Hooks

## useState

- 如果初始 state 需要通过复杂计算获得，则可以传入一个函数，在函数中计算并返回初始的 state，此函数只**在初始渲染时被调用**：

```js
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);
  return initialState;
});
```

- 调用 hook 的 setState 如果传入当前 state，就会跳过子组件的渲染及 effect 的执行。

```js
const [state, setState] = useState(0);

//...
if (state > 10) {
  setState(state);
}
```

- 如果 state 是一个包含多个参数的对象，更好的方式是使用`useReducer`

### 和 setState 的区别

相同：

- 底层原理近似，事件驱动情况下都有批量更新规则。

不同：

- 在不是 `pureComponent` 组件模式下， `setState` 不会浅比较两次 state 的值，只要调用 `setState`，在没有其他优化手段的前提下，就会执行更新。但是 `useState` 中的 `dispatchAction` 会默认比较两次 state 是否相同，然后决定是否更新组件。
- `setState` 有专门监听 state 变化的回调函数 `callback`，可以获取最新 state；但是在函数组件中，只能通过 `useEffect` 来执行 state 变化引起的副作用。
- `setState` 在底层处理逻辑上主要是和老 state 进行合并处理，而 `useState` 更倾向于重新赋值。

## useEffect

> 在函数组件主体内（这里指在 React 渲染阶段）改变 DOM、添加订阅、设置定时器、记录日志以及执行其他包含副作用的操作都是不被允许的，因为这可能会产生莫名其妙的 bug 并破坏 UI 的一致性。

对于函数组件，函数内部的部分会在 render 阶段被执行，每次 render 都会从头到尾执行一次组件函数；因此如果在函数内部直接定义的变量，每次都会初始化：

```js
const [num, setNum] = useState(0);
let times = 0;
console.log(times); // 始终是0，因为每次更新状态，函数执行一次，都会let times = 0;
return (
  <div>
    <button
      onClick={() => {
        setNum((num) => ++num);
        times += 1;
      }}
    >
      {num}
    </button>
  </div>
);
```

如果不在`useEffect`而是在函数顶层直接执行副作用，也是这样的效果。
因此函数组件内部应该尽可能的是纯函数，把所有的副作用都放在`useEffect`中。

> 与 `componentDidMount`、`componentDidUpdate` 不同的是，传给 `useEffect` 的函数会在浏览器完成布局与绘制**之后**，在一个延迟事件中被调用。React 会像`setTimeout`回调函数一样，放入任务队列，等到主线程任务完成，DOM 更新，js 执行完成，视图绘制完毕，才执行。

如果想要使执行时间在布局绘制之前，可以使用`useLayoutEffect`；这种场景常见于手动修改 dom、样式等。
一句话概括如何选择 `useEffect` 和 `useLayoutEffect` ：修改 DOM、改变布局就用 `useLayoutEffect` ，其他情况就用 `useEffect`。

> `React.useEffect` 回调函数 和 `componentDidMount` / `componentDidUpdate` 执行时机有什么区别 ？
> useEffect 对 React 执行栈来看是异步执行的，而 `componentDidMount` / `componentDidUpdate` 是同步执行的，`useEffect`代码不会阻塞浏览器绘制。在时机上 ，`componentDidMount` / `componentDidUpdate` 和 `useLayoutEffect` 更类似。

### 关于依赖列表

> 只有当函数（以及它所调用的函数）不引用 `props`、`state` 以及由它们衍生而来的值时，你才能放心地把它们从依赖列表中省略。

（虽然官方文档一直推荐把所有依赖都放到数组，但是如果只是想在挂载时执行一次，确实可以放一个空数组）

## useReducer

```js
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

第一个参数是一个 `reducer` 函数，第二个是初始值；返回一个当前状态和控制状态的 `dispatch`
返回值和`useState`基本类似，同样是可以直接使用 `state`，并且用 `dispatch` 做更改。只是 `dispath` 的更改要符合 `reducer` 的定义。
`useReducer`适用于有很多属性的对象 `state`，比如这样：

1. 首先定义一个 `reducer`，和 `redux` 中的 `reducer` 类似

注意参数的顺序，第一个参数是将来返回给解构数组中的第一个，即`state`；后一个参数会接收`dispatch`传入

```js
function reducer(state, { type, value }) {
  switch (type) {
    case "changeName":
      return { ...state, userName: value };
    case "changeAge":
      return { ...state, userAge: value };
    case "changeId":
      return { ...state, userId: value };
    default:
      return state;
  }
}
```

2. 然后设置初始值和使用，其中返回值的第一个实际上就是类似 `state` 的数据，初始为第二个参数初始值

修改时调用`dispatch`方法，传递一个`reducer`需要的参数。**这个参数会被传入到`reducer`的第二个参数中**，返回值会交给`userInfo`（state）

```js
const [userInfo, dispatchUserInfo] = useReducer(reducer, {
  userName: "zzx",
  userAge: 19,
  userId: "001",
});

const handleChange = (e) => {
  const tar = e.target;
  dispatchUserInfo({ type: tar.innerText, value: "aaa" });
};

// 正常使用state
{
  Object.entries(userInfo).map((user, index) => {
    return (
      <React.Fragment key={index}>
        <h3>{user[0]}</h3>
        <span>{user[1]}</span>
      </React.Fragment>
    );
  });
}
```

3. `useReducer`的第三个参数可以用来初始化初始值，相当于如果传入这个参数，则初始值是这个函数返回的值

```js
function init(initialCount) {
  return {count: 0};
}

function Counter({initialCount}) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);

  return (
    ...
  );
}

```

## useCallback 和 useMemo

`useCallback`，参数是一个函数，或者一个返回函数的函数；`useCallback`返回一个函数。

```js
// 引用一个函数，适用于props传入的
const multiply = (a, b) => a * b;
const memoizedMultiply = useCallback(mutiply, [a, b]);

// 或者直接定义在内部
const memoizedMultiply = useCallback(() => {
  return a * b;
}, [a, b]);
```

`useMemo`，参数是一个返回变量的函数，`useMemo`返回一个变量。

```js
const memoizedValue = useMemo(() => doSomethingAndReturn(a, b), [a, b]);
```

`useCallback`和`useMemo`类似，都有如下的特点：

1. 返回的值是原值的`memoized 版本`，实际上功能并没有任何变化，依然可以正常调用或使用。就像防抖和节流函数只是返回函数的防抖和节流版本，函数本身的功能没有改变。
2. 前者传入一个函数，返回一个函数，并让这个函数依据依赖数组而改变，依赖不变则函数不变；后者其实相当于是针对一个变量，依赖不变则参数不变。
3. 这两者的主要应用场景都是，父组件传入 变量/函数，但是不想让子组件中接收的 变量/函数 也随`props`中无关变量变化而重新计算，造成不必要的开销

比如：

`useMemo`返回的变量不变很好理解，参数是返回变量的函数，`useMemo`返回一个变量。

```js
export default ({ a, b }) => {
  const Header1 = useMemo(() => {
    console.log("a");
    return <h2>hello,{a}</h2>;
  }, [a]);

  const Header2 = useMemo(() => {
    console.log("b");
    return <h2>hello,{b}</h2>;
  }, [b]);

  return (
    <div>
      {Header1}
      {Header2}
    </div>
  );
};
```

a 和 b 都依赖于 props 的传入；如果只改变 a，`Header2`的值并不会改变，因为它依赖于 b，但 b 本身没变。

`useCallback`差不多，参数是一个函数，或者一个返回函数的函数；`useCallback`返回一个函数。

```js
export default ({ a, b, callback }) => {
  const callbackMemo = useCallback(callback, [a, b]); // 只有a和b改变时函数才会变

  return <div>{callbackMemo(a, b)}</div>;
};
```

## useRef

除了可以用于获取 dom 元素实例，`useRef`还可以用于缓存值；`useRef`会在每次渲染时返回同一个 ref 对象，因此相当于保存了一个恒定的值，不会受到更新影响。
当 ref 对象内容发生变化时，useRef 并不会通知，也不会引起更新；
但是官方文档提供了一种回调 Ref 的方法可以使得 ref 在组件**挂载**和**卸载**时触发一个回调函数：

即，把原先一个对象 ref 变成了函数 ref，并且是由函数返回而不是用`useRef`创建，当 ref 变化的时候会调用这个回调。

```js
function MeasureExample() {
  const measuredRef = useCallback((node) => {
    console.log(node.innerText);
  }, []);

  return (
    <>
      <h1 ref={measuredRef}>Hello, world</h1>
    </>
  );
}
```

## hooks 替代生命周期

### componentDidMount/componentWillUnmount

```js
useEffect(() => {
  return () => {
    console.log("will unmount");
  };
}, []);
```

### componentDidUpdate

采用自定义 hooks 实现

```js
import { useEffect, useRef } from "react";

export default function useUpdateEffect(callback, deps) {
  const mounted = useRef();
  useEffect(() => {
    if (!mounted.current) {
      mounted.current = true;
    } else {
      callback();
    }
  }, [deps]);
}
```

### getDerivedStateFromProps

```js
function Child(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    if (props.count !== count) {
      setCount(props.count);
    }
  }, [props.count]);
}
```

### shouldComponentUpdate

```js
export default React.memo(() => {
  return <div>...</div>;
});
```

# React 八股

## React 概述

React 是一个用于构建用户界面的 JavaScript 库。

React 算不上一个框架，因为 React 主要专注于视图和数据的交互，并不包含数据访问和路由等其他功能。结合 React 的其他各种插件和库，才可以算得上是一个框架。

React 可以作为 MVVM 中第二个 V，也就是 View，但是并不是 MVVM 框架。MVVM 一个最显著的特征：双向绑定。React 没有这个，它是单向数据绑定的。React 是一个单向数据流的库，状态驱动视图。react 整体是函数式的思想，把组件设计成纯组件，状态和逻辑通过参数传入，所以在 react 中，是单向数据流，推崇结合 immutable 来实现数据不可变。

> 关于 MVVM、MVC，可以参考：https://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html
> 简单来说：
>
> - MVC：即视图（View）用户界面；控制器（Controller）业务逻辑；模型（Model）数据保存 三部分组成。
> - MVVM：把 Controller 改为 ViewModel，即 Model/View/ViewModel 三部分。Model 用纯 JavaScript 对象表示，View 负责显示，ViewModel 负责把 Model 的数据同步到 View 显示出来，还负责把 View 的修改同步回 Model，即双向绑定。

## React 和其他框架的区别（Vue）

React 官网介绍 React 是一个用于构建用户界面的 JavaScript 库。React 推荐 JSX + inline style, 也就是把 HTML 和 CSS 全都写进 JavaScript 了,即 ”all in js“，HTML 和 css 都可以放到 js 中。React 主张函数编程，推荐使用纯函数，数据不可变，单向数据流，但是可以手动编写 onChange 等事件处理函数实现双向数据流。结合 JSX 轻松实现渲染模板

Vue 官网介绍 Vue 是一套用于构建用户界面的渐进式框架,与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 保留了 html、css、js 分离的写法，使得现有的前端开发者在开发的时候能保持原有的习惯，更接近常用的 web 开发方式，模板就是普通的 html，数据绑定使用 mustache 风格，样式直接使用 css。Vue 认为数据是可变的，使用 v-model 实现双向数据流。

## 函数组件和类组件

类组件是使用 ES6 的 class 来定义的组件。 函数组件是接收一个单一的 props 对象并返回一个 React 元素。

关于 React 的两套 API（类（class）API 和基于函数的钩子（hooks） API）。官方推荐使用钩子（函数），而不是类。因为钩子更简洁，代码量少，用起来比较"轻"，而类比较"重"。而且，钩子是函数，更符合 React 函数式的本质。

函数一般来说，只应该做一件事，就是返回一个值。 如果你有多个操作，每个操作应该写成一个单独的函数。而且，数据的状态应该与操作方法分离。根据函数这种理念，React 的函数组件只应该做一件事情：返回组件的 HTML 代码，而没有其他的功能。函数的返回结果只依赖于它的参数。不改变函数体外部数据、函数执行过程里面没有副作用。

---

类（class）是数据和逻辑的封装。 也就是说，组件的状态和操作方法是封装在一起的。如果选择了类的写法，就应该把相关的数据和操作，都写在同一个 class 里面。

类组件的缺点 :

- 大型组件很难拆分和重构，也很难测试。
- 业务逻辑分散在组件的各个方法之中，导致重复逻辑或关联逻辑。
- 组件类引入了复杂的编程模式，比如 render props 和高阶组件。
- 难以理解的 class，理解 JavaScript 中 this 的工作方式。

区别：

1. 状态的有无
   hooks 出现之前，函数组件没有实例，没有生命周期，没有 state，没有 this，所以我们称函数组件为无状态组件。 hooks 出现之前，react 中的函数组件通常只考虑负责 UI 的渲染，没有自身的状态没有业务逻辑代码，是一个纯函数。它的输出只由参数 props 决定，不受其他任何因素影响。
2. 调用方式的不同
   函数组件重新渲染，将重新调用组件方法返回新的 react 元素。类组件重新渲染将 new 一个新的组件实例，然后调用 render 类方法返回 react 元素，这也说明为什么类组件中 this 是可变的。
3. 因为调用方式不同，在函数组件使用中会出现问题
   在操作中改变状态值，类组件可以获取最新的状态值，而函数组件则会按照顺序返回状态值

## 组件通信方式

- 父组件向子组件通信：props
- 子组件向父组件通信：回调 props
- 跨级组件通信：context
- 非嵌套关系的组件通信：状态管理库

## key 的作用

简单来说：

0. 减少不必要的遍历，有了 key 就可以简单的匹配 key 来复用或发现更改。
1. 让 React 在 diff 算法时，清楚的知道是哪些元素发生了变化、发生了怎样的变化（删除、增加、移动）
2. diff 算法中 key 是必须的，要通过 key 判断元素应该向哪里移动。

# React 优化

## 什么时候要注意优化

**没有性能问题不要刻意优化**
在正常情况下，无须过分在乎 React 没有必要的渲染，要理解执行 render 不等于真正的浏览器渲染视图，render 阶段执行是在 js 当中，js 中运行代码远快于浏览器的 Rendering 和 Painting 的，更何况 React 还提供了 diff 算法等手段，去复用真实 DOM 。
但是对于以下情况，值得开发者注意，需要采用渲染节流：

- 第一种情况数据可视化的模块组件（展示了大量的数据），这种情况比较小心因为一次更新，可能伴随大量的 diff ，数据量越大也就越浪费性能，所以对于数据展示模块组件，有必要采取 memo ， shouldComponentUpdate 等方案控制自身组件渲染。

- 第二种情况含有大量表单的页面，React 一般会采用受控组件的模式去管理表单数据层，表单数据层完全托管于 props 或是 state ，而用户操作表单往往是频繁的，需要频繁改变数据层，所以很有可能让整个页面组件高频率 render 。

- 第三种情况就是越是靠近 app root 根组件越值得注意，根组件渲染会波及到整个组件树重新 render ，子组件 render ，一是浪费性能，二是可能执行 useEffect ，componentWillReceiveProps 等钩子，造成意想不到的情况发生。


## React.memo/PureComponent/shouldComponentUpdate 防止子组件每次都被更新

这三个的目标和效果是类似的，都会保证在父组件更新时不更新子组件以及其后的组件，但细节上这三个方法还是有区别的。

### useMemo 保存子组件

方法就是使用 useMemo 或 useRef 包裹一下子组件，这样父组件的更新就不会导致创建新的子组件 element 对象，也就不会导致子组件的更新了。

```js
function Index() {
  const [numberA, setNumberA] = React.useState(0);
  const [numberB, setNumberB] = React.useState(0);
  return (
    <div>
      {useMemo(
        () => (
          <Children number={numberA} />
        ),
        [numberA]
      )}
      <button onClick={() => setNumberA(numberA + 1)}>改变numberA</button>
      <button onClick={() => setNumberB(numberB + 1)}>改变numberB</button>
    </div>
  );
}
```

这个实现原理其实没有那么复杂，根本原因是useMemo依赖于numberA，如果numberA不改变，Children就不会重新渲染，而是作为一个变量保存在useMemo该保存的地方，始终保持成为初始化的组件。

### PureComponent

PureComponent 是类组件的渲染优化方式，让类组件继承自 PureComponent，在组件更新时会自动比较本次的 props 和上次的，**如果相同还会比较以及本次的 state 和上次的**，两个都相同就不会触发更新。
比较的过程是浅比较，这一点和 React.memo 一样；

PureComponent 的实现原理是这样：

```js
function PureComponent(props, context, updater) {
  this.props = props;
  this.context = context;
  this.refs = emptyObject;
  this.updater = updater || ReactNoopUpdateQueue;
}

const pureComponentPrototype = (PureComponent.prototype = new ComponentDummy());
pureComponentPrototype.constructor = PureComponent;
assign(pureComponentPrototype, Component.prototype); // 复制React.Component上的方法
pureComponentPrototype.isPureReactComponent = true;
```

PureComponent 本质上就是 React.Component 的一个子类，并添加了一个 isPureReactComponent 属性到原型对象上。在 updateClassInstance 方法中（这个方法是 BeginWork 时用于执行类组件更新时的生命周期的）会检查这个属性，配合 shouldComponentUpdate 来一起确定是否需要更新组件：

```js
function checkShouldComponentUpdate() {
  if (typeof instance.shouldComponentUpdate === "function") {
    return instance.shouldComponentUpdate(
      newProps,
      newState,
      nextContext
    ); /* shouldComponentUpdate 逻辑 */
  }
  if (ctor.prototype && ctor.prototype.isPureReactComponent) {
    return (
      !shallowEqual(oldProps, newProps) || !shallowEqual(oldState, newState)
    );
  }
}
```

### React.memo

父组件的每次状态更新，都会导致子组件重新渲染，即使传入子组件的状态没有变化，为了减少重复渲染，我们可以使用 React.memo 来缓存组件，这样只有当传入组件的状态值发生变化时才会重新渲染。如果传入相同的值，则返回缓存的组件。

```js
export default React.memo((props) => {
  return <div>{props.value}</div>;
});
```

但是还有有一个问题，React.memo 只会对 props 进行浅比较，因此它保留了第二个参数可以自定义比较函数。这个函数会得到两个参数，分别是之前的 props 和当前的 props，使用起来和 shouldComponentUpdate 几乎一样

```js
React.memo(MyComponent, (prevProps, currProps) => {
  return deepCompare(prevProps, currProps);
});
```

memo 函数实际很简单，如下：

```ts
export function memo<Props>(
  type: React$ElementType,
  compare?: (oldProps: Props, newProps: Props) => boolean
) {
  const elementType = {
    $$typeof: REACT_MEMO_TYPE,
    type, // 原本的组件
    compare: compare === undefined ? null : compare,
  };
  return elementType;
}
```

可以看到它实际上是给组件对应的 fiber 修改成了 REACT_MEMO_TYPE 这个类型；React 对 MemoComponent 类型的 fiber 有单独的更新处理逻辑 updateMemoComponent

```js
function updateMemoComponent(
  current: Fiber | null,
  workInProgress: Fiber,
  Component: any,
  nextProps: any,
  renderLanes: Lanes
){
  const prevProps = currentChild.memoizedProps;// 从current树的对应fiber上取到旧props
  let compare = Component.compare;
  compare = compare !== null ? compare : shallowEqual; // compare函数没指定就是默认的浅比较
  if (compare(prevProps, nextProps) && current.ref === workInProgress.ref) {
    // 如果compare返回true，则说明两者相等，那么不更新，直接跳过本组件以及其后的所有组件
    return bailoutOnAlreadyFinishedWork(current, workInProgress, renderLanes);
  }
  // 后面就是设置当前组件的child fiber
}
```

## useMemo 缓存大量的计算

> React 文档：
> 可以使用新的 useMemo 钩子来“记忆”这个计算函数的计算结果。这样只有传入的参数发生变化后，该计算函数才会重新调用计算新的结果。通过这种方式，您可以使用从先前渲染计算的结果来挽救昂贵的计算耗时。总体目标是减少 JavaScript 在呈现组件期间必须执行的工作量，以便主线程被阻塞的时间更短。

```js
// 避免这样做
function Component(props) {
  const someProp = heavyCalculation(props.item);
  return <AnotherComponent someProp={someProp} />;
}

// 只有 `props.item` 改变时someProp的值才会被重新计算
function Component(props) {
  const someProp = useMemo(() => heavyCalculation(props.item), [props.item]);
  return <AnotherComponent someProp={someProp} />;
}
```

## 减少引用

> 使用内联对象（即定义在组件内部的对象）时，react 会在每次渲染时重新创建对此对象的引用，这会导致接收此对象的组件将其视为不同的对象,因此，该组件对于 prop 的浅层比较始终返回 false,导致组件一直重新渲染。

核心是：保证一个对象只初始化一次，指向相同引用

主要体现在两方面：

1. 把一些几乎不会改变的对象放在函数组件外部（比如样式对象、一些功能函数），这样每次重新 render 就不会为这个对象再创建一次引用；
2. 把要传给子组件的**对象类型**props 通过`{...props}`的形式传递；这样组件接收到的便是基本类型的 props，组件通过浅层比较发现接受的 prop 没有变化，则不会重新渲染

```js
function Component(props) {
  const aProp = { someProp: "someValue" };
  return <AnotherComponent style={{ margin: 0 }} aProp={aProp} />;
}

// Do this instead :)
const styles = { margin: 0 };
function Component(props) {
  const aProp = { someProp: "someValue" };
  return <AnotherComponent style={styles} {...aProp} />;
  // 展开运算符相当于 <AnotherComponent someProp='someValue' />
  // 是个基本类型，因此不会出现因为引用改变而重新渲染的问题。
}
```

## 使用`React.lazy`和`<Suspense>`延迟加载组件



## 用控制 css 可见代替条件渲染

```js
// 避免对大型的组件频繁对加载和卸载
function Component(props) {
  const [view, setView] = useState('view1');
  return view === 'view1' ? <SomeComponent /> : <AnotherComponent />
}

// 使用该方式提升性能和速度
const visibleStyles = { opacity: 1 };
const hiddenStyles = { opacity: 0 };
function Component(props) {
  const [view, setView] = useState('view1');
  return (
    <>
      <SomeComponent style={view === 'view1' ? visibleStyles : hiddenStyles}>
      <AnotherComponent style={view !== 'view1' ? visibleStyles : hiddenStyles}>
    </>
  )
}

```


# React18更新

## Concurrent

因为Concurrent的主要变化体现在React内部，因此大部分内容在原版部分讲解。
概括的说，Concurrent带来的新变化主要有：
1. 可中断渲染。曾经的React render阶段是不可中断的，一旦开始由初始化或更新启动的渲染阶段，就会一直到完成渲染，中间不会暂停。

> 在并发渲染中，React 可能会开始呈现更新，在中间暂停，然后稍后继续。它甚至可能完全放弃正在进行的渲染。 React 保证即使渲染被中断，UI 也会保持一致。React 可以在后台准备新的屏幕，而不会阻塞主线程。这意味着 UI 可以立即响应用户输入，即使它正处于大型渲染任务的中间，从而创造流畅的用户体验。

2. 新增的Concurrent相关api，比如startTransition、useDeferedValue等

## Suspense

曾经的Suspense只能支持React.lazy，但在React18中对Suspense可以支持的范围进行了扩充。Suspense现在可以支持常规组件的加载状态，但是还是需要特殊的形式。

官方文档的例子如下：

```js
import { Suspense } from 'react';
import Albums from './Albums.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<Loading />}>
        <Albums artistId={artist.id} />
      </Suspense>
    </>
  );
}

function Loading() {
  return <h2>🌀 Loading...</h2>;
}


// Albums.js
import { fetchData } from './data.js';
export default function Albums({ artistId }) {
  const albums = use(fetchData(`/${artistId}/albums`));
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}

// 这里是关键，当前还并不能直接在组件中渲染一个promise，而是通过这种“等一会”的机制，暂时返回一个promise
function use(promise) {
  if (promise.status === 'fulfilled') {
    return promise.value;
  } else if (promise.status === 'rejected') {
    throw promise.reason;
  } else if (promise.status === 'pending') {
    throw promise;
  } else {
    promise.status = 'pending';
    promise.then(
      result => {
        promise.status = 'fulfilled';
        promise.value = result;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },      
    );
    throw promise;
  }
}

```

suspense组件会对内部挂起的组件在稍后尝试“再次渲染”。初始化时调用use函数，会执行这个promise并抛出；稍后再次调用时则返回的是具体的promise结果。
在Suspense源码那部分讲过，Suspense重渲染机制靠的就是throw一个promise触发Suspense的二次遍历，从而在fallback状态结束后显示promise的结果。这里正是这种方式。
这种形式被称为是“Suspense数据源”，只有启用了 Suspense 的数据源才会激活 Suspense 组件。目前能用的有：
- React.lazy懒加载的组件
- 如上的形式
- 使用支持 Suspense 的框架（如 Relay 和 Next.js）获取数据

当然目前Suspense还是一个实验性的api。使用这种use显然不是一个很稳定的形式。Suspense的更多使用，比如嵌套、显示旧内容等参考https://beta.reactjs.org/reference/react/Suspense


## 自动批处理

即批量更新。在18中，即使更新处在React不可控的范围内（promise等异步任务），依旧可以和同步更新一起完成批量更新。
这个机制的实现靠的是ensureRootIsScheduled函数中的callbackPriority机制，比较两次更新的优先级，如果相同就不再更新。
注意这个批量更新指的是异步任务之内可以，比如同一个promise.then内的所有更新，而不包括同步和异步之间，或者不同异步之间的更新。

## Transition机制

详见react-source-code

## 五个新hooks

- useId：用于在客户端和服务器上生成唯一的 ID。这个id不能作为列表元素的keys，但可以用于html的id属性，比如label标签的htmlFor就需要指定input的id属性，这时就可以用到。
```js
export default function Form() {
  const id = useId();
  return (
    <form>
      <label htmlFor={id}>First Name:</label>
      <input id={id} type="text" />
    </form>
  );
}

```
在同一个组件内部，同一个useId的返回结果id是单次相同的，但是每次重新render得到id是不同的，在不同组件之间的id也不同。
并且，对同构组件来说，useId可以保证在服务端和客户端生成的id相同，前提是组件树相同。这就比使用随机数id或者自增id要好很多。

- useTransition：startTransition的hook形式
- useDeferredValue：包裹一个state，使这个state的更新变为Transition形式
- useSyncExternalStore：用于外部状态库的接入，它消除了在实现对外部数据源的订阅时对 useEffect 的需求
- useInsertionEffect：用于css-in-js库

# 其他React实践

## React表单

React表单主要要解决几个问题：

- 状态的管理。所有的组件都应该是可控的，那么这些组件要怎么去管理状态，怎么触发值的改变，用什么结构来整理状态，是很重要的
- 表单校验。考虑校验方式以及校验之后的提示方式
- 组件关系。即类似Form和FormItem的嵌套、FormItem和具体元素的嵌套，这种嵌套关系之间怎么实现状态的统一和数据的获取。

具体可以参考这篇文章：https://github.com/varHarrie/varharrie.github.io/issues/28

下面是简单内容。

### 基本思路和简单实现

这个表单组件实现起来主要分为三部分：

- Form：用于传递表单上下文。
- Field（也可以叫FormItem）： 表单域组件，用于自动传入value和onChange到表单组件。
- FormStore： 存储表单数据，封装相关操作。

```jsx
<Form store={this.store} onSubmit={this.onSubmit}>
  <Field name="username">
    <input />
  </Field>
  <Field name="password">
    <input type="password" />
  </Field>
  <button>Submit</button>
</Form>
```

formStore可以是一个类：

```js
class FormStore {
  constructor(defaultValues = {}, rules = {}) {
    // 表单值
    this.values = defaultValues;
    // 事件回调
    this.listeners = [];
  }
  subscribe(listener) {
    this.listeners.push(listener);
    // 返回一个用于取消订阅的函数
    return () => {
      const index = this.listeners.indexOf(listener);
      if (index > -1) this.listeners.splice(index, 1);
    };
  }
  // 通知表单变动，调用所有listener
  notify(name) {
    this.listeners.forEach(listener => listener(name));
  }
  get(name) {
    // 如果传入name，返回对应的表单值，否则返回整个表单的值
    return name ? this.values : this.values[name];
  }
  // 设置表单值
  set(name, value) {
    //如果指定了name
    if (typeof name === "string") {
      // 设置name对应的值
      this.values[name] = value;
      this.notify(name);
    } else if (Array.isArray(name)) {
      const values = name;
      Object.keys(values).forEach(key => this.set(key, values[key]));
    }
  }
}
```

获取数据的核心是通过发布订阅模式，即：

- 当每个FormItem创建时，就向FormStore增加一个订阅。当表单内数据变化时，通知所有的订阅者，传递的参数可以是具体的表单的哪一项。

FormStore内还有的其他方法有：

- get和set，一个用于获取value，一个用于修改value。当修改value时，可以再通过notify触发所有的订阅者，传递更新后的value
- 校验。校验可以采用传入一个校验对象，每个键是对应的字段，值是校验函数。每次校验时通过get获取对应字段的值，然后调用校验函数，根据结果设置。
  校验这里，其实可以再用一个发布订阅，或者复用原来的发布订阅传入一个错误。对于订阅者来说，如果收到错误触发，就可以改变样式。

Form和FormItem组件主要如下：

- Form：使用context下发FormStore

```js
const FormStoreContext = React.createContext(undefined);

function Form(props) {
  const { store, children, onSubmit } = props;

  return (
    <FormStoreContext.Provider value={store}>
      <form onSubmit={onSubmit}>{children}</form>
    </FormStoreContext.Provider>
  );
}
```

- FormItem：核心是给子组件传入value和onChange两个参数。当子组件调用onChange时，把值通过store.set修改到FormStore中。
  还要订阅store。当store对应字段的值发生更改时，通过setState修改自己内部的值

```js
function FormItem(props) {
  const { label, name, children } = props;

  const store = useContext(FormStoreContext);

  // 传给子组件的value和onChange
  const [value, setValue] = useState(
    name && store ? store.get(name) : undefined
  );
  const onChange = useCallback(
    // 当onChange触发时，修改FormStore内的value
    (...args) => name && store && store.set(name, formatValue(...args)),
    [name, store]
  );

  // 订阅表单数据变动
  useEffect(() => {
    if (!name || !store) return;

    return store.subscribe(n => {
      // 当前name的数据发生了变动，获取数据并重新渲染
      if (n === name || n === "*") {
        setValue(store.get(name));
      }
    });
  }, [name, store]);

  let child = children;

  // 如果children是一个合法的组件，传入value和onChange
  if (name && store && React.isValidElement(child)) {
    const childProps = { value, onChange };
    child = React.cloneElement(child, childProps);
  }

  // 表单结构
  return (
  );
}
```

如果考虑到组件校验，可以再加上一个error状态用于渲染错误样式。然后通过订阅store内的错误信息，如果匹配到自己内部的字段就提示错误。




## React 表格



