---
title: konva学习和原理浅析
date: 2023-03-05 17:23:35
tags: 日常学习
categories: 数据可视化相关
---

# konva 结构

![](https://pic.imgdb.cn/item/64045feef144a010077052c5.jpg)

Konva Tree 主要包括这么几部分：

- Stage 根节点：这是应用的根节点，会创建一个 div 节点，作为事件的接收层，根据事件触发时的坐标来分发出去。一个 Stage 节点可以包含多个 Layer 图层。
- Layer 图层：Layer 里面会创建一个 Canvas 节点，主要作用就是绘制 Canvas 里面的元素。一个 Layer 可以包含多个 Group 和 Shape。
- Group 组：Group 包含多个 Shape，如果对其进行变换和滤镜，里面所有的 Shape 都会生效。
- Shape：指 Text、Rect、Circle 等图形，这些是 Konva 封装好的类。

## 绘制 shape

每个 shape 都是调用其上的\_sceneFunc 方法绘制的，比如 Circle：

```ts
export class Circle extends Shape<CircleConfig> {
  _sceneFunc(context) {
    context.beginPath();
    context.arc(0, 0, this.attrs.radius || 0, 0, Math.PI * 2, false);
    context.closePath();
    context.fillStrokeShape(this);
  }
  getWidth() {
    return this.radius() * 2;
  }
  getHeight() {
    return this.radius() * 2;
  }
  setWidth(width) {
    if (this.radius() !== width / 2) {
      this.radius(width / 2);
    }
  }
  setHeight(height) {
    if (this.radius() !== height / 2) {
      this.radius(height / 2);
    }
  }

  radius: GetSet<number, this>;
}
```

每个具体图形都继承自 Shape 类，Shape 类包含一些基本的方法和属性。在 Shape 和 Node 两个基类上面只负责调用，具体的实现放到具体的 Shape 实现上面。

在 shape 基类中，konva 会获取每个图形的\_sceneFunc 函数，然后放到自己的 drawScene 函数中

```ts
drawScene(can?: SceneCanvas, top?: Node) {
  // basically there are 3 drawing modes
  // 1 - simple drawing when nothing is cached.
  // 2 - when we are caching current
  // 3 - when node is cached and we need to draw it into layer
  var layer = this.getLayer(),
    canvas = can || layer.getCanvas(),
    context = canvas.getContext() as SceneContext,
    cachedCanvas = this._getCanvasCache(),
    drawFunc = this.getSceneFunc(), // 获取绘制函数
    hasShadow = this.hasShadow(),
    stage,
    bufferCanvas,
    bufferContext;
  var skipBuffer = canvas.isCache;
  var cachingSelf = top === this;
  if (!this.isVisible() && !cachingSelf) {
    return this;
  }
  // 从缓存中绘制图形
  if (cachedCanvas) {
    context.save();
    var m = this.getAbsoluteTransform(top).getMatrix();
    context.transform(m[0], m[1], m[2], m[3], m[4], m[5]);
    this._drawCachedSceneCanvas(context);
    context.restore();
    return this;
  }
  if (!drawFunc) {
    return this;
  }
  // 每次绘制前先保存
  context.save();
  // 离屏绘制，在bufferCanvas上绘制
  if (this._useBufferCanvas() && !skipBuffer) {
    stage = this.getStage();
    bufferCanvas = stage.bufferCanvas;
    // 获取canvas context
    bufferContext = bufferCanvas.getContext();
    bufferContext.clear();
    bufferContext.save();
    bufferContext._applyLineJoin(this);
    // layer might be undefined if we are using cache before adding to layer
    var o = this.getAbsoluteTransform(top).getMatrix();
    bufferContext.transform(o[0], o[1], o[2], o[3], o[4], o[5]);
    // 在canvas context上执行绘制函数
    // 注意这里是在bufferContext上渲染的，是一种离屏渲染
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
  } else {
    // 直接在canvas上绘制
    context._applyLineJoin(this);
    if (!cachingSelf) {
      var o = this.getAbsoluteTransform(top).getMatrix();
      context.transform(o[0], o[1], o[2], o[3], o[4], o[5]);
      context._applyOpacity(this);
      context._applyGlobalCompositeOperation(this);
    }
    if (hasShadow) {
      context._applyShadow(this);
    }
    drawFunc.call(this, context, this);
  }
  context.restore();
  return this;
}
```

这里的bufferCanvas其实相当于一个缓冲的作用，类似于预渲染或者预加载的机制。如果允许使用buffer，那么绘制过程不是直接向sceneCanvas绘制，而是先绘制到bufferCanvas上再drawImage上去。

konva中还有其他利用离屏渲染的地方。比如需要主动调用的cache方法，对于某个节点采用cache时，下次渲染则会渲染这个节点的cache版本，而非执行全部绘制。

不过cache常用于复杂图形的绘制，对于简单的图形，cache反而有可能造成内存问题。

参考：https://konvajs.org/docs/performance/Shape_Caching.html


# konva 事件

事件系统是 konva 最具特色的一项。
Konva 里面的事件是在 Canvas 外层创建了一个 div 节点，在这个节点上面接收了 DOM 事件，再根据坐标点来判断当前点击的是哪个 Shape，然后进行事件分发。所以关键就在如何判断当前点击的 Shape 是哪个。

konva 的事件系统也是自己创建的。和分析 react 事件系统一样，分析 konva 事件也可以从事件绑定、事件合成、事件触发这几个角度分析。

## 绑定事件

初始化会把事件绑定在 Stage 元素对应的 div 上：

```ts
EVENTS = [
    [MOUSEENTER, '_pointerenter'],
    [MOUSEDOWN, '_pointerdown'],
    [MOUSEMOVE, '_pointermove'],
    [MOUSEUP, '_pointerup'],
    [MOUSELEAVE, '_pointerleave'],
    [TOUCHSTART, '_pointerdown'],
    [TOUCHMOVE, '_pointermove'],
    [TOUCHEND, '_pointerup'],
    [TOUCHCANCEL, '_pointercancel'],
    [MOUSEOVER, '_pointerover'],
    [WHEEL, '_wheel'],
    [CONTEXTMENU, '_contextmenu'],
    [POINTERDOWN, '_pointerdown'],
    [POINTERMOVE, '_pointermove'],
    [POINTERUP, '_pointerup'],
    [POINTERCANCEL, '_pointercancel'],
    [LOSTPOINTERCAPTURE, '_lostpointercapture'],
  ];
  // 绑定事件
  _bindContentEvents() {
    if (!Konva.isBrowser) {
      return;
    }
    EVENTS.forEach(([event, methodName]) => {
      // 事件绑定在 content 这个 dom 节点上面
      this.content.addEventListener(event, (evt) => {
        this[methodName](evt);
      });
    });
  }
```

这个数组，前面是绑定的 dom 事件名称，后面是 Stage 类中的具体的每个方法名。比如 onmousedown 事件的处理函数就是`_pointerdown`

## 事件触发

以 onmousedown 事件为例

```ts
_pointerdown(evt: TouchEvent | MouseEvent | PointerEvent) {
    const events = getEventsMap(evt.type);
    const eventType = getEventType(evt.type);

    if (!events) {
      return;
    }
    // 这一步计算当前鼠标点击的坐标，存到this._changedPointerPositions里
    this.setPointersPositions(evt);

    var triggeredOnShape = false;
    // 这里是鼠标点击坐标
    this._changedPointerPositions.forEach((pos) => {
      // 这一步很重要，通过点击坐标找到对应的shape
      var shape = this.getIntersection(pos);

      // 检查这个位置的shape是不是有监听的事件
      const hasShape = shape && shape.isListening();
      if (!hasShape) {
        return;
      }

      if (Konva.capturePointerEventsEnabled) {
        shape.setPointerCapture(pos.id);
      }

      // 保存点击事件开始的坐标，这个主要是给mousemove事件用
      this[eventType + 'ClickStartShape'] = shape;
      // 在对应的shape上，触发并冒泡该事件
      shape._fireAndBubble(events.pointerdown, {
        evt: evt,
        pointerId: pos.id,
      });
      triggeredOnShape = true;

      // TODO: test in iframe
      // only call preventDefault if the shape is listening for events
      const isTouch = evt.type.indexOf('touch') >= 0;
      if (shape.preventDefault() && evt.cancelable && isTouch) {
        evt.preventDefault();
      }
    });
  }
```

### getIntersection 匹配当前坐标的 shape

这个方法在 Stage 上。因为一个 Stage 可能有多个 Layer，因此它从最顶层的 Layer 开始，寻找是否存在一个 shape：

```ts
getIntersection(pos: Vector2d) {
  if (!pos) {
    return null;
  }
  var layers = this.children,
    len = layers.length,
    end = len - 1,
    n;
  for (n = end; n >= 0; n--) {
    const shape = layers[n].getIntersection(pos);
    if (shape) {
      return shape;
    }
  }
  return null;
}
```

Layer 上的 getIntersection 方法如下：(简化)

```ts
getIntersection(pos: Vector2d) {
  if (!this.isListening() || !this.isVisible()) {
    return null;
  }
  while (true) {
    for (let i = 0; i < INTERSECTION_OFFSETS_LEN; i++) {
      const intersectionOffset = INTERSECTION_OFFSETS[i];
      const obj = this._getIntersection({
        x: pos.x + intersectionOffset.x * spiralSearchDistance,
        y: pos.y + intersectionOffset.y * spiralSearchDistance,
      });
      const shape = obj.shape;
      if (shape) {
        return shape;
      }
    }
  }
}
_getIntersection(pos: Vector2d): { shape?: Shape; antialiased?: boolean } {
  const ratio = this.hitCanvas.pixelRatio;
  const p = this.hitCanvas.context.getImageData(
    Math.round(pos.x * ratio),
    Math.round(pos.y * ratio),
    1,
    1
  ).data;
  const p3 = p[3];
  // 完全不透明的像素点，说明找到了某个shape
  if (p3 === 255) {
    const colorKey = Util._rgbToHex(p[0], p[1], p[2]);
    // 从shapes数组中取到那个shape
    const shape = shapes[HASH + colorKey];
    if (shape) {
      return {
        shape: shape,
      };
    }
    return {
      antialiased: true,
    };
  }
  return {};
}
```

当 Shape 初始化的时候，会生成一个随机的颜色，以这个颜色作为 key 存入到 shapes 数组里面。
也就是说，**每个 shape 都是用一个独特的颜色标识的，只要能找到这个颜色，就能找到这个 shape**

```ts
constructor(config?: Config) {
  super(config);
  // set colorKey
  let key: string;
  while (true) {
    // 生成随机色值
    key = Util.getRandomColor();
    if (key && !(key in shapes)) {
      break;
    }
  }
  this.colorKey = key;
  // 存入 shapes 数组
  shapes[key] = this;
}
```

因此当事件触发，执行到\_getIntersection 时，通过 getImageData 获取到这个位置上的元素的颜色，然后就能找到对应的 shape 了。

### hitCanvas 和 sceneCanvas

Konva 在创建 Layer 的时候会创建两个 Canvas，一个用于 sceneCanvas 用于绘制 Shape，另一个 hitCanvas 在内存里面，用于判断是否被点击。

```js
canvas = new SceneCanvas();
hitCanvas = new HitCanvas({
  pixelRatio: 1,
});
```

每次在 sceneCanvas 上面绘制的时候，同样会在内存中的 hitCanvas 里面绘制一遍，并且将上面随机生成的色值作为 fill 和 stroke 的颜色填充。因此 sceneCanvas 显示的是正常的颜色，而 hitCanvas 显示的是用于表示 shape 的随机的颜色。

上面获取 imageData，实际上是获取的 hitCanvas 上的 imageData。

### \_fireAndBubble 触发事件

这个函数就是调用\_fire 函数触发用户设置的事件，然后再递归向上到父元素，模拟冒泡过程。

```ts
_fireAndBubble(eventType, evt, compareShape) {
  if (evt && this.nodeType === SHAPE) {
    evt.target = this;
  }
  // 判断是否到当前节点停止触发事件
  var shouldStop = ...
  if (!shouldStop) {
      // 在当前节点触发事件
    this._fire(eventType, evt);
    // 判断是否冒泡
    var stopBubble = ...
    if (
      ((evt && !evt.cancelBubble) || !evt) &&
      this.parent &&
      this.parent.isListening() &&
      !stopBubble
    ) {
      // 递归向上冒泡
      if (compareShape && compareShape.parent) {
        this._fireAndBubble.call(this.parent, eventType, evt, compareShape);
      } else {
        this._fireAndBubble.call(this.parent, eventType, evt);
      }
    }
  }
}

_fire(eventType, evt) {
  evt = evt || {};
  evt.currentTarget = this;
  evt.type = eventType;

  // 获取用户设置在当前节点上的 listeners
  const topListeners = this._getProtoListeners(eventType);
  if (topListeners) {
    for (var i = 0; i < topListeners.length; i++) {
      topListeners[i].handler.call(this, evt);
    }
  }
}
```

# react-konva 和 react 的结合

react-konva 的实现利用了 react 的 react-reconciler 来自定义渲染。React 将组件分为 Host 类型和 Custom 类型，Host 类型默认使用的是浏览器的 document api 进行渲染，但可以通过 Reconciler 来自定义 HostComponent 渲染，比如覆写 appendChild 方法、自定义元素 instance 实例等。

[官方文档](https://github.com/facebook/react/tree/main/packages/react-reconciler)有关于 Reconciler 的介绍，并且有一些简单的例子。

类似的实现方案有很多，比如 Remax、react-pdf 等

简单来说，就是自己创建一个 Reconciler，用以代替 react 的`ReactDOM.render`方法来渲染。比如：

```js
import ReactReconciler from "react-reconciler";

const HostConfig = {

};
const ReactReconcilerInstance = ReactReconciler(hostConfig);
const canvasRender = {
  render(){

  }
}
export default canvasRender

// index.tsx
import App from './App.tsx';

const canvasDom = document.getElementById('canvas-root') as HTMLCanvasElement;

CanvasRender.render(<App />, canvasDom)
```

hostconfig 就是关于 Reconciler 上的自定义方法、属性等。这些方法都是为渲染自定义组件而用的，其中有几个主要的方法：

- createInstance: 为 element 创建实例。这个实例不一定是 dom 元素，对于 konva 来说，实例就是 konva 中的每个节点，比如圆形、方形等图案、stage、layer 等。

```ts
export function createInstance(type, props, internalInstanceHandle) {
  let NodeClass = Konva[type];
  if (!NodeClass) {
    NodeClass = Konva.Group;
  }

  // 复制事件类型的props
  const propsWithoutEvents = {};
  const propsWithOnlyEvents = {};

  for (var key in props) {
    var isEvent = key.slice(0, 2) === "on";
    if (isEvent) {
      propsWithOnlyEvents[key] = props[key];
    } else {
      propsWithoutEvents[key] = props[key];
    }
  }
  // 这个就是konva中的node对象
  const instance = new NodeClass(propsWithoutEvents);

  applyNodeProps(instance, propsWithOnlyEvents);

  return instance;
}
```

除了创建的实例，一般还要有一个用于更新该实例的函数。当 react 触发更新时，就调用更新函数来更新这个实例。
可以把 createInstance 看做是 element 的挂载阶段执行的步骤，commitUpdate 则是更新时要执行的步骤。

```ts
export function commitUpdate(
  instance,
  updatePayload,
  type,
  oldProps,
  newProps
) {
  // 处理更新时的操作
  // 这个函数的内部大致是根据新的props更新instance上的属性
  applyNodeProps(instance, newProps, oldProps);
}
```

- 一些需要在 commit 阶段实现的方法，包括对文本节点、其他节点的插入、删除、更新的操作。

比如 appendChild 方法，用于代替 domcument.appendChild 方法，告知 react 怎么执行 appendChild 操作：

```ts
export function appendChild(parentInstance, child) {
  if (child.parent === parentInstance) {
    child.moveToTop();
  } else {
    parentInstance.add(child);
  }

  updatePicture(parentInstance);
}
```

类似的还有 insertBefore、removeChild 等。这些方法很多只是按照类型填充，如果不需要，直接返回 null、boolean 或某个特定值就可以。konva 的实现可以参考[它的源码](https://github1s.com/konvajs/react-konva/blob/HEAD/src/ReactKonvaHostConfig.ts)

当这些方法都填充完毕，最后就能实现自定义的渲染。
不过，konva 并不是直接替换根部的 ReactDOM，而是编写了一个 StageWrap 组件，手动调用 KonvaRenderer 上的方法

```ts
const StageWrap = (props) => {
  const container = React.useRef();
  const stage = React.useRef<any>();
  const fiberRef = React.useRef();

  const oldProps = usePrevious(props);
  const Bridge = useContextBridge();

  const _setRef = (stage) => {
    // 用于处理设置在stage上的 ref
  };

  // 初始化：调用createContainer创建实例，updateContainer调度mount
  React.useLayoutEffect(() => {
    stage.current = new Konva.Stage({
      width: props.width,
      height: props.height,
      container: container.current,
    });

    _setRef(stage.current);

    // @ts-ignore
    fiberRef.current = KonvaRenderer.createContainer(
      stage.current,
      LegacyRoot,
      false,
      null
    );
    KonvaRenderer.updateContainer(
      React.createElement(Bridge, {}, props.children),
      fiberRef.current
    );

    return () => {
      if (!Konva.isBrowser) {
        return;
      }
      _setRef(null);
      KonvaRenderer.updateContainer(null, fiberRef.current, null);
      stage.current.destroy();
    };
  }, []);

  // 更新：调用updateContainer更新元素，并通过applyNodeProps更新stage
  React.useLayoutEffect(() => {
    _setRef(stage.current);
    applyNodeProps(stage.current, props, oldProps);
    KonvaRenderer.updateContainer(
      React.createElement(Bridge, {}, props.children),
      fiberRef.current,
      null
    );
  });

  // 返回一个stage
  return React.createElement("div", {
    ref: container,
    accessKey: props.accessKey,
    className: props.className,
    role: props.role,
    style: props.style,
    tabIndex: props.tabIndex,
    title: props.title,
  });
};
```

# 其他部分

## 绘制流程

konva 的绘制开始是从 add 方法开始的

```js
add(...children: ChildType[]) {
    const child = children[0];
    if (child.getParent()) {
      child.moveTo(this);
      return this;
    }
    this._fire('add', {
      child: child,
    });
    this._requestDraw();
    // chainable
    return this;
  }
```

然后进入\_requestDraw 方法，调用 batchDraw 方法进行批量绘制

```js
_requestDraw() {
    if (Konva.autoDrawEnabled) {
      const drawNode = this.getLayer() || this.getStage();
      drawNode?.batchDraw();
    }
  }
batchDraw() {
    if (!this._waitingForDraw) {
      this._waitingForDraw = true;
      Util.requestAnimFrame(() => {
        this.draw();
        this._waitingForDraw = false;
      });
    }
    return this;
  }
```

注意这里调用了requestAnimFrame来实现绘制过程的控制。通过raf在每一帧执行一次绘制，而不会导致同时有大量的绘制任务执行而导致卡顿。

_waitingForDraw其实就是一种批量渲染的控制方式，类似react老版本中的isBatchedRendering这样的。当这个值为true时，开启批量渲染

draw函数会调用真正的绘制函数drawScene，然后遍历通过add方法加入的子元素（shape），再调用每个元素上的drawScene方法

```js
_drawChildren(drawMethod, canvas, top) {
  var context = canvas && canvas.getContext()
  this.children?.forEach(function (child) {
    child[drawMethod](canvas, top);
  });
}
```

可以看到这个过程实际上是通过循环的方式绘制每个元素，但同时也有raf等方式来保证一次执行的绘制任务不会太多。

