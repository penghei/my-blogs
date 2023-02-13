---
title: webpack学习
date: 2022-01-09 13:14:15
tags: 日常学习
cover: /img/webpack.png
categories: 工具
sticky: 3
---

# Webpack 概述

1. Webpack 是一个模块打包工具，可以使用它管理项目中的模块依赖，并编译输出模块所需的静态文件。
2. 它可以很好地管理、打包开发中所用到的 HTML,CSS,JavaScript 和静态文件（图片，字体）等，让开发更高效。
3. 对于不同类型的依赖，Webpack 有对应的模块加载器，而且会分析模块间的依赖关系，最后合并生成优化的静态资源

## 核心概念

- `Entry`：入口，Webpack 执行构建的第一步将从 Entry 开始，可抽象成输入。

- `Module`：模块，在 Webpack 里一切皆模块，一个模块对应着一个文件。Webpack 会从配置的 Entry 开始递归找出所有依赖的模块。`module`本质上是 webpack 对打包的基本单位，每一个文件都算一个 module；module 下有一个 rules 字段，rules 下有就是处理模块的规则，配置哪类的模块，交由哪类 loader 来处理。

- `Chunk`：代码块，Chunk 是 Webpack 打包过程中一堆 module 的集合。Webpack 的打包是从一个入口文件开始，也可以说是入口模块，入口模块引用这其他模块，模块再引用模块。Webpack 通过引用关系逐个打包模块，这些 module 就形成了一个 Chunk；
一条路径一般只会形成一个 Chunk；如果module是异步的（比如动态导入的module），就会创建新的Async chunk，并把该异步模块及其子模块放入。
  chunk 还有一些属性，被称为内置变量，可以在`filename`等配置中用`[]`引用，主要有：
  |变量名| 含义|
  |---|---|
  |id |Chunk 的唯一标识，从 0 开始|
  |name| Chunk 的名称|
  |hash |Chunk 的唯一标识的 Hash 值|
  |chunkhash| Chunk 内容的 Hash 值|
  |contenthash|根据文件内容生成的 hash|

> 这几种 hash 值的用处在哪里，区别在哪里？
>
> - 首先，hash 和项目相关，当项目中的某个文件改变时，整个项目的 hash 都会发生改变，那些没改变的 chunk、content 也会改变 hash
> - chunkhash 和对应的 chunk 相关。由于 chunk 通常是从一个 entry 开始的所有文件的打包结果，因此如果 chunk 内部的某个文件内容改变，可能会导致该 chunk 包含的其他文件的 chunkhash 也改变
> - contenthash 则是根据文件内容，相对来说最精确
>  除了便于输出文件不重名，这几个 hash 还有更重要的作用，就是在缓存时表示哪个文件被修改，需要更新缓存。
>  注意这里的更新一般指的是版本上的更新。比如说当前页面已经上线，现在做了一次更新；但是由于用户客户端的缓存还没有过期，客户端还用的是之前的代码，该如何让客户端主动请求最新的资源呢？方法就是直接修改资源的文件名。当文件名发生变换，相当于静态资源的路径改了，这样，就相当于第一次访问这些资源，从而更新缓存，完成版本更新
>  大多数情况下，我们可以对可能需要更新的资源文件名，比如 js、css 和图片等静态资源加入 hash 值，方便后续的更新。

- `Loader`：模块转换器，用于把模块原内容按照需求转换成新内容。
- `Plugin`：扩展插件，在 Webpack 构建流程中的特定时机注入扩展逻辑来改变构建结果或做你想要的事情。

> Loader 和 Plugin 的区别
> Loader 本质就是一个函数，在该函数中对接收到的内容进行转换，返回转换后的结果。 因为 Webpack 只认识 JavaScript，所以 Loader 就成了翻译官，对其他类型的资源进行转译的预处理工作。
> Plugin 就是插件，可以扩展 Webpack 的功能，在 Webpack 运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果。

- `Output`：输出结果，在 Webpack 经过一系列处理并得出最终想要的代码后输出结果。
- `Bundle`：最终的打包结果，即一般情况下输出的`bundle.js`文件。大多数时候`chunk`和`bundle`是一一对应的，但是如果开启了`sourceMap`可能会使`chunk`多一个。
- `Dependence`：依赖对象，webpack 基于该类型记录模块间依赖关系

### chunk

chunk由module在seal阶段组成，这部分的具体流程可以参考：

https://juejin.cn/book/7115598540721618944/section/7119035921680302115

chunk在打包过程中会被分成三种类型：
- Entry Chunk：同一个 entry 下触达到的模块组织成一个 Chunk；一个entry就会创建一个chunk，相应的n个入口就有n个chunk
- Async Chunk：异步模块单独组织为一个 Chunk，从异步导入的那个入口模块开始，它的所有子模块都会被打包到异步模块中
- Runtime Chunk：entry.runtime 不为空时，会将运行时模块单独组织成一个 Chunk。这种chunk主要包含的是运行时代码，在输出时会被注入到代码中，实现部分功能，比如
  - ` __webpack_require__.f`、`__webpack_require__.r` 等功能实现最起码的模块化
  - 动态加载特性，则需要写入 `__webpack_require__.e` 函数

## 安装使用及配置

### 安装

当前默认版本都是最新的 webpack5，对应的 loader、plugins 等也都是配套 5 的；当然 4 版本也有对应的 loader、plugin 版
这里除了 webpack 还会给出一些常用的安装

```
npm install webpack webpack-cli --save-dev

# devServer
npm install webpack-dev-server --save-dev

# loader

npm install --save-dev style-loader css-loader
npm install --save-dev file-loader
npm install --save-dev babel-loader
npm install --save-dev typescript ts-loader

# plugin

npm install --save-dev html-webpack-plugin

```

### 命令配置

这两个命令分别实现基础的配置和 devServer 的运行；

```json
{
    "build": "webpack",
    "start": "webpack serve --open"
},
```

如果不生效或者报错，也可以用 npx xxx 来运行

### 文件目录

