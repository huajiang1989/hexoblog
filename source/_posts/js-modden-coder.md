---
title:  JavaScript 模块化编程 - Module Pattern
category: js
tags: js
---
前言

The Module Pattern，模块模式，也译为模组模式，是一种通用的对代码进行模块化组织与定义的方式。这里所说的模块（Modules），是指实现某特定功能的一组方法和代码。许多现代语言都定义了代码的模块化组织方式，比如 Golang 和 Java，它们都使用 package 与 import 来管理与使用模块，而目前版本的 JavaScript 并未提供一种原生的、语言级别的模块化组织模式，而是将模块化的方法交由开发者来实现。因此，出现了很多种 JavaScript 模块化的实现方式，比如，CommonJS Modules、AMD 等。
<!--more-->
以 AMD 为例，该规范使用 define 函数来定义模块。使用 AMD 规范进行模块化编程是很简单的，大致上的结构是这样的：

```js
define(factory(){
  // 模块代码
  // return something;
});
```
目前尚在制定中的 Harmony/ECMAScript 6（也称为 ES.next），会对模块作出语言级别的定义，但距离实用尚遥不可及，这里暂时不讨论它。

作为一种模式，模块模式其实一直伴随着 JavaScript 存在，与 ES 6 无关。最近我需要重构自己的一些代码，因此我参考和总结了一些实用的模块化编程实践，以便更好的组织我的代码。需要注意的是，本文只是个人的一个总结，比较简单和片面，详尽的内容与剖析请参看文后的参考资料，它们写得很好。本文并不关心模块如何载入，只关心现今该如何组织模块化的代码。还有，不必过于纠结所谓的模式，真正重要的其实还是模块代码及思想。所谓模式，不过是我们书写代码的一些技巧和经验的总结，是一些惯用法，实践中应灵活运用。

模块模式
--
### 闭包与 IIFE (Immediately-Invoked Function Expression)

模块模式使用了 JavaScript 的一个特性，即闭包（Closures）。现今流行的一些 JS 库中经常见到以下形式的代码：

```js
;(function (参数) {
  // 模块代码
  // return something;
})(参数);
```
上面的代码定义了一个匿名函数，并立即调用自己，这叫做自调用匿名函数（SIAF），更准确一点，称为立即调用的函数表达 (Immediately-Invoked Function Expression, IIFE–读做“iffy”)。

在闭包中，可以定义私有变量和函数，外部无法访问它们，从而做到了私有成员的隐藏和隔离。而通过返回对象或函数，或是将某对象作为参数传入，在函数体内对该对象进行操作，就可以公开我们所希望对外暴露的公开的方法与数据。

这，其实就是模块模式的本质。

注1：上面的代码中，最后的一对括号是对匿名函数的调用，因此必不可少。而前面的一对围绕着函数表达式的一对括号并不是必需的，但它可以用来给开发人员一个指示 -- 这是一个 IIFE。也有一些开发者在函数表达式前面加上一个惊叹号（!）或分号（;)，而不是用括号包起来。比如 knockoutjs 的源码大致就是这样的：

```js
!function (参数) {
  // 代码
  // return something
}(参数);
```
还有些人喜欢用括号将整个 IIFE 围起来，这样就变成了以下的形式：

```js
(function (参数) {
  // 代码
  // return something
}(参数));
```
注2：在有些人的代码中，将 undefined 作为上面代码中的一个参数，他们那样做是因为 undefined 并不是 JavaScript 的保留字，用户也可以定义它，这样，当判断某个值是否是 undefined 的时候，判断可能会是错误的。将 undefined 作为一个参数传入，是希望代码能按预期那样运行。不过我认为，一般情况下那样做并没太大意义。

### 参数输入

JavaScript 有一个特性叫做隐式全局变量（implied globals），当使用一个变量名时，JavaScript 解释器将反向遍历作用域链来查找变量的声明，如果没有找到，就假定该变量是全局变量。这种特性使得我们可以在闭包里随处引用全局变量，比如 jQuery 或 window。然而，这是一种不好的方式。

