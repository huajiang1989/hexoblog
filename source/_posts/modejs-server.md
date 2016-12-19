
---
title:  使用Mock.js进行独立于后端的前端开发
category:  mock
tags: mock
---
# Mock.js实现的功能
基于 数据模板 生成数据
基于 HTML模板 生成数据
拦截并模拟 Ajax请求
<!--more-->
# 用法
## 浏览器：
```js
<!-- （必选）加载 Mock -->
<script src="http://mockjs.com/dist/mock.js"></script>
<script>
// 使用 Mock
var data = Mock.mock({
    'list|1-10': [{
        'id|+1': 1
    }]
});
$('<pre>').text(JSON.stringify(data, null, 4))
.appendTo('body')
</script>
```
返回值:
```js
{
"list": [
    {
        "id": 1
    },
    {
        "id": 2
    },
    {
        "id": 3
    }
    ]
}
```
## JQuery：
配置模拟数据：
```js
Mock.mock('http://g.cn', {
    'name'     : '@name',
    'age|1-100': 100,
    'color'    : '@color'
});
发送Ajax请求：

$.ajax({
    url: 'http://g.cn',
    dataType:'json'
    }).done(function(data, status, xhr){
    console.log(
    JSON.stringify(data, null, 4)
    )
})；
```
返回数据：
```js
// 结果1
{
"name": "Elizabeth Hall",
"age": 91,
"color": "#0e64ea"
}

// 结果2
{
"name": "Michael Taylor",
"age": 61,
"color": "#081086"
}
```
## Node.js：
```bash
// 安装
npm install mockjs
```

```js
// 使用
var Mock = require('mockjs');
var data = Mock.mock({
    'list|1-10': [{
        'id|+1': 1
    }]
});

console.log(JSON.stringify(data, null, 4))
```

## Angular.js:
```js
<!-- 引用 -->
<script src="http://mockjs.com/dist/mock-min.js"></script>
<script src="http://cdn.staticfile.org/angular.js/1.3.0-beta.13/angular.min.js"></script>

<!-- 兼容angular (mock.js默认不兼容angular，需额外引用兼容包)-->
<script src="./src/mock.angular.js"></script>
<!-- 模拟数据 -->
<script src="./mockData.js"></script>
<!-- 使用 -->
<script>
(function() {
    (function() {
        var app;
        app = angular.module('app', []);
        //使用mockjax方法覆盖Ajax请求
        Mock.mockjax(app);
        return app.controller('appCtrl', function($scope, $http) {
            var box;
            box = $scope.box = [];
            $scope.get = function() {
                $http({
                    url: 'http://www.baidu.com',
                    method: 'POST',
                    params: {a: 1},
                    data  : {b:1}
                }).success(function(data) {
                return box.push(data);
            });

            $http({
                url: 'http://baidu.com'
                }).success(function(data) {
                console.log(data);
                });
            };
        return $scope.get();
        });
    })();

}).call(this);
</script>
```

mock数据mockData.js:
```js
Mock.mock('http://www.baidu.com', {
    'name': '@name()',
    'age|1-100': 100,
    'color': '@color'
});
```
# 语法
Mock.js 的语法规范包括两部分：
```bash
#数据模板定义（Data Temaplte Definition，DTD）
#数据占位符定义（Data Placeholder Definition，DPD）
```
## 数据模板定义 DTD
数据模板中的每个属性由 3 部分构成：属性名、生成规则、属性值：
```js
// 属性名   name
// 生成规则 rule
// 属性值   value
'name|rule': value
```
注意：

属性名 和 生成规则 之间用 | 分隔。
生成规则 是可选的。
生成规则 有 7 种格式：
```js
'name|min-max': value
'name|count': value
'name|min-max.dmin-dmax': value
'name|min-max.dcount': value
'name|count.dmin-dmax': value
'name|count.dcount': value
'name|+step': value
```
生成规则 的 含义 需要依赖 属性值 才能确定。
属性值 中可以含有 @占位符。
属性值 还指定了最终值的初始值和类型。
生成规则和示例：

