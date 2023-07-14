---
title: React Native 学习
date: 2023-05-17 14:58:28
tags: 日常学习
categories: React
---

# 基本使用

rn 和 react 有一点不太相同的是，rn 更像是一个完善的框架，其内部包含了大量的内置组件，这些组件通常含有很多 props，因此学习 rn 的一个关键点就是搞清这些组件的使用方式，以及他们的内部实现、区别等。

## 交互

### 导航

rn 使用的导航是 react-navigation 开源库，本质上是一个类似 react router 的路由功能，通过堆栈记录的方式来完成页面的跳转、数据的传递等。
rnav 实现的导航和 react router 类似，实际上是应用内部的页面切换，比如很多应用的底部 tab 切换等。如果要在不同的 rn 打包出的页面间跳转，就不能使用 rnav 了。

> A key difference between how this works in a web browser and in React Navigation is that React Navigation's native stack navigator provides the gestures and animations that you would expect on Android and iOS when navigating between routes in the stack.

正如官方文档所说，rnav 就是在模拟 web 端的路由实现，在基础上还增加了适用于不同平台的跳转动画、手势识别等。
在使用方面，rnav 和 react router 也很相似，通过 Stack.Navigator 包裹 Stack.Screen，即可实现不同页面的切换。

在 rnav 中有一个特点，即用户离开上一个页面时，该页面对应的页面组件不会被卸载；当用户返回该页面时，也不会再次触发渲染。这点和在 web 端是不同的。

> 具有屏幕 A 和 B 的堆栈导航器。导航到 A 后，调用其 componentDidMount。当压入 B 时，它的 componentDidMount 也会被调用，但 A 仍然挂载在堆栈上，因此不会调用它的 componentWillUnmount 。

可以通过 rnav 提供的 hooks 或事件监听来得到用户当前是否进入或离开页面。

### 动画

rn 的动画有两个类别：

- Animated: 声明式动画。即，rn 通过预先声明一个动画的值、时间等，将其传递给特定组件的 style 属性。然后在恰当的时机内通过调用 start、stop 等方法来开始、结束动画。

举个例子：

```js
import React, { useRef, useEffect } from "react";
import { Animated, Text, View } from "react-native";

const FadeInView = (props) => {
  const fadeAnim = useRef(new Animated.Value(0)).current; // 透明度初始值设为0

  React.useEffect(() => {
    Animated.timing(
      // 随时间变化而执行动画
      fadeAnim, // 动画中的变量值
      {
        toValue: 1, // 透明度最终变为1，即完全不透明
        duration: 10000, // 让动画持续一段时间
      }
    ).start(); // 开始执行动画
  }, [fadeAnim]);

  return (
    <Animated.View // 使用专门的可动画化的View组件
      style={{
        ...props.style,
        opacity: fadeAnim, // 将透明度绑定到动画变量值
      }}
    >
      {props.children}
    </Animated.View>
  );
};
```

Animated.View 就是专门执行动画的组件，类似的还有 Animated.Text、Image 等等。
Animated 动画通常有着比修改 state 实现的动画更好的性能。一般来说动画的实现可能是通过 rAF 等 js 代码，不断通过 bridge 将动画数据发送给原生端进行渲染，并不会直接修改动画组件的 state。

如果在 timing 函数传入的参数中加一个属性，就可以启用原生动画驱动。通过下发给原生的形式，让动画在 native 上执行，从而大大提升动画的性能，避免和 js 线程的冲突。

> 启用原生驱动，我们在启动动画前就把其所有配置信息都发送到原生端，利用原生代码在 UI 线程执行动画，而不用每一帧都在两端间来回沟通。如此一来，动画一开始就完全脱离了 JS 线程，因此此时即便 JS 线程被卡住，也不会影响到动画了。

```js
Animated.timing(this.state.animatedValue, {
  toValue: 1,
  duration: 500,
  useNativeDriver: true, // <-- 加上这一行
}).start();
```

但是有一个缺陷是，这样的动画通常不能应用于布局改变，比如改变元素宽高、定位等，而只能用于元素的透明度、颜色等修改。

- LayoutAnimation：可以用于一些布局动画的实现，并且用法更简单。

比如：

```js
// part 1: 使用普通的state来定义变量
state = {
  bottom: 0,
};

// part 2:
// 此处假设我们在某个View的style中，使用了this.state.bottom这个变量作为bottom的值
LayoutAnimation.spring();
this.setState({ bottom: 20 }); // 只要改变state，就可以生效动画
```

