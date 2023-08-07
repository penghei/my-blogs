# 关于项目

对于项目中用到的轮播组件，有什么可以扩展的地方？如果想要进一步完善组件可以怎么做？如果要把组件发布成一个库，又要注意什么？有哪些可以做的优化？

项目收获，第二个项目收获还是挺能说的，konva原理、canvas绘制方法、状态管理库等等。第一个项目其实主要收获和项目难点不相关，收获的地方可能是优化知识。如果非要说难点以及收获的话，就是轮播图的实现方式、实现原理，以及优化方式、虚拟列表的实现和应用这些的。

轮播图自行实现的目的，为啥不用现成，这个要说自己实现的轮播图有什么功能是其他没有的，而不仅仅是因为框架的要求。

项目要注意应该要弱化rn的部分，尽可能让项目偏向大众化，偏向比较通用的方向。比如优化的思想也可用到web端，轮播图这个业务也算在web端常见的。


关于项目里的地图，还有什么可以做的地方？对于地图的功能，业内有没有什么实现方案、优化方案，以及各种放缩、绘制的实现？（目前想到的：分片）地图拖拽和放大缩小的性能问题？

token的过期处理，如何处理同时多个请求过期导致重复跳转首页？如何前端刷新token？（掘金收藏里有相关文章）

学习了react的源码，有没有在实际项目中用到源码中的设计思想，或者是比较优秀的代码设计的（考虑：位运算，合并、比较、检查、分离）？如果不是react，其他源码有没有（webpack的tapable可以说下）？React18的新特性有没有用在项目中的？（transition）

# 关于八股

- TLS不同版本的差异？✅
- https的证书链✅
- QUIC协议的原理，如果想在udp的基础上实现http，应该怎么做？https://xiaolincoding.com/network/3_tcp/quic.html ✅
- token相关八股 ✅
- 发布订阅模式的基础和高级实现，更多的功能（比如异步、错误处理）。可参考[pubsubjs](https://github.com/mroderick/PubSubJS/blob/master/src/pubsub.js)和[tapable](https://github.com/webpack/tapable/blob/master/lib/Hook.js)
- why-did-you-render等调试工具的原理（简历上写的东西都要起码了解原理，还有eslint）✅
    - wdyr的简单原理：重写React.createElement等方法，获取到element对象的props及其他属性。对组件进行监听，更新时比较props
- SSR！SSR的基本原理和实现方式
- 新知识，即问题“有没有了解前端领域新知识”这样的
- IntersectionObserver
- 怎么让倒计时更精准？（与服务端的配合）
- 图片懒加载、预加载方案？如何实现一个懒加载组件，如何在h5、rn中手动预加载图片或视频
- rn判断可视区域的方法、曝光检查方法[react-native-intersection-observer](https://github.com/zhbhun/react-native-intersection-observer#readme)；rn吸顶方法
- canvas深入
- rem的设置，rem布局如何用js设置合适的font-size，值应该是多少？js执行滞后导致页面布局变化怎么办？移动端键盘弹出导致rem变化怎么处理？媒体查询实现的缺点（不能覆盖全部情况）

（rem的设置找了段代码）
```js
docEl = document.documentElement
var clientWidth = docEl.clientWidth;
var clientHeight = docEl.clientHeight;
if (!clientWidth || !clientHeight) return;
var ratio = clientHeight / clientWidth;
if (ratio < 1.7 && ratio > 1) {
  // 防屏幕过矮
  clientWidth = Math.round(clientHeight / 1.7);
}
if (ratio < 0.526) {
  // 防屏幕过矮
  clientWidth = Math.round(clientHeight / 0.526);
}
//主要是对宽高的比较
if (clientWidth < clientHeight) {
  //竖屏
  docEl.style.fontSize = 100 * (clientWidth / 750) + "px";
} else {
  //横屏
  docEl.style.fontSize = 100 * (clientWidth / 1400) + "px";
}
```

- modal的实现和封装，定位方式和背景遮罩实现，背景滚动控制
- useContext底层
- 上传和下载的区别（不太清楚具体考什么，请求格式？）
- 线上错误监控，之前考过
- 报警组件，react错误边界，包含处理网络请求、异步错误
- 线程，为什么有的应用需要很多线程，IO密集型和计算密集型
- 进程间通信的具体实现方式，管道实现方式
- 常见组件库组件设计思路，如何封装，复杂一点的，比如表单表格、treeSelect等
- 微前端，了解
- 错误上报和性能监控使用的上报方式有没有更好的（除了请求方面）https://juejin.cn/post/6844903648355418120#heading-13
- Priority组件的设计，有一点像React的更新优先级设置？能不能想办法串起来