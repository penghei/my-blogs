---
title: tailwindcss学习笔记和常用总结
date: 2021-11-29 22:27:19
tags: 日常学习
cover: /img/tailwindcss.png
categories: CSS
---

# 配置

## vite + vue3

> ts 的配置有一点问题(ts-node)的配置，所以暂时采用 js

1. 创建 vite 项目

```
yarn create vite

yarn install
```

2. 安装 tailwindcss

```
yarn add -D tailwindcss@latest postcss@latest autoprefixer@latest

npx tailwindcss init -p
```

3. 在 tailwind.config.js 文件中添加：

```js
module.exports = {
  purge: ["./index.html", "./src/**/*.{vue,js,ts,jsx,tsx}"], //这一行
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
};
```

4. 在 src 创建 index.css

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

5. 把 index.css 引入 main.js

这里其实不用 index.css 也可以，只需要在 main 里直接`import "tailwindcss/tailwind.css"`也可以

## create-react-app

1. 同理，安装 react 项目

```
create-react-app test
```

2. 安装 tailwindcss，注意需要安装的是 Craco，后续用 craco 来代替编译

```
yarn add -D tailwindcss@npm:@tailwindcss/postcss7-compat postcss@^7 autoprefixer@^9
yarn add @craco/craco
```

3. 更新 package.json 文件中的 scripts，将 eject 以外的所有脚本都用 craco 代替 react-scripts。

```json
"scripts": {
    "start": "craco start",
    "build": "craco build",
    "test": "craco test",
    "eject": "react-scripts eject"
},
```

4. 创建 tailwind.config.js，**注意这里不是手动创建**

```
npx tailwindcss-cli@latest init
```

5. 同上，更改配置

```js
module.exports = {
  purge: [],
  purge: ["./src/**/*.{js,jsx,ts,tsx}", "./public/index.html"], //这一行
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
};
```

6. 在本身有的 index.css 加上这几句，然后确认引入到了 main.js 里边

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

# 使用

## 默认配置(很重要！！！)

1. 所有的 h1/h2/h3 等都是初始字体，可以正常设置字体属性
2. 去掉了所有边框，所以按钮没有边框；需要开启可以在 class`直接class="border ..."`开启
3. 没有默认 margin，包括 html、body、p、hn 等
4. ul li 没有样式
5. 图片是块级元素

## 常用属性

### 颜色

默认只提供了 8 种基础色，每种从 50 到 900 有不同强度

- **基础色**：需要指明强度，不然不能用  
  `gray red yellow green blue indigo purple pink `
- **其他**：不能指明强度  
  `white black`

### 宽度、高度

**写法**：`h-xxx` 或` w-xxx`，xxx 是大小，可以是数字、分数或者 full 等

- 数字的单位是 rem，4 代表一个 rem，**1rem 等于 html 元素的字体大小 16px，也就是说每个单位都相当于 4px**
- 数字在 1-12 是连续的，上面就不是了；从 16 开始每 4 个才有（比如只有 h-20 h-24 没有 h-22），一直到 h-96；所以没有 h-50 h-25 这种，别写错了
- 分数：分母是 2、3、4、5、6、12 可用
- 其他：full 代表 100%，screen 代表 100vh(100vw)，max 和 min 代表 max-content 等

### 最小、最大宽高

**写法**：基本就是在宽高前面加个 min 或者 max，但是单位不一样

- 最小的单位只有 0 和 full；**高有 min-h-screen 代表 100vh（唯一特别的）**
- 最大的宽度没有具体数字，用 xs、md、sm 等划分代替
- 最大高度和高度写法一样（数字）

### display

**写法**：就是把正常 css 里边的 display 后面的单独写出来，比如`display:flex`就是直接写`flex`

- `display:none` 可以用 `hidden`

### 位置

| class    | properties          |
| -------- | ------------------- |
| static   | position: static;   |
| fixed    | position: fixed;    |
| absolute | position: absolute; |
| relative | position: relative; |
| sticky   | position: sticky;   |

sticky 是可以用的（好耶

### 移动

**写法**：`top-xxx`，xxx 是数字；其他方向一样

- xxx 的写法和宽高一样，同样可以使用分数等，full 表示 100%
- 有个 inset 属性，inset 同时设置四个元素，inset-y 同时上下，inset-x 同时左右

### z-index

**写法**：直接写 `z-xxx`，xxx 代表层级数字

### flex

**写法**：直接写 flex 即可
其他常用写法：
|class|properties|
|---|---|
|flex-direction:colmun|flex-col|
|flex-direction:row|flex-row|
|justify-content:xxx|justify-xxx|
|align-items:xxx|items-xxx|

### grid

**写法**：直接写 grid 即可
其他常用写法：
|class|properties|
|---|---|
|grid-template-columns: repeat(xxx,);|grid-cols-xxx(注意带 s)|
|grid-template-row: repeat(xxx,);|grid-rows-xxx(注意带 s)|
|grid-column-start:xxx|col-start-xxx|
|end 和 row 同理，就不写了|end 和 row 同理，就不写了|
|gap: xxx|gap-xxx|
|column-gap: xxxpx|gap-x-xxx|
|row 同理|row 同理|