考虑模块的独立性和封装，对其它对象的引用应该通过参数来引入。如果模块内需要使用其它全局对象，应该将这些对象作为参数来显式引用它们，而非在模块内直接引用这些对象的名字。以 jQuery 为例，若在参数中没有输入 jQuery 对象就在模块内直接引用 $ 这个对象，是有出错的可能的。正确的方式大致应该是这样的：
```js
;(function (q, w) {
  // q is jQuery
  // w is window
  // 局部变量及代码
  // 返回
})(jQuery, window);
```
相比隐式全局变量，将引用的对象作为参数，使它们得以和函数内的其它局部变量区分开来。这样做还有个好处，我们可以给那些全局对象起一个别名，比如上例中的 "q"。现在看看你的代码，是否没有经过对 jQuery 的引用就到处都是"$"？

### 模块输出（Module Export）

有时我们不只是要使用全局变量，我们也要声明和输出模块中的对象，这可以通过匿名函数的 return 语句来达成，而这也构成了一个完整的模块模式。来看一个完整的例子：

```js
var MODULE = (function () {
    var my = {},
        privateVariable = 1;

    function privateMethod() {
        // ...
    }

    my.moduleProperty = 1;
    my.moduleMethod = function () {
        // ...
    };

    return my;
}());
```
这段代码声明了一个变量 MODULE，它带有两个可访问的属性：moduleProperty 和 moduleMethod，其它的代码都封装在闭包中保持着私有状态。参考以前提过的参数输入，我们还可以通过参数引用其它全局变量。

#### 输出简单对象

很多时候我们 return 一个对象作为模块的输出，比如上例就是。

另外，使用对象直接量（Object Literal Notation）来表达 JavaScript 对象是很常见的。比如：var x = { p1: 1, p2: "2", f: function(){ /*... */ } }

很多时候我们都能见到这样的模块化代码：

```js
var Module1 = (function () {
  var private_variable = 1;
  function private_method() { /*...*/ }

  var my = {
    property1: 1,
    property2: private_variable,
    method1: private_method,
    method2: function () {
        // ...
    }
  };
  return my;
}());
```
另外，对于简单的模块化代码，若不涉及私有成员等，其实也可以直接使用对象直接量来表达一个模块：

```js
var Widget1 = {
  name: "who am i?",
  settings: {
    x: 0,
    y: 0
  },
  call_me: function () {
    // ...
  }
};
```
有一篇文章讲解了这种形式： How Do You Structure JavaScript? The Module Pattern Edition

不过这只是一种简单的形式，你可以将它看作是模块模式的一种基础的简单表达形式，而把闭包形式看作是对它的一个封装。

#### 输出函数

有时候我们希望返回的并不是一个对象，而是一个函数。有两种需求要求我们返回一个函数，一种情况是我们需要它是一个函数，比如 jQuery，它是一个函数而不是一个简单对象；另一种情况是我们需要的是一个“类”而不是一个直接量，之后我们可以用 "new" 来实例它。目前版本的 JavaScript 并没有专门的“类”定义，但它却可以通过 function 来表达。

```js
var Cat = (function () {
  // 私有成员及代码 ...

  return function(name) {
    this.name = name;
    this.bark = function() { /*...*/ }
  };
}());

var tomcat = new Cat("Tom");
tomcat.bark();
```
为什么不直接定义一个 function 而要把它放在闭包里呢？简单点的情况，确实不需要使用 IIFE 这种形式，但复杂点的情况，在构造我们所需要的函数或是“类”时，若需要定义一些私有的函数，就有必要使用 IIFE 这种形式了。

另外，在 ECMAScript 第五版中，提出了 Object.create() 方法。这时可以将一个对象视作“类”，并使用 Object.create() 进行实例化，不需使用 "new"。

### Revealing Module Pattern

前面已经提到一种形式是输出对象直接量（Object Literal Notation），而 Revealing Module Pattern 其实就是这种形式，只是做了一些限定。这种模式要求在私有范围内中定义变量和函数，然后返回一个匿名对象，在该对象中指定要公开的成员。参见下面的代码：

```js
var MODULE = (function () {
  // 私有变量及函数
  var x = 1;
  function f1() {}
  function f2() {}

  return {
    public_method1: f1,
    public_method2: f2
  };
}());
```
 模块模式的变化
---
### 扩展

上面的举例都是在一个地方定义模块，如果我们需要在数个文件中分别编写一个模块的不同部分该怎么办呢？或者说，如果我们需要对已有的模块作出扩展该怎么办呢？其实也很简单，将模块对象作为参数输入，扩展后再返回自己就可以了。比如：

