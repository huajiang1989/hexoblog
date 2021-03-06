---
title: js设计模式-代理模式
category:  js-design-pattern
tags: js-design-pattern
---
代理是一个对象，它可以用来控制对本体对象的访问，它与本体对象实现了同样的接口，代理对象会把所有的调用方法传递给本体对象的；代理模式最基本的形式是对访问进行控制，而本体对象则负责执行所分派的那个对象的函数或者类，简单的来讲本地对象注重的去执行页面上的代码，代理则控制本地对象何时被实例化，何时被使用；我们在上面的单体模式中使用过一些代理模式，就是使用代理模式实现单体模式的实例化，其他的事情就交给本体对象去处理；
<!--more-->
代理的优点：

代理对象可以代替本体被实例化，并使其可以被远程访问；
它还可以把本体实例化推迟到真正需要的时候；对于实例化比较费时的本体对象，或者因为尺寸比较大以至于不用时不适于保存在内存中的本体，我们可以推迟实例化该对象；
我们先来理解代理对象代替本体对象被实例化的列子；比如现在京东ceo想送给奶茶妹一个礼物，但是呢假如该ceo不好意思送，或者由于工作忙没有时间送，那么这个时候他就想委托他的经纪人去做这件事，于是我们可以使用代理模式来编写如下代码：

```js
// 先申明一个奶茶妹对象
var TeaAndMilkGirl = function(name) {
    this.name = name;
};
// 这是京东ceo先生
var Ceo = function(girl) {
    this.girl = girl;
    // 送结婚礼物 给奶茶妹
    this.sendMarriageRing = function(ring) {
        console.log("Hi " + this.girl.name + ", ceo送你一个礼物：" + ring);
    }
};
// 京东ceo的经纪人是代理，来代替送
var ProxyObj = function(girl){
    this.girl = girl;
    // 经纪人代理送礼物给奶茶妹
    this.sendGift = function(gift) {
        // 代理模式负责本体对象实例化
        (new Ceo(this.girl)).sendMarriageRing(gift);
    }
};
// 初始化
var proxy = new ProxyObj(new TeaAndMilkGirl("奶茶妹"));
proxy.sendGift("结婚戒"); // Hi 奶茶妹, ceo送你一个礼物：结婚戒
```

代码如上的基本结构，TeaAndMilkGirl 是一个被送的对象(这里是奶茶妹)；Ceo 是送礼物的对象，他保存了奶茶妹这个属性，及有一个自己的特权方法sendMarriageRing 就是送礼物给奶茶妹这么一个方法；然后呢他是想通过他的经纪人去把这件事完成，于是需要创建一个经济人的代理模式，名字叫ProxyObj ；他的主要做的事情是，把ceo交给他的礼物送给ceo的情人，因此该对象同样需要保存ceo情人的对象作为自己的属性，同时也需要一个特权方法sendGift ，该方法是送礼物，因此在该方法内可以实例化本体对象，这里的本体对象是ceo送花这件事情，因此需要实例化该本体对象后及调用本体对象的方法(sendMarriageRing).

最后我们初始化是需要代理对象ProxyObj；调用ProxyObj 对象的送花这个方法(sendGift)即可；

对于我们提到的优点，第二点的话，我们下面可以来理解下虚拟代理，虚拟代理用于控制对那种创建开销很大的本体访问，它会把本体的实例化推迟到有方法被调用的时候；比如说现在有一个对象的实例化很慢的话，不能在网页加载的时候立即完成，我们可以为其创建一个虚拟代理，让他把该对象的实例推迟到需要的时候。

# 理解使用虚拟代理实现图片的预加载

在网页开发中，图片的预加载是一种比较常用的技术，如果直接给img标签节点设置src属性的话，如果图片比较大的话，或者网速相对比较慢的话，那么在图片未加载完之前，图片会有一段时间是空白的场景，这样对于用户体验来讲并不好，那么这个时候我们可以在图片未加载完之前我们可以使用一个loading加载图片来作为一个占位符，来提示用户该图片正在加载，等图片加载完后我们可以对该图片直接进行赋值即可；下面我们先不用代理模式来实现图片的预加载的情况下代码如下：

## 第一种方案：不使用代理的预加载图片函数如下

