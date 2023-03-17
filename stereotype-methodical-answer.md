本文旨在梳理八股问题有条理的回答。回答不一定详细，以清晰的分点为主
对于展开比较丰富的题目，主要是分点，梳理回答的方式
对于答案比较直接的题目，主要参考原文，可能不会写很多的分点

# HTML

- link 标签的作用和常见属性
  - 概括：链接外部资源
  - href：路径
  - rel：类型
    - stylesheet：样式
    - icon
    - 各种 pre（见下）
  - media：媒体查询
- **link 标签的各种预请求**
  - preload & prefetch
    - 二者作用和特点
      - preload
        - 不阻塞
        - 优先级高
        - 本页面使用
        - 各种资源，包括 js 和 css 内的资源，不仅限于 html 内
      - prefetch
        - 低优先级
        - 强制缓存
        - 下个页面使用
  - dns-prefetch
    - 预加载可能的 dns 解析
    - 和 pre-connect 一起用
  - pre-connect
    - 和目标建立连接，包括 http、tcp、TLS
- meta 标签的作用和常见属性
  - 概括：文档元数据，帮助浏览器解析
  - 添加关键字、描述等帮助 seo 的字段
  - http-equiv：设置 http 头，csp
- **viewport**
  - 基本概念
    - 移动端的窗口
    - 响应式布局的基础
    - meta 标签控制
  - 三种 viewport
    - 布局视口：浏览器默认视口，布局的基础
    - 视觉视口：可视区域大小
    - 理想视口：屏幕尺寸
    - 响应式基础：布局视口 = 理想视口
- script 标签的 async 和 defer 属性效果和作用
  - async：异步下载同步执行，多个 script 不保证顺序
  - defer：异步下载异步执行，多个 script 执行顺序不变
- html 语义化的含义和意义
  - 含义：有名字的元素
  - 意义：
    - 对开发者：可读性
    - 对机器：seo、爬虫、读屏

# CSS

- 什么是 css 的层叠和继承，哪些属性可以继承，哪些不能？
  - 层叠：后覆盖前、高优先覆盖低优先
  - 继承：部分属性可以继承其父元素的属性
    - 可继承：font、text、color、visiblity
    - 不可继承：display、position
    - 控制继承：inherit、initial、unset
- **css 选择器有哪些，优先级是怎么样的，性能如何，选择器匹配顺序**
  - 概述：id > class = 伪类 = 属性 > 元素 = 伪元素 = 联合
    - 行内 > 内部 > 外部 > 浏览器用户自定义 > 浏览器默认
  - 优先级机制
    - 如上
    - 跨级不可超越
    - 越叠加越高
  - 效率
    - 概述：具体效率排名
    - 主要依据：最后一个选择器
  - 匹配顺序：倒序
    - 原因：减少匹配次数
- 伪类和伪元素有什么区别，常见的有哪些
  - 概念
  - 举例：
    - 伪类：
      - 爱恨原则
      - （-child 和-of-type 的区别）
    - 伪元素：after/before
- css 的单位都有哪些
  - 举例
    - 大小单位
      - 绝对单位 px pt
        - **逻辑像素和物理像素**
          - 概念：
            - 逻辑像素和物理像素是什么
            - 两者性质（特点）
          - 逻辑像素和物理像素的对应关系（放大缩小）
          - dpr 是什么
      - 相对单位
        - em：用于字体/用于元素
        - 百分比：用于字体/用于元素
    - 颜色单位：rgb rgba hsl hsla
- 常见的块级元素、行内块、行内元素都有哪些，什么特点？
  - 概括：
    - 块级元素有哪些
    - 行内元素有哪些
    - 行内块有哪些
  - 特点：
    - 块：宽度最大，单独一行
    - 行内：收缩宽度，纵向 padding 无效，合并一行
- 盒模型是什么
  - 概括：概念
  - 包含内容：content、padding……
  - 类型
    - 分类依据
    - 两种类型
- 外边距折叠是什么情况，怎么处理
  - 概括
  - 三种情况，每种的：
    - 出现情况
    - 解决方式
