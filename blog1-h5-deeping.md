---
title: 前端知识深入--HTML5深入
date: 2021-12-05 12:53:28
tags: 日常学习
cover: /img/Chrome.png
categories: HTML
sticky: 8
---

# 标签

## <!DOCTYPE>

文档类型定义（DTD）可定义合法的 XML 文档构建模块。它使用一系列合法的元素来定义文档的结构。
为了规定文档类型，需要 doctype 制定文档类型，html 文件头部的`<!DOCTYPE html>`就是指明文档是 html 型的。
浏览器会根据 doctype 区分标准模式和怪异模式。在怪异模式下，有些样式会和标准模式存在差异，而 html 标准和 dom 标准值规定了标准模式下的行为，没有对怪异模式做出规定，因此不同浏览器在怪异模式下的处理也是不同的，所以一定要在 html 开头使用 doctype。
doctype 大小写都可以，但一般采用完全大写的形式，以便区别于正常的 HTML 标签。因为`<!doctype>`本质上不是标签，更像一个处理指令。

## `<link>`标签

HTML 外部资源链接元素 (`<link>`) 规定了当前文档与外部资源的关系。该元素最常用于链接样式表，此外也可以被用来创建站点图标

```html
<link rel="stylesheet" href="./style.css" />
<link rel="icon" href="favicon.ico" />
<link
  rel="apple-touch-icon-precomposed"
  sizes="114x114"
  href="apple-icon-114.png"
  type="image/png"
/>
```

常用属性：

- href：指明外部资源文件的路径，即告诉浏览器外部资源的位置，可以是绝对或相对的
- rel：**必填**，表明当前文档和外部资源的关系。rel 是核心参数，定义后在 href 规定具体值，主要属性如下：
  - bookmark：定义文档在收藏夹中显示的书签图标
  - help：链接帮助信息
  - icon：定义网站或网页在浏览器标题栏中的图标，就是定义网站图标的地方
  - license：链接到文档的版权信息
  - search：链接到文档的搜索工具
  - stylesheet：指定作为样式表的外部资源，引入 css 的方法
  - tag：指定当前文档使用的标签、关键词
  - preload：对资源进行预加载，加载会在 html 的主要渲染之前，可以通过在预加载标签中添加一些媒体元素或者包含请求的 js 来加快渲染之后的资源加载速度。
    一般需要和 as 属性配合确定预加载的资源属于哪种类型。as属性在发起具体的HTTP请求时会充当一个Accept头的效果
  ```html
  <link rel="preload" href="style.css" as="style" />
  <link rel="preload" href="main.js" as="script" />
  ```
  可以预加载的类型很多，as 的属性值可能是各种媒体元素，比如：
  ```html
  <link rel="preload" href="music.mp3" as="audio" />
  <link rel="preload" href="pic.png" as="image" />
  ```
  preload请求到的js、css等并不会立即执行。他只是开启了并行的请求，和页面load同步开始请求资源，当遇到了具体的script标签才会开始执行

> 和preload对应的还有prefetch。prefecth和preload的主要区别体现在预加载的资源什么时候使用、给不给当前页面使用
> - preload 是一个声明式 fetch，可以强制浏览器在不阻塞 document 的 onload 事件的情况下请求资源。preload的优先级更高，会强制浏览器立即开始请求资源，不管用不用的上
> - prefetch 告诉浏览器这个资源将来可能需要，但是什么时间加载这个资源是由浏览器来决定的。并且请求到资源不一定要给本页面使用，可能是其他第三方页面或者子页面。prefetch请求到的资源会将其放入缓存至少5分钟（无论资源是否可以缓存）；并且，当页面跳转时，未完成的prefetch请求不会被中断；

上述两个是最主要的属性，一般引入 css、设置网站 icon 等都只需要 href 和 rel 属性；还有一些其他的属性和这两者同级：

