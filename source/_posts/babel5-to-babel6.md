---
title:  babel5升级到babel6总结
category: es6
tags: es6
---
babel5 to babel6

为什么要升级？
---
性能提升：据说compile速度提升20%（但在测试后发现，速度慢了20%+）。
可配置的插件：更强的灵活性，以及更简单的插件API。
更简洁的配置。
<!--more-->
重大变化
---
不再提供babel包
命令行工具由babel-cli包提供
node api由babel-core提供
polyfill由babel-polyfill提供
.babelrc配置变化
移除“blacklist”、“whitelist”、“optional”、“nonStandard”和“modules”等
增加plugins：转码逻辑通过一系列插件实现，默认不使用任何插件
增加presets配置，提供一些预设的插件集合
external-helper选项由插件实现
项目升级过程中遇到问题

babel官方未提供decorator transfrom插件

decorator是es7的提案，如果要是用的话，需要用用非官方插件：transform-decorators-legacy。

babel提供的es2015插件集默认包含transform-es2015-modules-commonjs插件

项目系统需要将es6模块转成AMD模块，需要在plugins中指定使用transform-es2015-modules-amd.

babel提供的AMD模块转码插件默认开启strict mode

关闭strict mode的方法：

不使用preset-es2015，需要自定义preset或者在plugins中引入所有需要的插件;
插件中做如下配置：
```js
{
    "plugins": [
        ["transform-es2015-modules-amd", {"strict": false}]
    ]
}
```
Notes：如果不提供es2015相关插件的preset，preset-stage-x也不能使用
备注下：preset是一系列plugin 的集合，而stage-x代表着支持es6哪个阶段的语法。
```js
stage-0 - Strawman: just an idea, possible Babel plugin.
stage-1 - Proposal: this is worth working on.
stage-2 - Draft: initial spec.
stage-3 - Candidate: complete spec and initial browser implementations.
stage-4 - Finished: will be added to the next yearly release.
```
es6代码中有require时会导致转码失败

es6代码中出现require时，babel5能正确转码，但babel6会报如下错误：

Property object of MemberExpression expected node to be of a type ["Expression"] but instead got null.
解决方法：讲require变更为import。

windows7下会出现Couldn’t find preset

当安装babel插件的是时候，在windows7环境上不能使用如下命令来安装，会报”Couldn’t find preset “es2015” relative to directory”。

npm install --save-dev babel-preset-es2015
解决方法：升级node版本至5+，npm3+或者安装preset安装的时候，使用

npm install --save babel-preset-es2015
原因是不同的操作系统cache策略不同。

Cache files are stored in ~/.npm on Posix, or ~/npm-cache on Windows.
This is controlled by the cache configuration param.

使用babel-core提供的api转码时，如果以对象形式传入配置，转码后AMD模块会有两层define

解决方法：传入配置中需要增加babelrc=false。

原因是：构建过程中，babel自身会去基于.babelrc来进行构建，如果我们不想让babel执行这个文件需要在调试的时候将babelrc设置为false，在构建的时候还是需要使用默认的true来调用和执行。

ES6 module to AMD的转码逻辑变更
--
babel6提供的AMD转码插件对export default的转码逻辑做了修改，详情如下：
针对如下代码：
```js
export default {
    foo: 1
};
```
使用babel5转码后：
```js
define(["exports", "module"], function (exports, module) {
    module.exports = {
        foo: 1
    };;
});
```
使用babel6转码后：
```js
define(["exports"], function (exports) {
    Object.defineProperty(exports, "__esModule", {
        value: true
    });
    exports.default = {
        foo: 1
    };
});
```
在AMD模块依赖es6模块的场景下，AMD模块代码会不能运行。

备注：
主要的大坑就是项目里面既有es5的代码也有es6的代码，然后在5里面require es6文件就会出问题，会多一层未经处理的default….解决核心就是把这层default给干掉。

结论
---
升级最大的阻碍是：ES6 module to AMD的转码逻辑变更并且暂时没有什么好的办法解决这个问题，所以暂不升级。

IoC支持插件后，可以考虑实施方法2解决模块转码问题，相比方法1代价更小。

总体来说方法3代价最小，但babel5的转码存在问题是：导致不符合es6模块使用规范的代码出现。如下：
```js
// foo.js
const foo = {baz: 42, bar: false}
export default foo
// bar.js
import {baz} from './foo'
```
项目可用的BABEL6 转码配置
---
不考虑ES6 module to AMD的转码逻辑变更问题，通过一下配置可以在项目中使用babel6

.babelrc
```js
{
    "compact": false,
    "ast": false,
    "sourceMaps": "inline",
    "highlightCode": true,
    "plugins": [
        ["transform-es2015-modules-amd", {"strict": false}],
        "transform-decorators-legacy"
    ],
    "presets": ["es2015", "stage-0", "stage-3"],
    "ignore": [
        "biz/report/**",
        "dsp-base/**/biz/tool/keywords/**"
    ]
}
```
package.json
```js
"dependencies": {
    "babel-plugin-transform-decorators-legacy": "^1.3.4"
    "babel-preset-es2015": "^6.6.0",
    "babel-preset-stage-0": "^6.5.0"
  }
```