- **BFC 是什么**，有什么用，怎么触发
  - FC 是什么
    - 概念：一个模型，将元素转化为一个个盒子，盒子内和盒子间的布局
    - 分类：BFC IFC FFC GFC
  - BFC 是什么
    - 概念：块级盒子的布局形式
    - 组成：
      - 每个块级盒子都会参与构建
      - 部分块级盒子会直接生成
    - 触发：一些属性
    - 性质：一些性质
  - IFC 是什么
    - 概念：行内盒子的布局形式，更多的是行内盒子之间的
    - 相对于 BFC 的特点：一个盒子（line-box）包裹行内元素
    - 创建方式
    - 性质
      - IFC 内部的布局规则
      - line-height
      - padding 和 margin 仅水平
      - vertical-align 和 text-align
  - 其他 FC
    - FFC：类似 BFC，但部分属性无效
    - GFC：外部表现类似 BFC，内部分格
- 浮动是什么，有什么用，怎么清除
  - 概念
  - 浮动后元素的性质
    - 自己的性质
    - 和其他正常元素的关系
  - 用处
  - 清除浮动
    - 目的：防止浮动元素的影响
    - 方式
- position/display 有哪些属性，有什么特点
- z-index 顺序：
  - 背景 负 z-index 块 浮动 行内 z-index0 正 z-index
- 隐藏元素的方法有哪些，各自什么特点
  - 列举：
    - display
    - visibility
    - opcity
    - z-index
    - transform scale
  - 每个特点
    - 是否可监听绑定事件
    - 是否继承
    - 是否占据原位置
- 怎么创建一个响应式图片
  - 基本 css 创建：max-width: 100%;height: auto;
  - html 创建：
    - srcset
      - 属性含义：图片+单位
        - 单位：w 和 x
    - size
      - 属性值：媒体查询
      - 和 srcset 配合
    - picture 和 source
      - 两者的性质，功能，类似于 video、audio
      - 配置方式
- **flex 怎么用，有哪些属性**
  - 基本概念
    - 弹性盒子
    - 主轴和交叉轴
  - 基本属性
    - 容器属性：wrap direction justify align
    - 元素属性 grow shrink basis
      - 基本含义
      - grow 的取值及其效果，几种情况
        - 元素全不定宽：按比例
        - 有元素定宽：划分空白
          - 定宽元素设置：额外增大
          - 不定宽元素设置：空白划分
        - 行内元素：生效
      - basis：
        - 含义
        - 优先级高
        - 0 和 auto
- grid
  - 基本概念
    - 行和列的划分
  - 使用
    - 划分行和列
    - 设定元素的起始和结尾行列位置
    - 设定元素在内部的位置
- 怎么判断元素是否进入可视区域？判断依据的属性都是什么意思？
  - 方法：
    - el.offsetTop - document.documentElement.scrollTop <= viewPortHeight
    - getBoundingClientRect
- css 怎么画一个三角形
  - border
- 垂直居中、水平居中、垂直水平居中，如果不知道父元素宽高的话怎么做？如果不知道子元素宽高怎么做？都不知道怎么做？
  - 水平居中：
    - flex 和 grid
    - margin -- 子元素宽度
    - text-align -- 行内块
    - position + transform
  - 垂直居中：
    - flex 和 grid
    - position + transform
    - position + margin auto
    - line-height + vertical-align
- 两栏布局怎么实现
  - flex
  - grid
  - float + margin-left
  - 绝对定位 + margin-left
- 三栏布局怎么实现
- 实现一个高度始终为宽度一半的矩形
  - aspect-ratio
  - padding-bottom
- 什么是**响应式布局**，怎么实现？
  - 概念：一套布局，动态变化
  - 不同方面
    - css 布局属性
      - rem 布局
      - flex 和 grid
    - css 元素属性
      - 百分比、vwvh 等单位设置宽高
      - 响应式图片
    - 媒体查询
      - 确定断点（bootstrap）
      - 移动优先/pc 优先
    - viewport
      - 保持 viewport 标签的设置，关键是 width，这个 width 决定了 vw、媒体查询、百分比等单位的基础大小。
