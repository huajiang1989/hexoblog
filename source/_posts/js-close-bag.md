---
title:   Javascript函数、递归与闭包
tags: js
---
# 函数表达式

JavaScript中定义函数有2钟方法：
## 函数声明
```js
function funcName(arg1,arg2,arg3){
  //函数体
}
```
      ①name属性：可读取函数名。非标准，浏览器支持:FF、Chrome、safari、Opera。
      ②函数声明提升：指执行代码之前会先读取函数声明。即函数调用可置于函数声明之前。
## 函数表达式
```js
var funcName = function(arg1,arg2,arg3){
  //函数体
};
```
<!--more-->
　　　　 ①匿名函数(anonymous function，或拉姆达函数)：function关键字后无标识符，name属性值为空字符串。在把函数当成值使用时，都可用匿名函数。
　　　　 ②类似其他表达式，函数表达式使用前需先赋值，故不存在"函数声明提升"那样的作用。
　　　　 ③ECMAScript中的无效函数语法：


if判断中的重复函数声明
```js
if(condition){
    function sayHi(){
        alert("Hi!");
    }
} else {
    function sayHi(){
        alert("Yo!");
    }
}
```
　　　　　　浏览器JavaScript引擎修正错误差异：大多浏览器会返回第二个声明，忽略condition；FF则会在condition为true时返回第一个声明。
　　　　　　使用函数表达式可解决并实现：


if判断 函数表达式
```js
var sayHi;
if(condition){
    sayHi = function(){
        alert("Hi!");
    }
} else {
    sayHi = function(){
        alert("Yo!");
    }
}
```
# 递归

　　递归函数，是在一个函数中通过名字调用自身的情况下构成的。
```js
function factorial(num){   //一个经典的递归阶乘函数
    if (num <= 1){
        return 1;
    } else {
        return num * factorial(num-1);
    }
}
```
 　　　　①若使用下列代码调用该函数，会出错：
```js
var anotherFactorial = factorial;
factorial = null;
alert(anotherFactorial(4));
```
　　　将factorial()函数保存到变量anotherFactorial中后，将factorial变量设为null后不再引用函数，而anotherFactorial(4)中要执行factorial()函数，故出错。
　　　使用argument.callee(指向正在执行的函数的指针)可解决：

```js
//解决方案
function factorial(num){
    if (num <= 1){
        return 1;
    } else {
        return num * arguments.callee(num-1);
    }
}
var anotherFactorial = factorial;
factorial = null;
alert(anotherFactorial(4));  //24
```
　　　　在非严格模式，使用递归函数时，用argument.callee代替函数名更保险
　　　　在严格模式下，使用argument.callee会出错，可用函数表达式 代替 函数声明：

函数表达式代替函数声明
```js
var factorial = function f(num){
    if (num <= 1){
        return 1;
    } else {
        return num * f(num-1);
    }
}
```
# 闭包

