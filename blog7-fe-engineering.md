---
title: 前端工程化总结
date: 2022-04-05 16:01:22
tags: 面试
categories: 原理
cover:
---

# 基本 Node 环境

## package.json

node 官方文档：https://docs.npmjs.com/cli/v8/configuring-npm/package-json

package.json 中的关键属性：

- name：当前库或包的名称。对于 name 的使用有一些限制，比如字符长度不能超过 214，必须使用 ascii 字符且不能有大写字母等，详情参考文档。
- version：版本号，即 semver 版本。如果库是需要发布的，版本号就是最重要的属性。
- main：即库的入口文件。在 main 定义的文件内导出的内容将会作为 require 的返回值，也就是说当用户下载并通过`require('xxx')`导入库时，导入的就是入口文件中的 export。如果没有定义，默认采取根目录下的 index.js 文件。
- scripts：即可执行的命令。每个 scripts 都有自己的生命周期，
- dependencies 和 devDependencies：详见下
- engines：主要用于限制 node 和 npm 的版本。在 engines 字段中添加对 node 或 npm 版本的要求，如果用户的版本不符合，则会导致警告（npm）或报错（yarn）

## npm scripts 的生命周期

每个 scripts 都可以为其创建一个 pre 版本和 post 版本，用于在这个 scripts 执行的前后分别执行。只要命名符合就可以执行。

```json
{
  "scripts": {
    "precompress": "{{ executes BEFORE the `compress` script }}",
    "compress": "{{ run command to compress files }}",
    "postcompress": "{{ executes AFTER `compress` script }}"
  }
}
```

实际上每个内置的脚本都包含自己的一些 pre、post 等脚本。比如执行 npm install，实际上是按照这个顺序执行的：

- preinstall
- install
- postinstall

执行 npm publish，是按照这个顺序执行的

- prepublishOnly: 发布时最重要的一个生命周期，在包的准备和打包之前运行，可以做一些整理、测试、构建等工作
- prepack
- prepare
- postpack
- publish
- postpublish