```js
// 不使用代理的预加载图片函数如下
var myImage = (function(){
    var imgNode = document.createElement("img");
    document.body.appendChild(imgNode);
    var img = new Image();
    img.onload = function(){
        imgNode.src = this.src;
    };
    return {
        setSrc: function(src) {
            imgNode.src = "http://img.lanrentuku.com/img/allimg/1212/5-121204193Q9-50.gif";
            img.src = src;
        }
    }
})();
// 调用方式
myImage.setSrc("https://img.alicdn.com/tps/i4/TB1b_neLXXXXXcoXFXXc8PZ9XXX-130-200.png");
```
如上代码是不使用代理模式来实现的代码；

## 第二种方案：使用代理模式来编写预加载图片的代码如下：

```js
var myImage = (function(){
    var imgNode = document.createElement("img");
    document.body.appendChild(imgNode);
    return {
        setSrc: function(src) {
            imgNode.src = src;
        }
    }
})();
// 代理模式
var ProxyImage = (function(){
    var img = new Image();
    img.onload = function(){
        myImage.setSrc(this.src);
    };
    return {
        setSrc: function(src) {
                         myImage.setSrc("http://img.lanrentuku.com/img/allimg/1212/5-121204193Q9-50.gif");
        img.src = src;
        }
    }
})();
// 调用方式
ProxyImage.setSrc("https://img.alicdn.com/tps/i4/TB1b_neLXXXXXcoXFXXc8PZ9XXX-130-200.png");
```
第一种方案是使用一般的编码方式实现图片的预加载技术，首先创建imgNode元素，然后调用myImage.setSrc该方法的时候，先给图片一个预加载图片，当图片加载完的时候，再给img元素赋值，第二种方案是使用代理模式来实现的，myImage 函数只负责创建img元素，代理函数ProxyImage 负责给图片设置loading图片，当图片真正加载完后的话，调用myImage中的myImage.setSrc方法设置图片的路径；他们之间的优缺点如下：

第一种方案一般的方法代码的耦合性太高，一个函数内负责做了几件事情，比如创建img元素，和实现给未加载图片完成之前设置loading加载状态等多项事情，未满足面向对象设计原则中单一职责原则；并且当某个时候不需要代理的时候，需要从myImage 函数内把代码删掉，这样代码耦合性太高。
第二种方案使用代理模式，其中myImage 函数只负责做一件事，创建img元素加入到页面中，其中的加载loading图片交给代理函数ProxyImage 去做，当图片加载成功后，代理函数ProxyImage 会通知及执行myImage 函数的方法，同时当以后不需要代理对象的话，我们直接可以调用本体对象的方法即可；
从上面代理模式我们可以看到，代理模式和本体对象中有相同的方法setSrc,这样设置的话有如下2个优点：

用户可以放心地请求代理，他们只关心是否能得到想要的结果。假如我门不需要代理对象的话，直接可以换成本体对象调用该方法即可。
在任何使用本体对象的地方都可以替换成使用代理。
当然如果代理对象和本体对象都返回一个匿名函数的话，那么也可以认为他们也具有一直的接口；比如如下代码：

```js
var myImage = (function(){
    var imgNode = document.createElement("img");
    document.body.appendChild(imgNode);
    return function(src){
        imgNode.src = src;
    }
})();
// 代理模式
var ProxyImage = (function(){
    var img = new Image();
    img.onload = function(){
        myImage(this.src);
    };
    return function(src) {
                myImage("http://img.lanrentuku.com/img/allimg/1212/5-121204193Q9-50.gif");
        img.src = src;
    }
})();
// 调用方式
ProxyImage("https://img.alicdn.com/tps/i4/TB1b_neLXXXXXcoXFXXc8PZ9XXX-130-200.png");
```
虚拟代理合并http请求的理解：

   比如在做后端系统中，有表格数据，每一条数据前面有复选框按钮，当点击复选框按钮时候，需要获取该id后需要传递给给服务器发送ajax请求，服务器端需要记录这条数据，去请求，如果我们每当点击一下向服务器发送一个http请求的话，对于服务器来说压力比较大，网络请求比较频繁，但是如果现在该系统的实时数据不是很高的话，我们可以通过一个代理函数收集一段时间内(比如说2-3秒)的所有id，一次性发ajax请求给服务器，相对来说网络请求降低了, 服务器压力减少了;

