---
title: js设计模式-职责连模式
category:  js-design-pattern
tags: js-design-pattern
---
优点是：消除请求的发送者与接收者之间的耦合。

    职责连是由多个不同的对象组成的，发送者是发送请求的对象，而接收者则是链中那些接收这种请求并且对其进行处理或传递的对象。请求本身有时候也可以是一个对象，它封装了和操作有关的所有数据，基本实现流程如下：

1. 发送者知道链中的第一个接收者，它向这个接收者发送该请求。

2. 每一个接收者都对请求进行分析，然后要么处理它，要么它往下传递。

3. 每一个接收者知道其他的对象只有一个，即它在链中的下家(successor)。

4. 如果没有任何接收者处理请求，那么请求会从链中离开。

<!--more-->

   我们可以理解职责链模式是处理请求组成的一条链，请求在这些对象之间依次传递，直到遇到一个可以处理它的对象，我们把这些对象称为链中的节点。比如对象A给对象B发请求，如果B对象不处理，它就会把请求交给C，如果C对象不处理的话，它就会把请求交给D，依次类推，直到有一个对象能处理该请求为止，当然没有任何对象处理该请求的话，那么请求就会从链中离开。

   比如常见的一些外包公司接到一个项目，那么接到项目有可能是公司的负责项目的人或者经理级别的人，经理接到项目后自己不开发，直接把它交到项目经理来开发，项目经理自己肯定不乐意自己动手开发哦，它就把项目交给下面的码农来做，所以码农来处理它，如果码农也不处理的话，那么这个项目可能会直接挂掉了，但是最后完成后，外包公司它并不知道这些项目中的那一部分具体有哪些人开发的，它并不知道，也并不关心的，它关心的是这个项目已交给外包公司已经开发完成了且没有任何bug就可以了；所以职责链模式的优点就在这里：

消除请求的发送者(需要外包项目的公司)与接收者(外包公司)之间的耦合。

下面列举个列子来说明职责链的好处：

天猫每年双11都会做抽奖活动的，比如阿里巴巴想提高大家使用支付宝来支付的话，每一位用户充值500元到支付宝的话，那么可以100%中奖100元红包，

充值200元到支付宝的话，那么可以100%中奖20元的红包，当然如果不充值的话，也可以抽奖，但是概率非常低，基本上是抽不到的，当然也有可能抽到的。

我们下面可以分析下代码中的几个字段值需要来判断：

1. orderType(充值类型)，如果值为1的话，说明是充值500元的用户，如果为2的话，说明是充值200元的用户，如果是3的话，说明是没有充值的用户。

2. isPay(是否已经成功充值了): 如果该值为true的话，说明已经成功充值了，否则的话 说明没有充值成功；就当作普通用户来购买。

3. count(表示数量)；普通用户抽奖，如果数量有的话，就可以拿到优惠卷，否则的话，不能拿到优惠卷。

```js
// 我们一般写代码如下处理操作
var order =  function(orderType,isPay,count) {
    if(orderType == 1) {  // 用户充值500元到支付宝去
        if(isPay == true) { // 如果充值成功的话，100%中奖
            console.log("亲爱的用户，您中奖了100元红包了");
        }else {
            // 充值失败，就当作普通用户来处理中奖信息
            if(count > 0) {
                console.log("亲爱的用户，您已抽到10元优惠卷");
            }else {
                console.log("亲爱的用户，请再接再厉哦");
            }
        }
    }else if(orderType == 2) {  // 用户充值200元到支付宝去
        if(isPay == true) {     // 如果充值成功的话，100%中奖
            console.log("亲爱的用户，您中奖了20元红包了");
        }else {
            // 充值失败，就当作普通用户来处理中奖信息
            if(count > 0) {
                console.log("亲爱的用户，您已抽到10元优惠卷");
            }else {
                console.log("亲爱的用户，请再接再厉哦");
            }
        }
    }else if(orderType == 3) {
        // 普通用户来处理中奖信息
        if(count > 0) {
            console.log("亲爱的用户，您已抽到10元优惠卷");
        }else {
            console.log("亲爱的用户，请再接再厉哦");
        }
    }
};
```
上面的代码虽然可以实现需求，但是代码不容易扩展且难以阅读，假如以后我想一两个条件，我想充值300元成功的话，可以中奖150元红包，那么这时候又要改动里面的代码,这样业务逻辑与代码耦合性相对比较高，一不小心就改错了代码；这时候我们试着使用职责链模式来依次传递对象来实现；

