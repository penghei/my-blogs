---
title: vue3+ts+vuex4配置及简单使用注意
date: 2021-11-25 15:56:27
tags: 日常学习
categories: Vue
cover: /img/vue3.png
---

## 安装配置

在创建项目时自选选择 vuex 和 router,但是 vuex 需要简单改动,router 基本不变:
vuex 改动配置如下

- 引入 InjectionKey,以及 createStore,useStore,Store

```ts
import { InjectionKey } from "vue";
import { createStore, useStore as baseUseStore, Store } from "vuex";
```

- 创建 state 的接口以及 key,用于在 vue 的任何地方确定 state 的形状,不然在其他组件里边 state 就是 any

```ts
export interface IState {
  //创建state类型
  count: number;
}
//创建一个key用于在vue的任何地方确定state的形状,不然在其他组件里边state就是any
export const key: InjectionKey<Store<IState>> = Symbol();
```

- 创建 store

```ts
export const store = createStore<IState>({
  state: {},
  mutations: {},
  actions: {},
  modules: {},
});
```

- **创建自己的 useStore**

```ts
export function useStore() {
  return baseUseStore(key);
}
```

自己导出一个 useStore,里边的是库函数 useStore,使用的时候相当于用的是自己的 useStore

- 完整文件:

```ts
//store/index.ts
import { InjectionKey } from "vue";
import { createStore, useStore as baseUseStore, Store } from "vuex";

export interface IState {
  //创建state类型
  count: number;
}
//创建一个key用于在vue的任何地方确定state的形状,不然在其他组件里边state就是any
export const key: InjectionKey<Store<IState>> = Symbol();
export const store = createStore<IState>({
  state: {
    count: 0,
  },
  mutations: {
    increment(state) {
      state.count++;
    },
    decrement(state) {
      state.count--;
    },
  },
  actions: {},
  modules: {},
});

//自己导出一个useStore,里边的是库函数useStore,使用的时候相当于用的是自己的useStore
export function useStore() {
  return baseUseStore(key);
}
```

- 在 main.ts 中引入
  **!!!注意这里一定要把 store 和 key 都用到,不然会提示 store 是 undefined**

```ts
//main.ts
import { store, key } from "./store";
createApp(App).use(store, key).mount("#app");
```

- 在组件中使用

```ts
import { useStore } from "../store";
const store = useStore();
store.commit("...");
//或
store.commit({
  type: "...",
  payload: value,
});
const value = computed(() => store.state.value);
```

**注意需要用 computed 保留响应性**

## 使用注意

### v-model

不能直接 commmitv-model 中的数据,可以通过@input 事件触发提交函数

```ts
<template>
  <input v-model="myInput" @input="getInput" />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { useStore } from "../store";
const store = useStore();

const myInput = ref("");
function getInput() {
  store.commit("getInput", myInput.value);
}
</script>
```

### 拆分

根据官方文档的建议是这样:

```
store
    ├── index.js          # 我们组装模块并导出 store 的地方
    ├── actions.js        # 根级别的 action
    ├── mutations.js      # 根级别的 mutation
    └── modules
        ├── cart.js       # 购物车模块
        └── products.js   # 产品模块
```

其中 modules 可以是各个 state,然后在 index 中组合使用
或者也可以一个 store 和 mutations 组合在一起,然后分成不同的文件

