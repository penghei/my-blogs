---
title: less学习 (感觉会)常用的部分
date: 2021-10-24 23:18:12
tags: 日常学习
categories: CSS
cover: /img/less.png
---

## 变量

```less
@maxW: 100%;
@maxH: 100vh;
@footerH: 100px;
@mainH:@maxH - @footerH;
```

@xxx 为变量,全局定义即可全局使用,除了高度还可以是任何 css 数值

变量可以用各种运算符进行计算，不同单位的运算结果可能会有所不同
```less
// cm mm
@conversion-1: 5cm + 10mm; // result is 6cm
@conversion-2: 2 - 3cm - 5mm; // result is -1.5cm

// cm可以转化为px，但是反过来不行
@incompatible-units: 2 + 5px - 3cm; // result is 4px

// n% 也可以进行计算
@base: 5%;
@filler: @base * 2; // result is 10%
@other: @base + @filler; // result is 15%
```

## 映射

```less
#colors() {
  header: blue;
  footer: black;
  nav: gray;
  text: black;
  main: pink;
} //
```

可以通过`#colors()[header]`来获取,类比为 js 中对象

## 嵌套

```less
#header {
  //嵌套式写法,相当于在header下的数个选择器
  display: flex;
  flex-direction: column;
  .header-input {
    width: 100%;
    text-align: center;
    input {
      height: 20px;
    }
  }
}
```

这种写法内部的值相当于经过了一层外部的选择器,也就是

```css
#header .header-input input {
  /*...*/
}
```

也可以在内部放上媒体查询,相当于对该元素进行媒体查询

```less
#main {
  @media (max-width: 1000px) {
    //相当于对main的媒体查询
    display: none;
    @media (max-width: 500px) {
      //相当于 范围(500,1000)
      display: block;
    }
  }
}
```

媒体查询是一个冒泡效果,也就是从内向外判断

## 混合

```less
.bordered {
  border-top: dotted 1px black;
  border-bottom: solid 2px black;
}
#main {
  .bordered();
}
```

就是`.bordered()`的形式可以把整个 bordered 下的样式导入

## 扩展

```less
h2 {
  &:extend(.style);
  font-style: italic;
}
.style {
  background: green;
}
```

`&`为 父选择符 ,意思就是父级元素,相当于在 h2 上
extend 可以直接写到选择器上

```less
h2:extend(.style) {
  font-style: italic;
}
.style {
  background: green;
}
```

如果扩展的选择器内有相同名称的元素,则以内部为准

## 函数

- `image-size('')` 参数是 url,可以确定图片尺寸;也可以用`image-width()`和`image-height()`

## 总结

感觉 less 还有很多颜色和字符串等的操作函数.但是如果只是为了方便日常 css,个人感觉最实用的还是变量 嵌套 扩展 混入这几个,以及映射可以做到结构清晰,其他的可能用处不大.