如下代码：

```js
function order500(orderType,isPay,count){
    if(orderType == 1 && isPay == true)    {
        console.log("亲爱的用户，您中奖了100元红包了");
    }else {
        // 自己不处理，传递给下一个对象order200去处理
        order200(orderType,isPay,count);
    }
};
function order200(orderType,isPay,count) {
    if(orderType == 2 && isPay == true) {
        console.log("亲爱的用户，您中奖了20元红包了");
    }else {
        // 自己不处理，传递给下一个对象普通用户去处理
        orderNormal(orderType,isPay,count);
    }
};
function orderNormal(orderType,isPay,count){
    // 普通用户来处理中奖信息
    if(count > 0) {
        console.log("亲爱的用户，您已抽到10元优惠卷");
    }else {
        console.log("亲爱的用户，请再接再厉哦");
    }
}
```
如上代码我们分别使用了三个函数order500，order200，orderNormal来分别处理自己的业务逻辑，如果目前的自己函数不能处理的事情，我们传递给下面的函数去处理，依次类推，直到有一个函数能处理他，否则的话，该职责链模式直接从链中离开，告诉不能处理，抛出错误提示，上面的代码虽然可以当作职责链模式，但是我们看上面的代码可以看到order500函数内依赖了order200这样的函数，这样就必须有这个函数，也违反了面向对象中的 开放-封闭原则。下面我们继续来理解编写 灵活可拆分的职责链节点。

```js
function order500(orderType,isPay,count){
    if(orderType == 1 && isPay == true)    {
        console.log("亲爱的用户，您中奖了100元红包了");
    }else {
        //我不知道下一个节点是谁,反正把请求往后面传递
        return "nextSuccessor";
    }
};
function order200(orderType,isPay,count) {
    if(orderType == 2 && isPay == true) {
        console.log("亲爱的用户，您中奖了20元红包了");
    }else {
        //我不知道下一个节点是谁,反正把请求往后面传递
        return "nextSuccessor";
    }
};
function orderNormal(orderType,isPay,count){
    // 普通用户来处理中奖信息
    if(count > 0) {
        console.log("亲爱的用户，您已抽到10元优惠卷");
    }else {
        console.log("亲爱的用户，请再接再厉哦");
    }
}
// 下面需要编写职责链模式的封装构造函数方法
var Chain = function(fn){
    this.fn = fn;
    this.successor = null;
};
Chain.prototype.setNextSuccessor = function(successor){
    return this.successor = successor;
}
// 把请求往下传递
Chain.prototype.passRequest = function(){
    var ret = this.fn.apply(this,arguments);
    if(ret === 'nextSuccessor') {
        return this.successor && this.successor.passRequest.apply(this.successor,arguments);
    }
    return ret;
}
//现在我们把3个函数分别包装成职责链节点：
var chainOrder500 = new Chain(order500);
var chainOrder200 = new Chain(order200);
var chainOrderNormal = new Chain(orderNormal);

// 然后指定节点在职责链中的顺序
chainOrder500.setNextSuccessor(chainOrder200);
chainOrder200.setNextSuccessor(chainOrderNormal);

//最后把请求传递给第一个节点：
chainOrder500.passRequest(1,true,500);  // 亲爱的用户，您中奖了100元红包了
chainOrder500.passRequest(2,true,500);  // 亲爱的用户，您中奖了20元红包了
chainOrder500.passRequest(3,true,500);  // 亲爱的用户，您已抽到10元优惠卷
chainOrder500.passRequest(1,false,0);   // 亲爱的用户，请再接再厉哦
```
如上代码;分别编写order500，order200，orderNormal三个函数，在函数内分别处理自己的业务逻辑，如果自己的函数不能处理的话，就返回字符串nextSuccessor 往后面传递，然后封装Chain这个构造函数，传递一个fn这个对象实列进来，且有自己的一个属性successor，原型上有2个方法 setNextSuccessor 和 passRequest;setNextSuccessor 这个方法是指定节点在职责链中的顺序的，把相对应的方法保存到this.successor这个属性上，chainOrder500.setNextSuccessor(chainOrder200);chainOrder200.setNextSuccessor(chainOrderNormal);指定链中的顺序，因此this.successor引用了order200这个方法和orderNormal这个方法，因此第一次chainOrder500.passRequest(1,true,500)调用的话，调用order500这个方法，直接输出，第二次调用chainOrder500.passRequest(2,true,500);这个方法从链中首节点order500开始不符合，就返回successor字符串，然后this.successor && this.successor.passRequest.apply(this.successor,arguments);就执行这句代码；上面我们说过this.successor这个属性引用了2个方法 分别为order200和orderNormal，因此调用order200该方法，所以就返回了值，依次类推都是这个原理。那如果以后我们想充值300元的红包的话，我们可以编写order300这个函数，然后实列一下链chain包装起来，指定一下职责链中的顺序即可，里面的业务逻辑不需要做任何处理;