- css 变量
  - :root 设置，`var()`使用，可用在 calc 中，元素内可覆盖
  - 相对于 less，更方便在 js 中处理
- 暗黑模式

# JS

- 数据类型有哪些，分为几种
- 引用数据类型和基本数据类型有什么区别？
  - 分别有哪些类型
  - 存储位置：栈和堆
  - 引用方式：直接和索引
  - 修改情况
- `==`的转换规则
  - 原始数据之间
    - 类型相同直接比
    - null 和 undefined 相等，和其他的都不转化
    - 其他跨类型：
      - boolean 和任意其他原始类型：boolean->number
      - number 和 string：string->number
  - 对象转化为原始
    - `Symbol[toPrimitive]`函数的返回值
    - valueOf 和 toString，取决于要转换的类型
  - 对象之间
    - 比较索引
- **检测数据类型的方法**，检测数组有几种方法，函数呢？
  - typeof 数组不行 null 有错
  - instanceOf 不能用于原始类型
  - constructor 属性 如果改变对象原型就会出错
  - toString
- 对象包装器
  - 概念：原始数据类型
  - 特点
    - 提供原始数据类型的方法
    - 但并不是真正的构造函数
    - 可以自定义添加 prototype 上的值
- Symbol 是什么，有哪些用处
  - 概念：原始数据之一，创建方式
  - 特点
    - 不会被大多数方法遍历，但可以被完整复制
    - 任何两个 symbol 都不相等
  - 应用：
    - 隐藏属性，比如 react 的$$typeof
    - 部分特殊内置方法，比如迭代器
- 迭代器是什么，如何创建，怎么使用
- 怎么把一个类数组对象转为数组
- 怎么创建一个指定大小的空数组
  - new Array,Array.from
- Map 和 Set 是什么，有什么特点
  - 概念
  - 形式
  - 特点
    - 哈希表特点 O(1)读写查
    - 不重复
- 对象怎么配置一个属性（比如可读、可写）
- 箭头函数有哪些限制
- **闭包是什么，怎么产生的，有什么用，利弊**
  - 概念：闭包的定义
  - 出现场景
    - 嵌套函数
    - 回调函数
    - 立即调用函数
  - 使用场景
    - 保存和保护
  - 影响：内存泄漏
    - 内存泄漏定义
    - 内存泄漏出现场景
      - 全局变量
      - 定时器引用的对象
      - 引用但销毁的 dom 元素
      - 闭包
    - 解决方案
      - 减少闭包
      - 清除闭包函数
      - 及时清除监听器
  - 产生原理
    - 执行上下文
- **作用域和作用域链是什么**
  - 作用域：
    - 概念：变量的可见情况
    - 产生：定义位置
    - 性质：静态
  - 作用域链
    - 概念：作用域的链
    - 产生：作用域相互连接
    - 作用：变量查找的依据
  - 和执行上下文的关系：
    - 区别：
      - 静态和动态
      - 产生时机
    - 联系：
      - 执行过程的变量查找符合作用域
- **执行上下文是什么，和作用域的区别，有哪些内容**
  - 概念：执行一段代码时的运行环境
  - 来源：全局和函数
  - 产生和应用时机
  - 结构：
    - 变量环境
      - 编译阶段将变量提升的变量声明
      - 执行阶段对变量赋值
    - 词法环境
      - 结构：栈
      - 存放块级作用域变量，每个块是栈的一项
  - 功能
    - 执行的变量查找依据：
      - 词法环境从上到下->变量环境->外部执行上下文
    - 作为函数的执行环境，影响 this
    - 提供闭包
- 块级作用域
  - 概念：
  - 产生：const 和 let
  - 性质：以块为边界
  - 相对的：变量提升作用域
    - 产生：var 和 function
      - 变量提升
        - 概念：编译阶段会先声明执行上下文内的变量和函数，执行阶段赋值
        - 效果：undefined
        - 影响：变量销毁和回收、提前使用的 undefined
    - 坏处：
  - 解决的问题
    - 及时销毁
    - 块内部生效
    - 作用域死区
