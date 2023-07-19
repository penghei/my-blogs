---
title: react原理浅析学习
date: 2022-07-11 14:07:32
tags: 日常学习
categories: React
cover: /img/react.png
sticky: 7
---

# React jsx

## jsx 的解析

JSX 元素节点会被编译成 React Element 形式。

```js
React.createElement(type, [props], [...children]);
```

createElement 参数：

1. 第一个参数：如果是组件类型，会传入组件对应的类或函数；如果是 dom 元素类型，传入 `'div'` 或者 `'span'` 之类的字符串。
2. 第二个参数：一个对象，在 dom 类型中为标签属性，在组件类型中为 props 。
3. 其他参数：依次为 children，根据顺序排列。（实际上 children 是 props 中的一个属性，在下方执行 createElement 时就会表现出来）

举个例子：

```js
<div>
  <TextComponent />
  <div>hello,world</div>
  let us learn React!
</div>
```

上面的代码会被 babel 编译成：

```js
React.createElement(
  "div", // dom类型，字符串"div"
  null, // 没有props，为null。如果有props，这里可能是对象数组或一个对象
  React.createElement(TextComponent, null), // 组件，组件的type就是引入的那个函数或类
  React.createElement("div", null, "hello,world"),
  "let us learn React!" // children是字符串
);
```

babel 编译 jsx 的过程，之前是依赖 React.createElement 方法，因此需要引入 React；
但是 babel 现在有两种方式编译，一种是 Automatic Runtime，自动注入 createElement 函数，但是需要手动开启；一种是 Classic Runtime，即传统的模式（需要引入 React）

### Automatic Runtime

现在主流 React 脚手架普遍采用的是 Automatic Runtime。
babel 在编译时会引入两个插件，@babel/plugin-syntax-jsx 和 @babel/plugin-transform-react-jsx。前者在编译时会向文件中注入 jsx-runtime 这个 api，就像这样：

```js
import { jsx as _jsx } from "react/jsx-runtime";
import { jsxs as _jsxs } from "react/jsx-runtime";
function Index() {
  return _jsxs("div", {
    children: [
      _jsx("h1", {
        children: "hello,world",
      }),
      _jsx("span", {
        children: "let us learn React",
      }),
    ],
  });
}
```

通过这个函数，就不再需要调用 React.createElement
需要在配置时开启：

```json
"presets": [
    ["@babel/preset-react",{
    "runtime": "automatic"
    }]
],
```

## React.createElement

接下来就是 createElement 的具体执行。这个函数会把上面的解析结果转成具体的 React 对象。具体的转换规则如下：

![](https://pic.imgdb.cn/item/62cbcf46f54cd3f9379a1ae7.jpg)

> 数组类型被转成 element 时，外层会添加一个 Fragment
> 相当于：
>
> ```js
> <div>
>   <>
>     {arr.map(...)}
>   </>
> </div>
> ```

转换结果一般是这样

![](https://pic.imgdb.cn/item/62cbcf92f54cd3f9379a7716.jpg)

可以看到，每个被转换的 React 对象都有一些共有的属性（转化后的类型是 react element 类型的）。具体来说主要是以下几个：

```js
const element = {
  // 标记这是个 React Element，这个属性是一个Symbol
  $$typeof: REACT_ELEMENT_TYPE, // 在React内部验证中(React.isValidElement)起到标识这是个React元素的作用

  type: type, // 类型，就是上面表格中的type属性
  key: key,
  ref: ref,
  props: {...}, // children包含在这里
  _owner: owner,
};
```

createElement 就是进行一些初始化后，返回这样一个对象表示具体的 React 元素。

## jsx 和 fiber

createElement 的解析结果只是一个包含 jsx 内显式数据的一个对象。不包含组件 schedule、reconcile、render 所需的相关信息。

比如如下信息就不包括在 JSX 中：

- 组件在更新中的优先级
- 组件的 state
- 组件被打上的用于 Renderer 的标记

这些内容都包含在 Fiber 节点中。

在**调和**阶段，上述 React element 对象的每一个子节点（每一个元素）都会形成一个与之对应的 fiber 对象，然后通过 sibling、return、child 将每一个 fiber 对象联系起来。
不同的 React element 会产生不同类型的 fiber，fiber 有一个 tag 属性，专门用于标识当前 fiber 是对应的那个 React element：

```js
export const FunctionComponent = 0; // 函数组件
export const ClassComponent = 1; // 类组件
export const IndeterminateComponent = 2; // 初始化的时候不知道是函数组件还是类组件
export const HostRoot = 3; // Root Fiber 可以理解为根元素 ， 通过reactDom.render()产生的根元素
export const HostPortal = 4; // 对应  ReactDOM.createPortal 产生的 Portal
export const HostComponent = 5; // dom 元素 比如 <div>
export const HostText = 6; // 文本节点
export const Fragment = 7; // 对应 <React.Fragment>
export const Mode = 8; // 对应 <React.StrictMode>
export const ContextConsumer = 9; // 对应 <Context.Consumer>
export const ContextProvider = 10; // 对应 <Context.Provider>
export const ForwardRef = 11; // 对应 React.ForwardRef
export const Profiler = 12; // 对应 <Profiler/ >
export const SuspenseComponent = 13; // 对应 <Suspense>
export const MemoComponent = 14; // 对应 React.memo 返回的组件
```

# React 组件

组件本质上就是类和函数，但是与常规的类和函数不同的是，组件承载了渲染视图的 UI 和更新视图的 setState 、 useState 等方法。React 在底层逻辑上会像正常实例化类和正常执行函数那样处理的组件。

在 React 的 reconcil 阶段，具体来说是 BeginWork 阶段，会执行调用函数组件的函数（renderWithHooks，会执行函数组件，返回 children），得到的 children（即函数组件返回的 React element）会拿去构建 fiber，组成到 wip 树上。
在组件被执行并被构建成 fiber 的途中，会根据组件的 type，设置对应的 fiber tag，最后会成为 fiber.tag 属性。这个值回在之后对 fiber 的调和中起到很大的标识作用。

## 类组件

类组件的定义都必须要继承一个 React.Component 的类。这个类实际上长这样：

```js
function Component(props, context, updater) {
  this.props = props; // 这个是使用class组件时，super传入的props
  this.context = context; // 同上
  this.refs = emptyObject; // refs默认为空
  // updater对象，包含enqueueSetState 和 enqueueForceUpdate 方法
  this.updater = updater || ReactNoopUpdateQueue;
}
Component.prototype.isReactComponent = {};

// 类组件的setState方法
Component.prototype.setState = function (partialState, callback) {
  if (
    typeof partialState !== "object" &&
    typeof partialState !== "function" &&
    partialState != null
  ) {
    throw new Error(
      "setState(...): takes an object of state variables to update or a " +
        "function which returns an object of state variables."
    );
  }
  // 调用updater对象的方法执行更新
  this.updater.enqueueSetState(this, partialState, callback, "setState");
};

Component.prototype.forceUpdate = function (callback) {
  this.updater.enqueueForceUpdate(this, callback, "forceUpdate");
};
```

从上面也可以看到，在调用类组件时必须先使用`super(props)`传入 props，即调用 Component 父类，绑定 props 和 updater 等，否则在类组件中就不能获取到 props

```js
constructor(){
    super()
    console.log(this.props) // undefined
}
/* 解决问题 */
constructor(props){
    super(props)
}
```

## 函数组件

函数组件本质上就是一个函数，唯一的区别就是对 hooks 的处理。
React 对函数的处理就是执行这个函数，然后把函数的返回值再返回出去（函数组件的返回值就是 React element）

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
  //...
  return children;
}
```

这部分的具体解析可以看 hooks 那一块。本质上对函数组件的处理就是对 hooks 的处理。

## 函数组件和类组件的本质区别

- 对于类组件来说，底层只需要实例化一次，实例中保存了组件的 state 等状态。对于每一次更新只需要调用 render 方法以及对应的生命周期就可以了。
- 在函数组件中，每一次更新都是一次新的函数执行，一次函数组件的更新，里面的变量会重新声明。为了能让函数组件可以保存一些状态，执行一些副作用钩子，React Hooks 可以帮助记录 React 中组件的状态，处理一些额外的副作用。

# React 事件

## 独立的事件系统

React 为什么要写出一套自己的事件系统呢？

1. 首先，对于不同的浏览器，对事件存在不同的兼容性，React 想实现一个兼容全浏览器的框架， 为了实现这个目标就需要创建一个兼容全浏览器的事件系统，以此抹平不同浏览器的差异。

2. 其次，v17 之前 React 事件都是绑定在 document 上，v17 之后 React 把事件绑定在应用对应的容器 container 上，将事件绑定在同一容器统一管理，防止很多事件直接绑定在原生的 DOM 元素上。造成一些不可控的情况。由于不是绑定在真实的 DOM 上，所以 React 需要模拟一套事件流：事件捕获-> 事件源 -> 事件冒泡，也包括重写一下事件源对象 event 。

3. 最后，这种事件系统，大部分处理逻辑都在底层处理了，这对后期的 ssr 和跨端支持度很高。

React 事件系统可分为三个部分：

- 第一个部分是事件合成系统，初始化会注册不同的事件插件。
- 第二个就是在一次渲染过程中，对事件标签中事件的收集，向 `container` 注册事件。
- 第三个就是一次用户交互，事件触发，到事件执行一系列过程。

## 事件合成

首先是事件合成，所谓事件合成是指 React 应用中，元素绑定的事件并不是原生事件，而是 React 合成的事件

比如 `onClick` 是由 `click` 合成，`onChange` 是由 `blur`、`change`、`focus` 等多个事件合成。

绑定事件并不是一次性绑定所有事件，比如发现了 `onClick` 事件，就会绑定 `click` 事件，比如发现 `onChange` 事件，会绑定 `[blur/change/focus/keydown/keyup]` 多个事件。

```js
// registrationNameDependencies， React 事件和原生事件对应关系
{
    onBlur: ['blur'],
    onClick: ['click'],
    onClickCapture: ['click'],
    onChange: ['blur', 'change', 'click', 'focus', 'input', 'keydown', 'keyup', 'selectionchange'],
    onMouseEnter: ['mouseout', 'mouseover'],
    onMouseLeave: ['mouseout', 'mouseover'],
    ...
}
```

另外，React 事件也不是直接绑定在具体 dom 元素上的（本来 jsx 中也就不是真实 dom），而是统一绑定在顶部容器上，在 v17 之前是绑定在 document 上的，在 v17 改成了 container 容器上（其实就是个 div）。这样更利于一个 html 下存在多个应用（微前端）。

### 事件插件

React 有一种事件插件机制，比如上述 `onClick` 和 `onChange` ，会有不同的事件插件 `SimpleEventPlugin`、`ChangeEventPlugin` 处理。
事件插件主要是帮助合成 React 自己的事件对象`event`，在合成期间会检索`registrationNameModules`对象，然后取出对应的合适的插件，并合成 event 对象。
比如上述的 onClick ，就会用 SimpleEventPlugin 插件处理，onChange 就会用 ChangeEventPlugin 处理。

关于事件插件，有几个比较重要的全局对象如下：

```js
// 事件插件和具体事件的对应关系
const registrationNameModules = {
    onBlur: SimpleEventPlugin,
    onClick: SimpleEventPlugin,
    onClickCapture: SimpleEventPlugin,
    onChange: ChangeEventPlugin,
    onChangeCapture: ChangeEventPlugin,
    onMouseEnter: EnterLeaveEventPlugin,
    onMouseLeave: EnterLeaveEventPlugin,
    ...
}
// 原生事件和合成事件的对应关系
const registrationNameDependencies = {
    onBlur: ['blur'],
    onClick: ['click'],
    onClickCapture: ['click'],
    onChange: ['blur', 'change', 'click', 'focus', 'input', 'keydown', 'keyup', 'selectionchange'],
    onMouseEnter: ['mouseout', 'mouseover'],
    onMouseLeave: ['mouseout', 'mouseover'],
    ...
}
```

每个事件插件大概是这个样子：

```js
const SimpleEventPlugin = {
  registerEvents(domEventName, reactName) {
    topLevelEventsToReactNames.set(domEventName, reactName);
    registerTwoPhaseEvent(reactName, [domEventName]);
  },
  extractEvents: function (
    dispatchQueue, // 事件队列
    domEventName, // 对应的dom事件名
    targetInst,
    nativeEvent, // 原生事件
    nativeEventTarget,
    eventSystemFlags,
    targetContainer
  ) {
    /* eventTypes 里面的事件对应的统一事件处理函数 */
  },
};
```

extractEvents 是事件插件的核心，在 React18 有大更改，18 中的主要任务是：

1. 设置事件构造函数，根据 domEventName（比如'keyup'、'mouseup'这样的）创建不同的事件对象。
2. 创建事件 listener，再和事件对象 event 一起放入 dispatchQueue 中，即收集事件的列表

当事件插件对应的事件被触发时，并不会直接触发绑定的回调，而是先经过事件插件的处理，创建并绑定事件对象和 listener 之后，才会在后面统一触发。

## React 18 新事件系统

新旧事件系统差别：
![](https://pic.imgdb.cn/item/63a5dc7808b6830163f662d1.jpg)

### 事件绑定

老版本的事件原理有一个问题就是，捕获阶段和冒泡阶段的事件都是模拟的，本质上都是在冒泡阶段执行的。事件冒泡到顶层 container，再由 react 经过收集事件、执行事件处理。
新版本的事件会在 createRoot 一开始，就把所有事件一次性绑定到 container 上。

```ts
function createRoot(container, options) {
  /* 省去和事件无关的代码，通过如下方法注册事件 */
  listenToAllSupportedEvents(rootContainerElement);
}
function listenToAllSupportedEvents(rootContainerElement) {
  /* allNativeEvents 是一个 set 集合，保存了大多数的浏览器事件 */
  allNativeEvents.forEach(function (domEventName) {
    if (domEventName !== "selectionchange") {
      /* nonDelegatedEvents 保存了 js 中，不会冒泡的事件，比如scroll；这种事件只能绑定在捕获阶段 */
      if (!nonDelegatedEvents.has(domEventName)) {
        /* 绑定冒泡阶段事件 */
        listenToNativeEvent(domEventName, false, rootContainerElement);
      }
      /* 绑定捕获阶段事件 */
      listenToNativeEvent(domEventName, true, rootContainerElement);
    }
  });
}
```

listenToNativeEvent 会调用 addTrappedEventListener 函数，同时会根据冒泡还是捕获的参数不同，创建一个 eventSystemFlags 并打上 capture 标签

```ts
function listenToNativeEvent(
  domEventName: DOMEventName,
  isCapturePhaseListener: boolean,
  target: EventTarget
): void {
  let eventSystemFlags = 0;
  if (isCapturePhaseListener) {
    eventSystemFlags |= IS_CAPTURE_PHASE;
  }
  addTrappedEventListener(
    target,
    domEventName,
    eventSystemFlags,
    isCapturePhaseListener
  );
}
```

addTrappedEventListener 函数就是实际添加 listener 的函数了：

```ts
function addTrappedEventListener(
  targetContainer: EventTarget,
  domEventName: DOMEventName,
  eventSystemFlags: EventSystemFlags,
  isCapturePhaseListener: boolean,
  isDeferredListenerForLegacyFBSupport?: boolean,
) {
  // 这个函数会根据事件优先级的不同，创建不同的dispatchEvent
  let listener = createEventListenerWrapperWithPriority(
    targetContainer,
    domEventName,
    eventSystemFlags,
  );

  // 这个值将作为addEventListener的第三个参数
  let isPassiveListener: void | boolean = undefined;
  if (
      domEventName === 'touchstart' ||
      domEventName === 'touchmove' ||
      domEventName === 'wheel'
    ) {
      // 对于这三个事件，addEventListener的第三个参数一定为true
      isPassiveListener = true;
    }

  // 目标容器，一般是root所在的container
  targetContainer =
    enableLegacyFBSupport && isDeferredListenerForLegacyFBSupport
      ? (targetContainer: any).ownerDocument
      : targetContainer;

  let unsubscribeListener;
  // 实际添加addEventListener
  if (isCapturePhaseListener) {
    if (isPassiveListener !== undefined) {
      unsubscribeListener = addEventCaptureListenerWithPassiveFlag(
        targetContainer,
        domEventName,
        listener,
        isPassiveListener,
      );
    } else {
      unsubscribeListener = addEventCaptureListener(
        targetContainer,
        domEventName,
        listener,
      );
    }
  } else {
    if (isPassiveListener !== undefined) {
      unsubscribeListener = addEventBubbleListenerWithPassiveFlag(
        targetContainer,
        domEventName,
        listener,
        isPassiveListener,
      );
    } else {
      unsubscribeListener = addEventBubbleListener(
        targetContainer,
        domEventName,
        listener,
      );
    }
  }
}
```

listener 是 react 创建的事件回调，当事件触发时执行的就是 listener 函数。
listener 通过`createEventListenerWrapperWithPriority`创建，这个函数会根据事件优先级的不同，创建不同的 dispatchEvent；详见下

```ts
function createEventListenerWrapperWithPriority(
  targetContainer: EventTarget,
  domEventName: DOMEventName,
  eventSystemFlags: EventSystemFlags
): Function {
  const eventPriority = getEventPriority(domEventName);
  let listenerWrapper;
  switch (eventPriority) {
    case DiscreteEventPriority:
      // 非连续性事件，比如click
      listenerWrapper = dispatchDiscreteEvent;
      break;
    case ContinuousEventPriority:
      // 持续性事件，比如mousemove
      listenerWrapper = dispatchContinuousEvent;
      break;
    case DefaultEventPriority:
    default:
      // 其他类型
      listenerWrapper = dispatchEvent;
      break;
  }
  return listenerWrapper.bind(
    null,
    domEventName,
    eventSystemFlags,
    targetContainer
  );
}
```

以 dispatchDiscreteEvent 为例，内部代码如下所示：

```ts
function dispatchDiscreteEvent(
  domEventName,
  eventSystemFlags,
  container,
  nativeEvent
) {
  const previousPriority = getCurrentUpdatePriority();
  const prevTransition = ReactCurrentBatchConfig.transition;
  ReactCurrentBatchConfig.transition = null;
  try {
    // 当事件触发时，将本次更新优先级设置为Discrete，即非连续事件的优先级（最高）
    setCurrentUpdatePriority(DiscreteEventPriority);
    dispatchEvent(domEventName, eventSystemFlags, container, nativeEvent);
  } finally {
    setCurrentUpdatePriority(previousPriority);
    ReactCurrentBatchConfig.transition = prevTransition;
  }
}
```

函数内部的核心还是 dispatchEvent，但是这里会设置更新优先级。当事件触发时，事件内的更新就会以此优先级执行。

### 事件触发

当我们触发事件是，比如点击一个按钮，那么事件会向上冒泡，最终到达 container 元素并被收集到。因为 react 预先添加了所有的捕获和冒泡事件，所以 dispatchEvent 实际上会触发两次，一次是捕获阶段触发，一次是冒泡阶段触发。

> 触发两次 dispatchEvent 也就意味着 react**每次实际上是处理了两次事件**（只针对同时能冒泡和捕获的事件），一次是捕获阶段的 addEventListener 触发的，一次是冒泡阶段的。
> 两次 dispatchEvent 也就意味着收集了两次事件，捕获阶段只会收集捕获类型的事件，冒泡阶段也只会收集冒泡阶段的事件，两者不会冲突；事件的收集都是从下向上的，只是收集的目标不同。
> 收集完成后，再根据本次是冒泡触发的还是捕获触发的，按照不同顺序执行收集的事件。

当事件触发，首先执行 addEventListener 传入的回调，即通过 createEventListenerWrapperWithPriority 创建的三种 dispatchEvent 之一；
在他们内部，还是会调用 dispatchEvent，然后调用 dispatchEventOriginal、dispatchEventForPluginEventSystem，最后来到这样一段代码：

```js
batchedUpdates(() =>
  dispatchEventsForPlugins(
    domEventName,
    eventSystemFlags,
    nativeEvent,
    ancestorInst,
    targetContainer
  )
);
```

batchedUpdates 很简单，就是那个`开启批量更新-执行-关闭批量更新的`函数。也就是说用批量更新执行了 dispatchEventsForPlugins 函数；这个函数具体如下：

```ts
function dispatchEventsForPlugins(
  domEventName: DOMEventName,
  eventSystemFlags: EventSystemFlags,
  nativeEvent: AnyNativeEvent,
  targetInst: null | Fiber,
  targetContainer: EventTarget
): void {
  // 从event事件对象找到事件的触发dom元素
  const nativeEventTarget = getEventTarget(nativeEvent);
  // 待更新队列，关键
  const dispatchQueue: DispatchQueue = [];
  // 找到待执行的事件
  extractEvents(
    dispatchQueue,
    domEventName,
    targetInst,
    nativeEvent,
    nativeEventTarget,
    eventSystemFlags,
    targetContainer
  );
  // 执行事件
  processDispatchQueue(dispatchQueue, eventSystemFlags);
}
```

那么现在清楚了：如果触发事件，就会创建一个 dispatchQueue，然后用 extractEvents 向 dispatchQueue 中放入某些值，最后执行 dispatchQueue。
extractEvents 函数就是调用事件插件的 extractEvent 方法（详见上），简化代码如下：

```ts
function extractEvents(
  dispatchQueue: DispatchQueue,
  domEventName: DOMEventName,
  targetInst: null | Fiber,
  nativeEvent: AnyNativeEvent,
  nativeEventTarget: null | EventTarget,
  eventSystemFlags: EventSystemFlags,
  targetContainer: EventTarget
): void {
  let SyntheticEventCtor = SyntheticEvent;
  let reactEventType: string = domEventName;
  switch (domEventName) {
    case "keydown":
    case "keyup":
      SyntheticEventCtor = SyntheticKeyboardEvent;
      break;
    case "focusin":
      reactEventType = "focus";
      SyntheticEventCtor = SyntheticFocusEvent;
      break;
    // 后面类似，就是各种事件
    default:
      // Unknown event. This is used by createEventHandle.
      break;
  }
  // 找到所有会触发的事件监听的函数，是一个数组
  // 也就是收集事件的过程
  const listeners = accumulateSinglePhaseListeners(
    targetInst,
    reactName, // 指定要收集的事件名，比如'onclick'，那么就只收集onclick事件
    nativeEvent.type,
    accumulateTargetOnly, // 控制遍历，如果当前事件不能冒泡，这个值为true，就会终止向上收集
    nativeEvent,
    inCapturePhase // 这个变量用于控制是否收集捕获事件
  );
  if (listeners.length > 0) {
    // 创建事件对象，SyntheticEventCtor就是上面switch case中设置的SyntheticEvent
    const event: ReactSyntheticEvent = new SyntheticEventCtor(
      reactName,
      reactEventType,
      null,
      nativeEvent,
      nativeEventTarget
    );
    dispatchQueue.push({ event, listeners });
  }
}
```

注意这里是怎么找到事件触发的那个 fiber：
react 在创建真实 dom 时向 dom 添加了一个随机的 internalInstanceKey，指向当前 dom 对应的 fiber 对象。fiber 和真实 dom 对应关系如图：
![](https://pic.imgdb.cn/item/640db4d3f144a0100762ecf9.jpg)

accumulateSinglePhaseListeners 函数，会获取存储在 Fiber 上的 Props 的对应事件，然后通过一个循环收集事件，也就是我们说的收集事件的环节：

```ts
export function accumulateSinglePhaseListeners(
  targetFiber: Fiber | null,
  reactName: string | null, // 要收集的事件名称
  nativeEventType: string,
  inCapturePhase: boolean, // 是否是捕获阶段的事件
  accumulateTargetOnly: boolean,
  nativeEvent: AnyNativeEvent
): Array<DispatchListener> {
  const captureName = reactName !== null ? reactName + "Capture" : null;
  const reactEventName = inCapturePhase ? captureName : reactName;
  // 创建用于收集事件的数组
  let listeners: Array<DispatchListener> = [];
  // 事件触发的那个fiber
  let instance = targetFiber;
  let lastHostComponent = null;

  // 从事件触发的那个节点开始，从下向上收集事件
  while (instance !== null) {
    const { stateNode, tag } = instance;
    // 处理放在dom类型element上的事件
    if (
      (tag === HostComponent ||
        (enableFloat ? tag === HostResource : false) ||
        (enableHostSingletons ? tag === HostSingleton : false)) &&
      stateNode !== null
    ) {
      lastHostComponent = stateNode;

      // Standard React on* listeners, i.e. onClick or onClickCapture
      if (reactEventName !== null) {
        // 获取fiber上绑定的listener函数
        const listener = getListener(instance, reactEventName);
        if (listener != null) {
          listeners.push(
            createDispatchListener(instance, listener, lastHostComponent)
          );
        }
      }
    }
    // 递归向上收集
    instance = instance.return;
  }
  return listeners;
}
```

然后通过 createDispatchListener 返回的对象（就是下面这个 listener 对象）加入到监听集合上，如果是不会冒泡的函数则会停止（比如：scroll）,反之会向上递归。收集可以指定收集哪种事件、是否是捕获，同一类事件，要么捕获要么冒泡。

listeners 是一个数组，里面每一项有三个属性：

```ts
type listener = {
  instance; // 发生事件的fiber
  listener; // 事件处理函数
  currentTarget; // 发生事件的dom元素
};
type listeners = listener[];
```

listeners 就是从 fiber 上获取的事件处理函数，是一个数组，因为绑定的事件可能有多个。
注意这个 listeners 并不是只有事件发生的那个元素上的，而是和旧版本的事件收集一样，从下向上收集所有相同的事件。
比如是 onClick，那就收集父级元素的所有 onClick 事件，比如是 onClickCapture，那就收集父级的所有 onClickCapture。

接下来就是 processDispatchQueue 执行事件了。他会遍历 dispatchQueue，取出每一项的 listeners 去执行。

> dispatchQueue 虽然是一个数组，但它通常只有一项。注意 dispatchQueue 并非存储具体事件处理函数的数组，其内部对象的 listeners 才是
> ![](https://pic.imgdb.cn/item/64032cd5f144a01007bd8a9c.jpg)

这里执行 listeners 的顺序，也就是冒泡和捕获的顺序。因为 listeners 存储的是一类事件的全部（比如全部 onclick），因此 listeners 要么顺序执行（冒泡），要么倒序执行（捕获）。

```ts
function processDispatchQueue(
  dispatchQueue: DispatchQueue,
  eventSystemFlags: EventSystemFlags
): void {
  const inCapturePhase = (eventSystemFlags & IS_CAPTURE_PHASE) !== 0;
  for (let i = 0; i < dispatchQueue.length; i++) {
    const { event, listeners } = dispatchQueue[i];
    if (inCapturePhase) {
      // 捕获类型的event，那就倒序执行
      for (let i = dispatchListeners.length - 1; i >= 0; i--) {
        const { instance, currentTarget, listener } = listeners[i];
        if (instance !== previousInstance && event.isPropagationStopped()) {
          return;
        }
        executeDispatch(event, listener, currentTarget);
      }
    } else {
      // 冒泡类型，正序执行
      for (let i = 0; i < dispatchListeners.length; i++) {
        const { instance, currentTarget, listener } = dispatchListeners[i];
        if (instance !== previousInstance && event.isPropagationStopped()) {
          return;
        }
        executeDispatch(event, listener, currentTarget);
      }
    }
  }
}
```

根据 event 的类型，选择 listeners 的执行方式，就模拟出了冒泡和捕获的差别。

这里和老版事件系统不一样的地方在于，老版事件收集的过程并没有分开捕获事件和冒泡事件，而新版收集时只会收集一种，另一种会单独放到另外一个数组中。

# React state 机制

## 类组件的 state

类组件的 state 核心控制函数是 setState。
当在某一次事件中调用一次 setState，会按照下面的步骤依次执行：

1. 首先，setState 会产生当前更新的优先级（老版本用 expirationTime ，新版本用 lane ），并创建一个更新对象 update，包含此次更新的 lane
2. 接下来 React 会从 fiber Root 根部 fiber 向下调和子节点，调和阶段将对比发生更新的地方，更新对比 lane ，找到发生更新的组件，合并 state，然后触发 render 函数，得到新的 UI 视图层，完成 render 阶段。
3. 接下来到 commit 阶段，commit 阶段，替换真实 DOM ，完成此次更新流程。
4. 此时仍然在 commit 阶段，会执行 setState 中 callback 函数，到此为止完成了一次 setState 全过程。

无论是函数组件的 dispatchState 还是类组件的 setState，在可控范围内调用时都会产生一个 update，伴随着一个 lane 优先级。

### setState

从上面类组件的分析可以得出，setState 实际上是执行了 Component 类的 updater 对象上的 enqueueSetState 方法。
这个方法大概如下：

```js
const classComponentUpdater = {
  //...
  enqueueSetState(inst, payload, callback) {
    // 获取当前触发更新的fiber节点。inst是组件实例
    const fiber = getInstance(inst);
    // eventTime是当前触发更新的时间戳
    const eventTime = requestEventTime();
    const suspenseConfig = requestCurrentSuspenseConfig();
    // 获取本次update的优先级
    const lane = requestUpdateLane(fiber, suspenseConfig);
    // 创建update对象
    const update = createUpdate(eventTime, lane, suspenseConfig);
    // payload就是setState的参数，回调函数或者是对象的形式。
    // 处理更新时参与计算新状态的过程
    update.payload = payload;
    // callback是setState的第二个参数
    if (callback !== undefined && callback !== null) {
      update.callback = callback;
    }
    // 将update放入fiber的updateQueue
    const root = enqueueUpdate(fiber, update, lane); // 开始进行调度
    if (root !== null) {
      // 调度的入口，
      scheduleUpdateOnFiber(root, fiber, lane, eventTime);
      entangleTransitions(root, fiber, lane);
    }
  },
  //...
};
```

update 的结构大概是这样：

```js
const update = {
  eventTime, // 更新的产生时间
  lane, // 优先级
  suspenseConfig,
  tag: UpdateState, // 更新类型
  payload: null, // 状态，即setState设置的状态
  callback: null,
  next: null, // 指向下一个更新，用于连接批量更新
};
```

enqueueSetState 作用实际很简单，就是创建一个 update ，然后放入当前 fiber 对象的待更新队列（updateQueue）中，最后开启调度更新（scheduleUpdateOnFiber）。

### 老版批量更新

> 以下是 react17 及之前的策略

如果在事件处理函数中调用 setState，那么会执行一个绑定批量更新的步骤 batchedEventUpdates。这个函数会在事件执行开始前开启一个批量更新的开关，再在执行之后关闭它。当开关开启时，enqueueSetState 函数中的 scheduleUpdateOnFiber 就会按照 fiber 上的更新队列依次执行批量更新。

```js
export function batchedEventUpdates(fn, a) {
  isBatchingEventUpdates = true; //打开批量更新开关
  try {
    fn(a); // 事件在这里执行
  } finally {
    isBatchingEventUpdates = false; //关闭批量更新开关
    if (executionContext === NoContext) {
      flushSyncCallbackQueue(); // 这个很重要，用来执行同步更新队列中的任务，也就是批量更新的任务
    }
  }
}
```

> 当执行批量更新时，几个优先级相同的更新结果会被合并，最后只形成一次更新，值为最新的那个值；
> 在 react 中，会将事件处理、生命周期这种 react 可以控制的“入口”进行批量处理更新标记。
> 在执行处理函数前，会将 isBatchingEventUpdates 标记为 true，然后执行函数，如果函数中有 setState 会将 updater 放入到队列中(updateQueue)。
> 当函数执行完，会清空队列合并 state，执行 render 进行渲染更新 dom，这些完成后会去执行 setState 里传入的 callback
> 如果处理函数中有 React.flushSync 会提高优先级，它会和前面的 setState 合并，执行一次更新，之后再去继续执行处理函数里后面的内容
> 当处理函数执行完，会将 isBatchingEventUpdates 标记为 false，意味着除了事件之外的其他更新不会进行批量更新；

批量更新示例：
![](https://pic.imgdb.cn/item/624710a527f86abb2a376e2b.jpg)

当更新不在 React 的控制范围内时，即不在事件处理函数内或者在异步函数内时，就会导致批量更新失效。比如给每个 setState 外面套一个 setTimeout
![](https://pic.imgdb.cn/item/62cbee7df54cd3f937cd9480.jpg)

可以通过手动控制的方式，让类组件的非批量更新转为批量更新。一个是通过`unstable_batchedUpdates`

```js
setTimeout(() => {
  unstable_batchedUpdates(() => {
    this.setState({ number: this.state.number + 1 });
    console.log(this.state.number);
    this.setState({ number: this.state.number + 1 });
    console.log(this.state.number);
    this.setState({ number: this.state.number + 1 });
    console.log(this.state.number);
  });
});
```

这个 api 实际的作用是绑定批量更新，回调内的 setState 会被强行合成为批量更新。
另外一种方法是更改更新的优先级，提高优先级可以使用`flushSync`，降低优先级可以使用`startTransition`

flushSync 在同步条件下，会合并之前的 setState | useState，可以理解成，如果发现了 flushSync ，就会先执行更新，如果之前有未更新的 setState ｜ useState ，就会一起合并了。

```js
setState(1);
setTimeout(() => setState(2));
flushSync(() => setState(3));
setState(4);
```

下面举个例子，讲解老版本批量更新是什么原理。React18 不再采用这种方式

1. 我们用一个 wrapEvent 函数包裹事件回调 handleClick，这是可以看到当 handleClick 执行时，会先`设置batchEventUpdate = true`，然后执行自己原本的逻辑。
2. 在原本的 handleClick 内部肯定是好几个 setState，这几个 setState 内部会判断当前是不是批量更新；如果是，就把更新放到队列中，如果不是就直接执行。这也就意味着同步条件下所有的更新都会被放到队列，而如果是异步执行 setState，批量更新肯定是关闭的，也就不可能放到数组中去。
3. 然后当收集完更新后，在 flushSyncCallbackQueue 函数中一次执行所有的更新任务，就实现了一次批量更新。

```js
let batchEventUpdate = false;
let callbackQueue = [];

// 以同步的方式执行所有任务
function flushSyncCallbackQueue() {
  console.log("-----执行批量更新-------");
  while (callbackQueue.length > 0) {
    const cur = callbackQueue.shift();
    cur();
  }
  console.log("-----批量更新结束-------");
}

function wrapEvent(fn) {
  return function () {
    /* 开启批量更新状态 */
    batchEventUpdate = true;
    fn(); // 这里的fn就是handleClick
    /* 立即执行更新任务 */
    flushSyncCallbackQueue();
    /* 关闭批量更新状态 */
    batchEventUpdate = false;
  };
}

function setState(fn) {
  /* 如果在批量更新状态下，那么批量更新 */
  if (batchEventUpdate) {
    callbackQueue.push(fn);
  } else {
    /* 如果没有在批量更新条件下，那么直接更新。 */
    fn();
  }
}

function handleClick() {
  setState(() => {
    console.log("---更新1---");
  });
  console.log("上下文执行");
  setState(() => {
    console.log("---更新2---");
  });
}
/* 让 handleClick 变成可控的  */
handleClick = wrapEvent(handleClick);

handleClick();
```

如果在 handleClick 中有异步任务，那么显然执行异步更新时，batchEventUpdate = false，因此就不会批量更新，所有的异步更新都是单独更新的。

## 函数组件的 state

函数组件的 state 主要依靠 usestate 实现。具体还是需要参考 hooks 部分的 useState 解析。

函数组件 state 底层实现和类组件类似。尽管函数组件主要依靠 hooks 和 fiber 的配合完成状态的更新和保留，但是他们底层都有批量更新机制，同时也都是调用了 scheduleUpdateOnFiber 方法。

具体参考下面 hooks 讲解内的 useState 原理。

## setState 的同步异步

> 概括：
>
> - 如果 `setState` 在 React 能够控制的范围被调用，即如果是由 React 引发的事件处理，它就是异步的。比如合成事件处理函数，生命周期函数， 此时会进行批量更新，也就是将状态合并后再进行 DOM 更新。
> - 如果 `setState` 在原生 JavaScript 控制的范围被调用，它就是同步的（除了上面情况的其他情况）。比如原生事件处理函数，定时器回调函数，Ajax 回调函数中，此时 `setState` 被调用后会立即更新 DOM 。

从代码层面来说，如果 setState 被能控制地调用两次，那就会被合并成一次调用；反之如果是在异步等不能控制的地方，那么一次 setState 就是一次更新，调用多少个 setState 就会导致多少个更新，不会合并。这一点在函数组件和类组件中是一样的：

```js
function Index() {
  const [number, setNumber] = React.useState(0);
  /* 同步条件下 */
  const handleClickSync = () => {
    setNumber(1);
    setNumber(2);
  };
  /* 异步条件下 */
  const handleClick = () => {
    setTimeout(() => {
      setNumber(1);
      setNumber(2);
    }, 0);
  };
  console.log("----组件渲染----");
  return (
    <div>
      {number}
      <button onClick={handleClickSync}>同步环境下</button>
      <button onClick={handleClick}>异步环境下</button>
    </div>
  );
}
```

点击同步环境下按钮，会输出一次，也就是说只经过了一次更新；
点击异步环境下，输出两次，说明没有合并，多少个 setState 就更新多少次。
**这种情况在 concurrent 模式下完全不同**，下面会说到。

### setState

首先是类组件的`setState`：

```js
state = {
    number:1
};
componentDidMount(){
    this.setState({number:3})
    console.log(this.state.number) // 1
}
```

`setState`并没有异步的说法，只是有一种异步方法的表现形式（批量更新），为了提高性能，将多个状态合并一起更新，减少 re-render 调用。

> React 会将多个 setState 的调用合并为一个来执行，也就是说，当执行 setState 的时候，state 中的数据并不会马上更新

获取最新的 state 方法主要有两种：

1. 使用`setState`的第二个参数，传入一个回调可以获取最新的 state
2. 用`setTimeout`等异步方法包裹 setState，这样在其他同步代码中获取的 state 就是最新的。
3. 使用 dom 的原生事件，即`addEventListener`添加的事件，在事件的回调上触发`setState`：

```js
state = {
    number:1
};
componentDidMount() {
    document.body.addEventListener('click', this.changeVal, false);
}
changeVal = () => {
    this.setState({
      number: 3
    })
    console.log(this.state.number) // 3
}
```

### useState

函数组件中`useState`的表现也和`setState`类似，同样是具有异步表现、合并更新的特点。

其实是函数组件更新就是函数的执行，在函数一次执行过程中，函数内部所有变量重新声明，所以改变的 state 只有在**下一次函数组件执行**时才会体现出来。因此本次更新的 state，在本次函数执行时无论如何也是拿不到最新的 state 的。

从这个角度来讲可以把`useState`看作是一个异步任务，而且是一定排在最后的异步任务。

这一点和类组件略有不同，函数组件无论如何都不会在 state 更新之前获取到的。

```js
export default () => {
  const [state, setState] = useState(0);

  setState(2);
  console.log(state); // 0
  setTimeout(() => console.log(state), 0); // 0
};
```

唯一的方法就是等待函数下一次 render，也就是函数下一次执行的时候。伴随函数下一次执行而执行的方法就是`useEffect`，它会在每次函数执行时销毁上一个`effect`并新建一个和当前 render 匹配的`effect`

```js
export default () => {
  const [state, setState] = useState(0);

  setState(2);
  useEffect(() => {
    console.log(state); // 2
  }, [state]);
};
```

## React18 的自动批量更新

如果开启 18 的 concurrent 模式，就没有了之前的批量更新的概念。React18 取消了之前的 isBatchingEventUpdates 机制，而是采用微任务的形式，将所有同步更新整理后立即执行。
具体来说是这一段代码：

```js
function ensureRootIsScheduled(root, currentTime) {
  // 采用最高优先级代表更新的优先级，这个newCallbackPriority就是所有lanes的最高值，不存在更大的lane
  const newCallbackPriority = getHighestPriorityLane(nextLanes);

  // 检查是否有存在的优先级（root上的），如果有就退出并重用root上的
  // root上存的是之前更新的优先级，也就是说如果本次更新优先级和上次一样，说明是在一次更新内部，会退出
  const existingCallbackPriority = root.callbackPriority;
  if (existingCallbackPriority === newCallbackPriority) {
    return;
  }

  // 调度一个新的更新任务
  let newCallbackNode;
  if (newCallbackPriority === SyncLanePriority) {
    // 这里根据Legacy模式和concurrent模式分别进入不同的调度流程
    // 但是核心都是performSyncWorkOnRoot，这个是调度的入口函数
    if (root.tag === LegacyRoot) {
      scheduleLegacySyncCallback(performSyncWorkOnRoot.bind(null, root));
    } else {
      scheduleSyncCallback(performSyncWorkOnRoot.bind(null, root));
    }
    // 接下来就是触发微任务，用微任务去立即执行更新
    // 微任务执行更新的详细可以看state那块的
    if (supportsMicrotasks) {
      scheduleMicrotask(() => {
        if (
          (executionContext & (RenderContext | CommitContext)) ===
          NoContext
        ) {
          // 这个地方和旧版一样，收集事件内部的所有setState，然后一次性执行更新
          flushSyncCallbacks();
        }
      });
    }
    newCallbackNode = null;
  } else {
    // 如果不是同步任务，就确定另一种优先级，即根据到期事件确定的优先级机制
    // 异步会直接执行scheduleCallback
    newCallbackNode = scheduleCallback(
      schedulerPriorityLevel, // 这个值是更改后的优先级，从lane变成由过期时间确定的优先级
      performConcurrentWorkOnRoot.bind(null, root)
    );
  }
  // 把当前任务的优先级绑定到root上
  root.callbackPriority = newCallbackPriority;
  root.callbackNode = newCallbackNode;
}
```

当同步状态下触发多次 setState 的时候。

- 首先第一次进入到 ensureRootIsScheduled ，会计算出 newCallbackPriority 可以理解成执行新的更新任务的优先级。那么和之前的 callbackPriority 进行对比，如果相等那么退出流程，那么第一次两者肯定是不相等的；如果是第二次以及以后再次触发 useState，显然这些任务都是一样的优先级 Synclane，那么就不会再执行后续的步骤
- 同步状态下常规的更新 newCallbackPriority 是等于 SyncLane 的，就是之前在 lane 机制说过的所有事件同步更新的优先级，那么会执行两个函数，scheduleSyncCallback 和 scheduleMicrotask。
  - scheduleSyncCallback 会把任务 syncQueue 同步更新队列中，即检查有没有 syncQueue，如果没有就创建，有就把任务 push 进去。这个队列专门收集优先级相同的任务，具体来说是同步任务
  ```js
  export function scheduleSyncCallback(callback: SchedulerCallback) {
    if (syncQueue === null) {
      syncQueue = [callback];
    } else {
      syncQueue.push(callback);
    }
  }
  ```
  - scheduleMicrotask 根据浏览器兼容性选择不同的微任务 api，可能是 queueMicrotask、Promise.resolve 或者 setTimeout。通过微任务执行 flushSyncCallbacks，也就是同步任务更新队列；借此就实现了批量更新。

---

异步情况下，会直接执行 scheduleCallback，即 Scheduler 内部的 unstable_ScheduleCallback。scheduler 是一个具有时间分片机制的调度器。
也就是说，异步情况下其实也可以说是 concurrent 模式下，此时 react 不会像同步状态那样直接顺序执行，而是采用时间分片的机制执行任务。具体时间分片的原理参考下面的 scheduler 讲解。

至于为什么微任务可以实现同步更新的效果，可以参考下面这个例子：
要实现任务的批量更新，本质上就是先收集任务，再在某个恰当的时机一起执行这些任务。对 React 来说就是收集各个 setState，再一次性执行完成这些 setState，达成只更新一次的效果。
那么微任务或者异步任务实现批量更新也就是这个原理，在同步代码中收集任务，在异步代码中执行任务

```js
class Scheduler {
  constructor() {
    this.callbacks = [];
    /* 微任务批量处理 */
    // 当mockOnclick执行完之后，就会执行queueMicrotask，开始执行之前收集的任务
    queueMicrotask(() => {
      this.runTask();
    });
  }
  /* 增加任务 */
  addTask(fn) {
    this.callbacks.push(fn);
  }
  runTask() {
    console.log("------合并更新开始------");
    while (this.callbacks.length > 0) {
      const cur = this.callbacks.shift();
      cur();
    }
    console.log("------合并更新结束------");
    console.log("------开始更新组件------");
  }
}
function nextTick(cb) {
  const scheduler = new Scheduler();
  cb(scheduler.addTask.bind(scheduler));
}

/* 模拟一次更新 */
function mockOnclick() {
  nextTick((addTask) => {
    addTask(function () {
      console.log("第一次更新");
    });
    console.log("----宏任务逻辑----");
    addTask(function () {
      console.log("第二次更新");
    });
  });
}
mockOnclick();
```

# React Fiber

## 引入 fiber 的原因

fiber 在 React 中是最小粒度的执行单元，无论 React 还是 Vue ，在遍历更新每一个节点的时候都不是用的真实 DOM ，都是采用虚拟 DOM ，所以可以理解成 fiber 就是 React 的虚拟 DOM 。

更新 fiber 的过程叫做 Reconciler（调和器），每一个 fiber 都可以作为一个执行单元来处理，所以每一个 fiber 可以根据自身的过期时间 expirationTime（ v17 版本叫做优先级 lane ）来判断是否还有空间时间执行更新，如果没有时间更新，就要把主动权交给浏览器去渲染，做一些动画，重排（ reflow ），重绘 repaints 之类的事情，这样就能给用户感觉不是很卡。然后等浏览器空余时间，在通过 scheduler （调度器），再次恢复执行单元上来，这样就能本质上中断了渲染，提高了用户体验。

## element / fiber / dom 的关系

![](https://pic.imgdb.cn/item/624a0ee2239250f7c564e273.jpg)

- element 是 React 视图层在代码层级上的表象，也就是写的 jsx 语法通过 createElement 转化后的结果。
- DOM 是元素在浏览器上给用户直观的表象。
- fiber 可以说是 element 和真实 DOM 之间的交流枢纽站，一方面每一个类型 element 都会有一个与之对应的 fiber 类型，element 变化引起更新流程都是通过 fiber 层面做一次调和改变，然后对于元素，形成新的 DOM 做视图渲染。

React 针对不同 React element 对象会产生不同 tag (种类) 的 fiber 对象。element 和 fiber 之间的关系按照数字对应，一个组件对应一个数字类型的 fiber

```js
export const FunctionComponent = 0; // 函数组件
export const ClassComponent = 1; // 类组件
export const IndeterminateComponent = 2; // 初始化的时候不知道是函数组件还是类组件
export const HostRoot = 3; // Root Fiber 可以理解为根元素 ， 通过reactDom.render()产生的根元素
export const HostPortal = 4; // 对应  ReactDOM.createPortal 产生的 Portal
export const HostComponent = 5; // dom 元素 比如 <div>
export const HostText = 6; // 文本节点
export const Fragment = 7; // 对应 <React.Fragment>
export const Mode = 8; // 对应 <React.StrictMode>
export const ContextConsumer = 9; // 对应 <Context.Consumer>
export const ContextProvider = 10; // 对应 <Context.Provider>
export const ForwardRef = 11; // 对应 React.ForwardRef
export const Profiler = 12; // 对应 <Profiler/ >
export const SuspenseComponent = 13; // 对应 <Suspense>
export const MemoComponent = 14; // 对应 React.memo 返回的组件
```

## fiber 的结构

fiber 相当于一个保存虚拟 dom 信息的对象，其上保存信息类似如下：

```js
function FiberNode() {
  this.tag = tag; // fiber 标签 证明是什么类型fiber，即上面的数字序号
  this.key = key; // key调和子节点时候用到。
  this.type = null; // dom元素是对应的元素类型，比如div，组件指向组件对应的类或者函数。
  this.stateNode = null; // 指向对应的真实dom元素，类组件指向组件实例，可以被ref获取。

  /* 联系多个fiber间关系 */
  this.return = null; // 指向父级fiber
  this.child = null; // 指向子级fiber
  this.sibling = null; // 指向兄弟fiber
  this.index = 0; // 索引

  this.ref = null; // ref指向，ref函数，或者ref对象。
  this.refClean = null;

  /* 存储state、hooks、dom等信息 */
  this.pendingProps = pendingProps; // 在一次更新中，代表element创建
  this.memoizedProps = null; // 记录上一次更新完毕后的props
  this.updateQueue = null; // 类组件存放setState更新队列，函数组件存放useState更新队列
  this.memoizedState = null; // 类组件保存state信息，函数组件保存hooks信息，dom元素为null
  this.dependencies = null; // context或是时间的依赖项

  this.mode = mode; //描述fiber树的模式，比如 ConcurrentMode 模式

  /* effect相关 */
  this.flags = NoFlags;
  this.subtreeFlags = NoFlags;
  this.deletions = null;

  this.lanes = NoLanes; // 优先级
  this.childLanes = NoLanes; // children的优先级

  /* 作为指针，指向wip */
  this.alternate = null; //双缓存树，指向缓存的fiber。更新阶段，两颗树互相交替。
}
```

### fiber 间关系

fiber 主要通过`return`/`child`/`sibling` 三个属性建立起联系的

- `return`： 指向父级 Fiber 节点。
- `child`： 指向子 Fiber 节点。
- `sibling`：指向兄弟 fiber 节点。

建立起的 fiber 树类似这样：
![](https://pic.imgdb.cn/item/624a11ab239250f7c56bd323.jpg)

## fiber 更新机制

### 初始化

初始化流程如下：

1. 创建`fiberRoot`和`rootFiber`，并将 `fiberRoot` 和 `rootFiber` 建立起关联。

```ts
function createFiberRoot(): FiberRoot {
  // 创建fiberRoot
  const root: FiberRoot = (new FiberRootNode(
    containerInfo,
    tag,
    hydrate,
    identifierPrefix,
    onRecoverableError,
  ): any);
  // 创建rootFiber
  const uninitializedFiber = createHostRootFiber(
    tag,
    isStrictMode,
    concurrentUpdatesByDefaultOverride,
  );
  // 把两者关联起来
  root.current = uninitializedFiber;
  uninitializedFiber.stateNode = root;

  return root;
}
```

效果如下：
![](https://pic.imgdb.cn/item/624a183a239250f7c57c9e75.jpg)

> - `fiberRoot`：首次构建应用， 创建一个 `fiberRoot` ，作为整个 React 应用的根基，一个 React 应用只能有一个；
> - `rootFiber`：作为 fiber 树的根基，一般有两个，分别对应`workInProgress`树和`current`树

> `workInProgress`：正在内存中构建的 Fiber 树称为 `workInProgress` Fiber 树。在一次更新中，所有的更新都是发生在 `workInProgress` 树上。在一次更新之后，`workInProgress` 树上的状态是最新的状态，那么它将变成 `current` 树用于渲染视图。
> `current`：正在视图层渲染的树叫做 `current` 树。

workInProgress 通常就是一个 fiber 元素，类似于树的节点。调和过程中的每一个节点 fiber 都可以是 workInProgress，因此它通常指的是 workInProgress 树内的节点，并非准确的 rootFiber 节点。

2. 进入渲染流程，创建一个 `fiber` 作为 `workInProgress` （初始化的 `rootFiber` 没有 `alternate`），建立起`wip`树和`current`树之间的关系。

首先会复用当前 current 树（ rootFiber ）的 alternate 作为 workInProgress ，如果没有 alternate （初始化的 rootFiber 是没有 alternate ），那么会创建一个 fiber 作为 workInProgress 。
alternate 将新创建的 workInProgress 与 current 树建立起关联。这个关联过程只有初始化第一次创建 alternate 时候进行。

```js
currentFiber.alternate = workInProgressFiber;
workInProgressFiber.alternate = currentFiber;
```

![](https://pic.imgdb.cn/item/624a18eb239250f7c57e82d2.jpg)

3. 深度调和子节点，渲染视图；通过 fiber 的`return`等关系属性，完成整个 fiber 树的遍历，包括 fiber 的创建。

![](https://pic.imgdb.cn/item/624a198c239250f7c58053d5.jpg)

4. 将 fiberRoot.current 指针指向构建好的 wip 树，这样他就会变成 current 树并开始渲染到页面上。

### 更新

更新的步骤和初始化类似：

1. 走如上的逻辑，重新创建一颗 `workInProgresss` 树；创建时会复用当前 `current` 树上的 `alternate` ，作为新的 `workInProgress` 。判断复用哪些、怎么复用的过程就是`diff`过程。
2. 两棵树上的每个`fiber`节点都通过`alternate`建立联系；
3. 让`fiberRoot`的两个指针分别指向两棵树，此时就是有了完整的双缓冲树。

> React 用 `workInProgress` 树(内存中构建的树) 和 `current` (渲染树) 来实现更新逻辑。双缓存一个在内存中构建，一个渲染视图，两颗树用 `alternate` 指针相互指向，在下一次渲染的时候，直接复用缓存树做为下一次渲染树，上一次的渲染树又作为缓存树，这样可以防止只用一颗树更新状态的丢失的情况，又加快了 DOM 节点的替换与更新。

![](https://pic.imgdb.cn/item/624a1a6d239250f7c582e3f1.jpg)

# React 调和

## 调和的开始

当组件更新，本质上是从 fiberRoot 开始深度调和 fiber 树。在调度那部分讲过，调度的主要对象，即调和的入口函数是 performSyncWorkOnRoot
在调度的入口函数 ensureRootIsScheduled 中，调度的对象实际上就是 performSyncWorkOnRoot 函数。Sync 优先级的更新都会触发这个函数，也就是开始了调和过程

```js
function performSyncWorkOnRoot(root) {
  /* render 阶段 */
  let exitStatus = renderRootSync(root, lanes);
  /* commit 阶段 */
  commitRoot(root);
  /* 如果有其他的等待中的任务，那么继续更新 */
  ensureRootIsScheduled(root, now());
}
```

render 阶段和 commit 阶段都是从 FiberRoot 开始的。也就是说在这两个阶段内部的源码中绝大部分的`root`都表示 FiberRoot。

另外，React18 中还会根据情况开启并发渲染或同步渲染，判断的依据有两个：

- 本次调度的所有更新的最高优先级不等于 SyncLane 时才开启 concurrent。也就是说，假如有一个 SyncLane 级别的更新，那么就一定不会选到 Concurrent 模式
- shouldTimeSlice 变量，如下。

```js
function performConcurrentWorkOnRoot(root, didTimeout) {
  const shouldTimeSlice =
    !includesBlockingLane(root, lanes) &&
    !includesExpiredLane(root, lanes) &&
    (disableSchedulerTimeoutInWorkLoop || !didTimeout);
  let exitStatus = shouldTimeSlice
    ? renderRootConcurrent(root, lanes)
    : renderRootSync(root, lanes);
}
```

## render 阶段

render 阶段即 React 生命周期中的 render 阶段。这个阶段会实例化组件。在 render 函数执行之前都被叫做 render 阶段
上面讲的 performSyncWorkOnRoot 调用 renderRootSync，其实就是内部调用了 workLoopSync。

```js
function renderRootSync(root, lanes) {
  workLoopSync();
  /* workLoop完毕后，证明所有节点都遍历完毕，那么重置状态，进入 commit 阶段 */
  workInProgressRoot = null;
  workInProgressRootRenderLanes = NoLanes;
}
```

render 阶段会首先通过`workLoop`遍历一次 fiber 树，执行每个 fiber 对应的 Reconciler：

```js
function workLoopSync() {
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
  }
}

