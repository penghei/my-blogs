---
title: canvas学习
date: 2022-01-28 14:28:26
tags: 日常学习
cover: /img/canvas.jpg
categories: 数据可视化相关
---

# canvas 基本用法

canvas 主要通过 js 绘制图像，可以配合 dom 事件和 requestAnimationFrame 完成一些游戏、动画等；同时 canvas 有一个 imageData api，可以对图片做很多处理
canvas 标签的形式：

```html
<canvas width="500" height="500" id="mycanvas"></canvas>
```

canvas 和其他块级元素差不多，在坐标系以及元素单位上和 svg 一致；因此上面的标签表示宽高为 500px 的 canvas 元素

canvas 进行操作的核心是**上下文**，在 2d 图像中是 2d 上下文，在 webGL 环境下还有 3d

```js
const ctx = canvas.getContext("2d");
```

canvas 的绝大部分绘图操作都在上下文上进行，而不是直接的 dom 元素 canvas
另外，在调用 ctx 之前最好检查是否支持 canvas 上下文，因此一般的写法如下：

```js
if (canvas.getContext) {
  //...
}
```

# canvas 绘制形状

canvas 绘制图像只有两种形式：矩形 rect 和路径 path，除了矩形之外的任何图形都可以用 path 绘制，当然也可以像 svg 的 path 一样填充

## 绘制矩形

绘制矩形的主要代码如下：

```js
function drawRect() {
  if (!canvas.getContext) return;
  const ctx = canvas.getContext("2d");
  ctx.fillStyle = "blue"; //先规定颜色再绘制
  ctx.fillRect(10, 10, 100, 100);
  ctx.clearRect(50, 50, 40, 40);
  ctx.strokeRect(50, 50, 30, 30);
}
```

主要的 api 有三个：

- `fillRect(x, y, width, height)`：绘制一个填充的矩形
- `strokeRect(x, y, width, height)`：绘制一个矩形的边框
- `clearRect(x, y, width, height)`：清除指定矩形区域，让清除部分完全透明。

另外在 fillRect 正式绘制之前，还可以对矩形区域进行操作：

- fillStyle：规定填充样式，除了颜色还有
- strokeStyle：规定边框样式
- transform：相关的如 translate/rotate 等变换函数

这些 api 都要在 fillRect 之前调用，否则绘制出来之后再更改就无效了

## 绘制路径

### 基本绘制（line）

代码如下：

```js
function drawPath() {
  if (!canvas.getContext) return;
  const context = canvas.getContext("2d");
  /**不填充三角形，需要closePath并用stroke绘制边框 */
  context.beginPath();
  context.moveTo(0, 0);
  context.lineTo(0, 200);
  context.lineTo(200, 200);
  context.closePath();
  context.stroke();
  /**填充三角形，会自动closePath，根据需要添加边框 */
  context.beginPath();
  context.moveTo(210, 0);
  context.lineTo(10, 0);
  context.lineTo(210, 200);
  context.fill();
}
```

绘制步骤：

1. 创建路径起始点
1. 使用画图命令去画出路径。
1. 把路径封闭。
1. 通过描边或填充路径区域来渲染图形。

具体来说就是：

1. 首先在任何路径开始之前都需要`beginPath()`新建一条路径，生成之后，图形绘制命令被指向到路径上生成路径。
1. 使用`moveTo`移到起始点位，这点和 svg 中 path 元素的 M 命令相同
1. 使用`lineTo`开始画线，参数分别为 x、y 坐标
1. 如果是填充，直接`fill()`即可自动闭合路径；否则需要`closePath()`闭合
1. 用`stroke()`绘制边框，填充元素可以不需要，而纯路径元素必须，否则无法显示

> 注意 fill()和 fillRect()的区别，前者是用在 path 的填充中的，后者用在直接绘制矩形中

---

### 绘制圆弧

画线的 api 除了 lineTo 之外还有 arc，用于绘制圆弧

```js
function drawCircle() {
  if (!canvas.getContext) return;
  const context = canvas.getContext("2d");
  context.beginPath();
  context.moveTo(200, 100);
  context.arc(100, 100, 100, 0, Math.PI * 0.5);
  context.fill();
}
```

api 为：

```
arc(x, y, radius, startAngle, endAngle, anticlockwise)
```

参数分别是：圆心 x 坐标、圆心 y 坐标、半径、起始角度（弧度表示）、结束角度（弧度表示）、是否逆时针（为 true 时是逆时针）

