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
- peerDependencies: 可以避免类似的核心依赖库被重复下载的问题。即一个库中把某个库声明为 peerDependencies 时，当用户下载该库时，就会检查是否已经安装了该库，如果已经有的话就不会安装。
  比如，开发者开发了一个基于 react 的库，然后把 react 写入 peerDependencies 而不是 dependies。这样在其他用户安装这个库时，npm 会自动检查是否已经安装了 React，如果已经安装了，则不会再次下载 React，否则会提示用户手动安装 React。如果版本不对或不兼容则会提示用户。

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

## dependencies 和 devDependencies

`devDependencies` 里面的插件只用于开发环境，不用于生产环境，
`dependencies` 是需要发布到生产环境的。

## semver

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

> 这里名为 webpack 的软链接有三个，即 webpack/webpack.cmd/webpack.ps1，这三个不同版本是在不同操作系统上使用的不同脚本，windows 一般执行.cmd，unix 一般执行 webpack，而 ps 指的是 shell 脚本，是一个通用的

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

这些被称为软链接。当通过 npm run 执行`webpack`命令时，就相当于在`.bin`目录下找到名为 webpack 的软链接来执行。

在 package-lock.json 中，可以查找到一个 bin 的配置项，这一项就是 npm 对于软链接的目标执行文件的配置，这一项的值就是要链接到的具体的文件，就像这样：

这个配置是用于指明 webpack 这软链接指向的具体要执行的文件的目录（即/node_modules/webpack/bin/webpack.js）。这样执行软链接时就会找到对应的文件来执行。

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

# npm & yarn & pnpm

## yarn

“Yarn 是由 Facebook、Google、Exponent 和 Tilde 联合推出了一个新的 JS 包管理工具 ，它的开发是为了提供 NPM 当时缺乏的更多高级功能（例如版本锁定），同时也使其更安全、更可靠、更高效。

yarn 的主要特点：

1. 速度快。速度快主要来自以下两个方面：

- 并行安装：无论 npm 还是 Yarn 在执行包的安装时，都会执行一系列任务。npm 是按照队列执行每个 package，也就是说必须要等到当前 package 安装完成之后，才能继续后面的安装。而 Yarn 是同步执行所有任务，提高了性能。
- 离线模式：如果之前已经安装过一个软件包，用 Yarn 再次安装时之间从缓存中获取，就不用像 npm 那样再从网络下载了。

2. 安全性。在 npm 早期版本中对安全性检查很差，而 yarn 在下载包时，它会在后台运行安全检查，利用包许可证信息来避免下载危险的脚本或导致依赖性问题。

## pnpm

特点：

1. 节省空间。主要是两个方面：

- 所有文件都保存在磁盘上的一个位置。安装包时，它们的文件是从那个地方硬链接的，不占用额外的磁盘空间。这允许跨项目共享相同版本的依赖项。
- 如果依赖不同版本的依赖项，则只会将不同的文件添加到存储中。例如，如果它有 100 个文件，而一个新版本只对其中一个文件进行了更改，则 pnpm update 只会将 1 个新文件添加到存储中，而不是仅为单个更改克隆整个依赖项。

如下图，可以看到当每次安装新的包时，大量的文件是被 reused 的。

