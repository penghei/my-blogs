---
title: webpack学习
date: 2022-01-09 13:14:15
tags: 日常学习
cover: /img/webpack.png
categories: 工具
sticky: 7
---

# Webpack 概述

1. Webpack 是一个模块打包工具，可以使用它管理项目中的模块依赖，并编译输出模块所需的静态文件。
2. 它可以很好地管理、打包开发中所用到的 HTML,CSS,JavaScript 和静态文件（图片，字体）等，让开发更高效。
3. 对于不同类型的依赖，Webpack 有对应的模块加载器，而且会分析模块间的依赖关系，最后合并生成优化的静态资源

## 核心概念

- `Entry`：入口，Webpack 执行构建的第一步将从 Entry 开始，可抽象成输入。

- `Module`：模块，在 Webpack 里一切皆模块，一个模块对应着一个文件。Webpack 会从配置的 Entry 开始递归找出所有依赖的模块。`module`本质上是 webpack 对打包的基本单位，每一个文件都算一个 module；module 下有一个 rules 字段，rules 下有就是处理模块的规则，配置哪类的模块，交由哪类 loader 来处理。

- `Chunk`：代码块，Chunk 是 Webpack 打包过程中一堆 module 的集合。Webpack 的打包是从一个入口文件开始，也可以说是入口模块，入口模块引用这其他模块，模块再引用模块。Webpack 通过引用关系逐个打包模块，这些 module 就形成了一个 Chunk；
  一条路径一般只会形成一个 Chunk；如果 module 是异步的（比如动态导入的 module），就会创建新的 Async chunk，并把该异步模块及其子模块放入。
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
>   除了便于输出文件不重名，这几个 hash 还有更重要的作用，就是在缓存时表示哪个文件被修改，需要更新缓存。
>   注意这里的更新一般指的是版本上的更新。比如说当前页面已经上线，现在做了一次更新；但是由于用户客户端的缓存还没有过期，客户端还用的是之前的代码，该如何让客户端主动请求最新的资源呢？方法就是直接修改资源的文件名。当文件名发生变换，相当于静态资源的路径改了，这样，就相当于第一次访问这些资源，从而更新缓存，完成版本更新
>   大多数情况下，我们可以对可能需要更新的资源文件名，比如 js、css 和图片等静态资源加入 hash 值，方便后续的更新。

- `Loader`：模块转换器，用于把模块原内容按照需求转换成新内容。
- `Plugin`：扩展插件，在 Webpack 构建流程中的特定时机注入扩展逻辑来改变构建结果或做你想要的事情。

> Loader 和 Plugin 的区别
> Loader 本质就是一个函数，在该函数中对接收到的内容进行转换，返回转换后的结果。 因为 Webpack 只认识 JavaScript，所以 Loader 就成了翻译官，对其他类型的资源进行转译的预处理工作。
> Plugin 就是插件，可以扩展 Webpack 的功能，在 Webpack 运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果。

- `Output`：输出结果，在 Webpack 经过一系列处理并得出最终想要的代码后输出结果。
- `Bundle`：最终的打包结果，即一般情况下输出的`bundle.js`文件。大多数时候`chunk`和`bundle`是一一对应的，但是如果开启了`sourceMap`可能会使`chunk`多一个。
- `Dependence`：依赖对象，webpack 基于该类型记录模块间依赖关系

### chunk

chunk 由 module 在 seal 阶段组成，这部分的具体流程可以参考：

https://juejin.cn/book/7115598540721618944/section/7119035921680302115

chunk 在打包过程中会被分成三种类型：

- Entry Chunk：同一个 entry 下触达到的模块组织成一个 Chunk；一个 entry 就会创建一个 chunk，相应的 n 个入口就有 n 个 chunk
- Async Chunk：异步模块单独组织为一个 Chunk，从异步导入的那个入口模块开始，它的所有子模块都会被打包到异步模块中
- Runtime Chunk：entry.runtime 不为空时，会将运行时模块单独组织成一个 Chunk。这种 chunk 主要包含的是运行时代码，在输出时会被注入到代码中，实现部分功能，比如
  - ` __webpack_require__.f`、`__webpack_require__.r` 等功能实现最起码的模块化
  - 动态加载特性，则需要写入 `__webpack_require__.e` 函数

绝大多数情况下，webpack 会为每个 chunk 都生成一个单独的 js 文件。比如项目中使用了 5 次 import()来动态导入，那么 webpack 就会额外生成 5 个 js 文件，分别表示这 5 个 chunk 的输出结果。


## 安装使用及配置

### 安装

当前默认版本都是最新的 webpack5，对应的 loader、plugins 等也都是配套 5 的；当然 4 版本也有对应的 loader、plugin 版
这里除了 webpack 还会给出一些常用的安装

```
npm install webpack webpack-cli --save-dev


npm install webpack-dev-server --save-dev



npm install --save-dev style-loader css-loader
npm install --save-dev babel-loader
npm install --save-dev typescript

npm install --save-dev html-webpack-plugin

```

# webpack.config.js

webpack.config.js 作为 webpack 的配置文件，通常会导出一个对象给 webpack 配置使用；由于 webpack 是一个黑盒，我们不需要直到内部运行，相关的配置几乎都是在这个文件中完成并实现的。

## 基本配置

### 配置项分类

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

> output.chunkFormat可以配置不同的输出模块化方案。可配置的有"commonjs"、"module"等，可以通过插件添加其他模式

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

- env: 通过 --env 传递的命令行参数。一般是自定义的，比如`webpack --env prod=1`收到的env的值就是`{ prod: "1" }`
- argv: 命令行 Flags 参数，一般是两个横线开头，这些值都是webpack的配置项。比如 `--entry`，就是设置入口，这个和在webpack.config.js中设置是一样的。还有常用的有`--config`用于指定配置文件。

详见：https://webpack.js.org/api/cli/#flags

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

argv 对象还可以获取更多的信息，比如通过 argv.mode 可以获取到当前配置采用的是那种模式：

```js
var config = {
  entry: "./app.js",
  //...
};

module.exports = (env, argv) => {
  if (argv.mode === "development") {
    config.devtool = "source-map";
  }

  if (argv.mode === "production") {
    //...
  }

  return config;
};
```

还可以导出一个 promise。以 async 函数为例，可以在函数内请求配置，或者异步读取本地文件来增加配置项：

```js
const getConfig = async () => {
  const config = {
    // ...其他配置
  };
  const response = await fetch("https://example.com/api/config");
  const json = await response.json();
  config.apiKey = json.apiKey;
  return config;
};

module.exports = getConfig();
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
**注意，dependOn 的对象通常是某个外部库，比如 lodash、react 等，一般不能用作自己编写的模块**

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
  - `chunkhash`：此 Chunk 独特的 Hash 值，相比于 hash，每个 chunk 对应的出口 filename 获得的[chunkhash]是不一样的。这样可以保证打包后每一个 JS 文件名都**不一样**
  - `contenthash`：chunk 内容的 hash 值。和 chunkhash 不同的是，contenthash 仅和内容型的文件的内容有关，即代码文件的内容，而和图片等资源文件无关。

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

#### output.chunkFilename

此选项决定了非初始（non-initial）chunk 文件的名称。取值和filename一样。

即，对于代码分割的模块，需要用这个属性来设置输出的非主chunk的filename，即async chunk、runtime chunk、splitChunk的分割结果等内容。

### module.rules

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

#### stats

用于配置 webpack 打包过程中命令行的显示内容，可以控制显示或不显示哪些。
这个配置项可以是一个字符串或对象。如果是字符串，则表示一种预设，即 webpack 预设的一些输出内容。

![](https://pic.imgdb.cn/item/6416a2d5a682492fcc677bb8.jpg)

如果配置对象，则可以配置每一项。比如指定不展示输出资源信息：

```js
module.exports = {
  //...
  stats: {
    assets: false,
  },
};
```

#### devtool

用于配置 sourceMap

> 当 webpack 打包源代码时，可能会很难追踪到 error(错误) 和 warning(警告) 在源代码中的原始位置。例如，如果将三个源文件（a.js, b.js 和 c.js）打包到一个 bundle（bundle.js）中，而其中一个源文件包含一个错误，那么堆栈跟踪就会直接指向到 bundle.js。你可能需要准确地知道错误来自于哪个源文件，所以这种提示这通常不会提供太多帮助。
> 为了更容易地追踪 error 和 warning，JavaScript 提供了 source maps 功能，可以将编译后的代码映射回原始源代码。如果一个错误来自于 b.js，source map 就会明确的告诉你。

source map 有很多种配置项，都是一些字符串。虽然配置项很多，但是这些枚举值内在有一个潜规则：都是由 inline、eval、source-map、nosources、hidden、cheap、module 七种关键字组合而成，这些关键词各自代表一项 Sourcemap 规则

主要的关键字有以下几种：

1. eval ：当 devtool 值包含 eval 时，生成的模块代码会被包裹进一段 eval 函数中，且模块的 Sourcemap 信息通过 //# sourceURL 直接挂载在模块代码内。例如：

```js
eval("var foo = 'bar'\n\n\n//# sourceURL=webpack:///./src/index.ts?");
```

eval 模式编译速度通常比较快，但产物中直接包含了 Sourcemap 信息，因此只推荐在开发环境中使用。

2. source-map: 当 devtool 包含 source-map 时，Webpack 才会生成 Sourcemap 内容。例如，对于 devtool = 'source-map'，产物会额外生成 .map 文件，形如：

```json
{
  "version": 3,
  "sources": ["webpack:///./src/index.ts"],
  "names": ["console", "log"],
  "mappings": "AACAA,QAAQC,IADI",
  "file": "bundle.js",
  "sourcesContent": ["const foo = 'bar';\nconsole.log(foo);"],
  "sourceRoot": ""
}
```

3. cheap: cheap 关键字：当 devtool 包含 cheap 时，生成的 Sourcemap 内容会抛弃列维度的信息，这就意味着浏览器只能映射到代码行维度，是一种降低精度减小体积的方式。

例如 devtool = 'cheap-source-map' 时，产物：

```json
{
  "version": 3,
  "file": "bundle.js",
  "sources": ["webpack:///bundle.js"],
  "sourcesContent": ["console.log(\"bar\");"],
  // 带 cheap 效果：
  "mappings": "AAAA",
  // 不带 cheap 效果：
  // "mappings": "AACAA,QAAQC,IADI",
  "sourceRoot": ""
}
```

4. inline 关键字：当 devtool 包含 inline 时，Webpack 会将 Sourcemap 内容编码为 Base64 DataURL，直接追加到产物文件中。例如对于 devtool = 'inline-source-map'，产物：

```js
console.log("bar");
//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vLi9zcmMvaW5kZXgudHMiXSwibmFtZXMiOlsiY29uc29sZSIsImxvZyJdLCJtYXBwaW5ncyI6IkFBQ0FBLFFBQVFDLElBREkiLCJmaWxlIjoiYnVuZGxlLmpzIiwic291cmNlc0NvbnRlbnQiOlsiY29uc3QgZm9vID0gJ2Jhcic7XG5jb25zb2xlLmxvZyhmb28pOyJdLCJzb3VyY2VSb290IjoiIn0=
```

inline 模式编译速度较慢，且产物体积非常大，只适合开发环境使用。

5. module: 关键字只在 cheap 场景下生效，例如 cheap-module-source-map、eval-cheap-module-source-map。当 devtool 包含 cheap 时，Webpack 根据 module 关键字判断按 loader 联调处理结果作为 source，还是按处理之前的代码作为 source。也就是说如果不开启 module，那么指向的是经过 loader 处理过后的代码；而开启 module 则指向的是开发环境下的源代码。

Webpack 的 devtool 值都是由以上七种关键字的一个或多个组成，虽然提供了 27 种候选项，但逻辑上都是由上述规则叠加而成，例如：

cheap-source-map：代表 不带列映射 的 Sourcemap ；
eval-nosources-cheap-source-map：代表 以 eval 包裹模块代码 ，且 .map 映射文件中不带源码，且 不带列映射 的 Sourcemap。

常用的推荐：

- 对于开发环境，适合使用速度快，精准度可以稍低的
  - `eval`：速度极快，但只能看到原始文件结构，看不到打包前的代码内容；
  - `eval-cheap-source-map`：速度比较快，可以看到打包前的代码内容，但看不到 loader 处理之前的源码；
  - `eval-cheap-module-source-map`：速度比较快，可以看到 loader 处理之前的源码，不过定位不到列级别；
  - `eval-source-map`：初次编译较慢，但定位精度最高；
- 对于生产环境，则适合使用精准度高、信息完整的
  - `source-map`：信息最完整，但安全性最低，外部用户可轻易获取到压缩、混淆之前的源码，慎重使用；
  - `hidden-source-map`：信息较完整，安全性较低，外部用户获取到 .map 文件地址时依然可以拿到源码；
  - `nosources-source-map`：源码信息缺失，但安全性较高，需要配合 Sentry 等工具实现完整的 Sourcemap 映射。

##### source map 是什么

Sourcemap 协议 最初由 Google 设计并率先在 Closure Inspector 实现，它的主要作用就是将经过压缩、混淆、合并的产物代码还原回未打包的原始形态，帮助开发者在生产环境中精确定位问题发生的行列位置。
sourcemap 的映射关系由浏览器完成，但是用于标识映射的文件需要用户生成。V3 版本 Sourcemap 文件由三部分组成:

- 开发者编写的原始代码；
- 经过 Webpack 压缩、转化、合并后的产物，且产物中必须包含指向 Sourcemap 文件地址的 //# sourceMappingURL=https://xxxx/bundle.js.map 指令；
- 记录原始代码与经过工程化处理代码之间位置映射关系 Map 文件。

例如，在 Webpack 中设置 devtool = 'source-map' 即可同时打包出代码产物 xxx.js 文件与同名 xxx.js.map 文件，Map 文件通常为 JSON 格式，内容如：

```json
{
  "version": 3,
  "sources": ["webpack:///./src/index.js"],
  "names": ["name", "console", "log"],
  "mappings": ";;;;;AAAA,IAAMA,IAAI,GAAG,QAAb;AAEAC,OAAO,CAACC,GAAR,CAAYF,IAAZ,E",
  "file": "main.js",
  "sourcesContent": ["const name = 'tecvan';\n\nconsole.log(name)"],
  "sourceRoot": ""
}
```

各字段含义分别为：

- version： 指代 Sourcemap 版本，目前最新版本为 3；
- names：字符串数组，记录原始代码中出现的变量名；
- file：字符串，该 Sourcemap 文件对应的编译产物文件名；
- sourcesContent：字符串数组，原始代码的内容；
- sourceRoot：字符串，源文件根目录；
- sources：字符串数组，原始文件路径名，与 sourcesContent 内容一一对应；
- mappings：字符串数组，记录打包产物与原始代码的位置映射关系

使用时，浏览器会按照 mappings 记录的数值关系，将产物代码映射回 sourcesContent 数组所记录的原始代码文件、行、列位置

mappings 数组中的每个串可以理解为指向了编译之后代码的具体的行和列以及其在源码中对应的文件、行和列。这个含义非常复杂并且极其浓缩；但只要了解这其中的每个串都是一种映射关系就好。

#### 环境变量

环境变量通常使用方式有几种
- 在webpack.config中根据不同的环境变量选择不同的配置
- 在nodejs环境下编译解析部分库时，根据环境变量来执行不同操作，比如react的开发和生产模式代码有很大区别
- 在实际项目代码中通过注入的自定义全局变量来操作

通常传入环境变量的方式是利用命令行的`--env`

```
webpack --env goal=local --env production
```

在webpack.config中，如果导出一个函数，就可以获取到这个配置的环境变量

```js
module.exports = (env) => {
  // Use env.<YOUR VARIABLE> here:
  console.log('Goal: ', env.goal); // 'local'
  console.log('Production: ', env.production); // true

  return {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist'),
    },
  };
};
```

如果是其他在nodejs环境下运行的代码，可以借助`cross-env`这个工具进行配置，在代码中可以通过process.env.xxx来获取：

比如在package.json中配置：

```json
{
  "scripts": {
     "build": "cross-env NAME_W=aaa webpack --config ./webpack.config.js"
  }
}
```
代码中：

```js
console.log(process.env.NAME_W, 'env'); // 'aaa'
```

如果希望在业务代码中使用，就需要适配浏览器。浏览器不存在process对象，因此需要借助DefinePlugin

```js
// webpack.config.js
...
new webpack.DefinePlugin({
  __WEBPACK__ENV: JSON.stringify('packages'),
  TWO: '1+1',
});