function performUnitOfWork() {
  next = beginWork(current, unitOfWork, renderLane);
  if (next === null) {
    next = completeUnitOfWork(unitOfWork);
  } else {
    workInProgress = next;
  }
}
```

### beginWork

beginWork 作用如下：

- 对于组件，执行部分生命周期，执行 render ，得到最新的 children （函数组件和类组件都是）。
- 向下遍历调和 children ，复用 oldFiber （diff 算法）。
- 给部分 fiber 打标记 flag，包括删除、移动、插入这三种（还有一种是更新，将在 completeWork 中打上），这些 flags 就是一种副作用，在 commit 阶段会根据这种副作用，对真实 dom 进行处理。

需要注意的是，beginWork 一般是 render 阶段的开始，但是一个 fiber 执行 beginWork，并不等同于执行了 render。

1. 所有类型的 fiber 都可以执行 beginWork，但是 render 只是针对于组件；具体来说，对于 HostComponent 或其他非组件 fiber，就不能像组件一样执行 renderWithHooks 这样的 render 函数，当然也不会主动触发更新、持有 context、state 等。不过他们也有特殊的处理函数（比如 updateHostComponent），还是会执行正常的调和、复用等步骤
2. 进入 beginWork 的组件也并不一定会 render。下面讲到的 childLanes 判断更新位置的方法，从根节点到 childLanes === renderLanes 的组件，中间经过的组件都会执行 beginWork，但不会 render；直到真正更新的组件及其后代才会执行 render

#### scheduleUpdateOnFiber 更新 fiber

首先有几个问题：

1. 父组件的更新为什么会连带所有子组件更新？
2. 组件更新任务的调度统一发生在 RootFiber 上，那是怎么确定具体在哪个组件更新呢？换句话说，即使知道对应的组件，又是靠什么方法搜索到组件的位置呢？如果是依靠遍历搜索，那时间复杂度就会大大增加。
3. 子组件更新会导致父组件进入 beginWork 吗，它的同级相邻组件呢？

以 state 更新为例，当我们调用一个组件的 setState 更新时，就会触发 scheduleUpdateOnFiber

```js
function dispatchAction(fiber, queue, action) {
  const lane = requestUpdateLane(fiber);
  scheduleUpdateOnFiber(fiber, lane, eventTime);
}
```

在 scheduleUpdateOnFiber 中有这么一个函数： markUpdateLaneFromFiberToRoot

```js
function scheduleUpdateOnFiber(fiber, lane) {
  /* 递归向上标记更新优先级 */
  const root = markUpdateLaneFromFiberToRoot(fiber, lane);
  if (root === null) return null;
  /* 如果当前 root 确定更新，那么会执行 ensureRootIsScheduled */
  ensureRootIsScheduled(root, eventTime);
}
```

这个函数的主要作用是给 fiber 打上优先级标记，即 lane。由于 scheduleUpdateOnFiber 是更新驱动的，也就是一次更新会调用一次 scheduleUpdateOnFiber。

markUpdateLaneFromFiberToRoot 的作用就是把 fiber 以及其所有父级链上的 childLane 都更新

> - lane ： 更新优先级。在一次更新任务中，将赋予给更新的 fiber 的一个更新优先级 lane，同时更新本身也有一个 lane
> - childLanes：表示当前组件 fiber 的 children 的更新优先级。如果当前 fiber 的 child 中有高优先级任务，那么当前 fiber 的 childLanes 等于 child 中的高优先级任务的优先级（总之就是和 child 对齐）

```js
function markUpdateLaneFromFiberToRoot(sourceFiber, lane) {
  /* 更新当前 fiber 上 */
  sourceFiber.lanes = mergeLanes(sourceFiber.lanes, lane);
  /* 更新缓存树上的 lanes */
  let alternate = sourceFiber.alternate;
  if (alternate !== null) alternate.lanes = mergeLanes(alternate.lanes, lane);
  /* 当前更新的 fiber */
  let node = sourceFiber;
  /* 找到返回父级 */
  let parent = sourceFiber.return;
  while (parent !== null) {
    /* TODO: 更新 childLanes 字段 */
    parent.childLanes = mergeLanes(parent.childLanes, lane);
    if (alternate !== null) {
      alternate.childLanes = mergeLanes(alternate.childLanes, lane);
    }
    /* 递归遍历更新 */
    node = parent;
    parent = parent.return;
  }
}
```

这里会一直向上标记知道 RootFiber 为止。标记完成后，就形成了一条从 root 开始的链路
childLanes 的关键作用，就在于可以通过查看某个组件的 childLanes，判断该组件的 child 中是否有更新任务。
如果一个组件 A 发生了更新，那么先向上递归更新父级链的 childLanes，接下来从 RootFiber 向下调和的时候，发现某个组件的 childLanes 等于当前正在处理的更新的优先级，那么说明它的 child 链上有新的更新任务，则会继续向下调和，反之退出调和流程。

childLanes 起到了一个“标识路径”的作用，如果一个节点的 childLanes 和当前正在处理的更新的 lanes 不相等，那么就不用继续向下寻找了，相当于对树的查找做了剪枝。
示意图如下：
![](https://pic.imgdb.cn/item/63a72baf08b6830163f68d71.jpg)

- 第一阶段是发生更新，那么产生一个更新优先级 lane 。
- 第二阶段向上标记 childLanes 过程。
- 第三阶段是向下调和过程

注意第三阶段时，A 也会被调和，原因是 A 和 B 是同级，如果父级元素调和，并且向下调和，那么父级的第一级子链上所有的 fiber 都会进入调和流程。
从 fiber 关系上看，Root 先调和的是 child 指针上的 A ，然后 A 会退出向下调和，接下来才是 sibling B，接下来 B 会向下调和，通过 childLanes 找到当事人 F，然后 F 会触发 render 更新。
也就是说，即使父元素、兄弟元素没有任何更新，他们依然会因为当前元素的更新而进入调和；
**调和过程并非 render 过程，调和过程有可能会触发 render（对函数组件来说就是执行函数组件），也有可能只是继续向下调和**。因此父元素、兄弟元素可能会调和，但并不会 render

#### performSyncWorkOnRoot ：调和入口

在下面的调度中多次提起了这个函数 performSyncWorkOnRoot，它是调度的入口函数，调度的对象基本都是它。之前的章节中介绍了调和的两大阶段 render 和 commit 都在这个函数中执行。
简化代码如下：

```js
function performSyncWorkOnRoot(root) {
  /* render 阶段 */
  let exitStatus = renderRootSync(root, lanes);
  /* commit 阶段 */
  commitRoot(root);
  /* 如果有其他的等待中的任务，那么继续更新，即继续调度过程 */
  ensureRootIsScheduled(root, now());
}
```

这个函数的参数 root 指的是 FiberRoot，即整个 fiber 树的唯一的根节点。

#### renderRootSync ：开始 render 流程

```js
function renderRootSync(root, lanes) {
  workLoopSync();
  /* workLoop完毕后，证明所有节点都遍历完毕，那么重置状态，进入 commit 阶段 */
  workInProgressRoot = null;
  workInProgressRootRenderLanes = NoLanes;
}
```

renderRootSync 核心功能：

- 执行 workLoopSync。
- workLoopSync 完毕后，证明所有节点都遍历完毕，那么重置状态，进入 commit 阶段。

这里是 workLoopSync 可以当做是上面说到的 workLoop，不过只针对于 legacy 模式。注意区分调和的 workLoop 和调度的 workLoop，后者是执行 taskQueue 的过程。

workLoop 的主要作用就是循环执行 performUnitOfWork ，一直到 workInProgress 为空。在循环过程中，wip 会被替换成 workInProgress.child，最后直到 fiber 树的叶子节点为止。
Concurrent 模式下会通过 shouldYield 来判断有没有过期的任务，如果有过期任务会中断 workLoop ，那么也就是说明了**render 阶段是可以被打断的**。

```js
function workLoopSync() {
  /* 循环执行 performUnitOfWork ，一直到 workInProgress 为空 */
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
  }
}

function workLoopConcurrent() {
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}
```

最后来到 performUnitOfWork，这个函数已经在上面说明过，就是执行 beginWork 和 completeUnitOfWork

#### beginWork：两个阶段

beginWork 源码在 17 和 18 中有出入，上面列出的是 v18 的源码，和 v17 的情况还有所不同，下面讲的是 v17 的。

```js
/**
 * @param {*} current         current 树 fiber
 * @param {*} workInProgress  workInProgress 树 fiber
 * @param {*} renderLanes     当前的 render 优先级
 * @returns
 */
function beginWork(current, workInProgress, renderLanes) {
  /* -------------------第一部分-------------------- */
  if (current !== null) {
    /* 更新流程 */
    /* current 树上上一次渲染后的 props */
    const oldProps = current.memoizedProps;
    /* workInProgress 树上这一次更新的 props  */
    const newProps = workInProgress.pendingProps;
    // 如果该fiber的props或者context改变
    if (oldProps !== newProps || hasLegacyContextChanged()) {
      // 这个变量是用于说明当前更新是不是来自父元素或者更上层的
      // 如果props改变或者context改变，显然这一项就是true
      didReceiveUpdate = true;
    } else {
      /* props 和 context 没有发生变化，检查是否更新来自自身或者 context 改变 */
      const hasScheduledUpdateOrContext = checkScheduledUpdateOrContext(
        current,
        renderLanes
      );
      // 判断更新是否来自自身，即比较当前fiber的lane和更新的lane
      if (!hasScheduledUpdateOrContext) {
        // 如果更新不是来自自身的
        // 那就说明更新来源于自己的后代，当前fiber不需要被更新。
        didReceiveUpdate = false;
        // 这个函数会判断后代
        return attemptEarlyBailoutIfNoScheduledUpdate(
          current,
          workInProgress,
          renderLanes
        );
      }
      /* 这里省略了一些判断逻辑 */
      didReceiveUpdate = false;
    }
  } else {
    didReceiveUpdate = false;
  }
  /* -------------------第二部分-------------------- */
  /* TODO: 走到这里流程会被调和 | 更新，比如函数执行会执行，类组件会执行 render 。 */
  switch (workInProgress.tag) {
    /* 函数组件的情况 */
    case FunctionComponent: {
      return updateFunctionComponent(
        current,
        workInProgress,
        Component,
        resolvedProps,
        renderLanes
      );
    }
    /* 类组件的情况 */
    case ClassComponent: {
      return updateClassComponent(
        current,
        workInProgress,
        Component,
        resolvedProps,
        renderLanes
      );
    }
    /* 元素类型 fiber <div>, <span>  */
    case HostComponent: {
      return updateHostComponent(current, workInProgress, renderLanes);
    }
    /* 其他 fiber 情况 */
  }
}
```

beginWork 的全流程可以分为两个阶段:

1. 判断更新情况。 didReceiveUpdate 这个变量主要证明当前更新是否来源于父级的更新。判断的主要依据是新老 props，

- 如果一个元素即使进入了调和，props 改变，证明当前更新来源于父级的更新，修改 didReceiveUpdate；
- 如果新老 props 相等，那就检查自身的 state 或消费的 context 有没有发生变化。检查的函数是 checkScheduledUpdateOrContext：

```js
function checkScheduledUpdateOrContext(current, renderLanes) {
  const updateLanes = current.lanes;
  /* 这种情况说明当前更新 */
  // 比较当前fiber的lane和当前更新的lane，如果相等就说明更新来自当前fiber
  if (includesSomeLane(updateLanes, renderLanes)) {
    return true;
  }
  /* 如果该 fiber 消费了 context ，并且 context 发生了改变。 */
  if (enableLazyContextPropagation) {
    const dependencies = current.dependencies;
    if (dependencies !== null && checkIfContextChanged(dependencies)) {
      return true;
    }
  }
  return false;
}
```

- 如果上面两个都不是，那就说明调和的启动来源于自己的后代，那么当前 fiber 不需要被更新。这时还会执行一个函数 attemptEarlyBailoutIfNoScheduledUpdate，里边会调用 bailoutOnAlreadyFinishedWork。后者很重要，主要作用是判断后代 fiber 是否需要被更新：

- 首先通判断 childLanes 是否是高优先级任务，如果不是，那么所有子孙 fiber 都不需要调和 ，那么直接返回 null，child 也不会被调和。
- 如果 childLanes 优先级高，那么证明 child 需要被调和，但是当前组件不需要，所以会克隆一下 children，返回 children ，那么本身不会 rerender。

```js
function bailoutOnAlreadyFinishedWork(current, workInProgress, renderLanes) {
  /* 如果 children 没有高优先级的任务，说明所有的 child 都没有更新，那么直接返回null，child 也不会被调和  */
  if (!includesSomeLane(renderLanes, workInProgress.childLanes)) {
    /* 这里做了流程简化 */
    return null;
  }
  /* 当前fiber没有更新。但是它的children 需要更新。  */
  cloneChildFibers(current, workInProgress);
  return workInProgress.child;
}
```

2. beginWork 的第二阶段，就是调用不同的函数更新，即上面代码的 Switch Case 部分。

以 updateFunctionComponent 为例，内部是这样的：

```ts
function updateFunctionComponent(
  current,
  workInProgress,
  Component,
  nextProps: any,
  renderLanes
) {
  let nextChildren;
  // renderWithHooks:执行函数组件，返回函数组件执行结果children
  nextChildren = renderWithHooks(
    current,
    workInProgress,
    Component,
    nextProps,
    context,
    renderLanes
  );

  workInProgress.flags |= PerformedWork;
  // 调和函数组件的children，把调和的结果安插在wip上
  reconcileChildren(current, workInProgress, nextChildren, renderLanes);
  return workInProgress.child; // 返回当前调和的fiber的子fiber，也就是nextChildren对应的fiber
}
```

从 renderWithHooks 获取到函数组件的返回值是一个 React element 对象，将它在 reconcileChildren 做处理，将其转化为 fiber 并挂载在当前 fiber 上。

reconcileChildren 是调和的核心，它内部会调用 Reconciler 进行 fiber 的创建和连接，以及 flags 的标记。
reconcliChildren 主要代码如下：

```ts
export function reconcileChildren(
  current: Fiber | null, // workInProgress.alternate
  workInProgress: Fiber,
  nextChildren: any, // 即函数组件的返回值，表示要构造成fiber的children
  renderLanes: Lanes
) {
  // current不存在，即现在是挂载阶段
  if (current === null) {
    workInProgress.child = mountChildFibers(
      workInProgress,
      null,
      nextChildren,
      renderLanes
    );
  } else {
    //更新阶段
    workInProgress.child = reconcileChildFibers(
      workInProgress,
      current.child,
      nextChildren,
      renderLanes
    );
  }
}
```

mountChildFibers 和 reconcileChildFibers 其实都是同一个函数 createChildReconciler，传入的参数不同而已；

```js
const reconcileChildFibers: ChildReconciler = createChildReconciler(true);
const mountChildFibers: ChildReconciler = createChildReconciler(false);
```

createChildReconciler 的作用就是将 element 转化为 fiber，再对 fiber 打上各种 flags，返回生成的 fiber。其中传入的参数 shouldTrackSideEffects，表示“是否追踪副作用”。
createChildReconciler 就是传说中的 Reconciler 本体。

#### Reconciler：调和的核心

createChildReconciler 函数实际上是创建并返回一个 Reconciler。Reconciler 并不是一个对象，而是一组函数和一个入口函数 reconcileChildFibers；通过调用 reconcileChildFibers，再调用这一组函数中的这些工具函数，可以实现主要的功能，即创建 fiber、进行标记、更新节点等。

createChildReconciler 代码如下：（v18）

```ts
function createChildReconciler(shouldTrackSideEffects): ChildReconciler {
  // 以这个为例，它的效果是向fiber上添加deletions的flag
  function deleteChild(returnFiber: Fiber, childToDelete: Fiber): void {
    // 每个方法都有这个判断。这个值为false则不添加任何flags，对应生成reconcileChildren在挂载阶段调用的mountChildFibers
    if (!shouldTrackSideEffects) {
      return;
    }
    // 添加deletions flag
    const deletions = returnFiber.deletions;
    if (deletions === null) {
      returnFiber.deletions = [childToDelete];
      returnFiber.flags |= ChildDeletion;
    } else {
      deletions.push(childToDelete);
    }
  }
  // 添加移动Placement标记
  function placeChild() {}

  // 更新节点的范例，这里是更新文本元素
  function updateTextNode(
    returnFiber: Fiber,
    current: Fiber | null,
    textContent: string,
    lanes: Lanes
  ) {
    if (current === null || current.tag !== HostText) {
      // 挂载阶段，创建节点fiber，设置return
      const created = createFiberFromText(textContent, returnFiber.mode, lanes);
      created.return = returnFiber;
      return created;
    } else {
      // 更新阶段
      const existing = useFiber(current, textContent);
      existing.return = returnFiber;
      return existing;
    }
  }
  // 更新元素，基本逻辑和上面更新文本节点类似
  function updateElement() {}
  // 多节点的diff算法
  function reconcileChildrenArray() {}
  // 单节点的diff算法
  function reconcileSingleElement() {}

  // 核心函数，返回的就是这个函数
  function reconcileChildFibers(
    returnFiber: Fiber, // 父fiber
    currentFirstChild: Fiber | null, // 对应current树的节点，即returnFiber.child.alternate
    newChild: any, // 组件要渲染的children，即函数组件的返回值
    lanes: Lanes
  ): Fiber | null {
    if (typeof newChild === "object" && newChild !== null) {
      switch (newChild.$$typeof) {
        case REACT_ELEMENT_TYPE:
          // 关键代码，创建元素并打上flags
          return placeSingleChild(
            reconcileSingleElement(
              returnFiber,
              currentFirstChild,
              newChild,
              lanes
            )
          );
      }
      // ...还有其他类型

      if (isArray(newChild)) {
        // 如果children是数组，就执行多节点diff算法
        return reconcileChildrenArray(
          returnFiber,
          currentFirstChild,
          newChild,
          lanes
        );
      }
    }
    // 在shouldTrackSideEffects为true时，删掉current树的对应的那个节点以及其全部兄弟节点
    // 即表示更新，更新后对应的current就应该被删掉了
    return deleteRemainingChildren(returnFiber, currentFirstChild);
  }
  return reconcileChildFibers;
}
```

内部的这些函数基本也可分为三类，即：

- 标记类，用于给 fiber 打上 Placement、Delection 等标记。这些标记会在 commit 阶段被识别，并对真实 dom 进行处理；如果没有标记，commit 阶段就不会操作真实 dom，因此就不会更新。标记非常重要
- 调和类，用于把 React element 变成 fiber，或者复用旧 fiber，总之就是返回一个有效的 fiber，后面会作为 workInProgress.child。比如 reconcileSingleElement 会创建组件 fiber，reconcileChildrenArray 会对数组类型的 React element 进行复用和新建的选择（即 diff 算法）
- 更新类，即`updatexxx`的函数，这类函数会判断是否是更新阶段（current != null），然后做不同的处理。初始化阶段会创建 fiber，更新阶段会比较 current 树上的对应节点选择复用。

> 更新类和调和类的功能近似，但是最大的区别是更新类只会出现在 reconcileChildrenArray 里，即 diff 算法内部，用作复用 current 节点。而调和类则是用于其他创建单个 fiber 的情况。

标记类的函数，比如 createChildReconciler 内部的 placeChild 方法：

```ts
function placeChild(
  newFiber: Fiber,
  lastPlacedIndex: number,
  newIndex: number
): number {
  newFiber.index = newIndex;
  if (!shouldTrackSideEffects) {
    // 不标记placement
    newFiber.flags |= Forked;
    return lastPlacedIndex;
  }
  // 标记placement
  newFiber.flags |= Placement | PlacementDEV;
  return lastPlacedIndex;
}
```

这里 flags，就是将在后面 completeUnitOfWork 中执行的任务，以及 commit 中进行对真实 dom 的操作。
标记也被称作是“副作用”，注意不同于 useEffect 的那个副作用，而是指 fiber 的副作用。

在 beginWork 阶段，可能的标记一共有 2 种：

- ChildDeletion 删除（子 fiber）
- Placement 移动

在后面的任务中，就会对被打上这些标记的 fiber 执行相应的操作，即实现更新对实际元素的的删除和移动操作。
并且只有在 reconcileChildFibers 函数内才会标记，即 shouldTrackSideEffects 的值为 true。
如果选择不标记，那么该 fiber 将不会被打上 Placement 的标记，在后面的 commit 中就不会把该元素执行“插入到页面上”的任务。

另外，还有一种标记是 Update 更新，将会在 completeWork 时被标记；

#### 打标记和真实操作

在 beginWork 阶段，react 并不会直接删除 fiber 对应的 dom 元素，也不会执行组件类型 fiber 的销毁流程，而只是打上 flags。所有具体 dom 的操作都是在 commit 阶段完成的。
也就是说，标记和真实操作是两个部分完成的，不要混淆。

我们以删除为例，可以看到 react 在调和阶段判断一个元素被删除时会在其父 fiber 打上 deletion 标记以及添加一个 deletions 数组：

```ts
function deleteChild(returnFiber: Fiber, childToDelete: Fiber): void {
  if (!shouldTrackSideEffects) {
    // Noop.
    return;
  }
  const deletions = returnFiber.deletions;
  if (deletions === null) {
    returnFiber.deletions = [childToDelete];
    returnFiber.flags |= ChildDeletion;
  } else {
    deletions.push(childToDelete);
  }
}
```

这个数组会在什么时候使用呢？在 commit 的 mutation_begin 阶段：

```ts
const deletions = parentFiber.deletions;
if (deletions !== null) {
  for (let i = 0; i < deletions.length; i++) {
    const childToDelete = deletions[i];
    try {
      commitDeletionEffects(root, parentFiber, childToDelete);
    } catch (error) {
      captureCommitPhaseError(childToDelete, parentFiber, error);
    }
  }
}
```

commitDeletionEffects 是一个负责删除的函数。这个函数内部的逻辑是：

- 如果 deletions[i]是一个 hostComponent 或 hostTextNode，就删除这个对应的 dom 元素，同时置空上面的 ref
- 如果 deletions[i]是一个其他类型的 fiber，比如组件会调用销毁组件的函数或生命周期

这个过程中，没有真正的 fiber 节点被“删除”，在 render 阶段被删除的 fiber 可以看做是“没有从 current 树中复用”。

比如更新前后的结构：

```js
<ul>
  <li key="1"/>
  <li key="2"/>
