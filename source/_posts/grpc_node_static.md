---
title:   grpc之node调用静态代码
category: grpc
tags: grpc
---

# 创建项目文件夹，初始化项目
我的文件夹为grpctest
cd到项目文件下，运行 npm init

<!--more-->

# 安装protoc以及grpc插件
这个用npm直接安装
```
npm install grpc-tools --save-dev
npm install google-protobuf --save
npm install grpc --save
```
在./node_modules/grpc-tools/bin下，你会找到 protoc.exe 和 grpc_node_plugin.exe两个文件。
# 编写.proto文件并使用protoc 进行编译
HelloWorldService.proto
```js
syntax = "proto3";
package com.sdp.grpctest;
message HelloRequest{
  string name = 1;
}
message HelloResponse{
  string helloString = 1;
}
// 服务HelloWorld
service HelloWorldService{
  rpc hello (HelloRequest) returns (HelloResponse){}
}
```
# 运行编译命令
```
./node_modules/grpc-tools/bin/protoc --js_out=import_style=commonjs,binary:./ --plugin=protoc-gen-grpc=./node_modules/grpc-tools/bin/grpc_node_plugin.exe --grpc_out=./ HelloWorldService.proto
```
运行完成后，会生成HelloWorldService_grpc_pb.js 和 HelloWorldServer_pb.js两个文件。
# 编写server.js文件
server.js
```js
var services = require('./HelloWorldService_grpc_pb.js');
var messages = require('./HelloWorldService_pb.js');
var grpc = require('grpc')
var hello = function(call, callback) {
  var response = new messages.HelloResponse();
  response.setHellostring("hello," + call.request.getName());
  callback(null, response);
}
var server = new grpc.Server();
server.addService(
  services.HelloWorldServiceService,
  {
    hello:hello
  }
);
server.bind('0.0.0.0:50051', grpc.ServerCredentials.createInsecure());
server.start(function(err,data){
  console.log(err);
  console.log(data);
});
```
运行node server.js启动server

# 编写client.js文件
```js
var grpc = require('grpc');
var messages = require('./HelloWorldService_pb.js');
var services = require('./HelloWorldService_grpc_pb.js')
var request = new messages.HelloRequest();
request.setName('world');
var client = new services.HelloWorldServiceClient(
  'localhost:50051',
  grpc.credentials.createInsecure()
);
client.hello(request, function(err,data){
  if(err){
    console.error(err);
  }
  console.log(data);
  console.log(data.getHellostring());
})
```
运行node client.js ， 最后成功打印出hello,world