基本文件目录如下：
![](https://pic.imgdb.cn/item/61da8cf42ab3f51d91c12781.png)

- assert 用于存放静态文件，包括图片等
- dist 为设定的输出文件所在位置，以及 html-webpack-plugin 生成 html 文件的位置；如果有模板 html 最好放在 assert 防止被 clean 清除
- src 为主要 js 或 css 文件
- webpack.config.js 是 webpack 的设定文件

> 注意：html 文件尽量不要自己写，容易导致 devServer 找不到 html 文件；
> 一般来说 index.html 放在 dist 文件夹下，使用绝对路径(src="bundle.js")引入打包 js 并插入到 head 中；这点很重要！！！devServer 的根路径就是 dist 文件夹，因此建议使用 html-webpack-plugin 自动生成，不容易出错

# webpack.config.js

webpack.config.js 作为 webpack 的配置文件，通常会导出一个对象给 webpack 配置使用；由于 webpack 是一个黑盒，我们不需要直到内部运行，相关的配置几乎都是在这个文件中完成并实现的。

## 基本配置

```js
const path = require("path");

module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
  target: "web",
  module: {
    rules: [],
  },
  plugins: [],
  mode: "none",
};
```

![](https://pic.imgdb.cn/item/63ddeb774757feff33b6026f.jpg)

webpack 的配置按照类型可以分为两个大类：

- 流程类：作用于打包流程某个或若干个环节，直接影响编译打包效果的配置项
- 工具类：打包主流程之外，提供更多工程化工具的配置项

流程类主要有：

- 输入输出：
  - entry：用于定义项目入口文件，Webpack 会从这些入口文件开始按图索骥找出所有项目文件；
  - context：项目执行上下文路径；
  - output：配置产物输出路径、名称等；
- 模块处理：
  - resolve：用于配置模块路径解析规则，可用于帮助 Webpack 更精确、高效地找到指定模块
  - module：用于配置模块加载规则，例如针对什么类型的资源需要使用哪些 Loader 进行处理
  - externals：用于声明外部资源，Webpack 会直接忽略这部分资源，跳过这些资源的解析、打包操作
- 后处理：
  - optimization：用于控制如何优化产物包体积，内置 Dead Code Elimination、Scope Hoisting、代码混淆、代码压缩等功能
  - target：用于配置编译产物的目标运行环境，支持 web、node、electron 等值，不同值最终产物会有所差异
  - mode：编译模式短语，支持 development、production 等值，可以理解为一种声明环境的短语

Webpack 首先需要根据输入配置(entry/context) 找到项目入口文件；之后根据按模块处理(module/resolve/externals 等) 所配置的规则逐一处理模块文件，处理过程包括转译、依赖分析等；模块处理完毕后，最后再根据后处理相关配置项(optimization/target 等)合并模块资源、注入运行时依赖、优化产物结构等。

工具类主要有：

- 开发效率类：
  - watch：用于配置持续监听文件变化，持续构建
  - devtool：用于配置产物 Sourcemap 生成规则
  - devServer：用于配置与 HMR 强相关的开发服务器功能
- 性能优化类：
  - cache：Webpack 5 之后，该项用于控制如何缓存编译过程信息与编译结果
  - performance：用于配置当产物大小超过阈值时，如何通知开发者
- 日志类：
  - stats：用于精确地控制编译过程的日志内容，在做比较细致的性能调试时非常有用
  - infrastructureLogging：用于控制日志输出方式，例如可以通过该配置将日志输出到磁盘文件

### 整体

整个 webpack.config.js 文件导出的部分，大多数情况是一个对象，但是还有两种形式：

- 配置对象数组：每个数组项都是一个完整的配置对象，每个对象都会触发一次单独的构建，通常用于需要为同一份代码构建多种产物的场景，比如把一个库打包成 ESM/CMD/UMD 等多种模块形式
- 函数：Webpack 启动时会执行该函数获取配置，我们可以在函数中根据环境参数(如 NODE_ENV)动态调整配置对象。

数组对象如下例子：

```js
// webpack.config.js
module.exports = [
  {
    entry: "./src/index.js",
    // 其它配置...
  },
  {
    entry: "./src/index.js",
    // 其它配置...
  },
];
```

使用数组方式时，Webpack 会在启动后创建多个 Compilation 实例，并行执行构建工作。但 Compilation 实例间基本上不作通讯，这意味着这种并行构建对运行性能并没有任何正向收益，例如某个 Module 在 Compilation 实例 A 中完成解析、构建后，在其它 Compilation 中依然需要完整经历构建流程，无法直接复用结果。

---

如果导出的是一个函数，那么 webpack 会调用这个函数，并且给函数传入两个参数。这两个参数都是启动 webpack 时传入的命令行参数

- env: 通过 --env 传递的命令行参数
- argv: 命令行 Flags 参数，即除了 env 参数之外的其他命令行参数，详见：https://webpack.js.org/api/cli/#flags

函数形式的配置可以更加灵活，函数内部可以根据命令行参数、环境变量等采取不同的配置、导入不同的 loader 和 plugin。

```js
module.exports = function (env, argv) {
  // ...
  return {
    entry: "./src/index.js",
    // 其它配置...
  };
};
```

### 环境治理

在现代前端工程化实践中，通常需要将同一个应用项目部署在不同环境(如生产环境、开发环境、测试环境)中，以满足项目参与各方的不同需求。这就要求我们能根据部署环境需求，对同一份代码执行各有侧重的打包策略，例如：

- 开发环境需要使用 webpack-dev-server 实现 Hot Module Replacement；
- 测试环境需要带上完整的 Soucemap 内容，以帮助更好地定位问题；
- 生产环境需要尽可能打包出更快、更小、更好的应用代码，确保用户体验。

Webpack 中有许多实现环境治理的方案，比如上面介绍过的，使用“配置函数”配合命令行参数动态计算配置对象。除此之外，业界比较流行将不同环境配置分别维护在单独的配置文件中，如：

```
.
└── config
  ├── webpack.common.js
  ├── webpack.development.js
  ├── webpack.testing.js
  └── webpack.production.js
```

之后配合 --config 选项指定配置目标，如：

```
npx webpack --config webpack.development.js
```

这种模式下通常会将部分通用配置放在基础文件中，如上例的 webpack.common.js，之后在其它文件中引入该模块并使用 webpack-merge 合并配置对象。

webpack-merge 是一个专为 Webpack 设计的数据合并(merge)的工具，功能逻辑与 Lodash 的 merge 函数、 Object.assign 等相似，但支持更多特性，如支持数组属性合并、支持函数属性合并、支持设定对象合并策略等。总之要合并配置项，就一定需要 webpack-merge 提供的 merge 函数，用 lodash 的 merge 会导致覆盖、函数不能合并等问题。

举个例子，比如先创建一个基本的配置，包含 entry/output/部分 loader 和 plugin

```js
// webpack.common.js
const path = require("path");
const HTMLWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: { main: "./src/index.js" },
  output: {
    filename: "[name].js",
    path: path.resolve(__dirname, "dist"),
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: ["babel-loader"],
      },
    ],
  },
  plugins: [new HTMLWebpackPlugin()],
};
```

然后创建一个 webpack.development.js，对应开发环境下的配置：

```js
// webpack.development.js
const { merge } = require("webpack-merge");
const baseConfig = require("./webpack.common");

// 使用 webpack-merge 合并配置对象
module.exports = merge(baseConfig, {
  mode: "development",
  devtool: "source-map", // 开启source-map
  devServer: { hot: true }, // 开启hmr
});
```

然后开发环境下调用 yarn dev 就可以按照这种形式配置

```json
{
  "script": {
    "dev": "webpack --config=webpack.development.js"
  }
}
```

其他的测试、生产等环境也类似。

### entry

Entry 类型可以是以下三种中的一种或者相互组合：

| 类型     | 例子                                                             | 含义                                                                                                                              |
| -------- | ---------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| string   | `'./src/entry'`                                                  | 入口模块的文件路径，可以是相对路径。                                                                                              |
| array    | `['./src/entry1', './src/entry2']`                               | 数组最后一个文件是资源的入口文件，数组其余文件会预先构建到入口文件。                                                              |
| object   | `{ a: './src/entry-a', b: ['./src/entry-b1', './src/entry-b2']}` | 配置多个入口，每个入口生成一个 Chunk，filename 的 `[name]` 对应这里的键名。对象内每个属性的值还可以是一个对象，作为该模块的配置项 |
| function | `() => './demo'`                                                 | 在 webpack 生命周期的 make 阶段调用，相当于动态导入                                                                               |

```js
module.exports = {
  entry: "./src/index.js",
  entry: ["./src/entry1", "./src/entry2"],
  entry: {
    app1: "./src/index.js",
    app2: "./src/main.js",
  },
};
```

如果是 string 和 array 型的 entry 生成的 chunk 名默认都是 `main`，而 object 形式的 entry 会依据键名生成对应的 chunk 名

对象类型的 entry 的属性，也可以是一个包含配置项的对象，比如：

```js
module.exports = {
  entry: {
    shared: ["react", "react-dom", "redux", "react-redux"],
    catalog: {
      import: "./catalog.js",
      filename: "pages/catalog.js",
      dependOn: "shared", // dependOn属性可以让本入口的配置和其他入口的配置使用相同的模块，比如这里就会使用shared配置的模块
      chunkLoading: false,
    },
    personal: {
      import: "./personal.js",
      filename: "pages/personal.js",
      dependOn: "shared",
      chunkLoading: "jsonp",
      asyncChunks: true,
      layer: "name of layer",
    },
  },
};
```

这里的配置项有：

- import：声明入口文件，支持路径字符串或路径数组(多入口)；
- dependOn：声明该入口的前置依赖 Bundle；这个属性对于多入口很重要，如果不配置，同一个模块就会被不同的入口导入多次，非常占用体积
- runtime：设置该入口的 Runtime Chunk，若该属性不为空，Webpack 会将该入口的运行时代码抽离成单独的 Bundle；不能和 dependOn 同时设置
- filename：效果与 output.filename 类同，用于声明该模块构建产物路径；
- library：声明该入口的 output.library 配置，一般在构建 NPM Library 时使用；
- publicPath：效果与 output.publicPath 相同，用于声明该入口文件的发布 URL；
- chunkLoading：效果与 output.chunkLoading 相同，用于声明异步模块加载的技术方案，支持 false/jsonp/require/import 等值；
- asyncChunks：效果与 output.asyncChunks 相同，用于声明是否支持异步模块加载，默认值为 true。

dependOn 属性的值可以是一个路径，也可以是 entry 中的其他配置入口。比如：

```js
module.exports = {
  // ...
  entry: {
    main: "./src/index.js",
    foo: { import: "./src/foo.js", dependOn: "main" },
  },
};
```

foo 的 dependOn 属性为 main，即说明 foo 依赖于 main 入口的打包结果，加载 foo 之前一定会加载 main，因此可以将重复的模块代码、运行时代码等都放到 main 产物，减少不必要的重复。
**注意，dependOn的对象通常是某个外部库，比如lodash、react等，一般不能用作自己编写的模块**

---

为支持产物代码在各种环境中正常运行，Webpack 会在产物文件中注入一系列运行时代码。runtime 配置项用于指定该模块的加载需要什么运行时代码，并且会把这部分运行时代码单独抽出来形成一个模块，然后其他模块可以引入此模块，减少重复。
比如这样的配置：

```js
module.exports = {
  entry: {
    main: { import: "./src/index.js", runtime: "common-runtime" },
    foo: { import: "./src/foo.js", runtime: "common-runtime" },
  },
  output: {
    clean: true,
    filename: "[name].js",
    path: path.resolve(__dirname, "dist"),
  },
};
```

这样的配置就会生成 main.js、foo.js，以及一个 common-runtime.js，就是前面两个模块的运行时代码。

---

如果 entry 传入一个函数，则是动态入口。entry 函数将会在每次 make 阶段执行（make 事件在 webpack 启动和每当 监听文件变化 时都会触发）

```js
module.exports = {
  //...
  entry: () => "./demo",
  entry: () => new Promise((resolve) => resolve(["./demo", "./demo2"])),
};
```

### output

配置输出代码，主要有以下几个配置项：

#### output.filename

- filename 除了可以是一个文件名称，也可以是相对地址，例如`'./js/bundle.js'`；最终打包输出的文件是 path 绝对路径与 filename 的拼接后的地址。
- filename 支持类似变量的方式生成动态文件名，主要写法有以下几种：
  - `hash`：`'[hash].js'`，表示文件的哈希值，用于区分不同文件；一般前 8 位即可，用`[hash:8]`可以取到前 8 位。hash 是根据打包中所有的文件计算出的 hash 值。在一次打包中，所有出口文件的 filename 获得的[hash]都是**一样**的。
  - `id`：Chunk 的唯一标识，从 0 开始
  - `name`：chunk 的名字，即原文件的名字，如果 entry 是对象形式就是和 entry 的键名保持一致
  - `chunkhash`：Chunk 内容的 Hash 值，相比于 hash，每个 chunk 对应的出口 filename 获得的[chunkhash]是不一样的。这样可以保证打包后每一个 JS 文件名都**不一样**

#### output.path

output 的 path 要求一定是**绝对路径**，因此要用 nodejs 的 path 功能拼接当前绝对位置和文件夹；
基本格式：

```js
module.exports = {
  entry: "./a.js",
  output: {
    path: path.resolve(__dirname, ""),
    filename: "bundle.js",
    clean: true,
  },
  mode: "none",
};
```

webpack5 中增加一个 clean 配置项，相当于清理上次打包文件的插件，并且在启动 devServer 时也能生效

#### output.publicPath

通常指资源访问路径，即相当于一个根目录，表示配置发布到线上资源的 URL 前缀，为 string 类型。 默认值是空字符串 ''，即使用相对路径。
比如

```js
filename: "[name]_[chunkhash:8].js";
publicPath: "https://cdn.example.com/assets/";
```

表示 html 中引入文件的路径就应该是

```html
<script src="https://cdn.example.com/assets/a_12345678.js"></script>
```

### loader（module）

loader 属于配置项的 module 部分，主要用于解析、处理项目中的非 js、html 部分解析或者对文件进行一些预处理
主要体现于 rules 数组中，数组里每一项都描述了如何去处理部分文件。 配置一项 rules 时大致通过以下方式：

1. `条件匹配`：通过 `test`（正则匹配）、 `include`（包含） 、 `exclude`（排除） 三个配置项来命中 Loader 要应用规则的文件。
1. `应用规则`：对选中后的文件通过 `use` 配置项来应用 Loader，可以只应用一个 Loader 或者按照从后往前的顺序应用一组 Loader，同时还可以分别给 Loader 传入参数。
1. `重置顺序`：Loader 的执行顺序默认是**从下向上、从右到左**执行，通过 `enforce` 选项可以让其中一个 Loader 的执行顺序放到最前或者最后。

```js

module.exports = {
  module: {
    rules: [
        {
            test: /\.css$/,
            use: ['style-loader', 'css-loader']
            exclude:/node_modules/,
            include:/src/,
            enforce:'pre',//强制use从左向右执行，post为从右向左，即默认的
            noParse: /jquery|chartjs/
        },
    ],
  },
};
```

- `test` 是正则表达式或者正则数组，一般用以解析文件后缀
- `use` 可以有字符串、数组、对象三种写法；字符串和数组都是指一个或多个 loader，对象形式表示对 loader 的额外配置
- `exclude` 与 `include` 可以表示排除或只选中哪些文件，同样是用正则，也可以使用类似`path`的写法指定文件
- `enforce` 用来强制当前 loader 在前或者在后执行，可以有`'pre'`和`'post'`两种写法
- `noParser`：可以让 Webpack 忽略对部分没采用模块化的文件的递归解析和处理，比如 jQuery 等；写法是正则、正则数组或返回正则值的函数；
- `parser`：parser 属性可以更细粒度的配置哪些模块语法要解析，哪些不解析，比如下面这样：

```js
module: {
  rules: [
    {
      test: /\.js$/,
      use: ["babel-loader"],
      parser: {
        amd: false, // 禁用 AMD
        commonjs: false, // 禁用 CommonJS
        system: false, // 禁用 SystemJS
        harmony: false, // 禁用 ES6 import/export
        requireInclude: false, // 禁用 require.include
        requireEnsure: false, // 禁用 require.ensure
        requireContext: false, // 禁用 require.context
        browserify: false, // 禁用 browserify
        requireJs: false, // 禁用 requirejs
      },
    },
  ];
}
```

#### use 的写法

如果只有一个 loader 且没有额外配置就用字符串即可；
use 为数组时，loader 会按照从左至右的顺序执行；对于 css 等需要严格依照顺序不然会报错；
use 为对象的形式：

```js
{
    test: /\.js$/,
    exclude: /node_modules/,
    use: {
      loader: 'babel-loader',
      options: {
        presets: ['@babel/preset-env']
      }
    }
}
```

### resolve

webpack 具有解析特殊配置路径的能力，也就是通过配置 resolve，可以把一些 import 的写法解析简化
基本写法：

```js
module.export = {
  //...
  resolve: {
    alias: {},
  },
};
```

#### resolve.alias

resolve.alias 配置项通过别名来把原导入路径映射成一个新的导入路径；实质上是一种代换，把匹配的字符替换为对应的值

```js
resolve: {
  alias: {
    components: "./src/components/";
  }
}
```

意思是 import 的路径中包含 components 的会被替换为后面的内容，比如`import _ from 'components/lodash`就相当于`import _ from './src/components/lodash`

精确匹配可以使用$结尾的字符串做键名
比如下面表示只有单纯的`'react'`才可以被解析，`'react/xxx'`或者`'xx/react'`都不会被解析；
并且会把`import 'react'`转化为`import '/path/to/react.min.js'`的形式。

```js
const path = require("path");

module.exports = {
  //...
  resolve: {
    alias: {
      react$: "/path/to/react.min.js",
    },
  },
};
```

这个配置提供了一种快速引入的方式；比如 vue 的@符引入的方式简化长串的导入

#### resolve.extensions

用于给导入语句的后缀自动加上
比如默认配置`extensions: ['.js', '.json']`，表示对于`'./b'`这样的无后缀引入会先查找 `b.js` 文件，再查找 `b.json`

> ts 不允许引入后缀为 ts 的文件，因此就必须要有`extensions: ['.ts', '.js', '.json']`这样的配置以保证解析

对应的一个配置是`enforceExtension`，如果设置为 true，那么所有的文件都必须开启后缀。

#### resolve.modules

配置 Webpack 去哪些目录下寻找第三方模块，默认是只会去 node_modules 目录下寻找。
这个配置可以用于简化一些超长的路径，比如经常需要`../../../../xxx`地翻找路径，可以通过配置 modules 为'./src/components'使得导入默认在路径下寻找

```js
modules: ["./src/components", "node_modules"];
```

就可以直接用`import xxx from 'xxx'`引入，webpack 会默认到`./src/components `下去寻找

### plugins

Plugin 的配置很简单，plugins 配置项接受一个数组，数组里每一项都是一个要使用的 Plugin 的实例，Plugin 需要的参数通过构造函数传入。

> 使用 Plugin 的难点在于掌握 Plugin 本身提供的配置项，而不是如何在 Webpack 中接入 Plugin。几乎所有 Webpack 无法直接实现的功能都能在社区找到开源的 Plugin 去解决

```js
plugins: [
  new HtmlWebpackPlugin({
      template: "template.html",
    }),
],
```

注意点：

- plugin 不同于 loader，使用时必须先引入；
- plugin 的形式都为 new xxxPlugin()；其中参数写在对象里，如`new HtmlWebpackPlugin({template: "template.html"}),`

### devServer

在项目根目录执行`npx webpack-dev-server`，就启动了 webpack-dev-server。

> 如果 devServer 启动后的网页出现问题，可以将路由导航至 /webpack-dev-server 将会展示服务文件的位置。例如： http://localhost:9000/webpack-dev-server。
> 效果如下：
> ![](https://pic.imgdb.cn/item/61f42a072ab3f51d917ddfd9.png)

devServer 也是可配置的，上面的 proxy 就是 devServer 配置的一种
常用的配置有：

```js
devServer: {
  hot:true, // 它是热更新：只更新改变的组件或者模块，不会整体刷新页面，在webpack5中默认开启
  open: true, // 是否自动打开浏览器，webpack5中默认开启
  proxy: { },//代理
  client:{
    logging:'log' | 'info' | 'warn' | 'error' | 'none' | 'verbose',//在浏览器中设置日志级别
    overlay: true,//当出现编译错误或警告时，在浏览器中显示全屏覆盖。
    progress: true,//以百分比显示编译进度。
    reconnect: true,//尝试重新连接客户端的次数。当为 true 时，它将无限次尝试重新连接。
  }
  http2: true,//启用HTTP2
  https: true,//启用https，自动生成证书
  historyApiFallback: true,
  contentBase: path.join(__dirname, 'public'),//配置 DevServer HTTP 服务器的文件根目录
  host:`127.0.0.1`,
  port:'8081'//启动的主机和端口
}
```

- `historyApiFallback`：SPA 需要， 浏览器端的 JavaScript 代码会从 URL 里解析出当前页面的状态，显示出对应的界面（返回一个 html 文件），也就是前端路由
  如果简单的配置为 true，都只会返回 index.html；因此需要明确配置，比如：
  ```js
  historyApiFallback: {
    // 使用正则匹配命中路由
    rewrites: [
      // /user 开头的都返回 user.html
      { from: /^\/user/, to: "/user.html" },
      { from: /^\/game/, to: "/game.html" },
      // 其它的都返回 index.html
      { from: /./, to: "/index.html" },
    ];
  }
  ```

#### devServer.proxy

可以在 webpack 的 devServer 中配置代理解决前端跨域

```js
devServer: {
  hot:true, // 它是热更新：只更新改变的组件或者模块，不会整体刷新页面
  open: true, // 是否自动打开浏览器
  proxy: { // 配置代理（只在本地开发有效，上线无效）
    '/api': {
      target: 'http://localhost:3000', // 这是本地用node写的一个服务，用webpack-dev-server起的服务默认端口是8080
      pathRewrite: {"/api" : ""}, // 后台在转接的时候url中是没有 /api 的
      changeOrigin: true, // 加了这个属性，那后端收到的请求头中的host是目标地址 target
      secure: false, // 若代理的地址是https协议，需要配置这个属性
    },
  }
}
```

### 其他配置

#### target

指定 webpack 的打包结果。多数时候 Webpack 都被用于打包 Web 应用，但实际上 Webpack 还支持构建 Node、Electron、NW.js、WebWorker 等应用形态
还可以是：

- `node[[X].Y]`：编译为 Node 应用，此时将使用 Node 的 require 方法加载其它 Chunk，支持指定 Node 版本，如：node12.13；
- `async-node[[X].Y]`：编译为 Node 应用，与 node 相比主要差异在于：async-node 方式将以异步(Promise)方式加载异步模块(node 时直接使用 require)。支持指定 Node 版本，如：async-node12.13；
- `nwjs[[X].Y]`：编译为 NW.js 应用；
- `node-webkit[[X].Y]`：同 nwjs；
- `electron[[X].Y]-main`：构建为 Electron 主进程；
- `electron[[X].Y]-renderer`：构建为 Electron 渲染进程；
- `electron[[X].Y]-preload`：构建为 Electron Preload 脚本；
- `web`：构建为 Web 应用；
- `esX`：构建为特定版本 ECMAScript 兼容的代码，支持 es5、es2020 等；

```js
const baseConfig = {
  mode: "development",
  target: "web",
  devtool: false,
  entry: {
    main: { import: "./src/index.js" },
  },
  output: {
    clean: true,
    path: path.resolve(__dirname, "dist"),
  },
};
```

#### cache

cache 可以让 webpack 缓存生成的 webpack 模块和 chunk，来改善构建速度。它能够将首次构建的过程与结果数据持久化保存到本地文件系统，在下次执行构建时跳过解析、链接、编译等一系列非常消耗性能的操作，直接复用上次的 Module/ModuleGraph/Chunk 对象数据，迅速构建出最终产物。
cache 的配置项主要有：

- cache.type：缓存类型，支持 'memory' | 'filesystem'，需要设置为 filesystem 才能开启持久缓存；
- cache.cacheDirectory：缓存文件路径，默认为 node_modules/.cache/webpack ；
- cache.buildDependencies：额外的依赖文件，当这些文件内容发生变化时，缓存会完全失效而执行完整的编译构建，通常可设置为各种配置文件，如 babelrc 等
- cache.managedPaths：受控目录，Webpack 构建时会跳过新旧代码哈希值与时间戳的对比，直接使用缓存副本，默认值为 `['./node_modules']`；
- cache.profile：是否输出缓存处理过程的详细日志，默认为 false；
- cache.maxAge：缓存失效时间，默认值为 5184000000 。

cache 的配置通常需要设置为 filesystem，才可以开启其他配置项。

```js
const path = require("path");

module.exports = {
  //...
  cache: {
    type: "filesystem",
    cacheDirectory: path.resolve(__dirname, ".temp_cache"),
  },
};
```

filesystem 类型的缓存被称为“持久性缓存”，即将构建数据在一定时间内保存在文件系统中，不会反复生成。
比如各种 loader，babel-loder、eslint-loader、ts-loader 等工具时可能需要重复生成 AST，就会耗费大量时间。而 Webpack5 的持久化缓存功能则将构建结果保存到文件系统中，在下次编译时对比每一个文件的内容哈希或时间戳，未发生变化的文件跳过编译操作，直接使用缓存副本，减少重复计算；发生变更的模块则重新执行编译流程。

![](https://pic.imgdb.cn/item/63e0e9414757feff3377df9e.jpg)

Webpack 在首次构建完毕后将 Module、Chunk、ModuleGraph 三类对象的状态序列化并记录到缓存文件中；在下次构建开始时，尝试读入并恢复这些对象的状态，从而跳过执行 Loader 链、解析 AST、解析依赖等耗时操作，提升编译性能。

## 配置类型

通过`module.exports`导出的不一定是个对象，还可以是一个返回对象的函数，一个包含多种配置的数组，甚至是一个 Promise

详见http://webpack.wuhaolin.cn/2%E9%85%8D%E7%BD%AE/2-9%E5%A4%9A%E7%A7%8D%E9%85%8D%E7%BD%AE%E7%B1%BB%E5%9E%8B.html

比如函数类型，能通过 JavaScript 灵活的控制配置，做到只用写一个配置文件，对于不同的环境采用不同的配置：

```js
const path = require("path");
const UglifyJsPlugin = require("webpack/lib/optimize/UglifyJsPlugin");

module.exports = function (env = {}, argv) {
  const plugins = [];

  const isProduction = env["production"];

  // 在生成环境才压缩
  if (isProduction) {
    plugins.push(
      // 压缩输出的 JS 代码
      new UglifyJsPlugin()
    );
  }

  return {
    plugins: plugins,
    // 在生成环境不输出 Source Map
    devtool: isProduction ? undefined : "source-map",
  };
};
```

在运行 Webpack 时，会给这个函数传入 2 个参数，分别是：

- `env`：当前运行时的 Webpack 专属环境变量，`env` 是一个 Object。读取时直接访问 Object 的属性，设置它需要在启动 Webpack 时带上参数。例如启动命令是 `webpack --env.production --env.bao=foo`时，则 env 的值是 `{"production":"true","bao":"foo"}`。
- `argv`：代表在启动 Webpack 时所有通过命令行传入的参数，例如 `--config`、`--env`、`--devtool`，可以通过 `webpack -h` 列出所有 Webpack 支持的命令行参数。


# 优化

关于 webpack 的优化有很多内容，因为其优化方式本身就有很多，原理有些也比较复杂。这里只总结一些常用或常问的，最重要的还是原理性理解

webpack5 优化参考：https://jelly.jd.com/article/61179aa26bea510187770aa3

## 构建速度优化

### 多进程并行构建

> 受限于 Node.js 的单线程架构，原生 Webpack 对所有资源文件做的所有解析、转译、合并操作本质上都是在同一个线程内串行执行，CPU 利用率极低，因此，理所当然地，社区出现了一些以多进程方式运行 Webpack，或 Webpack 构建过程某部分工作的方案。
> 这些方案的核心设计都很类似：针对某种计算任务创建子进程，之后将运行所需参数通过 IPC 传递到子进程并启动计算操作，计算完毕后子进程再将结果通过 IPC 传递回主进程，寄宿在主进程的组件实例，再将结果提交给 Webpack。

#### thread-loader

把任务分解给多个子进程去并发执行，子进程处理完后再把结果发送给主进程。

在整个 Webpack 构建流程中，最耗时的流程可能就是 Loader 对文件的转换操作了，因为要转换的文件数据巨多，而且这些转换操作都只能一个个挨着处理。
thread-loader 的原理就是将耗时的 loader 放在一个独立的 worker 池中运行，每个 worker 都是一个独立的 node.js 进程，加快 loader 构建速度。

> webpack5 之前用的是 happypack，5 之后就建议使用 thread-loader 了

thread-loader 是一个 loader，一般放在需要优化的 loader 的前面使用；将耗时的 loader 转化放在其后就可以。

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        include: path.resolve("src"),
        use: [
          "thread-loader",
          // 耗时的 loader （例如 babel-loader）
        ],
      },
    ],
  },
};
```

此外，还有一些额外配置，包括：

- workers：子进程总数，默认值为 require('os').cpus() - 1；
- workerParallelJobs：单个进程中并发执行的任务数；
- poolTimeout：子进程如果一直保持空闲状态，超过这个时间后会被关闭；
- poolRespawn：是否允许在子进程关闭后重新创建新的子进程，一般设置为 false 即可；
- workerNodeArgs：用于设置启动子进程时，额外附加的参数。

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          {
            loader: "thread-loader",
            options: {
              // 产生的 worker 的数量，默认是 (cpu 核心数 - 1)，或者，
              // 在 require('os').cpus() 是 undefined 时回退至 1
              workers: 2,
              // 一个 worker 进程中并行执行工作的数量
              // 默认为 20
              workerParallelJobs: 50,
              // 额外的 node.js 参数
              workerNodeArgs: ["--max-old-space-size=1024"],
              // 允许重新生成一个僵死的 work 池
              // 这个过程会降低整体编译速度
              // 并且开发环境应该设置为 false
              poolRespawn: false,
              // 闲置时定时删除 worker 进程
              // 默认为 500（ms）
              // 可以设置为无穷大，这样在监视模式(--watch)下可以保持 worker 持续存在
              poolTimeout: 2000,
              // 池分配给 worker 的工作数量
              // 默认为 200
              // 降低这个数值会降低总体的效率，但是会提升工作分布更均一
              poolParallelJobs: 50,
              // 池的名称
              // 可以修改名称来创建其余选项都一样的池
              name: "my-pool",
            },
          },
          "babel-loader",
          "eslint-loader",
        ],
      },
    ],
  },
};
```