</ul>

// 更新后：
<ul>
  <li key="1" />
</ul>
```

在 diff 算法中，会比较得知 current 树上的`<li key="1"/>`这个对应的 fiber 是可以复用的，那么就会复用它，而对其他的节点不再进行复用。相当于就是，只复用了一个 fiber，其他 fiber 不做复制，也就相当于删除了 fiber。
然后，diff 算法为 ul 元素的 fiber.deletions 数组添加`<li key="2"/>`的 fiber，表示这个 fiber 对应的 dom 元素应该被删除。于是在 commit 阶段就会执行删除 dom 元素的操作。

### completeWork

completeUnitOfWork 会在 beginWork 返回 null 时调用，也就是当 beginWork 从上至下一直到了最底层的 fiber 之后，再由 completeUnitOfWork 向上返回。类似于树的 dfs 遍历，completeUnitOfWork 就是那个向上回溯的过程。
因此 completeUnitOfWork 和 beginWork 会夹杂着进行，对一个 fiber 节点来说，一定会同时经历两者的处理。

completeUnitOfWork 总体分为两个步骤：

1. 创建或更新真实 dom 元素。具体来说是：

- 创建：在整个组件树的 mount 阶段创建 dom 元素。对于类型为 dom 元素的 fiber，创建成为真实 dom 元素，然后把它的子孙 dom 元素插入它，并给 dom 元素添加属性（比如 styles、部分事件等）。等到 completeWork 执行完成后，已经形成了一颗完成的真实 dom 树
- 更新：在组件树的 update 阶段主要是更新 HostComponent 的属性，并打上 update 的 flag。所有的 HostComponent 的更新都会存在对应的 fiber.updateQueue 中，比如 style 值变化、className 值变化等等。

2. flags 冒泡

#### flags 冒泡

当执行完成 beginWork 之后，除了完成对 fiber 的从上至下遍历，还为部分 fiber 节点打上了 flags 标签（createChildReconciler）。
接下来的步骤需要对之前被标记的 fiber 执行操作，但是查找 flags 又成了一个问题。因此第一步就是让 flags 自己“冒泡”出来，方便统一收集。

冒泡依赖于 fiber.subtreeFlags 属性，这个属性用于收集该 fiber 的所有子孙 fiber 上被标记的 flags。
然后，每个 fiber 进行如下操作，将自己的 subtreeFlags 向上冒泡一层：

```js
// bubbleProperties函数
let subtreeFlags = NoFlags;

subtreeFlags |= child.subtreeFlags; // 收集当前fiber的子代fiber的子孙flags
subtreeFlags |= child.flags; // 收集当前fiber的子代fiber的flags
completeWork.subtreeFlags |= subtreeFlags; // 附加在当前遍历到的fiber的subtreeFlags属性上
```

按照此方法，可以把整个 fiber 树上所有被标记的 flags 都在 FibeRoot 上记录；
对于任意一个 fiber，只需要把自己的 flags 和 FiberRoot 上的 subtreeFlags 进行位运算，就得到自己的 subtreeFlags 值，从而确定是否有子树需要执行副作用。

---

在这种方式之前，React 采用的是 EffectList，即副作用链表的形式，将有副作用的 fiber 形成链表挂载在 FiberRoot 上。在 completeWork 阶段会检查每个 fiber 上是否有副作用，如果有就连接到 EffectList 上。在 commit 阶段直接执行所有的 effectList 上的节点即可。

在 completeWork 阶段产生了 fiber 上的 subtreeFlags 属性，在 commit 阶段执行具体操作时也会用到。commit 阶段会遍历每个 fiber 节点，检查 subtreeFlags，判断是否需要处理。

实际上，effectList 是一种更有效的方式。但 react18 需要兼容 suspense

#### completeWork

和 beginWork 类似，completeWork 也分成了 mount 和 update 两种处理，即更新阶段和非更新阶段的处理。

在 completeWork 函数内部，也是根据不同的 fibertag 做不同的操作；简化代码如下：

```ts
function completeWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes
): Fiber | null {
  const newProps = workInProgress.pendingProps;
  switch (workInProgress.tag) {
    case IndeterminateComponent:
    case LazyComponent:
    case SimpleMemoComponent:
    case FunctionComponent:
    case ForwardRef:
    case Fragment:
    case Mode:
    case Profiler:
    case ContextConsumer:
    case MemoComponent:
      // flags冒泡
      bubbleProperties(workInProgress);
      return null;
    // 对于dom类型的fiber，有特殊处理
    case HostComponent: {
      const type = workInProgress.type;
      if (current !== null && workInProgress.stateNode != null) {
        // current不为null，那就是更新阶段，调用更新
        updateHostComponent(current, workInProgress, type, newProps);
      } else {
        // 挂载阶段，执行下面的流程
        const rootContainerInstance = getRootHostContainer();
        const instance = createInstance(
          type,
          newProps,
          rootContainerInstance,
          currentHostContext,
          workInProgress
        );
        appendAllChildren(instance, workInProgress, false, false);
        workInProgress.stateNode = instance;
        // 处理属性
        finalizeInitialChildren(instance, type, newProps, currentHostContext);
      }
      // flags冒泡
      bubbleProperties(workInProgress);
      return null;
    }
  }
}
```

##### mount

对于类型为 HostComponent 的 fiber，completeWork 的**mount**步骤如下：

1. 首先通过 createInstance 创建 fiber 对应的真实 dom 元素。
2. 执行 appendAllChildren 方法。这个函数的主要作用是从当前 fiber.child 开始，按照`child->sibiling->return`的顺序遍历所有 fiberNode，将 fiber 类型为 dom 的 fiber 创建真实 dom 并插入到上一步创建的真实 dom 元素中

```ts
appendAllChildren = function (
  parent: Instance,
  workInProgress: Fiber,
  needsVisibilityToggle: boolean,
  isHidden: boolean
) {
  let node = workInProgress.child;
  while (node !== null) {
    if (node.tag === HostComponent || node.tag === HostText) {
      // 如果当前fiber的child是一个dom类型的元素，那就直接插入到上一步创建的dom上去
      appendInitialChild(parent, node.stateNode);
    } else if (node.child !== null) {
      // 向下遍历
      node.child.return = node;
      node = node.child;
      continue;
    }
    // 如果遍历到parent就终止，因为这个遍历是会回溯的，最终会在绕完一圈之后终止
    if (node === workInProgress) {
      return;
    }
    // 如果没有兄弟，就向父fiber遍历
    while (node.sibling === null) {
      // 如果没有父fiber或者父fiber就是parent就终止
      if (node.return === null || node.return === workInProgress) {
        return;
      }
      // 向父fiber倒序遍历
      node = node.return;
    }
    // 如果有兄弟，就对兄弟执行操作
    node.sibling.return = node.return;
    node = node.sibling;
  }
};
```

appendAllChildren 的插入顺序是`子->兄弟->父`。比如说这样一段结构：

```js
function App() {
  return (
    <div>
      'hello'
      <span>world</span>
    </div>
  );
}
```

在 appendChild 中的执行顺序为：

```
1. span: 因为是最后一个元素，child为空，退出
2. div: child为'hello'这个纯文本fiber
  1. div.appendChild('hello')
  2. 遍历到span，因为是'hello'的兄弟元素
  3. div.appendChild('span')
3. 回到div，div为App的child，退出
```

这一步执行完成后，真实 dom 结构已经建立。

3. 执行 finalizeInitialChildren 函数，对上一步创建好的 dom 设置属性，包括 styles、innerHTML、不会冒泡的事件等。
4. 冒泡 flags

##### update

上面的 updateHostComponent 函数就是在 completeWork 中对 HostComponent 的更新处理函数。mount 阶段已经完成了属性的初始化，在 update 阶段就负责这些属性的更新。

update 阶段处理的对象是 host 组件和 hostText 上的属性。他会比较 fiber 上的 props，如果发生更新就打上 Update 标记

updateHostComponent 的关键是 diffProperties 函数。这个函数负责对更新前后的属性进行 diff，一共两次遍历

- 第一次标记删除更新前有更新后没有的属性
- 第二次标记更新 update 前后属性值改变的属性

```ts
function diffProperties(
  updatePayload: null | Object,
  prevProps: Object,
  nextProps: Object,
  validAttributes: AttributeConfiguration
): null | Object {
  let attributeConfig;
  let nextProp;
  let prevProp;

  for (const propKey in nextProps) {
    prevProp = prevProps[propKey];
    nextProp = nextProps[propKey];

    if (prevProp === nextProp) {
      continue;
    }
  }

  for (const propKey in prevProps) {
    if (nextProps[propKey] !== undefined) {
      continue;
    }
    prevProp = prevProps[propKey];
    if (prevProp === undefined) {
      continue;
    }
  }
  return updatePayload;
}
```

完成之后，会把发生变动的属性挂载在对应 fiber 的 updateQueue 上去。

```ts
updateHostComponent = function(
    current: Fiber,
    workInProgress: Fiber,
    type: Type,
    newProps: Props,
  ) {
    // 比较新旧fiber上的props是否相同
    const oldProps = current.memoizedProps;
    if (oldProps === newProps) {
      return;
    }

    const instance: Instance = workInProgress.stateNode;
    const currentHostContext = getHostContext();

    const updatePayload = prepareUpdate(
      instance,
      type,
      oldProps,
      newProps,
      currentHostContext,
    );
    workInProgress.updateQueue = (updatePayload: any);
    // 如果更新，就给当前fiber打上update标记。
    if (updatePayload) {
      markUpdate(workInProgress);
    }
  };
```

以及，检查新旧 ref 是否相同，并 markRefz 也是在这里。

completeWork 主要是对 HostComponent 做处理，执行完成后已经创建了真实 dom，以及标记了 flags 的 workInProgress tree。接下来就到了 commit 阶段了。

## commit 阶段（React18）

commit 阶段会完成上一步 render 构建的 dom 树的最终“提交”，即会真正的更新 dom 节点。主要功能包括：

- 处理 render 阶段打上的标记 flags，对一些生命周期和副作用钩子做处理，比如 componentDidMount ，函数组件的 useEffect ，useLayoutEffect ；
- 另一方面就是在一次更新中，添加节点（ Placement ），更新节点（ Update ），删除节点（ Deletion ），还有就是一些细节的处理，比如 ref 的处理。

commit 细分可以分为：

1. Before mutation 阶段（执行 DOM 操作前）；也叫`pre-commit`阶段。因为 Before mutation 还没修改真实的 DOM ，是获取 DOM 快照的最佳时期，如果是类组件有 getSnapshotBeforeUpdate ，那么会执行这个生命周期。**会异步调用 useEffect** ，在生命周期章节讲到 useEffect 是采用异步调用的模式，其目的就是防止同步执行时阻塞浏览器做视图渲染
2. mutation 阶段（执行 DOM 操作），会进行的操作有：

- 对新增元素，更新元素，删除元素**进行真实的 DOM 操作**。

3. layout 阶段（执行 DOM 操作后），这个阶段 dom 已经更新完毕

- 会执行`componentDidMount`等生命周期和`useLayoutEffect`等钩子，以及 setState 的 callback

### commit 基础

在讲解具体内容之前，先了解一下 commit 阶段的一些基础内容

#### usexxxEffect 的执行

首先，commit 阶段有一个功能是执行函数组件的 usexxxEffect 钩子。一共有三种类型的 useEffect，按照执行时机顺序为：

- useInsertionEffect 是在 mutation 阶段执行的，虽然 mutation 是更新 DOM ，但是 useInsertionEffect 是在更新 DOM 之前 。
- useLayoutEffect 是在 layout 阶段执行，此时 DOM 已经更新了。
- useEffect 是在浏览器绘制之后，异步执行的。

如果在 commit 阶段的核心执行函数 commitRootImpl 插入 console.log 进行调试，就能看出来具体的关系：

```js
function commitRootImpl() {
  if (
    (finishedWork.subtreeFlags & PassiveMask) !== NoFlags ||
    (finishedWork.flags & PassiveMask) !== NoFlags
  ) {
    /* 通过异步的方式处理 useEffect  */
    scheduleCallback$1(NormalPriority, function () {
      flushPassiveEffects();
      return null;
    });
  }

  /* BeforeMutation 阶段执行 */
  const text = document.getElementById("text");
  console.log("-----BeforeMutation 执行-------");
  commitBeforeMutationEffects(root, finishedWork);
  console.log("-----BeforeMutation 执行完毕------");
  /* Mutation 阶段执行 */
  console.log("-----Mutation 执行-----");
  if (text) console.log("颜色获取：", window.getComputedStyle(text).color);
  commitMutationEffects(root, finishedWork, lanes);
  console.log("-----Mutation 执行完毕-----");
  if (text) console.log("颜色获取：", window.getComputedStyle(text).color);
  /* Layout 阶段执行 */
  console.log("-----Layout 执行-----");
  commitLayoutEffects(finishedWork, root, lanes);
  console.log("-----Layout 执行完毕-----");
}
```

![](https://pic.imgdb.cn/item/63a8327708b68301633f78ca.jpg)

#### 标志 flags

在前面的 beginWork 中，会给 fiber 树上的部分节点打上对应的标志 flags，这些标志就会在 commit 阶段被处理，对 dom 做具体的操作。
具体来说，更新标志主要有这几种：

```js
/* Before Mutation 阶段标志 */
var BeforeMutationMask =
  Update | //  类组件包含componentDidUpdate并且更新、dom类型组件属性变化、函数组件useLayoutEffect
  Snapshot; // 类组件包含getSnpaShotBuforeUpdate生命周期并且更新

/* Mutation 阶段标志 */
var MutationMask =
  Placement | //  当前fiber或后代fiber需要移动
  Update | //  更新
  ChildDeletion | // 子fiber要被删除（只有直接子代）
  ContentReset | //  清空文本类型节点
  Ref | // ref属性更新和创建
  Visibility; // Suspense组件中fallback和懒加载组件的显示与否

/* Layout 阶段标志 */
var LayoutMask =
  Update |
  Callback | // 指this.setState的第二个参数执行
  Ref | //
  Visibility; //

/* useEffect 阶段标志 */
var PassiveMask =
  Passive | // 函数组件是否定义了useEffect并且需要触发
  ChildDeletion; //
```

按照功能分类，又可以分为三种：

- 更新相关：
  - Update-组件更新标志，
  - Ref：处理绑定元素和组件实例，
- 元素相关：
  - Placement-插入元素
  - Update-更新元素
  - ChildDeletion-删除元素
  - Snapshot-元素快照
  - Visibility-offscreen 新特性
  - ContentReset-文本内容更新。
- 更新回调，执行 effect：
  - Callback-root 回调函数
  - 类组件回调
  - Passive-useEffect 的钩子函数
  - Layout-useLayoutEffect 的钩子函数
  - Insertion-useInsertionEffect 的钩子函数。

#### 三部分流程

commit 分为 before mutation、mutation 和 layout 三个阶段，这三个阶段的执行流程类似，都遵循`子->兄弟->父`的递归形式，入口函数也基本相同。
在进入流程之前，需要先检查当前 fiber 以及子树上是否包含三个阶段需要的标志；
比如检查当前 fiber.flags 和 fiber.subtreeFlags 是否包含 Update 或 Snapshot 标志，即`finishWork.flags & BeforeMutationMask === Noflags`；如果结果是 Noflags，那就说明当前 fiber 并不包含需要更新或快照的节点，也就是说不需要进行 Before Mutation 处理

然后进入流程，基本流程都是：

```js
function commitxxxEffects(root, firstChild) {
  /* root 为 fiberRoot,
  firstChild 为 render 阶段调和完毕的 fiber 节点，在commitWorkImpl中被设置为root.current.alternate，即wip树的根节点
  */
  // nextEffect表示正在处理的fiber
  nextEffect = firstChild;
  /* 开始进入 Before Mutation 流程 */
  commitxxxEffects_begin();
}

function commitxxxEffects_begin() {
  while (nextEffect !== null) {
    var fiber = nextEffect;
    var child = fiber.child;
    if ((fiber.subtreeFlags & xxxMask) !== NoFlags && child !== null) {
      /* 这里如果子代 fiber 树有 xxx 的标志，那么把 nextEffect 赋值给子代 fiber  */
      // 可以理解为向下递归
      nextEffect = child;
    } else {
      /* 找到最底层有 xxx 的标志的 fiber ，执行 complete */
      commitxxxEffects_complete();
    }
  }
}

function commitxxxEffects_complete(){
  while (nextEffect !== null) {
    var fiber = nextEffect;
    try{
      /* 真正的处理 xxx 需要做的事情。 */
      commitxxxEffectsOnFiber(fiber);
    }
    /* 优先处理兄弟节点上的 xxx  */
    var sibling = fiber.sibling;
    if (sibling !== null) {
      nextEffect = sibling;
      return;
    }
    /* 如果没有兄弟节点，那么返回父级节点，继续进行如上流程。 */
    nextEffect = fiber.return;
  }
}
```

上面的`xxx`只要替换成三个阶段的一个，就是三个阶段的实际执行函数，当然内部有不同阶段的独特处理。但是还是能看出来，三个阶段都有近似的流程，都是按照这个顺序来的
示意图：![](https://pic.imgdb.cn/item/63a85fe808b68301638b6765.jpg)

```
commitxxxEffects: 入口，初始化nextEffect为render执行完成后的fiber
-> commitxxxEffects_begin：从nextEffect向下到最底层有xxx标志的fiber或者到达叶子fiber
-> commitxxxEffects_complete：从上一步的fiber开始，先遍历全部兄弟，兄弟遍历完后再返回父节点，直到返回到顶层
-> commitxxxEffectsOnFiber：具体操作
```

### Before Mutation

上面的三部分流程，基本就是 Before Mutation 阶段的主要流程，就不细说了；主要说一下独特的执行部分，即 commitBeforeMutationEffectsOnFiber 内的内容

```js
function commitBeforeMutationEffectsOnFiber(finishedWork: Fiber) {
  const current = finishedWork.alternate;
  const flags = finishedWork.flags;

  switch (finishedWork.tag) {
    case FunctionComponent: {
      if (enableUseEffectEventHook) {
        if ((flags & Update) !== NoFlags) {
          commitUseEffectEventMount(finishedWork);
        }
      }
      break;
    }
    case ClassComponent: {
      if ((flags & Snapshot) !== NoFlags) {
        if (current !== null) {
          const prevProps = current.memoizedProps;
          const prevState = current.memoizedState;
          const instance = finishedWork.stateNode;
          // 下面两步就是执行getSnapshotBeforeUpdate获取snapshot，作为更新前的快照信息
          const snapshot = instance.getSnapshotBeforeUpdate(
            finishedWork.elementType === finishedWork.type
              ? prevProps
              : resolveDefaultProps(finishedWork.type, prevProps),
            prevState
          );
          instance.__reactInternalSnapshotBeforeUpdate = snapshot;
        }
      }
      break;
    }
    case HostRoot: {
      if ((flags & Snapshot) !== NoFlags) {
        if (supportsMutation) {
          // 清空HostRoot挂载的内容，方便Mutation阶段渲染
          const root = finishedWork.stateNode;
          clearContainer(root.containerInfo);
        }
      }
      break;
    }
  }
}
```

可以看出 Before Mutation 阶段的主要作用：

- 对于函数组件，调度 useEffect。但并不是立即执行 useEffect，而是启动一个异步任务去执行 useEffect
- 对于类组件，获取新旧 props，执行 getSnapshotBeforeUpdate。
- 清空 HostRoot，即 dom 元素挂载的根节点的内容，方便 mutation 阶段渲染。

### Mutation

Mutation 阶段切切实实地更新了 DOM 元素，这个阶段对于整个 commit 阶段起着举足轻重的作用。
当然，不仅是修改实际 dom 结构，也会修改相应的 fiber 结构。比如，commitDeletions 不仅会删除 dom 节点，还会删除对应的 fiber 节点。

#### commitMutationEffects_begin

Mutation 阶段的 begin 略有不同：

```js
function commitMutationEffects_begin(root, lanes) {
  while (nextEffect !== null) {
    var deletions = fiber.deletions;
    if (deletions !== null) {
      for (var i = 0; i < deletions.length; i++) {
        var childToDelete = deletions[i];
        commitDeletion(root, childToDelete, fiber);
      }
    }
    // 后面就是一样的套路
    if ((fiber.subtreeFlags & MutationMask) !== NoFlags && child !== null) {
      nextEffect = child;
    } else {
      commitMutationEffects_complete();
    }
  }
}
```

Mutation 在开始阶段就调用了 commitDeletion，这个函数会取出 fiber.deletions。在 beginWork 阶段，具体来说是 Reconciler 中的 deletChild 方法把要删除的元素加入到 deletions 数组中。因此 commitDeletion 就是删除这些 dom 元素。
删除一个 dom 元素要考虑的很多，包括但不仅限于：

- 卸载其子树所有组件
- 卸载自身的 ref 和后代 ref
- 执行各种卸载时调用的回调，比如 useEffect 返回的函数，或者 componentWillUnmount

commitDeletions 内部会根据 childToDelete 的类型进行不同的操作：

- 如果 fiber 类型是 HostComponent 或 HostText，就会调用 removeChild 卸载 dom 元素
- 如果是函数组件，则会调用 destroy 函数（即 useEffect 的返回值）
- 如果是类组件，则会调用 componentWillUnmount，并且会清空 ref

接下来进入 commitMutationEffects_complete，还是类似的遍历方式，然后对每个节点调用 commitMutationEffectsOnFiber

#### commitMutationEffectsOnFiber

commitMutationEffectsOnFiber 大致如下：

```js
function commitMutationEffectsOnFiber() {
  /* 如果是文本节点，那么重置节点内容 */
  if (flags & ContentReset) {
    commitResetTextContent(finishedWork);
  }
  /* 如果是 ref 更新，那么重置 alternate 属性上的 ref */
  if (flags & Ref) {
    const current = finishedWork.alternate;
    if (current !== null) {
      commitDetachRef(current);
    }
  }

  var primaryFlags = flags & (Placement | Update);
  switch (primaryFlags) {
    /* 如果新插入节点或移动节点 */
    case Placement: {
      commitPlacement(finishedWork);
      finishedWork.flags &= ~Placement;
      break;
    }
    /* ... 省去其他的相关逻辑 */
    /* 对于更新会有 Update */
    case Update: {
      const current = finishedWork.alternate;
      commitWork(current, finishedWork);
      break;
    }
  }
}
```

commitResetTextContent 的主要作用就是置空文本节点，commitDetachRef 也是置空 ref 的值。

commitPlacement 用于处理 Placement 操作；Placement 标记同时表示插入和移动，处理方式类似

```ts
function commitPlacement(finishedWork: Fiber): void {
  // 获取finishedWork的父fiber，并且是Host类型的祖先fiber（不一定是父）
  const parentFiber = getHostParentFiber(finishedWork);

  switch (parentFiber.tag) {
    case HostComponent: {
      // 从parentFiber获取到对应的dom元素
      const parent: Instance = parentFiber.stateNode;
      if (parentFiber.flags & ContentReset) {
        resetTextContent(parent);
        parentFiber.flags &= ~ContentReset;
      }
      // 获取下一个兄弟节点
      const before = getHostSibling(finishedWork);
      // 如果before存在，就用instertBefore插入/移动到before前面即可
      // 如果before不存在，就插入/移动到父节点后面
      insertOrAppendPlacementNode(finishedWork, before, parent);
      break;
    }
  }
}
```

getHostParentFiber 和 insertOrAppendPlacementNode 的遍历逻辑和 completeWork 中的 appendChild 逻辑近似。并且，getHostParentFiber 找到的 parentFiber 也必须是“稳定”的，即这个 parentFiber 不能也被标记 Placement。

---

接下来是更新的处理，即 commitWork

```ts
function commitWork(current, finishedWork) {
    switch (finishedWork.tag) {
        case FunctionComponent:
        case ForwardRef:
        case MemoComponent:
        case SimpleMemoComponent:
            /* 先执行上一次 useInsertionEffect 的 destroy */
            commitHookEffectListUnmount(Insertion | HasEffect, finishedWork, finishedWork.return);
            /* 执行 useInsertionEffect 的 create  */
            commitHookEffectListMount(Insertion | HasEffect, finishedWork);
        case HostComponent:
            const instance: Instance = finishedWork.stateNode;
          if (instance != null) {
            const newProps = finishedWork.memoizedProps;
            const oldProps = current !== null ? current.memoizedProps : newProps;
            const type = finishedWork.type;

            const updatePayload: null | UpdatePayload = (finishedWork.updateQueue: any);
            finishedWork.updateQueue = null;
            if (updatePayload !== null) {
              commitUpdate(
                instance,
                updatePayload,
                type,
                oldProps,
                newProps,
                finishedWork,
              );
            }
          }
    }
}
```

commitWork 的处理关键内容有两个，一个是对于函数组件的处理，另一个是对于 Host 组件（dom 类型）的处理。

- 对于函数组件：commitHookEffectListUnmount 是执行上一次 useInsertionEffect 的 destroy，commitHookEffectListMount 执行本次执行 useInsertionEffect 的 回调。这两个部分，其实就是执行了 useInsertionEffect 这个 hooks
- 对于 Host 组件：会调用 commitUpdate，这个函数会对 Host 组件对应的 dom 节点执行更新。前面在 completeWork 中，对于更新的处理方式就是把 props 的变动找出来放到 fiber 的 updateQueue 中，在这里就正是消费这个 updateQueue，并对实际 dom 元素进行更改，比如 style 属性、innerHTML 等。

```js
if (propKey === STYLE) {
  /* 更新 style 信息。 */
  setValueForStyles(domElement, propValue);
} else if (propKey === DANGEROUSLY_SET_INNER_HTML) {
  /* 更新 innerHTML 。 */
  setInnerHTML(domElement, propValue);
} else if (propKey === CHILDREN) {
  /* 更新 nodeValue 属性 */
  setTextContent(domElement, propValue);
} else {
  /* 更新元素的 props  */
  setValueForProperty(domElement, propKey, propValue, isCustomComponentTag);
}
```

### 切换 fiber 树

在 Mutation 和 Layout 之间，会执行如下代码切换 fiber 树：

```js
root.current = finishWork;
```

此时刚执行完 Mutation，因此 finishWork 就是 wip 树的根 fiber。
在这里切换之后，Layout 阶段访问到 fiber 都是 current；当在 layout 阶段执行某些内容时（比如 useLayoutEffect、componentDidUpdate），调用的 fiber 应该是已经构建好的才对。

### Layout

Mutation 阶段做了些真实的 DOM 操作，比如元素删除，元素更新，元素添加等操作，那么 layout 阶段，已经能够获取更新之后的 DOM 元素。
同样的流程，关键点还是在 commitLayoutEffectOnFiber

```js
function commitLayoutEffectOnFiber(
  finishedRoot,
  current,
  finishedWork,
  committedLanes
) {
  if ((finishedWork.flags & LayoutMask) !== NoFlags) {
    switch (finishedWork.tag) {
      /* 对于函数组件，执行  useLayoutEffect */
      case FunctionComponent:
      case ForwardRef:
      case SimpleMemoComponent:
        commitHookEffectListMount(Layout | HasEffect, finishedWork);
      /* 对于类组件，如果初始化会执行 componentDidMount，如果更新会执行 componentDidUpdate  */
      case ClassComponent:
        var instance = finishedWork.stateNode;
        if (finishedWork.flags & Update) {
          if (current === null) {
            /* 执行 componentDidMount 生命周期 */
            instance.componentDidMount();
          } else {
            /* 执行 componentDidUpdate 生命周期，最后一个参数是snapshot */
            instance.componentDidUpdate(
              prevProps,
              prevState,
              instance.__reactInternalSnapshotBeforeUpdate
            );
          }
        }
        var updateQueue = finishedWork.updateQueue;
        /* 如果有 setState 的 callback ，执行回调函数。 */
        if (updateQueue !== null) {
          commitUpdateQueue(finishedWork, updateQueue, instance);
        }
    }
    if (finishedWork.flags & Ref) {
      /* 更新 ref 属性 */
      commitAttachRef(finishedWork);
    }
  }
}
```

commitLayoutEffectOnFiber 做了非常重要的事：

1. 首先对于函数组件，执行 useLayoutEffect。
2. 对于类组件，如果初始化会执行 componentDidMount，如果更新会执行 componentDidUpdate。如果类组件触发 setState 并且有第二个参数 callback，那么这些 callback 会被放进 updateQueue 中，那么接下来会通过 commitUpdateQueue 执行每个 callback 回调函数。
3. 接下来会更新 ref 属性。

整个 Layout 阶段就结束了，Layout 阶段主要是执行回调函数，比如 setState 的 callback 和生命周期等，还有比如 useLayoutEffect 的钩子就是在这里执行 。

### useEffect

useEffect 是异步处理的，即在执行完上面三个步骤之后异步开启执行 useEffect 的函数。

对于 useEffect 处理，主要在 commitRootImpl 开始的时候通过 flushPassiveEffects 来执行。
下面就是从 commitRootImpl 截取的代码，可以看到这里用 scheduleCallback 调度了一次异步执行 flushPassiveEffects

```ts
if (
  (finishedWork.subtreeFlags & PassiveMask) !== NoFlags ||
  (finishedWork.flags & PassiveMask) !== NoFlags
) {
  if (!rootDoesHavePassiveEffects) {
    rootDoesHavePassiveEffects = true;
    pendingPassiveEffectsRemainingLanes = remainingLanes;
    pendingPassiveTransitions = transitions;
    scheduleCallback(NormalSchedulerPriority, () => {
      flushPassiveEffects();
      return null;
    });
  }
}
```

> 另外，为了保证下一次 commit 之前所有的 effect 都被执行完，commitRootImpl 还会在一开始同步调用一次 flushPassiveEffects
>
> ```ts
> do {
>   flushPassiveEffects();
> } while (rootWithPendingPassiveEffects !== null);
> ```
>
> 之所以在循环中，是因为 flushPassiveEffects 内部调用的 flushSyncCallback 会执行 syncQueue 中的任务，这些任务可能还会再标记一些 HasEffect。因此要持续循环，保证所有的 effect 执行完。

而 flushPassiveEffects 经过几次跳转，最终会来到 flushPassiveEffectsImpl，执行这两个函数：

```ts
commitPassiveUnmountEffects(root.current);
commitPassiveMountEffects(root, root.current, lanes, transitions);
```

这里就是执行了 useEffect 的地方。
commitPassiveMountEffects 函数最后会执行 commitHookEffectListMount，这个函数内部很简单，就是取出 fiber 上挂载的 updateQueue 的 lastEffect，lastEffect 是 effect hooks 创建的 effect 链表的头结点
接下来遍历链表，并检查 effect 的 flag。对于 useEffect 来说，flags 需要满足 `HookPassive | HookHasEffect`（前面的表示 useEffect 而不是 layout。insertion 等其他，后面的表示由于 deps 改变或初始化，要执行 create）

```ts
function commitHookEffectListMount(flags: HookFlags, finishedWork: Fiber) {
  const updateQueue: FunctionComponentUpdateQueue | null = (finishedWork.updateQueue: any);
  const lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
  if (lastEffect !== null) {
    const firstEffect = lastEffect.next;
    let effect = firstEffect;
    do {
      if ((effect.tag & flags) === flags) {
        const create = effect.create
        effect.destroy = create();
      }
      effect = effect.next;
    } while (effect !== firstEffect);
  }
}
```

# React diff

## diff 源码

单节点 diff 函数为 reconcileSingleElement，代码如下：

```ts
function reconcileSingleElement(
  returnFiber: Fiber,
  currentFirstChild: Fiber | null,
  element: ReactElement,
  lanes: Lanes
): Fiber {
  const key = element.key;
  let child = currentFirstChild;
  while (child !== null) {
    // 先比较key
    if (child.key === key) {
      const elementType = element.type;
      // 然后比较type
      if (
        child.elementType === elementType ||
        (typeof elementType === "object" &&
          elementType !== null &&
          elementType.$$typeof === REACT_LAZY_TYPE &&
          resolveLazy(elementType) === child.type)
      ) {
        // 标记删除current fiber的兄弟节点
        deleteRemainingChildren(returnFiber, child.sibling);
        // 复用current fiber创建wip fiber
        const existing = useFiber(child, element.props);
        // 复用ref
        existing.ref = coerceRef(returnFiber, child, element);
        existing.return = returnFiber;
        return existing;
      }
      // 剩余的current都不能复用，全删除
      deleteRemainingChildren(returnFiber, child);
      break;
    } else {
      // key不相同，一定不能复用，删除
      deleteChild(returnFiber, child);
    }
    child = child.sibling;
  }
  const created = createFiberFromElement(element, returnFiber.mode, lanes);
  created.ref = coerceRef(returnFiber, currentFirstChild, element);
  created.return = returnFiber;
  return created;
}
```

diff 算法用于处理类型为数组的 children（多节点）部分的代码如下：

```ts
function reconcileChildrenArray(
  returnFiber: Fiber,
  currentFirstChild: Fiber | null,
  newChildren: Array<any>,
  lanes: Lanes
): Fiber | null {
  let resultingFirstChild: Fiber | null = null;
  let previousNewFiber: Fiber | null = null;

  let oldFiber = currentFirstChild;
  let lastPlacedIndex = 0; // 最后一个可复用节点在 oldFiber 中的位置，也就是第一次遍历停下来后指针在oldFiber中的位置
  let newIdx = 0; // 在newFiber中的指针
  let nextOldFiber = null; // 指向下一个oldFiber

  // 第一次遍历，终止条件是所有新节点遍历完或旧节点不存在
  for (; oldFiber !== null && newIdx < newChildren.length; newIdx++) {
    // 如果oldFiber的序号比当前newFiber还要多，说明oldFiber多了，将oldFiber设为null，下一次就跳出循环
    if (oldFiber.index > newIdx) {
      nextOldFiber = oldFiber;
      oldFiber = null;
    } else {
      // 取下一个oldFiber
      nextOldFiber = oldFiber.sibling;
    }
    // updateSlot 内部会判断当前的 tag 和 key 是否匹配，如果匹配，复用老 fiber 形成新的 fiber
    // 如果不匹配，返回 null ，此时 newFiber 等于 null 。
    const newFiber = updateSlot(
      returnFiber,
      oldFiber,
      newChildren[newIdx],
      lanes
    );
    // 如果两个都走到空那就直接break
    if (newFiber === null) {
      if (oldFiber === null) {
        oldFiber = nextOldFiber;
      }
      break;
    }
    // 如果是处于更新流程，找到与新节点对应的老 fiber
    // 但是如果不能复用，即 alternate === null ，那么会删除老 fiber 。
    if (shouldTrackSideEffects) {
      if (oldFiber && newFiber.alternate === null) {
        deleteChild(returnFiber, oldFiber);
      }
    }
    // lastPlacedIndex表示为最后停下来的在oldFiber中的位置
    lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
    // previousNewFiber表示newFiber中最后一个能复用的newFiber，也就是lastPlacedIndex的前一个
    if (previousNewFiber === null) {
      resultingFirstChild = newFiber;
    } else {
      previousNewFiber.sibling = newFiber;
    }
    previousNewFiber = newFiber;
    oldFiber = nextOldFiber;
  }

  // 当第一次循环结束时有两种情况：
  // 1. newFiber遍历完，但oldFiber !== null：oldFiber多余，删除多余的oldFiber（下面第一个if）
  // 2. oldFiber === null，newFiber没遍历完：oldFiber不足，直接创建新的newFiber（下面第二个if）
  // 3. newFiber遍历完，并且oldFiber也遍历完，或者是他俩都没遍历完：这时说明是发生了移动或者更复杂的情况，进入下一次循环（下面的for）

  // 走到这里说明newFiber遍历完
  // 如果进入下面的if说明所有newFiber都被遍历完，但oldFiber还有，那么剩下没有遍历 oldFiber 也就没有用了，
  // 调用 deleteRemainingChildren 统一删除多余的 oldFiber 。
  if (newIdx === newChildren.length) {
    deleteRemainingChildren(returnFiber, oldFiber);
    return resultingFirstChild;
  }

  // 当经历过第一步，oldFiber 为 null ， 证明 oldFiber 复用完毕
  // 如果还有新的 children ，说明都是新的元素，只需要调用 createChild 创建新的 fiber 。
  if (oldFiber === null) {
    for (; newIdx < newChildren.length; newIdx++) {
      const newFiber = createChild(returnFiber, newChildren[newIdx], lanes);
      if (newFiber === null) {
        continue;
      }
      lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
      if (previousNewFiber === null) {
        resultingFirstChild = newFiber;
      } else {
        previousNewFiber.sibling = newFiber;
      }
      previousNewFiber = newFiber;
    }
    return resultingFirstChild;
  }

  // 如果走到这里，说明节点发生移动，或者是移动+删除+增加等多种复杂情况，而不只是单纯的增加或删除
  // existingChildren里存放剩余的老的 fiber 和对应的 key (或 index )的映射关系。键是key，值是oldFiber
  const existingChildren = mapRemainingChildren(returnFiber, oldFiber);

  for (; newIdx < newChildren.length; newIdx++) {
    // 判断 existingChildren 中有没有可以复用 oldFiber
    // 如果有，那么复用，如果没有，新创建一个 newFiber 。
    const newFiber = updateFromMap(
      existingChildren,
      returnFiber,
      newIdx,
      newChildren[newIdx],
      lanes
    );
    if (newFiber !== null) {
      if (shouldTrackSideEffects) {
        if (newFiber.alternate !== null) {
          // 复用的 oldFiber 会从 existingChildren 删掉
          existingChildren.delete(
            newFiber.key === null ? newIdx : newFiber.key
          );
        }
      }
      lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
      if (previousNewFiber === null) {
        resultingFirstChild = newFiber;
      } else {
        previousNewFiber.sibling = newFiber;
      }
      previousNewFiber = newFiber;
    }
  }
  // 删除剩余没有复用的oldFiber
  if (shouldTrackSideEffects) {
    existingChildren.forEach((child) => deleteChild(returnFiber, child));
  }
  // 这一步执行完成后，newFiber要么是在第一次遍历中就复用，要么是在第二次遍历中复用，或者在第二次遍历中被创建；总之newFiber一定是完成构建了的。
  // oldFiber要么在第一次遍历被直接复用，要么在第二次遍历被从map中找出并删掉，要么在上面作为多余的被删掉；总之oldFiber一定没有多余的。

  if (getIsHydrating()) {
    const numberOfForks = newIdx;
    pushTreeFork(returnFiber, numberOfForks);
  }
  return resultingFirstChild;
}
```

基本流程已经写在注释里了。
除此之外还有注意点：

- 第一次遍历判断能不能复用的 updateSlot 函数，其实是比较新老 fiber 的 key。(就只是 key，没有其他别的属性)。因此 React 判断能不能复用的首要属性就是 key

## 单节点 Diff

单节点 diff 针对的是在更新后只有一个节点的 fiber，即并非是数组类型的。

单节点 diff 针对单个组件类型、html 元素类型以及一些 react 特殊组件（比如 fragment），不包括字符串（文本类型）、数组（多节点 diff）等。即使有多个相同的 element，依然还是被视作单节点，然后按照单节点 diff 一个一个处理。因此，这里探讨的过程实际上是针对一个节点的。

先来看一下源码：

```ts
function reconcileSingleElement(
  returnFiber: Fiber,
  currentFirstChild: Fiber | null,
  element: ReactElement,
  lanes: Lanes
): Fiber {
  const key = element.key;
  let child = currentFirstChild;

  // 如果child为null，说明是挂载阶段，直接创建
  while (child !== null) {
    // 首先比较key
    if (child.key === key) {
      const elementType = element.type;
      if (elementType === REACT_FRAGMENT_TYPE) {
      } else {
        if (child.elementType === elementType) {
          // key相同比较type
          // 如果type也相同，那就复用这个了，其他的兄弟节点不用再考虑。
          deleteRemainingChildren(returnFiber, child.sibling);

          const existing = useFiber(child, element.props);
          existing.ref = coerceRef(returnFiber, child, element);
          existing.return = returnFiber;
          return existing;
        }
      }
      // 如果key相同但type不同，还是删掉所有兄弟节点
      // 因为其他节点一定是key和type都不同的，没有意义
      deleteRemainingChildren(returnFiber, child);
      break;
    } else {
      // 不能复用，删掉对应current树中的fiber
      deleteChild(returnFiber, child);
    }
    // 循环，向当前节点对应的current树中的那个节点的兄弟节点循环
    child = child.sibling;
  }

  // 创建当前fiber
  if (element.type === REACT_FRAGMENT_TYPE) {
  } else {
    const created = createFiberFromElement(element, returnFiber.mode, lanes);
    created.ref = coerceRef(returnFiber, currentFirstChild, element);
    created.return = returnFiber;
    return created;
  }
}
```

这里有一个 while 循环，主要目的是检查 current 树上的兄弟节点能不能用。也就是说，这个过程实际上是从 current 树中**一堆兄弟节点**中挑出来**一个**可以复用的，然后把其他的删掉

确定 dom 节点是否可以复用需要经过几个步骤：

首先：比较 key

- key 不同则直接将该 fiber 标记为删除

这点很好理解。如果 key 不相同，那就完全不能复用，直接将 current fiber 标记为删除就可以了

key 相同会比较 type，即节点类型（tag）

- type 相同则可复用，同时也会删除被复用的 currentfiber 的所有兄弟节点

之所以删除兄弟节点，其实出于这样的考虑：
比如更新前的节点数量有三个，更新后只有一个：

```ts
<ul>
  <li key='1'></li>
  <li key='2'></li>
  <li key='3'></li>
