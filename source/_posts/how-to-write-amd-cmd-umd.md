---
title:  AMD、CMD、UMD 模块的写法
category: es6
tags: es6
---

简介

最近几年，我们可以选择的Javascript组件的生态系统一直在稳步增长。虽然陡增的选择范围是极好的，但当组件混合匹配使用时就会出现很尴尬的局面。开发新手们会很快发现不是所有组件都能彼此“和平相处”。

为了解决这个问题，两种竞争关系的模块规范AMD和CommonJS问世了，它们允许开发者遵照一种约定的沙箱化和模块化的方式来写代码，这样就能避免“污染生态系统”。
<!--more-->
AMD
---
随着RequireJS成为最流行的实现方式，异步模块规范（AMD）在前端界已经被广泛认同。

下面是只依赖jquery的模块foo的代码：

JavaScript
```js
//    文件名: foo.js
define(['jquery'], function ($) {
    //    方法
    function myFunc(){};

    //    暴露公共方法
    return myFunc;
});
```
```js
//    文件名: foo.js
define(['jquery'], function ($) {
    //    方法
    function myFunc(){};

    //    暴露公共方法
    return myFunc;
});
```
还有稍微复杂点的例子，下面的代码依赖了多个组件并且暴露多个方法:

JavaScript
```js
// 文件名: foo.js
define(['jquery', 'underscore'], function ($, _) {
// 方法
function a(){}; // 私有方法，因为没有被返回(见下面)
function b(){}; // 公共方法，因为被返回了
function c(){}; // 公共方法，因为被返回了
     //    暴露公共方法
    return {
        b: b,
        c: c
    }
});
```
```js
// 文件名: foo.js
define(['jquery', 'underscore'], function ($, _) {
// 方法
function a(){}; // 私有方法，因为没有被返回(见下面)
function b(){}; // 公共方法，因为被返回了
function c(){}; // 公共方法，因为被返回了
     //    暴露公共方法
    return {
        b: b,
        c: c
    }
});
```
定义的第一个部分是一个依赖数组，第二个部分是回调函数，只有当依赖的组件可用时（像RequireJS这样的脚本加载器会负责这一部分，包括找到文件路径）回调函数才被执行。

注意，依赖组件和变量的顺序是一一对应的（例如，jquery->$, underscore->_）。

同时注意，我们可以用任意的变量名来表示依赖组件。假如我们把$改成$$，在函数体里面的所有对jQuery的引用都由$变成了$$。

还要注意，最重要的是你不能在回调函数外面引用变量$和_，因为它相对其它代码是独立的。这正是模块化的目的所在！

CommonJS
--
如果你用Node写过东西的话，你可能会熟悉CommonJS的风格（node使用的格式与之相差无几）。因为有Browserify，它也一直被前端界广泛认同。

就像前面的格式一样，下面是用CommonJS规范实现的foo模块的写法：

JavaScript
```js
//    文件名: foo.js
//    依赖
var $ = require('jquery');
//    方法
function myFunc(){};

//    暴露公共方法（一个）
module.exports = myFunc;
```
```js
//    文件名: foo.js
//    依赖
var $ = require('jquery');
//    方法
function myFunc(){};

//    暴露公共方法（一个）
module.exports = myFunc;
```
还有更复杂的例子，下面的代码依赖了多个组件并且暴露多个方法：

JavaScript
```js
//    文件名: foo.js
var $ = require('jquery');
var _ = require('underscore');

//    methods
function a(){};    //    私有方法，因为它没在module.exports中 (见下面)
function b(){};    //    公共方法，因为它在module.exports中定义了
function c(){};    //    公共方法，因为它在module.exports中定义了

//    暴露公共方法
module.exports = {
    b: b,
    c: c
};
```
```js
//    文件名: foo.js
var $ = require('jquery');
var _ = require('underscore');

//    methods
function a(){};    //    私有方法，因为它没在module.exports中 (见下面)
function b(){};    //    公共方法，因为它在module.exports中定义了
function c(){};    //    公共方法，因为它在module.exports中定义了

//    暴露公共方法
module.exports = {
    b: b,
    c: c
};
```
UMD: 通用模块规范
--
既然CommonJs和AMD风格一样流行，似乎缺少一个统一的规范。所以人们产生了这样的需求，希望有支持两种风格的“通用”模式，于是通用模块规范（UMD）诞生了。

不得不承认，这个模式略难看，但是它兼容了AMD和CommonJS，同时还支持老式的“全局”变量规范：
JavaScript
```js
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD
        define(['jquery'], factory);
    } else if (typeof exports === 'object') {
        // Node, CommonJS之类的
        module.exports = factory(require('jquery'));
    } else {
        // 浏览器全局变量(root 即 window)
        root.returnExports = factory(root.jQuery);
    }
}(this, function ($) {
    //    方法
    function myFunc(){};

    //    暴露公共方法
    return myFunc;
}));
```
```js
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD
        define(['jquery'], factory);
    } else if (typeof exports === 'object') {
        // Node, CommonJS之类的
        module.exports = factory(require('jquery'));
    } else {
        // 浏览器全局变量(root 即 window)
        root.returnExports = factory(root.jQuery);
    }
}(this, function ($) {
    //    方法
    function myFunc(){};

    //    暴露公共方法
    return myFunc;
}));
```
保持跟上面例子一样的模式，下面是更复杂的例子，它依赖了多个组件并且暴露多个方法:
JavaScript
```js
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD
        define(['jquery', 'underscore'], factory);
    } else if (typeof exports === 'object') {
        // Node, CommonJS之类的
        module.exports = factory(require('jquery'), require('underscore'));
    } else {
        // 浏览器全局变量(root 即 window)
        root.returnExports = factory(root.jQuery, root._);
    }
}(this, function ($, _) {
    //    方法
    function a(){};    //    私有方法，因为它没被返回 (见下面)
    function b(){};    //    公共方法，因为被返回了
    function c(){};    //    公共方法，因为被返回了

    //    暴露公共方法
    return {
        b: b,
        c: c
    }
}));
```
```js
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD
        define(['jquery', 'underscore'], factory);
    } else if (typeof exports === 'object') {
        // Node, CommonJS之类的
        module.exports = factory(require('jquery'), require('underscore'));
    } else {
        // 浏览器全局变量(root 即 window)
        root.returnExports = factory(root.jQuery, root._);
    }
}(this, function ($, _) {
    //    方法
    function a(){};    //    私有方法，因为它没被返回 (见下面)
    function b(){};    //    公共方法，因为被返回了
    function c(){};    //    公共方法，因为被返回了

    //    暴露公共方法
    return {
        b: b,
        c: c
    }
}));
```