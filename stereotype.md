# HTML

1. `<link>`、`<meta>`标签都是干什么用的
2. `<script>`标签的 defer 和 async 表示什么，还有哪些属性
3. preload 和 prefetch 有什么区别
4. 语义化是什么，作用？

# CSS

1.  什么是 css 的层叠和继承，**哪些属性可以继承**，哪些不能？
2.  **css 选择器有哪些，优先级是怎么样的**
3.  伪类和伪元素有什么区别，常见的有哪些
4.  **css 的单位都有哪些**
5.  常见的块级元素、行内块、行内元素都有哪些，什么特点？
6.  **盒模型是什么**
7.  外边距折叠是什么情况，怎么处理
8.  **BFC 是什么**，有什么用，怎么触发
9.  浮动是什么，有什么用，怎么清除
10. position/display 有哪些属性，有什么特点
11. 隐藏元素的方法有哪些，各自什么特点
12. 单论 css，有哪些优化方法
13. **怎么创建一个响应式图片**
14. flex 和 grid 怎么用，有哪些属性
15. css 雪碧图
16. **怎么判断元素是否进入可视区域？判断依据的属性都是什么意思？**
17. css 怎么画一个三角形
18. **垂直居中、水平居中、垂直水平居中**，如果不知道父元素宽高的话怎么做？如果不知道子元素宽高怎么做？都不知道怎么做？
19. 两栏布局怎么实现
20. **三栏布局怎么实现**
21. 实现一个高度始终为宽度一半的矩形
22. **什么是响应式布局，怎么实现？**

# JS

1. 数据类型有哪些，分为几种
1. 引用数据类型和基本数据类型有什么区别？
1. `==`的转换规则
1. 对象转化为原始数据类型的规则
1. **检测数据类型的方法**，检测数组有几种方法，函数呢？
1. 怎么实现一个浅拷贝、深拷贝
1. Symbol 是什么，有哪些用处
1. 迭代器是什么，如何创建，怎么使用
1. 怎么把一个类数组对象转为数组
1. 怎么创建一个指定大小的空数组
1. Map 和 Set 是什么，有什么特点
1. 对象怎么配置一个属性（比如可读、可写）
1. 函数声明有几种方式
1. **柯里化是什么**，怎么实现一个一层柯里化，怎么实现任意多层？
1. 箭头函数有哪些限制
1. **闭包是什么，怎么产生的，有什么用，利弊**
1. **作用域和作用域链是什么**
1. **执行上下文是什么，和作用域的区别，有哪些内容**
1. **this 是什么，有什么用，怎么指定 this 的指向？**
1. new 的执行过程？
1. **原型链是什么**
1. js 的继承方式
1. **Promise 的功能，解决的问题，怎么实现的，怎么使用，特点，有哪些 api**
1. async/await 的原理（协程），使用
1. 模块化的发展历程
1. **commonjs 和 es6 module 的不同**
1. Proxy 和 Reflect 是啥
1. 事件循环，怎么实现的异步任务，宏任务和微任务？
1. js 的内存管理、垃圾回收方式
1. dom 树概述，dom 节点的概念（七种节点）
1. dom 事件流？
1. 事件对象是什么，怎么产生的，有什么用，自定义事件是什么。浏览器的事件模型
1. 浏览器默认事件有哪些，怎么阻止
1. 怎么封装一个 ajax（xhr 的用法）
1. 怎么使用 fetch
1. 防抖和节流怎么实现
1. webcomponents 是啥
1. shadowdom 是啥
1. 一个 HTML 页面的生命周期
1. ArrayBuffer 是什么，有什么用
1. Blob 和 File 是什么，有什么用
1. Generator 相关
1. H5 history api

# 网络

