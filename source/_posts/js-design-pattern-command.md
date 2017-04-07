---
title: js设计模式-命令模式
category:  js-design-pattern
tags: js-design-pattern
---
 命令模式中的命令指的是一个执行某些特定事情的指令。

   命令模式使用的场景有：有时候需要向某些对象发送请求，但是并不知道请求的接收者是谁，也不知道请求的操作是什么，此时希望用一种松耦合的方式来设计程序代码;使得请求发送者和请求接受者消除彼此代码中的耦合关系。
<!--more-->
我们先来列举生活中的一个列子来说明下命令模式：比如我们经常会在天猫上购买东西，然后下订单，下单后我就想收到货，并且希望货物是真的，对于用户来讲它并关心下单后卖家怎么发货，当然卖家发货也有时间的，比如24小时内发货等，用户更不关心快递是给谁派送，当然有的人会关心是什么快递送货的; 对于用户来说，只要在规定的时间内发货，且一般能在相当的时间内收到货就可以，当然命令模式也有撤销命令和重做命令，比如我们下单后，我突然不想买了，我在发货之前可以取消订单，也可以重新下单（也就是重做命令）;比如我的衣服尺码拍错了，我取消该订单，重新拍一个大码的。

# 命令模式的列子

   记得我以前刚做前端的那会儿，也就是刚毕业进的第一家公司，进的是做外包项目的公司，该公司一般外包淘宝活动页面及腾讯的游戏页面，我们那会儿应该叫切页面的前端，负责做一些html和css的工作，所以那会儿做腾讯的游戏页面，经常会帮他们做静态页面，比如在页面放几个按钮，我们只是按照设计稿帮腾讯游戏哪方面的把样式弄好，比如说页面上的按钮等事情，比如说具体说明的按钮要怎么操作，点击按钮后会发生什么事情，我们并不知道，我们不知道他们的业务是什么，当然我们知道的肯定会有点击事件，具体要处理什么业务我们并不知道，这里我们就可以使用命令模式来处理了：点击按钮之后，必须向某些负责具体行为的对象发送请求，这些对象就是请求的接收者。但是目前我们并不知道接收者是什么对象，也不知道接受者究竟会做什么事情，这时候我们可以使用命令模式来消除发送者与接收者的代码耦合关系。

我们先使用传统的面向对象模式来设计代码：

假设html结构如下：
```html
<button id="button1">刷新菜单目录</button>
<button id="button2">增加子菜单</button>
<button id="button3">删除子菜单</button>
```
JS代码如下：

```js
var b1 = document.getElementById("button1"),
     b2 = document.getElementById("button2"),
     b3 = document.getElementById("button3");
```
 // 定义setCommand 函数，该函数负责往按钮上面安装命令。点击按钮后会执行command对象的execute()方法。
```js
 var setCommand = function(button,command){
    button.onclick = function(){
        command.execute();
    }
 };
 // 下面我们自己来定义各个对象来完成自己的业务操作
 var MenuBar = {
    refersh: function(){
        alert("刷新菜单目录");
    }
 };
 var SubMenu = {
    add: function(){
        alert("增加子菜单");
    },
    del: function(){
        alert("删除子菜单");
    }
 };
 // 下面是编写命令类
 var RefreshMenuBarCommand = function(receiver){
    this.receiver = receiver;
 };
 RefreshMenuBarCommand.prototype.execute = function(){
    this.receiver.refersh();
 }
 // 增加命令操作
 var AddSubMenuCommand = function(receiver) {
    this.receiver = receiver;
 };
 AddSubMenuCommand.prototype.execute = function() {
    this.receiver.add();
 }
 // 删除命令操作
 var DelSubMenuCommand = function(receiver) {
    this.receiver = receiver;
 };
 DelSubMenuCommand.prototype.execute = function(){
    this.receiver.del();
 }
 // 最后把命令接收者传入到command对象中，并且把command对象安装到button上面
 var refershBtn = new RefreshMenuBarCommand(MenuBar);
 var addBtn = new AddSubMenuCommand(SubMenu);
 var delBtn = new DelSubMenuCommand(SubMenu);

 setCommand(b1,refershBtn);
 setCommand(b2,addBtn);
 setCommand(b3,delBtn);
```
从上面的命令类代码我们可以看到，任何一个操作都有一个execute这个方法来执行操作;上面的代码是使用传统的面向对象编程来实现命令模式的，命令模式过程式的请求调用封装在command对象的execute方法里。我们有没有发现上面的编写代码有点繁琐呢，我们可以使用javascript中的回调函数来做这些事情的，在面向对象中，命令模式的接收者被当成command对象的属性保存起来，同时约定执行命令的操作调用command.execute方法，但是如果我们使用回调函数的话，那么接收者被封闭在回调函数产生的环境中，执行操作将会更加简单，仅仅执行回调函数即可，下面我们来看看代码如下：

