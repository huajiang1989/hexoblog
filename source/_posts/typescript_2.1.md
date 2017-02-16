
---
title: TypeScript 2.1 新特性一览
category: TypeScript
tags: TypeScript
---

继 TypeScript 2.0 正式版发布两个半月后, TypeScript 2.1 也终于来了. 这次更新带来了不少有用的改进, 特别是查找/映射类型及 any 类型的推断. 除此之外, 也用上了新的代码生成机制. 以下则是译自官方 Wiki 的内容.
keyof 与查找类型

<!--more-->

# keyof 与查找类型
在 JavaScript 生态里常常会有 API 接受属性名称作为参数的情况, 但到目前为止还无法表达这类 API 的类型关系.

入口索引类型查询或者说 keyof; 索引类型查询 keyof T 会得出 T 可能的属性名称的类型. keyof T 类型被认为是 string 的子类型.

例子
```js
interface Person {
    name: string;
    age: number;
    location: string;
}

type K1 = keyof Person; // "name" | "age" | "location"
type K2 = keyof Person[];  // "length" | "push" | "pop" | "concat" | ...
type K3 = keyof { [x: string]: Person };  // string
```
与之对应的是索引访问类型, 也叫作查找类型 (lookup types). 语法上, 它们看起来和元素访问完全相似, 但是是以类型的形式使用的:

例子
```js
type P1 = Person["name"];  // string
type P2 = Person["name" | "age"];  // string | number
type P3 = string["charAt"];  // (pos: number) => string
type P4 = string[]["push"];  // (...items: string[]) => number
type P5 = string[][0];  // string
```
你可以将这种形式与类型系统中的其他功能组合, 来获得类型安全的查找.
```js
function getProperty<T, K extends keyof T>(obj: T, key: K) {
    return obj[key];  // 推断的类型为 T[K]
}

function setProperty<T, K extends keyof T>(obj: T, key: K, value: T[K]) {
    obj[key] = value;
}

let x = { foo: 10, bar: "hello!" };

let foo = getProperty(x, "foo"); // number
let bar = getProperty(x, "bar"); // string

let oops = getProperty(x, "wargarbl"); // 错误! "wargarbl" 不满足类型 "foo" | "bar"

setProperty(x, "foo", "string"); // 错误! string 应该是 number
```
# 映射类型

一个常见的需求是取一个现有的类型, 并将他的所有属性转换为可选值. 假设我们有 Person 类型:
```js
interface Person {
    name: string;
    age: number;
    location: string;
}
```
它的部分类型 (partial) 的版本会是这样:
```js
interface PartialPerson {
    name?: string;
    age?: number;
    location?: string;
}
```
有了映射类型, PartialPerson 就可以被写作对于 Person 类型的一般化转换:
```js
type Partial<T> = {
    [P in keyof T]?: T[P];
};

type PartialPerson = Partial<Person>;
```
映射类型是获取字面量类型的并集, 再通过计算新对象的属性集合产生的. 它们和 Python 中的列表解析 相似, 但不是在列表中创建新的元素, 而是在类型中创建新的属性.

除了 Partial 之外, 映射类型可以表达很多有用的类型转换:
```js
// 保持类型一致, 但使每一个属性变为只读
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};

// 相同的属性名称, 但使值为 Promise 而不是具体的值
type Deferred<T> = {
    [P in keyof T]: Promise<T[P]>;
};

// 为 T 的属性添加代理
type Proxify<T> = {
    [P in keyof T]: { get(): T[P]; set(v: T[P]): void }
};
```
# Partial, Readonly, Record 以及 Pick

Partial 与 Readonly, 就像之前提到的, 是非常有用的结构. 你可以使用它们来描述一些常见的 JS 实践, 比如:

function assign<T>(obj: T, props: Partial<T>): void;
function freeze<T>(obj: T): Readonly<T>;
正因为如此, 它们现在默认被包含在了标准库中.

