百亿补贴项目总结

# 技术方面

## 功能实现

### 轮播图

- 实现方式：轮播图组件[react-native-snap-carousel](https://github.com/meliorence/react-native-snap-carousel)

- 关键点

  - 组件引入：由于组件是 react native 的组件，因此不能直接在 max 组件内引入。在项目中，需要通过导入到**src/mix/Components/index.mrn.ts** 中再导出才能使用。并且导入不能使用@符

    ![](https://pic.imgdb.cn/item/64a012631ddac507ccfc59dc.jpg)

  - 组件使用：基本的 props 传入参考官方文档即可。因为项目需要轮播图视频，因此我们需要知道当前正在展示的是哪个图，renderItem 方法可以拿到对应的 index。然后再将其赋给 state，就可以实现当正在渲染的 index 改变时，轮播组件更新，触发展示元素更新，从而从商卡换为视频。

    ![](https://pic.imgdb.cn/item/64a012771ddac507ccfc7688.jpg)

    ![](https://pic.imgdb.cn/item/64a012831ddac507ccfc89ee.jpg)

  - 出现问题和解决：

    主要问题有：

    - 有一个 loopClonesPerSide 属性，用于控制循环时每次在尽头增加的元素个数。这个属性会导致初始化渲染的数量比元素数量多。比如一共有 8 个元素，当这个属性设置为 8 时，第一次就会渲染出 16 个组件来。这会导致 renderItem 的参数 index 变成了一个[0,16)范围内的数，而非[0,8)。所以这里 realIndex 判断当前正在激活的元素时就采用的是 index - data.length，否则可能同时激活两个组件（比如第 1 个和第 9 个，第 2 个和第 10 个，依次类推）

  - 组件优化

    轮播组件需要优化的地方很多。对于组件的每个元素，这个库会将其初始化全部渲染出来，然后后续每次只会更新两个元素（一个元素 active->inactive，另一个元素 inactive->active），其他的元素只是在改变位置。但即使是两个元素的更新，仍要保证尽可能减小渲染消耗；在项目中主要有：

    - 每个子元素组件（Card）使用 memo 包裹，当然这个是最基本的
    - Card 子组件使用的对象类型的 props，用 useMemo 包裹，还是减少子组件渲染
    - removeClippedSubviews 属性，用于删除视图之外的项目，使得轮播图只会在内存中保存可视区域的子组件。
    - 使用虚拟列表的 onEndReached 和 onEndReachedThreshold，实现懒加载，类似虚拟长列表的用法。轮播组件默认是没有做这个的，我们可以把实际数据分块（比如一共 50 个商品，分成每 10 个一组），然后每次触发时额外加载一块即可。

### 轮播图视频

- 实现方式：在轮播图基础上，将轮播图卡片替换为视频组件。

- 关键点：

  - “替换”的方式。项目中采用的商品卡片和视频组件的交替显示的方式是利用修改 z-index 来控制元素的显隐，而非直接使用条件渲染控制渲染的商卡还是视频。这个方案的好处在于切换时不需要重新渲染组件，只需要控制 z-index 即可，性能比条件渲染好；同时比较便于实现显隐动画。而且由于轮播商卡需要比较频繁的切换，因此 zindex 是一个更好的方案。

  - 视频组件单实例。由于视频组件很庞大，占用内存很大，因此页面上同时最多只能有一个组件实例。因此需要及时销毁不在展示中的视频组件，当视频重新激活时再创建。这里其实有很多扩展考虑，比如当组件不在可视区域时将视频组件销毁掉，销毁的时机应该是轮播组件开始移动的时刻等

  - 声音控制。这里其实是比较麻烦的地方，主要分为两个部分

    - 声音倒计时。即倒计时 3 秒钟，时间到了之后启动动画，将元素收起，然后关闭视频静音，之后所有的视频都不再静音；如果三秒之内离开当前视频，就重置倒计时。

      这个部分其实是一个比较常见的倒计时需求，这里学习一下树哥的代码。

      1. 首先，倒计时的基本实现应该是 setInterval + useState，每 1 秒钟修改一次 state。当 state=0 时，执行收起动画，然后关闭静音，**清除 interval**

         ```js
         useEffect(
           () => {
             // 静音状态下且当前是可见选中状态，开启倒计时
             if (
               videoMuteFlag &&
               isActive &&
               zoneVisible &&
               countDownTipVisible &&
               !countDownTimeRef.current
             ) {
               countDownTimeRef.current = setInterval(() => {
                 setCountDownTime((time) => {
                   const nextCount = time - 1;
                   if (nextCount <= 0) {
                     clearCountDown();
                     triggerVideoMuteFlag(false); // 关闭静音
                     triggerVoiceTip(false).then(() => {
                       // 执行关闭静音条的动画
                       triggerCountDownTipVisible(false); // redux，修改静音图标状态
                     });
                     return 0;
                   }
                   return nextCount;
                 });
               }, 1000);
             }
           },
           [
             //...
           ]
         );
         ```

      2. 如果用户在倒计时结束之前就点击了静音按钮，那么应该立即清除倒计时，执行收起动画，然后取消静音。因此这里控制的函数内部应该需要清除倒计时的 interval。同时还要注意，如果倒计时已经结束，那么这里就不再需要清除倒计时，只需要控制声音即可。

         ```tsx
         const triggerVideoVoice = async () => {
           if (countDownTipVisible) {
             // 清楚倒计时
             clearCountDown();
             // 执行关闭动画
             await triggerVoiceTip(false);
             // 标注用户关闭静音，全部的静音条都要
             triggerCountDownTipVisible(false);
             triggerVideoMuteFlag(true);
             return;
           }
           // 只需要关注静音开关即可
           triggerVideoMuteFlag(!videoMuteFlag);
           onMuteButtonMC(videoMuteFlag);
         };

         ...

         <View
           style={styles.voiceBox}
           onClick={(e) => {
             triggerVideoVoice();
           }}
         >
         ```

      3. 如果视频在倒计时结束前切走，那么 triggerCountDownTipVisible 不会被执行，全局状态 countDownTipVisible 不会更改，因此下一个视频组件被创建时，仍然按照第一种情况下开启倒计时。

         但如果在倒计时结束之后切走，那么 triggerCountDownTipVisible 被执行，全局状态 countDownTipVisible 被修改为 false，下一个视频不会再显示倒计时

         ![](https://pic.imgdb.cn/item/64a012b21ddac507ccfcd413.jpg)

    - 动画。动画主要在于状态框的收起，基本思路其实就是利用 Animated 实现在一段时间内修改展示组件的 width。

      ```js
      // 声音提示栏动画
      export const useVoiceTipAnimation = (
        videoMuteFlag,
        countDownTipVisible
      ) => {
        const voiceAnimationRef = useRef(
          new Animated.Value(VOICE_ANIMATION_WIDTH)
        );
        const triggerVoiceTip = useCallback((visibleFlag) => {
          return new Promise((resolve) => {
            Animated.timing(voiceAnimationRef.current, {
              toValue: visibleFlag ? VOICE_ANIMATION_WIDTH : 0,
              useNativeDriver: false,
              duration: ANIMATION_DURATION,
            }).start();
            setTimeout(() => {
              resolve(true);
            }, ANIMATION_DURATION);
          });
        }, []);

        const videoVoiceIcon = useMemo(() => {
          if (countDownTipVisible) {
            return { uri: VOICE_ICON_MUTE };
          }
          return { uri: videoMuteFlag ? VOICE_ICON_MUTE : VOICE_ICON };
        }, [videoMuteFlag, countDownTipVisible]);
        return {
          triggerVoiceTip,
          videoVoiceIcon,
          voiceAnimationStyle: { width: voiceAnimationRef.current },
        };
      };
      ```

      这里有几个其他关键点：

      - 这里动画用了 promise 包裹，目的是希望改变状态的操作是异步执行的，即在动画完成之后再执行。当然这里动画完后执行一个 sleep 也是可以的，不过现在这种写法更加直观；
      - 动画的写法，这个需要后面再学习。

### 导航栏渐变

- 实现方式：通过监听页面滚动的 onScroll 事件，计算偏移量；通过全局事件的形式传递数据，然后通过动画修改导航栏颜色百分比。

- 关键点：

  1. 传递 onScroll 事件的滚动偏移量时，采用的还是全局事件的方式，而非 redux。原因和之前的一样，减少无用的重复渲染。

  2. 修改导航栏颜色采用的是 Animation 动画，而非修改状态。这个考虑同样是降低渲染次数，因为动画组件的更新不会影响到其他组件；并且将背景颜色单独抽成一个组件，同样也是在最大程度降低渲染消耗，防止子元素重复渲染。

     ![](https://pic.imgdb.cn/item/64a012e31ddac507ccfd1e2e.jpg)

     ![](https://pic.imgdb.cn/item/64a012ee1ddac507ccfd30a7.jpg)

     ![](https://pic.imgdb.cn/item/64a012fe1ddac507ccfd48c7.jpg)

### 背景图放大

- 实现方式：和上面的方式一样，监听滚动事件，只不过本次的是负值。然后通过动画放大图片即可。

## 代码质量

### 代码简洁和清晰

- 注意冗余代码。冗余代码主要体现在于 hooks 之间的联动，经常把数据在一些 hooks 之间进行没有必要的传递和捕获。常见的几个问题有：

  - props 以及 props 的衍生值没必要再用 state 传递。比如商卡组件中商卡角标判断，用于判断的值本质上都是来自于 props.dealInfo 的，因此直接判断 dealInfo 的子属性的相等关系即可，不需要再创建一个状态。

  - 减少 state 的数量。并非是所有变化的值都需要 state 包裹，大多数值通过 useMemo、useRef 即可。注意 redux 的值也算 state，不要出现把 redux 的值和其衍生值又赋给新的 state 这种情况。

    比如计算 navColors 的代码中，需要的 tabTopOffset、scrollOffset 来自 redux，计算后又赋给了新的 state percent，然后再从 percent 计算出最后的颜色。这个过程其实完全可以简化为：用 offset 直接计算出 percent 得到颜色，并不需要一个颜色状态。（实际上最后使用的是 Animation）

### 逻辑整合

逻辑整合的主要应用在于多种条件判断的整合。当某个条件需要多条综合判断时，就需要对其进行合理的封装和整合，然后把所有判断抽离成一个入口，在业务组件内部只通过这一个入口进入。

比如，关于展示会场类型的判断，两个会场（简称 E 和 B）分别判断的位置，一共有多种情况：

- 数据合法，比如场次合适，数据数量合适等。E 和 B 判断数据合法的逻辑是不同的
- 模板合适，这个字段来自接口，配置了模板才能显示对应的会场
- 是否登录，不登录都不能显示

所以我们可以做这样的整合：

1. 对于第一和第二个，分别写一个函数用于表示一种情况，一共四个函数。比如 isEatFullDataValid、isBillionDataValid、isEatFullTemplate、isBillionTemplate 这四个

2. 整合第一步这四个函数，一共有两种情况

   1. 展示 E，此时为 isEatFullDataValid && isEatFullTemplate
   2. 展示 B，此时为 isBillionDataValid && isBillionTemplate

3. 整合之后返回一个对象，同时返回这两个情况，即

   ![](https://pic.imgdb.cn/item/64a013111ddac507ccfd652b.jpg)

4. 最后整合一下 login。因为 login 的判断是需要 hooks，所以需要把前面的放到 hooks 中，然后加上 login 的判断即可

这种方式的核心在于，把每个条件打散到每个函数中，再整合到一起，最后返回一个对象或数组这样的形式即可。这样**不管在哪里使用都只需要调用这一个函数**即可。

## 代码错误和检查

### 边界条件和代码 bug

这次项目中主要出现的比较明显的 bug 有：

- useCallback 的使用，主要有两点
  - 一定不要直接在 props 上使用，如果该组件是条件渲染的，就会导致 hooks 在条件语句中使用的问题。为了避免这个文件就要保证不要在 props 中使用
  - 如果要用，就一定要把 deps 数组设定好，不要漏掉关键变量；useCallback 的 deps 数组不包括函数参数，但除了参数之外的 state、ref 等值都要加入
- 注意事件的安插位置。有些元素会阻止事件冒泡，或者他上面不能安插 onclick 这样的回调。典型的比如 Sensor、ImageBackground 等组件。注意把事件安插的低一些。
- 在样式中，写死的高度要注意不同设备的适配情况。在大多数情况下，由于 style 是经过 remStyleSheet 包裹的，不一定会注意到这个问题；但是当某些高度、宽度的固定数值在组件中计算时，就记得需要 rem 包裹。比如本次项目的 nav 高度，虽然写死是某个值，但是由于不同设备的问题，我们计算时仍然要以 rem 包裹一下，不然这个值就不会是实际高度。
- 不同文本横向对齐问题（比如价格文本），有个解决的关键点是给每个元素设置好 lineheight，然后再尝试用 baseline 等方式
- Sensor 尽量少用。尤其是对于某个元素的宽高测量，不要使用 Sensor 去量，有条件可以用 rem+固定值这种方式。Sensor 测量值有可能为空，一定要考虑兜底。在项目中，tabTopOffset 的测量经常为 0，这时需要一个固定值去兜底，比如以有效的测量值 340 作为兜底。
- 注意边界条件的考虑和控制。本次出现的边界条件导致的问题主要包括：

  - 兜底值。不仅是接口返回值的兜底，更多的是一些业务逻辑的兜底。比如 nav 颜色，当计算出错或接口请求为空时应该有个颜色的兜底。
  - 计算值。尤其是在数学运算中，要考虑到 NaN、Infinity、负数、0、分母为 0、null 等多种情况
  - 条件判断。这个主要体现在业务逻辑中，即一个条件判断的完整性。比如项目中对于专区展示条件的判断，除了考虑模板要求、数据合法之外，是否登录也要考虑。之前少考虑了登录问题，导致出错。
  - 可选符。不要忽略任何一个可选符。对于接口参数，一定要尽可能全部加上可选符，同时 interface 中也要注意保持属性类型为可选。
  -

- 函数参数修改。如果需要修改一个函数参数，就一定要保证所有调用该函数位置的传参也要修改。建议修改之后使用 yarn tsc --strict 检查一下，防止参数问题在犄角旮旯出现而没发现。

## 功能优化和代码优化

本次项目中主要的优化方向在于减少无意义的渲染。主要方向有：

- 上面说过的，减少 state 的使用，避免反复使用 state

- 对于高频事件，尽可能不要修改 state 或 redux。

- 不需要 redux 时就尽量不用 redux。大多数时候使用 redux 就只是为了共享数据，但问题在于 redux 的数据是 state，会有重复渲染的风险。典型的例子就是 scroll 事件触发 redux 内的状态更新，从而导致使用属性的组件被频繁更新，然后导致轮播组件闪动。这种方式可以优化为采用其他方式共享数据，比如项目中使用的全局事件（类似 Pubsub），然后再通过 Animation 等方式实现

- 降低 state 的影响范围。还是 nav 组件，如果直接把颜色状态更新放到整个 nav 组件上，那么这个状态频繁更新势必会导致 nav 整体的重新渲染。但我们可以将背景颜色单独抽成一个组件，只更新这个组件内部的状态即可，这样就可以尽可能降低重复渲染的影响范围。

其他优化点还有一些性能优化。这些性能优化可以参考之前代码的做法，比如：

- 减小图片大小，设置低分辨率兜底图。在图片加载之前，利用骨架屏、纯色背景等方式进行兜底

- 分散请求，这个和本次项目无关，但是和会场刷新有关。

  > 以 18:00 场次刷新为准。
  >
  > 1. 将结束时间延后 2s，并将这三秒，分为随机 20 份。2000ms，每份 100ms。
  > 2. 用户随机命中 20 份延迟中的一份。如果用户命中 100ms 那份策略，其刷新场次时间为 18:00.100
  >
  > 前端处理策略
  >
  > 1. 接口返回当前场次结束时间后，前端进行分块，判断当前用户应该延迟多久，然后加在结束时间上
  > 2. 如果用户命中 1000ms 策略，结束时间自动延后一秒，前端倒计时会相较于正常整体延后一秒，用户看到倒计时与现有样式一致
  > 3. 延迟判定逻辑，以用户每次进入会场为准。
  >
  > 举个 🌰
  >
  > 1. 用户 17：50 进入会场，命中 1000ms 延迟策略。倒计时会显示 0:10:1 结束
  > 2. 用户 17：51 退出会场
  > 3. 17：57 进入会场，重新计算延时策略，如果命中 100ms 延时策略，倒计时会显示 0:3:0 结束
  > 4. 用户在会场等待，时间来到 18:00。倒计时显示 0:0:1 结束。18:00.100 倒计时显示 0:0:0 前端发送请求

## 未应用的点

项目中有一些设想，在实际项目中未应用，可以作为可尝试的选择。

1. 视频组件的固定：由于当前视频组件本质上是反复创建与销毁，不在 active 状态的视频组件会被销毁而其他元素保留。这种方式有可能造成一些性能开销。

   因此可以考虑将视频组件固定在屏幕中间，当轮播图停止后更新视频源并重置视频进行播放，轮播过程中通过 z-index 隐藏视频。这样视频组件一直存在而不会反复创建和删除，同时也可以控制保存视频状态。

2. 轮播图和视频的懒加载：具体参考优化部分的内容。由于轮播图和视频组件很大，导致首屏加载时间变慢，因此考虑放弃对这两个组件的首屏加载，而是采用商卡提前占位，延迟加载轮播图和视频，加载完成后再进行播放。

3. 懒加载的另一种方法：通过 Priority 组件降低 Carousel 的渲染优先级，使其延后渲染，详见[Priority 组件](https://sky.sankuai.com/max/components/functional/max-priority)的使用
