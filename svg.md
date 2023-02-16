---
title: svg学习
date: 2022-01-28 14:00:12
tags: 日常学习
cover: /img/svg.png
categories: 数据可视化相关
---

### svg 基本语法

svg 可以作为一个单独的文件，但是更多是作为`<svg>`标签插入在 html 文件中
基本语法：

```html
<svg
  width="500"
  height="500"
  viewBox="0 0 50 50"
  xmlns="http://www.w3.org/2000/svg"
  xmlns:xlink="http://www.w3.org/1999/xlink"
  version="1.1"
>
  <circle cx="0" cy="0" r="10" fill="blue" stroke="black" stroke-width="1px" />
</svg>
```

svg 标签上还有额外的属性：

- width、height 表示 svg 元素大小，默认是 300px\*150px；可以修改为任意值，效果和普通的块级元素相同
- xmlns：表示 svg 元素的**默认命名空间**。该属性表示<svg>标记及其子标记属于名称空间为`http://www.w3.org/2000/svg` 的 XML 方言
  如果在 html5 中使用不需要这个标签和之后的两个，但是“如果将页面作为 image/svg+xml 或 application/xhtml+xml 或任何其他导致用户代理使用 XML 解析器的 MIME 类型，则需要 xmlns 属性”
- xmlns:xlink：xlink 是命名空间前缀，其后的属性是一个新的命名空间。
- version：表示版本
- viewBox 属性的值有四个数字，分别是左上角的横坐标和纵坐标、视口的宽度和高度。

```html
<svg viewBox="0 0 50 50" width="100" height="100"></svg>
```

视口相当于**截取**svg 图像的一部分，其中截取的开始是前两个值；  
后两个值表示截取宽高，但是**视口必须适配所在的空间**。上面代码中，视口的大小是 50 x 50，由于 SVG 图像的大小是 100 x 100，所以视口会放大去适配 SVG 图像的大小，即放大了四倍。

### svg 坐标系

svg 坐标以左上角为(0,0)，向右、下为正方向，单位为 px；
svg 内部的坐标系是基于 svg 元素本身的相对坐标

### 图形元素

#### circle

```html
<circle cx="0" cy="0" r="10" fill="blue" stroke="black" stroke-width="1px" />
```

- cx 和 cy 为圆心坐标
- r 表示半径，注意上面代码只会显示 1/4 圆，因为圆心坐标在原点
- fill 表示元素内部填充颜色，默认为黑；fill 的值可以是任意 css 颜色，包括文字或者 rgba
- stroke 表示边框，stroke 属性指边框的颜色，其下还有几个分支属性，后面会讲到

#### line

直线

```html
<line x1="10" x2="50" y1="110" y2="150" />
```

1 和 2 分别表示起点和终点；x1 表示起点的横坐标，y1 表示起点的纵坐标，其他同理
另外 line 的 fill 属性一般不生效，所以改变颜色或宽度要使用 stroke

#### rect

矩形

```html
<rect x="10" y="10" width="30" height="30" />
<rect x="60" y="10" rx="10" ry="10" width="30" height="30" />
```

- x：矩形左上角的 x 位置
- y：矩形左上角的 y 位置
- width：矩形的宽度
- height：矩形的高度
- rx：圆角的 x 方位的半径
- ry：圆角的 y 方位的半径

#### ellipse

椭圆

```html
<ellipse cx="75" cy="75" rx="20" ry="5" />
```

cx 和 cy 类似圆的中心坐标，rx 和 ry 表示横轴和纵轴长度

#### polyline 折线、polygon 多边形

这两个用的不多，都是利用多个点来绘制多边形，一般被更灵活的 path 代替

#### path

path 元素的形状是通过属性 d 定义的，属性 d 的值是一个“命令+参数”的序列
在 d 之后的命令形式为字符串

```
M x y
或者
m x y
```

##### 直线命令

主要有五种：

- M，表示把笔刷移动到的位置，但是不画线；M 表示绝对位置，m 表示基于上一个点的相对位置，其他命令类似
- L，画线，基于 m 定位的点画线
- H/V，分别表示画水平/垂直线，因此参数只有一个，表示向 x 轴/y 轴延伸的距离；当然也有 h/v 的形式，可以用负数表示反向
- Z，闭合当前路径，不分大小写，会把最后一个点和起点连起来；尽管我们不总是需要闭合路径，但是它还是经常被放到路径的最后。