- **this 是什么，有什么用，怎么指定 this 的指向？**
  - 概念：this 是函数执行环境的引用
  - 性质
    - 动态
    - 指代某个环境
  - 用处：
    - （个人理解）访问不属于自己上下文内，并且制定其他上下文中的变量
    - 构造函数
  - 应用场景
    - 对象属性
    - 构造函数和类
  - 取值
    - 全局使用
    - 普通函数
    - 构造函数
    - 对象方法
  - 绑定 this
    - 方法
      - new
      - callbindapply
      - 对象
      - 默认
    - 优先级：如上顺序
- new 的执行过程？
-
- **原型链是什么**
  - 原型：
    - 定义：两个方面
      - 所有对象：`__proto__`
      - 所有函数：prototype
    - 作用：属性和方法的链式访问，模拟面向对象的继承
  - 原型链
    - 定义：原型组成的链
    - 举例
- js 的继承方式
- **Promise 的功能，解决的问题，怎么实现的，怎么使用，特点，有哪些 api**
  - 概念：
  - 解决问题
    - 回调地狱
  - 解决方式
    - 延迟绑定
    - 返回值穿透
    - 错误冒泡
  - 性质
    - 三种状态
    - exector
    - resolve 和 reject
  - 使用
  - api
- async/await 的原理（协程），使用
  - 基本使用
  - 协程
- 模块化的发展历程
  - 全局函数
  - IIFE
  - namespace
  - AMD UMD
  - cjs
  - CMD
  - esm
- **commonjs 和 es6 module 的不同**
  - 不同
    - 静态和动态
    - 浅拷贝和引用
  - 相同
- Proxy 和 Reflect 是啥
- 事件循环，怎么实现的异步任务，宏任务和微任务？
  - 概念：
    - 事件循环是以浏览器为宿主环境实现的事件调度
    - 相关结构
      - 调用栈
      - 任务队列
      - webapi
  - 为什么需要
    - 异步任务非阻塞
    - 循环执行任务
  - 宏任务和微任务
    - 分类，如何产生的
    - 性质
    - 关系
- js 的内存管理、垃圾回收方式
- dom 树概述，dom 节点的概念（七种节点）
- dom 事件流？
- 事件对象是什么，怎么产生的，有什么用，自定义事件是什么。浏览器的事件模型
- 浏览器默认事件有哪些，怎么阻止
- 怎么封装一个 ajax（xhr 的用法）
- 怎么使用 fetch
- 防抖和节流怎么实现
- webcomponents 是啥
- shadowdom 是啥
- 一个 HTML 页面的生命周期
- ArrayBuffer 是什么，有什么用
- Blob 和 File 是什么，有什么用
- Generator 相关
- H5 history api

# 网络

URL 和 URI 是什么
同源策略，跨域解决方案
    什么是跨域        
    什么是同源策略
        概念
        同源条件
        不同源的限制
    为什么有跨域限制
        安全考虑
    怎么解决跨域
        CORS
            概念
            请求分类
                简单请求
                    定义
                    处理方式：
                        origin
                        access-allow
                复杂请求
                    定义
                    处理方式
                        预检
                        access-allow
        JSONP
            原理：script跨域请求
            实现
        代理
            原理
            实现：http-proxy-middleware
                devServer
OSI7 层网络模型、TCP/IP4 层模型，5 层模型
session、token、cookie 是什么，有什么特点和区别
    cookie
        概念：一小段数据
        作用：获取用户身份信息
        属性
            httponly
            samesite
                跨站的概念
                三个取值及效果
                    lax
                    strict
                    none
                举例
                    第三方cookie
                    csrf攻击
            domain
            secure
            expires
        跨域问题
            cors控制是否发送cookie
            xhr和fetch也可以控制
        js读写cookie
    session
        概念
        和cookie的区别
    token
        概念
        jwt
            本质：base64格式的json对象
            优缺点
            使用方式
