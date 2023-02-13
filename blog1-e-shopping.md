---
title: vue练手项目-电商网页
date: 2021-10-20 20:00:00
tags: 项目
cover: /img/starry4.jpg
categories: Vue
description: 一个小的实战练习,算是为自己增加了一点点经验吧,在这里总结一下,希望以后vue的使用更熟练
---

# vue 项目实战--电商网页

---

## 写在前面

&emsp;&emsp; 上篇博客打算做点 vue 项目练练手,最近在网上找了一个练手的项目.使用的是 vue2+elementui+nodejs 做的电商网站,其中还是有很多新尝试的东西和之前从没有过的船新体验,打算写一篇博客记录一下这个小项目做完之后的经验和感想.这个项目整体可能有还需要完善的地方,但是现在已经是一个 demo 版了,代码大概在 3000 行左右.

放一下github链接:https://github.com/penghei/e-commerce-web

## 使用技术

- vue vuex vue-router
- elementUI 感觉不是很好用,回头考虑一下其他库,那个 el-container 是真的不能当做整体布局用真的恶心
- nodejs 第一次写,主要还是接发请求和文件读写

## 项目展示

### 页面构建

#### 登录注册

![login.png](https://i.loli.net/2021/10/20/abh81owmU5jZLiK.png)
&emsp;&emsp; 首先是登录界面,很简单的一个登录框和注册框。上面的登录注册标签实际上是 elementui 自带的标签，但是其实可以做成路由，只是这里没有特别的必要。登录和注册会有非空验证。登录和注册都会触发服务器的验证账户事件，如果是注册会新创建一个用户信息，登录则会先获取并在 vue 再进行验证。这个后面在 node 再详细介绍。注册完成会跳转到登录，目的是为了创造在线状态；登录之后就是`this.$router.replace("/home");`跳转
&emsp;&emsp; 这里注册的时候其实是完整创建了一个用户信息,后面只做填充.

```js
let userInfo = {
        account: {
          name: this.formInner.userName,
          code: this.formInner.userCode,
        },
        privacy:{
          sex:'',
          age:'',
          avatar:'',
          hobby:[]
        },
        shopping:{
          shoppingCar:[],
          money:100,
          payCode:''
        },
        isOnline:false
      };
```

#### 主界面

![home.png](https://i.loli.net/2021/10/20/3Y4rmd8uL5BqSZT.png)
![footer.png](https://i.loli.net/2021/10/20/P4dVKXFnYUtcgSa.png)
&emsp;&emsp; 主界面分为:导航栏/走马灯/面包屑导航/商品展示/侧边推荐栏/底部.导航栏和走马灯伴随每个组件,底下的都是路由而已

##### 导航栏

![navlist.png](https://i.loli.net/2021/10/20/GwOfl3NxprXmbBk.png)
&emsp;&emsp; 左边是主页,右边图片是用户头像,购物车图案就是购物车了.导航栏有自适应效果,媒体查询不能获取页面大小,所以这里用了 js

```js
window.onresize = () => {
      return (() => {
        this.screenWidth = document.body.clientWidth;
        this.getWidth();
      })();
    };
//调整
getWidth() {
      if (this.screenWidth < 600) {
        return;
      } else {
        this.left = this.screenWidth - 600;
        this.left1 = this.screenWidth - 500;
      }
    },
```

简单来说就获取页面宽度,然后改变头像和购物车的左移量.

##### 走马灯

![zoumadeng.png](https://i.loli.net/2021/10/20/pykPeHfxTijh59g.png)
&emsp;&emsp; 使用的是 elementui 自带的走马灯组件.这里有个小坑就是响应式的问题,最开始页面缩小的话图片比例不会变而会自己缩上去.后来改了图片的 min-height 把图片大小卡死就不会出现这种情况;同理也可以设置 max-height 等

##### 筛选

![selecte.png](https://i.loli.net/2021/10/20/Iacfn3ig4GrsFXR.png)
&emsp;&emsp; 筛选是一个单独的组件,使用的 eleu 提供的多选框.然后把选择的值通过 pubsub 传给 shoppingList 来进行更改.因为数据较少而且要实时更新就用了 pubsub 而不是 vuex

##### 排序

![sort.png](https://i.loli.net/2021/10/20/oUlnOG96wxVkmaI.png)
&emsp;&emsp; 可选择排序方式,这里价格排序的方法就是把商品元素的价格属性提出来排序.这是具体排序:

```js
//compare函数
 const compare = (prop) => (obj1, obj2) => {
        var val1 = obj1[prop];
        var val2 = obj2[prop];
        if (val1 < val2) {
          return -1;
        } else if (val1 > val2) {
          return 1;
        } else {
          return 0;
        }
      };
//排序
switch (this.sortType) {
        case "sort1":
          this.goods.sort(compare("price"));
          break;
        case "sort2":
          this.goods = this.goods.sort(compare("price")).reverse();
          break;
        case "sort3":
          this.goods.sort(compare("name"));
          break;
        default:
          break;
      }
```

compare 函数的写法:实际上是先 return 这个参数为(obj1,obj2)的函数,然后这个函数内部比较的是两个的值(相当于相邻两者),分别返回 01-1,表示从小到大排序,然后放到 sort 函数的参数就行

##### 商品列表

![home.png](https://i.loli.net/2021/10/20/3Y4rmd8uL5BqSZT.png)
&emsp;&emsp; 这个就是重头戏了,商品的信息/渲染方式/点击后的交互都跟这个组件有关系.首先从服务器获取商品信息然后通过循环渲染到界面上,展示内容;这个没啥说的.然后就是点击查看详细信息的问题了.刚开始一直想不到怎么可以在新界面拥有当前渲染的信息,换句话说就是怎么可以确定循环中是哪个 goods 具体在被渲染.然后这是解决办法:

```js
@click="goodDetail(good.id, good.name, good.price, good.picUrl)"
//......中间
goodDetail(id, name, price, picUrl) {
      this.$router.push({
        path: `/home/goodsdetail/goodsCommit`,
      });

      this.$store.commit("setSelectedGoodsInfo", { id, name, price, picUrl });
    },
```

&emsp;&emsp; 其实很简单,就是传参数!因为在 v-for 内部可以拿到具体的元素,那就把元素通过参数传进去就完事了.这个方法还可适用于好多地方.这里交互的方法是用了 vuex 把选中的信息交上去,然后后面的组件都可以利用这个 selectedGoods 来渲染.后面会说 vuex 内的具体交互

#### 购物车

![shoppingCar.png](https://i.loli.net/2021/10/20/DFdzT2mjehfOa3l.png)
&emsp;&emsp; 当用户把商品加入购物车时是把当前的 selectedGoods 交给了 vuex 和服务器;服务器不负责直接渲染,而是绑定购物车和用户;vuex 的购物车数据则会在挂载时直接渲染.

```js
if (this.$store.state.shoppingCar.length !== 0) {
      this.list = [...this.$store.state.shoppingCar];
    }
```

&emsp;&emsp; 删除没啥说的,从 this.list 删除的同时也从 vuex 中删除;但是立即购买这个选项有点特殊:他是直接更改 vuex 中的 selectedGoods,然后跳转到支付.由于支付依赖的是 vuex 的 selectedGoods,因此就可以做到"选对正确的商品及其属性"

#### 用户信息

![user.png](https://i.loli.net/2021/10/20/vk5AcinzoXdYZ8T.png)
&emsp;&emsp; 展示用户信息,如果没有就用的默认.这里用户头像是自己上传的:
![changeUser.png](https://i.loli.net/2021/10/20/KIZPATkLiqWsnQg.png)
&emsp;&emsp; 其他的都是同时改变 vuex 和服务器,这样可以保证渲染和数据改变.这个上传图片很麻烦,整了半天没出来;后来直接把上传的图片转成 base64 然后存起来,展示的时候直接 src=""就可以了.转码方法如下:

```js
 let reader = new FileReader();
      reader.readAsDataURL(file)
      reader.onload = ()=>{
        let pic = {
          picBase:reader.result
        }
      }
```

![changeUserAccount.png](https://i.loli.net/2021/10/20/kol8RiwGcIDayxN.png)
&emsp;&emsp; 更改账户信息:就是更改密码/支付密码/充值什么的,操作跟上面差不多,但是不再展示直接调用接口修改数据就行

#### 商品详情

![goodsDetails.png](https://i.loli.net/2021/10/20/bvdyWMeN19mGSlz.png)
&emsp;&emsp; 这一步展示的商品就是上面说的来自于 vuex 的 selectedGoods,图片信息/价格/名称都是来自于此.然后加入购物车也就相当于把这个 selectedGoods 加上当前页面的数量属性塞入到 vuex 的 shoppingCarList,后面有用时直接读取就行
&emsp;&emsp; 评论组件:本来评论组件写了一段很复杂的和缓存之间的交互,大概逻辑是为了跟用户绑定,所以输入评论时判断当前登录的是哪个,然后名字为他,评论加在他的数据中;再把这条评论和商品绑定,根据商品来展示评论.等于说一个评论要绑定用户和商品两个对象.

```js
let commits = {
      commits: this.commitList,
      goodsName: this.$store.state.selectedGoods.name,
    };
    let oldCommits = JSON.parse(localStorage.getItem("commits")) || [];
    if (oldCommits.length !== 0) {
      let addObj = oldCommits.find((obj) => {
        return obj.goodsName === this.$store.state.selectedGoods.name;
      });
      if (addObj !== undefined) {
        addObj.commits = [...this.commitList];
      } else {
        oldCommits.push(commits);
      }
    } else {
      oldCommits.push(commits);
    }
    localStorage.setItem("commits", JSON.stringify(oldCommits));
  },
```

#### 订单和支付

![orders.png](https://i.loli.net/2021/10/20/JnvE43UNXMdICPR.png)
![pay.png](https://i.loli.net/2021/10/20/48QlWR2uaovDMTX.png)
&emsp;&emsp; 这两个其实就是简单的表单验证,这个确认商品就是利用的 vuex 的 selectedGoods;支付接口会先获取用户的 money,然后计算 number\*price,再扣钱;如果钱不够就会返回交易失败.这个支付密码的输入是输完 6 位数字立马验证,实现方法:

```js
 examineCode() {
      let len = this.payCode.length;
      if (len >= 6) {
        this.$refs.input.blur();
        this.handleConfirm();
      }
    },
```

原生的 change 方法就可以,这里 eleu 没有所以用的@input 事件

#### vuex

vuex 的全部内容:

```js
export default new vuex.Store({
  state: {
    checkSelect:1,
    goodsList: [],
    selectedGoods:{},
    username:'',
    useravatarUrl:'',
    cascaderValue:[],
    shoppingCar:[],
    selectedFromCar:{}
  },
  mutations: {
    setCheck(state,data){
      state.checkSelect = data;
    },
    setGoodsList(state, data) {
      state.goodsList = data;
    },
    setSelectedGoodsInfo(state,data){
      state.selectedGoods = {...data}
    },
    emptySelectedGoods(state){
      state.selectedGoods = {};
    },
    setSelectedGoodsNum(state,data){
      state.selectedGoods["number"]=data
    },
    selectFromCar(state,data){
      state.selectedFromCar = data
    },
    setUserName(state,data){
      state.username = data
    },
    setavatar(state,data){
      state.useravatarUrl = data;
    },
    setCascader(state,data){
      state.cascaderValue = [...data];
    },
    emptyCascaderValue(state){
      state.cascaderValue = []
    },
    addShoppingCar(state,data){
      let foundGoods = state.shoppingCar.find(obj=>{
        return data.name === obj.name
      })
      if(foundGoods !== undefined){
        foundGoods.number += data.number
      }else{
        state.shoppingCar.push(data)
      }
    },
    changeShoppingCar(state,data){
      let newcar = state.shoppingCar.filter(obj=>{
        return obj.name !== data;
      })
      state.shoppingCar = [...newcar];
    },
    allPushShoppingCar(state,data){
      state.shoppingCar = [...data]
    }
  },
});

```

这次用 vuex 大多数只是存数据,对数据的操作没怎么写都是写在了 vue 组件里边;下次使用可以试着改进
简单解释一下几个重点数据:

- goodsList:首页的商品列表
- selectedGoods:交互最多的那个了,个人感觉是解决了好多问题
- shoppingCar:购物车
- emptySelectedGoods:清除掉 selectedGoods,在回到首页的时候会调用,确保不会选重了

### nodejs 服务器

#### 跨域代理问题

&emsp;&emsp; vue 解决跨域问题最常用的方法:在 vue.config.js 设置:

```js
module.exports = {
    devServer: {
      proxy: {
        //配置跨域
        '/api': {
          target: 'http://127.0.0.1:7999', //这里填写项目真实的后台接口地址
          changOrigin: true, //设置允许跨域
          pathRewrite: {//这个重写不可省略！因为我们真正请求的地址并不含 /api
            '^/api': ''
              /* 当我们在浏览器中看到请求的地址为：http://localhost:8080/api/data/getdata 时
              因为重写了 /api ，所以实际上访问的地址是：http://x.x.x.x:x/data/getdata，
              */
          }
        }
      }
    }
  }
```

后面单独写个博客总结一下吧

#### 服务端

&emsp;&emsp; 因为之前从没用过 nodejs,这次算是第一次实践.

- 基本的 get/post
  `.get("/api", (req, res) => {}`
  `.post("/api", (req, res) => {}`
  大概就是这样.然后返回的值写简单点就是 res.end(),**记住 end 的参数一定要是字符串**,否则就要 JSON.stringfy 了不然会报错
- 文件读写
  使用 json 做一个简单的数据存储,每次 post 修改都要先读再写
  读:
  `fs.readFile("database.json", "utf8", (err, data) => {}`
  写:
  `fs.writeFile("database.json", JSON.stringify(list), (err) => {}`
  注意写的时候也是要 stringify
  当然这只不过是最简单的文件读写,后面的一些其他操作还要再好好学习......
- res.body 读不出来的问题
  一开始出现了 res.body 读不出数据的问题,谷歌一下方法如下:

```js
app.use(
  bodyParser.json({
    limit: "50mb", //nodejs 做为服务器，在传输内容或者上传文件时，系统默认大小为100kb,改为10M
  })
);
app.use(
  bodyParser.urlencoded({
    limit: "50mb", //nodejs 做为服务器，在传输内容或者上传文件时，系统默认大小为100kb,改为10M
    extended: true,
  })
);
```
  如果不需要改get的大小,limit不用写,extened写false即可
  这里还扩大了get的数据量,不然会报get获取数据过大的错
- 存留的问题:
  post一般是新建一个数据,但我的api有很多都是更新和删除,所以应该考虑使用`patch`和`delete`
  这个可能得后面学习restful规范,留个坑等学到了再来填坑
### json 临时充当数据库

&emsp;&emsp; 因为之前没接触过数据库,这次就是简单使用了 json 来存数据.后面学会了 mongodb 的话还是用数据库好一些
&emsp;&emsp; 数据格式:

```json
[
    {
        account: {
          name: this.formInner.userName,
          code: this.formInner.userCode,
        },
        privacy:{
          sex:'',
          age:'',
          avatar:'',
          hobby:[]
        },
        shopping:{
          shoppingCar:[],
          money:100,
          payCode:''
        },
        isOnline:false
    }
]
```

按照这个格式存用户信息.内部一个对象就是一个用户

### 其他

- 商品数据:
  &emsp;&emsp; 同样存在了 json 里,数据格式:

```json
[
  {
    "name": "可爱白色小猫猫",
    "price": 1299,
    "picUrl": "https://i.loli.net/2021/10/18/PMuY6oJEhBik82f.jpg"
  },
]

```

- 路由
  &emsp;&emsp; 说实话这次路由写的很不好,很多跳转没有使用参数,routerview 也放得很乱.下次需要提前规划好路由,有哪些跳转,在哪展示等等

```js
routes: [
    {
      path: "/login",
      component: login,
    },
    {
      path: "/main",
      component: allMainPages,
      children: [
        {
          path: "/home",
          component: home,
          children: [],
        },
        {
          path: "/home/user",
          component: userData,
        },
        {
          path: "/home/goodsdetail",
          component: goodsDetails,
          children: [
            {
              path: "/home/goodsdetail/address",
              component: address,
            },
            {
              path: "/home/goodsdetail/parchase",
              component: parchase,
            },
            {
              path: "/home/goodsdetail/goodsCommit",
              component: goodsCommit,
            },
            {
              path: "/home/goodsdetail",
              redirect: "goodsCommit",
            },
          ],
        },
        {
          path: "/home/shoppingCar",
          component: shoppingCar,
        },
      ],
    },

    {
      path: "/",
      redirect: "login",
    },
  ],
```

&emsp;&emsp; 经过这次写完之后大概有了一点点经验:

- 首先,最开始一般都是分的比较开的路由,比如登录和主界面没啥关系,直接在最开始分开写;而后面的几乎都是 home 内部的路由关系,就都在子路由里边
- 有子路由,就要父路由下放 router-view 在合适的地方.比如这个/main 的组件内部就是这样的:

```html
<div>
    <nav-list></nav-list>
    <carousel></carousel>
    <router-view></router-view>
  </div>
```

然后,goodDetails 组件也有子组件,于是就在他下面也需要放 router-view
这样就基本理清了展示和组件的关系.最大的组件就直接放在 app 里边,等于说 app 只是一个所有路由的根页面

- 重定向:redirect 后面的值是字符串!代表的是在上面这个 path 下展示的组件

## 经验总结

### 客户端
#### 整体
- 文件名和组件名
&emsp;&emsp; **一定要设计好文件名!** 详细的可以看vue的风格指南.这是更改之后的文件名:(看着整齐多了)
![QQ截图20211021000617.png](https://i.loli.net/2021/10/21/Eu7BOIzAkhVxyHo.png)
- 路由的规划和设计
&emsp;&emsp; 1. 首先要规划好路由,什么地方需要什么地方不需要,以及要把路由插入在哪,谁是谁的子路由,谁会跳转到谁那里
&emsp;&emsp; 2. 有子路由,父路由下就要放 router-view 在合适的地方;最大的组件就直接放在 app 里边,等于说 app 只是一个所有路由的根页面
&emsp;&emsp; 3. 数据不多的情况下可以考虑多用query pamar参数而不是pubsub和vuex
- vuex的使用
&emsp;&emsp; 1. vuex的使用相对于redux实在是简单太多了,因此建议交互的组件5个以上就可以写一下,反正也不难写
&emsp;&emsp; 2. vuex除了存数据还可以执行纯函数方法,计算什么的,可以把一些复用性高的方法(比如查找遍历删除这些)放在commit或者dispatch里,不一定只是存数据
&emsp;&emsp; 3. 活用生命周期和vuex做好交互(这个有点玄但是确实很是经验)
- 数据
&emsp;&emsp; 1. 其实之前写别的项目就有过这样的经验.当数据量增大的时候光凭脑子记忆其实很容易乱,所以可以把一些对象的结构写一个作为笔记的md里边.比如这次的用户就是一个很庞大的对象,如果不是反复查看肯定早就忘了
&emsp;&emsp; 2. vuex真是很有用的东西,哪怕是单个组件使用都可以放进去,但是**一定要设计好变量名或者写好注释**,不然更乱了
&emsp;&emsp; 3. 多采用对象数组find filter这种比较高效的方式,但是同时也要注意好变量名设置
&emsp;&emsp; 4. **一定要设计好变量名或者写好注释**
&emsp;&emsp; **一定要设计好变量名或者写好注释**
&emsp;&emsp; **一定要设计好变量名或者写好注释**
- 组件和复用
&emsp;&emsp; 1. 这次组件加上页面的话差不多有27个,如果不设计好名字的话真的会找不到在哪.所以还是要设计好组件名,保证有辨识度并且自己能看懂
&emsp;&emsp; 2. 在写nodejs的时候有很多重复写的部分,可以考虑写一个可复用的代码然后export出来用,但是还没试过,**下个项目尝试一下**(或者vue3的hooks);vue2也有自带的mixins,这次同样也没用到多少
&emsp;&emsp; 3. 文件夹分类也得弄好,不然新增加功能连往哪放都不知道
- 布局
&emsp;&emsp; 1. 这次主要用了UI库提供的来布局,以后可以多尝试其他UI库的布局和原生的flex布局
&emsp;&emsp; 2. 尽可能少嵌套div,不然后面找个元素css都不知道从哪改
&emsp;&emsp; 3. **scoped很重要!!!** 除非一定要改原生UI库,不然一定要加scoped!
&emsp;&emsp; 4. 响应式!!!很多简单的地方比如栏的划分,在一开始就应该考虑到
#### 局部
- v-for/v-if/v-bind/v-model
&emsp;&emsp; 1. v-for上面写到过的传参解决确定哪个正在被渲染的问题
&emsp;&emsp; 2. v-if和v-else解决展示el-empty的情况很好用
&emsp;&emsp; 3. v-model yyds!!
- axios
&emsp;&emsp; 这次用了很多.then有点繁琐,如果是单函数的话其实await是更好的选择;比如:
```js
axios
  .get('/api')
  .then(res=>{
      let list = res.data;
      //....
  }) 
```
可以写成:
```js
async function getList(){
    let res = await axios.get('/api')
    //...
}
```
其实就是这么理解:get获取的res其实包了一层promise,需要.then解开;而await也可以解开,只不过是写在正常语句即可
- css
&emsp;&emsp;  之前css基础不好的我吃大亏,很多可能很好调的部分调了很久,这里写几个经验点吧
&emsp;&emsp; 1. vh可以代替百分比,尤其是height:100vh
&emsp;&emsp; 2. 多用min-height/max-width这种
&emsp;&emsp; 3. 固定式底部布局:
```css
.content {
  min-height: calc(100vh - 100px);
}
.footer {
  height: 100px;
  width: 100%;
}

```
&emsp;&emsp; 4. color可以操作子元素字体颜色,但是直接放在p/h1就不行;同理font-size之类的最好放在字体的直接父元素
&emsp;&emsp; 5. 解决元素并排的问题可以多用float,inline感觉不太有用;同理为了不并排可以加个<br>
&emsp;&emsp; 6. 其他还没想好,以后遇到再补充吧
### 服务端
- nodejs
&emsp;&emsp; 1. 因为是第一次写所以还是很多不太熟练的地方,但是基本的请求和文件读写差不多会了,当然还是要了解上传和下载文件
&emsp;&emsp; 2. 代码重复的地方很多,应该有很多可以复用的地方,后续思考一下怎么复用代码
&emsp;&emsp; 3. post请求最后要记得写入!要不然很多就没改!!

## 后期规划
- 先不着急再做项目练手,可以先巩固css相关/vue3/vue2知识然后学完整vue之后再考虑做下一个项目.
- 多看文档,好好看视频,不要放过边角,搞懂原理,甚至可以去看源码
- 下一次做项目要尽可能用上这次没用到的:
    - 插槽
    - 混入
    - props
    - 自定义事件
    - 新东西:
        - vue3
        - typescript
        - mongodb
- 寻找学习一些css库,做一些简单的css练习,然后把常用的好用的会用到的可以写在博客里

## 总结
&emsp;&emsp; 这次算是我上学以来写过最长的代码了.虽然还是有很多不完善的地方,但是代码量和组件数都是以前从来没有达到过的.加上这次,js的代码量可能都已经超过5000行了甚至更多,而vue就占了将近3/4.要说感想,我个人感觉就是更熟练了,但是需要学习新东西并使用;不然一直在用旧的东西反复练习也只是原地踏步,就像为了增加代码量写一万行helloworld一样.未来需要学更多用更多并且挑战更难的,至少在今年,我希望我的js/ts代码量可以接近1w行,vue也最起码要再写3000行以上;剩下的时间可以给react,总之要在这学期熟悉这两门框架,并且把边边角角都要搞懂,而不是一知半解只会使用.