注意圆心坐标和起始点 moveTo 并不冲突，可以在起始点为任意的情况下绘制圆心在确定点的圆，结果可能不是一个标准扇形；

---

### 贝塞尔曲线

canvas 的贝塞尔曲线画法如下：

```js
function drawBezier() {
  if (!canvas.getContext) return;
  const context = canvas.getContext("2d");
  context.beginPath();
  context.moveTo(0, 0);
  context.quadraticCurveTo(100, 0, 100, 100);
  //   context.bezierCurveTo(cp1x, cp1y, cp2x, cp2y, x, y)
  context.stroke();
}
```

和 svg 类似，canvas 提供了二三次贝塞尔曲线的绘制，参数和 svg 的也类似，分别表示控制点 1、(控制点 2)、终点

### 矩形

path 同样可以绘制矩形，api 为：

```
rect(x, y, width, height)
```

和`context.fillRect()`的参数相同，但是是绘制路径画法，因此需要和其他路径一样，先 beginPath()，再 moveTo()，然后才能开始绘制

### Path2D

可以直接创建一个 path2D 对象，然后对这个对象 new 出的实例进行类似的操作

```js
function drawPath2D() {
  if (!canvas.getContext) return;
  const context = canvas.getContext("2d");

  const rec = new Path2D();
  rec.rect(0, 0, 100, 200);
  context.stroke(rec);

  const cir = new Path2D();
  cir.moveTo(50, 50);
  cir.arc(50, 50, 50, 0, Math.PI * 1.5);
  context.fill(cir);
}
```

另外 path2D 的参数可以是 svg path 的 d 参数字符串，也就是可以直接应用 svgpath 的绘制

```js
const svgPath = new Path2D("M10 10 h 80 v 80 h -80 Z");
context.stroke(svgPath);
```

Path2D 的特点是可以进行图案的组合，即`addPath`方法，可以添加一个路径到另一个路径

```js
Path2D.addPath(path [, transform])​
```

其中 transform 是一个二维数组，作为变换矩阵，即控制被组合的图案的位置等。
比如可以先用 ctx.rect 创建一个矩形，再用 ctx.circle 创建一个圆；这两个对象都是用`new Path2D()`创建而来。然后把圆的 path2D 对象用 addPath 方法添加到矩形上，再直接绘制该矩形就可以得到两者结合之后的图案

```js
function draw() {
  if (canvas.getContext) {
    const ctx = canvas.getContext("2d");

    const rectangle = new Path2D();
    rectangle.rect(10, 10, 50, 50);

    const circle = new Path2D();
    circle.moveTo(125, 35);
    circle.arc(100, 35, 25, 0, 2 * Math.PI);

    rect.addPath(circle);
    ctx.stroke(rectangle);
  }
}
```

## 样式

### 基本样式

如果想要给图形上色，有两个重要的属性：fillStyle 和 strokeStyle。

- `fillStyle = color`：设置图形的填充颜色。
- `strokeStyle = color`：设置图形轮廓的颜色。

color 和 svg 的相同，都是任意 css 颜色

> 一旦设置了 strokeStyle 或者 fillStyle 的值，这个新值就会成为新绘制的图形的默认值。如果你要给每个图形上不同的颜色，需要重新设置 fillStyle 或 strokeStyle 的值。

- `globalAlpha = transparencyValue`：影响到 canvas 里**所有图形**的透明度

如果设置单个图形透明度，可以再 rgba 中设置

### 线条样式

可以通过一系列属性来设置线的样式。

- `lineWidth = value`
  设置线条宽度，单位 px
- `lineCap ="butt | round | square"`
  设置线条末端样式。
- `lineJoin = "round | bevel | miter"`
  设定线条与线条间接合处的样式。

这三个样式和 svg 中的样式相似，分别规定宽度、末端、交接样式

- `miterLimit = value`
  限制当两条线相交时交接处最大长度；所谓交接处长度（斜接长度）是指线条交接处内角顶点到外角顶点的长度。
- `getLineDash()`
  返回一个包含当前虚线样式，长度为非负偶数的数组。
- `setLineDash([m,n])`
  设置当前虚线样式。和 svg 的 stroke-dasharray 参数相同
- `lineDashOffset = value`
  设置虚线样式的起始偏移量，和 stroke-dashoffset 相同

### pattern

```
createPattern(image, type)
```

- image 可以是一个 Image 对象的引用，或者另一个 canvas 对象。
- Type 必须是下面的字符串值之一：repeat，repeat-x，repeat-y 和 no-repeat。

