---
title:  vue开发中的常见问题
category:  vue
tags: vue
---

最近在学习vue的过程中，这里做一下总结：

# eslint静态检查

在大家用vue-cli创建工程的时候，会有一项，使用使用eslint，如果选择了y，那么工程就会安装并启用eslint。

<!--more-->

这里列举一下常见的错误：

1.多余的分号
![](http://img0.tuicool.com/bmqqMrB.png)
2.定义了却未使用的变量
![](http://img0.tuicool.com/AVbu2eB.png)

3.结尾多余空格
![](http://img1.tuicool.com/NFjAzae.png)

4.超过一行的空行
![](http://img1.tuicool.com/rYVFv2.png)

5.代码尾行应该有空行
![](http://img1.tuicool.com/m2yUri.png)

错误肯定是列举不完的，那么提示错误的时候，我们应该先去看提示信息（翻译），如果发现没有错误，可以对照 eslint的官方文档

在大家适应了eslint的写法后，效率和正确率会直线上升，这里安利下我的另一篇文章， 提升效率的eslint+vscode的开发环境搭建

# this指向

经常会有朋友问一些undifined的错误，比如：
```js
<script>
import Hello from './components/Hello'

export default {
  data () {
    return {
      list: ['a', 'b', 'c'],
      idx: 0,
      current: ''
    }
  },
  methods: {
    iter () {
      this.list.map(function (v, k) {
        if (k === this.idx) {
          this.current = v

          console.log(this.current)
        }
      })
    }
  },
  components: {
    Hello
  }
}
</script>
```
这是刚创建的工程，我们定义了list，idx和current，在执行iter方法时，我们就给current赋值以idx为下标的值，当我们执行后会发现，浏览器报了一个错误


这么回事，我们不是定义了idx了吗？

其实是因为我们在map里的this是指向当前map的迭代对象，而非我们vue的实例，所以this里没有我们需要的idx。

解决方式有两种，其一是通过保存this
```js
let _this = this
```
其二是使用es6箭头函数
```js
methods: {
    iter () {
      this.list.map((v, k) => {
        if (k === this.idx) {
          this.current = v

          console.log(this.current)
        }
      })
    }
  },
```
现在再看我们的浏览器


已经可以达到我们预期的效果了！

# 方法传值

我们在input中的方法希望获取input的value，怎么获取呢？

可以通过$event这个对象，通过将$event传入方法
```html
<input type="text" value="value" @input="change($event)"/>
```
我们可以成功的拿到我们希望的值
```js
change (e) {
  console.log(e.target.value)
  this.value = e.target.value
}
```
# 迭代判断

比如我们有一个列表，我们希望能显示我们当前选中的那一个，如何实现呢？

基本思路是通过$index来判断是否是当前迭代对象，然后去增减class或者style来实现不同的样式
```js
<ul>

  <!-- 方法1 class-->
  <li v-for="item in list" :class="{'active': $index === activeId}">{item}</li>

  <!-- 方法2 style-->
  <li v-for="item in list" :style="{backgroundColor: $index === activeId ? 'red' : 'white'}">{item}</li>
</ul>
data () {
  return {
    list: ['a', 'b', 'c'],
    activeId: 0
  }
},
```
# camelCase vs. kebab-case

HTML 特性不区分大小写。名字形式为 camelCase 的 prop 用作特性时，需要转为 kebab-case（短横线隔开）(官网例子)
```js
Vue.component('child', {
  // camelCase in JavaScript
  props: ['myMessage'],
  template: '<span>{ myMessage }</span>'
})

<!-- kebab-case in HTML -->
<child my-message="hello!"></child>
```
# 另外props的写法

## 简写
```js
props: ['demo-first', 'demo-second']
```
## 带类型
```js
props: {
    'demo-first': Number,
    'demo-second': Number
}
```
## 带多种检查
```js
props: {
    'demo-first': {
        type: Number,
        default: 0
    }
}
```
所以， 当你获取不到props的值的时候，可以先仔细检查拼写是否正确。

## 传值与传字面量

在vue的组件中传递数据，如果是单纯传递字面量，如
```html
<hello result="success"></hello>
```
那么在hello中获取的props result的值就是“success”，如果希望进行值传递，需要在指令前加 ':' 冒号，这样，父层的success的值改变，hello的值也会跟着改变。

# 转场动画

在vue中有个很好用的指令，transition，通过它我们可以实现自定义的router切换中的动画

方法就是在
```html
<router-view transition="fade"></router-view>
```
加入自定义的class fade-transition , fade-leave , fade-enter即可。

# 数据驱动 vs dom

vue是基于数据驱动的，最好不要直接去修改dom（虽然官方给出了这样的方法）

# v-cloak

如果出现{}的短暂出现的情况，可以通过添加v-cloak来处理。

这个指令保持在元素上直到关联实例结束编译。和 CSS 规则如 [v-cloak] { display: none } 一起用时，这个指令可以隐藏未编译的 Mustache 标签直到实例准备完毕。