1. URL 和 URI 是什么
1. 同源策略，跨域解决方案（jsonp 手写）
1. OSI7 层网络模型、TCP/IP4 层模型，5 层模型
1. session、token、cookie 是什么，有什么特点和区别
1. HTTP 的基本特点
1. HTTP 的请求和响应报文结构
1. HTTP 的常见字段
1. HTTP 的方法有哪些
1. **HTTP 状态码**
1. HTTPS 是什么，怎么保证安全，**握手过程**？
1. **HTTP1.0 1.1 2.0 3.0 都有哪些变化**
1. HTTP 缓存，有哪些字段，缓存存放在本地的位置
1. DNS 是啥，怎么查询的
1. CDN 是啥，有什么用
1. UDP 协议的特点、应用、报文结构，和 TCP 的区别
1. TCP 的特点
1. **TCP 的三次握手四次挥手**
1. TCP 的报文段结构
1. TCP 的可靠数据传输实现
1. TCP 的流量控制实现
1. TCP 的拥塞控制
1. IP 协议中的子网掩码和前缀是什么
1. 特殊的 IP 地址
1. 链路层有什么作用
1. **从浏览器输入 url 到显示页面的过程**
1. CSP 是什么，怎么开启
1. **XSS 攻击和防范**
1. CSRF 攻击和防范

# 浏览器

1. 浏览器中的主要进程和线程有哪些
1. **浏览器渲染页面的过程**
1. 怎么优化浏览器加载过程、渲染过程
1. **重绘重排，触发原因，优化**
1. 解释和编译型语言
1. v8 怎么执行一段 js 代码的

# webpack

概述，功能，为什么需要 webpack
核心概念

- chunk
  - 三种chunk
  - hash
- module
- bundle
- dependence
- plugin
- loader

配置：

- 配置文件
  - 导出的值的不同：
    - 对象
    - 函数
    - 数组
  - 多个配置文件
    - webpack.common.js/webpack.development.js/webpack.production.js
    - merge函数合并配置
    - webpack --config选定配置文件
- 基本配置项及其配置方式，比如 entry 的多种配置方式及其效果
  - entry
    - 四种形式
      - 对象
        - 配置项：import、dependOn、runtime、filename
      - 数组
      - 函数：make阶段调用
      - 字符串
  - output
    - 输出hash和缓存
  - module
    - rules配置loader
  - resolve
  - plugins
  - devServer
  - 其他：optimitize、target、cache

优化：

- 开发环境
    - 构建速度
      - 减小解析范围
        - loader
        - resolve
        - 极端：noParse/externals
      - 多进程
        - thread-loader
        - 压缩过程的并行
      - 关闭某些生产环境的优化
        - splitChunk
        - minisize
        - useExports
      - cache
- 生产环境
    - 构建产物
      - 压缩代码
        - terserwebpackplugin-压缩js
        - MiniCssExtractPlugin-分离并压缩css
      - tree-shaking
        - 效果和目的
        - 开启方式
        - 基本原理
        - 注意事项
          - 减少导入赋值
          - 使用es6的库
          - 禁止babel编译import
      - 代码分割
        - 三种方式
          - 多入口手动分离
          - 动态导入
          - splitChunk
            - `'all'`配置在所有chunk之间共享模块
            - maxSize和minSize
            - cacheGroup
- 其他
  - http缓存
    - contenthash
    - 利用splitChunk对不经常变动的模块分包
      - runtime
      - 第三方库单独打包

原理：

- 一些核心概念，比如 module、chunk、compiler、compilation、hook、dependencies、chunkGraph、moduleGraph
- 基本工作流程，三步的具体工作内容
  - 初始化
    - 收集参数
    - 创建compiler对象
    - 启动编译流程
  - 构建
    - 从入口进入
    - 调用loader翻译模块，生成模块ast
    - 解析导入导出，递归生成module
  - 生成
    - 创建chunkgraph
    - 从入口开始递归生成各个chunk
    - 生成代码，输出文件
- hook 机制，hook 是什么，怎么用 hook，webpack 是怎么用 hook 的，有什么用，我们能用到什么
- loader：原理，以及自己编写的 loader 的原理，使用了哪些 api 和上下文
- plugin：原理，自己编写 plugin 的原理，使用了哪些 hook 和 api
- hmr
  - 概念，用处
  - 使用：hot，module.hot.accept
  - fast refresh 两者区别
    - 使用上的区别
    - 原理上的区别
  - 原理
    - 注入runtime
    - 建立 浏览器-devserver ws连接
    - 文件变化 -> hash事件 -> 浏览器请求manifest
    - devserver返回manifest -> 浏览器请求对应的更新文件 -> express返回该文件
    - 执行module.hot.accept更新