- hreflang：说明外部资源使用的语言
- media：说明外部资源用于哪种设备，值必须是媒体查询（css 中的`@media`之后的内容），**只有在媒体条件满足时（满足条件同 css 中的媒体查询），才会加载此资源**；例如：

```html
<link href="print.css" rel="stylesheet" media="print" />
<link href="mobile.css" rel="stylesheet" media="all" />
<link
  href="desktop.css"
  rel="stylesheet"
  media="screen and (min-width: 600px)"
/>
<link
  href="highres.css"
  rel="stylesheet"
  media="screen and (min-resolution: 300dpi)"
/>
```

- sizes：指定图标的大小，只对属性 `rel="icon"`生效
- type：说明外部资源的 MIME 类型，如 text/css、image/x-icon


关于预请求的部分还有一些，可以参考：https://cloud.tencent.com/developer/article/1552083

## `<meta>`标签

### 基本概念

元数据（Metadata）是数据的数据信息。
`<meta>` 标签提供了 HTML 文档的元数据。元数据不会显示在客户端，但是会被浏览器解析。
META 元素通常用于:

- 指定网页的描述，关键词，
- 文件的最后修改时间，作者及其他元数据。
- 指定如何布局、重载页面
- 搜索引擎和网络服务（SEO，搜索引擎优化）

### 组成

meta 标签共有三个属性，分别是 http-equiv、name 和 charset。

#### 1. name

meta 标签的 name 属性通用用法：name 指定参数，content 指定具体值
比如：

```html
<meta name="keywords" content="前端" />
```

name 属性的主要作用是描述网页，关键词信息。参数有：

- keywords：搜索引擎的网页关键字
- description：网页内容描述
- robots：定义搜索引擎爬虫的索引方式
- author：作者
- copyright：版权
- **viewport**：用于在 content 中设置视图尺寸、缩放比例等，主要用于移动端适配。

viewport 是移动端的网页可视窗口，通常这个虚拟的"窗口"（viewport）比屏幕宽，这样就不用把每个网页挤到很小的窗口中（这样会破坏没有针对手机浏览器优化的网页的布局），用户可以通过平移和缩放来看网页的不同部分。可以理解为，没有 viewport 时的移动端网页相当于是同比例的 pc 端网页大小，字体等元素都很小；添加了 viewport 之后则会自动放大，<a href="https://www.runoob.com/css/css-rwd-viewport.html">可以看这里的解释</a>。

一个常用的针对移动网页优化过的页面的 viewport meta 标签大致如下：

```html
<meta
  name="viewport"
  content="width=device-width,initial-scale=1,maximum-scale=1"
/>
```

- `width`：控制 viewport 的大小，可以指定一个具体值，或者特殊的值；如 device-width 为设备的宽度。如果有一个宽度为 320px 的手机屏幕，网页在手机上实际上只能显示 320px 左右的宽度，用户可以拖动或缩放来查看网页上的各处。
- `height`：和 width 相对应，指定高度。
- `initial-scale`：初始缩放比例，也即是当页面第一次 load 的时候缩放比例。
  - 该值为 1 时，即 1 个视口像素 = 1 个 CSS 像素。
  - 如果该值小于 1（比如 0.5），相当于网页被缩小了 1/2，也就是相当于手机的 viewport 可容纳的宽高扩大了一倍。大于 1 也是同理。
- `maximum-scale`：允许用户缩放到的最大比例。
- `minimum-scale`：允许用户缩放到的最小比例。
- `user-scalable`：用户是否可以手动缩放

#### 2. http-equiv：

可以定义一些 http 头、字符集等，其定义属性相当于定义了http响应头。比如设置csp的值，就相当于服务端发来的响应头的设置

- X-UA-Compatible：浏览器采取何种版本渲染当前页面
- cache-control：指定请求和响应遵循的缓存机制，可以用来避免页面转码，比如设置为

```html
<meta http-equiv="Cache-Control" content="no-siteapp" />
```

- expires: 网页到期时间，参数是 utc 时间
- Set-Cookie：cookie 设定，已废弃
- content-security-policy：用于设置CSP，详见网络

