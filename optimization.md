---
title: 优化问题总结
date: 2022-04-05 16:01:22
tags: 面试
categories: 原理
cover:
---

![](https://pic.imgdb.cn/item/637d175716f2c2beb1133594.jpg)

优化的方式可以从这几个方向出发：

1. 网络优化

- http2.0
- cdn
- dns 预解析
- 预连接
- 压缩

2. 缓存优化

- http 缓存，充分利用 cache-control 字段
- 运行时缓存，利用好 cookie 等
- 资源文件的文件指纹，即文件名和内容相关。内容不改变就一直使用缓存

3. 静态资源优化

- 预加载
- 懒加载
- 控制体积、压缩 js、css、html、图片等静态资源
- 图片选择合适的格式

4. 运行时优化（即 css、js 代码的执行过程）

- html
- css
- js
- webpack 构建优化
- react 优化

5. 请求优化（即发送的网络请求）

- 并发请求
- jwt 等方式加快服务端响应速度

# 网络相关优化

## HTTP

在 HTTP1.1 中，主要的优化方式是减少请求数量。由于队头阻塞的存在，同一个 TCP 连接中，只有前一个 HTTP 请求完成，后一个才会继续。因此减少请求数量也就是相当于优化了请求的总时间。
常用的减少请求数量的方法有：

1. 减少关键资源数量、降低关键资源大小，比如合并 CSS、JavaScript 和图片
2. 充分利用缓存，尽可能减少资源的重复请求
3. DNS 和 CDN，详见下
4. 减少 HTTP 请求和响应的大小；请求大小可以通过最小化数据实现，响应大小则主要通过压缩代码（webpack 的那几种压缩代码的方式）、gzip 压缩资源（主要通过在 nginx 中配置，或者服务端控制开启）等方式减小
5. 减少不必要的重定向。当站点发生迁移时，重定向会导致一圈额外的浏览器到服务端往返，这会增加不必要的延迟；
6. 细分域名。浏览器对于最大 TCP 连接数量有限制，但是这个限制是针对一对域名之间（一个客户端和一个确定域名的服务端）的；如果把资源分散到为不同的域名上，就不会受到最大 TCP 连接的限制。

---

以上主要是针对 HTTP1.1 的以及一些通用的优化 HTTP 的方法。对于 HTTP2，则优化策略略有不同。
HTTP2 带来了流和帧实现的多路复用，在一个 TCP 连接中不会存在队头阻塞的问题。因此，像 HTTP1.1 的主要优化方式（减少请求数量）就没有必要，反倒是可以通过增加请求数量减少单个请求的大小，加块请求速度。
具体来说，主要是：

1. 传输轻量、细粒度的资源，以便独立缓存和并行传输。
2. 减少资源和文件的合并
3. 停止细分域名。同样是多路复用带来的优化，细分域名不但没有意义，还会增加由于额外的 DNS 查询、TCP 连接和 TLS 握手带来的时间消耗。

## DNS

优化 DNS。这一步主要是通过 HTML 的`<link rel="dns-prefetch">`实现。这个 rel 可以对 dns 进行预解析（当然不包括这个 html 本身的 dns），将 html 内部的 link 链接提前进行 dns 解析，加快请求速度。

与之配合的还有`<link rel="preconnect">`，表示预连接。preconnect 会建立与服务器的连接。如果站点是通过 HTTPS 服务的，则此过程包括 DNS 解析，建立 TCP 连接以及执行 TLS 握手。配合 dns 预解析，能够最大程度预处理 HTTP 请求的前期过程。

## CDN

内容分发网络（CDN）是一组分布在多个不同地理位置的 Web 服务器。我们都知道，当服务器离用户越远时，延迟越高。CDN 就是为了解决这一问题，在多个位置部署服务器，让用户离服务器更近，从而缩短请求时间。

- **使用第三方的 CDN 服务**：如果想要开源一些项目，可以使用第三方的 CDN 服务
- **使用 CDN 进行静态资源的缓存**：将自己网站的静态资源放在 CDN 上，比如 js、css、图片等。可以将整个项目放在 CDN 上，完成一键部署。
- **直播传送**：直播本质上是使用流媒体进行传送，CDN 也是支持流媒体传送的，所以直播完全可以使用 CDN 来提高访问速度。CDN 在处理流媒体的时候与处理普通静态文件有所不同，普通文件如果在边缘节点没有找到的话，就会去上一层接着寻找，但是流媒体本身数据量就非常大，如果使用回源的方式，必然会带来性能问题，所以流媒体一般采用的都是主动推送的方式来进行。

## 压缩

压缩主要有两种压缩，一种是通过 gzip 压缩报文体，还有一种是借助 webpack 压缩代码。这里说一下前者

gzip 是 GNUzip 的缩写，最早用于 UNIX 系统的文件压缩。HTTP 协议上的 gzip 编码是一种用来改进 web 应用程序性能的技术，Web 服务器和客户端（浏览器）必须共同支持 gzip。目前主流的浏览器，Chrome，firefox，IE 等都支持该协议。常见的服务器如 Apache，Nginx，IIS 同样支持，gzip 压缩效率非常高，通常可以达到 70% 的压缩率，也就是说，如果你的网页有 30K，压缩之后就变成了 9K 左右

gzip 可以通过 nginx 配置，也可以通过 express：

```js
const compression = require("compression");
const app = express();
app.use(compression());
```

设置了 gzip 之后，会出现在报文头部的`Content-Encoding: gzip`中。
大多数情况下 gzip 会出现在响应体的压缩中，当然请求体也可以压缩来减少流量消耗。

## 防抖节流

详见 https://xingyeblog.top/2021/12/05/blog30-js-deeping/

# 静态资源优化

## 懒加载

懒加载也叫做延迟加载、按需加载，指的是在长网页中延迟加载图片数据，是一种较好的网页性能优化的方式。如果使用图片的懒加载就可以解决以上问题。在滚动屏幕之前，可视化区域之外的图片不会进行加载，在滚动屏幕时才加载。懒加载适用于图片较多，页面列表较长（长列表）的场景中。

实现原理：主要是通过判断元素是否到达用户可视窗口，在快到达之前提前进行加载。
判断是否到达可视区域的方式详见 https://xingyeblog.top/2021/12/10/blog32-css-deeping/ ，总之核心公式是这样：

```js
el.offsetTop - document.documentElement.scrollTop <= viewPortHeight;
```

如果这个条件为 true，则开始进行加载。加载的方式可以是设置图片的 src 为有效值即可。

## 预加载

预加载方案主要有：

- 浏览器机制：`<link>`的 prefetch 和 preload；
- webpack 魔法注释：通过魔法注释可以直接通过 webpack 实现 prefetch & preload；
- 应用框架工具库：比如 react-loadable；

### preload 和 prefetch

preload：对资源进行预加载，加载会在 html 的主要渲染之前，可以通过在预加载标签中添加一些媒体元素或者包含请求的 js 来加快渲染之后的资源加载速度。一般需要和 as 属性配合确定预加载的资源属于哪种类型

```html
<link rel="preload" href="style.css" as="style" />
<link rel="preload" href="main.js" as="script" />
```

可以预加载的类型很多，as 的属性值可能是各种媒体元素，比如：

```html
<link rel="preload" href="music.mp3" as="audio" />
<link rel="preload" href="pic.png" as="image" />
```

和 preload 对应的还有 prefetch。prefecth 和 preload 的主要区别体现在预加载的资源什么时候使用、给不给当前页面使用

- preload 是一个声明式 fetch，可以强制浏览器在不阻塞 document 的 onload 事件的情况下请求资源。preload 的优先级更高，会强制浏览器立即开始请求资源，不管用不用的上
- prefetch 告诉浏览器这个资源将来可能需要，但是什么时间加载这个资源是由浏览器来决定的。并且请求到资源不一定要给本页面使用，可能是其他第三方页面或者子页面。preftech 请求到的资源会将其放入缓存至少 5 分钟（无论资源是否可以缓存）；并且，当页面跳转时，未完成的 prefetch 请求不会被中断；

### 通过 css

把要加载的图片 url 设置为某个元素的背景并在 css 中引入，然后通过设置 opacity 或者 z-index 的方式隐藏该元素即可。

```css
#preload-01 {
  background: url(http://domain.tld/image-01.png) no-repeat -9999px -9999px;
}
#preload-02 {
  background: url(http://domain.tld/image-02.png) no-repeat -9999px -9999px;
}
#preload-03 {
  background: url(http://domain.tld/image-03.png) no-repeat -9999px -9999px;
}
```

### 通过 js

创建一个 new Image 实例并设置 src，当执行到设置 src 时，浏览器就会请求该图片。因此并不需要真正的插入，只需要引入即可。
示例：

```js
const srcList = ['src1','src2','src3',...]
function preLoad() {
    for (let i = 0; i < srcList.length; i++) {
        var img = new Image()
        img.src = srcList[i]
        img.onload = img.onerror = function () {	// 无论加载成功还是失败都会执行
            //
        }
    }
}
```

当然实际的 preload 实现要考虑的很多，因此可以参考一个库 PreloadJS

## 图片

不同的图片要采取不同的加载方式：

![](https://pic.imgdb.cn/item/6400cd08f144a0100754a607.jpg)

除此之外，针对图片还有一些优化方式：

- 小图合并：即 css 雪碧图等方式，降低请求次数
- 媒体查询：小屏使用小图片，无需请求高分辨率的图片。通常可以通过图片的 srcset 属性控制，或者通过 picture 和 source 配合在小屏使用小图片。
- 使用 webp 格式的图片：webp 可以极大程度上减少 png/jpg 格式的图片大小。webp 本质是一种图片的压缩，分为有损和无损 webp。但即便是无损 webp，也能减少图片的大小。
- 图片压缩：webpack 的有些 loader 和 plugin 可以压缩图片。最常见的就是 file-loader 和 url-loader，可以设置图片的最大大小，起到压缩作用。`image-webpack-loader`是专门压缩图片的 loader

# 运行时优化

通常一个页面有三个阶段：加载阶段、交互阶段和关闭阶段

- 加载阶段，是指从发出请求到渲染出完整页面的过程，影响到这个阶段的主要因素有网络和 JavaScript 脚本。
- 交互阶段，主要是从页面加载完成到用户交互的整合过程，影响到这个阶段的主要因素是 JavaScript 脚本。
- 关闭阶段，主要是用户发出关闭指令后页面所做的一些清理操作

因此可以得到影响浏览器加载的主要因素：

1. **加载阶段**
   - 关键资源个数。关键资源个数越多，首次页面的加载时间就会越长。（能阻塞网页首次渲染的资源称为关键资源）
   - 关键资源大小
2. **交互阶段**，即渲染进程渲染帧的速度
   - 减少重绘重排，详见下
   - 减少 js 脚本执行时间

## 针对 JavaScript

JavaScript 既会阻塞 HTML 的解析，也会阻塞 CSS 的解析。因此我们可以对 JavaScript 的加载方式进行改变，来进行优化：

1. 尽量将 JavaScript 文件放在 body 的最后，body 中间尽量不要写`<script>`标签。`<script>`标签的引入资源方式有三种，有一种就是我们常用的直接引入，还有两种就是使用 async 属性和 defer 属性来异步引入，两者都是去异步加载外部的 JS 文件，不会阻塞 DOM 的解析（尽量使用异步加载）

> 需要注意的是，无论是哪种方式引入的js文件都会导致页面渲染过程的阻塞，即使是defer形式的script，也会在dom构建完成后，浏览器暂停去等待js代码的执行。

2. 不要覆盖原生方法，即 js 的内置方法，比如 sort；js 的内置方法通常是经过极力优化的，并且这部分代码也会在后期的编译中变成“热点代码”而直接成为机器码，相对于自己写的方法来说要快很多很多。
3. 简化 dom 操作，不要频繁操作 dom，不要操作顶层 dom，可以通过事件委托的形式给多个元素安插一个事件，从而减少事件的监听数量
4. 减少使用 js 动画，css 动画的性能远高于 js；如果一定要使用，requestAnimationFrame 要优于其他
5. 防抖和节流
6. 代码内部的优化，比如减少闭包、减少无用代码、压缩代码、代码质量优化等

## 针对 CSS

使用 CSS 有三种方式：使用 link、@import、内联样式，其中`link`和`@import`都是导入外部样式。它们之间的区别：

- `link`：浏览器会派发一个新线程(HTTP 线程)去加载资源文件，与此同时 GUI 渲染线程会继续向下渲染代码（构建 DOM），如果已经完成构建，就会等待 css 脚本的下载；如果期间还有 js 脚本，两者会并行下载，最终都下载完成才继续渲染。
- `@import`：GUI 渲染线程会**暂时停止渲染**，去服务器加载资源文件，资源文件没有返回之前不会继续渲染(阻碍浏览器渲染)；
- `style`：GUI 直接渲染

外部样式如果长时间没有加载完毕，浏览器为了用户体验，会使用浏览器会默认样式，确保首次渲染的速度。所以 CSS 一般写在`<head>`中，让浏览器尽快发送请求去获取 css 样式。
所以，在开发过程中，导入外部样式使用 link，而不用`@import`。如果 css 少，尽可能采用内嵌样式，直接写在 style 标签中。

> 关于`@import`和`<link>`加载 css 的方式，网上基本上都在说`@import引用的CSS会等到页面全部被下载完再被加载`。
> 但是经过实测，以浏览器网络监听的加载顺序为例，这两者的加载顺序是按照从上到下正序排列的；也就是说如果包含`@import`的 style 标签在 link 之前，他也会在 link 之前加载，并不是所说的一定在页面加载完之后才加载

### 加载性能

1. css 压缩：将写好的 css 进行打包压缩，可以减小文件体积。
2. css 单一样式：即减少缩写，直接指明具体的属性会更快。
3. 减少使用@import，建议使用 link，因为后者在页面加载时一起加载，前者是等待页面加载完成之后再进行加载。

### 选择器性能

3. 避免使用通配规则，如`*{}`计算次数惊人，只对需要用到的元素进行选择。
4. 多用 class 指明具体元素。
5. 尽量少的去使用后代选择器，降低选择器的权重值。后代选择器的开销是最高的，尽量将选择器的深度降到最低，最高不要超过三层，更多的使用类来关联每一个标签元素。
6. 了解哪些属性是可以通过继承而来的，然后避免对这些属性重复指定规则。

### 渲染性能

1. 慎重使用高性能属性：浮动、定位。
2. 尽量减少页面重排、重绘。
3. 去除空规则：｛｝。空规则的产生原因一般来说是为了预留样式。去除这些空规则无疑能减少 css 文档体积。
4. 属性值为 0 时，不加单位。
5. 属性值为浮动小数 0.xx，可以省略小数点之前的 0。
6. 标准化各种浏览器前缀：带浏览器前缀的在前。标准属性在后。
7. 不使用@import 前缀，它会影响 css 的加载速度。
8. 选择器优化嵌套，尽量避免层级过深。
9. css 雪碧图，同一页面相近部分的小图标，方便使用，减少页面的请求次数，但是同时图片本身会变大，使用时，优劣考虑清楚，再使用。
10. 正确使用 display 的属性，由于 display 的作用，某些样式组合会无效，徒增样式体积的同时也影响解析性能。
11. 不滥用 web 字体。对于中文网站来说 WebFonts 可能很陌生，国外却很流行。web fonts 通常体积庞大，而且一些浏览器在下载 web fonts 时会阻塞页面渲染损伤性能。

### 可维护性、健壮性

1. 将具有相同属性的样式抽离出来，整合并通过 class 在页面中进行使用，提高 css 的可维护性。
2. 样式与内容分离：将 css 代码定义到外部 css 中。

## 减少回流与重绘

- CSS
  - 避免使用 table 布局。尽可能采用 flex、grid 等新式布局方案
  - 尽可能在 DOM 树的最末端改变 class。
  - 避免设置多层内联样式。
  - 将动画效果应用到 position 属性为 absolute 或 fixed 的元素上。
  - 避免使用 CSS 表达式（例如：`calc()`）。

- JavaScript
  - 避免频繁操作样式，最好一次性重写 style 属性，或者将样式列表定义为 class 并一次性更改 class 属性。
  - 避免频繁操作 DOM，创建一个 documentFragment，在它上面应用所有 DOM 操作，最后再把它添加到文档中。
  - 也可以先为元素设置 display: none，操作结束后再把它显示出来。因为在 display 属性为 none 的元素上进行的 DOM 操作不会引发回流和重绘。
  - 避免频繁读取会引发回流/重绘的属性，如果确实需要多次使用，就用一个变量缓存起来。
  - 对具有复杂动画的元素使用绝对定位，使它脱离文档流，否则会引起父元素及后续元素频繁回流。

## webpack 优化

详见 https://xingyeblog.top/2022/01/09/blog35-webpack/

# 性能指标

## RAIL 性能模型

RAIL 是 Response, Animation, Idle, 和 Load 的首字母缩写, 是⼀种由 Google Chrome 团队于 2015 年提出的性能模型, ⽤于提升浏览器内的⽤户体验和性能。

![](https://pic.imgdb.cn/item/637df49e16f2c2beb12563cf.png)

这个名字的由来是四个英⽂单词的首字母：

- 响应（Response）：应该尽可能快速的响应⽤户, 应该在 100ms 以内响应⽤户输⼊。
- 动画（Animation）：在展示动画的时候，每⼀帧应该以 16ms 进⾏渲染，这样可以保持动画效果 的⼀致性，并且避免卡顿。
- 空闲（Idle）：当使⽤ Javascript 主线程的时候，应该把任务划分到执行时间小于 50ms 的片段中去，这样可以释放线程以进行用户交互。
- 加载（Load）：应该在⼩于 1s 的时间内加载完成你的⽹站，并可以进⾏⽤户交互。
  这四个单词代表与⽹站或应⽤的⽣命周期相关的四个⽅⾯，这些⽅⾯会以不同的⽅式影响整个⽹站的性能

## 基于用户体验的性能指标

以下性能都可以参考`https://web.dev/fcp/`，把 FCP 换成其他渲染指标也可以查看

首先是首屏渲染的指标，主要表示从用户输入 URL 并回车开始，到加载出可视元素的这一段的过程。

- load（onload），它代表页面中依赖的所有资源加载完的事件。
- DCL（DOMContentLoaded），DOM 解析完毕。
- FP（First Paint），表示渲染出第一个像素点。FP 一般在 HTML 解析完成或者解析一部分时候触发。FP 事件的触发也可以被看作是白屏时间的结束，即从输入 URL 回车开始，到显示出页面上的第一个元素的时间被称为**白屏时间**，白屏时间的结束就是 FP 事件的触发。
- FCP（First Contentful Paint），表示渲染出第一个内容，这里的“内容”可以是文本、图片、canvas。FCP 的触发可以看作是**首屏时间**的结束，表示浏览器第一屏渲染完成
- FMP（First Meaningful Paint），首次渲染有意义的内容的时间，“有意义”没有一个标准的定义，FMP 的计算方法也很复杂。大体思路是，将页面中最大布局变化后的第一次渲染事件作为 FMP 事件，并且计算中考虑到了可视区的因素。FMP 在逐渐被 LCP 代替，因为相对于 LCP 和 FCP，FMP 不太好确定具体的细节。
- LCP（largest contentful Paint），最大内容渲染时间，具体是指页面开始加载到最大文本块内容或图片显示在页面中（必须是渲染完成）的时间。LCP 是对以往的通过 load、DCL 判断渲染性能的方式的改进，也就是说可以不使用 load、DCL，而是改用判断 LCP。
  由于页面的动态加载，最大的元素可能发生变化。比如图片加载出来之前最大元素是某个 div，加载出来后最大元素可能就是 img；为了解决这种情况，浏览器会在绘制第一帧后立即分发一个`largest-contentful-paint`类型的`PerformanceEntry`，用于识别最大内容元素。如果最大内容元素发生变化，就会分发另一个 PerformanceEntry，其中的 element 属性就会更改为新的最大元素。
  对比 FCP 和 LCP，可以看到 LCP 在图片加载出来后发生变化
  ![](https://pic.imgdb.cn/item/637ddfbb16f2c2beb103db56.jpg)

load、DCL 都可以直接通过监听 window 对象上的对应事件监听，FC/FCP 则可以通过 preformance 对象上的方法访问

---

上面的几个都是加载时间的体现，除此之外还有可交互时间的一些指标

- TTI（Time to Interactive），首次可交互时间，计算的是页面从开始加载到主要资源渲染完成并能响应用户交互的所需时间。具体计算是这样：
  1. 先进行 First Contentful Paint 首次内容绘制 (FCP)。
  2. 沿时间轴正向搜索时长至少为 5 秒的安静窗口，其中，安静窗口的定义为：没有长任务（执行时间超过 50ms 的任务）且安静窗口内最多并行有两个正在处理的网络 GET 请求。
  3. 沿时间轴反向搜索安静窗口之前的最后一个长任务，如果没有找到长任务，则在 FCP 步骤停止执行。
  4. TTI 是从页面渲染开始到安静窗口之前最后一个长任务结束的中间这段的时间。（如果没有找到长任务，则与 FCP 值相同）。
  如图所示：
  ![](https://pic.imgdb.cn/item/637de1a216f2c2beb106b667.jpg)
- FID（First input delay），首次输入延迟，即计算 FCP 和 TTI 之间的时间。通常情况下，FCP 只是保证渲染出了页面上的元素，比如一些按钮、链接等；当用户尝试交互时，由于 js 代码还在执行或其他任务的阻塞，可能导致“点击没有效果”，直到 TTI 结束才可以交互。因此就有必要缩减从“看到按钮”到“可以点击按钮”之间的时间，这就是 FID 表示的值。
- TBT（Total blocking time）总阻塞时间。记录从 FCP 到 TTI 之间的所有长任务的阻塞时间总和
- CLS（Cumulative layout shift）累积布局偏移。用于衡量页面中某个元素的位置改变量。比如说页面突然出现的一个很大的元素导致在原位的按钮被挤下去，会导致用户体验下降。
  CLS 的计算依赖于渲染帧的变化；每当一个可见元素的位置从一个已渲染帧变更到下一个已渲染帧时，就发生了布局偏移 。一连串的布局偏移，也叫会话窗口，是指一个或多个快速连续发生的单次布局偏移，每次偏移相隔的时间少于 1 秒，且整个窗口的最大持续时长为 5 秒。最大的一连串是指窗口内所有布局偏移累计分数最大的会话窗口。
  如图所示，下面一共有三个窗口，每个窗口内的条形表示布局偏移量，当布局偏移结束 1s 后就会结束当前窗口。CLS 的值就是和最大的那个窗口的值
  ![](https://pic.imgdb.cn/item/637de4f816f2c2beb10d9130.jpg)
- Speed Index 速度指数，用于表示页面可见区域内容显示的平均速度。

---

在以上这些指标中，最核心的指标其实是三个：
![](https://pic.imgdb.cn/item/637de6e116f2c2beb11047df.jpg)

## 性能的检测和计算方式

### performance

首先可以通过原生 js api 测量。最核心的就是 Performance 对象；这个对象可以获取到当前页面中与性能相关的信息
Performance 对象常用的方法有以下几个：

1. `performance.now`，返回一个精确的时间戳，它相比于 Date.now 跟精确的地方在于它实际上是一个浮点数，精度可以达到微秒级
2. `performance.getEntries`，返回一个数组，数组的每一项都是一个 performanceEntry 对象。这些对象大多数都是 script、img、css 等资源，少数是诸如 FP、FCP 等的性能监控指标。

> PerformanceEntry 对象
> PerformanceEntry 对象可以被看作是一种性能条目，页面加载过程中所有资源都会生成一个条目。除此之外还可以通过 performance.mark 来创建自定义条目
> PerformanceEntry 对象包含一些属性，其中根据类型不同，并不是所有属性都有。常见的可能会是这样（资源条目）：
> ![](https://pic.imgdb.cn/item/637dfa0116f2c2beb12c7b4b.jpg)
> 如果是性能监控条目（比如 FP 和 FCP）就是这样：
> ![](https://pic.imgdb.cn/item/637dfa4116f2c2beb12cc1a2.jpg)
> 常用的属性有：
>
> - name：表示该条目的名字。如果通过`getEntriesByName()`获取具体的 entry，参数就是这个 name
> - duration：该事件的持续时间。通常对于类型是资源的条目，duration 表示资源的请求时间；而类似 FP、FCP 则该属性为 0
> - startTime：该事件的开始时间。开始时间的基准应该是从页面开始加载算起，因此对于 FP/FCP 来说，startTime 就表示该项监控的值，级从开始到触发 FP 事件一共走过了多少时间。
> - entryType：该条目的类型。资源的话这一项的值是`resource`，而 FP/FCP 这一项为`paint`，初次之外还有 mark（通过`performance.mark()`创建的标签）、measure（通过`performance.measure()`创建的测量）、frame 等类型

3. `performance.mark()`：创建一个标记，也就是说创建了一个 PerformanceEntry；这个标记可以在某段代码之前创建，然后在这段代码之后再创建一个 mark。通过`performance.measure`就可以计算两个标记之间的 duration，从而计算某段代码的同步耗时。
   `performance.measure(measureName,startMark,endMark)`传入三个参数，并创建一个 PerformanceEntry。后续可以通过获取对应的 entry 得到测量的结果；
   比如：

```js
performance.mark("beginSquareRootLoop");
for (let i = 0; i < 1000000; i++) {
  var ii = Math.sqrt(i);
}
performance.mark("endSquareRootLoop");
performance.measure(
  "measureSquareRootLoop",
  "beginSquareRootLoop",
  "endSquareRootLoop"
);
console.log(performance.getEntriesByName("beginSquareRootLoop"));
// {detail: null,
// name: "beginSquareRootLoop",
// entryType: "mark",
// startTime: 3745.360000000801,
// duration: 0}
console.log(performance.getEntriesByName("measureSquareRootLoop"));
// {detail: null,
// name: "measureSquareRootLoop",
// entryType: "measure",
// startTime: 3745.360000000801, // beginSquareRootLoop的开始时间
// duration: 9.904999984428287 // 即两个mark之间的耗时
//}
```

---

因此，获取 FP 和 FCP 的方法为：

```js
// FP
const fp = performance
  .getEntries("paint")
  .filter((entry) => entry.name == "first-paint")[0].startTime;
// FCP
const fcp = performance
  .getEntries("paint")
  .filter((entry) => entry.name == "first-contentful-paint")[0].startTime;
```

上面的 api 可以得到 FP 和 FCP 的值。对于 LCP，则需要通过`PerformanceObserver()`来监控。
这个 Observer 会在浏览器每次产生一个新的 PerformanceEntry 时调用其回调函数。通过`new PerformanceObserver()`得到一个 Observer 对象

```js
const observer = new PerformanceObserver(
  (
    performanceEntriesList, // 一个PerformanceEntry对象，可以通过getEntries获取列表。这个列表的值并不是全部entry，而是下面指定的监控的entry
    observer
  ) => {
    //...
  }
);
observer.observe({ entryTypes: ["mark", "frame"] }); // entryTypes的值就是PerformanceEntry对象的entryTypes值，包括mark、paint。resource等
observer.observe({ type: "first-input" }); // 还可以是type属性，值为PerformanceEntry对象的name值
```

通过这个 api，可以获取 LCP、FID 的值：

```js
// LCP
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    console.log("LCP candidate:", entry.startTime, entry);
  }
}).observe({ type: "largest-contentful-paint", buffered: true });

// FID
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    const delay = entry.processingStart - entry.startTime;
    console.log("FID candidate:", delay, entry);
  }
}).observe({ type: "first-input", buffered: true });
```

CLS 的值也是通过 Oberserver，但他并不是计算时间，而是计算偏移量总和：

```js
// CLS
let cls = 0;
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    if (!entry.hadRecentInput) {
      // 500ms内是否有用户交互
      cls += entry.value;
      console.log("Current CLS value:", cls, entry);
    }
  }
}).observe({ type: "layout-shift", buffered: true });
```

### web-vitals

实际开发中有一个 web-vitals 库，通过它就可以建议获取三大关键指标的值
https://www.npmjs.com/package/web-vitals

```js
import { getLCP, getFID, getCLS } from "web-vitals";

getCLS(console.log);
getFID(console.log);
getLCP(console.log);
```

# 性能检测工具

对于页面性能的分析和优化之后的检验，除了肉眼查看之外，还需要一些自动化工具。常用的有：

- Lighthouse
- PageSpeed Insights：这是一个在线工具，可以对已经线上的网站进行性能检测，类似于 lighthouse
- Chrome DevTools：可以通过 devTools 的性能模块录制并检查性能问题
- Search Console：类似 PageSpeed Insights，也是一个在线检查网站性能的工具
- Web Vitals 扩展

## LightHouse

LightHouse 是谷歌浏览器自带的一个性能检测工具。通过 LightHouse 测试页面，chrome 会给出上面各个指标的计算结果，并给出优化建议。
![](https://pic.imgdb.cn/item/637de71e16f2c2beb1108c9c.jpg)
![](https://pic.imgdb.cn/item/637de75c16f2c2beb110cc5a.jpg)
