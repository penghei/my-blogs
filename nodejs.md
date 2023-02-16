---
title: nodejs学习
date: 2021-12-30 20:28:12
tags: 日常学习
cover: /img/nodejs.png
categories: NodeJS
---

主要了解 nodejs 的相关原理、主要功能完成和 express 框架的使用；更具体的原理性知识可以参考<a href="https://www.yuque.com/sunluyong/node/what-is-node">这篇文章</a>；当前对 nodejs 的掌握主要在于基本原理和基础知识、框架使用，可以从路由接口到数据库简单操作以及静态文件服务器的上传下载等功能完成一个完整但不大的后端项目；

# nodejs 基本概念

## nodejs 结构

![](https://pic.imgdb.cn/item/61e93e652ab3f51d916f38cf.png)

- acron、node-inspect：npm 社区的包
- 用户代码：js 编写，v8 编译，具体业务代码
- nodejsCore：nodejs 的核心部分，包括 js 和 C++；js 代码部分，比如一些模块、中间件等；但是大部分代码都是 C++，主要是一些底层架构等
- N-API：js 不能满足的需求，通过一些模块完成
- V8：nodejs 经过改进的 v8 引擎，相对于浏览器更适合
- libuv: nodejs 实现异步 IO、eventLoop、syscall（系统调用，即封装操作系统 api）的主要模块, 含有 uv 线程池用于处理异步操作
- nghttp2: 和 http2 相关的一些模块
- zlib: 压缩和解压算法
- c-ares: DNS 查询库
- llhttp: http 协议解析
- OpenSSL: 网络层的加密解密工具

## nodejs 运行时结构

- 异步 io
  - 当 Node.js 执行 IO 操作时，会在响应返回后恢复操作，而不是阻塞线程并占用额外内存等待，减少了内存消耗
  - nodejs 通过 linuv 的 uv 线程池处理异步 io 操作
- 单线程
  - 指 js 主线程是单线程的，实际上 node 的进程中有 uv 线程池、js 主线程、v8 任务线程池等等
  - 不用考虑状态同步、同步锁等问题
  - 但是阻塞会产生更多负面影响，解决方法就是多进程或多线程
- 跨平台

## Event Loop和异步机制

Node.js 启动后会初始化事件轮询，过程中可能处理异步调用、定时器调度和 process.nextTick()，然后开始处理 event loop。官网中有这样一张图用来介绍 event loop 操作顺序

```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```

上面每个块内部都是一个任务队列，对应一类任务；
在一次 event loop 中，这些任务队列按照从上到下的顺序依次执行；
每个任务队列中的任务清空或达到上限后才会执行下一个任务队列中的任务。
当所有任务执行完成时，相当于走完了一圈，node 称作是一个`tick`。`process.nextTick()`的任务就是在进入下一个 tick 之前执行。

所有的同步任务都在任务队列开启一个 tick 之前执行，因此所有的异步代码、非阻塞 IO 都被放在了下一个 tick 中。
并且，在 nodejs 中，每个任务队列执行完成后都会执行一次微任务队列（包括第一次进入一个 tick 时，在 timer 之前）。这点和浏览器不同。

各个阶段主要任务

1. timers：执行 `setTimeout`、`setInterval` 回调
2. pending callbacks：执行 I/O（文件、网络等） 回调
3. idle, prepare：仅供系统内部调用
4. poll：获取新的 I/O 事件，执行相关回调，在适当条件下把阻塞 node
5. check：`setImmediate` 回调在此阶段执行
6. close callbacks：执行 socket 等的 close 事件回调
   日常开发中绝大部分异步任务都是在 timers、poll、check 阶段处理的

看这样一个例子：

```js
setImmediate(console.log, 1);
setTimeout(console.log, 1, 2);
Promise.resolve(3).then(console.log);
process.nextTick(console.log, 4);
console.log(5);

// 5 4 3 2 1
```

解释如下：

1. 同步代码先执行，打印 5
2. 开始执行异步代码，进入第一个 tick，进入之前先调用`process.nextTick`，打印 4
3. 进入 tick，先执行微任务`Promise.resolve()`打印 3
4. 进入 timer 队列，打印 2
5. 进入 check 队列，打印 1

## 异步、非阻塞I/O

I/O 即Input/Output, 输入和输出的意思。在浏览器端，只有一种 I/O，那就是利用 Ajax 发送网络请求，然后读取返回的内容，这属于网络I/O。nodejs 中主要分为两种:

- 文件 I/O。比如用 fs 模块对文件进行读写操作。
- 网络 I/O。比如 http 模块发起网络请求。

## 模块

- nodejs 使用 commonjs 的模块系统，用 require 和 export 导入导出
- 模块在第一次加载后会被缓存到 Module.\_cache ，如果每次调用 require('foo') 都解析到同一文件，则返回相同的对象
- 第三方模块查找过程会遵循就近原则逐层上溯（可以在程序中打印 module.paths 查看具体查找路径），直到根据 NODE_PATH 环境变量查找到文件系统根目录
  可以通过 babel 使用 import/export

## Buffer

Buffer 类的实例类似于 0 到 255 之间的整型数组，是一种处理二进制数据的格式；比如用 readFile 读取的图片等二进制文件就是 buffer 类型的
buffer 可以在字符串之间转换，可以通过字符串形式打印出来

## Stream

从术语上讲流是对输入输出设备的抽象，是一组有序的、有起点和终点的字节数据传输手段
使用 createReadStream 和 createWriteStream 可以分别创建读写流，可以对读写流对象进行操作
stream 内部进行了一些处理, 能更好利用空间, 加快速度

### 连接流

使用 pipe 可以连接两个流，注意必须是从一个 readable 流 pipe 到 writable 流
在流之间的 pipe 还可以用一些参数进行流的操作，比如压缩、转码等

```js
const fs = require("fs");
const rs = fs.createReadStream("./package.json");
const ws = fs.createWriteStream("./package-lower.json");
rs.pipe(lowercase).pipe(ws);
```

在大文件的传递中需要流的参与，比如发送一个电影文件：

```js
const http = require("http");
const fs = require("fs");
const oppressor = require(oppressor); //压缩

http
  .createServer((req, res) => {
    fs.createReadStream(moviePath).pipe(oppressor).pipe(res);
  })
  .listen(8080);
```

对流(一般是可读流)监听 data 事件可以看到传递的数据
同理还有 end 和 error 事件

```js
const rs = fs.createReadStream("./public/index.txt", { start: 2, end: 10 });
rs.on("data", (chunk) => {
  console.log(chunk.toString());
})
  .on("end", () => {
    console.log("end!");
  })
  .on("error", (err) => {
    console.error(err);
  });
```

对于可写流，在 data 事件监听中调用 write 方法就可以把数据写到流中

```js
rs.on("data", (chunk) => {
  ws.write(chunk, "utf8");
});
```

## 事件

nodejs 是以事件驱动的；事件的处理和绑定类似 addEventListener

- on 绑定事件
- once 可以只触发一次
- emit 可以依次按照注册的顺序触发监听器，参数是相同的事件名和传递给事件的参数
- off 可以解除绑定

```js
const event = new EventEmitter();
event.on("e", () => {});
event.on("e", () => {});
event.once("e", () => {});

event.emit("e", ...args);
event.off("e", () => {});
```

nodejs 也有异步事件；listener 可以使用 `setImmediate()` 和 `process.nextTick()` 方法切换到异步的操作模式
这两个方法参数都是一个 callback，表示把这个 callback 变成异步的形式

```js
myEmitter.on("event", (a, b) => {
  setImmediate(() => {
    console.log("setImmediate");
  });
  process.nextTick(() => {
    console.log("nextTick");
  });
});
```

# 基础操作

## 文件操作

### 基本 fs

Node.js 对文件、文件夹读写操作主要靠内置的 fs 模块

```js
const fs = require("fs");

fs.readFile("", (err, res) => {
  //...
});
```

可以像上面一样使用回调，也可以使用提供的 promise api
这里 stat 可以获取文件信息，在 stats 对象中保存

```js
const fs = require("fs").promises;
//或者
const fs = require("fs/promise");

fs.stat(".")
  .then((stats) => {})
  .catch((error) => {});
```

更好的方法是下面的 promisefy，只用把需要的部分 promisefy 即可

```js
const fs = require("fs");
const promisify = require("util").promisify;

const open = promisify(fs.open);

async function test() {
  const fd = await open("./test.txt");
}
```

当然可以用 promise，就也可以用 async/await；但是需要 trycatch 包裹，所以不一定会简单很多

```js
const fs = require("fs").promises;

async function readfiles() {
  let stats = await fs.stat(".");
  console.log(stats);
}
```

对于一些同步操作，比如需要读完文件再进行操作，可以用结尾为 sync 的 api，比如`fs.readFileSync()`

### 文件读取

使用 readFile 是最常用的读取文件方式

- 有三个参数
  - filepath 可以是相对路径或绝对路径，一般常用的操作是先定义一个静态资源根目录，然后根据 url 的请求拼接得到 path
  - utf8 是读取格式，这个参数还可以是一个对象，比如`{ encoding: 'utf8', flag: 'r' }`还规定了只读
-

```js
fs.readFile(filePath, "utf8", (err, res) => {
  if (err) {
    res.end("文件不存在！");
  } else {
    res.end(res);
  }
});
```

大文件一般用文件流读写；可以先创建一个流，然后对流进行事件监听

```js
const rs = fs.createReadStream("./public/index.txt", { start: 2, end: 10 });
rs.on("data", (chunk) => {
  console.log(chunk.toString());
});
```

对于 createReadStream 的第二个参数还可以选择：

- fd: 如果指定了 fd，则 ReadStream 会忽略 path 参数，使用指定的文件描述符（不会再次触发 open 事件）
- autoClose: 默认值: true，文件读取完成或者出现异常时是否自动关闭文件描述符
- start: 开始读取位置
- end: 结束读取位置
- highWaterMark: 默认值: 64 \* 1024，普通可读流一般是 16k

当然事件还有 open/ready/end/close/error 等多种

### 文件写入

文件写入主要的 api 是 fs.writeFile，但是注意这种写法会覆盖文件内容

```js
const fs = require("fs");

const data = Buffer.from("Hello, Node.js");
fs.writeFile("./test.txt", data, (err) => {
  if (err) throw err;
  console.log("文件已被保存");
});
```

如果想要追加在文件后可以使用 appendFile，参数和 writeFile 类似

### 其他 api

- fs.watch()可以监听文件或文件夹信息，watchFile 可以更详细监听文件信息（轮询方式），包括修改时间、修改前后的 stats 对象

```js
const fs = require("fs");

fs.watch("./", { recursive: true }, (eventType, filename) => {
  console.log(eventType, filename);
});

fs.watchFile("./test.txt", { interval: 100 }, (curr, prev) => {
  console.log("当前的最近修改时间是: " + curr.mtime);
  console.log("之前的最近修改时间是: " + prev.mtime);
});
```

- fs.existsSync 判断路径是否存在，返回布尔值

## HTTP 相关

### 基本服务器

使用 http 模块的 createServer 来启动一个服务器

```js
const http = require("http");

const server = http.createServer((req, res) => {
  res.write("Hello\n");
  res.end();
});

server.listen(8080, () => {
  console.log("Web Server started at port 9527");
});
```

- req 代表本次 http request，是一个可读流，常用有几个属性

  - url：本地请求的地址，如果没有地址默认是`/`，有地址就是`/xxx/xxx`的形式，可以和根目录使用 path.join 拼接
  - method：HTTP 请求的方法（GET、POST、DELETE、PUT 等）
  - headers：请求的 HTTP header

- res 代表本次 http response，是一个可写流，如果是前端请求就会通过 res.end 或者 res.send 返回数据
  - writeHead(statusCode, StatusMessage, headers)：发送响应首部，包含状态码、状态信息、响应头
  - write(chunk)：向响应主体中写入字符串或者 buffer
  - end(chunk)：向服务器发出信号，可以携带最后发送的数据，表明已发送所有响应头和主体，每个响应都需要调用一次
  - getHeader(name)：返回指定 name 的 header
  - getHeaders()：返回包含了所有 header 信息的对象
  - setHeader(name, value)：设置响应头，和 writeHead() 合并，有冲突时优先使用 writeHead()
  - statusCode：设置响应 HTTP status

req 和 res 都是流，因此可以进行一些流操作，比如可以通过流传输电影等大文件

```js
res.writeHead(200, {
  "content-type": mime.contentType(path.extname(url)), //mime可以用于获取文件content类型
});
fs.createReadStream(filePath).pipe(res); //通过流读取文件并pipe回去
```

其他基本操作可以参考node官网上的一个案例：https://nodejs.org/zh-cn/docs/guides/anatomy-of-an-http-transaction/

## 框架

这里框架只用到 express，后续学习到更多框架如 nextjs/eggjs 等再来补充
相对于 express，koa 显得不是很好用，因此还是以 express 为主开发就够用了

### express

通过引入 express 并 express()的方法得到 app，对 app 实现操作

```js
const express = require("express");
const app = express();
app.listen(8080);
```

express 的核心是路由，即 get 和 post 等方法的第一个参数；请求通过路由匹配不同的处理函数

#### 中间件

express 使用中间件的方式是 app.use；在执行 nodejs 文件时会执行所有的中间件后再启动服务器；
中间件可以是一个有 req、res、next 参数的函数

- req 和 res 是请求的
- next 表示继续，必须再尾部调用否则会终止在当前中间件

```js
function getReqInfo(req, res, next) {
  const time = new Date();
  console.log(`${time.toLocaleString()},${req.url},${req.method}`);
  next();
}
app.use(getReqInfo);
```

中间件适合执行一些独立工作，比如 http 压缩、缓存等

#### express 基本用法

##### 基本请求

```js
app.get("/hello", (req, res) => {
  console.log(req);
  res.send("hello world");
  res.sendFile(`${__dirname}/public/home.html`);
});
app.post("/userinfo", (req, res) => {
  fs.appendFile("./public/userinfo.json", JSON.stringify(req.body), (err) => {
    if (err) console.log(err);
  });
  res.sendFile(`${__dirname}/public/home.html`);
});
```

get 和 post 基本方法对应请求 get 和 post
express 的 req 和 res 对象有很多封装好的属性和方法：

Req：

- req.body / req.cookies：获得「请求主体」/ Cookies
- req.hostname / req.ip：获取主机名和 IP 地址
- req.originalUrl：获取原始请求 URL
- req.params：获取路由的 parameters
- req.path：获取请求路径
- req.query：获取 URL 的查询参数串，返回一个解析后的对象，可以直接解构
- req.route：获取当前匹配的路由
- req.get()：获取指定的 HTTP 请求头
- req.is()：判断请求头 Content-Type 的 MIME 类型

Res：

- res.append()：追加指定 HTTP 头
- res.set()在 res.append()后将重置之前设置的头
- res.cookie(name，value [，option])：设置 Cookie
- res.clearCookie()：清除 Cookie
- res.get()：返回指定的 HTTP 头
- res.json()：传送 JSON 响应
- res.redirect()：设置响应的 Location HTTP 头，并且设置状态码 302
- res.send()：传送 HTTP 响应
- res.sendFile(path [，options] [，fn])：传送指定路径的文件 -会自动根据文件 extension 设定 Content-Type
- res.set()：设置 HTTP 头，传入 object 可以一次设置多个头
- res.status()：设置 HTTP 状态码
- res.type()：设置 Content-Type 的 MIME 类型

##### 上传文件

上传文件或者表单上传表单数据需要 multer；用 multer 确定临时文件存放位置和上传类型
注意点：

- multer 规定的 dest 路径格式应该是`xxx/xxx/xxx/`形式的，注意前后/的位置和有无
- post 的参数也可以有 upload.single；因为 app.post 的第二个参数就是在当前 post 请求中使用的中间件
- 创建 upload 对象之后使用`.single("xxx")`表示单文件上传，括号中是 html 中 input 的 name 属性
- 文件上传默认是一个特殊的 buffer 类型的文件，需要进行转换；可以用 readFile 读取并拿到文件 buffer 对象，然后用 writeFile 写入另一个路径；这里核心在于改变了文件名为`req.file.originalname`，即源文件名

```js
const express = require("express");
const multer = require("multer");
const fs = require("fs");

const app = express();
const upload = multer({
  //用multer中间件规定临时文件存放位置
  dest: "public/temp/",
});

app.use(upload.single("pic"));
app.use(express.static("public")); //静态文件采用的是约定式，确定根目录之后可在url中根据路径访问;

app.post("/upload", upload.single("pic"), (req, res, next) => {
  //上传文件
  console.log(req.file);
  let des_file = `${__dirname}/public/upload/${req.file.originalname}`; //确定完整文件存放位置，这里文件名被改成原文件名以及后缀
  fs.readFile(req.file.path, (err, data) => {
    //读取文件，得到data为文件的buffer对象
    fs.writeFile(des_file, data, (err) => {
      //把文件写入另外文件夹
      if (err) {
        console.log(err);
        return;
      }
      res.end("success");
    });
  });
});

app.listen(8080);
```

还可以上传多个文件，把 upload.single 改为`upload.array('photos', num)`，其中 num 为限制数量；注意多文件是 req.files 文件数组

##### 静态资源

express 只需要设定好`express.static("xxx")`即可选定文件夹为根目录，直接通过路由请求就可以获得静态文件，是一种约定式的配置；
选定了静态文件目录后会自动把 index.html 设为默认首页，也就是不通过任何路由访问时就会打开此文件作为首页；
如果客户端需要跳转页面可以通过 sendFile 发送 html 文件；相当于 res.send 了一个 html 文件

```js
app.use(express.static("public"));
app.get("/submit", (req, res) => {
  let { username, password } = req.query;
  res.sendFile(`${__dirname}/public/home.html`);
});
```

## Debug

通过 v8 inspector 调试
启动命令：

```
node --inspect index.js
```

然后可以打开一个浏览器 devtools

- 在 source 中通过命令打开文件就可以进行断点调试等类似 js 的调试方法
  - 断点，类似 html 中 js 的断点调试
  - logpoint：很神奇的功能，在某一行添加 logpoint 相当于加入 console.log，可以方便的进行查看变量。写法就是 console.log()括号内部的内容
- memory：内存，可以通过 snapshot 查看各种对象内存栈
- profiles：录制一段时间内运行情况，可以看到一些函数调用时间、函数名等等内容

# ts 写 node

ts 写 node 用到了 ts-node 这个包

## 安装配置

安装:

```
yarn add ts-node -D
yarn add typescript -D
```

注意**ts-node 不能全局安装**, 否则会报错
然后产生 tsconfig.json 文件

```
tsc --init
```

需要改动的部分:

- `"module": "es2015",`
- 如果出现 include 报错, 就在`compilerOptions`之外加上`"include":["src/**/*"]`,或者不写`src/`也可以, 然后在范围内创建一个 ts 文件
- package.json 文件中加一行`"type":"module"`.**注意是 module 不是 modules!!!**不要被网上垃圾回答误导了

然后新建 server.ts 文件, 注意这时就不再使用 Commonjs, 而是使用 import/export 来引入各种包和文件
启动:

```
node --loader ts-node/esm src/index.ts
```

注意启动时文件路径是根目录下的相对路径

## 相关报错和坑

```
SyntaxError: Cannot use import statement outside a module
```

无法在模块外使用 import，解决这个问题需要在 package.json 文件中添加"type":"module"。

```
Error [ERR_MODULE_NOT_FOUND]: Cannot find module 'C:Users1Desktopmy-projectdata' imported from 'C:Users1Desktopget-data.ts'
```

找不到导入的模块，要在导入文件后加上后缀名, 但是 ts 文件不允许后缀为 ts, 因此要引入其他非模块文件(比如中间件文件)可以把后缀加上`.js`
