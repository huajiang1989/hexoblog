
---
title:  vue-router 让链接处于活跃状态
category:  vue
tags: vue
---
如何解决以下问题：

当用户点击Home链接或About链接后，链接没有显示为选中
当用户点击News或Message链接后，链接没有显示为选中
<!--more-->
设置activeClass
--
1.可以通过设定v-link指令的activeClass解决。
```html
<a class="list-group-item" v-link="{ path: '/home', activeClass: 'active'}">Home</a>
<a class="list-group-item" v-link="{ path: '/about', activeClass: 'active'}">About</a>
```
设定了v-link指令的activeClass属性后，默认的v-link-active被新的class取代。

2.为v-link指令设定activeClass是不起作用的，因为我们使用的是bootstrap的样式，需要设置a标签的父元素<li>才能让链接看起来处于选中状态，就像下面的代码所展现的：
```html
<ul class="nav nav-tabs">
    <li class="active">
        <a v-link="{ path: '/home/news'}">News</a>
    </li>
    <li>
        <a v-link="{ path: '/home/message'}">Messages</a>
    </li>
</ul>
```
如何实现这个效果呢？你可能会想到，为Home组件的data选项追加一个currentPath属性，然后使用以下方式绑定class。
```html
<ul class="nav nav-tabs">
    <li :class="currentPath == '/home/news' ? 'active': ''">
        <a v-link="{ path: '/home/news'}">News</a>
    </li>
    <li :class="currentPath == '/home/message' ? 'active': ''">
        <a v-link="{ path: '/home/message'}">Messages</a>
    </li>
</ul>
```
现在又出现了另一个问题，在什么情况下给currentPath赋值呢？

用户点击v-link的元素时，是路由的切换。
每个组件都有一个route选项，route选项有一系列钩子函数，在切换路由时会执行这些钩子函数。
其中一个钩子函数是data钩子函数，它用于加载和设置组件的数据。
```js
var Home = Vue.extend({
    template: '#home',
    data: function() {
        return {
            msg: 'Hello, vue router!',
            currentPath: ''
        }
    },
    route: {
        data: function(transition){
            transition.next({
                currentPath: transition.to.path
            })
        }
    }
})
```