#### 3. charset

规定网页的编码，一般最好是 utf-8，浏览器会选择该编码模式解析 html 文档中的字符。

## `<base>`标签

`<base>`标签是一个 HTML 页中所有的相对路径的根路径，标签为页面上的所有链接规定默认地址或默认目标。
设置了 base 之后，html 中所有的路径都会变成 base 标签规定的绝对路径下的相对路径；浏览器随后将不再使用当前文档的 URL，而使用指定的基本 URL 来解析所有的相对 URL。这其中包括 `<a>`、`<img>`、`<link>`、`<form>` 标签中的 URL。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <base href="https://xingyeblog.top/" />
    <base target="_blank" />
  </head>
  <body>
    <a href="2021/12/05/blog29-h5-deeping">h5</a>
  </body>
</html>
```

点击 a 标签相当于链接`https://xingyeblog.top/2021/12/05/blog29-h5-deeping/`，img 等元素同理

## `<script>`标签

根据浏览器解析 html 文档的方法，script 标签一般放在 body 的最后，即最好在所有 html 元素之后，在`</body>`之前。
script 标签还有一些特殊的属性：

- async：该属性指定 JavaScript 代码为异步执行，即异步请求（不挤占 html 解析）、但立即执行（请求完成后依然会占用 html 解析时间执行）
- defer：该属性指定 JavaScript 代码不是立即执行，即异步请求（和 async 一样）、末尾执行（整个 html 加载完毕后才执行）

具体来说，是在html解析到script标签时启动请求，然后在dom树构建完毕后开始执行，并在DomContentLoaded之前完成执行。

