---
title: CSS学习总结
date: 2021-12-10 10:41:12
tags: 日常学习
cover: /img/CSS.png
categories: CSS
sticky: 9
---

# 基本概念

## 层叠和继承

### 层叠

css 的“C”就是层叠（cascade）的意思，css 中的层叠意思是一个元素的不同设定可以给与元素相互覆盖的样式；比如

- 开发者设定的样式优先级最高，在其中又分为行内样式、内联样式、外部引入样式，优先级递减
- 用户在浏览器调试工具设定的样式其次
- 浏览器默认样式最低

### 继承

一些属性在父元素上的 css 属性是可以被子元素继承的，有些则不能。
一般来说子元素指的都是直接子元素，也就是说，继承一般只会继承给自己的直接子元素。
虽然子元素还会往后继承到子元素的子元素，一直到所有后代元素；但是如果中间某个元素设置了不继承，那从这个元素之后的也都不会继承。

关于可继承和不可继承的元素，常用的参考 https://segmentfault.com/a/1190000018411761

常见的可继承：

- 字体系列
  - font
  - font-family
  - font-size
  - font-weight
- 文本系列
  - text-align
  - text-indent 首行缩进
  - line-height
  - color
- 可见性
  - visibility
- 光标 cursor

常见的不可继承：

- 所有 position、display、margin、padding、border、width、height 等盒模型属性
- 所有背景相关属性
- 其他，比如 overflow、clip、filter、z-index

### 控制继承

CSS 为控制继承提供了四个特殊的通用属性值。**每个** css 属性都接收这些值。

- inherit
  设置该属性会使子元素属性和父元素相同。即 "开启**继承**".
- initial
  设置属性值和浏览器**默认**样式相同。如果浏览器默认样式中未设置且该属性是自然继承的，那么会设置为 inherit 。
- unset
  将属性重置为自然值，也就是如果属性是可继承的，并且父元素设置了该属性值，那么就是 inherit 继承父元素设置的属性值，否则就是 initial，将值改成默认值（默认值就是浏览器默认样式）

另外还有一个`all`属性可以一次更改所有的属性为这三个继承值

```css
.block {
  all: unset;
}
```

## 简写

一般来说简写的情况优先级更高；
如果 background 简写没有设置颜色但是设置了 background-color 的值是不会生效的。  
值不相同时顺序无所谓；但是相同值顺序重要，比如下面几个：

- margin/padding： 上右下左的顺序
- border-width：
  - 一个值表示四个边宽度
  - 两个值，第一个是上下宽度， 第二个是左右宽度
  - 三个值，第一个是 top， 第二个是左右， 第三个是 bottom
  - 四个值，上右下左
- border-radius：和上面一样
- border：可以把`width`/`style`/`color`写在一起， 但是 radius 不行；所以简写出现尺寸一般是指宽度
- background

> 此属性是一个 简写属性，可以在一次声明中定义一个或多个属性：background-clip、background-color、background-image、background-origin、background-position、background-repeat、background-size，和 background-attachment。对于所有简写属性，任何没有被指定的值都会被设定为它们的 初始值。

主要规则如下：

1. 在每一层中，下列的值可以出现 0 次或 1 次：

- `background-attachment`
- `background-image`
- `background-position`
- `background-size`
- `background-repeat`

2. size 只能紧接着 position 出现，以"/"分割，如： "center/80%"。注意这并不是说`/`之后的部分全都是一个全新的，只是用来连接 size 和 position 而已
3. 可以设置多层背景，用逗号隔开，每一层背景单独渲染；`background-color` 只能被包含在最后一层。

```css
background: left 5% / 15% 60% repeat-x url("star.png");

background: center / contain no-repeat url("firefox-logo.svg"), #eee 35%
    url("lizard.png");
```

## 浏览器前缀

> **实验性的或者非标准的** CSS 属性常常被添加前缀，这样开发者就可以用这些新的特性进行试验，同时（理论上）防止他们的试验代码被依赖，从而在标准化过程中破坏 web 开发者的代码。

主流浏览器引擎前缀:

- `-webkit-` （chrome，Safari，新版 Opera 浏览器，以及几乎所有 iOS 系统中的浏览器（包括 iOS 系统中的火狐浏览器）；基本上所有基于 WebKit 内核的浏览器）
- `-moz-` （火狐浏览器）
- `-o-` （旧版 Opera 浏览器）
- `-ms-` （IE 浏览器 和 Edge 浏览器）

## css 选择器

### 优先级

> 浏览器是根据优先级来决定当多个规则有不同选择器对应相同的元素的时候需要使用哪个规则。它基本上是一个衡量选择器具体选择哪些区域的尺度：

选择器大致优先级如下：

> 行内 style > id > class > type（元素类型，比如 div）

不同选择器的重叠也会有不同的样式

| 选择器               | 权重       |
| -------------------- | ---------- |
| !important           | 无穷       |
| style=""             | 1000       |
| #my-id #ur-id        | 200        |
| #my-id               | 100        |
| #my-id .my-class     | 110        |
| .my-class            | 10         |
| :hover（伪类）、属性 | 10         |
| div p                | 2          |
| p                    | 1          |
| ::before（伪元素）   | 1          |
| `*` `>` `+` `~`      | 0          |
| 继承样式             | -1（最低） |

在这个表中除了基本的权值比较，同数量级的权值大小也会影响优先级；比如权值为 110 的类和 id 选择器会比单纯的 id 选择器权值高；但是并不会越界，比如就算 100 个类选择器也不会比一个 id 优先

注意事项：

- !important 声明的样式的优先级最高；
- 如果优先级相同，则最后出现的样式生效；
- 继承得到的样式的优先级最低；
- 样式表的来源不同时，优先级顺序为：行内样式 > 内部样式 > 外部样式 > 浏览器用户自定义样式 > 浏览器默认样式。注意内联样式和外部样式比较的前提是选择器相同，内联会比外部高；如果选择器不同那就按照选择器来比较。
- 对于复合选择器(即空格、`>`、`+`、`~`这种)，越具体的选择一般权重越高，可以理解为他们是**把选择器的权重加起来**。对于同一个元素来说，选择的越具体、复合选择器的数量越多，选择器优先级就越高。

### 选择器权重