</ul>

<ul>
  <li key='1'></li>
</ul>
```

更新后由于只有一个节点，因此会进入单节点 diff。但显然此时只能复用一个，多余的就应该删除。

- type 不同则将该 fiber 及其兄弟 fiber 标记为删除。因为 key 相同的那一个已经不能复用，它的兄弟自然也就没什么可以复用的必要了

## 多节点 diff

主要是针对列表渲染，比如

```js
function List() {
  return (
    <ul>
      <li key="0">0</li>
      <li key="1">1</li>
      <li key="2">2</li>
      <li key="3">3</li>
    </ul>
  );
}
```

他的返回值 JSX 对象的 children 属性不是单一节点，而是包含四个对象的数组
这时会对节点进行两轮遍历

- 第一轮遍历：处理更新的节点，更新包括增删以及属性变化；
- 第二轮遍历：处理剩下的不属于更新的节点，即移动节点。

### 第一轮遍历

1. 遍历`children[i]`，分别和 `oldFiber` 对应节点比较，比较方式等同单节点的比较；

- 可复用，则 i++，继续下一个
- 不可复用：
  - key 不同导致不可复用，立即跳出整个遍历，第一轮遍历结束。
  - key 相同 type 不同导致不可复用，会将 `oldFiber` 标记为 `DELETION`，并继续遍历

2. 第一轮遍历的结果会有 4 种：

- `newChildren` 与 `oldFiber` 同时遍历完：直接结束，只需要在第一轮更新即可

```js
//oldFiber
<li key="0">0</li>
<li key="1">1</li>
<li key="2">2</li>
<li key="3">3</li>
//newChildren
<li key="0">4</li>
<li key="1">5</li>
<li key="2">6</li>
<li key="3">7</li>
```

- `newChildren` 没遍历完，`oldFiber` 遍历完：已有的 DOM 节点都复用了，这时还有新加入的节点，只需要遍历剩下的 `newChildren` 为生成的 `workInProgress fiber` 依次标记 `Placement`。

```js
//oldFiber
<li key="0">0</li>
<li key="1">1</li>
<li key="2">2</li>
<li key="3">3</li>
//newChildren
<li key="0">4</li>
<li key="1">5</li>
<li key="2">6</li>
<li key="3">7</li>
<li key="4">8</li>//新节点，创建新的fiber
```

- `newChildren` 遍历完，`oldFiber` 没遍历完：节点数量少，有节点被删除了。所以需要遍历剩下的 `oldFiber`，依次标记 `Deletion`

```js
//oldFiber
<li key="0">0</li>
<li key="1">1</li>
<li key="2">2</li>
<li key="3">3</li>
//newChildren
<li key="0">4</li>
<li key="1">5</li>
<li key="2">6</li>
```

- `newChildren` 与 `oldFiber` 都没遍历完

```js
//oldFiber
<li key="0">0</li>
<li key="1">1</li>
<li key="2">2</li>
<li key="3">3</li>
//newChildren
<li key="0">4</li>
<li key="2">5</li>//这里发现和oldFiber的key不对应，不可复用，这时两者都没遍历完
<li key="1">6</li>
```

对于第四种情况还有额外的处理：

### 第二轮遍历

> 关于第二轮的操作，我发现了一个不太一样的版本。
>
> 因为进入第二次遍历就一定是有移动的，可能只是移动，也可能是移动、更新、删除等多种情况的复合。但不管什么样，都是利用 map；
>
> 这时的操作就是用一个 map 记录 oldFiber 中的节点，然后用 newChild 里剩下的元素和 map 中的一一对应。
>
> - 如果 map 中有就复用（无所谓顺序），并删掉 map 中的这个元素
> - 如果 map 中没有就新创建
>
> 最后就能正确生成新的 fiber 节点。

第二轮遍历主要处理 key 不同的元素，即有可能产生移动的元素

1. 以最后一个可复用节点在 `oldFiber` 中的位置为 `lastIndex`，在 `newChild` 中第一个不可复用的为 `lasindex+1`

```
<li key=0> --- <li key=0>
<li key=1> --- <li key=1>//last index = 1
<li key=2> --- <li key=3>//last index + 1 = 2
<li key=3> --- <li key=2>
```

2. 选出 `newChildren` 在 `lastIndex+1` 的元素，即上次比较终止的下一个，在 `oldFiber` 中找，并取出序号和 `lastIndex` 比较。比如上面的例子，`key=3` 在 `oldFiber` 中找到，index 为 3，规定这个值为 `oldIndex`
3. 比较 `oldIndex` 和 `lastIndex`

- 如果 `oldIndex` >= `lastIndex` 代表该可复用节点不需要移动，并将 `lastIndex = oldIndex`;上面的例子 `oldindex === 3 > lastindex === 1`，因此 `oldIndex` 中 `key=3` 的元素位置不动（指相对于 old 不动，仍然在最后一个）
- 如果 `oldIndex` < `lastIndex` 该可复用节点之前插入的位置索引小于这次更新需要插入的位置索引，代表该节点需要向右（后）移动。

所以比较完成后，相当于只是把 `key=2` 的向后移动了一位，完成遍历

这里还有一个更清晰的例子：

```
// 之前
abcd

// 之后
dabc

===第一轮遍历开始===
d（之后）vs a（之前）
key改变，不能复用，跳出遍历
===第一轮遍历结束===

===第二轮遍历开始===
newChildren === dabc，没用完，不需要执行删除旧节点
oldFiber === abcd，没用完，不需要执行插入新节点

将剩余oldFiber（abcd）保存为map

继续遍历剩余newChildren

// 当前oldFiber：abcd
// 当前newChildren dabc

key === d 在 oldFiber中存在
const oldIndex = d（之前）.index;
此时 oldIndex === 3; // 之前节点为 abcd，所以d.index === 3
比较 oldIndex 与 lastPlacedIndex;
oldIndex 3 > lastPlacedIndex 0
则 lastPlacedIndex = 3;
d节点位置不变

继续遍历剩余newChildren

// 当前oldFiber：abc
// 当前newChildren abc

key === a 在 oldFiber中存在
const oldIndex = a（之前）.index; // 之前节点为 abcd，所以a.index === 0
此时 oldIndex === 0;
比较 oldIndex 与 lastPlacedIndex;
oldIndex 0 < lastPlacedIndex 3
则 a节点需要向右移动

继续遍历剩余newChildren

// 当前oldFiber：bc
// 当前newChildren bc

key === b 在 oldFiber中存在
const oldIndex = b（之前）.index; // 之前节点为 abcd，所以b.index === 1
此时 oldIndex === 1;
比较 oldIndex 与 lastPlacedIndex;
oldIndex 1 < lastPlacedIndex 3
则 b节点需要向右移动

继续遍历剩余newChildren

// 当前oldFiber：c
// 当前newChildren c

key === c 在 oldFiber中存在
const oldIndex = c（之前）.index; // 之前节点为 abcd，所以c.index === 2
此时 oldIndex === 2;
比较 oldIndex 与 lastPlacedIndex;
oldIndex 2 < lastPlacedIndex 3
则 c节点需要向右移动

===第二轮遍历结束===
```

## diff 和 index key

经过上面的解释可以得知，diff 算法依赖显式的 key 判断哪些元素发生什么样的更改。
如果 key 使用不规范（通常出现于用 index 代替 key），则会导致 diff 算法无法判断具体是哪个元素的变化，可能会导致需要重新渲染整个列表（完全不复用），甚至导致列表渲染错误。

比如这样一组元素：

```html
<ul>
  <li>a</li>
  <!-- key==0 -->
  <li>b</li>
  <!-- key==1 -->
  <li>c</li>
  <!-- key==2 -->
  <li>d</li>
  <!-- key==3 -->
  <li>e</li>
  <!-- key==4 -->
</ul>
```

如果使用 index 作为 key，当删除第一个元素时会变成这样：

```html
<ul>
  <li>b</li>
  <!-- key==0 -->
  <li>c</li>
  <!-- key==1 -->
  <li>d</li>
  <!-- key==2 -->
  <li>e</li>
  <!-- key==3 -->
</ul>
```

这个时候，**每个 li 元素的 key 都改变了**，相当于 React 判断所有元素都不能复用，直接重新渲染。

另外一种情况，如果因为某些 bug 导致每个 key 都一样，那么删除一个元素时会导致 React 不清楚是哪一个元素具体被删除，也可能导致完全重新渲染，或者删除错误的元素。

# React 调度

由于对于大型的 React 应用，会存在一次更新，递归遍历大量的虚拟 DOM ，造成占用 js 线程，使得浏览器没有时间去做一些动画效果，伴随项目越来越大，项目会越来越卡。因此 React 提出了一个异步调度的概念，即通过浏览器的控制，将一次更新任务分解到不同的空闲时间，如果浏览器有绘制任务那么执行绘制任务，在空闲时间执行更新任务即可。

## 时间分片

浏览器每次执行一次事件循环（一帧）都会做如下事情：处理事件，执行 js ，调用 requestAnimation ，布局 Layout ，绘制 Paint ，在一帧执行后，如果没有其他事件，那么浏览器会进入休息时间，那么有的一些不是特别紧急 React 更新，就可以执行了。

React 通过自己构建一个类似`requestIdleCallback`的方式，将任务分配到浏览器有空的时候再去执行。
为了防止浏览器没有充足空闲时间而导致卡死，React 为不同任务分配不同的**优先级**。
优先级意味着不同时长的任务过期时间。如果一个任务的优先级是 ImmediatePriority，对应 IMMEDIATE_PRIORITY_TIMEOUT 为-1，那么该任务的过期时间比当前时间还短，表示他已经过期了，需要立即被执行。

- `Immediate -1` 需要立刻执行。
- `UserBlocking 250ms` 超时时间 250ms，一般指的是用户交互。
- `Normal 5000ms` 超时时间 5s，不需要直观立即变化的任务，比如网络请求。
- `Low 10000ms` 超时时间 10s，肯定要执行的任务，但是可以放在最后处理。
- `Idle` 一些没有必要的任务，可能不会执行。

React 实际上通过的是`MessageChannel`完成时间分片。它是一个宏任务，但是不像`setTimeout`强制最短调度时间为 4ms，这个宏任务可以保证在 1ms 内触发，减少了时间浪费。

> MessageChannel 接口允许开发者创建一个新的消息通道，并通过它的两个 MessagePort 属性发送数据。
> 开发者可以创建两个 port，在两个 port 之间发送消息，并分别用 onmessage 接收
> `MessageChannel`的基本原理实际上是利用了回调函数的特性。回调和 setTimeout 都属于宏任务，但是却能保证立即触发。当一个任务需要执行时，将任务回调赋给一个全局变量并触发回调，回调执行时就是调度该任务开启的时间，也就做到了异步调度。

```js
let scheduledHostCallback = null;
/* 建立一个消息通道 */
const channel = new MessageChannel();
/* 建立一个port发送消息 */

channel.port1.onmessage = function () {
  /* 执行任务 */
  scheduledHostCallback();
  /* 执行完毕，清空任务 */
  scheduledHostCallback = null;
};
/* 向浏览器请求执行更新任务 */
requestHostCallback = function (callback) {
  scheduledHostCallback = callback;
  if (!isMessageLoopRunning) {
    isMessageLoopRunning = true;
    channel.port2.postMessage(null);
  }
};
```

如果一个任务执行期间时间片完，那么 react 就会终止任务执行，转而去交还线程给浏览器渲染线程。这个过程体现在 schduler 阶段的 shouldYield 函数中断 taskQueue 循环执行（下面会讲到）；如果中断时 taskQueue 中还有任务，那就会再通过 MessageChannel 注册一个宏任务去执行剩下的 taskQueue 任务。
从这个角度来说，就相当于是 js 代码和浏览器渲染相互协作，js 代码被拆成多段，一旦时间片完就交出线程控制权，否则会持续执行。两者关系类似下图：
![](https://pic.imgdb.cn/item/63ee7075f144a010074e492a.jpg)

我们可以把这种调度过程看作是这种形式：

```js
function scheduler() {
  do {
    workLoop();
  } while (shouleYield() || taskFinished);

  if (state === finished) {
    // 执行完任务
  } else {
    // 没执行完，异步安排下一次任务
    requestIdleCallback(count);
  }
}

scheduler();
```

MessageChannel 的选择主要基于以下几点考虑。详情参考：[react 为什么选择 MessageChannel](https://juejin.cn/post/6953804914715803678)

- 微任务和宏任务：时间分片不能采用微任务，因为 react 分片的目的是防止阻塞渲染，但微任务会在下一次页面更新之前执行，从某种角度上来说和同步任务是没有区别的。
- setTimeout：由于 setTimeout 递归执行可能导致 4ms 的延迟。scheduler 本质上是一种递归（调度一个任务，这个任务内还会再调度一个任务），不能采用。
- rAF：执行时间不确定。在 rAF() 的回调中再次调用 rAF()，会将第二次 rAF() 的回调放到下一帧前执行，而不是在当前帧前执行

## 异步调度

### 调度是什么

所谓调度，其实就是一个处理多个任务之间执行顺序、执行时机等的过程，目的是让这些任务依次尽快完成，同时保证尽量不阻塞浏览器渲染。
在 v17 及其以下版本，所有的任务都是紧急任务，各个任务之间平等调度，只需要保证顺序即可；
在 v18 模式下，正常紧急的任务都可以看作是会员，一些优先级低的任务比如 transtion 过渡任务，可以看作非会员。如果会员和非会员排列到一起，那么优先会办理会员的业务（正常的紧急优先任务），正常情况下，会办理完所有的会员业务，才开始办理非会员任务；但是在一些极端的情况下，怕会员一直办理，非会员无法办理（被饿死的情况），所以设置一个超时时间，达到超时时间，会破格执行一个非会员任务。

在绝大多数情况下，**调度的对象都是更新**，因此调度其实和更新任务分不开的。更新本质上有两种：

- 第一种就是初始化的时候第一次页面的呈现。
- 第二种就是初始化完毕，state 的更新，比如点击按钮，触发 setState 或者 useState。

我们的 React 从 ReactDOM.render 或 createRoot 开始执行，就开始了初始化过程。下面以 React17 的 render 为例，看看是怎么创建一个更新并进入调度的。

调度和调和的关系又是什么呢？
通常来说，调和是 React 进行具体更新的地方，就是根据 state 的改变，去切实地更新视图的过程。因此如果通过事件触发更新，那么会先进入调度、安排更新、更新 state，接下来才会根据更新的 state 做调和工作。
调和的过程可以看作是 workLoop，对象是 fiber，经过 render 和 commit 过程，最终修改真实 dom，触发浏览器渲染，更新视图。

这里说一下调度和调和的入口函数，以后在源码中看到这两个函数就知道作用了：

- 调度入口函数：`ensureRootIsScheduled`
- 调和入口函数：`performSyncWorkOnRoot`或`performUnitOfWork`

### 从 ReactDOM.render 看初始化流程

```js
import ReactDOM from "react-dom";
/* 通过 ReactDOM.render  */
ReactDOM.render(<App />, document.getElementById("app"));
```

在 ReactDOM.render 做的事情是形成一个 Fiber Tree 挂载到 app 上。大致如下：

```js
function legacyRenderSubtreeIntoContainer(
  parentComponent, // null
  children, // <App/> 跟部组件
  container, // app dom 元素
  forceHydrate,
  callback // ReactDOM.render 第三个参数回调函数。
) {
  let root = container._reactRootContainer;
  let fiberRoot;
  if (!root) {
    /* 创建 fiber Root */
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
      container,
      forceHydrate
    );
    fiberRoot = root._internalRoot;
    /* 处理 callback 逻辑，这里可以省略 */
    /* 注意初始化这里用的是 unbatch，即非批量更新 */
    unbatchedUpdates(() => {
      /*  开始更新  */
      updateContainer(children, fiberRoot, parentComponent, callback);
    });
  }
}
```

调用 ReactDOM.render 本质上就是 legacyRenderSubtreeIntoContainer 方法。这个方法的主要做的事情是创建整个应用的 FiberRoot ，然后调用 updateContainer 开始初始化更新。

updateContainer 就是初始化更新的函数，它的主要作用就是调度一次更新，具体来说是：

1. 首先计算更新优先级 lane ，老版本用的是 expirationTime。
2. 然后创建一个 update ，通过 enqueueUpdate 把当前的 update 放入到待更新队列 updateQueue 中。
3. 接下来开始调用 scheduleUpdateOnFiber ，开始进入调度更新流程中。

```js
export function updateContainer(element, container, parentComponent, callback) {
  /* 计算优先级，在v16及以下版本用的是 expirationTime ，在 v17 ,v18 版本，用的是 lane。  */
  const lane = requestUpdateLane(current);
  /* 创建一个 update */
  const update = createUpdate(eventTime, lane);
  enqueueUpdate(current, update, lane);
  /* 开始调度更新 */
  const root = scheduleUpdateOnFiber(current, lane, eventTime);
}
```

可以看到最后调用了`scheduleUpdateOnFiber`这个函数。这个函数就是所有调度的入口，包括后面的 setState 导致的调度也是从这里进入的。

### 从 useState | setState 看更新流程

触发 setState 本质上是调用 enqueueSetState，内部也是和初始化中的 updateContainer 近似

```js
enqueueSetState(inst,payload,callback){
    const update = createUpdate(eventTime, lane);
    enqueueUpdate(fiber, update, lane);
    const root = scheduleUpdateOnFiber(fiber, lane, eventTime);
}
```

而 useState 的 dispatchAction，其实也是调用了 scheduleUpdateOnFiber

```js
function dispatchAction(fiber, queue, action) {
  const lane = requestUpdateLane(fiber);
  scheduleUpdateOnFiber(fiber, lane, eventTime);
}
```

### 更新入口 scheduleUpdateOnFiber

上面所有更新的矛头都指向了 scheduleUpdateOnFiber 这个函数，其内容大致如下：

```js
export function scheduleUpdateOnFiber(root, fiber, lane, eventTime) {
  if (lane === SyncLane) {
    // 同步事件更新优先级
    if (
      (executionContext & LegacyUnbatchedContext) !== NoContext && // unbatch 情况，比如初始化
      (executionContext & (RenderContext | CommitContext)) === NoContext
    ) {
      /* 开始同步更新，进入到 workloop 流程 */
      performSyncWorkOnRoot(root);
    } else {
      /* 进入调度，把任务放入调度中 */
      ensureRootIsScheduled(root, eventTime);
      if (executionContext === NoContext) {
        /* 当前的执行任务类型为 NoContext ，说明当前任务是非可控的，那么会调用 flushSyncCallbackQueue 方法。 */
        flushSyncCallbackQueue();
      }
    }
  }
}
```

大多数情况下更新任务的优先级都是 SyncLane，少数可能是 transition 带来的低优先级任务，或者不在事件内部触发的更新。
在 scheduleUpdateOnFiber 中对 SyncLane 优先级的更新分成了两种：

1. unbatch 情况，即非批量更新，上面说过只有初次更新才会是这种情况，因此会直接进入 workLoop，即调和阶段，接下来就是 render/commit 等流程了
2. batch 情况，即会触发批量更新的任务，基本上都是事件内部的 setState。这时会调用 ensureRootIsScheduled 进行调度，接下来进入调度流程。

## 调度流程

上面说到了 React 是怎么发起一次调度的，那么接下来就是具体是如何调度的。

### 从更新到调度

调度流程开始的关键是 ensureRootIsScheduled 函数，所有非初始化的更新都会走到这个函数流程中，也就是说每一个更新都会触发这个函数；它主要做的事情有：

1. 计算最新的调度更新优先级 newCallbackPriority，接下来获取当前 root 上的 callbackPriority 判断两者是否相等。如果两者相等，那么将直接退出不会进入到调度中。可以看作是将相同优先级任务合并了。

> 如果在正常模式下（非异步）一次更新中触发了多次 setState 或者 useState ，那么第一个 setState 进入到 ensureRootIsScheduled 就会有 `root.callbackPriority = newCallbackPriority`，那么接下来如果还有 setState | useState，那么就会退出，将不进入调度任务中，这才是批量更新的原理，多次触发更新只有第一次会进入到调度中。
> 后面的更新虽然不进入 ensureRootIsScheduled，但更新的 update 对象在之前就已经绑定到了 root 上，也就是说 root 上的 update 的值是最终的值（即有多个 setState 的话，root 上就有最后传入的 state）；这样在后面执行更新任务时（不管是同步执行 flushSyncCallbackQueue 还是异步执行 schduleCallback）就会拿到最新的值去更新。

2. 如果最新任务和之前的任务优先级不相等，就会进入调度任务 scheduleSyncCallback 中，开始调和
3. 最后会将 newCallbackPriority 赋值给 root.callbackPriority。下一次更新就可以和上一次的作比较，然后回到第一步

```ts
function ensureRootIsScheduled(root: FiberRoot, currentTime: number) {
  // 采用最高优先级代表更新的优先级，这个newCallbackPriority就是所有lanes的最高值，不存在更大的lane
  const newCallbackPriority = getHighestPriorityLane(nextLanes);

  // 检查是否有存在的优先级（root上的），如果有就退出并重用root上的
  // root上存的是之前更新的优先级，也就是说如果本次更新优先级和上次一样，说明是在一次更新内部，会退出
  const existingCallbackPriority = root.callbackPriority;
  if (existingCallbackPriority === newCallbackPriority) {
    return;
  }

  // 调度一个新的更新任务
  let newCallbackNode;
  if (newCallbackPriority === SyncLanePriority) {
    // 这里根据Lagacy模式和concurrent模式分别进入不同的调度流程
    // 但是核心都是performSyncWorkOnRoot，这个是调度的入口函数
    if (root.tag === LegacyRoot) {
      scheduleLegacySyncCallback(performSyncWorkOnRoot.bind(null, root));
    } else {
      scheduleSyncCallback(performSyncWorkOnRoot.bind(null, root));
    }
    // 接下来就是触发微任务，用微任务去立即执行更新
    // 微任务执行更新的详细可以看state那块的
    if (supportsMicrotasks) {
      scheduleMicrotask(() => {
        if (
          (executionContext & (RenderContext | CommitContext)) ===
          NoContext
        ) {
          flushSyncCallbacks();
        }
      });
    }
    newCallbackNode = null;
  } else {
    let schedulerPriorityLevel;
    // 如果不是同步任务，就确定另一种优先级，即根据到期事件确定的优先级机制
    switch (lanesToEventPriority(nextLanes)) {
      case DiscreteEventPriority:
        schedulerPriorityLevel = ImmediateSchedulerPriority;
        break;
      case ContinuousEventPriority:
        schedulerPriorityLevel = UserBlockingSchedulerPriority;
        break;
      case DefaultEventPriority:
        schedulerPriorityLevel = NormalSchedulerPriority;
        break;
      case IdleEventPriority:
        schedulerPriorityLevel = IdleSchedulerPriority;
        break;
      default:
        // setTimeout中的setState任务优先级通常会被转为NormalSchedulerPriority
        schedulerPriorityLevel = NormalSchedulerPriority;
        break;
    }
    // 然后根据新的优先级调度任务
    // 这个scheduleCallback就是schduler的unstable_SchedulerCallback，可以在下面Scheduler查看
    newCallbackNode = scheduleCallback(
      schedulerPriorityLevel,
      performConcurrentWorkOnRoot.bind(null, root)
    );
  }
  // 把当前任务的优先级绑定到root上
  root.callbackPriority = newCallbackPriority;
  root.callbackNode = newCallbackNode;
}
```

### 进入调度任务

进入调度的其实主要是三个函数，核心是 scheduleSyncCallback，另外的 scheduleLegacySyncCallback 是这个的变体。而 scheduleCallback 则是调度的执行方法，即通过 MessageChannel 向浏览器请求下一空闲帧，在空闲帧中执行更新任务这个过程。

```js
function scheduleSyncCallback(callback) {
  // syncQueue是一个全局变量，和flushSyncCallbackQueue共享
  if (syncQueue === null) {
    syncQueue = [callback];
    // 这里原本有一个开启调度的函数执行，但是现在的源码中貌似删除了
    // 并不是在这里执行，而是在上面ensureRootIsScheduled中通过scheduleMicrotask以微任务的形式执行
    // 或者在某些特殊条件下由flushSyncCallbackQueue立即同步执行
  } else {
    syncQueue.push(callback);
  }
}
function scheduleLegacySyncCallback(callback: SchedulerCallback) {
  includesLegacySyncCallbacks = true; // 其实就是开启legacy模式
  scheduleSyncCallback(callback);
}
```

scheduleSyncCallback 有一点小改动，做的事情如下：

- 如果执行队列为空，那么把当前任务放入队列中。
- 如果队列不为空，此时已经在调度中，那么不需要执行调度任务，只需要把当前更新放入队列中就可以，调度中心会一个个按照顺序执行更新任务。

放入 syncQueue 之后，接下来就需要调用 flushSyncCallbackQueue 去执行；注意这个 flushSyncCallbackQueue 的执行时机：

- 如果是正常 SyncLanePriority 的更新任务，参考上面 ensureRootIsScheduled，有一段这样的代码：

```js
if (supportsMicrotasks) {
  scheduleMicrotask(() => {
    if ((executionContext & (RenderContext | CommitContext)) === NoContext) {
      flushSyncCallbacks();
    }
  });
}
```

可以看出，这里先通过 syncQueue 收集所有的任务，然后并不立即执行，而是通过 scheduleMicrotask 开启一个微任务，在微任务中执行所有任务。这个就是 state 那章中讲到的微任务批量执行的方式

> 注意：这里 scheduleMicrotask 实际上是 queueMicrotask 这个 api，它是一种异步执行的方式，但它和调度的“不阻塞渲染”完全不是一个东西
>
> - scheduleMicrotask 之所以采用微任务，主要还是为了收集事件和批量更新，其实 flushSyncCallbacks 也完全可以正常同步执行，微任务只是一种批量更新的实现，和调度本身的那种依赖 MessageChannel 实现不阻塞的原理不同。**scheduleMicrotask 的对象是放入 syncQueue 中的调和任务**
> - 而我们说的 scheduler 的异步调度，实际上是 scheduleCallback 这个函数的实现，即通过 MessageChannel 将阻塞任务改为空闲时执行，它的执行对象是所有 React 中的任务，里面有 taskQueue、timerQueue 这样存储各种任务的结构，可以把它理解为 Scheduler 本身。
>   并且，通过微任务收集事件的方法是 v18 新增的，在之前的版本中都是通过批量更新开关和同步执行 flushSyncCallbacks 来实现的。因此 v18 之前的版本在 setTimeout 内部的 setState 不会被合并。v18 中会在 setTimeout 内部等 React 不可控的地方仍然实现批量更新；至于怎么实现的，参考 state 那部分的章节。

### 空闲期的同步任务(React17)

> 这部分的内容是 React17 的。
> 在 React18 中，不管是不是空闲，都要走正常流程，即经过 scheduleMicrotask 内的 flushSyncCallbackQueue 执行
> React18 中删去了 batchedEventUpdates，也就不存在空闲期立即执行了

上面的任务调度有一个特点，就是最后会用到 scheduleCallback 这种调度方式，即 MessageChannel 向浏览器请求下一空闲帧，在空闲帧中执行更新任务这个过程。

但是如果浏览器当前就正处于空闲期，那就完全没必要等到下一个空闲帧，而是可以立即同步执行这次更新，即**跳过异步调度**。同步更新的函数就是 flushSyncCallbackQueue，他会立即执行更新队列，发起更新任务，目的就是让任务不延时到下一帧。之后，异步调度任务还是会执行，不过等到异步执行时任务队列已经为空了。

```js
function batchedEventUpdates(fn, a) {
  /* 批量更新流程，没有更新状态下，那么直接执行任务 */
  var prevExecutionContext = executionContext;
  executionContext |= EventContext;
  try {
    return fn(
      a
    ); /* 执行事件本身，React 事件在这里执行，useState 和 setState 也会在这里执行 */
  } finally {
    /* 重置状态 */
    executionContext = prevExecutionContext;
    if (executionContext === NoContext) {
      /* 批量更新流程，没有更新状态下，那么直接执行任务 */
      flushSyncCallbackQueue();
    }
  }
}
```

flushSyncCallbackQueue 函数存在于所有调度更新的位置，作为异步调度的备选项，如果能立即更新就会立即更新。
上面的 scheduleUpdateOnFiber 这个入口函数，即所有调度任务的入口，有一段这个代码：

```js
if (executionContext === NoContext) {
  /* 当前的执行任务类型为 NoContext ，说明当前任务是非可控的，那么会调用 flushSyncCallbackQueue 方法。 */
  flushSyncCallbackQueue();
}
```

非可控在上面说过，就是 setTimeout、Promise 中的更新，这些更新会立即执行，不会合并，即一个更新触发一次 flushSyncCallbackQueue。因此在这些内的更新就是非批量更新，会导致多次渲染。

flushSyncCallbackQueue 内部代码也很简单，就是取出 syncQueue，依次遍历执行内部的 callbacks。

## schduler

### scheduleCallback 统一调度

只有在 concurrent 模式下才会进入 scheduleCallback，并且是否开启时间分片还需要进一步判断

```ts
if (root.tag === LegacyRoot) {
  scheduleLegacySyncCallback(performSyncWorkOnRoot.bind(null, root));
} else {
  // scheduleSyncCallback就是把performSyncWorkOnRoot放入syncQueue中，下面flushSyncCallbacks就是执行syncQueue
  scheduleSyncCallback(performSyncWorkOnRoot.bind(null, root));
}
// supportsMicrotasks类似于一个配置，如果开启这个配置，就不会进入scheduleCallback，反之则会进入
if (supportsMicrotasks) {
  scheduleMicrotask(() => {
    if ((executionContext & (RenderContext | CommitContext)) === NoContext) {
      flushSyncCallbacks();
    }
  });
} else {
  // Flush the queue in an Immediate task.
  // 调度的任务仍然是flushSyncCallbacks函数，这个函数会循环执行syncQueue中的任务
  scheduleCallback(ImmediateSchedulerPriority, flushSyncCallbacks);
}
```

对 React18 来说，当调用 schedulerCallback 时需要先确定一个优先级，即第一个参数。这个优先级不同于更新任务的 lane 优先级，而是一套调度优先级，根据任务的过期时间确定任务的优先级。

- `Immediate` -1 需要立刻执行。
- `UserBlocking` 250ms 超时时间 250ms，一般指的是用户交互。
- `Normal` 5000ms 超时时间 5s，不需要直观立即变化的任务，比如网络请求。
- `Low` 10000ms 超时时间 10s，肯定要执行的任务，但是可以放在最后处理。
- `Idle` 一些没有必要的任务，可能不会执行。

然后就是调用 scheduleCallback，传入优先级以及调和入口函数 performConcurrentWorkOnRoot。scheduleCallback 详见下：

```js
function unstable_scheduleCallback(
  priorityLevel: PriorityLevel, // 根据过期时间确定的优先级
  callback: Callback, // 在更新产生的调度期间为performConcurrentWorkOnRoot
  options?: { delay: number }
): Task {
  var currentTime = getCurrentTime();

  var startTime;
  // 如果有options参数，则startTime表示为currentTime + delay，这时是要调度timerQueue中的任务，即可以稍等一段时间（delay）再调度
  // 如果没有options，则startTime = currentTime，这时就是调度taskQueue中的任务，即立即调度
  if (typeof options === "object" && options !== null) {
    var delay = options.delay;
    if (typeof delay === "number" && delay > 0) {
      startTime = currentTime + delay;
    } else {
      startTime = currentTime;
    }
  } else {
    startTime = currentTime;
  }

  // 设置时延
  // 当前时间+时延 = 过期时间
  var timeout;
  switch (priorityLevel) {
    case ImmediatePriority:
      timeout = IMMEDIATE_PRIORITY_TIMEOUT;
      break;
    case UserBlockingPriority:
      timeout = USER_BLOCKING_PRIORITY_TIMEOUT;
      break;
    case IdlePriority:
      timeout = IDLE_PRIORITY_TIMEOUT;
      break;
    case LowPriority:
      timeout = LOW_PRIORITY_TIMEOUT;
      break;
    case NormalPriority:
    default:
      timeout = NORMAL_PRIORITY_TIMEOUT;
      break;
  }

  // 过期时间
  var expirationTime = startTime + timeout;

  var newTask: Task = {
    id: taskIdCounter++,
    callback,
    priorityLevel,
    startTime,
    expirationTime,
    sortIndex: -1,
  };
  if (enableProfiling) {
    newTask.isQueued = false;
  }

  if (startTime > currentTime) {
    // 延迟任务，排序标准是startTime即开始事件
    newTask.sortIndex = startTime;
    // 放入timerQueue中
    push(timerQueue, newTask);
    // timerQueue和taskQueue都是最小堆
    if (peek(taskQueue) === null && newTask === peek(timerQueue)) {
      // taskQueue为空，但timerQueue不为空，说明所有的非延时任务都执行完了，现在可以开始执行延时任务
      // requestHostTimeout就是setTimeout
      // startTime - currentTime === delay，也就是在delay秒之后执行handleTimeout
      requestHostTimeout(handleTimeout, startTime - currentTime);
    }
  } else {
    // 执行taskQueue中的非延时任务，排序标准是expirationTime即过期时间
    newTask.sortIndex = expirationTime;
    push(taskQueue, newTask);

    if (!isHostCallbackScheduled && !isPerformingWork) {
      isHostCallbackScheduled = true; // 这个变量用于判断是否正在处于taskQueue的调度中
      // requestHostCallback有几种选择，如果某一种api不支持就选择下一个
      // setImmidiate -> messageChannel -> setTimeout
      requestHostCallback(flushWork);
    }
  }

  return newTask;
}
```

这里有几个关键变量和函数：

- taskQueue，里面存的都是要立即执行的任务，依据任务的过期时间( expirationTime ) 排序，需要在调度的 workLoop 中循环执行完这些任务。
- timerQueue 里面存的都是可以稍等一会再执行的任务，依据任务的开始时间( startTime )排序，如果时间超过了延时，就放入 taskQueue 队列。
- requestHostTimeout：就是 setTimeout
- handleTimeout：通过 setTimeout 指定延时之后调用 handleTimeout，handleTimeout 流程：

1. 先关闭调度(isHostTimeoutScheduled = false)
2. 把 timerQueue 中到时间的任务放到 taskQueue 中，即 advanceTimers 的作用
3. 继续调度任务

因此 handleTimeout 的主要作用就是调整 timerQueue 中的任务，把到时间的放到 taskQueue 去执行。而 handleTimeout 函数会在本次调度任务执行后的 delay 秒执行，恰好就是 timerQueue 中的任务过期、准备开始执行 timerQueue 中的任务的时候。

```ts
function handleTimeout(currentTime: number) {
  isHostTimeoutScheduled = false;
  advanceTimers(currentTime); // 这个函数会把timerQueue中所有到时间的任务放到taskQueue中

  if (!isHostCallbackScheduled) {
    // 如果不在taskQueue调度中
    // 并且taskQueue还有任务
    if (peek(taskQueue) !== null) {
      isHostCallbackScheduled = true;
      // 启动调度任务
      requestHostCallback(flushWork);
    } else {
      const firstTimer = peek(timerQueue);
      // 如果timerQueue有任务，那就再启动timerQueue的调度
      if (firstTimer !== null) {
        requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime);
      }
    }
  }
}

