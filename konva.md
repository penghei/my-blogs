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
也就是说，**每个shape都是用一个独特的颜色标识的，只要能找到这个颜色，就能找到这个shape**

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

因此当事件触发，执行到_getIntersection时，通过getImageData获取到这个位置上的元素的颜色，然后就能找到对应的shape了。

### hitCanvas和sceneCanvas

Konva 在创建 Layer 的时候会创建两个 Canvas，一个用于 sceneCanvas 用于绘制 Shape，另一个 hitCanvas 在内存里面，用于判断是否被点击。
```js
canvas = new SceneCanvas();
hitCanvas = new HitCanvas({
  pixelRatio: 1,
});
```
每次在 sceneCanvas 上面绘制的时候，同样会在内存中的 hitCanvas 里面绘制一遍，并且将上面随机生成的色值作为 fill 和 stroke 的颜色填充。因此sceneCanvas显示的是正常的颜色，而hitCanvas显示的是用于表示shape的随机的颜色。

上面获取imageData，实际上是获取的hitCanvas上的imageData。


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