![](https://pic.imgdb.cn/item/64174b28a682492fcc9bdc8c.jpg)

2. 速度快。

- pnpm 分三个阶段执行安装：
  - 依赖解析
  - 目录结构计算
  - 链接依赖项

这种方法比传统的解析、获取和写入所有依赖项到 node_modules 的三阶段安装过程要快得多。

如下图：

![](https://pic.imgdb.cn/item/6417471ea682492fcc9411f0.jpg)
![](https://pic.imgdb.cn/item/6417472aa682492fcc9434eb.jpg)

3. 非扁平的 node_modules 目录。使用 npm 或 Yarn 安装依赖项时，所有包都将提升到模块目录的根目录。而 pnpm 使用符号链接仅将项目的直接依赖项添加到模块目录的根目录中。

什么意思呢？举个例子，使用 yarn 或 npm 在安装包时，如果一个包依赖其他包，那么就会把这个包依赖的所有包全部安装。我们经常遇到的我只是在 package.json 里添加了几个包，装完之后 node_modules 多了一堆包，因为 npm / yarn 的安装会把包里面依赖的包也给安装到 node_modules，也就是安装之后 node_modules 其实是一个拍平的结构，避免出现依赖嵌套的结构

这种形式有一个致命的问题。比如现在有两个包 A 和 B，B 依赖于 A；当两者都不变化时没有影响，但如果 B 更新，B 新依赖的 A 是一个和旧的 A 不兼容的版本，那么就会出现问题，因为 npm/yarn 并不会主动更新 B 的所有依赖，而是只会更新 B 本身。

pnpm 在项目中安装的依赖实际上是一个软链接，链接到了.pnpm 文件夹。如果在项目中安装 react，那么实际上就是在.pnpm 文件夹下安装的 react。
实际上 pnpm 的安装结果就像这样，非常简洁：
![](https://pic.imgdb.cn/item/64174b66a682492fcc9c2049.jpg)

这里的 react 和 react-dom 都是软链接，链接到.pnpm 文件夹下的 react 文件夹。
而.pnpm 内的文件夹实际上是这样：
![](https://pic.imgdb.cn/item/64174bc0a682492fcc9c816d.jpg)
![](https://pic.imgdb.cn/item/64174beaa682492fcc9cb37d.jpg)

loose-envify 是 react 的依赖项，可以看到这也是一个软链接。对 react-dom 也同理，除了自己的库本体，其他的依赖项都是软链接的形式，相当于形成了一个嵌套结构，一个库套着另一个库，而非传统的扁平结构。

> 注意，pnpm 中既有软链接也有硬链接
> 硬链接是 pnpm 在各个项目中共享 node_modules 的方式，即在磁盘的一块连续地址上存储所有下载的 npm 包，并且通过硬链接的方式将 npm 包链接到各个项目的 node_modules 中。
> 软链接是在一个项目中形成嵌套关系的方式，即每个依赖内部有其他依赖的软链接。
> 硬链接和软链接的区别：
>
> - 硬链接是同一个文件的不同名称，可以理解为从不同的角度看同一个事物，所有对硬链接的操作都是对文件本身的操作，删除一个硬链接就像限制你的视角，文件本身并不会消失，当所有的硬链接都被删除，则文件本身也被删除了
> - 软链接只是一个快捷方式，删除所有的软链接也不会对文件有任何影响，对软链接的所有操作会被文件系统替换为对软链接指向的文件的操作，如果原始文件的位置被移动了，那么软链接就找不到对应的文件了，改动自然也无法同步到文件上，与硬链接不同，因为软链接只是个快捷方式，所以软链接甚至可以指向不存在的文件

## 幽灵依赖和 pnpm

幽灵依赖的定义是：子依赖提升造成的，虽然不会出现在 package.json 中（声明缺失），但是仍可以在项目中正常被 import，即可以访问非法（未声明的） npm 包。

举个例子：假设 react 依赖于 lodash，那么当我们在一个项目中安装 react 时，lodash 也会被同时安装。lodash 并不会出现在 package.json 中，但由于 npm 的扁平依赖方案，lodash 也会被安装在 node_modules 里。这种情况下，如果在项目中直接引入 lodash，是不会报错的；
这样的问题有两个：

1. 如果我们的项目是一个要发布的库，当我们发布到 npm，其他用户下载之后，由于 package.json 中没有 lodash，但项目中确实使用了 lodash，那么就可能会导致 lodash 找不到，或者版本不确定等问题。
2. 如果 react 不再依赖 lodash，或 react 依赖的 lodash 的版本发生变化了，当我们更新 react 不会更新 lodash，这时就可能导致 react 运行异常。

幽灵依赖产生的根本原因就是 npm 的扁平规则。npm 把所有安装的库的依赖，甚至依赖的依赖都扁平化到了整个 node_modules 内；这样的好处的方便管理和控制，坏处就是容易导致一些问题：

- 幽灵依赖：上面说的
- 依赖版本不确定：同样的 package.json 文件，install 依赖后可能不会得到同样的 node_modules 目录结构。
  比如还是假设 react 会安装 lodash1.0 版本，而另一个库安装了 lodash2.0，或者用户先安装了 lodash2.0，那么 node_modules 内的 lodash 版本就不确定了，很可能在 react 目录下（/node_modules/react/node_modules，即 react 自己的 node_modules）维护的是 1.0 版本的 lodash，而在外部（/node_modules，整个项目的 node_modules）又使用 lodash2.0 版本。
- 重复安装：还是上面的例子，react 内部的 lodash 和外部的 lodash 版本冲突，那么这时就会重复安装 lodash。除非版本不冲突，否则库都会在 react 内部的 node_modules 安装一次，不会复用外部的，就会导致重复安装的问题。

而 pnpm 解决上述问题的关键就是它的树形结构（有向图）。
具体来说，如果每个库的依赖都由他们自己维护，不会扁平化到整个 node_modules 中，就可以减少这些情况。
举个例子：

```
react
  |__node_modules
              |__lodash@1.0
react-dom
  |__node_modules
              |__lodash@2.0
```

每个库都维护自己的依赖，不会把 lodash 在根 node_modules 中安装；同时，这两个 lodash 也并非重复安装，而是都来自于 pnpm 的硬链接。也就是说，lodash1.0 和 2.0 版本并非都被安装到了项目的 node_modules 中，而是都维护在 pnpm 的根目录下。当项目中需要使用时，只需要将其硬链接到位置即可。这样就解决了重复安装的问题，也不会发生版本冲突。

至于幽灵依赖的问题，也不会发生，因为lodash是react和react-dom自己的node_modules里的，根本就不会安装到项目的node_modules中。这样的话，项目的package.json是什么结构，实际的node_modules内也就是什么结构，不会出现未声明的依赖。


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

## vite 的特点

1. 依赖预构建

Vite 通过在一开始将应用中的模块区分为**依赖**和**源码**两类，改进了开发服务器启动时间。Vite 将会使用 `esbuild` 预构建依赖。Esbuild 使用 Go 编写，并且比以 JavaScript 编写的打包器预构建依赖快 10-100 倍。

所谓“预构建”指的是在开发模式下，把一些模块转化成 vite 需要的形式，比如：

- 把非 esm 模块打包成 esm 形式
- 把很多文件的库合并到一个或者几个文件中，类似 webpack 打包过程。比如 lodash-es 模块数量非常多，预构建过程会将其打包到一个文件中。

> esbuild 执行的是开发模式下的依赖构建以及源码编译工作，并不是完成生产模式打包的工具。

2. 源码的直接引入

对于用户源码，在开发模式下，vite 不需要像 webpack 一样把代码打包成 bundle 再给浏览器执行，而是直接利用浏览器对模块的支持特性，配合上类似 devserver 的内置服务端，可以实现直接在用户浏览器中运行源代码。这个过程导致 vite 开发模式下的编译过程比 webpack 快了非常多

3. 缓存

Vite 会将预构建的依赖缓存到 node_modules/.vite，而解析后的依赖请求会以 HTTP 头 max-age=31536000,immutable 强缓存，一旦被缓存，这些请求将永远不会再到达开发服务器。
而源码通常采用协商缓存，这样同时也有利于 hmr。

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
  "parse": "Babel-ESLint", // 解析器，表示用什么工具把代码转成ast。如果是ts项目则用@typescript-eslint/parser
  // 配置解析器的属性
  "parserOptions": {
    "ecmaVersion": 6, // es版本，推荐"latest"
    "sourceType": "module", // 是否使用esmodule
    "ecmaFeatures": {
      // 启用jsx，但是还需要特殊的react插件
      "jsx": true
    }
  },
  // extends和plugins都可以加载预设配置
  // 区别是extends可以使用预设配置的组合，即官网上提供的配置的组合
  // 如果希望是全新的配置，就需要使用plugins
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

## 插件编写

### 前置知识

[eslint 官方文档](https://zh-hans.eslint.org/docs/latest/extend/custom-rules)给出了编写 eslint 插件的规范和方法。

1. espree：eslint 使用的 ast 生成工具，不同于 webpack 使用的 acron。
2. 选择器：选择器是可以用来匹配抽象语法树（AST）中节点的字符串。AST 选择器的语法与 CSS 选择器的语法类似。

[选择器](https://zh-hans.eslint.org/docs/latest/extend/selectors)是 ast 中很重要的概念。比如下面这张图，圈出来的部分都是选择器：

![](https://pic.imgdb.cn/item/6421e141a682492fcc7d03fe.jpg)

选择器的作用和 css 选择器类似，就是通过指定的语法访问到所有节点。比如选择器名字叫"CallExpression"，那就会匹配所有的调用语句，把他们对应的节点传入到函数中去。

选择器有很多种，常见的有：

- AST 节点类型：ForStatement
- 通配符（匹配所有节点）：\*
- 字段：FunctionDeclaration > Identifier.id （意思就是选择所有 FunctionDeclaration 节点下的 Identifier 节点的 id 属性）
- 后裔：FunctionExpression ReturnStatement
- 子项：UnaryExpression > Literal

看起来和 css 选择器很类似。只不过 css 选择器选择的是 dom 元素，而 ast 选择的是 ast 元素
在 eslint 插件中，可以利用选择器选出部分特定的 ast 节点，从而减小搜索范围。

通常将选择器写在 create 函数的返回对象中：

```js
module.exports = {
  create(context) {
    // ...

    return {
      // 这个监听器将为所有有块的 IfStatement 节点被调用。
      "IfStatement > BlockStatement": function (blockStatementNode) {
        // ...你的逻辑在此
      },

      // 这个监听器会调用所有有 3个 以上参数的函数声明。
      "FunctionDeclaration[params.length>3]": function (
        functionDeclarationNode
      ) {
        // ... 你的逻辑在此
      },
    };
  },
};
```

在拿到 AST 之后，ESLint 会以"从上至下"再"从下至上"的顺序遍历每个选择器两次。我们所监听的选择器默认会在"从上至下"的过程中触发，如果需要在"从下至上"的过程中执行则需要添加:exit，在上文中 CallExpression 就变为 CallExpression:exit。

### 具体编写

编写的全过程可以参考https://obkoro1.com/web_accumulate/accumulate/tool/ESLint%E6%8F%92%E4%BB%B6.html#%E5%8F%91%E5%B8%83%E6%8F%92%E4%BB%B6
按照这个方式试了，确实成功了。

插件源码比较简单，就是一个用于检查 react hooks 的：

- useState 的第二个返回值必须是 setXXX 的形式
- useRef 的返回值必须是 xxxRef 的形式
- useReducer 的返回值必须是 xxxReducer 的形式

编写的逻辑就是查看[ast explorer](https://astexplorer.net/)对 ast 的解析，写一个示例代码，然后对照着查看什么样的 ast 结构。
比如下面这个例子，就是先查找一个 node，为函数调用类型，名称为`useState`；然后在其父元素获取它的返回值，检查是否是解构数组形式。最后获取数组的第二项，检查其是否满足要求

参考下图：
![](https://pic.imgdb.cn/item/64188312a682492fcccb8ed6.jpg)

```js
// setstate-startwith-set.js
module.exports = {
  meta: {
    type: "problem",
    docs: {
      description: "useState的第二个返回值应该以set开头，并且为camelCase写法",
    },
    fixable: null, // Or `code` or `whitespace`
  },

  create: function (context) {
    return {
      CallExpression: function (node) {
        if (
          node.callee.type === "Identifier" && //
          node.callee.name === "useState" && //找到useState调用语句
          node.parent.type === "VariableDeclarator" &&
          Array.isArray(node.parent.id.elements) && // 从父元素找到useState的返回值数组
          node.parent.id.elements.length === 2 &&
          node.parent.id.elements[0].type === "Identifier" &&
          node.parent.id.elements[1].type === "Identifier" // 取出数组的两项
        ) {
          const stateName = node.parent.id.elements[0].name;
          const setStateName = node.parent.id.elements[1].name;
          const isStartWithSet = setStateName.startsWith("set");
          const isSetStateCamelCase = isUpperCase(setStateName.slice(3)[0]);
          const targetSetStateName =
            "set" + stateName.slice(0, 1).toUpperCase() + stateName.slice(1);

          if (!isStartWithSet || !isSetStateCamelCase) {
            context.report({
              node,
              message: `useState返回值应该遵循更好的格式：'[${stateName},${targetSetStateName}]'`,
            });
          }
        }
      },
    };
  },
};
```

一个规则对应一个文件。编写其他 hooklint 的方式类似。
注意必须通过 npm publish 发布到 npm 上，再下载并在 eslint 中使用：

```js
plugins: ["react", "@typescript-eslint", "sngthrdlint"],
rules: {
  "sngthrdlint/setstate-startwith-set": "error",
},
```

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
实现这一目的的方法有两种：git fetch 和 git pull。两者的区别在于：

- git fetch 只会下载远程库，但不会修改文件，需要自己去合并更新。即，当通过 git fetch 拉取了远程分支之后，会在本地创建一个远程分支的副本，然后我们需要使用`git merge origin/<branch name>`合并到当前分支上去
- git pull 会直接拉取分支并合并，相当于直接修改了文件。也就是说，`git pull = git fetch + git merge`。如果远程分支的最新提交和本地出现冲突，就可能导致冲突，需要手动解决冲突才能合并。

因此大多数情况下拉取远程分支的命令是：

```
git pull origin <branch name>

// or
git fetch origin <branch name>
git merge origin/<branch name>
```

## 标签

Git 的标签是版本库的快照，其实它就是指向某个 commit 的指针（分支可以移动，标签不能移动）
标签的主要目的是方便 commit 的标记。相比于哈希值 id，tag 明显更方便识别

命令：

```
$ git tag v1.0
```

默认标签是打在最新提交的 commit 上的。如果在后面加上指定的 commit id，可以向指定 id 上添加标签

```
$ git tag v0.9 f52c633
```

标签可以作为 commit 的代指。可以像使用 commit 一样使用 tag

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

## git 工作流

Git 作为一个源码管理系统，不可避免涉及到多人协作。

协作必须有一个规范的工作流程，让大家有效地合作，使得项目井井有条地发展下去。"工作流程"在英语里，叫做"workflow"或者"flow"，原意是水流，比喻项目像水流那样，顺畅、自然地向前流动，不会发生冲击、对撞、甚至漩涡。

三种广泛使用的工作流程：

- Git flow
- Github flow
- Gitlab flow

### Git flow

首先，项目存在两个长期分支。

- 主分支 master
- 开发分支 develop

前者用于存放对外发布的版本，任何时候在这个分支拿到的，都是稳定的分布版；后者用于日常开发，存放最新的开发版。

其次，项目存在三种短期分支。

- 功能分支（feature branch）
- 补丁分支（hotfix branch）
- 预发分支（release branch）

一旦完成开发，它们就会被合并进 develop 或 master，然后被删除。

Git flow 的优点是清晰可控，缺点是相对复杂，需要同时维护两个长期分支。大多数工具都将 master 当作默认分支，可是开发是在 develop 分支进行的，这导致经常要切换分支，非常烦人。

更大问题在于，这个模式是基于"版本发布"的，目标是一段时间以后产出一个新版本。但是，很多网站项目是"持续发布"，代码一有变动，就部署一次。这时，master 分支和 develop 分支的差别不大，没必要维护两个长期分支。

### Github flow

它只有一个长期分支，就是 master

- 第一步：根据需求，从 master 拉出新分支，不区分功能分支或补丁分支。
- 第二步：新分支开发完成后，或者需要讨论的时候，就向 master 发起一个 pull request（简称 PR）。
- 第三步：Pull Request 既是一个通知，让别人注意到你的请求，又是一种对话机制，大家一起评审和讨论你的代码。对话过程中，你还可以不断提交代码。
- 第四步：你的 Pull Request 被接受，合并进 master，重新部署后，原来你拉出来的那个分支就被删除。（先部署再合并也可。）

Github flow 的最大优点就是简单，对于"持续发布"的产品，可以说是最合适的流程。

问题在于它的假设：master 分支的更新与产品的发布是一致的。也就是说，master 分支的最新代码，默认就是当前的线上代码。

可是，有些时候并非如此，代码合并进入 master 分支，并不代表它就能立刻发布。比如，苹果商店的 APP 提交审核以后，等一段时间才能上架。这时，如果还有新的代码提交，master 分支就会与刚发布的版本不一致。另一个例子是，有些公司有发布窗口，只有指定时间才能发布，这也会导致线上版本落后于 master 分支。

上面这种情况，只有 master 一个主分支就不够用了。通常，你不得不在 master 分支以外，另外新建一个 production 分支跟踪线上版本。

### Gitlab flow

Gitlab flow 是 Git flow 与 Github flow 的综合。它吸取了两者的优点，既有适应不同开发环境的弹性，又有单一主分支的简单和便利。
Gitlab flow 的最大原则叫做"上游优先"（upsteam first），即只存在一个主分支 master，它是所有其他分支的"上游"。只有上游分支采纳的代码变化，才能应用到其他分支。

![](https://pic.imgdb.cn/item/64175692a682492fccaa5685.jpg)

对于"持续发布"的项目，它建议在 master 分支以外，再建立不同的环境分支。比如，"开发环境"的分支是 master，"预发环境"的分支是 pre-production，"生产环境"的分支是 production。

开发分支是预发分支的"上游"，预发分支又是生产分支的"上游"。代码的变化，必须由"上游"向"下游"发展。比如，生产环境出现了 bug，这时就要新建一个功能分支，先把它合并到 master，确认没有问题，再 cherry-pick 到 pre-production，这一步也没有问题，才进入 production。

只有紧急情况，才允许跳过上游，直接合并到下游分支。

![](https://pic.imgdb.cn/item/641756a5a682492fccaa8ed0.jpg)

对于"版本发布"的项目，建议的做法是每一个稳定版本，都要从 master 分支拉出一个分支，比如 2-3-stable、2-4-stable 等等。

以后，只有修补 bug，才允许将代码合并到这些分支，并且此时要更新小版本号。

### Gitflow flow

![](https://pic.imgdb.cn/item/641756f0a682492fccab4983.jpg)

分支作用：

- master：主分支，负责上线版本
- hotfix：直接从 master 创建，修复后直接合并到 master
- release：作为发布前的一个前置分支，可以用于测试
- develop：开发分支，所有开发完成的功能都在这里记录
- feature：功能分支，从开发分支分下来，开发每个具体功能，再合并到开发分支中。

1. Hot-fix 分支是唯一一个从 master 分支创建的分支，并且直接合并到 master 分支而不是 develop 分支。仅在必须快速修复生产环境问题时使用。该分支的一个优点是，它使你可以快速修复并部署生产环境的问题，而无需中断其他人的工作流，也不必等待下一个发布周期。
   将修复合并到 master 分支并进行部署后，应将其合并到 develop 和当前的 release 分支中。这样做是为了确保任何从 develop 分支创建新功能分支的人都具有最新代码。

在将所有准备发布的功能的代码成功合并到 develop 分支之后，就可以从 develop 分支创建 release 分支了。

2. Release 分支不包含新功能相关的代码。仅将与发布相关的代码添加到 release 分支。例如，与此版本相关的文档，错误修复和其他关联任务才能添加到此分支。
   一旦将此分支与 master 分支合并并部署到生产环境后，它也将被合并回 develop 分支中，以便之后从 develop 分支创建新功能分支时，新的分支能够具有最新代码。

# Rollup

rollup 是一个和 webpack 功能类似的打包工具以及模块化构建工具。

## 特点（和 webpack 的区别）

1. 对 es 模块更好的支持。

相对于 webpack，像 rollup 这样现代化的构建工具一般对 esm 的支持都会更友好。rollup 的打包结果默认是 es 模式的，相对于通过 cjs 引入全部模块的方式，部分导入的 es 模块构建结果更小、更轻量。因此 rollup 也更适合打包一些小型的项目和库

2. 更好的 tree-shaking

rollup 相对于 webpack 将会有更好的 tree-shaking 效果，同时它对 tree-shaking 的支持也比 webpack 更早。

3. 对静态资源的处理不是很好

相对于 webpack，rollup 对图片、css 等静态资源并不能处理。rollup 是一个纯处理 js 的打包工具，因此它更偏向于打包一些 js 的库；但它也可以通过插件来实现类似 webpack 的静态资源打包能力。

也就是说，rollup 其实更适合于打包一个 js 库，而非启动一个完整的项目。

# 模块化

## 模块化对循环加载的解决方案

“循环加载”简单来说就是就是脚本之间的相互依赖，比如 a.js 依赖 b.js，而 b.js 又依赖 a.js。例如：

```js
// a.js
const b = require("./b.js");

// b.js
const a = require("./a.js");
```

这是一个很常见的场景，如果没有处理机制，则会造成递归循环，而递归循环是应该被避免的。

cjs 和 esm 对循环加载的解决方式不同：

### CommonJS 模块的循环加载

CommonJS 的一个模块，就是一个脚本文件。require 命令第一次加载该脚本，就会执行整个脚本，然后在内存生成一个对象。对于同一个模块无论加载多少次，都只会在第一次加载时运行一次，之后再重复加载，就会直接返回第一次运行的结果（除非手动清除系统缓存）。

```js
// module
{
  id: '...', //模块名，唯一
  exports: { ... }, //模块输出的各个接口
  loaded: true, //模块的脚本是否执行完毕
  ...
}
```

CommonJS 模块解决循环加载的策略就是：**一旦某个模块被循环加载，就只输出已经执行的部分，没有执行的部分不输出。**
意思就是，require 语句可以看做是跳到另一个模块中去执行代码，当一个模块执行过程中发现某个 require 语句导入的形成了循环依赖，就不会再进入去执行这个模块，而是继续执行本模块剩下的内容，直到本模块执行完成后才退回上一个模块。

举个例子：

```js
// a.js
console.log("a starting");
exports.done = false;
const b = require("./b.js");
// 这里之后的语句是执行完b.js才执行的
console.log("in a, b.done = %j", b.done);
exports.done = true;
console.log("a done");

// b.js
console.log("b starting");
exports.done = false;
const a = require("./a.js"); // 循环依赖的位置，这里不会再进去执行a.js内的语句，而是直接提取之前加载的a.js内的exports
// 这里之后的语句会连着上面的执行，因为a.js是加载过的依赖，不会反复进入
console.log("in b, a.done = %j", a.done);
exports.done = true;
console.log("b done");

// main.js
console.log("main starting");
const a = require("./a.js");
const b = require("./b.js");
console.log("in main, a.done = %j, b.done = %j", a.done, b.done);
```

上面代码的执行顺序：

1. 输出字符串 main starting 后，加载 a 脚本
2. 进入 a 脚本，a 脚本中输出的 done 变量被设置为 false，随后输出字符串 a starting，然后加载 b 脚本
3. 进入 b 脚本，随后输出字符串 b starting，接着 b 脚本中输出的 done 变量被设置为 false，然后加载 a 脚本，发现了循环加载，此时不会再去执行 a 脚本，只输出已经执行的部分（即 a.js 中的`exports.done = false`，就是在 b.js 内的 a 变量），随后输出字符串 in b, a.done = false，接着 b 脚本中输出的 done 变量被设置为 true，最后输出字符串 b done，b 脚本执行完毕，回到之前的 a 脚本
4. a 脚本继续从第 4 行开始执行，随后输出字符串 in a, b.done = true，接着 a 脚本中输出的 done 变量被设置为 true，最后输出字符串 a done，a 脚本执行完毕，回到之前的 main 脚本
5. main 脚本继续从第 3 行开始执行，加载 b 脚本，发现 b 脚本已经被加载了，将不再执行，直接返回之前的结果，最终输出字符串 in main, a.done = true, b.done = true，至此 main 脚本执行完毕

### ESM 的循环加载

ES6 模块是编译时加载：即编译时遇到模块加载命令 import，不会去执行这个模块，只会输出一个只读引用，等到真的需要用到这个值时（即运行时），再通过这个引用到模块中取值。换句话说，模块内部这个值改变了，仍旧可以根据输出的引用获取到最新变化的值。

跟 CommonJS 模块一样，ES6 模块也不会再去执行重复加载的模块，并且解决循环加载的策略也一样：一旦某个模块被循环加载，就只输出已经执行的部分，没有执行的部分不输出。

但 ES6 模块的循环加载与 CommonJS 存在本质上的不同。由于 ES6 模块是动态引用，用 import 从一个模块加载变量，那个变量不会被缓存（是一个引用），所以只需要保证真正取值时能够取到值，即已经声明初始化，代码就能正常执行。

举个 🌰：

```js
// a.js
import { bar } from "./b";
console.log("a.js");
console.log(bar);
export let foo = "foo";

// b.js
import { foo } from "./a";
console.log("b.js");
console.log(foo);
export let bar = "bar";
```

从 a.js 开始执行，就会发现报错 foo is not defined，

简单分析一下 a 脚本执行过程：

1. 开始执行 a 脚本，加载 b 脚本
2. 进入 b 脚本，加载 a 脚本，发现了循环加载，此时不会再去执行 a 脚本，只输出已经执行的部分，但此时 a 脚本中的 foo 变量还未被初始化，接着输出字符串 a.mjs，之后尝试输出 foo 变量时，发现 foo 变量还未被初始化，所以直接抛出异常

### webpack 对循环依赖的处理

无论是 esm 还是 cjs，webpack 都会把它翻译成自己的 webpackrequire 形式的模块化。
但是这种方式仍然会出现循环依赖的问题。以上面 ESM 循环依赖的例子测试，得到的报错和直接在 node 环境下调用一样，foo is not defined。

默认 webpack 也只是对导入导出做出解析，而不会检查循环依赖，包括编译之后的文件内容也是可以看出来循环依赖的。

### 循环依赖的解决

#### 语句顺序

要知道循环依赖本身不是一个问题。对 esm 来说，所出现的问题实际上调用顺序的错误。比如上面 esm 的例子中，本质上相当于这样：

```js
console.log(foo);
let foo = "foo";
```

这种情况肯定会报错。类似这样的循环依赖在项目中出现的比较少，通常常见的循环依赖都是很多个文件串起来的。
因此想要解决循环依赖导致的问题，核心就是要调整好代码顺序。具体来说，就是调整好 import 的顺序。

还是上面的例子，假如我们让 foo 变量在导入之前就声明，就不会报错了：

```js
// a.js
export let foo = "foo";
import { bar } from "./b";
console.log("a.js");
console.log(bar);

// b.js
import { foo } from "./a";
console.log("b.js");
console.log(foo);
export let bar = "bar";
```

这样加载 bar 之前，foo 变量已经在 a.js 中声明，再去执行 b.js 时得到的 foo 就是一个确定的值。

不过这种方式太随机了，对于复杂的循环依赖，其实很难知道到底是哪里出了问题。
并且如果有多个 import 的话，不同 import 之间的顺序也很重要。通常某些 import 加载的模块可能依赖于其他 import 加载的模块，因此需要把被依赖的调整到前面

#### 统一导出

参考：https://medium.com/visual-development/how-to-fix-nasty-circular-dependency-issues-once-and-for-all-in-javascript-typescript-a04c987cf0de

另外一种方式更好一些，就是：

- 引入一个 index.js 和 internal.js 文件
- internal.js 模块从项目中的每个本地模块导入和导出所有内容
- 项目中的每个其他模块仅从 internal.js 文件导入，从不直接从项目中的其他文件导入
- 如果是一个库，那么 index.js 内部需要从 internal.js 导入和导出想要向外界公开的所有内容

internal.js 起到的作用就是把所有模块想要导出的内容统一导入进来再导出。可以使用`export * from '...'`这样的语句
在 internal 内部调整各个 import 之间的顺序

```js
// main.js
import { a, b } from "./internal";
export const main = { a, b };

// a.js
import { main } from "./internal";
export let a = "a";

// b.js
import { main } from "./internal";
export let b = "b";

// internal.js
export * from "./main";
export * from "./a";
export * from "./b";
```

另外，像上面那种两个模块循环依赖的方式是很难解决的。不过这种方式也比较容易定位，需要自己手动解除问题，比如把变量拆分到单独的文件中，或者删除部分导入等。

# MVC/MVVM/MVP

## MVC

MVC 全名是 Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范，用一种业务逻辑、数据、界面显示分离的方法组织代码，将业务逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。MVC 被独特的发展起来用于映射传统的输入、处理和输出功能在一个逻辑的图形化用户界面的结构中。

![](https://pic.imgdb.cn/item/64218586a682492fccdc1c8c.jpg)

- Model（模型）表示应用程序核心（如数据库）。
- View（视图）显示效果（HTML 页面）。
- Controller（控制器）处理输入（业务逻辑）。通常是用于获取模型中的数据（数据库），然后提供给视图（html）。

举个例子，对于一个 web 应用来说，这三个部分分别应该是：

- model：数据库，即该 web 应用存储数据的数据库
- view：展示的界面，可以是 html、css 以及用于实现交互效果的 js
- controller：通常指后端代码，从 model 中读取数据并提供给 view。

controller 可以修改 model（后端代码操作数据库），同时 view 也可以对 controller 进行控制（比如 dom 事件触发的 ajax）。最后 model 把数据嵌入到 view 中并向用户展示。
当然这是传统的 mvc 模式。如果是前端框架，那么通常 controller 就是控制数据的 js 代码，model 则是一些数据结构，这时 js 是运行在框架提供的运行时内的。

React 是一种类 MVC 架构。React 组件充当了 View 和 Controller 的角色，组件内的 state 就是 Model；contoller 可以接收 view 的控制（事件）来修改 model 中的数据（setState），最后反映在 view 上（state）

优点：

- 耦合性低：可以理解为 M、V、C 三个部分不互相依赖，三个部分比较独立，改变其中一个并不会影响另外两个。同样，一个应用的业务流程或者业务规则的改变只需要改动 MVC 的模型层即可
- 利于修改
- 重用性高

缺点：

- 复杂：对于小型、简单的应用，如果使用 mvc，就会显得比较复杂。
- 控制器负担重：在 MVC 模式中，控制器负责处理所有的用户请求和数据更新，当应用程序变得复杂时，控制器会变得越来越臃肿，难以维护。

## MVVM

![](https://pic.imgdb.cn/item/642187a2a682492fccdf5cec.jpg)

MVVM 的核心是双向绑定，即 ViewModel 和 View 之间的数据流向是双向的。View 的变动会自然更新 ViewModel，而 ViewModel 内的数据改变也会更新 View。
MVVM 模式的核心思想是将视图和数据绑定在一起，使得视图的状态和数据模型的状态能够自动同步，从而简化开发人员的工作。

MVVM 中的三个部分分别负责以下内容：

- 模型（Model）：负责管理应用程序的数据和业务逻辑，例如数据的获取、处理、存储等。
- 视图（View）：负责展示应用程序的用户界面，例如 HTML 页面、CSS 样式等。
- 视图模型（ViewModel）：负责将模型中的数据转换为视图所需要的格式，并且将视图中的用户操作转换为模型中的数据操作。视图模型充当着视图和模型之间的桥梁，通过数据绑定将视图和模型连接在一起，实现视图和模型的自动同步。

在 MVC 模式中，如果想要把 Model 中的数据展现在 view 上，需要开发者手动操作（比如通过 dom、jsp 等方式）；而 MVVM 模式的宗旨就是让数据和视图自动同步，从而避免了手动操作 DOM 元素的复杂性和错误风险。

vue 就是很典型的 MVVM 架构模式。vue 充当了一个 viewModel 的角色，将用户对 view 的操作（事件及传递的数据）修改到 model 上（js 对象或数组），而负责这部分同步的 vue 代码及框架本身都是 viewModel 角色。

优点：和 MVC 基本一样，包括低耦合、方便维护和分工等
缺点：最大的缺点是双向绑定的性能和调试问题。当数据量较大或数据变化频繁时，会影响应用程序的响应速度。

## MVP

MVP 是 MVVC 的前身，同时是 MVC 的衍生。它将视图和控制器分开，引入了 Presenter 层来协调和管理视图和模型之间的交互。

![](https://pic.imgdb.cn/item/64218d14a682492fcce85569.jpg)

在 MVP 架构中，模型（Model）仍然负责管理应用程序的数据，视图（View）负责展示数据给用户，Presenter 则负责处理用户的操作并更新数据和视图。Presenter 是一个介于视图和模型之间的中间层，它负责将视图和模型解耦，并协调它们之间的交互。Presenter 通过向视图提供接口来控制视图的显示和行为，并通过向模型提供接口来更新和获取数据。

与 MVVM 架构不同的是，Presenter 负责将模型中的数据格式化后传递给视图，而不是直接和视图进行双向数据绑定。View 非常薄，不部署任何业务逻辑，称为"被动视图"（Passive View），即没有任何主动性，而 Presenter 非常厚，所有逻辑都部署在那里。

也就是说，MVP 中模型和视图之间还需要一个手动修改的过程。相对于 MVVC 的自动绑定来说需要更多的代码。

MVP 架构模式的缺点：（优点和 MVC 差不多）

- 开发成本高：MVP 模式需要开发人员手动实现 Presenter 层，相对于 MVVM 模式，开发成本和学习成本都会更高。
- 代码量多：MVP 模式需要开发人员手动实现 Presenter 层，因此代码量会比 MVVM 模式多，对于小型项目来说可能会显得冗余。但相对 MVVM 来说，如果 present 层实现的好，是要比 MVVM 的双向绑定更加效率高的。

# Turbopack

## 优点

turbopack 是一个针对 JavaScript 和 TypeScript 优化的增量打包器。
turbopack 由 rust 编写，主要的优势有：

1. 开发环境下的打包：类似 webpack，turbopack 也会在开发环境下对源代码进行打包工作。不同与 vite 直接利用浏览器的 module 功能，Turbopack 依然会对开发环境下的源代码进行打包，不过这个打包过程使用的是 rust，因此比 webpack 更快。

2. 增量计算能力：充分利用函数级别的缓存。按照官方文档的说法就是，“永远不会把一个函数执行第二遍”。因此 Turbo 可以缓存程序中任何函数的结果。当程序再次运行时，函数将不会重新运行，除非它的参数改变了。

这样说可能比较抽象。实际上这种“函数”并不是指代码中实现的功能的函数，更多是导入导出的组件级别的函数。
举个例子：

```js
import Header from "../components/header";
import Footer from "../components/footer";
export default function Home() {
  return (
    <div>
            <Header />      <h1>Home</h1>
            <Footer />   {" "}
    </div>
  );
}
```

上面这段代码中，如果更新 Footer 这个组件并保存，在 Webpack 中 Header 和 Footer 两个组件都会被重新编译，而仔细观察 Turbopack 输出的缓存文件，会发现只有 Footer 组件被重新编译，而 Header 组件则使用的是上一次的编译结果。
通过这种缓存机制，去除大量重复的工作，使得编译的效率大幅度的提升。

3. 懒加载：turbopack 提供自动懒加载的功能。相对于 webpack 需要手动代码分割，turbopack 可以自动得出所需的最小代码，将其打包，而不会将整个模块全部打包。对于第一次启动来说，只需要计算入口所需的模块及其依赖，将其打包之后发送到浏览器，这个过程要比原生 esm 快得多。

在懒加载之上，turbopack 还引入了请求级编译的功能，即浏览器需要什么才编译什么。比如浏览器请求 HTML，只编译 HTML，而不是 HTML 引用的任何内容。

4. 编译工具：Turbopack 的速度如此之快有一个很大的原因是使用了 SWC 作为编译器。大部分 Webpack 的项目编译都是使用 Babel 编译和转换，由于 Babel 本身也是使用 Javascript 编写，转换效率并不理想，而 Turbopack 原生使用 SWC 作为编译器。SWC 是一款 Rust 编写的 Javascript 代码编译器，官方宣称其编译速度是 Babel 的 20 倍（ Webpack 也可以使用 SWC）。

## 缺点

turbopack 的缺点也很明显

1. 未成熟：目前只能在 nextjs 中体验，官方文档上使用的是一个 nextjs 项目的实例，而 next12 之后用的都是 turbopack，不能单独拿出来用。更深入的了解也只能依赖官方文档
2. 生态一般：webpack、vite 相关的插件目前还无法兼容和创建新的生态，因此生态比较一般。
