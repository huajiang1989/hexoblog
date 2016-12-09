---
title: 使用r.js进行简单的js/css压缩
category: r.js
tags: r.js
---
最近用require写了一个网站的模块,到压缩的时候,各种查资料学习,但由于时间较紧,将所有代码都压缩成一个文件,导致代码的体积很大,今天抽时间网上参考下官网的说明配置,将这次压缩代码的配置及运行命令记录下来,以后留着用.
<!--more-->
 js的压缩
--

这次只是初步应用,如果有好的方法或有误的地方,还望各位大侠们指教.

第一种配置的文件: 需要把所有依赖的文件都压缩到当前代码中.
```js
({
   baseUrl: "./",
   paths: {
       amd_modules: "../amd_modules",
       lib : "../lib"
   },
   name: "main",
   optimizeCss: "none",
   out: "index-built.js"
})
```
　　baseUrl: 指当前配置文件的路径

　　paths: 配置一些下面配置时需要的路径

　　name: 需要被压缩的文件

　　optimizeCss: none不压缩, standard标准压缩

　　out: 输出的文件

第二种配置的文件,去掉常用的库(单引用)

```js
({
    mainConfigFile : "main.js",
    baseUrl: "./",
    paths : {
        amd_modules: "../amd_modules" ,
        lib: "../lib"
    },
    dir: "built",
    keepBuildDir: false,  //不复制依赖文件

    modules: [
        {
            name: "main",
            exclude: ["amd_modules/jquery/1.8.3/jquery", "lib/header/0.0.1/header"]
        }
    ]
})
```
　　dir: 输出目录的路径

　　modules: 压缩合并的模块,exclude: 筛选掉的文件,不被压缩到main的压缩文件里

　　执行代码:
```
node r.js -o config.js
```

css的压缩
--
第一种情况,直接压缩某个文件,执行代码
```
node r.js -o cssIn=index.css out=built/index.css
```
第二种情况,压缩多个文件

需要先将要压缩的文件放到一个css中,用@import引入,如下所示:存储文件为main.css
```css
@import url("icons.css");
@import url("window.css");
@import url("tabs.css");
@import url("index.css");
```
执行代码:(标准压缩)
```
node r.js -o cssIn=main.css out=built/main.css optimizeCss=standard
```