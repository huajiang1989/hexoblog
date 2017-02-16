---
title:   js使用 arguments 参数实现函数重载
category: js
tags: js
---

arguments
--
js进行函数调用时，除了指定的参数外，还创建一个隐含的对象——arguments。
<!--more-->

arguments可以用arguments[index]这样的语法取值，拥有长度属性length。arguments对
象存储的是实际传递给函数的参数，而不局限于函数声明所定义的参数列表，例如：
```js
  function func(a,b){
　　alert(a);
　　alert(b);
　　for(var i=0;i<arguments.length;i++){
      alert(arguments[i]);
　　}
  }
　func(1,2,3,4);
```
代码运行时会依次显示：1，2，1，2，3，4。函数定义了两个参数，但是在调用的时候传递了4个参数。

arguments的callee属性
--
它表示对函数对象本身的引用，这有利于实现无名函数的递归或者保证函数的封装性。例如：用递归来计算1到n的自然数之和：
```js
   var sum=function(n){
　   if(1==n) {
       return 1;
　   } else {
       return n + arguments.callee(n-1);
     }
　 }
　 alert(sum(100));
```