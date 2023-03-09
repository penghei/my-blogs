---
title: next.js 学习
date: 2022-09-15 15:37:22
tags: 日常学习
categories: React
cover: /img/nextjs.png
---

# 基本概念

## 为什么要用 nextjs

首先纯粹的 React 很难作为大型项目的开发工具，一般来说都会有各种基于 React 的脚手架。nextjs 就是一种 React 框架；但是通常的诸如 umi 这类的框架都只是 SPA 创建器，构建出的 React 应用是一个几乎全由 JavaScript 执行渲染的、很容易变得非常庞大的应用。这样的 SPA 会产生几个问题：

1. 首页加载太长；因为浏览器需要等到 JS 全部执行完成后才能渲染出页面。如果应用很大，首页加载就会很长。
2. SEO 很差。因为通常 SPA 的 html 文件只有短短几行，然后就是引入主要的 js 文件，这样很不利于 SEO 和信息的爬取

解决这两个问题的方法是服务器渲染（server rendering），也叫静态预渲染（static pre-rendering）。

Next.js 是一个 React 框架，以一种非常简单的方式完成所有这些工作，但它并不限于此。它的创造者把它宣传为一个零配置（zero-configuration）、单指令（single-command）的 React 应用工具链。

## nextjs 功能

这是 nextjs 官网首页的功能截图：