我们还引入了另外两种工具类型: Record 和 Pick.
```js
// 从 T 挑选一些属性 K
declare function pick<T, K extends keyof T>(obj: T, ...keys: K[]): Pick<T, K>;

const nameAndAgeOnly = pick(person, "name", "age");  // { name: string, age: number }
// 对所有 T 类型的属性 K, 将它转换为 U
function mapObject<K extends string | number, T, U>(obj: Record<K, T>, f: (x: T) => U): Record<K, U>;

const names = { foo: "hello", bar: "world", baz: "bye" };
const lengths = mapObject(names, s => s.length);  // { foo: number, bar: number, baz: number }
```
# 对象的展开与剩余运算符

TypeScript 2.1 带来了对 ES2017 展开与剩余运算符的支持.

和数组的展开类似, 展开一个对象可以很方便地获得它的浅拷贝:
```js
let copy = { ...original };
```
相似的, 你可以合并多个不同的对象. 在下面的例子中, merged 会有来自 foo, bar 和 baz 的属性.
```js
let merged = { ...foo, ...bar, ...baz };
```
你也可以覆盖已有的属性和添加新的属性:
```js
let obj = { x: 1, y: "string" };
var newObj = {...obj, z: 3, y: 4}; // { x: number, y: number, z: number }
```
指定展开操作的顺序决定了那些属性的值会留在创建的对象里; 在靠后的展开中出现的属性会 "战胜" 之前创建的属性.

对象的剩余操作和对象的展开是对应的, 这样一来我们可以导出解构一个元素时被漏掉的其他属性.
```js
let obj = { x: 1, y: 1, z: 1 };
let { z, ...obj1 } = obj;
obj1; // {x: number, y: number};
```
# 异步函数的向下编译

这一特性在 TypeScript 2.1 前就已经被支持, 但仅仅是当编译到 ES6/ES2015 的时候. TypeScript 2.1 带来了编译到 ES3 和 ES5 运行时的能力, 意味着你可以自由地运用这项优势到任何你在使用的环境.

注意: 首先, 我们需要确保我们的运行时有和 ECMAScript 兼容的全局 Promise. 这可能需要使用一个 Promise 的实现, 或者依赖目标运行时中的实现. 我们还需要通过设置 lib 选项为像 "dom", "es2015" 或者 "dom", "es2015.promise", "es5" 这样的值来确保 TypeScript 知道 Promise 存在.

例子tsconfig.json
```js
{
    "compilerOptions": {
        "lib": ["dom", "es2015.promise", "es5"]
    }
}
```
dramaticWelcome.ts
```js
function delay(milliseconds: number) {
    return new Promise<void>(resolve => {
        setTimeout(resolve, milliseconds);
    });
}

async function dramaticWelcome() {
    console.log("你好");

    for (let i = 0; i < 3; i++) {
        await delay(500);
        console.log(".");
    }

    console.log("世界!");
}

dramaticWelcome();
```
编译和运行, 在 ES3/ES5 引擎中应该也会有正确的行为.

# 支持外部工具库 (tslib)

TypeScript 会注入一些工具函数, 比如用于继承的 __extends, 用于对象字面量与 JSX 元素中展开运算符的 __assign, 以及用于异步函数的 __awaiter.

过去我们有两个选择:

在所有需要的文件中注入这些工具函数, 或者
使用 --noEmitHelpers 完全不输出工具函数.
这两个选项很难满足已有的需求; 在每一个文件中加入这些工具函数对于关心包大小的客户来说是一个痛点. 而不包含工具函数又意味着客户需要维护自己的工具库.

TypeScript 2.1 允许在你的项目中将这些文件作为单独的模块引用, 而编译器则会在需要的时候导入它们.

首先, 安装 tslib 工具库:
```bash
npm install tslib
```
接下来, 使用 --importHelpers 选项编译你的文件:
```bash
tsc --module commonjs --importHelpers a.ts
```
所以使用以下作为输入, 输出的 .js 文件就会包含对 tslib 的引入, 并且使用其中的 ___assign 工具函数而不是将它输出在文件中.
```js
export const o = { a: 1, name: "o" };
export const copy = { ...o };
"use strict";
var tslib_1 = require("tslib");
exports.o = { a: 1, name: "o" };
exports.copy = tslib_1.__assign({}, exports.o);
```
# 未添加类型的导入

