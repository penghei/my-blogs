# 主要工具回顾

## Sass

Sass 是一款强化 CSS 的辅助工具，它在 CSS 语法的基础上增加了变量 (variables)、嵌套 (nested rules)、混合 (mixins)、导入 (inline imports) 等高级功能，这些拓展令 CSS 更加强大与优雅。使用 Sass 以及 Sass 的样式库（如 Compass）有助于更好地组织管理样式文件，以及更高效地开发项目。

Sass 有两种语法格式：

- Sass：使用缩进而没有大括号、分号；这种形式的文件以.sass 为后缀
- SCSS：和 css 格式几乎完全相同，支持所有 css 写法，文件以.scss 为后缀

### 使用

sass 可以通过命令行启用，类似 babel 等工具的编译方式处理 sass 文件

```
sass input.scss output.css
```

通常不这样使用；在 webpack 中安装 sass-loader 并放在 css-loader 后面即可翻译 scss 文件。
Vite 内置支持 scss 文件的编译，只需要安装 sass 就可以，并不需要手动配置插件。

### 常用语法

1. **嵌套**：嵌套功能避免了重复输入父选择器，而且令复杂的 CSS 结构更易于管理。同时媒体查询的嵌套可以避免在媒体查询内部写入对应的样式。
2. **父选择器**：常用于伪类的添加
3. **属性嵌套**：指的是这种形式：

```css
font: {
  family: fantasy;
  size: 30em;
  weight: bold;
}
```

4. **双斜杠注释**
5. **变量**：变量用$开头；sass 中的变量基本使用和在 js 等编程语言类似，通常可以替代大多数尺寸、颜色等以及他们的组合。好处在于方便统一和管理，比如一个颜色在多处使用时，使用变量可以统一这个样式，方便统一修改。

变量通过插值语法可以在属性名或选择器中使用，比如这样：

```scss
$name: foo;
$attr: border;
p.#{$name} {
  #{$attr}-color: blue;
}

// 相当于
p.foo {
  border-color: blue;
}
```

6. **运算**：sass 可以直接进行一些属性的运算，比如不同单位之间的尺寸、颜色等，相当于 css 中的`calc()`，但是可以直接写出来。

7. **控制指令**，即以`@`开头的一些指令，主要包括：

   1. @import：允许导入 SCSS 或 Sass 文件。被导入的文件将合并编译到同一个 CSS 文件中，被导入的文件中所包含的变量或者混合指令 (mixin) 都可以在导入的文件中使用。如果在文件名前添加下划线，可以使 sass 不编译该文件。这样可以仅在书写的时候使用其他文件的变量、混入等，而并不需要真正编译；

   2. @extend：将一个选择器下的所有样式继承给另一个选择器

   ```scss
   .error {
     border: 1px #f00;
     background-color: #fdd;
   }
   .seriousError {
     @extend .error;
     border-width: 3px;
   }
   ```

   3. @if/@else/@else if：和在 js 中使用类似

   ```scss
   $type: monster;
   p {
     @if $type == ocean {
       color: blue;
     } @else if $type == matador {
       color: red;
     } @else if $type == monster {
       color: green;
     } @else {
       color: black;
     }
   }
   ```

8. **混入@mixin**：可以封装一部分样式。在@mixin 之后写上变量名和一组属性值，可以通过@include 导入并引用：

```scss
@mixin large-text {
  font: {
    family: Arial;
    size: 20px;
    weight: bold;
  }
  color: #ff0000;
}
.page-title {
  @include large-text;
  padding: 4px;
  margin-top: 10px;
}
```

和变量类似，可以把一组通用的样式封装起来，然后提到单独的文件中去，使用的时候再去引用即可。

@mixin 还支持参数和默认值，相当于 js 中的函数：

```scss
@mixin sexy-border($color, $width: 1px) {
  border: {
    color: $color;
    width: $width;
    style: dashed;
  }
}

p {
  @include sexy-border(blue, 1in);
}
```

9. **函数**：sass 支持很多内置函数，比如 max、min、random 等数字处理函数，或者 join、index、append 等数组处理函数（一组逗号隔开的简写样式被看作一个 list）。还可以自定义函数。

## Less

Less 总体的语法和 sass 相近，只有少数功能和语法不同，比如 less 的变量用@开头。

Less 的内置函数多于 sass，同样有着和 sass 类似的 if、else 等逻辑语句。

## React-Router