![](https://pic.imgdb.cn/item/6322d98c16f2c2beb1ecba40.jpg)

其中有几个主要功能：

- 0 配置：即 nextjs 的路由采取的是约定式路由，根据目录自动生成路径
- SSR 和 SSG：下面会说
- api 路由：可以创建一些 api，这些 api 会在服务端运行，可以直接在 React 组件代码中调用这些 api 取完成数据库读取、文件读写等操作，而不需要单独创建一个 nodejs 应用
- 自动代码拆分：Next.js 不会生成一个包含所有应用程序代码的单一 JavaScript 文件，而是将应用程序自动分解为几个不同的资源。加载一个页面只加载该特定页面所需的 JavaScript。比如只有一个页面导入了 axios，那么就只会在生成这个页面的 bundle 时打包 axios。

## SSR 和 SSG

### SSR 服务端渲染

顾名思义就是在服务端渲染出 html，再发给浏览器显示。注意这个“渲染”并不是对 html 的解析、绘制等，而是从 React 生成 html、打包 js 的过程。
这样的好处有：

- 客户端不需要实例化 React 来渲染，这使得网站对用户来说更快。
- 搜索引擎会对页面进行索引，而不需要运行客户端的 JavaScript。

虽然 next 有针对 ssr 的专门的函数，但是实际上 nextjs 默认就是开启 ssr 的。如果打开一个默认生成的 nextjs 应用，然后在主页面中查看网页源代码，就会看到引入了一堆 JavaScript 文件，大概是这样：

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta
      name="viewport"
      content="width=device-width,minimum-scale=1,initial-scale=1"
    />
    <meta name="next-head-count" content="2" />
    <link
      rel="preload"
      href="/_next/static/development/pages/index.js?ts=1572863116051"
      as="script"
    />
    <link
      rel="preload"
      href="/_next/static/development/pages/_app.js?ts=1572863116051"
      as="script"
    />
    <link
      rel="preload"
      href="/_next/static/runtime/webpack.js?ts=1572863116051"
      as="script"
    />
    <link
      rel="preload"
      href="/_next/static/runtime/main.js?ts=1572863116051"
      as="script"
    />
  </head>

  <body>
    <div id="__next">
      <div>
        <h1>Home page</h1>
      </div>
    </div>
    <script src="/_next/static/development/dll/dll_01ec57fc9b90d43b98a8.js?ts=1572863116051"></script>
    <script id="__NEXT_DATA__" type="application/json">
      {
        "dataManager": "[]",
        "props": { "pageProps": {} },
        "page": "/",
        "query": {},
        "buildId": "development",
        "nextExport": true,
        "autoExport": true
      }
    </script>
    <script
      async=""
      data-next-page="/"
      src="/_next/static/development/pages/index.js?ts=1572863116051"
    ></script>
    <script
      async=""
      data-next-page="/_app"
      src="/_next/static/development/pages/_app.js?ts=1572863116051"
    ></script>
    <script
      src="/_next/static/runtime/webpack.js?ts=1572863116051"
      async=""
    ></script>
    <script
      src="/_next/static/runtime/main.js?ts=1572863116051"
      async=""
    ></script>
  </body>
</html>
```

首先有 4 个就是文件被 preload，前两个都正好是 nextjs 中主要编写的代码文件（\_app.js 和 index.js），后面则是 nextjs 和 webpack 相关的。 这 4 个文件被加载到 body 的末尾，因此 js 是会在 html 解析之后才执行的，这样就不会阻止 html 的渲染。

> 所加载的 4 个 bundle 文件已经实现了一个叫做代码分割(code splitting)的功能。index.js 文件提供了 index 组件所需的代码，它为/路由提供服务，如果我们有更多的页面，我们将为每个页面提供更多的 bundle，然后只有在需要时才会被加载——为页面提供一个更高性能的加载时间。

### SSG 静态生成

和 ssr 类似，ssg 的主要功能也是在服务器端生成页面，发送到浏览器显示。但是它和 ssr 的区别如下（官方文档）：

- Static Generation 是在构建时生成 HTML 的预渲染方法，这个预渲染的 HTML 会被**反复使用**，也就是说除非是开发环境下的代码更改，否则这个 HTML 不会再重新生成
  ![](https://pic.imgdb.cn/item/6322e0e916f2c2beb1f998d2.jpg)
- Server-side Rendering 在**每次请求**时生成新的 HTML。
  ![](https://pic.imgdb.cn/item/6322e0f616f2c2beb1f9ac7e.jpg)

也就是说，如果是通过 ssg 生成的页面，并且调用了 getStaticProps 获取了数据，那么后面你不管怎么刷新，数据都不会改变；而 ssr 则会在每次刷新之后重新调用 getServerSideProps 去重新获取数据。
这里要注意两点：

1. 不更新不意味着不能交互，这里的不更新指的是数据的不更新，即生成的页面不再重新生成、重新获取数据，而不是 react 的更新
2. ssg 也不是一定不能更新，可以通过 fallback 设置；并且如果开启 revalidate，还能实现在 revalidate 设定的时间范围内不更新，超出时间则重新生成页面，

> 关于官方文档上的“请求”这个词，实际的意思并不是比如 fetch、axios 等数据请求方法，而是指浏览器对页面的请求
> 比如说，在 url 栏输入一个路由并跳转，就相当于对 nextjs 服务器请求了一个新页面。ssr 就是在这个“请求”时生成新的 html，可以理解为每次跳转到这个页面就会新生成一遍。而 ssg 则只会在一开始渲染 html 一次，后续不会再生成

关于什么时候该用 ssr 和 ssg（官方文档）：

- 如果可以在“请求”之前预渲染页面，就应该采用 ssg；也就是说如果页面只是一些不经常变动的、不需要频繁更新的，就可以采用。通常用于博客、商城商品页面、文档等
- 如果页面需要频繁更新数据，就应该采用 ssr。它的特点是更新但稍慢，由于每次跳转都会重新生成 HTML，所以数据总是新的

#### 动态路由下的 ssg

ssg 本质上是一种预渲染，即提前渲染好页面，加载时直接呈现这些页面。因此纯的 ssg 生成的页面如果遇到了事先未生成的路由，就会导致 404。因此如果使用动态路由，显然预先不知道会有什么样的页面生成，就不能使用 ssg（吗）
其实是可以的，但是需要配合 getStaticPaths 函数，即**使用动态路由的页面组件如果要用 ssg，就一定要有 getStaticPaths 和 getStaticProps**，前者为后者指明“要生成哪些页面”。
比如下面，getStaticPaths 返回的 paths 将会作为真正生成的页面，即这里会生成`/posts/1` 和`/posts/2`两个静态页面。
fallback 属性主要用于处理未匹配的情况，详见下

```ts
// pages/posts/[id].js

// Generates `/posts/1` and `/posts/2`
export async function getStaticPaths() {
  return {
    paths: [{ params: { id: "1" } }, { params: { id: "2" } }],
    fallback: false, // can also be true or 'blocking'
  };
}

// `getStaticPaths` requires using `getStaticProps`
export async function getStaticProps(context) {
  return {
    // Passed to the page component as props
    props: { post: {} },
  };
}

export default function Post({ post }) {
  // Render post...
}
```

### 三个主要函数

关于 ssr 和 ssg 的核心函数其实就是这三个：

![](https://pic.imgdb.cn/item/6322e31716f2c2beb1fd3563.jpg)

#### getServerSideProps

上面的`getServerSideProps`和下面两个的根本区别是执行时机的不同，前者是在每次请求运行，后者是在生成静态页面时执行（一般是启动脚手架，或者 build 时）

举个例子，现在有个页面需要向一个 api 请求一个宝可梦数据，然后渲染。

```js
const Pokemon = ({ data }) => {
  return (
    <div>
      <p>{data.name}</p>
      <Image
        src={data.sprites.other["official-artwork"]}
        height={500}
        width={500}
        alt="pokemon"
      />
    </div>
  );
};

export default Pokemon;
```

通常如果不用 ssr 或者 ssg 的话，需要在 useEffect 中请求并渲染。
现在如果使用这两个，就需要单独写一个函数；
首先是 ssr 的核心函数`getServerSideProps`

```js
export async function getServerSideProps({ params }) {
  const { id } = params;
  const res = await fetch("...").then((res) => res.json());
  return {
    props: {
      data: res,
    },
  };
}
```

`getServerSideProps`会接收一个路由的数据，返回值给组件的 props（后面会说）。
它会在**每次请求**时运行，即，每次更改 url，都会重新执行这个函数，请求新的 Pokemon 数据，返回给组件的 props 并更新。因此这个效果其实和客户端请求类似，只是它是以 url 为核心。
我们不需要预先知道用户需要访问哪些宝可梦的 id，因为每次都会单独请求，如果请求超出那是请求的问题，而不是页面本身的问题。

#### getStaticProps

如果要用 ssg，核心函数有两个，`getStaticPaths`和`getStaticProps`。

> 这两个的必须是`getStaticProps`，前者则主要用于动态路由。

静态生成的特点是预先生成页面，这个请求部分如果放在 getStaticProps 中，只会在启动应用时执行一次，全程不论 url 怎么改变都不会重新执行。
基本内容差不多：

```js
export async function getStaticProps({ params }) {
  const { id } = params;
  const res = await fetch("...").then((res) => res.json());
  return {
    props: {
      data: res,
    },
  };
}
```
getStaticProps的返回值还有一个属性revalidate，用于控制刷新时间，即在revalidate设置的时间内重新请求不会重新生成，而在时间之外则会重新生成页面。

```js
export async function getStaticProps({ params }) {
  const { id } = params;
  const res = await fetch("...").then((res) => res.json());
  return {
    props: {
      data: res,
    },
    revalidate: 10 // 10秒之后重新请求会重新生成，10秒之内不会重新生成
  };
}
```

实际上nextjs会每10秒钟重新生成一次。这对于可能会出错的请求至关重要。
即，如果getStaticProps内的错误没有被捕获，那么nextjs并不会直接导致页面崩溃，而是继续渲染最近的一次渲染结果，即最后一个能正常显示的页面。然后按照revalidate设定的时间重新生成，直到没有再次抛出错误为止（就会用最新的没有错误的页面）

#### getStaticPaths

如果要在动态路由页面组件中使用ssg，就必须添加getStaticProps和getStaticPaths。
`getStaticPaths`函数会考虑到用户可能需要的各种页面，然后将这些全部生成

```js
export async function getStaticPaths() {
  const paths = getAllPokemonIds(); // paths的格式：[{params:{id:'aaa'}},{params:{id:'bbb'}}]
  return {
    paths,
    fallback: false,
  };
}
```

这个函数的返回值是固定的，必须返回一个含有 paths 和 fallback 属性的对象；其中，paths 是一个路径数组，它其中的每一项都会传给`getStaticProps`作为参数。相当于多个 paths 数据依次执行`getStaticProps`。

fallback 属性主要用于处理未匹配的情况，取值有：

- false：完全 SSG 的形式，只根据 paths 去预渲染页面，如果有规则外的 paths，前端会显示 404 页面。
- true：遇到匹配不了的 paths 时，也就是服务器上并没有请求对应的已生成好的页面时，服务器会立即去生成一份，而不是返回 404 页面，但这时候需要在页面组件里设置一个 loading 反馈。

```ts
export async function getStaticPaths({params}) {
    return {
        paths: [...]
        fallback: true
    }
}
export const Article = ({article}) => {
    const router = useRouter();
    if(router.isFallback){ return <>Loading...</> }
}
```

- 'blocking'：此时遇到新的path服务端也是会去生成一份新的html的，但没有fallback状态，也就是服务端生成html时前端没有加载状态，可能会造成用户点击后过一两秒才能跳转，这也是为啥叫blocking(阻塞， 堵塞的意思)

这样，在构建时就会预先生成多个可能的页面，当用户 url 跳转时就会把构建好的放上来。

# 基本使用

## 安装

创建和 create-react-app 类似

```
npx create-next-app
npx create-next-app --typescript
```

或者用 create：

```
yarn create next-app
yarn create next-app --typescript
```

启动：

```
npm run dev
yarn dev
```

## 基本目录

![](https://pic.imgdb.cn/item/6322f21e16f2c2beb118115f.jpg)

默认创建下没有 lib（通常用于放一些工具函数）和 components 目录，可以手动创。

- pages：页面组件，按照路径会形成约定式的路由。
- styles：默认开启 css modules，这里是全局 css 和用于示例的 index.js 的样式

# 路由

nextjs 的路由不需要任何额外配置，只需要创建目录即可
唯一需要注意的是动态路由：

常规的路由像这样：
![](https://pic.imgdb.cn/item/6322f32216f2c2beb119e171.jpg)

## 路由对象

获取路由对象的方法有三种：

1. 通过`userRouter()`创建 router 对象

```js
import { useRouter } from "next/router";
const router = useRouter();
```

router 对象的主要属性和方法：
![](https://pic.imgdb.cn/item/6322f70516f2c2beb120737c.jpg)

- pathname：完整路径的字符串，如果是动态路由则会包含中括号而不是具体的值
- query：对象，即 url 的 query 参数，动态路由的 params 也会是 query 的一部分；如果不用三个函数之一，这个对象就是空的。
  举个例子，`/post/abc?foo=bar`的 query 对象就是这样
  ```js
  { "foo": "bar", "pid": "abc" }
  ```
- isFallback：即`getStaticPaths`返回的 fallback 值，通常在组件中获取这个值，用于处理 404 的情况
- push：手动控制路由的方法

  ```js
  router.push(url, as, options);
  ```

  - url：路由路径
  - as：旧版用于控制动态路由，
  - options：配置项，详细参考<a href="https://nextjs.org/docs/api-reference/next/router">官方文档</a>

  参数还可以是一个对象：

  ```js
  router.push({
    pathname: "/post/[pid]",
    query: { pid: post.id },
  });
  ```

  其中 query 值会被直接放到跳转之后的页面的 router.query 中，这里相当于动态路由的参数

2. `getServerSideProps`和`getStaticProps`的参数

```js
export async function getServerSideProps(context) {
  return {
    props: {},
  };
}
```

主要属性：

- params：核心属性，动态路由传入的参数值，是一个对象
- req/res：由于这两个函数都是在服务端运行的，因此包含这两个内容，其中 req 就是上面经常说的“请求”，**`getStaticProps`的参数没有这个属性**
- query：和上面的一样，**`getStaticProps`的参数没有这个属性**

## 动态路由

如果是动态路由，路由页面的文件名需要是中括号：

![](https://pic.imgdb.cn/item/6322f35a16f2c2beb11a4b86.jpg)

注意的是中括号内部的内容会被当成是一个变量，而不是`[id]`这个字符串。
比如这个访问的路由应该是`http://localhost:3000/posts/xxx`这样，至于 xxx 的内容，就需要上面说的三个函数的配置。xxx 的内容会被当作`router.params`传入组件，可以通过`useRouter`创建的`router`对象获取，当然也可以直接在`getStaticProps`或`getServerSideProps`的参数中获取。

> 如果有 getStaticPaths，那 getStaticProps 的参数就是 getStaticPaths 的返回值，getStaticPaths 函数中需要自行创建并返回一个这种形式的对象。
>
> ```js
> {
>     paths:[
>         {
>             params:{
>                 id:1
>             }
>         },
>         {
>             params:{
>                 id:2
>             }
>         }
>     ],
>     fallback: true | false | 'blocking'
> }
> ```

还有一种路由的形式是这样：

![](https://pic.imgdb.cn/item/6322f45d16f2c2beb11c1090.jpg)

类似数组解构，这个形式的路由会匹配后面的所有可能路径

比如

```
/user/a
/user/a/b/c/d/e/f
/user
```

这种情况下 query 的值则是一个包含这些路径的数组

```js
{ "slug": ["a", "b"] }
```

---

还可以是多个动态路由，比如：

```
pages/post/[pid]/[comment].js
```

这时如果访问`post/abc/a-comment`，query 参数就是：

```js
{ "pid": "abc", "comment": "a-comment" }
```

# css

nextjs 默认支持 sass、postcss 和 css modules

这里主要说一下配置相关的问题：css modules 的配置，如果想要配置允许类名从短横线连接转化为小驼峰，需要在 next.config.js 中这样配置：

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  // 从这开始
  webpack: (config) => {
    const rules = config.module.rules
      .find((rule) => typeof rule.oneOf === "object")
      .oneOf.filter((rule) => Array.isArray(rule.use));

    rules.forEach((rule) => {
      rule.use.forEach((moduleLoader) => {
        if (/css-loader[/\\](?:cjs|dist|src)/.test(moduleLoader.loader)) {
          if (typeof moduleLoader.options.modules === "object") {
            moduleLoader.options.modules = {
              ...moduleLoader.options.modules,
              // 如果更改输出类名也在这里
              exportLocalsConvention: "camelCaseOnly", // https://github.com/webpack-contrib/css-loader#exportlocalsconvention
            };
          }
        }
      });
    });

    return config;
  },
};

module.exports = nextConfig;
```

如果要更改输出类名，也是在这个位置。

另外，css modules 的全局样式需要被放入`_app.js`文件中，注意不是`index.js`。

# 数据获取和预渲染

这一部分主要就是讲三个函数，即 nextjs 核心的三个函数的使用

## getStaticProps

上面说过，静态生成（ssg）是在服务端预先生成需要的页面，并且是在构建时期完成，后续除非代码变动，否则就不再改变。
ssg 的核心函数就是 getStaticProps，生成的过程可以选择不需要外来数据，这时也就不需要这个函数，nextjs 默认就会按照静态生成的方式生成页面。

![](https://pic.imgdb.cn/item/632318bb16f2c2beb163a91d.jpg)

如果需要预先获取一些数据，比如向 api 发起请求，读写文件等，就需要 getStaticProps 函数。

getStaticProps 函数的特点如下：

1. 永远在服务端执行。即，在 next 创建的 nodejs 环境下执行，函数里边的内容实际上也都是被放在 nodejs 服务端的。这也就意味着可以在函数内访问文件系统、获取 node 环境变量、操作数据库等通常的服务端操作

> 有个小的注意的点，如果要操作文件，路径应该用`process.cwd()`代替`__dirname`，原因是函数的执行实际上被放在了单独的`.next`文件夹下，因此后者用到可能会出错。

2. 函数接受一个路由对象 context（详见上），返回值作为对应页面组件的 props。所以返回值必须是一个包含 props 属性的对象：

```js
export async function getStaticProps(context) {
  return {
    props: { message: `Next.js is awesome` },
  };
}
```

除了 props，还可以有其他属性：

```js
return {
  props: {
    posts,
  },
  revalidate: 10, // 单位秒
};
```

这个属性用于控制多久重新生成当前页面，相当于刷新页面。

还有几个属性是需要单独返回的，不和 props 属性合并在一起

- redirect：导向一个内部或外部的 url，如果返回这个对象，就会直接导向另一个 url

```js
export async function getStaticProps(context) {
  const res = await fetch(`https://...`);
  const data = await res.json();

  // 如果获取不到就返回某个路由
  if (!data) {
    return {
      redirect: {
        destination: "/",
        permanent: false, // 为true的话，还可以控制301、302等重定向状态码
        // statusCode: 301
      },
    };
  }

  return {
    props: { data },
  };
}
```

- notfound：如果返回，那么即使页面已经生成，还是会返回 404。常用于比如商品被下架、文章被作者删除这些情况

```js
export async function getStaticProps(context) {
  const res = await fetch(`https://.../data`);
  const data = await res.json();

  if (!data) {
    return {
      notFound: true,
    };
  }

  return {
    props: { data },
  };
}
```

## getStaticPaths

为 getStaticProps 返回可能的路径

```js
export async function getStaticPaths() {
  return {
    paths: [
      { params: { ... } }
    ],
    fallback: true, false or "blocking"
  };
}
```

### paths

paths 的可能格式有几种：

1. 通常情况：

```js
 paths: [
      { params: { ... } },
      { params: { ... } },
      { params: { ... } },
    ],
```

2. 多个动态路由，比如`pages/posts/[postId]/[commentId]`这样的，这时 params 的属性就应该包含这两个

```js
paths: [
      { params: {
        postId:001
        commentId:002
       } },

    ],
```

需要注意的是，params 的属性是大小写敏感的，即`params:{id:1}`和`params:{ID:1}`是两个不同的值

### fallback

fallback 用于处理访问到没有对应的路由时的情况。
比如预先考虑到的 paths 包含的 id 只有 1-20，但是用户此时访问了 50，明显不在生成范围内，这时就要对这种情况处理。
fallback 的值有三个：

- false：不处理，即超出直接返回 404
- 'blocking'：如果超出范围就会转化为 ssr，即开始进行动态生成
- true：不会返回 404，而是正常返回给`getStaticProps`；这时如果超出了返回，数据可能就变成 undefined，因此通常用于自行处理 404 的情况。

比如：

```js
function Post({ post }) {
  const router = useRouter();

  // 如果访问到的超出paths包含的内容，这个情况就会被触发，即自行处理404的情况
  if (router.isFallback) {
    return <div>Loading...</div>;
  }
}

export async function getStaticPaths() {
  return {
    paths: [{ params: { id: "1" } }, { params: { id: "2" } }],
    fallback: true,
  };
}
```

## getServerSideProps

基本使用和 getStaticProps 差不多，返回值的形式完全相同，参数略有不同

getServerSideProps 的参数 context 看起来更像服务端的形式，大概是这样：
![](https://pic.imgdb.cn/item/63232ab716f2c2beb1871c31.jpg)
可以看到相对于 getStaticProps，添加了 req 和 res 两个主要对象，即来自客户端的“请求”和当前的“响应”。

# api 路由

在 pages/apis 下创建的文件会被当作 api 路由。所谓 api 路由实际上就是创建了一个函数作为接口；
和三个主要函数一样，api 路由的函数也是完全运行在服务端的，因此可以进行文件读写、数据库操作等。

比如官方示例给的 hello.js

```js
export default function handler(req, res) {
  res.status(200).json({ name: "John Doe" });
}
```

函数的参数 req 和 res 就是 nodejs 中的 req 和 res；其中 req 来自客户端代码发送的请求。**注意不要从三个函数内部向 api 路由发请求**，通常是在客户端代码中写。