至于其他的可以参考[官方文档](https://docs.npmjs.com/cli/v8/using-npm/scripts#npm-cache-add)

### dependencies 和 devDependencies

`devDependencies` 里面的插件只用于开发环境，不用于生产环境，
`dependencies` 是需要发布到生产环境的。

### semver

即安装库时后面的版本号，版本号有三个数，分别为`[major, minor, patch]`：

- major: 主版本号，该值不同的版本之间 api 不能兼容，比如 vue2 和 vue3
- minor: 新增功能并且能保证向后兼容，相当于一次小更新
- patch: 修复了一个向后兼容的 Bug 时，几乎不改变

除了这三个数，有时候版本号前面还会带一个`^`或者`~`，这两个字符表示：

- `^`：minor 和 patch 可以不同，即对于 `^1.2.3` 而言，它的版本号范围是 `[1.2.3, 2.0.0)`（区间左闭右开）
- `~`：patch 可以不同，对于 ~1.2.3 而言，它的版本号范围是 `[1.2.3, 1.3.0)`

当我们 npm i 时，默认的版本号是 ^，可最大限度地在向后兼容与新特性之间做取舍，但是这导致库的版本并不是一个确切的值而是一个范围，所以项目中使用的确切版本号实际上是在 yarn.lock/package-lock.json 中锁定的版本号。

- 如果库的版本更新，但是在 lock 文件中依旧锁定，并且更新的范围在`^`或`~`的范围内，那就不会更新库的实际版本。
- 如果库的版本更新导致范围超出`^`或`~`的范围，即如果手动修改了 package.json 中的版本号，与 package-lock.json 中不是一致的版本范围。此时 npm i 将下载最新版本号，并重写 package-lock.json 中锁定的版本号。

## package-lock.json

官方文档：https://docs.npmjs.com/cli/v8/configuring-npm/package-lock-json

> package-lock.json 会为 npm 修改 node_modules 树或 package.json 的任何操作自动生成。它描述了生成的确切树，以便后续安装能够生成相同的树，而不管中间依赖项更新如何。
> 该文件旨在提交到源存储库中，并用于各种目的：
>
> - 描述依赖关系树的单一表示，以保证团队成员、部署和持续集成安装完全相同的依赖关系。即 lock 文件中会展示每个包的依赖、依赖的依赖、依赖的依赖的依赖，直到展开到每一个包的确切版本以及 npm 链接等数据。相当于一棵完整的依赖树
> - 为用户提供“时间旅行”到 node_modules 先前状态的工具，而无需提交目录本身。
> - 通过可读的源代码控制差异促进树变化的更大可见性。
> - 通过允许 npm 跳过以前安装的包的重复元数据解析来优化安装过程。
> - 从 npm v7 开始，lockfiles 包含足够的信息来获得库的依赖树的完整结构，减少了读取 package.json 文件的需要，并显著提高性能。

packagelock.json/yarn.lock 用以锁定版本号，保证开发环境与生产环境的一致性，避免出现不兼容 API 导致生产环境报错
当有了 lock 文件时，每一个依赖的版本号都被锁死在了 lock 文件，每次依赖安装的版本号都从 lock 文件中进行获取，避免了不可测的依赖风险。

> 一个问题: 当项目中没有 lock 文件时，生产环境的风险是如何产生的?
> 演示风险过程如下:
> pkg 1.2.3: 首次在开发环境安装 pkg 库，为此时最新版本 1.2.3，dependencies 依赖中显示 ^1.2.3，实际安装版本为 1.2.3
> pkg 1.19.0: 在生产环境中上线项目，安装 pkg 库，此时最新版本为 1.19.0，满足 dependencies 中依赖 ^1.2.3 范围，实际安装版本为 1.19.0，但是 pkg 未遵从 semver 规范，在此过程中引入了 Breaking Change，如何此时 1.19.0 有问题的话，那生产环境中的 1.19.0 将会导致 bug，且难以调试
> 如果使用 lock 文件锁住版本号，就会确保版本是可用的，减少可能出现的问题。

## npm install 发生了什么

![](https://pic.imgdb.cn/item/6277cfcf0947543129564ce3.jpg)

1. npm install 执行后，会**检查并获取 npm 配置**

> 优先级为:
> 项目级别的.npmrc 文件 > 用户级别的.npmrc 文件 > 全局的.npmrc 文件 > npm 内置的.npmrc 文件

.npmrc 文件就是 npm 的配置文件。查看 npm 的所有配置, 包括默认配置，可以通过下面的命令:

```
npm config ls -l
```

2. 检查当前目录下的 node_modules 是否已经包含了该包，如果有就不再下载。

3. **检查项目中是否有 package-lock.json 文件**，根据 package-lock.json 的依赖关系递归构建依赖树，准备下一步的下载。
   从 npm 5.x 开始，执行 npm install 时会自动生成一个 package-lock.json 文件。
   package-lock.json 文件精确描述了 node_modules 目录下所有的包的树状依赖结构，每个包的版本号都是完全精确的。
   因此 npm 会先检查项目中是否有 package-lock.json 文件，分为两种情况：

- 如果有，检查 package-lock.json 和 package.json 中声明的依赖是否一致
  - 一致：直接使用 package-lock.json 中声明的依赖，从缓存或者网络中加载依赖
  - 不一致：各个版本的 npm 处理方式如上图
- 如果没有，根据 package.json 递归构建依赖树，然后根据依赖树下载完整的依赖资源，在下载时会检查是否有相关的资源缓存
  - 存在：将缓存资源解压到 node_modules 中
  - 不存在：从远程仓库下载资源包，并校验完整性，并添加到缓存，同时解压到 node_modules 中

4. npm 向 registry 查询模块压缩包的网址、最新版本等信息并**下载资源包**，存放在缓存目录中；**解压资源包到当前项目的 node_modules 目录**；并生成 package-lock.json 文件。

5. 因为不同包的依赖很可能有重复的，为了避免重复安装一个依赖，会在解压时跳过已经拥有的包。构建依赖树时，不管是直接依赖还是子依赖，都会按照扁平化的原则，优先将其放置在 node_modules 根目录中(最新的 npm 规范), 在这个过程中，如果遇到相同的模块，会检查已放置在依赖树中的模块是否符合新模块的版本范围，如果符合，则跳过，不符合，则在当前模块的 node_modules 下放置新模块。

---

另外，在执行 npm install 或 npm update 命令下载依赖后，除了将依赖包安装在 node_modules 目录下外，还会在本地的缓存目录缓存一份。

再次安装依赖的时候，会根据 package-lock.json 中存储的信息生成一个唯一的 key，然后拿着 key 去目录中查找对应的缓存记录，如果有缓存资源，就把对应的二进制文件解压到相应的项目 node_modules 下面，省去了网络下载资源的开销。

## npm run 发生了什么

> 1. 运行 npm run xxx 的时候，npm 会先在当前目录的 node_modules/.bin 查找要执行的程序，如果找到则运行；
> 2. 没有找到则从全局的 node_modules/.bin 中查找，npm i -g xxx 就是安装到到全局目录；
> 3. 如果全局目录还是没找到，那么就从 path 环境变量中查找有没有其他同名的可执行程序。

详见：https://juejin.cn/post/7078924628525056007

npm run 可以运行定义在 package.json 中的脚本配置项。通常这些命令的启动项都是不存在在系统环境变量的，直接调用会报错，而 npm run 则会通过一系列机制保证其运行。
比如：

```json
"scripts": {
  "build": "webpack",
  "start": "webpack serve --open"
}
```

webpack 直接在命令行中启动会报错，因为它并非一个可执行文件，并且操作系统中没有存在 webpack 这一条指令。

npm run 能正常运行的原因如下：

1. 安装依赖的时候，是通过 npm i xxx 来执行的，例如 `npm i webpack`，npm 在 安装这个依赖的时候，就会在`node_modules/.bin/` 目录中创建好这几个几个可执行文件。

![](https://pic.imgdb.cn/item/6277d2da094754312964b429.jpg)

> 这里名为webpack的软链接有三个，即webpack/webpack.cmd/webpack.ps1，这三个不同版本是在不同操作系统上使用的不同脚本，windows一般执行.cmd，unix一般执行webpack，而ps指的是shell脚本，是一个通用的

这个目录不是任何一个 npm 包。目录下的文件，表示这是一个个软链接（这些文件就叫做软链接），打开文件可以看到文件顶部写着 #!/bin/sh ，表示这是一个 shell 脚本。
比如打开 webpack，里边是这样的：

```shell
#!/bin/sh
basedir=$(dirname "$(echo "$0" | sed -e 's,\\,/,g')")

case `uname` in
    *CYGWIN*|*MINGW*|*MSYS*) basedir=`cygpath -w "$basedir"`;;
esac

if [ -x "$basedir/node" ]; then
  exec "$basedir/node"  "$basedir/../webpack/bin/webpack.js" "$@"
else
  exec node  "$basedir/../webpack/bin/webpack.js" "$@"
fi
```

这些被称为软链接。当通过npm run执行`webpack`命令时，就相当于在`.bin`目录下找到名为webpack的软链接来执行。

在 package-lock.json 中，可以查找到一个 bin 的配置项，这一项就是 npm 对于软链接的目标执行文件的配置，这一项的值就是要链接到的具体的文件，就像这样：

这个配置是用于指明webpack这软链接指向的具体要执行的文件的目录（即/node_modules/webpack/bin/webpack.js）。这样执行软链接时就会找到对应的文件来执行。

```json
"bin":{
  "webpack":"bin/webpack.js"
}
```

2. 在 npm install 时，会将要运行的文件（即上面配置的 bin 对应的文件）软链接到 ./node_modules/.bin 目录下，而 npm 还会自动把 node_modules/.bin 加入$PATH，这样就可以直接作为命令运行依赖程序和开发依赖程序，不用全局安装了。这种软连接相当于一种映射，执行 npm run xxx 的时候，就会到 node_modules/bin 中找对应的映射文件，然后再找到相应的 js 文件来执行。

3. 当执行`npm run start`命令时，npm 会到 ./node_modules/.bin 中找到`webpack`文件执行，相当于执行了`./node_modules/.bin/webpack start --open`。在系统终端中，相当于执行了`./node_modules/.bin/webpack start --open`；然后根据原先脚本的设置，将会用 node 去执行`node ../webpack/bin/webpack.js`，最后完成执行过程。

> npx
> npx 想要解决的主要问题，就是调用项目内部安装的模块，即前面说的不能直接运行诸如`webpack`这样的命令。可以通过`npx webpack`直接运行
> npx 还可以避免全局安装包，比如`npx create-react-app myapp`这样的命令可以在没有安装 createreactapp 时直接使用。

# Babel

> Babel 是一个工具链，主要用于将 ECMAScript 2015+ 代码转换为当前和旧浏览器或环境中向后兼容的 JavaScript 版本。babel 能做的事情有:
> 语法转换
> 通过 Polyfill 方式在目标环境中添加缺失的特性 （通过引入第三方 polyfill 模块，例如 core-js）
> 源码转换（codemods）,比如转换 React 的 jsx

## babel 的基本使用

https://www.jiangruitao.com/babel/quick-start/

### 安装

babel 基本使用需要安装至少三个包，分别是 babel 的命令行工具、最常用的 preset，以及 babel 的核心库(core)

```
npm install -D @babel/cli @babel/preset-env @babel/core
```

注意安装需要-D，因为 babel 只在开发环境下编译，编译完成后不再需要使用

### 配置

babel 的所有配置，都离不开两个配置项：presets 和 plugins

- `presets`指预设，是一组插件的集合，是一个数组；Babel 官方的 preset，我们实际可能会用到的其实就只有 4 个：
  - `@babel/preset-env`，最常用的，日常的转换只需要这个就可以；
  - `@babel/preset-flow`
  - `@babel/preset-react`，用于转换 jsx
  - `@babel/preset-typescript`，用于转换 ts
- `plugins`指插件，插件的类型类似于 webpack 中的插件。babel 将文件翻译成 AST，经过插件的处理之后再转义成代码，因此插件才是 Babel 的核心。常用的插件现在只有一个：
  - `@babel/plugin-transform-runtime`，会将常用的辅助函数（即用来代替或提供新 api 的函数）保存，不用每次编译时都生成，减少代码

另外，插件和预设的运行规则如下：

- 插件比预设先执行
- 插件执行顺序是插件数组从前向后执行
- 预设执行顺序是预设数组从后向前执行

#### 配置文件

> 配置文件的具体区别，参考https://github.com/willson-wang/Blog/issues/100

babel 的配置文件有多种类型，主要有以下三种：

- `babel.config.json`：官网推荐的类型，json 格式。适用于全局（包括 node_modules）都可能需要翻译的情况，最基本的配置如下：

```json
{
  "presets": [...],
  "plugins": [...]
}
```

其中，presets 中每一项可以是一个字符串，表示一个预设；如果这个预设要传入参数，就展开成数组：

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "entry"
      }
    ]
  ]
}
```

插件也是同理。

- `.babelrc`，脚手架中最常用的配置文件，实际上是`.babelrc.json`；适用于只在部分代码中需要转义的情况，比如只需要转义 src 下的代码而不需要改变 node_modules 下的，就可以使用它。
  配置项和上面的没有任何区别

- `babel.config.js`和`.babelrc.js`，即 js 文件的配置，需要通过 module.exports 导出：

```js
 //  这里只是举个例子，实际项目中，我们可以传入环境变量等来做处理
  var year = 2020;
  var presets = [];
  if (year > 2018) {
    presets = ["@babel/env"];
  } else {
    presets = "presets": ["es2015", "es2016", "es2017"],
  }
  module.exports = {
    "presets": presets,
    "plugins": []
  }