### 1. 属性值是字符串 String
```js
'name|min-max': 'value' 通过重复 'value' 生成一个字符串，重复次数大于等于 min，小于等于 max。
'name|count': 'value' 通过重复 'value' 生成一个字符串，重复次数等于 count。
```
### 2. 属性值是数字 Number
```js
'name|+1': 100 属性值自动加 1，初始值为 100
'name|1-100': 100 生成一个大于等于 1、小于等于 100 的整数，属性值 100 只用来确定类型。
'name|1-100.1-10': 100 生成一个浮点数，整数部分大于等于 1、小于等于 100，小数部分保留 1 到 10 位。
    {
    'number1|1-100.1-10': 1,
    'number2|123.1-10': 1,
    'number3|123.3': 1,
    'number4|123.10': 1.123
    }
    // =>
    {
    "number1": 12.92,
    "number2": 123.51,
    "number3": 123.777,
    "number4": 123.1231091814
    }
```
### 3. 属性值是布尔型 Boolean
```js
'name|1': value 随机生成一个布尔值，值为 true 的概率是 1/2，值为 false 的概率是 1/2。
'name|min-max': value 随机生成一个布尔值，值为 value 的概率是 min / (min + max)，值为 !value 的概率是 max / (min + max)。
```
### 4. 属性值是对象 Object
```js
'name|min-max': {} 从属性值 {} 中随机选取 min 到 max 个属性。
'name|count': {} 从属性值 {} 中随机选取 count 个属性。
```
### 5. 属性值是数组 Array
```js
'name|1': [{}, {} ...] 从属性值 [{}, {} ...] 中随机选取 1 个元素，作为最终值。
'name|min-max': [{}, {} ...] 通过重复属性值 [{}, {} ...] 生成一个新数组，重复次数大于等于 min，小于等于 max。
'name|count': [{}, {} ...] 通过重复属性值 [{}, {} ...] 生成一个新数组，重复次数为 count。
```

### 6. 属性值是数组 Function
```js
'name': function(){} 执行函数 function(){}，取其返回值作为最终的属性值，上下文为 'name' 所在的对象。
```

## 数据占位符定义 DPD
占位符 只是在属性值字符串中占个位置，并不出现在最终的属性值中。占位符 的格式为：
```js
@占位符
@占位符(参数 [, 参数])
```
注意：

用 @ 来标识其后的字符串是 占位符。
占位符 引用的是 Mock.Random 中的方法。
通过 Mock.Random.extend() 来扩展自定义占位符。
占位符 也可以引用 数据模板 中的属性。
占位符 会优先引用 数据模板 中的属性
```js
{
 name: {
 first: '@FIRST',
 middle: '@FIRST',
 last: '@LAST',
 full: '@first @middle @last'
    }
}
// =>
{
 "name": {
 "first": "Charles",
 "middle": "Brenda",
 "last": "Lopez",
 "full": "Charles Brenda Lopez"
    }
}
```
# 常用方法
## Mock.mock( rurl?, rtype?, template|function(options) )
根据数据模板生成模拟数据。

参数的含义和默认值如下所示：
```bash
参数 rurl：#可选。表示需要拦截的 URL，可以是 URL 字符串或 URL 正则。例如 /\/domain\/list.json/、'/domian/list.json'。
参数 rtype：#可选。表示需要拦截的 Ajax 请求类型。例如 GET、POST、PUT、DELETE 等。
参数 template：#可选。表示数据模板，可以是对象或字符串。例如 { 'data|1-10':[{}] }、'@EMAIL'。
参数 function(options)：#可选。表示用于生成响应数据的函数。
参数 options：#指向本次请求的 Ajax 选项集。
```
## Mock.mockjax(library)
覆盖（拦截） Ajax 请求，目前内置支持 jQuery、Zepto、KISSY。

## Mock.Random
Mock.Random 是一个工具类，用于生成各种随机数据。Mock.Random 的方法在数据模板中称为“占位符”，引用格式为 @占位符(参数 [, 参数]) 。

## Mock.tpl(input, options, helpers, partials)
基于 Handlebars、Mustache 的 HTML 模板生成模拟数据。