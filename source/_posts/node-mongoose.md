---
title: Nodejs基于mongoose模块的增删改查
category: nodejs
tags: nodejs
---
安装模块mongoose

```
npm install mongoose
```
[注意] mongoose模块依赖mongodb
<!--more-->
npm常用命令
--
```
npm install <name> -g      #将包安装到全局环境中

npm install <name> –-save  #安装的同时，将信息写入package.json中,便于后期维护查看

npm remove <name>          #移除

npm update <name>          #更新

npm root -g                #查看全局的包安装路径

npm -v                     #查看npm的版本
```
开启mongodb数据库
--
进入mongod所在目录 执行命令 ./mongod --dbpath=存放数据的位置
```
例1：./mongod --dbpath=../data/dbname
例2：./mongod --dbpath=../data/dbname --port 自定义端口号，默认27017(了解即可，不推荐使用，修改默认端口号后期维护麻烦)
```
插入数据
--
```js
// 引入模块
var mongoose = require('mongoose');
// 连接数据库
var db = mongoose.createConnection('mongodb://127.0.0.1:27017/test');
// 设置数据类型
var monSchema = new mongooose.Schema({
    name:{type:String,default:"username"},
    age：{type:Number},
    sex:{type:String}
});
// 选择集合
var monModel = db.model('user',monSchema);
// 数据集
var content = {name:"Nick",age:23,sex:'男'};
// 实例化对象并插入数据
var monInsert = new monModel(content);
monInsert.save(function(err){
  if(err){
    console.log(err);
  }else{
    console.log('成功插入数据');
  }
  db.close();
});
```
删除数据
--
```js
// 引入模块
var mongoose = require('mongoose');
// 连接数据库
var db = mongoose.createConnection('mongodb://127.0.0.1:27017/test');
// 设置数据类型
var monSchema = new mongooose.Schema({
    name:{type:String,default:"name"},
    age：{type:Number},
    sex:{type:String}
});
// 选择集合
var monModel = db.model('user',monSchema);
// 要删除的条件
var del  = {name:"Nick"};

monModel.remove(del,function(err,result){
  if(err){
    console.log(err);
  }else{
    console.log("update");
  }
  db.close();
});
```
修改数据
--
```js
// 引入模块
var mongoose = require('mongoose');
// 连接数据库
var db = mongoose.createConnection('mongodb://127.0.0.1:27017/test');
// cosole.log(db);
// 设置数据类型
var monSchema = new mongooose.Schema({
    name:{type:String,default:"name"},
    age：{type:Number},
    sex:{type:String}
});
// 选择集合
var monModel = db.model('user',monSchema);
// 原数据字段值
var oldValue  = {name:"Nick"};
// 单条件更新
var newData1 = {$set:{name:"内容"}};
// 多条件更新
var newData2 = {$set:{name:"内容",age:2}};
monModel.update(oldValue,newData,function(err,result){
  if(err){
    console.log(err);
  }else{
    console.log("update");
  }
  db.close();
});
```
查询数据
--
```js

// 引入模块
var mongoose = require('mongoose');
// 连接数据库
var db = mongoose.createConnection('mongodb://127.0.0.1:27017/test');
// cosole.log(db);
// 设置数据类型
var monSchema = new mongooose.Schema({
    name:{type:String,default:"name"},
    age：{type:Number},
    sex:{type:String}
});
// 选择集合
var monModel = db.model('user',monSchema);
var content = {name:"姓名2"};
var field = {name:1,age:1,sex:1};
monModel.find(content,field,function(err,result){
  if(err){
    console.log(err);
  }else{
    console.log(result);
  }
  db.close();
});
```