// src/main.js
console.log('hello, Environment variable', __WEBPACK__ENV)
```

**注意**，webpack.definePlugins本质上是打包过程中的字符串替换。比如上面的代码，在打包过程中，如果我们代码中使用到了`__WEBPACK__ENV`，webpack会将它的值替换成为对应definePlugins中定义的值，本质上就是匹配字符串替换。

也就是说输出的代码会被替换为定义的环境变量；如果我们定义的是一个表达式，那么输出的也是一个表达式。

```js
// webpack.config.js
new webpack.DefinePlugin({
  __WEBPACK__ENV: JSON.stringify('packages'),
});

// 输出的代码，注意packages是字符串
console.log('hello, Environment variable', 'packages')

// webpack.config.js
new webpack.DefinePlugin({
  __WEBPACK__ENV: 'packages',
});

// 输出代码，这里的packages变成了一个变量
console.log('hello, Environment variable', packages)
```


# 优化

关于 webpack 的优化有很多内容，因为其优化方式本身就有很多，原理有些也比较复杂。这里只总结一些常用或常问的，最重要的还是原理性理解

webpack5 优化参考：https://jelly.jd.com/article/61179aa26bea510187770aa3

## 优化分析工具

speed-measure-webpack-plugin：计算各步骤的花费时间
webpack-bundle-analyzer：分析各部分打包体积

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

并且开启多线程也是需要启动时间,大约 600ms 左右。因此要考虑好时间成本

#### Parallel-Webpack

Parallel-Webpack 其实是一个 webpack 的套壳升级版。因为 Thread-loader、HappyPack 这类组件所提供的并行能力都仅作用于文件加载过程，对后续 AST 解析、依赖收集、打包、优化代码等过程均没有影响；而 Parallel-Webpack 则可以影响整个过程。具体来说，Parallel-Webpack 实际上是创建了不同的独立进程来运行 webpack

如果要使用，首先安装：

```
yarn add -D parallel-webpack
```

然后最重要的是，导出类型必须是一个数组，即多个 webpack 配置对象。这也就说明 Parallel-Webpack 本质上是为每个配置对象创建一个进程，让他们独立、并行构建，最后统一输出。如果只导出一个对象，优化效果就不是很明显。

```js
module.exports = [
  {
    entry: "pageA.js",
    output: {
      path: "./dist",
      filename: "pageA.js",
    },
  },
  {
    entry: "pageB.js",
    output: {
      path: "./dist",
      filename: "pageB.js",
    },
  },
];
```

#### 并行压缩

Webpack5 使用 Terser 实现了多进程并行压缩能力。

TerserWebpackPlugin 插件默认已开启并行压缩，开发者也可以通过 parallel 参数（默认值为 require('os').cpus() - 1）设置具体的并发进程数量，如：

```js
const TerserPlugin = require("terser-webpack-plugin");

module.exports = {
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        parallel: 2, // number | boolean
      }),
    ],
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

还可以设置 noParse 属性跳过文件编译。**noParse 适合完全独立、不依赖于外部模块的包，比如 jQuery、lodash 等。**
可以在 noParse 属性中添加相应的路径，以跳过构建。这样在我们编写的代码中如果引入了该模块，那么就会跳过 webpack 对它的解析，而是直接使用该文件。
具体来说，noParse 的作用是，忽视指定文件中的导入、导出，相当于不去解析这个文件中的导入导出，但不会影响 loader、plugin 的处理过程，也不会影响文件本身的 ast 构建等。
比如设置了 lodash，那么就会忽视 lodash/index.js 中的所有静态、动态、cjs 导入，只是解析 index.js 本身。

```js
// webpack.config.js
module.exports = {
  //...
  module: {
    noParse: /lodash|react/,
  },
};
```

举个例子，如果我们想用 noParse 跳过 react 的构建过程，就不能直接使用`'react'`这样的导入；因为这种实际上是`node_module/react/index.js` 文件，包含了模块导入语句 require。
而真正没有导入导出的文件是 react.development.js 和 react.production.min.js。因此我们需要我们可以先找到适用的代码文件，然后用 resolve.alias 配置项重定向到该文件；这样在文件中引用 react 时，引用的实际上就是 noParse 的 react.development.js 或 react.production.min.js 文件，相当于跳过了 react 的构建。

```js
// webpack.config.js
module.exports = {
  // ...
  module: {
    noParse: /react|lodash/,
  },
  resolve: {
    // 注意：不一定需要，有些库（比如lodash）的index.js可能不能作为别名的入口
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

除了 noParse，还有一个 externals 属性也可以设置不处理部分模块。
区别在于，externals 配置的实际上相当于完全不打包，即完全忽视掉指定的模块，转而让用户通过 cdn 或某些方式将其继续融入到用户依赖中去。而 noParse 实际上只是无视了模块内的依赖，还不至于完全不处理。

```js
module.exports = {
  //...
  externals: {
    jquery: "jQuery",
  },
};

// html
<script
  src="https://code.jquery.com/jquery-3.1.0.js"
  integrity="sha256-slogkvB1K3VOkzAI8QITxV3VzpOnkeNVsKvtkYLMjfk="
  crossorigin="anonymous"
></script>;
```

这里的意思就是，把 jQuery 模块

#### 最小化 watch 监控范围

在 watch 模式下（通过 npx webpack --watch 命令启动），Webpack 会持续监听项目目录中所有代码文件，发生变化时执行 rebuild 命令。

不过，通常情况下前端项目中部分资源并不会频繁更新，例如 node_modules ，此时可以设置 watchOptions.ignored 属性忽略这些文件，例如：

```js
// webpack.config.js
module.exports = {
  //...
  watchOptions: {
    ignored: /node_modules/,
  },
};
```

### optimization 部分配置

常见的构建速度优化，适用于开发环境

- optimization.removeAvailableModules: false，如果一个模块已经包含在所有父级模块中，就从当前 chunk 中检查出该模块并剔除。这个过程能减少构建大小，但是很耗时。应该设置为 false
- optimization.removeEmptyChunks: false，移除空 chunk，同样是设置为 false 可以加快
- optimization.splitChunks: false，关闭代码分割
- optimization.minimize: false，关闭代码压缩
- optimization.usedExports: false，如果为 true 则会检查每个 chunk，未使用的导出内容不会被生成，有利于 treeshaking 这种清除无用代码的工具。

### preload & prefetch

动态导入的 import 函数可以通过添加注释来控制模块的 preload 和 prefetch。实际上就是创建一个包含 preload、prefetch 属性的 link 标签并插入到 html 中，因此含义和在 html 中的相同。
使用方式：

```js
import(/* webpackPrefetch: true */ "./path/to/LoginModal.js");
```

这会生成 `<link rel="prefetch" href="login-modal-chunk.js">` 并追加到页面头部

关于两者的区别：

- preload chunk 会在父 chunk 加载时，以并行方式开始加载。prefetch chunk 会在父 chunk 加载结束后开始加载。
- preload chunk 具有中等优先级，并立即下载。prefetch chunk 在浏览器闲置时下载。
- preload chunk 会在父 chunk 中立即请求，用于当下时刻。prefetch chunk 会用于未来的某个时刻。

### 缓存（webpack 持久化缓存）

webpack5 较于 webpack4,新增了持久化缓存、改进缓存算法等优化,通过配置 webpack 持久化缓存,来缓存生成的 webpack 模块和 chunk,改善下一次打包的构建速度,可提速 90% 左右

```js
// webpack.base.js
// ...
module.exports = {
  // ...
  cache: {
    type: "filesystem", // 使用文件缓存
  },
};
```

缓存的存储位置在 node_modules/.cache/webpack,里面又区分了 development 和 production 缓存

当然缓存并不是一项完全完美的工具。对 webpack 来说，他并非默认开启，就一定说明它存在一些问题。具体可以看这篇文章：https://github.com/webpack/changelog-v5/blob/master/guides/persistent-caching.md

## 产物优化

产物优化主要针对于生产环境的构建，即通过压缩、剔除冗余代码的形式，减小打包结果，尽可能降低生产环境产物的体积。
需要注意的是，这些优化方式虽然减少体积，但是对开发环境并不是很好，因为这些功能都要花费大量的时间。在开发环境下，反而应该去掉 Tree-Shaking、SplitChunks、Minimizer 等压缩类插件，以及关闭一些 optimization 的配置项。

### 压缩代码

#### 压缩 html

现代 Web 应用大多会选择使用 React、Vue 等 MVVM 框架，这衍生出来的一个副作用是原生 HTML 的开发需求越来越少，HTML 代码占比越来越低，所以大多数现代 Web 项目中其实并不需要考虑为 HTML 配置代码压缩工作流。
不过凡事都有例外，某些场景如 SSG 或官网一类偏静态的应用中就存在大量可被优化的 HTML 代码

html-minifier-terser 就是一个压缩 html 的工具。需要借助借助 html-minimizer-webpack-plugin 插件接入 html-minifier-terser

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

主要通过 optimization 选项开启。多数情况下使用默认 Terser 配置即可，也可以向 TerserPlugin 传入更详细的配置

```js
module.exports = {
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            reduce_vars: true,
            pure_funcs: ["console.log"],
          },
        },
      }),
    ],
  },
};
```

压缩之后的 js 会变得非常简单，大量变量名、函数名被重新命名，并且会合并到同一行

#### 压缩 css

压缩 css 需要将 css 从 js 中分离出来，方法是采用 MiniCssExtractPlugin，可以参考通用优化里的代码分割分离 css。

配置好 MiniCssExtractPlugin 后，在 minimizer 中引入`CssMinimizerPlugin`即可。

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
    minimizer: [new CssMinimizerPlugin()],
  },
  plugins: [new MiniCssExtractPlugin()],
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
不过这时函数的生命依然还存在，但他已成为了“未使用”的变量。在之后的 terser 压缩中，将会删去这种未使用的变量。

假如没有开启 tree-shaking，这里的导出对象则会包含 sqrt 函数，就像这样：
![](https://pic.imgdb.cn/item/637e243b16f2c2beb1700e7c.jpg)

因此，treeshaking 的优化并不是直接把没用的导出对象从打包结果中删去，而是将它从导出对象中删去

#### 原理

tree-shaking 不是某个插件或 loader，而是 webpack 内部的一种机制，因此 tree-shaking 的原理和 webpack 的工作流程分不开。
Tree-Shaking 的实现大致上可以分为三个步骤：

1. make 阶段，「收集」 模块导出变量并记录到模块依赖关系图 ModuleGraph 对象中；
1. seal 阶段，遍历所有模块，「标记」 模块导出变量有没有被使用；
1. 使用代码优化插件 —— 如 Terser，删除无效导出代码。

##### make 阶段

1. 将模块的所有 ESM 导出语句转换为 Dependency 对象，并记录到 module 对象的 dependencies 集合，转换规则：

- 具名导出转换为 HarmonyExportSpecifierDependency 对象；
- default 导出转换为 HarmonyExportExpressionDependency 对象。

例如对于下面的模块：

```js
export const bar = "bar";
export const foo = "foo";