```js
var MODULE = (function (my) {
  my.anotherMethod = function () {
    // added method...
  };

  return my;
}(MODULE));
```
上面的代码为对象 MODULE 增加了一个 "anotherMethod" 方法。


### 松耦合扩展（Loose Augmentation）

上面的代码要求 MODULE 对象是已经定义过的。如果这个模块的各个组成部分并没有加载顺序要求的话，其实可以允许输入的参数为空对象，那么我们将上例中的参数由 MODULE 改为 MODULE || {} 就可以了：

```js
var MODULE = (function (my) {
  // add capabilities...
  return my;
}(MODULE || {}));
```

### 紧耦合扩展（Tight Augmentation）

与上例不同，有时我们要求在扩展时调用以前已被定义的方法，这也有可能被用于覆盖已有的方法。这时，对模块的定义顺序是有要求的。

```js
var MODULE = (function (my) {
  var old_moduleMethod = my.moduleMethod;

  my.moduleMethod = function () {
    // 方法重载
    // 可通过 old_moduleMethod 调用以前的方法...
  };

  return my;
}(MODULE));
```

### 克隆与继承（Cloning and Inheritance）




```js
var MODULE_TWO = (function (old) {
    var my = {},
        key;

    for (key in old) {
        if (old.hasOwnProperty(key)) {
            my[key] = old[key];
        }
    }

    var super_moduleMethod = old.moduleMethod;
    my.moduleMethod = function () {
        // override method on the clone, access to super through super_moduleMethod
    };

    return my;
}(MODULE));
```
有时我们需要复制和继承原对象，上面的代码演示了这种操作，但未必完美。如果你可以使用 Object.create() 的话，请使用 Object.create() 来改写上面的代码：

```js
var MODULE_TWO = (function (old) {
  var my = Object.create(old);

  var super_moduleMethod = old.moduleMethod;
  my.moduleMethod = function () {
    // override method ...
  };

  return my;
}(MODULE));
```

### 子模块（Sub-modules）

模块对象当然可以再包含子模块，形如 MODULE.Sub=(function(){}()) 之类，这里不再展开叙述了。

### 各种形式的混合

以上介绍了常见的几种模块化形式，实际应用中有可能是这些形式的混合体。比如：

```js
var UTIL = (function (parent, $) {
    var my = parent.ajax = parent.ajax || {};

    my.get = function (url, params, callback) {
        // ok, so I'm cheating a bit :)
        return $.getJSON(url, params, callback);
    };

    // etc...

    return parent;
}(UTIL || {}, jQuery));
```

与其它模块规范或 JS 库的适配
---
### 模块环境探测

现今，CommonJS Modules 与 AMD 有着广泛的应用，如果确定 AMD 的 define 是可用的，我们当然可以使用 define 来编写模块化的代码。然而，我们不能假定我们的代码必然运行于 AMD 环境下。有没有办法可以让我们的代码既兼容于 CommonJS Modules 或 AMD 规范，又能在一般环境下运行呢？

其实我们只需要在某个地方加上对 CommonJS Modules 与 AMD 的探测并根据探测结果来“注册”自己就可以了，以上那些模块模式仍然有用。

AMD 定义了 define 函数，我们可以使用 typeof 探测该函数是否已定义。若要更严格一点，可以继续判断 define.amd 是否有定义。另外，SeaJS 也使用了 define 函数，但和 AMD 的 define 又不太一样。

对于 CommonJS，可以检查 exports 或是 module.exports 是否有定义。

现在，我写一个比较直白的例子来展示这个过程：

```js
var MODULE = (function () {
  var my = {};
  // 代码 ...

  if (typeof define == 'function') {
    define( function(){ return my; } );
  }else if (typeof module != 'undefined' && module.exports) {
    module.exports = my;
  }
  return my;
}());
```
上面的代码在返回 my 对象之前，先检测自己是否是运行在 AMD 环境之中（检测 define 函数是否有定义），如果是，就使用 define 来定义模块，否则，继续检测是否运行于 CommonJS 中，比如 NodeJS，如果是，则将 my 赋值给 module.exports。因此，这段代码应该可以同时运行于 AMD、CommonJS 以及一般的环境之中。另外，我们的这种写法应该也可在 SeaJS 中正确执行。

### 其它一些 JS 库的做法