应用如下:

```js
function drawPattern() {
  if (!canvas.getContext) return;
  const context = canvas.getContext("2d");

  const img = new Image();
  img.src = "https://mdn.mozillademos.org/files/222/Canvas_createpattern.png";
  img.onload = () => {
    // 创建图案
    const ptrn = context.createPattern(img, "repeat");
    context.fillStyle = ptrn;
    context.fillRect(0, 0, 150, 150);
  };
}
```

创建 pattern 对象后通过 fillStyle 赋给上下文，然后创建路径或 rect 均可

# canvas 图像

## drawImage

canvas 可以对图像进行很多操作，操作的基础是创建 img 实例

```js
const img = new Image();
img.src = "...";
img.onload = () => {
  //...
};
```

在 onload 的回调函数中主要执行一个重要的 api：drawImage

```js
drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight);
```

drawImage 参数：

- `image`：图片源，即 canvas 可以绘制的对象，称作 CanvasImageSource 对象。主要包括以下几种：
  - `HTMLImageElement`：即图片对象，可以是<img>元素，也可以是通过`new Image()`创建的图片对象。一般如果想更灵活控制绘制的话（比如上传图片绘制），选用后者较多
  - `HTMLVideoElement`：视频帧绘制，这里可以是直接的<video>元素，方法见下
  - `HTMLCanvasElement`：即另一个 canvas 对象
- 后面的参数，具体如下

```js
drawImage(image, dx, dy);
drawImage(image, dx, dy, dWidth, dHeight);
drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight);
```

区别：

![](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/drawImage/canvas_drawimage.jpg)

- dxdy 指绘制的图片相对于 canvas 坐标系的位置，即相对于 canvas 原点的位置。通常选择(0,0)即从左上角原点开始绘制
- dWidth, dHeight 表示要绘制多大的图片。如果不设置这两项，绘制时就会按照图片原尺寸绘制，有可能导致图片超出范围等情况
- sxsy 可选，但是要和 swidth 和 sheight 同时出现；表示对应图片的裁剪。上面的 dxdy 是相对于 canvas 坐标系的位置，而 sxsy 则是**相对于图片的位置**。控制这四个值就可以实现不缩放图片的裁剪

drawImage 实际上是绘制了一个“背景”，在 drawImage 绘制之后还可以继续在 image 上绘制图案，比如下面的就是在一个图片上绘制折线图

```js
function drawPics() {
  const context = canvas.getContext("2d");
  /**可以直接在已有的图像上绘制，调用drawImage然后正常绘制即可 */
  const img = new Image();
  img.onload = () => {
    //drawImage五个参数：image、x、y、宽、高，其中后两个是类似svg的viewPoint放缩
    context.drawImage(img, 0, 0);
    context.beginPath();
    context.moveTo(40, 90);
    context.lineTo(70, 66);
    context.lineTo(103, 76);
    context.lineTo(170, 15);
    context.stroke();
  };
  img.src = "https://mdn.mozillademos.org/files/5395/backdrop.png";
}
```

---

drawImage 还可以用于视频关键帧的绘制，直接把视频 dom 元素放入 img 参数即可绘制。绘制的结果就是当前视频正在播放的那个帧
比如这样：

```js
export default (props) => {
  const canvasRef = useRef(null);
  const canvasCtx = useRef(null);
  const videoRef = useRef(null);
  useEffect(() => {
    if (canvasRef) {
      canvasCtx.current = canvasRef.current.getContext("2d");
    }
  }, []);

  const drawVideoFrame = () => {
    const ctx = canvasCtx.current;
    ctx.drawImage(
      videoRef.current, // 直接传入video元素的html对象
      0,
      0,
      canvasRef.current.width,
      canvasRef.current.height
    );
  };
  return (
    <div style={{ border: "1px solid" }}>
      <video
        src="http://clips.vorwaerts-gmbh.de/big_buck_bunny.mp4"
        ref={videoRef}
        autoPlay
        controls
      ></video>
      <canvas width={500} height={500} ref={canvasRef}></canvas>
      <button onClick={drawVideoFrame}>draw</button>
    </div>
  );
};
```

这里我每次点击 draw 按钮时，就会绘制当前视频正在播放的那个帧。如果视频一直播放，就会绘制不同的帧。

## imageData 对象

ImageData 对象中存储着 canvas 对象真实的像素数据，它包含以下几个只读属性：