export default "foo-bar";
```

对应的 dependencies 值为：

![](https://pic.imgdb.cn/item/63e32a084757feff33c82a78.jpg)

2. 所有模块都编译完毕后，触发 compilation.hooks.finishModules 钩子，开始执行 FlagDependencyExportsPlugin 插件回调；
3. FlagDependencyExportsPlugin 插件 遍历 所有 module 对象；
4. 遍历 module 对象的 dependencies 数组，找到所有 HarmonyExportXXXDependency 类型的依赖对象，将其转换为 ExportInfo 对象并记录到 ModuleGraph 对象中。

这一步结束之后，moduleGraph 中就包含了各个模块的导出情况

##### seal 阶段

接下来，Webpack 需要再次遍历所有模块，逐一标记出模块导出列表中，哪些导出值有被其它模块用到，哪些没有。然后在 seal 阶段最后的生成代码中，针对两种不同的导出值生成不同的语句
这个过程主要发生在 FlagDependencyUsagePlugin 插件中：

1. 触发 compilation.hooks.optimizeDependencies 钩子，执行 FlagDependencyUsagePlugin 插件回调；
2. 在 FlagDependencyUsagePlugin 插件中，遍历每一个 module 对象的 exportInfo 数组，为每一个 exportInfo 确定其对应的 dependency 对象有否被其它模块使用；如果导出被其他模块使用，就将其标记为已被使用；
3. 修改 exportInfo.\_usedInRuntime 属性，记录该导出被如何使用。执行完毕后，Webpack 会将所有导出语句的使用状况记录到 exportInfo.\_usedInRuntime 字典中。

然后到了代码生成阶段，即执行 compilation.codeGeneration 函数生成最终代码的时候。这部分具体逻辑由导出语句对应的 HarmonyExportXXXDependency 类实现。简单说，这一步的逻辑就是，用前面收集好的 exportsInfo 对象为模块的导出值分别生成导出语句。具体流程为：

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

需要注意的是，最新版的 webpack5 实际上已经在生产模式下自动进行了 tree-shaking。也就是说，即使你没有设置任何的 tree-shaking 选项，生产模式的打包结果依然会进行 tree-shaking，包括自动删除死代码、分析副作用结构等。

#### 实践

tree-shaking 虽然可以保证在大多数情况下实现 shaking 效果，但是还有部分情况可能会影响效果，在实践中应该注意这些情况

1. 无意义赋值，导致导入模块没有被使用，只是单纯的赋值

比如：

```js
// bar.js
export const bar = "bar";
export const foo = "foo";

// index.js
import { bar, foo } from "./bar.js";
const f = foo;
```

这里的 f 变量虽然引用了 foo 导出，但是 f 实际上也没有被使用。这种情况下 treeshaking 不会把 foo 看作是没使用的导出，因此 tree-shaking 就不会剔除 foo 的导出。

2. 标记纯函数或副作用

默认情况下 Webpack 并不会对函数调用做 Tree Shaking 操作，因为函数可能产生副作用，因此 tree-shaking 不会删去无用的函数导出。
如果希望删去函数，就需要指明这个函数没有副作用，可以放心删除。
方法有两个

- 用 sideEffects 标记，详见下
- 在**调用**函数的语句前面加上注释`/*#__PURE__*/`，明确告诉 Webpack 该次函数调用并不会对上下文环境产生副作用

![](https://pic.imgdb.cn/item/63e32f4c4757feff33d16e4f.jpg)

> `/*#__PURE__*/`注释的主要作用，是告诉webpack这个函数是纯函数，可以安全地被内联。
> 所谓“内联”，就是指将函数内的代码直接放到调用的位置。比如：
> ```js
> function add(a, b) {
>   return a + b;
> }
> 
> var result = add(1, 2);
> // 如果编译器进行函数内联优化，可以将代码转换为：
> 
> var result = 1 + 2;
> ```
> 如果函数调用前面加上PURE，webpack就会放心删去原函数体，转而在其他使用该函数的地方使用内联形式。
> 另外，如果该函数是从其他模块调用的，这个注释也会告诉webpack该函数在自己的模块中不会产生副作用。这样webpack就可以对模块进行tree-shaking，而不用担心模块内部导出的函数在外部调用时会有副作用的风险。


3. 禁止 babel 编译 esm 语句。需要将`@babel/preset-env`的配置中的 modules 属性改为 false

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

4. 使用 es6 模块的库。比如如果想对 lodash 进行 shaking 就应该采用 es6 模块的 lodash-es。

#### sideEffects

sideEffects 的配置，用于处理导出中包含副作用的情况。sideEffects 配置的目的，是开发者在知道某些模块包含副作用的前提下，希望 webpack 正常 tree-shaking 掉这些包含副作用的模块。也就是说相当于是“刻意”指定 webpack 去删除它之前不会删除的副作用代码。
所谓副作用其实就是当执行某个模块时，该模块除了返回导出值之外，还会产生一些额外影响，比如修改全局作用域下的变量等。

举个例子，在 src 下创建一个 polyfill.js

```js
window.a = "aaaaaaaaa";
```

然后在 index.js 中引入它：

```js
import "./polyfill";
```

在 index.js 中，仅仅只是引入这个模块，而并没有使用模块导出的任何内容。但 webpack 分析 polyfill.js 中包含副作用，因此它不会将其 tree-shaking 掉。
我们开始 minisizer，然后查看输出：

```js
(self.webpackChunkwebpack_test2 = self.webpackChunkwebpack_test2 || []).push([
  ["app"],
  {
    "./src/polyfill.js": () => {
      window.a = "aaaaaaaaa";
    },
  },
  (s) => {
    s((s.s = "./src/index.js"));
  },
]);
```

可以看到这里有一行函数，用于执行 polyfill 中的副作用代码。
但是现在我们作为开发者，不希望这行代码的副作用生效，也就是说希望删除这段代码。webpack 不会自动删除，因为它认为这是副作用。
因此需要在 packages.json 中开启一个选项：

```json
{
  "sideEffects": false
}
```

如果所有代码都不包含副作用，我们就可以简单地将该属性标记为 false，来告知 webpack 它可以安全地删除未用到的 export。
相反，如果有一些代码有副作用，就需要在这里指明具体的路径，防止 webpack 将其 shaking 掉。

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

当设置完成 sideEffects 后，其余没有包含在内的文件都被视为是无副作用的，这样 webpack 就可以放心地进行 tree-shaking 了。

比如上面的例子，开启 sideEffects 之后，可以看到被删除：

```js
(self.webpackChunkwebpack_test2 = self.webpackChunkwebpack_test2 || []).push([
  ["app"],
  (s) => {
    s((s.s = "./src/index.js"));
  },
]);
```

### gzip

前端代码在浏览器运行,需要从服务器把 html,css,js 资源下载执行,下载的资源体积越小,页面加载速度就会越快。一般会采用 gzip 压缩,现在大部分浏览器和服务器都支持 gzip,可以有效减少静态资源文件大小,压缩率在 70% 左右。
nginx 可以配置 gzip: on 来开启压缩,但是只在 nginx 层面开启,会在每次请求资源时都对资源进行压缩,压缩文件会需要时间和占用服务器 cpu 资源，更好的方式是前端在打包的时候直接生成 gzip 资源,服务器接收到请求,可以直接把对应压缩好的 gzip 文件返回给浏览器,节省时间和 cpu。
webpack 可以借助 compression-webpack-plugin 插件在打包时生成 gzip

```js
const CompressionPlugin = require("compression-webpack-plugin");
module.exports = {
  // ...
  plugins: [
    // ...
    new CompressionPlugin({
      test: /.(js|css)$/, // 只生成css,js压缩文件
      filename: "[path][base].gz", // 文件命名
      algorithm: "gzip", // 压缩格式,默认是gzip
      test: /.(js|css)$/, // 只生成css,js压缩文件
      threshold: 10240, // 只有大小大于该值的资源会被处理。默认值是 10k
      minRatio: 0.8, // 压缩率,默认值是 0.8
    }),
  ],
};
```

配置完成后再打包,可以看到打包后 js 的目录下多了一个 .gz 结尾的文件

### 代码分割

代码分割是 webpack 最常见的减少打包体积的方式。同时，代码分割也可以减少主模块的大小，从而加快浏览器加载页面。
代码分割的主要方式有三个：

1. 多个 entry 手动分割。这种方式一般是额外创建几个入口文件，然后把部分代码移入。也可以通过 dependOn 的形式将公共依赖单独打包。这种方式比较麻烦而且容易出错
2. 动态导入。通过 import 语法，webpack 会自动为动态导入的模块创建 async chunk，从而减小核心模块大小。动态导入的对象可以是 react 组件、大型的库、资源文件等。
3. splitChunk。splitChunk 实际上是在 chunk 的单位上进行分割，可以将 chunk 之间公共的依赖模块单独拆分到一个 chunk 中，也可以直接拆分 node_modules 中的某些库和模块。比如可以通过`chunks: 'all'`，webpack 将找出所有 chunk 中重复使用的 module，将其单独打包或统一打包到一个 bundle 中，减少了模块冗余。

#### SplitChunksPlugin

SplitChunksPlugin 是 Webpack 4 之后内置实现的最新分包方案，它能够基于一些更灵活、合理的启发式规则将 Module 编排进不同的 Chunk，最终构建出性能更佳，缓存更友好的应用产物。

SplitChunksPlugin 是为了解决 chunk 构建的问题的。chunk 是 module 对象的集合，通常是 module 的构建结果。从模块类型可以将 chunk 类型分为三种：

- Initial Chunk：entry 模块及相应子模块打包成 Initial Chunk；
- Async Chunk：通过 import('./xx') 等语句导入的异步模块及相应子模块组成的 Async Chunk；
- Runtime Chunk：运行时代码抽离成 Runtime Chunk，即设置 entry.runtime 时就会生成

Initial Chunk 与 Async Chunk 这种略显粗暴的规则会带来两个明显问题：

1. 模块重复打包：

假如多个 Chunk 同时依赖同一个 Module，那么这个 Module 会被不受限制地重复打包进这些 Chunk，这样对于多个 chunk 来说，每个都包含了相同的 module，就会造成体积增大
![](https://pic.imgdb.cn/item/63e103c24757feff33a5310e.jpg)

2. 所有模块都被打入同一个包。

Async Chunk 默认会和 Initial Chunk 打包在一起，如果没有多个入口的话，那么最终的 chunk 可能只有一个。对于一个庞大的项目来说，打包在一起是很致命的：

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
};
```

##### 设置分包范围

SplitChunksPlugin 默认情况下只对 Async Chunk 生效，我们可以通过 splitChunks.chunks 调整作用范围。

- `'all'` ：对 Initial Chunk 与 Async Chunk 都生效，建议优先使用该值；
- `'initial'` ：只对 Initial Chunk 生效；
- `'async'` ：只对 Async Chunk 生效；
- 函数 `(chunk) => boolean` ：该函数返回 true 时生效；

> 这三个值用于配置，对什么类型的 chunk，将其公共模块单独打包到一个 chunk 中去。
> 比如一次打包过程中有 5 个 chunk，其中有三个是动态导入产生的 async chunk，其他的是 initial chunk
>
> - async:默认值，这三个 async chunk 才会被处理，webpack 将这几个 chunk 之间相同的依赖单独打包，防止重复
> - all: webpack 在所有模块之间处理，即包括 async 和 initial，把这些 chunk 之间重复引用的模块单独打包。
> - initial: 只对初始模块生效。

##### 根据 Module 使用频率分包

SplitChunksPlugin 支持按 Module 被 Chunk 引用的次数决定是否分包，借助这种能力我们可以轻易将那些被频繁使用的模块打包成独立文件，减少代码重复。

```js
module.exports = {
  //...
  optimization: {
    splitChunks: {
      // 设定引用次数超过 2 的模块才进行分包
      minChunks: 2,
    },
  },
};
```

##### 限制分包体积

Webpack 提供了一系列与 Chunk 大小有关的分包判定规则，借助这些规则我们可以实现当包体过小时直接取消分包，防止产物过"碎"；
当包体过大时尝试对 Chunk 再做拆解，避免单个 Chunk 过大。

这一规则相关的配置项有：

- minSize： 超过这个尺寸的 Chunk 才会正式被分包；
- maxSize： 超过这个尺寸的 Chunk 会尝试进一步拆分出更小的 Chunk，但不会分割出小于 minSize 的 chunk。设置 maxSize 形成的零碎 chunk 有利于 http/2 的缓存优化，也可以减小 chunk 大小。
- maxAsyncSize： 与 maxSize 功能类似，但只对异步引入的模块生效；
- maxInitialSize： 与 maxSize 类似，但只对 entry 配置的入口模块生效；
- enforceSizeThreshold： 超过这个尺寸的 Chunk 会被强制分包，忽略上述其它 Size 限制。

##### 设置 cacheGroup

splitChunks.cacheGroups 可以自定义打包范围。举个例子：

```js
module.exports = {
  //...
  optimization: {
    splitChunks: {
      cacheGroups: {
        commons: {
          test: /[\\/]node_modules[\\/]/,
          name: "vendors",
          chunks: "all",
        },
      },
    },
  },
};
```