HTTP 的基本特点
    无状态
    一收一发
    基于tcp
    数据类型丰富
HTTP 的请求和响应报文结构
HTTP 的常见字段
    四类字段
        请求
        响应
        通用
        实体
HTTP 的方法有哪些
    get head post put delete options
**HTTP 状态码**
    100 101
    200
    301 302 304
    400 401 403 404 405 408 413 414 429
    500 502 504 503
HTTPS 是什么，怎么保证安全，**握手过程**？
    https概念
        增加tls层
        增加tls握手
        数字证书
    https安全依据
        非对称和对称结合的加密算法
        数字证书
    https握手
        步骤
            client hello
            server hello
            发送证书
            取出公钥并生成密钥发给服务端
            解析得到密钥
            server done
            client done
**HTTP1.0 1.1 2.0 3.0 都有哪些变化**
    1.0
        基本头
        基本缓存
        状态码
        问题：短连接
    1.1
        长连接
        并发tcp连接
        更多缓存头
        新增请求方法
        问题：队头阻塞
    2.0
        二进制协议
        帧和流
        多路复用
        请求优先级
        服务端推送
        头部压缩
        一个tcp连接
        问题：依赖tcp，tcp的队头阻塞
    3.0 
        解决http2的问题：tcp队头阻塞
        基于udp连接
        quic协议
        连接更快
        连接迁移不中断
        独立数据流
HTTP 缓存，有哪些字段，缓存存放在本地的位置
    概念：根据一些头信息保存指定资源
    分类
        强缓存
        协商缓存
    控制字段
        Cache-Control
            max-age
            no-cache
            public、private
        Last Modified & If-Modified-Since
        Etag & If-None-Match
DNS 是啥，怎么查询的
CDN 是啥，有什么用
UDP 协议的特点、应用、报文结构，和 TCP 的区别
    特点
        无连接
        一对多、多对一、多对多
        不可靠
        高效
    应用
        时效性强的连接
    报文结构
        源端口、目标端口
        报文长度
        检验和
        报文体
TCP 的特点
    面向连接
    一对一传输
    字节流
    可靠传输
    拥塞控制
TCP 的报文段结构
    seq和ack
    接收窗口
    标志字段
        FIN ACK SYN
    报文体
**TCP 的三次握手四次挥手**
    三次握手
        全过程
        为什么三次，不是两次
    四次挥手
        全过程
        为什么四次
        为什么等待2个MSL
TCP 的可靠数据传输实现
    重传机制
        超时重传
        快速重传
            为什么是3个冗余ack
TCP 的流量控制实现
    滑动窗口
        发送方：四个部分
        接收方：三个部分
    流量控制的流程
        基本流程：收发
        收缩的流程：接收窗口变小，通知发送窗口变小
TCP 的拥塞控制
    概念：
        处理丢包、超时等行为
        和流量控制的异同
            同：都是控制发送速度，cwnd和rwnd
            异：主要控制重传
    单位：MSS/RTT ，即一次收发时间内发送多少个最大报文段长度
    处理方式
        慢启动
        超时->拥塞避免
        冗余ack->快速恢复
IP 协议中的子网掩码和前缀是什么
特殊的 IP 地址
链路层有什么作用
**从浏览器输入 url 到显示页面的过程**
    url解析
    检查缓存
    dns解析
    获取mac地址
    从上到下封装报文
    发送到服务器
    建立tcp连接
    https握手
    发送httpget请求
    响应html
    浏览器网络进程交给渲染进程
    解析html
    构建dom树、cssom
    构建布局树
    layer
    paint
    栅格化
    合成
    显示
CSP 是什么，怎么开启
XSS 攻击和防范
    概念
    基本原理：执行脚本窃取数据
    三种类型
        反射型
        dom型
        存储型
    防范
        httponly
        不信任用户输入
        不使用拼接字符串形成html
CSRF 攻击和防范
    概念
    基本原理：模拟用户发起请求

# 浏览器