```html
// 首先html结构如下：
<p>
    <label>选择框</label>
    <input type="checkbox" class="j-input" data-id="1"/>
</p>
<p>
    <label>选择框</label>
    <input type="checkbox" class="j-input" data-id = "2"/>
</p>
<p>
    <label>选择框</label>
    <input type="checkbox" class="j-input" data-id="3"/>
</p>
<p>
    <label>选择框</label>
    <input type="checkbox" class="j-input" data-id = "4"/>
</p>
```
一般的情况下 JS如下编写

```html
<script>
    var checkboxs = document.getElementsByClassName("j-input");
    for(var i = 0,ilen = checkboxs.length; i < ilen; i+=1) {
        (function(i){
            checkboxs[i].onclick = function(){
                if(this.checked) {
                    var id = this.getAttribute("data-id");
                    // 如下是ajax请求
                }
            }
        })(i);
    }
</script>
```
下面我们通过虚拟代理的方式，延迟2秒，在2秒后获取所有被选中的复选框的按钮id，一次性给服务器发请求。

  通过点击页面的复选框，选中的时候增加一个属性isflag，没有选中的时候删除该属性isflag，然后延迟个2秒，在2秒后重新判断页面上所有复选框中有isflag的属性上的id，存入数组，然后代理函数调用本体函数的方法，把延迟2秒后的所有id一次性发给本体方法，本体方法可以获取所有的id，可以向服务器端发送ajax请求，这样的话，服务器的请求压力相对来说减少了。

代码如下：

```js
// 本体函数
var mainFunc = function(ids) {
    console.log(ids); // 即可打印被选中的所有的id
    // 再把所有的id一次性发ajax请求给服务器端
};
// 代理函数 通过代理函数获取所有的id 传给本体函数去执行
var proxyFunc = (function(){
    var cache = [],  // 保存一段时间内的id
        timer = null; // 定时器
    return function(checkboxs) {
        // 判断如果定时器有的话，不进行覆盖操作
        if(timer) {
            return;
        }
        timer = setTimeout(function(){
            // 在2秒内获取所有被选中的id，通过属性isflag判断是否被选中
            for(var i = 0,ilen = checkboxs.length; i < ilen; i++) {
                if(checkboxs[i].hasAttribute("isflag")) {
                    var id = checkboxs[i].getAttribute("data-id");
                    cache[cache.length] = id;
                }
            }
            mainFunc(cache.join(',')); // 2秒后需要给本体函数传递所有的id
            // 清空定时器
            clearTimeout(timer);
            timer = null;
            cache = [];
        },2000);
    }
})();
var checkboxs = document.getElementsByClassName("j-input");
for(var i = 0,ilen = checkboxs.length; i < ilen; i+=1) {
    (function(i){
        checkboxs[i].onclick = function(){
            if(this.checked) {
                // 给当前增加一个属性
                this.setAttribute("isflag",1);
            }else {
                this.removeAttribute('isflag');
            }
            // 调用代理函数
            proxyFunc(checkboxs);
        }
    })(i);
}
```
# 理解缓存代理：

   缓存代理的含义就是对第一次运行时候进行缓存，当再一次运行相同的时候，直接从缓存里面取，这样做的好处是避免重复一次运算功能，如果运算非常复杂的话，对性能很耗费，那么使用缓存对象可以提高性能;我们可以先来理解一个简单的缓存列子，就是网上常见的加法和乘法的运算。代码如下：

```js
// 计算乘法
var mult = function(){
    var a = 1;
    for(var i = 0,ilen = arguments.length; i < ilen; i+=1) {
        a = a*arguments[i];
    }
    return a;
};
// 计算加法
var plus = function(){
    var a = 0;
    for(var i = 0,ilen = arguments.length; i < ilen; i+=1) {
        a += arguments[i];
    }
    return a;
}
// 代理函数
var proxyFunc = function(fn) {
    var cache = {};  // 缓存对象
    return function(){
        var args = Array.prototype.join.call(arguments,',');
        if(args in cache) {
            return cache[args];   // 使用缓存代理
        }
        return cache[args] = fn.apply(this,arguments);
    }
};
var proxyMult = proxyFunc(mult);
console.log(proxyMult(1,2,3,4)); // 24
console.log(proxyMult(1,2,3,4)); // 缓存取 24

var proxyPlus = proxyFunc(plus);
console.log(proxyPlus(1,2,3,4));  // 10
console.log(proxyPlus(1,2,3,4));  // 缓存取 10
```