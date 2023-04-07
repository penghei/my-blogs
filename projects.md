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

关于plugin的

#### husky

配置 husky 在 eslint 校验通过之前不允许提交。
[husky 文档](https://typicode.github.io/husky/#/)
[husky 配置](https://juejin.cn/post/7038143752036155428#heading-9)

husky 是一个操作 git 钩子的工具。可以使用预设的钩子，在 git 各种操作的时间进行补捕获和拦截。
比如，在 pre-commit 阶段，即提交之前先进行 lint 的检查，如果不通过就不允许提交。

> Git hooks 是一种在 Git 仓库中预定义的脚本，它可以在特定的 Git 操作时自动执行。Git hooks 可以在 Git 操作之前或之后执行一些自定义代码，比如在代码提交之前运行测试，或者在代码合并之前运行代码格式化工具等。
> 在 Git 中，每个仓库都有一个 .git/hooks 目录，其中包含了一些可用的 Git hooks 脚本模板。这些模板可以被复制并重命名为不同的钩子名称，然后在其中编写脚本代码来实现自定义操作。Git hooks 脚本必须是可执行的（即要有执行权限）。
> husky的核心代码其实很简单，就是通过配置git hooks的方式来实现要在git hooks阶段执行的任务。

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

1. 事件冒泡规则。

问题：
在项目中确实遇到了这样的问题：有一个这样的需求，即显示一个实时地图，实时地图内有很多点位，希望点击这些点位时，能获取到具体点的是哪个点位。
这是一个常见的事件委托的需求，但是需要事件冒泡作为基础。由于konva的事件系统是自己模拟实现的，因此与dom的事件机制不一样，在使用时出现了一些问题。

个人尝试：
- 尝试把onclick事件放到实时地图的矩形上，发现虽然地图表现为“包裹”着点位，但是实际上不能触发冒泡
- 然后猜测可能是跟先后顺序有关，尝试后也无效。
- 最后猜测应该把实时点位作为地图的矩形的子元素，或者用一个大的元素包裹这两个元素，都不行
- 如果把事件绑定在layer或stage上，这两个上面已经有其他事件的绑定了，并且实时地图和实时点位是选择渲染的，不一定有，因此希望把事件绑定在实时地图容器上，而不是绑定在顶层。

查阅资料：
- 官方文档，有讲事件的部分，但是没有详述冒泡的规则，并且react-konva文档中也没有说事件流的相关概念
- github issues：查找有关事件冒泡的issues，没有，然后查找其他关于事件系统的issues，也没有
- 最后，查看源码
  - 先是了解konva事件系统的实现原理
    - 调试源码
    - 搜一些资料，查看事件原理
  - 然后搞清楚事件冒泡的机制：fireAndBubble，以及查找的“parent”到底是谁
  - 最后了解到每个元素都有一个parent，但是parent只在Container中被设置，并且必须调用add方法。Container只有stage、layer和group，普通元素不行。

解决方案
- 在Container上放置事件监听。使用Group把实时地图和实时点位包裹起来，然后在Group上监听事件，即可。
- 同时还了解到了取消事件冒泡的机制，在Group上通过cancelBubble防止事件进一步冒泡

2. 地图拖动、点位的可视区域渲染，边界控制。（可以不说）

问题
地图的底图比实际的canvas大小大很多，如果想显示完整的地图就有两种方式
- 创建一个和底图一样大的canvas，然后通过css的滚动条进行移动
- canvas 大小固定，通过拖动的方式移动地图

第一种的问题：过大的canvas会导致性能出现问题，比如分辨率不足、渲染负担重等问题。
解决方式：canvas大小固定，改变渲染区间。

具体做法：
- 首先创建固定大小的canvas，然后把整个底图渲染上去。canvas的大小不变，超出的区域是没有渲染的区域
- 实现拖动查看。具体功能点有
  - 基本的拖动，通过监听mousemove等事件，修改canvas的translate，然后重新渲染，实现拖动效果
  - 边界控制，不能让拖动超出底图的范围，因此拖动时计算边界，如果拖动时offset超出边界范围就不发生移动。
  - 可视区域渲染：渲染所有点位时，先通过他们的坐标判断是否在可视区域，如果在才渲染，否则不进行渲染。

拖动的实现

基本思路：
1. 记录一个offsetX和offsetY，表示canvas的translate值。每次绘制，都先`translate(offsetX,offsetY)`，然后执行绘制
2. 监听canvas的mousedown、mousemove和mouseup事件，当mousemove执行时，获取实时的鼠标坐标，和mousedown时获取的鼠标坐标的差值就是偏移量，即offsetX和offsetY。
3. 通过修改之后的offsetX和offsetX，如第一步操作。

另外：react-konva中的实现：

- 其他方式基本不变，但是不需要自己手动绘制，只需要修改layer的x和y即可。

```js
const canvasRef = useRef<HTMLCanvasElement>(null);
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
  return (
    shapeX >= left && shapeX <= right && shapeY >= top && shapeY <= bottom
  );
};

const draw = (type: TranslateType) => {
  // 绘制前先translate
  ctx.clearRect(0,0,width,height)
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
  - 组件：默认支持tree-shaking
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

### 其他

- 倒计时的实现：setTimeout 不一定准确。

> setTimeout、setInterval 属于定时触发器线程属于 macrotask，它的回调会受到 GUI 渲染、事件触发、http 请求、等的影响。所以这两个不适合做精准的定时。

怎么实现一个精准的倒计时？setInterval 不行，需要用 setTimeout 实现，并且不断修正。
