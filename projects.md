# 超管项目

遇到问题：
- 代码问题
  - 写好注释
  - 充分实现和利用代码复用性，使用工具库，自己编写工具函数、自定义 hooks
  - 分割组件，提高组件复用性
  - 代码规范，变量、函数名，eslint
- 需求问题
  - 部分功能实现困难，比如 canvas 地图的点位和比例计算：理清思路，多做推理和演算
  - canvas 绘制复杂，很难处理事件：采用 konva
  - 组件多，数据杂，context 不合适：先使用 redux，但类型不好且冗余代码过多：recoil
  - 数据量大，数据流杂乱，采用画数据流图的方式理清思路？

项目重点：
- 技术选型，即
  - konva和其他canvas框架的技术选型，以及canvas本身的api、性能优化相关
  - recoil和其他状态管理库的技术选型（可以说的：redux、mobx、zustand、joita等），从哪些方面考虑，这些状态管理库的通用特点有哪些。（可以参考那篇飞书文档）
- 框架相关问题，即
  - nextjs的ssr导致的使用问题，比如某些浏览器api不适用的情况
- 项目规范
  - 代码可读性、规范性、复用性各方面

## 遇到问题

### 代码规范

规范主要有四类规范：

- （文件）命名规范
- ts规范
- React规范
- git规范

规范来源：
- 部门规范
- 公司内规范
- 个人建议/自用规范

#### 命名规范

1. 文件名、目录：

- kebab-case形式。语义要明确，比如modal组件的文件名称都叫modal-xxx
- 不要简写
- 目录要对于复数结构采取复数形式。

#### ts规范

1. 类的私有方法前要加private和下划线
2. 接口PascalCase，前面要加I
3. 枚举PascalCase
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

格式遵循TSDoc

6. 尽量不使用any，可以用unknown代替
7. 字符串拼接统一使用模版字符串
8. 使用 interface 定义对象结构体，而不是使用 type，对于interface接口名前要有I
9. 使用 async/await 代替 Promise，并且避免 async/await 与 Promise 混用
10. 使用统一的 import 顺序，按 内置库 → 外部库 → 内部模块 的顺序进行 import。


#### react项目规范

1. useCallback、useMemo、React.memo 一起结合使用才能减少不必要的子组件渲染，分工如下：
- 父组件：在父组件中应该使用 useCallback() 和 useMemo() 尽量确保相应的 fn 或 value 不变，这样才能确保传递给子组件的 prop 不变。
- 子组件：在子组件中应该使用 React.memo() 包裹原始的子函数组件，从而可以通过利用浅比较 props 的方式尽量使得子组件在 prop没有发生变化时不进行渲染。

2. 不要在 JSX 中写 JavaScript/TypeScript
在 JSX 中写代码，会让调试变得困难。(这个可以详细说一下)

```js
// Bad

return (
  <>
    {users.map((user) => (
      <a onClick={event => {
        console.log(event.target); //bad
        }} key={user.id}>{user.name}
      </a>
    ))}
  </>
);

// Good

const onClickHandler = (event) => {
  console.log(event.target);
}

const userLinks = users.map((user) => (
  <a onClick={onClickHandler} key={user.id}> {user.name} </a>
));

return (
  <>
    {userLinks}
  </>
);
```

3. husky：确保在提交代码时也能够运行 lint 检查

#### git规范

按照git工作流的形式，主要是四个分支

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
  - 提测前将 feature 分支 merge 到 release  分支，此处的 MR 需要严格 code review
- Hotfix
  - 从 master checkout 创建，然后需要修改 package.json 中的 version，创建 hotfix 分支时应该更改 patch version，比如从 0.6.0 改为 0.6.1
  - 修改完成后创建 merge request 到 master
  - 注意在将 hotfix 分支合并到 master 分之后，记得更新 release 分支以及 feature 分支


#### eslint

使用`eslint --init`即可初始化配置。

配置内容
- eslintrecommended、reactrecommended、tsrecommended
- @byted/eslint-config：公司内部的eslint推荐配置
- 自定义的一些
  - 禁用魔法数字
  - useState建议是`[xxxState,setXxxState]`的形式，以及useRef必须是xxxRef的形式，useReducer必须返回xxxReducer的形式。这三个都是自定义的插件，参考工程化中编写插件的部分。

#### husky

配置husky在eslint校验通过之前不允许提交。
[husky文档](https://typicode.github.io/husky/#/)
[husky配置](https://juejin.cn/post/7038143752036155428#heading-9)

husky是一个操作 git 钩子的工具。可以使用预设的钩子，在git各种操作的时间进行补捕获和拦截。
比如，在pre-commit阶段，即提交之前先进行lint的检查，如果不通过就不允许提交。

配置流程：
1. 安装lint-staged、husky、commitlint
2. 配置husky在pre-commit阶段进行line-stage检查：

```
yarn husky add .husky/pre-commit "npx lint-staged"
```

当然还要配置.lintstagedrc.json 文件控制检查和操作方式

```json
{
  "*.{js,jsx,ts,tsx}": ["eslint  --fix"],
}
```

3. commitlint配置commit规范
[commitlint文档](https://commitlint.js.org/#/?id=getting-started)

使用husky在commit-msg钩子上增加对commit的检查：
```
husky add .husky/commit-msg 'npx --no-install commitlint --edit "$1"'
```
创建commitlint.config.js，使用预设：
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

- 技术选型：状态管理库的了解，redux、mobx的原理、recoil的优势，其他状态管理库之间的优劣比较
- konva的使用，konva的事件原理和设计模式，react-konva和react的结合方式
- canvas api，canvas基础功能的实现，canvas优化
- 代码规范，四个方面的代码规范
- 工程化代码规范的方式（eslint、husky）
- （其他）nextjs的使用问题，代码复用性，数据量大的处理

# 秒杀平台项目

遇到问题：
- 请求数量大，请求需要添加jwt，不能每次请求都配置一次；并且请求部分代码多，影响代码质量
- 首屏加载速度很慢，第一次加载可能等数秒才显示内容
- 针对模拟线上环境的优化，模拟秒杀系统应用场景

项目重点：
- 需要符合秒杀系统的应用场景，在前端做出优化。具体优化方向为减轻服务器负担，比如降低资源大小、减少请求次数等。
- 关于技术选型相关的，比如为什么采用原生webpack而不是cra，cra相比手动配优势在哪里，cra做了哪些配置？
- 项目中jwt的使用，可能有刷新token的机制？
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

- 动态加载，使用React.lazy包裹页面组件。
- splitChunk，分割vendor
- 分割css

2. 开发模式优化

- 持久化缓存
- sourcemap配置 eval-cheap-module-source-map
- 关闭生产模式优化

3. 生产模式优化

- 代码压缩，css、js压缩
- tree-shaking
- 利用import对图片等资源进行prefetch

4. 其他优化

- 图片：
  - webp格式：webp-webpack-plugin
  - 压缩：image-minimizer-webpack-plugin
- loader：自己编写的loader
- 按需导入antd的组件和css
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

- 倒计时的实现：setTimeout不一定准确。

> setTimeout、setInterval 属于定时触发器线程属于 macrotask，它的回调会受到GUI渲染、事件触发、http请求、等的影响。所以这两个不适合做精准的定时。

怎么实现一个精准的倒计时？setInterval不行，需要用setTimeout实现，并且不断修正。
