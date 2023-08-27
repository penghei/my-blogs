组件设计和组件库设计

要注意哪些内容呢？

- 组件设计：组件本身的设计，封装业务组件和封装组件库组件。组件库内的单个组件设计要注意什么，比如样式、属性、api、方法，以及如何封装逻辑、复杂组件拆分等等
- 组件库设计：组件整体的打包和整合方式，不同框架组件，甚至跨端组件的设计、打包方式，以及文档生成等其他内容

# 基本组件设计

## 基本思想

组件设计的基本思想其实和设计模式里的很多是重合的，比如单一职责、开闭原则等。总结下来有主要的几个：

1. 单一职责。

单一职责指的是，**只负责一个功能，并且只有一个改变状态的理由**。
组件的复杂度的一个重要参考就是这个组件内部自己维护的、非外部状态的数量。因此对于这个组件需要维护的状态和修改这个状态的逻辑，我们单独将其维护在组件内，而可能会影响其他重渲染或产生其他不必要的状态的内容，就应该拆分出去。

拆分的原则应该是可复用，即，被拆分出去的组件应该是可以被很好地复用的。如果不能复用那就只需要在组件内部维护就行。
比如我们需要在某个组件内添加一个 Button，那么这个 Button 显然应该单独提供，而不是聚合在这个组件本身，因为 Button 可以在很多地方被复用。
但比如有个评分的组件，内部的每个星星，这些元素虽然需要抽成单个组件来方便列表渲染，但是这些组件在其他组件内不能再复用，因此不用单独抽出来，整合在评分组件本身就可以。

还有一方面的单一职责是单个组件功能本身的设计。比如一个上传组件，可能包含上传组件的网络请求逻辑和 ui 部分。这两部分就应该做拆分。
以 arco-design 为例，upload 组件单独分离了一个 Uploader 组件，这个组件内部封装了 input 元素，然后包含了文件处理、上传操作、终止、重新上传、错误处理等逻辑。其他的组件，比如拖拽上传、文件列表、图片预览等功能则是单独用其他组件。最后将这些组件整合在 Upload 组件内部，通过条件渲染控制即可。
这个过程将上传逻辑和其他 ui 分离开，虽然 Uploader 本身不需要复用，但是这种方式便于调试和解构，也便于扩展。

2. 通用性、扩展性。

举个栗子，一个 Counter 组件，点击两边的按键来对数字进行加减。这个组件通常需要的 props 可能就是 count 状态、增加函数、减少函数等
但是如果有个需求是，希望把增加和减少的按钮替换为别的图标，或者在鼠标悬浮时增加一个 ToolTip，那应该怎么做？
这种情况下，如果需要实现功能就得修改组件内部的代码，其实是破坏了开闭原则。

我们可以让组件接收一个 renderProps 或者插槽，让用户自定义按钮。只提供最基础的 DOM、交互逻辑，将 DOM 的结构转移给开发者。

结合上面的单一职责，其实这两项都是在讲如何去分离和复用代码。在分离的时候也应该有着相应的考量，比如：

- 存在代码重复吗？如果只使用一次，或者只是某个特定用例，可能嵌入组件中更好。
- 如果它只是几行代码，分隔它反而需要更多的代码，那是否可以直接嵌入组件中？
- 性能会收到影响吗？当组件越来越复杂，就越要考虑这个问题。可以将状态下放来降低渲染造成的消耗
- 是否有一个明确的理由？分离代码我想要实现什么？更松散的耦合、可以被复用等，如果回答不了这个问题，那最好先不要从组件中抽离。
- 这些好处是否超过了成本？分离代码需要花费一定的时间和精力，我们要在业务中去衡量，有所取舍。

3. 封装和组合，也可以叫做高内聚+低耦合

即提供 props 和 ref 给外部，而隐藏内部细节。外部不应该和内部的状态、属性等过于耦合，换句话说内部的状态应该保护好，不应该由外部设置。