浏览器中的主要进程和线程有哪些
    进程：
        渲染进程
            gui渲染线程
            js线程
            定时器
            异步任务
            事件处理
        网络进程
        主进程
        插件进程
        gpu进程
    新式架构
        多进程+服务，把ui、storage、file等放到各个服务内
**浏览器渲染页面的过程**
gpu加速的原理
**重绘重排，触发原因，优化**
解释和编译型语言
v8 怎么执行一段 js 代码的
    过程
        词法分析（分词）
        语法分析（创建AST）
        生成字节码
        生成机器码
        执行机器码
    工具
        解释器：ast->字节码
        编译器：字节码->机器码
        即时编译：部分热点代码提前编译

# react源码

函数组件和类组件的区别
  组件本质（函数和类）
  保存数据的方式
  生命周期
  React element的获取
类组件的生命周期
  挂载
    constructor -> getDerivedStateFromProps -> render -> componentDidMount
  更新
    gdsf -> shouleComponentUpdate -> render ->  getSnapshotBeforeUpdate -> commit -> componentDidUpdate
  废弃的声明周期，为什么？
React事件系统
  概念：
    React自己创建的
    为什么？磨平浏览器差异，配合虚拟dom，模拟事件
  三个部分
    事件合成
      概念：事件对象是自己合成的
      事件插件
        概念
        功能
          注册事件，保存当前类型事件的合成的dom事件列表
          当事件触发时：合成事件对象，创建listener放入执行队列
    事件绑定
      createRoot绑定所有事件
      绑定的listener实际上是不同事件优先级的dispatchEvent
    事件触发
      事件触发
        根dom捕获
        dispatchEvent
        通过事件插件合成事件对象和listener对象、dispatchQueue
      事件收集
        找到事件触发的那个fiber
        从下向上收集事件，获取每个fiber上的事件props，收集同名事件
      事件执行
        根据事件类型确定顺序
          如果是捕获就倒序
          如果是冒泡就正序
React fiber
  概念：虚拟dom
  引入原因
    方便控制
    减小阻塞，中断渲染
    双缓冲树 复用
  fiber结构
    属性
      React element相关属性
      fiber树的连接属性
      组件属性
      副作用相关
      优先级
      alternate
  双缓冲树
React 异步调度
  目的：解决阻塞
  方式
    异步调度
      通过messageChannel将任务放到空闲时间执行
    时间分片
      scheduler
      五个优先级
      taskQueue和timerQueue
      任务被拆散到每个时间片空闲执行
      使用messageChannel调度workLoop
      workLoop 中断重新注册任务到下个时间片
      中断条件
        isInputPending
        任务已经执行时间 >= 一帧剩余时间
    渲染中断
      render workLoopConcurrent
React 调和过程
  render
    beginwork
      功能
      Reconciler
      diff算法
        单节点
        多节点（数组类型）
          两轮遍历
    completeWork
      功能
  commit
    每个阶段及其功能
  flags机制
    标记时机
    subtreeFlags
    消费时机
    代替EffectList
React 更新流程
React hooks原理
  hook和fiber的关系
    fiber.memoizedState保存hook链表
  hook对象
    创建hook
      mountHook
      hook.memorizedState：核心存储位置
      hook.next：核心链表
    更新hook
      updateHook
      取hook.next复用
  useState
    update：更新对象
      updateQueue:update链表
      baseQueue：低优先级更新
    mountState：初始化updateQueue和dispatch
    updateState：执行updateQueue和baseQueue
  useEffect
    执行时机：before mutation异步启动
    effect链表
React 并发渲染
  createRoot（FiberRoot）
  scheduler
  可中断render
React 优先级机制
  事件优先级
    DiscreteEventPriority ContinuousEventPriority DefaultEventPriority IdleEventPriority
  lane优先级
    SyncLane InputContinuousLane DefaultLane IdleLane
  scheduler优先级  
    ImmediateSchedulerPriority UserBlockingSchedulerPriority NormalSchedulerPriority IdleSchedulerPriority
