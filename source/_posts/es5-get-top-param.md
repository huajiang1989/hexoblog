---
title: ES5中获取顶层变量的方法
category: js
tags: js
---
ES5的顶层对象，本身也是一个问题，因为它在各种实现里面是不统一的。

浏览器里面，顶层对象是window，但Node和Web Worker没有window。
浏览器和Web Worker里面，self也指向顶层对象，但是Node没有self。
Node里面，顶层对象是global，但其他环境都不支持。
同一段代码为了能够在各种环境，都能取到顶层对象，现在一般是使用this变量，但是有局限性。
<!--more-->

全局环境中，this会返回顶层对象。但是，Node模块和ES6模块中，this返回的是当前模块。
函数里面的this，如果函数不是作为对象的方法运行，而是单纯作为函数运行，this会指向顶层对象。但是，严格模式下，这时this会返回undefined。
不管是严格模式，还是普通模式，new Function('return this')()，总是会返回全局对象。但是，如果浏览器用了CSP（Content Security Policy，内容安全政策），那么eval、new Function这些方法都可能无法使用。
综上所述，很难找到一种方法，可以在所有情况下，都取到顶层对象。下面是两种勉强可以使用的方法。
```js
// 方法一
(typeof window !== 'undefined'
? window
: (typeof process === 'object' &&
typeof require === 'function' &&
typeof global === 'object')
? global
: this);

// 方法二
var getGlobal = function () {
if (typeof self !== 'undefined') { return self; }
if (typeof window !== 'undefined') { return window; }
if (typeof global !== 'undefined') { return global; }
throw new Error('unable to locate global object');
};
```
现在有一个提案，在语言标准的层面，引入global作为顶层对象。也就是说，在所有环境下，global都是存在的，都可以从它拿到顶层对象。

垫片库system.global模拟了这个提案，可以在所有环境拿到global。
```js
// CommonJS的写法
 require('system.global/shim')();
 // ES6模块的写法
 import shim from 'system.global/shim'; shim();
```
上面代码可以保证各种环境里面，global对象都是存在的。
```js
// CommonJS的写法
 var global = require('system.global')();
 // ES6模块的写法
 import getGlobal from 'system.global';
 const global = getGlobal();
```
上面代码将顶层对象放入变量global。