# 关于项目

对于项目中用到的轮播组件，有什么可以扩展的地方？如果想要进一步完善组件可以怎么做？如果要把组件发布成一个库，又要注意什么？有哪些可以做的优化？

轮播图自行实现的目的，为啥不用现成，这个要说自己实现的轮播图有什么功能是其他没有的，而不仅仅是因为框架的要求。

项目要注意应该要弱化rn的部分，尽可能让项目偏向大众化，偏向比较通用的方向。比如优化的思想也可用到web端，轮播图这个业务也算在web端常见的。


关于项目里的地图，还有什么可以做的地方？对于地图的功能，业内有没有什么实现方案、优化方案，以及各种放缩、绘制的实现？（目前想到的：分片）

token的过期处理，如何处理同时多个请求过期导致重复跳转首页？如何前端刷新token？（掘金收藏里有相关文章）

学习了react的源码，有没有在实际项目中用到源码中的设计思想，或者是比较优秀的代码设计的（考虑：位运算，合并、比较、检查、分离）？如果不是react，其他源码有没有（webpack的tapable可以说下）？React18的新特性有没有用在项目中的？（transition）

# 关于八股

TLS不同版本的差异？
https的证书链
QUIC协议的原理（深入）
如果想在udp的基础上实现http，应该怎么做？https://xiaolincoding.com/network/3_tcp/quic.html
token相关八股
发布订阅模式的基础和高级实现，更多的功能（比如异步、错误处理）。可参考[pubsubjs](https://github.com/mroderick/PubSubJS/blob/master/src/pubsub.js)和[tapable](https://github.com/webpack/tapable/blob/master/lib/Hook.js)