- `imageData.width`：图片宽度，单位是像素
- `imageData.height`：图片高度，单位是像素
- `imageData.data`：Uint8ClampedArray（8 位无符号整型固定数组） 类型的一维数组，包含 RGBA 格式的整型数据，范围在 0 至 255 之间（包括 255），四个值分别为 rgba 参数的四个值，可以用于取色。

> 这个数组实际上包含图片的每个像素点以及其 rgba 数据，每个像素用 4 个 1byte 值表示。比如 data[0]~data[3]这四个值就分别代表第一个像素的 r、g、b、a 值，依次类推直到最后一个像素。
> 因此该数组的大小(length)就是图片的`高*宽*4`（单位都是像素）；通过这个数组，就可以精确读取每个像素点的属性。
> 每一行的像素个数为`imageData.width*4`
> 每一行从左到右，到第 n 列的的像素个数为`n*4`
> 因此对于任意一个位置的元素（m 行 n 列），其在数组中的位置应该是
>
> ```
> imageData.data[((m * (imageData.width * 4)) + (n * 4)) + 0/1/2/3]
> ```
>
> 其中`+ 0/1/2/3`就表示该像素对应的 rgba 四个值

### getImageData

使用 imageData 处理现有图像的方法是`getImageData()`api

```js
const myImageData = ctx.getImageData(left, top, width, height);
```

> 代表了画布区域的对象数据，此画布的四个角落分别表示为(left, top), (left + width, top), (left, top + height), 以及(left + width, top + height)四个点。这些坐标点被设定为画布坐标空间元素。

可以用此写出一个颜色选择器：

```js
const canvas = document.getElementById("canvas");
window.onload = imageData();

/*  创建image对象在canvas中
    注意一定是canvas才能使用，普通img元素不能用getImageData 
*/
function imageData() {
  const img = new Image();
  img.src = "./rhino.png";
  const context = canvas.getContext("2d");

  img.onload = () => {
    context.drawImage(img, 0, 0);
  };
}

canvas.addEventListener("click", (e) => {
  const context = canvas.getContext("2d");
  const pixel = context.getImageData(e.layerX, e.layerY, 1, 1); //表示当前点击点的1px*1px大小区域
  const data = pixel.data;
  //data元素的4项正好是rgba的四个参数，因此可以直接用作元素颜色color
  const rgba = `rgba(${data[0]}, ${data[1]}, ${data[2]}, ${data[3] / 255})`;

  document.getElementById("display").style = `background:${rgba}`;
});

function getCanvasPic() {
  const dataUrl = canvas.toDataURL();
  document.getElementById("myImg").src = dataUrl;
}
```

这里最后一个`toDataURL()`可以把当前 canvas 图像转成图片，返回值是 base64 格式，默认为 png 格式；可以根据参数设定为 jpg 图片

```js
canvas.toDataURL("image/jpeg", quality);
```

### putImageData

可以写入提取出来或者创建出的 imageData 对象

```js
ctx.putImageData(myImageData, dx, dy);
```

putImageData 和 getImageData 交替使用，就可以实现对 canvas 上已绘制内容的保存（注意和 save 不一样，save 保存的是样式、变换等状态）

## 图像处理

既然可以获得图片每个像素的 rgba 值，就可以通过改变其值来控制其灰度等属性

比如，灰化，就是取每个像素点 rgb 值的平局值，然后把 rgb 值都变成这个平均值：

```js
const imageData = canvasCtx.current.getImageData(0, 0, width, height);
const data = imageData.data;
for (let i = 0; i < data.length; i += 4) {
  const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;
  data[i] = avg; // red
  data[i + 1] = avg; // green
  data[i + 2] = avg; // blue
}
canvasCtx.current.putImageData(imageData, 0, 0);
```

负片，即对每个像素点的 rgb 值取反（255-该值）

```js
const imageData = canvasCtx.current.getImageData(0, 0, width, height);
const data = imageData.data;
for (let i = 0; i < data.length; i += 4) {
  data[i] = 255 - data[i]; // red
  data[i + 1] = 255 - data[i + 1]; // green
  data[i + 2] = 255 - data[i + 2]; // blue
}
canvasCtx.current.putImageData(imageData, 0, 0);
```

以及还有各种滤镜，都是这样的思路

## 缩放

通过裁剪图片的某个位置并单独绘制裁剪结果，就可以得到放大的图片。比如淘宝商品界面的放大部分图片，把鼠标移到图片上就可以看到鼠标附近位置的放大结果。
这个的实现就是通过裁剪鼠标位置周边的部分图像，然后单独 drawImage 到旁边的 canvas 上即可。