spring 方法是 LayoutAnimation 的一个预设，其他预设还有 linear、easeout 等，和通常的动画速度曲线一致。
当然也可自定义动画，可以参考[官方文档](https://reactnative.dev/docs/animations#layoutanimation-api)

## 原生模块

React Native 的原生模块是指使用原生代码编写的 JavaScript 模块，可以在 React Native 应用程序中使用。这些模块可以访问设备硬件和 API，例如相机、传感器、文件系统等，以及执行与 React Native 框架本身无法直接处理的任务。
原生代码可以直接调用原生功能，然后通过一些打包的方式让 js 能够调用(或者是通过发布 npm 包)，还可以使用 React Native 框架提供的 Bridge API 与 JavaScript 代码进行通信
rn 中的原生模块主要是帮助一些需要原生 api 的功能实现，比如打开日历等。

## 原生方法

在 rn 中有时候需要直接修改某个原生模块的状态或样式，而不是通过 react 的 state。类似于在 web 端的 react 中，通过 dom 的方式直接修改元素。
rn 提供了一个 setNativeProps 方法，可以用于直接操作原生组件的状态。

setNativeProps 常用于出现性能问题的时候，比如频繁的动画、更新等。
比如，TouchableOpacity 这个组件就在内部使用了 setNativeProps 方法来更新其子组件的透明度：

```js
setOpacityTo(value) {
  // Redacted: animation related code
  this.refs[CHILD_REF].setNativeProps({
    opacity: value
  });
},
```

这个过程相当于跳过了 react，直接去更新原生组件的状态。

其他类似的原生方法还有

- measure：测量元素宽高，异步返回
- measureInWindow：测量元素位置，异步返回
- focus 和 blur：聚焦 input 或 view

## 性能

### 性能综述

rn 的性能主要关注点有以下几个：

1. 帧率。帧率又分为 ui 帧率和 js 帧率

- js 帧率：即 react 的执行在 js 线程上有没有每一帧都被响应。如果线程中的任务发生冲突，就有可能造成不能及时响应的情况，相当于是 js 的丢帧。最常见的情况就是 react 的渲染开销非常大，导致线程在很长一段时间内都在执行 react 的渲染，而不会响应事件回调和动画的执行，表现上就是卡顿。
- ui 帧率：即视觉上的，页面有没有发生卡顿，比如动画是否能稳定帧数、滚动等操作有没有卡顿等。不同于浏览器，在 native 端的渲染并不会被 js 阻塞，因此由 native 控制的动画、响应等（比如 scroll）不会被 js 线程所影响。

2. 编译性能。通常指的是 rn 项目 build 时的时间开销和产物。前者通常不用过于关注，而后者则可以通过合理的优化方式来减少包体积。
3. 列表性能。这是一个专项的性能，通常指的是列表的渲染速度、加载、更新时的性能，在组件那一部分有优化的详述
4. React 渲染性能。本质上，在 web 端也要关注 React 渲染，这和 rn 本身无关，而是 react 的性能要求。具体来说就是尽可能减少额外渲染，比如通过 memo 等优化方式。

> 官方文档额外提到了 immutable，即不可变数据结构。immutable 是指一类不可变的对象等，对其修改会返回一个新的对象，同时不会造成过大的深拷贝性能开销。具体可以参考https://blog.logrocket.com/immutability-react-should-you-mutate-objects/和https://github.com/camsong/blog/issues/3

### RAM Bundles 和内联引用优化

类似于在 web 端进行的分包优化，在 rn 中也有一些手段可以对模块进行分包、动态加载等方式。具体来说有两种

- 使用 RAM 格式的包
- 通过 require 方式进行内联引用优化。其实就是一种代码分割

RAM 的全称为 Random Access Modules，它是⼀种 bundle 的格式。RAM bundle 的作⽤是：让打包后的产物以 JS 模块（⼀个 JS ⽂件会被打包成⼀个JS 模块）粒度获取，可有实现按需进⾏加载。
如下图所示，加载 bundle 的时候会⾸先加载⼀段初始代码，代码⾥⾯会执⾏⼊⼝模块，但是这个时候⼊⼝模块 A 还没有被加载到 JS 引擎中，所以会去加载 A 模块并执⾏，执⾏ A 模块时⼜会执⾏ A 模块的依赖模块，直到所有依赖模块被执⾏，⾸屏展示。

![](https://pic.imgdb.cn/item/64b04c571ddac507cc240b51.jpg)

RAM bundle 需要配合内联引⽤⼀起使⽤才会有优化效果，否则的话，模块虽然是被⼀个个加载进去的，但是在⼊⼝模块加载后，会直接将当前 bundle 所⽤到的所有模块都⼀个个的加载进去，反⽽拖慢了⻚⾯加载时间。配合内联引⽤可以将⾮⾸屏模块，延迟到需要使⽤时才加载，从⽽实现⾸屏优化。

RN官⽅打包⼯具 metro 对于 RAM bundle 提供了两种实现，分别是：File RAM bundle 和 Indexed RAM Bundle。

- File RAM Bundle：每个模块都存储为⼀个⽂件，名称为 js-modules/${id} .js，外加⼀个名为 UNBUNDLE 的⽂件。这种打包⽅式存在⼩⽂件过多，⽂件损坏⼏率变⼤的问题

![](https://pic.imgdb.cn/item/64b04dae1ddac507cc25b78b.jpg)

- Indexed RAM Bundle：这种打包⽅式，打包出来的包格式为⼆进制⽂件，但仍然以每个文件为单位。内部维护了一个偏移表，偏移表⾥⾯存储了每个 module 的代码开始位置和代码⻓度。通过偏移表可以确定每个module的位置

---

按需引用不等同于`import()`方式的懒加载。具体来说，有以下区别：

1. 包内懒加载的 import() 的返回值是⼀个 Promise ，导致代码逻辑会编程异步，处理起来不优雅。⽽ require 是⼀个同步⽅法不存在这种问题
2. 由于 import(xxx) 会⽣成很多⼩⽂件，会使⽂件损坏出现的概率变⾼，造成业务不可⽤，⽽ indexed RAM bundle 是将若⼲⼩⽂件组合成⼀个⼆进制⽂件，避免了这个问题
3. 再包内懒加载中，假如我们懒加载的 A 模块 和 B模块都同时依赖了 C 模块，那么 C 是打进⼊⼝⽂件 (index.js)，还是 A 和 B 分开打呢？如果打进⼊⼝⽂件，common js 会越来越多，同样会影响⾸屏性能。如果分别打包到 A 和 B 模块中，会造成 Bundle 体积增加以及 C 被重复加载的问题，⽽内联引⽤ + indexed RAM bundle 的最⼩加载粒度是⽂件，所以不存在这种问题。



## 组件

### 核心组件

核心组件列表：https://reactnative.cn/docs/components-and-apis

rn 中的基础组件主要指 View、Text、TextInput 这些没有经过过多封装的组件，使用方式比较基础。
除了基础组件之外，还有诸如 Button 这样的交互组件、FlatList 这样的列表组件，以及 Modal 这样的功能组件。这些组件统称核心组件，指的是 rn 默认提供的一些组件。

### 列表组件

React Native 有好几个列表组件，先简单介绍一下：

- ScrollView：会把视图里的所有 View 渲染，直接对接 Native 的滚动列表，也不会复用，就是最基本的渲染。当列表数据非常多时，view 会等到所有项都渲染完成再显示
- VirtualizedList：虚拟列表核心文件，使用 ScrollView，长列表优化配置项主要是控制它
- FlatList：使用 VirtualizedList，实现了一行多列的功能，大部分功能都是 VirtualizedList 提供的
- SectionList：使用 VirtualizedList，底层使用 VirtualizedSectionList，把二维数据转为一维数据

后三者其实都是虚拟列表，师出同源。rn 的虚拟列表实现方式和简单原理可以参考https://www.cnblogs.com/skychx/p/react-native-flatlist.html。
这篇文章有对源码的简单解析：https://shengshuqiang.github.io/2019/07/17/%E8%BF%9B%E5%87%BBReactNative-FlatList%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.html

#### rn 中的虚拟列表

基本结构：都采用下图这种方式：

![](https://pic.imgdb.cn/item/649172e11ddac507cc145937.jpg)

FlatList、VirtualizedList、SectionList 的基本实现原理都相同，关键点有以下几个：

1. 只渲染可视区域。这是最基本的内容，VirtualizedList 会通过异步计算的方式确定当前渲染的位置、需要渲染的元素，然后只渲染这部分内容。而之前的列表项则会被选择性的回收复用或销毁以减小内存，并使用上下空白区域去填充空间。并且可视区域的列表项也并不只是在可视区域内的大小，还会超出一些部分。

2. 复用和回收。虚拟列表的复用过程与 React 的 diff 算法类似。在 React 中，diff 算法通过比较元素的 key 和 type 来决定是否复用之前的虚拟 DOM 节点，从而减少对实际 DOM 的操作，提高性能。同样地，在虚拟列表中，组件会根据列表项的 key 属性来判断是否复用之前的列表项组件。

当用户滚动虚拟列表时，组件会根据可见区域计算出需要渲染的列表项，并根据这些列表项的 key 属性来判断是否复用之前的列表项组件。如果列表项的 key 属性稳定且类型相同，组件会将之前渲染过的列表项组件进行复用，只更新列表项的数据，而不重新创建新的组件实例。这样可以避免不必要的渲染和 DOM 操作，提高性能和内存效率。

虚拟列表的复用过程是一种优化策略，通过减少组件的创建和销毁操作，可以有效降低 CPU 消耗，提高列表的滚动流畅度。它与 React 的 diff 算法相互补充，共同提供了高效的列表渲染和更新机制。

3. 异步计算和渲染。VirtualizedList 通过异步方式加载和渲染新的列表项，以避免阻塞主线程和用户操作。异步加载的过程可以分为两个阶段：计算和渲染。

在用户滚动到未渲染的列表项范围时，VirtualizedList 会触发异步计算的过程。在这个过程中，组件会根据当前滚动位置和可见区域的变化，计算出需要加载的新列表项的数量和位置。这个计算过程是异步的，它会在后台线程中进行，不会阻塞主线程的运行。
在计算完成后，VirtualizedList 会触发异步渲染的过程。在这个过程中，组件会根据计算得到的新列表项的数量和位置，异步地加载数据并渲染新的列表项。这个渲染过程也是异步的，它会在后台线程中进行，并在渲染完成后将新的列表项渲染到屏幕上。
同时渲染和计算的过程都会在用户操作和动画执行之后，尽可能减少 cpu 的抢占而造成的阻塞。

4. 渲染优先级。在 FlatList 中，列表项的渲染优先级是根据滚动速度和方向来动态调整的。当用户进行快速滚动时，FlatList 会根据当前的滚动情况，优先加载和渲染可见区域附近的列表项，以保持滚动的流畅性。这个优化策略可以减少不必要的渲染和资源消耗，提高性能。

具体而言，FlatList 会根据以下几个因素来决定列表项的渲染优先级：

- 滚动速度：如果用户进行快速滚动，FlatList 会尽量快速地加载和渲染新的列表项，以避免出现白屏。在这种情况下，FlatList 可能会优先加载和渲染离可见区域较远的列表项，以跟上滚动的速度。
- 滚动方向：当用户滚动到一个新的位置时，FlatList 会优先加载和渲染滚动方向上的列表项。例如，如果用户向下滚动，FlatList 会优先加载和渲染下方的列表项。
- 可见区域：FlatList 会根据可见区域的大小和位置来确定哪些列表项是可见的。只有可见区域内的列表项才会被加载和渲染，而不可见区域的列表项会被延迟加载。这样可以减少不必要的渲染和资源消耗。

#### 列表优化

对于不同的 rn 列表组件，优化方式也不尽相同。以常用的 FlatList 为例，优化的大方向有两个：

1. 通过特殊的 props 进行优化，比如 getItemLayout 等
2. 通过对数据的处理或其他列表本身之外的方式进行优化

优化方式在官方文档已经讲的很清楚了：https://reactnative.cn/docs/optimizing-flatlist-configuration

#### FlatList

FlatList 除了包含虚拟列表的基本功能之外，还有一些其他组件功能，常用于 native 开发中，比如：

- 支持水平布局模式。
- 行组件显示或隐藏时可配置回调事件。
- 支持单独的头部组件。
- 支持单独的尾部组件。
- 支持自定义行间分隔线。
- 支持下拉刷新。
- 支持上拉加载。
- 支持跳转到指定行（ScrollToIndex）。
- 支持多列布局。

这其中比较关键的是上拉下拉以及跳转，这也是在开发中常用的方法。

FlatList 有几个比较重要的使用要点：

1. 当某行滑出渲染区域之外后，其内部状态将不会保留。也就是说对于每个 ListItem 组件来说，滑出渲染区域之后组件将会被卸载或重置复用，其上的状态不会保留；

比如一个功能是修改列表的某一项，如果只是修改了对应的 ListItem 组件内部的 state 或 props 则不能保留，除非修改的是 data 源数据。

2. 组件继承自 PureComponent，如果其 props 在浅比较中是相等的，则不会重新渲染。所以请先检查你的 renderItem 函数所依赖的 props 数据（包括 data 属性以及可能用到的父组件的 state），如果是一个引用类型（Object 或者数组都是引用类型），则需要先修改其引用地址（比如先复制到一个新的 Object 或者数组中），然后再修改其值，否则界面很可能不会刷新。

上面这一段是官方文档的内容。其实大概意思就是更新 FlatList 的 data 属性时需要修改索引，否则不会更新。也就是说即使你通过 setState 修改了 data，但 data 如果没有被重新创建，就还是不会更新。

举个例子：

下面这样修改 state 是不会让 FlatList 更新的：

```js
data.push({ id: "xxx", name: "ttt" });
setData(data);
```

这样是可以的：

```js
data.push({ id: "xxx", name: "ttt" });
setData([...data]);
```

> 当然这种方式容易造成内存的浪费，如果频繁增加元素，那么每次 data 都需要重新创建。为了优化这个问题，我们可以采用两个数组交替复制的方式。这是一个小技巧，可以参考https://juejin.cn/post/7201425436835151930

除了上面这种情况之外，如果列表还依赖于其他 state，还可以使用 extraData 属性来控制 FlatList 更新。当然 extraData 也和 data 一样，也有浅比较的检查。

我们可以修改 extraData，来让 FlatList 强制更新。比如：

```js
const App = () => {
  const [data, setData] = useState(["Item 1", "Item 2", "Item 3"]);
  const [extraData, setExtraData] = useState(false);

  const updateData = () => {
    setData(["Item 1", "Item 2", "Item 3", "Item 4"]);
    setExtraData(!extraData); // 切换extraData的值，强制FlatList重新渲染
  };

  return (
    <View>
      <FlatList
        data={data}
        keyExtractor={(item) => item}
        renderItem={renderItem}
        extraData={extraData} // 将extraData属性设置为依赖项的状态值
      />
      <Button title="Update Data" onPress={updateData} />
    </View>
  );
};
```

当然这种方式不是很好。大多数情况下，extraData 主要用于处理额外依赖数据的情况。

3. 为了优化内存占用同时保持滑动的流畅，列表内容会在屏幕外异步绘制。这意味着如果用户滑动的速度超过渲染的速度，则会先看到空白的内容。这是为了优化不得不作出的妥协，你可以根据自己的需求调整相应的参数，而我们也在设法持续改进。

这也是上面介绍虚拟列表时提到的，可能的白屏情况。FlatList 提供了很多可能的调优属性，比如

- getItemLayout：返回列表元素的偏移、高度等信息，避免动态测量内容尺寸的开销。如果不提供，FlatList 需要通过自行测量的方式得知每个元素的偏移量等属性，就会导致每次计算开销增大。（但是如果每个元素高度不确定这个属性就不能用）
- keyExtractor：用于为给定的 item 生成一个不重复的 key。如果没有这个属性，默认抽取 item.key 作为 key 值。若 item.key 也不存在，则使用数组下标。因此最好提供好 key 值，key 在 react 的重要性不言而喻。
- maxToRenderPerBatch：每次批处理更新时更新多少项目，或者说在渲染时每批最多渲染多少个列表项。默认情况下，FlatList 会将更新操作批处理成一批，以减少重绘的次数，从而提高性能。这个和下面的属性都是用于控制批处理过程的。
- updateCellsBatchingPeriod：用于设置批处理的时间间隔（以毫秒为单位），如果在这个时间间隔内有多个更新操作，则这些更新操作会被批处理成一批。如果设置为 0，则表示禁用批处理，每个更新操作都会立即重绘。**如果不设置 updateCellsBatchingPeriod 属性，则默认禁用批处理。**

> 批处理，可以让 FlatList 能够在一段时间内更新一组 Cell，而不是一次更新所有 Cell。
> 批处理和虚拟列表的渲染无关，它主要指的是当 FlatList 需要更新时如何进行分批更新，而非一次性更新全部项。
> 比如在 FlatList 中进行一系列更新操作，例如添加、删除、移动或更新列表项。这些更新操作将被添加到批处理队列中，而不会立即执行重绘操作。
> FlatList 还会等待 updateCellsBatchingPeriod 属性所设置的时间间隔。在这个时间间隔内，如果有其他更新操作，则这些更新操作也将被添加到批处理队列中。
> 最后当时间间隔到达后，FlatList 将执行批处理操作，将批处理队列中的所有更新操作合并成一个单独的操作，并执行重绘操作。这样，就可以减少重绘的次数，从而提高性能。
> 这个过程类似 react 的批量更新，大致思路是相同的。

4. 默认情况下每行都需要提供一个不重复的 key 属性。你也可以提供一个 keyExtractor 函数来生成 key。

# 原理

rn 的原理部分，除开 react 和 native 端的实现，核心部分其实就是两个：

- 渲染：即如何从 jsx 渲染成为 native 的元素
- 通信：即 js 和 native 语言之间是怎么交互的，怎么样传递信息、互相调用；

参考：https://km.sankuai.com/page/337425524
https://juejin.cn/post/6844903442079563784
http://blog.cnbang.net/tech/2698/
https://km.sankuai.com/page/883771103

## 架构（旧版）

rn 的基本架构如下图

![](https://pic.imgdb.cn/item/64aeeecc1ddac507cc02f153.jpg)

从图中可以看到有几个关键的部分

- **js 代码**：即 react 代码的部分
- **js 引擎**：解释和执行 JavaScript 代码。在 React Native 里面，JavaScriptCore 负责 bundle 产出的 JS 代码的解析和执行。

js 引擎通常有几个：

- Hermes：最新的安卓端 js 引擎，优势是比较轻便，并且针对安卓做了专门的优化
- JavaScriptCore：在 ios 上的引擎，也是 safari 的引擎
- V8：web 端引擎，在 native 端的性能上不如上面两个

- **bridge**：原生端和 JavaScript 交互是通过 Bridge 进行的，Bridge 的作用就是给 React Native 内嵌的 JS Engine 提供原生接口的扩展供 JS 调用。

所有的本地存储、图片资源访问、图形图像绘制、3D 加速、网络访问、震动效果、NFC、原生控件绘制、地图、定位、通知等都是通过 Bridge 封装成 JS 接口以后注入 JS Engine 供 JS 调用。理论上，任何原生代码能实现的效果都可以通过 Bridge 封装成 JS 可以调用的组件和方法, 以 JS 模块的形式提供给 RN 使用。

- **原生模块**：即 native 侧的 api、模块等，可以通过 bridge 让 js 进行调用

## 渲染

### 渲染线程

参考：https://zhuanlan.zhihu.com/p/388681402

和渲染树一样，rn 的渲染线程也有三个。无论是还在开发中的新架构，还是当前的旧架构，一个 React Native App 渲染过程都会涉及到三个线程

- Main thread: 主线程又称 UI thread，主要负责 UI 的渲染以及用户行为的监听等等，是 App 启动时首先创建的线程
- JavaScript thread: js 执行的线程
- Shadow thread: 可以理解为是在 native 侧，负责和 react 的的虚拟 dom 进行交互，并控制 native 渲染的一个线程。

react 存在 diff 算法，通过 diff 算法尽可能减小虚拟 dom 的更新损耗。在 rn 中也有类似的虚拟 dom 结构，也有 diff 的过程；但是不同于浏览器环境中 React DOM 直接调用浏览器 API 来完成真正的 DOM 更新操作，Native 环境下 React Native 是通过 Bridge ，将需要变更的指令（Commands）以字符串的方式发送到 Native Side，而对应在 Native 这边负责处理这些指令的进程就是 Shadow Thread。
Shadow Thread 通过维护一个 Shadow Tree 来计算 "Virtual DOM" 在 Native 页面的实际布局，然后通过 Bridge 异步通知 Main Thread 渲染 UI。
Shadow Tree 可以理解为是 "Virtual DOM" 在 Native 的映射，拥有和 Virtual DOM 相同的树形层级关系
因此我们可以理解为，shadowThread 就是一个用于“接收布局消息”的线程，然后控制 native 的渲染结构。

### 渲染树

在 React Native 的世界里，总共存在三种树，他们分别是

- Fiber 树
- Shadow 树
- 原生 UI 树

> React Native 维护和更新开发者创建的 Component 的状态，创建出表达 UI 的第一种数据结构——Fiber 树。然后通过 Fiber 算法高效地执行树的更新动作，输出的结果表现为原生侧的 Shadow 树数据结构，再经过一些最终调整后，创建出原生平台上的 UI 结构——Android View 或 iOS UIView 所构成的树结构，由原生平台的渲染机制来完成屏幕内容的渲染。

其中，fiber 树很熟悉就不再赘述，而 shadow 树则是原生渲染过程中的一个重要结构，作为最终渲染之前的一步。

大致的流程为：

- JavaScript 层通过维护 Fiber 树，计算出数据结构变化的副作用，生成结点更新指令（createView / updateView / setChildren 等）送往 Java 层
- 根据上面的节点更新指令，实体结构 shadow tree 在 Java 层 UIImplementation 中生成。
- UIImplementation 的下一级 NativeViewHierarchyOptimizer 类，负责将 shadow tree 转译为真正的原生 view 树。

shadow tree 和原生 view 树有何不同呢？后者去掉了前者结构之中多余的 “纯布局 view”（layout-only view，即只影响 layout 但自身不绘制任何东西的 view）。
简单理解就是原生 view 树去掉了一些不进行绘制的部分，就像浏览器中的 dom 树和 layout 树之间的差别一样。

### 渲染过程

rn 项目的首次渲染流程可以分为几步：

1. Native 打开 RN 页面（在此之前，js 上下文、引擎已被加载完毕，桥已建立，bundle 也被加载到 native 中开始执行）
1. JS 线程运行，Virtual DOM Tree 被创建
1. JS 线程异步通知 Shadow Thread 有节点变更
1. Shadow Thread 创建 Shadow Tree
1. Shadow Thread 计算布局，异步通知 Main Thread 创建 Views
1. Main Thread 处理 View 的创建，展示给用户

![](https://pic.imgdb.cn/item/64ae96351ddac507ccc1a3fc.jpg)

更新时的流程则可以分为四步

1. Virtual DOM Tree 发生变更（比如 background-color 发生变化）
1. JS 线程异步通知 Shadow Thread 有节点变更
1. Shadow Tread 更新 Shadow Tree，计算新布局，同时异步通知 Main Thread 更新 View
1. Main Thread 更新 UI

如上面的图所示，在不同线程之间进行交互和信息传递都是通过 bridge 实现的。Bridge 由 C++ 实现，可以复用在 iOS & Android，用与 JS side 和 Native side 相互通讯 (two-way-communacation)

> bridge 是老的 rn 架构，当前新的架构则是 jsi 为核心的动态方案，具体后面会提到

![](https://pic.imgdb.cn/item/64ae96af1ddac507ccc3c77c.jpg)

这种方式的优点是，Main Thread 不会阻塞 (block)，也就是说，UI 渲染是流畅的，所以用户体验不会卡顿
但是桥调用方式使得所有交互都是异步的。对于需要比较强烈实时性的场景，比如频繁的交互、快速的更新、动画等都有可能造成一种延迟感。并且当需要传输的数据非常大时，本身的内存消耗和对于数据处理的时间消耗也会非常大。

另外，rn 的渲染消耗要比 web 端大得多。因为 react 完成 fiber 的创建之后，shadow 线程也要维护一棵树来实现构建，这个过程比 web 端的 dom 操作要繁琐很多，更何况线程之间的通信本就是异步、效率不高的。因此 rn 的优化中对于渲染控制应该要求更高

从上向下看 rn 的渲染过程则为：

![](https://pic.imgdb.cn/item/64ae974a1ddac507ccc65dbe.jpg)

1. React Component，也就是 JS 代码 render 的 React 组件
1. React Native Renderer 会将 React Component 转换为 "Virtual DOM tree"
1. React Native Bridge 会将 "Virtual DOM tree" 更新的 commands 传递到 Native Side
1. UI Manager Module 是 Native Side 处理 commands 的模块，负责根据 commands 创建/更新（与 "Virtual DOM tree" 对应的 Native 这边的） Shadow Tree
1. Shadow Tree / Layout，一旦 Shadow Tree 完成更新，就会触发每个 Shadow Tree Node 的布局信息
1. View Managers，一旦有了每个节点的布局信息，便会通知对应 View Manager 渲染 UI

---

rn 和 react 的渲染核心区别就在于，react 会把虚拟 dom 结构利用 dom api 转化成真实 dom 结构，而 rn 则是通过一个 API 去创建 Native 的 View，再由 native 进行原生的渲染操作。
React Native 使用的全部是自定义标签，和 Native View 一一对应，所以需要通过一个 API 去创建 Native 的 View，这个 API 就是 UIManager，主要执行从 Virtual DOM -> RenderDOM 节点的 CRUD 操作

- UIManager.createView：创建节点
- UIManager.measure / measureXXX：获取节点布局信息
- UIManager.findSubView
- UIManager.updateView：更新节点信息
- UIManager.manageChildren / setChildren：设置树结构信息，批量删除、增加、移动节点等

UIManager.js 是什么？它是从 NativeModules.js 中导出的，NativeModules.js 通过 Native 设置的 NativeModuleRegistry 的信息生成名字和实际类型的映射表，NativeModules 本身也是类似 Java 代理的风格去生成的，最终 UIManager.createView 这样一个函数调用，会变成消息队列的一个命令调用

```js
BatchedBridge.enqueueNativeCall(moduleID, methodID, args, onFail, onSuccess);
```

对于每个要渲染的虚拟 dom 元素，rn 会像传递其他消息一样传递一个包含 moduleId、methodId 等参数的数据，然后通过通信的方式传递给 native 侧。比如一个`<Text>`jsx 元素可能会变成这样的参数

```
{
	moduleId: 5,
  methodId: 2,
  args: [
    15405,
    "RCTText",
    31,
    {
      accessible: true, allowFontScaling: true
      ellipsizeMode: "tail"
      fontSize: 14
      fontWeight: "500"
    }
  ]
}
```

最后这个数据会到达 native 侧，再由 native 侧的代码执行具体的渲染步骤。

具体的渲染过程可以参考：https://km.sankuai.com/page/337425524#id-3.%E6%8E%92%E7%89%88%E6%B8%B2%E6%9F%93

## 通信

这里的通信指的是 js 和 native 语言之间的交互、通信，具体来说其实是两者的互相调用。在 ios 和安卓端的 native 语言有所不同，前者是 objective-c，后者是 java。在具体实现上有所不同，但在大致原理上是近似的。

### 基本通信原理

假设不通过 rn，js 和 native 仅通过引擎也可完成基本的通信。比如 native 侧和 js 侧都把想共享的模块暴露到全局，让对方获取并调用。但这样会导致大量的全局变量污染，所以为了规范这个通信过程，React Native 自己实现了 Bridge。

bridge 本质上就是 js 和 native 的互相调用，夹杂着一些参数的传递。

### js 调用 native

js 是一个不能自执行的语言，意思就是 js 必需一个执行它的“引擎”。在 ios 上是 JavaScript Core，在安卓上则有专门执行 js 的 c++部分。
因此 js 调用 native 的过程，本质上可以理解为是引擎执行 js，然后将 js 的执行结果或在 js 中的某些操作传递给 native 的过程。

以 ios 上的 objective-c 为例，React Native 解决这个问题的方案是在 Objective-C 和 JavaScript 两端都保存了一份配置表，里面标记了所有 Objective-C 暴露给 JavaScript 的模块和方法。这样，无论是哪一方调用另一方的方法，实际上传递的数据只有 ModuleId、MethodId 和 Arguments 这三个元素，它们分别表示类、方法和方法参数，当 Objective-C 接收到这三个值后，就可以通过 runtime 唯一确定要调用的是哪个函数，然后调用这个函数。

简单来说，当 JS 调用 Native 模块的时候，会调用一个 Native 暴露出来的全局方法，并通过传入要调用的 moduleName 、methodName、callback 参数给这个方法，然后这个方法再*通知*给 Native 侧找到相应的模块并执行。

![](https://pic.imgdb.cn/item/64aef3a61ddac507cc0aafd9.jpg)

---

另外，js 不会主动将数据发送给 oc。也就是说必须当 oc 在某些条件下执行 js 时，才会执行上面步骤，而不是由 js 代码主动驱动数据的发送。JS 不会主动传递数据给 OC，在调 OC 方法时，会把 ModuleID,MethodID 等数据加到一个队列里，等 OC 过来调 JS 的任意方法时，再把这个队列返回给 OC，此时 OC 再执行这个队列里要调用的方法。

这样的执行方式就类似于事件触发。native 开发里只在有事件触发的时候才会执行代码，这个事件可以是启动事件，触摸事件，timer 事件，系统事件，回调事件。而在 React Native 里，这些事件发生时 OC 都会调用 JS 相应的模块方法去处理，处理完这些事件后再执行 JS 想让 OC 执行的方法，而没有事件发生的时候，是不会执行任何代码的。

### native 调用 js

native 调用 js 代码就比较简单了，通过 moduleid 和 methodid 完成方法的调用，通过这两个参数可以找到 JS 侧定义的方法模块。

![](https://pic.imgdb.cn/item/64aef4881ddac507cc0c090f.jpg)

---

对于 Objective-C 来说，执行完 JavaScript 代码再执行 Objective-C 回调毫无难度，难点依然在于 JavaScript 代码调用 Objective-C 之后，如何在 Objective-C 的代码中，回调执行 JavaScript 代码。
目前 React Native 的做法是：在 JavaScript 调用 Objective-C 代码时，注册要回调的 Block，并且把 BlockId 作为参数发送给 Objective-C，Objective-C 收到参数时会创建 Block，调用完 Objective-C 函数后就会执行这个刚刚创建的 Block。
Objective-C 会向 Block 中传入参数和 BlockId，然后在 Block 内部调用 JavaScript 的方法，随后 JavaScript 查找到当时注册的 Block 并执行。

本质上 native 调用 js 就是一个回调的过程，即 js 预先把自己的回调也传给 oc，然后 oc 只需要执行这个 block 即可。

![](https://pic2.imgdb.cn/item/6464bee50d2dde5777c3dfd9.jpg)

## 其他概念

### bundle

在 React web 应用中，打包，部署到上线的产物，是一个 html ，css，js 文件的集合体，最后把这些产物放在服务器上就可以了。
但是在 RN 中，最后打包产物是一个 js 文件，叫做 jsbundle ，在 Native 端运行 RN 项目，本质上是远程拉取了 jsbundle ，并通过上述的 js 引擎运行当前 jsbundle，每次运行一个 bundle 就需要外层容器提供一个 js 引擎。

一个 bundle 可以对应一个页面，也可通过路由的形式对应多个页面。

bundle 的产生其实就是在 rn 的入口文件中注册的组件。在 RN 中每一个应用都有一个入口文件，RN 中提供了注册根本应用的方法，那就是 AppRegistry，这一点和 React web 应用会有一些区别，web 应用中，主要依赖于 react-dom 中提供的 api ，但是在 RN 项目中，无需再下载 react-dom，取而代之的是 react-native 包。

```js
import { AppRegistry } from "react-native";
/* 根组件 */
import App from "./app";

AppRegistry.registerComponent("Root", () => <App />);
```

在应用程序启动阶段，当 native 端加载完成引擎、建立起桥后，就会下载 js 文件，然后通过 js 引擎来加载 bundle，执行如上所示的代码。从这里就会进入 react 的渲染过程，比如执行组件等

### rn 应用的启动流程

以安卓侧为例子，RN 应用的启动流程如下：

1. 创建 JS 引擎，注册 Native 和 C++ ,C++ 和 JS 层的通信桥，同时会创建 JS 和 Native UI 线程队列。
1. 异步加载 JS Bundle，这一部分是 JS 交给 JS 引擎去处理，会对 JS 文件进行加载和解析，当然解析的时长受到 JS 文件大小的影响。
1. 当 JS 解析完毕之后，接下来就要启动 RN 应用了，包括运行 RN 提供的 AppRegistry 入口。
1. 构建组件树，包括执行 React 运行时代码，渲染组件，接下来通过 Native 提供的 UIManager，把虚拟 DOM 树在 Native 应用中渲染出来，视图也就正常呈现了。

注意这里是启动流程，上面提到的渲染流程实际上是启动流程的最后一步，也就是在 js 解析之后，开始执行渲染才会进入渲染流程

这个启动的过程可以简单分解为

```
上下文创建 + 渲染
```

上下文创建，包括 JS 引擎的构建，解析并运行 JS Bundle，准备 JS 上下文是最占用时间的一部分，但这部分也是前端无法优化的地方，需要依赖 native 和 rn 的底层技术进行优化。

### rn 的底层优化

针对上下文创建的消耗，rn 当然也有一些优化手段。主要有

- 预加载
- 引擎复用

引擎预加载和业务场景息息相关，对于一些上下游的页面会有一定的要求，在加载当前页面的时候，如果下游页面是 RN 页面，那么会进行引擎的预加载，构建初始化的 JS 环境。

![](https://pic.imgdb.cn/item/64aeed4d1ddac507cc0039ef.jpg)

如上一个业务线上存在 A，B，C 三个页面，其中 C 是 RN 页面，那么当从 A 进入到 B 的时候，开始启动预加载，加载 C 页面的 Bundle，这样进入到 C 页面后，就不需要做初始化 JS 运行环境等操作，大幅度提高了页面的秒开率。
但是预加载的 JS 引擎不能一直存在，所以可以在 从 B -> A 的时候，回收引擎。还有一点需要注意的是，预加载的引擎需要在内存中保留一段时间后才会被回收，所以在进入一个页面中的时候，不要预加载很多页面，这样就会造成内存瞬间暴涨，容易引起 APP 闪退。

---

引擎复用，也是一种对页面初始化加载的优化手段，比如 A 进入 RN 的 B 页面，当 B 离开回到 A 的时候，B 的引擎并没有直接回收，而是被保存下来，当 A 再次进入到 B 的时候，直接服用引擎，这样当第二次进入 B 的时候，打开的速度非常快。

![](https://pic.imgdb.cn/item/64aeeda51ddac507cc00c63f.jpg)

引擎复用比较适合从列表页到详情页的场景，比如从商品列表到商品详情，用户可能多次从商品详情返回列表，然后再次进入商品详情。

## 新架构

https://medium.com/coox-tech/deep-dive-into-react-natives-new-architecture-fb67ae615ccd
https://juejin.cn/post/7063738658913779743

新架构带来的调整主要在于以下四点：

- JavaScript Interface(JSI)
- Fabric
- Turbo Modules
- CodeGen

### JSI

JSI 的主要功能有三个：

1. 替代原来的 bridge，实现 js 直接调用原生方法，反过来也一样，解决了之前通信的异步、数据量大的问题。（不需要序列化，不需要异步）
2. 对不同引擎做统一处理，比如在安卓和 ios 的引擎不同，JSI 可以让 js 侧不感知引擎类型，通过自己来选择合适的引擎进行编译
3. 定义了与 JS 侧对应的各种数据类型 (undefined, null, boolean, number, symbol, string, or object) 及 JS Value 与 Native Value 相互转化的方法，本质上也是方便和 native 端的通信。

JSI 的主要优势有

- 同步执行：现在可以同步执行那些本来就不应该是异步的函数。
- 并发：可以在 JavaScript 中调用在不同线程上执行的函数。
- 更低的开销：新架构不需要再对数据进行序列化/反序列化，因此可以避免序列化的开销。
- 代码共享：通过引入 C++，现在有可能抽象出所有与平台无关的代码，并在平台之间轻松共享它。
- 类型安全：为了确保 JS 可以正确调用 C++ 对象的方法，反之亦然。即 js 和 c++可以完成类型转换然后相互调用方法、传递参数，这个过程是安全的。

最大的功能其实就在于 js 侧可以直接调用原生方法。
具体是怎么做到的呢？其实可以类比一下 dom 的使用方式。 Web 里 JS 代码可以保存对任何 DOM 元素的引用，并在它上面调用方法：

```js
const container = document.createElement("div");
```

在这里的 container 会包含一些在 C++ 中初始化的 DOM 元素的引用，这时候如果我们调用 container 上的任何方法，它就会调用 DOM 元素上的方法。
JSI 就是以类似的方式运行，JSI 将允许 JS 代码保存对 Native Modules 的引用，并且 JS 可以直接通过引用去调用 Native 上的方法。

![](https://pic.imgdb.cn/item/64aef4881ddac507cc0c090f.jpg)

当 js 访问这个对象上的方法时，c++层可以通过类似代理的形式监听到 js 的访问，从而达到对 js 的感知。这种对象被称为 HostObject
HostObject 是 Native Module 在运行时注入到 JSI 中的一个对象，这是一个特殊的对象，并不像普通的 JS 对象一样可以随意的创建或者访问，这个流程需要 Native 的支持，才能在 JS 中创建和使用 HostObject

### Fabric

Fabric 是新的渲染系统，它将取代当前的 UI Manager。
在 Fabric 之前，当 App 运行时，React 会执行你的代码并在 JS 中创建一个 ReactElementTree ，基于这棵树渲染器会在 C++ 中创建一个 ReactShadowTree。

UI Manager 会使用 Shadow Tree 来计算 UI 元素的位置，而一旦 Layout 完成，Shadow Tree 就会被转换为由 Native Elements 组成的 HostViewTree（例如：RN 里的 View 元素会变成 Android 中的 ViewGroup 和 iOS 中的 UIView）。

![](https://pic.imgdb.cn/item/64af66381ddac507ccde4726.jpg)

而之前线程之间的通信都发生在 Bridge 上，这就意味着需要在传输和数据复制上耗费时间。

例如如果一个 ReactElementTree 节点恰好是一个 `<Image/>`，那么 ReactShadowTree 的节点也会是一个图像，但是这些数据必须被复制并分别存储在两个节点中。

另外由于 JS 和 UI 线程不同步，因此在某些情况下 App 可能会因为丢帧而显得卡顿（例如滚动有大量数据的 FlatList ）
而得益于前面的 JSI， JS 可以直接调用 Native 方法，其实就包括了 UI 方法，所以 JS 和 UI 线程可以同步执行从而提高列表、跳转、手势处理等的性能。
使用新的 Fabric 渲染，用户交互（如滚动、手势等）可以优先在主线程或 Native 线程中同步执行，而 API 请求等其他任务使用异步执行。

另外新的 Shadow Tree 将成为 immutable，它会在 JS 和 UI 线程之间共享，以两端进行直接交互。
在以前 RN 必须维护两个层次结构的 DOM 节点，但因为现在 Shadow Tree 可以共享，在减少内存消耗的部分也会得到相应的优化。

#### 新架构的渲染流程

rn 的 Fabric 同时也带来了新的渲染方式。由于 jsi 的存在，原来通过 bridge 难以实现的渲染方式现在也变得可以实现了。

rn 把渲染流程分为三大类

- 初次渲染
- react 状态更新重渲染
- 原生状态更新重渲染

每个渲染部分将会有类似的流程。
首先是初次渲染的流程，大致分为三个阶段：

1. **渲染**（Render）：可以理解为 react 执行组件，获得 fiber 树，然后在 shadow threaded 中创建一棵 shadow tree 并和 fiber tree 相连。

这一步主要是 js 侧的执行。当应用启动并开始执行 js 后，就会通过入口进入 js，执行 react 的 render 过程。
每调用一个 React 元素，渲染器同时会同步地创建 React 影子节点。但只会创建 HostComponent，符合组件不会创建。
最后创建出的 fiber tree，每一个 “fiber” 都代表一个宿主组件，存着一个 C++ 指针，指向 React 影子节点。这些都是因为有了 JSI 才有可能实现的。

另外，fiber tree 和 shadow tree 都是 immutable 的。也就是说如果 react 发生更新，并不是复制+修改创建一棵新的树，而是绝大部分复用旧的树。这个和在 web 端 diff 算法的复制其实不同，后者大部分时候是在复制元素。

![](https://pic.imgdb.cn/item/64af6f371ddac507ccfef19b.jpg)

2. **提交**（Commit）：shadow 创建完成之后，在 ui 线程（native）计算具体布局信息，然后提交成为`next tree`。

这些操作都是在后台线程中异步执行的。
为什么要在 native 端计算布局，是因为不同的端的情况不同，布局信息由每个端单独维护，因此也要在 native 侧进行计算。
最后的提交成为 next tree，和下面的 rendered tree 构成双缓冲树。这里类似 web 端的行为；不过初次渲染不存在 rendered tree。

![](https://pic.imgdb.cn/item/64af6f4c1ddac507ccff28dd.jpg)

3. **挂载**（Mount）：通过创建之后的 next tree 生成具体的 native 布局树，创建、修改具体的 native 元素，然后把 next tree 变为 rendered tree。
   挂载阶段由三个步骤组成：
   1. 树对比： 这个步骤完全用的是 C++ 计算的，会对比 rendered tree 和 next tree 之间的差异。然后对于变更，去生成 native 端的原子变更操作，比如 createView, updateView, removeView, deleteView 等等（还没有真正执行，只是生成操作队列）。这个过程类似 web 端 diff 算法执行后，在 commit 阶段执行具体的 dom 元素操作。在这个步骤中，还会将 React _影子树拍平_，来避免不必要的宿主视图创建。
   1. 树提升，Next Tree → Rendered Tree，即交换两棵树
   1. 视图挂载：具体执行对 native 的操作。

![](https://pic.imgdb.cn/item/64af6f5a1ddac507ccff4cf8.jpg)

---

然后是 react 更新导致的更新渲染。这一步其实和上面的初次渲染略有不同

- 更新的 fiber tree 和 shadow tree 不会重新创建，而是通过 immutable 数据结构来大量复用，只修改更新存在的路径的元素。
- 更新时存在 next tree 和 rendered tree，在 commit 阶段会对比两棵树，找到变更。
- 最后在挂载阶段，根据上一步的变更执行具体的 native 渲染操作。

#### 视图拍平

视图拍平（View Flattening）是 React Native 渲染器避免布局嵌套太深的优化手段。在 Fabric 渲染的挂载阶段，会通过拍平来减小 shadow tree 的嵌套层级，减少 native 元素的创建。

具体来说，拍平的结果对于那些只参与布局的节点（比如只有某个样式的 View），将其多层合并为一个节点
比如有这样一段代码：

```js
function MyComponent() {
  return (
    <View>                          // ReactAppComponent
      <View style={{margin: 10}} /> // ContainerComponent
        <View style={{margin: 10}}> // TitleComponent
          <Image {...} />
          <Text {...}>This is a title</Text>
        </View>
      </View>
    </View>
  );
}
```

这里三个 view 都是只参与布局的 view，创建 shadow tree 和 fiber tree 时存在，但没必要在 native 端都创建。因此后面两个 view 会被合并到第一个中，同时保留他们的样式。

![](https://pic.imgdb.cn/item/64af71651ddac507cc04f88a.jpg)

![](https://pic.imgdb.cn/item/64af716e1ddac507cc0514e9.jpg)

### Turbo Modules

在之前的架构中 JS 使用的所有 Native Modules（例如蓝牙、地理位置、文件存储等）都必须在应用程序打开之前进行初始化，这意味着即使用户不需要某些模块，但是它仍然必须在启动时进行初始化。
Turbo Modules 基本上是对这些旧的 Native 模块的增强，正如在前面介绍的那样，现在 JS 将能够持有这些模块的引用，所以 JS 代码可以仅在需要时才加载对应模块，这样可以将显着缩短 RN 应用的启动时间。

# 性能优化

rn 的性能优化是一个很大的话题，因为其并不像 web 端的 react，很多优化手段需要 native 的支持。并且诸如分包、预加载等方法也要比 web 端复杂的多。
而且如果优化仅在前端做，其实能优化的点比较有限。通过一些监控和测量手段有时候也未必能看到明显的提升。
总之就是比较玄学，及时了解方案，也不一定能了解原理。在项目中应用的话，只能说是尽可能理解吧。

rn 性能优化主要有几个大的方面

- 首屏优化，即首屏加载时间。一般指的是页面第一次打开的时间，但也要考虑到后续打开的情况。类比指标可以有 FCP、FMP 等
- 帧数优化，如果页面上有动画或者连续的交互，那么帧数要保持稳定，不能出现卡顿、闪烁等情况
- 响应优化，即用户交互到做出响应的时间优化
- 专项优化，比如列表、轮播图等功能性的优化
- 其他优化

## 首屏优化

### 指标和测量方式

```
首屏耗时 ≈ js bundle 资源下载及解压耗时 + RN 视图创建耗时 + RN 资源加载耗时 + js bridge 及应用启动耗时 + 首屏视图渲染耗时。
```

由于 RN 视图创建耗时，RN 资源加载耗时，js bridge 及应用启动耗时为 react native 上下文初始化的一个固定开销，所以我们可以将其统称为 react native 上下文初始化时间。所以最终我们暂且使用：

```
首屏耗时 ≈ js bundle 资源下载及解压耗时 + react native 上下文初始化时间 + 首屏视图渲染耗时
```

首屏渲染的指标应该主要参考几项

![](https://pic.imgdb.cn/item/64afb7031ddac507cc0bf7a3.jpg)

- FCP 或 FMP，用户看到具体内容的时间（非骨架屏和 loading 页面）
- LCP，也是一种指标。至于到底是 FP、FCP、FMP 还是 LCP，主要还是取决于不同需求。
- TTI，用户可交互的延迟，一般是从 FMP 到可交互的时间计算。

这些渲染指标其实都是从 web 端沿用的指标。在 web 端有 performance 这样的 api，以及 lightHouse 这样的性能监测工具。在 rn 中则需要借助 native 的参与，来更好测算渲染时间。
举个例子，比如说要测算 LCP。LCP 是 Web 的标准，在 RN 中并没有实现，应该怎么实现呢？

整体上讲，实现上大致分为 5 个步骤：

- 在用户进入时，由 Native 线程记录 Start 时间戳。
- 由 Native 线程将 Start 时间戳注入到 JS Context 中。
- 由 JS 线程，监听页面中渲染元素的布局事件。
- 由 JS 线程，在页面渲染过程中进行计算，并不断更新 LCP 值。
- 由 JS 线程，计算得到 End 时间戳，并上报最终的 LCP 值。

此时，最终上报的 LCP = End Time - Start Time。

其中难点是怎么收敛 LCP，也就是如何判断可视区完全加载。
可以采用的规则是，当所有元素都加载完成，且底部的元素也已经加载完成时，可视区加载完成。元素有一个调用周期，先调用 render，再调用 layout。只调用了 render 的元素，是没有加载完成的元素。调用了 render 且调用了 layout 的元素，是加载完成的元素。能够判断一个元素是否加载完成了，也就能够判断可视区是否加载完成了。

![](https://pic.imgdb.cn/item/64afb8131ddac507cc114840.jpg)

---

对于 react 的单个组件，渲染时长可以通过生命周期或 hooks 来测算。如果放在整个 App 组件上去监控，那么就是整个页面的渲染时间。

> 由于 react 的特性，组件的 componentDidUpdate 或 componentDidMount 会在该组件的所有子组件执行完成之后才会执行。因此，作为根组件的 componentDidUpdate 执行时，所有的子组件的更新都已经完成，也就表示 js 侧的渲染完成。

```
渲染耗时 = 接口数据返回并更新完成的时间(componentDidUpdate) - 初次进入组件的时间(componentWillMount)
```

渲染耗时的计算，则可以通过 native 记录的方式。好处是可以调节至 release 模式下进行测算，更加真实。
比如，Native 侧统计手机耗时时间：

```java
public class LogModule extends ReactContextBaseJavaModule {
    // ...一些方法
    @ReactMethod
    public void log(String type) {
        System.out.println("type = " + type);
        if (RNDynamicActivity.bundleType == "static") {
            switch (type) {
                case "mounted":
                    TimeRecord.mStaticLog.componentMountedTime = new Date().getTime();
                    break;
                case "render":
                    TimeRecord.mStaticLog.startRenderTime = new Date().getTime();
                    break;
                case "constructor":
                    TimeRecord.mStaticLog.initTime = new Date().getTime();
                    break;
                case "updated":
                    TimeRecord.mStaticLog.updatedTime = new Date().getTime();
                    break;
            }
        } else {
            //...动态化时间记录
        }
    }
    @ReactMethod
    public void showDynamicTimes(Callback callback) {
        boolean isDynamic = RNDynamicActivity.bundleType.equals("dynamic");
        HashMap<String, Long> hashMap = new HashMap<>();
        //...一些时间统计
        callback.invoke(new Gson().toJson(hashMap));
    }
}
```

React Native 侧触发：

```js
class App extends Component {
  constructor(props) {
    super(props);
    Log.log("constructor");
  }

  componentDidMount() {
    this.fetchData();
    Log.log("mounted");
  }
  componentDidUpdate() {
    Log.log("updated");
  }
}
```

### 优化方式

优化的核心是我们可控的视图渲染耗时，可分为几个部分

- 初次渲染。参考上面的渲染流程，这一步是第一次执行 bundle 时，react 渲染、shadow 渲染到 native 渲染的过程，直到显示出元素为止（一般可能是骨架屏）。
- 网络请求。一般来说请求到来之前都会显示骨架屏，因此请求时间也会影响可交互时间以及完整的首屏渲染时间。
- 首次更新。当网络请求完成后，则会再次更新一次，这时就会显示出具体的元素，并且一般进入可交互阶段。
- 资源加载。比如图片、视频等非阻塞资源会在最后加载完成。到这一步，完整的首屏加载才完成。

首屏优化主要有以下一些方式。

1. 缓存。通过 AsyncStorage 等工具，将上一次请求的数据缓存起来，每次进入页面时先取缓存中的数据，然后同时发起请求。当请求到达时，和当前数据作比较，如果有不同再去更新。这种方法可以让首屏渲染跳过网络请求的消耗时间，但是如果页面是实时性比较强的就不太合适。并且当页面加载完毕之后再次刷新数据也有可能影响用户体验。

缓存还有一种更高级的方案，即页面直出，其实就是在页面 bundle 加载时就进行网络请求，然后将数据和页面一起展示出来。

2. 渲染控制。渲染控制是通过异步渲染、延迟渲染、分级渲染等方式，使某些不重要、不在首屏的元素比首屏元素更晚渲染。或者是通过占位等形式先放上去一部分视图，再延迟渲染一些消耗较大的组件。

- 异步渲染/延迟渲染：通过 setTimeout 等形式，让一些组件在某些组件完成渲染之后再去渲染。比如一个长页面下有不同的模块（🌰，商详页，上面是商品详情，下面是其他推荐，可以把可视区域内按优先级渲染，区域外按延迟渲染或懒渲染），部分模块不在首屏显示，就可以将其放在主要模块渲染完成之后再执行渲染。
- 分级渲染/分批渲染：通过类似 Priority 这样的机制，让元素按照一定优先级进行渲染。这种不同于异步渲染，渲染的过程本质是同步的，当一个元素渲染完成后会立即调用下一优先级的元素渲染。而且一般不需要手动控制渲染。这种一般用于页面的主要模块和次要模块之间的分批次。（🌰，页面主要的是列表，其他元素都可以放较低优先级。🌰，或者比如多级 tab，按照优先级顺序依次渲染多个 tab 栏）
- 懒加载或渲染：类似虚拟列表的效果，当某个区域滑动到可视区域时才渲染。这种方式虽然最省性能，但是有可能导致用户快速滑动时，对应的模块才开始渲染，就会感觉到卡顿

3. bundle 优化。bundle 大小降低、不需要的页面分包，这样的方式和渲染无关，而是加快 native 下载、执行包的时间。

优化包体积可以从 tree-shaking、删除一些库等方面入手；而分包则是比较深奥的东西。
一般来说，只针对少部分库进行分包是没有必要的，过于零碎的包在下载时也容易造成过大的服务器消耗。分包的对象一般是页面级的，多个页面之间要进行必须的分包，首次只加载首屏的页面，其他页面可以执行懒加载。

4. 预加载、预请求。一般是上级页面进行预处理，并且要控制预加载包的数量、时机。

预请求其实是一种比较通用的优化方式，除了对静态资源的优化之外，一些主要接口也可以进行预请求。也可配合缓存实现数据预取的效果
预加载则通常指的是整个加载的过程，包括容器加载、引擎加载、bundle 下载、bundle 执行，甚至初次渲染等步骤。预加载涉及到比较大的开销，因此最好由开发者手动控制预加载哪些部分。比如可以根据埋点数据得到用户经常访问的下一个页面，然后在当前页面加载完后预加载下一个页面。
预加载这一块的水很深，包括引擎复用、回收等操作。不过这些通常是由 native 底层控制。

## 帧数优化

帧数优化的主要方式有

- 阻止不必要的渲染。主要可能产生不必要渲染的地方有
  - redux 的 state，一些频繁更新的数据通过 redux 传递，可能导致其他无关组件被连带更新。通过改用 pubsub 等方式传递数据，而不是利用 redux 仅做状态共享
  - 没有使用 memo、pureComponent 等方式，或没有生效，导致父组件更新时，大量子组件被连带更新
  - 本该使用 Animated 动画的地方由 state 控制实现
- 降低渲染消耗
  - 降低 state 影响的层级，把一些需要频繁更新的组件提取出去，不要影响父组件
- 针对列表、动画等做专项优化

关于重复渲染的测量工具，可以使用 why-did-you-render 来检查。

## 专项优化

长列表优化其实在上面已经说过，关于 FlatList 的优化方式，主要是通过传递恰当的 api 来保证 rn 自己能形成足够的优化。

其他特殊组件的优化，则和项目本身息息相关。

## 其他优化

1. 内存优化。对于页面的视频、gif、高负荷动画等，当离开页面时，需要将其置空或销毁，否则将造成很大的内存消耗。

比如当 A 页面中有视频播放的模块，而 B 页面是 A 的二级页面，在融合模式下，进入 A 页面之后会开始播放视频流，但是当从 A 页面进入到 B 页面之后，本质上 A 页面并没有被回收，但是这个时候，还在加载着视频资源。那么这样下去，会让内存越来越大。
那么如何解决这个问题呢？ 当 A 跳转到 B 页面之后，应该停止 A 页面加载资源，或者清空视频资源，让内存维护一个健康的水平。
对于一些超多 gif 图片的页面，并还有列表加载功能，这样在向下加载数据的过程中，会渲染更多的 gif 组件，这样就会让内存越来越大，并且不容易下来，或者一些低端的机型，根本无法渲染太多的 gif 图片，那么此时应该如何解决呢？
这个时候可以做一个优化，就是只有在视图范围内的元素才渲染真正的 gif 图片，而其他看不见的直接渲染图片或者是占位图。

2. 图片优化。图片优化的方面有很多，包括但不仅限于

- 图片大小，为了同时兼顾图片的清晰和大小，应该选择合适的图片大小和分辨率，不能过大也不能过小（指的是体积而不是尺寸）
- 预加载图片，对其他静态资源也是类似的
- 占位图。对于背景图，可以采用纯色占位+图片覆盖的形式，防止图片突然出现导致的突兀情况。
- 懒加载。图片懒加载可以说是最常见的情况了，可以通过封装一个包含懒加载功能的 Image 组件实现图片的懒加载。

3. 网络请求优化。

- 预请求
- 页面带参：重要参数可以通过上游⻚⾯带过来，比如拿一些基础信息和头图部分信息带到详情⻚，使得头图和基础信息模块的数据可以第⼀时间得以展示，加快了⻚⾯⾸屏展示速度。
- 请求前置：将请求时机前置到Native⻚⾯初始化时，和MRN框架创建、加载JS等操作并⾏执⾏。比如直接让native侧执行请求，而不需等待js端的执行
- 请求分级：降低请求并发，或通过聚合的形式减少发请求的次数

4. 包体优化

- tree-shaking，可以参考webpack章节对tree-shaking的讲解，在rn中其实是类似的，大致都是减少副作用、规范代码、设置babel等。这里有一些tree-shaking的参考文章：
https://zhuanlan.zhihu.com/p/32831172
https://juejin.cn/post/6844903544756109319
https://juejin.cn/post/6844903687412776974
- 删除lodash、moment.js等非常庞大的js库。只使用其中的小功能，可以自己造轮子，或者采用内部库的一些精简过的功能库
- 对于icon使用的svg、image，要注意不要全量引入庞大的svg库，导入大量无用的svg文件等。其他资源类似



## 优化工具

### why-did-you-render

wdyr在rn中的使用可以参考：https://yajanarao.medium.com/how-to-use-why-did-you-render-library-in-react-native-a95121978a75

wdyr可以检测执行的组件的渲染情况，在控制台打印出渲染原因。

![](https://pic.imgdb.cn/item/64afde8f1ddac507cca1dfbf.jpg)

下面是一个codesandbox例子：https://codesandbox.io/s/why-did-you-render-sandbox-forked-q73lpx

wdyr还可以用来检查redux的useSelector，具体可参考官方文档https://github.com/welldone-software/why-did-you-render

### Profiler

使用真机和电脑链接后，可以通过chrome的开发者工具进行调试，其中也包括chrome的profiler。

和web端的使用方式类似，profiler可以检查各个线程的执行时间，并以火山图的形式展示，便于查看问题所在。



### RN debugger

RN debugger提供的React⾯板也有一些功能。
比如，它可以用于显示组件重复渲染次数，便于发现有问题的组件。

![](https://pic.imgdb.cn/item/64affa4c1ddac507cc15eddb.jpg)

用它可以和wdyr配合，找到准确的重复渲染元素。

RN debugger还有的其他功能有

- js函数调用次数监测
- 显示页面的性能指标时间，比如FCP等指标
- 监测JS与native通信情况
