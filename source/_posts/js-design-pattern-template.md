---
title: js设计模式-模板模式
category:  js-design-pattern
tags: js-design-pattern
---
模板方法模式由二部分组成，第一部分是抽象父类，第二部分是具体实现的子类，一般的情况下是抽象父类封装了子类的算法框架，包括实现一些公共方法及封装子类中所有方法的执行顺序，子类可以继承这个父类，并且可以在子类中重写父类的方法，从而实现自己的业务逻辑。
<!--more-->
比如说我们要实现一个JS功能，比如表单验证等js，那么如果我们没有使用上一章讲的使用javascript中的策略模式来解决表单验证封装代码，而是自己写的临时表单验证功能，肯定是没有进行任何封装的，那么这个时候我们是针对两个值是否相等给用户弹出一个提示，如果再另外一个页面也有一个表单验证，他们判断的方式及业务逻辑基本相同的，只是比较的参数不同而已，我们是不是又要考虑写一个表单验证代码呢？那么现在我们可以考虑使用模板方法模式来解决这个问题；公用的方法提取出来，不同的方法由具体的子类是实现。这样设计代码也可扩展性更强，代码更优等优点~

我们不急着写代码，我们可以先来看一个列子，比如最近经常在qq群里面有很多前端招聘的信息，自己也接到很多公司或者猎头问我是否需要找工作等电话，当然我现在是没有打算找工作的，因为现在有更多的业余时间可以处理自己的事情，所以也觉得蛮不错的~ 我们先来看看招聘中面试这个流程；面试流程对于很多大型公司，比如BAT，面试过程其实很类似；因此我们可以总结面试过程中如下：

1. 笔试：(不同的公司有不同的笔试题目)。

2. 技术面试(一般情况下分为二轮)：第一轮面试你的有可能是你未来直接主管或者未来同事问你前端的一些专业方面的技能及以前做过的项目，在项目中遇到哪些问题及当时是如何解决问题的，还有根据你的简历上的基本信息来交流的，比如说你简历说精通JS，那么人家肯定得问哦~ 第二轮面试一般都是公司的牛人或者架构师来问的，比如问你计算机基本原理，或者问一些数据结构与算法等信息；第二轮面试可能会更深入的去了解你这个人的技术。

3. HR和总监或者总经理面试；那么这一轮的话，HR可能会问下你一些个人基本信息等情况，及问下你今后有什么打算的个人规划什么的，总监或者总经理可能会问下你对他们的网站及产品有了解过没有？及现在他们的产品有什么问题，有没有更好的建议或者如何改善的地方等信息；

4. 最后就是HR和你谈薪资及一般几个工作日可以得到通知，拿到offer(当然不符合的肯定是没有通知的哦)；及自己有没有需要了解公司的情况等等信息；

一般的面试过程都是如上四点下来的，对于不同的公司都差不多的流程的，当然有些公司可能没有上面的详细流程的，我这边这边讲一般的情况下，好了，这边就不扯了，这边也不是讲如何面试的哦，这边只是通过这个列子让我们更加的理解javascript中模板方法模式；所以我们现在回到正题上来；

我们先来分析下上面的流程；我们可以总结如下：

首先我们看一下百度的面试；因此我们可以先定义一个构造函数。

var BaiDuInterview = function(){};

那么下面就有百度面试的流程哦~

1. 笔试

那么我们可以封装一个笔试的方法，代码如下：
```js
// baidu 笔试

BaiDuInterview.prototype.writtenTest = function(){

    console.log("我终于看到百度的笔试题了~");

};
```
2. 技术面试：
```js
// 技术面试

BaiDuInterview.prototype.technicalInterview = function(){

    console.log("我是百度的技术负责人");

};
```
 3.  HR和总监或者总经理面试，我们可以称之为leader面试；代码如下：
