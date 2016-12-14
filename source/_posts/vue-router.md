
---
title:  vue-router 快速入门
category:  vue
tags: vue
---
vue-router是Vue.js官方的路由插件，它和vue.js是深度集成的，适合用于构建单页面应用。vue的单页面应用是基于路由和组件的，路由用于设定访问路径，并将路径和组件映射起来。传统的页面应用，是用一些超链接来实现页面切换和跳转的。在vue-router单页面应用中，则是路径之间的切换，也就是组件的切换。
<!--more-->
# 第一个单页面应用

## 创建组件
首先引入vue.js和vue-router.js：
```js
<script src="js/vue.js"></script>
<script src="js/vue-router.js"></script>
```
然后创建两个组件构造器Home和About：
```js
var Home = Vue.extend({
    template: '<div><h1>Home</h1><p>{{msg}}</p></div>',
    data: function() {
        return {
            msg: 'Hello, vue router!'
        }
    }
})

var About = Vue.extend({
    template: '<div><h1>About</h1><p>This is the tutorial about vue-router.</p></div>'
})
```
## 创建Router
```js
var router = new VueRouter()
```
调用构造器VueRouter，创建一个路由器实例router。

## 映射路由
```js
router.map({
    '/home': { component: Home },
    '/about': { component: About }
})
```
调用router的map方法映射路由，每条路由以key-value的形式存在，key是路径，value是组件。
例如：'/home'是一条路由的key，它表示路径；{component: Home}则表示该条路由映射的组件。

## 使用v-link指令
```html
<div class="list-group">
    <a class="list-group-item" v-link="{ path: '/home'}">Home</a>
    <a class="list-group-item" v-link="{ path: '/about'}">About</a>
</div>
```
在a元素上使用v-link指令跳转到指定路径。

## 使用<router-view>标签
```html
<router-view></router-view>
```
在页面上使用<router-view></router-view>标签，它用于渲染匹配的组件。

## 启动路由
```js
var App = Vue.extend({})
router.start(App, '#app')
```
路由器的运行需要一个根组件，router.start(App, '#app') 表示router会创建一个App实例，并且挂载到#app元素。
注意：使用vue-router的应用，不需要显式地创建Vue实例，而是调用start方法将根组件挂载到某个元素。

## 执行过程
### JavaScript

创建组件：创建单页面应用需要渲染的组件
创建路由：创建VueRouter实例
映射路由：调用VueRouter实例的map方法
启动路由：调用VueRouter实例的start方法
### HTML

使用v-link指令
使用<router-view>标签
router.redirect
应用在首次运行时右侧是一片空白，应用通常都会有一个首页，例如：Home页。
使用router.redirect方法将根路径重定向到/home路径：
```js
router.redirect({
    '/': '/home'
})
```
router.redirect方法用于为路由器定义全局的重定向规则，全局的重定向会在匹配当前路径之前执行。

### 执行过程
当用户点击v-link指令元素时，我们可以大致猜想一下这中间发生了什么事情：
vue-router首先会去查找v-link指令的路由映射
然后根据路由映射找到匹配的组件
最后将组件渲染到<router-view>标签

# 嵌套路由
嵌套路由是个常见的需求，假设用户能够通过路径/home/news和/home/message访问一些内容，一个路径映射一个组件，访问这两个路径也会分别渲染两个组件。 

实现嵌套路由有两个要点：

在组件内部使用<router-view>标签
在路由器对象中给组件定义子路由
现在我们就动手实现这个需求。

组件模板：
```html
<template id="home">
    <div>
        <h1>Home</h1>
        <p>{{msg}}</p>
    </div>
    <div>
        <ul class="nav nav-tabs">
            <li>
                <a v-link="{ path: '/home/news'}">News</a>
            </li>
            <li>
                <a v-link="{ path: '/home/message'}">Messages</a>
            </li>
        </ul>
        <router-view></router-view>
    </div>
</template>

<template id="news">
    <ul>
        <li>News 01</li>
        <li>News 02</li>
        <li>News 03</li>
    </ul>
</template>
<template id="message">
    <ul>
        <li>Message 01</li>
        <li>Message 02</li>
        <li>Message 03</li>
    </ul>
</template>
```
组件构造器：
```js
var Home = Vue.extend({
    template: '#home',
    data: function() {
        return {
            msg: 'Hello, vue router!'
        }
    }
})

var News = Vue.extend({
    template: '#news'
})

var Message = Vue.extend({
    template: '#message'
})
```
路由映射：
```js
router.map({
    '/home': {
        component: Home,
        // 定义子路由
        subRoutes: {
            '/news': {
                component: News
            },
            '/message': {
                component: Message
            }
        }
    },
    '/about': {
        component: About
    }
})
```
在/home路由下定义了一个subRoutes选项，/news和/message是两条子路由，它们分别表示路径/home/news和/home/message，这两条路由分别映射组件News和Message。

注意：这里有一个概念要区分一下，/home/news和/home/message是/home路由的子路由，与之对应的News和Message组件并不是Home的子组件。

# v-link指令
用了这么久的v-link指令，是该介绍一下它了。

v-link 是一个用来让用户在 vue-router 应用的不同路径间跳转的指令。该指令接受一个 JavaScript 表达式，并会在用户点击元素时用该表达式的值去调用 router.go。

具体来讲，v-link有三种用法：
```js
<!-- 字面量路径 -->
<a v-link="'home'">Home</a>

<!-- 效果同上 -->
<a v-link="{ path: 'home' }">Home</a>

<!-- 具名路径 -->
<a v-link="{ name: 'detail', params: {id: '01'} }">Home</a>
```
v-link 会自动设置 a标签的 href 属性，你无需使用href来处理浏览器的调整，原因如下：

它在 HTML5 history 模式和 hash 模式下的工作方式相同，所以如果你决定改变模式，或者 IE9 浏览器退化为 hash 模式时，都不需要做任何改变。

在 HTML5 history 模式下，v-link 会监听点击事件，防止浏览器尝试重新加载页面。

在 HTML5 history 模式下使用 root 选项时，不需要在 v-link 的 URL 中包含 root 路径。

# 路由对象
在使用了 vue-router 的应用中，路由对象会被注入每个组件中，赋值为 this.$route ，并且当路由切换时，路由对象会被更新。

路由对象暴露了以下属性：

$route.path 
字符串，等于当前路由对象的路径，会被解析为绝对路径，如 "/home/news" 。
$route.params 
对象，包含路由中的动态片段和全匹配片段的键值对
$route.query 
对象，包含路由中查询参数的键值对。例如，对于 /home/news/detail/01?favorite=yes ，会得到$route.query.favorite == 'yes' 。
$route.router 
路由规则所属的路由器（以及其所属的组件）。
$route.matched 
数组，包含当前匹配的路径中所包含的所有片段所对应的配置参数对象。
$route.name 
当前路径的名字，如果没有使用具名路径，则名字为空。
在页面上添加以下代码，可以显示这些路由对象的属性：
```html
<div>
    <p>当前路径：{{$route.path}}</p>
    <p>当前参数：{{$route.params | json}}</p>
    <p>路由名称：{{$route.name}}</p>
    <p>路由查询参数：{{$route.query | json}}</p>
    <p>路由匹配项：{{$route.matched | json}}</p>
</div>
```
$route.path, $route.params, $route.name, $route.query这几个属性很容易理解，看示例就能知道它们代表的含义。