使用经验：

- 如何配置 react 脚手架
- webpack.config.ts
- babel 配置，解析 ts 和 react
- 引入 hmr 和 fastrefresh
- 配置 css module

# 工程化

- package.json 和 package-lock.json 是什么，有哪些属性
  - 锁定版本号
  - 确定依赖树
- semver版本的机制
- node 查找模块的方式、顺序
- **npm install 发生了什么** 
- npm run 发生了什么，为什么能直接执行
  - 软链接
  - npx
- **Babel 是什么**，核心配置，和 webpack 结合
- ESLint 怎么配置

# 优化

- 优化方式
  - **网络优化**
    - 网络请求优化
      - http优化
        - 1.1版本优化
          - 减少资源数量
          - 减少请求数量
          - 充分利用缓存
          - 减小请求大小
        - 2.0版本优化
          - 减小请求大小
          - 增大请求数量
      - 使用http2.0/3.0
      - dns预解析
      - 预连接
      - cdn
      - http报文体压缩
      - js请求的防抖节流
    - http缓存优化
      - webpack配置缓存的方式
      - 服务端配置合理的缓存
  - **静态资源优化**
    - 预加载
      - 手动添加preload、prefetch库
      - webpack动态导入
      - js预加载
      - react-loadable/React.lazy
    - 懒加载
      - 判断进入可视区域
      - 列表懒加载
    - 图片
      - 小图合并
      - 图片压缩
      - 采用webp格式
      - 媒体查询、响应式图片采用低分辨率
  - **运行时优化（代码优化）**
    - js
      - 减少dom操作、集中dom操作
      - script标签位置
      - 代码质量优化
    - css
      - 不使用@import
      - css代码性能
        - 减少缩写
        - 少用浮动、绝对定位
        - 减少重绘重排
      - css选择器性能
        - 少用低性能选择器
        - 少用通配符
    - 减少重绘重排
      - 不用table布局，用弹性布局
      - 改变class而不是style
      - css动画
      - gpu加速
      - 减少影响文档流
      - 操作dom集中
    - webpack优化
    - react优化
- 性能指标
  - RAIL性能模型
  - web用户指标
    - FP
    - FCP
    - FMP
    - **LCP**
    - TTI
    - **FID**
    - **CLS**
  - 检测方式
    - PerformanceEntry
    - web-vitals库
- 性能检测工具
  - lighthouse
  - chrome devtools


# React（原理）

- jsx
  - jsx 文件为什么要引入 react，为什么现在不需要
  - React element 怎么产生，有哪些种
  - jsx、fiber、ReactElement、真实 dom 有什么关系
- 组件
  - 类组件和函数组件有什么区别
  - 类组件生命周期：都有哪些，什么作用，什么时候执行，为什么有几个不能用
- 事件系统
  - React 事件系统为什么是独立的
  - React 事件合成是什么，事件插件是什么，有什么作用
  - React 事件回调是怎么绑定到真实 dom 上的
  - 从触发 dom 事件到执行回调流程
- fiber
  - 为什么引入 Reactfiber，fiber 是什么？
  - fiber 的结构，保存哪些数据？
  - fiber 原理，怎么构建，怎么更新
- 调度
  - React 怎么解决卡顿问题（时间分片、异步调度）
  - 调度流程？怎么开启一轮调度？
  - ensureRootIsScheduled 函数内部的实现，怎么实现的批量更新
  - scheduler 的原理，taskQueue 和 timerQueue，怎么实现任务执行中断并放到下个时间片？
- 调和
  - 调和是什么？调和的全过程（哪几步，每步什么作用，怎么执行，结果如何）？
  - render 阶段做了什么？beginWork 阶段做了什么
  - 组件更新任务的调度为什么统一发生在 RootFiber 上，怎么确定具体在哪个组件更新呢？
  - Reconciler 是什么，实现？
  - React diff 算法，单节点、多节点流程
  - completeWork 阶段做了什么
  - commit 阶段做了什么，其中的三步每一步又做了什么