另外多进程优化方式有一个共同的问题，就是频繁创建、销毁进程的开销。如果让 loader 自己去管理进程，那就有可能导致任务繁多时增加进程、任务简洁时销毁增加的进程，如此反复就会浪费很多时间。
解决方式就是线程池机制，或者是类似的方式，即先创建一些子进程，然后后续的 loader 转化任务将会转发到空闲进程处理，预创建的进程不会被销毁
Thread-loader 提供了 warmup 接口：

```js
const threadLoader = require("thread-loader");

threadLoader.warmup(
  {
    // 可传入上述 thread-loader 参数
    workers: 2,
    workerParallelJobs: 50,
  },
  [
    // 子进程中需要预加载的 node 模块
    "babel-loader",
    "sass-loader",
  ]
);
```

但是 thread-loader 有一些限制，即被 thread-loader 处理的 loader 会被放入一个独立的 worker 池，在 worker 池中运行的 loader 是受到限制的。例如：

- loader 不能生成新的文件。
- loader 不能使用自定义的 loader API（也就是说，不能通过插件来自定义）。
- loader 无法获取 webpack 的配置。

#### Parallel-Webpack


Parallel-Webpack其实是一个webpack的套壳升级版。因为Thread-loader、HappyPack 这类组件所提供的并行能力都仅作用于文件加载过程，对后续 AST 解析、依赖收集、打包、优化代码等过程均没有影响；而Parallel-Webpack则可以影响整个过程。具体来说，Parallel-Webpack实际上是创建了不同的独立进程来运行webpack

如果要使用，首先安装：
```
yarn add -D parallel-webpack
```

然后最重要的是，导出类型必须是一个数组，即多个webpack配置对象。这也就说明Parallel-Webpack本质上是为每个配置对象创建一个进程，让他们独立、并行构建，最后统一输出。如果只导出一个对象，优化效果就不是很明显。

```js
module.exports = [{
    entry: 'pageA.js',
    output: {
        path: './dist',
        filename: 'pageA.js'
    }
}, {
    entry: 'pageB.js',
    output: {
        path: './dist',
        filename: 'pageB.js'
    }
}];
```

#### 并行压缩

Webpack5使用Terser实现了多进程并行压缩能力。

TerserWebpackPlugin 插件默认已开启并行压缩，开发者也可以通过 parallel 参数（默认值为 require('os').cpus() - 1）设置具体的并发进程数量，如：

```js
const TerserPlugin = require("terser-webpack-plugin");

module.exports = {
    optimization: {
        minimize: true,
        minimizer: [new TerserPlugin({
            parallel: 2 // number | boolean
        })],
    },
};

```


### 缩小范围

#### 缩小解析范围

- 在 loader 中这一点体现为用 `include`、`exclude` 规定搜索范围从而减少搜索时间
- 设置 `resolve.modules` ，使用绝对路径指明第三方模块存放的位置，以减少搜索步骤
- 设置 `resolve.alias` 使复杂的路径减少搜索
- `resolve.extensions` 设置后缀减少无后缀文件搜索范围
- `resolve.mainFields` 只采用 main 字段作为入口文件描述字段 (减少搜索步骤，需要考虑到所有运行时依赖的第三方模块的入口文件描述字段)

resolve 设置:

```js
module.exports = {
  resolve: {
    // 使用绝对路径指明第三方模块存放的位置，以减少搜索步骤
    modules: [path.resolve(__dirname, "node_modules")],
    // 只采用 main 字段作为入口文件描述字段，以减少搜索步骤
    mainFields: ["main"],
    //使用 alias 把导入 react 的语句换成直接使用单独完整的 react.min.js 文件
    alias: {
      react: path.resolve(
        __dirname,
        "./node_modules/react/umd/react.production.min.js"
      ),
    },
    //尽可能的减少后缀尝试的可能性,相当于这一项配置尽可能少;比如如果只导入jsx文件就可以只填jsx,减少对于js/json等的匹配
    extensions: ["js"],
  },
};
```


#### 不解析部分模块

还可以设置noParse属性跳过文件编译。因为有不少 NPM 库已经提前做好打包处理，可以直接放在浏览器上运行，比如lodash；
可以在noParse属性中添加相应的路径，以跳过构建。这样在我们编写的代码中如果引入了该模块，那么就会跳过webpack对它的解析，而是直接使用该文件。
具体来说，noParse的作用是，忽视指定文件中的导入、导出，相当于不去解析这个文件中的导入导出，但不会影响loader、plugin的处理过程，也不会影响文件本身的ast构建等。
比如设置了lodash，那么就会忽视lodash/index.js中的所有静态、动态、cjs导入，只是解析index.js本身。

```js
// webpack.config.js
module.exports = {
  //...
  module: {
    noParse: /lodash|react/,
  },
};
```

举个例子，如果我们想用noParse跳过react的构建过程，就不能直接使用`'react'`这样的导入；因为这种实际上是`node_module/react/index.js` 文件，包含了模块导入语句 require。
而真正没有导入导出的文件是react.development.js 和 react.production.min.js。因此我们需要我们可以先找到适用的代码文件，然后用 resolve.alias 配置项重定向到该文件；这样在文件中引用react时，引用的实际上就是noParse的react.development.js或react.production.min.js文件，相当于跳过了react的构建。

```js
// webpack.config.js
module.exports = {
  // ...
  module: {
    noParse: /react|lodash/,
  },
  resolve: {
    alias: {
      react: path.join(
        __dirname,
        process.env.NODE_ENV === "production"
          ? "./node_modules/react/cjs/react.production.min.js"
          : "./node_modules/react/cjs/react.development.js"
      ),
    },
  },
};
```

---

除了noParse，还有一个externals属性也可以设置不处理部分模块。
区别在于，externals配置的实际上相当于完全不打包，即完全忽视掉指定的模块，转而让用户通过cdn或某些方式将其继续融入到用户依赖中去。而noParse实际上只是无视了模块内的依赖，还不至于完全不处理。

```js
module.exports = {
  //...
  externals: {
    jquery: 'jQuery',
  },
};

// html
<script
  src="https://code.jquery.com/jquery-3.1.0.js"
  integrity="sha256-slogkvB1K3VOkzAI8QITxV3VzpOnkeNVsKvtkYLMjfk="
  crossorigin="anonymous"
></script>
```
这里的意思就是，把jQuery模块


#### 最小化 watch 监控范围

在 watch 模式下（通过 npx webpack --watch 命令启动），Webpack 会持续监听项目目录中所有代码文件，发生变化时执行 rebuild 命令。

不过，通常情况下前端项目中部分资源并不会频繁更新，例如 node_modules ，此时可以设置 watchOptions.ignored 属性忽略这些文件，例如：

```js
// webpack.config.js
module.exports = {
  //...
  watchOptions: {
    ignored: /node_modules/
  },
};
```

### optimization部分配置

常见的构建速度优化，适用于开发环境

- optimization.removeAvailableModules: false，如果一个模块已经包含在所有父级模块中，就从当前chunk中检查出该模块并剔除。这个过程能减少构建大小，但是很耗时。应该设置为false
- optimization.removeEmptyChunks: false，移除空chunk，同样是设置为false可以加快
- optimization.splitChunks: false，关闭代码分割
- optimization.minimize: false，关闭代码压缩
- optimization.usedExports: false，如果为true则会检查每个chunk，未使用的导出内容不会被生成，有利于treeshaking这种清除无用代码的工具。

### preload & prefetch

动态导入的import函数可以通过添加注释来控制模块的preload和prefetch。实际上就是创建一个包含preload、prefetch属性的link标签并插入到html中，因此含义和在html中的相同。
使用方式：
```js
import(/* webpackPrefetch: true */ './path/to/LoginModal.js');
```
这会生成 `<link rel="prefetch" href="login-modal-chunk.js">` 并追加到页面头部

关于两者的区别：
- preload chunk 会在父 chunk 加载时，以并行方式开始加载。prefetch chunk 会在父 chunk 加载结束后开始加载。
- preload chunk 具有中等优先级，并立即下载。prefetch chunk 在浏览器闲置时下载。
- preload chunk 会在父 chunk 中立即请求，用于当下时刻。prefetch chunk 会用于未来的某个时刻。

## 产物优化

产物优化主要针对于生产环境的构建，即通过压缩、剔除冗余代码的形式，减小打包结果，尽可能降低生产环境产物的体积。
需要注意的是，这些优化方式虽然减少体积，但是对开发环境并不是很好，因为这些功能都要花费大量的时间。在开发环境下，反而应该去掉Tree-Shaking、SplitChunks、Minimizer等压缩类插件，以及关闭一些optimization的配置项。

### 压缩代码

#### 压缩 html

现代 Web 应用大多会选择使用 React、Vue 等 MVVM 框架，这衍生出来的一个副作用是原生 HTML 的开发需求越来越少，HTML 代码占比越来越低，所以大多数现代 Web 项目中其实并不需要考虑为 HTML 配置代码压缩工作流。
不过凡事都有例外，某些场景如 SSG 或官网一类偏静态的应用中就存在大量可被优化的 HTML 代码