　　指有权访问另一个函数作用域中的变量的函数。(常见形式为函数嵌套)
复制代码代码如下:
```js
function wai(pro){
    return function(obj1,obj2){
        var val1 = obj1[pro];
        var val2 = obj2[pro];
        if(val1<val2){
            return -1;
        }else if(val1>val2){
            return 1;
        }else{
            return 0;
        };
    }
}
```
　　　　return匿名函数时，匿名函数的作用域链初始化为包含函数的活动对象和全局变量对象。即匿名函数包含wai()函数的作用域。
　　每个函数被调用时，会创建一个执行环境、一个变量对象 及 相应的作用域链。
##执行环境 及 作用域
　　执行环境execution context简称环境，定义了变量和函数有权访问的其他数据，并决定他们的各自行为。
　　①每个执行环境都有一个变量对象variable object，保存环境定义的所有变量和函数。该对象无法编码访问，但解析器在处理数据时会在后台使用它。
　　  全局变量对象是最外围的一个执行环境。在Web浏览器中被认为是window对象，故所有全局对象和函数都是window对象的属性和方法创建的。
　　  执行环境中的代码执行完后，该环境就被销毁，保存其中的变量和函数定义也随之销毁。
　　②代码在环境中执行时，会创建变量对象的一个作用域链scope chain，用于保证对执行环境有权访问的所有变量和函数的有序访问。
　　  作用域链前端，始终是当前执行的代码所在环境的变量对象。当该环境为函数时，会将活动对象作为变量对象。
　　  活动对象最开始只包含一个变量，即argumnt对象。
　　  作用域链中的下一个变量对象来自包含环境，而下一个变量对象来自下一个包含环境，直至延续到全局执行环境。
　　③标识符解析：从前段开始，沿着作用域链一级一级地搜索标识符的过程。【找不到通常会导致错误发生】
##函数创建、执行时：
```js
function compare(val1,val2){
     if(val1<val2){
        return -1;
    }else if(val1>val2){
        return 1;
    }else{
        return 0;
    };
}
var result = compare(5 , 10);
```
　　①创建函数compare()时，会创建一个预先包含全局变量对象的作用域链，并保存在内部[[scope]]属性中。
　　②局部函数compare()的变量对象，只在函数执行的过程中存在。
　　　当调函数时，会创建一个执行环境，再通过复制函数的[[scope]]属性中的对象 构建起执行环境的作用域链。
　　③第一次调用函数时，如compare()，会创建一个包含this、argument、val1 和 val2的活动对象。
　　④全局执行环境的变量对象(包括this、result、compare)在compare()执行环境的作用域链中处于第二位。
　　⑤作用域链 本质是一个指向变量对象的指针列表，只引用但不实际包含变量对象。
　　⑥无论什么时候在函数中访问一个变量，都会行作用域链中搜索具有相应名字的变量。
## 闭包的作用域链
　　在另外一个函数内部定义的函数会将包含函数的活动对象添加到它的作用域链中。
　　①将函数对象赋值null，等于通知垃圾回收例程将其清除，随着函数作用域链被销毁，其作用域链(不除了全局作用域)也会被安全销毁。
　　②由于闭包会携带包含函数的作用域，所以会比其他函数占用更多内存。
## 闭包与变量
　　作用域链的一个副作用：闭包只能取得包含函数中任何变量的最后一个值。
```js
function createFunctions(){
    var result = new Array();
    for (var i=0; i < 10; i++){
        result[i] = function(){
            return i;
        };
    }
    return result;
}
```
　　①createFunctions()函数，将10个闭包赋值给数组result，再返回result数组。每个闭包都返回自己的索引，但实际上都返回10。
　　　因为每个函数(闭包)的作用域链中都保存着createFunctions()函数的活动对象，所以它们引用的是同一个变量i，当createFunctions函数执行完后i的值10，故闭包中的i也都为10。
　　②解决办法，不使用闭包，创建一个匿名函数，将i值赋值给其参数：
```js
function createFunctions(){
    var result = new Array();
    for (var i=0; i < 10; i++){
        result[i] = function(num){
            return function(){
                return num;
            };
        }(i);
    }
    return result;
}
```
　　创建一个每次循环都会执行一次的匿名函数：将每次循环时包围函数的i值作为参数，存入匿名函数中。因为函数参数是按值传递的，而非引用，所以每个匿名函数中的num值 都为每此循环时i值的一个副本。
## this对象
　　this对象是在运行时基于函数的执行环境绑定的。
　　　　在全局函数中，this等于window；当函数被某对象调用时，this为该对象。
　　　　匿名函数的执行环境有全局性，其this对象通常指window。通过call()或spply()改变函数执行环境时，this指向其对象。
　　①每个函数在被调用时，都会自动取得两个特殊变量：this和argument。内部函数在搜索这两个变量时，只会搜索到期活动对象为止，永远不可能访问外部函数的这两个变量。
　　　　不过将外部作用域的this对象保存在一个闭包能访问的变量里，就可让闭包访问该对象。

## 访问外部函数的this对象
```js
var name = "The Window";
var object = {
    name : "My Object",
    getNameFunc : function(){
        var that = this;
        return function(){
            return that.name;
        };
    }
};
alert(object.getNameFunc()());  //"MyObject"
```
　　　包围函数的argument对象 也可通过此方法被闭包访问。
5、函数声明 转换为 函数表达式
　　JavaScript将function关键字昨晚函数声明的开始，但函数声明后面不能跟圆括号，所以function(){......}();会出错。
　　要将函数声明转换为函数表达式，需为函数声明加一对圆括号：
```js
(function(){
   //块级作用域
})();
```