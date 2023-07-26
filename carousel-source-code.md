轮播图是怎么实现的？
下面是 react-native-snap-carousel 的主要代码。忽略 animated 相关的模块并删除一些非核心代码，剩下的代码主要如下所示。

在 rn 上轮播组件的设计可以通过 FlatList 来实现。FlatList 本身具有虚拟列表的优化，即使创建多个元素，也能有较好的性能。

这个轮播组件有几个关键的设计点

1. 无限循环：轮播组件的最常用的无限循环方式是这样：

假设一共有 N 个元素，那么列表的总宽度设置为 N+2 个元素的宽度。
例子如下，两端填充如图，当处于一端时，且即将进入循环状态的时候，如第二张图，从状态 1 滑动到状态 2，在滑动结束的时候，将当前的位置直接转到状态 3，直接 scrollTo 滚动到状态 3 的元素的位置。由于这个 scrollTo 是不带动画的，因此用户感觉不到滚动的发生。

![](https://pic.imgdb.cn/item/64b1131f1ddac507cc53a3a6.jpg)

我们可以将元素扩充，直接把两边的用于缓冲的元素加入到渲染的 data 中。如下代码就是库内的实现

```js
// loopClonesPerSide表示要在两边增加的额外元素数量
// loopClonesPerSide会影响在用户快速拖动时，看到的空白区域时间。如果过小，那么可能还没来得及瞬间跳转（scroll还没停止），这时会显示空白；如果数量合适，那么即使还没scroll停止，也能看到最新的元素

let previousItems = []; // previousItems是data从后向前裁剪，放在data前面
let nextItems = []; // nextItems则是从前向后，放在data后面

if (loopClonesPerSide > dataLength) {
  const dataMultiplier = Math.floor(loopClonesPerSide / dataLength);
  const remainder = loopClonesPerSide % dataLength;

  for (let i = 0; i < dataMultiplier; i++) {
    previousItems.push(...data);
    nextItems.push(...data);
  }

  previousItems.unshift(...data.slice(-remainder));
  nextItems.push(...data.slice(0, remainder));
} else {
  previousItems = data.slice(-loopClonesPerSide);
  nextItems = data.slice(0, loopClonesPerSide);
}

// 最后按照顺序合并。参考上面的实现原理就能理解
return previousItems.concat(data, nextItems);
```

2. 元素定位：即如何从 offset 对应到具体的元素，以及用户滚动得到偏移量后，如何定位到最近的元素。

方法就是设置一个数据结构 positions 数组，用于记录每个元素的 start 和 end

```js
// _getCustomData就是如上所述，为了实现loop扩充了data
this._getCustomData(props).forEach((itemData, index) => {
  // 这里的index是不真实的index，是复制过后的，但在布局中是基准
  // _index才是原本在data中的index
  const _index = this._getCustomIndex(index, props);
  let animatedValue;

  this._positions[index] = {
    start: index * sizeRef,
    end: index * sizeRef + sizeRef,
  };
  //...
});
```

在滚动得到本次的 offset 时，根据 offset 得出中心的位置，然后检查有哪个元素恰好在中心线的两边，那就选定是这个元素了

```js
_getActiveItem(offset) {
  const { activeSlideOffset, swipeThreshold } = this.props;
  const center = this._getCenter(offset); //
  const centerOffset = activeSlideOffset || swipeThreshold;
  // 这里是定位元素的关键
  // 如果没有centerOffset，那么确定当前是哪个元素的方法就是center >= start && center <= end，即center应该在元素的宽度范围内
  // 加上了centerOffset，其实是缩小了范围，相当于center >= start - centerOffset && center <= end + centerOffset，这样可能移动更小的距离就会到下一个元素去
  for (let i = 0; i < this._positions.length; i++) {
    const { start, end } = this._positions[i];
    if (center + centerOffset >= start && center - centerOffset <= end) {
      return i;
    }
  }
}
```

3. 自动播放：全局维护两个变量来控制是否自动播放

- autoplaying：是否正在自动播放，如果在播放就无视开启自动播放的调用，如果不在播放也不能停止。
- \_autoplay：表示是否需要自动播放，从 props 得到。如果为 false 那么不会调用 startAutoPlay

具体实现，其实就是开启一个setInterval，然后在延迟delay之后开始，每次移动到下一个。

pauseAutoPlay用于在触摸事件发生时暂停自动播放。也就说在用户拖动期间不会进行自动播放。

```js
startAutoplay () {
    const { autoplayInterval, autoplayDelay } = this.props;
    this._autoplay = true;
    if (this._autoplaying) {
        return;
    }
    clearTimeout(this._autoplayTimeout);
    this._autoplayTimeout = setTimeout(() => {
        this._autoplaying = true;
        this._autoplayInterval = setInterval(() => {
            if (this._autoplaying) {
                this.snapToNext();
            }
        }, autoplayInterval);
    }, autoplayDelay);
}

pauseAutoPlay () {
    this._autoplaying = false;
    clearTimeout(this._autoplayTimeout);
    clearTimeout(this._enableAutoplayTimeout);
    clearInterval(this._autoplayInterval);
}
stopAutoplay () {
    this._autoplay = false;
    this.pauseAutoPlay();
}
```

下面是精简后的全部代码：

```js
// 根据props选择scrollView或FlatList
const AnimatedFlatList = FlatList
  ? Animated.createAnimatedComponent(FlatList)
  : null;
const AnimatedScrollView = Animated.createAnimatedComponent(ScrollView);

export default class Carousel extends Component {
  static propTypes = {
    // 一大堆props
  };

  static defaultProps = {
    // props默认值
  };

  constructor(props) {
    super(props);

    this.state = {
      hideCarousel: true,
      interpolators: [],
    };

    // 一些关键的全局变量
    const initialActiveItem = this._getFirstItem(props.firstItem); // 初始化的第一个元素
    this._activeItem = initialActiveItem; // 当前active的元素
    this._previousActiveItem = initialActiveItem;
    this._previousFirstItem = initialActiveItem;
    this._previousItemsLength = initialActiveItem;

    this._positions = []; // 记录每个元素的start、end偏移量，很重要。通过positions可以确定每个元素的显隐，以及当前active的元素是哪个。
    this._setScrollHandler(props);
  }

  // 在挂载阶段定位第一个渲染的元素，移动到第一个元素，并启动autoplay
  componentDidMount() {
    const { apparitionDelay, autoplay, firstItem } = this.props;
    const _firstItem = this._getFirstItem(firstItem);
    const apparitionCallback = () => {
      this.setState({ hideCarousel: false });
      if (autoplay) {
        this.startAutoplay();
      }
    };

    this._mounted = true;
    this._initPositionsAndInterpolators(); // 初始化position信息

    requestAnimationFrame(() => {
      if (!this._mounted) {
        return;
      }

      this._snapToItem(_firstItem, false, false, true, false); // 滚动到第一个元素，让第一个元素的start（左边）对齐容器的起点（左边）

      if (apparitionDelay) {
        this._apparitionTimeout = setTimeout(() => {
          apparitionCallback();
        }, apparitionDelay);
      } else {
        apparitionCallback();
      }
    });
  }
  // 这里就是一个简单的优化，浅比较props
  shouldComponentUpdate(nextProps, nextState) {
    if (this.props.shouldOptimizeUpdates === false) {
      return true;
    } else {
      return shallowCompare(this, nextProps, nextState);
    }
  }

  // 这里主要是针对宽高等关键布局信息的props改变时，重新计算的过程
  componentDidUpdate(prevProps) {
    const { interpolators } = this.state;
    const {
      firstItem,
      itemHeight,
      itemWidth,
      scrollEnabled,
      sliderHeight,
      sliderWidth,
    } = this.props;
    const itemsLength = this._getCustomDataLength(this.props);

    if (!itemsLength) {
      return;
    }

    const nextFirstItem = this._getFirstItem(firstItem, this.props);
    let nextActiveItem =
      this._activeItem || this._activeItem === 0
        ? this._activeItem
        : nextFirstItem;

    const hasNewSliderWidth =
      sliderWidth && sliderWidth !== prevProps.sliderWidth;
    const hasNewSliderHeight =
      sliderHeight && sliderHeight !== prevProps.sliderHeight;
    const hasNewItemWidth = itemWidth && itemWidth !== prevProps.itemWidth;
    const hasNewItemHeight = itemHeight && itemHeight !== prevProps.itemHeight;
    const hasNewScrollEnabled = scrollEnabled !== prevProps.scrollEnabled;

    // Prevent issues with dynamically removed items
    if (nextActiveItem > itemsLength - 1) {
      nextActiveItem = itemsLength - 1;
    }

    // Handle changing scrollEnabled independent of user -> carousel interaction
    if (hasNewScrollEnabled) {
      this._setScrollEnabled(scrollEnabled);
    }

    if (
      interpolators.length !== itemsLength ||
      hasNewSliderWidth ||
      hasNewSliderHeight ||
      hasNewItemWidth ||
      hasNewItemHeight
    ) {
      this._activeItem = nextActiveItem;
      this._previousItemsLength = itemsLength;

      this._initPositionsAndInterpolators(this.props);

      // Handle scroll issue when dynamically removing items (see #133)
      // This also fixes first item's active state on Android
      // Because 'initialScrollIndex' apparently doesn't trigger scroll
      if (this._previousItemsLength > itemsLength) {
        this._hackActiveSlideAnimation(nextActiveItem, null, true);
      }

      if (
        hasNewSliderWidth ||
        hasNewSliderHeight ||
        hasNewItemWidth ||
        hasNewItemHeight
      ) {
        this._snapToItem(nextActiveItem, false, false, false, false);
      }
    } else if (
      nextFirstItem !== this._previousFirstItem &&
      nextFirstItem !== this._activeItem
    ) {
      this._activeItem = nextFirstItem;
      this._previousFirstItem = nextFirstItem;
      this._snapToItem(nextFirstItem, false, true, false, false);
    }

    if (this.props.onScroll !== prevProps.onScroll) {
      this._setScrollHandler(this.props);
    }
  }

  // 清除定时器
  componentWillUnmount() {
    this._mounted = false;
    this.stopAutoplay();
    clearTimeout(this._apparitionTimeout);
    clearTimeout(this._hackSlideAnimationTimeout);
    clearTimeout(this._enableAutoplayTimeout);
    clearTimeout(this._autoplayTimeout);
    clearTimeout(this._snapNoMomentumTimeout);
    clearTimeout(this._edgeItemTimeout);
    clearTimeout(this._lockScrollTimeout);
  }

  /**
   * 如果开启了loop，则根据loopClonesPerSide创建额外的元素
   * @param {*} props
   * @returns
   */
  _getCustomData(props = this.props) {
    const { data, loopClonesPerSide } = props;
    const dataLength = data && data.length;

    if (!dataLength) {
      return [];
    }

    if (!this._enableLoop()) {
      return data;
    }

    let previousItems = [];
    let nextItems = [];

    // 设置loopClonesPerSide控制的两边渲染的元素数量，会修改data。
    // 修改之后data数组的索引和原本的会不同，因此后面的所有index操作都需要下面几个函数进行转换。即真实index和组件内index的对照
    if (loopClonesPerSide > dataLength) {
      const dataMultiplier = Math.floor(loopClonesPerSide / dataLength);
      const remainder = loopClonesPerSide % dataLength;

      for (let i = 0; i < dataMultiplier; i++) {
        previousItems.push(...data);
        nextItems.push(...data);
      }

      previousItems.unshift(...data.slice(-remainder));
      nextItems.push(...data.slice(0, remainder));
    } else {
      previousItems = data.slice(-loopClonesPerSide);
      nextItems = data.slice(0, loopClonesPerSide);
    }

    return previousItems.concat(data, nextItems);
  }

  /**
   * 获取数据长度，即data.length。如果开启loop那就要两边都加上loopClonesPerSide
   * @param {*} props
   * @returns
   */
  _getCustomDataLength(props = this.props) {
    const { data, loopClonesPerSide } = props;
    const dataLength = data && data.length;

    if (!dataLength) {
      return 0;
    }

    return this._enableLoop() ? dataLength + 2 * loopClonesPerSide : dataLength;
  }

  // 从组件内index映射到customIndex，找到真实的index
  _getCustomIndex(index, props = this.props) {
    const itemsLength = this._getCustomDataLength(props);

    if (!itemsLength || (!index && index !== 0)) {
      return 0;
    }

    return this._needsRTLAdaptations() ? itemsLength - index - 1 : index;
  }

  // 上面函数的反向，从真实index转化到组件内的index，和customData能对应上
  _getDataIndex(index) {
    const { data, loopClonesPerSide } = this.props;
    const dataLength = data && data.length;

    if (!this._enableLoop() || !dataLength) {
      return index;
    }

    if (index >= dataLength + loopClonesPerSide) {
      return loopClonesPerSide > dataLength
        ? (index - loopClonesPerSide) % dataLength
        : index - dataLength - loopClonesPerSide;
    } else if (index < loopClonesPerSide) {
      if (loopClonesPerSide > dataLength) {
        const baseDataIndexes = [];
        const dataIndexes = [];
        const dataMultiplier = Math.floor(loopClonesPerSide / dataLength);
        const remainder = loopClonesPerSide % dataLength;

        for (let i = 0; i < dataLength; i++) {
          baseDataIndexes.push(i);
        }

        for (let j = 0; j < dataMultiplier; j++) {
          dataIndexes.push(...baseDataIndexes);
        }

        dataIndexes.unshift(...baseDataIndexes.slice(-remainder));
        return dataIndexes[index];
      } else {
        return index + dataLength - loopClonesPerSide;
      }
    } else {
      return index - loopClonesPerSide;
    }
  }

  /**
   * 把输入的index转化为真实元素的index
   * 由于开启loop后会复制index，这里就是在做转化
   * @param {*} index
   * @returns
   */
  _getPositionIndex(index) {
    const { loop, loopClonesPerSide } = this.props;
    return loop ? index + loopClonesPerSide : index;
  }

  /**
   * 获取第一个渲染的元素的真实index。参数是逻辑上的index
   * @param {} index
   * @param {*} props
   * @returns
   */
  _getFirstItem(index, props = this.props) {
    const { loopClonesPerSide } = props;
    const itemsLength = this._getCustomDataLength(props);

    if (!itemsLength || index > itemsLength - 1 || index < 0) {
      return 0;
    }

    return this._enableLoop() ? index + loopClonesPerSide : index;
  }

  /**
   * 获取轮播组件的ref，一般是ScrollView或FlatList的ref
   * @returns
   */
  _getWrappedRef() {
    if (
      this._carouselRef &&
      ((this._needsScrollView() && this._carouselRef.scrollTo) ||
        (!this._needsScrollView() && this._carouselRef.scrollToOffset))
    ) {
      return this._carouselRef;
    }
    // https://github.com/facebook/react-native/issues/10635
    // https://stackoverflow.com/a/48786374/8412141
    return (
      this._carouselRef &&
      this._carouselRef.getNode &&
      this._carouselRef.getNode()
    );
  }

  /**
   * 获取ScrollView或FlatList的ScrollOffset，用于用户滚动事件的计算
   * @param {*} event
   * @returns
   */
  _getScrollOffset(event) {
    const { vertical } = this.props;
    return (
      (event &&
        event.nativeEvent &&
        event.nativeEvent.contentOffset &&
        event.nativeEvent.contentOffset[vertical ? "y" : "x"]) ||
      0
    );
  }

  /**
   * 容器内部的两侧边距，相当于item和Carousel的间距
   * @param {*} opposite
   * @returns
   */
  _getContainerInnerMargin(opposite = false) {
    const {
      sliderWidth,
      sliderHeight,
      itemWidth,
      itemHeight,
      vertical,
      activeSlideAlignment,
    } = this.props;

    if (
      (activeSlideAlignment === "start" && !opposite) ||
      (activeSlideAlignment === "end" && opposite)
    ) {
      return 0;
    } else if (
      (activeSlideAlignment === "end" && !opposite) ||
      (activeSlideAlignment === "start" && opposite)
    ) {
      return vertical ? sliderHeight - itemHeight : sliderWidth - itemWidth;
    } else {
      return vertical
        ? (sliderHeight - itemHeight) / 2
        : (sliderWidth - itemWidth) / 2;
    }
  }

  /**
   * 如果选定中间对齐，那么就是sliderWidth/2，相当于轮播图容器的中间位置
   * @returns
   */
  _getViewportOffset() {
    const {
      sliderWidth,
      sliderHeight,
      itemWidth,
      itemHeight,
      vertical,
      activeSlideAlignment,
    } = this.props;

    if (activeSlideAlignment === "start") {
      return vertical ? itemHeight / 2 : itemWidth / 2;
    } else if (activeSlideAlignment === "end") {
      return vertical
        ? sliderHeight - itemHeight / 2
        : sliderWidth - itemWidth / 2;
    } else {
      return vertical ? sliderHeight / 2 : sliderWidth / 2;
    }
  }

  /**
   * 获取中心位置的偏移量，根据ScrollView当前的偏移量得出
   * 传入的offset是ScrollView等容器当前的偏移，得出的是在这个偏移基础上，视图上的中间位置，一般就是在偏移量上加上sliderWidth / 2
   * @param {*} offset
   * @returns
   */
  _getCenter(offset) {
    return offset + this._getViewportOffset() - this._getContainerInnerMargin();
  }

  /**
   * 获取当前在中心的item。遍历_positions数组找到在中心位置的元素
   * 参数offset是当前ScrollView或FlatList的滚动offset，根据这个找到最近的一个元素
   * @param {number} offset
   * @returns
   */
  _getActiveItem(offset) {
    const { activeSlideOffset, swipeThreshold } = this.props;
    const center = this._getCenter(offset); //
    const centerOffset = activeSlideOffset || swipeThreshold;

    // 这里是定位元素的关键
    // 如果没有centerOffset，那么确定当前是哪个元素的方法就是center >= start && center <= end，即center应该在元素的宽度范围内
    // 加上了centerOffset，其实是缩小了范围，相当于center >= start - centerOffset && center <= end + centerOffset，这样可能移动更小的距离就会到下一个元素去
    for (let i = 0; i < this._positions.length; i++) {
      const { start, end } = this._positions[i];
      if (center + centerOffset >= start && center - centerOffset <= end) {
        return i;
      }
    }

    const lastIndex = this._positions.length - 1;
    if (
      this._positions[lastIndex] &&
      center - centerOffset > this._positions[lastIndex].end
    ) {
      return lastIndex;
    }

    return 0;
  }

  /**
   * 初始化各个元素的位置信息，设置插值动画
   * @param {*} props
   * @returns
   */
  _initPositionsAndInterpolators(props = this.props) {
    const { data, itemWidth, itemHeight, scrollInterpolator, vertical } = props;
    const sizeRef = vertical ? itemHeight : itemWidth;

    if (!data || !data.length) {
      return;
    }

    let interpolators = [];
    this._positions = [];

    this._getCustomData(props).forEach((itemData, index) => {
      // 这里的index是不真实的index，是复制过后的，但在布局中是基准
      // _index才是原本在data中的index
      const _index = this._getCustomIndex(index, props);
      let animatedValue;

      this._positions[index] = {
        start: index * sizeRef,
        end: index * sizeRef + sizeRef,
      };

      if (!this._shouldAnimateSlides(props)) {
        animatedValue = new Animated.Value(1);
      } else if (this._shouldUseCustomAnimation()) {
        animatedValue = new Animated.Value(_index === this._activeItem ? 1 : 0);
      } else {
        let interpolator;

        if (scrollInterpolator) {
          interpolator = scrollInterpolator(_index, props);
        } else if (this._shouldUseStackLayout()) {
          interpolator = stackScrollInterpolator(_index, props);
        } else if (this._shouldUseTinderLayout()) {
          interpolator = tinderScrollInterpolator(_index, props);
        }

        if (
          !interpolator ||
          !interpolator.inputRange ||
          !interpolator.outputRange
        ) {
          interpolator = defaultScrollInterpolator(_index, props);
        }

        animatedValue = this._scrollPos.interpolate({
          ...interpolator,
          extrapolate: "clamp",
        });
      }

      interpolators.push(animatedValue);
    });

    this.setState({ interpolators });
  }

  /**
   * 滚动到指定的offset
   * @param {*} offset
   * @param {*} animated
   * @returns
   */
  _scrollTo(offset, animated = true) {
    const { vertical } = this.props;
    const wrappedRef = this._getWrappedRef();

    if (!this._mounted || !wrappedRef) {
      return;
    }

    const specificOptions = this._needsScrollView()
      ? {
          x: vertical ? 0 : offset,
          y: vertical ? offset : 0,
        }
      : {
          offset,
        };
    const options = {
      ...specificOptions,
      animated,
    };

    if (this._needsScrollView()) {
      wrappedRef.scrollTo(options);
    } else {
      wrappedRef.scrollToOffset(options);
    }
  }

  // 这里是用户滚动事件的监听回调，这个onScroll会被传递给ScrollView
  _onScroll(event) {
    const { callbackOffsetMargin, enableMomentum, onScroll } = this.props;

    const scrollOffset = event
      ? this._getScrollOffset(event)
      : this._currentContentOffset;
    const nextActiveItem = this._getActiveItem(scrollOffset);
    const itemReached = nextActiveItem === this._itemToSnapTo;
    const scrollConditions =
      scrollOffset >= this._scrollOffsetRef - callbackOffsetMargin &&
      scrollOffset <= this._scrollOffsetRef + callbackOffsetMargin;

    this._currentContentOffset = scrollOffset;
    this._onScrollTriggered = true;
    this._lastScrollDate = Date.now();

    if (this._activeItem !== nextActiveItem && itemReached) {
      if (scrollConditions) {
        this._activeItem = nextActiveItem;
        if (this._canFireCallback) {
          // 滚动到指定元素
          this._onSnap(this._getDataIndex(nextActiveItem));
        }
      }
    }
  }

  // 这个方法同样是会传递给ScrollView，表示用户的滚动操作结束
  _onScrollEnd(event) {
    const { autoplayDelay, enableSnap } = this.props;

    if (this._currentContentOffset === this._scrollEndOffset) {
      return;
    }

    this._scrollEndOffset = this._currentContentOffset;
    this._scrollEndActive = this._getActiveItem(this._scrollEndOffset);

    if (enableSnap) {
      // 这里会执行滚动
      this._snapScroll(this._scrollEndOffset - this._scrollStartOffset);
    }

    if (this._autoplay && !this._autoplaying) {
      clearTimeout(this._enableAutoplayTimeout);
      this._enableAutoplayTimeout = setTimeout(() => {
        this.startAutoplay();
      }, autoplayDelay + 50);
    }
  }

  _snapScroll(delta) {
    const { swipeThreshold } = this.props;

    if (!this._scrollEndActive && this._scrollEndActive !== 0 && IS_IOS) {
      this._scrollEndActive = this._scrollStartActive;
    }

    if (this._scrollStartActive !== this._scrollEndActive) {
      // 滚动到下一个元素
      this._snapToItem(this._scrollEndActive);
    } else {
      // 按照偏移量滚动。可以看出这里不管偏移量有多少，最多也只能滚动一个
      if (delta > 0) {
        if (delta > swipeThreshold) {
          this._snapToItem(this._scrollStartActive + 1);
        } else {
          this._snapToItem(this._scrollEndActive);
        }
      } else if (delta < 0) {
        if (delta < -swipeThreshold) {
          this._snapToItem(this._scrollStartActive - 1);
        } else {
          this._snapToItem(this._scrollEndActive);
        }
      } else {
        this._snapToItem(this._scrollEndActive);
      }
    }
  }

  /**
   * 通过index移动到指定的item，关键函数
   * @param {*} index
   * @param {*} animated
   * @param {*} fireCallback
   * @param {*} initial
   * @param {*} lockScroll
   * @returns
   */
  _snapToItem(
    index,
    animated = true,
    fireCallback = true,
    initial = false,
    lockScroll = true
  ) {
    const { enableMomentum, onSnapToItem, onBeforeSnapToItem } = this.props;
    const itemsLength = this._getCustomDataLength();
    const wrappedRef = this._getWrappedRef();

    if (!itemsLength || !wrappedRef) {
      return;
    }

    if (!index || index < 0) {
      index = 0;
    } else if (itemsLength > 0 && index >= itemsLength) {
      index = itemsLength - 1;
    }

    this._itemToSnapTo = index;
    // 通过positions数组获取到指定元素的start和end偏移量
    this._scrollOffsetRef =
      this._positions[index] && this._positions[index].start;
    this._onScrollTriggered = false;

    if (!this._scrollOffsetRef && this._scrollOffsetRef !== 0) {
      return;
    }

    // 滚动到指定偏移位置
    this._scrollTo(this._scrollOffsetRef, animated);

    this._scrollEndOffset = this._currentContentOffset;
  }

  startAutoplay() {
    const { autoplayInterval, autoplayDelay } = this.props;
    this._autoplay = true;

    if (this._autoplaying) {
      return;
    }

    clearTimeout(this._autoplayTimeout);
    this._autoplayTimeout = setTimeout(() => {
      this._autoplaying = true;
      this._autoplayInterval = setInterval(() => {
        if (this._autoplaying) {
          this.snapToNext();
        }
      }, autoplayInterval);
    }, autoplayDelay);
  }

  pauseAutoPlay() {
    this._autoplaying = false;
    clearTimeout(this._autoplayTimeout);
    clearTimeout(this._enableAutoplayTimeout);
    clearInterval(this._autoplayInterval);
  }

  stopAutoplay() {
    this._autoplay = false;
    this.pauseAutoPlay();
  }

  /**
   * 这个是暴露为props的函数，index需要和真实的index进行转换
   * @param {*} index
   * @param {*} animated
   * @param {*} fireCallback
   * @returns
   */
  snapToItem(index, animated = true, fireCallback = true) {
    if (!index || index < 0) {
      index = 0;
    }

    const positionIndex = this._getPositionIndex(index);

    if (positionIndex === this._activeItem) {
      return;
    }

    this._snapToItem(positionIndex, animated, fireCallback);
  }

  snapToNext(animated = true, fireCallback = true) {
    const itemsLength = this._getCustomDataLength();

    let newIndex = this._activeItem + 1;
    if (newIndex > itemsLength - 1) {
      if (!this._enableLoop()) {
        return;
      }
      // 这里的loop实际上就是在index超出时，将newIndex归为0
      // 然后执行一个不包含动画的scrollTo，就可以实现滚动效果了
      newIndex = 0;
    }
    this._snapToItem(newIndex, animated, fireCallback);
  }

  snapToPrev(animated = true, fireCallback = true) {
    const itemsLength = this._getCustomDataLength();

    let newIndex = this._activeItem - 1;
    if (newIndex < 0) {
      if (!this._enableLoop()) {
        return;
      }
      newIndex = itemsLength - 1;
    }
    this._snapToItem(newIndex, animated, fireCallback);
  }

  _getSlideInterpolatedStyle(index, animatedValue) {
    const { layoutCardOffset, slideInterpolatedStyle } = this.props;

    if (slideInterpolatedStyle) {
      return slideInterpolatedStyle(index, animatedValue, this.props);
    } else if (this._shouldUseTinderLayout()) {
      return tinderAnimatedStyles(
        index,
        animatedValue,
        this.props,
        layoutCardOffset
      );
    } else if (this._shouldUseStackLayout()) {
      return stackAnimatedStyles(
        index,
        animatedValue,
        this.props,
        layoutCardOffset
      );
    } else if (this._shouldUseShiftLayout()) {
      return shiftAnimatedStyles(index, animatedValue, this.props);
    } else {
      return defaultAnimatedStyles(index, animatedValue, this.props);
    }
  }

  _renderItem({ item, index }) {
    const { interpolators } = this.state;
    const {
      hasParallaxImages,
      itemWidth,
      itemHeight,
      keyExtractor,
      renderItem,
      sliderHeight,
      sliderWidth,
      slideStyle,
      vertical,
    } = this.props;

    const animatedValue = interpolators && interpolators[index];

    if (!animatedValue && animatedValue !== 0) {
      return null;
    }

    const animate = this._shouldAnimateSlides();
    const Component = animate ? Animated.View : View;
    const animatedStyle = animate
      ? this._getSlideInterpolatedStyle(index, animatedValue)
      : {};

    const parallaxProps = hasParallaxImages
      ? {
          scrollPosition: this._scrollPos,
          carouselRef: this._carouselRef,
          vertical,
          sliderWidth,
          sliderHeight,
          itemWidth,
          itemHeight,
        }
      : undefined;

    // 每个渲染的组件被View包裹，View设置指定的itemWidth
    const mainDimension = vertical
      ? { height: itemHeight }
      : { width: itemWidth };
    const specificProps = this._needsScrollView()
      ? {
          key: keyExtractor
            ? keyExtractor(item, index)
            : this._getKeyExtractor(item, index),
        }
      : {};

    // 这里通过包裹一层View的形式控制每个元素的宽度为itemWidth，并且包含了动画样式
    return (
      <Component
        style={[mainDimension, slideStyle, animatedStyle]}
        pointerEvents={"box-none"}
        {...specificProps}
      >
        {renderItem({ item, index }, parallaxProps)}
      </Component>
    );
  }

  /**
   * 配置ScrollView或FlatList需要的属性，如initialNumToRender
   * @returns
   */
  _getComponentOverridableProps() {
    const {
      enableMomentum,
      itemWidth,
      itemHeight,
      loopClonesPerSide,
      sliderWidth,
      sliderHeight,
      vertical,
    } = this.props;
    // 根据设置的宽度确定能显示的元素数量
    const visibleItems =
      Math.ceil(
        vertical ? sliderHeight / itemHeight : sliderWidth / itemWidth
      ) + 1;
    const initialNumPerSide = this._enableLoop() ? loopClonesPerSide : 2; // 每边初始渲染的数量，默认为2
    // 初始渲染的总数量，即可见元素数量 + 每边额外渲染的数量 * 2
    // 这个值将会直接给到FlatList
    const initialNumToRender = visibleItems + initialNumPerSide * 2;
    const maxToRenderPerBatch = 1 + initialNumToRender * 2;
    // 可视区域外最大能被渲染的数量，等于初始渲染数量 * 2 + 1
    const windowSize = maxToRenderPerBatch;

    const specificProps = !this._needsScrollView()
      ? {
          initialNumToRender: initialNumToRender,
          maxToRenderPerBatch: maxToRenderPerBatch,
          windowSize: windowSize,
          // updateCellsBatchingPeriod
        }
      : {};
    // 这些props交给ScrollView或FlatList
    return {
      decelerationRate: enableMomentum ? 0.9 : "fast",
      showsHorizontalScrollIndicator: false,
      showsVerticalScrollIndicator: false,
      overScrollMode: "never",
      automaticallyAdjustContentInsets: false,
      directionalLockEnabled: true,
      pinchGestureEnabled: false,
      scrollsToTop: false,
      removeClippedSubviews: !this._needsScrollView(),
      inverted: this._needsRTLAdaptations(),
      // renderToHardwareTextureAndroid: true,
      ...specificProps,
    };
  }

  /**
   * 主要配置一些样式属性，以及安装renderItem等回调
   * @returns
   */
  _getComponentStaticProps() {
    const { hideCarousel } = this.state;
    const {
      containerCustomStyle,
      contentContainerCustomStyle,
      keyExtractor,
      sliderWidth,
      sliderHeight,
      style,
      vertical,
    } = this.props;

    const containerStyle = [
      containerCustomStyle || style || {},
      hideCarousel ? { opacity: 0 } : {},
      vertical
        ? { height: sliderHeight, flexDirection: "column" }
        : // LTR hack; see https://github.com/facebook/react-native/issues/11960
          // and https://github.com/facebook/react-native/issues/13100#issuecomment-328986423
          {
            width: sliderWidth,
            flexDirection: this._needsRTLAdaptations() ? "row-reverse" : "row",
          },
    ];
    const contentContainerStyle = [
      vertical
        ? {
            paddingTop: this._getContainerInnerMargin(),
            paddingBottom: this._getContainerInnerMargin(true),
          }
        : {
            paddingLeft: this._getContainerInnerMargin(),
            paddingRight: this._getContainerInnerMargin(true),
          },
      contentContainerCustomStyle || {},
    ];

    const specificProps = !this._needsScrollView()
      ? {
          // extraData: this.state,
          renderItem: this._renderItem,
          numColumns: 1,
          keyExtractor: keyExtractor || this._getKeyExtractor,
        }
      : {};

    return {
      ref: (c) => (this._carouselRef = c),
      data: this._getCustomData(),
      style: containerStyle,
      contentContainerStyle: contentContainerStyle,
      horizontal: !vertical,
      scrollEventThrottle: 1,
      onScroll: this._onScrollHandler,
      onScrollBeginDrag: this._onScrollBeginDrag,
      onScrollEndDrag: this._onScrollEndDrag,
      onMomentumScrollEnd: this._onMomentumScrollEnd,
      onResponderRelease: this._onTouchRelease,
      onStartShouldSetResponderCapture: this._onStartShouldSetResponderCapture,
      onTouchStart: this._onTouchStart,
      onTouchEnd: this._onScrollEnd,
      onLayout: this._onLayout,
      ...specificProps,
    };
  }

  render() {
    const { data, renderItem, useScrollView } = this.props;

    if (!data || !renderItem) {
      return null;
    }

    const props = {
      ...this._getComponentOverridableProps(),
      ...this.props,
      ...this._getComponentStaticProps(),
    };

    const ScrollViewComponent =
      typeof useScrollView === "function" ? useScrollView : AnimatedScrollView;

    return this._needsScrollView() ? (
      <ScrollViewComponent {...props}>
        {this._getCustomData().map((item, index) => {
          return this._renderItem({ item, index });
        })}
      </ScrollViewComponent>
    ) : (
      <AnimatedFlatList {...props} />
    );
  }
}
```