# 理解异步的职责链

上面的只是同步职责链，我们让每个节点函数同步返回一个特定的值”nextSuccessor”，来表示是否把请求传递给下一个节点，在我们开发中会经常碰到ajax异步请求，请求成功后，需要做某某事情，那么这时候如果我们再套用上面的同步请求的话，就不生效了，下面我们来理解下使用异步的职责链来解决这个问题;我们给Chain类再增加一个原型方法Chain.prototype.next，表示手动传递请求给职责链中的一下个节点。

如下代码：

```js
function Fn1() {
    console.log(1);
    return "nextSuccessor";
}
function Fn2() {
    console.log(2);
    var self = this;
    setTimeout(function(){
        self.next();
    },1000);
}
function Fn3() {
    console.log(3);
}
// 下面需要编写职责链模式的封装构造函数方法
var Chain = function(fn){
    this.fn = fn;
    this.successor = null;
};
Chain.prototype.setNextSuccessor = function(successor){
    return this.successor = successor;
}
// 把请求往下传递
Chain.prototype.passRequest = function(){
    var ret = this.fn.apply(this,arguments);
    if(ret === 'nextSuccessor') {
        return this.successor && this.successor.passRequest.apply(this.successor,arguments);
    }
    return ret;
}
Chain.prototype.next = function(){
    return this.successor && this.successor.passRequest.apply(this.successor,arguments);
}
//现在我们把3个函数分别包装成职责链节点：
var chainFn1 = new Chain(Fn1);
var chainFn2 = new Chain(Fn2);
var chainFn3 = new Chain(Fn3);

// 然后指定节点在职责链中的顺序
chainFn1.setNextSuccessor(chainFn2);
chainFn2.setNextSuccessor(chainFn3);

chainFn1.passRequest();  // 打印出1，2 过1秒后 会打印出3
```
调用函数 chainFn1.passRequest();后，会先执行发送者Fn1这个函数 打印出1，然后返回字符串 nextSuccessor;

 接着就执行return this.successor && this.successor.passRequest.apply(this.successor,arguments);这个函数到Fn2，打印2，接着里面有一个setTimeout定时器异步函数，需要把请求给职责链中的下一个节点，因此过一秒后会打印出3;

职责链模式的优点是：

 1. 解耦了请求发送者和N个接收者之间的复杂关系，不需要知道链中那个节点能处理你的请求，所以你

    只需要把请求传递到第一个节点即可。

 2. 链中的节点对象可以灵活地拆分重组，增加或删除一个节点，或者改变节点的位置都是很简单的事情。

 3. 我们还可以手动指定节点的起始位置，并不是说非得要从其实节点开始传递的.

 缺点：职责链模式中多了一点节点对象，可能在某一次请求过程中，大部分节点没有起到实质性作用，他们的作用只是让

 请求传递下去，从性能方面考虑，避免过长的职责链提高性能。