React-Router 的最新版本是 v6，相对于以前的版本做了很大改动。项目中使用的都是 v5，也是最常用的一个版本。

React-Router 负责路由分发、页面跳转，是用于提供给 React 的单页面应用的路由库。

### 单页面应用

用 React 或者 Vue 构建的应用都是单页面应用，单页面应用是使用一个 html 前提下，一次性加载 js ， css 等资源，所有页面都在一个容器页面下，**页面切换实质是组件的切换**。

### 基本使用

首先安装

```
yarn add react-router-dom
```

这里有一个区别：react-router 和 react-router-dom：在 react 复习那一块我们讲过后者包含前者，并且增加了两个 Link 和两个 Router。因此前者内部的组件后者都有，所以理论上只需要安装后者即可

#### 常用组件

1. BrowserRouter、HashRouter：两种路由根组件，是所有路由组件的根组件，所有路由组件都必须要在这个组件的包裹下才能使用。两个的区别可以参考 react 复习，简单来说就是使用不同的原理来实现路由的（hash 值和 history api）

2. Route：Route 组件是 React Router 中最重要的组件，最基本的职责是在其路径与当前 URL 匹配时呈现 UI。即**匹配路由**和**渲染组件**

3. Switch：通常用于包裹在多个 Route 组件外部，作用是匹配唯一正确的路由并渲染；如果不通过 swicth 包裹，页面上可能会出现多个 route 组件。
   注意 Swicth 会渲染第一个匹配的路径，因此需要尽可能把长的 path 写在前面。

4. Link 和 NavLink：相当于一个跳转路由的`<a>`标签。后者可以用于配置标签的 active 样式。

### 项目使用

#### 手动跳转

有两种方式：withRouter 包裹组件和使用 hooks

前者实际上是一个高阶组件，包裹之后给组件的 props 增加了三个对象：match, location, history。

hooks 主要有四个：useHistory、useLocation、useParams 和 useRouteMatch。前两个 hooks 都是调用会返回一个 history 和 location 对象。后两个主要用于路由动态参数。

#### 路由配置

使用 react-router-config 可以实现在外配置路由并导入

使用方式：https://juejin.cn/post/6911497890822029326

#### 路由鉴权

React 的路由鉴权主要通过外部判断权限，然后选择渲染 Route 组件还是重定向到其他位置。即这种形式：

```js
return isMatch ? <Route {...props} /> : <Redirect to={"/NoPermission"} />;
```

可以将其封装成一个组件，判断是否有权限的状态 isMatch 可以从全局状态中获取。

### 使用问题

## Recoil

Recoil 是一个状态管理库。不同于 Redux 这样的外部状态，Recoil 使用的是 React 自己的状态管理的改进。
Recoil 定义了一个有向图 (directed graph)，正交同时又天然连结于你的 React 树上。状态的变化从该图的顶点（我们称之为 atom）开始，流经纯函数 (我们称之为 selector) 再传入组件。
Recoil 的特点就是简单，并且使用上类似于 React 原生的 context；它通过 atom 定义并保存一个顶层状态，然后在需要该状态的组件中引入该状态，使用这个顶层状态的方法就像用 useState 一样。

- Atom 是组件可以订阅的 state 单位。
- selector 可以同步或异步改变此 state。

# 项目 1

关键点：

- Audio 元素和 api，即音乐播放器，分几个点来说：
  - 音乐从哪里来（网易云音乐 Nodejs api）
  - 怎么插入歌曲（src 属性）
  - 怎么播放暂停音乐（play、pause）
  - 进度条和歌曲进度控制（timeupdate、currentTime、duration）
  - 怎么自动播放下一首（动态 src 和 load()）
  - 怎么控制歌词滚动（）
- 了解了浏览器获取摄像头视频流并捕获照片的方法。
  - Navigator.mediaDevices api 用法
  - video 元素播放摄像头获取的视频流
  - imageCapture 捕获图片并转化格式
- scss 使用，样式统一

## 音乐播放器

### Audio 元素 api

Audio 元素是指 HTML5 提供的新元素以及其携带的新 api。`<audio>` 元素可以包含一个或多个音频资源， 这些音频资源可以使用 src 属性或者`<source>` 元素来进行描述

audio 元素的属性很多，项目中用到的属性主要有：

