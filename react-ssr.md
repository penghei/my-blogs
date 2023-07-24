# SSR概述

SSR 是 JSP、PHP 时代就存在的古老的技术，只不过之前是通过模版引擎，而现在是通过 node 服务渲染组件成字符串，客户端再次渲染，这种叫做同构渲染的模式。

React SSR 是服务端通过 renderToString 把组件树渲染成 html 字符串，浏览器通过 hydrate 把 dom 关联到 fiber 树，加上交互逻辑和再次渲染。

服务端 renderToString 就是递归拼接字符串的过程，遇到组件会传入参数执行，遇到标签会拼接对应的字符串，最终返回一段 html 给浏览器。

浏览器端 hydrate 是在 reconcile 的 beginWork 阶段，依次判断 dom 是否可以复用到当前 fiber，可以的话就设置到 fiber.stateNode，然后在 completeWork 阶段就可以跳过节点的创建。

# SSR原理

## SSR/CSR/React SSR

关于ssr和csr的区别，其实可以看这张图来更直观地感受：

![](https://pic.imgdb.cn/item/64be5cb61ddac507cc986f42.jpg)

ssr其实是传统ssr的代指。现代ssr的代表则是react ssr，它并不等同于传统ssr的直接把数据嵌入页面内输出，而是结合了ssr和csr的优点的产物。
第一次打开页面是服务端渲染，基于第一次访问，用户的后续交互是 SPA 的效果和体验，于此同时还能解决SEO问题

> csr的优缺点自然不必多说，ssr的有一个缺点就是所有页面的加载都需要向服务器请求完整的页面内容和资源，访问量较大的时候会对服务器造成一定的压力。这在现代ssr中也是无法避免的