![](https://pic.imgdb.cn/item/6224a74b5baa1a80ab63d1c8.jpg)

其中蓝色代表 js 脚本网络加载时间，红色代表 js 脚本执行时间，绿色代表 html 解析。

```html
<script defer>
  console.log("hello world");
</script>
```

> type="module"也可以保证script代码的异步执行：
> ![](https://pic.imgdb.cn/item/6400ca41f144a01007503551.jpg)

- crossorigin：如果采用这个属性，就会采用跨域的方式加载外部脚本，即 HTTP 请求的头信息会加上 origin 字段。
  script标签本来就是可以跨域进行请求的，即可以执行不同源的脚本，这也是jsonp的原理。
  不过script执行跨域脚本有一些限制：比如不能捕获到跨域脚本内部出现的具体错误，而只能得到一个“script error”的错误。并且请求script的请求头中也不携带origin字段。
  如果有crossorigin，则会按照CORS的规则去请求。具体来说request会带上origin头，然后会要求服务器进行cors校验，后面就和简单请求的cors流程相同了。如果服务端没有做cors的相关响应头设置的话，那么请求就会抛出跨域错误，同时也不会执行脚本。
  crossorigin的取值有：
  - anonymous：默认值，表示使用cors，但不会发送cookie到不同源。
  - use-credentials：可以跨域发送cookie等
  除了script，img、video等元素都有crossorigin属性，他们的执行原则和script标签相同。
- integrity：给出外部脚本的哈希值，防止脚本被篡改。只有哈希值相符的外部脚本，才会执行。
- nonce：一个密码随机数，由服务器在 HTTP 头信息里面给出，每次加载脚本都不一样。它相当于给出了内嵌脚本的白名单，只有在白名单内的脚本才能执行。
- referrerpolicy：HTTP 请求的 Referer 字段的处理方法。


# viewport

参考：https://www.jianshu.com/p/7c5fdf90c0ef


## 概念

移动设备上的viewport就是设备的屏幕上能用来显示我们的网页的那一块区域，即浏览器上用来显示网页的那部分区域。

viewport的大小决定了网页的大小。对pc端来说，可以看做是viewport就是电脑屏幕的大小

可以理解为我们使用vw、媒体查询等方式获取的页面宽度就是viewport的大小，也就是屏幕显示网页的大小。
我们设置`width=media-width`就是将页面的宽度值设为设备宽度值，这样的话，比如说某个元素的样式是`width:100%`，那么在pc端可能就是1920px，而在移动端可能就是300px，这取决于viewport的大小。

对于移动端来说，宽度肯定是要远小于pc端的。但是如果简单粗暴的把移动端的viewport设置为300px这种比较小的单位，那么页面的总宽度就变成了300px，这样的话如果没有做响应式，整个页面都会乱掉。

移动设备上的浏览器都会把自己**默认的viewport**设为980px或1024px（也可能是其它值，这个是由设备自己决定的），但带来的后果就是浏览器会出现横向滚动条，因为浏览器可视区域的宽度是比这个默认的viewport的宽度要小的。


## 三个viewport

实际上，移动设备上有三个viewport：

1. layout viewport(布局视口)：在PC端上，布局视口等于浏览器窗口的宽度。而在移动端上，由于要使为PC端浏览器设计的网站能够完全显示在移动端的小屏幕里，此时的布局视口会**远大于移动设备的屏幕**，就会出现滚动条。js获取布局视口：`document.documentElement.clientWidth | document.body.clientWidth`；

可以理解为，这是移动端浏览器的一种保护措施。如果这个默认的viewport宽度过小，那么那些没有做移动端适配的网页的布局就会完全乱掉（相当于页面宽度从几千缩小到了几百）。
布局视口是决定css相对属性值的根据，即百分比、vw这样的相对单位的基础就是布局视口的宽度。比如布局视口大小为980px，那么在移动端上的100%宽度就是980px。

![](https://pic.imgdb.cn/item/640b8ca5f144a01007e3195a.jpg)

2. visual viewport(视觉视口)：用户正在看到的网页的区域。用户可以通过缩放来查看网站的内容。如果用户缩小网站，我们看到的网站区域将变大，此时视觉视口也变大了，同理，用户放大网站，我们能看到的网站区域将缩小，此时视觉视口也变小了。不管用户如何缩放，都不会影响到布局视口的宽度。js获取视觉视口：`window.innerWidth`；

这个值即真正意义上的“视口”的大小，即**浏览器可视区域的大小**。如果浏览器可视区域发生变化，这个值也会变化。
比如打开控制台，那么浏览器显示页面的区域变小了，这个值也会变小。又比如放大页面时，相当于视觉视口变小。因此这个值并非固定值，是一个很容易动态变化的值。

![](https://pic.imgdb.cn/item/640b8cb8f144a01007e33203.jpg)

3. ideal viewport(理想视口)：布局视口的一个理想尺寸，只有当布局视口的尺寸等于设备屏幕的尺寸时，才是理想视口。js获取理想视口：`window.screen.width`；

ideal viewport 是最适合移动设备的viewport，它的**宽度等于移动设备的屏幕宽度**，只要在css中把某一元素的宽度设为ideal viewport的宽度(单位用px)，那么这个元素的宽度就是设备屏幕的宽度了，也就是宽度为100%的效果。
ideal viewport 的意义在于，无论在何种分辨率的屏幕下，那些针对ideal viewport 而设计的网站，不需要用户手动缩放，也不需要出现横向滚动条，都可以完美的呈现给用户。

相对于visual viewport，ideal viewport是一个相对固定的值。比如在pc端不管打不打开控制台，这个值都是固定的1920，表示一种“理想”状态下的宽度，而非实际可视区域的宽度。


如果我们把布局视口的尺寸定为理想视口，相当于在移动端上的所有布局大小都是以理想视口的大小为依据。
比如一台手机宽度为540px，那么我们以540px为基础宽度，元素的百分比、媒体查询、vw等都以540px为基准，就可以实现在移动端上的布局。这就是响应式布局的基础。

方式就是这样：
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0">
```

如果不这样的设定的话，就会使用那个比屏幕宽的默认viewport，也就是说会出现横向滚动条。

# 常见事件

下面一些事件都是经常会用到，但是有些细节会被忽略的；事件本身的名字可以用做 addEventListener 的事件名参数， 如果想直接监听需要 onxxx. 比如 click 事件实际上是 onclick

## window 事件

- error: 网页资源加载失败
- abort: 资源加载终止
- load: 加载完成
- online: 在线
- pagehide/pageshow: 类似 load/unload, 但是主要用于移动端网页,
- fullscreenchange: 窗口最大化事件
- resize: 窗口尺寸变化
- scroll: 滚动
- storage: 浏览器存储事件触发; 域名不同会导致触发多次

## 元素事件

### 文本和输入事件

- focus/blur: 注意事件**不会冒泡**
- cut/copy/paste: 文本被剪切/复制/粘贴;**如果被阻止默认事件就不会被复制/剪切, 向文本框中粘贴也不会生效**, 可以安插在最顶级父元素上
- select: 文本被选中
- change: 单选框/选择器等被选择时触发, 但是输入框失焦才会触发

### 键盘事件

- keydown/keypress/keyup: 键被按下/按住/松开, **其中 keypress 事件在被按住时会一直触发**

### 鼠标事件

- mousedown: 在元素上按下鼠标上的任意按钮(侧键也会被监听)
- mouseenter/leave/: 进出元素
- mousemove: 在元素上面持续触发
- mouseover: 进出元素都会被触发, 和 enter 的区别在于:**mouseover 在鼠标进出子元素时会在父元素再触发一次,** 而 mouseenter 不会
- mouseout: 移除元素, 或**移到子元素上**
- mouseup: 释放鼠标键

### 拖拽事件

- drag: 正在拖动元素, 事件目标是被拖动的, 拖动时持续触发
- dragstart/end: 事件目标是被拖动的, 开始拖动
- dragenter/leave: 事件目标是进入的元素
- dragover: 在进入的元素之上持续触发
- drop: 松开鼠标,注意是在有效区域释放才会触发

### 媒体事件

- canplay/canplaythrough: 浏览器可以播放媒体, 但可能需要缓冲/不需要缓冲
- ended: 播放完成, 是一个布尔值, 只读
- timeupdate: 常用于监听播放时间获取正在播放进度等

# 语义标签

新增的语义标签：

- 布局标签，都是和 div 一样的块级标签：

```html
<header>
  <nav>
    <aside>
      <main>
        <article>
          <section>
            <footer></footer>
          </section>
        </article>
      </main>
    </aside>
  </nav>
</header>
```

- header：页面头部
- main：页面主要内容
- footer：页面底部
- nav：导航栏
- aside：侧边栏
- article：加载页面一块独立内容
- section：表示一个含有主题的独立部分，通常用在文档里面表示一个章节，比如`<article>`可以包含多个`<section>`。`<section>`总是多个一起使用，一个页面不能只有一个`<section>`。
  （article 和 section：前者可以用作文章的父元素，而后者推荐用作小组件，比如弹出框、抽屉什么的）
- figure：加载独立内容（上图下字）
- figcaption：figure 的标题
- hgroup：标题组合标签，用于包含 h1~h6 等多个元素，可以表示副标题的效果
- mark 高亮显示
- dialog 加载对话框标签（必须配合 open 属性）
- embed 加载插件的标签
- video 加载视频
- audio 加载音频（支持格式 ogg，mp3，wav）

除了布局标签，还可能是一些 H5 新增的功能性的标签，比如 details/summary（用于实现“展开”效果）、mark（高亮）等

---

对语义化的理解：

> 语义化是指根据内容的结构化（内容语义化），选择合适的标签（代码语义化）。通俗来讲就是用正确的标签做正确的事情。
> 语义化的优点如下：
>
> - 对机器友好，带有语义的文字表现力丰富，更适合搜索引擎的爬虫爬取有效信息，有利于 SEO。除此之外，语义类还支持读屏软件，根据文章可以自动生成目录；
> - 对开发者友好，使用语义类标签增强了可读性，结构更加清晰，开发者能清晰的看出网页的结构，便于团队的开发与维护。

# html 元素属性

除了常见的 class、id 等属性，html 中还有一些冷门的属性，可以参考<a href="https://wangdoc.com/html/attribute.html">网道-网页元素的属性</a>

# HTML5 的更新

### 1. 新增语义化标签

见上文

### 2. 音频、视频标签

主要是 audio、video，以及他们的新 api 和 html 元素上的新属性

```html
<video src="" poster="imgs/aa.jpg" controls></video>
<audio src="" controls autoplay loop="true"></audio>
```

- `controls` 控制面板
- `autoplay` 自动播放
- `loop='true'` 循环播放
- `poster` 指定视频还没有完全下载完毕，或者用户还没有点击播放前显示的**封面**。默认显示当前视频文件的第一针画面，当然通过 poster 也可以自己指定。

---

还可以使用 source 标签针对不同的格式提供不同的资源，也适用于`<picture>`元素

```html
<video>
 	<source src='aa.flv' type='video/flv'></source>
 	<source src='aa.mp4' type='video/mp4'></source>
</video>
```

### 3. 数据存储

localStorage、sessionStorage、indexDB，详见 WebAPI

### 4. 表单相关

#### 表单类型：

- `email` ：能够验证当前输入的邮箱地址是否合法
- `url` ： 验证 URL
- `number` ： 只能输入数字，其他输入不了，而且自带上下增大减小箭头，max 属性可以设置为最大值，min 可以设置为最小值，value 为默认值。
- `search` ： 输入框后面会给提供一个小叉，可以删除输入的内容，更加人性化。
- `range` ： 可以提供给一个范围，其中可以设置 max 和 min 以及 value，其中 value 属性可以设置为默认值
- `color` ： 提供了一个颜色拾取器
- `time` ： 时分秒
- `data` ： 日期选择年月日
- `datatime-local` ：日期时间控件
- `week` ：周控件
- `month`：月控件

#### 表单属性：

- `placeholder` ：提示信息
- `autofocus` ：自动获取焦点
- `autocomplete="on"` 使用这个属性需要有两个前提：
  - 表单必须提交过
  - 必须有 name 属性。
- `required`：要求输入框不能为空，必须有值才能够提交。
- `pattern=" "` 里面写入想要的正则模式，例如手机号 `pattern="^(+86)?\d{10}$"`
- `multiple`：可以选择多个文件或者多个邮箱

#### 表单事件：

- `oninput` 每当 input 里的输入框内容发生变化都会触发此事件。
- `oninvalid` 当验证不通过时触发此事件。

关于表单验证相关还有很多 api 以及 css 类型、伪元素等

### 5. DOM 查询操作

- `document.querySelector()`
- `document.querySelectorAll()`
  它们选择的对象可以是标签，可以是类(需要加点)，可以是 ID(需要加#)

### 6. 其他 api

- 拖拽事件相关，详见 https://developer.mozilla.org/zh-CN/docs/Web/API/Document/drag_event
- canvas 和 svg
- history api，主要是控制浏览器历史访问（即左上角常见的前进后退等），详见 https://developer.mozilla.org/zh-CN/docs/Web/API/History_API
- geolocation 地理定位

# Web Worker

通过使用 Web Workers，Web 应用程序可以在独立于主线程的后台线程中，运行一个脚本操作。这样做的好处是可以在独立线程中执行费时的处理任务，从而允许主线程（通常是 UI 线程）不会因此被阻塞/放慢。Web Workers 相当于浏览器提供的多线程，但是依旧不能多线程操作 dom；对于 js 来说依旧是单线程。

## 主要特点

- 不能直接在 `worker` 线程中操纵 DOM 元素
- 不能使用 `window` 对象中对象的默认方法和属性，但依然可以使用ws、indexDB等
- `workers` 运行在另一个全局上下文中,不同于当前的`window`. 因此，在 `Worker` 内通过 `window`获取全局作用域 (而不是 self) 将返回错误。
- 主线程和 `worker` 线程相互之间使用 `postMessage()` 方法来发送信息, 并且通过 `onmessage` 来接收信息（传递的信息包含在 `Message` 这个事件的 data 属性内) 。数据的交互方式为传递副本，而不是直接共享数据。
- worker 可以另外生成新的 worker，这些 worker 与它们父页面的宿主相同。 此外，worker 可以通过 `XMLHttpRequest `来访问网络，只不过 `XMLHttpRequest` 的 `responseXML` 和 `channel` 这两个属性的值将总是 `null` 。

## 使用 web workers

使用 web workers 主要流程如下：

1. 检查浏览器的支持

```js
if (window.Worker) {

  ...

}
```

2. 通过`new Worker()`创建一个 worker，参数是一个新 js 文件的 url

```js
const worker = new Worker("./myworker.js");
```

3. 主线程和 worker 互相收发信息：

```js
//main.js
worker.postMessage(...)

//myworker.js
onmessage = (e) => {
  const res = e.data;
  console.log(res)
  postMessage(res**res)
}

//main.js
worker.onmessage = (e) {
  console.log(e.data)
}

```

可以看到在主线程和子线程中使用略有不同：

> 在主线程中使用时，`onmessage`和`postMessage()` 必须挂在 worker 对象上，而在 worker 中使用时不用这样做。原因是在 worker 内部，worker 是有效的全局作用域。
> 当一个消息在主线程和 worker 之间传递时，它被复制或者转移了，而不是共享。

5. 终止 worker
   调用 worker 的`terminate` 方法，worker 线程会被立即杀死，不会有任何机会让它完成自己的操作或清理工作。
   而在 worker 线程中，workers 也可以调用自己的 `close` 方法进行关闭

```js
worker.terminate();

//worker内部
close();
```

更多 api 可以参考 https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers


## webworker和主线程的通信

webworker的本质还是运行在浏览器进程内的多线程架构，因此webworker和js主线程的通信基本也和进程下的多线程通信方式一致：

通信方式有三种：
1. Structured Clone：Structured Clone 是 postMessage 默认的通信方式，如下图所示，复制一份线程 A 的 js object 内存给到线程 B，线程 B 能获取和操作新复制的内存。
Structured Clone 通过复制内存的方式简单有效的隔离了不同线程的内存，避免冲突；且传输的 object 数据结构很灵活，但复制过程中，线程 A 要 同步执行 Object Serialization，线程 B 要 同步执行 Object Deserialization，如果 object 规模过大，会占用大量的线程时间。

![](https://pic.imgdb.cn/item/641d6a8ea682492fccabd1ba.jpg)

2. Transfer Memory
Transfer Memory 意味着转移内存，它不需要序列化和反序列化，能大大减少传输过程占用的线程时间。如下图所示，线程 A 将制定内存的所有权和操作权转交给线程 B，但转然后线程 A 无法在访问这块内存。
Transfer Memory 以失去控制权来换取高效传输，通过内存独占给多线程并发加锁，但只能转让 ArrayBuffer 等大小规整的二进制数据，对矩阵数据（比如 RGB图片）比较适用，实践上要考虑从 js object 生成二进制数据的运算成本

![](https://pic.imgdb.cn/item/641d6b4ea682492fccacff45.jpg)

3. Shared Array Buffers
Shared Array Buffers 是共享内存，线程 A 和线程 B 可以 同时访问和操作 同一块内存空间，数据都共享了，也就没什么传输的事了。
但多个并行的线程共享内存，会产生竞争问题，不像前两种传输方式默认加锁，Shared Array Buffers 把难题抛给了开发者，开发者可以用 Atomics 来维护这块共享的内存。作为较新的传输方式，浏览器兼容性可想而知，目前只有 Chrome 68+ 支持。

![](https://pic.imgdb.cn/item/641d6b5ca682492fccad16dc.jpg)




