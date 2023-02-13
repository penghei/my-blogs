---
title: react练手项目-网易云音乐总结
date: 2021-11-09 17:00:38
tags: 项目
categories: React
cover: /img/cloudmusic.jpg
---

# 写在前面

之前就有想用 react 练手做点东西的打算,最近终于弄出来了一个小项目,加上后面的复习,react 的学习也算是阶段性胜利了那么一点点(当然路还很长).这个项目使用 react+js,react-redux 以及 react-router 和其他一些库完成,后续如果学习了新技术新方法还可能做改进.现在先来总结一下这个项目的经验和想法
另外,放一下链接:https://github.com/penghei/myCloudMusic
具体的页面展示和介绍就不写了,包括网易云音乐的 api 也就不做介绍了,接下来只谈谈自己的开发过程和总结的经验

# 技术栈

- react + js
- react-router
- react-redux
- sass
- pubsub-js uuid-js 等库

# 主要功能

利用网易云音乐的 api(实际上是一个小服务器)获取一些如歌曲/用户信息等数据,再做一个模仿网易云音乐的界面.总体功能不多,但是主要功能是有了,其他的很多是相似的数据展示和接口处理,就不再做了

- 登录
- 首页日推歌单-歌单展示
- 首页最新音乐
- 音乐播放
- 音乐详情,歌词滚动
- 搜索
- 播放列表
- 歌单列表和歌曲列表
- 排行榜

# 开发经验总结

## CSS 部分(其实是 SCSS)

### 布局

这次的布局几乎全程用到了 flex 和少数的 grid,出乎意料的好用,比 UI 库提供的不知道好用了多少倍.大可到页面布局,小可到位置微调,都可以用 flex,真的是好用的一批

#### vh/px/百分比的取舍

**首先,vh 不是万能的**.这次开发开始的时候有个很大的问题就是很多地方一直用 vh,导致后面响应式非常难做.**vh 只是页面高度,除元素高度之外的属性尽量不要用!!!**而 px 在很多时候也是必要的,设定固定的 px 值才不会在页面改变时元素乱跑和变形.而不是一味使用 vh 或百分比

#### 宽度和高度的限制

在一些元素最开始由于想看到布局,会直接定死一个高度或宽度.但是这个不是万能的,比如页面高度,如果确定没有页脚或者页脚不会凑上来的情况下,尽量还是让元素自己充满页面好一点.宽度也是同理,如果定死当页面缩小时里边元素并不会浮动.
块级元素默认宽度是 100%,如果不为了缩小就尽量不要定死宽度;
为了撑开主界面,布局时主页面元素可以用 100vh,但是其他地方尽量不要用

#### 页头和页脚

```css
#head {
  height: 20px;
}
#foot {
  height: 30px;
}
#main {
  height: calc(100vh - 20px - 30px);
}
```

这样的布局能做到纵向响应式,变化的只有主界面;页头和页脚相当于定死了

#### 内部滚动

如果想做到页头页脚和侧边栏不动,只有内部页面滚动的效果,只需要在内部 div 设定高度即可

```html
<div>
  <div id="head"></div>
  <div id="main">
    <div id="sider"></div>
    <!--sider在外面也可以-->
    <div id="scoll-main" style="height:'150vh'"></div>
  </div>
  <div id="foot"></div>
</div>
```

```css
#main {
  display: flex;
  height: calc(100vh - 50px - 45px);
  flex-direction: row; /*sider固定住*/
}
#scoll-main {
  width: 100%;
  min-width: 100%;
  overflow-y: scroll; /*滚动起来*/
}
```

### 小样式

#### justify 和 align 的使用

首先,**row 横 colmun 列**要记清;其次,justify 是主轴,也是 row 的横向和 colmun 的纵向;align 则是另外一个 center 也不是唯一的选项,space-around 之类的也很好用
留两个链接:
https://www.ruanyifeng.com/blog/2015/07/flex-grammar.html (flex 的)
https://www.ruanyifeng.com/blog/2019/03/grid-layout-tutorial.html (grid 的)

#### max 和 min

**不要滥用**.一般宽度如果要定死直接 width 就可以,而高度最好让元素自适应充满

### SCSS 使用

前面写了一篇 scss 使用的博客,主要用到的语法其实不多,主要是以下几个:

- 嵌套,节省了很多选择器
- 引入,把常用的(比如 flex)的几句写到单独文件中,再反复引入使用
- 父选择器,可以在里边就用伪类伪元素和媒体查询,不单独写
- 同名前缀样式提取,比如这样

```css
border: {
  style: solid;
  width: 1px;
  color: gray;
}
```

- 变量和混入,如果实在用了很多或者统一样式可以用;但是写一两遍那种就没必要了

### 其他

这次写完发现 UI 库也不是那么万能,当然类似弹框/抽屉/按钮这一类可以使用,但是表格这类需要比较细的操作的组件还是自己手写比较好

## React 部分

### 总-经验总结

#### react