function advanceTimers() {
  var timer = peek(timerQueue);
  while (timer !== null) {
    if (timer.callback === null) {
      pop(timerQueue);
    } else if (timer.startTime <= currentTime) {
      /* 如果任务已经过期，那么将 timerQueue 中的过期任务，放入taskQueue */
      pop(timerQueue);
      timer.sortIndex = timer.expirationTime;
      push(taskQueue, timer);
    }
    timer = peek(timerQueue);
  }
}
```

- requestHostCallback：即 taskQueue 的调度，通常会选择 MessageChannel，因此就是通过 MessageChannel 调用 callback，也就是 flushWork

```ts
function requestHostCallback(callback) {
  scheduledHostCallback = callback;
  if (!isMessageLoopRunning) {
    isMessageLoopRunning = true;
    schedulePerformWorkUntilDeadline();
  }
}
let schedulePerformWorkUntilDeadline; // 三选一，支持哪个就先选哪个
// 不管选的是哪种，目的都是以异步宏任务的形式启动调和任务开始调和
if (typeof localSetImmediate === "function") {
  // setImmediate api，只能在nodejs环境下用
  schedulePerformWorkUntilDeadline = () => {
    localSetImmediate(performWorkUntilDeadline);
  };
} else if (typeof MessageChannel !== "undefined") {
  // MessageChannel api
  const channel = new MessageChannel();
  const port = channel.port2;
  channel.port1.onmessage = performWorkUntilDeadline;
  schedulePerformWorkUntilDeadline = () => {
    port.postMessage(null);
  };
} else {
  // setTimeout
  schedulePerformWorkUntilDeadline = () => {
    localSetTimeout(performWorkUntilDeadline, 0);
  };
}
```

scheduleCallback 流程如下:

1. 创建一个新的任务 newTask。
2. 通过任务的开始时间( startTime ) 和 当前时间( currentTime ) 比较:当 startTime > currentTime, 说明未过期, 存到 timerQueue，当 startTime <= currentTime, 说明已过期, 存到 taskQueue。
3. 如果任务过期，并且没有调度中的任务，那么调度 `requestHostCallback`。本质上调度的是 flushWork。
4. 如果任务没有过期，用 `requestHostTimeout` 延时执行 handleTimeout。

大致的调度流程如下：
![](https://pic.imgdb.cn/item/62cd9084f54cd3f937e88618.jpg)

### performWorkUntilDeadline

requestHostCallback 会以 MessageChannel 的形式调度一个函数 performWorkUntilDeadline，这是整个调度中的中转部分，即前面都是同步过程，在这里转为一个“等待浏览器空闲时才执行”的步骤。
当浏览器空闲时，就会执行 performWorkUntilDeadline，这个函数简化代码如下：

```js
const performWorkUntilDeadline = () => {
  // scheduledHostCallback就是flushWork
  if (scheduledHostCallback !== null) {
    const currentTime = getCurrentTime();
    startTime = currentTime;
    const hasTimeRemaining = true;

    let hasMoreWork = true;
    try {
      hasMoreWork = scheduledHostCallback(hasTimeRemaining, currentTime);
    } finally {
      if (hasMoreWork) {
        schedulePerformWorkUntilDeadline();
      } else {
        isMessageLoopRunning = false;
        scheduledHostCallback = null;
      }
    }
  } else {
    isMessageLoopRunning = false;
  }
};
```

scheduledHostCallback 在 requestHostCallback 中就是 flushWork，这个要记好，后面会多次出现；
performWorkUntilDeadline 函数的主要作用就是调用 flushWork，从 flushWork 中获取一个返回值 hasMoreWork，表示是否还有剩余未完成的工作。(这个返回值其实来自于 workLoop，下面会说到)
**如果 hasMoreWork 为 true，也就是 taskQueue 中还有未完成的任务，就会再调用一次 schedulePerformWorkUntilDeadline，也就是用 MessageChannel 等到下一个浏览器空闲时，再执行剩下未完成的任务。**
这一步也就是当 taskQueue 的任务执行时间超过 schduler 的时间分片（通常是 5ms）时，就再注册一个 MessageChannel，等到下一次有时间再执行。

### flushWork 和 workLoop

```js
function flushWork(){
  if (isHostTimeoutScheduled) { /* 如果有延时任务，那么先暂停延时任务的调度*/
    isHostTimeoutScheduled = false;
    cancelHostTimeout();
  }
  try{
     /* 执行 workLoop 里面会真正调度我们的事件  */
     workLoop(hasTimeRemaining, initialTime)
  }
}
```

flushWork 如果有延时任务执行的话，那么会先暂停延时任务，然后调用 workLoop ，去真正执行超时的更新任务。

在一次更新调度过程中，workLoop 会依次执行任务队列 taskQueue 中的任务。
显然所有 react 调度的任务最终都会进入 taskqueue，因此只要像浏览器的 EventLoop 一样不断调用 taskQueue 就可以完成执行。

```js
function workLoop() {
  var currentTime = initialTime;
  // advanceTimers上面说过，检查timerQueue，把任务放到taskQueue中
  advanceTimers(currentTime);
  /* 获取taskQueue中的第一个 */
  currentTask = peek(taskQueue);
  while (currentTask !== null) {
    /* 真正的更新函数 callback */
    if (
      currentTask.expirationTime > currentTime &&
      (!hasTimeRemaining || shouldYieldToHost())
    ) {
      // workLoop可以中断
      // 如果当前任务的超时事件大于当前时间（还没过期）并且shouldYieldToHost返回true（表示其他任务到来，应该中断），就会中断
      // 此时taskQueue中还有任务，留到下一次执行
      break;
    }
    var callback = currentTask.callback;
    if (callback !== null) {
      /* 执行更新 */
      callback();
      /* 先看一下 timeQueue 中有没有 过期任务。 */
      advanceTimers(currentTime);
    }
    /* 再一次获取任务，循环执行 */
    currentTask = peek(taskQueue);
  }
  // 检查是否还有未完成的任务
  if (currentTask !== null) {
    // 这个值会返回给performWorkUntilDeadline，会注册新的回调，等到下一次有空闲再去执行
    return true;
  } else {
    // 如果全部完成，那就调度timerQueue
    const firstTimer = peek(timerQueue);
    if (firstTimer !== null) {
      requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime);
    }
    return false;
  }
}
```

workLoop 是一个 while 循环，类似 EventLoop 一样执行 taskQueue 中的任务，每次循环还会检查 timerQueue 中是否还有任务到时间了，把它加入到 taskQueue 中。
workLoop 执行的单位是在 scheduleCallback 中创建的 task。实际上是 task.callback，即具体的调和任务。
当这个循环结束，可能有两种情况：

1. shouldYieldToHost 导致的中断。中断的原因可能有很多，比如需要给浏览器渲染线程让出 cpu、某些任务超时等，这时 taskQueue 还没执行完。workLoop 会返回 true，然后在上面说的 performWorkUntilDeadline 中，再注册一个回调，等到浏览器下一次空闲时再继续执行 taskqueue 中的任务
2. taskQueue 执行完毕，timerQueue 可能还有任务，但暂时还没到时间。这时会返回 false，即不用再注册一个任务了，并且如果 timerQueue 还有任务，就调用 requestHostTimeout 继续调度 timerQueue。

#### shouldYieldToHost

> 这个函数既用于调度阶段的 workLoop，也用于 concurrent 模式下的调和阶段的 workLoopConcurrent 的中断。

这个函数是用于判断什么时候中断的，当返回 true 时说明要中断了。简化代码如下：

```ts
function shouldYieldToHost(): boolean {
  // 当前任务已经执行的时间
  const timeElapsed = getCurrentTime() - startTime;
  //
  if (timeElapsed < frameInterval) {
    return false;
  }

  if (enableIsInputPending) {
    if (needsPaint) {
      return true;
    }
    if (timeElapsed < continuousInputInterval) {
      if (isInputPending !== null) {
        return isInputPending();
      }
    } else if (timeElapsed < maxInterval) {
      if (isInputPending !== null) {
        return isInputPending(continuousOptions);
      }
    } else {
      return true;
    }
  }

  // `isInputPending` isn't available. Yield now.
  return true;
}
```

有几个关键变量：

- frameInterval: 一帧的时间。默认是 16.6，实际会根据`1000 / 帧率`来计算，帧率范围在 0-125 之间。
- enableIsInputPending: 能使用 isInputPending api，这个 api 下面会说到
- needsPaint: 浏览器需要去绘制。这个变量取决于 requestPaint 函数，同样取决于 isInputPending
- continuousInputInterval: 直译为持续输入的间隔，比如 mousemove 事件会在每 n 毫秒触发一次，那么 n 毫秒就是这个间隔。如果当前任务的执行耗时没有超过这个间隔，就不会阻塞用户对持续事件的输入，因此并不需要立即中断。
- maxInterval: 最大间隔，默认为 300 毫秒，任务执行不能超出这个时间。

以下情况会返回 true 导致中断：

- 任务执行时间超过用户持续输入的最小间隔，即可能会阻塞用户输入
- 任务执行时间超过最大间隔 300 毫秒
- 浏览器需要绘制
- isInputPending 返回 true，即用户此时正在输入。

isInputPending，参考https://developer.chrome.com/articles/isinputpending/

这个 API 当浏览器有需要处理的输入事件时，调用`isInputPending()`会返回 true
在不传入任何参数的情况下，将会检测所有类型的输入事件，包括按键、鼠标、滚轮触控等 DOM UI 事件，也可以手动传入一个包含事件类型的数组参数。

也就是说，当浏览器检测到用户输入时，这个函数会实时返回 true。比如我们将其放入一个循环中，如果用户触发事件，这个循环就会中断：

```js
while (workQueue.length > 0) {
  if (navigator.scheduling.isInputPending()) {
    break;
  }
  let job = workQueue.shift();
  job.execute();
}
```

这种方式可以更灵活地检测用户输入，从而保证不会阻塞用户的输入。

## 渲染中断

react 的 concurrent 模式下还有一个非常重要的地方，就是渲染的可中断。
Scheduler 通过时间分片控制了每个任务的最大执行时间，给任务设置不同的过期时间，分为 timerQueue 和 taskQueue，通过 MessageChannel 来手动调度 taskQueue 中每个任务的执行。
但是有一个问题是，scheduler 没有考虑到单个任务的执行问题，即 scheduler 调度的对象是任务，但每个任务可能会执行很长时间。
对于这种 long task 可以通过时间分片 + 优先级调度的方式在执行和暂停之间切换状态，就像 Scheduler 那样。

这里就有一个很重要的问题。对于 react 的渲染过程（render 的中断），当一次渲染中断之后，怎么在下一次执行时恢复到上次中断的位置？总不能全部重新渲染吧

在 performConcurrentWorkOnRoot 函数内有这么一段：

```js
function performConcurrentWorkOnRoot(root) {
  // 省略一部分代码 ... 执行 renderRootConcurrent 前的一些准备

  // 开始 Reconciler 的 render 阶段
  // exitStatus 用来表示 render 流程的结果状态
  let exitStatus = renderRootConcurrent(root, lanes);

  if (
    includesSomeLane(
      workInProgressRootIncludedLanes,
      workInProgressRootUpdatedLanes
    )
  ) {
    // ... 处理边缘场景
    // 在 rendering 阶段中，如果发生了新的 update，但是这个 update 是把一些隐藏的组件重新展示出来
    // 但是新的 update 的 lane 已经在当前的 rendering 阶段中被标记过了（执行过了），所以需要重头开始
    prepareFreshStack(root, NoLanes);
    // 看到这段注释，想必大家会很疑惑，不着急，我们现在还不需要理解它
  } else if (exitStatus !== RootIncomplete) {
    // 如果退出状态不等于未完成，那么说明有可能是一下几种
    // 1、已完成
    // 2、有报错被捕获
    // 3、有报错未被捕获，严重级别的错误
  }

  // ensureRootIsScheduled内部会修改callbackNode，这个可以详情看下面模拟实现的部分
  ensureRootIsScheduled(root, now());
  if (root.callbackNode === originalCallbackNode) {
    // 如果当前执行的 scheduler task 未发生变化，还是最初在执行的那个 task
    // 那么返回一个新的函数，注意这一步，很关键
    return performConcurrentWorkOnRoot.bind(null, root);
  }
  // 如果当前执行的 scheduler task 已经发生了变化或者被取消了，返回 null
  // 这一步没有返回函数，也很关键
  return null;
}
```

一般来说，如果没有出现更高优先级的 lane priority，任务 task 就不会取消，也就是 root.callbackNode 和 root.callbackPriority 都没有发生变化
若任务处于 RootInComplete - 未完成 状态，那么 performConcurrentWorkOnRoot 会返回 `performConcurrentWorkOnRoot.bind(null, root)` 作为恢复任务的关键，Scheduler 在执行 workloop 流程中会保存这个返回的回调，并重新赋值到 task.callback 上，在下次调度时重新执行。这时任务层面的中断和执行。

注意这个 root 不是真的 root，而指的是具体的节点！因此相当于当任务中断时，performConcurrentWorkOnRoot 返回一个 bind 的函数，保留了当前正在渲染的节点位置。然后 scheduler 保存这个回调，在任务下一次执行时重新赋给 task，那么就会继续执行渲染了。

对于任务内部来说，主要是通过这个部分：

```js
let exitStatus = renderRootConcurrent(root, lanes);
```

函数是这样：

```js
function renderRootConcurrent(root: FiberRoot, lanes: Lanes) {
  // ... 省略流程
  do {
    try {
      if (
        workInProgressSuspendedReason !== NotSuspended &&
        workInProgress !== null
      ) {
        // ...
        workLoopConcurrent();
        // 如果workLoop渲染中断，就退出
        break;
      }
    } catch (thrownValue) {
      handleError(root, thrownValue);
    }
  } while (true);
  // ... 省略流程
}
// 一些恢复操作
resetContextDependencies();

popDispatcher(prevDispatcher);
popCacheDispatcher(prevCacheDispatcher);
executionContext = prevExecutionContext;

// Check if the tree has completed.
if (workInProgress !== null) {
  // Still work remaining.
  if (enableSchedulingProfiler) {
    markRenderYielded();
  }
  return RootInProgress;
} else {
  // Completed the tree.
  if (enableSchedulingProfiler) {
    markRenderStopped();
  }

  // Set this to null to indicate there's no in-progress render.
  workInProgressRoot = null;
  workInProgressRootRenderLanes = NoLanes;

  // It's safe to process the queue now that the render phase is complete.
  finishQueueingConcurrentUpdates();

  // Return the final exit status.
  return workInProgressRootExitStatus;
}
```

workLoopConcurrent 就是上面说过的 render 阶段的 workLoop。
这时会判断是否还有未完成渲染的节点。如果还有剩余节点需要处理，返回未完成的状态 RootInProgress；如果已完成，重置 workInProgress 状态，返回 workInProgressRootExitStatus

对于 RootInProgress 这个状态，表示渲染被中断，在 ensureRootIsScheduled 函数内，会得到这个状态：

```js
let exitStatus = shouldTimeSlice
  ? renderRootConcurrent(root, lanes)
  : renderRootSync(root, lanes);
```

然后下面会对 exitStatus 做出判断，如果不是 RootInProgress，那么就会考虑几种情况，比如因为高优先级中断，或者也可能是 render 阶段完成这样的。

到这里为止，可以看到 react 内部其实没有显式地说明，如果渲染过程因为时间片原因中断，应该怎么恢复之前的节点；
可以大胆猜测一下，既然没有说怎么保存，只是说怎么“清除”，那就可以认为，react 本来就会保存当前正在渲染的节点（workInProgress）。如果本次 workLoop 被中断了，然后通过 scheduler 重新调度一次 workLoop，那么就还是会从 workInProgress 开始执行

```js
let workInProgress: Fiber | null = null;

// ...
function workLoopConcurrent() {
  // Perform work until Scheduler asks us to yield
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}
```

只要 workInProgress 没被清除，那么 loop 还是会从这里开始，即使是下一个时间片再调度。

反过来说，如果主动清除 workInProgress，将其重新设为 root，那就相当于放弃了已经生成的树，转而去重建一颗。如果渲染时因为高优先级任务打断的，那么 react 会重新从根部开始渲染，而不是保留上次的 wip 节点。

在 prepareFreshStack 函数内。这个函数会在 scheduleUpdateOnFiber 的一开始调度

```js
export function scheduleUpdateOnFiber(
  root: FiberRoot,
  fiber: Fiber,
  lane: Lane,
  eventTime: number
) {
  if (
    workInProgressSuspendedReason === SuspendedOnData &&
    root === workInProgressRoot
  ) {
    // The incoming update might unblock the current render. Interrupt the
    // current attempt and restart from the top.
    prepareFreshStack(root, NoLanes);
  }
}
```

这个函数主要是做了状态的初始化。即，如果因为高优先级打断的渲染，则会初始化之前保存的 workInProgress 为根部。

```js
function prepareFreshStack(root: FiberRoot, lanes: Lanes): Fiber {
  root.finishedWork = null;
  root.finishedLanes = NoLanes;

  const timeoutHandle = root.timeoutHandle;
  if (timeoutHandle !== noTimeout) {
    // The root previous suspended and scheduled a timeout to commit a fallback
    // state. Now that we have additional work, cancel the timeout.
    root.timeoutHandle = noTimeout;
    // $FlowFixMe Complains noTimeout is not a TimeoutID, despite the check above
    cancelTimeout(timeoutHandle);
  }

  resetWorkInProgressStack();
  workInProgressRoot = root;
  const rootWorkInProgress = createWorkInProgress(root.current, null);
  workInProgress = rootWorkInProgress;
  workInProgressRootRenderLanes = renderLanes = lanes;
  workInProgressSuspendedReason = NotSuspended;
  workInProgressThrownValue = null;
  workInProgressRootDidAttachPingListener = false;
  workInProgressRootExitStatus = RootInProgress;
  workInProgressRootFatalError = null;
  workInProgressRootSkippedLanes = NoLanes;
  workInProgressRootInterleavedUpdatedLanes = NoLanes;
  workInProgressRootRenderPhaseUpdatedLanes = NoLanes;
  workInProgressRootPingedLanes = NoLanes;
  workInProgressRootConcurrentErrors = null;
  workInProgressRootRecoverableErrors = null;

  finishQueueingConcurrentUpdates();

  if (__DEV__) {
    ReactStrictModeWarnings.discardPendingWarnings();
  }

  return rootWorkInProgress;
}
```

如果清除了当前保存的 workInProgress，就意味着下一次渲染将会从头开始执行。

# Hooks 原理

## hooks 与 fiber

hooks 本质是离不开函数组件对应的 fiber 的。 hooks 可以作为函数组件本身和函数组件对应的 fiber 之间的沟通桥梁。
![](https://pic.imgdb.cn/item/6247da2027f86abb2a44f2e2.jpg)

### hooks 的四种状态

函数组件的触发在一个函数中，在这里，函数组件被执行并获取返回的 jsx 对象，除此之外一个重要的环节就是设置、保存 hooks，以及确定 hooks 的调用状态。
hooks 的调用状态有三种：

1. `ContextOnlyDispatcher`： 第一种形态是防止开发者在函数组件外部调用 hooks ，所以第一种就是报错形态，只要开发者调用了这个形态下的 hooks ，就会抛出异常。
2. `HooksDispatcherOnMount`： 第二种形态是函数组件初始化 mount ，因为之前讲过 hooks 是函数组件和对应 fiber 桥梁，这个时候的 hooks 作用就是建立这个桥梁，初次建立其 hooks 与 fiber 之间的关系。
3. `HooksDispatcherOnUpdate`：第三种形态是函数组件的更新，既然与 fiber 之间的桥已经建好了，那么组件再更新，就需要 hooks 去获取或者更新维护状态。
4. `HooksDispatcherOnRerender`：第四种形态，用于重渲染，比如组件执行时就执行了 setState，就会导致一次和挂载接近的重渲染。其实内部和 update 的一致，只是用处不同

代码中引用的 hooks 都是来自于这三种对象之一的。具体来说，来自于`ReactCurrentDispatcher.current`；React 会赋予这个对象不同的 hooks 类型，当使用 hooks 时，显然就是在使用这三个对象中的 hooks，也就可以判断 hooks 在哪里调用、什么情况下调用了。

```js
const HooksDispatcherOnMount = { /* 函数组件初始化用的 hooks */
    useState: mountState,
    useEffect: mountEffect,
    ...
}
const HooksDispatcherOnUpdate ={/* 函数组件更新用的 hooks */
   useState:updateState,
   useEffect: updateEffect,
   ...
}
const ContextOnlyDispatcher = {  /* 当hooks不是函数内部调用的时候，调用这个hooks对象下的hooks，所以报错。 */
   useEffect: throwInvalidHookError,
   useState: throwInvalidHookError,
   ...
}

const HooksDispatcherOnRerender: Dispatcher = {
  useCallback: updateCallback,
  useContext: readContext,
  useEffect: updateEffect,
  。。。
};
```

### 函数组件的触发（进入 hooks）

在 fiber 调和过程中，遇到函数类型的 fiber（函数组件），就会调用 `renderWithHooks` 方法。具体来说是在调和的 beginWork->updateFunctionComponent 内部。

```ts
export function renderWithHooks<Props, SecondArg>(
  current: Fiber | null,
  workInProgress: Fiber,
  Component: (p: Props, arg: SecondArg) => any,
  props: Props,
  secondArg: SecondArg,
  nextRenderLanes: Lanes
): any {
  // 记录当前渲染 fiber
  currentlyRenderingFiber = workInProgress;
  // 清空 state， 后面存放 hook 链表 信息
  workInProgress.memoizedState = null;
  workInProgress.updateQueue = null;
  // 清空 lanes
  workInProgress.lanes = NoLanes;
  // 通过current判断是更新还是挂载，选择对应的hooks
  ReactCurrentDispatcher.current =
    current === null || current.memoizedState === null
      ? HooksDispatcherOnMount
      : HooksDispatcherOnUpdate;
  // 运行函数组件
  let children = Component(props, secondArg);
  // 检查在组件运行时，是否有重新渲染的任务
  // 比如在函数组件运行时，出现 setState 操作
  if (didScheduleRenderPhaseUpdateDuringThisPass) {
    // 执行重渲染
  }
  //将hooks设置为不可用
  ReactCurrentDispatcher.current = ContextOnlyDispatcher;

  renderLanes = NoLanes;
  return children;
}
```

这个函数有几个注意的地方：

- `workInProgress.memoizedState`用于存储该组件内的 hooks 链表。这个值在函数组件的 fiber 中表示 state；
- `workInProgress.updateQueue`：存放副作用(passive Effect)链表，即 useEffct、useLayoutEffect 等副作用组成的链表。在 commit 阶段的不同时机会执行这些副作用
- `ReactCurrentDispatcher.current`：引用的 hooks 都是从这里取到的，因此通过修改它为不同 hooks 类型，就可以控制当前函数组件调用的是哪种 hooks

### hooks 和 fiber 建立联系

hooks 初始化流程使用的是 mountState，mountEffect 等初始化节点的 hooks，将 hooks 和 fiber 建立起联系
建立起关系的方式，是通过让每一个 hooks 初始化都执行 mountWorkInProgressHook。这个函数主要作用就是创建 hook 对象，以链表的形式保存在 fiber.memoizedState

这个函数其实相当于一个创建 hook 对象的构造函数，在每个 mountxxx 的 hook 里都会先调用它创建一个 hook 对象，然后在执行相关操作。
比如说初始化的 HooksDispatcherOnMount 里的 mountState，即 useState 的初次执行形式：

```ts
function mountState<S>(
  initialState: (() => S) | S,
): [S, Dispatch<BasicStateAction<S>>] {
  // 创建一个hook对象
  const hook = mountWorkInProgressHook();
  if (typeof initialState === 'function') {
    initialState = initialState();
  }
  hook.memoizedState = hook.baseState = initialState;
  const queue: UpdateQueue<S, BasicStateAction<S>> = {
    pending: null,
    lanes: NoLanes,
    dispatch: null,
    lastRenderedReducer: basicStateReducer,
    lastRenderedState: (initialState: any),
  };
  hook.queue = queue;
  const dispatch: Dispatch<
    BasicStateAction<S>,
  > = (queue.dispatch = (dispatchSetState.bind(
    null,
    currentlyRenderingFiber,
    queue,
  ): any));
  return [hook.memoizedState, dispatch];
}
```

这个函数代码就是这样：

```js
function mountWorkInProgressHook() {
  // hook本身是一个保存状态等信息的对象，并且有一个指针next指向下一个hook
  // 每次执行一个hooks函数，都产生一个hook对象
  const hook = {
    memoizedState: null, //不同hooks表现不同。保存 state信息/effect对象/ref值 等
    baseState: null, // 一次更新中 ，产生的最新state值
    baseQueue: null, // usestate中 保存最新的更新队列。
    queue: null, // 保存待更新队列 pendingQueue ，更新函数 dispatch 等信息
    next: null, // 下一个 hooks对象
  };
  if (workInProgressHook === null) {
    // 只有一个 hooks
    currentlyRenderingFiber.memoizedState = workInProgressHook = hook;
  } else {
    // 有多个 hooks
    // 把hooks连接起来
    workInProgressHook = workInProgressHook.next = hook;
  }
  return workInProgressHook;
}
```

hook 对象虽然属性相同，但是保存的值（hook.memoizedState）并非相同的性质

- useState 中 保存 state 信息
- useEffect 中 保存着 effect 对象
- useMemo 中 保存的是缓存的值和 deps
- useRef 中 保存的是 ref 对象

假如一个组件中有多个 hooks，每个 hooks 内部执行 `mountWorkInProgressHook` ，然后每一个 hook 通过 next 和下一个 hook 建立起关联，最后在 fiber 上的结构会变成这样：
![](https://pic.imgdb.cn/item/6247e0d027f86abb2a514575.jpg)

### hooks 更新

更新时会首先取出 `workInProgress.alternate`（即 current） 里面对应的 hook ，然后根据之前的 hooks 复制一份，形成新的 hooks 链表关系。
这个过程中解释了一个问题，就是 hooks 规则，hooks 为什么要通常放在顶部，hooks 不能写在 if 条件语句中？

因为在更新过程中，如果通过 if 条件语句，增加或者删除 hooks，在复用 hooks 过程中，`会产生复用 hooks 状态和当前 hooks 不一致的问题`。左右两棵树不能一一对应，就不能正常读取 state 等信息。
因此要始终保持执行过程中 hooks 数量一致、不能动态变化。

![](https://pic.imgdb.cn/item/6247e26927f86abb2a54716b.jpg)

更新 hook 时会调用 updateWorkInProgressHook 函数。和 mountWorkInProgressHook 一样，updateWorkInProgressHook 也是在更新的 hook 中被触发的，比如 updateReducer：

```ts
function updateReducer<S, I, A>(
  reducer: (S, A) => S,
  initialArg: I,
  init?: I => S,
): [S, Dispatch<A>] {
  const hook = updateWorkInProgressHook();
  // ...
  return [hook.memoizedState, dispatch];
}
```

而这个函数的主要流程，其实就是找到之前的 hooks 链表并复用。
另外还有一点，这个函数中有几个变量需要注意：

- currentHook、nextCurrentHook：表示在 current 树的对应节点中，走到的 hook 或下一个要走的 hook。这四个值其实都是 hook 链表的节点
- workInProgressHook、nextWorkInProgressHook：表示在 workInProgress 节点中，走到的 hook 或下一个要走的 hook。

nexthook 通常会是 currenthook 的下一个，并且在后面也会将 currenthook 改变为 nexthook。也就是说，这个函数每执行一次，就会将两个 hook 链表的遍历位置向后一个。这样只要一开始执行函数，两边的 hooks 就应该是一一对应、可以复用的，如果出现一边为 null 的情况，就说明 hook 没有对应上。

```ts
function updateWorkInProgressHook(): Hook {
  let nextCurrentHook: null | Hook;
  // 找到更新前的 hook 链
  // currentHook是一个全局变量，表示当前正在处理的hook，是hook链表的节点
  if (currentHook === null) {
    // 如果当前是hook链表中的第一个hook，那就从currentFiber上取下来hook链表作为nextCurrentHook
    const current = currentlyRenderingFiber.alternate;
    if (current !== null) {
      nextCurrentHook = current.memoizedState;
    } else {
      nextCurrentHook = null;
    }
  } else {
    // 如果当前hook不是第一个，那nextCurrentHook就代表下一个hook
    nextCurrentHook = currentHook.next;
  }
  // 当前的 hook 链
  let nextWorkInProgressHook: null | Hook;
  // workInProgressHook指的是正在调和的这个fiber的hook
  if (workInProgressHook === null) {
    // 如果是更新阶段，workInProgressHook一定是null
    // 因为更新阶段执行的renderWithHooks将当前正在构建的workInProgressFiber.memorizedState置空了
    nextWorkInProgressHook = currentlyRenderingFiber.memoizedState;
  } else {
    // 只有是重渲染的情况下，才不是null，直接复用
    nextWorkInProgressHook = workInProgressHook.next;
  }

  if (nextWorkInProgressHook !== null) {
    workInProgressHook = nextWorkInProgressHook;
    nextWorkInProgressHook = workInProgressHook.next;
    currentHook = nextCurrentHook;
  } else {
    // 如果本该对应的current的hook为null，说明本次渲染的hook多了，就会报错
    if (nextCurrentHook === null) {
      throw new Error("Rendered more hooks than during the previous render.");
    }
    currentHook = nextCurrentHook;
    // 复用原来的 hook 逻辑
    const newHook: Hook = {
      memoizedState: currentHook.memoizedState,

      baseState: currentHook.baseState,
      baseQueue: currentHook.baseQueue,
      queue: currentHook.queue,

      next: null,
    };
    // 将 hook 链 连接到 fiber 中。这个步骤和第一次创建相同
    if (workInProgressHook === null) {
      currentlyRenderingFiber.memoizedState = workInProgressHook = newHook;
    } else {
      workInProgressHook = workInProgressHook.next = newHook;
    }
  }
  return workInProgressHook;
}
```

## hooks 和 state （useState）

### updateQueue

在讲 useState 之前，要先说明一个非常重要的结构--updateQueue

update 是计算 state 的最小单位。对于函数组件来说，update 绝大多数情况下产生于 dispatch，即 useState 的第二个返回值；
而 updateQueue 是一种保存 update 的结构，它在第一次执行 useState 时（mountState）就被创建，作为 hook.queue 属性保存在 hook 上。
数据结构如下：

```ts
const fiber = {
  //...其他fiber属性
  memorizedState: { // hook链表
    baseState: null, // 参与计算的初始state
    baseQueue: null, //
    next: null, // 下一个 hooks对象
    memorizedState: state // 存储的state
    queue: { // updateQueue，里面是queue对象
      pending: { // 待更新队列，里面是update对象组成的链表
        lane: Lane, // 更新优先级
        action: A,  // setState的参数，函数或state
        hasEagerState: boolean, // 是否有最新的state
        eagerState: S | null, // 最新state
        next: Update<S, A>, // 下一个update
      }
      lanes: Lanes, // update的lanes的合并
      dispatch: (A => mixed) | null, // useState返回的dispatch
      // 上一次渲染时用的reducer，这个就相当于useReducer传入的reducer。
      // 只不过useState是预设好的reducer，而useReducer是自定义的reduer
      lastRenderedReducer: ((S, A) => S) | null,
      lastRenderedState: S | null,// 上一次渲染时用的state
    }
  }
}
```

fiber.memorizedState.queue 就是 updateQueue 的结构。重要属性：

- pending ：存放着更新组成的成环链表。
- baseState ：上一次的 state，将根据这个属性计算出本次的 state
- baseQueue ：保存了上一次更新时由于优先级太低而没有处理的 update，将在本次更新中和 queue.pending 合并。

baseQueue 和 pending 都是环链表的形式。在更新步骤中（updateState），会将他们合并到 baseQueue 中成为一个链表，用于表示该 fiber 上要更新的 queue；如果本次还是因为低优先级没有及时处理，那就会延到下一次更新。这也是 baseQueue 在 fiber 上的原因，因为每次更新 queue 要重置，要想保存上次的更新队列就只能放在 fiber 上。

合并之后，在本次更新流程中，就会从 baseState 开始，遍历 baseQueue，对每个 update 对象，取上面的 eagerState 或 action 得到 state，一个一个替换，最后就会得到一个最终的 state。用这个最终 state 再更新 fiber 上的值并返回，就完成了 state 的更新过程。

#### 产生 update

每一次执行 setState 操作就会产生一个 update，通常一次更新可能有多种触发方式，这里以事件触发的 setState 为例。
如果是通过事件回调触发的更新，那么这个 update 不会被立即消费，而是会先插入到 updateQueue 中，然后调用 scheduleUpdateOnFiber 去调度一次更新。由于调度的异步性，这次更新可能会在稍后执行。因此在这段时间内，有两种情况：

- 当前 fiberNode 上不存在未被消费的 update：这时这个 update 就会和自己形成一个环链表
- 当前 fiberNode 上存在未被消费的 update：将 update 插入到链表中

这个”未被消费的 update“的链表，就是 queue.pending。

```js
const pending = queue.pending;
if (pending === null) {
  update.next = update;
} else {
  update.next = pending.next;
  pending.next = update;
}
queue.pending = update;
```

#### 消费 update

所谓消费，其实就是在 updateReducer 方法中，把 update 对象存储的 state 取出来作为 fiber 上的 memorizedState 并返回最终结果的过程，可以理解为“用了”update。

update 有自己的优先级，而如果本次更新遇到了优先级不足的更新，那就会把不足的优先级及其后的所有更新都推迟到下一次更新。
举个例子，假如有这样四个更新，更新的数字表示优先级，数字越小优先级越高

```
A1 -> B2 -> C1 -> D2
```

> 这里有一个概念：优先级“不足”；不足并不等于“低”，虽然大多数时候确实是因为低而不足，但是不足的比较方式并不是比较大小，而是和 renderLanes 相与
> 每个更新有一个 updateLane，每次 render 也有一个 renderLanes（在每次调和 workLoop 开始时就确定了，一般初始化为 NoLanes），每次的比较是这样的：
>
> ```ts
> isSubsetOfLanes(renderLanes, updateLane);
> export function isSubsetOfLanes(set: Lanes, subset: Lanes | Lane): boolean {
>   return (set & subset) === subset;
> }
> ```
>
> 如果 subset = 0，那么上面的函数肯定返回 true。因此只要把 updateLane 改为 0，就一定可以让优先级“足够”

那么当消费完 A，到了 B 的时候，这时 B 的优先级不足，因此它和它后面所有的更新都会被放入 baseQueue 中，并且他们的优先级还会被置为 NoLanes，这样在下一次更新一定会被消费。下一次更新一般就是下一次 render

**但是要注意的是，虽然 C 也被放入了 baseQueue，但是 C 的优先级和 A 相同；因此在这一次执行中，C 也会被消费，并把结果和 A 合并。因此本次消费完成后的计算得到的 state（memorizedState）是 AC 而不是 A。**

**因此在每一次更新中，会遍历更新队列，执行优先级足够且相同的更新，把最终结果作为 memorizedState；同时从第一个优先级不足的更新开始，将后面的更新全部放入 baseQueue 中，等待下一次更新时消费。**

更新完成后是这样：

```
{
  baseState: A
  baseQueue: B0 -> C0 -> D0, // 所有优先级被置为0，下一次一定会执行
  memorizedState: AC
}
```

如果没有更新被跳过，那么 memorizedState 和 baseState 应该是一样的。

---

具体的消费过程，主要在 updateReducer 函数内，详见下面 updateState 部分

### mountState

上面说过每个 hook 实际上都是一个保存有各种属性的对象，其中就包含有 state 相关的。
下面以 useState 为例，介绍 state 是如何存储和更新的。

当挂载阶段调用 useState 时，会进入 mountState。这个函数主要做了：

1. 连接 fiber ，获得当前 hook 对象（上面 mountWorkInProgressHook）
2. 创建 queue 对象，绑定 dispatchSetState 方法
3. 返回当前 state / dispatchSetState

```ts
function mountState(initialState){
  const hook = mountWorkInProgressHook();
  if (typeof initialState === 'function') {
    initialState = initialState();
  }
  hook.memoizedState = hook.baseState = initialState;
  const queue = { // 初始化 updateQueue
    pending: null, // 当前更新
    interleaved: null, // 交错更新
    lanes: NoLanes, // 更新优先级
    dispatch: null,
    lastRenderedReducer: basicStateReducer, // reducer，这个就相当于useReducer传入的reducer。只不过useState是预设好的reducer，而useReducer是自定义的reduer
    lastRenderedState: (initialState: any),
  };
  hook.queue = queue;
  const dispatch = (queue.dispatch = (dispatchSetState.bind(
      null,
      currentlyRenderingFiber, // 当前 fiber
      queue,
   ));
  return [hook.memoizedState, dispatch];
}
// useState使用的reducer
// action就是传入setState的函数或变量
function basicStateReducer(state: S, action): S {
  return typeof action === 'function' ? action(state) : action;
}
```

### dispatch

dispatchSetState 就是调用 dispatch 时触发的函数，也是 updateState 的 dispatch。这个函数的主要工作有：

1. 获取更新优先级、创建更新对象 update 等准备工作
2. 最重要的一个工作，即形成和添加 updateQueue。

```ts
function dispatchSetState<S, A>(
  fiber: Fiber,
  queue: UpdateQueue<S, A>,
  action: A,
) {
  // 获取当前更新优先级，忽略这个方法，现在只需知道会返回一个优先级
  const lane = requestUpdateLane(fiber);
  // 生成 update 对象
  const update: Update<S, A> = {
    lane,
    action, // 在useState中，action就是dispatch传入的新的状态或一个函数
    hasEagerState: false,
    eagerState: null,
    next: (null: any),
  };

  if (isRenderPhaseUpdate(fiber)) {
    // 当前 fiber 正在render阶段
    // 在render阶段的更新，不会触发调度更新，而是会直接把本次更新合并到pending中，在useState消费时就能获取到本次更新的效果
    enqueueRenderPhaseUpdate(queue, update);
  } else {
    // 不在调和阶段
    // 将 update 形成环状链表， 保存在 queue.pending 属性中
    enqueueUpdate(fiber, queue, update, lane);

    const alternate = fiber.alternate;
    if (
      fiber.lanes === NoLanes &&
      (alternate === null || alternate.lanes === NoLanes)
    ) {
      // 当前 fiber 没有更新任务
      // 那么会拿出上一次 state 和 这一次 state 进行对比，
      // 如果相同，那么直接退出更新
      // 这就解释了，为什么函数组件 useState 改变相同的值，组件不更新了
      const lastRenderedReducer = queue.lastRenderedReducer;
      if (lastRenderedReducer !== null) {
        let prevDispatcher;
        try {
          const currentState: S = (queue.lastRenderedState: any);
          const eagerState = lastRenderedReducer(currentState, action);

          update.hasEagerState = true;
          update.eagerState = eagerState;
          if (is(eagerState, currentState)) {
            return;
          }
        } catch (error) {
          // Suppress the error. It will throw again in the render phase.
        } finally {
        }
      }
    }
    // 请求调度更新
    const root = enqueueConcurrentHookUpdate(fiber, queue, update, lane);
    if (root !== null) {
      const eventTime = requestEventTime();
      scheduleUpdateOnFiber(root, fiber, lane, eventTime);
      entangleTransitionUpdate(root, queue, lane);
    }
  }
}
```

### updateState / updateReducer

如果不是第一次更新，那么后面的 useState 就会被更换为 updateState。这个函数内部实际调用的是 updateReducer
这个函数主要流程：

1. 复用更新前 hook，拿到更新 queue，拿到上一次更新优先级不够的 baseQueue
2. 将本次更新 queue 与 baseQueue 合并成一个环形链表， 赋给 baseQueue 变量
3. 遍历 baseQueue ，存在两种情况
4. 优先级够 判断之前有没有优先级不够的更新，如果有的话保存当前更新（为了保证 react 在运行中结果唯一，不会因为更新优先级影响结果）
5. 优先级不够，保存当前更新
6. 存在优先级不够的更新，将其形成环状链表，赋给 fiber.baseQueue
7. 返回当前计算 state

这部分的原理可以看上面的 updateQueue 讲解。

updateState 其实就是 useState。当触发更新、调度和新一轮的调和使得函数组件再一次执行时，就会执行 updateState；这时，之前每一次调用 setState 都会在当前 fiber 的 queue 上添加一个更新，这里的步骤就是把这些更新计算出一个最终结果，作为最新的 state 并返回，这样本次后续的调和就会得到这个最新的 state，从而形成视图上的更新。
这里就是 useState“消费”updateQueue 的地方

```ts
function updateState<S>(
  initialState: (() => S) | S,
): [S, Dispatch<BasicStateAction<S>>] {
  return updateReducer(basicStateReducer, (initialState: any));
}

function updateReducer<S, I, A>(
  reducer: (S, A) => S,
  initialArg: I,
  init?: I => S,
): [S, Dispatch<A>] {
  // 获取复用后的 hook
  const hook = updateWorkInProgressHook();
  // 拿到queue
  const queue = hook.queue;
  // 设置reducer。reducer实际上就是一个接受旧state传出新state的函数
  queue.lastRenderedReducer = reducer;

  const current: Hook = (currentHook: any);

  // 上一次执行因为优先级不够而没执行完的任务，称作baseQueue
  let baseQueue = current.baseQueue;

  // 本次待更新队列
  const pendingQueue = queue.pending;
  if (pendingQueue !== null) {
    // 将pendingQueue和baseQueue合并
    if (baseQueue !== null) {
      const baseFirst = baseQueue.next;
      const pendingFirst = pendingQueue.next;
      baseQueue.next = pendingFirst;
      pendingQueue.next = baseFirst;
    }
    // 合并之后把合并完成的放在current.baseQueue上，并把wip.queue.pending置空
    current.baseQueue = baseQueue = pendingQueue;
    queue.pending = null;
  }

  // 准备遍历baseQueue，即旧的baseQueue和pendingQueue合并的新baseQueue
  // 遍历完成后，如果优先级够且之前没有优先级不够的更新，那就会得到最新的state，赋给hook.memorizedState作为更新结果
  if (baseQueue !== null) {
    const first = baseQueue.next;
    let newState = current.baseState;

    let newBaseState = null;
    let newBaseQueueFirst = null; // 这两个变量分别指向newBaseQueue的首尾
    let newBaseQueueLast = null;
    let update = first;
    // 遍历的对象是updateQueue中的update
    // 这里边所有update都是针对同一个state的，所以只需要遍历update，计算出最后的结果作为最新state就可以，这也是批量更新的实现之一
    do {
      const updateLane = update.lane;
      if (!isSubsetOfLanes(renderLanes, updateLane)) {
        // 如果优先级不足
        // 先复制一份
        const clone: Update<S, A> = {
          lane: updateLane,
          action: update.action,
          hasEagerState: update.hasEagerState,
          eagerState: update.eagerState,
          next: (null: any),
        };

        // 把优先级不足的update加入到newBaseQueue中
        if (newBaseQueueLast === null) {
          newBaseQueueFirst = newBaseQueueLast = clone; // 创建newBaseQueue，首尾都是当前update
          newBaseState = newState;
        } else {
          // 将更新插入newBaseQueue尾部
          newBaseQueueLast = newBaseQueueLast.next = clone;
        }
        // 将这个优先级不足的更新的lane和本次渲染的lane合并，这样下一次优先级一定足够，就可以执行了
        currentlyRenderingFiber.lanes = mergeLanes(
          currentlyRenderingFiber.lanes,
          updateLane,
        );
        markSkippedUpdateLanes(updateLane);
      } else {
        // 有足够的优先级

        if (newBaseQueueLast !== null) {
          // 如果之前有优先级不够的更新，那么这个更新也要放入newBaseQueue中
          const clone: Update<S, A> = {
            lane: NoLane,
            action: update.action,
            hasEagerState: update.hasEagerState,
            eagerState: update.eagerState,
            next: (null: any),
          };
          newBaseQueueLast = newBaseQueueLast.next = clone;
        }
        // 无论之前有没有优先级不够的更新，当前更新都会被计算到newState，保证在一次更新中，所有优先级足够的更新都计算
        // 计算state，并赋给newState
        if (update.hasEagerState) {
          newState = ((update.eagerState: any): S);
        } else {
          const action = update.action;
          newState = reducer(newState, action);
        }
      }
      update = update.next;
    } while (update !== null && update !== first);

    if (newBaseQueueLast === null) {
      // 如果没有还未执行的更新，那baseState和memoizedState就应该都是newState，即计算的state值
      newBaseState = newState;
    } else {
      // 反之，那么把newBaseQueue首尾相连，赋给baseQueue，等到下一次render消费queue再执行
      newBaseQueueLast.next = (newBaseQueueFirst: any);
    }

    // 更新state
    hook.memoizedState = newState; // memoizedState是真实state，包含了本次全部优先级足够的更新的计算结果（即newState）
    hook.baseState = newBaseState; // newBaseState只记录不在baseQueue中、被消费了的更新，优先级足够但没被消费的不算
    hook.baseQueue = newBaseQueueLast;

    queue.lastRenderedState = newState;
  }

  // 这个dispatch和mountState里的相同，是挂载在queue上的
  const dispatch: Dispatch<A> = (queue.dispatch: any);
  // 更新之后的useState返回值已经是计算的state了
  return [hook.memoizedState, dispatch];
}
```

### eagerState

## hooks 和 effect （useEffect）

### mountEffect

在 mount 阶段 useEffect / useLayoutEffect 本质上都是调用 mountEffectImpl 方法，唯一的区别是 fiberFlags 入参不同。

```ts
function mountEffect(
  create: () => (() => void) | void,
  deps: Array<mixed> | void | null
): void {
  return mountEffectImpl(
    PassiveEffect | PassiveStaticEffect,
    HookPassive,
    create,
    deps
  );
}

function mountLayoutEffect(
  create: () => (() => void) | void,
  deps: Array<mixed> | void | null
): void {
  let fiberFlags: Flags = UpdateEffect | LayoutStaticEffect;
  return mountEffectImpl(fiberFlags, HookLayout, create, deps);
}
```

初始化执行的函数如下：

```ts
function mountEffectImpl(
  fiberFlags: Flags, // 这个入参决定是哪种effect
  hookFlags: HookFlags,
  create: () => (() => void) | void,
  deps: Array<mixed> | void | null
): void {
  const hook = mountWorkInProgressHook(); // 创建hook

  const nextDeps = deps === undefined ? null : deps;
  currentlyRenderingFiber.flags |= fiberFlags;
  hook.memoizedState = pushEffect(
    HookHasEffect | hookFlags,
    create,
    undefined,
    nextDeps
  );
}
```

pushEffect 会创建 effect 对象，effect 会和其他 effect 形成一个单向环链表，并保存在 fiber 上

effect 相关的数据结构如下：

```ts
const fiber = {
  updateQueue: {
    events,
    stores,
    memoCache,
    lastEffect:{ // effect链表
      tag,
      create,
      destroy,
      deps,
      next: (null: any),
    }
  }
}
```

```ts
function pushEffect(
  tag: HookFlags,
  create: () => (() => void) | void,
  destroy: (() => void) | void,
  deps: Array<mixed> | void | null,
): Effect {
  const effect: Effect = {
    tag, // 用于区分effect类型 Passive | Layout | Insertion
    create, // useEffect的第一个参数，即回调
    destroy, // create的返回值
    deps, // 依赖数组
    next: (null: any),
  };
  let componentUpdateQueue: null | FunctionComponentUpdateQueue = (currentlyRenderingFiber.updateQueue: any);
  if (componentUpdateQueue === null) {
    // 如果没有updateQueue，就创建一个并挂载到fiber上，并把当前effect插入
    componentUpdateQueue = createFunctionComponentUpdateQueue();
    currentlyRenderingFiber.updateQueue = (componentUpdateQueue: any);
    componentUpdateQueue.lastEffect = effect.next = effect;
  } else {
    // 如果有updateQueue，就取它上面的laseEffect，即effect链表的尾部
    const lastEffect = componentUpdateQueue.lastEffect;

    if (lastEffect === null) {
      // 同样，没有就创建effect链表
      componentUpdateQueue.lastEffect = effect.next = effect;
    } else {
      // 有，就把effect接到链表上
      const firstEffect = lastEffect.next;
      lastEffect.next = effect;
      effect.next = firstEffect;
      componentUpdateQueue.lastEffect = effect;
    }
  }
  return effect;
}
```

effect 链表的连接方式，是这样的：
我们以数字表示 effect 的顺序，数字越小表示越早连接

```
lastEffect -> e4 -> e3 -> e2 -> e1 -> e0
            |                       |
            <- <- <- <- <- <- <- <- <-
```

即 lastEffect 是链表的头结点，指向的是最晚插入链表的；lastEffect.next 是最新插入的，如果有新的 effect，也是插入到这两者之间。

### updateEffect

更新阶段其实也是调用 updateEffectImpl。updateEffectImpl 和 mountEffectImpl 的差别很小，可以说唯一的区别就是 updateEffectImpl 会判断 deps 是否改变，如果不改变就不更新 effect 对象。
但即使前后的 deps 数组相同，也还是会创建 effect，保证更新前后 effect 数量相同。而区分 create 函数是否应该执行靠的是 HookHasEffect flag。在之后 commit 阶段执行 effect 时，会根据这个 flag 确定是否要执行 create 函数。

判断改变不改变的是 areHookInputsEqual 函数，其实就是一个遍历两个数组进行浅比较的函数。

```ts
function updateEffect(
  create: () => (() => void) | void,
  deps: Array<mixed> | void | null
): void {
  return updateEffectImpl(PassiveEffect, HookPassive, create, deps);
}
function updateLayoutEffect(
  create: () => (() => void) | void,
  deps: Array<mixed> | void | null
): void {
  return updateEffectImpl(UpdateEffect, HookLayout, create, deps);
}

function updateEffectImpl(fiberFlags, hookFlags, create, deps): void {
  const hook = updateWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  let destroy = undefined;

  if (currentHook !== null) {
    // 更新前的 hook
    const prevEffect = currentHook.memoizedState;
    destroy = prevEffect.destroy;
    if (nextDeps !== null) {
      const prevDeps = prevEffect.deps;
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        // 如果前后的deps数组相同
        // 会进入pushEffect，但没有HookHasEffect，create不会执行
        hook.memoizedState = pushEffect(hookFlags, create, destroy, nextDeps);
        return;
      }
    }
  }

  currentlyRenderingFiber.flags |= fiberFlags;
  // 前后deps不同，会打上HookHasEffect的flag
  hook.memoizedState = pushEffect(
    HookHasEffect | hookFlags,
    create,
    destroy,
    nextDeps
  );
}
```

通过控制不同的 flags，在 commit 阶段结束后异步调用 useEffect 时，就可以控制是否执行 create 函数了。

## useRef

`useRef` 就是创建并维护一个 ref 原始对象；

```js
function mountRef(initialValue) {
  const hook = mountWorkInProgressHook();
  const ref = { current: initialValue };
  hook.memoizedState = ref; // 创建ref对象。
  return ref;
}