现在许多 JS 库都加入了对 AMD 或 CommonJS Modules 的适应，比如 jQuery, Mustache, doT, Juicer 等。

jQuery 的写法可参考 exports.js:

```js
if ( typeof module === "object" && module && typeof module.exports === "object" ) {
    module.exports = jQuery;
} else {
    if ( typeof define === "function" && define.amd ) {
        define( "jquery", [], function () { return jQuery; } );
    }
}

if ( typeof window === "object" && typeof window.document === "object" ) {
    window.jQuery = window.$ = jQuery;
}
```
与前面我写的那段代码有些不同，在对 AMD 和 CommonJS 探测之后，它将 jQuery 注册成了 window 对象的成员。

然而，jQuery 是一个浏览器端的 JS 库，它那样写当然没问题。但如果我们所写的是一个通用的库，就不应使用 window 对象了，而应该使用全局对象，而这一般可以使用 this 来得到。

我们看看 Mustache 是怎么做的：

```js
(function (root, factory) {
  if (typeof exports === "object" && exports) {
    factory(exports); // CommonJS
  } else {
    var mustache = {};
    factory(mustache);
    if (typeof define === "function" && define.amd) {
      define(mustache); // AMD
    } else {
      root.Mustache = mustache; // <script>
    }
  }
}(this, function (mustache) {
  // 模块主要的代码放在这儿
});
```
这段代码与前面介绍的方式不太一样，它使用了两个匿名函数。后面那个函数可以看作是模块代码的工厂函数，它是模块的主体部分。前面那个函数对运行环境进行检测，根据检测的结果对模块的工厂函数进行调用。另外，作为一个通用库，它并没使用 window 对象，而是使用了 this，因为在简单的函数调用中，this 其实就是全局对象。

再看看 doT 的做法。doT 的做法与 Mustache 不同，而是更接近于我在前面介绍 AMD 环境探测的那段代码：

```js
(function() {
    "use strict";

    var doT = {
        version: '1.0.0',
        templateSettings: { /*...*/ },
        template: undefined, //fn, compile template
        compile:  undefined  //fn, for express
    };

    if (typeof module !== 'undefined' && module.exports) {
        module.exports = doT;
    } else if (typeof define === 'function' && define.amd) {
        define(function(){return doT;});
    } else {
        (function(){ return this || (0,eval)('this'); }()).doT = doT;
    }
    // ...
}());
```
这段代码里的 (0, eval)('this') 是一个小技巧，这个表达式用来得到 Global 对象，'this' 其实是传递给 eval 的参数，但由于 eval 是经由 (0, eval) 这个表达式间接得到的，因此 eval 将会在全局对象作用域中查找 this，结果得到的是全局对象。若是代码运行于浏览器中，那么得到的其实是 window 对象。这里有一个针对它的讨论： http://stackoverflow.com/questions/14119988/return-this-0-evalthis/14120023#14120023

其实也有其它办法来获取全局对象的，比如，使用函数的 call 或 apply，但不给参数，或是传入 null：

```js
var global_object = (function(){ return this; }).call();
```
你可以参考这篇文章： Javascript的this用法

Juicer 则没有检测 AMD，它使用了如下的语句来检测 CommonJS Modules：

```js
typeof(module) !== 'undefined' && module.exports ? module.exports = juicer : this.juicer = juicer;
```
另外，你还可以参考一下这个： https://gist.github.com/kitcambridge/1251221
```js
(function (root, Library) {
  // The square bracket notation is used to avoid property munging by the Closure Compiler.
  if (typeof define == "function" && typeof define["amd"] == "object" && define["amd"]) {
    // Export for asynchronous module loaders (e.g., RequireJS, `curl.js`).
    define(["exports"], Library);
  } else {
    // Export for CommonJS environments, web browsers, and JavaScript engines.
    Library = Library(typeof exports == "object" && exports || (root["Library"] = {
      "noConflict": (function (original) {
        function noConflict() {
          root["Library"] = original;
          // `noConflict` can't be invoked more than once.
          delete Library.noConflict;
          return Library;
        }
        return noConflict;
      })(root["Library"])
    }));
  }
})(this, function (exports) {
  // ...
  return exports;
});
```
我觉得这个写得有些复杂了，我也未必需要我的库带有 noConflict 方法。不过，它也可以是个不错的参考。

 JavaScript 模块化的未来