代码如下：

```js
var setCommand = function(button,func) {
    button.onclick = function(){
        func();
    }
 };
 var MenuBar = {
    refersh: function(){
        alert("刷新菜单界面");
    }
 };
 var SubMenu = {
    add: function(){
        alert("增加菜单");
    }
 };
 // 刷新菜单
 var RefreshMenuBarCommand = function(receiver) {
    return function(){
        receiver.refersh();
    };
 };
 // 增加菜单
 var AddSubMenuCommand = function(receiver) {
    return function(){
        receiver.add();
    };
 };
 var refershMenuBarCommand = RefreshMenuBarCommand(MenuBar);
 // 增加菜单
 var addSubMenuCommand = AddSubMenuCommand(SubMenu);
 setCommand(b1,refershMenuBarCommand);

 setCommand(b2,addSubMenuCommand);
```
我们还可以如下使用javascript回调函数如下编码：

复制代码
```js
// 如下代码上的四个按钮 点击事件
var b1 = document.getElementById("button1"),
    b2 = document.getElementById("button2"),
    b3 = document.getElementById("button3"),
    b4 = document.getElementById("button4");
/*
 bindEnv函数负责往按钮上面安装点击命令。点击按钮后，会调用
 函数
 */
var bindEnv = function(button,func) {
    button.onclick = function(){
        func();
    }
};
// 现在我们来编写具体处理业务逻辑代码
var Todo1 = {
    test1: function(){
        alert("我是来做第一个测试的");
    }
};
// 实现业务中的增删改操作
var Menu = {
    add: function(){
        alert("我是来处理一些增加操作的");
    },
    del: function(){
        alert("我是来处理一些删除操作的");
    },
    update: function(){
        alert("我是来处理一些更新操作的");
    }
};
// 调用函数
bindEnv(b1,Todo1.test1);
// 增加按钮
bindEnv(b2,Menu.add);
// 删除按钮
bindEnv(b3,Menu.del);
// 更改按钮
bindEnv(b4,Menu.update);
```
# 理解宏命令：

   宏命令是一组命令的集合，通过执行宏命令的方式，可以一次执行一批命令。

其实类似把页面的所有函数方法放在一个数组里面去，然后遍历这个数组，依次

执行该方法的。

代码如下：

```js
var command1 = {
    execute: function(){
        console.log(1);
    }
};
var command2 = {
    execute: function(){
        console.log(2);
    }
};
var command3 = {
    execute: function(){
        console.log(3);
    }
};
// 定义宏命令，command.add方法把子命令添加进宏命令对象，
// 当调用宏命令对象的execute方法时，会迭代这一组命令对象，
// 并且依次执行他们的execute方法。
var command = function(){
    return {
        commandsList: [],
        add: function(command){
            this.commandsList.push(command);
        },
        execute: function(){
            for(var i = 0,commands = this.commandsList.length; i < commands; i+=1) {
                this.commandsList[i].execute();
            }
        }
    }
};
// 初始化宏命令
var c = command();
c.add(command1);
c.add(command2);
c.add(command3);
c.execute();  // 1,2,3
```