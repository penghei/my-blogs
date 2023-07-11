---
title: React Native 学习
date: 2023-05-17 14:58:28
tags: 日常学习
categories: React
---


# 基本使用

rn和react有一点不太相同的是，rn更像是一个完善的框架，其内部包含了大量的内置组件，这些组件通常含有很多props，因此学习rn的一个关键点就是搞清这些组件的使用方式，以及他们的内部实现、区别等。

## 交互

### 导航

rn使用的导航是react-navigation开源库，本质上是一个类似react router的路由功能，通过堆栈记录的方式来完成页面的跳转、数据的传递等。
rnav实现的导航和react router类似，实际上是应用内部的页面切换，比如很多应用的底部tab切换等。如果要在不同的rn打包出的页面间跳转，就不能使用rnav了。

> A key difference between how this works in a web browser and in React Navigation is that React Navigation's native stack navigator provides the gestures and animations that you would expect on Android and iOS when navigating between routes in the stack.

正如官方文档所说，rnav就是在模拟web端的路由实现，在基础上还增加了适用于不同平台的跳转动画、手势识别等。
在使用方面，rnav和react router也很相似，通过Stack.Navigator包裹Stack.Screen，即可实现不同页面的切换。

在rnav中有一个特点，即用户离开上一个页面时，该页面对应的页面组件不会被卸载；当用户返回该页面时，也不会再次触发渲染。这点和在web端是不同的。

> 具有屏幕 A 和 B 的堆栈导航器。导航到 A 后，调用其 componentDidMount。当压入 B 时，它的 componentDidMount 也会被调用，但 A 仍然挂载在堆栈上，因此不会调用它的 componentWillUnmount 。

可以通过rnav提供的hooks或事件监听来得到用户当前是否进入或离开页面。

### 动画

rn的动画有两个类别：

- Animated: 声明式动画。即，rn通过预先声明一个动画的值、时间等，将其传递给特定组件的style属性。然后在恰当的时机内通过调用start、stop等方法来开始、结束动画。

举个例子：

```js
import React, { useRef, useEffect } from 'react';
import { Animated, Text, View } from 'react-native';

const FadeInView = (props) => {
  const fadeAnim = useRef(new Animated.Value(0)).current  // 透明度初始值设为0

  React.useEffect(() => {
    Animated.timing(                  // 随时间变化而执行动画
      fadeAnim,                       // 动画中的变量值
      {
        toValue: 1,                   // 透明度最终变为1，即完全不透明
        duration: 10000,              // 让动画持续一段时间
      }
    ).start();                        // 开始执行动画
  }, [fadeAnim])

  return (
    <Animated.View                 // 使用专门的可动画化的View组件
      style={{
        ...props.style,
        opacity: fadeAnim,         // 将透明度绑定到动画变量值
      }}
    >
      {props.children}
    </Animated.View>
  );
}
```

Animated.View就是专门执行动画的组件，类似的还有Animated.Text、Image等等。
Animated动画通常有着比修改state实现的动画更好的性能。一般来说动画的实现可能是通过rAF等js代码，不断通过bridge将动画数据发送给原生端进行渲染，并不会直接修改动画组件的state。

如果在timing函数传入的参数中加一个属性，就可以启用原生动画驱动。通过下发给原生的形式，让动画在native上执行，从而大大提升动画的性能，避免和js线程的冲突。

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
    bottom: 0
}

