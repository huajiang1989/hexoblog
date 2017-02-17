
---
title: 使用gulp脚本配合TypeScript开发
category: TypeScript
tags: TypeScript
---

目标：编写TypeScript时，保存即生成js文件。


<!--more-->
# tsconfig.json

运行 tsc --init 生成配置文件

tsconfig.json

```json
{
    "compilerOptions": {
        "target": "ES5",
        "module": "commonjs",
        "sourceMap": true,
        "outDir": "dist",
        "emitDecoratorMetadata": true,
        "experimentalDecorators": true,
        "removeComments": false,
        "noImplicitAny": false
    },
    "includes": [
        "src/**/*"
    ]
}
```

# 编写gulpfiles.js


使用npm安装以下组件
gulp
gulp-rename
through-gulp
gulp-typescript

```js
var gulp = require('gulp');
var ts = require('gulp-typescript');
var sourcemaps = require('gulp-sourcemaps');
var nodemon = require('gulp-nodemon');

var tsProject = ts.createProject('tsconfig.json');

gulp.task('compile', function () {
    return tsProject.src()
        .pipe(sourcemaps.init())
        .pipe(tsProject())
        .pipe(sourcemaps.write())
        .pipe(gulp.dest('dist'));
});

gulp.task('default', ['compile'], function () {
    gulp.watch(["**/*.ts", "!**/node_modules/**"], ['compile']);
});
```

执行脚本时把所有的*.ts文件生成一次，然后检测到有修改时生成对应的js。