组合指的则是将多个独立的组件配合完成工作，其实是和分离类似的。
react 的组合可以通过组件间本身的组合方式，比如 jsx 的嵌套。常见的就是 Form 和 FormItem 这样的组合方式。

4. 纯净

不仅适用于组件，在一些库的 api 中也是一样的思想，即尽可能保证组件不包含副作用。
所谓副作用，其实主要是两个方面：

- 多次调用的返回结果应该相同。对组件来说就是参数不变的情况下，组件内部不能发生变化，不管是样式还是内容。
- 不对外部数据进行修改版，比如修改全局变量、增加全局属性、修改数据本身等等。

为了保证纯净，主要要做到两方面：

- 分离：比如上面的 upload 组件的例子，我们把上传部分和 ui 部分分离，保证 ui 部分是纯净的；如果是展示数据的组件，那么数据来源应该是外部或者其他组件传入的，而不应该是在组件内部维护的。
- 对传入的 props、全局的变量等不做修改。如果确实要使用全局值，那也应该是通过 props 传入

第二个方面的解决方法，则可以考虑沙箱或冻结：

- 沙箱：参考[这篇文章](https://www.51cto.com/article/710911.html)。大致是通过 proxy、with 等方式，阻止对关键内容的 get、set 等操作。比如我们可以把 window、history 等对象通过 proxy 封装一份，然后返回的是代理之后的副本，组件内通过这个副本进行操作。
- 冻结：就是对关键对象采用冻结的方式阻止修改，其实和沙箱是类似的原理

5. 测试

组件库不同于业务开发，需要有非常完备的测试过程，并且可能需要想到各种场景，通过 mock 的形式来完成测试。

6. 其他设计原则。

这部分主要是通用的一些设计方法，不仅是组件库内的组件，也是业务组件的设计思想。
参考文章：
https://juejin.cn/post/6844903767108747278
https://dmitripavlutin.com/7-architectural-attributes-of-a-reliable-react-component/

7. 性能保证，尤其是在一些渲染元素较多的场景，比如长列表、Select 和 Options 等，可能会用到虚拟列表等多种方式

## 具体实现

对于一个组件库内的每一个组件，能说的内容非常多。从思想和设计上，以及从具体代码上都是有不同的内容。具体可以分为几个方面：

1. 组件核心代码。这部分内容和组件本身强相关，比如 button 组件的核心是 button，Calender 组件的核心逻辑是日期等等。这些内容可以放在组件本身逻辑来讲，并且应该是和组件其他方面的设计分离的

2. 组件外围设计，包括：
   - props、ref 等 api 设计
   - 样式的管理
   - 技术选型，比如 scss/less/css-in-js，各种库的选择、js/ts 选择等
   - 打包、发布、测试、代码质量等等开发流程方面

下面针对的主要是核心逻辑之外的内容，即对于通用的组件设计来说，应该注意些什么

### api 设计

api 主要包括组件的 props 和 ref，以及一些通过 context 来设置的内容。
props 的设计主要从两个方面考虑：属性的选择和规范控制

1. props 也要满足单一职责。

一个 props 通常只表示一个属性的控制即可，不要出现过于耦合的情况。比如 Table 组件，column 和内部的数据的 props 应该分离开，而不是合成一个 props 再去分解。

2. 提供扩展性。扩展性就是指用户能通过 props 更灵活定义组件，而不是只能在基本的样式和内容中使用。

主要包含几个方面：

- style 和 className 属性，在一定程度上可以让用户自定义样式。className 可以通过 classnames 库和内部类名合并，而 style 可以在合并、筛选等操作过后直接插入组件内部。
- renderProps 也是提供扩展性的方法，用户可以自定义插入的内容，组件内提供对应的插槽位置。甚至可以把大部分内容都暴露出去，通过默认值来实现默认 dom 结构和样式，然后大多数位置上的具体结构则几乎完全可以由用户自定义得来。
- 有些 props 的值要支持更多选项。比如 button 的样式有 primary、danger 等几种，这种方式就比简单的 true/false 要来的更有扩展性。并且在日后更新中也只需要增加属性，而不是完全修改属性。

3. props 的取值要尽可能简单。

不管是数组、函数、字符串还是其他各种类型，都要保证 props 维持一个简洁易用的属性。
比如如果能采用基本类型数据，就不要用数组、对象等嵌套的复杂的数据。

4. 维持内部的封闭

也是开闭原则的体现，其实就是把内部封装好，尽可能不要通过各种方法把组件内部的状态、实例等暴露出去让外部修改。
比如 ref 虽然是一种易用的形式，但是要注意对 ref 的控制，不能任由外部随意通过 ref 修改 dom 结构或内部的 state。

5. props 的纯粹。纯粹和上面说的组件要维护纯净其实是类似的，或者说 props 本身就是维护组件纯净的最重要的方式。

组件不应该通过 props 获取一些可能会修改外部的值的方法，也不应该传入全局变量等可能会导致副作用的值。
如果存在非纯部分，那么要做的就是和纯部分分离。分离的方式有很多，子组件、自定义 hooks、hoc、context 等都是分离的方法

---

上面的主要是一些抽象的设计思想。在具体的代码中，props 也要注意很多，比如：

1. 类型的设置，即组件的 props 的 interface。

越是复杂的组件，其 props 的类型也就越复杂。一个组件几十个 props，一般也不会写在一个 interface 内部，而是会根据组件的不同用处以及 props 的类型，将 interface 拆分，再利用一些工具方法组合起来。

举个栗子，这是 arco-design 的 button 组件的基础类型：

```ts
export interface BaseButtonProps {
  style?: CSSProperties;
  className?: string | string[];
  children?: ReactNode;
  /**
   * @zh
   * 按钮主要分为六种按钮类型：主要按钮、次级按钮、虚框按钮、文字按钮、线性按钮，`default` 为次级按钮。
   * @en
   * A variety of button types are available: `primary`, `secondary`, `dashed`,
   * `text`, `linear` and `default` which is the secondary.
   * @defaultValue default
   */
  type?: "default" | "primary" | "secondary" | "dashed" | "text" | "outline";
  /**
   * @zh 按钮状态
   * @en Status of the button
   * @defaultValue default
   */
  status?: "warning" | "danger" | "success" | "default";
  /**
   * @zh 按钮的尺寸
   * @en Size of the button
   * @defaultValue default
   */
  size?: "mini" | "small" | "default" | "large";
  /**
   * @zh 按钮形状，`circle` - 圆形， `round` - 全圆角， `square` - 长方形
   * @en Three button shapes are available: `circle`, `round` and `square`
   * @defaultValue square
   */
  shape?: "circle" | "round" | "square";
  /**
  // ...
  /**
   * @zh 点击按钮的回调
   * @en Callback fired when the button is clicked
   */
  onClick?: (e: Event) => void;
}
```

在基础类型上，可以再做扩展、修改和减少。并且可以看到每个类型都有对应的 ts-doc 注释。

2. props 默认值的设置。除非是与数据相关的、没有就展示不了的 props，其他大多数功能性的 props 都要尽可能提供默认值；默认值、传入值和全局的值可以合并成最终的 props。

对于用户传入的 props，也应该做校验，尤其是类型比较复杂的，比如 table 的 columns。如果数据不合法，也要做兜底和报警

3. props 名称的设计。一般来说风格要统一，比如事件回调统一叫 onXXX，布尔值属性叫 xxxable 等等。并且也要符合英语语法、语序等

### 样式管理

现在组件库通用的样式方案主要还是外部样式，比如 scss、less 等预处理框架，或者直接写成 js 的形式，配合 classnames 来实现样式的管理。
以 arco-design 为例，一个按钮的样式分为几个部分：

- 全局样式 global.less，里边定义了各种变量和他们对应的数值。

```less
/******** borderSize *******/

@border-none: 0;
@border-1: 1px;
@border-2: 2px;

/******** borderStyle *******/

@border-solid: solid;
@border-dashed: dashed;
@border-dotted: dotted;

/******** radius *******/

@border-radius-none: 0;
@border-radius-small: 2px;
@border-radius-medium: 4px;
@border-radius-large: 8px;
@border-radius-circle: 50%;

// ...
```

这些是尺寸相关的，还有颜色相关的

- 默认样式 default.less，设置组件库的默认样式，比如前缀、z-index、transition 效果、字体大小等全局的默认样式
- token.less，设置当前组件的全部样式，实际上是用于后面拼接类名之后的结果。比如：

```less
@btn-size-mini-height: @size-mini;
@btn-size-small-height: @size-small;
@btn-size-default-height: @size-default;
@btn-size-large-height: @size-large;
```

这几个值，在 index.less 样式文件中，只需要传入对应的尺寸参数即可，比如`@btn-size-@{size}-height`。

- index.less，即最终导入的样式。这里大量应用的 less 的 mixin 和变量。比如刚刚尺寸相关的：

```less
.btn-size(@size) {
  // mixin，传入size参数
  .@{btn-prefix-cls}-size-@{size} {
    // 类名使用size，和组件内的元素类名对应
    padding: 0 ~"@{btn-size-@{size}-padding-horizontal}"; // 值采用变量，即上面的token中的
    font-size: ~"@{btn-size-@{size}-font-size}";
    height: ~"@{btn-size-@{size}-height}";
    border-radius: ~"@{btn-size-@{size}-radius}";
    // line-height: calc(~'@{size-@{size}}' - ~'@{btn-size-@{size}-border-width}' * 2);
  }
}
```

最后，这个 index.less 并没有被引入到 Button/index.tsx 中，而是和其他样式打包在一起，最后统一导入。这样也有利于类名和样式的分离

组件内的类名，则通过 classnames 来合并。合并的内容一般有：

- 全局的类名前缀，一般是组件库专属的，比如 ant-design-xxx
- 和布局相关的 props，比如下面的 type、size 和 shape
- 可选的部分类名，比如 disable、rtl 等，这些值为空的话不会加上类名

```js
const prefixCls = getPrefixCls("btn");
const _type = type === "default" ? "secondary" : type;
const classNames = cs(
  prefixCls,
  `${prefixCls}-${_type}`,
  `${prefixCls}-size-${size || ctxSize}`,
  `${prefixCls}-shape-${shape}`,
  {
    [`${prefixCls}-long`]: long,
    [`${prefixCls}-status-${status}`]: status,
    [`${prefixCls}-loading-fixed-width`]: loadingFixedWidth,
    [`${prefixCls}-loading`]: loading,
    [`${prefixCls}-link`]: href,
    [`${prefixCls}-icon-only`]:
      iconOnly || (!children && children !== 0 && iconNode),
    [`${prefixCls}-disabled`]: disabled,
    [`${prefixCls}-two-chinese-chars`]: isTwoCNChar,
    [`${prefixCls}-rtl`]: rtl,
  },
  className
);
```

---

另外一种解决方案是使用 css-in-js，比如 styled-components。这种写法最终的产物不包含单独需要引入的 css 文件，同时也不需要全量引入 css，因此也是一种不错的选择。

### 组件库打包

当开发完成后，组件库就需要完整的打包。打包的结果其实就是安装在用户 node_modules 目录下的内容。

组件库打包应该要注意几点：

1. 打包工具的选择

主流的两个：webpack 和 rollup。虽然后者更为推荐，但我观察到 antd、arcod 都用的是 webpack 打包。webpack5 之后对各方面支持更好了，其实在和 rollup 方面差距也就小了不少。

2. 组件库目录的设计

参考 arco-design 的目录，核心内容就是 components 目录
![](https://pic.imgdb.cn/item/64d2220c1ddac507cc336864.jpg)

每个组件内都是独立的，包含这个组件的入口文件、子组件、样式、接口、测试文件等等

![](https://pic.imgdb.cn/item/64d2223a1ddac507cc33d4f7.jpg)

在 components 目录根下保存一个 index.ts，导入所有的组件并导出，作为打包的入口文件

```ts
export type { AlertProps } from "./Alert/interface";
export { default as Alert } from "./Alert";
export type { AnchorProps, AnchorLinkProps } from "./Anchor/interface";
export { default as Anchor } from "./Anchor";
// ...
```

这样做的目的是形成多入口。index.ts 在编译之后就成为 esm 的入口，在实际项目中作为 tree-shaking 的支持

3. 打包主要配置项

一般来说组件库的打包结果分为几种输出：

![](https://pic.imgdb.cn/item/64d223261ddac507cc364b27.jpg)

- lib: 一般是 commonjs 模块化的打包结果
- es: esm 的打包结果，也就是支持 tree-shaking 的
- dist: 全量打包，一般打包出来的是 xxx.min.js 和 xxx.js，作为 umd 的模块化方案，可以直接在 html 中通过 cdn 引入。

webpack 支持不同模块化方案的打包，通过不同的 target 配置就可以生成这些方案。

具体的打包配置，比如 plugin、loader 的使用都要根据实际情况去做选择。比如我们要分离样式，那么就需要用到分离样式的 MinCssPlugin
有几个关键配置内容：

- entry：如果希望每个组件都独立打包，那么就需要配置多入口，每个组件的入口函数都可以作为一个 entry；同时还可以把一些公共引入的库单独打包出来
- externals：一般把 vue、react 这类单独分离，因为他们不需要和组件库打包在一起，只用作为一个 devDependices 就行。
- output：如果组件单独打包，那么就要配置输出的文件名。一般配置为`[name].js`即可
- library：配置库的模块名，即用户导入时用什么名称导入这个库。比如声明为`xxx-design`，那么用户就通过`import xxx from 'xxx-design'`来导入。还可以配置其他方式，比如我们设置`{name: 'MyLibrary',type: 'var'}`，那么当通过 cdn 的形式导入时，就会存在一个名为 MyLibrary 的全局变量，然后通过 MyLibrary.xxx 就可以得到内部的组件。不过这种方法通常只用于 umd 和部分 cjs（可以通过 global.xxx 访问），esm 不需要配置。

4. 按需导入的实现

这里说的按需导入是指组件库通过一些配置的保证，然后在用户使用的时候能正常配合形成按需导入的效果。
按需导入有两种实现方式：

- tree-shaking：对于支持 tsk 的项目，可以直接使用 tsk 来实现。在组件库中其实就是体现为入口文件应该是一个使用 export 具名导出各个组件的文件。

具体来说有几个部分：
首先，需要保证组件目录是单独的，即每个组件独立打包
然后入口文件如下：

```js
export { default as Button } from "./Button";
export { default as Calendar } from "./Calendar";
export { default as Card } from "./Card";
// ...
```

> 注意这是 esm 导入的结果。对于 umd 和 cjs，一般不支持这么导出，也不能使用 tsk

最后设置一下 package.json 的入口文件：

```js
{
  "module": "./es/index.js",
  "main": "./lib/index.js",
  "unpkg": "./dist/xxx.min.js",
  "types": "./es/index.d.ts",
}
```

module 就表示当使用 esm 导入该模块时的入口。其他两个同理

- babel-plugin-import：第二种方式，用于对不支持 tsk 的项目使用，或者放弃上一种直接用这种也行。

注意 babel-plugin-import 这个东西是应该在实际使用的项目内安装配置，而不是在组件库配置。也就是说组件库只要做好组件独立目录，babel-plugin-import 会解析模块并修改为具体的那一个的组件目录。

5. 其他工作

- tsc 可以配置自动生成声明文件。每个组件的 js 文件都应该有一个对应的自动生成的声明文件
- 样式：通常采用的方式是利用 MiniCssExtractPlugin 将样式完全分离，打包的时候所有组件的样式都被整合在一起，使用时通过在入口导入。比如 arco-design 和 antd 都是这样：

```js
import React from "react";
import ReactDOM from "react-dom";
import { Button } from "@arco-design/web-react";
import "@arco-design/web-react/dist/css/arco.css";
```

### 后续流程

后面的主要就是测试、link 调试、发布等过程了，和组件库本身设计思想无关，不做赘述。

# 跨框架组件设计

如果希望开发一个跨框架的组件，在 react、vue、原生 html 都能使用，那么最好的选择就是 WebComponent

当然直接写原生的 WC 也比较费力，有一些框架是基于 WC 实现的，比如[Stencil](https://stencil.docschina.org/docs/introduction)
还有些库可以完成将 react、vue 等库编译成 WC 的形式。

# 组件逻辑设计

之前的内容大多数是一些比较抽象的整体思想，具体到单个组件的实现原理和注意事项，以及思考路径，还要做进一步的探讨。

## 组件设计思路

前端组件的一般分类:

- 通用型组件: 比如 Button, Icon 等.
- 布局型组件: 比如 Grid, Layout 布局等.
- 导航型组件: 比如面包屑 Breadcrumb, 下拉菜单 Dropdown, 菜单 Menu 等.
- 数据录入型组件: 比如 form 表单, Switch 开关, Upload 文件上传等.
- 数据展示型组件: 比如 Avator 头像, Table 表格, List 列表等.
- 反馈型组件: 比如 Progress 进度条, Drawer 抽屉, Modal 对话框等.
- 其他业务类型

所以我们在设计组件系统的时候可以参考如上分类去设计,该分类也是 antd, element, zend 等主流 UI 库的分类方式.

如果我们要设计一个组件，应该的思路流程差不多是：

1. 组件的基本需求，即需要哪些功能
2. 组件的拆分，哪些部分是核心，哪些部分是可以通过拆分或复用其他组件得来的，哪些具体逻辑需要进一步分离。
   - 拆分还包括从 ui 角度的拆分，一个组件可能会分成不同部分
3. 根据具体的需求，确定需要哪些 api、类型如何，哪些是必须哪些是可选
4. 组件的具体实现原理，包括 js 和 css，不同组件有所不同
5. 组件的整理，即该分离的分离、该整合的整合，使得开发完后的组件目录下是完善的、可测试的。

以 Modal 组件为例，后面的组件大题开发流程都是这样的，只是核心逻辑和细节实现不同。因此后面只说实现原理和关键点即可。

## Modal

参考：https://juejin.cn/post/6844904064392626183

Modal 的基本原理比较简单，其实就是绝对定位一个元素，蒙层则是一个层级稍低的 fixed 元素。还可以扩展显示和消失的动画。
具体需求点可以总结为：

- style: 能控制 Modal 主体的样式
- afterClose: 提供 Modal 完全关闭后的回调
- cancelText: 能控制取消按钮文字和样式
- confirmText: 能控制确认按钮文字和样式
- centered: 控制 modal 展示的位置
- closeable: 控制是否显示右上角的关闭按钮
- closeIcon: 可以配置自定义关闭图标
- destroyOnClose: 配置关闭时是否销毁 Modal 里的子元素
- footer: 自定义模态框底部内容
- keyboard: 控制是否支持键盘 esc 关闭
- mask: 控制是否展示遮罩
- maskCloseable: 控制点击蒙层是否允许关闭
- maskStyle: 自定义遮罩样式
- title: 自定义标题
- visible: 控制对话框是否可见
- width: 自定义对话框宽度
- onCancel: 暴露点击遮罩层或右上角叉或取消按钮的回调
- onConfirm: 提供点击确定回调

开发流程如下：

1. 实现基础配置功能

基础配置功能往往和业务逻辑无关， 仅仅用来控制元素的显示隐藏等，可以先控制实现这些简单的布局相关的功能，包括

- cancelText
- closable
- closeIcon
- footer
- mask
- maskStyle
- okText
- title
- width

按照 modal 的 ui 结构将其分成四个部分：header 和 title、content、footer 和 mask。大致的结构如下：

```jsx
// ...
function Modal(props) {
  // ...
  return (
    <div className="wrap">
      <div className="content">
        <div className="header">
          <div className="title">{title}</div>
        </div>
        {closable && (
          <span className="closeBtn">
            {closeIcon || <Icon type="FaTimes" />}
          </span>
        )}
        <div className="body" style={bodyStyle}>
          {children}
        </div>
        {footer === null ? null : (
          <div className="footer">
            {footer ? (
              footer
            ) : (
              <div className="footerBtn">
                <Button className="footerBtnCancel" type="pure">
                  {cancelText}
                </Button>
                <Button className="footerBtnOk">{okText}</Button>
              </div>
            )}
          </div>
        )}
      </div>
      {mask && <div className="mask" style={maskStyle}></div>}
    </div>
  );
}
```

样式上，外部的 wrap 作为整体可以采用 fixed 定位。mask 通过绝对定位充满背景，z-index 稍低，content 同样是绝对定位，可以通过 position+transform 来居中。

```css
// 示例样式，不是最终实现
.wrap {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}
.mask {
  background-color: rgba(105, 104, 104, 0.464);
  z-index: 9;
  position: absolute;
  width: 100%;
  height: 100%;
}
.content {
  position: fixed;
  z-index: 10;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

2. 实现 visible 的控制。

通常组件内的关键状态不会直接采用外部传入作为自己的，即，虽然有 props.visible，但还是需要在组件内维护一个 visible，可以从 props 中派生，但不能只依赖于外部的 props。这样做主要是为了保证组件的封闭性，并且不会受到 props 不合法的影响。

```js
const [isHidden, setHidden] = useState(!props.visible);
const handleClose = () => {
  setHidden(false);
};
```

很多 modal 打开的时候会包含动画效果，一般是控制 scale 属性来实现，可以通过 transition、css 动画或 js 动画。

- modalContent：scale 从 0 到 1 的变化
- mask：透明度从 0 到 1 的变化

3. 样式

- 类名：通过 classnames 库来合并，包括全局的前缀、自定义类名和部分 props。
  在 Modal 中，和布局有关的 props 只有 centered。我们通过 classnames 库来将其整合：

```js
const prefixCls = getPrefixCls("modal");
const classNames = cs(
  `${prefixCls}-wrapper`,
  {
    [`${prefixCls}-wrapper-align-center`]: centered,
    [`${prefixCls}-wrapper-rtl`]: rtl,
  },
  wrapClassName
);
```

- 具体样式：如果用的是 less 或 scss，配合上面的类名就可以。

```less
@modal-prefix-cls: ~"@{prefix}-modal";

.@{modal-prefix-cls}-mask {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: @z-index-modal;
  display: none;
  background-color: var(~"@{arco-cssvars-prefix}-color-mask-bg");
}

.@{modal-prefix-cls}-wrapper {
  position: fixed;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  z-index: @z-index-modal;
  overflow: auto;

  &&-align-center {
    text-align: center;
    white-space: nowrap;
  }
}
```

当然这个是比较复杂的实现方式。更简单的其实就是直接在 className 属性里拼接，然后在 css 中编写对应样式即可

```jsx
<div className={`wrap${centered ? "-centered" : ""}`}></div>
```

```css
.wrap {
  ...;
}
.wrap-centered {
  ...;
}
```

4. 其他完善。

上面两步基本已经完成了核心内容，剩下的工作就是完善 props 的默认值兜底、类型设置、样式开发等等

## Drawer

Drawer 的实现思路和 Modal 差不多，主要有两个区别比较大的特点

- 上下左右四个方向弹出：其实就是控制布局，如果左边弹出就是`left: 0`，隐藏是`left: -100%`，其他几个方向同理
- 显示和隐藏：通过`width: visible ? '100%' : '0'`设置，配合`transition: all .5s`就可以实现动画
- 任意元素内插入：主要是利用 ReactDOM.createPortal。向外暴露一个 getContainer 的方法，如果返回一个 dom 元素就设置为 Portal，否则使用 body（就是正常返回）。

```js
getContainer === false
  ? childDom
  : ReactDOM.createPortal(childDom, getContainer);
```