// part 2: 
// 此处假设我们在某个View的style中，使用了this.state.bottom这个变量作为bottom的值
LayoutAnimation.spring();
this.setState({ bottom: 20 }); // 只要改变state，就可以生效动画
```

spring方法是LayoutAnimation的一个预设，其他预设还有linear、easeout等，和通常的动画速度曲线一致。
当然也可自定义动画，可以参考[官方文档](https://reactnative.dev/docs/animations#layoutanimation-api)




## 组件

### 核心组件

核心组件列表：https://reactnative.cn/docs/components-and-apis

rn中的基础组件主要指View、Text、TextInput这些没有经过过多封装的组件，使用方式比较基础。
除了基础组件之外，还有诸如Button这样的交互组件、FlatList这样的列表组件，以及Modal这样的功能组件。这些组件统称核心组件，指的是rn默认提供的一些组件。

### 列表组件

React Native 有好几个列表组件，先简单介绍一下：

- ScrollView：会把视图里的所有 View 渲染，直接对接 Native 的滚动列表，也不会复用，就是最基本的渲染。当列表数据非常多时，view会等到所有项都渲染完成再显示
- VirtualizedList：虚拟列表核心文件，使用 ScrollView，长列表优化配置项主要是控制它
- FlatList：使用 VirtualizedList，实现了一行多列的功能，大部分功能都是 VirtualizedList 提供的
- SectionList：使用 VirtualizedList，底层使用 VirtualizedSectionList，把二维数据转为一维数据

后三者其实都是虚拟列表，师出同源。rn的虚拟列表实现方式和简单原理可以参考https://www.cnblogs.com/skychx/p/react-native-flatlist.html。
这篇文章有对源码的简单解析：https://shengshuqiang.github.io/2019/07/17/%E8%BF%9B%E5%87%BBReactNative-FlatList%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.html

#### rn中的虚拟列表

基本结构：都采用下图这种方式：

![](https://pic.imgdb.cn/item/649172e11ddac507cc145937.jpg)

FlatList、VirtualizedList、SectionList的基本实现原理都相同，关键点有以下几个：

1. 只渲染可视区域。这是最基本的内容，VirtualizedList会通过异步计算的方式确定当前渲染的位置、需要渲染的元素，然后只渲染这部分内容。而之前的列表项则会被选择性的回收复用或销毁以减小内存，并使用上下空白区域去填充空间。并且可视区域的列表项也并不只是在可视区域内的大小，还会超出一些部分。

2. 复用和回收。虚拟列表的复用过程与React的diff算法类似。在React中，diff算法通过比较元素的key和type来决定是否复用之前的虚拟DOM节点，从而减少对实际DOM的操作，提高性能。同样地，在虚拟列表中，组件会根据列表项的key属性来判断是否复用之前的列表项组件。

当用户滚动虚拟列表时，组件会根据可见区域计算出需要渲染的列表项，并根据这些列表项的key属性来判断是否复用之前的列表项组件。如果列表项的key属性稳定且类型相同，组件会将之前渲染过的列表项组件进行复用，只更新列表项的数据，而不重新创建新的组件实例。这样可以避免不必要的渲染和DOM操作，提高性能和内存效率。

虚拟列表的复用过程是一种优化策略，通过减少组件的创建和销毁操作，可以有效降低CPU消耗，提高列表的滚动流畅度。它与React的diff算法相互补充，共同提供了高效的列表渲染和更新机制。

3. 异步计算和渲染。VirtualizedList通过异步方式加载和渲染新的列表项，以避免阻塞主线程和用户操作。异步加载的过程可以分为两个阶段：计算和渲染。

在用户滚动到未渲染的列表项范围时，VirtualizedList会触发异步计算的过程。在这个过程中，组件会根据当前滚动位置和可见区域的变化，计算出需要加载的新列表项的数量和位置。这个计算过程是异步的，它会在后台线程中进行，不会阻塞主线程的运行。
在计算完成后，VirtualizedList会触发异步渲染的过程。在这个过程中，组件会根据计算得到的新列表项的数量和位置，异步地加载数据并渲染新的列表项。这个渲染过程也是异步的，它会在后台线程中进行，并在渲染完成后将新的列表项渲染到屏幕上。
同时渲染和计算的过程都会在用户操作和动画执行之后，尽可能减少cpu的抢占而造成的阻塞。


4. 渲染优先级。在FlatList中，列表项的渲染优先级是根据滚动速度和方向来动态调整的。当用户进行快速滚动时，FlatList会根据当前的滚动情况，优先加载和渲染可见区域附近的列表项，以保持滚动的流畅性。这个优化策略可以减少不必要的渲染和资源消耗，提高性能。

具体而言，FlatList会根据以下几个因素来决定列表项的渲染优先级：
- 滚动速度：如果用户进行快速滚动，FlatList会尽量快速地加载和渲染新的列表项，以避免出现白屏。在这种情况下，FlatList可能会优先加载和渲染离可见区域较远的列表项，以跟上滚动的速度。
- 滚动方向：当用户滚动到一个新的位置时，FlatList会优先加载和渲染滚动方向上的列表项。例如，如果用户向下滚动，FlatList会优先加载和渲染下方的列表项。
- 可见区域：FlatList会根据可见区域的大小和位置来确定哪些列表项是可见的。只有可见区域内的列表项才会被加载和渲染，而不可见区域的列表项会被延迟加载。这样可以减少不必要的渲染和资源消耗。



#### 列表优化

对于不同的rn列表组件，优化方式也不尽相同。以常用的FlatList为例，优化的大方向有两个：
1. 通过特殊的props进行优化，比如getItemLayout等
2. 通过对数据的处理或其他列表本身之外的方式进行优化

优化方式在官方文档已经讲的很清楚了：https://reactnative.cn/docs/optimizing-flatlist-configuration

#### FlatList

FlatList除了包含虚拟列表的基本功能之外，还有一些其他组件功能，常用于native开发中，比如：

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

FlatList有几个比较重要的使用要点：

1. 当某行滑出渲染区域之外后，其内部状态将不会保留。也就是说对于每个ListItem组件来说，滑出渲染区域之后组件将会被卸载或重置复用，其上的状态不会保留；

比如一个功能是修改列表的某一项，如果只是修改了对应的ListItem组件内部的state或props则不能保留，除非修改的是data源数据。

2. 组件继承自PureComponent，如果其props在浅比较中是相等的，则不会重新渲染。所以请先检查你的renderItem函数所依赖的props数据（包括data属性以及可能用到的父组件的 state），如果是一个引用类型（Object 或者数组都是引用类型），则需要先修改其引用地址（比如先复制到一个新的 Object 或者数组中），然后再修改其值，否则界面很可能不会刷新。

上面这一段是官方文档的内容。其实大概意思就是更新FlatList的data属性时需要修改索引，否则不会更新。也就是说即使你通过setState修改了data，但data如果没有被重新创建，就还是不会更新。

举个例子：

下面这样修改state是不会让FlatList更新的：
```js
data.push({id: 'xxx', name: 'ttt' });
setData(data);
```

这样是可以的：

```js
data.push({id: 'xxx', name: 'ttt' });
setData([...data]);
```

> 当然这种方式容易造成内存的浪费，如果频繁增加元素，那么每次data都需要重新创建。为了优化这个问题，我们可以采用两个数组交替复制的方式。这是一个小技巧，可以参考https://juejin.cn/post/7201425436835151930

除了上面这种情况之外，如果列表还依赖于其他state，还可以使用extraData属性来控制FlatList更新。当然extraData也和data一样，也有浅比较的检查。

我们可以修改extraData，来让FlatList强制更新。比如：

```js
const App = () => {
  const [data, setData] = useState(['Item 1', 'Item 2', 'Item 3']);
  const [extraData, setExtraData] = useState(false);

  const updateData = () => {
    setData(['Item 1', 'Item 2', 'Item 3', 'Item 4']);
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

当然这种方式不是很好。大多数情况下，extraData主要用于处理额外依赖数据的情况。


3. 为了优化内存占用同时保持滑动的流畅，列表内容会在屏幕外异步绘制。这意味着如果用户滑动的速度超过渲染的速度，则会先看到空白的内容。这是为了优化不得不作出的妥协，你可以根据自己的需求调整相应的参数，而我们也在设法持续改进。

这也是上面介绍虚拟列表时提到的，可能的白屏情况。FlatList提供了很多可能的调优属性，比如

- getItemLayout：返回列表元素的偏移、高度等信息，避免动态测量内容尺寸的开销。如果不提供，FlatList需要通过自行测量的方式得知每个元素的偏移量等属性，就会导致每次计算开销增大。（但是如果每个元素高度不确定这个属性就不能用）
- keyExtractor：用于为给定的 item 生成一个不重复的 key。如果没有这个属性，默认抽取item.key作为 key 值。若item.key也不存在，则使用数组下标。因此最好提供好key值，key在react的重要性不言而喻。
- maxToRenderPerBatch：每次批处理更新时更新多少项目，或者说在渲染时每批最多渲染多少个列表项。默认情况下，FlatList会将更新操作批处理成一批，以减少重绘的次数，从而提高性能。这个和下面的属性都是用于控制批处理过程的。
- updateCellsBatchingPeriod：用于设置批处理的时间间隔（以毫秒为单位），如果在这个时间间隔内有多个更新操作，则这些更新操作会被批处理成一批。如果设置为0，则表示禁用批处理，每个更新操作都会立即重绘。**如果不设置updateCellsBatchingPeriod属性，则默认禁用批处理。**

> 批处理，可以让FlatList能够在一段时间内更新一组Cell，而不是一次更新所有Cell。
> 批处理和虚拟列表的渲染无关，它主要指的是当FlatList需要更新时如何进行分批更新，而非一次性更新全部项。
> 比如在FlatList中进行一系列更新操作，例如添加、删除、移动或更新列表项。这些更新操作将被添加到批处理队列中，而不会立即执行重绘操作。
> FlatList还会等待updateCellsBatchingPeriod属性所设置的时间间隔。在这个时间间隔内，如果有其他更新操作，则这些更新操作也将被添加到批处理队列中。
> 最后当时间间隔到达后，FlatList将执行批处理操作，将批处理队列中的所有更新操作合并成一个单独的操作，并执行重绘操作。这样，就可以减少重绘的次数，从而提高性能。
> 这个过程类似react的批量更新，大致思路是相同的。

4. 默认情况下每行都需要提供一个不重复的 key 属性。你也可以提供一个keyExtractor函数来生成 key。


# 简单原理

rn的原理部分，除开react和native端的实现，核心部分其实就是两个：
- 渲染：即如何从jsx渲染成为native的元素
- 通信：即js和native语言之间是怎么交互的，怎么样传递信息、互相调用；

参考：https://km.sankuai.com/page/337425524
https://juejin.cn/post/6844903442079563784
http://blog.cnbang.net/tech/2698/
https://km.sankuai.com/page/883771103

## 渲染

### 渲染树

在 React Native 的世界里，总共存在三种树，他们分别是 

- Fiber 树
- Shadow 树
- Android 原生 UI 树

> React Native 维护和更新开发者创建的 Component 的状态，创建出表达 UI 的第一种数据结构——Fiber 树。然后通过 Fiber 算法高效地执行树的更新动作，输出的结果表现为原生侧的 Shadow 树数据结构，再经过一些最终调整后，创建出原生平台上的 UI 结构——Android View 或 iOS UIView 所构成的树结构，由原生平台的渲染机制来完成屏幕内容的渲染。

其中，fiber树很熟悉就不再赘述，而shadow树则是原生渲染过程中的一个重要结构，作为最终渲染之前的一步。

大致的流程为：

- JavaScript 层通过维护 Fiber 树，计算出数据结构变化的副作用，生成结点更新指令（createView / updateView / setChildren 等）送往 Java 层
- 根据上面的节点更新指令，实体结构 shadow tree 在 Java 层 UIImplementation 中生成。
- UIImplementation 的下一级 NativeViewHierarchyOptimizer 类，负责将 shadow tree 转译为真正的原生 view 树。

shadow tree 和原生 view 树有何不同呢？后者去掉了前者结构之中多余的 “纯布局 view”（layout-only view，即只影响 layout 但自身不绘制任何东西的 view）。
简单理解就是原生view树去掉了一些不进行绘制的部分，就像浏览器中的dom树和layout树之间的差别一样。

### 渲染过程

rn和react的渲染核心区别就在于，react会把虚拟dom结构利用dom api转化成真实dom结构，而rn则是通过一个 API 去创建 Native 的 View，再由native进行原生的渲染操作。
React Native 使用的全部是自定义标签，和 Native View 一一对应，所以需要通过一个 API 去创建 Native 的 View，这个 API 就是 UIManager，主要执行从 Virtual DOM -> RenderDOM 节点的 CRUD 操作

- UIManager.createView：创建节点
- UIManager.measure / measureXXX：获取节点布局信息
- UIManager.findSubView
- UIManager.updateView：更新节点信息
- UIManager.manageChildren / setChildren：设置树结构信息，批量删除、增加、移动节点等

UIManager.js 是什么？它是从 NativeModules.js 中导出的，NativeModules.js 通过 Native 设置的 NativeModuleRegistry 的信息生成名字和实际类型的映射表，NativeModules 本身也是类似 Java 代理的风格去生成的，最终 UIManager.createView 这样一个函数调用，会变成消息队列的一个命令调用

```js
BatchedBridge.enqueueNativeCall(
  moduleID,
  methodID,
  args,
  onFail,
  onSuccess,
);
```

对于每个要渲染的虚拟dom元素，rn会像传递其他消息一样传递一个包含moduleId、methodId等参数的数据，然后通过通信的方式传递给native侧。比如一个`<Text>`jsx元素可能会变成这样的参数

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

最后这个数据会到达native侧，再由native侧的代码执行具体的渲染步骤。

具体的渲染过程可以参考：https://km.sankuai.com/page/337425524#id-3.%E6%8E%92%E7%89%88%E6%B8%B2%E6%9F%93

## 通信

这里的通信指的是js和native语言之间的交互、通信，具体来说其实是两者的互相调用。在ios和安卓端的native语言有所不同，前者是objective-c，后者是java。在具体实现上有所不同，但在大致原理上是近似的。

### js调用native

js是一个不能自执行的语言，意思就是js必需一个执行它的“引擎”。在ios上是JavaScript Core，在安卓上则有专门执行js的c++部分。
因此js调用native的过程，本质上可以理解为是引擎执行js，然后将js的执行结果或在js中的某些操作传递给native的过程。

以ios上的objective-c为例，React Native 解决这个问题的方案是在 Objective-C 和 JavaScript 两端都保存了一份配置表，里面标记了所有 Objective-C 暴露给 JavaScript 的模块和方法。这样，无论是哪一方调用另一方的方法，实际上传递的数据只有 ModuleId、MethodId 和 Arguments 这三个元素，它们分别表示类、方法和方法参数，当 Objective-C 接收到这三个值后，就可以通过 runtime 唯一确定要调用的是哪个函数，然后调用这个函数。

当js生成包含ModuleId、MethodId 和 Arguments的对象后，会放入到 MessageQueue 中，等待 Objective-C 主动拿走，或者超时后主动发送给 Objective-C。

---

另外，js不会主动将数据发送给oc。也就是说必须当oc在某些条件下执行js时，才会执行上面步骤，而不是由js代码主动驱动数据的发送。JS不会主动传递数据给OC，在调OC方法时，会把ModuleID,MethodID等数据加到一个队列里，等OC过来调JS的任意方法时，再把这个队列返回给OC，此时OC再执行这个队列里要调用的方法。

这样的执行方式就类似于事件触发。native开发里只在有事件触发的时候才会执行代码，这个事件可以是启动事件，触摸事件，timer事件，系统事件，回调事件。而在React Native里，这些事件发生时OC都会调用JS相应的模块方法去处理，处理完这些事件后再执行JS想让OC执行的方法，而没有事件发生的时候，是不会执行任何代码的。

### native调用js

对于 Objective-C 来说，执行完 JavaScript 代码再执行 Objective-C 回调毫无难度，难点依然在于 JavaScript 代码调用 Objective-C 之后，如何在 Objective-C 的代码中，回调执行 JavaScript  代码。
目前 React Native 的做法是：在 JavaScript 调用 Objective-C 代码时，注册要回调的 Block，并且把 BlockId 作为参数发送给 Objective-C，Objective-C 收到参数时会创建 Block，调用完 Objective-C 函数后就会执行这个刚刚创建的 Block。
Objective-C 会向 Block 中传入参数和 BlockId，然后在 Block 内部调用 JavaScript 的方法，随后 JavaScript 查找到当时注册的 Block 并执行。

本质上native调用js就是一个回调的过程，即js预先把自己的回调也传给oc，然后oc只需要执行这个block即可。


![](https://pic2.imgdb.cn/item/6464bee50d2dde5777c3dfd9.jpg)