function updateRef(initialValue) {
  const hook = updateWorkInProgressHook();
  return hook.memoizedState;
}
```

ref 的核心并不在于 useRef 这个 api，而是在于 React 对 ref 本身的处理。

## useMemo

`useMemo`会缓存一个值，如果依赖项改变就更新这个值。useMemo 传入的 create 的返回值将会保存在 hook.memoizedState 上

```js
function mountMemo(nextCreate, deps) {
  const hook = mountWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  const nextValue = nextCreate(); // 执行第一个参数，即一个返回要缓存的值的函数，获取这个缓存值
  hook.memoizedState = [nextValue, nextDeps]; // 向hooks状态中添加值和依赖
  return nextValue;
}
```

如果发生更新，会执行类似`useEffect`的对比，对比两次的依赖值，如果发生变化就重新调用第一个函数，即更新值；

```js
function updateMemo(nextCreate, nextDeps) {
  const hook = updateWorkInProgressHook();
  const prevState = hook.memoizedState;
  const prevDeps = prevState[1]; // 之前保存的依赖
  if (areHookInputsEqual(nextDeps, prevDeps)) {
    //判断两次依赖
    return prevState[0]; // 如果不变就正常返回值
  }
  const nextValue = nextCreate(); // 如果依赖项，发生改变，重新执行
  hook.memoizedState = [nextValue, nextDeps];
  return nextValue;
}
```

## useCallback

useCallback 和 useMemo 的基础逻辑类似，只是 Memo 保存的是返回值，而 callback 保存的是函数本身。

```ts
function mountCallback<T>(callback: T, deps: Array<mixed> | void | null): T {
  const hook = mountWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  hook.memoizedState = [callback, nextDeps]; // 不执行函数，直接保存函数
  return callback;
}

function updateCallback<T>(callback: T, deps: Array<mixed> | void | null): T {
  const hook = updateWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  const prevState = hook.memoizedState;
  if (prevState !== null) {
    if (nextDeps !== null) {
      const prevDeps: Array<mixed> | null = prevState[1];
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        return prevState[0];
      }
    }
  }
  hook.memoizedState = [callback, nextDeps];
  return callback;
}
```

# React Concurrent

Concurrent 有几个方面：

1. createRoot。在旧的 React 版本中没有专门的 RootFiber 这个概念，在 Concurrent 模式下，React 组件树需要先通过调用 createRoot 创建一个 root，再去调用 render 做实际的渲染。可以参考`https://bytedance.feishu.cn/wiki/wikcnl5V0pWZDtLC6ljFpDWlj93`，这里讲了 React 的运行流程