TypeScript 过去对于如何导入模块有一些过于严格. 这样的本意是避免拼写错误, 并且帮助用户正确地使用模块.

然而, 很多时候, 你可能仅仅是想导入一个没有它自己的 .d.ts 文件的现有模块. 之前这是会导致错误. 从 TypeScript 2.1 开始, 则会容易很多.

使用 TypeScript 2.1, 你可以导入一个 JavaScript 模块而无需类型声明. 类型声明 (比如 declare module "foo" { ... } 或者 node_modules/@types/foo) 如果存在的话仍具有更高的优先级.

对没有声明文件的模块的导入, 在 --noImplicitAny 时仍会被标记为错误.

例子
```js
// 如果 `node_modules/asdf/index.js` 存在, 或 `node_modules/asdf/package.json` 定义了合法的 "main" 入口即可
import { x } from "asdf";
```
# 对 --target ES2016, --target ES2017 及 --target ESNext 的支持

TypeScript 2.1 支持了三个新的目标版本值 --target ES2016, --target ES2017 及 --target ESNext.

使用目标版本 --target ES2016 会告诉编译器不要对 ES2016 的特性进行转换, 比如 ** 运算符.

相似的, --target ES2017 会告诉编译器不要转换 ES2017 的特性, 比如 async/await.

--target ESNext 则对应最新的 ES 提案特性的支持.

改进的 any 推断

之前, 如果 TypeScript 不能弄明白一个变量的类型, 它会选择 any 类型.
```js
let x;      // 隐式的 'any'
let y = []; // 隐式的 'any[]'

let z: any; // 显式的 'any'.
```
在 TypeScript 2.1 中, 不同于简单地选择 any, TypeScript 会根据之后的赋值推断类型.

这仅会在 --noImplicitAny 时开启.

例子
```js
let x;

// 你仍可以将任何值赋给 'x'.
x = () => 42;

// 在上一次赋值后, TypeScript 2.1 知道 'x' 的类型为 '() => number'.
let y = x();

// 得益于此, 它现在会告诉你你不能将一个数字和函数相加!
console.log(x + y);
//          ~~~~~
// 错误! 运算符 '+' 不能被使用在类型 '() => number' 和 'number' 上.

// TypeScript 仍允许你将任何值赋给 'x'
x = "Hello world!";

// 但现在它也会知道 'x' 是 'string'!
x.toLowerCase();
```
同样的追踪现在对于空数组也会生效.

一个没有类型标注, 初始值为 [] 的变量声明被认为是一个隐式的 any[] 变量. 不过, 接下来的 x.push(value), x.unshift(value) 或者 x[n] = value 操作将依据添加的元素去演进变量的类型.
```js
function f1() {
    let x = [];
    x.push(5);
    x[1] = "hello";
    x.unshift(true);
    return x;  // (string | number | boolean)[]
}

function f2() {
    let x = null;
    if (cond()) {
        x = [];
        while (cond()) {
            x.push("hello");
        }
    }
    return x;  // string[] | null
}
```
# 隐式 any 错误

这个特性的一大好处就是, 使用 --noImplicitAny 时你会看到的隐式 any 错误会比之前少非常多. 隐式 any 错误仅仅会在编译器不通过类型声明就无法知道变量类型时被报告.

例子
```js
function f3() {
    let x = [];  // 错误: 变量 'x' 隐式地有类型 'any[]' 在一些位置的类型无法被确定.
    x.push(5);
    function g() {
        x;    // 错误: 变量 'x' 隐式地有类型 'any[]'.
    }
}
```
# 对字面量类型更好的推断

字符串, 数字和布尔值字面量类型 (例如 "abc", 1, 和 true) 在之前仅会在有显式的类型标注时被使用. 从 TypeScript 2.1 开始, 对于 const 变量和 readonly 属性, 字面量类型会始终作为推断的结果.

