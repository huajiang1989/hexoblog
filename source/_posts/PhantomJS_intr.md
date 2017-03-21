---
title:  PhantomJS入门
category:  test
tags: test
---

PhantomJS 是一个基于 WebKit 的服务器端 JavaScript API。它全面支持web而不需浏览器支持，其快速，原生支持各种Web标准： DOM 处理, CSS 选择器, JSON, Canvas, 和 SVG。 PhantomJS 可以用于 页面自动化 ， 网络监测 ， 网页截屏 ，以及 无界面测试 等。
<!--more-->
# 安装
安装包下载地址： http://phantomjs.org/download.html ，包括 Windows ，Mac OS，Linux版本，自行选择对应 版本下载解压即可（ 为方便使用，可自已为phantomjs设置环境变量 ），其中带有一个example文件夹，里面有很多已经写好的代码供使用。本文假设phantomjs已经安装好并已设置了环境变量。

# 使用
Hello， World！
新建一个包含下面两行脚本的文本文件：
```js
console.log('Hello, world!');
phantom.exit();
```
将文件另存为 hello.js ,然后执行它：
```js
phantomjs hello.js
```
输出结果为：Hello, world!
第一行将会在终端打印出字符串，第二行 phantom.exit 将退出运行。
在该脚本中调用 phantom.exit 是非常重要的，否则 PhantomJS 将根本不会停止。

# 脚本参数 – Script Arguments
Phantomjs如何传递参数呢？如下所示 ：
```js
phantomjs examples/arguments.js foo bar baz
```
其中的foo, bar, baz就是要传递的参数，如何获取呢：
```js
var system = require('system');
if (system.args.length === 1) {
    console.log('Try to pass some args when invoking this script!');
} else {
    system.args.forEach(function (arg, i) {
            console.log(i + ': ' + arg);
    });
}
phantom.exit();
```
它将输出 ：
```bash
0: foo
1: bar
2: baz
```
# 页面加载 – Page Loading
通过创建一个网页对象，一个网页可以被加载，分析和渲染。

下面的脚本将示例页面对象最简单的用法，它加载 example.com 并且将它保存为一张图片， example.png 。
```js
var page = require('webpage').create();
page.open('http://example.com', function () {
    page.render('example.png');
    phantom.exit();
});
```
由于它的这个特性，PhantomJS 可以用来 网页截屏 ，截取一些内容的快照，比如将网页、SVG存成图片，PDF等，这个功能很牛X。

接下来的 loadspeed.js 脚本加载一个特殊的URL (不要忘了http协议) 并且计量加载该页面的时间。
```js
var page = require('webpage').create(),
    system = require('system'),
    t, address;

if (system.args.length === 1) {
    console.log('Usage: loadspeed.js <some URL>');
    phantom.exit();
}

t = Date.now();
address = system.args[1];
page.open(address, function (status) {
    if (status !== 'success') {
        console.log('FAIL to load the address');
    } else {
        t = Date.now() - t;
        console.log('Loading time ' + t + ' msec');
    }
    phantom.exit();
});
```
在命令行运行该脚本：
```js
phantomjs loadspeed.js http://www.google.com
```
它输出像下面的东西：
```js
Loading  http://www.google.com  Loading time 719 msec
```
# 代码运算 – Code Evaluation
要想在网页的上下文中对JavaScript 或 CoffeeScript 进行运算，使用 evaluate() 方法。代码是在“沙箱”中运行的，它没有办法读取在其所属页面上下文之外的任何JavaScript对象和变量。 evaluate() 会返回一个对象，然而它仅限制于简单的对象并且不能包含方法或闭包。

这有一个示例来显示网页标题：
```js
var page = require('webpage').create();
page.open(url, function (status) {
    var title = page.evaluate(function () {
        return document.title;
    });
    console.log('Page title is ' + title);
});
```
任何来自于网页并且包括来自 evaluate() 内部代码的控制台信息，默认不会显示的。要重写这个行为，使用 onConsoleMessage 回调函数，前一个示例可以被改写成：
```js
var page = require('webpage').create();
page.onConsoleMessage = function (msg) {
    console.log('Page title is ' + msg);
};
page.open(url, function (status) {
    page.evaluate(function () {
        console.log(document.title);
    });
});
```
# DOM操作 – DOM Manipulation
由于脚本好像是一个Web浏览器上运行的一样，标准的DOM脚本和CSS选择器可以很好的工作。这使得PhantomJS适合支持各种 页面自动化任务 。

下面的  useragent.js  将读取 id  为myagent的元素的  textContent  属性：
```js
var page = require('webpage').create();
console.log('The default user agent is ' + page.settings.userAgent);
page.settings.userAgent = 'SpecialAgent';
page.open('http://www.httpuseragent.org', function (status) {
    if (status !== 'success') {
        console.log('Unable to access network');
    } else {
        var ua = page.evaluate(function () {
            return document.getElementById('myagent').textContent;
        });
        console.log(ua);
    }
    phantom.exit();
});
```
上面示例同样提供了一种自定义 user agent 的方法。

使用JQuery及其他类库：
```js
var page = require('webpage').create();
page.open('http://www.sample.com', function() {
    page.includeJs("http://ajax.googleapis.com/ajax/libs/jquery/1.6.1/jquery.min.js", function() {
        page.evaluate(function() {
            $("button").click();
        });
        phantom.exit()
    });
});
```
# 网络请求及响应 – Network Requests and Responses
将一个页面从一台远程服务器请求一个资源的时候，请求和响应均可以通过 onResourceRequested  和  onResourceReceived  回调方法追踪到。示例  netlog.js ：
```js
var page = require('webpage').create();
page.onResourceRequested = function (request) {
    console.log('Request ' + JSON.stringify(request, undefined, 4));
};
page.onResourceReceived = function (response) {
    console.log('Receive ' + JSON.stringify(response, undefined, 4));
};
page.open(url);
```
获取如何把该特性用于HAR 输出以及基于YSlow的性能分析的更多信息，请参阅 网络监控页面 。

PhantomJs官网： http://phantomjs.org/

GitHub: https://github.com/ariya/phantomjs/wiki/Quick-Start