> 为什么需要 createRoot 创建根节点，而不是老版本的 ReactDOM.render？
>
> - createRoot 方式把 react 创建的 FiberRootNode 暴露了出来，将 render 方法分离，便于使用者控制，比如官网上提到的[只渲染部分内容](https://zh-hans.react.dev/reference/react-dom/client/createRoot#rendering-a-page-partially-built-with-react)
> - 对于 ssr，react 还提供了 hydrateRoot。将 root 的创建分离出来，也便于通过 hydrateRoot 来实现渲染
> - createRoot 内部实现了一些 React 18 的内容，同时也有对 concurrent 的支持。FiberRootNode 上的一些属性是支持 concurrent 所必须的，比如 root 的更新优先级、特殊的 lane 等。

2. 调度，ensureRootIsScheduled 里的机制

- scheduler 的异步调度和时间分片
- 自动批处理
- 如何开启 concurrent 调度：concurrentRoot 和 shouldTimeSlice

3. 调和

- performConcurrentWorkOnRoot
- workLoopConcurrent，可中断

4. lane

- getNextLanes 和 markStarvedLanesAsExpired
- pendingLanes 机制和修改
- lane 过期机制

## 从调度角度看

从调度角度看，其实是 ensureRootIsScheduled 函数内的情况。下面将会频繁引用该函数内的片段。

### 理清调度逻辑

调度这一块的逻辑非常复杂，为了方便理清逻辑，这里采用书上的一个模拟实现。
我们设置两个主要函数，Schedule 函数用于代表 ensureRootIsScheduled，perform 函数用于代表 performSyncWorkOnRoot，scheduleCallback 就采用原版的函数，代码如下：

接下来是一些设定：

- 每个 work 对象有一个优先级，模拟更新任务的优先级机制；
- Schedule 函数的主要流程是：
  1. 获取当前正在调度的任务的 callback，这个下面讲 callback 机制会讲到
  2. 通过对 workList 排序，取出最高优先级的任务
  3. 比较取出的任务和之前的任务的优先级，如果相同就返回。
  4. 如果不同执行 cancelCallback 中断之前的任务
  5. 调用 scheduleCallback 调度当前优先级最高的任务。scheduleCallback 函数会返回创建的 task 对象，即以当前 work 为 callback 的 task，也就是一开始获取的那个
- perform 函数：接受具体任务和一个 didTimeout
  1. 先判断是否需要同步执行，满足 1.工作是同步优先级 2.当前调度的任务过期了，需要同步执行
  2. 通过 while 循环执行任务，这里模拟的是调和里的 workLoopConcurrent 函数
  3. 如果任务执行完，相当于调和过程完毕了，那就把任务剔除。如果没执行完就先保存任务，然后调用 schedule 函数在进行一次调度
  4. 对比两次 callback，如果不同说明当前任务被替换了，然后在 workLoop 中就会执行下一个任务。
- workLoop：这个是调度的，类比源码中的 workLoop 就行

```ts
btn.onclick = () => {
  // 插入工作
  workList.push({
    priority,
    count: 100,
  });
  schedule();
};

/**
 * 调度的逻辑
 */
function schedule() {
  // 当前可能存在正在调度的回调
  // cbNode就是task对象
  const cbNode = getFirstCallbackNode();
  // 取出最高优先级的工作
  const curWork = workList.sort((w1, w2) => {
    return w1.priority - w2.priority;
  })[0];

  if (!curWork) {
    // 没有工作需要执行，退出调度
    curCallback = null;
    cbNode && cancelCallback(cbNode);
    return;
  }

  const { priority: curPriority } = curWork;

  if (curPriority === prevPriority) {
    // 有工作在进行，比较该工作与正在进行的工作的优先级
    // 如果优先级相同，则不需要调度新的，退出调度
    return;
  }

  // 准备调度当前最高优先级的工作
  // 调度之前，如果有工作在进行，则中断他
  cbNode && cancelCallback(cbNode);

  // 调度当前最高优先级的工作
  curCallback = scheduleCallback(curPriority, perform.bind(null, curWork));
}

// 执行具体的工作
function perform(work: Work, didTimeout?: boolean): any {
  // 是否需要同步执行，满足1.工作是同步优先级 2.当前调度的任务过期了，需要同步执行
  const needSync = work.priority === ImmediatePriority || didTimeout;
  while ((needSync || !shouldYield()) && work.count) {
    work.count--;
    // 执行具体的工作
    insertItem(work.priority + "");
  }
  prevPriority = work.priority;

  if (!work.count) {
    // 完成的work，从workList中删除
    const workIndex = workList.indexOf(work);
    workList.splice(workIndex, 1);
    // 重置优先级
    prevPriority = IdlePriority;
  }

  const prevCallback = curCallback;
  // 调度完后，如果callback变化，代表这是新的work
  schedule();
  const newCallback = curCallback;

  if (newCallback && prevCallback === newCallback) {
    // callback没变，代表是同一个work，只不过时间切片时间用尽（5ms）
    // 返回的函数会被Scheduler继续调用
    return perform.bind(null, work);
  }
}

// workLoop简化，留下关键部分
function workLoop(hasTimeRemaining: boolean, initialTime: number) {
  currentTask = peek(taskQueue);
  while (
    currentTask !== null &&
    !(enableSchedulerDebugging && isSchedulerPaused)
  ) {
    if (
      currentTask.expirationTime > currentTime &&
      (!hasTimeRemaining || shouldYieldToHost())
    ) {
      break;
    }
    const callback = currentTask.callback;
    if (typeof callback === "function") {
      currentTask.callback = null;
      currentPriorityLevel = currentTask.priorityLevel;
      const didUserCallbackTimeout = currentTask.expirationTime <= currentTime;

      const continuationCallback = callback(didUserCallbackTimeout);
      if (typeof continuationCallback === "function") {
        currentTask.callback = continuationCallback;
        return true;
      } else {
        if (currentTask === peek(taskQueue)) {
          pop(taskQueue);
        }
      }
    } else {
      pop(taskQueue);
    }
    currentTask = peek(taskQueue);
  }

  if (currentTask !== null) {
    return true;
  } else {
    const firstTimer = peek(timerQueue);
    if (firstTimer !== null) {
      requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime);
    }
    return false;
  }
}
```

### callback 和 cancelCallback 机制

在前面的调度讲解中，说到过 scheduleCallback 会为传入的 Callback 创建一个 task，同时也会返回这个 task

```ts
var newTask: Task = {
  id: taskIdCounter++,
  callback,
  priorityLevel,
  startTime,
  expirationTime,
  sortIndex: -1,
};
```

这个 task 是 taskQueue、timerQueue 中的值。callback 就是创建时传入的，通常是 performConcurrentWorkOnRoot 或 performSyncWorkOnRoot 这样的调和函数。
这个 callback 有什么用呢？这里要综合上面的代码来看。

#### callback

先看 curCallback 这个全局变量。它被赋值是执行完 scheduleCallback 之后，即新创建的 task 对象。虽然 scheduleCallback 函数调度的任务是微任务，但是这个函数本身是同步的，也就是说只要执行这个函数就会更新 curcallback。
那什么时候回执行 scheduleCallback 呢？有两种情况：

- 第一次执行 schedule 函数，也就是第一个任务。此时 curCallback 就是这个任务创建的 task
- 在已经有一个任务的基础上，来了一个优先级更高的任务。因为优先级相等或更低的任务会被拦截（这个机制参考调度阶段），只有优先级更高、并且不是 Synclane 的任务到来。这时会重新创建 task，然后更新 curCallback

然后再看 perform 函数

```ts
const prevCallback = curCallback;
// 调度完后，如果callback变化，代表这是新的work
schedule();
const newCallback = curCallback;

if (newCallback && prevCallback === newCallback) {
  // callback没变，代表是同一个work，只不过时间切片时间用尽（5ms）
  // 返回的函数会被Scheduler继续调用
  return perform.bind(null, work);
}
```

1. 首先，我们通过一个事件调用 schedule 函数，经过 workList 等，最后调用 scheduleCallback。
2. scheduleCallback 会返回一个由当前任务创建的 task，然后异步调用 workLoop 去执行 callback
3. 接下来进入 callback 的流程内，即 perform 函数的上下文。
4. 这时其他任务到来时，当前任务可能还在循环里，任务被暂时放到 workList，还没有执行 schedule。
   因为这里循环模拟的是调和的 workLoop，因此退出条件有三个：

- 任务执行完
- 时间片到时，触发 shouldYield
- didTimeout 为 true，这个后面再说
  这里假设是第二种情况，那么显然还需要再调度一次，继续执行未完成的任务。
  那么当离开循环，走到这里时，先通过 prevCallback 保存一下旧的 curCallback，然后执行 schedule；
  接下来就会更新 workList、取出任务，这时有两种情况：
- 新任务优先级小于等于当前任务
- 新任务优先级高于当前任务

先说第一种，接下来的流程是：

1. 在 schedule 函数内部会被拦截，不会执行 scheduleCallback，因此不会更新 curCallback
2. 回到 perform 函数，prevCallback 和 newCallback 相等，这时会返回一个 perform.bind(null, work);，就是本函数。
3. 再回到 workLoop 函数，callback 的返回值赋给了 continuationCallback，然后在下面将 callback 设为这个值

```ts
if (typeof continuationCallback === "function") {
  currentTask.callback = continuationCallback;
  return true;
}
```

这也就意味着，callback 不为 null，还会再执行 callback，也就是再执行 perform 函数 4. 回到第三步，直到 work 被执行完。当任务执行完后，会把当前任务清除出 workList，这样下次 schedule 函数就会调用其他 work 了。

可以看到，这里形成了两个循环，一个是 schedule -> scheduleCallback -> workLoop -> perform 这个大循环，一个是 callback -> continuationCallback 这个小循环。
由于后一种循环的机制，未被高优先级打断任务可以一直执行，直到执行完为止。

#### cancelCallback

还是刚才的流程，如果新来的任务优先级高于当前任务呢？

这时因为优先级更高，不会返回，会继续向下执行，并执行这么一步：

```ts
cbNode && cancelCallback(cbNode);
```

cancelCallback 很简单，就是把 cbNode.callback 置空

```ts
function cancelCallback(task) {
  task.callback = null;
}
```

置空之后，还是进入 scheduleCallback，并更新 curCallback 的值。

再次回到 perform 函数，会发现两次的 callback 不一致了。这时 callback 不会返回任何值，也就意味着在 workLoop 函数中的 continuationCallback 也为 null

```ts
while (currentTask !== null) {
  const callback = currentTask.callback;
  if (typeof callback === "function") {
    // 再次循环时因为callback为空，也不会在走到这里
    currentTask.callback = null;

    const continuationCallback = callback(didUserCallbackTimeout);
    if (typeof continuationCallback === "function") {
      // 走不通
      currentTask.callback = continuationCallback;
      return true;
    } else {
      if (currentTask === peek(taskQueue)) {
        pop(taskQueue);
      }
    }
  } else {
    // 最后会来到这里
    pop(taskQueue);
  }
  currentTask = peek(taskQueue);
}
```

又因为 callback 也被置空了，因此再一次循环不会走到执行 callback 的地方，而是会弹出当前任务。
这时原本的 callback task 已经被弹出，接下来的 workLoop 将会执行新来的高优先级任务，和上一部分的情况一样。

**因此，cancelCallback 函数，就是取消正在调度的任务的关键。**

另外，callback 被置为 null 并不代表这个任务就完全消失了。commit 阶段的 markRootFinished 不会被调用，被取消任务的 Lanes 仍然存在于 pendingLanes 中，等待下一轮 getNextLanes 调用时安排新任务。

#### 实际情况

在实际代码中其实也是类似这样的，ensureRootIsScheduled：
workLoop 中的代码就是实际代码。

```ts
function ensureRootIsScheduled(root: FiberRoot, currentTime: number) {
  const existingCallbackNode = root.callbackNode;

  if (existingCallbackNode != null) {
    cancelCallback(existingCallbackNode);
  }

  let newCallbackNode;
  newCallbackNode = scheduleCallback(
    schedulerPriorityLevel,
    performConcurrentWorkOnRoot.bind(null, root)
  );

  root.callbackNode = newCallbackNode;
}
```

### 饥饿问题

所谓饥饿问题是指某些任务由于优先级过低导致一直被阻塞，不能执行的情况。
饥饿问题的解决有两个方面

- didTimeout
- 过期 lane

#### didTimeout

还是上面的示例代码，可以看到这里：

```ts
const needSync = work.priority === ImmediatePriority || didTimeout;
while ((needSync || !shouldYield()) && work.count) {
  work.count--;
  // 执行具体的工作
  insertItem(work.priority + "");
}
```

如果传入的 didTimeout 为 true，就会无视 shouldYield 的阻塞而一直执行直到任务完毕。
didTimeout 是 workLoop 执行 callback 时传入的，表示的是当前任务的过期与否

```ts
const didUserCallbackTimeout = currentTask.expirationTime <= currentTime;
const continuationCallback = callback(didUserCallbackTimeout);
```

当一个任务长时间未执行完，最后 didUserCallbackTimeout 就会为 true，该任务的优先级会被提升为同步；这样在下一次的 callback 中就会同步执行这个过期的 work。

实际上，didTimeout 仅用于传递给 performConcurrentWorkOnRoot，并且也仅用于 timeSlice 的生成，最终决定 render 阶段是否可以被中断。

#### 过期 Lane

这里主要是 markStarvedLanesAsExpired 函数。主要做的事情是：

- 接受一个 fiberroot，取出 root.pendingLanes，对其上每一位都设置一个过期时间
- 对已经过期的 lane，存在 root.expiredLanes 中

具体的过期时间是一个数组，保存在 root.expirationTimes 上。
给 lane 设置过期时间的主要意义是，让对应的更新任务也能确定过期时间。即持有某个 lane 的更新任务，过期时间就可以确定了。

```ts
export function markStarvedLanesAsExpired(
  root: FiberRoot,
  currentTime: number
): void {
  const pendingLanes = root.pendingLanes;
  const suspendedLanes = root.suspendedLanes;
  const pingedLanes = root.pingedLanes;
  const expirationTimes = root.expirationTimes;

  let lanes = pendingLanes & ~RetryLanes;
  while (lanes > 0) {
    // 从第一位有效位开始遍历lane
    const index = pickArbitraryLaneIndex(lanes);
    const lane = 1 << index;
    // 取出这一位之前确定的过期时间
    const expirationTime = expirationTimes[index];
    if (expirationTime === NoTimestamp) {
      // 如果没设置过过期时间
      if (
        (lane & suspendedLanes) === NoLanes ||
        (lane & pingedLanes) !== NoLanes
      ) {
        // 计算并设置过期时间，值 = currentTime + timeout，算法和scheduler的一样
        expirationTimes[index] = computeExpirationTime(lane, currentTime);
      }
    } else if (expirationTime <= currentTime) {
      // 对已经过期的lane，存在root.expiredLanes中
      root.expiredLanes |= lane;
    }

    lanes &= ~lane;
  }
}
```

### getNextLanes

这个函数虽然在前面位运算里讲过，但是在 ensureRootIsScheduled 函数内其实是比较的 root.expiredLanes 和 workInProgressRootRenderLanes

```ts
const nextLanes = getNextLanes(
  root,
  root === workInProgressRoot ? workInProgressRootRenderLanes : NoLanes
);
```

root.expiredLanes 刚刚说过，实际上是所有过期 lanes 的合并。因此 nextLane 相当于是在当前 renderLanes 和过期 Lanes 中选一个优先级高的，作为本次调度任务的优先级。
通常情况下由于过期 Lanes 优先级会升高，因此会使得优先级选为过期 Lanes，这样这些过期任务就有执行的机会。

### timeSlice

timeSlice 主要是用于控制选择哪种渲染模式的。

除了 render 的控制，shouldTimeSlice 也会影响 scheduleSyncCallback 和 scheduleConcurrentCallback 的选择，即对于调度方式的选择。
对于可能出现的某个任务长时间未执行的情况，不能一直让任务挂起不执行，而是需要改变调度方式，从 concurrent 改为 sync，保证任务同步执行，而不是一直挂起。

```ts
const shouldTimeSlice =
  !includesBlockingLane(root, lanes) &&
  !includesExpiredLane(root, lanes) &&
  (disableSchedulerTimeoutInWorkLoop || !didTimeout);
let exitStatus = shouldTimeSlice
  ? renderRootConcurrent(root, lanes)
  : renderRootSync(root, lanes);
```

shouldTimeSlice 的判断来源有三个：

- 不包含阻塞的 lane。常见的阻塞的 lane 就是 DefaultLane 和 InputContinuousLane，即第 2、3Lane
- 不包含过期的 lane。即不能存在于 root.expriedLane
- didTimeout 不能为 true，即任务不能超时但未执行

```ts
export function includesBlockingLane(root: FiberRoot, lanes: Lanes): boolean {
  if (
    allowConcurrentByDefault &&
    (root.current.mode & ConcurrentUpdatesByDefaultMode) !== NoMode
  ) {
    return false;
  }
  const SyncDefaultLanes =
    InputContinuousHydrationLane |
    InputContinuousLane |
    DefaultHydrationLane |
    DefaultLane;
  return (lanes & SyncDefaultLanes) !== NoLanes;
}

export function includesExpiredLane(root: FiberRoot, lanes: Lanes): boolean {
  // This is a separate check from includesBlockingLane because a lane can
  // expire after a render has already started.
  return (lanes & root.expiredLanes) !== NoLanes;
}
```

## 从调和角度看

调和方面相对调度来说比较少，主要是那几种 Concurrent 更新。

### render 的中断

在 Concurrent 模式下，render 可以中断，这种机制主要来自于调和的 workLoopConcurrent 函数

具体来说，是这样的关系：

```ts
newCallbackNode = scheduleCallback(
  schedulerPriorityLevel,
  performConcurrentWorkOnRoot.bind(null, root)
);

function performConcurrentWorkOnRoot(root, didTimeout) {
  const shouldTimeSlice =
    !includesBlockingLane(root, lanes) &&
    !includesExpiredLane(root, lanes) &&
    (disableSchedulerTimeoutInWorkLoop || !didTimeout);
  let exitStatus = shouldTimeSlice
    ? renderRootConcurrent(root, lanes)
    : renderRootSync(root, lanes);
}

function renderRootConcurrent(root: FiberRoot, lanes: Lanes) {
  workLoopConcurrent();
}

function workLoopConcurrent() {
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}
```

可以看到，关键就在于 workLoopConcurrent 的!shouldYield()判断。这个函数来自于 scheduler，会根据 cpu 占用等情况分析当前任务能不能被继续执行。

> shouldYield 函数实际上是 scheduler 中的 shouldYieldToHost 函数，具体参考上面的 scheduler 部分。

Concurrent 模式的选择其实来自于两次选择：

1. 根据当前的 FiberRoot 是 LegacyRoot 还是 ConcurrentRoot 决定。如果是通过 createRoot 创建的 ConcurrentRoot，就会使用 scheduleCallback 来调度任务，即开启了 Concurrent 模式。

```ts
if (root.tag === LegacyRoot) {
  // legacy模式
  scheduleLegacySyncCallback(performSyncWorkOnRoot.bind(null, root));
} else {
  // Concurrent模式
  scheduleSyncCallback(performSyncWorkOnRoot.bind(null, root));
}
```

2. performConcurrentWorkOnRoot 函数内部的选择，取决于 shouldTimeSlice 变量。大多数时候这个值都为 true，即开启时间分片；只有当某些任务过长时间得不到执行（饥饿）或者在其他地方禁用了时间分片时才会关闭时间分片。

# React FiberRoot

React 的入口在 React18 被修改

```ts
ReactDOM.createRoot(...).render(...)
```

## createRoot

createRoot 函数：

```ts
export function createRoot(
  container: Element | DocumentFragment,
  options?: CreateRootOptions
): RootType {
  // ...
  const root = createContainer(
    container,
    ConcurrentRoot,
    null,
    isStrictMode,
    concurrentUpdatesByDefaultOverride,
    identifierPrefix,
    onRecoverableError,
    transitionCallbacks
  );
  // ...
  return new ReactDOMRoot(root);
}
```

主要做了两件事

- 调用 createContainer 初始化 FiberTree
- 返回一个 ReactDomRoot 对象

createContainer 实际调用的是 createFiberRoot
初始化 FiberTree 的工作包括构建 root 节点和 host root 节点

```ts
// react-reconciler/src/ReactFiberRoot.js
export function createFiberRoot(
    // ...
): FiberRoot {
  // 创建root
  const root: FiberRoot = (new FiberRootNode(
    containerInfo,
    tag,
    hydrate,
    identifierPrefix,
    onRecoverableError,
  ): any);
  // ...
  // 创建hostRootFiber
  const uninitializedFiber = createHostRootFiber(
    tag,
    isStrictMode,
    concurrentUpdatesByDefaultOverride,
  );

  // 这里将root和host root互相关联
  root.current = uninitializedFiber;
  uninitializedFiber.stateNode = root;

  // ...
  return root;
}
```

ReacDOMRoot 会再在外面套一个壳子，然后在 ReactDOMRoot 的 prototype 上安插 render、unmount 等方法
最后，createRoot 函数返回的就是套壳之后的 ReactDOMRoot

```ts
function ReactDOMRoot(internalRoot: FiberRoot) {
  this._internalRoot = internalRoot; // 保存root
}

ReactDOMHydrationRoot.prototype.render = ReactDOMRoot.prototype.render =
  function (children: ReactNodeList): void {
    // 省略
  };

// unmount的实现
ReactDOMHydrationRoot.prototype.unmount = ReactDOMRoot.prototype.unmount =
  function (): void {
    // 省略
  };
```

## render

render 的核心逻辑在其调用的 updateContainer 方法中，做的事情可以大致分为三个部分，每个部分对应一个方法，从语意上来理解，这三个步骤是一次连贯的行为， 创建 update 对象->把 update 对象加入循环链表->根据 update 对象来更新 Fiber

```ts
ReactDOMRoot.prototype.render = function (children: ReactNodeList): void {
  const root = this._internalRoot;
  const container = root.containerInfo;
  if (
    !enableFloat &&
    !enableHostSingletons &&
    container.nodeType !== COMMENT_NODE
  ) {
    const hostInstance = findHostInstanceWithNoPortals(root.current);
  }

  updateContainer(children, root, null, null);
};
```

updateContainer 主要调用了 createUpdate 和 enqueueUpdate 创建并插入更新；然后在 FiberRoot 上通过 scheduleUpdateOnFiber 调度一次更新。
所以实际上，整个 React 组件树的第一次挂载也是一次更新，只是这时的 current 为 null

![](https://pic.imgdb.cn/item/63aba26308b6830163e39d06.jpg)

```ts
export function updateContainer(
  element: ReactNodeList,
  container: OpaqueRoot,
  parentComponent: ?React$Component<any, any>,
  callback: ?Function
): Lane {
  const current = container.current;
  const eventTime = requestEventTime();
  const lane = requestUpdateLane(current);

  const update = createUpdate(eventTime, lane);
  update.payload = { element };

  callback = callback === undefined ? null : callback;
  if (callback !== null) {
    update.callback = callback;
  }

  const root = enqueueUpdate(current, update, lane);
  if (root !== null) {
    scheduleUpdateOnFiber(root, current, lane, eventTime);
    entangleTransitions(root, current, lane);
  }

  return lane;
}
```

createUpdate 创建了一个 update 对象

```ts
export function createUpdate(eventTime: number, lane: Lane): Update<*> {
  const update: Update<*> = {
    eventTime,
    lane,

    tag: UpdateState,
    payload: null,
    callback: null,

    next: null,
  };
  return update;
}
```

enqueueUpdate 将 update 对象加入到当前 Fiber 节点的更新链表中，即加入到 updateQueue 中。
这里的和调用 dispatch 加入 update 对象的方法一致

```ts
export function enqueueUpdate<State>(
  fiber: Fiber,
  update: Update<State>,
  lane: Lane
) {
  const updateQueue = fiber.updateQueue;
  //...
  const pending = sharedQueue.pending;
  if (pending === null) {
    update.next = update;
  } else {
    update.next = pending.next;
    pending.next = update;
  }
  sharedQueue.pending = update;
  // ...
}
```

# React18 Transition

> 在大屏幕视图更新的时，startTransition 能够保持页面有响应，这个 api 能够把 React 更新标记成一个特殊的更新类型 transitions ，在这种特殊的更新下，React 能够保持视觉反馈和浏览器的正常响应。

Transition 的主要作用就是降低更新优先级，使得某些高消耗的更新延后进行，保证浏览器的流畅。

Transition 基于 concurrent 模式，即通过 createRoot 创建的 FiberRoot。

Transition 的 api 主要有三个：

- startTransition：核心 api，接受两个参数，第一个参数是用于设置 pending 的函数；第二个参数是一个回调，回调内的更新将会被降低优先级，大概率会在下一次渲染时更新
- useTransition：其实就是 startTransition 的封装，用 useState 创建了一个 pending 状态，当低优先级更新还在等待时 pending 为 true，低优先级被消费时 pending 为 false。
- useDeferedValue：接受一个状态，返回一个新状态；当原始状态更新时新状态也会更新，但是新状态的更新优先级会低于原始状态，也就是比原始状态晚一些更新。

下面来看一下三个函数的代码：

startTransition 和旧版本的批量更新差不多，通过修改`ReactCurrentBatchConfig.transition`，然后执行 callback。

```js
function startTransition(
  setPending: (boolean) => void,
  callback: () => void,
  options?: StartTransitionOptions
): void {
  // 保存之前的更新优先级
  const previousPriority = getCurrentUpdatePriority();
  // 设置优先级，即将优先级设置为TransitionLane
  setCurrentUpdatePriority(
    higherEventPriority(previousPriority, ContinuousEventPriority)
  );

  setPending(true);
  // 保存之前的Transition上下文，重置为空
  const prevTransition = ReactCurrentBatchConfig.transition;
  ReactCurrentBatchConfig.transition = {};

  try {
    setPending(false);
    callback();
  } finally {
    // 重置更新优先级和上下文
    setCurrentUpdatePriority(previousPriority);

    ReactCurrentBatchConfig.transition = prevTransition;
  }
}
```

在 callback 内的更新会通过 requestUpdateLane 降低更新优先级；这个函数在 dispatchSetState 已进入就会调用；如果之前没有更改 Transition 上下文就原样返回，如果更改了就降低优先级。

```ts
function dispatchSetState<S, A>(
  fiber: Fiber,
  queue: UpdateQueue<S, A>,
  action: A
): void {
  const lane = requestUpdateLane(fiber);
  // ...
}

export function requestUpdateLane(fiber: Fiber): Lane {
  // 获取ReactCurrentBatchConfig.transition
  const isTransition = requestCurrentTransition() !== NoTransition;
  // 如果有Transition
  if (isTransition) {
    if (currentEventTransitionLane === NoLane) {
      // 降低优先级
      currentEventTransitionLane = claimNextTransitionLane();
    }
    return currentEventTransitionLane;
  }
}

export function requestCurrentTransition(): Transition | null {
  return ReactCurrentBatchConfig.transition;
}

export function claimNextTransitionLane(): Lane {
  const lane = nextTransitionLane;
  nextTransitionLane <<= 1;
  if ((nextTransitionLane & TransitionLanes) === NoLanes) {
    nextTransitionLane = TransitionLane1;
  }
  return lane;
}
```

---

剩下的 useTransition 和 useDeferedValue 就是基于 startTransition 和 useState 的封装。
useTransition：

```ts
function mountTransition(): [
  boolean,
  (callback: () => void, options?: StartTransitionOptions) => void
] {
  const [isPending, setPending] = mountState(false);
  const start = startTransition.bind(null, setPending);
  const hook = mountWorkInProgressHook();
  hook.memoizedState = start;
  return [isPending, start];
}

// 更新时其实就是获取最新的startTransition并返回
function updateTransition(): [
  boolean,
  (callback: () => void, options?: StartTransitionOptions) => void
] {
  const [isPending] = updateState(false);
  const hook = updateWorkInProgressHook();
  const start = hook.memoizedState;
  return [isPending, start];
}
```

useDeferedValue，可以看到实际上是把插入的 value 用 useState 包裹，然后将对应的 dispatch（setValue）改为 startTransition 更新的模式就可以了。
可以看到它内部即使用了 useEffect 使更新变成异步，同时也用到了 startTransition 让更新降级，这两种方式使返回的值的更新要延后于传入的值。

```ts
function updateDeferredValue(value) {
  const [prevValue, setValue] = updateState(value);
  updateEffect(() => {
    const prevTransition = ReactCurrentBatchConfig.transition;
    ReactCurrentBatchConfig.transition = 1;
    try {
      setValue(value);
    } finally {
      ReactCurrentBatchConfig.transition = prevTransition;
    }
  }, [value]);
  return prevValue;
}
```

## Transition Lane

为了表示 Transition 模式下的更新优先级，React 额外设立了 Transition Lane；普通的 Lane 最多到第 5 位，而 Transition Lane 则覆盖了 6-21 ，一共 16 个位。用于表示足够多可能的 Transition 更新。

```ts
export const DefaultLane: Lane = /*                     */ 0b0000000000000000000000000100000;

const TransitionHydrationLane: Lane = /*                */ 0b0000000000000000000000001000000;
const TransitionLanes: Lanes = /*                       */ 0b0000000011111111111111110000000;
const TransitionLane1: Lane = /*                        */ 0b0000000000000000000000010000000;
const TransitionLane2: Lane = /*                        */ 0b0000000000000000000000100000000;
const TransitionLane3: Lane = /*                        */ 0b0000000000000000000001000000000;
const TransitionLane4: Lane = /*                        */ 0b0000000000000000000010000000000;
const TransitionLane5: Lane = /*                        */ 0b0000000000000000000100000000000;
const TransitionLane6: Lane = /*                        */ 0b0000000000000000001000000000000;
const TransitionLane7: Lane = /*                        */ 0b0000000000000000010000000000000;
const TransitionLane8: Lane = /*                        */ 0b0000000000000000100000000000000;
const TransitionLane9: Lane = /*                        */ 0b0000000000000001000000000000000;
const TransitionLane10: Lane = /*                       */ 0b0000000000000010000000000000000;
const TransitionLane11: Lane = /*                       */ 0b0000000000000100000000000000000;
const TransitionLane12: Lane = /*                       */ 0b0000000000001000000000000000000;
const TransitionLane13: Lane = /*                       */ 0b0000000000010000000000000000000;
const TransitionLane14: Lane = /*                       */ 0b0000000000100000000000000000000;
const TransitionLane15: Lane = /*                       */ 0b0000000001000000000000000000000;
const TransitionLane16: Lane = /*                       */ 0b0000000010000000000000000000000;
```

和其他 lane 的用法一致，通过这种方式就可以确定某个更新是不是 Transition 更新了

```js
return (lane & TransitionLanes) !== NoLane;
```

# React 位运算

React 中运用了很多位运算的场景，比如在更新优先级模型中采用新的 lane 架构模型，还有判断更新类型中 context 模型，以及更新标志 flags 模型等。

位运算的效率够高，并且更加灵活。比如说用于标志更新的 flags，仅仅通过二进制数字的形式就可以实现每种更新和副作用的标记，并且这些标记之间可以合并、比较、包含等，相对于之前的 effectList 要节省不少内存。

React 位运算的对象都是 32 位有符号整数（int32），实际表示的是 31 位（有一位符号位）

React 对位运算基本上用的都是三个操作：与（`&`）、或（`|`）、非（`~`），以及部分的左移和右移。

## 应用场景

比如有一个场景下，会有很多状态常量 A，B，C...，这些状态在整个应用中在一些关键节点中做流程控制，比如：

```js
if (value === A && value === B && value !== C) {
  //...
}
```

实际场景下 value 可能是好几个枚举常量的集合，也就是一对多的关系，那么此时 value 可能同时代表 A 和 B 两个属性
![](https://pic.imgdb.cn/item/63aabca008b6830163a92b2c.jpg)
可以把一些状态常量用 32 位的二进制来表示（这里也可以用其他进制），比如：

```js
const A = 0b0000000000000000000000000000001;
const B = 0b0000000000000000000000000000010;
const C = 0b0000000000000000000000000000100;
```

通过移位的方式让每一个常量都单独占一位，这样在判断一个属性是否包含常量的时候，可以根据当前位数的 1 和 0 来判断。

这样如果一个值即代表 A 又代表 B 那么就可以通过位运算的 | 来处理。就有

```
AB = A | B = 0b0000000000000000000000000000011
```

如果需要判断某个值是否属于 A 和 B 的集合，就可以让这个值和 AB 相与，检查结果是不是为全 0 的情况

```js
const A = 0b0000000000000000000000000000001;
const B = 0b0000000000000000000000000000010;
const C = 0b0000000000000000000000000000100;
const N = 0b0000000000000000000000000000000;
const value = A | B;
console.log((value & A) !== N); // true
console.log((value & B) !== N); // true
console.log((value & C) !== N); // false
```

因为 A、B 等每个值都是只占一位的二进制数，这些值通过或运算合并之后，就意味着结果中的 1 一定是这些值的位置上的 1。
比如 A 和 B 分别是第 1、第 2 位上的 1，那么 A|B 的结果就是第一、第二位上为 1，其他位为 0；此时如果是一个其他位为 1 的值，那么相与的结果一定是全 0，即 N；

```
A | B = 0011
C = 0100
C & (A | B) = 0011 & 0100 = 0000 === N
A & (A | B) = 0011 & 0001 = 0001 !== N
```

由此也就要求了，这些二进制数只能有一位为 1，并且相互之间不能相同。

## Lane

React 定义的 Lane 如下：

```ts
export const TotalLanes = 31;

// Lanes类型是Lane的合并类型
export const NoLanes: Lanes = /*                        */ 0b0000000000000000000000000000000;
export const NoLane: Lane = /*                          */ 0b0000000000000000000000000000000;

export const SyncHydrationLane: Lane = /*               */ 0b0000000000000000000000000000001;
export const SyncLane: Lane = /*                        */ 0b0000000000000000000000000000010;

export const InputContinuousHydrationLane: Lane = /*    */ 0b0000000000000000000000000000100;
export const InputContinuousLane: Lane = /*             */ 0b0000000000000000000000000001000;

export const DefaultHydrationLane: Lane = /*            */ 0b0000000000000000000000000010000;
export const DefaultLane: Lane = /*                     */ 0b0000000000000000000000000100000;
```

lane 的代表的数值越小，此次更新的优先级就越大。
lane 是可以合并的，如果在执行某个任务时有一个高优先级任务要先执行，那么此次执行的优先级就会和高优先级任务的优先级合并；再后续通过优先级分离，分离出高优先级和低优先级的任务。

lane 的比较通常是直接比较大小，但前提是这些 lane 必须是这种只有一个 1 的形式。其他几个常见的 lane 操作：

```ts
export function includesSomeLane(a: Lanes | Lane, b: Lanes | Lane): boolean {
  return (a & b) !== NoLanes;
}

// 判断set是否包含subset
export function isSubsetOfLanes(set: Lanes, subset: Lanes | Lane): boolean {
  return (set & subset) === subset;
}

// 合并a和b
export function mergeLanes(a: Lanes | Lane, b: Lanes | Lane): Lanes {
  return a | b;
}

// 从set移除subset
export function removeLanes(set: Lanes, subset: Lanes | Lane): Lanes {
  return set & ~subset;
}

export function intersectLanes(a: Lanes | Lane, b: Lanes | Lane): Lanes {
  return a & b;
}
```

### 优先级分离

React 底层就是通过 getHighestPriorityLane 分离出高优先级的任务

```js
function getHighestPriorityLane(lanes) {
  return lanes & -lanes;
}
```

通过执行这个函数，可以把合并的优先级的最高的优先级（即最低位）提取出来。
流程如下：

比如 SyncLane 和 InputContinuousLane 合并之后的任务优先级 lane 为

```
SyncLane = 0b0000000000000000000000000000001
InputContinuousLane = 0b0000000000000000000000000000100

lane = SyncLane ｜ InputContinuousLane
lane = 0b0000000000000000000000000000101
```

那么通过 lanes & -lanes 分离出 SyncLane。

首先我们看一下 -lanes，在二进制中需要用补码表示为：

```
-lane = 0b1111111111111111111111111111011
```

那么接下来执行 lanes & -lanes 看一下，& 的逻辑是如果两位都是 1 则设置改位为 1，否则为 0。

那么 lane & -lane ，只有一位（最后一位）全是 1，所有合并后的内容为：

```
lane & -lane = 0b0000000000000000000000000000001
```

可以看得出来 lane & -lane 的结果是 SyncLane，所以通过 lane & -lane 就能分离出最高优先级的任务。

```js
const SyncLane = 0b0000000000000000000000000000001;
const InputContinuousLane = 0b0000000000000000000000000000100;
const lane = SyncLane | InputContinuousLane;
console.log((lane & -lane) === SyncLane); // true
```

## Context-更新上下文

进入了更新阶段，也有一个属性用于判断现在更新上下文的状态，这个属性就是 ExecutionContext

所谓上下文其实就是类似于批量更新的 isBatchedUpdate 这种方式，通过先设置一个上下文再执行某个步骤，这样在这个步骤内部就可以通过判断上下文来改变一些操作，最后再清除掉上下文。

常见的上下文比如 BatchedContext（处于批量更新）、RenderContext（处于 render 阶段）、CommitContext（commit 阶段）等

```js
export const NoContext = /*             */ 0b000;
const BatchedContext = /*               */ 0b001;
export const RenderContext = /*         */ 0b010;
export const CommitContext = /*         */ 0b100;
```

ExecutionContext 类型变量采用 4 位的二进制表示。

### executionContext

在 WorkLoop（调度）的全称都有一个全局变量 executionContext，作为一个全局状态，指引 React 更新的方向。
executionContext 一般初始化为 NoLanes（全 0），当需要改变当前所处的上下文时，就将对应的上下文和 executionContext 相与或直接修改，executionContext 就表示当前正在处在的上下文。

比如判断是否在 commit 阶段或 render 阶段：

```js
if((executionContext & RenderContext) !== NoLanes) // render stage
if((executionContext & CommitContext) !== NoLanes) // commit stage
```

修改

```ts
function renderRootSync(root: FiberRoot, lanes: Lanes) {
  executionContext |= RenderContext;
}

function commitRootImpl() {
  executionContext |= CommitContext;
}
```

在 scheduleUpdateOnFiber 函数中有这么一小段：

```ts
if (
  (executionContext & RenderContext) !== NoLanes &&
  root === workInProgressRoot
) {
  // 直接更新
} else {
  // 调度更新
}
```

## flag

React 主要的 flags 如下：

```ts
export const NoFlags = /*                      */ 0b000000000000000000000000000;
export const PerformedWork = /*                */ 0b000000000000000000000000001;
export const Placement = /*                    */ 0b000000000000000000000000010;
export const DidCapture = /*                   */ 0b000000000000000000010000000;
export const Hydrating = /*                    */ 0b000000000000001000000000000;

export const Update = /*                       */ 0b000000000000000000000000100;

export const ChildDeletion = /*                */ 0b000000000000000000000010000;
export const ContentReset = /*                 */ 0b000000000000000000000100000;
export const Callback = /*                     */ 0b000000000000000000001000000;

export const ForceClientRender = /*            */ 0b000000000000000000100000000;
export const Ref = /*                          */ 0b000000000000000001000000000;
export const Snapshot = /*                     */ 0b000000000000000010000000000;
export const Passive = /*                      */ 0b000000000000000100000000000;

export const Visibility = /*                   */ 0b000000000000010000000000000;
export const StoreConsistency = /*             */ 0b000000000000100000000000000;

export const LifecycleEffectMask =
  Passive | Update | Callback | Ref | Snapshot | StoreConsistency;
export const HostEffectMask = /*               */ 0b00000000000011111111111111;
```

其中大部分 flag 是用于调和阶段对于 dom 元素的实际操作的，比如在 beginWork 和 completeWork 阶段打上 Placement、Deletion，然后在 commit 阶段执行具体的 dom 操作等。
这种 flags 被称作“副作用”标记。

比如说在 beginWork 阶段，通过 Reconciler 发现某个 fiber 应该被删除，那就给这个 fiber 的父 fiber 打上 ChildDeletion。在 commit 阶段如果检查某个 fiber 上有这个 flag，就会执行删除操作。

```ts
let flag = NoFlags;

//发现更新，打更新标志
flag = flag | PerformedWork | Update;

//判断是否有  PerformedWork 种类的更新
if (flag & PerformedWork) {
  //执行
  console.log("执行 PerformedWork");
}

//判断是否有 Update 种类的更新
if (flag & Update) {
  //执行
  console.log("执行 Update");
}

if (flag & Placement) {
  //不执行
  console.log("执行 Placement");
}
```

# React context

## context 对象

context 对象由 React 的 createContext 创建，这个函数大致如下：

```ts
export function createContext<T>(defaultValue: T): ReactContext<T> {
  const context: ReactContext<T> = {
    $$typeof: REACT_CONTEXT_TYPE,
    _currentValue: defaultValue,
    _currentValue2: defaultValue,
    _threadCount: 0,
    Provider: (null: any),
    Consumer: (null: any),
    _defaultValue: (null: any),
    _globalName: (null: any),
  };

  context.Provider = {
    $$typeof: REACT_PROVIDER_TYPE,
    _context: context,
  };

  let hasWarnedAboutUsingNestedContextConsumers = false;
  let hasWarnedAboutUsingConsumerProvider = false;
  let hasWarnedAboutDisplayNameOnConsumer = false;

  context.Consumer = context;

  return context;
}
```

可以看到关键的两个部分，一个是创建的 context 对象，一个是 Context.Provider 对象。Provider 本质上是引用了 Context 的一个 element 对象，而 Provider 的 value 参数则会被放入 context 对象的\_currentValue 属性去。

## Provider

我们常用的 Provider jsx，实际上就会被编译为上面的 Provider 对象，然后在调和阶段转化为 ContextProvider fiber。
在 beginWork 函数中，对于该 fiber 有特殊的处理函数 updateContextProvider

```ts
function beginWork(){
  //...
  case ContextProvider:
      return updateContextProvider(current, workInProgress, renderLanes);
  //...
}

function updateContextProvider(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
) {
  const providerType: ReactProviderType<any> = workInProgress.type;
  const context: ReactContext<any> = providerType._context;

  const newProps = workInProgress.pendingProps;
  const oldProps = workInProgress.memoizedProps;

  const newValue = newProps.value;

  // 将value props赋给context对象
  pushProvider(workInProgress, context, newValue);

  if (oldProps !== null) {
    const oldValue = oldProps.value;
    // 比较两次的value props
    if (is(oldValue, newValue)) {
      if (
        oldProps.children === newProps.children &&
        !hasLegacyContextChanged()
      ) {
        // 终止更新
        return bailoutOnAlreadyFinishedWork(
          current,
          workInProgress,
          renderLanes,
        );
      }
    } else {
      // 传播context的改变
      propagateContextChange(workInProgress, context, renderLanes);
    }
  }

  const newChildren = newProps.children;
  // 调和子节点
  reconcileChildren(current, workInProgress, newChildren, renderLanes);
  return workInProgress.child;
}
```

updateContextProvider 函数主要干了这几件事：

- `pushProvider` 会获取 type 属性上的 \_context 对象，就是上述通过 createContext 创建的 context 对象。然后将 Provider 的 value 属性，赋值给 context 的 \_currentValue 属性上。
- 获取新旧 fiber 上的 Provider 传入的 value props，进行比较，如果相同就不需要更新；如果不同，执行 propagateContextChange 去传播 context 改变。

propagateContextChange 是更新的关键，代码如下：

```ts
function propagateContextChange_eager<T>(
  workInProgress: Fiber,
  context: ReactContext<T>,
  renderLanes: Lanes
): void {
  // 从provider fiber的child开始
  let fiber = workInProgress.child;
  if (fiber !== null) {
    fiber.return = workInProgress;
  }
  // 从providerFiber.child开始遍历
  while (fiber !== null) {
    let nextFiber;

    const list = fiber.dependencies;
    if (list !== null) {
      nextFiber = fiber.child;

      let dependency = list.firstContext;
      while (dependency !== null) {
        // 对比 dependencies 中的 context 和当前 Provider 的 context 是否是同一个
        if (dependency.context === context) {
          // 如果是，那么就在该fiber上调度一次更新
          if (fiber.tag === ClassComponent) {
            // 如果是类组件，会调用forceUpdate
            const lane = pickArbitraryLane(renderLanes);
            const update = createUpdate(NoTimestamp, lane);
            update.tag = ForceUpdate;
            // 下面就是把update放入pending，然后执行forceUpdate的过程
          }

          // 提高找到了的fiber的优先级，然后调度更新
          fiber.lanes = mergeLanes(fiber.lanes, renderLanes);
          const alternate = fiber.alternate;
          if (alternate !== null) {
            alternate.lanes = mergeLanes(alternate.lanes, renderLanes);
          }
          // 这里会把从根节点到消费context的组件之间的所有节点，即消费context组件的祖先节点全部提高优先级
          scheduleContextWorkOnParentPath(
            fiber.return,
            renderLanes,
            workInProgress
          );

          list.lanes = mergeLanes(list.lanes, renderLanes);
          break; // 如果找到了匹配的context，就终止循环
        }
        dependency = dependency.next;
      }
    } else if (fiber.tag === ContextProvider) {
      // 如果遇到了Provider嵌套的情况，那就把内部的跳过
      nextFiber = fiber.type === workInProgress.type ? null : fiber.child;
    } else {
      // 继续向下遍历
      nextFiber = fiber.child;
    }

    // 遍历是从上向下的，如果遍历到了nextFiber为空，此时就应该遍历兄弟节点，然后再向上返回，最终回到起点
    // 这种遍历方式和commit过程的类似

    // 每次遍历fiber向下走一步
    fiber = nextFiber;
  }
}
```

这里有几个关键点：

- dependencies 属性，这个属性可以把当前的 fiber 和 context 建立起关联。也就是说，使用了当前 context 的 fiber 会把 context 放在 dependencies 中，dependencies 属性本身是一个链表结构，一个 fiber 可以有多个 context 与之对应。因此这里是检查 fiber 上有没有用过 context（`list !== null`），没有就找下一个。
- forceUpdate：对类组件来说，由于普通更新可能经过 shouldComponentUpdate 限制，因此最好使用 forceUpdate 来调度一次更新。

## readContext

readContext 是组件读取 context 信息的关键。

```ts
function readContext<T>(context: ReactContext<T>): T {
  // 创建一个contextItem对象
  const contextItem = {
    context: ((context: any): ReactContext<mixed>),
    memoizedValue: value,
    next: null,
  };
  // 根据初次渲染和更新选择context对象上的属性返回
  const value = isPrimaryRenderer
    ? context._currentValue
    : context._currentValue2;

  if (lastContextDependency === null) {
    lastContextDependency = contextItem;
    // 将contextItem形成链表连接到fiber的dependencies属性上
    // 在Provider的处理函数内，就是遍历fiber.dependencies.firstContext，即dependency
    currentlyRenderingFiber.dependencies = {
      lanes: NoLanes,
      firstContext: contextItem,
    };
  } else {
    lastContextDependency = lastContextDependency.next = contextItem;
  }
  return value;
}
```

对于函数组件来说，useContext 就是 readContext 函数，即取出 context 对象并返回值，同时将 context 经过处理连接到 fiber 上，方便 Provider 进行遍历和调度。

更新流程

1. Provider 组件持有的 value 更新
2. 触发 Provider 组件更新，执行 updateContextProvider
3. 执行 propagateContextChange，会从 Provider 节点的子节点开始，向下遍历一圈所有节点并返回；如果遇到了消费 context 的 fiber，就看看这个 context 是不是和当前 Provider 持有的匹配；如果匹配，就会将该节点和其祖先节点的优先级都提高，对类组件还会安排 forceUpdate
4. 回到 updateContextProvider，调用 reconcileChildren 执行后续的调和。在第 3 步中提升优先级的组件将会在后面被更新

# React 优先级机制

## React 与 Scheduler 的结合

React 和 Scheduler 其实是分属两个包的，因此优先级不能共用。
React 的优先级是 Lane 和事件优先级 EventPriority，而 Scheduler 的优先级则是以过期时间为标准的 timeout；这两个优先级的不同就需要在进入 Schedule 时进行转换。

React 的优先级：
事件优先级其实和部分 Lane 是对应的，四种事件优先级恰好对应四种更新优先级，并且含义可以理解为就是这种事件产生的优先级。比如 SyncLane 的更新就可以理解为是由离散事件产生的更新。

```ts
export const DiscreteEventPriority: EventPriority = SyncLane; // 离散事件优先级，比如click
export const ContinuousEventPriority: EventPriority = InputContinuousLane; // 连续事件优先级，比如move
export const DefaultEventPriority: EventPriority = DefaultLane; // 默认事件优先级，比如定时器
export const  : EventPriority = IdleLane; // 空闲优先级

let currentUpdatePriority: EventPriority = NoLane;
```

> lane 其实一共有 8 种，除了上面四种外，还有每种的 Hydration 形式
>
> ```ts
> export const NoLanes: Lanes = /*                        */ 0b0000000000000000000000000000000;
> export const NoLane: Lane = /*                          */ 0b0000000000000000000000000000000;
>
> export const SyncHydrationLane: Lane = /*               */ 0b0000000000000000000000000000001;
> export const SyncLane: Lane = /*                        */ 0b0000000000000000000000000000010;
>
> export const InputContinuousHydrationLane: Lane = /*    */ 0b0000000000000000000000000000100;
> export const InputContinuousLane: Lane = /*             */ 0b0000000000000000000000000001000;
>
> export const DefaultHydrationLane: Lane = /*            */ 0b0000000000000000000000000010000;
> export const DefaultLane: Lane = /*                     */ 0b0000000000000000000000000100000;
> ```

在进入调度的 ensureRootIsScheduled 函数内部，就会将 React 优先级和 Scheduler 优先级进行转换，先转为事件优先级，然后转成对应的 Scheduler 优先级。

```ts
let schedulerPriorityLevel;
switch (lanesToEventPriority(nextLanes)) {
  case DiscreteEventPriority:
    schedulerPriorityLevel = ImmediateSchedulerPriority;
    break;
  case ContinuousEventPriority:
    schedulerPriorityLevel = UserBlockingSchedulerPriority;
    break;
  case DefaultEventPriority:
    schedulerPriorityLevel = NormalSchedulerPriority;
    break;
  case IdleEventPriority:
    schedulerPriorityLevel = IdleSchedulerPriority;
    break;
  default:
    // setTimeout中的setState任务优先级通常会被转为NormalSchedulerPriority
    schedulerPriorityLevel = NormalSchedulerPriority;
    break;
}
newCallbackNode = scheduleCallback(
  schedulerPriorityLevel,
  performConcurrentWorkOnRoot.bind(null, root)
);
```

lanesToEventPriority 函数其实也不是直接用一一对应的形式把 lane 转成事件优先级，而是需要将 lane 拆解，取出最高优先级作为事件优先级。主要是一个组件可能不止一种更新方式，也就意味着不止一种更新优先级。
getHighestPriorityLane 的算法在上面位运算部分说过了。

```ts
export function lanesToEventPriority(lanes: Lanes): EventPriority {
  const lane = getHighestPriorityLane(lanes);
  if (!isHigherEventPriority(DiscreteEventPriority, lane)) {
    return DiscreteEventPriority;
  }
  if (!isHigherEventPriority(ContinuousEventPriority, lane)) {
    return ContinuousEventPriority;
  }
  if (includesNonIdleWork(lane)) {
    return DefaultEventPriority;
  }
  return IdleEventPriority;
}
```

## Lane 的算法

### FiberRoot 上的 Lane

除了那八种关于更新的 lane 之外，还有一些 lane 是在 FiberRoot 上的，主要是这两个：

- pendingLanes: 表示当前 root 下待执行的 update 对应的 lane，即 pending 队列中的 updatelane 合并
- expiredLanes: 由于过期需要执行、且不可中断的 update 的 lane 的集合

同样是在进入调度的 ensureRootIsScheduled 函数中，一开始就会通过 getNextLanes 函数计算出本次 render 阶段的 lane，即 renderLane；

```ts
function ensureRootIsScheduled(root: FiberRoot, currentTime: number) {
  const existingCallbackNode = root.callbackNode;

  const nextLanes = getNextLanes(
    root,
    root === workInProgressRoot ? workInProgressRootRenderLanes : NoLanes,
  );
  let schedulerPriorityLevel;
  switch (lanesToEventPriority(nextLanes)){...} // 这个就是上面把React优先级替换为Scheduler优先级的地方
  scheduleCallback( // 进行调度，依然和nextLanes分不开
      schedulerPriorityLevel,
      performConcurrentWorkOnRoot.bind(null, root),
  );
}
```

### 初始化 Lane - requestUpdateLane

接下来和后面几个部分的内容，是以 React 一次更新的流程来介绍 lane 在其中的参与。

首先当通过触发事件来触发一次更新时，在事件的回调内执行 dispatch 函数，然后开启调度。在调用调度函数 ScheduleUpdateOnFiber 时，就会传入 FiberRoot。
所以不管在哪里触发的更新，其实都会归结在 FiberRoot 上，从 root 开始初始化。
初始化的过程，其实包含了主要两步：

- 创建 update 对象
- 确定 lane 信息

确定 lane 信息靠的是 requestUpdateLane 函数，这个函数在 dispatchSetState 一开始就调用了（这两步都是在 dispatch 中调用的）：

```ts
function dispatchSetState<S, A>(
  fiber: Fiber,
  queue: UpdateQueue<S, A>,
  action: A,
): void {

  const lane = requestUpdateLane(fiber);

  const update: Update<S, A> = {
    lane,
    action,
    hasEagerState: false,
    eagerState: null,
    next: (null: any),
  };
}
```

requestUpdateLaned 的作用是获取更新优先级，获取的逻辑如下：

- 如果当前是非并发模式，那么更新优先级都是 SyncLane
- 如果是渲染阶段更新，即在函数执行期间就调用 dispatch 而不是在事件或回调中；那么就是当前 renderLane 的最高优先级
- 如果和 Transition 相关，那么会降低优先级
- 如果不是上面这几种，那就默认是事件中的优先级

> workInProgressRootRenderLanes，即 renderLanes，表示本次 render 阶段要处理的 lanes。每次 render 一般只会选出一部分优先级去处理，只有符合这部分优先级的更新才会被处理

```ts
export function requestUpdateLane(fiber: Fiber): Lane {
  const mode = fiber.mode;
  if ((mode & ConcurrentMode) === NoMode) {
    // 非并发模式，返回SyncLane
    return (SyncLane: Lane);
  } else if (
    !deferRenderPhaseUpdateToNextBatch &&
    (executionContext & RenderContext) !== NoContext &&
    workInProgressRootRenderLanes !== NoLanes
  ) {
    // 渲染阶段更新，返回当前renderLane的最高优先级
    return getHighestPriorityLane(workInProgressRootRenderLanes);
  }

  // 这个在Transition介绍过，降低更新优先级
  const isTransition = requestCurrentTransition() !== NoTransition;
  if (isTransition) {
    if (currentEventTransitionLane === NoLane) {
      currentEventTransitionLane = claimNextTransitionLane();
    }
    return currentEventTransitionLane;
  }

  const updateLane: Lane = (getCurrentUpdatePriority(): any);
  if (updateLane !== NoLane) {
    return updateLane;
  }

  const eventLane: Lane = (getCurrentEventPriority(): any);
  return eventLane;
}
```

在 dispatch 函数内通过执行 requestUpdateLane 得到的 lane 会作为本次更新的 lane，将会在很多地方用到。
其中，在 beginWork 部分讲到的 childLanes，就是本次更新附加到祖先元素的 lane 上的。这部分内容参考 beginWork 那部分。

### 确定 renderLanes - getNextLanes

进入 ensureRootIsScheduled 函数后，会通过调用 getNextLanes 获取本次 render 的 lane。这个函数比较复杂：

```ts
export function getNextLanes(root: FiberRoot, wipLanes: Lanes): Lanes {
  const pendingLanes = root.pendingLanes; // 待更新的update的lane的集合
  if (pendingLanes === NoLanes) {
    return NoLanes;
  }

  let nextLanes = NoLanes;

  // 非空闲的lane，即更新的lane中只要不是idleLane的其他三种
  const nonIdlePendingLanes = pendingLanes & NonIdleLanes;
  if (nonIdlePendingLanes !== NoLanes) {
    const nonIdleUnblockedLanes = nonIdlePendingLanes & ~suspendedLanes;
    // 获取非空闲lanes的最高lane
    if (nonIdleUnblockedLanes !== NoLanes) {
      nextLanes = getHighestPriorityLanes(nonIdleUnblockedLanes);
    } else {
      const nonIdlePingedLanes = nonIdlePendingLanes & pingedLanes;
      if (nonIdlePingedLanes !== NoLanes) {
        nextLanes = getHighestPriorityLanes(nonIdlePingedLanes);
      }
    }
  } else {
    // 获取空闲lanes的最高的lane
    const unblockedLanes = pendingLanes & ~suspendedLanes;
    if (unblockedLanes !== NoLanes) {
      nextLanes = getHighestPriorityLanes(unblockedLanes);
    } else {
      if (pingedLanes !== NoLanes) {
        nextLanes = getHighestPriorityLanes(pingedLanes);
      }
    }
  }

  if (nextLanes === NoLanes) {
    return NoLanes;
  }
  // 其他情况下获取最高lane，这里省略

  return nextLanes;
}
```

排除掉 suspense、entangle 这种，其实这个函数就是在获取一个“最高 lane”，即从 root.pendingLanes 中取一个最高的 lane 作为本次更新的优先级，在本次调度中所有符合的更新都会被消费，而后续的执行则会执行剩下优先级不够的更新。

在获取 nextLane 之后，如果 nextLane 为 SyncLane，即当前 renderLanes 的最高优先级是 SyncLane，就会进入同步调度。后续的就在调度部分了。

```ts
if (includesSyncLane(newCallbackPriority)) {
  if (root.tag === LegacyRoot) {
    scheduleLegacySyncCallback(performSyncWorkOnRoot.bind(null, root));
  } else {
    scheduleSyncCallback(performSyncWorkOnRoot.bind(null, root));
  }
}
```

同步调度意味着后续的调和也是同步的。但是真正执行调和任务（flushSyncCallbacks）还是通过微任务的形式，因此不管是类组件还是函数组件，都不能在 state 更新后立即获取最新的 state。

---

另一种情况，如果 nextLane 不是 Synclane，那就会进入后面的转换，将 lane 转为 Schedule 优先级，并调度 performConcurrentWorkOnRoot

```ts
let schedulerPriorityLevel;
switch (
  lanesToEventPriority(nextLanes)
  // ...转为Schedule优先级
) {
}
newCallbackNode = scheduleCallback(
  schedulerPriorityLevel,
  performConcurrentWorkOnRoot.bind(null, root)
);
```

这时就会进入并发渲染了。

### 解决饥饿问题 - markStarvedLanesAsExpired

# 生命周期

官方文档的周期描述：
![](https://pic.imgdb.cn/item/6243fc8427f86abb2ab38f97.jpg)

> 图链接：https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/

老版生命周期：
![](https://pic.imgdb.cn/item/62485ffd27f86abb2a69b3d7.jpg)

## 生命周期的执行函数

如果在一次调和（render，严格来说是 beginWork）的过程中，发现了一个 fiber tag 为类组件的情况，就会按照类组件的逻辑来处理。该处理的基本代码如下：

这里两个关键的函数是`mountClassInstance`和`updateClassInstance`。前者是挂载类组件时的步骤，后者是更新时的。在两个函数内部，将会分别按照顺序执行类组件中定义的生命周期函数。

```js
function updateClassComponent() {
  let shouldUpdate;
  const instance = workInProgress.stateNode; // stateNode 是 fiber 指向 类组件实例的指针。
  if (instance === null) {
    // instance 为组件实例,如果组件实例不存在，证明该类组件没有被挂载过，那么会走初始化流程
    constructClassInstance(workInProgress, Component, nextProps); // 组件实例将在这个方法中被new。
    mountClassInstance(/*...*/); //初始化挂载组件流程
    shouldUpdate = true; // shouldUpdate 标识用来证明 组件是否需要更新。
  } else {
    shouldUpdate = updateClassInstance(/*...*/); // 更新组件流程
  }
  if (shouldUpdate) {
    nextChildren = instance.render(); /* 执行render函数 ，得到子节点 */
    reconcileChildren(/*...*/); /* 继续调和子节点 */
  }
}
```

类组件实例和类组件 fiber 之间存在对应关系，具体来说是这样：
![](https://pic.imgdb.cn/item/63a30459b1fccdcd362baef4.jpg)
fiber.stateNode 属性在 fiber tag 为类组件时表示其对应的类组件实例

### 生命周期的执行（源码）

生命周期本质上就是挂载在组件实例或组件类本身上的函数；因此执行生命周期的过程，其实本质上也就是取出这些函数并执行。
以 mountClassInstance 为例，代码简化如下：

```js
// 组件挂载过程才会执行的函数
// 即getDerivedStateFromProps/componentWillMount/componentDidMount三个生命周期
function mountClassInstance(
  workInProgress: Fiber,
  ctor: any,
  newProps: any,
  renderLanes: Lanes
): void {
  const instance = workInProgress.stateNode; // 获取类组件实例，从上面取到生命周期函数
  instance.props = newProps; // 设置类组件的一些属性
  // 由于更新state实际上是改变fiber上的memoizedState，因此还需要设置一下instance上的state，才能保证在类组件中this.state是最新的
  instance.state = workInProgress.memoizedState;
  instance.refs = {};

  initializeUpdateQueue(workInProgress);

  // getDerivedStateFromProps是static方法，要单独处理
  const getDerivedStateFromProps = ctor.getDerivedStateFromProps;
  if (typeof getDerivedStateFromProps === "function") {
    // 这个函数就是执行getDerivedStateFromProps的
    applyDerivedStateFromProps(
      workInProgress,
      ctor,
      getDerivedStateFromProps,
      newProps
    );
    instance.state = workInProgress.memoizedState;
  }

  // 没有getDerivedStateFromProps和getSnapshotBeforeUpdate时，才会执行componentWillMount生命周期
  if (
    typeof ctor.getDerivedStateFromProps !== "function" &&
    typeof instance.getSnapshotBeforeUpdate !== "function" &&
    (typeof instance.UNSAFE_componentWillMount === "function" ||
      typeof instance.componentWillMount === "function")
  ) {
    callComponentWillMount(workInProgress, instance);
    processUpdateQueue(workInProgress, newProps, instance, renderLanes);
    instance.state = workInProgress.memoizedState;
  }
  // 设置componentDidMount，但还未执行
  if (typeof instance.componentDidMount === "function") {
    let fiberFlags: Flags = Update | LayoutStatic;
    workInProgress.flags |= fiberFlags;
  }
}
```

componentDidMount 的执行时机是 commit 阶段，但现在是 render 阶段，暂时还未执行；一旦到达 commit 阶段，就会执行

```js
function commitLifeCycles(finishedRoot,current,finishedWork){
    switch (finishedWork.tag){
     case ClassComponent: {
       const instance = finishedWork.stateNode
       if(current === null){                          /* 类组件第一次调和渲染 */
         instance.componentDidMount()
       }else{                                         /* 类组件更新 */
         instance.componentDidUpdate(prevProps,prevState，instance.__reactInternalSnapshotBeforeUpdate);
       }
     }
    }
}
```

全过程如下（挂载阶段）：
![](https://pic.imgdb.cn/item/63a30df8b1fccdcd3640cfa2.jpg)
这里的 constructor 和 render 并不是显式的生命周期。

- constructor 实际上是在组件那章说过的执行类组件、绑定 props 和 updater 的`constructClassInstance`；
- render 就是渲染过程，也就是上面`updateClassComponent`函数的`nextChildren = instance.render()`这个地方。显然这里是在执行完上面几个生命周期之后的。

---

更新阶段的生命周期执行是在 updateClassInstance 里的。执行过程类似，不过有几个特殊的生命周期：

- getSnapshotBeforeUpdate：执行也是在 commit 阶段，但具体来说是在 before Mutation(DOM 修改前)阶段，代码大概是这样：

```js
function commitBeforeMutationLifeCycles(current, finishedWork) {
  switch (finishedWork.tag) {
    case ClassComponent: {
      const snapshot = instance.getSnapshotBeforeUpdate(
        prevProps,
        prevState
      ); /* 执行生命周期 getSnapshotBeforeUpdate   */
      instance.__reactInternalSnapshotBeforeUpdate =
        snapshot; /* 返回值将作为 __reactInternalSnapshotBeforeUpdate 传递给 componentDidUpdate 生命周期  */
    }
  }
}
```

## 生命周期概览

挂载：

1. constructor: 实例化组件，初始化 props 和 state
2. getDerivedStateFromProps: 静态方法，获取上一次的 props，返回值和本次的 state 合并
3. render: 通过 React.createElement 创建、解析元素，把元素变成 fiber 并进行以后的步骤
4. componentDidMount: 组件已经挂载，dom 渲染完毕

更新：

1. getDerivedStateFromProps
2. shouldComponentUpdate: 传入当前 props 和 state，返回值为 false 则跳过本次更新
3. render
4. getSnapshotBeforeUpdate: 获取更新前的最后一次快照，记录更新前的 state 和 props
5. componentDidUpdate: 更新完毕

卸载：

1. componentWillUnmount: 即将卸载前调用

### 初始化阶段

#### constructor

即`constructor` 的执行，实例化 React 组件。进行 state 、props 的初始化，在这个阶段修改 state，不会执行更新阶段的生命周期，可以直接对 state 赋值。

在 React 组件挂载之前，会调用它的构造函数。**在为 `React.Component` 子类实现构造函数时，应在其他语句之前调用 `super(props)`**

原因是绑定 `props` 是在父类 `Component` 构造函数中，执行 `super` 等于执行 `Component` 函数，此时 `props` 没有作为第一个参数传给 `super()` ，在 `Component` 中就会找不到 `props` 参数，从而变成 `undefined` ，在接下来 `constructor` 代码中打印 `props` 为 `undefined` 。

`Component` 函数大概是这样：

```js
function Component(props, context, updater) {
  this.props = props; //绑定props
  this.context = context; //绑定context
  this.refs = emptyObject; //绑定ref
  this.updater = updater || ReactNoopUpdateQueue; // updater 对象
}
/* 绑定setState 方法 */
Component.prototype.setState = function (partialState, callback) {
  this.updater.enqueueSetState(this, partialState, callback, "setState");
};
/* 绑定forceupdate 方法 */
Component.prototype.forceUpdate = function (callback) {
  this.updater.enqueueForceUpdate(this, callback, "forceUpdate");
};
```

> 通常，在 React 中，构造函数仅用于以下两种情况：
>
> - 通过给 this.state 赋值对象来初始化内部 state。
> - 为事件处理函数绑定 this

### 挂载阶段

#### 1. `getDerivedStateFromProps`

是第二个执行的生命周期，每次渲染前都会触发，是类上的静态方法，不能访问到类实例；
参数是当前 props 和上一次的 state，返回值将和之前的 state 合并，作为新的 state；如果返回 null 就不更新。

作用：

- 代替 `componentWillMount` 和 `componentWillReceiveProps`
- 组件初始化或者更新时，将 props 映射到 state。
- 返回值与 state 合并完，可以作为 `shouldComponentUpdate` 第二个参数 `newState` ，可以判断是否渲染组件。

使用示例：

```js
static getDerivedStateFromProps(newProps){
  /*接受 props 变化,返回值将作为新的 state,用于渲染或传递给shouldComponentUpdate */
    const { type } = newProps
    switch(type){
        case 'fruit' :
        return { list:['苹果','香蕉','葡萄' ] }
        case 'vegetables':
        return { list:['菠菜','西红柿','土豆']}
    }
}
```

#### 2. `render`

render 函数是 jsx 的各个元素被 `React.createElement`创建成 React element 对象的形式。一次 render 的过程，就是创建 `React.element` 元素的过程。

当 render 被调用时，它会检查 `this.props` 和 `this.state` 的变化并返回值，这个值大多数情况下是 jsx，当然也可以是其他允许的类型，比如单独返回一个字符串。

`render()` 函数应该为纯函数，这意味着在不修改组件 state 的情况下，每次调用时都返回相同的结果，并且它不会直接与浏览器交互。因此不能在 render 中执行有副作用的操作，应该放在`componentDidMount`或其他中。
因此可以在 render 里面做一些诸如`createElement`创建元素 , `cloneElement` 克隆元素 ，`React.children` 遍历 children 的操作等

> `render`并不一定执行，如果`shouldComponentUpdate()` 返回 false 就不会执行。但是对于初次挂载来说，是唯一一个一定要执行、不可缺少的生命周期。

#### 3. `componentDidMount`

发生在 render 函数之后，已经挂载 DOM 之后立即调用。
它的调用发生在`commit`阶段，即在 React 调和完所有的 fiber 节点之后，就会到 commit 阶段，调用 `componentDidMount` 生命周期。

作用：

- 因为此时已经挂载 dom 了，因此可以做一些关于 DOM 操作，比如基于 DOM 的事件监听器。
- 对于初始化向服务器请求数据，渲染视图等

### 更新阶段

老版本 React 更新会分为 props 更新和 state 更新两种情况，执行的方法会有不同。

props 更新：

1. `componentWillReceiveProps(nextProps,nextState)`
2. `shouldComponentUpdate(nextProps,nextState)`
3. `componentWillUpdate(nextProps,nextState)`
4. `render`
5. `componentDidUpdate(prevProps, prevState)`

state 更新：

1. `shouldComponentUpdate(nextProps,nextState)`
2. `componentWillUpdate(nextProps,nextState)`
3. `render`
4. `componentDidUpdate(prevProps, prevState)`

后面去掉了`componentWillUpdate`等三个生命周期后，两者的执行流程已经没有区别，都是：

1. `static getDerivedStateFromProps`
2. `shouldComponentUpdate`
3. `render`
4. `getSnapshotBeforeUpdate`
5. `componentDidUpdate`

#### 1. `UNSAFE_componentWillReceiveProps`

该生命周期的特点：

1. 即使 props 没变，该生命周期也会执行。
2. pureComponent 并不能在 props 不变的情况下阻止该生命周期执行。纯组件是在 componentWillReceiveProps 执行之后浅比较 props 是否发生变化
3. 可以在内部通过异步的方式更改 state。具体来说就是相当于 componentDidUpdate 中异步请求并修改 state 的方式，但是会带来两次子组件的更新，这也是不建议使用的原因之一

该生命周期执行驱动是因为父组件更新带来的 props 修改，但是只要父组件触发 render 函数，调用 React.createElement 方法，那么 props 就会被重新创建，生命周期 componentWillReceiveProps 就会执行了。

这里会首先判断 `getDerivedStateFromProps` 生命周期是否存在，如果不存在就执行`componentWillReceiveProps`生命周期。传入该生命周期两个参数，分别是 `newProps` 和 `nextContext` 。

#### 2. `getDerivedStateFromProps`

和在挂载时的主要功能类似；
一旦出现该函数，就会在两种更新情况下都最先执行并无视`componentWillReceiveProps`。

#### 3. `shouldComponentUpdate`

如果有`getDerivedStateFromProps`，就会在其后紧接执行。
如果没有，则会在`componentWillReceiveProps`之后紧接执行；
如果是 state 触发的更新，则会首先执行。

作用：

- 传入新的 props、state 和 context ，返回值决定是否继续执行 render 函数，调和子节点。
- 一般用于性能优化，对比 state 或 props 决定是否重新渲染。但优化更好的方式是采用`PureComponent`或`React.memo`。

使用示例：

```js
shouldComponentUpdate(newProps,newState){
    if(newProps.a !== this.props.a ){ /* props中a属性发生变化 渲染组件 */
        return true
    }else if(newState.b !== this.props.b ){ /* state 中b属性发生变化 渲染组件 */
        return true
    }else{ /* 否则组件不渲染 */
        return false
    }
}
```

#### 4. `UNSAFE_componentWillUpdate`

详见下。
这个方法会紧接着`shouldComponentUpdate`执行，但是如果前者返回 false 就不会执行

#### 5. render

接下来会执行 render 函数，得到最新的 React element 元素。然后继续调和子节点。

#### 6. `getSnapshotBeforeUpdate`

紧接着 render 之后，并在`componentDidUpdate`之前执行。
执行的时间被称为一个`pre-commit`阶段，即此时仍未 commit，但是已经渲染完毕。

> commit 阶段细分为 `before Mutation`( DOM 修改前)，`Mutation` ( DOM 修改)，`Layout`( DOM 修改后) 三个阶段；`getSnapshotBeforeUpdate` 发生在`before Mutation` 阶段

作用：

- 获取更新前 DOM 的状态，配合`componentDidUpdate` 一起使用，计算形成一个 `snapShot` 传递给 `componentDidUpdate`的第三个参数，用于保存一次更新前的信息。

```js
getSnapshotBeforeUpdate(prevProps,preState){
    const style = getComputedStyle(this.node)
    return { /* 传递更新前的元素位置 */
        cx:style.cx,
        cy:style.cy
    }
}
componentDidUpdate(prevProps, prevState, snapshot){
    /* 获取元素绘制之前的位置 */
    console.log(snapshot)
}
```

#### 7. `componentDidUpdate`

在更新后会被立即调用，位于整个更新步骤的最后一步。首次渲染不会执行此方法。

作用

- 此时 DOM 已经更新，可以直接获取 DOM 最新状态。这个函数里面如果想要使用 setState ，一定要加以限制，否则会引起无限循环。
- 用于类似`componentDidMount`一样执行副作用。
- 接受 `getSnapshotBeforeUpdate` 保存的快照信息。

使用示例：
调用时三个参数：

- `prevProps` 更新之前的 props ；
- `prevState` 更新之前的 state ；
- `snapshot` 为 `getSnapshotBeforeUpdate` 返回的快照，可以是更新前的 DOM 信息。

```js
componentDidUpdate(prevProps, prevState, snapshot){
    const style = getComputedStyle(this.node)
    const newPosition = { /* 获取元素最新位置信息 */
        cx:style.cx,
        cy:style.cy
    }
}
```

### 卸载阶段

#### `componentWillUnmount`

在组件卸载及销毁之前直接调用，是销毁阶段唯一的生命周期。

作用：

- 执行必要的清理操作，例如，清除 timer，取消网络请求或清除在 `componentDidMount` 中创建的订阅等。

> `componentWillUnmount` 中不应调用 `setState`，因为该组件将永远不会重新渲染。组件实例卸载后，将永远不会再挂载它。

```js
componentWillUnmount(){
    clearTimeout(this.timer)  /* 清除延时器 */
    this.node.removeEventListener('click',this.handerClick) /* 卸载事件监听器 */
}
```

### 三个被抛弃的生命周期

#### 1. `UNSAFE_componentWillMount`

> 如果存在 `getDerivedStateFromProps` 和 `getSnapshotBeforeUpdate` 就不会执行生命周期`componentWillMount`。

发生在 render 函数之前，还没有挂载 DOM 的时候。

这个周期被挂上了`UNSAFE_`的前缀，即 React 准备弃用这个生命周期。主要原因是：

1. 功能不必要，即可以用其他声明周期代替。如果想在挂载之前进行一些操作，可以在`getDerivedStateFromProps`或`constructor`中进行，比如想提前网络请求、启动定时器等。
2. 不稳定，由于发生在 render 之前，很可能因为高优先级任务的出现而打断现有任务导致它们会被执行多次（另外两个`UNSAFE_`被废弃也是有这个原因）
3. 约束开发者，防止开发者滥用这几个生命周期，导致生命周期内的上下文多次被执行。

比如说，React18 的 Concurrent 模式下，render 阶段可能会被反复执行，或者中断到之后才执行，那么这三个生命周期也就可能会被反复执行，而导致他们没有对应的 componentDidUpdate 这种应该匹配的函数。如果这些生命周期内有副作用，那就会导致更复杂的问题（副作用被反复调用是很恐怖的事情）。
React18 的 Suspense 也会导致这个问题，由于在新版的 Suspense 中，正在挂起的组件树将会被“丢弃”，然后等到挂起状态结束后才会再次渲染，因此也是会出现重复调用的问题。

注意，这三个生命周期被重复执行不是因为时间分片机制导致的 workLoopConcurrent 中断，而是由于高优先级任务的进入。
前者不会放弃已经渲染的节点，因此这三个生命周期不会重复执行；但高优先级任务会导致整个 workInProgress 树被重新构建，就可能导致这三个生命周期被重复调用。

> 一起被挂上该标签的一共有三个：
>
> - `UNSAFE_componentWillMount`
> - `UNSAFE_componentWillReceiveProps`
> - `UNSAFE_componentWillUpdate`
>   这三个都是在`render`之前进行，废弃原因都类似。

#### 2. `UNSAFE_componentWillReceiveProps`

该生命周期执行是在更新组件阶段，在`render`之前，父组件或 props 发生更新之后调用，并且是收到 props 更新时第一个调用的；挂载阶段不会调用。
但是实际上只要父元素触发`render`，即有可能还是父元素自己状态更新，都会调用该函数。因此有可能即使 props 没变，该生命周期也会执行。

作用：

- 监听父元素的`render`
- 关联 props 和 state：如果某个 state 和 props 关系紧密，则会在这里获取到最新的 props，然后进行`setState`操作。
- 相对于`getDerivedStateFromProps`，这个方法可以访问组件实例，因此可以进行一些异步更新 state 的操作。

> 作用其实也说明了该函数被废弃的原因：可以在没做优化前提下会带来两次子组件的更新，第一次 props 改变，第二次 props 改变再改变 state。两次改变可能会导致闪烁或者其他更多的问题。

使用示例：

```js
UNSAFE_componentWillReceiveProps(nextProps){
  const { type } = newProps
  console.log('父组件render执行')
  this.setState(...)
}
```

#### 3. `UNSAFE_componentWillUpdate`

发生在在`render`之前，`shouldComponentUpdate`确认要更新之后，此时的 DOM 还没有更新。挂载阶段不会调用。

作用：

- 做一些获取 DOM 的操作，如在一次更新中，保存 DOM 之前的信息

```js
UNSAFE_componentWillUpdate(){
    const position = this.getPostion(this.node) /* 获取元素节点 node 位置 */
}
```

> `getSnapshotBeforeUpdate`可以替代该生命周期。

## 函数组件的生命周期

### useEffect

```js
useEffect(() => {
  return destory;
}, dep);
```

- 第一个参数 `callback`, 返回的 `destory` ， `destory` 作为下一次`callback`执行之前调用，用于清除上一次 `callback` 产生的副作用。
- 第二个参数作为依赖项，是一个数组，可以有多个依赖项，依赖项改变，执行上一次`callback` 返回的 `destory` ，和执行新的 `effect` 第一个参数 `callback` 。

对于 useEffect 执行， React 处理逻辑是采用`异步调用` ，对于每一个 `effect` 的 `callback`， React 会像`setTimeout`回调函数一样，放入任务队列，等到主线程任务完成，DOM 更新，js 执行完成，视图绘制完毕，才执行。所以 `effect` 回调函数不会阻塞浏览器绘制视图。

### useLayoutEffect

`useLayoutEffect` 和 `useEffect` 不同的地方是采用了`同步执行`

- `useLayoutEffect` 是在 DOM 绘制之前，这样可以方便修改 DOM ，这样浏览器只会绘制一次，如果修改 DOM 布局放在 `useEffect` ，那 `useEffect` 执行是在浏览器绘制视图之后，接下来又改 DOM ，就可能会导致浏览器再次回流和重绘。而且由于两次绘制，视图上可能会造成闪现突兀的效果。
- `useLayoutEffect callback` 中代码执行会阻塞浏览器绘制。

> 一句话概括如何选择 `useEffect` 和 `useLayoutEffect` ：修改 DOM ，改变布局就用 `useLayoutEffect` ，其他情况就用 `useEffect` 。

## 特殊的需要注意的生命周期

1. 在 constructor 中，不要把 props 直接赋给 state。这一点在函数组件中也适用

```js
constructor(props) {
 super(props);
 // 不要这样做
 this.state = { color: props.color };
}

// 函数组件
const FC = (props)=> {
  const [state,setState] = useState(props)
}
```

props 本身就是一个“上层的 state”，当 props 更新时组件也会更新，因此最好直接用 props，而不是赋给 state

2. `componentDidMount()` 里直接调用 `setState()`。它将触发额外渲染，但此渲染会发生在浏览器更新屏幕之前。如此保证了即使在 `render()` 两次调用的情况下，用户也不会看到中间状态。
3. `componentDidUpdate()` 中调用 `setState()`要注意它必须被包裹在一个条件语句里，即给予一个停止条件或限制，否则会导致死循环。

上面两点在`useEffect`中类似；如果没有指明依赖数组，在 useEffect 内部更新 state 就会导致死循环

```js
useEffect(()=>{
  setState(...)
})
```

`componentDidUpdate()`有三个参数，分别为：上一次的 props、上一次的 state 和`getSnapshotBeforeUpdate()`方法的返回值（不常用）

```js
componentDidUpdate(prevProps, prevState, snapshot);
```

因此可以在这个生命周期内部比较本次的 state 和上次的 state，props 也是同理。

4. `componentWillUnmount()` 中不应调用 `setState()`，是无效的，因为组件会被卸载。理论上除非是路由的形式，否则大多数情况下组件一被卸载就不会再挂载。

# React Ref

## ref 执行时机和处理逻辑

如果 ref 是一个函数，就可以看到 ref 的回调会在 ref 更新时执行两次。并且如果是第一次挂载，ref 指向的 dom 元素的值还会是空。
具体来说，对函数组件来说其实是比较两次的 ref 回调是否相同。

1. 如果 ref 函数跟上一次的 ref 函数不一致（引用比较），那么会在 Mutation 和 Layout 阶段分别调用`commitDetachRef` 和 `commitAttachRef`。观察下面的代码可知，这两个调用都会执行 refCallback，并且第一次参数是 null，第二次是 fiber.stateNode（即真实 dom）
2. 如果 ref 函数跟上一次的 ref 函数一致，则更新时不会调用 ref 函数

下面这个例子，两个 ref 一个是用 useRef 保存，一个是普通函数。由于普通函数会在更新时被重新声明，因此两次调用的函数都不同，结果就是 anonymous 这里会输出两次，h1Ref 这个函数会执行两次。useCallback 也可以；
![](https://pic.imgdb.cn/item/639ed32eb1fccdcd365d3009.jpg)
考虑到这种情况，refCallback 中一定要用判断是否是 null 的部分，始终记得 ref 可能是 null，只有不是 null 才能进行操作。

```js
export default function App() {
  const [, forceUpdate] = useState({});
  // 这个是保存的ref callback，不会改变
  const { current: standaloneRefCallback } = useRef((f: HTMLHeadingElement) => {
    console.log(`standalone`, f);
  });
  // 这个callback每次都会是新的
  const h1Ref = (d: HTMLHeadingElement) => {
    console.log(`anonymous`, d);
  };
  return (
    <div className="App">
      {visible && (
        <>
          <h1 ref={h1Ref}>Hello</h1>
          <h1 ref={standaloneRefCallback}>World</h1>
        </>
      )}
      <button type="button" onClick={() => forceUpdate({})}>
        forceUpdate
      </button>
    </div>
  );
}
```

出现这种情况，主要是因为 ref 的处理会执行两次。整个 Ref 的处理，都是在 commit 阶段发生的；React 底层用两个方法处理：`commitDetachRef` 和 `commitAttachRef` ，这两次正正好好，一次在 DOM 更新之前（Before Mutation），一次在 DOM 更新之后（Layout）。

```js
function commitDetachRef(current: Fiber) {
  const currentRef = current.ref;
  if (currentRef !== null) {
    if (typeof currentRef === "function") {
      /* function 和 字符串获取方式。 */
      currentRef(null);
    } else {
      /* Ref对象获取方式 */
      currentRef.current = null;
    }
  }
}

function commitAttachRef(finishedWork: Fiber) {
  const ref = finishedWork.ref;
  if (ref !== null) {
    const instance = finishedWork.stateNode;
    let instanceToUse;
    switch (finishedWork.tag) {
      case HostComponent: //元素节点 获取元素
        instanceToUse = getPublicInstance(instance);
        break;
      default: // 类组件直接使用实例
        instanceToUse = instance;
    }
    if (typeof ref === "function") {
      ref(instanceToUse); /* function 和 字符串获取方式。 */
    } else {
      ref.current = instanceToUse; /* ref对象方式 */
    }
  }
}
```

可以看出，commitDetachRef 会清空 ref 的值，而 commitAttachRef 才会绑定具体的 dom 元素上去。

## ref 什么时候才会被处理两次

上面的例子只是说明了 ref 被处理两次的现象和表面原因（两次 ref 不一样），那么具体到源码，这种执行两次的操作主要是因为当前 fiber 被打上了 Ref 标签，在稍后的更新中就会既执行 commitDetachRef 又执行 commitAttachRef。

标记 Ref 的方法是 markRef：

```ts
function markRef(current: Fiber | null, workInProgress: Fiber) {
  const ref = workInProgress.ref;
  if (
    (current === null && ref !== null) ||
    (current !== null && current.ref !== ref)
  ) {
    workInProgress.flags |= Ref;
    workInProgress.flags |= RefStatic;
  }
}
```

可以看到，标记的条件是 mount 阶段，或 update 阶段且 ref 和 current.ref 不同，即 ref 发生变化。

标记之后，在 Mutation 和 Layout 阶段会分别对此做处理。

如果组件更新时 ref 改变，那么该 ref 一定会被处理两次，其中第一次的 ref 值是 null，第二次指向正常的值。

![](https://pic.imgdb.cn/item/63a34527b1fccdcd36b08ce9.jpg)

---

当元素被卸载时，则会调用 safelyDetachRef 清除 ref 的值。具体来说则是该元素的 fiber 被打上了 delection 的 tag，在 commit 阶段就会清除 ref 的值。

```js
function safelyDetachRef(current) {
  const ref = current.ref;
  if (ref !== null) {
    if (typeof ref === "function") {
      // 函数式 ｜ 字符串
      ref(null);
    } else {
      ref.current = null; // ref 对象
    }
  }
}
```

## ref 工作流程

ref 可以根据 render 和 commit 分为两个阶段

- render 阶段：执行 ref 的标记
- commit：根据 ref 标记，在 Mutation 执行 commitDetachRef 清除 ref，在 Layout 阶段执行 commitAttachRef 绑定 ref。

比如 Mutation 阶段：

```ts
function commitMutationEffectsOnFiber() {
  /* 如果是 ref 更新，那么重置 ref */
  if (flags & Ref) {
    const current = finishedWork.alternate;
    if (current !== null) {
      commitDetachRef(current);
    }
  }
}
```

commitDetachRef 代码上面已经说过。

在 Layout 阶段，则会调用 commitAttachRef 重新给 ref 赋值

```ts
if (finishedWork.flags & Ref) {
  /* 更新 ref 属性 */
  commitAttachRef(finishedWork);
}
```

# React.lazy 和 Suspense

> 参考：https://segmentfault.com/a/1190000042711084

Suspense 是 React 提出的一种同步的代码来实现异步操作的方案。Suspense 让组件‘等待’异步操作，异步请求结束后在进行组件的渲染，也就是所谓的异步渲染。
理论上 Suspense 应该支持任何异步渲染的组件，但是目前只能支持由 React.lazy 创建的 LazyComponent。因此要说到 Suspense 的执行原理就离不开 lazy 的实现。

lazy 函数简化如下：

```js
function lazy(ctor){
  // payload，status表示当前状态，result初始化为动态导入的函数，后面then拿到具体值后就是具体模块对象
  const payload = {
    _status: Uninitialized,
    _result: ctor,
  };

  return {
    $$typeof: REACT_LAZY_TYPE,
    _payload: payload,
    _init(payload) {
      if (payload._status === Uninitialized) {
        const ctor = payload._result;
        const thenable = ctor();
        // 执行动态导入函数，取到返回值
        thenable.then(
          moduleObject => {
            if (payload._status === Pending || payload._status === Uninitialized) {
              // 一旦resolve，就将payload的属性修改，这样第二次调用就可以取到具体的模块
              payload._status = Resolved;
              payload._result = moduleObject;
            }
          },
          error => {
            if (payload._status === Pending || payload._status === Uninitialized) {
              payload._status = Rejected;
              payload._result = error;
            }
          },
        );
      }
      if (payload._status === Resolved) {
        const moduleObject = payload._result;
        return moduleObject.default;
      } else {
        // 第一次调用status一定没有改，走这里抛出result，这时的result仍然是动态导入函数
        throw payload._result;
      },
    }
  }
}
```

lazy 函数内部并没有立即返回动态导入的结果，而是先 throw 出一个 result。可以把 lazy 理解成一个返回实际模块值的 Promise，只是它通过一些骚操作（抛出错误、promise 绑定 Suspense 更新、两次遍历 Suspense）使得 Suspense 可以先渲染 fallback 再渲染 primary

具体流程是（下文将被 lazy 处理的组件成为 primary 组件）：

![](https://pic.imgdb.cn/item/63a4a44808b683016351afa1.jpg)
（这个图很笼统，还是得看下面的文字

1. 当 react 在 beginWork 的过程中遇到一个 Suspense 组件时，会首先将 primary 组件作为其子节点，根据 react 的遍历算法，下一个遍历的组件就是未加载完成的 primary 组件。
2. 当遍历到 primary 组件时，primary 组件会抛出一个异常。该异常内容为组件 promise（就是动态导入的函数），react 捕获到异常后，发现其是一个 promise，**会将其 then 方法添加一个回调函数，该回调函数的作用是触发 Suspense 组件的更新**（这一步是触发 Suspense 两次执行的关键）。并且将下一个需要遍历的元素重新设置为 Suspense，因此在一次 beginWork 中，Suspense 会被访问两次。
3. 又一次遍历到 Suspense，本次会将 primary 以及 fallback 都生成。primary 作为 Suspense 的直接子节点，但是 Suspense 会在 beginWork 阶段直接返回 fallback。使得直接跳过 primary 的遍历。因此此时 primary 必定没有加载完成，所以也没必要再遍历一次。本次渲染结束后，屏幕上会展示 fallback 的内容
4. 当 primary 组件加载完成后，会触发步骤 2 中 then，使得在 Suspense 上调度一个更新，由于此时加载已经完成，Suspense 会直接渲染加载完成的 primary 组件，并删除 fallback 组件。

注意这里 Suspense 的两次执行和两次遍历不一样

- 在一个同步任务中（一次调和）内 Suspense 就会被访问两次，原因是 primary 组件抛出异常，react 会同步地再走一次 suspense，目的是更改返回的子组件为 fallback ，但把 primary 作为 Suspense fiber 的 child；这样既跳过了 primary，又保证下一次更新时 child 就是 primary。
- 当 primary 加载完成后，此时是一个 Promise 微任务，这时会再触发一次 Suspense 内的更新，渲染 primary 并将 fallback 标记为删除

## 处理 primary 组件

lazy 会在 beginwork 执行到 primary 组件时被执行。beginwork 内的 mountLazyComponent 函数会负责执行，简化代码如下：

```ts
const props = workInProgress.pendingProps;
const lazyComponent: LazyComponentType<any, any> = elementType;
const payload = lazyComponent._payload;
const init = lazyComponent._init;
let Component = init(payload); // 如果未加载完成，则会抛出异常，否则会返回加载完成的组件
```

其实就是执行了上面的 init 函数，传入的是包含有动态导入函数的 payload 对象。

## 异常捕获

上面说过第一次同步执行完 init 之后会抛出一个异常。这个异常会被 react 统一处理，具体是这样：

```js
do {
  try {
    workLoopSync();
    break;
  } catch (thrownValue) {
    handleError(root, thrownValue);
  }
} while (true);
```

在 handleError 中有这样一段相关代码:

```ts
throwException(
  root,
  erroredWork.return,
  erroredWork,
  thrownValue,
  workInProgressRootRenderLanes
);
// 处理完错误后再次执行Suspense的地方
completeUnitOfWork(erroredWork);
```

核心代码需要继续深入到 throwException:

```ts
// 首先判断是否是为 promise
if (
  value !== null &&
  typeof value === "object" &&
  typeof value.then === "function"
) {
  // 获取到 Suspens 父组件
  const suspenseBoundary = getNearestSuspenseBoundaryToCapture(returnFiber);
  if (suspenseBoundary !== null) {
    suspenseBoundary.flags &= ~ForceClientRender;
    // 给 Suspens 父组件 打上一些标记，让 Suspens 父组件知道已经有异常抛出，需要渲染 fallback
    markSuspenseBoundaryShouldCapture(
      suspenseBoundary,
      returnFiber,
      sourceFiber,
      root,
      rootRenderLanes
    );
    // 将抛出的 promise 放入Suspens 父组件的 updateQueue 中，后续会遍历这个 queue 进行回调绑定
    attachRetryListener(suspenseBoundary, root, wakeable, rootRenderLanes);
    return;
  }
}
```

可以看到 throwException 逻辑主要是判断抛出的异常是不是 promise，如果是的话，就给 Suspens 父组件打上 ShoulCapture 的 flags，并且把抛出的 promise 放入 Suspens 父组件的 updateQueue 中。

throwException 完成后会执行一次 completeUnitOfWork，根据 ShoulCapture 打上 DidCapture 的 flags。 并将下一个需要遍历的节点设置为 Suspense，也就是下一次遍历的对象依然是 Suspense。这也是之前提到的 Suspense 在整个 beginWork 阶段会遍历两次。

## 绑定二次更新

上面说到如果 primary 组件加载完毕，就会更新一次 Suspense 组件。
因此在 Suspense 的 commit 阶段时，会遍历 updateQueue，绑定 promise 回调；当 promise 回调执行，就会在触发一次 suspense 的更新。

```js
function attachSuspenseRetryListeners(finishedWork: Fiber) {
  // 刚刚说抛出异常之后会向suspense组件的updateQueue中添加一个promise
  // 这里取出updateQueue，遍历找到那个promise，添加一个会更新Suspense的回调。
  const wakeables = finishedWork.updateQueue;
  if (wakeables !== null) {
    finishedWork.updateQueue = null;
    wakeables.forEach((update) => {
      // resolveRetryWakeable会在Suspense 的组件上调度一次更新
      const retry = resolveRetryWakeable.bind(null, finishedWork, update);
      // 将 retry 绑定 promise 的 then 回调
      update.then(retry, retry);
    });
  }
}
```

# React 更新

这部分主要是解决之前没弄清的地方。

## 更新方式

导致一个组件更新的情况有这几种：

- 主动调度 setState
- props 改变
- context
- 父组件 rerender，且当前组件没有采用 memo 等方式
- 外部状态导致的更新，比如状态管理库

第一种已经在上面说的很清楚了，这里来说一下 props 或 context 导致的更新。

### props/context 更新

实际上，在 beginwork 函数一开始，就比较了 props：

```ts
function beginWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
): Fiber | null {
  if (current !== null) {
    const oldProps = current.memoizedProps;
    const newProps = workInProgress.pendingProps;

    if (
      oldProps !== newProps ||
      hasLegacyContextChanged() ||
    ) {
      // 这个变量，用于控制通过props或context引起的更新
      didReceiveUpdate = true;
    } else {
      // 检查是否包含由updateLane引起的更新
      // 即是否由setState引起的更新。如果有的话那么didReceiveUpdate也是true
    }
  } else {...}
}
```

didReceiveUpdate 会在后续的过程中起到标识是否更新的作用，也可以直接调度更新，比如：

```ts
if (didReceiveUpdate || hasContextChanged) {
  // This boundary has changed since the first render. This means that we are now unable to
  // hydrate it. We might still be able to hydrate it using a higher priority lane.
  const root = getWorkInProgressRoot();
  if (root !== null) {
    const attemptHydrationAtLane = getBumpedLaneForHydration(root, renderLanes);
    if (
      attemptHydrationAtLane !== NoLane &&
      attemptHydrationAtLane !== suspenseState.retryLane
    ) {
      // Intentionally mutating since this render will get interrupted. This
      // is one of the very rare times where we mutate the current tree
      // during the render phase.
      suspenseState.retryLane = attemptHydrationAtLane;
      // TODO: Ideally this would inherit the event time of the current render
      const eventTime = NoTimestamp;
      enqueueConcurrentRenderForLane(current, attemptHydrationAtLane);
      // 直接调度更新
      scheduleUpdateOnFiber(root, current, attemptHydrationAtLane, eventTime);
    }
  }
}
```

在 updateFunctionComponent 函数内也有类似的：

```ts
if (current !== null && !didReceiveUpdate) {
  // 如果不需要更新，那就跳过当前组件的更新
  bailoutHooks(current, workInProgress, renderLanes);
  return bailoutOnAlreadyFinishedWork(current, workInProgress, renderLanes);
}
if (getIsHydrating() && hasId) {
  pushMaterializedTreeId(workInProgress);
}
// 如果didReceiveUpdate为true并且current存在，就会继续调和（rerender）当前组件
reconcileChildren(current, workInProgress, nextChildren, renderLanes);
return workInProgress.child;
```

### 父组件导致的更新

实际上，父组件更新导致子组件更新的原因并不是子组件真的要更新，而是子组件的 rerender 被视作是“更新”

react 判断是否是更新、执行更新还是挂载步骤的核心就是判断 current 是否存在。对于子组件来说，父组件更新必然执行 render，也必然会 render 到子组件。
即，即使子组件没有任何改变，也仍然会进入 reconcile 进行调和。

```ts
if (current === null) {
  workInProgress.child = mountChildFibers(
    workInProgress,
    null,
    nextChildren,
    renderLanes
  );
} else {
  // current存在，进入更新
  workInProgress.child = reconcileChildFibers(
    workInProgress,
    current.child,
    nextChildren,
    renderLanes
  );
}
```

这个差异对于函数组件来说并不明显，因为函数组件无论是初始化还是更新都是执行一遍函数，只是在 updateQueue 上会有不同（rerender 并不会增加 updateQueue）
但是对于类组件，由于调用的不同，执行的生命周期也不同。

```ts
if (instance === null) {
  resetSuspendedCurrentOnMountInLegacyMode(current, workInProgress);
  // In the initial pass we might need to construct the instance.
  constructClassInstance(workInProgress, Component, nextProps);
  mountClassInstance(workInProgress, Component, nextProps, renderLanes);
  shouldUpdate = true;
} else if (current === null) {
  // In a resume, we'll already have an instance we can reuse.
  shouldUpdate = resumeMountClassInstance(
    workInProgress,
    Component,
    nextProps,
    renderLanes
  );
} else {
  shouldUpdate = updateClassInstance(
    current,
    workInProgress,
    Component,
    nextProps,
    renderLanes
  );
}
```

resumeMountClassInstance/mountClassInstance/updateClassInstance 内部执行的生命周期是不同的，他们分别对应完全初始化状态（没有类组件的 instance）、初始化（没有 current）和更新（current）

---

这里引入了一个问题，对于父组件更新，以及它有的一个子组件，这两个组件内的生命周期执行顺序是什么样的？如果是挂载阶段呢？

其实大概就是这样的流程：

- 更新：父 gdsfp -> 父 shup -> 父 render -> 子 gdsfp -> 子 shup -> 子 render -> 子 gsn -> 父 gsn -> 子 cdup -> 父 cdup

render 之前的生命周期都是在 beginwork 阶段调用的，因此按照 beginwork 从上到下的顺序，应该是先父组件内的这些，一直到 render，进入子组件执行这些生命周期。
然后，render 阶段结束，进入 commit；commit 阶段内的 before mutation 会执行 getSnapShotBeforeUpdate，Layout 阶段会执行 componentDidUpdate，并且这两个都是在“归”的过程执行的（参考上面 commit 阶段的解析）。因此这两个阶段都是先执行子再执行父（从下到上）

如果考虑到旧的生命周期（cwrp、cwup），那就是这样（注意这两个和 gdsfp 和 gsn 不能共存）

- 父 cwrp（前提是组件新旧 props 必须不同） -> 父 shup -> 父 cwup -> 父 render -> 子 cwrp -> 子 shup -> 子 render -> 子 cdup -> 父 cdup

注意 cwrp 应该是在 shup 前面的。

最后，挂载阶段执行的应该是：

- 父 ctor -> 父 gdsfp -> 父 render -> 子 ctor -> 子 gdsfp -> 子 render -> 子 cdm -> 父 cdm

原理和更新的流程基本上差不多。