![](https://pic.imgdb.cn/item/621da1bf2ab3f51d91decb71.jpg)

### 选择器效率

从上到下选择器效率逐渐降低：
![](https://pic.imgdb.cn/item/63e72a494757feff331d82a0.jpg)

参考文章：https://www.w3cplus.com/css/css-selector-performance

复合选择器（多个选择器）也会受到单个效率的影响
比如

```css
p #my-id {
}
#my-id p {
}
```

第一个效率比第二个高，原因是**选择器的最后一部分，也就是选择器的最右边的那个被称为“关键选择器”，它将决定你的选择器的效率如何。**复合选择器基本上决定于关键选择器的效率。
同时选择器叠加越多，效率也就越低。因此选择简短的 id 或类选择器是最好的。
选择器效率优化的建议：

- 避免普遍规则，即全选选择器`*`
- 不要在 ID 选择器或类选择器前加标签名或类名，即`p.classname`或`p.idname`这种形式
- 尽可能使用具体的类别
- 避免使用后代选择器，包括全部后代`.parent .child`、直接子代`.parent > .child`以及两个兄弟选择器

### 选择器的匹配顺序

对于子选择器、后代选择器、兄弟选择器等，选择器的匹配顺序是从右向左而不是从左向右

即，对于一个选择器比如：

```css
.main p span {
}
```

匹配顺序应该是

```
先找到所有的span ---> 找到span父节点对应的p ---> 找到p父节点对应的.main
```

如果是后代选择器，那么从下向上查找的过程就会一直查找到顶端的 html 元素。而如果是直接子代的话只需要查找到其父元素即可。这也是子选择器性能优于后代选择器的原因

匹配期间不符合条件的就不会继续向上查找，比如某个 span 的父节点中没有 p 元素，那它一定就不是匹配的对象，不再查找，这条查找路径就会被放弃掉，再从下一个 span 元素开始。

之所以不是从左向右、从上向下的查找顺序，主要原因是从右向左的查找会减少很多不匹配的花费时间

如果是从上向下的回溯查找，那么大量的时间都花费在不匹配的路径上。比如先找到.main 元素，然后再从.main 元素开始，遍历每个子节点查找 p 元素，再从 p 元素开始查找 span 元素。这期间如果有一个不存在，这条路径就会直接回溯，那么之前的查找就会全部作废，相当于浪费了很多时间。

而从右向左的查找则是先找最后的元素，这样匹配的成功率就会高很多，减少了查找时间

### type 选择器（属性选择器）

比如`input[type="text"]`这样的叫做类型选择器；当然除了简单的"="还可以选择以某个开头或者以某个结尾的属性

- `img[alt^="film"]` ：alt 属性以 film 开头的元素
- `[attribute*="value"]`：属性值包含“value”的元素
  - attribute 表示属性，比如 src、alt、class 等都算属性；也可以指定属性具体，比如上面的 alt；
  - 选择元素属性后面的引号部分中有 value 字符串；
  - 可以不需要前置选择器；
- `[attribute$="value"]` ：以 value 结尾

### 易混淆选择器区分

- `ul li{}`和`ul > li{}`区别在于，前者会选择 ul 下的全部后代元素（包括孙子），但后者只会选择直接子代。注意第一种是后代，不要理解为几个元素的多选（用逗号隔开）。
- `h1 + p {}`和`h2 ~ p {}`前者是直接兄弟，后者是同级全部兄弟

### 伪类和伪元素

> 伪类和伪元素的区别：
> 伪类：其核心就是用来**选择**那些不能够被普通选择器选择的文档之外的元素，比如`:hover`。
> 伪元素：其核心就是需要**创建**通常不存在于文档中的元素，比如`::before`。

下面整理了一些容易记错效果或者混淆的伪元素和伪类。这些伪元素、伪类还是有很多使用细节的。

- `::after`：用来创建一个伪元素，作为已选中元素的最后一个**子元素**。注意是子元素，并不是在被设置的元素后面
- `::before`：和 after 同理，是第一个**子元素**
- `::first-line`：在某**块级元素**的第一行应用样式。一般来说都是应用在`<p>`元素上的，并且能设置的属性都是和字体相关的。至于位置、边距等其他属性都对其无效
- `::selection`：应用于文档中被用户高亮（选中）的部分。只有文本颜色、阴影、背景颜色、cursor 等属性才能生效

---

下面是伪类

##### `:link` — `:visited` — `:hover` — `:active` （LoVe/HAte）

这几个伪类的定义要一句一定的顺序，比如一个链接应该按照如下顺序设置伪元素：

```css
a:link {
}
a:visited {
}
a:hover,
a:focus {
}
a:active {
}
```

因为当鼠标经过未访问的链接，会同时拥有`a:link`、`a:hover`两种属性，`a:link`离它最近，所以它优先满足`a:link`，而放弃`a:hover`的重复定义。
当鼠标经过已经访问过的链接，会同时拥有`a:visited`、`a:hover`两种属性，`a:visited`离它最近，所以它优先满足`a:visited`，而放弃`a:hover`的重复定义。
因此这两者应该都在`:hover`之前；
`active`是在点击时出现，应该放在最后防止覆盖未点击的样式。

这种链接伪类先后顺序被称为 LVHA（爱恨原则，或驴哈(lvha)） 顺序：`:link` — `:visited` — `:hover` — `:active`。

##### `:active`

代表的是用户按下按键和松开按键之间的样式。除了可以用在`<a>`和`<button>`，对于设置 for 的**label 标签**，也可以让被指向的元素被选中。在 css3 中该伪类只能是鼠标左键触发的。

##### `:checked`

本身没什么特点，该元素适用于`<input>`中 type 为 radio/checkbox/select 的被选中元素。关键在于，由于可以通过 label 标签触发`:checked`，因此可以在不需要 js 的情况下就完成一定的样式转换，比如最常见的主题转换。下面是一个例子，效果就是点击 label 时 article 的背景颜色会改变：

```html
  <div id="main">
      <header>
          <label for="theme">click me</label>
      </header>
      <input type="checkbox" id="theme">
      <article>aaa</article>
    </div>
  </body>
  <style>
    html,
    body {
      margin: 0;
    }
    #theme {
        display: none;
    }
    #theme:checked + article{
        background-color: whitesmoke;
    }
    article{
        min-height: 30vh;
        width: 500px;
        background-color: black;
    }
```

##### `:first-child`

表示在一组**兄弟元素**中的第一个元素。注意并不是作用于父元素，而是**在一个父元素下的一组子元素**，并且是同级兄弟元素之间的第一个；但是也不是作用于单个子元素，而是**一组**相同选择器的子元素，比如一个 ul 下的所有 li。如果被匹配的元素不在第一位，则可能出现匹配不到任何元素的情况。

```css
ul li:first-child {
  /*...*/
}
```

##### `:first-of-type`

表示一个父元素下的一组兄弟元素的第一个该类型元素，同样是作用于子元素，表示这个类型子元素的第一个。区别在于，该伪类是**这个父元素下这类元素的第一个**，而不是所有元素的第一个。看个例子：

```html
<body>
  <ul>
      <li>a</li>
      <span>b</span>
      <li>c</li>
  </ul>
</body>
<style>
  li:first-child{
    color: red;
  }
  span:first-of-type{
    color: blue;
  }
```

![](https://pic.imgdb.cn/item/621ce2612ab3f51d910cb999.jpg)

- `first-child`选的是 ul 元素下的所有子元素的第一个，因此`span:first-child`匹配不到任何元素，因为没有一个 span 元素在所有元素的第一个
- `:first-of-type` 匹配的是某父元素下相同类型子元素中的第一个，`span:first-of-type`意思是所有 span 中的第一个

##### `:focus-within`

一个很实用的伪类，作用在一个有可以被 focus 的子元素对应的父元素上；比如一个 div 内部包含几个 input，可以让这个伪类作用在该父 div 上，**当子元素的 input 获得焦点时父元素会被选中**。

```html
<form>
  <label for="given_name">Given Name:</label>
  <input id="given_name" type="text" />
  <br />
  <label for="family_name">Family Name:</label>
  <input id="family_name" type="text" />
</form>

...

<style>
  form:focus-within {
    background: #ff8;
    color: black;
  }
</style>
```

##### `:last-child` & `:last-of-type`

选择特点和 first 相同，只是选择最后一个。

##### `:not()`

一个接受参数的选择器，表示“不为 xxx”的元素，参数是一个选择器
作用元素没有限制，可以作用在父元素上表示子元素中的非，也可以作用在一组元素上；但是一般要有相同选择器的其他元素，否则“非”就没有其他的了

```css
.fancy {
  text-shadow: 2px 2px 3px gold;
}

/* 类名不是 `.fancy` 的 <p> 元素 */
p:not(.fancy) {
  color: green;
}

/* 非 <p> 元素 */
body :not(p) {
  text-decoration: underline;
}

/* 既不是 <div> 也不是 <span> 的元素 */
body :not(div):not(span) {
  font-weight: bold;
}
```

##### `:nth-child()` & `:nth-of-type()`

> `:nth-child()`首先找到（一个父元素下）所有当前元素的兄弟元素，然后按照位置先后顺序**从 1 开始**排序，选择的结果为 CSS 伪类`:nth-child`括号中表达式`（an+b）`匹配到的元素集合（n=0，1，2，3...）

该伪类作用于一个父元素下的子元素上；
和上面的几个 child 伪类相同，可能会因为匹配的子元素不在参数所在的顺序导致不能匹配到任何值

```html
<body>
  <div id="main">
    <ul>
      <li>a</li>
      <span>b</span>
      <li>c</li>
    </ul>
  </div>
</body>
<style>
  li:nth-child(3) {
    color: red;
  }
  span:nth-child(1) {
    color: blue;
  }
</style>
```

![](https://pic.imgdb.cn/item/621ce86d2ab3f51d91187b82.jpg)

下面这个情况，`li:nth-child(3)`会匹配到第 3 个元素，并且该元素在整体的位置为第三个（并不是 li 中的第三个）；
但`span:nth-child(1)`匹配不到任何，因为没有一个 span 在第 1 位

---

借此就可以引出`:nth-of-type()`，和上面的 type 伪类相同，也是匹配同类型的参数序号元素

```css
li:nth-child(3) {
  color: red;
}
span:nth-of-type(1) {
  color: blue;
}
```

![](https://pic.imgdb.cn/item/621ce88c2ab3f51d9118b8fd.jpg)
这样就会匹配到第一个 span 元素。

---

另外，这两个伪类还可以使用表达式作为参数：

> 示例：
> `0n+3` 或简单的 3 匹配第三个元素。
> `n` 匹配每个元素，n 必须为**正整数**，负数无效
> `n+a` 匹配从第 a 个开始的元素，`n+1`表示从第一个开始，也就是全部匹配；`n+2`表示从第二个开始，以此类推
> `2n` 匹配位置为 2、4、6、8...的元素（n=0 时，2n+0=0，第 0 个元素不存在，因为是从 1 开始排序)。你可以使用关键字 `even` 来替换此表达式。
> `2n+1` 匹配位置为 1、3、5、7...的元素。你可以使用关键字 `odd` 来替换此表达式。
> `3n+4` 匹配位置为 4、7、10、13...的元素。
> `-n+b` 匹配前三个，n 前面不能有参数，否则就只是表示第 b 个
> a 和 b 都必须为整数，并且元素的第一个子元素的下标为 1。换言之就是，该伪类匹配所有下标在集合 `{ an + b; n = 0, 1, 2, ...}`中的子元素。另外需要特别注意的是，an 必须写在 b 的前面，即不能是 `b+an` 的形式。

##### `:nth-last-child()` & `:nth-last-of-type()`

和上面两个完全一致，但是是从最后一个为 1 号开始排列。

##### `:only-child` & `:only-of-type`

选择没有兄弟元素的元素，即一堆子元素中只有一个的。
前者选择的是匹配没有任何兄弟元素的元素，完全等效于`:first-child`
后者选择只有一个该类型的元素，上面几个例子中的 span 就可以用这个筛选

## 单位

css 常用单位除了 px 之外,其他常用的如下:

### 绝对单位

- px: 像素
- cm/mm/in：厘米/毫米/英寸，几乎用不到
- pt：绝对像素，`1pt = 1/72(inch)`

相关概念：

- 物理像素、设备像素（dp）：设备不能改变的、在设备一生产出来后就确定的像素
- 逻辑像素、设备独立像素（dip）：用于表示图片、视频等分辨率或尺寸大小、可时刻改变的像素称作逻辑像素；
- 设备像素比（dpr）：设备像素 / 设备独立像素，默认是 1，如果放缩页面就会变化

逻辑像素是可以改变的，逻辑像素可以等同于物理像素，比如一个逻辑像素小格子可以也是一个物理像素的小格子；但是如果将图片放大三倍，那么就相当于一个逻辑像素是九个物理像素。物理像素是不会改变的，但逻辑像素会随着放大缩小等操作变化。

逻辑像素的出现是为了解决同样大小的元素因为物理像素不同而导致肉眼看到的大小不同的。通常同一逻辑像素的实际大小在不同设备上都是相同的（比如一个 300 像素的图片，在什么样的显示器上都应该一样大）。

css 的像素单位 px 就是一个逻辑单位，但他并不是逻辑像素，而是由浏览器规定的一个虚拟单位。同时他也是一个相对单位，相对的是设备像素 dp；当网页放大缩小时，px 的具体值会发生变化。通常默认时一个 css 像素就等于一个物理像素，但不同分辨率的显示器上的 css 像素默认值就可能不相同。

css 的相对像素和物理像素的比值可以通过`window.devicePixelRato`获取（css 像素/物理像素），即 dpr。如上面所说，这个值会随页面放大缩小和显示器不同而变化。因此，`CSS像素 === 设备独立像素 === 逻辑像素`

而 css 的真正绝对单位是 pt，pt 就是物理像素的值

### 相对单位

- vh/vw: 可视窗口高/宽的 1%
- ex：相对于字符“x”的高度。通常为字体高度的一半
- em:
  - 在 `font-size` 中使用是相对于父元素中的字体的字体大小（不是父元素本身，是父元素中的字体），如果父元素没设置就是默认 16px
  - 在其他属性中使用是相对于自身内部文本元素的字体大小, 如果该元素内部没有也是默认 16px
- rem：相对于 HTML 根元素的字体大小（即`<html>`元素），可以通过手动设置 html 元素的 font-size 来修改, 没设置就是默认 16px
- `%`：用于元素是父元素宽度的百分比，用于字体是父元素字体大小的百分比
- lh：元素的`line-height`
- vmin/vmax 视窗较小尺寸的 1%/视图大尺寸的 1%。即 vh 和 vw 比较较大/较小的值，比如一般情况下`vw > vh`，因此 vmin 即 vh，vmax 即 vw；如果把网页拉成长条，就会相反。
- rpx：可以根据屏幕宽度进行自适应。规定屏幕宽为`750rpx`。
  如在 iPhone6 上，屏幕宽度为 375px，共有 750 个物理像素，则 750rpx = 375px = 750 物理像素，1rpx = 0.5px = 1 物理像素。
  rpx 为小程序中使用的相对单位，用法和 rem 类似， 1rpx = 屏幕宽度/750 px, 所以在屏幕宽度为 750 的设计稿中，1rpx = 1px，其他的按比例计算。

### 颜色单位

`hsl()` 函数接受色调、饱和度和亮度值作为参数，而不是红色、绿色和蓝色值，这些值的不同方式组合，可以区分 1670 万种颜色。他的三个属性值如下：

- 色调： 颜色的底色。这个值在 0 和 360 之间，表示色轮周围的角度。
- 饱和度： 值为 0 - 100%，其中 0 为无颜色(灰色阴影)，100%为全色饱和度
- 亮度：从 0 - 100%中获取一个值，其中 0 表示没有光，100%表示完全亮

`hsla()`增加了透明度参数，其他的和 hsl 相同

## 流和元素类型

### 文档流

文档流(Normal flow)也称为常规流，普通流。常规流指的是元素按照其在 HTML 中的位置顺序决定排布的过程
主要规则有:

- 自上而下（块级元素）,一行接一行,每一行从左至右（行内元素）。
- 内联元素不会独占一行
- 每个非浮动块级元素都独有一行, 浮动元素则按规则浮在行的一端. 若当时行容不下, 则另起新行再浮动。
- 浮动元素不占任何正常文档流空间，而浮动元素的定位照样基于正常的文档流.
- 当一个元素脱离正常文档流后，依然在文档流中的其他元素将忽略该元素并填补其原先的空间。

### 文本流

文本流，概括地说其实就是一系列字符，是文档的读取和输出顺序，就是文本的"流"; position 属性可以将元素从文本流脱离出来显示。

### 脱离文档流

元素脱离文档流, 不再占据原来的空间, 但是如果不设置位置则还会在原来的位置
脱离文档流的方法:

- 设置`position:absolute/fixed/sticky;`
  - 注意 relative 定位虽然可以让元素脱离文档流，但元素原本的位置仍占用空间
- 设置浮动

### 块级元素和内联元素

常用的块状元素有：

```
<div>
<p>
<h1>
<ol>
<ul>
<li>
<table>
<form>
```

块状元素特点:

- 一个占据一行, 宽度默认为父容器的 100%, 高度自伸展
- 默认直接换行, 即便多个 p 在一起设置宽度也会换行
- 元素有关盒模型的尺寸都可以设置；

常用的内联元素有：

```
<a>
<span>
<br>
<label>
<code>
```

内联元素特点：

- 和其他元素都在一行上；
- 元素的高度、宽度及顶部和底部边距不可设置
- 注意**左右 margin/padding**是可以设置的
- 元素的宽度就是它包含的文字或图片的宽度，不可改变。

常用的内联块状元素`inline-block`有：

```
<img>
<input>
<button>
```

inline-block 元素特点：

- 和其他元素都在一行上；
- 元素的高度、宽度、行高以及顶和底边距都可设置。

## 盒模型

![](https://www.runoob.com/images/box-model.gif)

盒模型有标准模型和 IE 模型两种，实际上 IE 模型就是下面说的`border-box`形式

![](https://pic.imgdb.cn/item/629ac9d60947543129b7884e.png)
![](https://pic.imgdb.cn/item/629ac9e30947543129b797df.png)

> outline 并不在标准盒模型中，它不占据空间，绘制于元素 border 外，会和 margin 重叠
> ![](https://pic.imgdb.cn/item/621c60a12ab3f51d911b7c5b.png)
> 下图中，外部的淡黄色是 margin 的范围，蓝色是 outline，黑色是 border，其余部分是 padding 和 content。
> 可以看到，outline 会和一部分 margin 重叠，并且包在 border 外面，box-sizing 对此没有影响

### box-sizing

- content-box 为默认盒，特点就是宽高只算 content 部分，而不算内边距和边框；

> （MDN）如果 box-sizing 为 content-box（默认），则**内容**区域的大小可明确地通过 width、min-width、max-width、height、min-height，和 max-height 控制。

- border-box 特点就宽高是算上边框和 padding 的部分；

> （MDN）如果 box-sizing 属性被设为 border-box，那么**边框**区域的大小可明确地通过 width、min-width, max-width、height、min-height，和 max-height 属性控制。

border-box 更值得全局声明，主要在于如果使用 content-box，想设置宽度就得减去 border 和 content；

举个栗子：
比如一个父元素框中一个元素占 1/4，不设置 padding 的时候没区别；
但是如果设置了 padding 或者 border，content-box 的宽度会超出 1/4，而 border-box 只会缩减内容 content 的大小，不会导致超出 1/4（有可能导致 content 的尺寸被缩减为 0 导致消失）。
当宽度不变时，border-box 看起来会比 content-box 小一些。

> 另外，`border-box`设置背景元素时（background-color 或 background-image），背景将会一直延伸至边框的外沿（默认为在边框下层延伸，边框会盖在背景上）。可通过 CSS 属性 background-clip 来改变。`content-box`只会更改 content 部分的背景

```css
.box {
  height: 5vh;
}

.box-content {
  padding: 2px;
  border: 2px;
  width: 25%;
}
.box-border {
  box-sizing: border-box;
  padding: 2px;
  border: 2px;
  width: 25%;
}
```

### margin 和 padding 的特殊情况

#### 百分比的 margin

如果定义`margin: n%`，即 margin 是百分数，则具体数值会是**其父元素宽度的`n%`**，并且即使父元素不设置明确的宽度，还是会通过计算的方式使得其值为父元素实际宽度的 n%。
padding 同理。
利用这一点，可以创造一个固定宽高比、但是可以自行收缩的盒子
即设置一个元素的高度为 0，然后用`padding-bottom: 50%`撑起来。因为这个值始终是父元素宽度的一半，因此该元素的宽度变化时，高度也会随之改变。

```css
.aa {
  width: 50vw;
}

.bb {
  background: red;
  /*子元素宽度和父元素一样，这样才能做到固定宽高比*/
  padding-bottom: 50%;
}
```

#### 外边距折叠

> 以下是概述，后面的可以不看
> 触发外边距折叠的条件：
>
> - 都是普通流中的元素且**属于同一个 BFC**
> - 没有被 padding、border、clear 或非空内容隔开（这点是针对父子元素）
> - 两个或两个以上**垂直**方向的「相邻元素」
>
> 解决外边距折叠的核心方法：
> 根据 BFC 的定义，两个元素只有在同一 BFC 内，才有可能发生垂直外边距的重叠，包括相邻元素、嵌套元素。要解决 margin 重叠问题，只要让它们**不在同一个 BFC 内就行**。
>
> - 对于相邻元素，只要给它们加上 BFC 的外壳，就能使它们的 margin 不重叠；
> - 对于嵌套元素，只要**让父级元素触发 BFC**，就能使父级 margin 和当前元素的 margin 不重叠。

外边距折叠（重合），指两个紧挨元素的外边距会重合，两者之间的距离有多种情况。
外边距折叠的情况根据两个元素的关系有所不同，主要有三种情况：

- **父子元素**边距重叠
- **兄弟元素**边距重叠
- **同一个元素的上边距和下边距**之间也会重叠

> 如果这些元素中有一个是 flex 或者 grid 布局就不会有这种问题，原因是产生了 BFC，详见后文

下面分别详述这三种情况和解决方案

##### 兄弟元素

兄弟元素在**常规文本流**的**块级元素**的**上下外边距**会重叠，其他情况都不会；
根据两个元素的外边距大小，可能有三种不同的结果：

- 全部都为正值，取最大者；
  ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2017/9/22/39c9f4ce80478c5032b9bcc18e8d4fdb~tplv-t2oaga2asx-watermark.awebp)
- 不全是正值，则都取绝对值，然后用正值的最大值减去绝对值的最大值；
  ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2017/9/21/0b52944ade824e9e5b36e4942dd8a5f0~tplv-t2oaga2asx-watermark.awebp)
- 没有正值，则都取绝对值，然后用 0 减去最大值。
  ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2017/9/21/1c46df1b01dbe05133dde944a4dd5098~tplv-t2oaga2asx-watermark.awebp)

---

兄弟元素的重叠很好解决，主要方法如下：

- 脱离文档流定位，即上面所说的脱离文档流方法（设置 position 为 fixed/static/absolute 或者设置浮动）
- 设置元素为`inline`/`inline-block`，总之避免块级元素
- 两个元素同时只设置`margin-top`或`margin-bottom`，防止一上一下产生重叠
- 设置两兄弟元素分属不同的 BFC

##### 父子元素

> 如果没有边框 border，内边距 padding，行内内容，
> 也没有创建块级格式上下文或清除浮动来分开一个块级元素的上边界 margin-top 与其内一个或多个后代块级元素的上边界 margin-top；
> 或没有边框，内边距，行内内容，高度 height，最小高度 min-height 或 最大高度 max-height 来分开一个块级元素的下边界 margin-bottom 与其内的一个或多个后代后代块元素的下边界 margin-bottom，
> 则就会出现父块元素和其内后代块元素外边界重叠，重叠部分最终会**溢出到父级块元素外面**。

比如下面这个例子，child 元素的 margin 比父元素的上下边距长，它最终的下边距会溢出父元素外边距的范围，但是子元素的外边距会被父元素折断，并没有起到“撑起”父元素的作用：

```css
.parent {
  width: 200px;
  background-color: blue;
  margin-bottom: 50px;
}
.child {
  height: 100px;
  width: 100px;
  background-color: red;
  margin-bottom: 100px;
}
```

本来期望底部的 margin 应该是 `50 + 100 = 150px`，但是实际上父元素外边距只有 50px  
父元素（实际情况）：
![](https://pic.imgdb.cn/item/621cad652ab3f51d91a367b7.png)

子元素（可以看到子元素的下边距本来要撑开到 150px 的，但是最后父元素只有自己的 50px 下边距，子元素的边距溢出了）：
![](https://pic.imgdb.cn/item/621cad652ab3f51d91a367ec.png)

解决方法就是上面造成原因的反向，即给两个元素之间设置“分隔”，即

- 设置父元素的 padding 或 border
- 设置子元素或父元素为 inline-block（inline 原理上也可以，但是会导致不能规定子元素宽高）
- 设置父元素为 BFC

##### 空块级元素

> 当一个块元素上边界 margin-top 直接贴到元素下边界 margin-bottom 时也会发生边界折叠。这种情况会发生在一个块元素完全没有设定边框 border、内边距 padding、高度 height、最小高度 min-height 、最大高度 max-height 、内容设定为 inline 或是加上 clear-fix 的时候。

```html
<div class="block"></div>
<style>
  .block {
    margin-top: 20px;
    margin-bottom: 10px;
  }
</style>
```

效果：
![](https://pic.imgdb.cn/item/621cc4402ab3f51d91ce865a.png)
可以看到实际上只有 20px 的 margin，原因是`margin-top`盖住了`margin-bottom`。
这种情况实际中很少出现，解决方法也很简单：

- 给元素设置一个高度或者填充该元素，使得元素被撑起
- 设置元素为 inline/inline-block
- 使用 clear-fix

## FC（格式化上下文）

严格来说，这个词应该叫做 visual formatting model，即 CSS 视觉格式化模型，视觉格式化模型会根据 CSS 盒子模型将文档中的元素转换为一个个盒子。该模型会根据盒子的包含块（containing block）的边界来渲染盒子。通常，盒子会创建一个包含其后代元素的包含块，但是盒子并不由包含块所限制，当盒子的布局跑到包含块的外面时称为溢出（overflow）。

- 包含块：containing block，包含其他盒子的块称为包含块。
- 盒子：box，一个抽象的概念，由 CSS 引擎根据文档中的内容所创建，主要用于文档元素的定位、布局和格式化等用途。**盒子与元素并不是一一对应的**，有时多个元素会合并生成一个盒子，有时一个元素会生成多个盒子（如匿名盒子）。盒子具体来说还分为三种：
  - 块级盒子：block-level box，由块级元素生成。**一个块级元素至少会生成一个块级盒子**，但也有可能生成多个（例如列表项元素）。块级盒子侧重于元素之间的布局关系，而下面的块容器盒子则侧重于元素内部的布局，
  - 块容器盒子：block container box 或 block containing box，块容器盒子侧重于当前盒子作为“容器”的这一角色，它不参与当前块的布局和定位，它所描述的仅仅是当前盒子与其后代之间的关系。换句话说，**块容器盒子主要用于确定其子元素的定位、布局**等。块容器盒子是创建 bfc 和 ifc 的关键，如果它只包含行内元素就会创建一个 ifc，如果包含块级元素就会生成 bfc。
  - 行内级盒子：inline-level box，由行内级元素生成。
  - 行内盒子：inline box，**参与行内格式化上下文创建的行内级盒子称为行内盒子**。与块盒子类似，行内盒子也分为具名行内盒子和匿名行内盒子（anonymous inline box）两种。

格式化上下文可以理解为是 css 自动创建的一个布局区域，遵循一些规则影响元素与元素之间和元素内的布局。

- 块级元素一定会生成至少一个块级盒子，每个块级盒子都会参与 bfc 的构建，某些块级盒子会直接创建一个 bfc。比如两个普通的 div 可能会和 html 元素组成一个 bfc，就可以说这两个 div 参与组成了一个 bfc。
- 行内级元素会生成行内级盒子，行内级盒子同时会参与 IFC 的创建。

FC 表示的是格式化上下文，每个盒子都有一种 FC 特性。FC 用于决定每个盒子之间的布局、盒子内部的元素如何布局以及一些其他相互作用。
有的 FC 表示盒子从上到下垂直排列（BFC），有的 FC 表示盒子从左到右水平排列等等（IFC）。而 IFC 则是表示盒子从左到右的水平排列方式。

CSS3 之前有 BFC 和 IFC 两种，CSS3 之后增加了 GFC 和 FFC。

### BFC

BFC 即 Block Formatting Contexts (块级格式化上下文)，它属于普通流，即元素按照从上到下按顺序排列。
具有 BFC 特性的元素可以看作是隔离了的独立容器，容器里面的元素不会在布局上影响到外面的元素，并且 BFC 具有普通容器所没有的一些特性。
通俗一点来讲，可以把 BFC 理解为一个封闭的大箱子，箱子内部的元素无论如何翻江倒海，都不会影响到外部。

#### 触发 BFC

只要元素满足下面任一条件即可触发 BFC 特性：

- html 根元素（`<html>`）
- 浮动元素：float 除 none 以外的值，即设置了浮动
- 绝对定位元素：position (absolute、fixed)
- display 为：
  - `inline-block`
  - `table-cells`
  - `flex`、`inline-flex`
  - `grid`、`inline-grid`
  - `flow-root`
- overflow 除了 visible 以外的值 (hidden、auto、scroll)
- column 相关的元素

如果触发 BFC，就会形成以下的几种特性：

#### BFC 功能和特性

MDN 上对 BFC 的完整特性如下：

> - BFC 是就像一道屏障，隔离出了 BFC 内部和外部，内部和外部区域的渲染相互之间不影响。BFC 有自己的一套内部子元素渲染的规则，不影响外部渲染，也不受外部渲染影响。
> - BFC 的区域不会和外部浮动盒子的外边距区域发生叠加。也就是说，外部任何浮动元素区域和 BFC 区域是泾渭分明的，不可能重叠。
> - BFC 在计算高度的时候，内部浮动元素的高度也要计算在内。也就是说，即使 BFC 区域内只有一个浮动元素，BFC 的高度也不会发生塌缩，高度是大于等于浮动元素的高度的。
> - HTML 结构中，当构建 BFC 区域的元素紧接着一个浮动盒子时，即是该浮动盒子的兄弟节点，BFC 区域会首先尝试在浮动盒子的**旁边**渲染，但若宽度不够（即本行由浮动元素占据的宽度剩下的部分不足以放下 BFC 元素），就在浮动元素的**下方**渲染。

##### 1. 同一个 BFC 内部的元素垂直外边距会发生折叠

这点和上面兄弟元素重叠的解决方法原理相同，一个 BFC 内的上下相邻元素的 margin 会发生折叠，但是如果放在两个不同的容器中，并且这两个容器是 BFC（通常是设置 overflow 或者 display:flex，因为不影响正常排版），就可以不导致折叠。

```html
<header class="container">
  <div></div>
</header>
<main class="container">
  <div></div>
</main>
<style>
  .container {
    overflow: hidden;
  }
  p {
    width: 100px;
    height: 100px;
    background: lightblue;
    margin: 100px;
  }
</styl>
```

##### 2. BFC 可以包含浮动的元素（父元素为 BFC）

当一个元素被设置为 BFC 时，就相当于清除了这个元素的浮动，这也是清除浮动的一种方法。

```html
<div style="border: 1px solid;">
  <div style="width: 100px;height: 100px;float: left"></div>
</div>
```

![](https://pic4.zhimg.com/80/v2-371eb702274af831df909b2c55d6a14b_720w.png)

由于内部元素浮动脱离文档流，因此父元素被压缩只有边框。
我们可以把**父元素**变成 BFC，方法同样是设置 overflow，当然其他类似方法都可以。

```html
<div style="border: 1px solid;overflow: hidden">
  <div style="width: 100px;height: 100px;float: left;"></div>
</div>
```

##### 3. BFC 可以阻止元素被浮动元素覆盖

从效果上来说，这也是清除浮动的一种，即让一个元素不会被其旁边的浮动元素盖住。
比如这样：
![](https://pic4.zhimg.com/80/v2-dd3e636d73682140bf4a781bcd6f576b_720w.png)

这张图反映出浮动元素会干扰文档流，但是并不会覆盖文本。我们要解决的就是让浮动元素不再盖住其后的元素，换句话说就是把**被盖住的**元素设置为 BFC。

```html
<div style="height: 100px;width: 100px;float: left;">我是一个左浮动的元素</div>
<div style="width: 200px; height: 200px;overflow: hidden">
  我是一个没有设置浮动的元素
</div>
```

> 另外，由上面 MDN 确定的 BFC 特点可以得知，如果该浮动元素使得宽度不够，就会把 BFC 渲染在下面，否则就是在紧挨着的旁边。

### IFC

每个一行内的内联元素组成了一行，而该行位于一个称为“行框（line box）”的矩形区域之中。该行框的大小将足以包含该行中所有的行内框（inline boxes）；当行内方向上没有剩余空间时，将会创建新行。

IFC 首先有一些关键概念：

- inline box 行内框/行内盒子：与行内格式化上下文创建的行内级盒子称为行内盒子
- line box 行框：包含 inline box 的一行的一个盒子，行框的大小足以包含该行中所有的 inline box。一个 line box 表示一行，超出的部分则会创建新的 line box

#### 创建 IFC

和 BFC 不同，IFC 的形成不需要对容器元素设置某个属性，而是在容器元素内部全部是行内元素时自动生成。换句话说，IFC 只有在一个块级元素中**仅包含内联元素**时才会生成。
比如下面的一个 div 元素，其中的每一个单词都参与一个行内格式化上下文中。每一个单词虽然没有直接包含在行内元素中，但是创建了一个匿名内联元素包裹他们，并且这三个行内元素形成了 IFC。

```html
<div class="example horizontal">One Two Three</div>

<!--下面的三个span也会形成IFC-->
<div class="example horizontal">
  <span>111</span>
  <span>222</span>
  <span>333</span>
</div>

<!--下面的不行，因为出现了块级元素-->
<div class="example horizontal">
  <div>111</div>
  <span>222</span>
  <span>333</span>
</div>
```

#### IFC 的特性

IFC 的布局规则：

1. 内部的盒子会在水平方向，一个接一个地放置。当行内方向上没有剩余空间时，将会创建新行。创建的新行并不是一个新的 IFC，而是继续生成了一行，如果设置了 border，就可以看到边框也中断掉了。

![](https://pic.imgdb.cn/item/63e9fe894757feff333f42a1.jpg)

2. line-box 的高度是由这一行中所包含的位置最高的元素的顶部和位置最低的元素的底部的距离计算出来的，其中参与计算的是元素的 line-height，元素的顶部说的是元素 line-height 的顶部，元素的底部说的是元素 line-height 的底部，当然，如果是 inline-block 的话，参与计算的是 inline block 元素的高度（height）。

也就是说，inline-box 的高度是根据子元素的实际高度计算出来的，而不是子元素的内容高度。而子元素的实际高度可以通过 line-height 来控制。
因此这里需要提一下 line-height 属性的用法，即用于确定 inline 元素的实际高度。**对块级元素设置 line-height 是没用的，但它可以继承到其内部的 inline 元素上，从而间接控制了块容器元素的高度**。

3. 摆放这些盒子的时候，它们在水平方向上的 padding、border、margin 所占用的空间都会被考虑在内，但垂直方向的不考虑。
4. 这些元素垂直方向上的定位取决于`vertical-align`，水平方向取决于`text-align`

行内元素的对齐线，也就是垂直方向上的对齐方式，可以通过 vertical-align 进行控制。属性：
![](https://pic.imgdb.cn/item/63e9ffd74757feff3349a6f1.jpg)

如图所示，默认情况下，内联元素在水平线上一个接一个排列，默认通过基线排列，也就是字符 x 的下边缘。

- 如果行内元素能在一行装下，子元素的水平排列方式由 text-align 决定。
- 如果行内元素不能在一行装下，默认此行内框会被分割，根据 white-space 决定。

IFC 的特性常常会带来一些布局问题，参考：https://mengsixing.github.io/blog/css-ifc.html#css-%E5%86%85%E8%81%94%E5%85%83%E7%B4%A0%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98

## 浮动和清除浮动

### 浮动元素

浮动的元素会脱离文档流，其本身宽高也会缩小到最小，除非设置宽度；
一个元素浮动之后，它附近的其他文档流里的元素会无视它，但这些元素内的文本会绕开这个浮动的元素，这样就可以形成一个元素“包着”这个浮动的元素，并且文本环绕的效果。

浮动元素和普通元素的位置关系如下：

1. 行内元素与浮动元素发生重叠，边框、背景、内容都会显示在浮动元素之上
2. 块级元素与浮动元素发生重叠，边框、背景会显示在浮动元素之下，内容仍会显示出来，但会出现在浮动元素的下面
3. 若不浮动的是块级元素，那么浮动的元素将显示在其上方
4. 若不浮动的是行内元素或者行内块元素，那么浮动的元素不会覆盖它，而是将其挤到边上，比如左浮动会把元素挤到浮动元素的右边

---

但是，如果一个元素内的元素都浮动了，这个元素内部会被视作“没有内容”，高度就可能变为 0。这时候就应该清除浮动；但是清除的不是浮动元素的浮动，而是再设置一个元素把它清除掉，因为清除浮动的意思是“不允许某个元素旁边有浮动元素”。

清除浮动的方法有几种，列在下面：

### 1. 单使用`clear`

> MDN:
> clear 属性适用于浮动和非浮动元素。
> 当应用于非浮动块时，它将非浮动块移动到所有相关浮动元素外边界的下方。这个非浮动块的垂直外边距会折叠。另一方面，两个浮动元素的垂直外边距将不会折叠。
> 当应用于浮动元素时，它将元素的外边界移动到所有相关的浮动元素外边框边界的下方。后面的浮动元素的位置无法高于它之前的元素。

值可以是 left、right 和 both 或者 none。这里的左和右指的是`清除哪边的浮动`，也就是说该元素不希望旁边有浮动元素。根据上面 MDN 的描述，非浮动元素会被置于浮动元素的下方。

如果单看字面意思，`clear:left` 是“清除左浮动”，`clear:right` 是“清除右浮动”，实际上，这种解释是有问题的，因为浮动一直还在，并没有清除。

官方对 clear 属性解释：“元素盒子的边不能和前面的浮动元素相邻”，对元素设置 clear 属性是为了**避免浮动元素对该元素的影响**，而不是清除掉浮动。

注意清除浮动实际上是清除该元素前面的浮动。这里的前面指的是 HTML 元素的代码顺序。使用 clear 清除浮动不会管后面的元素。

还是上面讲 bfc 的例子，设置了清除浮动后是这样的：

```html
<div style="height: 100px;width: 100px;float: left;">我是一个左浮动的元素</div>
<div style="width: 200px; height: 200px;clear:left">
  我是一个没有设置浮动的元素
</div>
```

![](https://pic.imgdb.cn/item/621cd0382ab3f51d91e785e7.png)

### 2. 设置一个空元素并清除（也可以用伪元素）

假设一个父元素由于内部元素浮动别压缩了，解决的方法就是在这个**父元素内部的所有子元素最后**插入一个空元素并清除

比如有这样一个结构：

```html
<main class="top-div">
  <div class="text-div">...</div>
  <div class="float-div">float left</div>
</main>
<footer class="bottom-div">...</footer>
```

![](https://pic.imgdb.cn/item/621cd2b82ab3f51d91ecb40a.jpg)
可以看到很明显的，父元素 top-div 被压缩了，只有其中的 text-div 的大小。

解决方法就是在 float-div 后面再加上一个空元素然后清除它

```html
<main class="top-div">
  <div class="text-div">...</div>
  <div class="float-div">float left</div>
  <div style="clear: both;"></div>
</main>
<footer class="bottom-div">...</footer>
```

当然也可以用伪元素来实现，即使用`::after`

> CSS 伪元素`::after`用来创建一个伪元素，作为已选中元素的最后一个子元素。通常会配合 content 属性来为该元素添加装饰内容。这个虚拟元素默认是**行内元素**。

即添加这样一个样式。注意由于 after 添加的是行内元素，因此需要设置 display 为非行内元素。

```css
.top-div::after {
  content: "";
  clear: both;
  display: block;
}
```

无论是上面的哪一种，原理都和第一种相同，就是清除这个元素浮动时，会把该元素置于所有浮动元素底部，也就是变相撑大了父元素。这也是该元素一定要在最后面的原因。

### 3. 设置 BFC

BFC 的原理上面已经讲过。
可以利用 BFC 的特点之一：`计算高度的时候，内部浮动元素的高度也要计算在内。`；因此只要设置父元素为 BFC，就可以使内部无论多少浮动都可以撑起来父元素。

## 定位

### 基本定位

position 有以下属性值：

| 属性值   | 概述                                                                                                                                                                           |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| absolute | 生成绝对定位的元素，浏览器会递归查找该元素的所有父元素，如果找到一个设置了 `position:relative/absolute/fixed` 的元素，就以该元素为基准定位，如果没找到，就以浏览器边界定位。   |
| relative | 生成相对定位的元素，相对于其原来的位置进行定位。和其他元素没关系，也不会影响其他元素，有可能会盖住周围的元素                                                                   |
| fixed    | 生成绝对定位的元素，指定元素相对于屏幕视⼝（viewport）的位置来指定元素位置。元素的位置在屏幕滚动时不会改变，⽐如回到顶部的按钮⼀般都是⽤此定位⽅式，会导致其他元素位置的变化。 |
| static   | 默认值，没有定位，元素出现在正常的文档流中，会忽略 top, bottom, left, right 或者 z-index 声明，块级元素从上往下纵向排布，⾏级元素从左向右排列                                  |
| inherit  | 规定从父元素继承 position 属性的值                                                                                                                                             |
| sticky   | 需要元素定义 top/right/bottom/left 之后才能生效。常见的固定导航栏就可以通过设置 nav 元素为`top:0;`                                                                             |

> MDN sticky：
> 粘性定位可以被认为是相对定位和固定定位的混合。元素在跨越特定阈值前为相对定位，之后为固定定位。例如：
>
> ```css
> #one {
>   position: sticky;
>   top: 10px;
> }
> ```
>
> 在 viewport 视口**滚动到元素 top 距离小于 10px 之前**，元素为相对定位。
> 之后，元素**将固定在与顶部距离 10px 的位置**，直到 viewport 视口回滚到阈值以下。
> 须指定 top, right, bottom 或 left 四个阈值其中之一，才可使粘性定位生效。否则其行为与相对定位相同。
> sticky 基于最近的滚动容器定位。所谓滚动容器就是指设定了`overflow:scroll/auto`的元素，sticky 基于最近的组件滚动容器定位。如果没有设置过，那就是 html 元素。

### 负 margin

对于想要 absolute 的元素可以通过负 margin 调整位置

- 左上的负 margin 可以把元素向左上拉，并且盖住其他元素
- 右下的负 margin 可以把其他元素向左上拉，并且可以盖住该元素

### 利用位置自定大小

元素的宽高有时候可以不用特别指定，可以设置位置和距离，让元素自己确定宽高
比如想在图片中下方加一个标题，可以不用指定宽度，只需要指定左右下的距离让元素自己确定宽度

```css
.header {
  position: relative;
  width: auto;
  bottom: 5em;
  left: 5em;
  right: 5em;
}
```

## 元素层叠顺序

![](https://pic.imgdb.cn/item/622452e85baa1a80ab25a1f7.jpg)

由上到下分别是：

1. 背景和边框：建立当前层叠上下文元素的背景和边框。
2. 负的 z-index：当前层叠上下文中，z-index 属性值为负的元素。
3. 块级盒：文档流内非行内级非定位后代元素。
4. 浮动盒：非定位浮动元素。
5. 行内盒：文档流内行内级非定位后代元素。
6. `z-index:0`：层叠级数为 0 的定位元素。
7. 正 z-index：z-index 属性值为正的定位元素。

注意: 当定位元素 z-index:auto，生成盒在当前层叠上下文中的层级为 0，不会建立新的层叠上下文，除非是根元素。

## display

display 属性主要如下：

| 值           | 属性                                                                                                                        |
| :----------- | --------------------------------------------------------------------------------------------------------------------------- |
| none         | 隐藏元素，渲染时完全不渲染，不会响应绑定事件                                                                                |
| block        | 块级元素，前后会带有换行符。                                                                                                |
| inline       | 内联元素，元素前后没有换行符，不能设置宽高，宽外边距不会折叠                                                                |
| inline-block | 默认为行内排列，但是可以设置宽高                                                                                            |
| list-item    | 会作为列表显示，被设置的元素前面会加上类似`<li>`元素的点，在外部表现上类似 block                                            |
| content      | 设置了`display: contents`的元素本身不会被渲染，但是**其子元素能够正常被渲染**。作用于不能有子元素的元素相当于`display:none` |
| flow-root    | 被声明的元素会变成 BFC，不会给元素带来任何额外的副作用                                                                      |

## 元素的隐藏

| 值                    | 效果                                                                                                     |
| --------------------- | -------------------------------------------------------------------------------------------------------- |
| display: none         | 渲染树不会包含该渲染对象，因此该元素不会在页面中占据位置，也不会响应绑定的监听事件。                     |
| visibility: hidden    | 元素在页面中仍占据空间，但是不会响应绑定的监听事件。                                                     |
| opacity: 0            | 将元素的透明度设置为 0，以此来实现元素的隐藏。元素在页面中仍然占据空间，并且能够响应元素绑定的监听事件。 |
| position: absolute    | 通过使用绝对定位将元素移除可视区域内，以此来实现元素的隐藏。                                             |
| z-index: 负值         | 来使其他元素遮盖住该元素，以此来实现隐藏。                                                               |
| clip/clip-path        | 使用元素裁剪的方法来实现元素的隐藏，这种方法下，元素仍在页面中占据位置，但是不会响应绑定的监听事件。     |
| transform: scale(0,0) | 将元素缩放为 0，来实现元素的隐藏。这种方法下，元素仍在页面中占据位置，但是不会响应绑定的监听事件。       |

display:none 与 visibility:hidden 的区别：

1. 在渲染树中

- display:none 会让元素完全从渲染树中消失，渲染时不会占据任何空间；
- visibility:hidden 不会让元素从渲染树中消失，渲染的元素还会占据相应的空间，只是内容不可见。

2. 是否是继承属性

- display:none 是非继承属性，子孙节点会随着父节点从渲染树消失，通过修改子孙节点的属性也无法显示；
- visibility:hidden 是继承属性，子孙节点消失是由于继承了 hidden，通过设置 visibility:visible 可以让子孙节点显示；

3. 修改常规文档流中元素的 display 通常会造成文档的重排，但是修改 visibility 属性只会造成本元素的重绘；
4. 如果使用读屏器，设置为 display:none 的内容不会被读取，设置为 visibility:hidden 的内容会被读取。

## 多栏设计

column 属性可以把元素中**文本**分为多列, 类似 grid 布局的分块, 常用相关属性有:

| 属性                          | 描述                       |
| ----------------------------- | -------------------------- |
| column-count                  | 指定元素应该被分割的列数。 |
| column-fill                   | 指定如何填充列             |
| column-gap                    | 指定列与列之间的间隙       |
| column-rule:style color width | 指定两列间边框的颜色       |
| column-span                   | 指定元素要跨越多少列       |
| column-width                  | 指定列的宽度               |
| columns: width count;         | 分别设置宽度和数量         |

常见用法: 设置分割列数和分割线; 对于设置了 column 的父元素, 其内部都会受到 column 影响; 可以通过设置 column-span 设置跨越行数

```css
.block {
  width: 80vw;
  text-align: justify;
  columns: 4 20em;
  column-gap: 2em;
  column-rule: 1px solid rgb(206, 206, 206);
}
```

另外, 借助分割线可以达到水平分割线的效果, 比如这样, 就可以创建一个 0.1em 厚的分割线, columns 等于元素个数即可

```css
.block {
  columns: 6;
  column-rule: 0.1em solid rgb(206, 206, 206);
}
```

如果只规定 width 而不限制宽度则可以达到根据页面自动改变栏数的效果
如果同时设置 count 和 width，则前者是最大栏数，后者是最小宽度

```css
main {
  columns: 20em 4; /*最多4栏，宽度为20em*/
  column-gap: 2em;
}
```

## 防混淆

- transform: 转换，后面一般跟着 translate(平移)/rotate(旋转)等几何变化
- translate: 平移，必须跟在 transform 后面
- transition: 过渡，定义在主元素, 效果在伪元素;其中伪元素的效果可以有 transform

## css 渲染原理（简单）

基本过程：

1. 首先 css 被解析时会像 dom 一样被构建 CSSOM，这是一个 css 可操作模型；CSSOM 和 DOM 可能会同时建立，主要取决于 style 元素的位置；
2. 然后当 DOM 建立后，进行样式规则匹配，通过标签名、选择器等为元素匹配样式
3. 进行布局计算；布局计算是一个递归的过程，因为一个节点的大小通常需要先计算它的子节点的位置、大小等信息。
   1. layout 函数会判断当前节点是否需要重新计算，通常依据检查数组中相应的标记位、子节点是否需要计算布局来确定
   2. 确定网页的宽度和垂直方向上的外边距
   3. 遍历当前节点的每一个子节点，依次计算它们的布局；每一个子元素也同样的计算（递归）
   4. 节点根据它的子节点的大小计算得出自己的尺寸，整个过程结束。
4. 如果发生变化，就需要重新计算布局，触发条件有以下几个：
   - 首次被打开，浏览器设置网页的可视区域（viewport），并调用计算布局的方法。就是说当可视区域发生变化的时候，Webkit 需要重新计算布局；
   - 网页的动画
   - JavaScript 通过 CSSOM 直接修改样式信息
   - 用户的交互，例如滚动网页。

## css 中的重绘和重排

简单地总结下两者的概念：

- 重排：无论通过什么方式影响了元素的几何信息(元素在视口内的位置和尺寸大小)，浏览器需要重新计算元素在视口内的几何属性，这个过程叫做重排。
- 重绘：通过构造渲染树和重排（回流）阶段，我们知道了哪些节点是可见的，以及可见节点的样式和具体的几何信息(元素在视口内的位置和尺寸大小)，接下来就可以将渲染树的每个节点都转换为屏幕上的实际像素，这个阶段就叫做重绘。

如何减少重排和重绘？

- 最小化重绘和重排，比如样式集中改变，使用添加新样式类名 `.class` 或 `cssText` 。
- 批量操作 DOM，比如读取某元素 `offsetWidth` 属性存到一个临时变量，再去使用，而不是频繁使用这个计算属性；又比如利用 `document.createDocumentFragment()` 来添加要被添加的节点，处理完之后再插入到实际 DOM 中。
- 使用 **absolute** 或 **fixed** 使元素脱离文档流，这在制作复杂的动画时对性能的影响比较明显。
- 开启 GPU 加速，利用 css 属性 `transform` 、`will-change` 等，比如改变元素位置，我们使用 translate 会比使用绝对定位改变其 left 、top 等来的高效，因为它不会触发重排或重绘，transform 使浏览器为元素创建⼀个 GPU 图层，这使得动画元素在一个独立的层中进行渲染。当元素的内容没有发生改变，就没有必要进行重绘。

更详细参考：https://juejin.cn/post/6844903779700047885

## css 变量

css 可以像 scss 等预处理工具一样，声明变量并使用。
声明变量的方法是在`:root`伪元素中通过`--xxxx`的形式声明。`:root` 表示 `<html>` 元素，即最顶层的元素。

```css
:root {
  --main-color: hotpink;
  --pane-padding: 5px 42px;
}
```

使用时需要通过`var()`函数引用变量：

```css
#main {
  background-color: var(--first-color);
}
```

变量的声明只能有一次，在`:root`中。但是后续可以更改；在任何一个元素下重新给变量赋值，都会更改该元素以及该元素的所有后代元素的变量的值，但不会影响同级元素的值。

```css
:root {
  --first-color: #488cff;
  --second-color: #ffff8c;
}
.container {
  --second-color: red;
  border: 1px solid var(--second-color); /*自己也会改变*/
}
.container p {
  color: var(--second-color); /*后代元素会变*/
}
```

通过这个特点，可以实现简单的主题切换。即在`document.body`上添加删除一个类，里边修改这些变量值即可。

并且，css 变量相对于 less、sass 等的变量，其更方便在 js 中修改：

```js
// 设置变量
document
  .getElementsByClassName("container")
  .style.setPropertyValue("--color", "pink");
// 读取变量
doucment
  .getElementsByClassName("container")
  .style.getPropertyValue("--color")
  .trim(); //pink
// 删除变量
document.getElementsByClassName("container").style.removeProperty("--color");
```

# 常用属性

## 设置滚动条样式

通过`::-webkit-scrollbar`可以改变滚动条样式

```css
.test::-webkit-scrollbar {
  /*滚动条整体样式*/
  width: 10px;
  height: 1px;
}
.test::-webkit-scrollbar-thumb {
  /*滚动条里面小方块*/
  border-radius: 10px;
  background-color: skyblue;
}
.test::-webkit-scrollbar-track {
  /*滚动条里面轨道*/
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
  background: #ededed;
  border-radius: 10px;
}
```

或者可以隐藏滚动条

```css
.demo::-webkit-scrollbar {
  display: none; /* Chrome Safari */
}
```

## 背景

背景应用到的主要属性：

### `background-image`

设置图片 url

> 也可以有多个背景图像——在单个属性值中指定多个 background-image 值，用逗号分隔每个值。
>
> ```css
> background-image: url(image1.png), url(image2.png), url(image3.png),
>   url(image1.png);
> ```

### `background-repeat`

对于尺寸不足的图片会有很大影响
在简写中，先水平（repeat-x）后垂直（repeat-y）

### `background-position`

背景的起始位置，默认的背景位置值是(0,0)，即从左上角开始。根据值类型的不同会有不同的效果：

- 如果为 top/right/center/left/bottom，则为紧挨着 上/右/中间/左/下 边框
- 如果为 50px 这种数值单位，则类似于 margin、padding 的表示方法，表示为离四个边的距离，第一个取值为背景图像与其容器在水平方向上的距离，第二个取值为背景图像与其容器在垂直方向上的距离（即`background-position-x`和`background-position-y`的简写）
- 如果是百分比，则是图片的某个点和页面某个点对齐，百分比表示这两个点在自己内部的位置占比；
  - 比如 0% 0%是图片左上角和页面左上角对齐
  - `25% 75%` 表示图像上的左侧 25% 和顶部 75% 的位置将放置在距容器左侧 25% 和距容器顶部 75% 的容器位置
  - 50% 50%表示图片中间和页面中间对齐
  - 若只有一个取值，则其第二个取值默认为 50%

> `background-position`是`background-position-x`和`background-position-y`的简写。  
> 上述单位可以混用，但是可以使用 4-value 语法来指示到盒子的某些边的具体距离，比如这样：
>
> ```css
> .box {
>   background-image: url(star.png);
>   background-repeat: no-repeat;
>   background-position: top 20px right 10px; /*距上20px，距右10px*/
> }
> ```

### `background-attachment`

它可以接受以下值:

- scroll: 使元素的背景在页面滚动时滚动。滚动元素内容背景不会移动。注意它是在**页面**滚动时才会滚动；如果有一个元素 A 内部设置了`overflow:scroll`，那么在这个元素 A 上设置的背景不会随着 A 内部滚动而滚动，只会跟随整个页面滚动而滚动。
- fixed: 使元素的背景固定在**视图**上，这样当页面或元素内容滚动时，它就不会滚动。它将始终保持在屏幕上相同的位置，即使是页面滚动也不会移动。
- local: 这个值是后来添加的，将背景固定在设置的元素上，因此当滚动元素时，背景也随之滚动。

可以参考 MDN 提供的一个 demo： https://mdn.github.io/learning-area/css/styling-boxes/backgrounds/background-attachment.html

### `background-size`

确定背景“响应式”的主要属性；

- 设置为固定尺寸则为固定尺寸，可以设置两个值分别为宽和高

  - 按照响应式图片的原理，设置` background-size:100% auto`可以让宽度始终 100%，并且不会改变原始比例；但如果缩小页面，上下会出现空白

- contain：保证比例的情况下尽可能自动布局
- cover：高度超过尺寸时充满左右；宽度超过时充满上下，即尽可能充满区域

## 表单

- label 标签有一个属性`for`可以绑定 input，for 后面跟上 input 的 name 属性，点击 label 就可以聚焦到 input 上；顺便可以加一个`cursor:pointer`。这一点用在 checkbox 上很好用
- fieldset 和 legend：
  - fieldset 可以在表单周围形成一个边框；
  - legend 在 fieldset 的边框偏左上方部分会嵌入一段文字，可以表示表单名等
    但是并不好用，除非是创建文字在边框上的形式，否则边框就很好用了
- input 要设置`font: inherit`，目的是和外部字体统一，否则看起来会小一些
- input:focus 会有一个浏览器默认黑框；这个要通过覆写 outline 去除，注意不是 border
- 设置 placeholder 的字体样式通过`::-webkit-input-placeholder`伪类

```css
input::-webkit-input-placeholder {
  /* Chrome/Opera/Safari */
  color: red;
}
```

## 文本

### 文本布局

`text-align` 属性用来控制文本如何和它所在的内容盒子对齐：

- left: 左对齐文本。
- right: 右对齐文本。
- center: 居中文字
- justify: 可以设置每一行被展开为宽度相等，左，右外边距是对齐；如果不设置这一条可能会出现左右有“突出”的字符

---

`line-height`属性设置文本每行之间的高，可以接受大多数单位。

当值没有单位时作为乘数，相当于`font-size`的倍数，比如 1.5 就是字体大小的 1.5 倍。这是官方推荐的方式，并且建议的 line-height 最好为 1.5 以上

当值为百分比时给定的百分比值乘以元素计算出的字体大小

```
line-height: 1.5;
```

---

其他的一些属性：

- `text-indent:`首行文本缩进
- `letter-spacing`设置字符间距
- `white-space`，本来是用来设置处理元素空白的，但是可以通过设置`white-space: nowrap;`让文字不换行

### 文字超出部分显示省略号

单行文本的溢出显示省略号（一定要有宽度）

```css
p {
  width: 200px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap; /*规定段落中的文本不进行换行*/
}
```

多行文本溢出显示省略号

```css
p {
  display: -webkit-box; /*作为弹性伸缩盒子模型显示。*/
  -webkit-box-orient: vertical; /*设置伸缩盒子的子元素排列方式：从上到下垂直排列*/
  -webkit-line-clamp: 3; /*显示的行数*/
  overflow: hidden;
}
```

## 盒阴影

box-shadow 属性确定盒阴影, 其后属性主要有:

- h-shadow: 水平阴影的位置。允许负值；默认是下方，负值就是上方
- v-shadow: 垂直阴影的位置。允许负值；默认在右边，负值则在左边
- blur: 模糊距离
- spread: 阴影的尺寸
- color: 阴影的颜色
- inset: 将外部阴影 (outset) 改为内部阴影

```css
.block {
  box-shadow: h-shadow v-shadow blur spread color inset;
}
```

## 光标

通过设置 cursor 属性可以改变鼠标移上去的光标

- `cursor:auto`: 文本选择
- `cursor:crosshair`: 十字
- `cursor:default`: 箭头
- `cursor:e-resize`: 水平改变方向箭头
- `cursor:help`: 问号
- `cursor:move`: 十字移动箭头
- `cursor:n-resize`: 上下改变方向箭头
- `cursor:pointer`: 点击(常用)
- `cursor:progress`: 半加载中
- `cursor:text`: 文本
- `cursor:wait`: 完全加载中
- `cursor: not-allowed`: 禁用

## 渐变

- 线性渐变（Linear Gradients）- 向下/向上/向左/向右/对角方向
  - 默认从上向下, 可以指定方向,也可以指定角度
  ```css
  #grad1 {
    height: 200px;
    background-image: linear-gradient(#e66465, #9198e5);
  }
  #grad2 {
    height: 200px;
    background: linear-gradient(to bottom right, red, blue);
  }
  ```
- 径向渐变（Radial Gradients）- 由中心向四周
  ```css
  #grad {
    background-image: radial-gradient(red, yellow, green);
  }
  ```

## 转换&过渡

transition 方法实现需要触发一个事件（比如鼠标移动上去，焦点，点击等）才执行动画。它是一种关键帧动画，设置一个开始关键帧，一个结束关键帧。

```css
.block {
  background: linear-gradient(red, blue);
  width: 20em;
  height: 20em;
  transition: width 1s ease-out, height 1s, transform 2s;
}
.block:hover {
  width: 30em;
  height: 10em;
  transform: translate(0, 5em);
}
```

transition 方法需要在伪类指定变化的时刻或者通过 js 添加新类。也就是说，单独添加 transition 并不能产生作用，因为只定义了一个原效果，还需要添加一个伪类来表示“变化之后的效果”。
比如我们希望一个块，当鼠标 hover 时可以变颜色：

```css
.block {
  background-color: red;
  transition: background-color 2s ease-out;
}
.block:hover {
  background-color: blue;
}
```

要转换的状态写在伪类的样式里。
另外，还可以通过 js 向该元素**添加**一个类的形式实现，即使用`block.classList.add()`方法添加新的转化类，从而实现样式的渐变。

```html
<style>
  .block {
    background-color: red;
    transition: background-color 2s ease-out;
  }
  .block-tran {
    background-color: blue;
  }
</style>
<script>
  const block = document.getElementsByClassName("block")[0];
  const btn = document.getElementsByTagName("button")[0];
  btn.addEventListener("click", () => {
    block.classList.add("block-tran");
  });
</script>
```

transition 可操作的属性有：

- 宽高
- transform 变化， 包括 2d 和 3d 的
  - translate 平移， 两个参数是 x/y 轴平移量
  - rotate 旋转， 参数是角度， 用 deg 为单位； 还有 rotateX 和 rotateY 设置沿 x/y 轴转动
  - scale 缩放， 参数分别指定 x/y 轴缩放
  - skew 错切， 参数分别指定 x/y 轴错切角度
  - matrix 就是图形学学到的变化矩阵的写法
- 背景
- 文本
- 边框
- margin/padding
- position
- 以及几乎所有属性

transition 可设置的属性还有：

- transition-delay 开始的延迟
- transition-duration 持续时间
- transition-property 过渡属性，就是宽高等属性
- transition-timing-function 过渡曲线函数
  - linear 匀速
  - ease 慢速开始，然后变快，然后慢速结束
  - ease-in 慢速开始
  - ease-out 慢速结束
  - ease-in-out 慢速开始和结束

简写：`transition： <property> <duration> <timing-function> <delay>`，空格隔开； 多个变换之间用逗号隔开

## 动画

使用`@keyframes` 规则定义关键帧

1. 首先在一个元素中声明 animation 属性

- `name`：名称, 用于下面 keyframes 的选择
- `duration`：持续时间，注意要在数字后面加`s`或者`ms`，单纯数字不行
- `timing-function`：动画时间函数, 同过渡，（默认为`ease`，即动画以低速开始，然后加快，在结束前变慢）
- `delay`：延迟
- `iteration-count`：动画播放次数, 默认为 1，**`infinite`为无限循环**
- `direction`：规定是否在下一周期反向播放,
  - `reverse`: 反向
  - `alternate`: 奇数次正向, 偶数次反向
  - `alternate-reverse`: 上面的反之
- `fill-mode` ：当动画不播放时（当动画完成时，或当动画有一个延迟未开始播放时），要应用到元素的样式。这个样式的具体规定就是原先的部分，比如：

  ```css
  .block{
      top:0
      animation: myAnimation 5s linear 0s 1 reverse forwards;
  }
  @keyframes myAnimation{
      from:{top:0}
      to:{top:100px}
  }
  ```

  这个例子中动画结束播放会返回到在.block 中定义的`top:0`

  - forwards：动画结束应用
  - backwards：动画启动应用
  - both：上面两个结合

2. 在`@keyframes` 后加上动画名, 定义动画效果

- 可以用 from/to 指定开始和结束的关键帧, 在内部写变换样式
- 可以用百分比, 可以任意指定, 但是至少要有 0% 和 100%
- 关键帧内的样式和过渡的相同, 同样可以设置宽高/位置/变换等
- 注意如果要在百分比中设置 transform, 只在一个百分比设置即可;一般只在 100%中设置

示例：

```css
.block {
  animation: name duration timing-function delay iteration-count direction
    fill-mode;
}
/*示例*/
.block {
  animation: myAnimation 5s linear 0 infinite reverse;
}
@keyframes myAnimation {
  0% {
    background: red;
    left: 0px;
    top: 0px;
  }
  25% {
    background: yellow;
    left: 200px;
    top: 0px;
  }
  50%,
  70% {
    /*表示从50%到70%之间不变*/
    background: blue;
    left: 200px;
    top: 200px;
  }
  75% {
    background: green;
    left: 0px;
    top: 200px;
  }
  100% {
    background: red;
    left: 0px;
    top: 0px;
    transform: rotate(360deg);
  }
}
```

## 图片相关

### 响应式图片

不在 css 中设定图片宽高, 把默认宽高放在 html 中的 img 元素属性上, 然后在 css 中设置 `max-width: 100%` 和 `height:auto`
当页面缩小, 图片先会不改变位置，直到充满页面, 随着再缩小窗口会按比例缩小
这个的核心是`max-width: 100%;`表示图片会保留其宽高比，但是在容器变大时，又不会超过最大尺寸

```css
img {
  max-width: 100%;
  height: auto;
}
```

主要原理就是限制图片最大不会超过自己的尺寸，但是因为是"max"，所以会自动缩小。如果设置`width:100%`，图片会被强制拉伸充满整个父容器，也就破坏了响应式。
同时可以设置 height:auto 来保证宽高比一致

响应式图片还可以通过 html 元素属性的 srcset 和 picture 元素设定：（更详细的参考<a href="https://wangdoc.com/html/image.html">网道-图像标签</a>）

### srcset

srcset 属性用来指定多张图像，适应不同像素密度的屏幕。它的值是一个逗号分隔的字符串，每个部分都是一张图像的 URL，后面接一个空格，然后是像素密度的描述符。

> 像素密度
> 前面说到 css 单位时提到的逻辑像素 px 和绝对像素 pt 的关系，就可以用像素密度表示
>
> 如果 1px>1pt，比如 1px 可以表示 2pt，就对应大屏幕的情况，像素密度 ke 能>1，越大表示屏幕分辨率越高
>
> 反之如果 1px<1pt 就对应小屏幕的情况，像素密度可能<1，越小表示屏幕分辨率越低

```html
<img
  src="scones_small.jpg"
  srcset="scones_medium.jpg 1.5x, scones_large.jpg 2x"
/>
```

这个例子里边既有 src 也有 srcset；srcset 其实就是一种另类的 src，但是 src 还要留着，保证像素密度都不匹配时的默认图像。

srcset 格式：

```html
srcset="./path/any.jpg 1.5x"
```

图片路径和后面的用空格隔开，后半部分的格式是`像素密度倍数 + 字母 x` 。1x 表示单倍像素密度，可以省略。浏览器根据当前设备的像素密度，选择需要加载的图像。因为浏览器是在一开始的页面生成就调整好图片，所以如果想调试可以先改变页面宽度再刷新。

### sizes

像素密度的适配，只适合显示**区域一样大小**的图像。如果希望不同尺寸的屏幕，显示不同大小的图像，srcset 属性就不够用了，必须搭配 sizes 属性。

```html
<img
  srcset="160.jpg 160w, 320.jpg 320w, 640.jpg 640w, 1280.jpg 1280w"
  sizes="(max-width: 440px) 100vw,
         (max-width: 900px) 33vw,
         254px"
  src="foo-1280.jpg"
/>
```

- srcset 属性列出四张可用的图像，每张图像的 URL 后面是一个空格，再加上宽度描述符。宽度描述符就是图像原始的宽度（单位 px），加上字符 w，**必须与引用图像的固有宽度相匹配**。上例的四种图片的原始宽度分别为 160 像素、320 像素、640 像素和 1280 像素。
- sizes 属性的值是一个逗号分隔的字符串，每个部分都是一个放在括号里面的媒体查询表达式，后面是一个空格，再加上图像的显示宽度。
  宽度不超过 440 像素的设备，图像显示宽度为 100vw；
  宽度 441 像素到 900 像素的设备，图像显示宽度为 33vw；
  宽度 900 像素以上的设备，图像显示宽度为 254px。

然后是浏览器加载图片的步骤：

1. 首先获取页面宽度（比如是 320px）
2. 根据 size 中符合条件的`max-width: 440px`确定显示宽度为 100vw
3. 根据这个 100vw 确定最 srcset 中最接近的图片；这里因为是 320px 的页面宽度，因此会选择 320w（100vw 正好是 320px） 的图片

### picture 和 source

`<picture>`元素类似 video/audio 的父元素
`<picture>`内部的`<source>`标签，主要使用 media 属性和 srcset 属性。media 属性给出媒体查询表达式(同 css 中的媒体查询，要带括号)，srcset 属性就是`<img>`标签的 srcset 属性，给出加载的图像文件。sizes 属性其实这里也可以用，但由于有了 media 属性，就没有必要了。
浏览器按照`<source>`标签出现的顺序，依次判断当前设备是否满足 media 属性的媒体查询表达式，如果满足就加载 srcset 属性指定的图片文件，并且不再执行后面的`<source>`标签和`<img>`标签。
`<img>`标签是默认情况下加载的图像，用来满足上面所有`<source>`都不匹配的情况，或者不支持`<picture>`的老式浏览器。

```html
<picture>
  <source media="(min-width: 30em)" srcset="cake-table.jpg" />
  <source media="(min-width: 60em)" srcset="cake-shop.jpg" />
  <img src="xxx.jpg" />
</picture>
```

### object-fit

用来设置图片在容器的布局方式，**注意要生效需要指定图片的 width 和 height**，最好是这样：

```css
img {
  height: 100%;
  width: 100%;
}
```

- contain：保持宽高，但是会有空白
- cover：保持宽高，但是会剪裁图片
  这两个属性的变化取决于图片宽高比和容器；如果图片本身宽高比和容器相同会有很好表现；
  **可以通过设置 `height:auto`、width 固定，或 `width:auto`、height 固定 来形成保持宽高比的元素**
  ```css
  .img-contain {
    width: auto;
    height: 200px;
  }
  ```
- fill: 拉伸填充
- none
- scale-down： none 或 contain ，取决于它们两个之间对象尺寸会更小一些的那个

---

object-fit 属性不仅可用于影响图片在容器的位置，实际上还可以控制所有的**替换元素**。所谓替换元素，就是指本身有一定样式的，css 无法操控其内部样式，但是可以更改其位置、大小等。
典型的可替换元素有：

- `<iframe>`
- `<video>`
- `<embed>`
- `<img>`

有些元素仅在特定情况下被作为可替换元素处理，例如：

- `<option>`
- `<audio>`
- `<canvas>`
- `<object>`

除了`object-fit`，`object-position`指定可替换元素的内容对象在元素盒区域中的位置，使用上类似于`background-position`。

### 图片滤镜

使用 filter 给图片加滤镜, 常见的的有模糊/黑白等效果

```css
img {
  filter: blur(10px);
  filter: grayscale(100%); /*黑白*/
}
```

## 按钮

- 圆角: 圆角设置为 9999px 或者更大时能形成胶囊效果; 设置为 50%则是圆形(默认为椭圆, 宽高一样是圆形)
- 阴影: 参考盒阴影

这里有一个"按下"效果的按钮例子, 核心就是 active 时按钮的 translateY 和盒阴影颜色变化

```css
button {
  height: 10rem;
  width: 20rem;
  background-color: green;
  color: white;
  font-weight: bolder;
  font-size: 3em;
  text-align: center;
  line-height: 10rem;
  border-radius: 5rem;
  border-style: solid;
  border-color: gray;
  border-bottom: 10rem;
  box-shadow: 0 2rem 0 gray;
  cursor: pointer;
}
button:hover {
  background-color: rgb(0, 83, 0);
}
button:active {
  transform: translateY(1rem);
  box-shadow: 0 1rem 0 rgb(56, 56, 56);
}
```

![](https://pic.imgdb.cn/item/621db7b92ab3f51d91067856.jpg)

## flex

Flex 是 FlexibleBox 的缩写，意为"弹性布局"，用来为盒状模型提供最大的灵活性。任何一个容器都可以指定为 Flex 布局。行内元素也可以使用 Flex 布局。注意，**设为 Flex 布局以后，子元素的 float、clear 和 vertical-align 属性将失效。**

flex 有一个主轴和交叉轴的概念。

默认布局（row）下的主轴是水平的，交叉轴和主轴垂直，元素会水平排成一排；justify 用于设置主轴的排列，即水平方向上元素的排列；align 会控制交叉轴的排列，即控制元素在垂直方向上的位置。

通过 flex-direction: column 将主轴转为垂直，这时元素会纵向排成一列。

### 容器属性

- flex-wrap: flex 本身是不换行的, 多个元素就算超出长度也会挤在一行, 并且会缩小宽度;
  - wrap: 换行
- flex-direction：设置主轴方向；这个默认值是 row，即水平方向的主轴，垂直方向为交叉轴，元素一列一列排列；
  如果设置为 column，则主轴变为垂直方向，元素一行一行排列

- justify-content: 设置主轴对齐方式,
  - flex-start/flex-end: 头/尾填充
  - center
  - space-between：父容器空的空间平均插在子元素之间, 头尾紧挨
  - space-around: 父容器空的空间平均插在子元素之间, 头尾各占一半的插入空间

注意, 如果一个元素设定 `margin-right/left` 为 `auto`, 就会吞掉剩下所有空间从而把其他元素挤在一边

> `justify-item` 和 `justify-self` 是用在 grid 里边的, 不要搞错了

- align-items: 设置元素在交叉轴上的位置和对齐方式
  - flex-start：元素和交叉轴的起点对齐。
  - flex-end：元素和交叉轴的终点对齐。
  - center：元素和交叉轴的中点对齐。
  - baseline: 元素和项目的第一行文字的基线对齐。
  - stretch（默认）：如果项目未设置高度或设为 auto，将占满整个交叉轴的大小。比如主轴为 row 时，元素将默认占满高度
- align-content: （个人理解）设置主轴在交叉轴上的位置。比如默认的 row 排列下，主轴为水平，交叉轴为垂直。这时如果设置 align-content 为 flex-end，则所有元素都会到最下方，像这样：
  ![image-20221015173358006](https://blog-img-1307852525.cos.ap-chengdu.myqcloud.com/img/image-20221015173358006.png)
  这个属性是用于调整主轴在交叉轴上的位置，前提是必须设置`flex-wrap: wrap/wrap-reverse`开启换行；如果主轴只有一条（比如上面的元素只能凑成一行），就不起作用
  - flex-start：主轴与交叉轴的起点对齐，即多根交叉轴轴线都顺序从上向下排列
  - flex-end：主轴与交叉轴的终点对齐。
  - center：主轴与交叉轴的中点对齐。
  - space-between：主轴与交叉轴两端对齐，轴线之间的间隔平均分布。
  - space-around：主轴每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。
  - stretch（默认）：主轴轴线占满整个交叉轴。
- align-self: 单个设置元素的 align 属性, 和 align-items 的值一样。区别在于它可以做到只设置单个元素而不影响整个的布局。比如设置大部分元素居中，而只有最后一个元素靠边，就可以单独设置最后一个元素的 align-self 为 flex-start 即可。

- gap：和 grid 布局一样，flex 也可以通过 gap 属性控制元素之间的间距

```css
.container {
  display: flex;
  gap: 10px; /* 行和列都为10px */
  gap: 10px 20px; /* 先row后column，row为10，colume为20 */
  row-gap: 10px;
  column-gap: 20px;
}
```

### 元素属性

flex-grow/flex-shrink/flex-basis

这三个属性都是作用于具体元素的，即父元素设置了 flex 属性之后的子元素。

前两个属性都有自己的默认值，也就是说即使不设置 grow 和 shrink，元素也会遵循默认值去处理。每个元素的默认值是：

```css
* {
  flex-grow: 0; /* 表示每个元素默认不会伸展 */
  flex-shrink: 1; /* 表示每个元素默认都会收缩，并且收缩时会等分空间 */
}
```

flex-basis 没有默认值，但设置他其实和设置 width、height 实际上是一样的，只不过同时设置 basis 和元素的 width、height 时，**basis 的优先级更高。**

#### flex-grow

这个属性规定了 flex-grow 项在 flex 容器中分配**剩余空间**的相对比例。

这句话的关键在于“剩余空间”，也就是说这个属性只会在元素大小小于容器大小时才会生效，即“填补空白”；
每个元素默认的 grow 都是 0，即默认不会伸展占满剩余空间。
使用时可能有以下几种情况：

1. 每个元素都设置 flex-grow：即每个元素会按照 grow 的数字比例划分整个容器空间。

如果每个元素预先不设置 width 或 basis，那么它的宽度将会由 grow 值确定，即自动拉伸到比例值。

但是如果一个元素如果有默认的 width 或者 basis，那么这个元素会在自己本来的 width 的基础上，再次占用一部分空余空间。

比如说下面这个图：
![](https://pic1.imgdb.cn/item/6334399616f2c2beb14ef7db.jpg)

这里我们给绿色元素设置了定宽，并且给三个元素都设置了`flex-grow:1`。可以看到这时绿色元素除了自己的定宽之外，还占据了一部分大小（除去绿色元素定宽之外剩余宽度的 1/3），恰好和红蓝的大小一样（各是 1/3）。这也就是说明绿色元素会和剩下的两个没有设置定宽的元素额外分割空白空间。

2. 只有一个或几个元素设置了 flex-grow：因为每个元素都有默认的`flex-grow: 0`，因此没有设置的元素可能会被压缩到宽度为 0（如果预先没设置宽度），或者保持自己预先的宽度（即除去这个元素才是空白部分）。

另外，如果元素是交替设置`flex-grow`（即一个设置 grow，相邻的设置定宽，下一个又设置 grow，逐个交替），那么空白部分的分配并不受分割的影响，grow 依据的是容器中的**总空白**大小。

3. 对行内元素设置 flex-grow：行内和行内块元素会受到 grow 的影响改变宽度，而并不是像设置 width 属性一样没有用。

4. grow 和 shrink 同时出现：在页面大小不变化时，这两个属性并不冲突。因为 grow 是在有空白时才生效，并且分配的是空白大小，绝不会超过容器大小；shrink 则是设置总宽度超出容器大小时的情况。因此不同元素间的 grow 和 shrink 应该是不冲突的，不会出现 grow 干扰 shrink 的情况。

#### flex-shrink

CSS flex-shrink 属性指定了 flex 元素的收缩规则。flex 元素**仅在默认宽度之和大于容器的时候才会发生收缩**，其收缩的大小是依据 flex-shrink 的值

每个元素默认的 flex-shrink 都是 1，即如果宽度超出容器大小，则会按照每个元素等分的形式收缩每个元素。shrink 值越大表示收缩的比例越大
情况：

1. 元素有定宽，同时设置 flex-shrink：如果宽度和超过容器宽度，会无视定宽强制收缩，除非把 shrink 设置为 0，相当于无论什么情况都不会收缩

> 如果元素设置了 min-width，那就不会收缩小于 min-width 的大小。其他属性都会强制收缩到 content 大小

2. 对行内元素设置 shrink：和 grow 相同，行内元素也会发生收缩，不会保持其定宽。

#### flex-basis

用于设置 flex 元素在主轴方向上的初始大小。如果不使用 box-sizing 改变盒模型的话，那么这个属性就决定了 flex 元素的内容盒（content-box）的尺寸。

> 当一个元素同时被设置了 flex-basis (除值为 auto 外) 和 width (或者在 flex-direction: column 情况下设置了 height) , flex-basis 具有更高的优先级。

basis 设置的是不考虑任何 grow 或 shrink 的情况下的基本大小，和设置 width 或 height 的效果是一样的

flex-basis 的取值有：

```css
/* 指定<'width'> */
flex-basis: 10em;
flex-basis: 3px;
flex-basis: auto;

/* 固有的尺寸关键词 */
flex-basis: fill;
flex-basis: max-content;
flex-basis: min-content;
flex-basis: fit-content;

/* 在 flex item 内容上的自动尺寸 */
flex-basis: content;

/* 全局数值 */
flex-basis: inherit;
flex-basis: initial;
flex-basis: unset;
```

basis 有两个特殊的值：0 和 auto

- 如果一个元素的 basis 值被设置为 0，那么 flex 会把这个元素视作尽可能小。正常排列时差别不大，但如果有 grow 或 shrink 就会出现差异

可以理解为，为 0 时这个元素在主轴上占用的空间被视为 0。
当 flex-grow 和 flex-shrink 计算剩余空间大小时，该元素会被视为不占大小。

比如说
例如：

```jsx
<div style={{ width: "200px", display: "flex" }}>
  <div style={{ flex: 1 }}>子元素</div>
  <div style={{ flex: 1, flexBasis: 50 }}>子元素</div>
</div>
```

上面这个例子中，第二个元素的 flex-basis 为 50，第一个的 basis 为 0。
首先（不设置任何 grow 和 shrink，空间足够）第一个元素会缩小到 content 的大小，即自己的最小值。这里 a 的宽度收缩到了内容的宽度，而 b 的宽度为 50px
![](https://pic.imgdb.cn/item/63ea357f4757feff33e89be9.jpg)
其次，如果两个都设置 flex-grow: 1，那么 b 会比 a 大 50px
![](https://pic.imgdb.cn/item/63ea35f44757feff33ebc4ac.jpg)
最后，如果触发 shrink，那么第一个元素不会再缩小，而第二个元素也会缩小到 content 大小之后，不再会缩小。
![](https://pic.imgdb.cn/item/63ea363e4757feff33edada4.jpg)

总结：设置了`basis: 0`的元素会缩小到最小大小，即 content 的大小，如果通过 flex-grow 让其伸展，也会从 0 开始计算。

- 如果元素的 basis 设置为 auto，那就是它的默认大小，和 width：auto 类似。即如果元素设置了宽度那就是它的宽度，如果没设置就是其内容的宽度

即，如果元素设置了 width 或 height 那就和正常一样，如果没设置就和上面`basis: 0`的情况一样。

## grid

### 行和列的划分

创建 grid 布局的第一步就是先划分好行和列。划分行列的主要属性有：

1. `grid-template-columns`/`grid-template-rows`：划分列和行。这两个属性的取值方式是完全相同的，划分之后并不会出现分界线，但是元素内部会被划分；如果元素内部放入子元素，则会自动按照划分的行和列排布。

这两个属性不一定全部都要出现，只划分行或者只划分列也可以。这样就可以很容易的做到仅用 grid-template-columns 来创建两栏、三栏布局。
比如可以创建一个左边自适应，右边固定的输入框：

```html
<div class="container">
  <input />
  <button></button>
</div>
<style>
  .container {
    display: grid;
    grid-template-columns: 1fr 100px;
  }
</style>
```

在取值上，则主要有几种类型的取值：

- 具体数值，包括 px、百分比、em、rem 等常用的绝对或相对单位。
  grid 还提供了一个特殊的单位是 fr，这个单位的取值是相对容器内部的**空白位置**按照比例划分。
  比如取值为`1fr 100px 2fr 200px`，假如容器总宽度是 600px，那么两个固定大小的栏占到了 300px，剩下的空白部分的 300px 则由两个 fr 按照 1:2 划分。其他取值同理。如果单位都是 fr，那就相当于划分了整个空间的大小。
- repeat：主要是用于简化多个栏大小相同时重复写。
  `repeat()`接受两个参数，第一个参数是重复的次数，第二个参数是所要重复的值。其中第一个参数也可以是 auto-fill/auto-fit 关键字，第二个参数可以是一个数值，也可以是一组数值。比如下面这几种形式都是可以的：
  ```css
  .container {
    display: grid;
    grid-template-columns: repeat(3, 30px);
    grid-template-columns: repeat(3, 1fr);
    grid-template-columns: repeat(3, 33%);
    grid-template-columns: repeat(auto-fill, 300px);
    grid-template-columns: repeat(3, 30px 50px 1fr);
  }
  ```
  其中两个关键字 auto-fill/auto-fit：
  - auto-fill：用于确定列或行的数量。当设定好栏的宽度时，则会尽量填充直到放不下更多栏为止。如果栏的总宽度超过容器宽度，那么就会减少栏的数量。比如说容器宽度为 1500px，一列设置为 300px，那就会放下五列；如果一列设置为 400px，则会放下 4 列，以此类推。如果`总宽度/栏宽度>当前元素数量`，还会补上和前面栏一样大小的空白。
  - auto-fit：和 autofill 基本一致，唯一的区别是如果总宽度不足容器宽度，则会拉伸所有的栏充满整个容器，类似 flex-grow，而不会补上空白。
- minmax：参数是范围的最大最小值；一般是一个相对单位和一个绝对单位，或者是两个相对单位，函数产生一个长度范围，表示长度就在这个范围之中，表示当前设定的栏大小不小于最小值，不大于最大值，在这个范围内根据容器宽度变化自动伸缩。

2. `grid-template-areas`：这个属性提供了一种自己创建格子的方法，可以将网格中的一些单元格组合成一个区域（area），并为该区域指定一个自定义名称。对于每个容器内的元素，只需要通过`grid-area`属性指定自己属于哪个格子，就可以自动布局到设计好的属性中。
   具体写法是按照行划分的字符串，每个字符串中间留空
   比如：

```css
.container {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  grid-template-rows: 1fr 1fr 1fr;
  grid-template-areas:
    "header header header"
    "advert content content"
    "advert footer footer";
}
.header {
  /*...*/
  grid-area: header;
}
```

3. `grid-gap`：可以设置行和列相互之间的间隔。单位和 padding、margin 一样。如果 grid-gap 只有一个值，那么这个值表示行与行之间、列与列之间的间距均为这个值。 如果有两个值，那么第一个值表示行间距，第二个值表示列间距。

### 设定内部元素

划分好行和列之后就是放置内部元素了。内部元素可以让 grid 自然排列，当然更好的方法是控制某个元素的起始行、列从而控制该元素在网格内的位置。

控制元素的位置主要有几个属性：

1. `grid-auto-flow`：即控制默认的位置。如果不通过 grid-column 等方法明确控制元素位置，元素就会按照文档流自动排列；这个属性就是控制元素排列是"先行后列"还是"先列后行"。默认值是 row，即"先行后列"。也可以将它设成 column，变成"先列后行"。

- row：先行后列
- column：先列后行
- row dense：某些项目指定位置以后，剩下的项目"先行后列"，并且尽可能紧密填满每一行，尽量不出现空格。
- column dense：某些项目指定位置以后，剩下的项目"先列后行"，并且尽可能紧密填满每一列，尽量不出现空格。

2. `grid-column-start`/`grid-column-end`/`grid-row-start`/`grid-row-end`：设置在容器内每个具体的元素上，用于控制元素的起始位置和终点位置。

单位是从 1 开始的序号，表示某一栏的起始，可以打开控制台查看具体是哪一行哪一列
![](https://pic1.imgdb.cn/item/6335644416f2c2beb152a9ce.jpg)
可以看到每个位置的序号实际上是每格的起始线。因此比如从列的第一格横跨两格，就应该是 start 设为 1，end 设为 3（不是 2）

3. `grid-column`/`grid-row`：和上面的属性功能相同，但是简写，更推荐这一种方式。

写法是通过`start/end`的形式指定起始或结束位置，比如

```
grid-column: 1 / 3;
```

这会让网格项从左侧第一条线开始到第三条线结束，占用两列。

4. `grid-area`：这个属性还可用作 grid-row-start、grid-column-start、grid-row-end、grid-column-end 的合并简写形式，直接指定项目的位置。

比如：

```css
.item {
  grid-area: <row-start> / <column-start> / <row-end> / <column-end>;
}
.item-1 {
  grid-area: 1 / 1 / 3 / 3;
}
```

除了控制位置，还可以设置元素在格子内的分布，主要是四个属性：

1. `justify-self`/`align-self`：

`justify-self`属性设置单元格内容的水平位置（左中右），只作用于**单个**项目。
`align-self`属性设置单元格内容的垂直位置（上中下），只作用于单个项目

```css
.item {
  justify-self: start | end | center | stretch;
  align-self: start | end | center | stretch;
}
```

这两个属性都可以取下面四个值。

- start：对齐单元格的起始边缘。
- end：对齐单元格的结束边缘。
- center：单元格内部居中。
- stretch：拉伸，占满单元格的整个宽度（默认值）。

2. `justify-item`/`align-item`：这两个属性的取值和上面的完全一致，唯一的区别是这两个属性作用于容器元素，用于同时设置内部**每个**元素的位置。

## CSS Sprites

> CSS Sprites 是一种将多个图像组合成单个图像文件以在网站上使用的方法，以帮助提高性能。

CSS Sprites 并不能明显减小图片的总体积，甚至还会增加；但是它可以减少 HTTP 请求数量，从而加快加载速度和访问速度。由于浏览器对于 http 并发请求数量的限制，越少的请求当然是越快的。尤其是对于一些网页上的小图标，多达几百个小图像一个一个请求势必不如一次请求来的更快、更节省网络资源。

### 使用 CSSSprites

一般来说形成雪碧图是通过 PS 等工具将数个图片拼接在一个图片内；生成雪碧图时还会告知每个小图片在大图片中的位置，在 css 使用的过程中，通过调整`background-position`从而实现在一个容器内展示不同的图像。

举个栗子：
![](https://segmentfault.com/img/bVGpCj?w=568&h=426)

```css
.ps_demo_wrap .weibo_icon,
.ps_demo_wrap .qq_icon,
.ps_demo_wrap .douban_icon,
.ps_demo_wrap .renren_icon {
  width: 54px;
  height: 54px;
  background: url("../images/CssGaga.png");
}
.ps_demo_wrap .weibo_icon {
  background-position: -168px 0px;
}
.ps_demo_wrap .qq_icon {
  background-position: -56px 0px;
}
.ps_demo_wrap .douban_icon {
  background-position: 0px 0px;
}
.ps_demo_wrap .renren_icon {
  background-position: -112px 0px;
}
```

## 媒体查询

> 媒体查询（Media queries）非常实用，尤其是当你想要根据设备的大致类型（如打印设备与带屏幕的设备）或者特定的特征和设备参数（例如屏幕分辨率和浏览器视窗宽度）来修改网站或应用程序时。
> 媒体查询常被用于以下目的：
> 有条件的通过 @media 和 @import at-rules 用 CSS 装饰样式。
> 用 media= 属性为<style>, <link>, <source>和其他 HTML 元素指定特定的媒体类型。

媒体查询的对象不仅是屏幕，还可以是 print（打印机）和 speech（语音）。但是最常见的还是屏幕 screen

媒体查询的基本语法：

```css
/* 可能是基本的查询 */
@media 查询对象 (查询参数: 参数值) {
  /* 如果查询成功的样式 */
}

/* 可以不需要查询对象 */
@media (max-width: 1000px) {
  /* ... */
}

/* 也可以同时查询多个，用and/or这样的逻辑符连接 */
@media 查询对象 (查询参数: 参数值) and/or 查询对象 (查询参数：参数值) ... {
  /* 如果查询成功的样式 */
}
```

对于 css 来说，查询的对象绝大多数情况都是 screen，常用的查询参数有：

- width/height：查询**视窗的宽高**。

  - max-width：当媒体查询值为 true 的最大宽度，也就是说当宽度小于 max-width 指定的值时，查询的值为 true，触发内部的样式。相当于`width <= xxx`
  - min-width：max-width 的反向，相当于`width >= xxx`

  另外，新增的语法可以直接这样写：

  ```css
  @media (width <= 30em) {
    ...;
  }
  @media (30em <= width <= 50em) {
    ...;
  }
  ```

- color：和屏幕的色彩相关

## 判断元素是否进入可视区域

### 几个数据

#### 1. 偏移量 offset

offset 主要有四个属性：

- offsetHeight
  元素在垂直方向上占用的空间大小，以像素计。包括元素的高度、(可见的) 水平滚动条的高度、上边框高度和下边框高度。
  `offsetHeight = content + padding + border + scrollX`
- offsetWidth
  元素在水平方向上占用的空间大小，以像素计。包括元素的宽度、(可见的)垂 直滚动条的宽度、左边框宽度和右边框宽度。
  `offsetWidth = content + padding + border + scrollY`
- offsetLeft
  元素的左外边框至包含元素的 offsetParent 元素的左内边框之间的像素距离。
- offsetTop
  元素的上外边框至包含元素的 offsetParent 元素的上内边框之间的像素距离。

![](https://pic.imgdb.cn/item/63ea3e054757feff331ecbbd.webp)

> 元素的 offsetTop 和 offsetLeft 都是基于其 offsetParent 元素的，即最近的 table,td,th,body 元素。一般情况下都是`body`元素，因此相当于一个元素距离文档最顶层的距离（包括不可见的部分）

#### 2. 客户区 client

client 的相关 api 是针对元素本身的

- `clientWidth`：元素内容区宽度加上上下内边距宽度，即`content+padding`
- `clientHeight`：元素内容区高度加上上下内边距高度
  上面两个属性是元素内部宽高；在`content-box`中，还需要算上 padding；并且不包含滚动条的宽度
  ![](https://pic.imgdb.cn/item/622aec9c5baa1a80abc8832a.jpg)
  下面两个是指元素边框宽度
- `clientTop`：元素顶部边框的宽度
- `clientLeft`：元素左边框宽度

#### 3. 滚动大小 scroll

主要有两个属性：

- `scrollTop`：以获取或设置一个元素的内容垂直滚动的像素数。

> 一个元素的 scrollTop 值是这个元素的内容顶部（卷起来的）到它的视口可见内容（的顶部）的距离的度量。当一个元素的内容没有产生垂直方向的滚动条，那么它的 scrollTop 值为 0。  
> 也就是说，scrollTop 一定是在一个可滚动的元素上才不为 0，表示的是该元素被滚动条“卷起来”或者说是“藏起来”的大小。  
> 针对整个页面，可以使用`document.documentElement.scrollTop`获取整个页面被卷起来的部分（或者用`window.scrollY/window.pageYOffest`也可以），也就是页面最顶部（`<html>`签）到用户可视区域最顶部的距离。

- `scrollLeft`：和 top 类似，不过是针对水平滚动条的。
- `scrollHeight`：在没有滚动条的情况下，元素内容的总高度。
- `scrollWidth`：在没有滚动条的情况下，元素内容的总宽度。

### 判断可视区域

#### 硬核计算

综合上面几个 api，就可以得到计算可视区域的方法
`公式: el.offsetTop - document.documentElement.scrollTop <= viewPortHeight`
![](https://pic.imgdb.cn/item/621ddc062ab3f51d914c3b47.jpg)
![](https://pic.imgdb.cn/item/622aeab05baa1a80abc70825.jpg)
原理是这张图片，很显然如果元素到达视口，`scrollTop`和`innerHeight`加起来就等于元素到文档顶部的距离，即`offsetTop`

```js
function isInViewPortOfOne(el) {
  // viewPortHeight 兼容所有浏览器写法
  const viewPortHeight =
    window.innerHeight ||
    document.documentElement.clientHeight ||
    document.body.clientHeight;
  const offsetTop = el.offsetTop;
  const scrollTop = document.documentElement.scrollTop;
  const top = offsetTop - scrollTop;
  console.log("top", top);
  // 这里有个+100是为了提前加载+ 100
  return top <= viewPortHeight + 100;
}
```

#### `getBoundingClientRect`

返回值是一个 DOMRect 对象，是包含整个元素的最小矩形（包括 padding 和 border-width）。该对象使用 left、top、right、bottom、x、y、width 和 height 这几个以像素为单位的只读属性描述整个矩形的位置和大小。属性是**相对于视图窗口的左上角**来计算的。

```js
const target = document.querySelector(".target");
const clientRect = target.getBoundingClientRect();
console.log(clientRect);

// {
//   bottom: 556.21875,
//   height: 393.59375,
//   left: 333,
//   right: 1017,
//   top: 162.625,
//   width: 684
// }
```

![](https://pic.imgdb.cn/item/6243edbc27f86abb2a8af251.jpg)
当页面发生滚动的时候，top 与 left 属性值都会随之改变，可能会有四种情况：

- 元素在页面上方（的不可视区域内）：top 和 bottom 都为负数。这时如果 bottom 由负转正，就说明移动到了可视区域
- 元素在页面下方：top 的值大于等于可视区域高度，即`top <= window.innerHeight`时进入可视区域
- 元素在页面左边：right 的值为负数
- 元素在页面右边：left 的值大于等于可视区域宽度

因此可以通过判断这四个条件是否满足，来确定是否在窗口内

```js
function isInViewPort(element) {
  const viewWidth = window.innerWidth || document.documentElement.clientWidth;
  const viewHeight =
    window.innerHeight || document.documentElement.clientHeight;
  const { top, right, bottom, left } = element.getBoundingClientRect();

  return {
    isVerticalVisible: top >= 0 && top <= viewHeight && bottom >= 0,
    isHorizontalVisible: left >= 0 && right >= 0 && left <= viewWidth,
  };
}
```

这个方法会随着滚动页面触发，所以触发频率会很高，建议使用时加上防抖。

---

还有几种方法可以参考：https://juejin.cn/post/6844903725249609741

# CSS 奇怪的用处和技巧

## 画三角形

画三角形的原理是 css 的边框相交处会形成斜线，也就是我们可以利用足够大的边框，再把没用的变透明就行

1. 首先看下原理：

```css
.triangle {
  width: 0;
  height: 0;
  border-top: 50px solid blue;
  border-right: 50px solid red;
  border-bottom: 50px solid green;
  border-left: 50px solid yellow;
}
```

![](https://segmentfault.com/img/bVx8Ve)

2. 所以如果把不需要的三角形颜色调成透明就行，如下：

```css
.triangle {
  width: 0;
  height: 0;
  border-width: 0 50px 50px;
  border-style: solid;
  border-color: transparent transparent red;
}
```

这里因为只需要三条边，所以只用设置右、下、左三边的宽度为 50px；左、右颜色为透明；

3. 因为第二个属性是 x 方向，如果增大第二个大小就可以增大底边，同理增大第三个就可以增大高

```css
.triangle {
  /*...*/
  border-width: 0 100px 50px; /*底边宽为100 高为50*/
}

/*相当于*/
.triangle {
  border-width: 0 100px 100px 50px;
}
```

如果设置两条相邻边，还可以形成直角三角形：

```css
div {
  width: 0;
  height: 0;
  border-top: 100px solid red;
  border-right: 100px solid transparent;
}
```

![](https://pic.imgdb.cn/item/62247b0a5baa1a80ab43611f.jpg)

### 画扇形

扇形的原理和三角形一样，只是设置`border-radius`

```css
.triangle {
  width: 0;
  height: 0;
  border-style: solid;
  border-color: blue transparent transparent transparent;
  border-width: 100px 100px 0;
  border-radius: 50%;
}
```

![](https://pic.imgdb.cn/item/62247ba85baa1a80ab43ce2c.jpg)

## CSS 使用硬件加速

在浏览器中用 css 开启硬件加速，使 GPU (Graphics Processing Unit) 发挥功能，从而提升性能。硬件加速在移动端尤其有用，因为它可以有效的减少资源的利用。
目前主流浏览器会检测到页面中某个 DOM 元素应用了某些 CSS 规则时就会开启，最显著的特征的元素的 3D 变换。如果不使用 3D 变形，我们可以通过下面方式来开启：

```css
.wrap {
  transform: translateZ(0);
}
```

关于这种方式的实现原理涉及到浏览器渲染 css 时的原理，具体可以参考这里：https://juejin.cn/post/6844903966573068301

## input 只是光标样式改变

```css
input {
  caret-color: red;
}
```

## max-content / min-content / fill-available / fit-content

这几个值都可用在 width, height, min-width, min-height, max-width 和 max-height 属性上。
display 必须为 inline-block 或者 block，否则不起作用。

- fill-available：元素撑满可用空间。参考的基准为父元素有多宽多高，块级元素默认就是此类型
- fit-content：表示元素自动伸缩到内容的宽度，典型代表就是浮动、绝对定位、inline-block 元素或 table 元素。
- max-content：宽度/高度会自动调整为刚刚好容纳下子元素；参考的基准为最大子元素有多宽多高。对于**文本内容**即使导致溢出也不会换行。
- min-content：同 max，为最小子元素大小

## 关于最小尺寸

### 0.5px

对于边框，如果直接设置边框宽度为 0.5px，正常情况下看不出差别，放大网页之后可以看到：

![](https://pic.imgdb.cn/item/622480415baa1a80ab47313a.jpg)

但是实际上在默认大小下 chrome 就是将 0.5px 渲染为 1px 的；因此直接设置 0.5px 不行。

可以有其他很多方法绘出一条 0.5px 的线：

#### `transform`

通过`transform: scaleY(0.5);`的形式可以使 1px 缩小为 0.5px。这种方法可以应用于块级元素的变化，但是不能直接使用在设置边框；

```css
.line {
  height: 1px;
  transform: scaleY(0.5);
}
```

和 1px 的线比较，可以看到是可行的：
![](https://pic.imgdb.cn/item/6224832a5baa1a80ab496615.jpg)

但是这种方法在一些设备上会使得线变虚；可以通过设置`transform-origin: 50% 100%;`解决。具体原理不清楚，总之神奇就对了。(我试的时候用了反而变虚了，当作八股背就好，反正实际不要用)

> transform-origin 更改一个元素变形的原点
> 转换起点是应用转换的点。例如，rotate()函数的转换原点是旋转中心。默认的转换原点是 center。设置的值就是相对于默认点的偏移量
> 设置为 50% 100% 即在下边框的中心位置：
> ![](https://pic.imgdb.cn/item/6224841b5baa1a80ab4a207a.jpg)

---

对于边框，不能直接缩小，因为会影响到被边框包着的整个元素的大小。更好的方式是使用伪元素：

```css
/*<main class="after-border"></main> */

.after-border {
  margin-top: 50px;
  width: 500px;
  height: 300px;
  position: relative;
}
.after-border::after {
  content: "";
  height: 1px;
  width: 100%;
  background-color: red;
  position: absolute;
  left: 0;
  top: 0;
  transform: scaleY(0.5);
  transform-origin: top left;
}
```

![](https://pic.imgdb.cn/item/622498505baa1a80ab5a0d8f.jpg)
这样会绘制元素的一条边，这个例子中是上方的一条边。

> 该元素的 position: relative;属性是必须的，因为只有这样才可以保证其 after 和 before 伪元素是基于它而不是基于整个文档的；
> 如果不设置，top 和 left 是相对于整个文档，width 同理；设置之后相当于是该元素的`top:0`位置。

如果需要四边边框，可以这样：

```css
<div class="border">0.5像素边框</div>
<style>
    .border {
        width: 200px;
        height: 200px;
        margin: 0 auto;
        position: relative;
    }
    .border::before {
        content: " ";
        position: absolute;
        top: 0;
        left: 0;
        width: 200%;
        height: 200%;
        border: 1px solid red;
        transform-origin: 0 0;
        transform: scale(0.5);
    }
</>
```

![](https://pic.imgdb.cn/item/622499b25baa1a80ab5b65b4.jpg)
原理是相当于创建了一个先扩大一倍、再缩小一半的伪元素，这样边框也会被恰好缩小一半。

#### svg

由于 svg 的描边等属性的 1px 是物理像素的 1px，相当于高清屏的 0.5px，因此可以用 svg 绘制 0.5px 的线

```html
<svg width="100%" height="1px">
  <line stroke="black" x1="0" x2="100" y1="100%" y2="100%"></line>
</svg>
```

#### viewport

相对于正常的 initial-scale 为 1 的情况，

```html
<meta name="viewport" content="width=device-width,initial-scale=0.5" />
```

由于 initial-scale 为 1 时，1 个 css 像素相当于 1 个设备像素；
如果设置为 0.5，即页面被缩小了 1/2，viewport 的 css 像素扩大了一倍，相当于 2 个 css 像素对应 1 个设备像素，这样设置 1px 的线在设备上就相当于 0.5px。
但是这种方法导致其他元素都会变化，因此如果只是一条线想改为 0.5px，就不能使用这种方法。

#### 其他方法

还可以用渐变或 box-shadow 的方法实现，具体不再赘述，可以参考这里：https://juejin.cn/post/6844903582370643975

### 12px

在 chrome 下 css 设置字体大小为 12px 及以下时，显示都是一样大小，都是默认 12px。
如果确实希望小于 12px 的字体，可以通过`transform:scale(0.5)`实现，但是会同样的改变被修改字体间的空白和间隔。
注意最好指明`transform-origin`在合适的位置，不然可能导致字体偏斜；

```css
.text2 span {
  font-size: 16px;
  transform: scale(0.5); /* font-size: 16*0.5=8px  */
  transform-origin: left top; /* 调整位置*/
}
```

## 暗黑模式

css 对于暗黑模式的处理大体分为两种思路

1. 页面滤镜。这种方法比较简单粗暴，通过 filter 等方式给页面加滤镜，从而达到反转颜色的效果
2. 变量适配。即以 css 的 prefers-color-scheme 媒体查询为核心，通过对每个元素的变量修改来实现

第一种方法可以参考：https://segmentfault.com/a/1190000023598551

具体来说，第二种方法还有几个具体的实现：

1. css 变量，比如

```css
:root {
  --text-color: #000;
  --dark-text-color: #fff;
}

.text {
  color: var(--text-color); // 亮色模式下字体为黑色
}

@media (prefers-color-scheme: dark) {
  .text {
    color: var(--dark-text-color); // 暗黑模式下字体为白色
  }
}
```

2. less/sass 变量

```less
@text-color: #000;
@dark-text-color: #fff;

.text {
  color: @text-color; // 亮色模式下字体为黑色
}

@media (prefers-color-scheme: dark) {
  .text {
    color: @dark-text-color; // 暗黑模式下字体为白色
  }
}
```

这两种方式都有一个致命问题，就是需要手写很多的媒体查询代码。
这个过程可以通过 postCss 插件来自动化实现：

```less
// 这部分为原代码
@text-color: #000;
@dark-text-color: #fff;

.text {
  color: @text-color; // 亮色模式下字体为黑色
}

// 这部分根据映射规则自动生成
@media (prefers-color-scheme: dark) {
  .text {
    color: @dark-text-color; // 暗黑模式下字体为白色
  }
}
```

但是还有缺陷，就是每个原本的样式都要用到变量，而且每个样式的黑色和白色样式需要自己编写，并且像 canvas、svg 这样的元素还得自己写样式。

3. darkReader：一个 npm 包，提供了自动化解决方案。这是比较工程化的解决方案。

在页面开启转换时，DarkReader 从三个地方获取页面的样式规则：

1. style 标签，element.sheet.cssRules
2. link 标签，element.sheet.cssRules
3. 行内样式，HTML style 属性

使用 MutationObserver API 监听 DOM 变更，当上述元素发生变化时，去获取最新的样式规则。

获取到样式规则后，DarkReader 通过一定的色值计算，将色值反转为暗黑模式下的色值。最后将新的规则插入到页面中。这里并没有直接覆盖原来的值，而是新生成 style 标签，对原有样式进行覆盖；行内样式则是通过新增 CSS Variable 进行覆盖。

# CSS 布局

## 圣杯布局

先上代码：

```html
<body>
  <div class="container">
    <main class="middle">middle</main>
    <aside class="left">left</aside>
    <aside class="right">right</aside>
  </div>
</body>
<style>
  * {
    margin: 0;
    padding: 0;
  }
  .container {
    padding: 0 200px;
    overflow: hidden;
  }
  .left,
  .middle,
  .right {
    position: relative;
    float: left;
    min-height: 130px;
  }
  .left {
    margin-left: -100%;
    left: -200px;
    width: 200px;
    background-color: red;
  }
  .right {
    margin-left: -200px;
    right: -200px;
    width: 200px;
    background-color: green;
  }
  .middle {
    width: 100%;
    background-color: blue;
  }
</style>
```

总结一下圣杯布局用到的原理：

1. 浮动，三个元素都是浮动的，正因为浮动才使得三个元素可以贴合，并且和下面的负外边距共同作用。
2. 负外边距，是圣杯布局的核心。原理是负外边距对元素有一个“牵拉”的效果，在浮动中这个效果甚至可以将元素拉到原本不属于他的行或列中。
3. 清除浮动，因为三个元素都是浮动的，因此父元素要清除浮动，否则会塌陷。

下面详细解释。

#### 元素的排列

中间行应该写在最前面，因为 middle 先出现，并且宽度设为 100%才能把左和右挤到下面去；否则左元素会在上方，不管什么方法也不能把一个元素拉下来。

#### 父元素的要求

包裹这三个元素的父元素有两个要求：

1. 设置左右 padding，让 padding 装下布局的左右栏；否则会使得左右栏盖住 middle。下图的 middle 中本来有文本“middle”的，但是显然被左右盖住了。

![](https://pic.imgdb.cn/item/621dfbcb2ab3f51d918741a5.jpg)

2. 设置为 BFC，主要原因是三个子元素都是浮动。

#### 中间元素限制

中间部分应该设置成宽度 100%，从而起到把剩下两栏“挤下去”的效果；
但是实际上由于父元素设置了 padding，中间元素只是占据了一部分，两边空白留给了剩下两栏。

#### 左右两栏

左右两栏的设置是布局的关键。
上面说到，负外边距对元素有一个“牵拉”的效果，比如我们先这样设置：

```css
.left {
  margin-left: -100px;
  width: 200px;
  background-color: red;
}
```

可以看到效果如下：
![](https://pic.imgdb.cn/item/621dfd382ab3f51d9189abdc.jpg)
左栏相当于被向前拉了 100px。
如果继续增大，达到-200px，就会这样：
![](https://pic.imgdb.cn/item/621dfd952ab3f51d918ae4dd.jpg)
可以看到元素已经被拉上去了。
对于左栏来说，还需要一直到最左边，也就是要拉过整个 middle 部分；但我们不知道 middle 的宽度，因此应该使用百分比。
前面说过，margin 设置百分比是父元素的宽度的比例；如果设置 100%，就可以实现直接跨到最左边了
![](https://pic.imgdb.cn/item/621dfe172ab3f51d918bfdca.jpg)
但是还是不太够：因为这里遮住了 middle 元素；所以应该让他向左移动，不盖住 middle 元素，即设置 left 和 position

```css
.left {
  margin-left: -100%;
  position: relative;
  left: -200px;
  width: 200px;
  background-color: red;
}
```

右边也是同理；不过右边只需要让自己移动“一个身位”就可以到上一排，也就是自己的宽度

```css
.right {
  margin-left: -200px;
  right: -200px;
  width: 200px;
  background-color: green;
}
```

最终效果如下：
![](https://pic.imgdb.cn/item/621dffb32ab3f51d918e9b40.jpg)

## 双飞翼布局

双飞翼相比于圣杯，左右排的方法差不多；但是左右位置的保留是通过中间列的 margin 值来实现的，而不是通过父元素的 padding 来实现的。本质上来说，也是通过浮动和外边距负值来实现的。
实现方法是，在中间排添加一个元素，然后把这个元素的两边 margin 设置为左右两栏的宽度，相当于是“挤”走左右两栏的。

> 为什么不能直接给 middle 元素设置 margin 呢
> 显然这里 middle 元素的宽度是 100%，直接给 middle 元素设置 margin 必然会把 left 和 right 挤到一边去
> 但如果 middle 没有 100%，那就失去了这两种布局的灵魂，也就是 middle 部分不能自然伸缩了

代码如下：

```html
<body>
    <div class="container">
      <main class="middle">
          <div class="inner">inner</div>
      </main>
      <aside class="left">left</aside>
      <aside class="right">right</aside>
    </div>
  </body>
  <style>
    * {
      margin: 0;
      padding: 0;
    }
    .container {
      /* padding: 0 200px; */
      overflow: hidden;
    }
    .left,
    .middle,
    .right {
      position: relative;
      float: left;
      min-height: 130px;
    }
    .left {
      margin-left: -100%;
      width: 200px;
      background-color: red;
    }
    .right {
      margin-left: -200px;
      width: 200px;
      background-color: green;
    }
    .inner{
      margin: 0 200px;
    }
    .middle {
      width: 100%;
      background-color: blue;
    }
```

可以看到中间的 inner 的 margin 留下了给左右两栏的空间：
![](https://pic.imgdb.cn/item/621e018e2ab3f51d9191b240.jpg)

## 居中

### 不知道父子宽高的解决方案

（水平居中）

1. 不知道父元素宽度，但知道子元素宽度

- `text-align:center`
- `margin:0 auto;`
- postion 的几种都可以
- flex 和 grid

2. 知道父元素宽度，不知道子元素宽度

- `text-align:center`
- flex 和 grid
- position，但只能用`transformX(-50%)`

3. 父子宽高都不知道

- `text-align:center`
- flex 和 grid
- position，但只能用`transformX(-50%)`

---

（垂直居中）

1. 不知道父元素高，知道子元素高

- flex 和 grid
- position 的几种：
  - position + margin auto
  - position + transform 或-margin 调整

2. 知道父元素高度，不知道子元素高度

- flex 和 grid
- position，但只能用`transformY(-50%)`调整，不能用 margin auto
- line-height

> 注意`margin:0 auto;`可以用于直到元素宽度的情况下水平居中，但是不能用于垂直居中，即使知道元素高度也不行。
> 垂直居中如果要使用 margin 居中，就必须有 position 的协助，将 top 和 bottom 都设为 0 并将 margin 设为 auto 就可以。

### 水平居中

水平居中比较简单，主要解决方法有以下几个：

#### 1. `text-align:center`

适用于任何**行内元素**，包括非文本行内块和行内元素，比如文本 text、图像 img、按钮超链接等。

具体来说，这种方式只适用于形成 ifc 的场合，即一个块级元素内部只包含 inline 或 inline-block 元素时，其内部的水平排列可以用 text-align 控制

对于子元素是块级元素的情况，也可以设置为`display: inline-block`，然后继续使用这种方法。

#### 2. `margin: auto;`

上下 margin 为任意值，左右为 auto，前提是**必须要指明子元素块级元素的宽度**（max-width 也可以），父元素宽度不需要指明

```html
<body>
    <div class="container">
      <p>container</p>
    </div>
  </body>
  <style>
    p{
      max-width: 50px;
      /*或 width:50px*/
      margin: 0 auto;
    }
```

对于不定宽的元素可以先定为`display:table`，然后用`margin:0 auto;`实现。

```css
.center {
  display: table;
  margin: 0 auto;
}
```

#### 3. position

利用绝对定位并用 left、right 等方法移动居中，然后再通过 margin 或者 transform 把元素中心移到父元素中心

父元素必须先定位为 relative:

```css
.container {
  position: relative;
}
```

否则绝对定位会定到 body

首先第一步都是这样的：

```css
.center {
  position: absolute;
  left: 50%;
}
```

但是此时只是元素的左侧边对准了父元素的中心，还需要移动子元素让其中心对准父元素中心。移动的方法有两个，根据子元素是否定宽区分：

- 定宽：`margin: -1/2width`

```css
.center {
  width: 50px;
  position: absolute;
  left: 50%;
  margin: -25px;
}
```

或者也可以再使用`margin: 0 auto`，但是还需要指明 left 和 right 为 0

```css
.center {
  width: 50px;
  position: absolute;
  left: 50%;
  margin: 0 auto;
  left: 0;
  right: 0;
}
```

- 如果子元素不确定宽度：`transform:translateX(-50%)`

```css
.center {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
}
```

### 垂直居中

实现的 html 代码：

```html
<div class="container">
  <div class="center"></div>
</div>
```

#### 1. `absolute`

这种方法和水平居中的原理相同，通过先`top:50%`的方法放到中间，然后再用 margin 或者 translate 对准

```css
.container {
  min-height: 100vh;
  position: relative;
}
.center {
  position: absolute;
  top: 50%;
  height: 200px;
  margin: -100px 0 0 0; /*transform:translateY(-50%)也可以*/
}
```

或者可以使用上下左右距离全为零的方法，然后设置 margin:auto，并且确定子元素的宽高。其原理是上下左右距离全为零时元素有拉伸充满的趋势，但是由于设置了宽高，因此实际只会改变 margin 以充满区域，而元素相当于被居中了

```css
.container {
  min-height: 100vh;
  position: relative;
}
.center {
  position: absolute;
  /* left: 0; 如果只需要垂直居中，左右可以省略*/
  /* right: 0; */
  top: 0;
  bottom: 0;
  margin: auto;
  width: 200px;
  height: 100px;
}
```

如果父元素高度不确定，可以使用`calc(50% - 1/2height)`的方式计算具体移动的距离。水平居中也适用

```css
.center {
  position: absolute;

  height: 100px;
  top: calc(50% - 50px);
}
```

#### 2. `padding`

令**父元素**padding 上下相等并且值都为自身高度减去子元素高度，先当与父元素用 padding 包住了子元素

```css
.container {
  height: 500px;
  padding: 200px 0;
}
.center {
  height: 100px;
}
```

但是限制很大，必须知道父子元素高度。

#### 3. `line-height`

如果用于单行文本垂直居中，只需要设置 `line-height` 和 父元素 height 相同即可，并不需要明确指定 height

```css
.container {
  height: 500px;
}
.center {
  line-height: 500px;
}
```

注意这个值不能是 100% 或者 1，因此必须明确知道父元素高度。
这种方式只能居中单行文本元素，如果想要居中 inline-block 就需要 vertical-align

#### 4. `vertical-align`

这个属性有很多限制，主要限制如下：

1. 父元素必须含有 `line-height`，子元素中的 `vertical-align` 才能起作用。
2. 被使用 `vertical-align` 属性的元素必须是行内元素（`inline-`或者 `table-cell`），主要包括 `span`, `img`,`input`, `button`, `td`
3. `middle` 属性并不是完全的居中位置，实际上是中间偏下一点，更多类似于四线三格中的第三线的位置

![](https://pic.imgdb.cn/item/622595c35baa1a80abf61f2e.jpg)

因此父子元素都为块级的情况下设置的方法如下：

```css
.container {
  line-height: 500px;
}
.center {
  height: 200px;
  display: inline-block;
  vertical-align: middle;
}
```

#### 5. flex 或 grid

不讲了

### 全部居中

（下面假设#main 是要居中的元素的选择器）

1. 可以通过将元素的左上角通过 top:50%和 left:50%定位到页面的中心，然后再通过 translate 来调整元素的中心点到页面的中心。该方法需要考虑浏览器兼容问题。

```css
#main {
  width: 200px;
  height: 200px;
  background-color: red;

  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(
    -100px,
    -100px
  ); /*这个值就是元素宽高的一半，相当于把元素从左上角对准中点变成中心对准中点*/
}
```

2. 类似上面的，使用-margin 的形式也可以将元素拉到中心位置

```css
#main {
  width: 200px;
  height: 200px;
  background-color: red;

  position: absolute;
  top: 50%;
  left: 50%;
  margin: -100px, 0, 0, -100px;
}
```

3. 还可以使用设置四个方向的值都为 0，并将 margin 设置为 auto，由于宽高固定，因此对应方向实现平分，可以实现水平和垂直方向上的居中。

```css
#main {
  width: 200px;
  height: 200px;
  background-color: red;

  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  margin: auto;
}
```

4. flex 布局，这就不讲了。

## div 垂直居中，左右 10px，高度始终为宽度一半

> 问题描述: 实现一个 div 垂直居中, 其距离屏幕左右两边各 10px, 其高度始终是宽度的 50%。

给这个 div 添加属性，利用`padding-bottom: 50%`和`height:0`；因为高度为 0，因此整个元素的高度被 padding-bottom 撑起；又因为这个值是该 div 宽度的一半，因此就形成了高度是宽度的一半效果。
代码如下：

```html
<body>
  <div class="main">
    <div class="block">A</div>
  </div>
</body>
<style>
  .main {
    height: 100vh;
    display: flex;
    align-items: center;
  }
  .block {
    background-color: red;
    margin: 0 10px;
    width: 100%;

    padding-bottom: 50%;
    height: 0;
  }
</style>
```

同时这个方法也适用于任何大小的固定宽高比元素，只需要改动 padding-bottom 值和宽度即可。

---

现代的解决方案是 aspect-ratio 属性 参考https://developer.mozilla.org/zh-CN/docs/Web/CSS/aspect-ratio

```css
.block {
  aspect-ratio: 1 / 2;
  width: auto;
  margin: 0 10px;
}
```

## 响应式布局（八股）

八股参考：https://juejin.cn/post/6844903814332432397

### 什么是响应式布局

指的是同一页面在不同屏幕尺寸下有不同的布局。传统的开发方式是 PC 端开发一套，手机端再开发一套，而使用响应式布局只要开发一套就够，缺点就是 CSS 比较重。

响应式开发一套界面，通过检测视口分辨率，针对不同客户端在客户端做代码处理，来展现不同的布局和内容；

### 如何实现（概述）

1. 媒体查询

关键点：

- 选择屏幕大小分割点 （参考 bootstrap 的 sm、xs 等）
- 移动优先（min-width）/pc 优先（max-width）

2. 百分比布局
3. rem 布局，当页面的 size 发生变化时，只需要改变 font-size 的值，那么以 rem 为固定单位的元素的大小也会发生响应的变化。可利用插件（webpack 或 postcss）转换 px 和 rem
4. vh/vw/vmin/vmax 的视口单位
5. 图片响应式（详见上）
6. flex、grid

## 两栏布局

1. 浮动方式
   左边左浮动，右边设置 margin-left 即可。主要是防止浮动盖住右边元素
   或者设置右边元素为 BFC 也可以

```html
<body>
  <div class="container">
    <aside>aside</aside>
    <main>main</main>
  </div>
</body>
<style>
  body {
    margin: 0;
  }
  aside {
    float: left;
    width: 500px;
    background-color: red;
    height: 100vh;
  }
  main {
    margin-left: 500px;
    /*display: flex;*/
    background-color: green;
    height: 100vh;
  }
</style>
```

同理右边浮动也可以，左边处理方式一样，设置 margin-right 或 BFC 都可。

2. flex、grid

flex 需要注意右边设置 flex:1 或者明确指定宽度

3. position 定位

```css
aside {
  position: absolute;
  left: 0;
  width: 500px;
  background-color: red;
  height: 100vh;
}
main {
  margin-left: 500px;
  background-color: green;
  height: 100vh;
}
```

## 三栏布局

1. float 布局
   **注意三个元素的 html 位置**，中间元素在最后。
   如果把中间元素放在中间，它会默认充满一行（即使宽度不足，margin 也会填充），右浮动元素就会因为这一行没有足够宽度而被挤下去。
   如果设置中间元素为 inline（即右边必然有足够空间），右浮动就会正常。

```html
<body>
    <div class="container">
      <aside class="side1">aside1</aside>
      <aside class="side2">aside2</aside>
      <main>main</main>
    </div>
  </body>
  <style>
    aside{
      width: 500px;
      background-color: red;
      height: 100vh;
    }
    main{
      margin: 0 500px;
      height: 100vh;
      background-color: green;
    }
    .side1{
      float: left;
    }
    .side2{
      float: right;
    }
```

2. position
   其实和浮动差不多，就是把浮动改成 position: absolute;就可以

```css
.container {
  position: relative;
}
aside {
  width: 500px;
  background-color: red;
  height: 100vh;
}
main {
  height: 100vh;
  background-color: green;
  margin: 0 500px;
}
.side1 {
  position: absolute;
  left: 0;
}
.side2 {
  position: absolute;
  right: 0;
}
```

3. 圣杯和双飞翼，参考上面的

4. flex、grid