```html
<path d="M10 10 h 80 v 80 h -80 Z" fill="transparent" stroke="black" />
<path d="M10 10 H 90 V 90 H 10 Z" fill="transparent" stroke="black" />
```

##### 曲线

path 绘制的是贝塞尔曲线，有关贝塞尔曲线的讲解可以参考这里<a href="https://zh.javascript.info/bezier-curve">贝塞尔曲线</a>
主要要知道：贝塞尔曲线绘制的核心是`控制点`和`t值`，n 个控制点对应 n-1 阶贝塞尔曲线；在 svg 中，绘制贝塞尔曲线只需要起点、终点、控制点三种坐标

以三阶贝塞尔曲线为例：

```html
<path d="M10 10 C 20 20, 40 20, 50 10" stroke="black" fill="transparent" />
```

三阶贝塞尔曲线的绘制是命令 C，C 的参数如下：

```
C x1 y1, x2 y2, x y
```

三阶贝塞尔曲线一共需要四个点，除了两个控制点之外还有起点和终点；起点用 M 命令指定

- x1 y1 和 x2 y2 表示两个控制点坐标，曲线不一定经过该两点，并且如果同时经过所有控制点则为直线（一阶贝塞尔）
- x y 表示终点，和 M 命令规定的起点连起来绘制，曲线一定经过起点和终点

同理还有二阶贝塞尔：

```
Q x1 y1,x y
```

二阶只需要一个控制点和起点终点
上面两个命令也可以用小写字母表示相对位置；

---

另外，在这两个命令之后可以用其他命令表示延伸  
C 对应 S 命令，S 命令可以用来创建与前面一样的贝塞尔曲线，如果 S 命令跟在一个 C 或 S 命令后面，则它的第一个控制点会被假设成前一个命令曲线的第二个控制点的中心对称点。

```
S x3 y3,x y
```

