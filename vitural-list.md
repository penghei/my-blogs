虚拟列表基础原理和实现

基本实现：

参考：https://juejin.cn/post/6995334008603148295

扩展内容：

参考：https://juejin.cn/post/6844903982742110216

用时间分片来更新大量元素：https://juejin.cn/post/6844903938894872589
用io实现虚拟列表：https://zhuanlan.zhihu.com/p/468212489

# 基本实现

整理一下虚拟列的实现关键

1. 关键高度：容器高度（可视区域高度）、滚动高度（滚动区域内部高度）、元素高度（可能不固定）
2. 初始化渲染 n 条数据，通过 data.slice 的方式来选择；n = containerHeight / itemHeight
3. 监听滚动事件，每次滚动的时候通过滚动的 offset 计算 slice 的 start 和 end

具体来说是通过 scrollTop 获取滚动高度，通过 scrollTop / itemHeight 获取偏移的元素数量。再减去预加载的元素的个数，就可以得到要加载的起始。同理加上可视区域内的元素个数和预加载末尾就可以得到渲染元素的起始

```js
/**
 * 计算元素范围
 */
const calculateRange = () => {
  const element = containerRef.current;
  if (element) {
    const offset: number = Math.floor(element.scrollTop / ITEM_HEIGHT) + 1;
    console.log(offset, "offset");
    const viewItemSize: number = Math.ceil(element.clientHeight / ITEM_HEIGHT);
    const startSize: number = offset - PRE_LOAD_COUNT;
    const endSize: number = viewItemSize + offset + PRE_LOAD_COUNT;
    setShowPageRange({
      start: startSize < 0 ? 0 : startSize,
      end: endSize > sourceData.length ? sourceData.length : endSize,
    });
  }
};
```

4. 由于此时页面上只有需要渲染的元素，scrollTop 内实际没有其他元素。因此我们为了保证这些元素显示在可视区域内，就需要在他们上方增加一个垂直偏移，把他们“移动”到可视区域内。

偏移量可以 MarinTop 和 TranForm 实现，值为当前元素的起始 index \* itemHeight，表示在上方模拟了这么多元素的高度。

同时用于撑开滚动的内部空白 div 的高度也要减去这个偏移量

```js
const scrollViewOffset = useMemo(() => {
  console.log(showRange.start, "showRange.start");
  return showRange.start * ITEM_HEIGHT;
}, [showRange.start]);

<div
  ref={containerRef}
  style={{
    height: SCROLL_VIEW_HEIGHT,
    overflow: "auto",
  }}
  onScroll={onContainerScroll}
>
  <div
    style={{
      width: "100%",
      height: scrollViewHeight - scrollViewOffset,
      marginTop: scrollViewOffset,
    }}
  >
    {sourceData.map((e) => (
      <div
        style={{
          height: ITEM_HEIGHT,
        }}
        className="showElement"
        key={e}
      >
        Current Position: {e}
      </div>
    ))}
  </div>
</div>;
```

5. 通过 scrollHeight - innerHeight >= scrollTop 的方法可以检查当前是否触底。配合触底就可以实现触底加载，分页效果。

```js
const reachScrollBottom = (): boolean => {
  //滚动条距离顶部
  const contentScrollTop = containerRef.current?.scrollTop || 0;
  //可视区域
  const clientHeight = containerRef.current?.clientHeight || 0;
  //滚动条内容的总高度
  const scrollHeight = containerRef.current?.scrollHeight || 0;
  if (contentScrollTop + clientHeight >= scrollHeight) {
    return true;
  }
  return false;
};

const onContainerScroll = (event: UIEvent<HTMLDivElement>) => {
  event.preventDefault();
  if (reachScrollBottom()) {
    // 模拟数据添加，实际上是 await 异步请求做为数据的添加
    let endIndex = showRange.end;
    let pushData: number[] = [];
    for (let index = 0; index < 20; index++) {
      pushData.push(endIndex++);
    }
    setSourceData((arr) => {
      return [...arr, ...pushData];
    });
  }
  calculateRange();
};
```

# 扩展内容

## 高度不统一

处理 itemHeight 高度不统一，或者未知的情况。

首先要明确一点，如果高度不统一，会造成什么问题？
我们需要 itemHeight 来确定的有几个关键值

- 偏移的元素个数，即 offsetAmount = scrollTop / itemHeight
- 起始 index，依赖于 offsetAmount
- 可视区域内的元素个数，即 visibleAmount = innerHeight / itemHeight
- 结束 index
- 样式，比如内容区高度、偏移量等等

这种情况的处理可以通过一个 props 来让用户预先告知高度，比如 FlatList 的 getItemLayout。
如果高度不统一，那么采取的方式是**以预估高度先行渲染，然后获取真实高度并缓存。**

具体方法是

1. 维护一个 positions 数组，存储每一项的数据，包括

```js
const positions = listData.map((item, index) => {
  return {
    index,
    height: this.estimatedItemSize,
    top: index * this.estimatedItemSize,
    bottom: (index + 1) * this.estimatedItemSize,
  };
});
```

2. 根据设置的预估高度 estimatedItemSize 来初始化 positions 之后，先以这些值计算出基本的内容，即上面所说的需要的关键值

3. 当渲染事件完成后（useEffect），获取具体的布局情况并更新 positions

```js
useEffect(() => {
  const nodes = containerRef.current.children; // 获取dom元素
  nodes.forEach((node) => {
    let rect = node.getBoundingClientRect();
    let height = rect.height;
    let index = +node.id.slice(1);
    let oldHeight = this.positions[index].height;
    let dValue = oldHeight - height;
    //存在差值
    if (dValue) {
      this.positions[index].bottom = this.positions[index].bottom - dValue;
      this.positions[index].height = height;
      for (let k = index + 1; k < this.positions.length; k++) {
        this.positions[k].top = this.positions[k - 1].bottom;
        this.positions[k].bottom = this.positions[k].bottom - dValue;
      }
    }
  });
}, []);
```

4. 当滚动事件发生时，找到距离起始位置最近的元素，确定 startIndex。

查找的过程可以通过二分查找优化，本质上是查找 positions 中元素的 bottom>=scrollTop

```js
//获取列表起始索引
getStartIndex(scrollTop = 0){
  let item = positions.find(i => i && i.bottom > scrollTop); // 可优化为二分
  return item.index;
}
```

然后其他的值获取方式类似：
- 顶部偏移量：顶部元素的bottom
- 可视区域的元素个数：查找top >= scrollHeight的元素
等等


这种实现方式其实还有一些问题：直接拉到底部再往上滚动，无法避免列表内容发生上下偏移。原因就是上面还未来得及渲染的列表，在渲染之后，其真实高度和预估高度有偏差，导致顶部占据高度发生变化。