这里我们在 cacheGroup 内创建了一个对象 commons（这个名字不重要），内部选择了`node_modules`目录作为分包的对象，采取 all 的分包方式。
这里每个对象可以看作是要分出来的一个 chunk。比如这里就会创建一个名为 vendor 的 chunk，相当于把项目中引用的所有`node_modules`内的模块打包到一个 chunk 中。

这种方式会把依赖和源代码拆成两个 chunk。因为依赖通常不会变化，而源代码变化较多，因此这种方式可以加快主应用的构建速度，减少主应用的包的大小。

或者单独把 react 打包：

```js
module.exports = {
  entry: {
    main: ".src/index.js",
  },
  output: {
    filename: "[name].bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
  optimization: {
    splitChunks: {
      cacheGroups: {
        react: {
          test: /[\\/]node_modules[\\/](react|react-dom)[\\/]/,
          name: "react",
          chunks: "all",
        },
      },
    },
  },
};
```

通过这种方式，可以任意打包自己想打包的指定的模块。

#### 动态导入

即 webpack 的代码分离。原理是利用 ES6 的动态导入实现按需加载，而不是任何情况都在文件头部静态导入
如果通过 import()动态导入一个模块，那么在构建期间就会产生额外的产物文件，即单独生成动态导入的模块的文件。
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

### source map

在 生产环境 中使用 source-map 选项。
避免在生产中使用 inline-*** 和 eval-***，因为它们会增加 bundle 体积大小，并降低整体性能。

## 通用优化

### 缓存（http）

> 我们使用 webpack 来打包我们的模块化后的应用程序，webpack 会生成一个可部署的 /dist 目录，然后把打包后的内容放置在此目录中。只要 /dist 目录中的内容部署到 server 上，client（通常是浏览器）就能够访问此 server 的网站及其资源。而最后一步获取资源是比较耗费时间的，这就是为什么浏览器使用一种名为 缓存 的技术。可以通过命中缓存，以降低网络流量，使网站加载速度更快，然而，如果我们在部署新版本时不更改资源的文件名，浏览器可能会认为它没有被更新，就会使用它的缓存版本。由于缓存的存在，当你需要获取新的代码时，就会显得很棘手。

如果想充分利用好 webpack 的缓存，最主要的方向有两个：

1. 代码分离。把经常变化和不经常变化的代码分离开
2. 文件名优化。对于经常变化的文件，其输出需要带上诸如 contenthash 这样的文件名，从而保证其文件内容改变时可以及时重新请求

#### 代码分割

代码分割的方式有三种，而对于缓存来说最主要的就是将不经常变动的库的代码和源代码分割。
分割库内的代码的方式是使用 splitChunk:

```js
optimization: {
  runtimeChunk: 'single', // runtime代码也可以单独分离
  splitChunks: {
    cacheGroups: {
      vendor: {
        test: /[\\/]node_modules[\\/]/,
        name: 'vendors',
        chunks: 'all',
      },
    },
  },
    },
```

这样的配置会把整个 node_modules 里的各种库打包到 vendors.js 文件中去。

但是这时还有一个问题：当源代码中文件导入变动时，由于模块解析顺序发生变化，每个 module.id 也会发生变化，这就导致 webpack 认为模块发生了变动，可能会重新编译 vendor.js。
我们需要让`vendor.[contenthash].js`文件名不会变动，即 vendor 不要重新编译，还需要配置

```js
optimization: {
  moduleIds: 'deterministic',
  runtimeChunk: 'single',
  splitChunks: {
    ...
  },
},
```

这个每个模块的 id 就不会随着文件导入导出顺序的变化而变化。

配置好缓存后，通过 devserver 启动并查看控制台的文件。当修改 app 中的代码内容时，可以看到 vendor 是由 304 缓存后的文件，而 runtime 和源代码 app 则是 200，表示每次都是重新请求；但当修改对不同库的引入时，比如引入 lodash 或不引入 lodash，会发现 vendor 也会被重新请求。

