---
title: vue3+ts配置及简单使用注意
date: 2021-11-25 12:08:29
tags: 日常学习
categories: Vue
cover: /img/vue3.png
---

- 先记录一下 vue 使用的坑

1. **key 尽量不要用 index,不然删除的时候会找不到元素导致奇怪的报错!key 一定要用特殊的 id**,因为找不到 key 导致 todolist 的元素删除不按照选定,折腾了一个多小时才找到问题,一定要记住!

## 安装

直接创建项目,然后自定义配置即可

> 这里用的是 vue-cli 创建，如果是 vite 的话就更方便，直接`yarn create vite`再按照指示操作就可以了

推荐直接把 router 和 vuex 也一起安装了
![QQ截图20211125135242.png](https://i.loli.net/2021/11/25/TplDo47yzaGRIY3.png)
然后是这个,推荐 N

```powershell
? Use class-style component syntax? (y/N) N
```

这个是编译 jsx 的,不用

```
? Use Babel alongside TypeScript (required for modern mode, auto-detected polyfills, transpiling JSX)? No
```

然后选代码检查,一个 eslint 就够了,加上 prettier 报错太多了

```
? Pick a linter / formatter config: (Use arrow keys)
```

这个是问这些文件放在哪,选第一个分开放

```
? Where do you prefer placing config for Babel, ESLint, etc.?
```

最后一个是是否保存,建议不保存
完成的话基本的配置就可以了,但是这里自动安装的 vuex 有点小问题,然后示例组件也不是用 script setup 写的,需要改一下.下面会说到

## 组件内基本配置

按照以下内容形成一个基本组件

```ts
<template>

</template>

<script setup lang='ts'>
import { reactive, ref } from 'vue'
const props = defineProps({
  myProps: String
})
</script>

<style scoped>

</style>
```

解释:

- 首先 template 不再需要父元素,直接写就行
- `<script setup lang='ts'>`是新语法糖,主要是有下面几点:
  - 不用 setup 函数了
  - 可以让下面声明的变量直接用在模板中,**不需要返回**: 包括 import 引入的,vue,vuex 和 router 自带的,所有定义的函数和响应式变量等等
  - lang="ts"是使用 ts 必须的
  - 其他比如生命周期,watch,computed 等正常使用就行
- props 和 emits 必须要 defineProps,不能直接写;包装之后可以直接使用 props 对象
  - 对于 ts,还可以直接通过类型声明(Iprops 接口)而不用具体对象

限制:

- props 和 emits 引入需要 defineProps defineEmits,不能直接引入
- 获取组件实例的 ref 需要 defineExpose 暴露,否则获取不到

```ts
interface IProps{
    name:string;
    age:number
}(
const props = defineProps<IProps>()
```

- 另外,组件文件名命名要规范(大驼峰),可以不写 name 属性

## 具体使用(重点讲和 vue2 还有 js 区别的地方)

### 指令

#### v-if v-show

使用基本不变,变量不需要响应式,直接数据就可

#### v-model

变量需要响应式

```ts
<template>
    <input v-model="inputValue"/>
</template>
<script setup lang='ts'>
const inputValue = ref('')
</script>
```

如果有多个绑定元素,可以使用名称相同的 v-model 值,然后使用`reactive([])`获取,最后的值会在`[]`中按顺序放
在 v-model 后面可以直接加修饰:

- `v-model.trim`,执行 trim()
- `v-model.number`,转为数字

#### v-for

##### 基本使用

不会在没有 key 时报错,但是需要 key:并且会提醒使用数据自带的 id,最好不要使用 index(参考上面的错误)
v-for 一般放在需要渲染的元素上,**注意不是父元素上**

##### 数组渲染

- 渲染的数组只会在以下方法更新渲染:
  `push()`
  `pop()`
  `shift()`
  `unshift()`
  `splice()`
  `sort()`
  `reverse()`
  直接通过序号更改元素
- 常用的一些不能触发更改的:
  `sort`
  `filter`
  替换数组,比如把新数组直接赋值给原数组(`arr1 = arr2`或`arr1 = [...arr2]`)
  赋值改变数组解决方法:

```ts
const newArr = todoList;
//do sth
todoList.length = 0;
todoList.push(...newArr);
```

### 事件处理

#### event

vue 中参数 e 一般简单直接定义为 Event,然后使用进行断言

```ts
<template>
<button @click="getButton">hello</button>
</template>

<script setup lang='ts'>
function getButton(e:Event){
    const textVal = (e.target as HTMLButtonElement).innerText
}
</script>
```

#### ref 指定元素

在 setup 包装下可以直接创建响应式对象使用

```ts
<template>
<input ref="inputRef" />
<button @click="getRef"/>
</template>

<script lang='ts'>
    setup(){
        const inputRef = ref<HTMLInputElement | null>(null)
        function getRef(){
        const val = inputRef.value?.focus()
        return {
            inputRef,
            getRef
        }
    }
}

</script>
```

**但是在 script setup**不可以,不能直接访问,需要通过 defineExpose 暴露给模板

```ts
<template>
<input ref="inputRef" />
<button @click="getRef"/>
</template>

<script lang='ts'>
const inputRef = ref<HTMLInputElement | null>(null)
function getRef(){
    const val = inputRef.value?.focus()
}
defineExpose({
  inputRef,
});
</script>
```

并且父组件通过 ref 获取时,也会直接获取这里暴露的值而不是整个子组件

### 自定义 hooks

基本使用:就相当于在外创建一个文件,导出对应的值或函数等等，区别在于可以使用 vue 相关的 api
在 hooks 中定义的函数操作变量,在模板同时引入后也可以正常使用

```ts
//useTimer.ts
import { ref } from "vue";

export const nowTime = ref("00:00:00");
export function getTime() {
  const date = new Date();
  const sec =
    date.getSeconds() < 10 ? `0${date.getSeconds()}` : date.getSeconds();
  const min =
    date.getMinutes() < 10 ? `0${date.getMinutes()}` : date.getMinutes();
  const hour = date.getHours() < 10 ? `0${date.getHours()}` : date.getHours();
  nowTime.value = `${hour}:${min}:${sec}`;
}
```

然后在组件中导入并使用

```ts
//xxx.vue
<template>
  <h1>{{nowTime}}</h1>
</template>

<script setup lang="ts">
import {getTime,nowTime} from '@/hooks/useTimer';
setInterval(()=>{
    getTime()
},1000)
</script>
```

## 其他

### 使用问题总结

1. `<script setup lang='ts'>`很爽，建议多用
2. `<script setup lang='ts'>`下的变量容易写乱,所以有以下几点可以让代码整齐：

- 把所有的 import 和引入之后的立即调用（比如`const store = useStore()`）放在最上面,import 的顺序为:
  1. vue 相关,如 ref
  2. vuex、vue-router 及其他相关库
  3. 其他组件
  4. 外部数据（接口等）
  5. 其他
  6. 引入之后的立即调用
- 全局变量写在 import 下面，功能上面，注释隔开
- defineProps 等写在全局下面，同样注释隔开
- 一个功能放在一个区域，不同功能之间最好写注释展一行区分，最少也得有空格
- 功能相关的变量、函数都声明在一起（包括响应式数据），不需要全部写在头部，除非是很多功能都依赖的
- 生命周期写在最后（功能的最后，不是整个页面最后，除非是整体的生命周期），顺序按照一般产生的顺序即可，watch 可以视情况放在最前或最后

3. 响应式数据不要随意赋值，容易丢失响应性。常见响应式：

- reactive 和 ref 的数据，不要把一般值直接赋给他们
- computed 包裹返回的数据
- router,store,route
- `store.state.xxx`，并且 xxx 是数组或对象
  常见不是的：
- 从 reatcive 对象直接赋值等于的，比如`const newList = todoList`，todoList 是但 newList 不是
- `store.state.xxx`，并且 xxx 不是数组或对象
- route.query
  store 本身是响应式的，但是 state 和 getter 需要 computed 包裹才能保留响应性；
  并且一般数组或对象不需要，但是一般的数据需要，**但是还是建议都用 computer 包一下**

4. vue 动态改变样式要用`:class`而最好不要用`:style`，可以通过三元改样式
