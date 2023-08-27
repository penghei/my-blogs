---
title: 网络知识学习总结
date: 2021-12-02 12:19:49
tags: 日常学习
cover: /img/web.png
categories: 原理
sticky: 6
---

# 网络基础知识

## 协议间关系

![协议关系.jpg](https://i.loli.net/2021/12/02/Uwe2Sb3oAJZKGVq.jpg)

## 链接/超链接

> 网站的三大支柱：
>
> - URL, 跟踪 Web 文档的地址系统
> - HTTP, 一个传输协议，以便在给定 URL 时查找文档
> - HTML, 允许嵌入超链接的文档格式

链接可以将任何文本与 URL 相关联，因此用户只要激活链接就可以到达目标文档。
链接的类型有：

- 内链：网页内的链接，用于网页里面的变化，比如滚动条滚动
- 外链：网页到另一个网页的链接
- 传入链接：从其他人的网页链接到你的网页的链接。

## URL

> URI 和 URL
> ![](https://pic.imgdb.cn/item/623429625baa1a80abfeda20.jpg)
> 统一资源标志符 URI 就是在某一规则下能把一个资源独一无二地标识出来的**字符串**。URI 是唯一的，一个资源对应一个 URI，且只能通过该 URI 定位该资源
> URI 分为 URL 和 URN，这两个是 URI 的一种形式：即，URL 通过标识位置的形式表示资源，同样起到了 URI 的作用，所以 URL 是 URI 的子集，URL 就是用定位的方式实现的 URI。URN 则是用资源名标识，不过不常用。

URL 的组成：
![](https://pic.imgdb.cn/item/623426a85baa1a80abfcb591.jpg)

```
http://www.example.com:80/path/to/myfile.html?key1=value1&key2=value2#SomewhereInTheDocument
```

1. `http://` ，超文本传输协议，它表明了浏览器必须使用何种协议，也可是 HTTPS；当然还可以是 `file://` 文件协议，或者邮箱协议等
2. `www.example.com` 是域名，这个也可以用 ip 地址代替
3. `:80`是端口。IP 地址与网络服务的关系是一对多的，所以需要表示用于访问 Web 服务器上的资源的技术“门”。如果 Web 服务器使用 HTTP 协议的标准端口（HTTP 为 80，HTTPS 为 443）来授予其资源的访问权限，则通常会被忽略。否则是强制性的。一般 80 端口就是网络服务器，如果你自己想有个网站，就需要通过 80 端口来向外展示
4. `/path/to/myfile.html `是文件或资源位于服务器上的路径（服务器也是一台电脑），也被称为路由；当然路由时一种抽象的路径
5. `?key1=value1&key2=value2`传输的参数，这种一般是 query 或者 parmas 参数
6. `#SomewhereInTheDocument` 是资源本身的另一部分的锚点. 锚点表示资源中的一种“书签”，给浏览器显示位于该“加书签”位置的内容的方向。

上面给出的 URL 是**绝对 URL**，因访问服务器没有上下文，所以要用绝对路径访问

### URL 的编码

URI 只能使用`ASCII`，其他编码方式不支持。因此可以直接表示在 URL 上的被称为“元字符”，也就是`ASCII`码中的全部字符。
其他编码的字符需要被转码，方式是将所有非 `ASCII` 码字符和界定符转为十六进制字节值，然后在前面加个%
详见 https://www.ruanyifeng.com/blog/2010/02/url_encoding.html

## 域名

域名需要从右到左阅读。

```
www.google.com
```

1. `.com`是顶级域名, 除了.com 之外还有一些顶级域名都有要求

- 地区的顶级域名，如.us，.fr，或.cn，可以要求必须提供给定语言的服务器或者托管在指定国家。
- 包含.gov 的顶级域名只能被政府部门使用。
- .edu 只能为教育或研究机构使用。

> 注: 域名用`.`分割,从右向左分别为顶级,二级,三级以及更后,上不设限,所以比如`www.a.b.c.d.e.f.com`,这里的 abcdef 都是不同级别的域名,每一级的域名控制它下一级域名的分配。每个域名内部是标签,标签在下面解释;

2. `google`是标签, 标签由 1 到 63 个大小写不敏感的字符组成，这些字符包含字母 A-z，数字 0-9，还有`-`号.一般都是小写字母,但可以是大写,都是一样

3. `www`表示提供资源的主机的名称，或者叫做服务器名。www 虽然原本含义是万维网，但是通常大家习惯将他用作表示提供 web 服务的主机；如果有一个网站是`abc.www.com`，那 www 就成了一级域名。类似的服务器名还有 blog、mail 等

> 关于域名和主机名，大概有这样的区别：
>
> - 主机名 = 服务器名 + 域名。以`http://www.sina.com.cn/`为例，`sina.com.cn`是域名，`www`是提供服务的机器的名字（计算机名），计算机名+域名才是主机名，即`www.sina.com.cn`是主机名。
> - 域名只存在于公网，而主机名可以用于局域网。在局域网中每台主机用于互相区分，就有了主机名这个概念

## MIME

MIME(Multipurpose Internet Mail Extensions, 多用途互联网邮件扩展)。它首先用在电子邮件系统中，让邮件可以发任意类型的数据，这对于 HTTP 来说也是通用的。

因此，HTTP 从 MIME type 取了一部分来标记报文 body 部分的数据类型，这些类型体现在`Content-Type`这个字段; 接收端想要收到特定类型的数据，也可以用`Accept`字段。

具体而言，这两个字段的取值可以分为下面几类:

- text： `text/html`, `text/plain`, `text/css` 等
- image: `image/gif`, `image/jpeg`, `image/png` 等
- audio/video: `audio/mpeg`, `video/mp4` 等
- application: `application/json`, `application/javascript`, `application/pdf`, `application/octet-stream`

## 跨域

### 同源策略

**浏览器**遵循同源策略，非同源站点有这样一些限制:

- 不能读取和修改对方的 DOM
- 不读访问对方的 Cookie、IndexDB 和 LocalStorage
- 限制 XMLHttpRequest 、fetch 等网络请求，即请求可以发出，但是不能接收到响
应。

同源的条件：

- 主机名相同（域名+服务器名）
- 协议相同（http 和 https 不同）
- 端口相同

因为不符合同源策略的 ajax 请求会被禁用，因此就要想办法解决同源策略带来的限制。解决的方法有如下几种：

- CORS：跨域资源共享
- JSONP
- nginx 反向代理
- 配置代理
- nodejs 中间件

### CORS

CORS 其实是 W3C 的一个标准，全称是`跨域资源共享`；它需要浏览器和服务器的共同支持。

#### 简单请求

浏览器根据请求方法和请求头的特定字段，将请求做了一下分类，具体来说规则是这样，凡是满足下面条件的属于简单请求:

**简单请求**的范围:

1. 请求方法是以下三种方法之一：

- HEAD
- GET
- POST

2. HTTP 的头信息不超出以下几种字段：

- Accept
- Accept-Language
- Content-Language
- Last-Event-ID
- Content-Type：
  - application/x-www-form-urlencoded
  - multipart/form-data
  - text/plain

浏览器画了这样一个圈，在这个圈里面的就是简单请求, 圈外面的就是非简单请求，然后针对这两种不同的请求进行不同的处理。

---

对于简单请求，浏览器会在请求头加上`Origin`字段，用来说明请求来自于哪个`源`；
服务器拿到请求之后，会在响应头添加一些字段，主要有：

- `Access-Control-Allow-Origin`：如果`Origin`不在这个字段的范围中，浏览器就会将响应拦截。
  > Access-Control-Allow-Origin 字段设置为\*有什么问题？
  > 除了安全问题之外，还有一个问题是只要设置为\*就不能发送 cookies，即使`Access-Control-Allow-Credentials`为 true 也不行。
  > 如果需要发送，就必须指定明确的、与请求网页一致的域名。
- `Access-Control-Allow-Credentials`：表示是否允许发送 Cookie
  这一项的属性只能为 true，否则就不添加这个字段
- `Access-Control-Expose-Headers`：允许 xhr 可以拿到除了 6 个基本响应字段（`Cache-Control`、`Content-Language`、`Content-Type`、`Expires`、`Last-Modified`和`Pragma`）之外的其他字段。

这其中最主要的是第一个字段；如果设置的 origin 中包含当前请求，就可以不受同源策略的限制；如果设置为`*`，相当于完全不受同源策略影响，任何源都可以访问。

#### 非简单请求

非简单请求和简单请求处理的不同主要体现在两方面：

- 预检请求
- 响应字段

非简单请求发送之前会先发送一个预检请求，方法固定是`options`；会加上 Origin 源地址和 Host 目标地址。同时也会加上两个关键的字段:

- `Access-Control-Request-Method`， 列出 CORS 请求用到哪个 HTTP 方法
- `Access-Control-Request-Headers`，指定 CORS 请求将要加上什么请求头

预检请求会像这样：

```
OPTIONS / HTTP/1.1
Origin: 当前地址
Host: xxx.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
```

随后响应字段也会有所不同；响应字段除了响应预检请求本身，还有对 CORS 的相关字段：

```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000
Content-Type: text/html; charset=utf-8
Content-Encoding: gzip
Content-Length: 0
```

其中有这样几个关键的响应头字段:

- `Access-Control-Allow-Origin`: 表示可以允许请求的源，可以填具体的源名，也可以填`*`表示允许任意源请求，和简单请求的该字段一样
- `Access-Control-Allow-Methods`: 表示允许的请求方法列表。
- `Access-Control-Allow-Credentials`: 同简单请求
- `Access-Control-Allow-Headers`: 表示允许发送的请求头字段
- `Access-Control-Max-Age`: 预检请求的有效期，在此期间，不用发出另外一条预检请求。

在预检请求的响应返回后：

- 如果请求不满足响应头的条件，则触发`XMLHttpRequest`的`onerror`，当然后面真正的 CORS 请求也不会发出去了。
- 如果满足条件，则会和简单请求一样，浏览器自动加上`Origin`字段，响应头返回`Access-Control-Allow-Origin`。

### JSONP

`XMLHttpRequest` 对象遵循同源政策，但是`<script>`标签不一样，它可以通过 src 填上目标地址从而发出 GET 请求，实现跨域请求并拿到响应。这也就是 JSONP 的原理。
基本原理：

```js
function callback(data) {
  console.log(data);
}
const script = document.createElement("script");
script.src = `http://example.com?callback=callback`;
document.body.appendChild(script);
```

简单实现如下：

```js
function jsonp({ url, params, callback }) {
  const script = document.createElement("script");
  const cbFnName = `JSONP_PADDING_${Math.random().toString().slice(2)}`;
  script.src = `${url}?${JSON.stringfy(params)}&callback=${cbFnName}`; // 这一步封装url还有更详细的操作，这里只是简单表示
  // 为了避免全局污染，使用一个随机函数名
  window[cbFnName] = callback; // 把回调插入window上，这里也可以直接定义一个全局函数
  document.body.appendChild(script);
}

jsonp({
  url: "http://localhost:8080",
  params: { id: 10000 },
  callback(data) {
    console.log("Data:", data);
  },
});
```

Promise 实现：

```js
/*封装jsonp*/
const jsonp = ({ url, params, callbackName }) => {
  //把params封装到url中
  const generateURL = () => {
    let dataStr = "";
    for (let key in params) {
      dataStr += `${key}=${params[key]}&`;
    }
    dataStr += `callback=${callbackName}`;
    return `${url}?${dataStr}`;
  };
  return new Promise((resolve, reject) => {
    // 初始化回调函数名称
    callbackName = callbackName || Math.random().toString.replace(",", "");
    // 创建 script 元素并加入到当前文档中
    let scriptEle = document.createElement("script");
    scriptEle.src = generateURL();
    document.body.appendChild(scriptEle);
    // 绑定到 window 上，为了后面调用
    window[callbackName] = (data) => {
      resolve(data);
      // script 执行完了，成为无用元素，需要清除
      document.body.removeChild(scriptEle);
    };
  });
};

/*使用*/
jsonp({
  url: "http://localhost:3000",
  params: {
    a: 1,
    b: 2,
  },
}).then((data) => {
  // 拿到数据进行处理
  console.log(data); // 数据包
});

/*服务端*/
let express = require("express");
let app = express();
app.get("/", function (req, res) {
  let { a, b, callback } = req.query;
  console.log(a); // 1
  console.log(b); // 2
  // 注意，返回给script标签，浏览器直接把这部分字符串执行；因此就相当于执行了一个全局定义的callback
  res.end(`${callback}('数据包')`);
});
app.listen(3000);
```

可以看到，JSONP 最大的优势在于兼容性好缺点也很明显，请求方法单一，只支持 GET 请求。

---

还有一种方法是显式添加`<script>`标签，然后通过回调的方式取出 script 中的数据：

把要获取的数据封装成一个 js 文件

```js
//weather.js
showWeather(
  JSON.parse({
    weather: "sunny",
    time: "2021-12-03 16:30",
  })
);
```

然后在 js 中的 script 标签获取

```html
<script src="/api/weather.js"></script>
<script>
  function showWeather(data) {
    console.log(data);
  }
</script>
```

#### JSOP 的安全问题

jsonp 是有一些很明显的安全问题的。

1. XSS 攻击：jsonp 获取 js 代码并执行的方式，非常容易导致 xss 攻击。假如攻击者篡改或伪造返回的代码内容，就可能执行 xss 攻击。解决方式是可以对返回内容进行校验和过滤，防止恶意代码的插入；
2. 数据泄露：由于 jsonp 返回的数据是明文形式的 json，因此有着和 jwt 类似的问题，即数据泄露的风险。解决方式是可以对数据进行加密，以及采用 https 传输防止窃听。

解决方式：数据泄露的问题可以通过https来辅助解决，但是数据本身可能需要其他的加密方式。
而xss攻击，我们可以利用到**js沙箱**，让jsonp返回的js在沙箱内执行，限制或阻止其对window、Object等全局对象的修改和访问。
前端沙箱的实现方式可以参考下面几篇文章：

https://juejin.cn/post/6865640710394249223
https://www.51cto.com/article/710911.html


### Nginx 反向代理

![](https://pic.imgdb.cn/item/6234574f5baa1a80ab235752.jpg)

- 正向代理帮助客户端访问客户端自己访问不到的服务器，然后将结果返回给客户端。
- 反向代理拿到客户端的请求，将请求转发给其他的服务器，主要的场景是维持服务器集群的负载均衡，换句话说，反向代理帮其它的服务器拿到请求，然后选择一个合适的服务器，将请求转交给它。

因此正向代理服务器是帮**客户端**做事情，而反向代理服务器是帮**其它的服务器**做事情。

Nginx 的配置原理很像一般脚手架提供的配置方法，即利用**服务器之间没有同源限制**，把代理服务器设置在和客户端同源的路由下，然后转发请求到服务器；收到响应后再转发回客户端。
比如这样的配置：

```
server {
  listen  80;
  server_name  client.com;
  location /api {
    proxy_pass server.com;
  }
}
```

Nginx 相当于起了一个跳板机，这个跳板机的域名也是`client.com`，让客户端首先访问 `client.com/api`，这当然没有跨域，然后 Nginx 服务器作为反向代理，将请求转发给`server.com`，当响应返回时又将响应给到客户端，这就完成整个跨域请求的过程。

### http-proxy-middleware

http-proxy-middleware 解决跨域的方案，是通过本地起了一个 node 代理服务器（`var httpProxy = require('http-proxy')`），通过代理服务器去请求目标服务器，然后返回请求结果。由于浏览器请求的是本地路径，所以不会有跨域问题。

这个中间件的使用有两个方式，一是在 nodejs 服务端设定该代理服务器，然后作为中间件的形式使用；

```js
// 中间代理服务器
const express = require("express");
const proxy = require("http-proxy-middleware");
const app = express();

app.use(
  "/",
  proxy({
    // 代理跨域目标接口
    target: "http://www.proxy2.com:8080",
    changeOrigin: true,
    // 修改响应头信息，实现跨域并允许带 cookie
    onProxyRes: function (proxyRes, req, res) {
      res.header("Access-Control-Allow-Origin", "http://localhost");
      res.header("Access-Control-Allow-Credentials", "true");
    },
    // 修改响应信息中的 cookie 域名
    cookieDomainRewrite: "localhost", // 可以为 false，表示不修改
  })
);
app.listen(3000);
```

还有一种形式是利用 webpack 的 devServer 配置项，在其中引入该代理服务器的配置。webpack 会把这个中间件用在配置本地启动的服务器上：

```js
module.exports = {
  //...
  devServer: {
    proxy: {
      "/api": {
        target: "http://localhost:3000",
        pathRewrite: { "^/api": "" },
        changeOrigin: true,
      },
    },
  },
};
```

## 网络模型

![](https://pic.imgdb.cn/item/623464365baa1a80ab2d7eee.png)

- 应用层 (application layer)：直接为应用进程提供服务。应用层协议定义的是应用进程间通讯和交互的规则，不同的应用有着不同的应用层协议，如 HTTP 协议（万维网服务）、FTP 协议（文件传输）、SMTP 协议（电子邮件）、DNS（域名查询）等。
  - 表示层：用于解决两个系统间交换信息的语法与语义问题，还有数据表示转化(转为主机无关编码)，加解密和压缩与解压缩功能。
  - 会话层：用于建立会话 SSL 等
- 传输层 (transport layer)：有时也译为运输层，它负责为两台主机中的进程提供通信服务。该层主要有以下两种协议：
  - 传输控制协议 (Transmission Control Protocol，TCP)：提供面向连接的、可靠的数据传输服务，数据传输的基本单位是报文段（segment）；
  - 用户数据报协议 (User Datagram Protocol，UDP)：提供无连接的、尽最大努力的数据传输服务，但不保证数据传输的可靠性，数据传输的基本单位是用户数据报。
- 网络层 (internet layer)：有时也译为网际层，它负责为两台主机提供通信服务，并通过选择合适的路由将数据传递到目标主机。
- 数据链路层 (data link layer)：负责将网络层交下来的 IP 数据报封装成帧，并在链路的两个相邻节点间传送帧，每一帧都包含数据和必要的控制信息（如同步信息、地址信息、差错控制等）。
- 物理层 (physical Layer)：确保数据可以在各种物理媒介上进行传输，为数据的传输提供可靠的环境。

![](https://pic.imgdb.cn/item/623464f65baa1a80ab2dcd4e.jpg)

这张图更清晰展现了每一层的协议处在计算机的什么位置上：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/%E9%94%AE%E5%85%A5%E7%BD%91%E5%9D%80%E8%BF%87%E7%A8%8B/7.jpg)

## 鉴权校验相关

### Session

> 有关于 session/token/JWT 的更多解释可以看这里：https://mubu.com/doc/12i79Sq9hmP

![](https://pic1.imgdb.cn/item/634bfad916f2c2beb1870975.jpg)
是另一种 cookie，同时依赖于 cookie，但是存储在服务端，sessionId 会被存储到客户端的 cookie 中

- 用户第一次请求服务器的时候，服务器根据用户提交的相关信息，创建对应的 Session
- 请求返回时将此 Session 的唯一标识信息 SessionID 返回给浏览器
- 浏览器接收到服务器返回的 SessionID 信息后，会将此信息存入到 Cookie 中，同时 Cookie 记录此 SessionID 属于哪个域名
- 当用户第二次访问服务器的时候，请求会自动判断此域名下是否存在 Cookie 信息，如果存在自动将 Cookie 信息也发送给服务端，服务端会从 Cookie 中获取 SessionID，再根据 SessionID 查找对应的 Session 信息，如果没有找到说明用户没有登录或者登录失效，如果找到 Session 证明用户已经登录可执行后面操作。

> session 和 cookie 有什么区别:
> cookie 是 Web 服务器发送给浏览器的一块信息。浏览器会在本地文件中给每一个 Web 服务器存储 cookie。以后浏览器在给特定的 Web 服务器发请求的时候，同时会发送所有为该服务器存储的 cookie。下面列出了 session 和 cookie 的区别：
> 无论客户端浏览器做怎么样的设置，session 都应该能正常工作。客户端可以选择禁用 cookie，但是，session 仍然是能够工作的，因为客户端无法禁用服务端的 session。
> 在存储的数据量方面 session 和 cookies 也是不一样的。session 能够存储任意的 Java 对象，cookie 只能存储 String 类型的对象。

session 有一个巨大的问题，在负载均衡中，一台服务器的 session 并不会共享给其他的，那么当请求被转发之后，就有可能导致 session 无法被验证。

### Token（令牌）

token 是一种鉴权方式，它和 session、cookie 等最大的不同在于，token 自身包含一定的数据，服务端可以通过解析数据来进行某些身份识别，从而降低或者完全避免查库。

常见的 token 类型有，access token（最基本的 token）、refresh token（用于刷新其他 token 的）、csrf token（用于预防 csrf 攻击）和 jwt。

token 有一些共同特点，包括：

- 不受同源策略限制。很多文章都说 token 是一种跨域认证的解决方案，这个“跨域”的意思并不是说 token 可以完成跨域，跨域的方法依然是 cors；只是说，token 一般是在请求时加在 http header 中的，header 可以被服务端正常接收和处理，即使不同源。而 cookie 在请求不同源时不会发送。
- 包含一定的信息。使用 token 时，服务端不用存放 token，而是解析 token 数据，然后用解析出来的数据再去数据库查询用户信息。使用 jwt 时，解析 jwt 出来的数据以及包含了用户信息，不用再查询数据库。相比于 session 和 cookie 这种直接查库的方式，token 可以减少查库的次数和消耗。

#### Access Token

访问资源接口（API）时所需要的资源凭证，相当于展示证件才能进入或访问的过程
简单 token 的组成： uid(用户唯一的身份标识)、time(当前时间的时间戳)、sign（签名，token 的前几位以哈希算法压缩成的一定长度的十六进制字符串）
![accesstoken.png](https://i.loli.net/2021/12/02/EKhFUQzAP3vcjxa.png)

- 客户端每一次请求都需要携带 token，需要把 token 放到 HTTP 的 Header 里
- token 完全由应用管理，所以它可以避开同源策略
- 客户端收到 token 以后，会把它存储起来，比如放在 cookie 里或者 localStorage 里

#### Refresh Token

是一种延长 accesstoken 的方法，主要解决方法是：

- 如果 accesstoken 没过期，就正常使用
- 如果 accesstoken 过期，但 Refresh Token 没过期，就可以获取到新的 Token
- 如果都过期就需要重新登录
- Refresh Token 及过期时间是存储在服务器的数据库中，只有在申请新的 Acesss Token 时才会验证

### JWT

jwt 和其他 token 的区别主要是，jwt 包含用户信息，有些情况下可以做到完全不查库。
不过这个用户信息有风险，即明文传输可能被篡改或窃取。这也是 jwt 最大的问题。

JSON Web Token（简称 JWT）是目前最流行的**跨域认证**解决方案。是一种认证授权机制。

JWT 的原理是，服务器认证以后，生成一个 JSON 对象，发回给用户，就像下面这样。

```json
{
  "姓名": "张三",
  "角色": "管理员",
  "到期时间": "2022年7月1日0点0分"
}
```

以后，用户与服务端通信的时候，都要发回这个 JSON 对象。服务器完全只靠这个对象认定用户身份。

实际上的 jwt 是经过 Base64 处理之后的，一个很长的字符串，通常包含有实际需要传递的数据。因此 JWT 可以一定程度上降低服务器查询数据库的次数。

session 是保存在服务端的，而 jwt 是保存在客户端的。
原理图：
![JWT.png](https://i.loli.net/2021/12/02/chSBlOHZ9dDIQiJ.png)
使用方式：

- 跨域的时候，可以把 JWT 放在 POST 请求的数据体里。
- 通过 URL 传输，比如`http://www.example.com/user?token=xxx`
- 放在 HTTP 请求头信息的 Authorization 字段里

缺点：

1. 安全性不好，因为 jwt 是明文传输的，并且经常需要从客户端发向服务端，不安全
2. 由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。

### token 刷新

这块内容不算计网的部分，属于是前端对 token 刷新做的处理。
最基础的问题是，如果后端 token 过期，用户请求就会报错，那么对于前端来说应该怎么做？

显然这时有两种方式

1. 通过路由让用户返回登录页，并进行报错提示。更进一步的问题在于，如果每个请求都做这样的操作，那么多个请求同时返回指定状态码的时候，返回操作和报错就会被执行很多次，这个怎么避免？

2. 通过 refresh token 等方式刷新 token。这种方式有个问题在于，我们可以通过获取 token 的请求获得最新 token，但怎么让用户不感知？也就是说，在获得 token 时报错的这些请求，也需要在获得新的 token 之后重新发送，或者通过通知用户刷新页的方式重新发送。

目前了解到的方式如下：

基本还是依赖拦截器。当收到返回特殊状态码的请求时，通过一个变量来阻止其他请求进入处理逻辑（阻止响应，而不是阻止请求）。

如果路由已经进入登录页，或者某个全局状态改变，那么我们也可以限制后续请求不再执行跳转和刷新。

举个例子

```js
let isRefreshToken = false;

axios.interceptors.response.use(
  (response) => {
    if (response.status == 200) {
      if (isRefreshToken) isRefreshToken = false; // 正常请求恢复状态
      return response;
    }
  },
  (err) => {
    let res = err.response || {};
    if (res.data.meta?.statusCode == 302) {
      if (当前路由不在登录页) {
        // 在登录页时不再重复请求
        if (!isRefreshToken) {
          isRefreshToken = true;
          // 路由跳转或刷新token
          isRefreshToken = false; // 刷新成功后
        }
      }
    } else {
      // 非302 报的错误;
      return err;
    }
  }
);
```

除了这种方式，axios 提供另一个 cancelToken 来取消请求。
我们可以把每个请求都放入一个数组中

- 如果请求成功或非鉴权报错，那么从数组中删除
- 如果请求出现鉴权问题，那么将其他全部请求取消，然后执行跳转或者刷新操作。
  更进一步，既然我们已经拿到了请求数组，那也可以在刷新之后，将这些请求再执行一遍。

```js
const cancelToken = axios.CancelToken; //阻止请求
const pendding = []; //把当前请求的状态都存到一个数组里
axios.interceptors.request.use((config) => {
  config.cancelToken = new cancelToken((cancel) => {
    pendding.push({
      cancelCallback: cancel,
      url: config.url
    });
  });
});
request.interceptors.response.use((response) => {
  if (response.data.code === 200) {
    //对于请求ok的数据
    // 从pending中查找删除
    return response.data;
  }
  if (response.data.code === 102 && 当前路由不在登录页) {
    for (let i = 0; i < pendding.length; i++) {
      pendding[i].cancelCallback(); //执行取消操作
      pendding.splice(i, 1); //把这条记录从数组中移除
    }
    // 路由跳转或刷新token
  }
});
```

### cookie

cookie 的详细解释可以看这篇文章:https://juejin.cn/post/6844904073934667790

> HTTP Cookie（也叫 Web Cookie 或浏览器 Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。通常，它用于告知服务端两个请求是否来自同一浏览器，如保持用户的登录状态。Cookie 使基于无状态的 HTTP 协议记录稳定的状态信息成为了可能。

因此 cookie 的主要工作方式是:在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。

- 创建方式

服务器在响应头里添加`Set-Cookie`选项,比如这样

```
HTTP/1.0 200 OK
Content-type: text/html
Set-Cookie: yummy_cookie=choco
Set-Cookie: tasty_cookie=strawberry
```

js 也可以直接通过`document.cookie`来读写 cookie

#### cookie 的跨域问题

cookie 受到同源策略（CORS）的限制。如果两个网站不同源，那就不能共享 cookie。至于 cookie 怎么跨域访问，在上面 cors 中说过了，主要通过 HTTP 头的`Access-Control-Allow-Credentials`可以控制跨域请求时的 cookie 发送与否，配合`XMLHttpRequest.withCredentials`设置为 true，就可以在发送跨域请求时携带 cookie。

cookie 可以通过设置 Domain 来允许子域名访问，也可以设置 SameSite 控制从一个页面跳转到另一个时能不能共享 cookie。

和 cookie 一样，localstorage 也会收到同源限制。不同源的 localstorage 和 sessionstorage 不能访问；如果同源的不同页面可以共享 localstorage，但不能共享
sessionstorage/localstorage 的同源限制可以通过 postMessage 解决。

#### 第三方 cookie

第三方 cookie 是建立在别的域名上的 cookie，而不是你当前访问的域名（地址栏中的）。

设我们现在访问的网站是 'bar.com'，当我们引入 'foo.com' 的图片时，图片服务（foo.com的来源服务器）如果设置了 Cookie，我们就称之为 “第三方 Cookie”

简单来说就是：在当前访问的网站和请求服务的网站是“跨站”（Cross Site）的情况下，第三方服务设置的 Cookie 就称之为 “第三方 Cookie”。

常见的场景有：
- 请求第三方网站的图片
- 向第三方网站发起请求（比如天气api）
- 表单提交到第三方网站
- 链接跳转到第三方网站。

其中，链接跳转也是一种。比如在A页面上嵌入一个跳转到github.com的链接，如果github在浏览器上设置的登录cookie能被发送，那么点击这个链接之后跳转到github就会是登录状态。

如何在向第三方网站请求的时候能携带cookie呢？必须满足几个条件：

> 什么叫做向第三方网站请求？举个栗子，我们想在页面上嵌入一个b站的播放器，那么这个播放器其实是一个iframe，他会向b站的服务器请求数据。如果下面的条件没有满足，那么这个播放器请求就不会携带b站在用户浏览器上设置的cookie，最直观的影响就是没有登录信息，不能三连。

- https，以及cookie的secure属性
- Access-Control-Allow-Origin不能为`*`
- Access-Control-Allow-Credentials 设置为 true
- SameSite 属性设置为 None，或者在部分情况下lax属性

---

第三方cookie有造成csrf攻击的风险。比如有一个银行网站 A 和恶意网站 B。

1. 银行网站 A 登录后在浏览器留下 cookie
2. 在 cookie 没有过期的前提下，用户进入恶意网站 B，B 中伪造了一个表单，向 A 的服务器发送了一个请求

这个cookie是属于A网站的， B 网站向 A 服务器发送请求，就相当于在跨站情况下使用该cookie，对应上面的场景。浏览器会带上了 cookie，然后A网站如果没有其他防范措施，就会视作是真正的用户。

当然这是在SameSite 属性设置为 None的情况下。如果是lax，那么除了a标签、预请求和get表单提交，其他的方式都不会携带A网站的cookie，也就不能发起攻击请求了。

#### 跨站和跨域

跨站和跨域完全不一样。一般来说，同源策略的限制比较严格，但是跨站的限制稍小一些；只要两个 url 的**顶级域名+二级域名相同**，就可以被视作同站。

比如`www.abc.com`和`blog.abc.com`就被视作同站，但他们显然不同源
同理，`www.abc.com`和`www.def.com`就被视作跨站，因为二级域名不一样。

但有时候，位于倒数第二个的域名未必是二级域名。比如github.io，github就不是一个二级域名，它仍属于顶级域名。
另外，协议不同一定不同站，但是端口不同可以被视作同站。

#### cookie 属性

- 名称: 表示该 cookie 的名字,一般常用`_`开头表示一些身份信息或 id 等
- 值: 就是 cookie 的值,可以是任意 ASCII 字符
- Domain: 限制访问域名,不设置就是都可以访问。domain 设置的域名下的子域名都可以访问，比如设置了`test.com`，那么其下的`child.test.com`也可访问。
- Path: 主机下哪些路径可以访问,比如登录路径访问,就可以设置为`/login`。同样也是会匹配子路径
- SameSite：用于判断跨站时是否发送 cookie。samesite 有三个值：
  - Strict：最严格，跨站完全不允许发送 cookie
  - Lax：![](https://pic.imgdb.cn/item/629df38c094754312972cc14.jpg)
  - None：都可以发送，前提是必须同时设置 Secure 属性
- Expires/Max-Age: 生命周期,定义 Cookie 的有效时间
- HttpOnly:为避免跨域脚本 (XSS) 攻击，通过 JavaScript 的 Document.cookie API 无法访问带有 HttpOnly 标记的 Cookie，它们只应该发送给服务端。
- Secure: 标记时只能通过 https 发给服务端

#### samesite的意义

举个栗子，github在你的浏览器留下了cookie，可以保证你登录github时保持状态。但是你进入github的方式有几种，包括：

- 直接输入url进入
- 点击书签、历史记录等记录
- 点击其他页面的链接进入
- 其他页面的js跳转

这里边的后两种，都属于跨站操作。比如从百度的搜索结果跳转到github，相当于是从baidu.com向github.com发请求。
这时用到的github的cookie就是一个第三方cookie，该cookie的samesite属性就会起到限制作用。
- 如果samesite设置为strict，那么除非是直接进入，否则其他网站的跳转都不能携带github的cookie，这样进去就总是没登录状态。
- 如果samesite属性是none，那么其他网站有可能会伪造一个请求，携带github的cookie，执行一些攻击操作。
- 如果是lax，那么只有a标签跳转、预请求和get表单才会发送，这样既不会影响跳转，也不会有较大风险。

除了需要跳转到github之外，请求github的图片、使用github提供的iframe、提交表单给github等操作也是一样的。

### OAuth

OAuth 就是一种授权机制。数据的所有者告诉系统，同意授权第三方应用进入系统，获取这些数据。系统从而产生一个短期的进入令牌（token），用来代替密码，供第三方应用使用。OAuth 在"客户端"与"服务提供商"之间，设置了一个授权层（authorization layer）。"客户端"不能直接登录"服务提供商"，只能登录授权层，以此将用户与客户端区分开来。"客户端"登录授权层所用的令牌（token），与用户的密码不同。用户可以在登录的时候，指定授权层令牌的权限范围和有效期。
![](https://obohe.com/i/2021/12/03/sf3lt0.png)
流程如下:

1. 用户打开客户端以后，客户端要求用户给予授权。
2. 用户同意给予客户端授权。
3. 客户端使用上一步获得的授权，向认证服务器申请令牌。
4. 认证服务器对客户端进行认证以后，确认无误，同意发放令牌。
5. 客户端使用令牌，向资源服务器申请获取资源。
6. 资源服务器确认令牌无误，同意向客户端开放资源。

### SSO

SSO(单点登录)指的是在多个应用系统中，只需登录一次，就可以访问其他相互信任的应用系统。
详细可以看这篇文章https://www.zhihu.com/question/35165725

# 应用层

应用层的任务是通过应用进程间的交互来完成特定网络应用。应用层协议定义的是应用进程（进程：主机中正在运行的程序）间的通信和交互的规则。对于不同的网络应用需要不同的应用层协议。应用层协议是应用到应用的协议，相应的，传输层是进程到进程，网络层是主机到主机。
应用层主要协议：

- HTTP
- SMTP
- FTP
- DNS，但是 DNS 更像是一个应用，服务于上面的协议。

## 模型

应用层的模型主要有两个：

1. C/S 模型，即客户端/服务器模型；服务器总是启动，等待客户端的请求连接并发回响应。Web 和电子邮件都是这种
2. P2P 模式，每一个主机既是客户端又是服务器，资源在主机间交换。

## HTTP

### 基本特点

超文本传输协议（HTTP，HyperText Transfer Protocol）是互联网上应用最为广泛的一种网络协议。所有的 WWW（万维网） 文件都必须遵守这个标准。

- HTTP 基于 TCP，要建立 HTTP 连接的进程需要通过 TCP 传输，因此是可靠数据传输，并且拥有拥塞控制等优化手段。
- HTTP 是无状态的协议，单靠 HTTP 的话，并不会记录之前的传输，如果再次请求将会再次发送（可以通过 HTTP 缓存解决）；同时也不会存储关于用户的任何信息
- HTTP 客户端进程默认位于 80 端口
- 请求--应答方式，一发一收连接
- 灵活可扩展，传输数据自由不限制，并且语义比较自由

### 持续和非持续连接

HTTP 默认采用持续连接，即客户端和服务器建立 TCP 连接后，会一直使用这条连接进行数据交互，直到没有数据传输或异常断开。在空闲期间，通常会使用`心跳数据包（Keep-Alive）`保持链路不断开。如果这条链接一段时间未使用，服务器就会关闭连接。在持续链接持续期间，不再需要额外的 TCP 握手、挥手，只需要进行数据传输；

可以采用非持续连接，每一次发送都需要建立和断开连接，具体步骤如下：

1. HTTP 客户端在 80 端口向服务器发起一个 TCP 连接
2. 三次握手连接建立后，通过 socket 向服务器发送 HTTP 请求报文
3. 服务器接受报文，返回响应报文
4. 服务器通知 TCP 断开连接
5. 客户端收到响应报文，TCP 经过四次挥手关闭连接。

持续链接实际上就是对第 3 步的扩充

### HTTP 报文

#### 请求报文

```
POST / HTTP1.1
Host:www.wrox.com
User-Agent:Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022)
Content-Type:application/x-www-form-urlencoded
Content-Length:40
Connection: Keep-Alive

name=Professional%20Ajax&publisher=Wiley
```

![request.png](https://i.loli.net/2021/12/02/geW1ol4FckqY7ty.png)

报文的第一行叫做请求行，后续叫做首部行（请求头）

1. 请求行，用来说明请求类型,要访问的资源以及所使用的 HTTP 版本。GET 说明请求类型为 GET,`/xxx`为要访问的资源，或者也可以是路由；最后一部分说明使用的是 HTTP1.1 版本。
2. 请求头部（首部行），紧接着请求行（即第一行）之后的部分，用来说明服务器要使用的附加信息。
3. 空行，请求头部后面的空行是必须的；即使第四部分的请求数据为空，也必须有空行。
4. 请求数据，也叫实体体，可以添加任意的其他数据，GET 请求没有这部分。

#### 响应报文

```
HTTP/1.1 200 OK
Date: Fri, 22 May 2009 06:07:21 GMT
Content-Type: text/html; charset=UTF-8

<html>
      <head></head>
      <body>
            <!--body goes here-->
      </body>
</html>
```

1. 状态行，由 HTTP 协议版本号、状态码、状态消息三部分组成。第一行为状态行，`HTTP/1.1`表明 HTTP 版本为 1.1 版本，状态码为 200，状态消息为（ok）
2. 消息报头（首部行/响应头），用来说明客户端要使用的一些附加信息
3. 空行，消息报头后面的空行是必须的
4. 响应正文（实体体），服务器返回给客户端的文本信息。空行后面的 html 部分为响应正文。

### HTTP 字段

http1.1 的字段主要有 47 个，按照分类主要有 4 类：

1. 通用首部字段，常见的有：

- `Connection` 连接管理、逐条首部
- `Upgrade` 升级为其他协议
- `Cache-Control` 缓存控制，即代替`Expires`字段的缓存控制字段

> 注意区分http的`Connection:Keep-Alive`和TCP的`Keepalive`：
> TCP 的 Keepalive，是由 TCP 层（内核态） 实现的，称为 TCP 保活机制；

2. 请求首部字段，包括 Accept 系列字段，以及`User-Agent` 客户端程序信息等请求相关的字段
3. 响应首部字段，响应时相关字段
4. 实体首部字段，主要是 Content 相关字段，主要规定实体特征。实体首部字段在请求和响应中都可能有，只和请求体相关；如果请求没有请求体（GET/HEAD），则不会携带该字段

HTTP 头会按照`请求/响应 - 通用 - 实体`顺序分别安排四种首部

全部字段见：
https://juejin.cn/post/6844903748196646920

每个字段的用处，可以参考《图解 HTTP》

#### 常见的请求头

- `Accept`:浏览器能够处理的内容类型，即 MIME type
- `Accept-Charset`:浏览器能够显示的字符集，优先选择该字符集发送
- `Accept-Encoding`：浏览器能够处理的压缩编码
  - gzip: 当今最流行的压缩格式
  - deflate: 另外一种著名的压缩格式
  - br: 一种专门为 HTTP 发明的压缩算法
- `Accept-Language`：浏览器当前设置的语言

Accept 字段通常表示一种“优先选择”，即浏览器希望服务器发送一个希望的类型，并不是直接对发送的指示和标识。

- `Authorization`: Web 认证信息，通常用于携带 jwt、token 等认证信息。
- `Cookie`：当前页面设置的任何 Cookie
- `Host`：标识请求的目标主机
- `Referer`：发出请求的页面的 URL，表示请求的“来源”

> Referer 和 Origin 都是用于标识请求来源的字段，但也有一些区别：
>
> - Origin 通常会携带源的端口，并且不会携带路径、参数等信息；Referer 则会携带具体的路径，但不会携带哈希值
> - Referer 更多表示“跳转”，它的值不一定是某个固定的 url，而是指请求来自哪里。比如先在浏览器中输入 `https://www.example.com/page1.html`，并点击该页面中的链接访问 `https://www.example.com/page2.html`，那么当浏览器向服务器发送请求获取 page2.html 时，它会在请求头中添加 Referer 头，告诉服务器该请求是从 page1.html 页面跳转过来的。
> - Referer头在某些情况下不会发送：
>   - 从一个https页面跳转到一个非https页面时。比如正在浏览一个银行网站，该网站使用HTTPS加密，然后点击了一个链接，跳转到了一个不安全的HTTP页面。如果此时的浏览器发送Referer请求头，那么请求就会泄漏到HTTP页面上，存在危险。
>   - 直接在浏览器地址栏中输入URL或从书签中访问网站
>   - 来源页面采用的协议为表示本地文件的 "file"，即通过file:///协议访问的资源 
>   - 使用 JavaScript 的 Location.href 或者是 Location.replace()直接跳转
> - Referer的作用，主要是让服务器识别请求来源，从而进行一些统计分析、日志记录以及缓存优化等，还有个常见的用途是图片防盗链。防盗链的意思就是通过referer头来拦截非白名单网站对图片的访问，比如某个图片在`pic.a.com`下，如果通过本地访问可以看到，但如果把网页部署到`b.com`上，然后再从网页上发起请求，这时referer头就是`b.com`，`pic.a.com`的服务器就会拦截这个请求，不返回该图片。

- `User-Agent`：浏览器的用户代理字符串
- `X-Forwarded-For`：用于保存最开始发起请求的客户端的 IP 地址。主要为了解决因为多个代理服务器导致的服务端无法获取到原始的客户端的 ip 地址的问题。

```
按顺序表示客户端ip、代理服务器1ip、代理服务器2ip等
X-Forwarded-For: 203.0.113.195, 70.41.3.18, 150.172.238.178
```

#### 常见的响应头

- `Date`：表示消息发送的时间，时间的描述格式由 rfc822 定义
- `Server`:服务器名称
- `Cache-Control`：控制 HTTP 缓存
- `Access-Control`等 cors 相关的头

#### 常见的实体首部

- `Content-Encoding` 实体主体适用的编码方式
- `Content-Language` 实体主体的自然语言
- `Content-Length` 实体主体的大小。一般用于设置定长包体的长度，实际请求体大于长度将会被截取，小于则会报错；

> content-length 主要是为了解决 tcp 的粘包问题。http 通过换行符、空行等方式来分割请求头部，而请求体或响应体就需要通过 content-length 来划分边界，超出 length 就属于下一个报文的内容了。

- `Content-Location` 替代对应资源的 URI
- `Content-MD5`实体主体的报文摘要
- `Content-Range` 实体主体的位置范围
- `Content-Type` 实体主体的媒体类型
- `Expires` 实体主体过期的日期时间
- `Last-Modified` 资源的最后修改日期时间

常见的 `Content-Type` 属性值有：

1. `application/x-www-form-urlencoded`：浏览器的原生 `form` 表单，如果不设置 `enctype` 属性，那么最终就会以 `application/x-www-form-urlencoded `方式提交数据。该种方式提交的数据放在 body 里面，数据按照 `key1=val1&key2=val2` 的方式进行编码，key 和 val 都进行了 URL 转码。
2. `multipart/form-data`：该种方式也是一个常见的 POST 提交方式，通常表单上传文件时使用该种方式。
3. `application/json`：服务器消息主体是序列化后的 JSON 字符串。
4. `text/html、text/plain`：该种方式主要用来提交 XML 格式、纯文本格式的数据。

> `multipart/form-data`形式的请求体是一种多部分的数据格式常用于文件上传场景。多部分数据格式由多个部分组成，每个部分都有自己的 Content-Type 和内容。每个部分之间通过一定的分隔符进行分隔。
> 在 HTTP 请求中，每个部分由一个首部和一个消息体组成。首部包含了该部分的元数据信息，例如 Content-Disposition、Content-Type 等；消息体则是该部分的实际内容，通常是一个文件或一段数据。
> 举个例子，一个上传图片的多部分数据请求体可能长这样：
> Content-Type: multipart/form-data;boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
>
> ------WebKitFormBoundary7MA4YWxkTrZu0gW
> Content-Disposition: form-data; name="title"
>
> My Image Title
> ------WebKitFormBoundary7MA4YWxkTrZu0gW
> Content-Disposition: form-data; name="image"; filename="example.jpg"
> Content-Type: image/jpeg
>
> (binary image data here)
> ------WebKitFormBoundary7MA4YWxkTrZu0gW--
> 在这个例子中，请求体由两个部分组成，一个是 title 字段，另一个是 image 文件。每个部分以分隔符（----WebKitFormBoundary7MA4YWxkTrZu0gW）开始和结束，并且包含了 Content-Disposition 和 Content-Type 等元数据信息，以及实际的消息体数据。

### HTTP 请求方法

- `GET` **请求**指定的页面信息，并返回实体主体。
- `HEAD` 类似于 get 请求，只不过返回的响应中没有具体的内容，用于获取**报头**
- `POST` 向指定资源**提交**数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST 请求可能会导致新的资源的建立和/或已有资源的修改。
- `PUT` 从客户端向服务器传送的数据**取代**指定的文档的内容。
- `DELETE` **删除**
- `CONNECT` **连接**，比如 websocket 的最开始的请求方法就是 connect
- `OPTIONS` 用于**检测**服务器允许的 http 方法，一般是浏览器进行预检的时候自动发送的，跨域之前非简单请求就需要先发 options 获知服务端是否允许跨域
- `TRACE` 回显服务器收到的请求，主要用于**测试**或诊断。

方法分类:

- `Safe`:一定不会修改服务器上资源的请求，get、head、options
- `Idempotent`：幂等，除安全以外的请求，**一个请求被执行一次和执行多次服务器状态一样，效果一样**

### HTTP 状态码

- 1xx：指示信息--表示请求已接收，继续处理
  - 100：继续 服务器返回此代码表示已收到请求的第一部分，正在等待其余部分。
  - 101：切换协议 请求者已要求服务器切换协议，服务器已确认并准备切换.HTTP 升级为 WebSocket 的时候，如果服务器同意变更，就会发送状态码 101。
- 2xx：成功--表示请求已被成功接收、理解、接受
  - 204：成功但没有 body
- 3xx：重定向--要完成请求必须进行更进一步的操作；
  - 301：所请求的页面已经永久重定向至新的 URL
  - 302：所请求的页面已经临时重定向至新的 URL
  - 303：查看其它位置 表示对应当前请求的响应可以在另一个 URI 上被找到，而且客户端应当采用 GET 的方式访问那个资源。
  - 304：缓存相关，表示资源未更改可以使用缓存
  - 305：必须通过指定的代理才能访问
- 4xx：客户端错误--请求有语法错误或请求无法实现
  - 400：Bad Request 请求错误，通常是客户端请求体的问题，比如字段错误、参数缺失、url 格式不正确、json 格式不正确等
  - 401：未授权 请求要求身份验证。对于需要登录的网页，服务器可能返回此响应
  - 403：禁止 服务器拒绝请求
  - 404：未找到 服务器找不到请求的资源
  - 405：方法禁用 禁用请求中指定的方法
  - 406 Not Acceptable: 资源无法满足客户端的条件。
  - 408 请求超时，服务端收到了部分请求，但因为丢包、网络中断等原因没有收到完整的请求。
  - 409 Conflict: 多个请求发生了冲突。
  - 413 Request Entity Too Large: 请求体的数据过大。
  - 414 Request-URI Too Long: 请求行里的 URI 太大。
  - 429 Too Many Request: 客户端发送的请求过多。
  - 431 Request Header Fields Too Large 请求头的字段内容太大。
- 5xx：服务器端错误--服务器未能实现合法的请求
  - 500：服务器内部错误 服务器遇到错误，无法完成请求
  - 501：尚未实施 服务器不具备完成请求的功能。例如，服务器无法识别请求方法时可能会返回此代码
  - 502：错误网关 通常是服务器脚本语言端未启动或无响应，以及反向代理端无响应。。502 通常是和服务端没有建立连接，在代理服务器上就出现了问题。比如如果用户尝试访问某个网站，但代理服务器无法连接到后端服务器，则可能会收到 502 错误
  - 503：服务器不可用 服务器目前无法使用（由于超载或者停机维护）。通常，这只是暂时状态
  - 504：网关超时。504 常出现于响应时间超过 nginx 代理服务器配置的超时时间，可能是后端代码出现死循环、sql 语句查询太慢等情况。如果客户端发出一个请求，但服务器无法在规定时间内响应，则可能会收到 504 错误。
    > 504 和 408 都表示请求在一定时间内没有收到响应。不过 504 更偏向服务端侧的理解，而 408 更偏向客户端。即，504 更偏向于表示服务器端的响应缓慢导致迟迟不能回复，而客户端请求没问题；而 408 一般表示客户端网络原因导致的超时。
    > 502和504都是由网关或代理服务器返回的，通常情况下是源服务器的异常导致不能正常返回，只能由代理服务器代为返回。
  - 505：HTTP 版本不受支持 服务器不支持请求中所用的 HTTP 协议版本

### HTTPS

![](https://pic.imgdb.cn/item/623400465baa1a80abe10fb0.jpg)

- HTTP 是超文本传输协议，信息是明文传输，存在安全风险的问题。HTTPS 则解决 HTTP 不安全的缺陷，在 `TCP` 和 `HTTP` 应用层之间加入了 SSL/TLS 安全协议，使得报文能够加密传输。实际上 ssl 在应用层和传输层之间，更偏向属于应用层。
- HTTPS 在 TCP 三次握手之后，还需进行 SSL/TLS 的握手过程，才可进入加密报文传输。
- HTTPS 的端口号是 443。
- HTTPS 协议需要向 CA（证书权威机构）申请数字证书，来保证服务器的身份是可信的。

![https.jpg](https://i.loli.net/2021/12/02/Wlm3TKsZYBtc67H.png)

#### https 解决的问题

HTTP 由于是明文传输，所以安全上存在以下三个风险：

- 窃听风险，比如通信链路上可以获取通信内容，即通过抓包的方式就可以获取到明文的 http 传输内容
- 篡改风险，比如强制植入垃圾广告，视觉污染，用户眼容易瞎。
- 冒充风险，比如冒充淘宝网站，用户钱容易没。

HTTPS 在 HTTP 与 TCP 层之间加入了 SSL/TLS 协议，可以很好的解决了上述的风险：

- 信息加密：交互信息无法被窃取。采用混合加密的方式实现信息的机密性，对 http 传输的报文进行加密，解决了窃听的风险。
  所谓混合加密，就是指对称和非对称加密都用到；建立连接时使用非对称，连接建立后使用对称。
  对称加密速度快，非对称加密强度高；两者混用可以兼顾。
- 校验机制：无法篡改通信内容，篡改了就不能正常显示。
  摘要算法的方式来实现完整性，发送方会生成数据的摘要值并和数据一起给接收方发送过去，接收方会采用同样的算法对接收到的数据再计算一遍摘要值，然后和发送方发送的摘要值比较。这样，如果数据中间被篡改，那么接收方和发送方的摘要值肯定不相同，这时就可以判定被篡改。
  摘要算法计算出的哈希值相当于传输数据的“指纹”。数据不更改时，指纹也就不会更改，反之则有可能更改。
- 身份证书：证明淘宝是真的淘宝网。将服务器公钥放入到数字证书中，解决了冒充的风险。

#### https 的安全依据

##### 混合加密

对称和非对称混合加密（对称：加密和解密用的同一套密钥，非对称就是不同套密钥）

> 对称加密，顾名思义就是加密和解密都是使用同一个密钥，常见的对称加密算法有 DES、3DES 和 AES 等，其优缺点如下：
> 优点：算法公开、计算量小、加密速度快、加密效率高，适合加密比较大的数据。
> 缺点：
> 交易双方需要使用相同的密钥，也就无法避免密钥的传输，而密钥在传输过程中无法保证不被截获，因此对称加密的安全性得不到保证。每对用户每次使用对称加密算法时，都需要使用其他人不知道的惟一密钥，这会使得发收信双方所拥有的钥匙数量急剧增长，密钥管理成为双方的负担。对称加密算法在分布式网络系统上使用较为困难，主要是因为密钥管理困难，使用成本较高。

> 非对称加密，顾名思义，就是加密和解密需要使用两个不同的密钥：公钥（public key）和私钥（private key）。公钥与私钥是一对，如果用公钥对数据进行加密，只有用对应的私钥才能解密；如果用私钥对数据进行加密，那么只有用对应的公钥才能解密。非对称加密算法实现机密信息交换的基本过程是：甲方生成一对密钥并将其中的一把作为公钥对外公开；得到该公钥的乙方使用公钥对机密信息进行加密后再发送给甲方；甲方再用自己保存的私钥对加密后的信息进行解密。

> ![](https://pic.imgdb.cn/item/623413cf5baa1a80abeee528.png)

混合加密主要应用在两个地方：

1. 握手时使用非对称加密，握手完成后后续数据使用对称加密

- 对称加密只使用一个密钥，运算速度快，密钥必须保密，无法做到安全的密钥交换。
- 非对称加密使用两个密钥：公钥和私钥，公钥可以任意分发而私钥保密，解决了密钥交换问题但速度慢。

建立连接时使用非对称加密可以确保连接的安全，并且密钥也是在这个过程中动态生成的，保存在服务端可客户端两边，不易被窃取；
建立连接后为了保证传输的效率，就使用对称加密。

2. 根据加解密采用的方式不同，可以起到不同的效果。

混合加密方式是可以双向加解密的，比如可以用公钥加密内容，然后用私钥解密，也可以用私钥加密内容，公钥解密内容。

流程的不同，意味着目的也不相同：

- 公钥加密，私钥解密。这个目的是为了保证内容传输的安全，因为被公钥加密的内容，其他人是无法解密的，只有持有私钥的人，才能解密出实际的内容；
- 私钥加密，公钥解密。这个目的是为了保证消息不会被冒充，因为私钥是不可泄露的，如果公钥能正常解密出私钥加密的内容，就能证明这个消息是来源于持有私钥身份的人发送的。

这个常用在哪里呢？即摘要算法。摘要算法是根据文件内容生成 hash 值，但是中间人可能会连带文件内容和 hash 一起篡改。为了保证 hash 值是可信的，就需要**确定 hash 的来源一定是服务端**，而不是某个攻击的中间人。
我们使用私钥对内容 hash 值加密，如果这个内容可以被公钥解密，那就可以认为这个数据是来自服务端的，因此可以确保 hash 值的有效性。这种就是常说的**数字签名算法**。数字签名算法加密的对象就正是摘要算法计算出的 hash。

![](https://pic.imgdb.cn/item/64155029a682492fcc0507d9.jpg)

##### 摘要算法

摘要算法，简单来说就是客户端发送明文数据之前会通过摘要算法算出明文的“指纹”，然后服务器需要进行比较客户端携带的和自己存储的，如果一致就是安全的

为了保证传输的内容不被篡改，我们需要对内容计算出一个「指纹」，然后同内容一起传输给对方。
对方收到后，先是对内容也计算出一个「指纹」，然后跟发送方发送的「指纹」做一个比较，如果「指纹」相同，说明内容没有被篡改，否则就可以判断出内容被篡改了。
那么，在计算机里会用摘要算法（哈希函数）来计算出内容的哈希值，也就是内容的「指纹」，这个哈希值是唯一的，且无法通过哈希值推导出内容。

当然，单纯的 hash 值有可能导致数据和 hash 值都被篡改，因此就还需要数字签名算法来做进一步的保护，参考上面的内容。

##### 数字证书

数字证书的出现主要是为了保证服务端的数据可信。
即，虽然上面两种方式可以杜绝数据的篡改以及数据的拦截读取，但是公钥和私钥的有效性并不一定能保证。

甚至攻击方可以伪造一对公私钥，然后把公钥发给客户端；由于数据是通过攻击方伪造的私钥加密的，因此客户端可以正常使用攻击方发给它的公钥来解析。客户端以为公钥私钥都是来自服务端的，但实际上是来自黑客的。

因此数字证书就是为了防止伪造公私钥。服务端并不会只发送一个公钥，而是发送一个数字证书，证书内包含服务端的一些信息和公钥。客户端会对证书进行校验，只有证书经过了权威机构的校验，这个公钥才是有效的，否则就会被认为服务端不可信。

一个数字证书通常包含了：

- 公钥；
- 持有者信息；
- 证书认证机构（CA）的信息；
- CA 对这份文件的数字签名及使用的算法；
- 证书有效期；

CA 对证书文件的签名主要是为了防止中间人对证书信息的篡改，这种签名算法的验证就和上面的的摘要算法类似。

证书颁发的时候存在一个“证书链”，即证书的验证过程中还存在一个证书信任链的问题，因为我们向 CA 申请的证书一般不是根证书签发的，而是由中间证书签发的。

中间证书可以看做是一个证书的“代理机构”。每次客户端收到一个新的服务端证书后，就会找到证书上的颁发机构，然后逐个向上查找直到根证书为止；然后用根证书的公钥去依次验证中间证书，最后验证服务端证书，并保存下来。

所以这个过程实际是一个递归查询的过程

![](https://pic.imgdb.cn/item/64bd51281ddac507cc4ff10d.jpg)

#### TLS 握手基本流程

TLS 的「握手阶段」涉及四次通信，使用不同的密钥交换算法，TLS 握手流程也会不一样的，现在常用的密钥交换算法有两种：RSA 算法和 ECDHE 算法

概述：

TLS 协议建立的详细流程：

1. ClientHello

首先，由客户端向服务器发起加密通信请求，也就是 ClientHello 请求。
在这一步，客户端主要向服务器发送以下信息：
（1）客户端支持的 TLS 协议版本，如 TLS 1.2 版本。
（2）客户端生产的随机数（Client Random），后面用于生成「会话秘钥」条件之一。
（3）客户端支持的密码套件列表，如 RSA 加密算法。

2. SeverHello

服务器收到客户端请求后，向客户端发出响应，也就是 SeverHello。服务器回应的内容有如下内容：
（1）确认 TLS 协议版本，如果浏览器不支持，则关闭加密通信。
（2）服务器生产的随机数（Server Random），也是后面用于生产「会话秘钥」条件之一。
（3）确认的密码套件列表，如 RSA 加密算法。
（4）服务器的数字证书。

3. 客户端回应

客户端收到服务器的回应之后，首先通过浏览器或者操作系统中的 CA 公钥，确认服务器的数字证书的真实性。
如果证书没有问题，客户端会从数字证书中取出服务器的公钥，然后使用它加密报文，向服务器发送如下信息：
（1）一个随机数（pre-master key）。该随机数会被服务器公钥加密。
（2）加密通信算法改变通知，表示随后的信息都将用「会话秘钥」加密通信。
（3）客户端握手结束通知，表示客户端的握手阶段已经结束。这一项同时把之前所有内容的发生的数据做个摘要，用来供服务端校验。
上面第一项的随机数是整个握手阶段的第三个随机数，会发给服务端，所以这个随机数客户端和服务端都是一样的。
服务器和客户端有了这三个随机数（Client Random、Server Random、pre-master key），接着就用双方协商的加密算法，各自生成本次通信的「会话秘钥」。

4. 服务器的最后回应

服务器收到客户端的第三个随机数（pre-master key）之后，通过协商的加密算法，计算出本次通信的「会话秘钥」。
然后，向客户端发送最后的信息：
（1）加密通信算法改变通知，表示随后的信息都将用「会话秘钥」加密通信。
（2）服务器握手结束通知，表示服务器的握手阶段已经结束。这一项同时把之前所有内容的发生的数据做个摘要，用来供客户端校验。
至此，整个 TLS 的握手阶段全部结束。接下来，客户端与服务器进入加密通信，就完全是使用普通的 HTTP 协议，只不过用「会话秘钥」加密内容。

概述：
![](https://pic.imgdb.cn/item/623415d65baa1a80abf041be.jpg)

具体参数传递：
![](https://pic.imgdb.cn/item/627b9fda0947543129f65d89.jpg)

#### RSA 和 ECHDE

在 TLS 握手中，ECDHE 和 RSA 是两种常用的密钥交换算法，它们有以下的差别：

1. 密钥长度（更短）：RSA 密钥长度通常为 2048 或 4096 位，而 ECDHE 密钥长度通常为 256 位。由于 ECDHE 使用的是椭圆曲线，所以它可以提供与 RSA 相同的安全性，同时具有更短的密钥长度，从而可以提高加密和解密的效率。
2. 前向保密性：ECDHE 具有前向保密性，这意味着即使攻击者截获了密文和密钥，也无法使用它们来破解以前的通信。RSA 则不具备前向保密性，因为它使用相同的密钥进行加密和解密。
3. 密钥交换过程的安全性：ECDHE 在密钥交换过程中使用的是临时密钥，这些密钥只在该次通信中使用，并且不会在以后的通信中使用。这样可以避免密钥被攻击者截获后进行破解。RSA 则使用相同的密钥进行加密和解密，因此在密钥交换过程中的安全性相对较低。

#### SSL/TLS

SSL 和 TLS 协议可以为通信双方提供识别和认证通道，从而保证通信的机密性和数据完整性。TLS 协议是从 SSL 协议演变而来的，不过这两种协议并不兼容，SSL 已经逐渐被 TLS 取代，所以下文就以 TLS 指代安全层。
TLS 握手是启动 HTTPS 通信的过程，类似于 TCP 建立连接时的三次握手。 在 TLS 握手的过程中，通信双方交换消息以相互验证，相互确认，并确立它们所要使用的加密算法以及会话密钥 (用于对称加密的密钥)。可以说，TLS 握手是 HTTPS 通信的基础部分。

TLS 握手的主要过程其实就是 HTTPS 建立连接的基本流程，主要工作有：

- 商定双方通信所使用的的 TLS 版本 (例如 TLS1.0, 1.2, 1.3 等等)；
- 确定双方所要使用的密码组合；
- 客户端通过服务器的公钥和数字证书 (上篇文章已有介绍)上的数字签名验证服务端的身份；
- 生成会话密钥，该密钥将用于握手结束后的对称加密。

#### TLS 版本

TLS 主要有两个大版本，分别是 TLS1.2 及以前，和 TLS1.3

TLS1.2 及以前和 SSL 的握手方式很像，都是四次握手，也就是上面说的握手方式。TLS1.3 的握手方式发生了改变，完成 TLS 握手只要 1 RTT，而且安全性更高。

![](https://pic.imgdb.cn/item/64bd4c3d1ddac507cc414c48.jpg)

TLS 1.3 把 Hello 和公钥交换这两个消息合并成了一个消息，于是这样就减少到只需 1 RTT 就能完成 TLS 握手。

减少的核心原因是 ECDHE 算法，该算法由于支持「False Start」，它是“抢跑”的意思，客户端可以在 TLS 协议的第 3 次握手后，第 4 次握手前，发送加密的应用数据，以此将 TLS 握手的消息往返由 2 RTT 减少到 1 RTT。

客户端在 Client Hello 消息里带上了支持的椭圆曲线，以及这些椭圆曲线对应的公钥。

服务端收到后，选定一个椭圆曲线等参数，然后返回消息时，带上服务端这边的公钥。经过这 1 个 RTT，双方手上已经有生成会话密钥的材料了，于是客户端计算出会话密钥，就可以进行应用数据的加密传输了。

可以简单总结为，TLS1.3 采用了新的 ECDHE 算法，可以在一个 RTT 内就完成加密数据的传入，并且不再需要三个随机数来生成秘钥，因此也就不用 2 个 RTT 了。

### HTTP 版本

#### HTTP0.9

HTTP/0.9 是于 1991 年提出的，主要用于学术交流，需求很简单——用来在网络之间传递 HTML 超文本的内容。

HTTP/0.9 的实现有以下三个特点：

- 第一个是只有一个请求行，并没有 HTTP 请求头和请求体，因为只需要一个请求行就可以完整表达客户端的需求了。也就是说它只能发送最简单的 GET 请求，而且只能请求到对应的 HTML 文档的字节流形式。
- 第二个是服务器也没有返回头信息，这是因为服务器端并不需要告诉客户端太多信息，只需要返回数据就可以了。
- 第三个是返回的文件内容是以 ASCII 字符流来传输的，因为都是 HTML 格式的文件，所以使用 ASCII 字节码来传输是最合适的。

#### HTTP1.0

相对于 HTTP0.9，1.0 增加了一些功能，包括：

- 请求头和响应头
- 除字节流之外的其他类型数据的传输
- 状态码
- 基本的缓存功能，比如`If-Modified-Since`、`Expires`等缓存头；这些在 http1.1 中得到更新替换

HTTP 1.0 规定浏览器与服务器只保持短暂的连接，浏览器的每次请求都需要与服务器建立一个 TCP 连接，服务器完成请求处理后立即断开 TCP 连接，服务器不跟踪每个客户也不记录过去的请求，类似于之前说过的非持续连接的 HTTP 请求。

http1.0 最大的两个问题是

- 连接无法复用，会导致每次请求都经历三次握手和慢启动，即使带宽很大、文件很小，依然要耗费不少时间；
- head of line blocking（队头阻塞）：致带宽无法被充分利用，以及后续健康请求被阻塞。

> 队头阻塞：
> 请求在客户端会形成队列，由于每次只能发送一个报文，并且在接受响应之前不会再次发送，因此队列里排在最前面的请求会被最优先处理。如果队首的请求因为处理的太慢耽误了时间，那么队列里后面的所有请求也不得不跟着一起等待，结果就是其他的请求承担了不应有的时间成本，造成了队头堵塞的现象。

#### HTTP1.1

HTTP 1.1 支持持久连接，在一个 TCP 连接上可以传送多个 HTTP 请求和响应，减少了建立和关闭连接的消耗和延迟。HTTP1.1 请求头中的`Connection`就是规定如何长连接的。
HTTP 1.1 通过增加更多的请求头和响应头来改进和扩充 HTTP 1.0 的功能。

HTTP1.1 和 HTTP1.0 主要区别有：

- 长连接，http1.0 默认使用非持久连接，而 http1.1 默认使用持久连接。长连接依然是一收一发的形式，并没有修复原有的队头阻塞问题
- 管线化（pipeline）：客户端可以在一个连接上发送多个请求，服务器按照请求的顺序依次响应，这样可以避免等待响应的时间，提高传输速度。客户端可以通过在请求头中添加"Connection: Pipeline"来请求管线化连接。
- 多连接：又被称为流水线，允许客户端和服务端建多个并发连接，减少队头阻塞。并发连接是建立多个 TCP 连接，在每个连接上执行一个 HTTP 的收发过程，和 http2 的多路复用不一样。

> Chrome 在同一时间内最多只能同时建立 6 个与**同一域名**的 TCP 连接，这个限制被称为"同源并发连接限制"，它是浏览器为了避免网络拥塞和提高资源利用率而设置的。同源并发连接限制是 Chrome 等现代浏览器的共同特点，不过有几个特例：
>
> - http2、3 版本不存在该限制
> - https 不受该限制

- 支持请求部分资源和断点续传，在 http1.0 中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，http1.1 则在请求头引入了 Range 头域，它允许只请求资源的某个部分，即返回码是 206（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。
- 缓存控制头增加，在 http1.0 中主要使用 header 里的 `If-Modified-Since`、`Expires` 来做为缓存判断的标准，http1.1 则引入了更多的缓存控制策略，例如`Etag`、`If-Unmodified-Since`、`If-Match`、`If-None-Match` 等更多可供选择的缓存头来控制缓存策略。
- HOST 字段的添加：http1.1 中新增了 host 字段，用来指定服务器的域名。http1.0 中认为每台服务器都绑定一个唯一的 IP 地址，因此，请求消息中的 URL 并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机，并且它们共享一个 IP 地址。因此有了 host 字段，这样就可以将请求发往到同一台服务器上的不同网站。
- 新增几个请求方法：http1.1 相对于 http1.0 还新增了很多请求方法，如 PUT、HEAD、OPTIONS 等。

**注意**：HTTP1.1也有队头阻塞问题，但和http1.0的不同。
http1.1支持请求的并发，也就是可以一次性发送多个请求，而在发送过程中不需先等待服务器的回应。
但是这个“一次性多个”是有数量限制的。同一批内请求发送之后，需要等待收到响应才会发下一批。所以http1.1的队头阻塞是按批的，而http1.0则是按个的，相比之下1.1稍微有些优化。

#### HTTP2.0

http1.1 的一个核心问题是其对带宽的利用率太低。主要原因是频繁触发 TCP 的慢启动而慢启动过程对带宽利用率太低，并且队头阻塞问题并没有从根本上获得解决。

![](https://pic.imgdb.cn/item/627ba1480947543129fa0dc1.jpg)

HTTP2.0 相对于 HTTP1.1 的变化：

- 永久连接，没有.1 中的 keep-alive。在整个传输过程中，始终有且只有一个 TCP 连接，在一个 TCP 连接上完成所有的请求响应过程，而不需要建立多个 TCP 连接。
- 二进制协议：HTTP/2 采用二进制格式传输数据，而非 HTTP 1.x 的文本格式；二进制是整个报文都是二进制，包括头部和实体，统称为"帧"
- 多路复用：在一个连接里，客户端和服务器都可以同时发送多个请求或回应，而且不用按照顺序一一发送
- 优先级机制：每个请求都可以带一个优先值，0 表示最高优先级， 数值越大优先级越低，这样可以自行选择处理帧的方式。优先级和多路复用都解决了 HTTP 的老问题“`队头阻塞`”
- 服务器推送：服务器可以主动向用户**推送** js、css，而不是只能拿通过解析 html 获得
  > server push 和 websocket 的区别：
  >
  > - HTTP2 Server Push，一般用以服务器根据解析 index.html 同时推送 图片/JS/CSS 等资源，而免了服务器发送多次请求
  > - websocket，用以服务器与客户端手动编写代码去推送进行数据通信
- 头部压缩：
  - 头信息使用 HPACK 算法进行压缩。

> HPACK 算法主要包含三个组成部分：
>
> - 静态字典：高频出现在头部的字符串和字段建立了一张静态表，以 index 对应具体的头部的 key。比如 server 头的 index 就是 56，二进制表示为 1110110
> - Huffman 编码（压缩算法）：字典只能得出头部的 key，而 value 则通过哈夫曼编码的形式压缩。Huffman 编码的原理是将高频出现的信息用「较短」的编码表示，从而缩减字符串长度。HTTP/2 根据出现频率将 ASCII 码编码为了 Huffman 编码表，相当于 value 的值用哈夫曼编码表示。
> - 动态字典：静态表只包含了 61 种高频出现在头部的字符串，不在静态表范围内的头部字符串就要自行构建动态表，它的 Index 从 62 起步，会在编码解码的时候随时更新。

---

HTTP/2 的多路复用是为了解决 HTTP1.1 的性能问题。
在 HTTP/2 中，有两个非常重要的概念，分别是帧（frame）和流（stream）。

![](https://pic.imgdb.cn/item/641576c8a682492fcc4fc6f3.jpg)

- 帧代表着最小的数据单位。帧包含一个用于标识的 ID，客户端和服务端都会根据 ID 将不同的帧合并成一个完整的请求
- 通信双方都可以给对方发送二进制帧，这种二进制帧的**双向传输的序列**，也叫做流(Stream)。同域名下所有通信都在单个连接上完成，该连接可以承载任意数量的双向数据流。每个数据流都以消息的形式发送，而消息又由一个或多个帧组成。多个帧之间可以乱序发送，根据帧首部的流标识可以重新组装。

多路复用，就是**在一个 TCP 连接中可以存在多条流**。换句话说，也就是**可以发送多个请求**，对端可以通过**帧中的标识知道属于哪个请求**。通过这个技术，可以避免 HTTP 旧版本中的队头阻塞问题，极大的提高传输性能。

通过多路复用，HTTP2 的请求和接收过程改变为如下：

1. 首先，浏览器准备好请求数据，包括了请求行、请求头等信息
2. 这些数据经过二进制分帧层处理之后，会被转换为一个个带有请求 ID 编号的帧，通过协议栈将这些帧发送给服务器。
3. 服务器接收到所有帧之后，会将所有相同 ID 的帧合并为一条完整的请求信息。
4. 然后服务器处理该条请求，并将处理的响应行、响应头和响应体分别发送至二进制分帧层。
5. 同样，二进制分帧层会将这些响应数据转换为一个个带有请求 ID 编号的帧，经过协议栈发送给浏览器。

浏览器接收到响应帧之后，会根据 ID 编号将帧的数据提交给对应的请求

> 二进制帧
> ![](https://pic.imgdb.cn/item/627ba15b0947543129fa3ad7.jpg)
> 每个帧分为帧头和帧体。先是三个字节的帧长度，这个长度表示的是帧体的长度。
> 然后是帧类型，大概可以分为数据帧和控制帧两种。数据帧用来存放 HTTP 报文，控制帧用来管理流的传输。

#### HTTP 3.0

http3 的出现解决了以前版本的一些问题。比如：

1. http2，由于只使用一条 tcp 连接，tcp 需要保证字节流连续才能交付给上层，因此 tcp 可能会造成队头阻塞问题
2. tcp 连接的不稳定性，虽然数据传输可靠，但可能有各种重传，导致效率很低。

![](https://pic.imgdb.cn/item/641577bba682492fcc51a800.jpg)

HTTP3.0 的改变主要体现在它不再基于 tcp，而是采用 udp 作为传输层协议。具体来说是基于 udp 的 QUIC 协议。

![](https://pic.imgdb.cn/item/63eb5322f144a0100775627e.jpg)

QUIC 可以看做是在 UDP 的基础上建立了一些功能，包括：

1. 无队头阻塞。

QUIC 协议也有类似 HTTP/2 Stream 与多路复用的概念，也是可以在同一条连接上并发传输多个 Stream，Stream 可以认为就是一条 HTTP 请求。
QUIC 有自己的一套机制可以保证传输的可靠性的。当某个流发生丢包时，只会阻塞这个流，其他流不会受到影响，因此不存在队头阻塞问题。这与 HTTP/2 不同，HTTP/2 只要某个流中的数据包丢失了，其他流也会因此受影响。
所以，QUIC 连接上的多个 Stream 之间并没有依赖，都是独立的，某个流发生丢包了，只会影响该流，其他流不受影响。

![](https://pic.imgdb.cn/item/64157819a682492fcc528cc2.jpg)

2. 建立连接更快

![](https://pic.imgdb.cn/item/64157d09a682492fcc5df4f7.jpg)

建立连接更快体现在两个方面

- QUIC 协议和 TLS 握手同步进行，省去了单独进行 TLS 握手的时间。并且由于是基于 UDP，也没有 TCP 的三次握手。
- 0-RTT 和 2-RTT：QUIC 的初始握手过程只有两次，即 2-RTT；而当第一次握手完成后，后续连接建立的时候就会顺带发送数据包，相当于没有建立连接时间的花费，即 0-RTT。

QUIC 的握手过程其实是在确认连接双方的连接 ID，连接 ID 标识双方的连接。
连接 ID 的作用主要有：

1. 标识双方连接。TCP 连接握手使用的是 4 元组（源 IP 地址、源端口、目的 IP 地址、目的端口）来标识一个连接，quic 只采用一个连接 id
2. 起到加密作用。通过连接 ID，可以将数据传输与特定的连接和会话绑定。注意这是传输层层面的加密，和 tls 不冲突
3. 便于连接迁移。当连接双方的一方之中的 ip 等发生变化时，可以通过链接 id 复用原来的连接
4. 减少握手时长。即 0-RTT 和 2-RTT
5. 连接迁移

每个连接过程都有一组连接标识符，或称连接 ID，该 ID 用以识别该连接。每个端点各自选择连接 ID。每个端点选择其对等方使用的连接 ID。这些连接 ID 的主要功能是确保较低协议层（UDP、IP 及其底层协议）的地址更改不会导致 QUIC 连接的数据包传递到错误的端点。
通常利用连接 ID，可以在 IP 地址和网络接口迁移的情况下得到保持，比如移动设备的网络变化后，导致 IP 地址变化了，只要仍保有上下文信息（比如连接 ID、TLS 密钥等），就可以“无缝”地复用原连接

---

quic 也有一些缺点：

1. 支持少。
2. 复杂度高，有一定的转换成本，并且可能造成某些不易察觉的漏洞
3. 消耗资源更多，相对于 tcp 来说消耗的内存和处理能力更多，可能对移动设备不友好

### HTTP 代理

代理实际上是一个服务器，假设在服务器和客户端之间，作为一个中间人的身份；请求并不是直接发送给服务器，而是先经过代理服务器进行一些操作，然后再发给服务器；响应也是同理。
代理服务器主要功能有：

- **负载均衡**。代理服务器可以拿到这个请求之后，可以通过特定的算法分发给不同的源服务器，让各台源服务器的负载尽量平均。
- **保障安全**。利用心跳机制监控后台的服务器，一旦发现故障机就将其踢出集群。并且对于上下行的数据进行过滤，对非法 IP 限流，这些都是代理服务器的工作。
- **缓存代理**。将内容缓存到代理服务器，使得客户端可以直接从代理服务器获得，也叫做 Web 缓存；从计算机网络的角度，这样的代理通常设置在区域 ISP 中。实际上 HTTP 缓存的实现就是依赖代理服务器，或者叫做缓存器。

#### 代理字段

- `Via`：记录代理服务器。每经过一个代理服务器就向这个字段添加当前代理的身份，添加的顺序就是经过代理服务器的顺序。

```
Via: proxy_server1, proxy_server2
```

- `X-Forwarded-For`：字面意思就是为谁转发, 它记录的是请求方的 IP 地址
- `X-Real-IP`：是一种获取用户真实 IP 的字段，不管中间经过多少代理，这个字段始终记录最初的客户端的 IP
- `Cache-Control`中的`s-maxage`字段：限定了缓存在代理服务器中可以存放多久，即相当于代理服务器中的`max-age`。

#### 代理缓存

对于 HTTP 缓存来说，如果每次客户端缓存失效都要到源服务器获取，那给源服务器的压力是很大的。
由此引入了缓存代理的机制。让代理服务器接管一部分的服务端 HTTP 缓存，客户端缓存过期后就近到代理缓存中获取，代理缓存过期了才请求源服务器，这样流量巨大的时候能明显降低源服务器的压力。
缓存代理的控制分为两部分，一部分是源服务器端的控制，一部分是客户端的控制。

1. **源服务器控制**：主要通过`Cache-Contro`字段控制。

- `private`或者`public`表示是否允许代理服务器缓存，前者禁止，后者为允许。
- `must-revalidate`的意思是客户端缓存过期就去源服务器获取，而`proxy-revalidate`则表示代理服务器的缓存过期后到源服务器获取。
- `s-maxage`:限定了缓存在代理服务器中可以存放多久，即相当于代理服务器中的`max-age`。

2. **客户端控制**：通过向请求头中添加几个字段：

- `max-stale` 和 `min-fresh`：`max-stale`表示客户端到代理服务器上拿缓存的时候，即使代理缓存过期了也不要紧，只要过期时间在多长时间之内，还是可以从代理中获取的。`min-fresh`正相反，要求至少提前多长时间。

### HTTP 缓存

#### 缓存的分类

缓存根据位置和使用的对象可以分成两类：

- web 缓存、代理缓存、共用缓存：以被多个用户使用，一般是架设在 ISP 的代理服务器，作为用户主机和外部的中介
- 浏览器缓存：即通过请求和响应头设置的缓存，一般存储在用户主机本地，只能服务于单独用户，并且常见的 http 缓存只能缓存 get 请求响应的资源；
  浏览器缓存可以根据缓存存放的**位置**分为 3 类
  - `Service Worker`：<a href="https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API/Using_Service_Workers">MDN-Service Worker</a>具体介绍了这个 api。它可以被理解为是一个介于浏览器和网络请求之间的拦截器，像 webworker 一样运行在主线程之外，其中的一个主要功能就是控制缓存的存储和读取，可以完成**离线缓存**、消息推送、网络代理等功能；若没有在 Service Worker 命中缓存，会根据缓存查找优先级去查找数据。
  - `Memory Cache`：内存缓存，效率最高最快，但一旦关闭 Tab 页面就不再保存。通常情况下打开一个页面的缓存会首先保存在 Memory Cache 中，如果通过 F5 刷新页面，就会从内存中直接获取缓存；关闭页面后消失，但会保留一份副本在 Disk Cache 中
  - `Disk Cache`：存储在硬盘中的缓存，覆盖面基本是最大的，会根据 HTTP Herder 中的字段判断哪些资源需要缓存，哪些资源可以不请求直接使用，哪些资源已经过期需要重新请求。关闭页面后将从这里获取，但不一定是全部读取，还要根据缓存策略、字段等信息判断是否需要缓存还是重新请求。
  - `Push Cache`：HTTP/2 的缓存，以上三种都没有才会使用，并且生命周期和 session 一致

上面的四个缓存，针对用户行为的不同有不同的访问优先级：

- 用户直接输入 url 或通过历史记录访问：检查 disk cache 中的缓存是否存在、是否符合强缓存要求；否则则发送请求
- 用户 F5 刷新：优先使用 Memory Cache，没有再取 Disk Cache
- 用户 Ctrl+F5 刷新：不选择缓存，直接重新请求

#### http 缓存

> HTTP 缓存主要是通过请求和响应报文头中的对应 Header 信息，来控制缓存的策略。响应头中相关字段为 `Expires`、`Cache-Control`、`Last-Modified`、`Etag`。
> 缓存两种类型: **强缓存** / **协商缓存**
> 从物理上，http 缓存依靠代理服务器（又叫 Web 缓存器）存储缓存信息；Web 缓存器相当于一个中转站：
>
> - 当客户端请求时，**将请求定向至缓存器**，会先经过缓存器并检查是否存有副本，如果有会直接返回；如果没有则会向服务器发送请求
> - 服务器响应请求给缓存器，缓存器将文件在本地保存，返回给客户端；当下一次客户端请求时会给予之前的副本而不是直接请求服务器

缓存类型有两种：

- 强缓存: 如果缓存在缓存器缓存数据库中存在,就强制使用缓存而不是请求服务器; 如果没有缓存就请求服务器并取得缓存,下次使用缓存
  ![](https://pic.imgdb.cn/item/623439926eeb7459a2832d75.jpg)
- 协商缓存/对比缓存: 无论有无缓存都会向服务器请求, 比较判断是否可以使用缓存。浏览器再次请求数据时，浏览器将备份的缓存标识发送给服务器，服务器根据缓存标识进行判断，判断成功后，返回 304 状态码，通知客户端比较成功，可以使用缓存数据。第二次的请求大小会比第一次小很多, 一般不包含静态资源缓存, 因此要快很多
  ![](https://pic.imgdb.cn/item/623439f15baa1a80ab0c3519.jpg)

> 不存在缓存数据或者未过期时会采用强缓存，获取之后在进行缓存；  
> 如果缓存过期，会采用协商缓存检查本地缓存是否还有效  
> 允许使用浏览器本地缓存的状态码是`304`，表示资源未改变，可以使用缓存
> 判断流程：![](https://pic.imgdb.cn/item/62343dc85baa1a80ab0e7589.jpg)
>
> 这张图从浏览器角度看更清晰：![](https://pic.imgdb.cn/item/6237f03627f86abb2af51f05.jpg)

#### 强缓存

在没有缓存数据的时候，浏览器向服务器请求数据时，服务器会将数据和缓存规则一并返回，缓存规则信息包含在响应 header 中。
在缓存数据未失效的情况下，浏览器会直接使用缓存数据；浏览器判断数据失效的依据是响应头部的缓存相关字段；如果失效，则会再次向服务器请求。

缓存相关字段如下：

- `Expires`：服务端返回的到期时间，即下一次请求时，请求时间小于服务端返回的到期时间，直接使用缓存数据。在 HTTP 1.1 的版本，Expires 被 Cache-Control 替代。替代的主要原因是Expires字段会受到客户端本地时间的影响。浏览器会以本地时间为标准检查是否过期，如果修改了本地时间，就会导致误认为过期的问题。
- `Cache-Control`：常见的取值有 private、public、no-cache、max-age，no-store，默认为 private。
  - `max-age=xxx`：用来设置资源（representations）可以被缓存多长时间，单位为秒；
  - `public`：指示响应可被任何缓存区缓存；
  - `private`：只能针对个人用户，而不能被代理服务器缓存；对前端来说这两个没什么太多区别
  - `no-cache`：强制客户端直接向服务器发送请求,也就是说每次请求都必须向服务器发送。服务器接收到请求，然后判断资源是否变更，是则返回新内容，否则返回 304，未变更。
  - `no-store`：禁止一切缓存（这个才是响应不被缓存的意思）。
  - `must-revalidate`：类似 no-cache，强制重新请求
  - `s-maxage`：限定了缓存在代理服务器中可以存放多久，和限制客户端缓存时间的 max-age 并不冲突。`s-maxage`出现时将无视 expires 字段。

#### 协商缓存

浏览器第一次请求数据时，服务器会将缓存标识与数据一起返回给客户端，客户端将二者备份至缓存数据库中。
再次请求数据时，客户端将备份的缓存标识发送给服务器，服务器根据缓存标识进行判断，判断成功后，返回 304 状态码，通知客户端比较成功，可以使用缓存数据。

> 协商缓存的判断字段分为两组，第一组依据修改时间判断是否过期、是否需要重传；第二组通过唯一标识符判断；  
> 两组内部的值其实相同，只是组内的前者是响应头内的，后者是请求头内的。  
> **第二组**的优先级高一些；

- `Last-Modified` ：最后修改时间
- `If-Modified-Since`：再次请求服务器时，通过此字段通知服务器上次请求时，服务器返回的资源最后修改时间。

服务器收到请求后发现有头`If-Modified-Since` 则与被请求资源的最后修改时间进行比对。

- 若资源的最后修改时间大于`If-Modified-Since`，说明资源又被改动过，则响应整片资源内容，返回状态码 200；
- 若资源的最后修改时间小于或等于`If-Modified-Since`，说明资源无新修改，则响应 HTTP 304，告知浏览器继续使用所保存的 cache。

> `Last-Modified`的问题：
>
> - 因为它是以秒计时的。如果在较短事件完成修改，可能会导致即使修改了但该值并没有改变。
> - 如果只是打开但没有修改也会更新该值。为了解决问题，更好的方式就是 Etag。
> - 有些服务器不能精确获取文件的最后修改时间。
>
> Etag 也不是完美的：
>
> - ETag 的值通常是由服务器生成的，而不是由资源本身确定的，因此在不同的服务器上生成的 ETag 可能会不同，导致在负载均衡等情况下无法正确地进行缓存。
> - 对于某些反向代理服务器或 CDN 等，可能会不支持 ETag 的协商缓存，从而导致缓存失效。
> - 在某些情况下，ETag 的粒度可能过细，导致客户端需要不断地请求新的资源，从而影响性能。
>
> Etag 的生成方式：
>
> - 类似 contenthash，根据文件内容生成
> - 可能把文件的修改时间和文件大小组合起来
> - 资源版本号

---

- `Etag`：服务器对资源的唯一标识
- `If-None-Match`：再次请求服务器时，通过此字段通知服务器客户段缓存数据的唯一标识。

服务器收到请求后发现有头`If-None-Match` 则与被请求资源的唯一标识（Etag）进行比对，

- 不同，说明资源又被改动过，则响应整片资源内容，返回状态码 200；
- 相同，说明资源无新修改，则响应 HTTP 304，告知浏览器继续使用所保存的 cache。

#### 条件 GET

浏览器通过条件 GET（也就是报文中首部行的`If-Modified-Since`）**判断缓存器上的副本是否是最新的**，具体步骤如下：

1. 代理缓存器第一次向服务器发送请求，服务器返回的报文首部行中有一个`Last-Modified`字段被一并保存；
2. 一段时间后当客户端请求时，代理缓存器会发送一个 GET 请求，其中首部行带有`If-Modified-Since`字段，并且值就是之前的`Last-Modified`字段，表示上次请求的时间

```
GET / HTTP/1.1
HOST: www.xxx.com
If-Modified-Since: Wed, 9 Sep 2020 09:23:24
```

3. 服务器对条件 GET 做出响应：

- 状态码 304，说明可以继续使用缓存
- 其他情况，发送新的响应体，重新缓存

4. 将缓存响应给客户端

### HTTP 其他功能

#### 大文件传输（分段传输）

HTTP 针对这一场景，采取了范围请求的解决方案，允许客户端仅仅请求一个资源的一部分。

要支持这个功能，就必须加上这样一个响应头:

```
Accept-Ranges: bytes
```

而对于客户端而言，它需要指定请求哪一部分，通过 Range 这个请求头字段确定，格式为 bytes=x-y。
比如：

```
// 单段数据
Range: bytes=0-9
// 多段数据
Range: bytes=0-9, 30-39
```

> Range 的书写格式:
> 0-499 表示从开始到第 499 个字节。
> 500- 表示从第 500 字节到文件终点。
> -100 表示文件的最后 100 个字节。
> 服务器收到请求之后，首先验证范围是否合法，如果越界了那么返回`416`错误码;否则读取相应片段，返回`206`状态码。

响应对于单端数据的处理和多段数据不同。

1. 单段数据：核心是`Content-Range`字段，表示响应的返回部分和总资源的大小。

```
HTTP/1.1 206 Partial Content
Content-Length: 10
Accept-Ranges: bytes
Content-Range: bytes 0-9/100

i am xxxxx
```

2. 多段数据：
1. `Content-Type: multipart/byteranges; boundary=00000010101`这个字段表示响应体是一个多段数据，用 boundary 值为 00000010101 来分割。
1. 每一段都会先用 boundary 分割，然后展示本段的类型（`Content-Type`）和范围（`Content-Range`），再在一个空行之后放入正常的响应体即可。
1. 在最后的分隔末尾添上--表示结束。

```
HTTP/1.1 206 Partial Content
Content-Type: multipart/byteranges; boundary=00000010101
Content-Length: 189
Connection: keep-alive


--00000010101
Content-Type: text/plain
Content-Range: bytes 0-9/96

i am xxxxx
--00000010101
Content-Type: text/plain
Content-Range: bytes 20-29/96

eex jspy e
--00000010101--
```

## DNS

DNS 是域名系统 (Domain Name System) 的缩写，提供的是一种主机名到 IP 地址的转换服务，就是我们常说的域名系统。它是一个由分层的 DNS 服务器组成的分布式数据库，是定义了主机如何查询这个分布式数据库的方式的应用层协议。能够使人更方便的访问互联网，而不用去记住能够被机器直接读取的 IP 数串。

在定义上，DNS 既是一个由分层的 DNS 服务器实现的 DNS**数据库**，又是一个提供这项功能的应用层**协议**。

作用：将域名解析为 IP 地址，客户端向 DNS 服务器（DNS 服务器有自己的 IP 地址）发送域名查询请求，DNS 服务器告知客户机 Web 服务器的 IP 地址。

> DNS 用作域名解析时运行在 UDP 上，53 号端口
> DNS 在区域传输时使用 TCP，即根服务器、权威服务器等之间的数据交换

### 查询过程

DNS 采用分布式层次数据库的方式，使 IP 地址--域名的映射分布在所有 DNS 服务器上。这些服务器大致分为三类：

- 根 DNS 服务器：提供顶级域（TLD）服务器的 IP 地址
- TLD DNS 服务器：顶级域即类似`.com`、`.cn`这样的域名中的顶级域，这些顶级域提供权威 DNS 服务器的 IP
- 权威 DNS 服务器：真正存储映射的地方，维护一个区域的映射。
- 本地 DNS 服务器：虽然不属于分布式的一部分，但是和用户直接相连，用于向其他服务器发送；同时也带有 DHCP 服务器，供用户获取自己的 IP 地址。

DNS 服务器解析域名的过程：

1. 首先会在**浏览器的缓存**中查找对应的 IP 地址，如果查找到直接返回，若找不到继续下一步
1. 将请求发送给本地 DNS 服务器，在**本地 DNS 服务器缓存**中查询，如果查找到，就直接将查找结果返回，若找不到继续下一步
1. 本地 DNS 服务器向**根域名服务器**发送请求，根域名服务器会返回一个所查询域的顶级域名服务器地址
1. 本地 DNS 服务器向**顶级域名服务器**发送请求，接受请求的服务器查询自己的缓存，如果有记录，就返回查询结果，如果没有就返回相关的下一级的权威域名服务器的地址
1. 本地 DNS 服务器向**权威域名服务器**发送请求，域名服务器返回对应的结果
1. 本地 DNS 服务器将返回结果保存在缓存中，便于下次使用
1. 本地 DNS 服务器将返回结果返回给浏览器

上面这种方式，本地 DNS 分别向三个层次发送请求，这种方式是`迭代查询`。
如果是`本地DNS->根DNS->TLD DNS->权威DNS->原路返回`这样的形式，就是`递归查询`。

## CDN

CDN（内容分发网络）：尽可能避开互联网上有可能影响数据传输速度和稳定性的瓶颈和环节，使内容传输的更快、更稳定。通过在网络各处放置节点服务器所构成的在现有的互联网基础之上的一层智能虚拟网络，CDN 系统能够实时地根据网络流量和各节点的连接、负载状况以及到用户的距离和响应时间等综合信息将用户的请求重新导向离用户最近的服务节点上。
也就是说，cdn 可以尽可能把你连接到最近、最快的服务器

cdn 由一个 dns 服务器和几台缓存服务器组成，dns 服务器用作接入并返回合适的缓存服务器；缓存服务器存储真正的内容，会被分配到不同的用户。

1. 当用户点击网站页面上的内容 URL，经过本地 DNS 系统解析，并不会直接返回对应的 IP 地址，而是返回一个 CDN 服务器的域名（注意是域名不是 ip）
2. 用户向返回的这个 CDN 服务器域名发起请求。
3. CDN 服务器根据用户 IP 地址，以及用户请求的内容 URL，选择一台用户所属区域的区域负载均衡设备，返回给这台设备的 IP 地址
4. 用户通过本地 dns 向该 ip 发起连接。

## WebSocket

### 轮询和长轮询

轮询和长轮询都是为了解决服务端不能主动向客户端推送的问题。

以扫码为例，当登录时登录页面二维码出现之后，前端网页根本不知道用户扫没扫，于是不断去向后端服务器询问，看有没有人扫过这个码。而且是以大概 1 到 2 秒的间隔去不断发出请求，这样可以保证用户在扫码后能在 1 到 2 秒内得到及时的反馈。这种方式就是轮询。

而长轮询，是客户端向服务器发送一个 HTTP 请求，服务器在收到请求后不立即响应，而是将该请求挂起，直到有新的数据可用或超时时间到期才返回响应。

举个例子，客户端发起一个请求，把超时时间设置为 30 秒；服务端收到请求时不立即响应，而是等待其他操作的完成（比如用户扫码）。当其他特定操作完成后，服务端再响应，这时客户端就可以立即完成这个请求（比如用户扫码之后立即跳转）

下面是一个简单的例子：

```js
function longPoll() {
  axios
    .get("/data", {
      cancelToken: new axios.CancelToken(function (cancel) {
        // 将cancel函数存储到变量中，以便在需要时取消请求
        longPollCancel = cancel;
      }),
      // 超时时间30秒
      timeout: 30000,
    })
    .then(function (response) {
      // 处理服务器返回的数据
      // ...
      // 发起下一次长轮询请求，可以不需要
      longPoll();
    })
    .catch(function (error) {
      if (axios.isCancel(error)) {
        // 请求被取消
      } else {
        // 处理错误情况
        // ...
        // 发起下一次长轮询请求
        longPoll();
      }
    });
}

// 启动长轮询
longPoll();
```

### websocket 握手过程

> - TCP 连接的两端，同一时间里，双方都可以主动向对方发送数据。这就是所谓的全双工。
> - HTTP/1.1，同一时间里，客户端和服务器只能有一方主动发数据，这就是所谓的半双工。
>   websocket 就是基于 tcp 的全双工协议

握手过程如下：

1. 客户端发起握手请求：客户端将发送一个相当标准的 HTTP 请求（HTTP 版本必须是 1.1 或更高，方法必须是 GET）：

```
GET /chat HTTP/1.1
Host: example.com:8000
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

在请求头中设置 upgrade 字段用于升级协议。`Sec-WebSocket-Key`将在下面用于服务端的`Sec-WebSocket-Accept`生成；

2. 服务端响应握手请求：当服务器收到握手请求时，它应该发回一个特殊的响应，表明协议将从 HTTP 变为 WebSocket。

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

Sec-WebSocket-Accept 参数需要服务器通过客户端发送的 Sec-WebSocket-Key 计算出来。服务端根据客户端生成的 base64 码，用某个公开的算法变成另一段字符串，放在 HTTP 响应的 Sec-WebSocket-Accept 头里，同时带上 101 状态码，发回给浏览器。

浏览器再用同样的公开算法将base64码转成另一段字符串，如果这段字符串跟服务器传回来的字符串一致，那验证通过。

因此这里的`Sec-WebSocket-`参数实际上是类似于身份校验的功能，双方交换之后就可以确认对方的身份，连接就能顺利进行

就这样经历了一来一回两次 HTTP 握手，WebSocket 就建立完成了，后续双方就可以使用 websocket 的数据格式进行通信了。

websocket 有自己独特的状态码，在关闭连接时可以传递这些状态码，用于表示关闭原因等信息，详见https://zhuanlan.zhihu.com/p/145628937

# 传输层

传输层建立在网络层之上和应用层之下，为运行在不同主机上的不同进程之间提供了逻辑通信。
传输层的主要功能：

1. 多路复用和多路分解：由于网络层只负责主机到主机的传输，具体到进程需要传输层控制。把进程和运输层之间不同的套接字中的数据块收集，并生成报文段的过程叫多路复用（封装数据）；把报文段的数据正确交付到对应的套接字叫做多路分解（解封和交付）
2. 提供可靠数据传输，以及拥塞控制等功能（TCP）

## UDP

UDP（用户数据报协议），和 TCP 协议一样用于处理数据包，是一种无连接的协议。通常适用于允许丢失、但对实时性要求很高的功能。

### UDP 特点

1. **无连接**

UDP 不需要和 TCP 一样在发送数据前进行三次握手建立连接的，想发数据就可以开始发送了。并且也只是数据报文的搬运工，不会对数据报文进行任何拆分和拼接操作。
具体来说就是：

- 在发送端，应用层将数据传递给传输层的 UDP 协议，UDP 只会给数据增加一个 UDP 头标识下是 UDP 协议，然后就传递给网络层了
- 在接收端，网络层将数据传递给传输层，UDP 只去除 IP 报文头就传递给应用层，不会任何拼接操作

2. **传输方式多样**

UDP 不止支持一对一的传输方式，同样支持一对多，多对多，多对一的方式，也就是说 UDP 提供了单播，多播，广播的功能。

3. **面向报文**
   发送方的 UDP 对应用程序交下来的报文，在添加首部后就向下交付 IP 层。UDP 对应用层交下来的报文，既不合并，也不拆分，而是保留这些报文的边界。因此，应用程序必须选择合适大小的报文
4. **不可靠性**
   首先不可靠性体现在**无连接**，通信都不需要建立连接，想发就发，这样的情况肯定不可靠。
   并且收到什么数据就传递什么数据，并且也不会备份数据，发送数据也**不会确定对方是否已经正确接收到数据**。
   再者网络环境时好时坏，但是 UDP 因为**没有拥塞控制**，一直会以恒定的速度发送数据。即使网络条件不好，也不会对发送速率进行调整。
   这样实现的弊端就是在网络条件不好的情况下可能会导致丢包，但是优点也很明显，在某些实时性要求高的场景（比如电话会议）就需要使用 UDP 而不是 TCP。
5. **高效**

UDP 头部包含了以下几个数据：

![image-20221017183928534](https://blog-img-1307852525.cos.ap-chengdu.myqcloud.com/img/image-20221017183928534.png)

- 两个十六位的端口号，分别为源端口（可选字段）和目标端口
- 整个数据报文的长度
- 整个数据报文的检验和（IPv4 可选字段），该字段用于发现头部信息和数据中的错误

> 检验和：将所有的字相加，得到的字如果溢出就回卷（去掉几个开头的位），然后取其反码作为检验和。
> 在验证时用检验和和所有字的和相加，如果不全为 1 就说明有差错。

因此 UDP 的头部开销小，只有 8 字节，相比 TCP 的至少 20 字节要少得多，在传输数据报文时是很高效的。

### 如何基于 UDP 实现 HTTP

TCP 协议有四个方面的缺陷：

- 升级 TCP 的工作很困难；
- TCP 建立连接的延迟；
- TCP 存在队头阻塞问题；
- 网络迁移需要重新建立 TCP 连接；

QUIC 协议基于 UDP，因此了解 QUIC 的实现其实就可以了。

大致总结为：

- quic 分为 packet 和 frame，都有各自的 header。frame header 中包含了 streamid 标识属于哪个流，offset 表示在该流内的偏移量，这两个共同构成了 frame 的归属。
- quic 的 packet number 是自增的
- quic 的 frame 属于一个 stream，每个 stream 有自己的滑动窗口，在自己 stream 内部实现了流量控制和拥塞控制，不会影响其他 stream
- quic 支持 2RTT 的初始化连接和 0RTT 的后续连接，以及网络变化之后的连接复用

#### 可靠数据传输

quic 的可靠数据传输方式主要依赖独特的头部。
以 http3 为例，在 udp 头和 http3 的头部之间还有三个头：

![](https://pic.imgdb.cn/item/64bdf3c51ddac507cc663f00.jpg)

关键的 header 其实就是这两个

1. QUIC Packet Header

![](https://pic.imgdb.cn/item/64bdf44c1ddac507cc676ca9.jpg)

QUIC 也是需要三次握手来建立连接的，主要目的是为了协商连接 ID。协商出连接 ID 后，后续传输时，双方只需要固定住连接 ID，从而实现连接迁移功能。
上图中右边的就是日常传输数据的 Short Packet Header， 不需要传输 Source Connection ID 字段了，只需要传输 Destination Connection ID。

Short Packet Header 中的 Packet Number 是*每个报文独一无二的编号*，它是**严格递增**的，也就是说就算 Packet N 丢失了，重传的 Packet N 的 Packet Number 已经不是 N，而是一个比 N 大的值。

Packet Number 主要对标 tcp 中的 seq，即报文的序列号；在 tcp 中重传事件发生时，重传的是之前丢失的那个报文段的序列号。而在 quic 中由于 Packet Number 单调递增，因此即使重传也不会发送相同的编号。

那这里就会有两个问题

- 为什么要这么做？主要是为了实现乱序接收。比如当数据包 Packet N 丢失后，只要有新的已接收数据包确认，当前窗口就会继续向右滑动，这样就不会因为丢包重传将当前窗口阻塞在原地，从而解决了队头阻塞问题。后面如果要重传的话，把新的包发送为 packet M，再按照新包的方式处理就行。
- 这样做，接收端怎么知道这个是重传的，而不是新的报文段？接收端主要通过下面的 frame 的每一个 frame 内的 streamID 和 offset 字段，确定具体是哪个端。这两个字段相当于是每个帧的一个标识，只要标识对应，那么序列号其实并不重要。

2. QUIC Frame Header

Frame Header 其实是 quic 的帧数据的头。一个 Packet 报文中可以存放多个 QUIC Frame，每个 frame 的头就是 QUIC Frame Header。
frame 的类型有很多，最常见的类型是 stream，也就是常说的“流”的一帧，我们可以把每个 frame 看做是某个流的一帧。一个 Packet 中可能有很多帧

![](https://pic.imgdb.cn/item/64be00c81ddac507cc86a031.jpg)

每个 frame header 结构如下：

![](https://pic.imgdb.cn/item/64be00d31ddac507cc86b9f9.jpg)

主要有三个部分：

- Stream ID 作用：多个并发传输的 HTTP 消息，通过不同的 Stream ID 加以区别，类似于 HTTP2 的 Stream ID，用于区分不同的 stream。
- Offset 作用：类似于 TCP 协议中的 Seq 序号，保证数据的顺序性和可靠性；
- Length 作用：指明了 Frame 数据的长度。

streamid 可以指定这个帧属于哪个 stream，offset 则指定这个帧在整个 stream 内的偏移量。

接收方每次接收的是一个 packet，然后根据 packet number 进行移动滑动窗口等操作；再从 packet 中取出不同的 frame，根据 frame header 中的 streamID 和 offset 确定该数据的归属，将其整合到一个完整的流中。

#### 滑动窗口和流量控制

quic 想要实现应用层协议，也需要实现自己的滑动窗口，不然就无法保证数据传输的有序性，同时也不能做流量控制。

在 tcp 中有个很大的问题，就是如果最靠近滑动窗口左边的那组数据没收到，那么滑动窗口的左边缘就会一直不能移动，直到收到为止；这是 tcp 为了实现数据有效性而做出的妥协。
这种机制就导致了在 http2 中，依然存在队头阻塞的问题。http2 虽然可以让不同流内的帧乱序发送，但这些请求都是在一个 tcp 连接上的。一旦 tcp 发生队头阻塞，http2 也不能避免

quic 避免这种情况的方式是，为每个 stream 都实现独立的滑动窗口。这样即使一个 stream 内因为没收到某个帧而阻塞，也不会导致其他 stream 受影响。

![](https://pic.imgdb.cn/item/64be179d1ddac507ccc7e915.jpg)

quic 做流量控制，依赖的就是每个 stream 的滑动窗口。实际上在 quic 中其实有两个方面的流量控制

- stream 级别：即 stream 内每个 frame 的控制，类似 tcp 的滑动窗口，发送的单位是属于这个 stream 的 frame
- connection 级别：上面说过一个 packet 中可能包含多个分属不同 stream 的 frame，那么以 packet 为发送单位的流量控制就是 connection 级别的。

下面主要说 stream 级别的流量控制。其实大致可以按照 tcp 的方式去理解，但是主要的不同在于

1. 接收窗口的左边界移动取决于接收到的最大偏移字节数，而不是最小的。

![](https://pic.imgdb.cn/item/64be19df1ddac507ccce45e1.jpg)

在 tcp 中，比如某一时刻收到了三个段，分别是 30-40、50-80、80-100，这时如果左边界一开始是 30，那么只会移动到 40，因为 40-50 这段没有接收。这就是最小偏移字节数；
而在 quic 中，左边界将会移动到 100。然后送方收到接收方的窗口更新帧后，发送窗口的右边界也会往右扩展，以此达到窗口滑动的效果。
那么这些没收到的数据怎么办呢？稍后发送方将会认为其超时，将其重新编号后发送。然后接收方收到后根据 streamid 和 offset 确定正确的位置。

2. 如果中途丢失了数据包，导致绿色部分（已经被正常接收和处理）的数据没有超过最大接收窗口的一半，那接收窗口就无法滑动了。这个也是为了保证右边界不会一直增长，要保证窗口内成功处理的数据至少为一般才行。

---

Connection 级别的流量窗口，其接收窗口大小就是各个 Stream 接收窗口大小之和。

![](https://pic.imgdb.cn/item/64be1b831ddac507ccd29f48.jpg)

#### 拥塞控制

QUIC 协议当前默认使用了 TCP 的 Cubic 拥塞控制算法（我们熟知的慢开始、拥塞避免、快重传、快恢复策略），同时也支持 CubicBytes、Reno、RenoBytes、BBR、PCC 等拥塞控制算法，相当于将 TCP 的拥塞控制算法照搬过来了。

不过 quic 的一个好处是，QUIC 是处于应用层的，应用程序层面就能实现不同的拥塞控制算法，不需要操作系统，不需要内核支持。QUIC 可以随浏览器更新，QUIC 的拥塞控制算法就可以有较快的迭代速度。

#### 连接

quic 的连接建立有两个特点

1. 握手过程：quic 也需要在第一次连接时通过握手来确定连接。握手是连接的必须，QUIC 也是需要三次握手来建立连接的，主要目的是为了协商连接 ID。但是 quic 的好处在于它在第一次连接时只需要 1RTT，即一次来回就够。

tcp 连接需要 1RTT 的 tcp 握手和 2RTT 的 TLS 握手，才能开始传输数据。
而 QUIC 内部包含了 TLS，它在自己的帧会携带 TLS 里的“记录”，再加上 QUIC 使用的是 TLS1.3，因此仅需 1 个 RTT 就可以「同时」完成建立连接与密钥协商，甚至在第二次连接的时候，应用数据包可以和 QUIC 握手信息（连接信息 + TLS 信息）一起发送，达到 0-RTT 的效果。

2. 连接迁移：TCP 传输协议的 HTTP 协议，由于是通过四元组（源 IP、源端口、目的 IP、目的端口）确定一条 TCP 连接。而 quic 只会依赖一个连接 id 来确定。客户端和服务器可以各自选择一组 ID 来标记自己，因此即使移动设备的网络变化后，导致 IP 地址变化了，只要仍保有上下文信息（比如连接 ID、TLS 密钥等），就可以“无缝”地复用原连接，消除重连的成本，没有丝毫卡顿感，达到了连接迁移的功能。

## TCP

TCP 的全称是传输控制协议是一种面向连接的、可靠的、基于字节流的传输层通信协议。TCP 是面向连接的、可靠的流协议（流就是指不间断的数据结构）。
TCP 应用场景： 效率要求相对低，但对**准确性**要求相对高的场景。因为传输中需要对数据确认、重发、排序等操作，相比之下效率没有 UDP 高。例如：文件传输（准确高要求高、但是速度可以相对慢）、接受邮件、远程登录

### TCP/UDP 零碎知识点

- TCP 和 UDP 是可以共用同一个端口的。主要原因是其在网络层的实现不同，当 IP 数据包从网络层传上来的时候，如果数据包的协议类型是 TCP，就将数据包传输给 TCP 连接（TCP socket）；如果数据包的协议类型是 UDP，就将数据包传输给 UDP 连接（UDP socket）。因此他们虽然共用同一个端口，但是并不冲突。就像一个高速公路收费站可以同时好几条入口一样。
- TCP 四元组可以唯一的确定一个连接，即源地址、源端口、目的地址、目的端口。其中 ip 地址是保存在 ip 数据包上的。
- UDP 报文头没有“首部长度”这个字段，TCP 有。原因是 TCP 有可变长的「选项」字段，而 UDP 头部长度则是不会变化的，无需多一个字段去记录 UDP 的首部长度。
- TCP 报文头没有「包长度」字段，UDP 有。原因是 TCP 的报文长度计算公式为`IP总长度 - IP首部长度 - TCP首部长度`，三者都知道，无需单独计算。而 UDP 有的原因是，UDP 的报文长度不固定。由于 UDP 直接把应用层数据加上一个头部就发出去，因此它的长度是不固定的，报文体长度由应用把控。

#### TCP 的粘包问题：

在 TCP 协议中，当发送方发送多个 TCP 报文段时，TCP 把这些小数据包合并成一个大的数据包进行发送。 TCP 数据传输的过程中，发送方和接收方之间的数据传输是基于字节流的，而不是基于消息的。这意味着 TCP 并不知道数据的消息边界，只是按照字节流进行传输。

例如，如果发送方发送了两个小数据包 A 和 B，它们的消息边界是不明确的，那么在网络传输过程中，这两个小数据包可能会被合并成一个大的数据包 AB 进行传输。

为了解决 TCP 粘包问题，通常可以采用以下方法：

1. 消息边界分隔符：在消息中添加特定的分隔符，例如换行符、空格等，以便接收方可以根据分隔符来判断消息的边界。HTTP 在每一行的末尾添加一个回车符，通过回车符就可以判断消息边界了
2. 固定长度消息：在消息中添加固定长度的消息头，用来表示消息的长度，接收方可以根据消息头中的长度信息来解析消息。
3. 应用层协议：在应用层协议中定义消息的格式和边界，例如 HTTP、FTP 等协议都采用了这种方式来解决 TCP 粘包问题。比如 http 的 content-length 字段。不过前提是 http 报文长度确定；如果不确定，就只能采取上面两种方式解决。

http 采取的解决方案是 1 和 2，通过设置回车符、换行符作为 HTTP header 的边界，通过 Content-Length 字段作为 HTTP body 的边界。

#### TCP的Keepalive

TCP 的 Keepalive其实就是 TCP 的保活机制。

如果两端的 TCP 连接一直没有数据交互，达到了触发 TCP 保活机制的条件，那么内核里的 TCP 协议栈就会发送**探测报文**。

如果对端程序是正常工作的。当 TCP 保活的探测报文发送给对端, 对端会正常响应，这样 TCP 保活时间会被重置，等待下一个 TCP 保活时间的到来。
如果对端主机**宕机**，比如因为一些物理原因直接断网（注意不是进程崩溃，进程崩溃后操作系统在回收进程资源的时候，会发送 FIN 报文，而主机宕机则是无法感知的，所以需要 TCP 保活机制来探测对方是不是发生了主机宕机），当 TCP 保活的探测报文发送给对端后，石沉大海，没有响应，连续几次，达到保活探测次数后，TCP 会报告该 TCP 连接已经死亡。

所以，TCP 保活机制可以在双方没有数据交互的情况，通过探测报文，来确定对方的 TCP 连接是否存活，这个工作是在内核完成的。

注意和http的keep-alive区分。

### TCP 特点

1. **面向连接**
   面向连接，是指发送数据之前必须在两端建立连接。建立连接的方法是“三次握手”，这样能建立可靠的连接。建立连接，是为数据的可靠传输打下了基础。
1. **仅支持单播传输**
   每条 TCP 传输连接只能有两个端点，只能进行点对点的数据传输，不支持多播和广播传输方式。
1. **面向字节流**
   TCP 不像 UDP 一样那样一个个报文独立地传输，而是在不保留报文边界的情况下以字节流方式进行传输。
1. **可靠传输**
   对于可靠传输，判断丢包、误码靠的是 TCP 的段编号以及确认号。TCP 为了保证报文传输的可靠，就给每个包一个序号，同时序号也保证了传送到接收端实体的包的按序接收。然后接收端实体对已成功收到的字节发回一个相应的确认(ACK)；如果发送端实体在合理的往返时延(RTT)内未收到确认，那么对应的数据（假设丢失了）将会被重传。
1. **提供拥塞控制**
   当网络出现拥塞的时候，TCP 能够减小向网络注入数据的速率和数量，缓解拥塞。
1. **提供全双工通信**
   TCP 允许通信双方的应用程序在任何时候都能发送数据，因为 TCP 连接的两端都设有缓存，用来临时存放双向通信的数据。当然，TCP 可以立即发送一个数据段，也可以缓存一段时间以便一次发送更多的数据段（最大的数据段大小取决于 MSS）

### TCP 报文段

![](https://pic.imgdb.cn/item/62348f555baa1a80ab774d02.jpg)
TCP 报文结构如上所示：

- 32 比特的序号（seq）和确认号（ack），这两个是报文中确定传输最重要的部分
- 16 比特的接收窗口大小值，用于流量控制
- 标志字段，包括 ACK、SYN、FIN 等多个用于标识状态的字段。常用的标志位：
  - `SYN`(synchronous)： 发送/同步标志，用来建立连接，和下面的第二个标志位 ACK 搭配使用。连接开始时，SYN=1，ACK=0，代表连接开始但是未获得响应。当连接被响应的时候，标志位会发生变化，其中 ACK 会置为 1，代表确认收到连接请求，此时的标志位变成了 SYN=1，ACK=1。
  - `ACK`(acknowledgement)：确认标志，表示确认收到请求。
  - `PSH`(push) ：表示推送操作，就是指数据包到达接收端以后，不对其进行队列处理，而是尽可能的将数据交给应用程序处理；
  - `FIN`(finish)：结束标志，用于结束一个 TCP 会话；
  - `RST`(reset)：重置复位标志，用于复位对应的 TCP 连接。
  - `URG`(urgent)：紧急标志，用于保证 TCP 连接不被中断，并且督促中间层设备尽快处理。
- 首部字段，指示 TCP 首部长度（不包括数据），一般是 20 字节，但是可变；
- “选项”字段：用来扩展 TCP 协议的，可以包含一些额外的信息，例如最大段大小、窗口扩大因子等。这些信息可以用于优化 TCP 协议的性能，提高数据传输的效率和可靠性。比如下文的 SACK 字段就是加在这里的
- 数据字段，由于 MSS（最大报文段长度） 的限制，数据字段一般不会大于`1500-20-20=1460`字节

#### 序号和确认号

TCP 把数据看作是一个无结构的、有序的字节流；因此序号标识的是数据的**字节**，而不是报文段。
报文中的序号（seq）通常是报文中数据段的段首字节位置；
![](https://pic.imgdb.cn/item/623493275baa1a80ab80f873.jpg)

> 比如有一个 500000 字节的文件，每个报文段数据为 1000 字节，那么 seq 就分别是 0、1000、2000 直到 499000

确认号表示期望从对方主机收到的下一个字节序号。

> 比如两个主机 A 和 B，主机 A 已经收到了来自主机 B 的编号为 0~535 的字节，因此主机 A 再次给主机 B 发送的 ack 就是 536，表示期望收到第 536 号字节之后的数据；
> 另外，如果主机 B 给主机 A 发送的不是 536 之后的，主机 A 仍会继续发送 ack=536，直到接收到 536 为止，再继续请求下一个最前面的字节序号；这也是后面 TCP 可靠数据传输的原理之一，即`累计确认`

### TCP 重传机制

TCP 可靠数据传输主要通过重传机制实现；其中导致 TCP 进行重传的条件主要有两个：

1. 定时器超时（超时重传）
2. 连续收到三个 ack（快速重传）

#### 超时重传

TCP 仅在刚发送第一个报文段时启动**一个**定时器，在定时器到期并且仍没有收到有效的 ACK 时，就会重新发送具有最小序号的报文段（即最早发送的那个）。
由于超时时间过长或过短都会导致传输数据出现异常（详见计网 P162），因此每一次重传都会将超时时间间隔设置为前一次的 2 倍；如果有新的重传事件发生（比如收到了 3 个冗余 ack），就会重新计算超时时间。

#### 超时重传时间 RTO

RTT 指的是数据发送时刻到接收到确认的时刻的差值，也就是包的往返时间。超时重传时间是以 RTO （Retransmission Timeout 超时重传时间）表示。

RTO 的设置既不能太大也不能太小，太小会导致重传发送过快，太大则会浪费很多时间
超时重传时间 RTO 的值应该略大于报文往返 RTT 的值。具体来说，「超时重传时间 RTO 的值」应该是一个动态变化的值。RTO 在建立连接时需要先进行采样，然后后续的 RTO 以第一次为标准进行更改。

估计往返时间，通常需要采样以下两个：

- 需要 TCP 通过采样 RTT 的时间，然后进行加权平均，算出一个平滑 RTT 的值，而且这个值还是要不断变化的，因为网络状况不断地变化。
- 除了采样 RTT，还要采样 RTT 的波动范围，这样就避免如果 RTT 有一个大的波动的话，很难被发现的情况。

![](https://pic.imgdb.cn/item/6415cb71a682492fccfefc70.jpg)

其中建议`α = 0.125，β = 0.25， μ = 1，∂ = 4`，这些都是实验数据。

如果超时重发的数据，再次超时的时候，又需要重传的时候，TCP 的策略是超时间隔加倍。
也就是每当遇到一次超时重传的时候，都会将下一次超时时间间隔设为先前值的两倍。两次超时，就说明网络环境差，不宜频繁反复发送。

#### 快速重传

当发送方连续收到 3 个相同的 ack 时，将会在定时器到期之前就重传这个报文段。
收到冗余 ack 的原因是 TCP 会在不同的情况下发送可能不同于平常的 ack，规则如下：（都是指接收方）

| 事件                                         | 动作                                                                       |
| -------------------------------------------- | -------------------------------------------------------------------------- |
| 收到正常顺序的数据，并且之前没有缺漏         | 延迟 500ms 发送 ack，即如果下一个报文段 500ms 内没有到达，就发送这个的 ack |
| 收到比期望序号大的报文段，即`seq > ack`      | 说明中间有缺失，会继续发送缺失部分的 ack                                   |
| 收到填补缺失的报文段，填补之后前面都已经完整 | 从整个接收区的末尾字节继续发送 ack，即完全填充之后就又正常发送 ack         |

因此，收到冗余 ack 的原因是**报文段缺失**。
一旦发送方收到连续 3 个相同的 ack，就会进行快速重传，即把缺失的报文段再发一次，以弥补接收方的缺失。

> 至于为什么是 3 个，其实是一种统计规律的结果。即如果出现了 3 个冗余 ACK 可能是乱序或丢包导致的，但是丢包一定会产生 3 个冗余 ACK，而乱序可能会产生 1、2、3 个不等的。
> 可以看一个回答：https://www.zhihu.com/question/21789252

这部分的图可以参考 计网 P164

---

快速重传并不是处理丢包最完美的重传方式。它有一个很大的问题是，如果出现三个重复的 ACK，那应该从这个 ACK 开始将之后的都重新发送，还是只发送丢包的那个？

后者明显效率更高，但是需要知道具体是哪个位置发生了丢包。因此就有了一种**SACK 重传**；

#### SACK 和 D-SACK

SACK（ Selective Acknowledgment）， 选择性确认。

这种方式需要在 TCP 头部「选项」字段里加一个 `SACK` 的东西，它**可以将已收到的数据的信息发送给「发送方」**，这样发送方就可以知道哪些数据收到了，哪些数据没收到，知道了这些信息，就可以**只重传丢失的数据**。

SACK 和 ACK 并不冲突，可以和 ACK 一起发送。

![](https://pic1.imgdb.cn/item/6368daca16f2c2beb1231c3c.jpg)

---

除了 SACK，还有一种更高级的 D-SACK，用于告诉「发送方」有哪些数据被重复接收了。

D-SACK 可以解决接受方的确认报文丢失问题。如下图：

![](https://pic.imgdb.cn/item/6415ccbea682492fcc01bb83.jpg)

D-SACK 的主要特点就是在 SACK 的基础上，除了告知发送方哪些数据收到或没收到，还告诉哪些数据发重复了，让发送方知道接收方**实际上是接收到了数据的，只是确认报文丢失了**。

### TCP 流量控制

#### 滑动窗口

TCP 的流量控制主要是基于滑动窗口机制。TCP 在发送端和接收端都设置了一个滑动窗口，用于可以保证一次性接收多组数据，而不是一收一发。窗口大小就是指**无需等待确认应答，而可以继续发送数据的最大值**。

发送方窗口：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/16.jpg?)

发送方可以随时发送#3 内的数据。当收到接收方的 ACK 时，则会缩小#2 的左边界，增大#1 的右边界，相当于窗口向右移动

![32 ~ 36 字节已确认](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/18.jpg)

接收方窗口：

![接收窗口](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/20.jpg)

接收窗口的大小和发送窗口的大小不同时相等。即他们在大多数时刻是相等的；接收方通过告知发送方当前自己的接收窗口大小来使得发送方减少发送窗口大小，相当于减少了发送数据的量；当他们同时做出改变时，两者的大小就是相等的；但是如果发送方还没来得及缩小或扩大窗口，那就是不相等，不过稍后还是会趋向相等。

通常情况下，双方的滑动窗口大小可能会“一伸一缩”的变动，但是最大的大小不会改变。

#### 流量控制

发送方不能无脑的发数据给接收方，要考虑接收方处理能力。
如果一直无脑的发数据给对方，但对方处理不过来，那么就会导致触发重发机制，从而导致网络流量的无端的浪费。
为了解决这种现象发生，TCP 提供一种机制可以让「发送方」根据「接收方」的实际接收能力控制发送的数据量，这就是所谓的流量控制。

根据上面的发送窗口和接受窗口图，我们划分一下区域：

- A 区：已发送已确认
- B 区：已发送未确认
- C 区：未发送未确认
- D 区：未发送并且超出窗口右边界

接收方：

- A 区：已确认已接收
- B 区：可接受未接收
- D 区：超出窗口

如下图所示，正常情况下的流量控制主要起到几个作用：

1. 发送方每发送一段数据，B 区的大小就会增大，同时 C 区大小相应减少。但如果未收到接收方的确认，发送方的滑动窗口不会移动，而是会充满整个 B 区。这时相当于不会再发送更多的数据，直到收到 ACK。
2. 如果接收方确认数据，那么确认的部分就会成为发送方的 A 区，窗口前移，新的 D 区进入成为 C 区
3. 接收方每接收到一段数据，如果数据顺序正常，就会将其转为 A 区，同时窗口前移，并告知发送方自己的窗口大小。

![流量控制](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/21.png?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

但是有特殊情况是，如果接收方因为应用层的原因不能及时取走缓冲区内的数据，那么接收方的滑动窗口大小就可能变得越来越小。这时为了防止发送方仍然发送很多数据导致接收方接收不到的情况，就需要让发送方通知接收方减少发送窗口的大小，最终甚至可能减为 0
这种情况下双方的变化为：

1. 相对上一种正常情况，接收方的窗口可能会减小。比如下图中，接收方窗口缩小到 80，就会通过`window=80`告诉发送方，随后发送方的窗口也会减小。
2. 如果极端情况下，接收方窗口收缩到 0，那么发送方窗口也会到 0。也就是发生了窗口关闭。不过此时发送方实际上会定时发送窗口探测报文，以便知道接收方的窗口是否发生了改变，并不是完全不发送。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/22.jpg?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

接收窗口的空闲区间大小用`rwnd`表示，表示缓存区中的**空闲空间**。
当接收方收到数据时，会把 rwnd 放在回复给发送方的报文段中。发送方收到接收方的 rwnd 之后，便可以通过这个值确定自己本次要发送多少数据，保证自己发送的数据量不会导致 rwnd 被充满。
发送方要保证发送的数据不会使得 rwnd 变为 0；同时如果 rwnd 变为 0 也并不是完全停止发送，而是会继续发送一个字节的报文段，用于获取新的 rwnd 值。

### TCP 拥塞控制

拥塞控制不等同于流量控制。虽然两者都是通过控制发送的数据量来影响传输，但是拥塞控制主要解决的是重传导致的恶性循环（拥堵导致丢包，丢包导致重传，越重传越拥堵），而流量控制通常不会考虑到重传问题。

TCP 的拥塞控制实际上是一种端到端的拥塞控制，即，网络层没有提供任何关于拥塞的信息，拥塞与否主要依靠 TCP 自己判断，比如三次冗余 ack 等方式。

TCP 拥塞控制就是控制拥塞窗口`cwnd`，限制**发送**方在未确认时能发送的数据量。因此接受方中未被确认的数据量不会大于`min{cwnd,rwnd}`。其实也就是说发送方要发送的数据量不能大于这两个的最小值。

> 注意 rwnd 和 cwnd 的区别：
> rwnd：流量控制机制，由接收端的缓存空间剩余量决定，由接收端发送给发送端
> cwnd：拥塞控制机制，由发送端跟踪并控制，可以由发送端直接改变

TCP 的拥塞控制机制主要是以下三种机制：

- 慢启动
- 拥塞避免
- 快速恢复

拥塞控制的特点：线性增加（拥塞避免或快速恢复的线性增加）、乘性减少（一旦有冗余 ack 就把阈值砍半）

> 注意拥塞控制中的速度单位 MSS/RTT ，即一次收发时间内发送多少个最大报文段长度。

#### 慢启动

慢启动流程：

1. TCP 连接刚开始时`cwnd`值通常被设为 1 个 MSS（最大报文段长度），然后每当收到一个确认时就指数增加，变成每次发送 2、4、8.....个 MSS；
2. 慢启动的停止条件和情况有三种：
3. 当出现一次**超时重传**时，就立即停止这种增长，取当前发送的 MSS 值的一半作为`ssthresh`值，即“阈值”（比如到一次发送 8 个 MSS 时出现超时，就设置 ssthresh = 4），并且把 cwnd 重设为 1，**重新开始慢启动**。
4. 当已经设置了 ssthresh 值，并且慢启动又到达了这个值，就会进入**拥塞避免**
5. 当出现**三个冗余 ack**时，进入快速恢复阶段。

#### 拥塞避免

进入拥塞避免时，由上面可以知道此时 cwnd 应该是刚到达 ssthresh，即阈值的一半。

1. 在拥塞避免阶段，增长变成线性，即从慢启动确定的阈值开始，一次只增加一个 MSS。
2. 结束拥塞避免阶段也有两种情况：
3. 当出现一次**超时重传**时，和慢启动一样，cwnd 被重设为 1，重新慢启动。
4. 当出现一次**快速重传**时（即三个冗余 ack），就立即停止增长，并类似的设置阈值为当前值的一半。比如这次到 12MSS 时出现重传，就设置阈值为 6；然后进入快速恢复阶段

#### 快速恢复

快速恢复启动的时候有两种情况：

1. 拥塞避免阶段结束，刚刚出现了一次快速重传
2. 慢启动结束，刚刚出现了一次快速重传

因此不管是哪种情况进入，都是因为快速重传的出现。快速恢复就是为了处理这种快速重传，把之前丢失的数据包赶紧发送给对方，所以这一步反而需要加快发送速度。

快速恢复阶段做的事是：

- 拥塞窗口 cwnd = ssthresh + 3 （ 3 的意思是确认有 3 个数据包被收到了），其中这里的 ssthresh 是在拥塞避免阶段设置的 1/2cwnd
- 重传丢失的数据包。这一步总共会重传三个，即冗余的三个 ack 对应的，从那个开始之后的三个数据包，表示把之前漏掉的补上。
  - 如果再收到重复的 ACK，那么 cwnd 增加 1；这个过程的目的是尽快将丢失的数据包发给目标。
  - 如果收到新数据的 ACK 后（即之前丢包的补上），把 cwnd 设置为拥塞避免确定的 ssthresh 值（1/2cwnd），原因是该 ACK 确认了新的数据，说明从重复的 ACK 时的数据都已收到，该恢复过程已经结束，可以回到恢复之前的状态了，也即再次进入拥塞避免状态；

在快速恢复中如果再次出现超时的情况，就执行在上面两个中一样的步骤，即 cwnd 重置为 1，并进入慢启动重新走一次流程。

全过程如下：
![](https://pic.imgdb.cn/item/6415d444a682492fcc13a2c2.jpg)

### TCP 三次握手

![](https://pic.imgdb.cn/item/6234ab885baa1a80abb8eabe.png)

步骤： 0. 初始两端都处于 closed 状态。

1. 客户端向服务端发送一个不含有有效载荷的 TCP 报文段；标志位`SYN=1`，seq 设置一个随机的值作为初始序号（设为 x）；发送之后客户端进入`SYN-SENT`阶段，发送时服务端处于`LISTEN`阶段。
2. 服务端收到该报文，分配缓存和变量，根据 seq 值确定自己要开始接收的序号（即 ack 值），然后给客户端告知自己的 seq 值，标志位`SYN=1, ACK=1`。同理服务端发送后也立即进入`SYN-RECEIVED`阶段；
   注意服务器返回的`ack = x + 1`，是因为`SYN`会消耗一个序列号；**凡是需要对端确认的，一定消耗 TCP 报文的序列号**，下一个连接的 ack 为 y+1 也是这个原因。同时还会发送自己经过分配得到的 seq 值（设为 y）。这段报文同样没有有效载荷，被称为`SYNACK报文段`
3. 客户端收到，同样给连接分配缓存和变量；将自己的 seq 设置为`x+1`，即上一步服务器的 ack 值，表示自己从这里开始发送；随后令自己的`ack = y + 1`，即服务器的 seq 值下一个，标志位`ACK=1,SYN=0`。客户端发送后进入`ESTABLISHED`阶段，此时客户端已经准备好；服务端接收到之后也会进入`ESTABLISHED`阶段；当两者都进入，连接成功，开始发送数据。

另外这个请求可以携带有效载荷，同时因为连接已经建立，`SYN=0`

概括一下就是：

1. 第一次握手： 客户端向服务端发送连接请求报文段。该报文段中包含自身的数据通讯初始序号。请求发送后，客户端便进入 `SYN-SENT` 状态。
1. 第二次握手： 服务端收到连接请求报文段后，如果同意连接，则会发送一个应答，该应答中也会包含自身的数据通讯初始序号，发送完成后便进入 `SYN-RECEIVED` 状态。
1. 第三次握手： 当客户端收到连接同意的应答后，还要向服务端发送一个确认报文。客户端发完这个报文段后便进入 `ESTABLISHED` 状态，服务端收到这个应答后也进入 `ESTABLISHED` 状态，此时连接建立成功。

> seq 的起始序号通常不会从 0 开始，而是随机取一个序列号。主要原因是防止历史报文被下一个相同四元组的连接接收。
> 虽然通常情况下，通过四次挥手断开的连接由于在最后等待了 2MSL 的时间，历史报文必然已经消失；但是不是所有的连接都是通过四次挥手断开的。
> 假如服务端由于物理原因重启，导致客户端发送的数据没有被正确接收，连接也没有被四次握手断开，那么这时就会有一些残留的历史报文留在发送的路途上。假如下一次连接的 seq 不随机取，而是仍然选择某个初始值，就会导致收到历史报文，把它当作客户端本次连接发送的报文处理，导致错误
>
> ![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/network/tcp/isn%E7%9B%B8%E5%90%8C.png)
>
> 如上图，最后一个红色的发送就是上次连接的历史报文。这里服务端的接收就会导致它发送的 ack 异常，导致后续请求都出问题。
> 每次初始化序列号不一样能够很大程度上避免历史报文被下一个相同的连接接收，当然也不是 100%避免。
> 客户端和服务端的初始化序列号是一个 ISN 算法生成的，随机数是会基于时钟计时器递增的，基本不可能会随机成一样的初始化序列号。
> 当然，虽然 seq 是随机的，但是 tcp 依然可以保证所有的数据都被发送

#### 为什么是三次

一定要三次握手的原因主要有以下几个：

1. **三次握手才可以阻止重复历史连接的初始化**（主要原因）

所谓“历史连接的初始化”，就是指建立连接的第一个请求 SYN=1 被搁置，导致客户端以为其超时而重新发送了一个 SYN=1 的请求。这个旧的连接请求就被称为是历史连接。而历史连接的初始化指的就是这个历史连接可能会晚到达并且导致服务端收到两次连接请求。如果两次连接请求都没有经过确认就建立连接，那么服务端就对于历史请求发送的数据就浪费了。

具体可以参考下面这张图：

![](https://pic.imgdb.cn/item/6415a20ca682492fcca66f85.jpg)

正常就应该是上面这样的。客户端连续发送多次 SYN（都是同一个四元组）建立连接的报文，在网络拥堵情况下：

- 一个「旧 SYN 报文」比「最新的 SYN」 报文早到达了服务端，那么此时服务端就会回一个 SYN + ACK 报文给客户端，此报文中的确认号是 91（90+1）。
- 客户端收到后，发现自己期望收到的确认号应该是 100 + 1，而不是 90 + 1，于是就会回 RST 报文。
- 服务端收到 RST 报文后，就会释放连接。
- 后续最新的 SYN 抵达了服务端后，客户端与服务端就可以正常的完成三次握手了。

服务端返回的 SYN=1 的响应中含有的 ack，可以帮助客户端检查自己的请求是不是自己想要的那个。即，上图中本来客户端发送的是 seq=90，那么如果服务端返回的是 ack=100，客户端就会认识到本次连接出了问题，就会使用 RST 重置连接。

如果只有两次握手，也就是 SYN 到达服务器后，服务器就变为 ESTABLISHED 状态，那就会这样：

![](https://pic.imgdb.cn/item/6415a257a682492fcca70e27.jpg)

服务端在向客户端发送数据前，并没有阻止掉历史连接，导致服务端建立了一个历史连接，又白白发送了数据，妥妥地浪费了服务端的资源。

如果采取三次握手，那么除非客户端再次发回一个请求（第三次握手），服务端不会进入 ESTABLISHED 状态。这样的话，即使有再多旧的 SYN=1 的请求到来，服务端都不会发送数据。

2. **三次握手才可以同步双方的初始序列号**

第一次握手发送客户端的 seq
第二次握手发送服务端的 ack 和 seq
第三次握手发送客户端的 ack

这样三次握手之后，双方都持有的对方的 seq 和 ack，以及自己的 seq 和 ack，并且这些序列号都已经被同步完成，接下来就按照这些序列号来发送，

3. **三次握手才可以避免资源浪费**

这种情况实际上是基于第一种的情况来的。

如下图：

![](https://pic.imgdb.cn/item/6415a711a682492fccb218ef.jpg)

由于没有第三次握手，服务端不清楚客户端是否收到了自己回复的 ACK 报文，所以服务端每收到一个 SYN 就只能先主动建立一个连接
如果客户端发送的 SYN 报文在网络中阻塞了，重复发送多次 SYN 报文，那么服务端在收到请求后就会建立多个冗余的无效链接，造成不必要的资源浪费。

#### 每次握手丢失会发生什么

1. 第一次握手丢失：客户端重传 SYN

当客户端想和服务端建立 TCP 连接的时候，首先第一个发的就是 SYN 报文，然后进入到 SYN_SENT 状态。在这之后，如果客户端迟迟收不到服务端的 SYN-ACK 报文（第二次握手），就会触发「超时重传」机制，重传 SYN 报文，而且重传的 SYN 报文的序列号都是一样的。

默认的重传次数是 5 次，每次的超时时间是上一次的 2 倍。通常，第一次超时重传是在 1 秒后，后面的分别为 2、4、8、16、32，直到大约一分钟后就不会再重传。

2. 第二次握手丢失：客户端和服务端都会发生重传

客户端迟迟没有收到第二次握手，那么客户端就觉得可能自己的 SYN 报文（第一次握手）丢失了，于是客户端就会触发超时重传机制，重传 SYN 报文。
服务端收不到第三次握手，于是服务端这边会触发超时重传机制，重传 SYN-ACK 报文。

当重传次数用尽后，同样会断开连接。

3. 第三次握手丢失：只有服务端会发生重传，SYN-ACK

第三次握手丢失了，如果服务端那一方迟迟收不到这个确认报文，就会触发超时重传机制，重传 SYN-ACK 报文，直到收到第三次握手，或者达到最大重传次数。

> 注意，**ACK 报文是不会有重传的，当 ACK 丢失了，就由对方重传对应的报文。**

### TCP 四次挥手

![](https://pic.imgdb.cn/item/6234b0a55baa1a80abc43d47.jpg)

1. 第一次挥手： 客户端会发送一个 FIN 报文，报文中会指定一个序列号。此时客户端处于 `FIN_WAIT1` 状态。

即发出连接释放报文段（FIN=1），并停止再发送数据，主动关闭 TCP 连接，进入`FIN_WAIT1`（半关闭）状态，等待服务端的确认。

2. 第二次挥手：服务端收到 FIN 之后，会发送 ACK 报文，且把客户端的序列号值 +1 作为 ACK 报文的序列号值，表明已经收到客户端的报文了，此时服务端处于 `CLOSE_WAIT `状态。

即服务端收到连接释放报文段后即发出确认报文段（ACK=1），服务端进入`CLOSE_WAIT`（关闭等待）状态，此时的 TCP 处于半关闭状态，**客户端到服务端的连接释放**。
客户端收到服务端的确认后，进入`FIN_WAIT2`（终止等待 2）状态，等待服务端发出的连接释放报文段。

3. 第三次挥手：如果服务端也想断开连接了，和客户端的第一次挥手一样，发给 FIN 报文，且指定一个序列号。此时服务端处于 `LAST_ACK `的状态。

即服务端没有要向客户端发出的数据，服务端发出连接释放报文段（FIN=1，ACK=1），服务端进入`LAST_ACK`（最后确认）状态，等待客户端的确认。

4. 第四次挥手：客户端收到 FIN 之后，一样发送一个 ACK 报文作为应答，且把服务端的序列号值 +1 作为自己 ACK 报文的序列号值，此时客户端处于 `TIME_WAIT` 状态。需要过一阵子以确保服务端收到自己的 ACK 报文之后才会进入 `CLOSED` 状态，服务端收到 ACK 报文之后，就处于关闭连接了，处于 `CLOSED` 状态。

即客户端收到服务端的连接释放报文段后，对此发出确认报文段（ACK=1），客户端进入`TIME_WAIT`（时间等待）状态。
此时 TCP 未释放掉，**需要经过时间等待计时器设置的时间 2MSL 后**，客户端才进入 CLOSED 状态。（1MSL 大概是 30s 左右，这是 Linux 的数据）

#### 为什么需要四次挥手

因为当服务端收到客户端的 SYN 连接请求报文后，可以直接发送 SYN+ACK 报文。其中 ACK 报文是用来应答的，SYN 报文是用来同步的。但是关闭连接时，当服务端收到 FIN 报文时，很可能并不会立即关闭 SOCKET，所以只能先回复一个 ACK 报文，告诉客户端，“你发的 FIN 报文我收到了”。**只有等到我服务端所有的报文都发送完了**，我才能发送 FIN 报文来确认关闭，因此不能一起发送，故需要四次挥手。

TCP 使用四次挥手的原因是因为 TCP 的连接是**全双工**的，所以**需要双方分别释放到对方的连接**，单独一方的连接释放，只代表不能再向对方发送数据，连接处于的是半释放的状态。

#### 一定需要四次挥手吗？

**在特定情况下，四次挥手是可以变成三次挥手的**。具体参考[小林 coding 上的内容](https://xiaolincoding.com/network/3_tcp/tcp_three_fin.html#%E4%BB%80%E4%B9%88%E6%83%85%E5%86%B5%E4%BC%9A%E5%87%BA%E7%8E%B0%E4%B8%89%E6%AC%A1%E6%8C%A5%E6%89%8B)

简单概述一下：四次挥手的第 2、3 次挥手是可以在一定条件下合并的。通常分开这两次挥手的原因是服务端还有一些数据要发送，因此需要先发送 ACK 让客户端等待，再当数据发送完时发送 FIN 表示关闭。

但是**如果「没有数据要发送」并且「开启了 TCP 延迟确认机制」，那么第二和第三次挥手就会合并传输，这样就出现了三次挥手。**

![](https://pic.imgdb.cn/item/6415b438a682492fccce7bf3.jpg)

所谓“延迟确认机制”，指的是 tcp 对于 ACK 的延迟发送。当发送没有携带数据的 ACK，它的网络效率也是很低的，因为它也有 40 个字节的 IP 头 和 TCP 头，但却没有携带数据报文。
TCP 延迟确认的策略：

- 当有响应数据要发送时，ACK 会随着响应数据一起立刻发送给对方
- 当没有响应数据要发送时，ACK 将会延迟一段时间，以等待是否有响应数据可以一起发送
- 如果在延迟等待发送 ACK 期间，对方的第二个数据报文又到达了，这时就会立刻发送 ACK

![](https://pic.imgdb.cn/item/6415b47ba682492fcccef6b4.jpg)

实际上，三次挥手的情况是比四次挥手要多的。

#### 为什么 TIME_WAIT 等待的时间是 2MSL？

最后一次挥手中，客户端会等待一段时间再关闭的原因，主要有两个原因：

1. **防止发送给服务器的确认报文段丢失或者出错**，从而导致服务器端不能正常关闭。 2MSL 时长 这其实是相当于至少允许报文丢失一次。比如，若第四次握手发送的 ACK 在一个 MSL 内丢失，这样服务端重发的 FIN 会在第 2 个 MSL 内到达，TIME_WAIT 状态的连接可以应对。

并且，2MSL 的时间是从客户端接收到 FIN 后发送 ACK 开始计时的。如果在 TIME-WAIT 时间内，因为客户端的 ACK 没有传输到服务端，客户端又接收到了服务端重发的 FIN 报文，那么 2MSL 时间将重新计时。

2. 防止下一次建立连接时，历史报文再次发送。比如挥手之前的某个报文由于时间原因耽搁了，如果不等这 2MSL，那么有可能紧接着的下一次连接就会收到这个报文，导致传输的数据出错。等待 2MSL 一定可以使本次连接内的所有报文都消失掉，避免出现历史报文重新发送的问题。

至于为什么值是 2MSL，并且单位为什么是 MSL 而不是 RTT，主要是这几个原因：

1. MSL 而不是别的单位：因为 TCP 报文基于是 IP 协议的，而 IP 头中有一个 `TTL` 字段，是 IP 数据报可以经过的最大路由数。**MSL 应该要大于等于 TTL 消耗为 0 的时间**，以确保报文已被自然消亡。
2. 为什么是 2 倍：网络中可能存在来自发送方的数据包，当这些发送方的数据包被接收方处理后又会向对方发送响应，所以**一来一回需要等待 2 倍的时间**。也就是说比如在 FIN_WAIT2 的最后一步，也就是最后一个报文没有被接收方收到，就会触发超时重发 FIN 报文，另一方接收到 FIN 后，会重发 ACK 给被动关闭方， 一来一去正好 2 个 MSL。

# 网络层

网络层运行在传输层之下，从结构上分为数据平面（IP）和控制平面（SDN），其主要的功能有两个：

- 转发，是数据平面的唯一功能，即主机和主机、主机和路由器、路由器和路由器之间转发分组；绝大部分的转发在路由器上完成
- 路由选择，控制平面的主要功能，即选择合适的路径、转发方式

网络层的主要作用是：**实现主机与主机之间的通信，也叫点对点（end to end）通信。**

## 网络层基础

网络层的主要作用是：实现主机与主机之间的通信，也叫点对点（end to end）通信。

这句话再强调一遍，关键点就在于这个“主机和主机”之间。

相比于链路层，网络层在传输的全程，源ip和目标ip都不会改变（NAT除外）。但MAC地址会在每一跳都改变。MAC 的作用则是实现「直连」的两个设备之间通信，而 IP 则负责在「没有直连」的两个网络之间进行通信传输。
![](https://pic.imgdb.cn/item/64cbfe711ddac507cc2d984f.jpg)

可以理解为，mac地址控制的是每一跳或者说每个单独区间的传输，而ip则是控制全程、整体的源和目标。

## IP（IPv4）

### IP 数据报

![](https://pic.imgdb.cn/item/623595a65baa1a80ab118e12.png)

- 版本：指 IP 协议的版本，占 4 位，如 IPv4 和 IPv6；
- 首部位长度：表示 IP 首部长度，占 4 位，最大数值位 15；
- 总长度：表示 IP 数据报总长度，占 16 位，最大数值位 65535；
- 生存时间（TTL）：表示 IP 数据报文在网络中的寿命，占 8 位；
- 协议：表明 IP 数据所携带的具体数据是什么协议的，如 TCP、UDP。

### IP 编址

好比一个人出行需要目的地和出发地一样，IP 地址起到了在网络世界中定址的作用

IP 地址采用等分十进制编码，即把本来的四个二进制字节写作类似`xxx.xxx.xxx.xxx`的格式。

#### 地址分类

以前，IP 地址的网络部分的长度被限制只能是 8、16、24 比特的一种，即掩码只能是 8、16 或 24。因此，网络部分长度为 8/16/24 比特的网络分别被成为 A/B/C 类网络：

![](https://pic.imgdb.cn/item/62359b5b5baa1a80ab1f39f1.png)
![](https://pic.imgdb.cn/item/6235a03d5baa1a80ab2a302e.png)

IP 地址可根据主机号和网络号所占字节分为 ABCDE 类：

- A 类地址:网络号占 1 个字节。网络号的第一位固定为 0。
- B 类地址：网络号占 2 个字节。网络号的前两位固定为 10。
- C 类地址：网络号占 3 个字节。网络号的前三位固定位 110。
- D 类地址：前四位是 1110，用于多播(multicast)，即一对多通信。D 和 E 类地址都是没有主机号的，因此不能用于特定的主机间一对一传输
- E 类地址：前四位是 1111，保留为以后使用。

其中，ABC 三类地址为单播地址（unicast）,用于一对一通信，是最常用的。

IP 地址分类的最大好处是，主机解析到一个 IP 地址时候，解析其 IP 地址前几位就可以得出它是哪类地址.
但是缺点也很明显，不灵活，而且不实际。

不同类型的网络拥有不同数量的主机和网络数。比如 A 类网络部分长度为 8，即最多只能有`2^8 - 2 = 254`个网络（全 1 和全 0 有特殊用处），但每个子网却有`2^24 - 2`台主机；其他的同理：
![](https://pic.imgdb.cn/item/62359c395baa1a80ab213a37.jpg)

> 特殊的 IP 地址：
>
> - `127.0.0.1`：回环地址。该地址指电脑本身
> - `10.x.x.x`、`172.16.x.x`～`172.31.x.x`、`192.168.x.x`：这些地址被用做内网中。用做私网地址，这些地址不与外网相连。
> - `0.0.0.0`：这个 IP 地址在 IP 数据报中只能用作**源**IP 地址，这发生在当设备启动时但又不知道自己的 IP 地址情况下。
> - `255.255.255.255`：广播地址，只能用作**目标**IP 地址，用于不知道自己 IP 的情况下寻找 DHCP 服务器时用
> - 主机号全为 0（`xxx.0.0.0`）：标识本子网
> - 主机号全为 1（`xxx.255.255.255`）：标识本子网下的全部主机

主机号全0和全1的情况在很多时候都要注意
首先是计算一个网络号下主机的个数，那应该是`2 ^ n - 2`，因为要减去全0和全1的情况。
全1很好理解，就是当一台主机想向子网内的所有主机发送消息时，就用到这个ip。
全0表示标识本子网，怎么理解呢？
全0主要用于跨子网的广播。

广播地址可以分为本地广播和直接广播两种。

- 在本网络内广播的叫做本地广播。例如网络地址为 192.168.0.0/24 的情况下，广播地址是 192.168.0.255（主机号全1） 。因为这个广播地址的 IP 包会被路由器屏蔽，所以不会到达 192.168.0.0/24 以外的其他链路上。
- 在不同网络之间的广播叫做直接广播。例如网络地址为 192.168.0.0/24 的主机向 192.168.1.255/24 的目标地址发送 IP 包。收到这个包的路由器，将数据转发给 192.168.1.0/24（主机号全0，表示外部子网下的全部主机），从而使得所有 192.168.1.1~192.168.1.254 的主机都能收到这个包

#### CIDR

CIDR 是无类别域间路由选择的简称，即可以任意划定主机部分和网络部分的长度，32 比特的 IP 地址被划分为两部分，前面是网络号，后面是主机号。

就比如：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/15.jpg)

划分的方式有两种：
- 前缀，表示形式 a.b.c.d/x，其中 /x 表示前 x 位属于网络号， x 的范围是 0 ~ 32
- 子网掩码，掩码是一个很大的值，网络号部分全1，主机号部分全0，和IP地址相与就能得到具体的主机号码

![](https://pic.imgdb.cn/item/64cc01181ddac507cc30658c.jpg)

划分网络号和主机号的最大原因，在于路由器的转发规则。

两台计算机要通讯，首先要判断是否处于同一个广播域内，即网络地址是否相同。如果网络地址相同，表明接受方在本网络上，那么可以把数据包直接发送到目标主机。如果不在本网络，那就需要路由器在网络之间进行转发，这个时候网络号就起到作用了。

#### 子网

网络中并不是所有的设备都分散在网络各处，而是会组成一个个的小块，互联这些小块中的主机接口和路由器接口的网络叫做**子网**。每个子网前面一部分的编址相同，只有后面一小部分的编址不同，用于区分子网内的主机；

注意子网和网络的区别：
只划分了网络地址和主机地址的不能算作子网。子网是在这个基础上，取出几位作为子网网络地址，才得到具体的子网。
- 网络其实本身就是一块一块的，网络之间靠网络号区分；
- 子网在网络内部再进行划分，子网靠设置的子网网络地址来区分

![](https://pic.imgdb.cn/item/64cc02d31ddac507cc322eeb.jpg)

子网掩码可以划分子网。子网划分实际上是将主机地址分为两个部分：子网网络地址和子网主机地址。形式如下：

![](https://pic.imgdb.cn/item/6415dc4ca682492fcc24d013.jpg)

即划分子网时，在主机号中选取几个位用作标识子网的网络地址，剩下部分用于标识子网的主机地址。
子网地址有几位，子网的数量就会有相应的值。比如 2 位数就会产生 00 01 10 11 四个子网。

#### 公有IP和私有IP

A、B、C 分类地址，实际上有分公有 IP 地址和私有 IP 地址。
平时常用的IP地址都是私有IP，这些地址允许组织内部的 IT 人员自己管理、自己分配，而且可以重复。
但是一旦想要在公网上访问，比如直接在互联网上访问，就需要一个独一无二的公网IP。

公有 IP 地址是由 ICANN 组织管理，中文叫「互联网名称与数字地址分配机构」。

### IP路由

上面说过，IP地址的网络地址这一部分是用于进行路由控制。

路由控制表中记录着网络地址与下一步应该发送至路由器的地址。在主机和路由器上都会有各自的路由器控制表。

在发送 IP 包时，首先要确定 IP 包首部中的目标地址，再从路由控制表中找到与该地址具有相同网络地址的记录，根据该记录将 IP 包转发给相应的下一个路由器。如果路由控制表中存在多条相同网络地址的记录，就选择相同位数最多的网络地址，也就是最长匹配。

注意：
- 每个路由器查找的是数据包的**目标IP**
- 查找的范围是路由表的key，也就是IP地址，实际上是只有网络号的IP。
- 查找的方式是匹配相同**网络**地址，即IP划分的网络号部分。如果多个网络号都匹配相等，那就继续往后走到主机号部分匹配，最终取到最长的匹配的那个。
- 查找的结果是下一跳的路由器的IP地址

![](https://pic.imgdb.cn/item/64cc045d1ddac507cc33c60d.jpg)

这张图上还可以看到，这些路由器用于在不同网络之间进行通信，因此都是网关路由器。在网络内部，相当于是从一个网关路由器转发到下一个网关路由器。

### IP分片和重组

最大传输单元 MTU是由链路层限制的最大传输体积，也就是最大的IP数据报的大小。
最常见数据链路是以太网，它的 MTU 是 1500 字节。

当 IP 数据包大小大于 MTU 时， IP 数据包就会被分片。
经过分片之后的 IP 数据报在被重组的时候，只能由目标主机进行，路由器是不会进行重组的。

假设发送方发送一个 4000 字节的大数据报，若要传输在以太网链路，则需要把数据报分片成 3 个小数据报进行传输，再交由接收方重组成大数据报。

在分片传输中，一旦某个分片丢失，则会造成整个 IP 数据报作废，所以 TCP 引入了 MSS 也就是在 TCP 层进行分片不由 IP 层分片，那么对于 UDP 我们尽量不要发送一个大于 MTU 的数据报文。总之就是尽量控制数据大小不要超过MTU，不要让IP层进行分包。



## IPV6

IPV6 的特点：

1. 128 位地址，采用 8 组 4 位 16 进制数表示，用冒号隔开
2. 可自动配置，即使没有 DHCP 服务器也可以实现自动分配 IP 地址
3. 去掉了包头校验和，简化了首部结构，减轻了路由器负荷

![IPv4 首部与 IPv6 首部的差异](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/31.jpg)

差异：
- 取消了首部校验和字段。 因为在数据链路层和传输层都会校验，因此 IPv6 直接取消了 IP 的校验。
- 取消了分片/重新组装相关字段。 分片与重组是耗时的过程，IPv6 不允许在中间路由器进行分片与重组，这种操作只能在源与目标主机，这将大大提高了路由器转发的速度。
- 取消选项字段。

## 其他网络层技术

### DHCP

用于为主机动态配置 IP 地址。通常一个主机连接到网络获得 IP 地址的过程，就是执行 DHCP 的过程。

![DHCP 工作流程](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/36.jpg)

DHCP 本质上是一个应用，所以你也可以说他是一个基于 UDP 之上的应用层协议。

这个过程的步骤：

1. 客户端首先发起 **DHCP 发现报文（DHCP DISCOVER）** 的 IP 数据报，由于客户端没有 IP 地址，也不知道 DHCP 服务器的地址，所以使用的是 UDP **广播**通信，其使用的广播目的地址是 255.255.255.255（端口 67） 并且使用 0.0.0.0（端口 68） 作为源 IP 地址。DHCP 客户端将该 IP 数据报传递给链路层，链路层然后将帧广播到所有的网络中设备。
2. DHCP 服务器收到 DHCP 发现报文时，用 **DHCP 提供报文（DHCP OFFER）** 向客户端做出响应。该报文仍然使用 IP 广播地址 255.255.255.255，该报文信息携带服务器提供可租约的 IP 地址、子网掩码、默认网关、DNS 服务器以及 **IP 地址租用期**。
3. 客户端收到一个或多个服务器的 DHCP 提供报文后，从中选择一个服务器，并向选中的服务器发送 **DHCP 请求报文（DHCP REQUEST**进行响应，回显配置的参数。
4. 最后，服务端用 **DHCP ACK 报文**对 DHCP 请求报文进行响应，应答所要求的参数。

注意点：
- 租约的 DHCP IP 地址快期后，客户端会向服务器发送 DHCP 请求报文，根据响应确定是否能继续用该ip
- DHCP 交互中全程都是使用 **UDP** 广播通信。

注意这个“广播”，也就是上面说的跨网络广播。路由器有可能不会发送广播消息，因此广播消息有可能会由*DHCP 中继代理*处理。代理的工作原理就和常见的那些代理方式一样，一层一层传递到目标主机上。

### NAT

简单的来说 NAT 就是同个公司、家庭、教室内的主机对外部通信时，**把私有 IP 地址转换成公有 IP 地址。**

![NAT](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/38.jpg)

NAT 的升级是 NAPT，把 私有IP 地址 + 端口号起进行转换成公有IP

由于 TCP 和 UDP 区分连接的方式都包含源端口，因此相同的 IP 不同的端口会被视作是不同的请求。因此 NAPT 在转换私有 IP 内部的主机时，为每个内部请求创建一个不同的端口，这样就相当于不同的请求。

![NAPT](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/39.jpg)

如图，有两个客户端 192.168.1.10 和 192.168.1.11 （都是该网络内的私有IP）同时与服务器 183.232.231.172（公网IP） 进行通信，并且这两个客户端的本地端口都是 1025。

NAT将两个私有 IP 地址都转换 IP 地址为公有地址 120.229.175.121，但是以不同的端口号作为区分。

- 对服务器的接收，他会收到IP相同但访问端口不同的两个请求。这两个请求在网络层没有任何区别，但是到达传输层后，tcp和udp就可以将他们利用端口号区分开
- 服务器的返回，仍然是向这个IP地址发送响应。NAT收到响应后，根据响应包含的不同端口，将其再分为两个私有IP地址，给网络内的两个主机返回。

### ICMP

ICMP 全称是 **Internet Control Message Protocol**，也就是**互联网控制报文协议**。

`ICMP` 主要的功能包括：**确认 IP 包是否成功送达目标地址、报告发送过程中 IP 包被废弃的原因和改善网络设置等。**

在 `IP` 通信中如果某个 `IP` 包因为某种原因未能达到目标地址，那么这个具体的原因将**由 ICMP 负责通知**。

![ICMP 目标不可达消息](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/40.jpg)

# 链路层

## 以太网

以太网（英语：Ethernet）是为了实现**局域网**通信而设计的一种技术（是一种技术方案或者说是技术标准，并不是一种网络），它规定了包括物理层的连线、电子信号和介质访问层协议的内容。以太网是目前应用最普遍的局域网技术，取代了其他局域网标准如令牌环、FDDI 和 ARCNET。

链路层、物理层都属于以太网技术的范畴

以太网帧其实就是链路层对 IP 数据包的封装。

## 链路层寻址（MAC）

主机和路由器本身不具有链路层地址，是他们的网卡（网络适配器、网络接口）具有。链路层地址可以认为就是 MAC 地址。
MAC 地址为 6 字节，一旦被赋予就是**永久**的，一个 MAC 地址始终和网卡绑定，采用十六进制表示。
![MAC.jpg](https://i.loli.net/2021/12/02/G4P5bBIxHwOtup7.jpg)

> MAC 地址只在子网内生效；如果想要发送到外部，则只需要网关路由器输入端口的 MAC 地址，再由网关路由器进行 ARP 查询和转发。

## ARP 协议

ARP 协议协议(Address Resolution Protocol)，地址解析协议，它是用于实现**IP 地址到 MAC 地址的映射**。
ARP 协议是自学习、零配置的，其能够通过反复把最新的映射添加到维护的 ARP 表中，实现自动配置。

> ARP 协议既是网络层又是链路层的协议，是一个介于两者之间的协议。

ARP 协议流程如下：

1. 首先，每台主机、交换机都会在自己的 ARP 缓冲区中建立一个 ARP 列表，以表示 IP 地址和 MAC 地址的对应关系。
2. 当源主机需要将一个数据包要发送到目的主机时，会首先检查自己的 ARP 列表，是否存在该 IP 地址对应的 MAC 地址；如果有﹐就直接将数据包发送到这个 MAC 地址；如果没有，就向本地网段发起一个 ARP 请求的广播包，查询此目的主机对应的 MAC 地址。此 ARP 请求的数据包里，包括源主机的 IP 地址、硬件地址、以及目的主机的 IP 地址。
3. 网络中所有的主机收到这个 ARP 请求后，会检查数据包中的目的 IP 是否和自己的 IP 地址一致。如果不相同，就会忽略此数据包；如果相同，该主机首先将发送端的 MAC 地址和 IP 地址添加到自己的 ARP 列表中，如果 ARP 表中已经存在该 IP 的信息，则将其覆盖，然后给源主机发送一个 ARP 响应数据包，告诉对方自己是它需要查找的 MAC 地址。
4. 源主机收到这个 ARP 响应数据包后，将得到的目的主机的 IP 地址和 MAC 地址添加到自己的 ARP 列表中，并利用此信息开始数据的传输。如果源主机一直没有收到 ARP 响应数据包，表示 ARP 查询失败。

### RARP

RARP 协议正好相反，它是**已知 MAC 地址求 IP 地址**。它常常用于一些只预先配置了 MAC 地址，而没有获得 IP 地址的小型设备，比如打印机。

RARP 有一个专属的 RARP 服务器，在这个服务器上注册设备的 MAC 地址及其 IP 地址。当有设备想要从自己的 MAC 地址获得对应的 IP 地址，就会类似 ARP 一样，广播并收到 RARP 服务器的响应，随后设置自己的 IP 地址。

# 从浏览器输入 url 到显示的过程

### 0. 连接网络

1. 主机生成 DHCP 请求报文，被放在具有 IP 广播地址（255.255.255.255）和源 IP（0.0.0.0）的 IP 数据报中
2. 将上一步的数据报放置在以太网帧，该帧具有`FF:FF:FF:FF:FF:FF`的广播 MAC 地址，然后广播出去
3. DHCP 服务器收到该广播，通过解封装抽取其 UDP 数据报，并给该主机分配一个 IP 地址，再次封装发回到主机的 MAC 地址（主机的 IP 位置但 MAC 已知且固定）
4. 主机收到该帧，解封装得知自己的 IP 地址，该主机的 DHCP 客户端记录下该主机的 IP，此时已经相当于“连接上网络”

### 1. 解析 URL

1. 首先会对 URL 进行解析，分析所需要使用的传输协议和请求的资源的路径。如果输入的 URL 中的协议或者主机名不合法，将会把地址栏中输入的内容传递给搜索引擎。如果没有问题，浏览器会检查 URL 中是否出现了非法字符，如果存在非法字符，则对非法字符进行转义后再进行下一过程。
2. 浏览器会判断所请求的资源是否在缓存里，如果请求的资源在缓存里并且没有失效，那么就直接使用，否则向服务器发起新的请求。
3. 主机生成 DNS 查询报文，获取到网关路由器的 IP 和 MAC 地址，准备向本地 DNS 服务器发送查询请求
4. 判断本地是否有该域名的 IP 地址的缓存，如果有则使用，如果没有则向本地 DNS 服务器发起请求。本地 DNS 服务器也会先检查是否存在缓存，如果没有执行 DNS 查询，可能会有迭代查询和递归查询两种方式。
5. 获取 MAC 地址： 当浏览器得到 IP 地址后，数据传输还需要知道目的主机 MAC 地址；

- 如果对应的 ip 和主机在同一个子网里，可以使用 ARP 协议获取到目的主机的 MAC 地址；
- 如果不在一个子网里，那么请求应该转发给网关，由它代为转发，此时同样可以通过 ARP 协议来获取网关的 MAC 地址，此时目的主机的 MAC 地址应该为网关的地址。（这一步其实在第 3 步已经完成）

### 2. TCP 和 HTTP 连接

1. 进行 TCP 三次握手
2. 如果连接是 HTTPS，就还需要一个 HTTPS 握手：
3. 发送 HTTP GET 请求，请求对应的 HTML 文档；
4. 浏览器响应，返回该 HTML 文件，浏览器接收到响应后，开始对 html 文件进行解析，开始页面的渲染过程。

### 3. 浏览器显示网页

1. 浏览器首先会根据 html 文件构建 DOM 树，对 HTML 中的标签进行词法解析。
2. **解析 css**：并通过一些 css 选择器，确定每个 dom 节点的样式；即使没有 css，浏览器内部也有自己的样式. 这个过程中会想创建 dom 树一样创建 CSSOM, CSSOM 的特点如下:
   - CSSOM 阻止任何东西渲染, 也就是说在 CSSOM 完全建立之前是不会展示界面的
   - CSSOM 在加载一个新页面时必须重新构建. 即使你的 CSS 文件被缓存了，也并不意味着这个已经构建好了的 CSSOM 可以应用到每一个页面。
   - CSSOM 是展示任何东西的必需品。在 CSSOM 构建之前，所有东西都不会展示，如果你阻塞了 CSSOM 的构建，CSSOM 的构建就会消耗更长的时间，这就意味着页面的渲染也需要更长的时间。
3. **布局（Layout）**，知道了每个节点的样式，还需要知道节点的位置：获取节点具体位置排版的过程叫做；主线程这时会通过像建立 dom 树一样的方式建立**Layout Tree**，每个节点上都记录元素的 xy 坐标、尺寸、边框等信息。layout 树是和展示在屏幕上的元素对应
4. **绘制（paint）**：还需要知道一定的绘制顺序。主线程会遍历 Layout Tree 生成**绘制记录表（Paint Record）**，然后后续按照记录表顺序进行绘制。
5. **栅格化（Rastering）**：把上面的 dom 树、绘制记录表、Layout Tree 等变成像素点展示在页面上
   - 这里介绍 chrome 栅格化方式的改进：现在采用的是合成方法，即对页面按一定规则分割成图层（Layer Tree），然后栅格化图层再进行拼接

这时主线程把信息传递给合成器线程，来把上面这些不同的表合成在一起

10. **分割图块（tiles）**：把图层分割成图块，每个图块都有一个栅格进程进行栅格化
11. **合成器帧（Frame）**：上一步栅格化之后，把每一部分存在 GPU 内存中；合成器线程收集（draw quads）的图块信息，拼接成合成器帧

接下来会通过 ipc 传输给浏览器进程中的 UI 线程

12. **最后渲染**：浏览器进程收到来自合成器帧拼接成的帧时，就会通过 GPU 渲染到屏幕上
13. **更新渲染**：当页面变化，就会生成新的合成器帧，然后继续按照 11、12 步渲染到页面上

![浏览器工作流程.png](https://i.loli.net/2021/12/02/I2sC6cwBHkgYESK.png)

# 网络安全相关

## CSP

内容安全策略( CSP )是一个额外的安全层，用于检测并削弱某些特定类型的攻击，包括跨站脚本 (XSS) 和数据注入攻击等
CSP 的实质就是白名单制度，开发者明确告诉客户端，哪些外部资源可以加载和执行，等同于提供白名单。
启动 CSP 的方法有两种:

- 通过 HTTP 头信息的 `Content-Security-Policy` 的字段

```
Content-Security-Policy: script-src 'self'; object-src 'none';
style-src cdn.example.org third-party.org; child-src https:;
```

- 另一种是通过网页的<meta>标签。

```html
<meta
  http-equiv="Content-Security-Policy"
  content="script-src 'self'; object-src 'none'; style-src cdn.example.org ; child-src https:"
/>
```

上面代码中，CSP 做了如下配置:

- 脚本：只信任当前域名
- `<object>`标签：不信任任何 URL，即不加载任何资源
- 样式表：只信任http://cdn.example.org
- 框架（iframe）：必须使用 HTTPS 协议加载
- 其他资源：没有限制

## XSS 攻击

XSS 即（`Cross Site Scripting`）中文名称为：跨站脚本攻击。
攻击者在网页插入一些 script 标签, 当用户浏览器加载到页面时会触发这段 script。攻击者会获取到比如 cookie 等信息，然后使用该信息来冒充合法用户
XSS 攻击最主要有如下分类：反射型、存储型、 DOM-based 型。

### 反射型

反射型 XSS 指的是恶意脚本作为网络请求的一部分。
反射型 XSS 的攻击步骤：

1. 攻击者构造出特殊的 URL，其中包含恶意代码。
2. 用户打开带有恶意代码的 URL 时，网站服务端将恶意代码从 URL 中取出，拼接在 HTML 中返回给浏览器。
3. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

```
http://aaa.com?q=<script>alert("你完蛋了")</script>

// 服务端拼接query参数到html中
```

在服务器端会拿到 q 参数,然后将内容返回给浏览器端，浏览器将这些内容作为 HTML 的一部分解析，发现是一个脚本，直接执行，这样就被攻击了。
之所以叫它反射型, 是因为恶意脚本是通过作为网络请求的参数，经过服务器，然后再反射到 HTML 文档中，执行解析。和存储型不一样的是，服务器并不会存储这些恶意脚本。

### 存储型

主要是将恶意代码上传或存储到服务器中，下次只要受害者浏览包含此恶意代码的页面就会执行恶意代码。
因此存储型 XSS 的攻击步骤如下：

1. 攻击者将恶意代码提交到目标网站数据库中。
2. 用户打开目标网站时，网站服务器将恶意代码从数据库中取出，然后拼接到 html 中返回给浏览器中。
3. 用户浏览器接收到响应后解析执行，那么其中的恶意代码也会被执行。
4. 那么恶意代码执行后，就能获取到用户数据，比如上面的 cookie 等信息，把该 cookie 发送到攻击者网站中，那么攻击者拿到该 cookie 然后会冒充该用户的行为，调用目标网站接口等违法操作。

这种攻击常见于带有用户保存数据的网站功能，如论坛发帖、商品评论、用户私信等

### DOM-based

DOM XSS 是基于文档对象模型的 XSS。一般情况下不需要经过服务端，是出现于前端的问题，比如直接将用户的输入通过`innerHTML`或者`document.write`打印在页面上，就有可能执行恶意代码。

恶意输入的来源可能有：

- 用于输入的地方，比如 input、textarea
- url 栏，js 脚本可能会检测 url 的变化并把其中的某些参数取出来直接插入 dom

### XSS 的预防

详见https://tech.meituan.com/2018/09/27/fe-security.html

XSS 主要是要预防恶意代码的执行，即：

1. 输入过滤。输入过滤不是一个很好的方式，因为：

- 如果通过转码的形式对输入进行编译，有可能导致字符串输出异常。比如如果使用 html 的转码把`<`改为了`&lt;`，那么这个值如果是通过 js 输出到 html 中，就不会再转成`<`。这样可能导致显示问题。
- 即使在前端过滤了用户输入，攻击者也可能绕过前端直接注入服务端。

但是这种方式对于一些语义明确的输入，比如姓名、电话号码等，可以通过校验用户输入，禁用非法字符的形式阻止恶意代码的输入。

2. 输出转码。输出转码主要是针对需要拼接或者输出文本的需求。比如需要在一个地方显示一些文字，但是这些文字内部可能会包含有`<script>`标签，有可能会导致浏览器执行这部分代码。

转码的方式可以参考下面的内容，大致就是对`<`这样的敏感字符进行转码，保证其不会被识别为html标签，但是显示正常。
当然html转码是个很复杂的事情，不只是把几个字符转码就行。为了保证安全要根据不同场景做不同转码。

3. 预防dom注入。输出转码是针对文本的情况，dom则是在一起写需要直接拼接html的场景，比如通过`innerHTML`输出。还包括一些能直接执行js代码或者执行字符串的情况，比如`<a>`标签的href、setTimeout、eval等。

这种的解决方法就是尽可能避免使用。如果真的要使用，就使用更加安全的，比如innerText、setAttribute等，而不是直接输出html；

但有些情况，比如markdown转化之后拼接到dom中显示，就必须使用到html注入。这种情况可以在输出的字符串中过滤掉所有可能有风险的标签，只保留div、p等受信任的标签。

#### 防御 xss 的转码方式

参考：https://juejin.cn/post/6844903684900388871

针对 HTML 代码的编码方式是 `HTMLEncode`，它的作用是将字符串转换成 HTMLEntities。目前来说，为了对抗 XSS ，以下转义内容是必不可少的：

| 特殊字符 | 实体编码 |
| -------- | -------- |
| `&`      | `&amp;`  |
| `<`      | `&lt;`   |
| `>`      | `&gt;`   |
| `"`      | `&quot;` |
| `'`      | `&#x27;` |
| `/`      | `&#x2F;` |

这些内容会在 html 中正常显示。比如：

```html
<div>
  &amp;
  <div>
    <div>
      &lt;
      <div>
        <div>
          &gt;
          <div></div>
        </div>
      </div>
    </div>
  </div>
</div>
```

可以正常显示为原本的字符，但是这只是显示，在 html 解析时并不把他当做是标准的字符。因此比如对于`script`标签，由于左右尖括号被转译，自然也就不能执行代码了。

在进行 innerHTML 等需要直接插入 html 的部分，一定要通过 HTMLEncode 进行转译。

除了 HTML，js 中其实也需要转义，即服务端返回的 js 代码，也有可能带有 xss 攻击。详细参考 https://juejin.cn/post/6844903684900388871#heading-9

## CSRF 攻击

### 原理解释

跨站请求伪造（Cross-site request forgery），也被称为 one-click attack 或者 session riding，通常缩写为 CSRF 或者 XSRF， 是一种挟制用户在当前已登录的 Web 上执行非本意的操作的攻击方法。  
简单来说，攻击者诱导你登录一个跟你想登录的网站很类似的网站, 该网站可以利用 form 表单或者其他方式，向具有你信息的网站发送请求，网站的接口请求格式一般很容易获取到，请求会自动携带上网站的 cookie 信息，即你的身份验证信息
举个栗子
比如原先你的操作是获取你有多少钱, 这里使用的是 GET 请求

```
http://www.mybank.com/userdata?toBankId=11&money=1000
```

危险网站 B，它里面有一段 HTML 的代码如下：

```html
<img src="http://www.mybank.com/userdata?toBankId=11&money=1000" />
```

当你登录 A 的时候, 同时打开了 B 网站（不登出 A 的时候打开 B，具体来说是 A 上的 cookie 等信息没有过期），浏览器会在 B 网站上向 A 网站发出一个请求，用于请求图片。这种情况比较多见，在一个网站上请求另一个服务器上的资源，比如图片、json 等，都是合法的。因此关键在于这里获取了 cookie。

当 B 网站发送这个请求后，你的浏览器会视作向 A 服务器发送一个请求，于是会带上你的银行网站 A 的 Cookie 发出 Get 请求，去获取资源`http://www.mybank.com/userdata?toBankId=11&money=1000`，结果银行网站服务器收到请求后，认为这是一个更新资源操作（转账操作），所以就立刻进行转账操作

这个过程中，用户其实是进入了一个恶意网站 B，而这个网站可能是通过某些方式诱导用户点入的，比如广告、邮件等。一旦用户点进去这样的请求就会被发送，然后盗取用户的 cookie。

再举个栗子
比如银行使用 POST 方法更新数据

这时危险网站就可以通过表单提交的方式伪造数据，流程和上面一样。总之只要用户点击了这个链接，浏览器携带了 cookie 去发送请求，就有 csrf 的风险。

```html
<form method="POST" name="transfer" action="http://www.myBank.com/userdata">
  <input type="hidden" name="toBankId" value="11" />
  <input type="hidden" name="money" value="1000" /> 　　　　　　
</form>
```

注意这个过程中，csrf 攻击全称没有获取 cookie，也没有执行任何脚本，只是想办法构造了请求，让浏览器自己携带 cookie 发送信息。

### 防范手段

CSRF 攻击是源于 WEB 的隐式身份验证机制。WEB 的身份验证机制虽然可以保证一个请求是来自于某个用户的浏览器，但却无法保证该请求是用户批准发送的（可能是别人代发送的）。

基本的思路有几个：

1. CSRF大多来自第三方网站，可以直接禁止外域（或者不受信任的域名）发起的请求。

- 方式：通过Origin或Referer头来确定请求的来源，如果不信任就不处理
- 问题：
  - 这两个头不一定存在，比如referer在https跳http时就会丢失。这种情况，如果这两个头都不存在，就直接阻止就行。
  - 在请求页面的情况下，比如某个跳转的a标签，此时是用get请求一个html。这种情况下一般referer头都不会是信任的（比如baidu），但是我们不能过滤了，不然就不能打开页面了。但是的但是，这样就会造成疏漏，可能会出现利用页面请求攻击的情况
  ```
  GET https://example.com/addComment?comment=XXX&dest=orderId
  Accept: text/html
  Method: GET
  ```

2. Samesite Cookie属性

Samesite在上面已经讲过。其实就是设置合适的值，让恶意网站发送数据不能携带cookie，从而从根源上杜绝了这个问题。
- 问题：不能防止内部被攻破。即，如果网站内部被嵌入了恶意代码或恶意链接，那么这个请求就是来自这个源内部的，这时就不能通过samesite拦截了。

3. csrf_token 

- 方式：CSRF攻击之所以能够成功，是因为服务器误把攻击者发送的请求当成了用户自己的请求。那么我们可以要求所有的用户请求都携带一个CSRF攻击者无法获取到的Token。服务器通过校验请求是否携带正确的Token，来把正常的请求和攻击的请求区分开，也可以防范CSRF的攻击。
- 注意点：
  - token应该存在session中，防止token泄漏，并且应该加密。
  - 一次会话只需要生成一次token，并且应该是随机的。下一次登录时就会再生成新的token
- 问题：太复杂，而且会给服务端造成压力，因为加解密、校验都是需要时间的。



## SYN 攻击

### 原理

SYN 攻击是针对 TCP 建立连接过程的。
TCP 连接建立是需要三次握手，假设攻击者短时间伪造不同 IP 地址的 SYN 报文（相当于同时和服务器建立很多个 TCP 连接），服务端每接收到一个 SYN 报文，就进入 SYN_RCVD 状态，但服务端发送出去的 ACK + SYN 报文，无法得到未知 IP 主机的 ACK 应答，久而久之就会占满服务端的半连接队列，使得服务端不能为正常用户服务。

半连接队列和全连接队列可以参考这张图：

![](https://pic.imgdb.cn/item/6415af8da682492fccc2e500.jpg)

当客户端发送第一次握手的 SYN 时，服务端会存在 Linux 内核中的 SYN 队列中。接着发送 SYN + ACK 给客户端，等待客户端回应 ACK 报文；当客户端返回 ACK 报文时，服务端才会从半连接队列中取出 SYN 对象，然后创建一个新的连接对象放入到「 Accept 队列」。
如果攻击者伪造了大量的 TCP 连接，最直接的表现就会把 TCP 半连接队列打满，这样当 TCP 半连接队列满了，后续再在收到 SYN 报文就会丢弃，导致客户端无法和服务端建立连接。

### 解决

避免 SYN 攻击方式，可以有以下四种方法：

1. 增大 TCP 半连接队列；这个是最直接的方法
2. 开启 tcp_syncookies；这个功能可以在 SYN 队列满状态下依然建立连接，相当于绕过了 SYN 半连接来建立连接。
3. 减少 SYN+ACK 重传次数：当服务端受到 SYN 攻击时，就会有大量处于 SYN_REVC 状态的 TCP 连接，处于这个状态的 TCP 会重传 SYN+ACK ，当重传超过次数达到上限后，就会断开连接。那么针对 SYN 攻击的场景，我们可以减少 SYN-ACK 的重传次数，以加快处于 SYN_REVC 状态的 TCP 连接断开。（断开之后重新建立连接就相当于清空 SYN 队列）

## DDoS 攻击

DDoS 是英文 Distributed Denial of Service 的缩写，意即“分布式拒绝服务”。可以这么理解，凡是能导致合法用户不能够访问正常网络服务的行为都算是拒绝服务攻击。也就是说拒绝服务攻击的目的非常明确，就是要阻止合法用户对正常网络资源的访问，从而达成攻击者不可告人的目的。分布式拒绝服务攻击一旦被实施，攻击网络包就会从很多 DOS 攻击源犹如洪水般涌向受害主机，从而把合法用户的网络包淹没，导致合法用户无法正常访问服务器的网络资源

DDoS 攻击的常见类型

- SYN 攻击，即上面讲的 SYN 攻击方式，大量发送连接请求，占满服务器的半连接队列
- HTTP Flood 攻击：攻击者向目标服务器发送大量的 HTTP 请求，以耗尽服务器的资源。攻击者通常会模拟正常的 HTTP 请求，使得服务器难以区分正常请求和攻击请求。
- UDP Flood 攻击：UDP Flood 攻击是一种利用 UDP 协议的 DDoS 攻击，攻击者向目标服务器发送大量的 UDP 数据包，以使其服务受到影响。由于 UDP 协议没有建立连接的过程，因此攻击者可以轻松地伪造源 IP 地址，使得攻击更加难以追溯。

除此之外还有 ICMP 攻击、DNS 攻击等方式。

防范方式：

- 使用 DDoS 保护服务，有许多公司提供 DDoS 保护服务，可以针对 DDoS 攻击提供额外的保护层。这些服务可以实时检测和缓解 DDoS 攻击，让服务器继续正常运行。
- 增加服务器带宽：增加服务器带宽有助于吸收 DDoS 攻击的影响。攻击者经常使用大量请求使服务器过载，增加带宽有助于处理增加的流量。
- 实施速率限制：速率限制可以限制服务器可以从单个 IP 地址接收的请求数量，这有助于减轻 DDoS 攻击的影响。这可以使用各种技术来完成，例如防火墙规则或专用软件。

## DNS劫持

DNS劫持一般是指修改了某些位置上的dns映射关系，使得原本正常解析的域名解析成了错误的ip。
dns劫持的发生位置一般有几种：

- 发生在本机，即当前的用户主机上，可能因为感染了病毒，被修改了操作系统中内置的host文件（dns查询的第一个路径），或者修改了浏览器的缓存。、
- 发生在本地dns服务器，也就是运营商提供的LDNS服务器上。一般是直接攻击这些服务器，感染缓存或影响查询结果。

对于dns劫持的解决和预防主要有一些策略

首先对于用户主机来说，要先确认是否发生了dns劫持。确认的方法比如：
- 通过一个定时器定期向dns服务器发出解析请求。如果某一次的解析结果和之前不一样，或者和以前的缓存内容不一样，那么就可能是本地的本篡改，或者LDNS发生了dns劫持。
- 某些LDNS服务器发生劫持时，会主动向用户主机发送警告

意识到了之后，就可以通过避免的方式来解决
- 如果是轮询发现了解析出问题，那么就选择不信任这个解析结果，转而向其他的本地dns服务器请求。这里有时候需要修改host等系统配置内容
- 如果是有发来的预警，其实也是同理，选择其他dns服务器。
- 还有一种情况，就是一个域名一般会对应多个ip，或者一个网站可以通过多个域名访问。如果一个ip被dns劫持，那么可以访问其他域名或者其他ip来绕过。
- HttpDNS ：HttpDNS是使用 HTTP 协议向 DNS 服务器的 80 端口进行请求，代替传统的 DNS 协议向 DNS 服务器的 53 端口进行请求，绕开了运营商的 Local DNS，从而避免了使用运营商 Local DNS 造成的劫持和跨网问题。一般情况下的dns是由dns协议控制，udp传输，而httpDNS则是http和tcp一同完成的。

## 其他安全问题

### localStorage / sessionStorage 的安全问题

安全问题主要有两个：

1. xss 攻击直接读取。localStorage 的主要风险在于 xss 攻击。而 localstorage 必须通过 js 访问，因此也不能像 cookie 那样设置一个 httponly 就了事。
2. 恶意注入 localstorage，即攻击者通过恶意注入大量垃圾内容来消耗 localstorage 的空间。

解决方式：

1. 防范 xss 攻击，参考上面的方案
2. 对 localstorage 保存的关键数据做定期失效。比如用户的 jwt，需要服务端配合做定期失效，尽可能不要保持太长时间
3. 对于失效的 jwt 等数据要做及时清除

### jwt 安全问题

#### jwt 基本结构

jwt 的基本结构：

1. Header（头部）

Header 部分是一个 JSON 对象，描述 JWT 的元数据，通常是下面的样子。

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

jwt 的头部承载两部分信息：

声明类型：typ 属性表示这个令牌（token）的类型（type），JWT 令牌统一写为 JWT。
声明加密的算法：通常直接使用 HMAC SHA256（写成 HS256）

2. Payload（负载）

Payload 就是存放有效信息的地方。这个名字像是特指飞机上承载的货品，这些有效信息包含三个部分

- 标准中注册的声明
  - ss: jwt 签发者
  - sub: jwt 所面向的用户
  - aud: 接收 jwt 的一方
  - exp: jwt 的过期时间，这个过期时间必须要大于签发时间
  - nbf: 定义在什么时间之前，该 jwt 都是不可用的.
  - iat: jwt 的签发时间
  - jti: jwt 的唯一身份标识，主要用来作为一次性 token,从而回避重放攻击。
- 公共的声明: 公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息.但不建议添加敏感信息，因为该部分在客户端可解密.
- 私有的声明: 私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为 base64 是对称解密的，意味着该部分信息可以归类为明文信息。

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

3. Signature（签名）

Signature 部分是对前两部分的签名，防止数据篡改。
首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。
然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照一定的公式用服务端保存的私钥产生签名。
当服务端收到 jwt 时，会解析并验证签名部分，判断内容是否被修改过。

生成 JWT 的过程如下：

- 将头部和载荷分别用 Base64 编码，并用点号连接起来，形成一个字符串。
- 使用私钥对该字符串进行签名，生成签名部分。
- 将头部、载荷和签名三部分用点号连接起来，形成最终的 JWT 字符串。

验证 JWT 的过程如下：

- 将 JWT 字符串按点号拆分成头部、载荷和签名三部分。
- 对头部和载荷部分进行 Base64 解码，然后将它们用点号连接起来，形成一个字符串。
- 使用公钥对该字符串进行验证签名，如果签名验证通过，就说明 JWT 的内容没有被篡改过，可以信任其中的信息。

#### jwt 安全问题

1. 敏感信息泄露：我们能够轻松解码 payload 和 header，因为这两个都只经过 Base64Url 编码，而有的时候开发者会误将敏感信息存在 payload 中。
2. 密钥的泄露：即 jwt 第三部分的密钥，通常保存在服务端。不过如果可能通过某些手段破解或泄露的话，就容易导致数据的篡改。
