
---
title:  gulp+browserSync+nodemon 实现express 全端自动刷新
category:  gulp
tags: gulp
---

# 技术细节
整个项目的技术栈是 jade+node-sass+js，实现全自动开发。首先在 views 下面，定义好 jade 模版文件，然后开始新加页面
<!--more-->

# 关系依赖
这个环境，主要基于 Express,Gulp 以及一系列的 Gulp 插件搭建，这个是开发环境的依赖关系：
```js
{
  "dependencies": {
    "body-parser": "~1.13.2",
    "cookie-parser": "~1.3.5",
    "debug": "~2.2.0",
    "express": "~4.13.1",
    "jade": "~1.11.0",
    "morgan": "~1.6.1",
    "serve-favicon": "~2.3.0"
  },
  "devDependencies": {
    "browser-sync": "^2.9.1",
    "del": "^2.0.2",
    "gulp": "^3.9.0",
    "gulp-autoprefixer": "^3.0.1",
    "gulp-jade": "^1.1.0",
    "gulp-nodemon": "^2.0.4",
    "gulp-sass": "^2.0.4",
    "gulp-sourcemaps": "^1.5.2",
    "jade": "^1.11.0"
  }
}
```
其中，express 主要用来渲染 jade 模版引擎以及提供路由功能，同时起一个本地的服务器。有人说，渲染 jade 模版，我直接用 gulp 也可以啊。是的，gulp 是可以直接渲染 jade 模版，但是有一个问题，就是如果你本地只修改了一个 jade 文件时候，gulp 也需要把所有的 jade 文件全部渲染一遍，这明显会浪费很多时间。

所以我引入了 express，只会按需渲染 jade，节省了很多时间。同时一个高效的开发环境必须有浏览器自动刷新，但是 express 没有这个功能。所以我引入了 browser-sync，用它来代理 express 启动的本地服务，然后来监测本地文件的变动，来做到浏览器自动刷新。

有同学可能会问了，如果我改了 express 相应的 js 文件怎么办，或者 jade 文件编译出错了怎么办。因为这些时候 express 服务器都会 crash 掉，需要重启。所以我引入了 nodemon，来做 express 服务器的自动重启功能。

在这个环境下面，我在开发中所能碰到的所有 jade，scss，js 文件的改动都会被检测到并作出相应的动作，而且速度还非常快。让我可以花更多时间在业务代码的开发上，而不是不断重启服务和刷新浏览器。

# 最后

同时附上 gulpfile 全文，以供参考：
```js
'use strict';

var gulp = require('gulp');
var browserSync = require('browser-sync');
var reload = browserSync.reload;
var sass = require('gulp-sass');
var prefix = require('gulp-autoprefixer');
var nodemon = require('gulp-nodemon');
var sourcemaps = require('gulp-sourcemaps');
var jade = require('gulp-jade');
var stylus = require('gulp-stylus');
var rename = require('gulp-rename');
var del = require('del');

//dev task start
//DONE can not compile the sass or less file
gulp.task('sass', function () {
  return gulp.src(['./sass/personal.scss'])
    .pipe(sourcemaps.init())
    .pipe(sass({errLogToConsole: true})
      .on('error', sass.logError))
    .pipe(prefix('last 2 versions', '> 1%', 'ie 8', 'Android 2'))
    .pipe(sourcemaps.write())
    .pipe(gulp.dest('./public/css'))
    .pipe(reload({stream: true}));
});

gulp.task('browser-sync', ['nodemon'], function () {
  browserSync.init(null, {
    proxy: 'http://localhost:3000',
    files: ['public/**/*.*', 'views/**/*.*', 'submodule/**/*.*'],
    browser: 'google chrome',
    notify: false,
    port: 5000
  });
});

gulp.task('movesub', function () {
  return gulp.src(['./submodule/images/**/*.*'], {base: './submodule'})
    .pipe(gulp.dest('./public'))
});

gulp.task('stylus', function () {
  return gulp.src('submodule/stylus/public.styl')
    .pipe(stylus())
    .pipe(rename({
      extname: '.scss'
    }))
    .pipe(gulp.dest('submodule/stylus/'))
});

gulp.task('nodemon', function (cb) {
  del(['./public/*.html']);

  var called = false;

  return nodemon({
    script: 'bin/www'
  }).on('start', function () {
    if (!called) {
      cb();
      called = true;
    }
  });
});
//dev task end

gulp.task('clean', function (cb) {
  del(['./dist/*'], cb)
});

gulp.task('copy', function () {
  return gulp.src([
    'public/css/**/*',
    'public/images/**/*',
    'public/js/**/*',
    'public/pageScripts/**/*'
  ], {base: './public'})
  .pipe(gulp.dest('./dist'))
});

//build task start
//DONE add build task
gulp.task('jade', function () {
  return gulp.src(['views/**/*.jade', '!views/layout/**/*.jade', '!views/includes/**/*.jade'])
    .pipe(jade({pretty: true}))
    .pipe(gulp.dest('./dist'));
});
//build task end

gulp.task('dist', ['clean', 'copy', 'jade']);

gulp.task('default', ['browser-sync', 'sass', 'movesub'], function () {
  gulp.watch(['sass/**/*.*', '.submodule/stylus/**/*.*'], ['sass']);
});

```