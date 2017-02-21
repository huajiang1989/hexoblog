
---
title:  vue+vuex构建单页应用
category:  vue
tags: vue
---

# 基本
构建工具： webpack
语言： ES6
分号：行首分号规则（行尾不加分好， [ , ( , / , + , - 开头时在行首加分号）
配套设施： webpack 全家桶， vue 全家桶

<!--more-->
# 项目结构

基本目录结构
![](http://img2.tuicool.com/IzeyAvR.png)

api ：封装与后端接口交互的操作

common ：放置一些 reset.css 之类的

components ：组件

entry ：项目入口文件 index.js,index.css,index.html

filters ：过滤器。注：虽然 vue2.0 已经基本废弃（只保留了对文本的过滤）了 Vue.filter 的用法，此目录下的方法仍然可用于官方推荐用来替代过滤器的计算属性的计算中

mixins ：一些通用类的混入部分。比如全选、多选可抽出通用的 list-toggle

mock ：本地开发的 mock 数据

utils ：封装的工具，如对上传文件、日期处理等的封装

views ：单页应用的视图（视图也是组件，也可放到 components 中，但个人觉得放在这里比较一目了然）

vuex ：放置 store，actions，mutations，state

fis-conf.js ：用于测试环境联调时 fis 实时将前端资源推送到开发机上

如果有自定义指令，还可以加上 directives 目录（ vue 的几个可扩展的地方都可以单独做一个目录）。对于项目目录，也可以使用官方提供的另一个工具 vue-cli 来生成,它还会自动构建单元测试( unit )和端对端测试( e2e )的目录和简单示例。

# 基础组件
vue 除了双向绑定外的一个最大特点就是提供了强大的组件树系统，组件化也是 web 发展的趋势.

每一个 Vue 实例就是一个组件，构造一个组件的也很简单:
```js
var myComponent = Vue.extend({
    template: '',
    ...
})
// 全局注册组件，tag 为 my-component
Vue.component('my-component', MyComponent)
```
更推荐的做法是写成 *.vue 形式的单文件组件，搭配 vue-loader 使用（下图来自官方文档）。
![](http://img2.tuicool.com/J77veyQ.png)

更多关于组件的内容，见官方文档 组件 。

另外，在使用单文件组件时，样式会被打包到 js 中并在运行时会以 style 节点的形式插入到 head 里面。此时如果想将组件的样式打包到输出的 css 文件中，只需要在 webpack.config.js 的 module.exports 中加上以下配置即可：
```js
vue: {
    loaders: {
        js: 'babel-loader?presets[]=es2015&plugins[]=transform-runtime&comments=false',
        css:ExtractTextPlugin.extract(['css-loader'])
    }
}
```
刚开始一个项目时，如果在有自己特定的 UI 设计风格，可能需要单独封装一些 textinput,checkbox,radio 等基础组件；如果没有的话（如普通的后台管理系统），也可以使用 Google Material Design ，已经有对应的实现 material-design-lite 。并且 vue 社区中也已经有针对它的 vue 组件封装 vue-mdl 。

# 应用骨架
以“xx管理后台”为例，首先分为上（导航）下（主体内容）两部分，基本结构为：
![](http://img1.tuicool.com/J3EZBbV.png)

接下来在 views 里面心间 user.vue ，作为用户管理模块入口，如果每个模块还需要包含二级导航（通常是在页面左侧部分）， user.vue 可以像这样：
![](http://img2.tuicool.com/AZRzia2.png)

这两个文件中用到的 router-view ，都是 vue 官方路由插件 vue-router 提供的。

然后是配置单页应用的路由：
![](http://img2.tuicool.com/6fQVJv6.png)

在对应的视图组件中，通过route选项的钩子函数，来确定时图在出现和消失的过程中需要执行的行为。更多路由相关，见 官方文档 。

这样，一个基本的：上 -> ［左｜右］的单页应用骨架就有了。（其他类型的应用也可依此类推）

# 应用状态管理
应用组件化之后，就需要解决组件之间的通信问题。针对组件之间的通信问题， vue 提供了三种方式： props 属性传递，直接通过引用调用组件方法，自定义事件通信，通过 v-ref （在 vue2.0 已简化为 ref ）来建立子组件索引从而调用子组件方法。

porps ：基于属性传递， vue 提供了单次绑定、单向绑定和双向绑定。（虽然双向绑定在 vue2.0 中已经废弃）

通过引用：子组件可以用 this.$parent 访问它的父组件。根实例的后代可以用 this.$root 访问它。父组件有一个数组 this.$children ，包含它所有的子元素。

通过自定义事件通信：每个 Vue 实例都是一个事件触发器：

使用 $on() 监听事件

使用 $emit() 在它上面触发事件

使用 $dispatch() 派发事件，事件沿着父链冒泡（ vue2.0 已废弃）

使用 $broadcast() 广播事件，事件向下传导给所有的后代（ vue2.0 已废弃）

在 vue2.0 中，可以单独使用一个 Vue 实例来来担任 eventBus 的作用。

除了这几种方式，当应用比较复杂时，官方推荐使用另一个官方插件 vuex

类似于 react 的 redux ， vue 的 vuex 的 store 也包含一个全局的状态树 state ；修改 state 的行为 mutations （对应 redux 的 actions ）；执行修改的动作 actions （对应 redux 的 createAction ）。
![](http://img1.tuicool.com/qAzIJju.png)

以全局 alert 组件的状态为例：


- 创建 state
```js
export default {
    alert: {
        show: false,
        type: 'alert',
        message: '',
        onSure: null,
        onClose: null
    }
}
```
-  创建 mutations
```js
export default {
    SHOW_ALERT (state, data) {
        data.show = true
        state.alert = data
    },
    HIDE_ALERT (state) {
        state.alert.show = false
    }
}
```

- 创建 actions
```js
/*主页面涉及到的actions*/
let noop = () => {}
/*显示浮层alert*/
export const showAlert = ({dispatch}, message = '') => {
    if(!message) {
        return false
    }
    dispatch('SHOW_ALERT', {
        type: 'alert',
        message: message,
        onClose: noop
    })
}
/*显示浮层confirm*/
export const showConfirm = ({dispatch}, data = {}) => {
    if(!data.message) {
        return false
    }
    data.type = 'confirm'
    if(typeof data.onClose != 'function') {
        data.onClose = noop
    }
    if(typeof data.onConfirm != 'function') {
        data.onConfirm = noop
    }
    dispatch('SHOW_ALERT', data)
}
/*隐藏浮层*/
export const hideAlert = ({dispatch}) => dispatch('HIDE_ALERT')
```

- 构建 store
```js
import Vue from 'vue'
import Vuex from 'vuex'
import actions from './actions'
import mutations from './mutations'
import state from "./state"
Vue.use(Vuex)
const debug = process.env.NODE_ENV !== 'production'
Vue.config.debug = debug
export default new Vuex.Store({
    state,
    mutations,
    actions,
    strict: debug
})
```
然后在应用的根组件中，通过以下方式获取 vuex 的功能：
```js
/*引入vuex*/
import store from "../vuex/store"
let App = Vue.extend({
    store,
    components: {
        'admin-header': adminHeader,
        'alert': alert
    }
})
```
然后再在自组件中的vuex模块通过以下方式获取状态以及触发状态改变的动作：
![](http://img2.tuicool.com/6b2AziY.png)

# 应用的数据交互
## api层

记得之前看过民工叔叔（徐飞）的某篇文章里说的，数据层能够跟UI层分离，这样即使 UI 底层库更换了，也可以使用数据层。同理如果想要对api交互进行替换（如想把某些 ajax 库换成浏览器支持的 fetch api ），也可以直接在这一层进行更改。

## mock数据

在开发阶段，有时需要mock一些数据来测试应用。推荐一个对 restful api 友好的第三方工具 json-server 。


index.js
```js
var users = require('./database/users')
module.exports = function() {
    return {
        "users": users
    }
}
```
database/users.js
```js
module.exports = {
    "users": [
        {
            "user_id": "233",
            "user_name": "哈哈哈",
        },
        {
            "user_id": "233",
            "user_name": "哈哈哈",
        }
    ],
    "more": true,
    "result": "SUCCESS"
}
```
然后终端执行 json-server mock/index.js －port 9999 就开启了一个 restful 的服务了。（也可以把这句写到 npm script 中）

接下来还差一步，就是需要用到 webpack-dev-server 的 proxy 配置：
![](http://img2.tuicool.com/NrM3yqu.png)

这样，所有访问 /rest/* 的接口都会被代理到 json-server 的服务上.

# 应用测试
一个完整的应用应该还具备单元测试、端对端测试等。目前比较成熟的测试框架社区中也有不少，但由于还没油深入研究过，此处不展开。