html-minifier-terser就是一个压缩html的工具。需要借助借助 html-minimizer-webpack-plugin 插件接入 html-minifier-terser

参考https://webpack.js.org/plugins/html-minimizer-webpack-plugin/

```js
const HtmlWebpackPlugin = require("html-webpack-plugin");
const HtmlMinimizerPlugin = require("html-minimizer-webpack-plugin");

module.exports = {
  // ...
  optimization: {
    minimize: true,
    minimizer: [
      // Webpack5 之后，约定使用 `'...'` 字面量保留默认 `minimizer` 配置
      "...",
      new HtmlMinimizerPlugin({
        minimizerOptions: {
          // 折叠 Boolean 型属性
          collapseBooleanAttributes: true,
          // 使用精简 `doctype` 定义
          useShortDoctype: true,
          // ...
        },
      }),
    ],
  },
};
```



#### 压缩 js

主要使用 TerserWebpackPlugin 来压缩 JavaScript；webpack5 自带最新的 terser-webpack-plugin，无需手动安装。

主要通过 optimization 选项开启。多数情况下使用默认 Terser 配置即可，也可以向TerserPlugin传入更详细的配置

```js
module.exports = {
  optimization: {
    minimize: true,
    minimizer: [new TerserPlugin({
      terserOptions: {
          compress: {
            reduce_vars: true,
            pure_funcs: ["console.log"],
          },
      }
    })],
  },
};
```

压缩之后的 js 会变得非常简单，大量变量名、函数名被重新命名，并且会合并到同一行

#### 压缩 css

可以通过 CssMinimizerWebpackPlugin 压缩。
实际上包括了两个 plugin，分别是 MiniCssExtractPlugin 和 CssMinimizerPlugin；使用时要注意配置。后者应该在 optimization 中开启

同样是在 optimization 选项中配置。安装后配置如下：

- 在 plugins 用引入`MiniCssExtractPlugin`
- 在 loader 中启用`MiniCssExtractPlugin.loader`，就是它自带的一个 loader，放置在所有 cssloader 最前面，并且代替style-loader的位置
- 在 minimizer 中引入`CssMinimizerPlugin`即可。

```js
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const CssMinimizerPlugin = require("css-minimizer-webpack-plugin");

module.exports = {
  module: {
    rules: [
      {
        test: /.s?css$/,
        use: [MiniCssExtractPlugin.loader, "css-loader", "sass-loader"],
      },
    ],
  },
  optimization: {
    minimizer: [
      // 在 webpack@5 中，你可以使用 `...` 语法来扩展现有的 minimizer（即 `terser-webpack-plugin`），将下一行取消注释
      // `...`,
      new CssMinimizerPlugin(),
    ],
  },
  plugins: [new MiniCssExtractPlugin()],
};
```

当然，只是用 CssMinimizerPlugin 也是可以的：

```js
const CssMinimizerPlugin = require("css-minimizer-webpack-plugin");

module.exports = {
  optimization: {
    minimizer: [
      new CssMinimizerPlugin({
        parallel: 4,
      }),
    ],
  },
};
```

### tree-shaking

#### 效果

`Tree Shaking` 可以用来剔除 JavaScript 中用不上的死代码，确切来说是没有使用的引用代码。

它依赖静态的 ES6 模块化语法，例如通过 import 和 export 导入导出，判断导入的对象中是否有没有使用的，如果有就不加载

Treeshaking 的主要优化对象是**导出变量中没有被导入的部分**。它可以将未被使用的导出变量标记为 unused 同时在将其重新导出的模块中不再 export。注意 treeshaking 并不会直接剔除死代码，它的主要工作是管理导出对象，将没有被导入的导出变量改为不导出。

比如

```js
// test.js
export function add(a, b) {
  return a + b;
}
export function sqrt(a) {
  return Math.sqrt(a);
}
// index.js
import { add } from "./test";
console.log(add(3, 4));
```