--
未来的模块化方案会是什么样的？我不知道，但不管将来如何演化，作为一种模式，模块模式是不会过时和消失的。

如前所述，尚在制定中的 ES 6 会对模块作出语言级别的定义。我们来看一个实例，以下的代码段摘自“ES6:JavaScript中将会有的几个新东西”：

```js
module Car {
  // 内部变量
  var licensePlateNo = '556-343';
  // 暴露到外部的变量和函数
  export function drive(speed, direction) {
    console.log('details:', speed, direction);
  }
  export module engine{
    export function check() { }
  }
  export var miles = 5000;
  export var color = 'silver';
};
```
我不知道 ES 6 将来会否对此作出改变，对上面的这种代码形式，不同的人会有不同的看法。就我个人而言，我十分不喜欢这种形式！

确实，我们可能需要有一种统一的模块化定义方式。发明 AMD 和 RequireJS 的人也说过 AMD 和 RequireJS 应该被淘汰了，运行环境应该提供模块的原生支持。然而，ES 6 中的模块定义是否是正确的？它是否是一个好的解决方案呢？我不知道，但我个人真的很不喜欢那种方式。很多人十分喜欢把其它语言的一些东西生搬硬套到 JavaScript 中，或是孜孜不倦地要把 JavaScript 变成另外一种语言，我相当讨厌这种行为。我并非一个保守的人，我乐意接受新概念、新语法，只要它是好的。但是，ES 6 草案中的模块规范是我不喜欢的，起码，我认为它脱离了现实，否定了开源社区的实践和经验，是一种意淫出来的东西，这使得它在目前不能解决任何实际问题，反而是来添乱的。

按目前的 ES6 草案所给出的模块化规范，它并没有采用既有的 CommonJS Modules 和 AMD 规范，而是定义了一种新的规范，而且这种规范修改了 JavaScript 既有的语法形式，使得它没有办法像 ES5 中的 Object.create、Array.forEach 那样可以利用现有版本的 JavaScript 编写一些代码来实现它。这也使得 ES 6 的模块化语法将在一段时期内处于不可用的状态。

引入新的语法也不算是问题，然而，为了模块而大费周折引出那么多新的语法和定义，真的是一种好的选择么？话说，它解决了什么实质性的问题而非如此不可？现今流行的 AMD 其实简单到只定义了一个 "define" 函数，它有什么重大问题？就算那些专家因种种原因或目的而无法接受 AMD 或其它开源社区的方案，稍作出一些修改和中和总是可以的吧，非要把 JavaScript 改头换面不可么？确实有人写了一些观点来解释为何不用 AMD，然而，那些解释和观点其实大都站不住脚。比如说，其中一个解释是 AMD 规范不兼容于 ES 6！可笑不可笑？ES 6 尚未正式推出，完全实现了 ES 6 的 JavaScript 运行时也没几个，而 AMD 在开源社区中早已十分流行，这个时候说 AMD 不兼容 ES 6，我不知道这是什么意思。

就我看来，现今各种形形色色的所谓标准化工作组，很多时候像是高高在上的神仙，他们拉不下脸全身心地参与到开源社区之中，他们就是要作出与开源社区不同的规范，以此来彰显他们的工作、专业与权威。而且，很多时候他们过于官僚，又或者夹杂在各大商业集团之间举棋不定。我不否认他们工作的重要性，然而，以专家自居而脱离或否定开源社区的实践，以及商业与政治的利益均衡等，使得他们的工作与开源社区相比，在技术的推动与发展上成效不足甚至添乱。

回到 ES 6 中的模块，想想看，我需要修改我的代码，在其中加上诸如 module, export, import 之类的新的语法，修改之后的代码却没办法在现今版本的 JavaScript 中运行，而且，与现今流行的模块化方案相比，这些工作也没什么实质性的帮助，想想这些，我只感觉像是吃了一个苍蝇。

ES 6 的发展当然不会因为我的吐嘈而有任何变化，我也不愿再展开讨论。未来的模块化方案具体是什么样的无法知晓，但起码我可以得到以下的结论：

模块模式不会过时
ES 6 不会接纳 AMD 等现有方案，但不管如何，JavaScript 将会有语言级别的模块定义
ES 6 中的模块在一段时期内是不可用的
即使 ES 6 已达到实用阶段，现今的模块化方案仍会存在和发展

