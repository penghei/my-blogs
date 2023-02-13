---
title: vue2平时没怎么用过但是很有用的知识点总结
date: 2021-10-24 16:37:56
tags: 日常学习 
categories: Vue
cover: /img/vue.png
---
## MVVM
Model-View-ViewModel
- Model代表数据模型，可以在Model中定义数据修改和操作的业务逻辑。比如vue中的methods/components/data等
- View 代表UI 组件,比如vue中的template
- ViewModel 监听模型数据的改变和控制视图行为、处理用户交互，简单理解就是一个同步View 和 Model的对象，连接Model和View。
View数据的变化会同步到Model中，而Model数据的变化也会立即反应到View 上。这是一种双向数据绑定,两者同步完全自动,因此不需要数据状态维护

## 生命周期
1. beforeCreate
2. created
3. mounted
4. beforeUpdate
5. update
6. beforeDestory
7. destoryed

## 插槽
在父组件中定义插槽分发内容,在子组件中放入<slot></slot>来确定放置位置
在父组件中这样写
```html
<Mycom>name</Mycom>
```
然后在Mycom的模板中写入要插入的位置
```html
<h1><slot></slot></h1>
```
具名插槽:给插槽一个名字,自组件可以根据名字确定放的位置
```html
<Mycom>
    <template v-slot:name1>
        <h1>zzx</h1>
    </template>
    <template v-slot:name2>
        <p>xiaoming</p>
    </template>
</Mycom>
```
```html
<div><slot name="name1"></slot></div>
<div><slot name="name2"></slot></div>
```
另外,插槽在模板中也可以用#简写,比如#name1 = v-slot:name1
## 指令
就是定义一个类似v-show的东西,分为全局和局部
- 全局
```js
Vue.directive('focus',{
    inserted:(el)=>{
        el.focus()
    }
})
```
- 局部
```js
directives:{
    focus:{
        inserted:(el)=>{
            el.focus()
        }
    }
}
```
然后就可以这么用
`<input v-focus/>`
效果就是自动聚焦
此外的几个钩子替代inserted:
- bind 第一次绑定到元素时触发,相当于一次性的
- update 所在组件的 VNode 更新时调用
- componentUpdated：指令所在组件的 VNode 及其子 VNode 全部更新后调用。
- unbind：只调用一次，指令与元素解绑时调用。
然后是写在括号中的参数
- el 指令插入的对象,相当于直接是dom元素
- binding
    - name：指令名，不包括 v- 前缀。
    - value：指令的绑定值，例如：v-my-directive="2" 中，绑定值为 2。
    - oldValue：指令绑定的前一个值，仅在 update 和 componentUpdated 钩子中可用。无论值是否改变都可用。
    - expression：字符串形式的指令表达式。例如 v-my-directive="1 + 1" 中，表达式为 "1 + 1"。
    - arg：传给指令的参数，可选。例如 v-my-directive:"foo" 中，参数为 "foo"。
    - modifiers：一个包含修饰符的对象。例如：v-my-directive.foo.bar 中，修饰符对象为 { foo: true, bar: true }。

## 混入
mixin相当于一个小的模块,里边可以有生命周期钩子/computed等
定义:
```js
let Mixin = {
    data(){
        return{
            name:'aaa'
        }
    },
    methods:{
        sayHello(){
            console.log(`hello,${this.name}!`)
        }
    }
}
```
可以把mixins写到外部一个文件中
应用:
```js
import Mixin from '../mixins.js'
export default{
    name:'Mycomponent',
    mixins:[Mixin],
    //...
    methods:{
        hello(){
            this.sayHello();//使用this引入
        }
    }
}
```
## 过滤器
在vm中定义一个filters,可以对`{{}}`中和v-bind的内容进行过滤
举个栗子:

```html
//html
<div>{{name|defaultName}}</div>
<div :id="id|defaultId"></div>
```
```js
//vm
export default{
    name:"Mycomponent"
    //...
    filter:{
        defaultName(value){
            if(typeof this.name.toString() === 'number'){
                return this.name.splice('').push('_').toString()
            }
        },
        defaultId(value){
            if( typeof +value !== 'number'){
                return Date.now()
            }
        }
    }
}
```
相当于在|之后定义了一个变量,并且能取到|前面的值;在过滤器对值进行操作,最后的值是你操作后的

