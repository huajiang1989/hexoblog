
---
title:  vue-router 生命周期
category:  vue
tags: vue
---
路由的切换过程，本质上是执行一系列路由钩子函数，钩子函数总体上分为两大类：

全局的钩子函数
组件的钩子函数
<!--more-->
全局的钩子函数定义在全局的路由对象中，组件的钩子函数则定义在组件的route选项中。

# 全局钩子函数
全局钩子函数有2个：
```bash
beforeEach： #在路由切换开始时调用
afterEach：  #在每次路由切换成功进入激活阶段时被调用
```
# 组件的钩子函数
组件的钩子函数一共6个：
```bash
data：         #可以设置组件的data
activate：     #激活组件
deactivate：   #禁用组件
canActivate:   #组件是否可以被激活
canDeactivate：#组件是否可以被禁用
canReuse：     #组件是否可以被重用
```
# 切换对象
每个切换钩子函数都会接受一个 transition 对象作为参数。这个切换对象包含以下函数和方法：
```bash
transition.to
#表示将要切换到的路径的路由对象。
transition.from
#代表当前路径的路由对象。
transition.next()
#调用此函数处理切换过程的下一步。
transition.abort([reason])
#调用此函数来终止或者拒绝此次切换。
#transition.redirect(path)
取消当前切换并重定向到另一个路由。
```
# 钩子函数的执行顺序
全局钩子函数和组件钩子函数加起来一共8个，为了熟练vue router的使用，有必要了解这些钩子函数的执行顺序。

为了直观地了解这些钩子函数的执行顺序，在画面上追加一个Vue实例：
```js
var well = new Vue({
    el: '.well',
    data: {
        msg: '',
        color: '#ff0000'
    },
    methods: {
        setColor: function(){
            this.color = '#' + parseInt(Math.random()*256).toString(16)
                        + parseInt(Math.random()*256).toString(16)
                        + parseInt(Math.random()*256).toString(16)
        },
        setColoredMessage: function(msg){
            this.msg +=  '<p style="color: ' + this.color + '">' + msg + '</p>'
        },
        setTitle: function(title){
            this.msg =  '<h2 style="color: #333">' + title + '</h2>'
        }
    }
})
```
well实例的HTML：
```html
<div class="well">
    {{{ msg }}}
</div>
```
然后，添加一个RouteHelper函数，用于记录各个钩子函数的执行日志：
```js
function RouteHelper(name) {
    var route = {
        canReuse: function(transition) {
            well.setColoredMessage('执行组件' + name + '的钩子函数:canReuse')
            return true
        },
        canActivate: function(transition) {
            well.setColoredMessage('执行组件' + name + '的钩子函数:canActivate')
            transition.next()
        },
        activate: function(transition) {
            well.setColoredMessage('执行组件' + name + '的钩子函数:activate')
            transition.next()
        },
        canDeactivate: function(transition) {
            well.setColoredMessage('执行组件' + name + '的钩子函数:canDeactivate')
            transition.next()
        },
        deactivate: function(transition) {
            well.setColoredMessage('执行组件' + name + '的钩子函数:deactivate')
            transition.next()
        },
        data: function(transition) {
            well.setColoredMessage('执行组件' + name + '的钩子函数:data')
            transition.next()
        }
    }
    return route;
}
```
最后，将这些钩子函数应用于各个组件：
```js
var Home = Vue.extend({
    template: '#home',
    data: function() {
        return {
            msg: 'Hello, vue router!',
            path: ''
        }
    },
    route: RouteHelper('Home')
})

var News = Vue.extend({
    template: '#news',
    route: RouteHelper('News')
})

var Message = Vue.extend({
    template: '#message',
    route: RouteHelper('Message')
})

var About = Vue.extend({
    template: '#about',
    route: RouteHelper('About')
})
```