### margin 和 padding

**写法**：`m-xxx` 或者 `p-xxx`，

- mt-xxx 代表`margin-top:xxx`，其他的同理
- mx/my/px/py 代表同时设置上下/左右

### 字体

#### 大小

**写法**：`text-xxx`,xxx 是划分符不是数字（sm 等）

- 可以从 xl 一直设置到 9xl，9xl 是最大的

#### weight

| Class           | Properties        |
| --------------- | ----------------- |
| font-extralight | font-weight: 200; |
| font-light      | font-weight: 300; |
| font-thin       | font-weight: 100; |
| font-normal     | font-weight: 400; |
| font-medium     | font-weight: 500; |
| font-semibold   | font-weight: 600; |
| font-bold       | font-weight: 700; |
| font-extrabold  | font-weight: 800; |
| font-black      | font-weight: 900; |

#### text-align

| Class        | Properties           |
| ------------ | -------------------- |
| text-left    | text-align: left;    |
| text-center  | text-align: center;  |
| text-right   | text-align: right;   |
| text-justify | text-align: justify; |

#### 文字颜色

**写法**：`text-color-xxx`，color 是颜色，xxx 是强度，同其他的颜色

#### 其他

| Class        | Properties                     |
| ------------ | ------------------------------ |
| underline    | text-decoration: underline;    |
| line-through | text-decoration: line-through; |
| no-underline | text-decoration: none;         |

### 背景

#### 背景颜色

**写法**：`bg-color-xxx`，color 是颜色，xxx 是强度，同其他的颜色

- 可以使用 `bg-opacity-{amount}` 控制元素背景色的不透明度，amount 只有 100 75 50 25 0，代表百分比

#### 背景图

| 常用写法    | 属性                                  |
| ----------- | ------------------------------------- |
| bg-bottom   | 背景图位置，bottom 可以是其他位置选项 |
| bg-repeat-x | 背景图重复，不重复是`bg-no-repeat`    |

背景图好像不能直接在 html 中设置，有必要还是写在 css 里边

### 边框

#### 圆角

**写法**：`rounded-xxx`，xxx 是划分符不是数字（sm 等）

- 直接设置圆角是四个角，可以使用-b -t -r -l 制定位置，或者-bl -tr 等（`rounded-b-xxx`）
- full 表示无限大的值，表现出来就是：如果宽高一样就是圆形，如果不一样就是胶囊

#### border-width

**写法**：`border-xxx`，xxx 是数字

- 直接设置是四个边，可以使用-b -t -r -l 制定位置，或者-bl -tr 等（`border-b-xxx`）
- 如果没有数字默认是 1px

#### border-color

**写法**：`border-color-xxx`，xxx 是强度，和其他颜色设置基本一样

- 同背景颜色，可以通过`border-opacity-{amount}`设置透明度

#### 边框样式

**写法**：`border-xxx`，xxx 是样式，比如`border-solid`

### 阴影

**写法**：`shadow-xxx`，xxx 是划分符不是数字（sm 等）

- shadow 默认外阴影，内阴影要用`shadow-inner`
- shadow 默认是右下的

### 伪元素

**写法**：`hover/focus/active/visited:xxx`，xxx 是各种效果

- xxx 的效果可以是颜色、大小以及几乎各种改变
- 如果需要多个效果，就分开写，比如`hover:bg-green-500 hover:text-lg`

### 响应式

主要是以划分为基础的伪元素，相当于屏幕在划分所对应的大小时会有改变
主要划分为：

| 断点前缀 | 最小宽度 | CSS                                |
| -------- | -------- | ---------------------------------- |
| sm       | 640px    | @media (min-width: 640px) { ... }  |
| md       | 768px    | @media (min-width: 768px) { ... }  |
| lg       | 1024px   | @media (min-width: 1024px) { ... } |
| xl       | 1280px   | @media (min-width: 1280px) { ... } |
| 2xl      | 1536px   | @media (min-width: 1536px) { ... } |

**写法**：`sm/md/lg/xl/2xl:xxx`，xxx 是各种效果

- 按照文档要求，应该把基础值定为移动设备，然后用断点依次往上到大屏幕

### 改变鼠标

可以让鼠标放在元素上面时改变鼠标样式（比如加载、点击等效果）
|Class|Properties|
|---|---|
|cursor-auto| 自动|
|cursor-default| 箭头|
|cursor-pointer|手（点击效果）|
|cursor-wait| 转圈等待|
|cursor-text| 文本|
|cursor-move| 四向箭头的移动|
|cursor-help| 右下角有个问号|
|cursor-not-allowed| 禁止|

### 其他交互

#### 轮廓

> 使用 `outline-none` 来隐藏焦点元素的默认浏览器轮廓。
> 强烈建议在使用这个功能时，应用您自己的焦点样式来实现可访问性。

输入框最好用到这个，这样就可以使用自己设置的边框

#### 缩放

**写法**：`scale-xxx`，当 xxx 小于 100 表示缩小，大于 100 表示放大

- xxx 不是随意的数字，一般常用的有 0 50 75 90 95 100 105 110 125 150
- **使用之前必须要写 transform，不然不生效**