## vue的一些api
- data:数据,这里要注意的是data中定义的以_或者$开头的属性是访问不了的
- props:父元素传来的props,一般用的时候最好规定传入格式:
```js
props:{
    name:Number,//[Number|String]表示多个类型
    age:{
        type:Number,
        default:18,//默认值,还可以是是个函数
        required: true,
        validator(){
            //自定义验证函数
        }
    }
}
```
- watch:一个对象，键是需要观察的表达式,一般是data中的内容;值是对应回调函数。值也可以是方法名，或者包含选项的对象。
```js
export default{
    data(){
        return{
            a:1,
            b:{
                c:2
            }
        }
    },
    watch:{
        a:function(val,oldval){
            if(oldval - val > 5){
                //...
            }
        },
        'b.c':function(val,oldval){//用引号表示一个表达式进行深拷贝

        },
        b:function(val,oval){
            deep:true//可以观测到对象内部值
        }
    }
}
```
注意不要用箭头函数来作为回调函数.引用官网上的一句话:
> 理由是箭头函数绑定了父级作用域的上下文，所以 this 将不会按照期望指向 Vue 实例

- $parent $children 可以直接访问父组件和子组件.this.$children是一个数组,是所有子组件.可以通过这直接更改父子组件的实例内容,甚至data等
- $root 根实例,嵌套的话就是最高层的实例
- $solt 具名插槽的访问,是插槽的具体内容 比如定义了一个`<Mycom v-solt:header>zzx</Mycom>`,就可以通过`this.$solt.header`获取'zzx';默认插槽也可以
- $isServer 判断是否运行于服务器
- $attrs "给爷传值",祖组件定义一个props,中间的父组件用v-bind='$attrs'接一下,最后的子组件用props正常接就可以.并且在子组件可以通过this.$attrs获取
- $listener 跟上面的差不多,但是要用v-on绑定
- $watch 简单检测一个值,就不用专门写在watch里边了.用法:
```js
this.$watch(a,function(newval,oldval){//...},{
    //其他属性
    deep:true,
    immediate:true
})
```
- $forceUpdate 可以令当前实例重新渲染.与之相配可以用$nextTick,就是在页面刷新后的一个钩子
- $props 可以获取到所有传过来的props
- $log,可以用来console.log一个模板里边的值.具体用法:
```js
// main.js
Vue.prototype.$log = window.console.log;
// 组件内部
<div>{{$log(info)}}</div>
```

## vue的官方风格指南里有用的东西
### 组件名
- 推荐用多个单词,比如"ToDoItem"不是"ToDo"
- 组件名和组件文件名最好用大驼峰,并且和引入时最好一样,比如"MyComponent"
- 比较基础的组件名要加一个合适的前缀. 比如说一个按钮  一个表格  或者一个图标可以统一加个"App""Base""V"这样的前缀
```
components/
|- BaseButton.vue
|- BaseTable.vue
|- BaseIcon.vue
```
- 如果一个组件经常用但是在其他页面组件或者其他别的组件中一次只出现一次(比如header  footer之类的),最好加一个前缀"The"
```
components/
|- TheHeading.vue
|- TheSidebar.vue
```
- 组件名称最好设计合理,并且相关的要紧密相连.比如说做一个后台管理页面,同一个类型的组件最好命名相似
```
components/
|- UserList.vue
|- UserListItem.vue
|- UserListItemButton.vue
```
- 组件名称应该以高阶的 (通常是一般化描述的) 单词开头，并以描述性的修饰词结尾。
这是官网上的推荐,个人理解就是首字母最好一样,这样看着整齐并且方便排序;然后比如说一个商品列表,就最好都命名为类似的"Shopping-"而不是"-shopping"
```
components/
|- SearchButtonClear.vue
|- SearchButtonRun.vue
|- SearchInputQuery.vue
|- SearchInputExcludeGlob.vue
|- SettingsCheckboxTerms.vue
|- SettingsCheckboxLaunchOnStartup.vue
```
- 组件名称应该是完整的,不要写缩写
### props
- 子组件引入props时,最好定义详细,规定好类型和是否必要等
```
props: {
  status: String
}
```
- 给props命名最好用小驼峰,而写在模板里最好用连字符  比如"myProps" "my-props"
### 属性
- 多个属性写在多行,不要堆在一行
- 带引号!!!
### 一些顺序
#### 实例内的书写顺序
```js
export default{
    name:'',

    components:{},
    directives:{},

    mixins:{},

    props:[],
    emits:[],

    setup(){},

    data(){},
    computed{},

    watch{},
    mounted(){}//以及其他生命周期

    methods:{},

    render(){}
}
```
#### 模板属性书写顺序
```html
<div
    :is=""
    v-for=""
    v-if=""//v-else v-else-if v-show 
    v-once=""
    id=""
    ref=""
    key=""
    v-model=""
    props
    @
    v-text//v-html
></div>
```
### 其他
- 永远不要在一个元素上同时使用 v-if 和 v-for。可以把v-if放在v-for的子组件上

