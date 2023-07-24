# 超管项目

遇到问题：

- 代码问题
  - 写好注释
  - 代码规范，变量、函数名，eslint
- 需求问题
  - 部分功能实现困难，比如 canvas 地图的点位和比例计算：理清思路，多做推理和演算
  - canvas 绘制复杂，很难处理事件：采用 konva
  - 组件多，数据杂，context 不合适：先使用 redux，但类型不好且冗余代码过多：recoil

项目重点：

- 技术选型，即
  - konva 和其他 canvas 框架的技术选型，以及 canvas 本身的 api、性能优化相关
  - recoil 和其他状态管理库的技术选型（可以说的：redux、mobx、zustand、joita 等），从哪些方面考虑，这些状态管理库的通用特点有哪些。（可以参考那篇飞书文档）
- 项目规范
  - 代码可读性、规范性、复用性各方面

## 遇到问题

### 代码规范

规范主要有四类规范：

- （文件）命名规范
- ts 规范
- React 规范
- git 规范

规范来源：

- 部门规范
- 公司内规范
- 个人建议/自用规范

#### 命名规范

1. 文件名、目录：

- kebab-case 形式。语义要明确，比如 modal 组件的文件名称都叫 modal-xxx
- 不要简写
- 目录要对于复数结构采取复数形式。

#### ts 规范

1. 类的私有方法前要加 private 和下划线
2. 接口 PascalCase，前面要加 I
3. 枚举 PascalCase
4. 常量使用 UPPER_CASE ，禁用魔法数字和魔法字符串
5. 注释：

需要为以下内容添加注释

- class
  - 需要在 class 的顶部添加注释，说明该类的主要作用，比如为 React class/hooks component 添加注释
  - 需要为 class 中的每个成员变量添加注释说明字段含义，无论是 public、protected、private
  - 需要为 class 中的每个函数添加注释，具体参见下方 function
- enum
  - 需要为枚举添加注释，以及为枚举中的每个成员添加注释说明
- interface
  - 需要为 interface 添加注释，并为其中的字段添加注释说明，如果该 interface 对应了后台接口，还需要链接相关接口文档
- function
  - 需要注释说明函数的作用，每个入参的含义并给出相关示例，说明返回值含义及给出示例，对于复杂的逻辑，需要给出该函数实现的核心逻辑思想
  ```ts
  /**
   * 返回两个数的平均值 ---- 函数作用
   *
   * @param x 第一个数，例如 1
   * @param y 第二个数，例如 2
   * @returns “x”和“y”的算数平均值
   */
  function getAverage(x: number, y: number): number {
    return (x + y) / 2.0;
  }
  ```
- 全局 let/const
  - 需要为 class 或 function 外部的 let/const 变量添加注释

格式遵循 TSDoc

6. 尽量不使用 any，可以用 unknown 代替
7. 字符串拼接统一使用模版字符串
8. 使用 interface 定义对象结构体，而不是使用 type，对于 interface 接口名前要有 I
9. 使用 async/await 代替 Promise，并且避免 async/await 与 Promise 混用
10. 使用统一的 import 顺序，按 内置库 → 外部库 → 内部模块 的顺序进行 import。

#### react 项目规范

1. useCallback、useMemo、React.memo 一起结合使用才能减少不必要的子组件渲染，分工如下：

- 父组件：在父组件中应该使用 useCallback() 和 useMemo() 尽量确保相应的 fn 或 value 不变，这样才能确保传递给子组件的 prop 不变。
- 子组件：在子组件中应该使用 React.memo() 包裹原始的子函数组件，从而可以通过利用浅比较 props 的方式尽量使得子组件在 prop 没有发生变化时不进行渲染。

2. 不要在 JSX 中写 JavaScript/TypeScript
   在 JSX 中写代码，会让调试变得困难。(这个可以详细说一下)

```js
// Bad

return (
  <>
    {users.map((user) => (
      <a
        onClick={(event) => {
          console.log(event.target); //bad
        }}
        key={user.id}
      >
        {user.name}
      </a>
    ))}
  </>
);

// Good

const onClickHandler = (event) => {
  console.log(event.target);
};

const userLinks = users.map((user) => (
  <a onClick={onClickHandler} key={user.id}>
    {" "}
    {user.name}{" "}
  </a>
));

return <>{userLinks}</>;
```

3. husky：确保在提交代码时也能够运行 lint 检查

#### git 规范

按照 git 工作流的形式，主要是四个分支

