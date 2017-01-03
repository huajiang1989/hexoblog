---
title: 使用gulp对Hexo性能优化篇
category: hexo
tags: hexo
date: 2016-11-01
---



hexo项目生成的public文件可以使用gulp进行压缩优化
<!--more-->
----------
静态文件压缩
-------------

静态文件包括: html,css,js,images . 我才用了gulp来跑自动压缩任务 。具体方法如下:

 1. npm 安装如下工具, 方法皆为 : npm install xxx –save
```bash
"gulp": "^3.9.1",
"gulp-htmlclean": "^2.7.6",
"gulp-htmlmin": "^1.3.0",
"gulp-imagemin": "^2.4.0",
"gulp-minify-css": "^1.2.4",
"gulp-uglify": "^1.5.3",
```
 2. 建立 gulpfile.js 文件
```js
var gulp = require('gulp');
var minifycss = require('gulp-minify-css');
var uglify = require('gulp-uglify');
var htmlmin = require('gulp-htmlmin');
var htmlclean = require('gulp-htmlclean');
// 获取 gulp-imagemin 模块
var imagemin = require('gulp-imagemin')
// 压缩 public 目录 css
gulp.task('minify-css', function() {
    return gulp.src('./public/**/*.css')
        .pipe(minifycss())
        .pipe(gulp.dest('./public'));
});
// 压缩 public 目录 html
gulp.task('minify-html', function() {
  return gulp.src('./public/**/*.html')
    .pipe(htmlclean())
    .pipe(htmlmin({
         removeComments: true,
         minifyJS: true,
         minifyCSS: true,
         minifyURLs: true,
    }))
    .pipe(gulp.dest('./public'))
});
// 压缩 public/js 目录 js
gulp.task('minify-js', function() {
    return gulp.src('./public/**/*.js')
        .pipe(uglify())
        .pipe(gulp.dest('./public'));
});
// 压缩图片任务
// 在命令行输入 gulp images 启动此任务
gulp.task('images', function () {
    // 1. 找到图片
    gulp.src('./photos/*.*')
    // 2. 压缩图片
        .pipe(imagemin({
            progressive: true
        }))
    // 3. 另存图片
        .pipe(gulp.dest('dist/images'))
});
// 执行 gulp 命令时执行的任务
gulp.task('default', [
    'minify-html','minify-css','minify-js','images'
]);
```
注意， 修改上面的各个目录为你的真实目录， ** 代表0或多个子目录