- currentTime：返回一个双精度浮点值，用以标明以秒为单位的当前音频的播放位置。该值可读可写，改变该值时会改变当前音频的播放进度
- duration：只读，返回当前音频的持续时间秒数
- loop：单曲循环
- muted：是否静音
- src：插入音频。当元素加载时向 src 赋予值就相当于插入了音频；同时替换歌曲也是修改 src 属性。

以及一些事件：

- ended：判断歌曲播放完毕，播放下一首
- play：播放歌曲。通过`audio.play()`调用时会返回一个 promise，可以用来捕获错误。如果 play 期间被 load 方法阻止了，则会报一个被中断的错误
- pause：暂停，可以通过`audio.pause()`触发，媒体的 pause 变成 true，并且只能触发一次；恢复需要通过 play()
- load：当 audio 元素的 src 属性被更改后，需要调用 audio.load()重新加载歌曲。
- timeupdate：当 currentTime 更新时会触发 timeupdate 事件，浏览器保证每秒触发 4-66 次；监听这个事件，当事件被触发时说明播放时间更改，可以用于修改进度条等功能。

### 具体功能写法

1. 播放音乐：在组件中放入 audio 元素，其上只有 src 属性；通过动态修改 src 属性实现歌曲的添加和更换。
2. 上一首/下一首：同样是动态更新 src 属性并调用 load()方法；组件中会获取一个播放列表的数组，当点击上一首/下一首时会判断超出/过头等情况并避免。
3. 进度条：通过 audio 元素的 timeupdate 事件监听，实时获取 currentTime 值，和 duration 相除得到进度，然后动态修改进度条的长度即可。
4. 歌词滚动：
   1. 首先获取歌词文件，歌词文件的每句话前面都会带上该句歌词的播放时间。
   1. 获取这个时间并和每句歌词绑定成一个对象
   1. 获取当前播放时间，取整之后和歌词时间匹配；如果没有匹配到就保持刚才渲染的不变。
   1. 动态渲染匹配的歌词（字体突出）以及其前面的后面的 n 个歌词

## 摄像头视频和拍照

### 主要 api 和功能实现

#### navigator.mediaDevices

获取用户摄像头使用的是`navigator.mediaDevices`api，这个属性上的`getUserMedia()`方法会申请获取打开用户摄像头的权限

```js
navigator.mediaDevices
  .getUserMedia(constraints)
  .then((MediaStream) => {})
  .catch((err) => {});
```

- constraints ：配置对象，详见https://developer.mozilla.org/zh-CN/docs/Web/API/MediaDevices/getUserMedia。大概来说包括获取音频还是视频、获取多少像素、获取前置还是后置等选项。项目中的配置为：

```js
{
  video: {
    facingMode: "user";
  }
}
```

即获取用户前置摄像头。

- MediaStream ： `getUserMedia`函数返回一个 promise。如果用户同意并且摄像头正常，该函数会 resolve 一个媒体流(MediaStream)对象

#### MediaStream

详见：https://developer.mozilla.org/zh-CN/docs/Web/API/MediaStream

MediaStream 接口是一个媒体内容的流。一个流包含几个轨道，比如视频和音频轨道。

项目中使用该对象赋值给 video 元素的`srcObject`属性。这个对象提供了一个与 HTMLMediaElement 关联的媒体源，源对象通常是 `MediaStream` ；直接把`MediaStream`赋值给该属性就可以将摄像头捕获的画面实时显示到 video 元素中。

> srcObject相当于video元素的src的URLObject形式。也就是说它的值可以直接是流、Blob等二进制对象，即这两个相等：
> ```js
> video.src = URL.createObjectUrl(MediaStream)
> video.srcObject = MediaStream
> ```

设置完成后并不会主动播放，也就是说video元素获取到了视频源，但需要手动播放。因此可以监听其canplay事件（addEventlistener或OnCanplay都可以）并调用`video.play()`自动播放。直接设置video元素的autoplay也可以。
```js
video.addEventListener('canplay',()=>video.play().catch(err=>console.log(err)))  
```

另外还需要把 MediaStream 的轨道存起来以便下一步使用。方法是调用`MediaStream.getVideoTracks()[0]`，会返回视频轨道

#### imageCapture

详见https://developer.mozilla.org/en-US/docs/Web/API/ImageCapture

imageCapture 可以用于捕获视频轨道并生成图片。
当用户点击拍照时，通过向 ImageCapture 构造函数传入视频轨道，可以获取一个 imageCapture 对象；然后调用其上的`takePhoto`方法可以返回一个 Blob，再转化成 base64 即可。