```

- 直接在`packages.json`中配置：

```json
{
  "name": "demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "babel": {
    "presets": ["es2015", "react"],
    "plugins": ["transform-decorators-legacy", "transform-class-properties"]
  }
}
```

- 配合 webpack 配置，同样是安装需要的包，以及`babel-loader`。详见https://webpack.docschina.org/loaders/babel-loader/#install

```js
// npm install -D babel-loader @babel/core @babel/preset-env webpack

module: {
  rules: [
    {
      test: /\.m?js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: "babel-loader",
        options: {
          presets: ["@babel/preset-env"],
          plugins: ["@babel/plugin-proposal-object-rest-spread"],
        },
      },
    },
  ];
}
```

#### polyfill

polyfill 广义上讲是为环境提供不支持的特性的一类文件或库，即提供旧版本中没有的诸如 Promise 和 WeakMap 之类的新的内置组件、 Array.from 或 Object.assign 之类的静态方法、 Array.prototype.includes 之类的实例方法以及生成器函数等。

babel 官方有一个`@babel/polyfill`可以使用。
安装：

```
npm install @babel/polyfill
```

`preset-env` 提供了一个 "useBuiltIns" 参数，当此参数设置为 "usage" 时，就会加载上面所提到的最后一个优化措施，也就是只包含你所需要的 polyfill。

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "edge": "17",
          "firefox": "60",
          "chrome": "67",
          "safari": "11.1"
        },
        "useBuiltIns": "usage"
      }
    ]
  ]
}
```