```js
 // 领导面试

BaiDuInterview.prototype.leader = function(){

    console.log("百度leader来面试了");

};
```
4. 和HR谈期望的薪资待遇及HR会告诉你什么时候会有通知，因此我们这边可以称之为这个方法为 是否拿到offer(当然不符合要求肯定是没有通知的哦)；
```js
// 等通知

BaiDuInterview.prototype.waitNotice = function(){

    console.log("百度的人力资源太不给力了，到现在都不给我通知");

};
```
如上看到代码的基本结构，但是我们还需要一个初始化方法；代码如下：
```js
// 代码初始化

BaiDuInterview.prototype.init = function(){

    this.writtenTest();

    this.technicalInterview();

    this.leader();

    this.waitNotice();

};

var baiDuInterview = new BaiDuInterview();

baiDuInterview.init();
```
综合所述：所有的代码如下：
```js
var BaiDuInterview = function(){};



// baidu 笔试

BaiDuInterview.prototype.writtenTest = function(){

    console.log("我终于看到百度的题目笔试题了~");

};

// 技术面试

BaiDuInterview.prototype.technicalInterview = function(){

    console.log("我是百度的技术负责人");

};

// 领导面试

BaiDuInterview.prototype.leader = function(){

    console.log("百度leader来面试了");

};

// 等通知

BaiDuInterview.prototype.waitNotice = function(){

    console.log("百度的人力资源太不给力了，到现在都不给我通知");

};

// 代码初始化

BaiDuInterview.prototype.init = function(){

    this.writtenTest();

    this.technicalInterview();

    this.leader();

    this.waitNotice();

};

var baiDuInterview = new BaiDuInterview();

baiDuInterview.init();
```

# 业务模型
上面我们可以看到百度面试的基本流程如上面的代码，那么阿里和腾讯的也和上面的代码类似(这里就不一一贴一样的代码哦)，因此我们可以把公用代码提取出来；我们首先定义一个类，叫面试Interview

那么代码改成如下：
```js
var Interview = function(){};
```
1. 笔试：

我不管你是百度的笔试还是阿里或者腾讯的笔试题，我这边统称为笔试(WrittenTest)，那么你们公司有不同的笔试题，都交给子类去具体实现，父类方法不管具体如何实现，笔试题具体是什么样的 我都不管。代码变为如下：
```js
// 笔试

Interview.prototype.writtenTest = function(){

    console.log("我终于看到笔试题了~");

};
```
2. 技术面试，技术面试原理也一样，这里就不多说，直接贴代码：
```js
// 技术面试

Interview.prototype.technicalInterview = function(){

    console.log("我是技术负责人负责技术面试");

};
```
3. 领导面试
```js
// 领导面试

Interview.prototype.leader = function(){

    console.log("leader来面试了");

};
```
4. 等通知
```js
// 等通知

Interview.prototype.waitNotice = function(){

    console.log("人力资源太不给力了，到现在都不给我通知");

};
```
代码初始化方法如下：
```js
// 代码初始化

Interview.prototype.init = function(){

    this.writtenTest();

    this.technicalInterview();

    this.leader();

    this.waitNotice();

};
```
# 创建子类

现在我们来创建一个百度的子类来继承上面的父类；代码如下：
```js
var BaiDuInterview = function(){};

BaiDuInterview.prototype = new Interview();
```
现在我们可以在子类BaiDuInterview 重写父类Interview中的方法；代码如下：
```js
// 子类重写方法 实现自己的业务逻辑

BaiDuInterview.prototype.writtenTest = function(){

    console.log("我终于看到百度的笔试题了");

}

BaiDuInterview.prototype.technicalInterview = function(){

    console.log("我是百度的技术负责人，想面试找我");

}

BaiDuInterview.prototype.leader = function(){

    console.log("我是百度的leader，不想加班的或者业绩提不上去的给我滚蛋");

}

BaiDuInterview.prototype.waitNotice = function(){

    console.log("百度的人力资源太不给力了，我等的花儿都谢了！！");

}

var baiDuInterview = new BaiDuInterview();

baiDuInterview.init();
```
如上看到，我们直接调用子类baiDuInterview.init()方法，由于我们子类baiDuInterview没有init方法，但是它继承了父类，所以会到父类中查找对应的init方法；所以会迎着原型链到父类中查找；对于其他子类，比如阿里类代码也是一样的，这里就不多介绍了，对于父类这个方法 Interview.prototype.init() 是模板方法，因为他封装了子类中算法框架，它作为一个算法的模板，指导子类以什么样的顺序去执行代码。

# Javascript中的模板模式使用场景

虽然在java中也有子类实现父类的接口，但是我认为javascript中可以和java中不同的，java中可能父类就是一个空的类，子类去实现这个父类的接口，在javascript中我认为完全把公用的代码写在父函数内，如果将来业务逻辑需要更改的话，或者说添加新的业务逻辑，我们完全可以使用子类去重写这个父类，这样的话代码可扩展性强，更容易维护。由于本人不是专业java的，所以描述java中的知识点有误的话，请理解~~