- hooks 比类组件好用!(除了万恶的死循环)
- 不是所有的变量都放用 state:如果不是更新页面的 state,可以用 useRef 或者全局变量;但是注意如果要把这种数据放在 jsx 上,要记得 state 更新页面后才会有变化
- useRef 可以用来存一些值以代替全局变量,更多的是用在"前一个值"或者"最新值"
- useEffect 的依赖可以当做一个监听效果,即把要监听的值放入依赖中;
- 如果不是为了完成特定的声明周期,依赖最好不要为空
- 如果 useEffect 出现奇奇怪怪的错误,先分析依赖;再考虑内部是不是有 state 更新;最后考虑使用 useLayoutEffect 或者 useMemo 等
- 多个同级组件处理事件善用事件委托;event 事件 yyds
- map 中的事件处理:`onClick={(e)=>handleDelete(obj,e)}`
- map 和 forEach 区别只在于是否返回,forEach 是没有返回的
- setState 是异步的,不要指望 set 万就可以取出值来用;实在需要可以考虑 useEffect 的监听效果,或者先取出 state 值用
- jsx 中 onclick 等事件不能直接写`onClick={setState()}`,会引起死循环;要写可以`{()=>{setState()}}`这样
- 条件渲染不止有三目运算,&&和||可以做到根据前面的值选择渲染,&&相当于`if(true) <div></div>`
- 数组和对象都不能用`if(x === [])`来判空,会一直为真;数组可以用`length===0`判断,对象可以`JSON.stringfy(x) === "{}"`判空

#### router

- Route 最好包在 Router 里,可以`BrowserRouter as Router`然后包裹;
- 如果出现判断不清楚的情况,可以考虑在 Router 的父元素加一个 key,`key={props.location.key}`
- 路由在哪显示就在哪注册,子路由前面加上父路由的路径
- 路由传参:state 和 query 最方便,直接`props.history.push({pathname: '/main/music',state:data})`即可,用`props.location.state`接收

#### react-redux

- react-redux 相比 redux 好的地方在于,react-redux 传来的 state 是相当于 props,因此如果 reducer 的 state 改变,使用 state 的组件会直接更新:就相当于不用监听了
- 使用 react-redux 的话最好直接把 state 放在 reducer 里边,不要把 reducer 当做临时存储,而是直接替换原来组件的 state
- reducer 实际上是维护了一个 state,所以 getState 就相当于 get 了这个 reducer 的 state,而 actions 就是对这个 state 做改动
- connect 第一个参数获取值的时候,取值应该是 props.+键名,而不是 reducer 的名称(不要搞错了)

### 文件结构

全部组件如图:
![组件.png](https://i.loli.net/2021/11/09/jantl3fKLZcJGIY.png)
确实很多,但是由于命名清晰和文件划分,其实文件结构还是比较清晰的.所以文件结构设计尽量要遵循这次或者以后更好的原则:

- components 内按照功能分类文件夹,只使用一次的可以加 The 之类的前缀;
  - 同文件夹(功能)内部文件名以相同词开头
  - css 文件单独放置,和 jsx 放在一起或者单独放在 css 都可以,最好与文件(夹)同名
- pages 文件不要写过多逻辑,最好不要维护状态,props 可以
  - pages 文件按照布局分类,有嵌套文件放置也要嵌套
- redux 和上述同级
  - reducers 命好名,单独放在文件夹里,最好小驼峰,与导出一致
- 自定义 hooks 写在单独 Hooks 文件夹中,文件名和导出名前面都要有 use

### audio 对象使用总结

audio 对象主要用到的属性和方法:

- src,用于指定播放 url,可以动态指定,但是需要播放前调用 load()
- currentTime,可读可写的值,表示当前播放时间
- duration,时长
- ended,布尔值,是否结束
- load() 加载方法,src 动态引入需要
- play()/pause(),播放暂停,**注意只有用同一个实例的情况下才可以播放暂停,不然会导致从头播放**(意思就是尽量使用同一个 audio 对象)
- addeventlistener('timeupdate')事件,每 250 毫秒触发一次,可以用于获取播放时间,判断是否播放完成等

#### 进度条

设计思路:根据`currentTime/duration`来计算百分比设置 div 的宽度,然后 timeupdate 事件更新进度条
点击事件:使用`e.pageX / document.body.scrollWidth`判断点击位置,然后更改 currentTime 即可

#### 滚动歌词

设计思路:

1. 获取歌词文件,把歌词文件每句话前面的事件拆出来并计算好,和每句歌词合成一个对象,相当于这样

```js
let lyricObj = {
  lyricTime: 32.09,
  lyricInner: "aaaaa",
};
```

2. timeupdate 事件获取播放时间,根据播放时间和歌词事件(均取整)找出对应的歌词对象,然后找出前 n 句和后 n 句,设置为 state
3. 每次找出新的当前歌词都会更新 partLyrics.当然开始和结束也要保证不溢出
4. 设置高亮歌词:前 n 句专门设置,n 句后就固定为中间的一句

### 其他

# 总结

这次项目总体代码行数不是很多,算上后期的反复改进也只不过是 2000 余行.但是代码量并不是衡量代码质量的标准,个人感觉 react 的代码确实会比 vue 少一点?而且由于 jsx 和一些 react 的特性,代码似乎更精简很多.当然还是有很多地方没做好的,比如歌词,音乐播放器等,由于是自己手写的代码,很多还是不成熟不精简.不过这次也算是有了这方面的经验,下次如果再使用音频或者文本动态匹配,就可以用类似的方法了
接下来的一段时间外部的安排有点多,先暂时不学太多新东西了.如果有时间,可以多看看 ts+react/ts+vue3,最起码要用 ts 把这两个框架常见的问题和常使用的都实现一遍,才算是初步学完 ts.ts 学完之后再考虑测试以及之后了
