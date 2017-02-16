---
title:   JS中的 call和apply的区别和作用
tags: js
---

Javascript的每个Function对象中有一个apply方法：
```js
function.apply([thisObj[,argArray]])
```
<!--more-->
还有一个类似功能的call方法：

```js
function.call([thisObj[,arg1[, arg2[, [,.argN]]]]])
```
它们各自的定义：

apply：应用某一对象的一个方法，用另一个对象替换当前对象。

call：调用一个对象的一个方法，以另一个对象替换当前对象。

它们的共同之处：



都“可以用来代替另一个对象调用一个方法，将一个函数的对象上下文从初始的上下文改变为由 thisObj 指定的新对象。”

它们的不同之处：



apply：

最多只能有两个参数——新this对象和一个数组 argArray。如果给该方法传递多个参数，则把参数都写进这个数组里面，当然，即使只有一个参数，也要写进数组里面。如果 argArray 不是一个有效的数组或者不是 arguments 对象，那么将导致一个 TypeError。如果没有提供 argArray 和 thisObj 任何一个参数，那么 Global 对象将被用作 thisObj，并且无法被传递任何参数。

call：

则是直接的参数列表，主要用在js对象各方法互相调用的时候，使当前this实例指针保持一致,或在特殊情况下需要改变this指针。如果没有提供 thisObj 参数，那么 Global 对象被用作 thisObj。

更简单地说，apply和call功能一样，只是传入的参数列表形式不同：如 func.call(func1,var1,var2,var3)   对应的apply写法为：func.apply(func1,[var1,var2,var3])

也就是说：call调用的为单个，apply调用的参数为数组

```js
function sum(a,b){
    console.log(this === window);//true
    console.log(a + b);
 }
 sum(1,2);
 sum.call(null,1,2);
 sum.apply(null,[1,2]);
```

作用　　

调用函数
--
```js
var info = 'tom';
function foo(){
    //this指向window
    var info = 'jerry';
    console.log(this.info);  //tom
    console.log(this===window)   //true
}
foo();
foo.call();
foo.apply();
```

call和apply可以改变函数中this的指向　　
--
```js
var obj = {
        info:'spike'
};
foo.call(obj);    //这里foo函数里面的this就指向了obj
foo.apply(obj);
```
c、借用别的对象的方法
求数组中的最大值

```js
var arr = [123,34,5,23,3434,23];
//方法一
var arr1 = arr.sort(function(a,b){
    return b-a;
});
console.log(arr1[0]);
//方法二
var max = Math.max.apply(null,arr)   //借用别的对象的方法
console.log(max);
```