- state 机制
  - React 批量更新是什么，React18 为什么不一样
  - 为什么 setState 是异步的
  - 对函数组件：当调用 setState 时发生了什么，为什么可以导致更新
  - useState 怎么消费 update
  - update 是什么？updateQueue 是什么？
  - 更新优先级是什么，怎么实现不同更新优先级，低优先级更新会怎么处理？如果优先级不同，怎么保证最终数据一致
- hook
  - hooks 为什么不能在条件语句中调用
  - hook 和 fiber 的关系？hook 在哪里保存数据，hook 的基本结构？
  - hook 怎么初始化，怎么处理更新，做了哪些事
  - useState 原理，内部逻辑？useState 是怎么消费一个 update，而这个 update 是怎么从 dispach 中产生的？useState 怎么处理更新优先级问题，低优先级的更新被怎么处理
  - useEffect 简单原理，以及 useMemo、useRef 等其他类似 hook
- Concurrent
  - React Concurrent 是什么，有哪些改变，在哪里体现？
  - 为什么要采取 createRoot 而不是直接 render？createRoot 做了什么，流程是什么？

# React（使用）

- React 优化
  - 核心思想：降低渲染成本，或者尽量避免渲染
- React 高阶组件是什么，怎么用
  - 功能
    - 强化props
    - 渲染拦截
    - 外层包装
- React18 带来了哪些更新？
- React 错误边界
- React-Router 简单原理，基本实现原理，两种路由方式的区别，以及基本使用
- React 状态管理库：
  - flux 理念
    - action -> dispatcher -> store -> view -> action -> dispatcher
  - Redux：基本理念、基本组成部分及其实现、常用的几个 api 和实现，怎么和 react 结合的，具体使用
    - 基本理念
      - 单向数据流
      - 数据不可变（state）
    - 基本结构
      - store
      - state
      - reducer
      - dispatch
      - action
      - subscriber
    - middleware机制
    - react-redux
      - react和redux连接的方式
        - provider添加订阅，状态改变触发listeners
        - useSelector获取最新state
        - useSyncExternalStore
        - connect
    - 现代使用
      - createSlice
  - Recoil：几个特点、atom 和 selector 两个核心概念、和 redux 的不同
    - 基本特点
      - 状态原子化
      - 有向图 数据流
      - 对Concurrentmode的支持
    - 和redux的差别
      - 状态更细分
      - 状态连接更随意
      - 轻量
      - 对Concurrent支持好
      - 使用层面：更简单，对ts支持更好
# OS

1. 进程的特点
2. 线程
3. 死锁
4. 僵尸进程和孤儿进程

# TS

1. type 和 interface 的区别
  - 写法区别
  - 性质区别，type本质是类型别名
  - 规范问题
  - 接口的extends、implement，接口和类的关系；type的`|`和`&`
  - 多次声明interface会合并
2. 声明文件，作用，写法，实际使用
3. 基本数据类型（ts 类型）
   - 基本类型（number、string 等）
   - 数组、对象类型
   - 接口的定义和使用、接口的继承、索引签名、接口和类
   - 函数类型声明、函数重载、this
   - 泛型
   - 高级类型：交叉、联合、索引
4. tsconfig.json：配置编译选项，存在的目录就是 TS 识别的根目录，指定了根目录下的 ts 文件编译选项

# git

1. git 的组成
   - 分区
   - HEAD 指针
   - 分支
2. 管理修改
   - 修改的跟踪和恢复
3. 回退操作
4. 分支操作
   - 创建分支
   - 合并分支和解决冲突
5. 远程仓库
   - push 和 pull 操作
   - PR

# 设计模式

1. 设计模式的三个基本类型、常见的设计模式
2. 各个设计模式的原理和实现：
   - 工厂模式
   - 单例模式
   - 装饰器
   - 原型
   - 适配器
   - 代理
   - 策略
   - 观察者和发布订阅
    - 两者区别

