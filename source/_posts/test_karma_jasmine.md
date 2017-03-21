---
title: karma & jasmine自动化测试
category:  test
tags: test
---

前端包管理工具

代码重用和复用是快捷开发的一种重要方式，但是原始的代码模块散布于各个平台上，不好寻找，程序员对其进行有效管理也成为了一大难题。此时，依赖（包、插件、工具都可以称呼，本质是他人写好封装后的代码模块）管理工具应需而生。依赖管理工具使用简单的命令即可提供 依赖的查找、安装、卸载等操作，深受广大程序员喜爱。
前端 Node.js 最为常用的依赖管理工具 是npm，npm 之于Node.js，就如 pip 之于 Python，gem 之于 Ruby，pear 之于 PHP , maven 之于Java 。

<!--more-->

# Karma 环境的搭建

安装 karma (karma用于run自动化测试脚本)
```bash
npm install karma --save-dev
```
安装karma-jasmine (jasmine用于编写单元测试用例)
```bash
npm install karma-jasmine --save-dev
npm install jasmine-core --save-dev
```
安装karma-chrome-launcher(用于启动chrome浏览器;如果是firefox可以使用karma-firefox-launcher;同理可得其他)
```bash
npm install karma-chrome-launcher --save-dev
npm install karma-firefox-launcher --save-dev
```
安装coverage(测试代码覆盖率)
```bash
npm install karma-coverage --save-dev
```

# Jasmine

jasmine有四种类型的函数:

## 分组 describe
```js
// 声明一类测试用例
describe('add algorithm',function(){
    // 在里面可以定义一些变量，如
    var a=1,b=2;
});
```
## 用例 it
```js
// 声明一类测试用例
describe('add algorithm',function(){
    // 在里面可以定义一些变量，如
    var a=1,b=2;
    // 声明一种测试用例
    it('test add one',function(){

    });

    it('test add two',function(){

    });
});
```
## 期望 expect

## 匹配to****
```js
// 声明一类测试用例
describe('add algorithm',function(){
    // 可以定义一些变量，如
    var a=1,b=2;
    // 声明一种测试用例
    it('test add one',function(){
        // 期望 自定义的函数 addOne(1) 结果为 2, 反向读代码
        expect(2).toEqual(addOne(a));
        expect(3).toEqual(addOne(b));
    });

    it('test add two',function(){
        expect(3).toEqual(addTwo(a));
        expect(5).toEqual(addTwo(b));
    });
});
```
你可以在 github 或者 入门指导网站 了解到 jasmine 的详细信息

github地址: https://github.com/jasmine/jasmine

guide地址:  https://jasmine.github.io/2.0/introduction.html



# Karma 配置文件

读到这里，可能会有疑问：被测试函数 和 测试脚本应该放在哪里？

下面来看 karma 配置文件

在 karma.exe 所在目录下 或者 已将 karma 安装至 global

命令行输入（当然你也可以 命名为 **.conf.js）
```bash
karma init karma.conf.js
```
然后根据提示配置文件



配置 被测试代码路径 和 测试脚本路径 ( ** / * 通配 文件路径/名称)



省略省略省略…………

出现以下提示表示配置完成



如果想做一些个性化的处理，可以打开文件并 添加/修改 配置项
```js

/**
 * Created by lonelydawn on 2017-03-04.
 */

module.exports = function (config) {
    config.set({

        // base path that will be used to resolve all patterns (eg. files, exclude)
        basePath: '',


        // frameworks to use
        // available frameworks: https://npmjs.org/browse/keyword/karma-adapter
        frameworks: ['jasmine'],


        // list of files / patterns to load in the browser
        files: [
            'public/bower_components/angular/angular.js',
            'app/javascripts/**/*.js',
            'test/**/*.js'
        ],


        // list of files to exclude
        exclude: [],


        // preprocess matching files before serving them to the browser
        // available preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
        //代码覆盖率测试 ,使用 karma-coverage
        preprocessors: {
            'app/javascripts/**/*.js': 'coverage'
        },


        // test results reporter to use
        // possible values: 'dots', 'progress'
        // available reporters: https://npmjs.org/browse/keyword/karma-reporter
        reporters: ['progress','coverage'],
        coverageReporter: {
            type: 'html',
            dir: 'coverage/'
        },

        // web server port
        port: 9876,


        // enable / disable colors in the output (reporters and logs)
        colors: true,


        // level of logging
        // possible values: config.LOG_DISABLE || config.LOG_ERROR || config.LOG_WARN || config.LOG_INFO || config.LOG_DEBUG
        logLevel: config.LOG_INFO,


        // enable / disable watching file and executing tests whenever any file changes
        autoWatch: true,


        // start these browsers
        // available browser launchers: https://npmjs.org/browse/keyword/karma-launcher
        browsers: ['Chrome'],


        // Continuous Integration mode
        // if true, Karma captures browsers, runs the tests and exits
        singleRun: false

    })
};
```
之后 命令行执行 ,即可开始测试 ( 在配置和启动的时候一定要注意路径问题 )
```bash
karma start karma.conf.js
```

# Gulp下 karma 的使用

gulp 是一款非常简单好用的自动化构建工具，中文文档很详细。

gulp 中文文档地址 : http://www.gulpjs.com.cn/



在 gulp 中使用karma 不再需要安装 gulp-karma组件

github原文:

Karma integration into Gulp.js-based build.



将 Karma 配置到项目 node_modules中并将配置文件建好之后

在 gulpfile.js 中写入
```js
var gulp=require('gulp');
var Karma=require('karma').Server;

// 前端自动化测试
gulp.task('test', function (done) {
    new Karma({
        // 配置文件所在路径
        configFile: __dirname + '/karma.conf.js',
        // 执行测试结束后退出
        singleRun:true
    }, done).start();
});

gulp.task('tdd', function (done) {
    new Karma({
        configFile: __dirname + '/karma.conf.js'
    }, done).start();
});
```
之后在命令行键入
```bash
gulp test
```
或
```bash
gulp tdd
```
执行测试即可.