这段代码打包之后:
![](https://pic.imgdb.cn/item/637e22c016f2c2beb16d8af9.jpg)

可以看到有一行注释“unused harmony export sqrt”，表示 sqrt 这个函数是没有被使用的导出。因此在导出中就不包含这个函数。
不过这时函数的生命依然还存在，但他已成为了“未使用”的变量。在之后的terser压缩中，将会删去这种未使用的变量。

假如没有开启 tree-shaking，这里的导出对象则会包含 sqrt 函数，就像这样：
![](https://pic.imgdb.cn/item/637e243b16f2c2beb1700e7c.jpg)

因此，treeshaking 的优化并不是直接把没用的导出对象从打包结果中删去，而是将它从导出对象中删去

#### 原理

tree-shaking不是某个插件或loader，而是webpack内部的一种机制，因此tree-shaking的原理和webpack的工作流程分不开。
Tree-Shaking 的实现大致上可以分为三个步骤：

1. make阶段，「收集」 模块导出变量并记录到模块依赖关系图 ModuleGraph 对象中；
1. seal阶段，遍历所有模块，「标记」 模块导出变量有没有被使用；
1. 使用代码优化插件 —— 如 Terser，删除无效导出代码。

##### make阶段

1. 将模块的所有 ESM 导出语句转换为 Dependency 对象，并记录到 module 对象的 dependencies 集合，转换规则：
  - 具名导出转换为 HarmonyExportSpecifierDependency 对象；
  - default 导出转换为 HarmonyExportExpressionDependency 对象。

例如对于下面的模块：
```js
export const bar = 'bar';
export const foo = 'foo';

export default 'foo-bar'
```
对应的dependencies 值为：

![](https://pic.imgdb.cn/item/63e32a084757feff33c82a78.jpg)

2. 所有模块都编译完毕后，触发 compilation.hooks.finishModules 钩子，开始执行 FlagDependencyExportsPlugin 插件回调；
3. FlagDependencyExportsPlugin 插件 遍历 所有 module 对象；
4. 遍历 module 对象的 dependencies 数组，找到所有 HarmonyExportXXXDependency 类型的依赖对象，将其转换为 ExportInfo 对象并记录到 ModuleGraph 对象中。

这一步结束之后，moduleGraph中就包含了各个模块的导出情况

##### seal阶段

接下来，Webpack 需要再次遍历所有模块，逐一标记出模块导出列表中，哪些导出值有被其它模块用到，哪些没有。然后在seal阶段最后的生成代码中，针对两种不同的导出值生成不同的语句
这个过程主要发生在 FlagDependencyUsagePlugin 插件中：

1. 触发 compilation.hooks.optimizeDependencies 钩子，执行 FlagDependencyUsagePlugin 插件回调；
2. 在 FlagDependencyUsagePlugin 插件中，遍历每一个 module 对象的 exportInfo 数组，为每一个 exportInfo 确定其对应的 dependency 对象有否被其它模块使用；如果导出被其他模块使用，就将其标记为已被使用；
3. 修改 exportInfo._usedInRuntime 属性，记录该导出被如何使用。执行完毕后，Webpack 会将所有导出语句的使用状况记录到 exportInfo._usedInRuntime 字典中。

然后到了代码生成阶段，即执行compilation.codeGeneration 函数生成最终代码的时候。这部分具体逻辑由导出语句对应的 HarmonyExportXXXDependency 类实现。简单说，这一步的逻辑就是，用前面收集好的 exportsInfo 对象为模块的导出值分别生成导出语句。具体流程为：

1. compilation.codeGeneration 中会逐一遍历模块的 dependencies ，并调用 HarmonyExportXXXDependency.Template.apply 方法生成导出语句代码；
2. 在 apply 方法内，读取 ModuleGraph 中存储的 exportsInfo 信息，判断哪些导出值被使用，哪些未被使用；
3. 对已经被使用及未被使用的导出值，分别创建对应的 HarmonyExportInitFragment 对象，保存到 initFragments 数组；
4. 遍历 initFragments 数组，生成最终结果。

#### 配置

启动 Tree Shaking 功能必须同时满足两个条件：

- 配置 optimization.usedExports 为 true，标记模块导入导出列表；
- 启动代码优化功能，可以通过如下方式实现：
  - 配置 mode = production
  - 配置 optimization.minimize = true
  - 提供 optimization.minimizer 数组

usedExports 字段表示告知 webpack 去决定每个模块使用的导出内容，也就是说让 webpack 自己取判断哪些导出被使用了，哪些没有被用到，没有被用到的导出就不会被引用。
比如：
```js
module.exports = {
  mode: "production",
  optimization: {
    usedExports: true,
  },
};
```

#### 实践

tree-shaking虽然可以保证在大多数情况下实现shaking效果，但是还有部分情况可能会影响效果，在实践中应该注意这些情况

1. 无意义赋值，导致导入模块没有被使用，只是单纯的赋值

比如：
```js
// bar.js
export const bar = 'bar'
export const foo = 'foo'

// index.js
import {bar,foo} from './bar.js'
const f = foo
```
这里的f变量虽然引用了foo导出，但是f实际上也没有被使用。这种情况下treeshaking不会把foo看作是没使用的导出，因此tree-shaking就不会剔除foo的导出。

2. 标记纯函数或副作用

默认情况下 Webpack 并不会对函数调用做 Tree Shaking 操作，因为函数可能产生副作用，因此tree-shaking不会删去无用的函数导出。
如果希望删去函数，就需要指明这个函数没有副作用，可以放心删除。
方法有两个
- 用sideEffects标记，详见下
- 在调用函数的语句前面加上注释`/*#__PURE__*/`，明确告诉 Webpack 该次函数调用并不会对上下文环境产生副作用

![](https://pic.imgdb.cn/item/63e32f4c4757feff33d16e4f.jpg)

3. 禁止babel编译esm语句。需要将`@babel/preset-env`的配置中的modules属性改为false
```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "modules": false
      }
    ]
  ]
}
```



#### sideEffects

后来添加了一个 sideEffects 的配置，用于处理导出中包含副作用的情况。所谓副作用其实就是当执行某个模块时，该模块除了返回导出值之外，还会产生一些额外影响，比如修改全局作用域下的变量等。
副作用的 treeshaking 可能包含一些问题，最常见的就是包含副作用的模块可能不会被“摇干净”，一些没用的导出依然会被打包。之所以这么做是因为 webpack 保守策略，即使被导入的文件没有使用，webpack 也不会剔除，因为有可能存在副作用。如果盲目剔除，可能开发者想要达到的副作用效果（比如修改原型链）就达不到了。
因此开发者需要手动指明哪些文件包含副作用，哪些文件不包含，可以让 webpack 放心剔除；在 packages.json 中开启一个选项：

```json
{
  "sideEffects": false
}
```

如果所有代码都不包含副作用，我们就可以简单地将该属性标记为 false，来告知 webpack 它可以安全地删除未用到的 export。
相反，如果有一些代码有副作用，就需要在这里指明具体的路径

```json
{
  "sideEffects": ["./src/some-side-effectful-file.js"]
}
```

**注意：css、less、sass 等样式文件，以及 polyfill 等都是包含副作用的文件，需要在配置项中排除掉，否则可能导致样式丢失的问题。**
像这样，排除 css 文件：

```json
{
  "sideEffects": ["**/*.css"]
}
```

### SplitChunksPlugin

SplitChunksPlugin 是 Webpack 4 之后内置实现的最新分包方案，它能够基于一些更灵活、合理的启发式规则将 Module 编排进不同的 Chunk，最终构建出性能更佳，缓存更友好的应用产物。

SplitChunksPlugin是为了解决chunk构建的问题的。chunk是module对象的集合，通常是module的构建结果。从模块类型可以将chunk类型分为三种：

- Initial Chunk：entry 模块及相应子模块打包成 Initial Chunk；
- Async Chunk：通过 import('./xx') 等语句导入的异步模块及相应子模块组成的 Async Chunk；
- Runtime Chunk：运行时代码抽离成 Runtime Chunk，即设置 entry.runtime 时就会生成

Initial Chunk 与 Async Chunk 这种略显粗暴的规则会带来两个明显问题：
1. 模块重复打包：
假如多个 Chunk 同时依赖同一个 Module，那么这个 Module 会被不受限制地重复打包进这些 Chunk，这样对于多个chunk来说，每个都包含了相同的module，就会造成体积增大
![](https://pic.imgdb.cn/item/63e103c24757feff33a5310e.jpg)

2. 所有模块都被打入同一个包。
Async Chunk默认会和Initial Chunk打包在一起，如果没有多个入口的话，那么最终的chunk可能只有一个。对于一个庞大的项目来说，打包在一起是很致命的：
- 资源冗余：客户端必须等待整个应用的代码包都加载完毕才能启动运行，但可能用户当下访问的内容只需要使用其中一部分代码
- 缓存失效：将所有资源达成一个包后，所有改动 —— 即使只是修改了一个字符，客户端都需要重新下载整个代码包，缓存命中率极低

所以需要一些分包策略，来解决上面的问题。方式就是直接修改 optimization.splitChunks 配置项即可实现自定义的分包策略：

```js
module.exports = {
  //...
  optimization: {
    splitChunks: {
      // ...
    },
  },
}
```


#### 设置分包范围

SplitChunksPlugin 默认情况下只对 Async Chunk 生效，我们可以通过 splitChunks.chunks 调整作用范围
- `'all'` ：对 Initial Chunk 与 Async Chunk 都生效，建议优先使用该值；
- `'initial'` ：只对 Initial Chunk 生效；
- `'async'` ：只对 Async Chunk 生效；
- 函数 `(chunk) => boolean` ：该函数返回 true 时生效；

#### 根据 Module 使用频率分包

SplitChunksPlugin 支持按 Module 被 Chunk 引用的次数决定是否分包，借助这种能力我们可以轻易将那些被频繁使用的模块打包成独立文件，减少代码重复。

```js
module.exports = {
  //...
  optimization: {
    splitChunks: {
      // 设定引用次数超过 2 的模块才进行分包
      minChunks: 2
    },
  },
}
```

#### 限制分包体积

Webpack 提供了一系列与 Chunk 大小有关的分包判定规则，借助这些规则我们可以实现当包体过小时直接取消分包，防止产物过"碎"；
当包体过大时尝试对 Chunk 再做拆解，避免单个 Chunk 过大。

这一规则相关的配置项有：

- minSize： 超过这个尺寸的 Chunk 才会正式被分包；
- maxSize： 超过这个尺寸的 Chunk 会尝试进一步拆分出更小的 Chunk；
- maxAsyncSize： 与 maxSize 功能类似，但只对异步引入的模块生效；
- maxInitialSize： 与 maxSize 类似，但只对 entry 配置的入口模块生效；
- enforceSizeThreshold： 超过这个尺寸的 Chunk 会被强制分包，忽略上述其它 Size 限制。

### 动态导入

即 webpack 的代码分离。原理是利用 ES6 的动态导入实现按需加载，而不是任何情况都在文件头部静态导入
如果通过import()动态导入一个模块，那么在构建期间就会产生额外的产物文件，即单独生成动态导入的模块的文件。
比如

```js
import(
  /* webpackChunkName: "my-chunk-name" */
  /* webpackMode: "lazy" */
  /* webpackExports: ["default", "named"] */
  "./sth.js"
).then((module) => {
  //...
});
```

`/* webpackChunkName: "my-chunk-name" */`的写法指定了 webpack 对于动态导入代码的设定。当 Webpack 遇到了类似的语句时会这样处理：

1. 以 `./sth.js` 为入口新生成一个 Chunk；
1. 当代码执行到 `import` 所在语句时才会去加载由 Chunk 对应生成的文件。
1. `import` 返回一个 `Promise`，当文件加载成功时可以在 `Promise` 的 `then` 方法中获取到 `sth.js` 导出的内容。

这段代码会在点击按钮时才会为 show 文件创建一个名字为 show 的 bundle；因此还需要配置：

```js
module.exports = {
  entry: {
    main: "./main.js",
  },
  output: {
    filename: "[name].js", // 本来的chunk
    chunkFilename: "[name].js", // 动态加载的文件的chunk
  },
};
```

另外，动态导入不宜过多。
- 过度使用会使产物变得过度细碎，产物文件过多，运行时 HTTP 通讯次数也会变多
- 使用时 Webpack 需要在客户端注入一大段用于支持动态加载特性的 Runtime，对于较小的模块反而会增大代码量，得不偿失


## http优化

Webpack 只是一个工程化构建工具，没有能力决定应用最终在网络分发时的缓存规则，但我们可以调整产物文件的名称(通过 Hash)与内容(通过 Code Splitting)，使其更适配 HTTP 持久化缓存策略。
具体来说，output配置项可以为导出的文件配置文件名，其中可以设置为hash值。包括：

- `[fullhash]`：整个项目的内容 Hash 值，项目中任意模块变化都会产生新的 fullhash；
- `[chunkhash]`：产物对应 Chunk 的 Hash，Chunk 中任意模块变化都会产生新的 chunkhash；
- `[contenthash]`：产物内容 Hash 值，仅当产物内容发生变化时才会产生新的 contenthash，因此实用性较高。

每个产物文件名都会带上一段由产物内容计算出的唯一 Hash 值，文件内容不变，Hash 也不会变化，这就很适合用作 HTTP 持久缓存 资源。一直到文件内容发生变化，引起 Hash 变化生成不同 URL 路径之后，才需要请求新的资源文件，能有效提升网络性能
通过这种方式，就可以放心的给产物文件的响应打上最长时间的max-age，保证缓存的最长可用。


# 常用 plugin 和 loader

## loader

1. `file-loader`：把文件输出到一个文件夹中，在代码中通过相对 URL 去引用输出的文件 (处理图片和字体)
2. `url-loader`：与 file-loader 类似，区别是用户可以设置一个阈值，大于阈值会交给 file-loader 处理，小于阈值时返回文件 base64 形式编码 (处理图片和字体)
3. `css-loader`：加载 CSS，把 css 按照字符串的形式编译，样式字符串放入模块数组中
4. `style-loader`：把 CSS 插入到 DOM 中，就是处理 css-loader 导出的模块数组，然后将样式通过 style 标签或者其他形式插入到 DOM 中；

> `css-loader`借助 postcss-value-parser 解析 CSS 为 AST，并将 CSS 中的 url() 与 @import 解析为模块。它会把原生的 css 编译成一个数组，大概是这个样子：
> ![](https://pic.imgdb.cn/item/6240272827f86abb2ab1732b.jpg)
>
> `style-loader`将`css-loader`解析后的内容挂载到 html 页面当中。实际上最简单的 style-loader 就是通过 createElement 创建一个 style 标签，然后插入 css。当然这种借助 dom api 插入样式的形式优化会很差
>
> ```js
> module.exports = function (source) {
>   return `
> function injectCss(css) {
>     const style = document.createElement('style')
>     style.appendChild(document.createTextNode(css))
>     document.head.appendChild(style)
> }
> 
> injectCss(\`${source}\`)
> `;
> };
> ```
>
> 因此`style-loader`要放在`css-loader`的左边，即先转化为字符串，然后再绑定到 dom 上。
> 如果有预处理器，就还要放在`css-loader`的右边，应该让预处理器先处理特殊格式的文件，然后再进行解析等操作。

5. `json-loader`: 加载 JSON 文件（默认包含）
6. `babel-loader`：把 ES6 转换成 ES5
7. `ts-loader`: 将 TypeScript 转换成 JavaScript
8. `less-loader`：将 less 代码转换成 CSS
9. `eslint-loader`：通过 ESLint 检查 JavaScript 代码
10. `vue-loader`: 加载 Vue 单文件组件

## plugin

1. `html-webpack-plugin`：根据模板页面生成打包的 html 页面，建议使用这个，极大减少因为 html 文件位置不对造成的 devServer404 问题；关于该 plugin 的配置可以看这篇文章：https://juejin.cn/post/6844903853708541959
2. `uglifyjs-webpack-plugin`：对 js 进行压缩，但不支持 ES6 压缩 ( Webpack4 以前)
3. `mini-css-extract-plugin`: 分离样式文件，CSS 提取为独立文件，支持按需加载
4. `clean-webpack-plugin`: 目录清理，webpack5 不需要插入，在 output 声明`clean:true`即可
5. `copy-webpack-plugin`: 拷贝文件
6. `webpack-bundle-analyzer`: 可视化 Webpack 输出文件的体积 (业务组件、依赖第三方模块)

# 工作原理

webpack集成了很多功能，包括模块打包、代码分割、按需加载、devserver等等，但是最核心的部分还是它的静态模块打包能力。即，Webpack 能够将各种类型的资源 —— 包括图片、音视频、CSS、JavaScript 代码等，通通转译、组合、拼接、生成标准的、能够在不同版本浏览器兼容执行的 JavaScript 代码文件，这一特性能够轻易抹平开发 Web 应用时处理不同资源的逻辑差异，使得开发者以一致的心智模型开发、消费这些不同的资源文件

## 流程

运行流程可以大致分为三步：
![](https://pic.imgdb.cn/item/63e1c02e4757feff33a92500.jpg)

Webpack 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：

1. 初始化阶段：修整配置参数，创建 Compiler、Compilation 等基础对象，并初始化插件及若干内置工厂、工具类，并最终根据 entry 配置，找到所有入口模块；
2. 构建阶段：从 entry 文件开始，调用 loader 将模块转译为 JavaScript 代码，调用 Acorn 将代码转换为 AST 结构，遍历 AST 从中找出该模块依赖的模块；之后 递归 遍历所有依赖模块，找出依赖的依赖，直至遍历所有项目资源后，构建出完整的 模块依赖关系图；
3. 生成阶段：根据 entry 配置，将模块组装为一个个 Chunk 对象，之后调用一系列 Template 工厂类翻译 Chunk 代码并封装为 Asset，最后写出到文件系统。


单次构建过程自上而下按顺序执行，如果启动了 watch ，则构建完成后不会退出 Webpack 进程，而是持续监听文件内容，发生变化时回到「构建」阶段重新执行构建。
三个阶段环环相扣，「初始化」的重点是根据用户配置设置好构建环境；「构建阶段」则重在解读文件输入与文件依赖关系；最后在「生成阶段」按规则组织、包装模块，并翻译为适合能够直接运行的产物包。

### 几个关键对象

webpack有几个基础对象贯穿工作流程始终，包括：
- Entry：编译入口。初始化是从配置中提取到的entry对象
- Compiler：编译管理器，Webpack 启动后会创建 compiler 对象，该对象一直存活直到构建结束进程退出；compiler对象包含非常多的内容，包括：
  - 各种hook
  - 整个编译过程的各种参数，比如配置options
  - 编译要用到的方法，最主要的是compile方法，还有run、emitAssets等方法
- Compilation：单次构建过程的管理器，主要用于构建过程，内容也主要是供构建过程使用的参数和方法，比如：
  - addModuleTree方法将文件转化为module并建立moduleTree
  - addEntry方法
  每次调用compile方法都会创建一个compilation对象，并且如果开启文件监听，那么每次文件刷新都会再创建compilation对象。
- Dependence：依赖对象，记录模块间依赖关系；在进入构建阶段时的EntryPlugin就会从entry解析依赖关系，创建dependence对象；后续的构建阶段就会根据这个对象确定的依赖关系来递归把文件解析为module。
- Module：Webpack 内部所有资源都会以 Module 对象形式存在，所有关于资源的操作、转译、合并都是以 Module 为单位进行的；
- Chunk：编译完成准备输出时，将 Module 按特定的规则组织成一个一个的 Chunk。
- ChunkGraph：所有关于module如何与chunk连接的信息都存储在 ChunkGraph 类中
- ChunkGroup：一个 ChunkGroup 内包含一个或多个 Chunk 对象
- ModuleGraph：所有关于module如何在相互连接的信息都存储在 ModuleGraph 类中。

graph类型的数据都是为了更好的显示关系的。在webpack5之前，module之间的关系都是直接使用module和dependency来得出的，而webpack5则对这种关系做了加深，通过图的形式使得module之间、module和chunk之间的关系更加清晰

#### ModuleGraph

参考https://juejin.cn/post/7138285996500025352

moduleGraph实际上是几个module之间的依赖关系的记录的数据结构的集合。具体来说，主要包含了两个重要属性：
- _dependencyMap ：记录入口dependency与module连接关系的信息。注意这个是入口文件的dependency和入口module的关系信息，它只会记录当前module和一个引用当前module的module
```js
{
	module:  Module, // 当前module
	originModule： Module// 引用当前module的module
}

_dependencyMap:{
    <dep-index, connection{originModule: undefined, module: mod-index}>,
}
```
- _moduleMap ：记录当前module被谁引用以及引用了谁。moduleMap内的数据结构如下
```js
{
	inComingConnections:[], // 表示一个有哪些modules引用了当前module
	outComingConnections:[], // 表示一个当前module引用了哪些modules
}
```

当moduleGraph被确定后，所有module之间的依赖关系、导入导出值都可以从其中直接获得。

#### ChunkGraph

参考 https://juejin.cn/post/7141067021734641671

ChunkGraph就是以chunk为中心描绘chunk与module关系对象，可以理解为webpack的分包规则。
ChunkGraph为例记录chunk和module的关系，就必然有这样的数据结构：
- 有哪些chunk，chunk里面有哪些module
- 有哪些module，module属于哪些chunk

而实际上ChunkGraph内部也正是两类数据结构：
- _chunks：`Map<chunk, ChunkGraphChunk>`，ChunkGraphChunk是记录一个chunk有哪些module 
- _modules：`Map<modules, ChunkGraphModule>，`ChunkGraphModule是记录一个module属于哪些chunk。

在seal阶段会初始化并构建完整的ChunkGraph，后续的SpiltChunkPlugin等分包手段也依赖于它。

### 初始化阶段

初始化阶段的主要功能是整理合并参数，然后根据参数创建compiler对象，并开始编译。

![](https://pic.imgdb.cn/item/63e1c1774757feff33ab514f.jpg)

在源码中，实际上是webpack.js中的webpack函数，大致如下：https://github1s.com/webpack/webpack/blob/HEAD/lib/webpack.js

```js
const webpack = (options, callback) => {
	const create = () => {
    // 初始化compiler
		let compiler;
		let watch = false;
		let watchOptions;
		if (Array.isArray(options)) {
      // 如果配置对象是数组，就创建多个compiler
			compiler = createMultiCompiler(
				options,
				options
			);
			watch = options.some(options => options.watch);
			watchOptions = options.map(options => options.watchOptions || {});
		} else {
      // 正常情况，创建compiler，传入配置对象，初始化watch对象
			const webpackOptions = options
			compiler = createCompiler(webpackOptions);
			watch = webpackOptions.watch;
			watchOptions = webpackOptions.watchOptions || {};
		}
		return { compiler, watch, watchOptions };
	};
	if (callback) {
		try {
      // 调用create方法创建这几个对象
			const { compiler, watch, watchOptions } = create();
			if (watch) {
        // 开启监听
				compiler.watch(watchOptions, callback);
			} else {
        // 调用run方法
				compiler.run((err, stats) => {
					compiler.close(err2 => {
						callback(err || err2, stats);
					});
				});
			}
			return compiler;
		} catch (err) {
			process.nextTick(() => callback(err));
			return null;
		}
	} else {
		const { compiler, watch } = create();
		return compiler;
	}
}
```

而createCompiler函数功能如下：
1. 收集各种配置。除了配置对象之外，还包括webpack的默认配置、命令行配置等
2. 遍历 配置中的 plugins 集合，执行插件的 apply 方法。
3. 调用 new WebpackOptionsApply().process 方法，根据配置内容动态注入相应插件

```js
const createCompiler = rawOptions => {
  // 默认配置
	const options = getNormalizedWebpackOptions(rawOptions);
	applyWebpackOptionsBaseDefaults(options);
	const compiler = new Compiler(options.context, options);
  //  注入环境变量
	new NodeEnvironmentPlugin({
		infrastructureLogging: options.infrastructureLogging
	}).apply(compiler);
  // 遍历执行plugins
	if (Array.isArray(options.plugins)) {
		for (const plugin of options.plugins) {
			if (typeof plugin === "function") {
        plugin.call(compiler, compiler);
			} else {
        // plugins大多数是类，因此调用上面的apply方法
				plugin.apply(compiler);
			}
		}
	}
	applyWebpackOptionsDefaults(options);
  // 调用environment和afterEnvironment两个hook，分别表示创建环境和创建环境完成
	compiler.hooks.environment.call();
	compiler.hooks.afterEnvironment.call();
  // 这一步是根据配置内容动态注入相应插件
	new WebpackOptionsApply().process(options, compiler);
	compiler.hooks.initialize.call();
	return compiler;
};
```

最后，调用 compiler.compile 方法开始执行构建。
compile方法没有具体逻辑，只是不断调用hook上的call方法执行回调，但他确定了整个编译过程的流程，使得编译过程一环套一环，过程如下：

1. 调用 newCompilation 方法创建 compilation 对象；
1. 触发 make 钩子，紧接着 EntryPlugin 在这个钩子中调用 compilation 对象的 addEntry 方法创建入口模块，主流程开始进入「构建阶段」；
1. make 执行完毕后，触发 finishMake 钩子；
1. 执行 compilation.seal 函数，进入「生成阶段」，开始封装 Chunk，生成产物；
1. seal 函数结束后，触发 afterCompile 钩子，开始执行收尾逻辑。

从触发make hook开始，就进入了构建阶段（make），初始化阶段完成

```js
compile(callback) {
    const params = this.newCompilationParams();
    this.hooks.beforeCompile.callAsync(params, err => {
      // ...
      const compilation = this.newCompilation(params);
      this.hooks.make.callAsync(compilation, err => {
        // ...
        this.hooks.finishMake.callAsync(compilation, err => {
          // ...
          process.nextTick(() => {
            compilation.finish(err => {
              // ...
              compilation.seal(err => {
                // ...
                this.hooks.afterCompile.callAsync(compilation, err => {
                    if (err) return callback(err);
                    return callback(null, compilation);
                });
              });
            });
          });
        });
      });
    });
  }
```


### 构建阶段

构建阶段从 entry 模块开始递归解析模块内容、找出模块依赖，按图索骥逐步构建出项目整体 module 集合以及 module 之间的 依赖关系图，这个阶段的主要作用就是读入并理解所有原始代码。
webpack内部一个plugin，EntryPlugin会在make hook被触发时调用，它的主要做用是调用addEntry函数找到入口文件，然后解析入口文件的导入，得到入口文件的依赖。也就是从入口文件开始了构建过程

```js
class EntryPlugin {
    apply(compiler) {
        const { entry, options, context } = this;
        // 解析entry对象，创建dependence对象
        const dep = EntryPlugin.createDependency(entry, options);

        compiler.hooks.make.tapAsync("EntryPlugin", (compilation, callback) => {
            compilation.addEntry(context, dep, options, err => {
                callback(err);
            });
        });
    }
}
```
addEntry内部会调用addModuleTree方法，它会创建并生成模块的依赖树。对每个依赖来说，都是调用 handleModuleCreation，根据文件类型构建 module 子类 —— 一般是 NormalModule
```js
addModuleTree({ context, dependency, contextInfo }, callback) {
		const Dep = dependency.constructor
		const moduleFactory = this.dependencyFactories.get(Dep);

		this.handleModuleCreation(
			{
				factory: moduleFactory,
				dependencies: [dependency],
				originModule: null,
				contextInfo,
				context
			},
			(err, result) => {
				if (err && this.bail) {
					callback(err);
					this.buildQueue.stop();
					this.rebuildQueue.stop();
					this.processDependenciesQueue.stop();
					this.factorizeQueue.stop();
				} else if (!err && result) {
					callback(null, result);
				} else {
					callback();
				}
			}
		);
	}
```

接下来就是调用一些外部库进行转化，比如：
1. 调用 loader-runner 转译 module 内容，将各类资源类型转译为 Webpack 能够理解的标准 JavaScript 文本；
1. 调用 acorn 将 JavaScript 代码解析为 AST 结构；

> acorn 需要将各种类型的模块内容（js、css、静态资源）解析为 AST 结构，要使用 loaders 将不同类型的资源转译为标准 JavaScript 代码，才能转化为ast。

当得到入口js代码的ast之后，就是最关键的一步，即解析ast，得到其中的模块依赖关系。具体来说是：
1. 解析ast，解析过程中如果检查到了导入语句，就触发相关hook。HarmonyExportDependencyParserPlugin插件监听到AST解析钩子exportImportSpecifier则会回调module.addDependency()将依赖对象添加到module的依赖列表dependencies
2. 遍历依赖列表dependencies，把每个Dependency 转换为 Module 对象。
3. 对于每个module对象，就再递归处理（执行上面的两步），直到所有文件都被处理完毕。

最后，所有的文件都被构建成module。其中第三步的递归，就是构建module的核心。注意这个过程中module和dependences数组的关系没有断掉，一个module会有它的dependences数组，表示它依赖的模块的module；最后会形成module tree这样的结构。
在下一步生成chunk的过程中，会利用到module和dependency的关系，将对应的module关联起来。

具体递归过程可以参考https://juejin.cn/book/7115598540721618944/section/7119035873802813475


### 生成阶段

当构建阶段完成后，所有的文件都变成了module；生成阶段则负责根据一系列内置规则，将上一步构建出的所有 Module 对象拆分编排进若干 Chunk 对象中，之后以 Chunk 粒度将源码转译为适合在目标环境运行的产物形态，并写出为产物文件

![](https://pic.imgdb.cn/item/63e264334757feff33c6d787.jpg)


生成阶段的名称是seal，因此生成阶段的入口函数就是`compilation.seal`方法。这一步的代码很复杂，我们只关注流程：
1. 创建本次构建的 ChunkGraph 对象。ChunkGraph是一个记录chunk和module关系的数据结构
2. 遍历 入口集合 compilation.entries：
  1. 调用addChunk 方法为每一个入口 创建 对应的 Chunk 对象（EntryPoint Chunk）；
  2. 遍历该入口module对应的 Dependency 集合，找到相应 Module 对象并关联到该 Chunk。
3. 到这里可以得到若干 Chunk，之后调用 buildChunkGraph 方法将这些 Chunk 处理成 Graph 结构，方便后续处理。
4. 之后，触发 optimizeModules/optimizeChunks 等钩子，由插件（如 SplitChunksPlugin）进一步修剪、优化 Chunk 结构。
5. 一直到最后一个 Optimize 钩子 optimizeChunkModules 执行完毕后，开始调用 compilation.codeGeneration 方法生成 Chunk 代码

这一步完成之后，其实已经结束了seal阶段。seal阶段的结果就是chunk的生成；接下来，需要将chunk变为实质输出的代码，将其写入文件系统中。这一步又被称为emit阶段。

6. 在codeGeneration回调中调用 createChunkAssets 函数，为每一个 Chunk 生成assets文件。这时还是一种“资源”类型的文件，还不是真正的代码文件。每个chunk都有对应的asset
7. 调用 compilation.emitAssets 函数“提交”文件，触发 callback 回调，控制流回到 compiler 函数。
8. 最后调用 compiler 对象的 emitAssets 方法，将需要生成的文件代码写入文件系统。到这一步之后，就可以看到webpack的输出结果了。

---


seal阶段的过程从chunk角度来说可以这样梳理：
![](https://pic.imgdb.cn/item/63e49d6c4757feff330dcd09.jpg)
1. 创建入口模块和初始化chunkgraph：调用 seal() 函数后，遍历 entry 配置，为每个入口创建一个空的 Chunk 与 EntryPoint 对象（一种特殊的 ChunkGroup），并初步设置好基本的 ChunkGraph 结构关系，在之后会逐渐填充chunkgraph内容。
这一步主要是将entry的几个入口模块生成chunk，并放入entryPoint中，形成这样的结构：
![](https://pic.imgdb.cn/item/63e49db54757feff330e651f.jpg)
若此时配置了 entry.runtime，Webpack 还会在这个阶段为运行时代码 创建相应的 Chunk 并直接注入到 entry 对应的 ChunkGroup对象。一切准备就绪后调用 buildChunkGraph 函数，进入下一步骤。
2. 把所有module生成chunk并装入：在 buildChunkGraph 函数内遍历 ModuleGraph，将所有 Module 按照依赖关系分配给不同 Chunk 对象；这个过程中若遇到**异步模块**，则为该模块创建新的 ChunkGroup 与 Chunk 对象，形成如下结构：
![](https://pic.imgdb.cn/item/63e49e244757feff330f3da6.jpg)
3. 完善chunkgraph对象：在 buildChunkGraph 函数中调用 connectChunkGroups 方法，建立 ChunkGroup 之间、Chunk 之间的依赖关系，生成完整的 ChunkGraph 对象


## loader

loader 本质是一个函数，类似 nodejs 的中间件
最简单的 loader 长这样：

```js
const sass = require("node-sass");

module.exports = function (source) {
  // source 为 compiler 传递给 Loader 的一个文件的原内容
  return sass(source);
};
```

loader的执行对象是经历过make阶段之后生成的module。loader可以通过配置文件后缀确定接收哪类文件的module，比如只接收css类型的module、只接受ts类型的module等。因此loader函数会被调用n次，n为符合类型要求的文件（module）数量
因此loader本身只用考虑单个文件的编译结果即可。比如css-loader只用考虑css文件的编译，babel-loader不必关心css的导入结果。

Loader 接收三个参数，分别为：

- source：资源输入，对于第一个执行的 Loader 为资源文件的内容；后续执行的 Loader 则为前一个 Loader 的执行结果，可能是字符串，也可能是代码的 AST 结构；
- sourceMap: 可选参数，代码的 sourcemap 结构；
- data: 可选参数，其它需要在 Loader 链中传递的信息，比如 posthtml/posthtml-loader 就会通过这个参数传递额外的 AST 对象。

此外loader内部还有一部分上下文接口，即通过`this.xxx`获取的参数。这些参数来自于webpack运行时的上下文，也非常重要。参考https://webpack.docschina.org/api/loaders/
常用的一些上下文有：
- fs：Compilation 对象的 inputFileSystem 属性，我们可以通过这个对象获取更多资源文件的内容；
- resource：当前文件的绝对路径，包括 query 参数，例如 import "abc/a?foo=bar" 的 resource 值为 abc/a?foo=bar；
- resourcePath: 不包含query的绝对路径。大多数时候用的是这个，因为query参数很少用到
- callback：可用于返回多个结果；
  callback函数参数为：
  ```js
  this.callback(
    err: Error | null,
    content: string | Buffer,
    sourceMap?: SourceMap,
    meta?: any
  );
  ```
  - 第一个参数必须是 Error 或者 null
  - 第二个参数是一个 string 或者 Buffer，即要输出的结果
  - 可选的：第三个参数必须是一个 source map。
  - 可选的：第四个参数，会被 webpack 忽略，可以是任何东西（例如一些元数据）。比如希望在loader之间共享ast，就可以通过这个接口传，不会影响到webpack本身，但是其他loader可以接收到。

  如果调用callback函数返回结果，那loader函数就不能返回任何值，防止造成混淆

- getOptions：用于获取当前 Loader 的配置对象；
- async：用于声明这是一个异步 Loader，开发者需要通过 async 接口返回的 callback 函数传递处理结果；
  大多数时候同步loader可以完成任务，但是有些库的解析过程本身就是异步的，比如less的编译、prettier的处理、ast的解析等过程的函数本身就是异步的。这时候就需要loader异步返回结果，而异步返回的callback就来自于this.async的返回值
  ```js
  async function lessLoader(source) {
    // 1. 获取异步回调函数
    const callback = this.async();
    // ...

    let result;

    try {
      // 2. 调用less 将模块内容转译为 css
      // 这一步less.render本身就是异步的，因此该loader必须要异步返回
      result = await (options.implementation || less).render(data, lessOptions);
    } catch (error) {
      // ...
    }

    const { css, imports } = result;

    // 3. 转译结束，返回结果
    callback(null, css, map);
  }

  export default lessLoader;
  ```
- emitWarning：添加警告；
- emitError：添加错误信息，注意这不会中断 Webpack 运行；
- emitFile：用于直接写出一个产物文件，例如 file-loader 依赖该接口写出 Chunk 之外的产物；
- addDependency：在loader中手动添加依赖。比如less-loader会解析less文件中的import语句，然后把所有import的部分都添加到依赖，这样这些文件发生变化时也会触发重新编译。也可以添加对配置文件的依赖，比如babel-loader可以添加.babelrc为依赖，当配置文件发生变化时就重新编译

### loader编写示例 typing-for-css-modules-loader

该loader的目的是自动为css module生成类型声明文件。
在ts中如果直接导入css module类型的模块，就会提示不是一个模块，因为css module没有类型，而我们需要对该模块声明一个类型。并且，还希望styles对象能包含具体的类名而不是一个any类型，这样通过style的语法提示就可以获取类名了
首先要清楚如何声明一个css module。
如果我们有这样一个css文件：
```css
.container{
  ...
}
.title{
  ...
}
```
经过css-loader编译之后（cssloader的module模式要开启），会变成这种形式：
```js
// Imports
import ___CSS_LOADER_API_NO_SOURCEMAP_IMPORT___ from "../node_modules/css-loader/dist/runtime/noSourceMaps.js";
import ___CSS_LOADER_API_IMPORT___ from "../node_modules/css-loader/dist/runtime/api.js";
var ___CSS_LOADER_EXPORT___ = ___CSS_LOADER_API_IMPORT___(___CSS_LOADER_API_NO_SOURCEMAP_IMPORT___);
// Module
___CSS_LOADER_EXPORT___.push([module.id, ".container_qe3E3 {\n  display: flex;\n}\n.container_qe3E3 .title_Q9ieQ {\n  color: green;\n}\n", ""]);
// Exports
___CSS_LOADER_EXPORT___.locals = {
        "container": "container_qe3E3",
        "title": "title_Q9ieQ"
};
export default ___CSS_LOADER_EXPORT___;
```
`___CSS_LOADER_EXPORT___.locals`对象包含了css文件中的类选择器的类名。css中没有使用类名选择器，而是直接元素或id选择器，就不会放入locals中。因此我们获取类名的范围应该是在locals对象中
然后通过style-loader不加额外处理插入到原本的js module中。这时显然`___CSS_LOADER_EXPORT___`这个对象就是我们导出的styles对象

```js
import styles from './app.module.css'
```

我们希望给`___CSS_LOADER_EXPORT___`对象添加类型，以保证styles也是有类型的。因此可以编写一个声明文件
```ts
// app.module.less.d.ts
// 因为app.module.less文件本来导出的是一个对象，所以这里用type或interface都可以
const cssTypes: {
  container:string;
  title:string
}
export default cssTypes // 注意这里应该是默认导出，因为 import styles 是默认导入
export = cssTypes // 这种形式也是默认导出

// 不能这样，这样是具名导出
export const cssTypes: {...}
```
这样就可以看到在ts文件中没有报错了，并且styles对象也有了类型。

我们的目的就是通过loader自动生成这样的一个类型声明文件。基本逻辑：
1. css-loader已经生成了js形式的css代码。从上可以看出`___CSS_LOADER_EXPORT___.locals`对象包含了我们所需要的类名，因此我们可以用正则提取其中的类名，作为生成的类名使用
2. 输出形式也已经确定好了，即上面的形式，我们可以通过模板字符串插入需要的变量名和类名，然后修改生成文件的文件名为`xxx.d.ts`，最后通过文件系统fs.writeFileSync将其写出到相同目录即可。

最简单的loader结构如下：
```js
const path = require("path");
const fs = require("fs");

// 通过正则提取类名
function getClassesKeys(content) {
  // 这个正则是提取 "xxx": 这个形式的中间部分xxx
  const keyRegex = /"([^"\n]+)":/g;
  let match;
  const cssModuleKeys = [];
  // 这是正则取值的常见写法，如果keyRegex.exec(content)的结果不为空就一直循环提取到match中
  while ((match = keyRegex.exec(content))) {
    if (!cssModuleKeys.includes(match[1])) {
      cssModuleKeys.push(match[1]);
    }
  }
  return cssModuleKeys;
}

module.exports = function (content) {
  // 从.locals开始切割字符串，从这里才开始寻找。如果没有locals对象就说明没有类选择器，就不用继续了
  const localsIndex = content.indexOf(`.locals`)
  if(localsIndex < 0) return content
  const sliceLocals = content.slice(localsIndex)
  // 获取类名
  const cssSelectorNames = getClassesKeys(content);
  const filename = this.resourcePath; // 这里的filename是处理的module的绝对路径
  const outputFilename = path.join( // 输出的文件名也应该包含绝对路径
    path.dirname(filename),
    `${path.basename(filename)}.d.ts`
  );
  const baseName = path.basename(filename).split(".")[0];
  const namespaceName = `Namespace${baseName}`;
  const interfaceName = `I${baseName}`;
  const moduleName = `Module${baseName}`;
  // 根据类名生成`xxx:string`这种形式
  const interfaceProperties = cssSelectorNames
    .map((selector) => `'${selector}':string`);
  // 创建输出字符串
  const outputContent = `const ${moduleName}:{${interfaceProperties}};export default${moduleName}`;
  // 直接将输出内容写入文件
  fs.writeFileSync(outputFilename, outputContent, "utf8");
  // 不需要更改原本内容，正常返回
  return content;
};
```

当然这个只是loader最简单的形式，实际还需要考虑其他情况，比如配置项、schema验证配置项、异步loader等。
有一个更完善的相同功能的loader：https://github.com/TeamSupercell/typings-for-css-modules-loader

## hook架构

hook架构是webpack的插件体系的核心。hook本质是一个发布-订阅模式的发布者，可以在hook上监听事件，然后在webpack的特定阶段执行这些事件。

在webpack中，hooks实际上是compiler类中的一个对象，包含了不同阶段的不同hook。这些hook可以被plugin获取到，在其上注册事件，而不同hook的执行事件的阶段不同，每个hook都会在某个特定阶段调用call方法去执行之前添加的任务。因此，**可以把这些hook看做是webpack编译过程的不同阶段**。通过在plugin内部调用不同的hook，就可以访问到webpack执行过程的各种阶段，从而实现不同效果。

比如：
- compiler.hooks.compilation ：
  - 时机：Webpack 刚启动完，创建出 compilation 对象后触发；
  - 参数：当前编译的 compilation 对象。
- compiler.hooks.make：
  - 时机：正式开始构建时触发；
  - 参数：同样是当前编译的 compilation 对象。
- compilation.hooks.optimizeChunks ：
  - 时机： seal 函数中，chunk 集合构建完毕后触发；
  - 参数：chunks 集合与 chunkGroups 集合。
- compiler.hooks.done：
  - 时机：编译完成后触发；
  - 参数： stats 对象，包含编译过程中的各类统计信息。

每个钩子传递的参数不同，大致包括：compiler、compilation、module、chunk、stats等对象。

```js
this.hooks = Object.freeze({
	/** @type {SyncHook<[]>} */
	initialize: new SyncHook([]),
	/** @type {SyncBailHook<[Compilation], boolean>} */
	shouldEmit: new SyncBailHook(["compilation"]),
	/** @type {AsyncSeriesHook<[Stats]>} */
	done: new AsyncSeriesHook(["stats"]),
	/** @type {SyncHook<[Stats]>} */
  ...
})
```

compiler对象从开始构建到结束，会触发以下hook：
![](https://pic.imgdb.cn/item/63e1da724757feff33d3ce00.jpg)
compilation对象也会按一定顺序触发各种hook：
![](https://pic.imgdb.cn/item/63e1dae24757feff33d48d30.jpg)

其他部分的各种阶段也会触发相应的hook。因此可以说，hook是webpack各个阶段的映射，控制hook就可以访问到执行过程的各种阶段，从而执行不同操作。


### Tapable

tapable本质上可以看作是一个加强的发布-订阅模式。普通的发布订阅，订阅者很少会影响到发布者本身，而tapable形式的发布订阅，订阅者（比如各种插件）可以获取到足够的执行上下文信息，并且可以影响到自己之后的编译流程和状态。

tapable导出了很多种hook，这些hook就可以看做是一个可以注册和执行事件的对象，比如：
```js
const { SyncHook } = require("tapable");

// 1. 创建钩子实例
const sleep = new SyncHook();

// 2. 调用订阅接口注册回调
sleep.tap("test", () => {
  console.log("callback A");
});

// 3. 调用发布接口触发回调
sleep.call();

// 运行结果：
// callback A
```
使用 Tapable 时通常需要经历三个步骤：

1. 创建钩子实例
1. 调用订阅接口注册回调，包括：tap、tapAsync、tapPromise
1. 调用发布接口触发回调，包括：call、callAsync、promise

大部分的hook使用都遵循上面的步骤，区别主要在于hook类型以及注册、调用方法的不同。

### hook类型

hook主要有以下几种：
![](https://pic.imgdb.cn/item/63e1c9f64757feff33b8da16.jpg)

- 按回调逻辑，分为：
  - 基本类型，名称不带 Waterfall/Bail/Loop 关键字：与通常 订阅/回调 模式相似，按钩子注册顺序，逐次调用回调，比如基本的SyncHook，就是一种同步调用回调的方式
  - waterfall 类型：前一个回调的返回值会被带入下一个回调；
  - bail 类型：逐次调用回调，若有任何一个回调返回非 undefined 值，则终止后续调用；
  - loop 类型：逐次、循环调用，直到所有回调函数都返回 undefined 。
- 按执行回调的并行方式，分为：
  - sync ：同步执行，启动后会按次序逐个执行回调，支持 call/tap 调用语句；
  - async ：异步执行，支持传入 callback 或 promise 风格的异步回调函数，支持 callAsync/tapAsync 、promise/tapPromise 两种调用语句。

不同类型的钩子会直接影响到回调函数的写法，以及插件与其他插件的互通关系。

以最简单的Synchook为例，简单使用如下：
```js
const { SyncHook } = require("tapable");

class Somebody {
  constructor() {
    this.hooks = {
      sleep: new SyncHook(),
    };
  }
  sleep() {
    //   触发回调
    this.hooks.sleep.call();
  }
}

const person = new Somebody();

// 注册回调
person.hooks.sleep.tap("test", () => {
  console.log("callback A");
});
person.hooks.sleep.tap("test", () => {
  console.log("callback B");
});
person.hooks.sleep.tap("test", () => {
  console.log("callback C");
});

person.sleep();
// 输出结果：
// callback A
// callback B
// callback C
```

这是最基本的SyncHook的使用，如果是其他类型的hook，基本上就是在这个基础上做一些改动。不同hook的应用场景不同
- SyncBailHook，就是在执行回调的过程中，如果有一个回调有非undefined的返回值，那就终止其他回调的执行，适用于发布者需要关心订阅回调运行结果的场景。比如shouldEmit这个hook，如果在plugin中注册的回调内返回一个false，那么就会终止其他回调的执行，然后这个hook的call返回一个false，可以被调用者接收到，表示出现了错误。
```js
compiler.hooks.shouldEmit.tap("NoEmitOnErrorsPlugin", compilation => {
	if (compilation.getStats().hasErrors()) return false;
});

// compiler.run方法：
if (this.hooks.shouldEmit.call(compilation) === false) {
	...
}
```
- SyncWaterfallHook，就是将前一个回调的返回值传入后一个回调的参数使用
- SyncLoopHook ，就是循环执行，直到所有回调都返回 undefined

每个hook对应执行的call函数都可以换为callAsync函数，callAsync在webpack内部应用更广一些。callAsync需要接收一个回调函数作为参数，用于处理可能抛出的错误
```js
this.hooks.sleep.callAsync(params, (err) => {
  if (err) {
    console.log(`interrupt with "${err.message}"`);
  }
});
```

还有一些异步的hook，比如最基本的AsyncSeriesHook，其实就是SyncHook的异步版
- AsyncSeriesHook，可以用tapAsync注册异步回调，或者用tapPromise注册promise。当异步回调执行完成，或返回的promise resolve之后，就会继续下一个回调。这些回调会依次顺序执行，一个执行完后才会执行下一个。
这个hook其实就是**允许回调内添加异步任务**，适用于需要执行异步任务的plugin
比如：
```js
const hook = new AsyncSeriesHook();

// 注册回调
hook.tapAsync("test", (cb) => {
  console.log("callback A");
  setTimeout(() => {
    console.log("callback A 异步操作结束");
    // 回调结束时，调用 cb 通知 tapable 当前回调已结束
    cb();
  }, 100);
});

hook.tapPromise("test", () => {
  console.log("callback C");
  return new Promise((resolve) => {
    setTimeout(() => {
      console.log("callback C 异步操作结束");
      resolve();
    }, 100);
  });
});

hook.tapAsync("test", () => {
  console.log("callback B");
});



hook.callAsync();
// 运行结果：
// callback A
// callback A 异步操作结束
// callback C
// callback C 异步操作结束
// callback B
```
- AsyncParallelHook，就是并行执行所有的回调，而AsyncSeriesHook是串行执行的。


### hook的应用

hook最大的应用是在plugin内部。这个plugin不仅可以是用户编写的plugin，也可以是webpack内部的plugin。
我们知道plugin类的apply方法会被传入一个compiler对象，那么plugin就可以借助这个compiler对象上的这些hook注册事件。之后在compile方法中，在不同的阶段执行这些事件，就实现了plugin在不同时期执行相应任务的效果。

比如eslint-webpack-plugin，其内部简单代码为：
```js
class ESLintWebpackPlugin {
  constructor(options = {}) {
    // ...
  }

  apply(compiler) {
    compiler.hooks.run.tapPromise(this.key, (c) =>
      this.run(c, options, wanted, exclude)
    );
  }

  async run(compiler, options, wanted, exclude) {
    compiler.hooks.compilation.tap(this.key, (compilation) => {
      ({ lint, report, threads } = linter(this.key, options, compilation));

      const files = [];

      // 单个模块成功编译后触发
      compilation.hooks.succeedModule.tap(this.key, ({ resource }) => {
        // 判断是否需要检查该文件
        if (
          isMatch(file, wanted, { dot: true }) &&
          !isMatch(file, exclude, { dot: true })
        ) {
          lint(file);
        }
      });

      // 所有模块构建完毕后触发
      compilation.hooks.finishModules.tap(this.key, () => {
        if (files.length > 0 && threads <= 1) {
          lint(files);
        }
      });

      // 等待检查结果
      compilation.hooks.additionalAssets.tapPromise(this.key, processResults);

      async function processResults() {
        const { errors, warnings } = await report();

        // 解析之后获取结果，添加eslint的error和warning，让webpack输出。
        if (warnings && !options.failOnWarning) {
          compilation.warnings.push(warnings);
        } else if (warnings && options.failOnWarning) {
          compilation.errors.push(warnings);
        }

        if (errors && options.failOnError) {
          compilation.errors.push(errors);
        } else if (errors && !options.failOnError) {
          compilation.warnings.push(errors);
        }
      }
    });
  }
}
```
这里就用到了如下 Hook：

- `compiler.hooks.compilation`：Compiler 环境初始化完毕，创建出 compilation 对象，准备开始执行构建前触发；
- `compilation.hooks.succeedModule`：Webpack 完成单个「模块」的读入、运行 Loader、AST 分析、依赖分析等操作后触发；这个最为关键，每当webpack完整解析一个模块后，就会去调用lint检查这个文件。
- `compilation.hooks.finishModules`：Webpack 完成「所有」模块的读入、运行 Loader、依赖分析等操作后触发；
- `compilation.hooks.additionalAssets`：构建、打包完毕后触发，通常用于为编译创建附加资产。


## plugin

plugin 是一个类，这点从 plugin 需要 new 调用可以看出来
最基本的 plugin：

```js
class BasicPlugin {
  // options是传入的配置
  constructor(options) {}

  // compiler是上面编译过程中提到的，初始化时会传入compiler对象给插件，以进行一些操作
  apply(compiler) {
    compiler.plugin("compilation", function (compilation) {});
  }
}
```

plugin可以通过Compiler上的各种hook，访问到编译周期的不同阶段的上下文。也可以获取到compilation对象，对make和seal阶段做更详细的控制
这两者的区别再总结一下：
- compiler：实际上是编译器本身，compiler对象贯穿编译过程始终，包含各种hook和大量变异相关的方法，其中compiler.compile方法是整个编译过程的主要方法
- compilation：是编译过程的一部分，即make和seal阶段，当初始化阶段完成后才创建，并且每次由watch等方式触发的重新编译都会重新创建compilation对象。相对来说compilation可控范围更细致，也更常用。

除了这两个对象之外，还有 Module、Resolver、Parser、Generator 等关键类型，也都相应暴露了许多 Hook。

这些对象上常用的方法有：
- complation 对象：构建管理器，使用率非常高，主要提供了一系列与单次构建相关的接口，包括：
  - addModule：用于添加模块，例如 Module 遍历出依赖之后，就会调用该接口将新模块添加到构建需求中；
  - addEntry：添加新的入口模块，效果与直接定义 entry 配置相似；
  - emitAsset：用于添加产物文件，效果与 Loader Context 的 emitAsset 相同；
  - getDependencyReference：从给定模块返回对依赖项的引用，常用于计算模块引用关系；
  等等。
- compiler 对象：全局构建管理器，提供如下接口：
  - createChildCompiler：创建子 compiler 对象，子对象将继承原始 Compiler 对象的所有配置数据；
  - createCompilation：创建 compilation 对象，可以借此实现并行编译；
  close：结束编译；
  - getCache：获取缓存接口，可借此复用 Webpack5 的缓存功能；
  - getInfrastructureLogger：获取日志对象；
  等等。
- module 对象：资源模块，有诸如 NormalModule/RawModule/ContextModule 等子类型，其中 NormalModule 使用频率较高，提供如下接口：
  - identifier：读取模块的唯一标识符；
  - getCurrentLoader：获取当前正在执行的 Loader 对象；
  - originalSource：读取模块原始内容；
  - serialize/deserialize：模块序列化与反序列化函数，用于实现持久化缓存，一般不需要调用；
  - issuer：模块的引用者；
  - isEntryModule：用于判断该模块是否为入口文件；
  等等。
- chunk 对象：模块封装容器，提供如下接口：
  - addModule：添加模块，之后该模块会与 Chunk 中其它模块一起打包，生成最终产物；
  - removeModule：删除模块；
  - containsModule：判断是否包含某个特定模块；
  - size：推断最终构建出的产物大小；
  - hasRuntime：判断 Chunk 中是否包含运行时代码；
  - updateHash：计算 Hash 值。
- stats 对象：构建过程收集到的统计信息，包括模块构建耗时、模块依赖关系、产物文件列表等。

## 环境变量

nodejs 的环境变量是 `process.env`,根据操作系统的不同会有不同的变化;`process.env` 是一个对象，通过访问内部的环境变量可以进行一些操作
webpack 可以利用`webpack.DefinePlugin`插件设置一些环境变量，相当于一个全局变量，可以在任意处被直接访问

```js
plugins: [
  new webpack.DefinePlugin({
    IS_OLD: true,
    MY_ENV: JSON.stringify("dev"),
    NAME: "'zzx'",
  }),
],
  //a.js
  console.log(NAME);
```

此外，webpack 也同样可以使用 process.env 访问环境变量；比如常用`process.env.NODE_ENV === 'production'`判断是否在生产环境

## 文件监听

文件监听是在发现源码文件发生变化时，自动重新构建出新的输出文件；文件监听功能是 webpack 模块提供的，直接在配置中使用`watch:true`即可开启。
文件监听是`devServer`和`HMR`的基础，只有通过文件监听发现了更改和要更改的地方，才会通知这两者进行自动刷新工作。

```js
module.export = {
  // 只有在开启监听模式时，watchOptions 才有意义
  // 默认为 false，也就是不开启
  watch: true,
  // 监听模式运行时的参数
  // 在开启监听模式时，才有意义
  watchOptions: {
    // 不监听的文件或文件夹，支持正则匹配
    // 默认为空
    ignored: /node_modules/,
    // 监听到变化发生后会等300ms再去执行动作，防止文件更新太快导致重新编译频率太高
    // 默认为 300ms
    aggregateTimeout: 300,
    // 判断文件是否发生变化是通过不停的去询问系统指定文件有没有变化实现的
    // 默认每隔1000毫秒询问一次
    poll: 1000,
  },
};
```

文件监听的原理主要如下：

1. 基本原理是定时获取文件的最后编辑时间，如果发现**当前获取**的和**最后一次保存**的最后编辑时间不一致，就认为该文件发生了变化。`poll`配置项就是配置这个“定时”，即每隔多久检查一次
2. 文件发生变化并不会立即告知监听者，而是会先缓存，积累一段时间后再一次性告知。`aggregateTimeout`就是配置这个项，相当于一个防抖的效果，即连续更新停止一段事件后才会把所有之前的任务提交给监听者。
3. 默认情况下 Webpack 会从配置的 Entry 文件出发，递归解析出 Entry 文件所依赖的文件，把这些依赖的文件都加入到监听列表中去，而不是一次性把所有文件都加入监听列表。

## 自动刷新

`webpack` 模块负责监听文件，`webpack-dev-server` 模块则负责刷新浏览器。 在使用 `webpack-dev-server` 模块去启动 `webpack` 模块时，`webpack` 模块的监听模式默认会被开启。 `webpack` 模块会在文件发生变化时告诉 `webpack-dev-server` 模块。

控制浏览器刷新有三种方法：

1. 借助浏览器扩展去通过浏览器提供的接口刷新，WebStorm IDE 的 LiveEdit 功能就是这样实现的。
1. 往要开发的网页中注入代理客户端代码，通过代理客户端去刷新整个页面。
1. 把要开发的网页装进一个 iframe 中，通过刷新 iframe 去看到最新效果。

devServer 采用的默认是第二种方法，可以配置为第三种（配置项`devServer`的`inline`设为 false 即可开启）。

### devServer

简单来说，devServer 相当于把打包出来的代码（dist 文件夹）部署到一个本地服务器，创建一个线上环境；然后启动一个 使用 express 的 Http 服务器用于服务网页请求，同时会帮助启动 Webpack ，并接收 Webpack 发出的文件更变信号，通过 WebSocket 协议自动刷新网页做到实时预览。

![](https://pic.imgdb.cn/item/61db12b62ab3f51d91244c5a.png)

> `webpack-dev-server`服务于的是资源文件，即主要为核心代码所在的 js 文件，不会对`index.html`的修改做出反应

## HMR

![](https://pic.imgdb.cn/item/624073d827f86abb2a67f487.jpg)

在 HMR 之前，应用的加载、更新都是一种页面级别的原子操作，即使只是单个代码文件发生变更，都需要刷新整个页面，才能将最新代码映射到浏览器上，这会丢失之前在页面执行过的所有交互与状态，例如：

对于复杂表单场景，这意味着你可能需要重新填充非常多字段信息；
- 弹框消失，你必须重新执行交互动作才会重新弹出。
- 再小的改动，例如更新字体大小，改变备注信息都会需要整个页面重新加载执行，整体开发效率偏低。而引入 HMR 后，虽然无法覆盖所有场景，但大多数小改动都可以通过模块热替换方式更新到页面上，从而确保连续、顺畅的开发调试体验，极大提升开发效率。

### 使用

hmr的使用比较麻烦。和自动刷新不同，hmr需要手动指定怎么把模块替换成最新的代码。

1. 设置 devServer.hot 属性为 true
```js
// webpack.config.js
module.exports = {
  // ...
  devServer: {
    // 必须设置 devServer.hot = true，启动 HMR 功能
    hot: true
  }
};
```

2. 之后，还需要在代码调用 module.hot.accept 接口，声明如何将模块安全地替换为最新代码，如：

```js
import component from "./component";
let demoComponent = component();

document.body.appendChild(demoComponent);
if (module.hot) {
  module.hot.accept("./component", () => {
    // 调用接口获取全新的component
    const nextComponent = component();

    // 更新成新的component
    document.body.replaceChild(nextComponent, demoComponent);

    demoComponent = nextComponent;
  });
}
```

可以看到这种形式非常麻烦，需要自行控制怎么替换

### 原理

热更新流程可以分为以下几步：
1. 使用 webpack-dev-server （后面简称 WDS）托管静态资源，同时以 Runtime 方式注入一段处理 HMR 逻辑的客户端代码；

在 HMR 场景下，执行 npx webpack serve 命令后，webpack-dev-server 首先会调用 HotModuleReplacementPlugin 插件向应用的主 Chunk 注入一系列 HMR Runtime，包括：

- 用于建立 WebSocket 连接，处理 hash 等消息的运行时代码；
- 用于加载热更新资源的接口；
- 用于处理模块更新策略的 module.hot.accept 接口

经过 HotModuleReplacementPlugin 处理后，构建产物中即包含了所有运行 HMR 所需的客户端运行时与接口。相当于在输出的产物中包含了建立websocket连接、处理加载资源等的一段代码，这部分代码将帮助完成热更新流程。
这些 HMR 运行时会在浏览器执行一套基于 WebSocket 消息的时序框架。当这部分内容运行时会和wds建立websocket连接，接收hash事件，获取manifest等操作。
![](https://pic.imgdb.cn/item/63e34e4f4757feff3308c3b3.jpg)

2. 浏览器加载页面后，与 WDS 建立 WebSocket 连接；
3. Webpack 监听到文件变化后，增量构建发生变更的模块，并通过 WebSocket 发送 hash 事件；

这一步依赖webpack提供的watch功能。当监视的文件发生变化时，webpack会重新执行make阶段对发生变化的文件重新编译，并生成：

- manifest 文件：JSON 格式文件，包含所有发生变更的模块列表，命名为 `[hash].hot-update.json`；
- 模块变更文件：js 格式，包含编译后的模块代码，命名为 `[hash].hot-update.js`

增量构建完毕后，Webpack 将触发 compilation.hooks.done 钩子，并传递本次构建的统计信息对象 stats。WDS 则监听 done 钩子，在回调中通过 WebSocket 发送模块更新消息，即hash事件

```js
{"type":"hash","data":"${stats.hash}"}
```

4. 浏览器接收到 hash 事件后，请求 manifest 资源文件，确认本次热更新涉及的chunk

在webpack5之前，热更新的单位是模块，每个模块的热更新都会生成对应的热更新文件；当前的方式是每个包含热更新文件的chunk在更新之后都会生成当前chunk的更新文件，即一个名为`main.[hash].hot-update.js`的更新文件

5. 浏览器加载发生变更的增量模块。
6. Webpack 运行时触发变更模块的 module.hot.accept 回调，执行代码变更逻辑；到这一步时浏览器已经加载完了最新模块代码，执行回调内的逻辑其实就是相当于一段额外代码，会修改原有的逻辑。




# 练习配置

## 使用webpack.config.ts

config文件的配置常常因为不清楚配置项的名称而搞错，因此可以考虑采用ts类型的配置文件。

步骤：
1. 首先需要安装：
```
yarn add --dev typescript ts-node @types/node @types/webpack
```

2. 生成tsconfig.json。
```
tsc --init
```

```json
{
  "compilerOptions": {
    "module": "commonjs", //该字段必须是commonjs
    "esModuleInterop": true
  },
  "exclude": ["node_modules", "webpack.config.ts"] // 这里注意添加
}
```

3. 然后编写webpack.config.ts。


```ts
import * as webpack from "webpack";
import * as path from "path";

const config: webpack.Configuration = {
  ...
};

export default config;
```

注意部分模块需要导入类型定义，比如devServer就需要额外安装@types/webpack-dev-server，然后在config中导入webpack-dev-server

```ts
import * as webpack from "webpack";
import * as path from "path";
import webpackDevServer from 'webpack-dev-server';

const config: webpack.Configuration = {
  devServer:{
    hot:true,
  },
};

export default config;
```


## 配置React开发环境

### 基本React

基本配置内容包括：React、babel以及typescript。

1. 安装webpack等一系列内容
```
yarn add webpack webpack-cli webpack-dev-server -D
yarn add css-loader babel-loader style-loader -D
yarn add react react-dom
yarn add html-webpack-plugin -D
```
如果全局没有安装ts的话，还需要安装typescript

2. 执行`npx webpack init`创建基本配置文件，以及`tsc --init`创建tsconfig.json。可以参考上面使用webpack.config.ts的配置，这样可以减少错误

3. 编写配置文件，主要有以下内容：

- entry：即React的入口文件index.tsx。
- output：正常输出即可
- module.rules：主要配置两个loader：
  - "style-loader"和"css-loader"
  - babel-loader
    - `@babel/preset-react`，按照如下方式配置。如果开启runtime配置项，项目中jsx、tsx文件就不需要引入react
    - `@babel/preset-typescript`，用于解析ts
- resolve.extensions：注意配置tsx和ts，因为导入ts/tsx时不能添加后缀，因此必须添加ts和tsx后缀
- plugins：HtmlWebpackPlugin，设置一个template，主要是要包含一个id为root的div，作为react的root element。

```ts
const config: webpack.Configuration = {
  entry: "./src/index.tsx",
  output: {
    filename: "[name]_[contenthash:4].js",
    path: path.join(__dirname, "dist"),
    clean: true,
  },
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.tsx?$/,
        include: path.join(__dirname, "src"),
        loader: "babel-loader",
        options: {
          presets: [
            [
              "@babel/preset-react",
              {
                runtime: "automatic",
              },
            ],
            "@babel/preset-typescript",
          ],
        },
      },
    ],
  },
  resolve: {
    extensions: [".tsx", ".ts", ".jsx", ".js"],
  },
  devServer: {
    hot:true
  },
  target: "web",
  plugins: [
    new HtmlWebpackPlugin({
      template: "./assets/index.html",
    }),
  ],
  mode: "development",
};

```

4. 配置scripts：
```json
"scripts": {
  "start": "webpack",
  "serve": "webpack serve --open"
},
```

然后执行yarn serve就可以启动

### 添加css预处理器和css-module

以less为例：
1. 安装
```
yarn add -D less less-loader
```

2. 添加less-loader。注意这里的test也要修改成`/\.less$/`

```js
{
  test: /\.less$/i,
  use: ["style-loader", "css-loader","less-loader"],
},
```

---

关于css module的配置，最简单的方式就是在cssloader的options开启`modules:true`即可，当然还有另外几个配置：
- modules:用于配置是否启用cssmodule及其内部配置。如果不为false则为开启，还可以是一个配置对象，具体配置项参考https://github.com/webpack-contrib/css-loader#auto

  这里主要使用了localIdentName配置，用于修改经过css module处理的类名，推荐这种形式：
  ```
  [local]_[hash:base64:5]
  ```
  - local：原本的样式名
  - hash:base64:5：css module为了区分类名的hash值，必须要有，可以修改长度

  其他可以设置的模板字符串参考https://github.com/webpack-contrib/css-loader#localidentname

- importLoaders：表示在css-loader之前还有几个loader，即use数组中，在css-loader之后的loader个数，默认是0，需要修改成1。这里只有less-loader一个，如果配置了postcss，那就是两个

```js
{
  test: /\.less$/i,
  use: [
    "style-loader",
    {
      loader: "css-loader",
      options: {
        modules: {
          localIdentName: "[local]_[hash:base64:5]",
        },
        importLoaders: 1,
      },
    },
    "less-loader",
  ],
},
```

另外还有个问题，ts中会对导入的css module文件报错`找不到模块 ./xxx.module.less 或其相应的类型声明`。这是因为modulecss文件没有类型声明。因此需要自己编写一个`.d.ts`文件为modulecss添加类型

```ts
declare module '*.module.less' {
  const classes: {[key:string]:string};
  export default classes
}
```
按照ts对声明文件的解析规则，这里是声明了一个module类型的模块，`.module.css`文件会导出一个包含classes对象类型的变量，即导入的style类型`{[key:string]:string}`

@teamsupercell/typings-for-css-modules-loader库可以自动根据module.css文件生成对应的声明文件，并且可以根据css的类名添加具体的属性名到style对象上去，库说明参考https://github.com/TeamSupercell/typings-for-css-modules-loader

首先安装：
```
yarn add -D @teamsupercell/typings-for-css-modules-loader
```
然后按照正常loader方式使用即可，注意要放在css-loader后面、style-loader前面
```js
use:[
  "style-loader",
  "@teamsupercell/typings-for-css-modules-loader",
  {
    loader: "css-loader",
    options: {
      modules: {
        localIdentName: "[local]_[hash:base64:5]",
      },
      importLoaders: 1,
    },
  },
  "less-loader",
],
```
对于每一个.module.css文件，都会生成对应的.d.ts声明文件。比如某个less文件为：
```less
.container{
    display: flex;
    .title{
        color: green;
    }
}
```
生成对应的.d.ts
```ts
declare namespace AppModuleLessNamespace {
  export interface IAppModuleLess {
    container: string;
    title: string;
  }
}

declare const AppModuleLessModule: AppModuleLessNamespace.IAppModuleLess & {
  /** WARNING: Only available when `css-loader` is used without `style-loader` or `mini-css-extract-plugin` */
  locals: AppModuleLessNamespace.IAppModuleLess;
};

export = AppModuleLessModule;
```

可以看到，具体的类名都已经声明完成了，在tsx文件中使用style可以看到类型

### 添加hmr

由于webpack的hmr需要开发者自行确定更新范围，这对于react这种复杂的框架来说很麻烦。因此可以使用已有的hmr loader来简化这一过程。

react-hot-loader用于处理react的热更新，参考https://github.com/gaearon/react-hot-loader
1. 安装
```
npm install react-hot-loader
```
2. 在babel中添加plugins：

```js
{
  test: /\.tsx?$/,
  include: path.join(__dirname, "src"),
  loader: "babel-loader",
  options: {
    presets: [
      [
        "@babel/preset-react",
        {
          runtime: "automatic",
        },
      ],
      "@babel/preset-typescript",
    ],
    plugins: ["react-hot-loader/babel"],
  },
},
```

3. 修改entry为如下形式。这一步的目的是在react和react-dom之间添加react-hot-loader
```js
entry: ['react-hot-loader/patch', './src/index.tsx'],
```

4. （非必须）添加@hot-loader/react-dom，然后在alias中修改react-dom的别名。用这个react-dom替换原react-dom的目的在于保证部分功能实现，比如useEffect的热更新。
但是这个库没有对react18做出更新，因此不建议修改react-dom

```
yarn add @hot-loader/react-dom
```
```js
resolve: {
  extensions: [".tsx", ".ts", ".jsx", ".js"],
  alias:{
    'react-dom': '@hot-loader/react-dom'
  }
},
```

5. 在需要热更新的组件中，用hot函数包裹：
```ts
import {hot} from 'react-hot-loader/root'

function App() {
  ...
}
export default hot(App)
```

只有用hot包裹的组件才能处理热更新，其内部的子组件也需要hot包裹。这种形式很麻烦，因此现在一般不采用hmr，而是采用 fast refresh（快速刷新）

### 添加快速刷新

快速刷新是目前更常用的、代替hmr的方式，包括cra在内的多种脚手架都采用的快速刷新。
快速刷新针对的不止是react，甚至包括普通js文件。
参考：https://github.com/pmmmwh/react-refresh-webpack-plugin/

1.  安装

```
yarn add -D @pmmmwh/react-refresh-webpack-plugin react-refresh
```

2. 添加babel plugins和plugins
```js
{
  test: /\.tsx?$/,
  include: path.join(__dirname, "src"),
  loader: "babel-loader",
  options: {
    presets: [
      [
        "@babel/preset-react",
        {
          runtime: "automatic",
        },
      ],
      "@babel/preset-typescript",
    ],
    plugins: ["react-refresh/babel"],
  },
},
```
```js
// 这个是总的plugins
plugins: [
  new ReactRefreshWebpackPlugin(),
],
```

3. 在文件中不需要任何额外添加的部分，直接可以实现hmr效果。

另外，快速刷新仍然依赖于hmr，因此还是需要开启`devServer.hot = true`.