当 polyfill 成功使用时，比如较低版本中缺少的 api 就会被通过`require`一个 url 的形式引入。

#### `@babel/preset-env`

参考https://www.jiangruitao.com/babel/babel-preset-env/

#### `@babel/plugin-transform-runtime`

参考https://www.jiangruitao.com/babel/transform-runtime/

### 编译

babel 的启动通常是通过命令行：

```
./node_modules/.bin/babel src --out-dir lib
```

表示将 src 下的所有文件都编译并输出到 lib 目录。
当然也可以通过 npx 或者 npm scripts：

```
"scripts": {
    "build": "babel src --out-dir lib"
},

或者

npx babel src --out-dir lib
```

其他基本命令：(详见https://www.babeljs.cn/docs/babel-cli)

- `babel index.js`：直接编译指定文件，**会输出到终端**
- `babel script.js --out-file script-compiled.js`：指定输出文件
- `babel script.js --watch --out-file script-compiled.js`：类似 webpack 的文件监听，源文件改变时会更新编译

# Vite

> Vite 旨在利用生态系统中的新进展解决上述问题：浏览器开始原生支持 ES 模块，且越来越多 JavaScript 工具使用编译型语言编写。

## 依赖和源码

Vite 通过在一开始将应用中的模块区分为**依赖**和**源码**两类，改进了开发服务器启动时间。

- `依赖`：大多为在开发时不会变动的纯 JavaScript。一些较大的依赖（例如有上百个模块的组件库）处理的代价也很高。依赖也通常会存在多种模块化格式（例如 ESM 或者 CommonJS）。
  Vite 将会使用 `esbuild` 预构建依赖。Esbuild 使用 Go 编写，并且比以 JavaScript 编写的打包器预构建依赖快 10-100 倍。
  不仅是依赖的预构建，传统的使用 js 编译的很多步骤都被用`esbuild`代替，比如`tsc`、`jsx`、`sass`等等。

- `源码`：即用户代码，需要转换（例如 JSX，CSS 或者 Vue 组件），时常会被编辑。同时，并不是所有的源码都需要同时被加载，有可能是按照需要进行加载。

Vite 使用 es6 Module 加载源码；因为 ESModule 是静态的、并且大多数时候是浏览器主要进行解析工作，再加上根据情景动态导入代码，要比先将代码打包成 bundle 再发送给浏览器的形式快很多。

## 热更新

从热更新上，Vite 同时利用 HTTP 头来加速整个页面的重新加载，即利用到了协商缓存（源码）和强缓存（依赖），一旦被缓存将不需要再次请求。
并且 vite 利用 ES Module 来帮助进行热更新，又是大大提升了构建速度。

## 主要功能

### 插件

类似 webpack 的插件，vite 也可以显式配置插件。

插件需要被单独安装，添加到项目的 `devDependencies`中，然后像 webpack 一样在 plugins 数组中配置：

```js
import legacy from "@vitejs/plugin-legacy";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [
    legacy({
      targets: ["defaults", "not IE 11"],
    }),
  ],
});
```

社区可用的插件可以见 https://github.com/vitejs/awesome-vite#plugins
官方提供的插件只有基本的几个：https://cn.vitejs.dev/plugins/
或者在这里搜素：https://www.npmjs.com/search?q=vite-plugin&ranking=popularity

## 依赖预构建

通常在模块被安装后的首次运行，vite 会预构建依赖。主要目的如下：

1. 将作为 CommonJS 或 UMD 发布的依赖项转换为 ESModule。
2. 将有许多内部模块的 ESModule 依赖关系转换（合并）为单个模块，以提高后续页面加载性能。

在服务器已经启动之后，如果遇到一个新的依赖关系导入，而这个依赖关系还没有在缓存中，Vite 将重新运行依赖构建进程并重新加载页面。

# ESlint

ESLint 是在 ECMAScript/JavaScript 代码中识别和报告模式匹配的工具，它的目标是保证代码的一致性和避免错误。
ESLint 使用 Espree 解析 JavaScript。
ESLint 使用 AST 去分析代码中的模式
ESLint 是完全插件化的。每一个规则都是一个插件并且你可以在运行时添加更多的规则。

## 配置

ESLint 是完全可配置的，也就说可以同时配置一些预设、插件和具体的某一项规则。
配置文件可以是：

- .eslintrc.\* 文件，最常用的配置方式，后缀可以是 js、ts 或 json
- package.json 中的 eslintConfig 字段，这个是 create-react-app 的默认配置方式

基本配置：

```json
{
  "env": {
    // 指定环境
    "browser": true,
    "node": true
  },
  "parserOptions": {
    "ecmaVersion": 6, // es版本，推荐"latest"
    "sourceType": "module", // 是否使用esmodule
    "ecmaFeatures": {
      // 启用jsx，但是还需要特殊的react插件
      "jsx": true
    }
  },
  "extends": ["eslint:recommended", "plugin:react/recommended"], // 继承推荐的配置项
  "plugins": ["react"],
  "rules": {
    // 具体规则配置项
    "semi": "error"
  }
}
```

其中，插件、继承和规则三个是最主要的配置
具体配置项：https://cn.eslint.org/docs/rules/

# CSS Modules

CSS Modules 是模块化 css 的一种解决方案。考虑到在 Vue 中有诸如 scoped 这样的局部 css 实现，React 中如果想实现这样的效果就需要借助 CSS Modules。

React 的样式文件是全局的，即虽然可能只在一个组件文件中引入了样式，但这个样式的类名将会是全局的。这样就很容易造成干扰，如果类名相同，就会导致冲突和覆盖。因此可以借助 CSS Modules，使得项目中可以像加载 js 模块一样加载 css ，本质上通过一定自定义的命名规则生成唯一性的 css 类名，从根本上解决 css 全局污染，样式覆盖的问题。

## 基本使用

CSS Modules 直接使用没有什么意义，这里直接说和 react 的配合使用。

create-react-app 自带 CSS Modules，并且它的默认规则是识别后缀为`.modules.css`的文件为 modules css，而正常的`.css`文件会当作全局样式。因此可以直接在组件中引入：

```js
import style from "./App.module.css";

function App() {
  return (
    <>
      <div className={style.main}>
        <h1 className={style.title}>hello world</h1>
      </div>
    </>
  );
}
```

引入的 style 对象就是解析后的 modules css。注意这个对象不能解构，类名必须通过`style.xxx`的形式得到；

类名默认会被编译成一个哈希值，也可以自己配置编译的类名结果，添加一些文件名、样式名之类的，以便于调试的时候查找。
添加方法需要先`npm run eject`暴露配置，然后在 webpack.config.js 中找到下面这一段：
![](https://pic.imgdb.cn/item/62df986bf54cd3f9373986db.jpg)

注释这一行是没有的，直接按照注释的这一行的写法添加即可，然后把下面的 getLocalIdent 去掉。其中几个项的含义如下：

- path：当前文件相对于根目录的路径，一般从 src 开始
- name：当前文件名
- local：当前样式对应的类名。
- hash：随机哈希值，这个和 webpack 配置出口文件名的规则相同，通常`[hash:5]`就可以

因此（个人觉得）比较方便的写法是这样：

```
"[path]_[name]-[local]_[hash:5]"
```

---

一旦经过 css modules 处理的 css 文件类名 ，再引用的时候就已经无效了。因为声明的类名，已经被处理成了哈希形式。
因此可以像 create-react-app 一样，当需要 CSS Modules 时将后缀变为`.modules.css`即可，全局样式正常写在`.css`中

## 和 sass 配合

同样是 create-react-app。首先安装好 sass（注意 create-react-app 不自带 sass，要手动安装）

```
npm i sass
```

然后同样找到 webpack.config.js 中，就跟在上面说的那部分的下面一点，即解析 sass 的地方：
![](https://pic.imgdb.cn/item/62df9ad6f54cd3f93747467c.jpg)

方法相同。

使用：

```js
import style from "./App.module.scss";

function App() {
  return (
    <>
      <div className={style.header}>
        <h1 className={style.title}>no hello</h1>
      </div>
      <div className={style.main}>
        <h1 className={style.title}>hello world</h1>
      </div>
    </>
  );
}
```

这里 sass 的大部分特性依旧还能正常用，比如变量、函数这一类。

关于嵌套问题，之前看到的教程说不能用嵌套的形式写类名了，但是我这里尝试还是可以的。
上面的代码，如果样式这样写：

```scss
.header {
  background-color: blue;
  .title {
    color: red;
  }
}

.main {
  background-color: red;
  .title {
    font-weight: 900;
    color: green;
  }
}
```

title 会被编译成相同的结果：

![](https://pic.imgdb.cn/item/62df9b60f54cd3f9374a531b.jpg)

但是由于父元素的类名不同，因此仍然是可以正常显示的

其他可以参考https://segmentfault.com/a/1190000039846173

## 在 Vite 中使用

Vite 也自带了 css Modules 的配置。同样可以通过`.module.css`的后缀使用，基本使用和上面一样的。

唯一的区别是配置输出类名，需要这样配置：

![](https://pic.imgdb.cn/item/62df9ef0f54cd3f9375e218c.jpg)

具体参考https://cn.vitejs.dev/config/shared-options.html#css-modules

并且 scss 等也是可以正常使用的，和上面一样

## 与 classnames 库配合

react 原生动态添加多个 className 会报错

```js
import style from './style.css'

<div className={style.class1 style.class2}></div>
```

这种情况如果不使用字符串的话，就需要借助 classnames 库来实现。
文档：https://github.com/JedWatson/classnames#readme

---

注意，类名在 css 文件中如果是连字符形式的（比如`light-theme`这样），就不能直接通过`style.light-theme`这样的形式使用，需要再配置一项：
![](https://pic.imgdb.cn/item/62dfa303f54cd3f93774bc75.jpg)
camelCase 可以使得在 js 文件中用小驼峰代替连字符形式的类名，上面的就可以写成`style.lightTheme`

# git

Git 是目前世界上最先进的分布式版本控制系统。分布式体现在每个用户个人电脑上都是一个完整的版本库，而不需要像集中式那样集中将版本提交到一个网络平台上。如果想要交互版本信息，也只需要把双方的修改推送给对方就可以获取。分布式版本控制系统的安全性要高很多。

## 基本概念

### 分区

git 中有三个主要的分区，分别是工作区、暂存区和版本库。严格来说，暂存区也是版本库的一部分

![](https://pic1.imgdb.cn/item/6356176216f2c2beb15ea74b.jpg)

工作区就是当前目录，也就是初始化 git 库所在的目录。
版本库就是初始化后创建的.git 文件夹，里边包含了暂存区和各个分支。

- 当使用 add 添加文件时，实际上就是把文件添加到了暂存区。暂存区内存储的是文件的修改，通过 git status 可以查看哪些文件在暂存区中。比如下面这段，readme.txt 是修改了，而 LICENSE 是新增加的，这两个都未被添加到暂存区（Changes not staged for commit 即修改未被暂存）

```
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   readme.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	LICENSE

no changes added to commit (use "git add" and/or "git commit -a")
```

如果给两者都用 add 添加到暂存区，则会这样：

```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   LICENSE
	modified:   readme.txt
```

即显示 Changes to be committed，暂存区将被提交。

- 当使用 commit 提交文件时，就是把文件提交到了当前分支上，同时暂存区的内容会消失，并且这时会视作工作区是“干净的”（即 working tree clean）

### 分支和 HEAD 指针

commit 提交之后，每次 commit 都会按照时间顺序相互连接，形成一个链表。当我们创建分支时，就会形成一棵提交树。
commit tree 可能是部分重合的。每个分支都有一个指针指向当前分支所在的最后一个提交，这个指针移动就相当于版本回退和前进。

为了方便查看树中的所有节点，还有一个自由指针 HEAD。HEAD 指针指向的位置表示“当前值”，即它所在的分支就是当前分支
这个指针默认和 master 指针同步，即 Git 用 master 指向最新的提交，再用 HEAD 指向 master：

![](https://pic1.imgdb.cn/item/63561cdc16f2c2beb166adb2.jpg)

当通过 git log 查看提交时，也可以看到最新的一次提交旁边有一个`(HEAD->master)`，表示 HEAD 正指向当前提交
![](https://pic1.imgdb.cn/item/63561e8716f2c2beb1691f32.jpg)
如果用 checkout -b 创建并切换到一个新的分支，则 HEAD 会指向新分支的指针：

![](https://pic1.imgdb.cn/item/63561d3216f2c2beb1672ac6.jpg)

实际上用 checkout 命令+任意提交的 id，就可以将 HEAD 指针从当前分离出来并指向目标提交。但此时 HEAD 指针相当于一个无主的指针，可以在其上提交，但他并不指向一个具体的分支，可能会导致不能合并。如果想要保持 HEAD 指向具体的分支，只需要`git checkout <分支名>`即可切换到指定分支。

另外为了和撤销修改区分，分支切换更推荐使用 switch：

```
创建并切换到新的dev分支，可以使用：
$ git switch -c dev
直接切换到已有的master分支，可以使用：
$ git switch master
```

---

在版本回退中，由于回退的依据常常是具体的某次提交，而输入 id 又非常麻烦。因此 HEAD 有一些可以访问其相对位置的语法。
HEAD 表示当前版本，`HEAD^`表示上一个版本，`HEAD~n`表示上 n 个版本
对分支指针也可也使用这种语法。比如`git checkout master^`表示切换到 master 分支的上一个节点

## 版本回退

版本回退的核心是 commit。每个 commit 都可以看作是一个存档，工作区可以从任意一个保存的 commit 中恢复。

查看 commit 的方法是 git log。注意不是 git status，status 是用于查看暂存区和工作区状态的。

git log 命令显示从最近到最远的提交日志，包括提交的版本号（commit id）、提交人、提交时间、以及提交时的注释（-m 后的内容）

```
commit eb34058d880bf18f701355a5601b26a8c5fa0860 (HEAD -> master, test)
Author: penghei <632885485@qq.com>
Date:   Tue Aug 2 19:58:30 2022 +0800

    third

commit e77b1fc694de190185d53da1c573e40da36a3cfb
Author: penghei <632885485@qq.com>
Date:   Tue Aug 2 19:41:03 2022 +0800

    second

commit fc87038c63eceaa2f5bc8fdc2f5f1ed579d03bb1
Author: penghei <632885485@qq.com>
Date:   Tue Aug 2 19:34:43 2022 +0800

    first
```

如果想要版本回退，就应该使用 git reset 命令。

```
$ git reset --hard HEAD^
HEAD is now at e77b1fc second
```

git reset 会导致回溯之后的最新的 commit 消失。比如这时再查看 git log，会发现 third 提交消失了

```
commit e77b1fc694de190185d53da1c573e40da36a3cfb (HEAD -> master, test)
Author: penghei <632885485@qq.com>
Date:   Tue Aug 2 19:41:03 2022 +0800

    second

commit fc87038c63eceaa2f5bc8fdc2f5f1ed579d03bb1
Author: penghei <632885485@qq.com>
Date:   Tue Aug 2 19:34:43 2022 +0800

    first
```

此时相当于 HEAD 指针从 third 提交指向了 second 提交，工作区的内容也改回了 second 提交时的状态。但是实际上 third 的 commit 并没有被删除；如果能找回其 commit id，也是可以返回的：

```
$ git reset --hard eb3405
```

也就说，git reset 可以将工作区内容指向任意一个之前的 commit，并且这个过程是通过修改 HEAD 指针指向实现的。

## 管理修改

Git 跟踪并管理的是修改，而非文件。如果修改没有被 add 添加到暂存区，也就不会被 commit 记录

`git diff HEAD -- readme.txt`命令可以查看工作区和版本库里面最新版本的区别。在 vscode 中也会显示某些行的变化，就是工作区和最新一次提交的区别。

注意 git 的回退单位是 commit；如果一次修改没有被提交，就不能恢复到这次修改。除非采用抛弃修改的方式，可以回到暂存区的状态，详见下：

---

`git checkout -- file`可以丢弃工作区的修改

```
$ git checkout -- readme.txt
```

这个命令可能有两种情况：

- readme.txt 自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态，即回到最新的 commit 状态
- readme.txt 已经添加到暂存区后，又作了修改，现在，撤销修改就回到**添加到暂存区后**的状态。

也就说如果添加到暂存区后再修改，撤销修改就只能回退到暂存区的状态，除非使用 reset 回退 commit。
但是还可以通过`git reset HEAD <file>`可以把暂存区的修改撤销掉

```
$ git reset HEAD a.txt
Unstaged changes after reset:
M       a.txt
```

然后再使用`git checkout -- file`就可以恢复到最初的状态，即使已经提交到工作区。

---

除了修改，文件的删除也可以被`git checkout -- file`恢复，但只能根据 commit 恢复，不可以通过工作区恢复。
只需要把删除看作是一种修改就可以

```
$ git add test.txt

$ git commit -m "add test.txt"

$ rm test.txt

# 这个命令可以恢复删除的文件，内容是最新的commit
$ git checkout -- test.txt
```

## 分支

### 创建和合并分支

创建和切换分支的简单操作在上面 HEAD 指针讲过，基本命令就是这几个：

- `git checkout -b <分支名>`或`git switch -c <分支名>`可以创建并切换到新分支。这两个命令都相当于用`git branch <分支名>`创建新分支，然后再切换
- `git checkout <分支名>`或`git switch <分支名>`可以切换到指定分支，即把 HEAD 指针指向指定分支的头指针。切换之后，提交操作就会在该分支上

接下来就是合并分支。在一个分支上使用`git merge <分支名>`，可以把 merge 后面的分支（称作副分支）合并到当前分支（主分支）
合并有两种情况：

- **无冲突**，最常见的情况是副分支在主分支的基础上修改（完成几次提交），而主分支没有修改过。这时可以把副分支合并到主分支上，工作区内的状态就会变成副分支的最新修改，同时 HEAD 指针指向主分支。

![](https://pic1.imgdb.cn/item/63562cd616f2c2beb17f7f35.jpg)
![](https://pic1.imgdb.cn/item/63562cca16f2c2beb17f7323.jpg)

这种合并被称为`Fast-forword`，相当于直接修改 master 指针的指向，让其指向 dev 即可。
合并完成之后还可以删除 dev 分支

```
$ git branch -d dev
```

- **有冲突**，常见的情况是主分支在分出副分支后，两个分支都做了不同的修改并提交

![](https://pic1.imgdb.cn/item/63562d6716f2c2beb1802c4b.jpg)
这时就不能进行自动合并。如果试图将 bug 合并到 main，就会提示有冲突，并且还会在冲突文件中用`<<<<<<<`，`=======`，`>>>>>>>`标记出不同分支的内容，像这样：

```
...某些内容

<<<<<< HEAD
Creating a new branch is quick & simple.
=======
Creating a new branch is quick AND simple.
>>>>>> bug
```

你需要自己选择接收某个分支的内容，或者自己修改新的内容。不管怎么样，如果删掉`<<<<<<<`，`=======`，`>>>>>>>`，git 就视为你解决了冲突，接下来就可以正常合并了。
但是并不需要 merge 命令，**解决冲突后再在主分支添加并提交一次就可以了**；用带参数的 git log 也可以看到分支的合并情况：

```
$ git add readme.txt
$ git commit -m "conflict fixed"

$ git log --graph --pretty=oneline --abbrev-commit
*   cf810e4 (HEAD -> master) conflict fixed
|\
| * 14096d0 (bug) AND simple
* | 5dc6824 & simple
|/
* b17d20e branch test
* d46f35e (origin/master) remove test.txt
* b84166e add test.txt
* 519219b git tracks changes
* e43a48b understand how stage works
* 1094adb append GPL
* e475afc add distributed
* eaadf4e wrote a readme file
```

### 保存状态和 cherry-pick

如果工作区有未添加到暂存区的修改，可以通过 git stash 保存当前工作区的修改

```
$ git stash
Saved working directory and index state WIP on dev: 74df696 dev1
```

保存之后，通过 git status 查看就会发现工作区没有新内容

```
$ git status
On branch dev
nothing to commit, working tree clean
```

这时就可以切换到其他分支任意执行操作。等到需要时，再切换回当前分支，通过`git stash list`查看历史保存，通过`git stash pop`恢复

```
$ git stash list
stash@{0}: WIP on dev: 74df696 dev1

$ git stash pop
On branch dev
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   b.txt

no changes added to commit (use "git add" and/or "git commit -a")
Dropped refs/stash@{0} (90330a3b4bfe41fa9357c72194e79786709b205d)
```

这种操作常用于正在开发时（dev），需要紧急修复一个位于 master 分支上的 bug（创建一个 bug 分支用于修复并合并回 master）

这时还有一个问题，dev 分支是早期从 master 分支分出来的，所以，这个 bug 其实在当前 dev 分支上也存在。同样的 bug，要在 dev 上修复，我们只需要把 bug 分支上修改 bug 的那个提交所做的修改“复制”到 dev 分支即可。

具体来说，假如此时在 master 上创建了一个 bug 分支，修改后提交，并合并到 master 上：

```
$ git checkout -b bug
$ git add .
$ git commit -m "bug fix 1"
[bug 3cb6cc3] bug fix 1
 1 file changed, 1 insertion(+), 1 deletion(-)

$ git switch master
$ git merge --no-ff -m "merged bug fix 1" bug
$ git branch
  bug
* dev
  master

```

注意`"bug fix 1"`这个提交，表示的就是修复 bug 的那个提交。如果我们想把这个修改也复制到 dev 上，只需要使用`git cherry-pick <提交id>`即可。
即在 dev 分支上，pick 指定的提交。注意 pick 之前要先用 stash pop 恢复现场：

```
$ git cherry-pick 3cb6cc3
[dev a67ca95] bug fix 1
 Date: Mon Oct 24 14:34:17 2022 +0800
 1 file changed, 1 insertion(+), 1 deletion(-)
```

cherry-pick 相当于一次新的提交，这里的`[dev a67ca95] bug fix 1`就表示了一个 id 不同，但提交信息相同的提交。

## 远程仓库

### 多人协作

当你从远程仓库克隆时，实际上 Git 自动把本地的 master 分支和远程的 master 分支对应起来了，并且，远程仓库的默认名称是 origin。

推送分支，就是把该分支上的所有本地提交推送到远程库。推送时，要指定本地分支，这样，Git 就会把该分支推送到远程库对应的远程分支上：

```
$ git push origin master
```

如果要推送其他分支，比如 dev，就改成：

```
$ git push origin dev
```

git clone 默认只会加载 master 分支。如果想从远程库加载指定分支，必须创建远程 origin 的 dev 分支到本地

```
$ git checkout -b dev origin/dev
```

执行之后，就会发现本地文件都改成了指定分支的内容，同时通过 git branch 也可以看到 dev 分支。
实际上这个命令就是建立了与远程库的`track`。如果没有 track 还需要手动创建，这个在下面多人冲突中还会说道

---

当完成本地修改后，可以通过 git push 提交修改。
但是如果远程库被其他人提交过了，也就是说其他人的最新提交和你试图推送的提交有冲突（git 会提示你远程库在本地库的几个提交之前）。这时就需要先通过`git pull`拉取最新提交，然后在本地解决冲突，再提交回去。

```
$ git pull
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details.

    git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

    git branch --set-upstream-to=origin/<branch> dev
```

git pull 也失败了，原因是没有指定本地 dev 分支与远程 origin/dev 分支的链接，根据提示，设置 dev 和 origin/dev 的链接：
（注意这个执行结果在上面切换远程分支时也出现过。如果你本地的 dev 是通过`checkout -b dev origin/dev`这样来的，也就是说建立了 track，这一步就不会出现问题）

```
$ git branch --set-upstream-to=origin/dev dev
Branch 'dev' set up to track remote branch 'dev' from 'origin'.
```

然后再 pull：

```
$ git pull
Auto-merging env.txt
CONFLICT (add/add): Merge conflict in env.txt
Automatic merge failed; fix conflicts and then commit the result.
```

提示合并失败。这里就和之前说的分支合并冲突的情况一样了；git 同样会在冲突文件中标记出冲突内容；修改冲突内容后再次提交，再 push 就没有问题了

```
$ git commit -m "fix env conflict"
[dev 57c53ab] fix env conflict

$ git push origin dev
Counting objects: 6, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (6/6), 621 bytes | 621.00 KiB/s, done.
Total 6 (delta 0), reused 0 (delta 0)
To github.com:michaelliao/learngit.git
   7a5e5dd..57c53ab  dev -> dev
```

### 拉取远程分支

这是一个比较常用的操作，即远程分支进行了更新，需要拉取远程以更新本地分支。
实现这一目的的方法有两种：git fetch和git pull。两者的区别在于：
- git fetch只会下载远程库，但不会修改文件，需要自己去合并更新。即，当通过git fetch拉取了远程分支之后，会在本地创建一个远程分支的副本，然后我们需要使用`git merge origin/<branch name>`合并到当前分支上去
- git pull会直接拉取分支并合并，相当于直接修改了文件。也就是说，`git pull = git fetch + git merge`。如果远程分支的最新提交和本地出现冲突，就可能导致冲突，需要手动解决冲突才能合并。

因此大多数情况下拉取远程分支的命令是：
```
git pull origin <branch name>

// or
git fetch origin <branch name>
git merge origin/<branch name>
```

## 标签

Git的标签是版本库的快照，其实它就是指向某个commit的指针（分支可以移动，标签不能移动）
标签的主要目的是方便commit的标记。相比于哈希值id，tag明显更方便识别

命令：
```
$ git tag v1.0
```
默认标签是打在最新提交的commit上的。如果在后面加上指定的commit id，可以向指定id上添加标签
```
$ git tag v0.9 f52c633
```
标签可以作为commit的代指。可以像使用commit一样使用tag

创建的标签都只存储在本地，不会自动推送到远程。
如果要推送某个标签到远程，使用命令`git push origin <tagname>`,或者，一次性推送全部尚未推送到远程的本地标签：
```
$ git push origin v1.0
Total 0 (delta 0), reused 0 (delta 0)
To github.com:michaelliao/learngit.git
 * [new tag]         v1.0 -> v1.0

$ git push origin --tags
Total 0 (delta 0), reused 0 (delta 0)
To github.com:michaelliao/learngit.git
 * [new tag]         v0.9 -> v0.9
```
