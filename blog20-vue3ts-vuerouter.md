---
title: vue3+ts+vue-router4配置及简单使用注意
date: 2021-11-25 22:24:33
tags: 日常学习
categories: Vue
cover: /img/vue3.png
---

## 安装配置

同 vuex,在选择时直接选择 vuerouter
然后会产生一个 router 文件夹下的 index 为核心配置文件
文件结构及简单介绍如下:

### index.ts

```ts
//index.ts
import { createRouter, createWebHistory, RouteRecordRaw } from "vue-router";
import Home from "../views/Home.vue";
import Message from "../views/Message.vue";

//一般这里是预设好的,routes的类型是RouteRecordRaw
const routes: Array<RouteRecordRaw> = [
  {
    path: "/",
    redirect: "/home",
  },
  {
    path: "/home",
    component: Home,
    name: "Home",
  },
  {
    path: "/about",
    name: "About",
    //懒加载
    component: () => import("../views/About.vue"),
    children: [
      //子路由不用再写父路由的路径,在组件中push也同理
      {
        path: "/",
        redirect: "/msg",
      },
      {
        path: "/msg",
        component: Message,
        name: "message",
      },
    ],
  },
];

const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),//后面的在生产环境要有，表示生产环境下的baseurl，在baseurl基础上的路由
  routes,
});

export default router;
```

解释:

- 使用 createRouter 创建路由,参数需要有 history 类型(通过 createWebHistory 或 createHashHistory 创建),还有 routes 数组
- routes 类型为`Array<RouteRecordRaw>`,注意记得导入
- 基本路由三项:path/component/name
- 可以通过`component: () => import("../views/About.vue")`懒加载,即在需要的时候才加载该页面
- 子路由不用再写父路由的路径,在组件中 push 也同理;子路由的'/'就是父路由的路径

### main.ts

use 到上面

```ts
import router from "./router";
createApp(App).use(router).mount("#app");
```

### 组件中使用

#### router-view

展示组件的地方,**注意为了防止混乱,建议尽量只在页面组件中使用**

##### 多个 router-view

> 有时候想同时 (同级) 展示多个视图，而不是嵌套展示，例如创建一个布局，有 sidebar (侧导航) 和 main (主内容) 两个视图，这个时候命名视图就派上用场了。你可以在界面中拥有多个单独命名的视图，而不是只有一个单独的出口。如果 router-view 没有设置名字，那么默认为 default。

比如下面这样:

```html
<router-view class="view left-sidebar" name="LeftSidebar"></router-view>
<router-view class="view main-content"></router-view>
<router-view class="view right-sidebar" name="RightSidebar"></router-view>
```

然后在对应的层级中需要配置名称对应的组件,即扩展 component 属性

```ts
{
    path: '/',
    //注意这里是components
    components: {
        default: Home,
        // LeftSidebar: LeftSidebar 的缩写
        LeftSidebar,
        // 它们与 `<router-view>` 上的 `name` 属性匹配
        RightSidebar:RightSidebar,
    },
},
```

#### 访问 router 对象

从 vue-router 导出 useRouter 和 useRoute,然后创建实例并使用

```ts
import { useRouter, useRoute } from "vue-router";
const router = useRouter();
const route = useRoute();
```

- useRouter 是访问一些操作路由的方法,比如 push,replace 等
- useRoute 访问路由属性,比如参数,fullpath 等
- 动态路由:

```ts
import { useRouter, useRoute } from "vue-router";
const router = useRouter();
router.push("/home");
//传参
router.push({ path: "/msg", query: { msg: "hello world!" } });
```

**router 和 route 都可以不引入而直接在模板里边用$router和$route 访问**

#### 路由传参

通过 query 传参,数据会自动放在路径中,注意 query 参数是一个对象

```ts
router.push({ path: "/msg", query: { msg: "hello world!" } });
//接受参数
const userName = route.query.msg;
//或者直接在模板
$route.query.msg;
```

#### 路由守卫

##### 在组件中直接

通过 onBeforeRouteLeave, onBeforeRouteUpdate,onBeforeRouteEnter 来做到检测

```ts
onBeforeRouteLeave((to, from) => {
  const answer = window.confirm(
    "Do you really want to leave? you have unsaved changes!"
  );
  // 取消导航并停留在同一页面上
  if (!answer) return false;
});

const userData = ref();

// 与 beforeRouteLeave 相同，无法访问 `this`
onBeforeRouteUpdate(async (to, from) => {
  //仅当 id 更改时才获取用户，例如仅 query 或 hash 值已更改
  if (to.params.id !== from.params.id) {
    userData.value = await fetchUser(to.params.id);
  }
});
```
##### 在配置中规定
###### 全局
一般写在组件后面,router.xxx调用
**类型**:
- beforeEach
- afterEach
- beforeResolve
- afterResolve
**参数**:to from next
- to/from 去 来的路由参数,是路由的name属性
- next是可选属性,用于正常执行下一步;不写的话会导致路由中断
**next的使用**:**确保 next 在任何给定的导航守卫中都被严格调用一次**,也就是如果需要判断,就需要在每个情况都next,防止中断
```ts
//官网的例子
router.beforeEach((to, from, next) => {
  if (to.name !== 'Login' && !isAuthenticated) next({ name: 'Login' })
  else next()
})
```
next可以传递参数,传递到的就是对应的路径;也可以不传参,就按照原本的路径走

**返回值**:false或一个路由地址
- false,阻止访问
- 路由地址,相当于push('/xxx')
```ts
router.beforeEach((to,from)=>{
    //...
    return false
})
```
###### 独享
一般写在路由对象内部
- beforeEnter
> beforeEnter 守卫 只在进入路由时触发，不会在 params、query 或 hash 改变时触发。
```ts
//官网的例子
const routes = [
  {
    path: '/users/:id',
    component: UserDetails,
    beforeEnter: (to, from) => {
      // reject the navigation
      return false
    },
  },
]
```