比如下面这个例子，把图片周围上下 5 像素的部分单独绘制到一个 100\*100 的 canvas 上，相当于给这个部分放大了 100 倍：

```js
const scaleHandleMouseMove = () => {
  // 这个事件安插在被放大的canvas的onMouseMove上
  const [mouseX, mouseY] = mousePos; // 鼠标位置
  const [top, bottom, left, right] = canvasEdge.current; // 被放大canvas的边界
  console.log(mouseX);
  const scaledCtx = scaledCanvasRef.current.getContext("2d");
  scaledCtx.drawImage(
    canvasRef.current,
    mouseX - 5 - left, // 鼠标左5像素作为裁剪图片的起点
    mouseY - 5 - top, // 上5像素
    10, // 裁剪10个像素，相当于鼠标左右10个像素的大小
    10,
    0,
    0,
    100, // 放大到100*100的canvas上
    100
  );
};
```

## 图片的保存

HTMLCanvasElement 提供两个方法，可以把 canvas 的绘制结果转化为 Base64 格式和 blob 格式。

```js
canvas.toDataURL("image/jpeg", quality);
canvas.toBlob(callback, type, encoderOptions);
```

如果要下载图片，可以通过`URL.createObjectURL(blob)`为该 blob 创建一个 url 并安插在一个`<a>`标签上，然后`click()`点击该标签即可下载 canvas 绘制的图片。详见 blob 的处理部分

# canvas 变换

在变换之前有两个很重要的 api

- `save()`：保存画布(canvas)的所有状态
- `restore()`：save 和 restore 方法是用来保存和恢复 canvas 状态的，都没有参数。Canvas 的状态就是当前画面应用的所有样式和变形的一个快照。

---

然后是 canvas 变换
变换主要有 4 个 api，都是对上下文操作的，必须先变换再绘制才能生效，和 css 的 transform 相同

```js
function transforms() {
  const context = canvas.getContext("2d");

  context.translate(100, 100); // 先变换再绘制
  context.rotate((Math.PI / 180) * 60);
  context.scale(2, 0.5);
  context.transform(a, b, c, d, e, f); // 变换矩阵，六个值分别对应矩阵横数前6个数

  context.fillRect(0, 0, 50, 30);
}
```

这是一个绘制 3\*3 个正方形的例子：

```js
function draw() {
  const ctx = document.getElementById("canvas").getContext("2d");
  for (let i = 0; i < 3; i++) {
    for (let j = 0; j < 3; j++) {
      ctx.save();
      ctx.translate(10 + j * 50, 10 + i * 50);
      ctx.fillRect(0, 0, 25, 25);
      ctx.restore();
    }
  }
}
```

## 转换矩阵

如图所示：