```js
const imageCapture = new ImageCapture(StreamTrack.current);
const res = await imageCapture.takePhoto();
```

注意此时为了给用户呈现拍照效果，可以通过`video.pause()`暂停视频，此时效果就是拍照那一刻的图片。

接下来就是上传、获取结果、匹配并播放的过程。

---

另外一种方法是采用canvas的drawImage方法。https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/drawImage
给这个方法传入video元素的dom对象，执行时就可以绘制当前帧图片。然后再通过`canvas.toDataURL`方法就可以获取当前图片的Base64格式

```js
// 首先得有一个canvas元素，但是可以隐藏掉
const ctx = canvas.getContext('2d')
ctx.drawImage(video,0,0)
img.src = canvas.toDataURL() // 这个方法是直接在canvas对象上的
```


## 样式的统一

SCSS 文件把样式写在顶层的其他文件中，然后通过@import 引入。

当文件名为下划线开头时，scss 不会解析为 css，而是直接引入其中的变量

这些文件使用变量、mixin 等集成了一些颜色、尺寸等样式

# 项目 2

- 采用请求响应拦截器、封装 axios 等方法，以解决请求数量大、并且需要统一处理的情况。
  - 怎么封装的 axios
  - 封装了哪些内容，做了哪些处理
  - 怎么处理错误的
- React 的一些优化方法，使用 React.lazy、useMemo 等 hooks 进行了初步的优化。

## 封装 axios

axios 本身就是请求的封装；但是当项目中的请求多起来的时候，需要一次性集中配置 axios，让配置适应项目的大部分场景。使用自定义配置新建一个 axios 实例，然后对实例进行基本配置，在请求前(请求体处理)，请求后(返回的结果处理)等这些阶段添加一些我们需要的处理，然后将其导出使用。

封装的总结：

1. `axios.create`创建 axios 实例

- 实例内部可以有配置，比如超时、baseURL、通用请求头等
- 通常实例只需要一个就好了，但是有几个接口不需要 jwt，就再创建一个单独的实例处理这些

2. 请求拦截器

- 添加 jwt 头
- 过滤空字段

3. 响应拦截器

- 解构响应，按照约定的处理方式返回值
  - 如果值正常，状态正常，直接返回值
  - 如果状态正常，但是值有问题（比如密码错误），根据和后端约定的 code 字段做不同的处理
- 处理响应错误
  - 响应错误的三种情况，根据状态码处理
  - 最后返回一个 reject

4. 封装请求函数，把请求函数单独归类写在文件中，使用时直接调用函数即可。这些单独处理的函数可能还要执行一些提醒功能，比如 antd 的`message`/`Modal`等。

### 封装方法

1. axios.create ：创建自定义 axios 实例，传入一个对象用于配置：

```js
const PostRequest = axios.create({
  baseURL: "http://localhost:8000/glimmer-bank/platform/",
  timeout: 5000,
  headers: {
    post: {
      "Content-Type": "application/json",
    },
  },
});
```

根据请求的不同，instance 在项目中创建了多个实例，分别对应不同组的请求形式。

2. 请求拦截器

通过`instance.interceptors.request.use()`传入请求拦截器参数；参数是两个函数，分别对应发送请求前和请求失败时的处理。
项目需要 jwt，因此在请求前从缓存中获取 jwt，并插入到请求头中
还可以配置一个清除请求体中空值的函数，比如过滤请求参数中的 null undefined ''

```js
service.interceptors.request.use((config) => {
  config.headers["Authorization"] = `${getStorage("jwt")}` || `Bearer `;
  cleanObject(config?.data);
  return config;
});
```

3. 响应拦截器

主要功能：

- **处理成功返回的数据**

比如后端返回的 data 数据可能嵌套了很多层，你可以直接返回你需要的 data 数据，这样业务代码就可以直接拿到最终的数据，而不用每次去解构。

由于每个 Get 请求的返回格式统一，因此在响应拦截器中就对其解构

```js
// 只有res.status为2xx才会调用第一个参数，只要不是2开头的都会调用第二个处理错误的函数
service.interceptors.response.use((response) => {
  const { res: data, status } = response; // 解构响应
  const { success, message, data, code } = res; // 根据实际情况解构
  // 下面是约定的处理方式
  if (success && message.includes("成功")) {
    // 成功直接返回data
    return data;
  } else {
    // 如果请求成功，但内部有一些问题，那么约定好message是处理字段返回即可
    return message;
  }
  if (!success && code === 401) {
    // jwt过期
    message.error("登录过期,请重新登录");
  }
  
  // 默认返回response
  return response;
});
```