对于没有类型标注的 const 变量和 readonly 属性, 推断的类型为字面量初始值的类型. 对于有初始值, 没有类型标注的 let 变量, var 变量, 参数, 或者非 readonly 的属性, 推断的类型为拓宽的字面量初始值的类型. 这里拓宽的类型对于字符串字面量来说是 string, 对于数字字面量是 number, 对于 true 或 false 来说是 boolean, 对于枚举字面量类型则是对应的枚举类型.

例子
```js
const c1 = 1;  // 类型 1
const c2 = c1;  // 类型 1
const c3 = "abc";  // 类型 "abc"
const c4 = true;  // 类型 true
const c5 = cond ? 1 : "abc";  // 类型 1 | "abc"

let v1 = 1;  // 类型 number
let v2 = c2;  // 类型 number
let v3 = c3;  // 类型 string
let v4 = c4;  // 类型 boolean
let v5 = c5;  // 类型 number | string
```
字面量类型的拓宽可以通过显式的类型标注来控制. 具体来说, 当一个有字面量类型的表达式是通过常量位置而不是类型标注被推断时, 这个 const 变量被推断的是待拓宽的字面量类型. 但在 const 位置有显式的类型标注时, const 变量获得的是非待拓宽的字面量类型.

例子
```js
const c1 = "hello";  // 待拓宽类型 "hello"
let v1 = c1;  // 类型 string

const c2: "hello" = "hello";  // 类型 "hello"
let v2 = c2;  // 类型 "hello"
```
# 使用 super 的返回值作为 'this'

在 ES2015 中, 返回对象的构造函数会隐式地替换所有 super() 调用者的 this 的值. 这样一来, 捕获 super() 任何潜在的返回值并使用 this 替代则是必要的. 这一项改变使得我们可以配合自定义元素, 而它正是利用了这一特性来初始化浏览器分配, 却是由用户编写了构造函数的元素.

Example
```js
class Base {
    x: number;
    constructor() {
        // 返回一个不同于 `this` 的新对象
        return {
            x: 1,
        };
    }
}

class Derived extends Base {
    constructor() {
        super();
        this.x = 2;
    }
}
```
生成:
```js
var Derived = (function (_super) {
    __extends(Derived, _super);
    function Derived() {
        var _this = _super.call(this) || this;
        _this.x = 2;
        return _this;
    }
    return Derived;
}(Base));
```
这一改变会引起对像 Error, Array, Map 等等的内建类的扩展行为带来不兼容的变化. 请参照扩展内建类型不兼容变化文档了解详情.

# 配置继承

一个项目通常会有多个输出目标, 比如 ES5 和 ES2015, 编译和生产或 CommonJS 和 System; 在这些成对的目标中, 只有少数配置选项会改变, 而维护多个 tsconfig.json 文件可以会比较麻烦.

TypeScript 2.1 支持通过 extends 来继承配置, 在这儿:

extends 是 tsconfig.json 中一个新的顶级属性 (同级的还有 compilerOptions, files, include 和 exclude).
extends 的值必须为一个包含了到另一个被继承的配置文件的路径的字符串.
基文件的配置会先被加载, 然后被继承它的文件内的配置覆盖.
配置文件不允许出现循环.
继承文件中的 files, include 和 exclude 会覆盖被继承的配置文件中对应的值.
所有配置文件中出现的相对路径会相对这些路径所配置文件的路径来解析.
例子
configs/base.json:
```json
{
  "compilerOptions": {
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
```
tsconfig.json:
```json
{
  "extends": "./configs/base",
  "files": [
    "main.ts",
    "supplemental.ts"
  ]
}
```
tsconfig.nostrictnull.json:
```json
{
  "extends": "./tsconfig",
  "compilerOptions": {
    "strictNullChecks": false
  }
}
```
# 新 --alwaysStrict 选项

使用 --alwaysStrict 来启动编译器会使:

所有代码以严格模式进行解析.
在所有生成文件的顶部输出 "use strict"; 指令.
模块会自动以严格模式进行解析. 对于非模块代码推荐使用该新选项.