![](https://pic.imgdb.cn/item/6429dce4a682492fcc55bc00.jpg)

使用getTransform可以获取这个矩阵（一个对象），通过setTransform可以设置矩阵。
这几个值的含义为：

a: 水平缩放。
b: 垂直倾斜。
c: 水平倾斜。
d: 垂直缩放。
e: 水平移动。
f: 垂直移动。

即使不是通过setTransform修改的变化，也可以通过getTransform获取。这点在获取canvas变换上非常方便。

# canvas 合成和剪切

- 合成`globalCompositeOperation`用于控制多个 canvas 图形绘制在一起时的重叠情况。默认时就是一个覆盖另一个，可以通过设置这个属性为不同的值来控制。
  比如让两个图形重合处透明、只绘制重合处等。详见`https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Compositing#%E8%A3%81%E5%88%87%E8%B7%AF%E5%BE%84`

> 注意，这个“合成”不只是两个 canvas 合成，还包括在同一个 canvas 上绘制的图像的合成

- 裁切`clip()`可以把最近的一次绘制的路径作为裁剪路径，后面绘制的图案都会被该路径裁剪掉。
  比如可以先创建一个圆形路径，然后调用 clip 转化成裁剪路径。后面的绘制就都会被这个路径裁掉外面的部分，只会保留该路径内部的部分。
  比如在绘制上传的图片之前用一个圆形裁剪一下，这也是某些网站上传图片时的常见操作：

```js
canvasCtx.current.arc(50, 50, 50, 0, Math.PI * 2, true);
canvasCtx.current.clip(); // 用上面那个圆创建了一个裁剪路径
canvasCtx.current.drawImage(img, 200, 200, 100, 100, 0, 0, width, height);
```

裁切和合成都属于 canvas 的“状态”，也就是和变换一样，可以被`save()`保存。裁切路径可以保存，再创建新的裁切路径，不会影响到之前的。

# canvas 动画

canvas 动画和 js+css 绘制动画类似，通过上面介绍的变换，以及 requestAnimationFrame 等 api 可以产生动画
关于 canvas 的动画水很深，更详细的部分可以参考<a href="https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Basic_animations">MDN 文档</a>
这里展示一个最基本的动画

```js
const canvas = document.getElementById("canvas");

function animation() {
  const context = canvas.getContext("2d");

  context.clearRect(0, 0, 500, 500); //先清除

  context.fillStyle = "red";
  context.translate(1, 1); //变换，考虑到60帧，要选择合适的值

  context.fillRect(0, 0, 100, 200);

  requestAnimationFrame(animation);
}
requestAnimationFrame(animation);
```

要注意的是，canvas 动画在绘制新一帧之前一定要清除之前帧，表现为调用`context.clearRect(0, 0, canvas.height,canvas.width);`，否则绘制出来的图像是连在一起的

动画可以通过 canvas 自带的 translate/rotate 等，也可以通过不断更改新绘制的位置实现，后者在一些动画中更容易实现（比如小球弹跳等效果）

# canvas 优化

## canvas 的性能问题

影响 canvas 性能的主要有两个因素：

- 「 绘制图形的个数 」。类似于 dom 元素过多的情况，过多的图形将会给 canvas 带来极大的负担，导致其不能及时响应之后的绘制任务或交互。
- 「 绘制图形的大小 」。越大的图形，一次绘制需要的像素点就越多。不过图形大小的影响不如数量的影响。

因此优化 canvas，最主要的就是这两个方面。主要的思想就是：

- 减少绘制次数，降低绘制频率，将绘图指令集中化。
- 将绘制过程变为非阻塞，或者缓存绘制过程。

## 优化方式

canvas 常见的优化方式：

1. 减少浮点数坐标，对坐标取整

绘制图形或调用 drawImage 这样的函数时，如果使用的是浮点数作为参数，浏览器为了达到抗锯齿的效果会做额外的运算。

```js
ctx.drawImage(myImage, 0.3, 0.5);
```

2. 使用多层 canvas 绘制一个复杂的场景。如果针对动画或者游戏的场景，有些内容可能保持静止，有些则需要频繁渲染。可以选择把静止的内容和频繁运动的内容分离到不同层的 canvas 上去。

```html
<div id="stage">
  <canvas id="ui-layer" width="480" height="320"></canvas>
  <canvas id="game-layer" width="480" height="320"></canvas>
  <canvas id="background-layer" width="480" height="320"></canvas>
</div>

<style>
  #stage {
    width: 480px;
    height: 320px;
    position: relative;
    border: 2px solid black;
  }
  canvas {
    position: absolute;
  }
  #ui-layer {
    z-index: 3;
  }
  #game-layer {
    z-index: 2;
  }
  #background-layer {
    z-index: 1;
  }
</style>
```

除了垂直分层之外，同层也可以。举个例子，假设我们现在需要实现 Web 端 VsCode，而整个界面都是由 Canvas 绘制。我们可以将 VsCode 拆分成几个区域：顶部栏、左侧栏、底部栏、编辑区。显然这个几个区域的变更频率、触发变更的前提都不一致，我们可以将其做拆分。

3. 将画布的函数调用集合到一起（例如，画一条折线，而不要画多条分开的直线）。这点类似于集中 dom 操作，都是为了将绘制过程合并。

4. 提高像素。如果 canvas 在高清屏幕上绘制，可能会因为分辨率更高而导致出现模糊的情况。解决方案有两个：

- 通过 ctx.scale 缩放 canvas。即获取 window.devicePixelRatio，然后把 canvas 的宽高乘上分辨率，再使用 scale 缩放回去，就相当于放大了像素密度。

```js
var canvas = document.getElementById("canvas");
var ctx = canvas.getContext("2d");
var scale = window.devicePixelRatio;
canvas.width = canvas.width * scale;
canvas.height = canvas.height * scale;
ctx.scale(scale, scale);
```

- 同样是缩放，除了 scale 之外还可以使用 css 对 canvas 元素进行缩放，比如修改 canvas 元素的宽、高，然后在 css 中再修改 style 属性的宽高。

```js
var canvas = document.getElementById("canvas");
var ctx = canvas.getContext("2d");
canvas.width = 500; // 设计分辨率
canvas.height = 500;
canvas.style.width = "250px"; // 显示分辨率
canvas.style.height = "250px";
```

5. 离屏渲染。对于离屏渲染的概念，大多数情况是指：使用一个不可见（或是屏幕外）的 Canvas 对即将渲染的内容的某部分进行提前绘制，然后频繁地将屏幕外图像渲染到主画布上，避免重复生成该部分内容的步骤。

比如，提前绘制好某个图像，在画布更新的时候直接使用该图像.

```js
const offscreenCtx = offscreenCanvas.getCanvas("2d");
offscreenCtx.beginPath();
offscreenCtx.arc(250, 250, 100, 0, 2 * Math.PI);
offscreenCtx.fillStyle = "red";
offscreenCtx.fill();

//...

const ctx = canvas.getCanvas("2d");
ctx.drawImage(offscreenCanvas, 0, 0);
```

对于需要频繁绘制的图形，这种形式相当于封装了绘制过程。离屏渲染的 canvas 相当于一种缓存，减少了在 canvas 上的重复绘制。

在 konva 的 Shape 类的绘制方法中，即对于每个元素的绘制，都采用了离屏渲染的方式。把绘制函数先在 bufferCanvas 上绘制，再把 bufferCanvas 绘制到 sceneCanvas 上。

```js
// Stage
_buildDOM() {
  this.bufferCanvas = new SceneCanvas({
    width: this.width(),
    height: this.height(),
  });
  // ...
}

// Shape
if (this._useBufferCanvas() && !skipBuffer) {
  stage = this.getStage();
  bufferCanvas = stage.bufferCanvas;
  bufferContext = bufferCanvas.getContext();
  bufferContext.clear();
  bufferContext.save();
  bufferContext._applyLineJoin(this);
  // 调用绘制函数，绘制到离屏的canvas上
  drawFunc.call(this, bufferContext, this);
  bufferContext.restore();
  var ratio = bufferCanvas.pixelRatio;
  if (hasShadow) {
    context._applyShadow(this);
  }
  context._applyOpacity(this);
  context._applyGlobalCompositeOperation(this);
  context.drawImage(
    bufferCanvas._canvas,
    0,
    0,
    bufferCanvas.width / ratio,
    bufferCanvas.height / ratio
  );
}
```

离屏渲染在 canvas 动画，或者用 canvas 执行大量绘制任务时比较有用。

除此之外离屏渲染还可以实现预渲染。比如一个小说软件，这一页的小说在显示时，可以预渲染前后几页的canvas，使得翻页时可以立即加载。

6. 只绘制可视区域

可以用一个 container 包裹 canvas 画布，使这个 html container 的大小比 canvas 更小。这样 canvas 就会表现出“溢出”，可以采用拖拽或滚动的方式移动 canvas 从而只显示可视区域。

```html
<!-- HTML代码 -->
<div id="canvas-container">
  <canvas id="my-canvas"></canvas>
</div>

<style>
  /* container比canvas小 */
  #canvas-container {
    width: 500px;
    height: 500px;
    overflow: scroll;
  }

  #my-canvas {
    width: 1000px;
    height: 1000px;
  }
</style>

<script>
  const canvas = document.getElementById("my-canvas");
  const ctx = canvas.getContext("2d");

  // 绘制需要显示的内容
  ctx.fillStyle = "red";
  ctx.fillRect(0, 0, 1000, 1000);

  // 监听用户的滚动和拖动事件
  const container = document.getElementById("canvas-container");
  let offsetX = 0;
  let offsetY = 0;

  container.addEventListener("touchmove", (e) => {
    e.preventDefault();
    offsetX += e.touches[0].clientX - e.touches[0].clientX;
    offsetY += e.touches[0].clientY - e.touches[0].clientY;
    draw();
  });

  // 绘制可视区域内的内容
  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.save();
    ctx.translate(-offsetX, -offsetY);
    ctx.beginPath();
    // 这里可以把rect范围稍微大一些，增加渲染范围，防止拖拽时显示空白
    ctx.rect(offsetX, offsetY, container.clientWidth, container.clientHeight);
    ctx.clip();
    ctx.fillStyle = "blue";
    ctx.fillRect(0, 0, 1000, 1000);
    ctx.restore();
  }
</script>
```

7. 减少对上下文的操作，防止非法赋值。

所谓上下文操作，就是对context进行赋值。比如`context.font`这个属性的赋值消耗非常大
，类似的还有line相关、fill相关等常用的

如果是非法赋值，那么消耗会更大。所谓非法赋值就是没有按照格式赋值，比如把一个对象赋给lineWidth。

因此我们可以通过适当地安排调用绘图 API 的顺序，降低 context 状态改变的频率。

8. 非阻塞渲染

和dom操作一样，js代码也会阻塞canvas的绘制。当然这点和其他js优化一样，可以通过webworker、时间分片等方式减少js代码造成的阻塞。

# canvas 其他

## canvas绘制原理

之前面试被问到一个问题：canvas 绘制和 dom 绘制有什么不同，以及 canvas 绘制原理是什么
感觉应该是这样：

先说原理：canvas 绘制原理本质上和 dom 元素是差不多的，都是经过浏览器的绘制、分层、合成过程。可以理解为 canvas 就是一块固定的元素，浏览器在其上执行绘制，和绘制其他 dom 元素一样。

canvas 元素和其他 dom 元素并不属于一个图层（合成层）。因此canvas上的绘制过程是由gpu完成的。也就是说，canvas自己的绘制并不会触发浏览器的重绘重排，而是类似gpu加速的dom元素一样，直接由gpu控制绘制。

但是有特例，即如果修改canvas的css样式，比如canvas的大小、可见等属性时，仍然会导致重绘重排的发生。不过使用transform改变canvas样式可以避免重绘重排。

canvas 本质上是像素点位图，它内部的绘制不需要像其他 dom 元素一样通过 layout 计算得出位置，而是直接在内部的像素点上绘制。因此相对于常规dom元素效率更高、性能更强。

## canvas放大缩小

放大缩小的方式有几种：

- 修改canvas样式。即修改canvas的尺寸，比如把canvas的宽度加倍。这种方法很不好，首先会导致分辨率降低，其次是当canvas扩大时，需要绘制的范围更大，对性能也有影响。
- 通过ctx.scale放缩。这是一种canvas原生的方式，缺点是可能写起来比较麻烦

注意scale放缩的是canvas的坐标系，也就是说放缩之后还是需要重新渲染的。一个简单的例子如下：

```js
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");
// 绘制初始地图
drawMap();
// 监听鼠标滚轮事件
canvas.addEventListener("wheel", (e) => {
  e.preventDefault();
  // 计算当前放大比例
  const delta = e.deltaY > 0 ? -0.1 : 0.1;
  const scale = Math.max(0.1, Math.min(2, ctx.getTransform().+delta));
  // 缩放 Canvas
  ctx.setTransform(scale, 0, 0, scale, 0, 0);
  // 清空 Canvas
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  // 重新绘制地图
  drawMap();
});
// 绘制地图
function drawMap() {
}
```

另外，如果需要实现鼠标绘制的功能，那么当地图比例变化时，还需要注意鼠标位置的计算。
具体来说，如果地图放大2倍，那么所有获取到的鼠标坐标就应该除以2，才能使得绘制位置和鼠标位置不会出现偏差。

如果有其他偏移，比如上下左右的位移，那么也需要这样做。比如通过transform向上移动了50，那么实际点位就需要减50。所有移动的值都可以通过getTransform获取。

```js
canvas.addEventListener("mousedown", (e) => {
  const scale = ctx.getTransform().a;
  let startX = e.offsetX / scale;
  let startY = e.offsetY / scale;
  ctx.beginPath();
  ctx.moveTo(startX, startY);
  const moveListener = (e) => {
    const scale = ctx.getTransform().a;
    const mouseX = e.offsetX / scale;
    const mouseY = e.offsetY / scale;
    ctx.lineTo(mouseX, mouseY);
    ctx.stroke();
  };
  canvas.addEventListener("mousemove", moveListener);
  canvas.addEventListener("mouseup", () => {
    canvas.removeEventListener("mousemove", moveListener);
  });
});
```

- drawImage。每次放大相当于使用drawImage重绘了内容，可以使用drawImage截取一部分图片内容，然后绘制到整个canvas上，就相当于放大了。

这个方法的关键是找准放大位置。比如实现鼠标点击某个位置放大，其实就是把鼠标周围一圈的 范围绘制到整个canvas上。

从性能角度来说，当canvas元素比较多时，scale方法可能效率不如drawImage，因为需要反复重新绘制每个元素。相比之下drawImage只是截取一部分重新绘制，对于复杂图形的放缩性能更好。



