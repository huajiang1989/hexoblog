---
title: js设计模式-工厂模式
category:  js-design-pattern
tags: js-design-pattern
---

   工厂模式类似于现实生活中的工厂可以产生大量相似的商品，去做同样的事情，实现同样的效果;这时候需要使用工厂模式。
<!--more-->

   简单的工厂模式可以理解为解决多个相似的问题;这也是她的优点;比如如下代码：

```js
function CreatePerson(name,age,sex) {
    var obj = new Object();
    obj.name = name;
    obj.age = age;
    obj.sex = sex;
    obj.sayName = function(){
        return this.name;
    }
    return obj;
}
var p1 = new CreatePerson("longen",'28','男');
var p2 = new CreatePerson("tugenhua",'27','女');
console.log(p1.name); // longen
console.log(p1.age);  // 28
console.log(p1.sex);  // 男
console.log(p1.sayName()); // longen

console.log(p2.name);  // tugenhua
console.log(p2.age);   // 27
console.log(p2.sex);   // 女
console.log(p2.sayName()); // tugenhua

// 返回都是object 无法识别对象的类型 不知道他们是哪个对象的实列
console.log(typeof p1);  // object
console.log(typeof p2);  // object
console.log(p1 instanceof Object); // true
```
如上代码：函数CreatePerson能接受三个参数name,age,sex等参数，可以无数次调用这个函数，每次返回都会包含三个属性和一个方法的对象。

工厂模式是为了解决多个类似对象声明的问题;也就是为了解决实列化对象产生重复的问题。

优点：能解决多个相似的问题。

缺点：不能知道对象识别的问题(对象的类型不知道)。

复杂的工厂模式定义是：将其成员对象的实列化推迟到子类中，子类可以重写父类接口方法以便创建的时候指定自己的对象类型。

 父类只对创建过程中的一般性问题进行处理，这些处理会被子类继承，子类之间是相互独立的，具体的业务逻辑会放在子类中进行编写。

 父类就变成了一个抽象类，但是父类可以执行子类中相同类似的方法，具体的业务逻辑需要放在子类中去实现；比如我现在开几个自行车店，那么每个店都有几种型号的自行车出售。我们现在来使用工厂模式来编写这些代码;

父类的构造函数如下：

```js
// 定义自行车的构造函数
var BicycleShop = function(){};
BicycleShop.prototype = {
    constructor: BicycleShop,
    /*
    * 买自行车这个方法
    * @param {model} 自行车型号
    */
    sellBicycle: function(model){
        var bicycle = this.createBicycle(mode);
        // 执行A业务逻辑
        bicycle.A();

        // 执行B业务逻辑
        bicycle.B();

        return bicycle;
    },
    createBicycle: function(model){
        throw new Error("父类是抽象类不能直接调用，需要子类重写该方法");
    }
};
```
上面是定义一个自行车抽象类来编写工厂模式的实列，定义了createBicycle这个方法，但是如果直接实例化父类，调用父类中的这个createBicycle方法,会抛出一个error，因为父类是一个抽象类，他不能被实列化，只能通过子类来实现这个方法，实现自己的业务逻辑，下面我们来定义子类，我们学会如何使用工厂模式重新编写这个方法，首先我们需要继承父类中的成员，然后编写子类;如下代码：

```js
// 定义自行车的构造函数
var BicycleShop = function(name){
    this.name = name;
    this.method = function(){
        return this.name;
    }
};
BicycleShop.prototype = {
    constructor: BicycleShop,
    /*
     * 买自行车这个方法
     * @param {model} 自行车型号
    */
    sellBicycle: function(model){
            var bicycle = this.createBicycle(model);
            // 执行A业务逻辑
            bicycle.A();

            // 执行B业务逻辑
            bicycle.B();

            return bicycle;
        },
        createBicycle: function(model){
            throw new Error("父类是抽象类不能直接调用，需要子类重写该方法");
        }
    };
    // 实现原型继承
    function extend(Sub,Sup) {
        //Sub表示子类，Sup表示超类
        // 首先定义一个空函数
        var F = function(){};

        // 设置空函数的原型为超类的原型
        F.prototype = Sup.prototype;

        // 实例化空函数，并把超类原型引用传递给子类
        Sub.prototype = new F();

        // 重置子类原型的构造器为子类自身
        Sub.prototype.constructor = Sub;

        // 在子类中保存超类的原型,避免子类与超类耦合
        Sub.sup = Sup.prototype;

        if(Sup.prototype.constructor === Object.prototype.constructor) {
            // 检测超类原型的构造器是否为原型自身
            Sup.prototype.constructor = Sup;
        }
    }
    var BicycleChild = function(name){
        this.name = name;
// 继承构造函数父类中的属性和方法
        BicycleShop.call(this,name);
    };
    // 子类继承父类原型方法
    extend(BicycleChild,BicycleShop);
// BicycleChild 子类重写父类的方法
BicycleChild.prototype.createBicycle = function(){
    var A = function(){
        console.log("执行A业务操作");
    };
    var B = function(){
        console.log("执行B业务操作");
    };
    return {
        A: A,
        B: B
    }
}
var childClass = new BicycleChild("龙恩");
console.log(childClass);
```
实例化子类，然后打印出该实例, 如下截图所示：


```js
console.log(childClass.name);  // 龙恩

// 下面是实例化后 执行父类中的sellBicycle这个方法后会依次调用父类中的A

// 和B方法；A方法和B方法依次在子类中去编写具体的业务逻辑。

childClass.sellBicycle("mode"); // 打印出  执行A业务操作和执行B业务操作
```
上面只是"龙恩"自行车这么一个型号的，如果需要生成其他型号的自行车的话，可以编写其他子类，工厂模式最重要的优点是：可以实现一些相同的方法，这些相同的方法我们可以放在父类中编写代码，那么需要实现具体的业务逻辑，那么可以放在子类中重写该父类的方法，去实现自己的业务逻辑；使用专业术语来讲的话有2点：第一：弱化对象间的耦合，防止代码的重复。在一个方法中进行类的实例化，可以消除重复性的代码。第二：重复性的代码可以放在父类去编写，子类继承于父类的所有成员属性和方法，子类只专注于实现自己的业务逻辑。