- **统一处理失败后的异常报错**

错误返回的 error 对象有三种情况

- 返回 error.response:请求收到返回，但返回码不是 2xx 开头的
- 返回 error.request：请求未收到返回,可能是网络问题等
- 其他情况：请求触发器出错

封装时应当分别处理这三种情况：

```js
service.interceptors.response.use(_, (error) => {
  const { response, request } = error;
  if (response) {
    const { status, headers } = response;
    // 这里定义一个对象,内部是处理函数,根据状态码处理
    statusDict(status)();
  } else if (request) {
    if (!window.navigator.onLine) {
      message.warning("网络异常，请检查网络是否正常连接");
    }
  } else {
    message.warning("服务器异常，请联系管理员");
  }
  // 不管怎么处理都会返回一个reject对象
  return Promise.reject(error);
});
```

最后不论中间如何处理，结果都要返回一个 Promise.reject。

4. 封装请求

除了拦截器，还可以把对应的接口和特殊处理方式单独封装成一个函数放入文件中，按照分类放入不同的文件。
比如项目中的用户相关接口，统一放在 user.js 文件中，里边是一些封装好的请求函数

```js
// user.js
import service from "./interceptors";

const getUserLoginState = async () => {
  const userInfo = await service.get("/customer/detail");
  if(typeof userInfo = 'string'){
    message.warn(message) return {}
  }else return userInfo
};
```

这些函数一般只是请求并返回值，同时做一些特殊的处理（比如返回 message 的时候只弹出提醒不返回 data）

因为已经统一处理了错误，所以不需要 trycatch 捕获；虽然会抛出错误，但是有用的错误信息都已经被处理，因此也就没必要捕获了。

### 其他处理

#### 自动重试

安装一个库 axios-retry

```js
import axios from "axios";
import axiosRetry from "axios-retry";

const instance = axios.create({
  // 你的配置
});

axiosRetry(client, { retries: 3 });

// 只有3次失败后才真正失败
const data = request("http://example.com/test");
```

## React 优化

优化方式参考 reactreview 里边说的 react 方法。

项目中使用的优化：

### 1. React.lazy

主要在路由中使用，因为路由组件内部还包含了大量的组件，第一次渲染的时候会很慢。
处理方式：

1. 用 React.lazy 包裹动态导入的路由组件，返回一个处理之后的组件
2. Route 的 component 组件改为处理后的
3. 用 Suspense 组件包裹所有处理后的组件，并添加 fallback 属性，这个属性可以是一个组件，在 loading 时渲染。

> 实现懒加载优化时，不仅要考虑加载态，还需要对加载失败进行容错处理。
> 方法就是使用错误边界，对加载失败的组件包裹一层，降级处理

### 2. React.memo

> 父组件的每次状态更新，都会导致子组件重新渲染，即使传入子组件的状态没有变化，为了减少重复渲染，我们可以使用 React.memo 来缓存组件，这样只有当传入组件的状态值发生变化时才会重新渲染。如果传入相同的值，则返回缓存的组件。

简单来说就是把负载大的子组件包裹，防止父组件更新带着子组件进行一些不必要的更新。
项目中使用（举几个例子）：几乎每个大一点的组件都用到了 memo，小的布局组件不需要。

### 3. useMemo/useCallback

项目中使用：

1. 给子组件的 props 是对象时，每次都是新的引用，因此用 useMemo 包裹变化频繁的父组件（表单、倒计时等）要传给子组件的 props，用 useCallback 包裹传入的回调。（比如用户信息的展示表单，父组件要把一个用户信息对象传入，这时可以用 useMemo 包裹，依赖数组就是全局的状态；当全局状态不改变时就不变值）
2. 减少计算：在最后的支付界面会计算用户余额和商品价格等，这个价格因为要计算折扣、利率和其他一些数字，并且要到小数点后很多位，因此计算开销很大；用 useMemo 包裹，只有用户余额、商品价格变化才重新计算。

### 4. 用样式更改代替组件卸载

### 在哪里看到优化效果

https://juejin.cn/post/6844903869378641933

1. 肉眼感知
2. 更新高亮框
3. React devtools 的 Profiler，选择 Ranked 视图可以看到组件的渲染时间