![](https://pic.imgdb.cn/item/641835e4a682492fcc153b12.jpg)

- master：
  - 所有线上的发布都必须基于 master 分支，不能基于 release、feature、hotfix 分支
  - 无论是 release 还是 hotfix 合并到 master，在 master 分支合并后，需要基于 package.json 中的 version 对 master 打 tag，注意只能对 master 分支打 tag，不能对 release、feature、hotfix 分支打 tag
  - 如果在某一时刻同时有 release1 和 release2 两个分支在开发，当 release1 测试通过并 merge 到 master 后，需要将 master 再 merge 回 release2，确保 release2 最新
- Release：
  - 从 master checkout 创建
  - 创建后更改 package.json 的 version，version 的组成形式为 majorVersion.minorVersion.patchVersion，创建 release 分支时应该更改 minor version，比如从 0.5.1 改为 0.6.0
  - release 分支是受保护分支，严禁直接提交代码到 release 分支，只能通过 MR 对 release 代码进行变更
  - 当 feature 分支完成一块相对独立的功能时，就可以从 feature 分支 发起 merge request 到 release 分支
  - 提测前需要确保所有 feature 分支的代码已经合并进 release 分支，然后将 release 分支部署到 boe 环境
  - 发布到线上前将 release 分支 merge 到 master，此处的 code reveiw 只需要大体过一下即可，因为 diff 内容都在 feature 往 release 分支合并时仔细 review 过了
  - 如果某一个时刻基于 release1 分支有 feature1 和 feature2 两个分支，当 feature1 通过 code review 合并到 release1 后，也需要将 release1 合并回 feature2
  - 需要注意的是，机器人前端项目每次给 QA deb 包进行测试前，都首先要确保 package.json 中的 version 变化了，因为机器人前端项目目前是根据 package.json 中的 version 生成 deb 包的版本
- Feature
  - 所有的 feature 分支都应该从 release 分支 checkout 创建，即便这次 release 分支只有一个开发人员也应如此，feature 分支需要包含个人名称简写
  - 提测前将 feature 分支 merge 到 release 分支，此处的 MR 需要严格 code review
- Hotfix
  - 从 master checkout 创建，然后需要修改 package.json 中的 version，创建 hotfix 分支时应该更改 patch version，比如从 0.6.0 改为 0.6.1
  - 修改完成后创建 merge request 到 master
  - 注意在将 hotfix 分支合并到 master 分之后，记得更新 release 分支以及 feature 分支

#### eslint

使用`eslint --init`即可初始化配置。

配置内容

- eslintrecommended、reactrecommended、tsrecommended
- @byted/eslint-config：公司内部的 eslint 推荐配置
- 自定义的一些
  - 禁用魔法数字
  - useState 建议是`[xxxState,setXxxState]`的形式，以及 useRef 必须是 xxxRef 的形式，useReducer 必须返回 xxxReducer 的形式。这三个都是自定义的插件，参考工程化中编写插件的部分。

关于 plugin 的

#### husky

配置 husky 在 eslint 校验通过之前不允许提交。
[husky 文档](https://typicode.github.io/husky/#/)
[husky 配置](https://juejin.cn/post/7038143752036155428#heading-9)

husky 是一个操作 git 钩子的工具。可以使用预设的钩子，在 git 各种操作的时间进行补捕获和拦截。
比如，在 pre-commit 阶段，即提交之前先进行 lint 的检查，如果不通过就不允许提交。

> Git hooks 是一种在 Git 仓库中预定义的脚本，它可以在特定的 Git 操作时自动执行。Git hooks 可以在 Git 操作之前或之后执行一些自定义代码，比如在代码提交之前运行测试，或者在代码合并之前运行代码格式化工具等。
> 在 Git 中，每个仓库都有一个 .git/hooks 目录，其中包含了一些可用的 Git hooks 脚本模板。这些模板可以被复制并重命名为不同的钩子名称，然后在其中编写脚本代码来实现自定义操作。Git hooks 脚本必须是可执行的（即要有执行权限）。
> husky 的核心代码其实很简单，就是通过配置 git hooks 的方式来实现要在 git hooks 阶段执行的任务。

配置流程：

1. 安装 lint-staged、husky、commitlint
2. 配置 husky 在 pre-commit 阶段进行 line-stage 检查：

```
yarn husky add .husky/pre-commit "npx lint-staged"
```

当然还要配置.lintstagedrc.json 文件控制检查和操作方式

```json
{
  "*.{js,jsx,ts,tsx}": ["eslint  --fix"]
}
```

3. commitlint 配置 commit 规范
   [commitlint 文档](https://commitlint.js.org/#/?id=getting-started)

使用 husky 在 commit-msg 钩子上增加对 commit 的检查：

```
husky add .husky/commit-msg 'npx --no-install commitlint --edit "$1"'
```

创建 commitlint.config.js，使用预设：

```js
module.exports = { extends: ["@commitlint/config-conventional"] };
```

这个预设比较简单，就是要求提交前面必须是`fix/feat/doc`等。

### 代码复用性

- 拆分函数。做到“一个函数只负责一个功能”。比如说某个点击事件的回调，可能包括网络请求、改变状态、整理数据等多步，就把这几个不同的功能分别拆到不同的函数中。
- 自定义工具函数。
  - 对于可能重复使用的处理一段数据的代码，比如一个数组数据的 map、filter，抽象成一个函数
  - 判断函数，比如 isXXX 类型的函数，用于代替条件判断
  - 构造对象的函数，比如构造出的对象是某个函数的参数，或者是修改 state 的，将其抽象为一个函数甚至构造函数的函数。比如 xxxAction 和 createXXXAction
  - 举例：深比较、深格式化数据、计算地图和真实点位对应
- 自定义 hook：代码编写时，注意到某些 useState、useRef 以及 useEffect 的组合貌似是固定的，就将其抽出来成为一个自定义 hook。尤其是 useEffect。

  - useMap：用于接收一个地图 id，调用一个外部 api，然后返回地图 url：

  ```ts
  export const loadMap = (fileId: string): Promise<string> => {
    return new Promise(async (resolve, reject) => {
      // ...
      resolve(imageUrl);
      // ...
    });
  };

  export function useMapUrl(tenantId: string, mapId: string) {
    const [mapUrl, setMapUrl] = useState("");
    useEffect(() => {
      if (mapId && mapId !== "0") {
        getMapDetail(mapId, tenantId, false)
          .then((map) => loadMap(map.png_file_id))
          .then(setMapUrl);
      }
    }, [mapId, tenantId]);
    return mapUrl;
  }
  ```

  - usePolling：轮询，虽然有现成的，但是自定义的原因是为了实现当 deps 变为 false 就停止轮询这个功能。

  ```ts
  export const usePolling = (
    callback,
    time: number,
    pollingDep?: boolean,
    isImmidiate = true
  ) => {
    const deps = pollingDep != null ? [pollingDep] : [];
    useEffect(() => {
      let interval = null;
      if (pollingDep === false) {
        clearInterval(interval);
        return;
      }
      if (isImmidiate) callback();
      interval = setInterval(() => {
        callback();
      }, time);
      return () => clearInterval(interval);
    }, deps);
  };
  ```

  - useImage：将 file 格式图片转为 base64，这个也很简单，在 useEffect 中调用 fr 即可：

  ```ts
  export function useImgBase64Url(img: File) {
    const [url, setUrl] = useState<string>("");
    useEffect(() => {
      if (img) {
        const fr = new FileReader();
        fr.readAsDataURL(img);
        fr.onload = () => {
          setUrl(fr.result as string);
        };
      }
    }, [img]);
    return url;
  }
  ```

- 分割组件

  - 将具有自己独立状态的组件分割。比如一个 modal 内的表单，一个 drawer 内的 table 等，尽量让组件自己维护一些状态，不要全提升到父组件中
  - 要复用的组件，比如四个 Modal，这四个 modal 虽然内部不一样，但是他们需要的外部数据基本一致（visible,robotData），内部结构也大体一致（都是一个 Alert + 一个表单），因此把表单抽成四个组件，Modal 就使用一个组件。

  如下所示，Modal 组件用 React.cloneElement 给子组件注入 props，这样子组件始终含有 robotData 等关键数据。子组件通过修改父组件 ref 的形式控制 modal 点击确定时提交表单（这里应该有更好的方式）

  ```tsx
  export function MyModal({
    robotData,
    visible,
    setVisible,
    children,
  }: ModalPropsType) {
    const onOk = useRef<(e?: MouseEvent) => Promise<any>>(async () => {
      setVisible(false);
    });
    return (
      <Modal
        visible={visible}
        onCancel={() => setVisible(false)}
        onOk={onOk.current}
      >
        {React.cloneElement(children, {
          // 注入props
          onOk,
          robotData,
          setVisible,
        })}
      </Modal>
    );
  }

  // MyModal不改变，只改变CreateUserForm即可。
  export function CreateUserForm({
    onOk,
    robotData,
    setVisible,
  }: CreateUserFormPropsType) {
    const [form] = Form.useForm();
    useEffect(() => {
      onOk.current = async () => {
        const res = await form.validate().catch(console.error);
        console.log(res);
        setVisible(false);
      };
    }, []);

    return (
      <>
        <Alert type="warning" content={`${robotData.basic.device_name}`} />
        <Form form={form}>
          <Form.Item field="username">
            <Input type="text" max={20} />
          </Form.Item>
        </Form>
      </>
    );
  }
  ```

- 代码规范：变量命名规范、函数命名规范、类型命名规范、导入语句、组件名、组件文件名等。
  - eslint 限制代码规范：默认的 extends 和 plugin，比如 react、babel、@typescript-eslint 等
  - 自定义 rules：
    - 禁用魔法数字
    - camelCase 强制变量名和对象属性为小驼峰
    - eqeqeq，配置`{"null": "ignore"}`
  - eslint-plugin-filenames 可以限制文件名和目录名。比如：
  ```json
  {
    "plugins": ["filenames"],
    "rules": {
      "filenames/match-regex": ["error", "^[a-z]+(-[a-z]+)*$", true] // 限制文件名为kebab-case形式
    }
  }
  ```

### 需求问题

##### canvas 问题

首先是计算地图坐标和 canvas 坐标，相互转化
主要有以下几点：

1. 地图是一张图片，通过 drawImage 绘制到 canvas 上。这个过程中，绘制的大小，即传入的绘制宽高会导致图片放缩。所以要记录比例 scale
2. 接口给出了地图的真实宽高、真实坐标原点的 xy 值，以及真实点位的坐标 xy 值。从真实坐标计算到 canvas 坐标就是`(真实点位x - 真实点位原点x)/比例尺 * 缩放比例`。

这个地方比较抽象，不容易想到

还有就是获取 canvas 上鼠标点击事件的坐标，基本方法是用到了 getBoundingClientRect

```js
const canvas = document.getElementById("myCanvas");
const ctx = canvas.getContext("2d");
function handleClick(event) {
  const rect = canvas.getBoundingClientRect();
  const x = event.clientX - rect.left;
  const y = event.clientY - rect.top;
  console.log(`Clicked at (${x}, ${y})`);
}
canvas.addEventListener("click", handleClick);
```

##### konva

- 为什么选用？
  - 减少 canvas 代码，直接用命令式绘制，或者使用 react-konva，直接采用组件的形式，像写 html 一样
  - 解决事件问题。这也是不选用其他库的一个原因，konva 有独特的事件系统。
    其他选项有：
    - Fabric：和 konva 类似，也有事件系统，但他和 react 的结合不如 konva 这种组件形式。
    - react-canvas：这个库非常老，已经没有人维护了
- 怎么用？
- 什么原理

##### recoil 和 redux

- 为什么用：组件拆分，零碎组件多，不用的话状态难以共享，context 太复杂
- redux：框架自带。主要有一些问题：
  - reducer 代码庞大。一个 reducer 只能维护一个状态，因此很多零散的状态就需要很多 reducer。如果把他们合并到一个 reducer，看起来很乱，并且会导致无谓的更新
  - 组件内很难用。dispatch 传入的 action 不能得到类型，需要不停地返回去查看 action.type 名称；后来封装了创建 action 的函数，但是导致了更多的代码
- recoil：自行选择。主要目的：
  - 代码简洁
  - 解决状态零碎问题，atom 本来就是推崇零碎状态。

## 学到知识

- 技术选型：状态管理库的了解，redux、mobx 的原理、recoil 的优势，其他状态管理库之间的优劣比较
- konva 的使用，konva 的事件原理和设计模式，react-konva 和 react 的结合方式
- canvas api，canvas 基础功能的实现，canvas 优化
- 代码规范，四个方面的代码规范
- 工程化代码规范的方式（eslint、husky）

## 项目难点

### 1. 事件冒泡规则

问题：
在项目中确实遇到了这样的问题：有一个这样的需求，即显示一个实时地图，实时地图内有很多点位，希望点击这些点位时，能获取到具体点的是哪个点位。
这是一个常见的事件委托的需求，但是需要事件冒泡作为基础。由于 konva 的事件系统是自己模拟实现的，因此与 dom 的事件机制不一样，在使用时出现了一些问题。

个人尝试：

- 尝试把 onclick 事件放到实时地图的矩形上，发现虽然地图表现为“包裹”着点位，但是实际上不能触发冒泡
- 然后猜测可能是跟先后顺序有关，尝试后也无效。
- 最后猜测应该把实时点位作为地图的矩形的子元素，或者用一个大的元素包裹这两个元素，都不行
- 如果把事件绑定在 layer 或 stage 上，这两个上面已经有其他事件的绑定了，并且实时地图和实时点位是选择渲染的，不一定有，因此希望把事件绑定在实时地图容器上，而不是绑定在顶层。

查阅资料：

- 官方文档，有讲事件的部分，但是没有详述冒泡的规则，并且 react-konva 文档中也没有说事件流的相关概念
- github issues：查找有关事件冒泡的 issues，没有，然后查找其他关于事件系统的 issues，也没有
- 最后，查看源码
  - 先是了解 konva 事件系统的实现原理
    - 调试源码
    - 搜一些资料，查看事件原理
  - 然后搞清楚事件冒泡的机制：fireAndBubble，以及查找的“parent”到底是谁
  - 最后了解到每个元素都有一个 parent，但是 parent 只在 Container 中被设置，并且必须调用 add 方法。Container 只有 stage、layer 和 group，普通元素不行。

解决方案

- 在 Container 上放置事件监听。使用 Group 把实时地图和实时点位包裹起来，然后在 Group 上监听事件，即可。
- 同时还了解到了取消事件冒泡的机制，在 Group 上通过 cancelBubble 防止事件进一步冒泡

### 2. 地图拖动

md，konva 已经有拖动和放大功能的实现了：
https://konvajs.org/docs/sandbox/Canvas_Scrolling.html
https://konvajs.org/docs/sandbox/Zooming_Relative_To_Pointer.html

问题
地图的底图比实际的 canvas 大小大很多，如果想显示完整的地图就有两种方式

- 创建一个和底图一样大的 canvas，然后通过 css 的滚动条进行移动
- canvas 大小固定，通过拖动的方式移动地图

第一种的问题：过大的 canvas 会导致性能出现问题，比如分辨率不足、渲染负担重等问题。
解决方式：canvas 大小固定，改变渲染区间。

具体做法：

- 首先创建固定大小的 canvas，然后把整个底图渲染上去。canvas 的大小不变，超出的区域是没有渲染的区域
- 实现拖动查看。具体功能点有
  - 基本的拖动，通过监听 mousemove 等事件，修改 canvas 的 translate，然后重新渲染，实现拖动效果
  - 边界控制，不能让拖动超出底图的范围，因此拖动时计算边界，如果拖动时 offset 超出边界范围就不发生移动。
  - 可视区域渲染：渲染所有点位时，先通过他们的坐标判断是否在可视区域，如果在才渲染，否则不进行渲染。

拖动的实现

基本思路：

1. 记录一个 offsetX 和 offsetY，表示 canvas 的 translate 值。每次绘制，都先`translate(offsetX,offsetY)`，然后执行绘制
2. 监听 canvas 的 mousedown、mousemove 和 mouseup 事件，当 mousemove 执行时，获取实时的鼠标坐标，和 mousedown 时获取的鼠标坐标的差值就是偏移量，即 offsetX 和 offsetY。
3. 通过修改之后的 offsetX 和 offsetX，如第一步操作。

另外：react-konva 中的实现：

- 其他方式基本不变，但是不需要自己手动绘制，只需要修改 layer 的 x 和 y 即可。

```js
const canvasRef = useRef < HTMLCanvasElement > null;
const mousePos = useRef([0, 0]);
const mouseOffset = useRef([0, 0]);
const currMouseOffset = useRef([0, 0]);
const scale = useRef(1);
const preScale = useRef(1);

// 检查点位是否在可视区域内
const isShapeVisible = (shapeX: number, shapeY: number) => {
  const [originX, originY] = mouseOffset.current;
  const [top, left, right, bottom] = [
    -originY,
    -originX,
    -originX + width,
    -originY + height,
  ];
  return shapeX >= left && shapeX <= right && shapeY >= top && shapeY <= bottom;
};

const draw = (type: TranslateType) => {
  // 绘制前先translate
  ctx.clearRect(0, 0, width, height);
  ctx.translate(mouseOffset.current[0], mouseOffset.current[1]);
  // ...
  // 点位的数组，每一项绘制之前先检查当前点位是否在可视区域
};

const onMouseMove = useCallback((e: MouseEvent) => {
  // 获取移动的量，即当前鼠标位置减去初始鼠标位置
  const moveX = e.clientX - mousePos.current[0];
  const moveY = e.clientY - mousePos.current[1];
  // 边界控制
  const [x, y] = mouseOffset.current;
  if (x > 1 || y > 1) return;
  if (x < width - backgroundWidth || y < height - backgroundHeight) return;
  mouseOffset.current = [
    // currMouseOffset用于记录已有的偏移，每次拖动理应在上一次偏移的基础上偏移
    currMouseOffset.current[0] + moveX,
    currMouseOffset.current[1] + moveY,
  ];
  draw();
}, []);

const onMouseUp = useCallback((e: MouseEvent) => {
  // 记录本次的偏移量，下一次就以curr为基础。
  currMouseOffset.current[0] = mouseOffset.current[0];
  currMouseOffset.current[1] = mouseOffset.current[1];
  canvasRef.current?.removeEventListener("mousemove", onMouseMove);
  canvasRef.current?.removeEventListener("mouseup", onMouseUp);
}, []);

const onMouseDown = useCallback((e: MouseEvent) => {
  // 设置一个初始鼠标位置
  mousePos.current = [e.clientX, e.clientY];
  canvasRef.current?.addEventListener("mousemove", onMouseMove);
  canvasRef.current?.addEventListener("mouseup", onMouseUp);
}, []);

useEffect(() => {
  if (!canvasRef.current) return;
  const canvas = canvasRef.current;
  const context = canvas.getContext("2d");
  draw();
  canvas.addEventListener("mousedown", onMouseDown);
}, []);
```

地图缩放实现：https://konvajs.org/docs/sandbox/Zooming_Relative_To_Pointer.html

基本原理就是当wheel事件发生时，记录下来鼠标当前坐标，然后计算和原点的偏移量，除以旧的放缩比例。这个值乘新的放缩比就可以得到新的鼠标坐标在新的放缩后的画布上的位置。然后执行放缩，并将画布移动到新鼠标位置即可。

```js
var width = window.innerWidth;
var height = window.innerHeight;

var stage = new Konva.Stage({
  container: "container",
  width: width,
  height: height,
});

var layer = new Konva.Layer();
stage.add(layer);

var circle = new Konva.Circle({
  x: stage.width() / 2,
  y: stage.height() / 2,
  radius: 50,
  fill: "green",
});
layer.add(circle);

var scaleBy = 1.01;
stage.on("wheel", (e) => {
  // stop default scrolling
  e.evt.preventDefault();

  var oldScale = stage.scaleX();
  var pointer = stage.getPointerPosition();

  var mousePointTo = {
    x: (pointer.x - stage.x()) / oldScale,
    y: (pointer.y - stage.y()) / oldScale,
  };

  // how to scale? Zoom in? Or zoom out?
  let direction = e.evt.deltaY > 0 ? 1 : -1;

  // when we zoom on trackpad, e.evt.ctrlKey is true
  // in that case lets revert direction
  if (e.evt.ctrlKey) {
    direction = -direction;
  }

  var newScale = direction > 0 ? oldScale * scaleBy : oldScale / scaleBy;

  stage.scale({ x: newScale, y: newScale });

  var newPos = {
    x: pointer.x - mousePointTo.x * newScale,
    y: pointer.y - mousePointTo.y * newScale,
  };
  stage.position(newPos);
});
```

### 3. 性能优化

优化方面可以参考：
https://www.jianshu.com/p/a72b6d53c3f6
https://juejin.cn/post/7135229172409958431
https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Optimizing_canvas

性能优化过程

1. 测试阶段，发现在有些电脑上出现卡顿，拖拽和放缩过程不流畅，尤其是地图缩到最小，并且点开所有点位的详细信息时
2. 检查性能，通过performance工具，分别在我的电脑上和出现问题的电脑上进行测试，发现在性能不太好的电脑上，某些渲染过程的耗时达到20甚至30ms以上，超过了一帧
3. 分析卡顿原因：可能是绘制元素过多，计算过多
4. 了解优化方式：下面的内容，需要知道哪些是konva已经做了的，哪些是需要我们去做的
  - 已经做的了：批量绘制、bufferCanvas缓冲、优化指令
  - 需要做的：分层、缓存、减少绘制物、整合数据
5. 优化效果：针对我们需要做的，做出优化，查看效果。效果主要体现依据是performance的绘制时间
  - 分层：将背景图和点位分层。但是点位变化不大，效果一般
  - 缓存：点位形状并不复杂，使用缓存优化效果不显著，反而内存占用上升
  - 减少绘制：可视区域外不进行挂载，缩小时删除文本元素，初始比例放的比较大，绘制数量少。效果明显，每一帧基本小于10ms绘制时间
  - 整合数据：将样式一样的图形合并并一次性渲染。效果不明显，因为konva底层已有类似的优化。



在拖拽和放缩基本功能的实现基础上，还需要保证 canvas 的性能稳定。性能问题要有针对性，并且有些性能优化其实在 konva 底层已经做了优化，我们需要知道哪些是 konva 做过的优化，哪些又是我们需要做的优化。

首先先列出针对性的优化方式

- 绘制方面
  - 减少绘制，包括减少绘制次数、可视区域外不绘制
  - 分层，把需要频繁绘制的部分创建成一个单独的层
  - 批量绘制，将绘制操作整合在一起，当短时间内触发重渲染时不进行渲染，而是整合到最后再进行渲染
  - 局部绘制，也称作脏矩形，这个比较复杂，相当于我们只会渲染需要变化的范围内的图形，将其通过一个包围盒包裹，然后和包围盒相交的其他图形也需要重新渲染。
  - 缓存，将绘制结果缓存起来，再次绘制时采用 drawImage 重渲染。这也是一种离屏渲染的方式。当被缓存的元素需要重新渲染时，也可以最大程度降低反复重绘的消耗
  - 减少绘制物。除了拖动时不绘制可视区域外的元素，还可以在放缩到一定程度时，比如文字被缩小到看不见时，删除这些元素。
- 计算方面
  - 优化指令，避免不必要的赋值
  - 降低计算开销，比如计算点位，可以通过 webworker、raf 等方式降低计算过程的开销
  - 整合数据。比如要绘制几百个点位，有些点位的样式一致，那么就可以按照样式将这些点位分组，让这些样式相同的元素执行一次性的绘制。对 konva 来说其实就是把元素分组后，按照分组进行 map，这样每一组的 map 内部元素样式相同，减少了因为修改样式而造成的开销。

这其中大部分的基础优化内容都已经在 konva 实现了，我们能做的只有：

- 分层：把地图底图和点位分别绘制到两个层。当然进行变化的时候也需要同时对两个层操作
- 减少绘制物：可视区域外不渲染，缩小到一定程度不渲染文字
- 降低计算开销：优化算法结构。利用 webworker
- 整合数据：将样式相同的点位整合在一起，统一绘制

性能优化还要考虑的两个重要点，是如何检测性能，以及优化之后的表现如何。

对于 canvas 来说，最好的性能监测方式就是利用 profiler，即 chrome devtools 中提供的 performance 页。他可以监控页面上的任务执行事件；
我们可以在 performance 内查看页面的任务执行耗时。对于 konva 来说，draw 函数的执行表示整个画布的重绘，可以被看做是 canvas 绘制的时间消耗。

通过性能优化手段可以查看优化前后的 draw 函数耗时，是否超过一帧，以及初次渲染的耗时等等。

![](https://pic.imgdb.cn/item/64bed7e61ddac507ccd2645b.jpg)

### 4. 其他业内实现

这里的业内实现其实主要包含两部分

1. canvas 的业内实现，即 canvas 应用在其他方面的使用包含了一些业内的、成熟的实现及优化方案。

实现上，其实大部分的 canvas 库都已经做到了比较完备的实现，除了封装和事件系统外，也包含了一些基础的性能优化能力。
优化上，除了框架自带的之外，还有一些更高级的优化方式，比如离屏渲染、脏矩阵、分块加载等。这些都需要更进一步配合实现

2. 地图的实现。地图本就是一个很庞大的体系，对于前端来说，地图的业内实现主要有以下几种：

- 瓦片地图：一张大的地图或者背景图分成不同的瓦片，把这些瓦片拼接在一起，一个完整的地图就组合出来了。瓦片地图金字塔模型是一种多分辨率层次模型，从瓦片金字塔的底层到顶层，分辨率越来越低，但表示的地理范围不变。
  比如我们放大地图时，相同区域的瓦片会越来越多，分辨率也会越来越高，相当于瓦片分的越来越细。
  前端实现瓦片的库是 Leaflet.js。用 Leaflet.js 实现的瓦片地图其实本质上是图片的加载，当我们放缩地图时，他可以控制请求可视区域内的瓦片，然后重新渲染显示。

如果考虑到 canvas 实现，其实就相当于我们要在 canvas 内实现一个类似瓦片加载的效果。比如对于地图的底图，我们可以通过瓦片库的生成结果（包含瓦片图片和坐标信息），对照实际的 canvas 坐标系通过 drawImage 渲染不同的地图。当放缩移动事件发生时，在重新计算瓦片并请求。

参考：https://segmentfault.com/a/1190000040678481
https://www.cnblogs.com/lishanyang/p/14290162.html

- 可视区域渲染，这个就是上面说的，当放缩小于一定比例后隐藏。

## 其他注意点

1. 移动端？

如果项目是在移动端的，可能需要什么样的操作使得地图可以在移动端正常使用？

- 响应式：由于移动端屏幕大小不同，因此 canvas 大小可能需要改变
- 性能：移动端性能可能不如 pc 端，因此需要考虑更多的性能问题，比如 web worker 等，以及 canvas 的一些优化方式
- 操作：移动端操作事件和 pc 端不同（比如 touch 事件和 mouse 事件），需要做出适配
- 地图加载：可以采用压缩、地图分块加载等方式加快地图加载速度，这个也适用于 pc 端

2. 定位到机器人的问题

还需要一个功能：把地图中心点定位到机器人的位置。核心其实还是通过控制 translate

3. 部署？

项目是怎么部署的？（这个可能有点来不及了）

4. 长连接的方式

除了轮询和 websocket，还可以怎么样保持地图的实时性？

- SSE：SSE 是一种基于 HTTP 协议的单向通信机制，它允许服务器向客户端推送事件流。客户端通过建立一个持久化的 HTTP 连接，从服务器接收事件数据。
  SSE 通常适用于服务端推送少量数据给客户端。
  SSE 的使用类似 websocket，但他发送数据的单位是事件流，即服务器端发送的响应内容应该使用值为 text/event-stream 的 MIME 类型。每个通知以文本块形式发送，并以一对换行符结尾。

```js
const evtSource = new EventSource("//api.example.com/ssedemo.php", {
  withCredentials: true,
});
evtSource.onmessage = function (event) {
  const newElement = document.createElement("li");
  const eventList = document.getElementById("list");

  newElement.innerHTML = "message: " + event.data;
  eventList.appendChild(newElement);
};
```

- 长轮询，不过不适用于本项目。

5. 项目中替换技术栈有什么工程化的方式？

- 渐进式替换，新的组件用新的技术栈，旧的组件用以前的
- 做好自测和 QA 的测试

6. 性能问题，主要是初次渲染、更新重渲染和拖拽、放大事件的性能问题考虑，尤其是最后一个。

实际上 konva 已经在内部做了很多性能优化

# 秒杀平台项目

遇到问题：

- 请求数量大，请求需要添加 jwt，不能每次请求都配置一次；并且请求部分代码多，影响代码质量
- 首屏加载速度很慢，第一次加载可能等数秒才显示内容
- 针对模拟线上环境的优化，模拟秒杀系统应用场景

项目重点：

- 需要符合秒杀系统的应用场景，在前端做出优化。具体优化方向为减轻服务器负担，比如降低资源大小、减少请求次数等。
- 关于技术选型相关的，比如为什么采用原生 webpack 而不是 cra，cra 相比手动配优势在哪里，cra 做了哪些配置？
- 项目中 jwt 的使用，可能有刷新 token 的机制？
- 关于秒杀倒计时，怎么确定倒计时准确？

## 遇到问题

### 请求数量大：请求封装

参考超管项目的封装

```tsx
// 先创建实例
const service = axios.create({
  // 添加jwt头
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${api_cfg.token}`,
  },
  timeout: 10000,
  baseURL: api_cfg.apiUrl,
});

// 添加响应拦截器
service.interceptors.response.use(function (response) {
  // 进行状态码判断
  const { code, data = {} } = response?.data;
  if (code === 10001) {
    Message.error("你的token已失效，请重新登录!");
    setTimeout(() => {
      if (window.location.pathname !== "/login") {
        window.location.href = "/login";
      }
    }, 1000);
  }

  return response;
});

// 封装请求方法request
// 接收泛型作为返回值类型
export default function request<T = unknown>(
  opts: AxiosRequestConfig, // 请求对象，即url、method、data、params等属性
  notify = true
): Promise<T> {
  return new Promise((resolve, reject) => {
    const reqFn = async () => {
      try {
        const res = await service(opts);
        // 这里是service正常resolve的处理，即收到了响应，但可能有误
        const { code, data = {} } = res?.data;
        // 这里只处理了部分类型的结果
        if (res.status === 200 && code === 0) {
          return resolve(data);
        } else if (code === 71002) {
          // 约定：如，服务端错误
          return reject(res?.data);
          // 还可以加几个，比如状态码404，403的处理，或者约定的code的处理
        } else {
          // 其他情况统一reject
          // 通过message组件全局提醒报错
          return reject(res?.data?.msg);
        }
      } catch (err) {
        // service执行错误，即根本没收到响应，比如跨域、网络中断、400/500等错误
        // 通过message组件全局提醒报错
        console.error(err);
        return reject(err);
      }
    };
    reqFn();
  });
}
```

然后具体到每个请求，具体封装：

```ts
// 定义类型：请求体类型，响应data类型
export const getBuildingInfo = (building_id: string) =>
  request<DtoGetBuildingInfoResp>({
    url: "/admin/api/v1/admin/building",
    method: "get",
    params: { building_id },
  });
```

有一个缺点是 request 函数还是会 reject，也就是具体函数还是要具体处理错误。
其实这样更好，因为不同上下文调用的函数可能错误处理不同，如果请求出错不是预设错误的话，就应该直接 reject。

### 优化

#### 优化内容总

1. 代码分割。代码分割的目的有两个：

- 缓存
- 优化代码体积，加快加载速度。

代码分割的方式：

- 动态加载，使用 React.lazy 包裹页面组件。
- splitChunk，分割 vendor
- 分割 css

2. 开发模式优化

- 持久化缓存
- sourcemap 配置 eval-cheap-module-source-map
- 关闭生产模式优化

3. 生产模式优化

- 代码压缩，css、js 压缩
- tree-shaking
- 利用 import 对图片等资源进行 prefetch

4. 其他优化

- 图片：
  - webp 格式：webp-webpack-plugin
  - 压缩：image-minimizer-webpack-plugin
- loader：自己编写的 loader
- 按需导入 antd 的组件和 css
  - 组件：默认支持 tree-shaking
  - 样式：使用 babel-plugin-import 的 style 配置来引入样式。参考https://blog.csdn.net/zwkkkk1/article/details/88823366和https://juejin.cn/post/7117293855958892574
- 骨架屏

#### 针对线上环境的优化

核心：请求优化

- 减小请求大小
- 降低请求次数

1. 减小请求大小，即减小打包产物大小，上述的所有压缩等减小体积的方法

2. 降低请求次数：

- 充分利用缓存
- 分包不要太细碎
- 部分数据持久化，比如秒杀到期时间戳不用频繁同步
- 秒杀接口的节流处理，多次点击只会生效一次

#### 其他方面优化

这些优化方式并不一定是显式的优化，更多的是一种提升项目健壮性的方式

1. 针对可能的服务端问题：做好对服务端处理速度缓慢或对服务端崩溃的处理，比如

- 购买链接设置足够长的超时时间，或者通过倒计时，当时间到的时候加长超时时间。
- 采用合适的 loading 方式，使得用户请求之后不能再请求，相当于一种节流的方式
- 告知用户加载可能很慢等
- 如果服务器崩溃导致不能正常显示，要采取 fallback 措施，比如服务端返回特定的错误码时（500/504）提示好用户，并暂时关闭购买按钮，设置定时器在一段时间后再开始

# 特价团购项目

难点：

- 轮播组件的开发。由于项目的未来扩展性（可能跨端），因此不能使用现成的 rn 库或 web 端库，只能依赖自行开发轮播组件。参考现有轮播组件的逻辑进行开发和设计

轮播组件参考为：https://github.com/meliorence/react-native-snap-carousel

- 轮播组件的优化。如何减小渲染次数，降低更新和初次渲染时的成本。
- 轮播组件嵌入视频，主要有
  - 视频的单例。只在页面上放置一个视频组件
  - 视频组件放置在轮播图中心，通过 z-index 控制显隐。轮播图滚动时，控制视频组件更换源，加载完成后显示。主要是放置重复销毁创建的消耗
- 页面性能优化
  - 会场页首页由于引入了视频和轮播图，导致加载速度变慢，并且出现视频闪烁的情况
  - 其他页面 RCF 指标偏低，尤其是 C 指标时间很长，会场页甚至达到 3000ms 以上。
  - rn 的优化方式和 web 端不同，应该采取更好的方式进行优化

技术栈：rn、mrn

## 项目难点

### 轮播组件

轮播组件的实现参考：https://github.com/meliorence/react-native-snap-carousel

在 rn 上轮播组件的设计可以通过 FlatList 来实现。FlatList 本身具有虚拟列表的优化，即使创建多个元素，也能有较好的性能。

轮播组件有几个关键的设计点

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

### 轮播+视频

- 视频的单例。只在页面上放置一个视频组件
- 视频组件放置在轮播图中心，通过 z-index 控制显隐。轮播图滚动时，控制视频组件更换源，加载完成后显示。主要是放置重复销毁创建的消耗
- 当页面离开会场页时，销毁视频组件。通过会场页面的回调检测到用户离开

这里其实要说的话，应该要结合轮播图来说。比如，轮播图增加了哪些功能能更好支持视频，否则自行实现轮播而不是采用组件就有些没有意义了。

目前想到的点有

- 视频的预加载和缓冲，比如播放到这个视频时，对下一个/上一个视频预加载。但是有个问题是页面只能有一个视频组件，那么这个功能就不太能实现
- 一个需求，比如拖拽事件发生的时候，需要隐藏视频显示商品，那么如果这时候发生了“回弹”，视频进度就难以控制。方法可以是组件提供一个回调，当真正切换时（就是偏移量超过了滚动的阈值，一定会发生切换）执行这个回调，这时就可以销毁视频了。否则只是暂停视频，当滑动事件结束之后再继续播放
- 一个 bug，之前出现过播放时出现两个音轨的情况，主要原因是按索引匹配的时候出现了两个 active 的元素。这个在自行实现的时候可能能通过一些方式避免
-

### RCF 优化

优化主要从这几方面来说

1. 优化原因：性能评分偏低
2. 优化指标

- RCF 指标的含义
- RCF 指标的测量方式和标准
- 优化期间的检测方式
- 优化完成之后的验证方式

3. 优化方式

- 具体优化方式
- 为什么采取这样的优化方式，为什么不采取其他的方式
- 优化接入，关键点，实际操作方法
- 优化效果，如何体现
- 业内通用的其他优化方式如何

4. 专项优化

- 怎么发现的问题（比如闪烁、卡顿等）
- 发现问题工具
- 解决问题
- 验证问题解决
- 工具的适用性，能不能每次开发完后都检查性能

#### 优化原因

1. 线上 RCF 指标偏低。公司内有针对 c 端页面的 RCF 指标监测，分析发现我们的多个页面评分都偏低，优化重点为评分最低的几个页面，主要为会场页和商详页。

根据两个页面的结构分析指标偏低的原因，主要原因为：

- 会场页
  - 引入轮播图和视频组件，包体较大，没有做好分包导致首屏加载时间过长
  - 轮播图、视频组件的渲染时间较长，没有做渲染分级，导致整体页面渲染时间变长
- 商详页
  - 模块较多，但首屏模块只有一个，其他模块都在首屏时渲染，拖慢页面整体渲染速度
  - tab 里不同页面都做了加载，但是根据埋点情况发现其他 tab 页访问不多，没必要在首屏包中加载，可以做按需加载

2. 测试时发现一些问题，比如闪烁、加载缓慢

- 会场页
  - redux 传递列表滚动偏移数据导致轮播组件频繁刷新，出现闪烁。通过 wdyr 确定问题出现的组件和原因
  - 吸顶时显示券列表出现卡顿，点击 tab 吸顶时会感到明显卡顿。通过 RN debugger 和 Profiler 等工具定位到是由于券列表采用 ScrollView，当券数量大时会出现卡顿

#### 优化指标

主要指标为

1. RCF 指标，即线上性能指标，是优化的主要方向。

- R：响应时间，主要是从用户触摸事件开始到页面做出第一帧变动的时间，主要依赖 native 测算。大致原理是“追随”点击事件的流转，直到点击事件被 JS 消费并且点击事件修改的布局在 UI 线程完成更新的时间。
- C：首屏加载时间。统计方式是从页面启动、native 代码加载开始，一直到 js 完成渲染、元素稳定为止。
- F：实时帧率，主要是页面帧率，会考察最大连续丢帧的情况

2. 其他调试指标，即优化期间的一些判定指标，主要通过调试工具体现

- 首屏加载时间测量，比如从 3000ms 优化到 1000ms 左右
- 包体积测量，可以检测到通过网络请求下载的包体积，用于检查包体积优化和分包情况
- 重复渲染，检查渲染次数和函数执行次数，定位重复渲染原因
- 页面帧率，是否出现掉帧情况

3. 调试工具指标，比如 wdyr 的输出、profiler 的合理渲染图等等

#### 优化方式

针对会场页和商详页做不同的优化

##### 会场页

会场页优化：C 指标得分只有 3 分，线上平均加载时间达到 3000ms，通过调试工具发现会场页包体积为整包，达到 3MB 左右

1. 通过 RAMBundles + InlineRequire 进行分包，将庞大模块分离出去减小包体积。

分包的对象主要是大量的组件。这些组件虽然有些不参与首屏渲染，但仍然包含在首屏加载的包体积中。包括

- 大量的 Modal 弹窗。有些弹窗内还有很多其他的功能组件，在首屏都是不需要的，可以在弹窗应该显示时再加载
- 轮播图和视频组件。虽然这两个在首屏，但是仍然占据了很大的包体积。考虑将其延迟加载，当页面的列表元素完成渲染后，再加载这两个元素。在此之前采用同位置的商卡组件做占位

内联导入的代码其实比较简单，简单来说就是

```js
let SomeComponent = null;
const App = () => {
  const [isRender, setRender] = useState(false);
  const didRender = () => {
    SomeComponent = require("/path/to/component");
    setRender(true);
  };
  return (
    <>
      ...
      {isRender && <SomeComponent />}
    </>
  );
};
```

由于 require 本身是同步的，因此可以看做就是在控制元素的条件渲染。
内联引入需要和 RAM 配合，后者通常是需要开启打包时的选项。
RAM 本质上是开启了特殊的打包方式，让 js 以模块为单位单个打包，可以以模块为单位获取。同时配合 metro 等打包工具实现的 Indexed RAM Bundle 形式，可以保证细粒度的同时，文件数量本身并不会增多，从而导致请求和加载负荷增加。

优化效果：分包之前主包体积达到 3MB，会场页主要进行三个分包，主包体积降低到 1.5MB

2. 优化包体积，删除无用库，剔除 lodash 改用内部的库（仅引入单个方法，如 debounce 等），进行 tree-shaking 优化

- 首先是清除一些库。从 package.json 出发，找到哪些库是没必要引入的。比如 lodash 只使用了几个方法，那就应该考虑从其他更小的内部库中导入。这些优化过的库支持按需加载，同时对 tree-shaking 的支持更好
- 通过 webpack 插件的形式，将导入优化为具体的文件路径，避免全量引入。
  比如：

```js
import { Button } from "xxx-desgin";

优化后;

import { Button } from "xxx/desgin/lib/button";
```

- 其他，比如 icon 库不要本地导入等

优化效果：主包和其他分包的体积都减小，总体降低到 900KB 左右

3. 分批渲染。优先渲染轮播图、列表组件，对 nav、浮窗等其他元素降低渲染优先级。同时抽象出优先级渲染控制组件 Priority，便于在其他地方也进行分批渲染控制

Priority 的实现可以参考另一篇博客。
主要应用在，将列表、轮播图组件赋予最高优先级，而降低其他组件优先级。

优化效果：肉眼难以观察。通过 raptor 测量的方式，显示首屏的 App 组件渲染时间从 700ms 下降到 400ms 左右

4. 重复渲染优化。通过 wydr 和 debugger 定位到轮播组件发生大量频繁渲染，原因是 redux 的状态，发现列表移动时更新了 redux 全局状态导致组件无用更新。

定位问题时发现页面闪烁。通过 Debugger 发现页面滚动时轮播组件大量重复渲染。
然后通过 wdyr 对轮播组件进行检查，发现更新原因是 redux 的 useSelector。
滚动事件触发 redux 的全局属性里的某个值更新，引起全部持有该状态的组件更新

解决方法是采用全局事件代替 redux 进行单纯的数据传递和状态共享，通过 Animated 组件代替之前通过修改 state 达到的动画效果。

5. 卡顿优化。由于吸顶时渲染券列表，券列表采用 ScrollView，当券数量多时会感到卡顿。

首先是从吸顶行为的周围排查，检查吸顶操作进行时有哪些组件发生状态变化。
然后通过 Profiler 工具检查到吸顶操作时，有一个 ScrollView 组件渲染耗时很长，定位发现是券列表组件
将其改为 FlatList；兜底方案：只显示少量券，当券数量多时不在首屏渲染，而是单开一个券列表浮层，首屏最多渲染三个。

6. 网络优化。主要是接入了 mrn 提供的请求前置，在启动期间就预请求会场所需数据，但效果不明显。

7. 其他优化。比如视频组件的离屏销毁等

总体优化结果：

- 包体积从 3MB 降低到 1.8MB，首次加载的包体积仅有 900KB
- 首屏加载速度从 3000ms 提升到 1000ms 左右，C 指标从 3 提升到 30
- 闪烁现象解决，卡顿现象解决

##### 商详页

1. 分批渲染。商详页模块较多，从上到下有商品详情、购买须知、猜你喜欢、评价等多个模块。首屏应该只展示详情模块，其他模块可以采用分批渲染的形式降低渲染优先级。

更进一步，猜你喜欢模块可以延迟到商品详情的头图加载完成之后再开始渲染。保证不会影响主要模块的渲染速度。

2. 预加载和预请求。商详页属于二级页，因此预加载可以做很多内容。通过业务埋点，可以得出用户通常从会场跳转到商详页的比例较高，大约有 30%-40%，而专区、搜索页等的跳转比例较低。

因此可以在会场页渲染完成时，预加载商详页。执行由 mrn 提供的方法，预热商详页组件。

3. 头图优化。在跳链中加入头图的 url，商详页可以快速取到对应的头图并进行加载。不同分辨率的区别仅是后缀不同

优化效果：商详页 C 指标提升 20%左右，加载时间从 600ms 降低到 400ms，同时由于预加载的引入，秒开率得到提升。

## 其他注意点

1. 样式适配问题。大部分样式的数值都需要 rpx 转换，写死的样式通常会有问题。即使是相同的样式，在不同机型上也会有不同的布局表现，在 ios 和安卓上也有不同表现。

这个不仅体现在样式的适配上，还有一些功能的适配。解决方法主要就是多做尝试，比如 xcode 切换不同机型，针对部分特殊机型做专门的测试，以及设计好兜底方式
更极端的，有时候样式还要区分 ios 和安卓。在细节样式的调试过程中经常会出现这种情况。

2. MRN 对 RN 做了⼤量的优化⼯作，总结下来主要有三点：加载流程优化、运⾏环境隔离以及⼯程化保障。