意思就是这样：下图中的蓝色点就是 S 的第一个控制点，是原先曲线第二个控制点（红色第二个）的对称，然后 x3 y3 就表图中第四个控制点
![](https://developer.mozilla.org/@api/deki/files/363/=ShortCut_Cubic_Bezier.png)

---

Q 对应 T 命令，T 的效果和 S 类似

```
T x y
```

T 只需要一个参数指明新终点即可，新曲线的控制点仍然是之前控制点的对称点

##### path的api

下面两个api可以用做获取路径长度

```js
SVGPathElement.getTotalLength()
```

指示路径总长度(以用户单位为单位)的浮点数。其中`SVGPathElement`指的是svg的path元素

```js
SVGGeometryElement.getPointAtLength(distance) 
```

沿路径返回给定距离的点，`SVGGeometryElement`表示任意svg元素，参数是一个浮点数，表示长度，返回一个距离起点为参数长度的点坐标



### 组合元素

内部包裹一些图形元素，组合起来有不同的作用

#### use

用于复制一个形状。`<use>`的 href 属性指定所要复制的节点，x 属性和 y 属性是`<use>`左上角的坐标。另外，还可以指定 width 和 height 坐标。

```html
<svg viewBox="0 0 30 10">
  <circle id="myCircle" cx="5" cy="5" r="4" />

  <use href="#myCircle" x="10" y="0" fill="blue" />
  <use href="#myCircle" x="20" y="0" fill="white" stroke="blue" />
</svg>
```

#### g

包裹数个图形元素为一个整体，g 元素上的 id 可以在 use 标签中直接引用；注意一定是 id，不能是 class 或者其他选择器

```html
<svg width="300" height="100">
  <g id="myCircle">
    <text x="25" y="20">圆形</text>
    <circle cx="50" cy="50" r="20" />
  </g>

  <use href="#myCircle" x="100" y="0" fill="blue" />
  <use href="#myCircle" x="200" y="0" fill="white" stroke="blue" />
</svg>
```

#### defs

用于自定义形状，它内部的代码不会显示，仅供引用。

```html
<svg width="300" height="100">
  <defs>
    <g id="myCircle">
      <text x="25" y="20">圆形</text>
      <circle cx="50" cy="50" r="20" />
    </g>
  </defs>

  <use href="#myCircle" x="0" y="0" />
  <use href="#myCircle" x="100" y="0" fill="blue" />
  <use href="#myCircle" x="200" y="0" fill="white" stroke="blue" />
</svg>
```

#### pattern

`<pattern>`标签用于自定义一个形状，该形状可以被引用来平铺一个区域。

```html
<svg width="500" height="500">
  <defs>
    <pattern
      id="dots"
      x="0"
      y="0"
      width="100"
      height="100"
      patternUnits="userSpaceOnUse"
    >
      <circle fill="#bee9e8" cx="50" cy="50" r="35" />
    </pattern>
  </defs>
  <rect x="0" y="0" width="100%" height="100%" fill="url(#dots)" />
</svg>
```

关于 pattern 标签的坐标系有不同的情况，也就是`patternUnits`属性的两个值：`userSpaceOnUse`和`objectBoundingBox`
`patternUnits="userSpaceOnUse"`是使用基于绝对的尺寸，如果是默认则为`objectBoundingBox`，pattern 的 width 和 height**不能大于 1**，否则只会显示一遍图形

```html
<defs>
  <pattern id="Pattern" x="0" y="0" width=".25" height=".25">
    <rect x="0" y="0" width="10" height="10" fill="blue"></rect>
    <circle cx="10" cy="10" r="10" fill="green" />
  </pattern>
</defs>
```

### fill 和 stroke

fill 属性设置对象内部的颜色，stroke 属性设置绘制对象的线条的颜色，可以使用与 css 相同的颜色，本身这两个属性也可以在 css 中设置
除了颜色属性，还有其他一些属性用来控制绘制描边的方式。

#### stroke-linecap

控制边框终点的形状
![](https://developer.mozilla.org/@api/deki/files/355/=SVG_Stroke_Linecap_Example.png)
主要生效在 line 或者 path 绘制的单个线条起点、终点上
stroke-linecap 属性的值有三种可能值：

- butt 用直边结束线段，它是常规做法，线段边界 90 度垂直于描边的方向、贯穿它的终点。
- square 的效果差不多，但是会稍微超出实际路径的范围，超出的大小由 stroke-width 控制。
- round 表示边框的终点是圆角，圆角的半径也是由 stroke-width 控制的。

#### stroke-linejoin

控制两条描边线段之间，用什么方式连接。
![](https://developer.mozilla.org/@api/deki/files/356/=SVG_Stroke_Linejoin_Example.png)
主要生效在 rect 这样的图形的角上，控制角的形状

#### stroke-dasharray

stroke-dasharray 属性的参数，是一组用逗号分割的数字组成的数列。每一组数字，第一个用来表示填色区域的长度，第二个用来表示非填色区域的长度。
比如`stroke-dasharray="5,10"`，表示实线部分长度为 5，空白部分长度为 10，然后依次排列成一条虚线；
注意：这个数字一定是偶数组，如果是奇数则会复制一遍；比如`"5,5,10"`会变成`"5,5,10,5,5,10"`，效果就是 5 实线、5 空白、10 实线的重复

#### stroke-dashoffset

设置线的偏移量，也就是边线到起点的距离
利用这两个属性可以做出一个“画线”的动画，如下所示

```html
<body>
  <svg width="500" height="500">
    <path
      id="mypath"
      d="M 0 0 h 500 v 250 h -500 z"
      fill="transparent"
      stroke="black"
      stroke-dasharray="5,10,5"
    ></path>
  </svg>
</body>
<style>
  #mypath:hover {
    animation: move 3s linear forwards;
    stroke-dasharray: 1500px, 1500px;
  }
  @keyframes move {
    0% {
      stroke-dashoffset: 1500px;
    }
    100% {
      stroke-dashoffset: 0;
    }
  }
</style>
```

原理：最初线条分为`总路径长度`实线，和`总路径长度`空隙，但是由于设置了 offset 使线条偏移不可见了，当不断修改 offset 后，线条慢慢出现。