![](https://pic.imgdb.cn/item/6416cde6a682492fccb6bead.jpg)

---

除了分割 js 代码，css 代码也可以被分割。方法就是使用 mini-css-extract-plugin，然后用 MiniCssExtractPlugin.loader 代替 style-loader，并且添加对应的 plugin

```js
module: {
  rules: [
    {
      test: /\.css$/,
      use: [MiniCssExtractPlugin.loader, "css-loader"],
    },
  ],
},
plugins: [
  new MiniCssExtractPlugin({
    filename: "static/css/[name].css",
  }),
],
```

配置完成后,在 css 会嵌入到 style 标签里面,方便样式热替换,打包时会把 css 抽离成单独的 css 文件。

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

webpack 集成了很多功能，包括模块打包、代码分割、按需加载、devserver 等等，但是最核心的部分还是它的静态模块打包能力。即，Webpack 能够将各种类型的资源 —— 包括图片、音视频、CSS、JavaScript 代码等，通通转译、组合、拼接、生成标准的、能够在不同版本浏览器兼容执行的 JavaScript 代码文件，这一特性能够轻易抹平开发 Web 应用时处理不同资源的逻辑差异，使得开发者以一致的心智模型开发、消费这些不同的资源文件

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

webpack 有几个基础对象贯穿工作流程始终，包括：

- `Entry`：编译入口。初始化是从配置中提取到的 entry 对象
- `Compiler`：编译管理器，Webpack 启动后会创建 compiler 对象，该对象一直存活直到构建结束进程退出；compiler 对象包含非常多的内容，包括：
  - 各种 hook
  - 整个编译过程的各种参数，比如配置 options
  - 编译要用到的方法，最主要的是 compile 方法，还有 run、emitAssets 等方法
- `Compilation`：单次构建过程的管理器，主要用于构建过程，内容也主要是供构建过程使用的参数和方法，比如：
  - addModuleTree 方法将文件转化为 module 并建立 moduleTree
  - addEntry 方法每次调用 compile 方法都会创建一个 compilation 对象，并且如果开启文件监听，那么每次文件刷新都会再创建 compilation 对象。
- `Dependence`：依赖对象，记录模块间依赖关系；在进入构建阶段时的 EntryPlugin 就会从 entry 解析依赖关系，创建 dependence 对象；后续的构建阶段就会根据这个对象确定的依赖关系来递归把文件解析为 module。
- `Module`：Webpack 内部所有资源都会以 Module 对象形式存在，所有关于资源的操作、转译、合并都是以 Module 为单位进行的；
- `Dependency Graph`：以一套独立的 Graph 数据结构记录模块间依赖关系，并基于 Map/Set 等原生模块实现更高效的模块搜索、校验、遍历算法。
- `Chunk`：编译完成准备输出时，将 Module 按特定的规则组织成一个一个的 Chunk。
- `ChunkGraph`：所有关于 module 如何与 chunk 连接的信息都存储在 ChunkGraph 类中。
- `ChunkGroup`：一个 ChunkGroup 内包含一个或多个 Chunk 对象。比如一个inital chunk就是一个chunk group，其后的async chunk也会成为一个chunk group。

graph 类型的数据都是为了更好的显示关系的。在 webpack5 之前，module 之间的关系都是直接使用 module 和 dependency 来得出的，而 webpack5 则对这种关系做了加深，通过图的形式使得 module 之间、module 和 chunk 之间的关系更加清晰

#### DependencyGraph

Dependency Graph并不是显式的数据结构，而是多种数据结构的结合，形成以 entry 为起点，以模块为节点，以导入导出依赖为边的有向图关系。

![](https://pic.imgdb.cn/item/6416fdf5a682492fcc09693e.jpg)

DependencyGraph主要包含三个结构，其中ModuleGraph是主要结构，后两个结构是ModuleGraph内部的结构。

- `ModuleGraph`：记录 Dependency Graph 信息的容器，记录构建过程中涉及到的所有 module、dependency 对象，以及这些对象互相之间的引用；
- `ModuleGraphConnection` ：记录**模块间引用关系**的数据结构
- `ModuleGraphModule` ：Module 对象在 Dependency Graph 体系下的补充信息，包含模块对象的 incomingConnections —— 指向模块本身的 ModuleGraphConnection 集合，即谁引用了模块自身；outgoingConnections —— 该模块对外的依赖，即该模块引用了其他那些模块。


#### ModuleGraph

参考https://juejin.cn/post/7138285996500025352

moduleGraph 实际上是几个 module 之间的依赖关系的记录的数据结构的集合。具体来说，主要包含了两个重要属性：

- \_dependencyMap ：记录 dependency对象 与 module 连接关系的信息。注意这个是入口文件的 dependency 和入口 module 的关系信息，它只会记录当前 module 和一个引用当前 module 的 module

```js
{
	module:  Module, // 当前module
	originModule： Module// 引用当前module的module
}

_dependencyMap:{
    <dep-index, connection{originModule: undefined, module: mod-index}>,
}
```

- \_moduleMap ：记录当前 module 被谁引用以及引用了谁。moduleMap 内的数据结构如下

```js
{
	inComingConnections:[], // 表示一个有哪些modules引用了当前module
	outComingConnections:[], // 表示一个当前module引用了哪些modules
}
```

当 moduleGraph 被确定后，所有 module 之间的依赖关系、导入导出值都可以从其中直接获得。

这两个数据结构有点像。不过_dependencyMap是module和dependency对象的对应，通常只包含直接连接的dependency对象（一个）；而_moduleMap则是module引用关系的集合（数组）。

举个例子，有一组这样的模块：
![](https://pic.imgdb.cn/item/6416fe26a682492fcc09b70a.jpg)

他们形成的moduleGraph就像这样：
```js
ModuleGraph: {
    _dependencyMap: Map(3){
        { 
            EntryDependency{request: "./src/index.js"} => ModuleGraphConnection{
                module: NormalModule{request: "./src/index.js"},  // 当前module
                // 入口模块没有引用者，故设置为 null
                originModule: null // 引用该module的module
            } 
        },
        { 
            HarmonyImportSideEffectDependency{request: "./src/a.js"} => ModuleGraphConnection{
                module: NormalModule{request: "./src/a.js"}, 
                originModule: NormalModule{request: "./src/index.js"}
            } 
        },
        { 
            HarmonyImportSideEffectDependency{request: "./src/a.js"} => ModuleGraphConnection{
                module: NormalModule{request: "./src/b.js"}, 
                originModule: NormalModule{request: "./src/index.js"}
            } 
        }
    },

    _moduleMap: Map(3){
        NormalModule{request: "./src/index.js"} => ModuleGraphModule{
            incomingConnections: Set(1) [ // 哪些module引用了当前module
                ModuleGraphConnection{ module: NormalModule{request: "./src/index.js"}, originModule:null }
            ],
            outgoingConnections: Set(2) [ // 当前module引用了哪些module
                // 从 index 指向 a 模块
                ModuleGraphConnection{ module: NormalModule{request: "./src/a.js"}, originModule: NormalModule{request: "./src/index.js"} },
                // 从 index 指向 b 模块
                ModuleGraphConnection{ module: NormalModule{request: "./src/b.js"}, originModule: NormalModule{request: "./src/index.js"} }
            ]
        },
        NormalModule{request: "./src/a.js"} => ModuleGraphModule{
            incomingConnections: Set(1) [
                ModuleGraphConnection{ module: NormalModule{request: "./src/a.js"}, originModule: NormalModule{request: "./src/index.js"} }
            ],
            // a 模块没有其他依赖，故 outgoingConnections 属性值为 undefined
            outgoingConnections: undefined
        },
        NormalModule{request: "./src/b.js"} => ModuleGraphModule{
            incomingConnections: Set(1) [
                ModuleGraphConnection{ module: NormalModule{request: "./src/b.js"}, originModule: NormalModule{request: "./src/index.js"} }
            ],
            // b 模块没有其他依赖，故 outgoingConnections 属性值为 undefined
            outgoingConnections: undefined
        }
    }
}
```

#### ChunkGraph

有关Chunk、ChunkGroup、ChunkGraph 三种关键对象：
Chunk：Module 用于读入模块内容，记录模块间依赖等；而 Chunk 则根据模块依赖关系合并多个 Module，输出成资产文件：

![](https://pic.imgdb.cn/item/6416ffe7a682492fcc0c75fb.jpg)

ChunkGroup：一个 ChunkGroup 内包含一个或多个 Chunk 对象；ChunkGroup 与 ChunkGroup 之间形成父子依赖关系：大多数情况下一个group内部就是一个chunk，但chunk本身是独立的，而chunkGroup则会形成依赖关系。

![](https://pic.imgdb.cn/item/6416fff4a682492fcc0c8ae0.jpg)

ChunkGraph：最后，Webpack 会将 Chunk 之间、ChunkGroup 之间的依赖关系存储到 compilation.chunkGraph 对象中，形成如下类型关系：

![](https://pic.imgdb.cn/item/6416fffda682492fcc0c9af5.jpg)

chunkgraph在生成阶段开始构建，以moduleGraph为基础。
ChunkGraph 就是以 chunk 为中心描绘 chunk 与 module 关系对象，可以理解为 webpack 的分包规则。
ChunkGraph 为例记录 chunk 和 module 的关系，就必然有这样的数据结构：

- 有哪些 chunk，chunk 里面有哪些 module
- 有哪些 module，module 属于哪些 chunk

而实际上 ChunkGraph 内部也正是两类数据结构：

- \_chunks：`Map<chunk, ChunkGraphChunk>`，ChunkGraphChunk 是记录一个 chunk 有哪些 module
- \_modules：`Map<modules, ChunkGraphModule>，`ChunkGraphModule 是记录一个 module 属于哪些 chunk。

在 seal 阶段会初始化并构建完整的 ChunkGraph，后续的 SpiltChunkPlugin 等分包手段也依赖于它。

### 初始化阶段

初始化阶段的主要功能是整理合并参数，然后根据参数创建 compiler 对象，并开始编译。

![](https://pic.imgdb.cn/item/63e1c1774757feff33ab514f.jpg)

在源码中，实际上是 webpack.js 中的 webpack 函数，大致如下：https://github1s.com/webpack/webpack/blob/HEAD/lib/webpack.js

```js
const webpack = (options, callback) => {
  const create = () => {
    // 初始化compiler
    let compiler;
    let watch = false;
    let watchOptions;
    if (Array.isArray(options)) {
      // 如果配置对象是数组，就创建多个compiler
      compiler = createMultiCompiler(options, options);
      watch = options.some((options) => options.watch);
      watchOptions = options.map((options) => options.watchOptions || {});
    } else {
      // 正常情况，创建compiler，传入配置对象，初始化watch对象
      const webpackOptions = options;
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
          compiler.close((err2) => {
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
};
```

而 createCompiler 函数功能如下：

1. 收集各种配置。除了配置对象之外，还包括 webpack 的默认配置、命令行配置等
2. 遍历 配置中的 plugins 集合，执行插件的 apply 方法。
3. 调用 new WebpackOptionsApply().process 方法，根据配置内容动态注入相应插件

```js
const createCompiler = (rawOptions) => {
  // 默认配置
  const options = getNormalizedWebpackOptions(rawOptions);
  applyWebpackOptionsBaseDefaults(options);
  const compiler = new Compiler(options.context, options);
  //  注入环境变量
  new NodeEnvironmentPlugin({
    infrastructureLogging: options.infrastructureLogging,
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
compile 方法没有具体逻辑，只是不断调用 hook 上的 call 方法执行回调，但他确定了整个编译过程的流程，使得编译过程一环套一环，过程如下：

1. 调用 newCompilation 方法创建 compilation 对象；
1. 触发 make 钩子，紧接着 EntryPlugin 在这个钩子中调用 compilation 对象的 addEntry 方法创建入口模块，主流程开始进入「构建阶段」；
1. make 执行完毕后，触发 finishMake 钩子；
1. 执行 compilation.seal 函数，进入「生成阶段」，开始封装 Chunk，生成产物；
1. seal 函数结束后，触发 afterCompile 钩子，开始执行收尾逻辑。

从触发 make hook 开始，就进入了构建阶段（make），初始化阶段完成

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
webpack 内部一个 plugin，EntryPlugin 会在 make hook 被触发时调用，它的主要做用是调用 addEntry 函数找到入口文件，然后解析入口文件的导入，得到入口文件的依赖。也就是从入口文件开始了构建过程

```js
class EntryPlugin {
  apply(compiler) {
    const { entry, options, context } = this;
    // 解析entry对象，创建dependence对象
    const dep = EntryPlugin.createDependency(entry, options);

    compiler.hooks.make.tapAsync("EntryPlugin", (compilation, callback) => {
      compilation.addEntry(context, dep, options, (err) => {
        callback(err);
      });
    });
  }
}
```

addEntry 内部会调用 addModuleTree 方法，它会创建并生成模块的依赖树。对每个依赖来说，都是调用 handleModuleCreation，根据文件类型构建 module 子类 —— 一般是 NormalModule

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

> acorn 需要将各种类型的模块内容（js、css、静态资源）解析为 AST 结构，要使用 loaders 将不同类型的资源转译为标准 JavaScript 代码，才能转化为 ast。

当得到入口 js 代码的 ast 之后，就是最关键的一步，即解析 ast，得到其中的模块依赖关系。具体来说是：

1. 解析 ast，解析过程中如果检查到了导入语句，就触发相关 hook。HarmonyExportDependencyParserPlugin 插件监听到 AST 解析钩子 exportImportSpecifier 则会回调 module.addDependency()将依赖对象添加到 module 的依赖列表 dependencies
2. 遍历依赖列表 dependencies，把每个 Dependency 转换为 Module 对象。
3. 对于每个 module 对象，就再递归处理（执行上面的两步），直到所有文件都被处理完毕。

最后，所有的文件都被构建成 module。其中第三步的递归，就是构建 module 的核心。注意这个过程中 module 和 dependences 数组的关系没有断掉，一个 module 会有它的 dependences 数组，表示它依赖的模块的 module；最后会形成 module tree 这样的结构。
在下一步生成 chunk 的过程中，会利用到 module 和 dependency 的关系，将对应的 module 关联起来。

具体递归过程可以参考https://juejin.cn/book/7115598540721618944/section/7119035873802813475

### 生成阶段

当构建阶段完成后，所有的文件都变成了 module；生成阶段则负责根据一系列内置规则，将上一步构建出的所有 Module 对象拆分编排进若干 Chunk 对象中，之后以 Chunk 粒度将源码转译为适合在目标环境运行的产物形态，并写出为产物文件

![](https://pic.imgdb.cn/item/63e264334757feff33c6d787.jpg)

生成阶段的名称是 seal，因此生成阶段的入口函数就是`compilation.seal`方法。这一步的代码很复杂，我们只关注流程：

1. 创建本次构建的 ChunkGraph 对象。ChunkGraph 是一个记录 chunk 和 module 关系的数据结构
2. 遍历 入口集合 compilation.entries：
3. 调用 addChunk 方法为每一个入口 创建 对应的 Chunk 对象（EntryPoint Chunk）；
4. 遍历该入口 module 对应的 Dependency 集合，找到相应 Module 对象并关联到该 Chunk。
5. 到这里可以得到若干 Chunk，之后调用 buildChunkGraph 方法将这些 Chunk 处理成 Graph 结构，方便后续处理。
6. 之后，触发 optimizeModules/optimizeChunks 等钩子，由插件（如 SplitChunksPlugin）进一步修剪、优化 Chunk 结构。
7. 一直到最后一个 Optimize 钩子 optimizeChunkModules 执行完毕后，开始调用 compilation.codeGeneration 方法生成 Chunk 代码

这一步完成之后，其实已经结束了 seal 阶段。seal 阶段的结果就是 chunk 的生成；接下来，需要将 chunk 变为实质输出的代码，将其写入文件系统中。这一步又被称为 emit 阶段。

6. 在 codeGeneration 回调中调用 createChunkAssets 函数，为每一个 Chunk 生成 assets 文件。这时还是一种“资源”类型的文件，还不是真正的代码文件。每个 chunk 都有对应的 asset
7. 调用 compilation.emitAssets 函数“提交”文件，触发 callback 回调，控制流回到 compiler 函数。
8. 最后调用 compiler 对象的 emitAssets 方法，将需要生成的文件代码写入文件系统。到这一步之后，就可以看到 webpack 的输出结果了。

---

seal 阶段的过程从 chunk 角度来说可以这样梳理：

![](https://pic.imgdb.cn/item/63e49d6c4757feff330dcd09.jpg)

1. 创建入口模块和初始化 chunkgraph：调用 seal() 函数后，遍历 entry 配置，为每个入口创建一个空的 Chunk 与 EntryPoint 对象（一种特殊的 ChunkGroup），并初步设置好基本的 ChunkGraph 结构关系，在之后会逐渐填充 chunkgraph 内容。
   这一步主要是将 entry 的几个入口模块生成 chunk，并放入 entryPoint 中，形成这样的结构：
   ![](https://pic.imgdb.cn/item/63e49db54757feff330e651f.jpg)
   若此时配置了 entry.runtime，Webpack 还会在这个阶段为运行时代码 创建相应的 Chunk 并直接注入到 entry 对应的 ChunkGroup 对象。一切准备就绪后调用 buildChunkGraph 函数，进入下一步骤。
2. 把所有 module 生成 chunk 并装入：在 buildChunkGraph 函数内遍历 ModuleGraph，将所有 Module 按照依赖关系分配给不同 Chunk 对象；这个过程中若遇到**异步模块**，则为该模块创建新的 ChunkGroup 与 Chunk 对象，形成如下结构：
   ![](https://pic.imgdb.cn/item/63e49e244757feff330f3da6.jpg)
3. 完善 chunkgraph 对象：在 buildChunkGraph 函数中调用 connectChunkGroups 方法，建立 ChunkGroup 之间、Chunk 之间的依赖关系，生成完整的 ChunkGraph 对象

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

loader 的执行对象是经历过 make 阶段之后生成的 module。loader 可以通过配置文件后缀确定接收哪类文件的 module，比如只接收 css 类型的 module、只接受 ts 类型的 module 等。因此 loader 函数会被调用 n 次，n 为符合类型要求的文件（module）数量
因此 loader 本身只用考虑单个文件的编译结果即可。比如 css-loader 只用考虑 css 文件的编译，babel-loader 不必关心 css 的导入结果。

Loader 接收三个参数，分别为：

- source：资源输入，对于第一个执行的 Loader 为资源文件的内容；后续执行的 Loader 则为前一个 Loader 的执行结果，可能是字符串，也可能是代码的 AST 结构；
- sourceMap: 可选参数，代码的 sourcemap 结构；
- data: 可选参数，其它需要在 Loader 链中传递的信息，比如 posthtml/posthtml-loader 就会通过这个参数传递额外的 AST 对象。

此外 loader 内部还有一部分上下文接口，即通过`this.xxx`获取的参数。这些参数来自于 webpack 运行时的上下文，也非常重要。参考https://webpack.docschina.org/api/loaders/
常用的一些上下文有：

- fs：Compilation 对象的 inputFileSystem 属性，我们可以通过这个对象获取更多资源文件的内容；
- resource：当前文件的绝对路径，包括 query 参数，例如 import "abc/a?foo=bar" 的 resource 值为 abc/a?foo=bar；
- resourcePath: 不包含 query 的绝对路径。大多数时候用的是这个，因为 query 参数很少用到
- callback：可用于返回多个结果；
  callback 函数参数为：

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
  - 可选的：第四个参数，会被 webpack 忽略，可以是任何东西（例如一些元数据）。比如希望在 loader 之间共享 ast，就可以通过这个接口传，不会影响到 webpack 本身，但是其他 loader 可以接收到。

  如果调用 callback 函数返回结果，那 loader 函数就不能返回任何值，防止造成混淆

- getOptions：用于获取当前 Loader 的配置对象；
- async：用于声明这是一个异步 Loader，开发者需要通过 async 接口返回的 callback 函数传递处理结果；
  大多数时候同步 loader 可以完成任务，但是有些库的解析过程本身就是异步的，比如 less 的编译、prettier 的处理、ast 的解析等过程的函数本身就是异步的。这时候就需要 loader 异步返回结果，而异步返回的 callback 就来自于 this.async 的返回值

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
- addDependency：在 loader 中手动添加依赖。比如 less-loader 会解析 less 文件中的 import 语句，然后把所有 import 的部分都添加到依赖，这样这些文件发生变化时也会触发重新编译。也可以添加对配置文件的依赖，比如 babel-loader 可以添加.babelrc 为依赖，当配置文件发生变化时就重新编译

### loader 编写示例 typing-for-css-modules-loader

源代码可以查看：https://github.com/penghei/css-modules-types-loader-generator

该 loader 的目的是自动为 css module 生成类型声明文件。
在 ts 中如果直接导入 css module 类型的模块，就会提示不是一个模块，因为 css module 没有类型，而我们需要对该模块声明一个类型。并且，还希望 styles 对象能包含具体的类名而不是一个 any 类型，这样通过 style 的语法提示就可以获取类名了
首先要清楚如何声明一个 css module。
如果我们有这样一个 css 文件：

```css
.container {
  ...;
}
.title {
  ...;
}
```

经过 css-loader 编译之后（cssloader 的 module 模式要开启），会变成这种形式：

```js
// Imports
import ___CSS_LOADER_API_NO_SOURCEMAP_IMPORT___ from "../node_modules/css-loader/dist/runtime/noSourceMaps.js";
import ___CSS_LOADER_API_IMPORT___ from "../node_modules/css-loader/dist/runtime/api.js";
var ___CSS_LOADER_EXPORT___ = ___CSS_LOADER_API_IMPORT___(
  ___CSS_LOADER_API_NO_SOURCEMAP_IMPORT___
);
// Module
___CSS_LOADER_EXPORT___.push([
  module.id,
  ".container_qe3E3 {\n  display: flex;\n}\n.container_qe3E3 .title_Q9ieQ {\n  color: green;\n}\n",
  "",
]);
// Exports
___CSS_LOADER_EXPORT___.locals = {
  container: "container_qe3E3",
  title: "title_Q9ieQ",
};
export default ___CSS_LOADER_EXPORT___;
```

`___CSS_LOADER_EXPORT___.locals`对象包含了 css 文件中的类选择器的类名。css 中没有使用类名选择器，而是直接元素或 id 选择器，就不会放入 locals 中。因此我们获取类名的范围应该是在 locals 对象中
然后通过 style-loader 不加额外处理插入到原本的 js module 中。这时显然`___CSS_LOADER_EXPORT___`这个对象就是我们导出的 styles 对象

```js
import styles from "./app.module.css";
```

我们希望给`___CSS_LOADER_EXPORT___`对象添加类型，以保证 styles 也是有类型的。因此可以编写一个声明文件

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

这样就可以看到在 ts 文件中没有报错了，并且 styles 对象也有了类型。

我们的目的就是通过 loader 自动生成这样的一个类型声明文件。基本逻辑：

1. css-loader 已经生成了 js 形式的 css 代码。从上可以看出`___CSS_LOADER_EXPORT___.locals`对象包含了我们所需要的类名，因此我们可以用正则提取其中的类名，作为生成的类名使用
2. 输出形式也已经确定好了，即上面的形式，我们可以通过模板字符串插入需要的变量名和类名，然后修改生成文件的文件名为`xxx.d.ts`，最后通过文件系统 fs.writeFileSync 将其写出到相同目录即可。

```js
const path = require("path");
const fs = require("fs");
const { getSelectorKeys, generateOutputString } = require("./utils");
const schema = require("./schema");
const { validate } = require("schema-utils");

module.exports = function (content) {
  const options = this.getOptions() || {};
  // 使用schema-utils进行options校验
  validate(schema, options, {
    name: "css-modules-types-loader-generator",
    strict: false,
  });

  if (this.cacheable) {
    this.cacheable();
  }

  const sourceFilename = this.resourcePath;
  const dirname = path.dirname(sourceFilename);
  const outputFilename = `${path.basename(sourceFilename)}.d.ts`;
  const outputFullFilename = path.join(dirname, outputFilename);

  const logger = this.getLogger("css-modules-types-loader-generator");
  const cssSelectorNames = getSelectorKeys(content);
  // 错误检查和报错
  if (cssSelectorNames.length === 0) {
    logger.warn(
      `${outputFilename} appears to be an empty file or this loader is not used after css-loader, nothing will be generated.`
    );
    return content;
  }
  const outputContent = generateOutputString(
    outputFilename,
    cssSelectorNames,
    options.type
  );
  // 同样有报错
  fs.writeFile(outputFullFilename, outputContent, "utf8", (err) => {
    if(err) logger.error(`${outputFilename} output failed`,err);
  });
  return content;
};
```

当然这个只是 loader 最简单的形式，实际还需要考虑其他情况，比如配置项、schema 验证配置项、异步 loader 等。
有一个更完善的相同功能的 loader：https://github.com/TeamSupercell/typings-for-css-modules-loader

## hook 架构

hook 架构是 webpack 的插件体系的核心。hook 本质是一个发布-订阅模式的发布者，可以在 hook 上监听事件，然后在 webpack 的特定阶段执行这些事件。

在 webpack 中，hooks 实际上是 compiler 类中的一个对象，包含了不同阶段的不同 hook。这些 hook 可以被 plugin 获取到，在其上注册事件，而不同 hook 的执行事件的阶段不同，每个 hook 都会在某个特定阶段调用 call 方法去执行之前添加的任务。因此，**可以把这些 hook 看做是 webpack 编译过程的不同阶段**。通过在 plugin 内部调用不同的 hook，就可以访问到 webpack 执行过程的各种阶段，从而实现不同效果。

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

每个钩子传递的参数不同，大致包括：compiler、compilation、module、chunk、stats 等对象。

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

compiler 对象从开始构建到结束，会触发以下 hook：
![](https://pic.imgdb.cn/item/63e1da724757feff33d3ce00.jpg)
compilation 对象也会按一定顺序触发各种 hook：
![](https://pic.imgdb.cn/item/63e1dae24757feff33d48d30.jpg)

其他部分的各种阶段也会触发相应的 hook。因此可以说，hook 是 webpack 各个阶段的映射，控制 hook 就可以访问到执行过程的各种阶段，从而执行不同操作。

### Tapable

tapable 本质上可以看作是一个加强的发布-订阅模式。普通的发布订阅，订阅者很少会影响到发布者本身，而 tapable 形式的发布订阅，订阅者（比如各种插件）可以获取到足够的执行上下文信息，并且可以影响到自己之后的编译流程和状态。

tapable 导出了很多种 hook，这些 hook 就可以看做是一个可以注册和执行事件的对象，比如：

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

大部分的 hook 使用都遵循上面的步骤，区别主要在于 hook 类型以及注册、调用方法的不同。

### hook 类型

hook 主要有以下几种：
![](https://pic.imgdb.cn/item/63e1c9f64757feff33b8da16.jpg)

- 按回调逻辑，分为：
  - 基本类型，名称不带 Waterfall/Bail/Loop 关键字：与通常 订阅/回调 模式相似，按钩子注册顺序，逐次调用回调，比如基本的 SyncHook，就是一种同步调用回调的方式
  - waterfall 类型：前一个回调的返回值会被带入下一个回调；
  - bail 类型：逐次调用回调，若有任何一个回调返回非 undefined 值，则终止后续调用；
  - loop 类型：逐次、循环调用，直到所有回调函数都返回 undefined 。
- 按执行回调的并行方式，分为：
  - sync ：同步执行，启动后会按次序逐个执行回调，支持 call/tap 调用语句；
  - async ：异步执行，支持传入 callback 或 promise 风格的异步回调函数，支持 callAsync/tapAsync 、promise/tapPromise 两种调用语句。

不同类型的钩子会直接影响到回调函数的写法，以及插件与其他插件的互通关系。

以最简单的 Synchook 为例，简单使用如下：

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

这是最基本的 SyncHook 的使用，如果是其他类型的 hook，基本上就是在这个基础上做一些改动。不同 hook 的应用场景不同

- SyncBailHook，就是在执行回调的过程中，如果有一个回调有非 undefined 的返回值，那就终止其他回调的执行，适用于发布者需要关心订阅回调运行结果的场景。比如 shouldEmit 这个 hook，如果在 plugin 中注册的回调内返回一个 false，那么就会终止其他回调的执行，然后这个 hook 的 call 返回一个 false，可以被调用者接收到，表示出现了错误。

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

每个 hook 对应执行的 call 函数都可以换为 callAsync 函数，callAsync 在 webpack 内部应用更广一些。callAsync 需要接收一个回调函数作为参数，用于处理可能抛出的错误

```js
this.hooks.sleep.callAsync(params, (err) => {
  if (err) {
    console.log(`interrupt with "${err.message}"`);
  }
});
```

还有一些异步的 hook，比如最基本的 AsyncSeriesHook，其实就是 SyncHook 的异步版

- AsyncSeriesHook，可以用 tapAsync 注册异步回调，或者用 tapPromise 注册 promise。当异步回调执行完成，或返回的 promise resolve 之后，就会继续下一个回调。这些回调会依次顺序执行，一个执行完后才会执行下一个。
  这个 hook 其实就是**允许回调内添加异步任务**，适用于需要执行异步任务的 plugin
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

- AsyncParallelHook，就是并行执行所有的回调，而 AsyncSeriesHook 是串行执行的。

### hook 的应用

hook 最大的应用是在 plugin 内部。这个 plugin 不仅可以是用户编写的 plugin，也可以是 webpack 内部的 plugin。
我们知道 plugin 类的 apply 方法会被传入一个 compiler 对象，那么 plugin 就可以借助这个 compiler 对象上的这些 hook 注册事件。之后在 compile 方法中，在不同的阶段执行这些事件，就实现了 plugin 在不同时期执行相应任务的效果。

比如 eslint-webpack-plugin，其内部简单代码为：

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
- `compilation.hooks.succeedModule`：Webpack 完成单个「模块」的读入、运行 Loader、AST 分析、依赖分析等操作后触发；这个最为关键，每当 webpack 完整解析一个模块后，就会去调用 lint 检查这个文件。
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

plugin 可以通过 Compiler 上的各种 hook，访问到编译周期的不同阶段的上下文。也可以获取到 compilation 对象，对 make 和 seal 阶段做更详细的控制
这两者的区别再总结一下：

- compiler：实际上是编译器本身，compiler 对象贯穿编译过程始终，包含各种 hook 和大量变异相关的方法，其中 compiler.compile 方法是整个编译过程的主要方法
- compilation：是编译过程的一部分，即 make 和 seal 阶段，当初始化阶段完成后才创建，并且每次由 watch 等方式触发的重新编译都会重新创建 compilation 对象。相对来说 compilation 可控范围更细致，也更常用。

除了这两个对象之外，还有 Module、Resolver、Parser、Generator 等关键类型，也都相应暴露了许多 Hook。

还有一些常用的钩子，这些钩子及其回调才是编写 plugins 的核心。常用的 Compiler 钩子有：

- beforeRun：在 webpack 执行前调用；
- run：在 webpack 执行时调用；
- emit：在生成资源之前调用；
- afterEmit：在生成资源之后调用；
- done：在 webpack 完成构建后调用；
- compilation：在每个新的编译(compilation)创建后调用；
- make：在 webpack 开始创建新的编译(compilation)时调用；
- afterPlugins：在插件集被应用之后调用；
- afterResolvers：在解析器被应用之后调用；
- watchRun：在每次编译前进行调用；
- failed：当构建失败时调用；
- invalid：在监听模式下，当某个文件被更新时调用。

常用的 Compilation 钩子有：

- buildModule：在模块构建开始之前触发，可以用来修改模块。
- finishModules：异步钩子所有模块都完成构建并且没有错误时执行。
- seal：seal 阶段开始时
- moduleAsset 和 chunkAsset：表示一个模块/一个 chunk 的 asstet 被添加到 Compilation 时调用。

更多参考 webpack api 文档：https://webpack.docschina.org/api/

### plugin 编写示例：RemoveConsolePlugin

这个插件用于去掉 js 文件中的 console.log 语句。主要功能逻辑如下：

1. 在 compiler 的 emit hook 中注册事件，获取 compilation 对象。compiler 的大多数钩子的回调都可以获取 compilation 对象，而 compilation 对象其实是控制输出输入的核心。emit 钩子会在 webpack 生成 chunk 之后、准备输入资源之前调用，可以在这个 hook 内对将要输出的 chunk 进行修改

chunk 对象结构如下：

```json
{
  "entry": true, // 指定 webpack 运行时是否包含 chunk
  "files": [
    // 包含 chunk 的文件名字符数组
  ],
  "filteredModules": 0, // 查看关于 [top-level structure](#structure) 描述
  "id": 0, // chunk 对应的 ID
  "initial": true, // 指定 chunk 是在页面初始化时加载还是[按需加载](/guides/lazy-loading)
  "modules": [
    // [module objects](#module-objects) 列表
    "web.js?h=11593e3b3ac85436984a"
  ],
  "names": [
    // 包含当前 chunk 的 chunk 名称列表
  ],
  "origins": [
    // 查看后面的描述...
  ],
  "parents": [], // 父级 chunk ID
  "rendered": true, // 指定 chunk 是否经过代码生成
  "size": 188057 // chunk 大小，单位字节
}
```

2. compilation.chunks 可以获取已经生成的 chunk 列表，chunk.files 获取 chunk 要输出的文件信息
3. compilation.assets 获取输出的资源列表，然后 asset.source()获取输出的值。剩下的处理就和在 loader 中很像，通过正则处理字符串形式的 asset 即可。
4. 通过 compilation.emitAsset 输出资源。

```js
class MyPlugin {
  apply(compiler) {
    compiler.hooks.emit.tap("MyPlugin", (compilation) => {
      compilation.chunks.forEach((chunk) => {
        chunk.files.forEach((filename) => {
          if (filename.endsWith(".js")) {
            let asset = compilation.assets[filename];
            let source = asset.source();
            source = source.replace(/console\.log\(.+?\);/g, "");
            compilation.emitAsset(filename, source);
          }
        });
      });
    });
  }
}
```

这个插件会在构建完成后遍历生成的资源，如果发现是 JS 文件，则会去掉其中的所有 console.log() 语句。在这个例子中，我们使用了 emit 钩子。

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

## HMR

![](https://pic.imgdb.cn/item/624073d827f86abb2a67f487.jpg)

在 HMR 之前，应用的加载、更新都是一种页面级别的原子操作，即使只是单个代码文件发生变更，都需要刷新整个页面，才能将最新代码映射到浏览器上，这会丢失之前在页面执行过的所有交互与状态，例如：

对于复杂表单场景，这意味着你可能需要重新填充非常多字段信息；

- 弹框消失，你必须重新执行交互动作才会重新弹出。
- 再小的改动，例如更新字体大小，改变备注信息都会需要整个页面重新加载执行，整体开发效率偏低。而引入 HMR 后，虽然无法覆盖所有场景，但大多数小改动都可以通过模块热替换方式更新到页面上，从而确保连续、顺畅的开发调试体验，极大提升开发效率。

### 使用

hmr 的使用比较麻烦。和自动刷新不同，hmr 需要手动指定怎么把模块替换成最新的代码。

1. 设置 devServer.hot 属性为 true

```js
// webpack.config.js
module.exports = {
  // ...
  devServer: {
    // 必须设置 devServer.hot = true，启动 HMR 功能
    hot: true,
  },
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

### 实际应用

在实际项目中，以 react 项目为例，大多数时候使用的是两个 hmr 工具：react hot loader 和 react fast refresh，其中后者是当前版本更常用的方式。

react hot loader 是一种传统的实现方式，主要针对 16.8 之前的类组件。它利用的是 webpack 原生的 hmr 实现，比如 module.hot.accept 这样的 api
当文件触发更新时，它会对比新旧版本的模块，为模块创建一组“patching”（补丁），用以描述模块新旧版本的差异。这些补丁可以用于修改组件的方法、props、state 等各方面，通过补丁的形式就可以更新组件树（当前组件及其后代组件），而不需要完全重新加载整个项目。
同时它的缺陷也很明显，首先是当应用体积逐渐变大时，补丁对组件树造成的更新范围会越来越大，导致更新过程可能会越来越慢。其次相对于快速刷新更容易出现错误。最后，它可能需要更详细的配置去处理 react 特殊组件。

react fast refresh 是更新的实现方式，它采用 webpack5 提供的`HMRv2`，从 react 的调和过程出发，和 hmr 功能结合起来的实现。具体来说是调和过程的 diff 算法；它结合 diff 算法得知 react 更新了哪些部分，然后利用 hmr 更新这部分内容。这样就不需要重新加载整个组件，只是相当于调度了一次更新。
fast refresh 相对于 hot reload，更新范围更小，只需要更新当前组件及其依赖，而不会影响到其他组件的内容。因此更新速度更快、更不易出错。

> HMR v2（不是这个名字，但大概意思是一个全新的版本）是 webpack5 提供的更新的一种 hmr 方式，无需刷新页面即可更新组件。另外它还借助了 babel 的一个 plugin，可以注入代码帮助确定在 hmr 期间需要更新哪些组件并保留状态。

### 原理

热更新流程可以分为以下几步：

1. 使用 webpack-dev-server （后面简称 WDS）托管静态资源，同时以 Runtime 方式注入一段处理 HMR 逻辑的客户端代码；

在 HMR 场景下，执行 npx webpack serve 命令后，webpack-dev-server 首先会调用 HotModuleReplacementPlugin 插件向应用的主 Chunk 注入一系列 HMR Runtime，包括：

- 用于建立 WebSocket 连接，处理 hash 等消息的运行时代码；
- 用于加载热更新资源的接口；
- 用于处理模块更新策略的 module.hot.accept 接口

经过 HotModuleReplacementPlugin 处理后，构建产物中即包含了所有运行 HMR 所需的客户端运行时与接口。相当于在输出的产物中包含了建立 websocket 连接、处理加载资源等的一段代码，这部分代码将帮助完成热更新流程。
这些 HMR 运行时会在浏览器执行一套基于 WebSocket 消息的时序框架。当这部分内容运行时会和 wds 建立 websocket 连接，接收 hash 事件，获取 manifest 等操作。
![](https://pic.imgdb.cn/item/63e34e4f4757feff3308c3b3.jpg)

2. 浏览器加载页面后，与 WDS 建立 WebSocket 连接；
3. Webpack 监听到文件变化后，增量构建发生变更的模块，并通过 WebSocket 发送 hash 事件；

这一步依赖 webpack 提供的 watch 功能。当监视的文件发生变化时，webpack 会重新执行 make 阶段对发生变化的文件重新编译，并生成：

- manifest 文件：JSON 格式文件，包含所有发生变更的模块列表，命名为 `[hash].hot-update.json`；
- 模块变更文件：js 格式，包含编译后的模块代码，命名为 `[hash].hot-update.js`

增量构建完毕后，Webpack 将触发 compilation.hooks.done 钩子，并传递本次构建的统计信息对象 stats。WDS 则监听 done 钩子，在回调中通过 WebSocket 发送模块更新消息，即 hash 事件

```js
{"type":"hash","data":"${stats.hash}"}
```

4. 浏览器接收到 hash 事件后，请求 manifest 资源文件，确认本次热更新涉及的 chunk

在 webpack5 之前，热更新的单位是模块，每个模块的热更新都会生成对应的热更新文件；当前的方式是每个包含热更新文件的 chunk 在更新之后都会生成当前 chunk 的更新文件，即一个名为`main.[hash].hot-update.js`的更新文件

5. 浏览器加载发生变更的增量模块。
6. Webpack 运行时触发变更模块的 module.hot.accept 回调，执行代码变更逻辑；到这一步时浏览器已经加载完了最新模块代码，执行回调内的逻辑其实就是相当于一段额外代码，会修改原有的逻辑。

## runtime

### 模块运行时代码

webpack 支持多种模块语法，如 esm、cjs、amd、umd 等。webpack 会对模块化内容进行转译，成为保证低版浏览器也可用的导入导出模式。
如果没有 loader 或 plugin 对输出文件控制，webpack 只会修改模块化语句，而不会修改其他代码内容。

webpack 采取的模块化方案不是 cjs、esm，而是他自己定义的一种`__webpack_require__`模块化。webpack 会为输出文件注入一部分包含`__webpack_require__`及类似`__webpack_require__.r`这样的运行时代码，协助实现模块化。

运行时代码类似这样：

```js
/******/ (() => {
  // webpackBootstrap
  /******/ "use strict";
  /******/ var __webpack_modules__ = {};
  /************************************************************************/
  /******/ // The module cache
  /******/ var __webpack_module_cache__ = {};
  /******/
  /******/ // 核心的函数，webpack利用这个函数导入其他模块
  /******/ function __webpack_require__(moduleId) {
    /******/
    /******/
  }
  /******/
  /******/ // 定义各种方法
  /******/ __webpack_require__.m = __webpack_modules__;
  /******/
  /************************************************************************/
  /******/ /* webpack/runtime/chunk loaded */
  /******/ (() => {
    /******/ var deferred = [];
    /******/ __webpack_require__.O = (result, chunkIds, fn, priority) => {
      /******/
    };
  })();
  /******/
  /******/ /* webpack/runtime/compat get default export */
  /******/ (() => {
    /******/ // getDefaultExport function for compatibility with non-harmony modules
    /******/ __webpack_require__.n = (module) => {
      /******/
    };
    /******/
  })();
  /******/
  /******/ /* webpack/runtime/define property getters */
  /******/ (() => {
    /******/ // define getter functions for harmony exports
    /******/ __webpack_require__.d = (exports, definition) => {
      /******/
      /******/
    };
    /******/
  })();
  /******/
  /******/ /* webpack/runtime/hasOwnProperty shorthand */
  /******/ (() => {
    /******/ __webpack_require__.o = (obj, prop) =>
      Object.prototype.hasOwnProperty.call(obj, prop);
    /******/
  })();
  /******/
  /******/ /* webpack/runtime/make namespace object */
  /******/ (() => {
    /******/ // define __esModule on exports
    /******/ __webpack_require__.r = (exports) => {
      /******/
      /******/
    };
    /******/
  })();
  /******/
})();
```

在具体的模块内部，通过这些函数实现导入和导出的翻译处理。
webpack 默认只会编译修改模块化部分，而不会改变源代码。

函数内部从上到下依次定义：

- `__webpack_modules__` 对象，包含了除入口外的所有模块，如示例中的 a.js 模块；`__webpack_modules__.m`就表示模块对象，里边会包含实际的代码
- `__webpack_module_cache__` 对象，用于存储被引用过的模块；
- `__webpack_require__` 函数，实现模块引用(require) 逻辑；
- `__webpack_require__.d` ，工具函数，实现将模块导出的内容附加的模块对象上；
- `__webpack_require__.o` ，工具函数，判断对象属性用；
- `__webpack_require__.r` ，工具函数，在 ESM 模式下声明 ESM 模块标识；

最后的 IIFE，对应 entry 模块即上述示例的 index.js ，用于启动整个应用。
这几个 __webpack_ 开头奇奇怪怪的函数可以统称为 Webpack 运行时代码，作用如前面所说的，是搭起整个业务项目的骨架，就上述简单示例所罗列出来的几个函数、对象而言，它们协作构建起一个简单的模块化体系，从而实现 ES Module 规范所声明的模块化特性。

上述函数、对象构成了 Webpack 运行时最基本的能力 —— 模块化，假如代码中用到更多 Webpack 特性，则会相应地注入更多运行时模块代码，例如：

- 使用异步加载时，注入 `__webpack_require__.e`、`__webpack_require__.f` 等模块；
- 使用 HMR 时，注入 `__webpack_require__.hmrF`、webpack/runtime/hot 等模块。

对于不同的模块化方案，webpack统一使用上面的运行时来解析加载这些模块。常见的esm和cjs都会被处理为`__webpack_modules__`，在模块对象内保存具体的导出内容。

以一个简单的esm模块为例，包含导入导出：

源文件

```js
// ========== 源文件 ========== start
// src/module-es6.js
export let moduleValue = "moduleValue"
export default function () {
  console.log("this is es6-module")
}

// src/index.js
import moduleDefault, { moduleValue } from "./module-es6.js";
// ========== 源文件 ========== end
```

编译打包结果如下：

```js
// ========== 编译打包结果 ========== start
'./src/index.js':
(function (module, __webpack_exports__, __webpack_require__) {
  "use strict";
  __webpack_require__.r(__webpack_exports__);
  var _module_es6_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(
    "./src/module-es6.js"
  );
  console.log(_module_es6_js__WEBPACK_IMPORTED_MODULE_0__["default"]);
});

'./src/module-es6.js':
/*! exports provided: moduleValue, default */
(function (module, __webpack_exports__, __webpack_require__) {
  "use strict";
  __webpack_require__.r(__webpack_exports__);
  __webpack_require__.d(__webpack_exports__, "moduleValue", function () {
    return moduleValue;
  });
  __webpack_require__.d(__webpack_exports__, "default", function () {
    return f;
  });
  var moduleValue = "moduleValue";
  function f() {
    console.log("this is es6-module");
  }
});

// ========== 编译打包结果 ========== end
```

分析打包结果:

```js
// 先从module-es6.js 导出对象来分析(即 导出过程__webpack_require__函数的返回值 module.exports)
{
    default: f(), // 对应 export default 关键字
    moduleValue: 'moduleValue', // 对应 export 关键字
    __esModule:true  // 用于标识 es6模块
}

// index.js 中通过import引入es6模块 
// import 实际上通过__webpack_require__函数引入
var _module_es6_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__( "./src/module-es6.js")

// 对应变量的使用也转化为导出对象的属性访问
_module_es6_js__WEBPACK_IMPORTED_MODULE_0__['default']
_module_es6_js__WEBPACK_IMPORTED_MODULE_0__['moduleValue']
```

每个模块都被整理成立一个IIFE，会在调用该模块时传入`module, __webpack_exports__, __webpack_require__`这三个对象。
模块导出的内容会通过` __webpack_require__.d`方法把具体的导出对象安插在`__webpack_exports__`内部。比如

```js
__webpack_require__.d(__webpack_exports__, "default", function() { return f; }))
```

实际上就是导出一个名为default的属性到`__webpack_exports__`对象上，值就是具体导出的那个函数。

导入的内容则先通过`__webpack_require__`方法，传入一个模块路径的参数，获取到导出的`__webpack_exports__`对象。然后在其上获取具体的导出内容。参考上面的分析打包结果，其上就是index.js内部的内容。

需要注意的是，由于是esm，因此模块内容是在编译阶段解析加载的，但没有执行。当执行到index.js时，可以看到才会去执行`__webpack_require__`函数来同步获取导入内容。

---

如果是cjs，那么有一点小差别，就是会通过`__webpack_require__.n`方法来在index.js中获取被导出的exports对象

```js
// ========== 编译打包结果 ========== start
"./src/index.js":

(function (module, __webpack_exports__, __webpack_require__) {
  "use strict";
  __webpack_require__.r(__webpack_exports__);
  var _module_commonjs_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(
    "./src/module-commonjs.js"
  );
  var _module_commonjs_js__WEBPACK_IMPORTED_MODULE_0___default =
    __webpack_require__.n(_module_commonjs_js__WEBPACK_IMPORTED_MODULE_0__);
  console.log(_module_commonjs_js__WEBPACK_IMPORTED_MODULE_0___default.a);
  console.log(
    _module_commonjs_js__WEBPACK_IMPORTED_MODULE_0__["moduleValue1"],
    _module_commonjs_js__WEBPACK_IMPORTED_MODULE_0__["moduleValue2"]
  );
});
```


# 练习配置

## 使用 webpack.config.ts

config 文件的配置常常因为不清楚配置项的名称而搞错，因此可以考虑采用 ts 类型的配置文件。

步骤：

1. 首先需要安装：

```
yarn add webpack webpack-cli webpack-dev-server style-loader css-loader babel-loader html-webpack-plugin -D

yarn add typescript ts-node @types/node @types/webpack -D
```

2. 生成 tsconfig.json。

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

3. 然后编写 webpack.config.ts。

```ts
import * as webpack from "webpack";
import * as path from "path";

const config: webpack.Configuration = {
  ...
};

export default config;
```

注意部分模块需要导入类型定义，比如 devServer 就需要额外安装@types/webpack-dev-server，然后在 config 中导入 webpack-dev-server

```ts
import * as webpack from "webpack";
import * as path from "path";
import webpackDevServer from "webpack-dev-server";

const config: webpack.Configuration = {
  devServer: {
    hot: true,
  },
};

export default config;
```

## 配置 React 开发环境

### 基本 React

基本配置内容包括：React、babel 以及 typescript。

1. 安装 webpack 等一系列内容

```
yarn add webpack webpack-cli webpack-dev-server -D
yarn add css-loader babel-loader style-loader -D
yarn add react react-dom
yarn add html-webpack-plugin -D
```

如果全局没有安装 ts 的话，还需要安装 typescript

2. 执行`npx webpack init`创建基本配置文件，以及`tsc --init`创建 tsconfig.json。可以参考上面使用 webpack.config.ts 的配置，这样可以减少错误

3. 编写配置文件，主要有以下内容：

- entry：即 React 的入口文件 index.tsx。
- output：正常输出即可
- module.rules：主要配置两个 loader：
  - "style-loader"和"css-loader"
  - babel-loader
    - `@babel/preset-react`，按照如下方式配置。如果开启 runtime 配置项，项目中 jsx、tsx 文件就不需要引入 react
    - `@babel/preset-typescript`，用于解析 ts
- resolve.extensions：注意配置 tsx 和 ts，因为导入 ts/tsx 时不能添加后缀，因此必须添加 ts 和 tsx 后缀
- plugins：HtmlWebpackPlugin，设置一个 template，主要是要包含一个 id 为 root 的 div，作为 react 的 root element。

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
    hot: true,
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

4. 配置 scripts：

```json
"scripts": {
  "start": "webpack",
  "serve": "webpack serve --open"
},
```

然后执行 yarn serve 就可以启动

### 添加 css 预处理器和 css-module

以 less 为例：

1. 安装

```
yarn add -D less less-loader
```

2. 添加 less-loader。注意这里的 test 也要修改成`/\.less$/`

```js
{
  test: /\.less$/i,
  use: ["style-loader", "css-loader","less-loader"],
},
```

---

关于 css module 的配置，最简单的方式就是在 cssloader 的 options 开启`modules:true`即可，当然还有另外几个配置：

- modules:用于配置是否启用 cssmodule 及其内部配置。如果不为 false 则为开启，还可以是一个配置对象，具体配置项参考https://github.com/webpack-contrib/css-loader#auto

  这里主要使用了 localIdentName 配置，用于修改经过 css module 处理的类名，推荐这种形式：

  ```
  [local]_[hash:base64:5]
  ```

  - local：原本的样式名
  - hash:base64:5：css module 为了区分类名的 hash 值，必须要有，可以修改长度

  其他可以设置的模板字符串参考https://github.com/webpack-contrib/css-loader#localidentname

- importLoaders：表示在 css-loader 之前还有几个 loader，即 use 数组中，在 css-loader 之后的 loader 个数，默认是 0，需要修改成 1。这里只有 less-loader 一个，如果配置了 postcss，那就是两个

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

另外还有个问题，ts 中会对导入的 css module 文件报错`找不到模块 ./xxx.module.less 或其相应的类型声明`。这是因为 modulecss 文件没有类型声明。因此需要自己编写一个`.d.ts`文件为 modulecss 添加类型

```ts
declare module "*.module.less" {
  const classes: { [key: string]: string };
  export default classes;
}
```

按照 ts 对声明文件的解析规则，这里是声明了一个 module 类型的模块，`.module.css`文件会导出一个包含 classes 对象类型的变量，即导入的 style 类型`{[key:string]:string}`

@teamsupercell/typings-for-css-modules-loader 库可以自动根据 module.css 文件生成对应的声明文件，并且可以根据 css 的类名添加具体的属性名到 style 对象上去，库说明参考https://github.com/TeamSupercell/typings-for-css-modules-loader

首先安装：

```
yarn add -D @teamsupercell/typings-for-css-modules-loader
```

然后按照正常 loader 方式使用即可，注意要放在 css-loader 后面、style-loader 前面

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

对于每一个.module.css 文件，都会生成对应的.d.ts 声明文件。比如某个 less 文件为：

```less
.container {
  display: flex;
  .title {
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

可以看到，具体的类名都已经声明完成了，在 tsx 文件中使用 style 可以看到类型

### 添加 hmr

由于 webpack 的 hmr 需要开发者自行确定更新范围，这对于 react 这种复杂的框架来说很麻烦。因此可以使用已有的 hmr loader 来简化这一过程。

react-hot-loader 用于处理 react 的热更新，参考https://github.com/gaearon/react-hot-loader

1. 安装

```
npm install react-hot-loader
```

2. 在 babel 中添加 plugins：

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

3. 修改 entry 为如下形式。这一步的目的是在 react 和 react-dom 之间添加 react-hot-loader

```js
entry: ['react-hot-loader/patch', './src/index.tsx'],
```

4. （非必须）添加@hot-loader/react-dom，然后在 alias 中修改 react-dom 的别名。用这个 react-dom 替换原 react-dom 的目的在于保证部分功能实现，比如 useEffect 的热更新。
   但是这个库没有对 react18 做出更新，因此不建议修改 react-dom

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

5. 在需要热更新的组件中，用 hot 函数包裹：

```ts
import {hot} from 'react-hot-loader/root'

function App() {
  ...
}
export default hot(App)
```

只有用 hot 包裹的组件才能处理热更新，其内部的子组件也需要 hot 包裹。这种形式很麻烦，因此现在一般不采用 hmr，而是采用 fast refresh（快速刷新）

### 添加快速刷新

快速刷新是目前更常用的、代替 hmr 的方式，包括 cra 在内的多种脚手架都采用的快速刷新。
快速刷新针对的不止是 react，甚至包括普通 js 文件。
参考：https://github.com/pmmmwh/react-refresh-webpack-plugin/

1.  安装

```
yarn add -D @pmmmwh/react-refresh-webpack-plugin react-refresh
```

2. 添加 babel plugins 和 plugins

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

3. 在文件中不需要任何额外添加的部分，直接可以实现 hmr 效果。

另外，快速刷新仍然依赖于 hmr，因此还是需要开启`devServer.hot = true`.

# 优化实录

没有任何优化手段，默认打包：
![](https://pic.imgdb.cn/item/64049ac9f144a01007cb34c6.jpg)

总打包大小约为 5M
![](https://pic.imgdb.cn/item/6404a0d3f144a01007d52429.jpg)

## 打包速度优化

1. 引入缓存

引入缓存后先打包一次，第二次及以后的时间都会骤降：
![](https://pic.imgdb.cn/item/6404a151f144a01007d5eb10.jpg)

2. 限制范围-loader：

对于 css 相关的 loder 以及小一点的 loader，几乎没有影响
但是 babel-loader 必须通过 include 指定 src 内编译，不然就会非常恐怖

3. 关闭部分优化项：即关闭诸如 minimize 这种减少体积的优化。

关闭之前为 10.24 关闭之后为 9.9 略有优化

4. noParse：不解析 react，而直接采用 react.development.js

有效果，主要体现在减少 module 打包时间上，减少约 1 秒以上；但是有点极端了，对于 lodash 这样的库尚可，对于 react 还是谨慎这么做

## 产物优化

1. 开启 production 模式.

只需要把 mode 设为 production，打包体积就从 5.56MB 骤降到 1.19MB
![](https://pic.imgdb.cn/item/6404a920f144a01007e455ce.jpg)

2. 压缩代码：

js 压缩和 css 压缩的效果都不是很明显，大致降低了 5%左右。
production 模式下，本来就对 js 进行了压缩，因此再压缩的变化并不明显。但是可以看出产物进行了合并一行、去除空位等改变。

3. splitChunk

如果只开启`chunks:"all"`的话，对单入口项目没有用。
但是可以配置`splitChunks.cacheGroups`，就可以自定义分包对象。我们可以设置`node_modules`内的模块也为分包对象。这样减少了输出的 index 的大小，但是总体大小没有改变。

4. 动态导入：配合 React.lazy，可以明显看到减少了一部分体积。如果把大多数组件都动态加载，那么打包的结果就会含有比较少的源代码

5. tree-shaking：配置成功，但效果一般，猜测是项目中能被优化的部分不多，大多数导出还是被正常使用的。

# create-react-app配置文件

cra的配置文件形式还算比较清晰和简单的，这里整理一下cra的配置项：

## 具体配置内容

0. 首先cra的配置类型是函数，主要是用于区分开发和生产环境的，采取不同的loader、plugin以及一些优化措施。并且还可以在内部使用变量、filter等方式，更加灵活；

```js
module.exports = function (webpackEnv) {
  const isEnvDevelopment = webpackEnv === 'development';
  const isEnvProduction = webpackEnv === 'production';
...
```


1. entry：就是单入口，入口为index.js

```js
{
  entry: appIndexJs: resolveModule(resolveApp, 'src/index'),
}
```

2. output: 开发模式下filename是不带hash值的，并且没有进行分包，直接全部打包到一个bundle中。

cra也支持代码分割，在chunkFilename项下配置的其他chunk的输出名。不过根据cra的文档，它的代码分割也就仅限于动态导入，并没有splitChunk相关的配置。

```js
output: {
      // The build folder.
      path: paths.appBuild,
      // Add /* filename */ comments to generated require()s in the output.
      pathinfo: isEnvDevelopment,
      // There will be one main bundle, and one file per asynchronous chunk.
      // In development, it does not produce real files.
      filename: isEnvProduction
        ? 'static/js/[name].[contenthash:8].js'
        : isEnvDevelopment && 'static/js/bundle.js',
      chunkFilename: isEnvProduction
        ? 'static/js/[name].[contenthash:8].chunk.js'
        : isEnvDevelopment && 'static/js/[name].chunk.js',
      assetModuleFilename: 'static/media/[name].[hash][ext]',
      // webpack uses `publicPath` to determine where the app is being served from.
      // It requires a trailing slash, or the file assets will get an incorrect path.
      // We inferred the "public path" (such as / or /my-project) from homepage.
      publicPath: paths.publicUrlOrPath,
      // Point sourcemap entries to original disk location (format as URL on Windows)
      devtoolModuleFilenameTemplate: isEnvProduction
        ? info =>
            path
              .relative(paths.appSrc, info.absoluteResourcePath)
              .replace(/\\/g, '/')
        : isEnvDevelopment &&
          (info => path.resolve(info.absoluteResourcePath).replace(/\\/g, '/')),
    },
```

3. optimization：只开启了js和css压缩，并且只在生产环境生效。

TerserPlugin没有开启并行构建

```js
optimization: {
  minimize: isEnvProduction,
  minimizer: [
    new TerserPlugin({
      terserOptions: {
        parse: {
          ecma: 8,
        },
        compress: {
          ecma: 5,
          warnings: false,
          comparisons: false,
          inline: 2,
        },
        mangle: {
          safari10: true,
        },
        keep_classnames: isEnvProductionProfile,
        keep_fnames: isEnvProductionProfile,
        output: {
          ecma: 5,
          comments: false,
          ascii_only: true,
        },
      },
    }),
    // This is only used in production mode
    new CssMinimizerPlugin(),
  ],
},
```

4. module。cra使用了很多loader，它使用loader的原则是尽可能提供功能，因此内部包含了非常多的loader，主要包括：

- 处理静态资源的asset相关loader，比如处理图片、svg、视频等

```js
{
  test: [/\.bmp$/, /\.gif$/, /\.jpe?g$/, /\.png$/],
  type: 'asset',
  parser: {
    dataUrlCondition: {
      maxSize: imageInlineSizeLimit,
    },
  },
},
```

- babel-loader，配置了jsx-runtime和fast-refresh，开启了缓存

```js
{
test: /\.(js|mjs|jsx|ts|tsx)$/,
include: paths.appSrc,
loader: require.resolve('babel-loader'),
options: {
  customize: require.resolve(
    'babel-preset-react-app/webpack-overrides'
  ),
  presets: [
    [
      require.resolve('babel-preset-react-app'),
      {
        runtime: hasJsxRuntime ? 'automatic' : 'classic',
      },
    ],
  ],
  
  plugins: [
    isEnvDevelopment &&
      shouldUseReactRefresh &&
      require.resolve('react-refresh/babel'),
  ].filter(Boolean),
  // This is a feature of `babel-loader` for webpack (not Babel itself).
  // It enables caching results in ./node_modules/.cache/babel-loader/
  // directory for faster rebuilds.
  cacheDirectory: true,
  // See #6846 for context on why cacheCompression is disabled
  cacheCompression: false,
  compact: isEnvProduction,
},
},
```

- 针对css的loader，将css和css module分开配置。项目中有对sass的支持，但没有less

```js
{
  test: cssModuleRegex,
  use: getStyleLoaders({ // 这个函数会配置其他css-loader，比如postcss、style-loader和minicssplugin的选择等
    importLoaders: 1,
    sourceMap: isEnvProduction
      ? shouldUseSourceMap
      : isEnvDevelopment,
    modules: {
      mode: 'local',
      getLocalIdent: getCSSModuleLocalIdent,
    },
  }),
},
// 注意这里比如test的是sass文件，那就排除sassModule文件。同理sassModule也要排除sass文件
{
  test: sassRegex,
  exclude: sassModuleRegex,
  use: getStyleLoaders(
    {
      importLoaders: 3,
      sourceMap: isEnvProduction
        ? shouldUseSourceMap
        : isEnvDevelopment,
      modules: {
        mode: 'icss',
      },
    },
    'sass-loader'
  ),
  sideEffects: true,
},
```

- 其他类型的资源（没有被上面的选中的），统一被视为“文件”类型

```js
{
  // Exclude `js` files to keep "css" loader working as it injects
  // its runtime that would otherwise be processed through "file" loader.
  // Also exclude `html` and `json` extensions so they get processed
  // by webpacks internal loaders.
  exclude: [/^$/, /\.(js|mjs|jsx|ts|tsx)$/, /\.html$/, /\.json$/],
  type: 'asset/resource',
},
// ** STOP ** Are you adding a new loader?
// Make sure to add the new loader(s) before the "file" loader.
// 如果还有其他自定义的loader，就需要在这个loader之前添加。比如less-loader
```

5. plugins。cra主要使用的plugin有：

- htmlwebpackplugin，并且内部配置的压缩html
- ReactRefreshWebpackPlugin，用于快速刷新
- MiniCssExtractPlugin，压缩css
- WebpackManifestPlugin，打包manifest文件
- ESLintPlugin，配置ESLint
- 其他，主要是一些小的功能，以及针对一个问题的修复性质的plugin。


6. 其他

- devtool：开发环境用的是`cheap-module-source-map`，生产环境用的是`source-map`

```js
devtool: isEnvProduction
? shouldUseSourceMap
  ? 'source-map'
  : false
: isEnvDevelopment && 'cheap-module-source-map',
```

## 配置内容总结

cra的配置基本覆盖了cra提供的功能。不过cra在优化上配置的不是很多，比如：

- 没有开启splitChunk分包，代码分割只能依靠动态导入
- 没有开发环境的tree-shaking相关配置，比如useExports、sideEffects等
- 没有alias，基础的alias就只有一个'src'，并且没有配置空间去增加alias
- 没有thread-loader这种并行构建的优化。
