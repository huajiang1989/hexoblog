
---
title:  vue-resource使用
category:  vue
tags: vue
---

# vue-resource特点

vue-resource插件具有以下特点：

## 体积小

vue-resource非常小巧，在压缩以后只有大约12KB，服务端启用gzip压缩后只有4.5KB大小，这远比jQuery的体积要小得多。
<!--more-->
## 支持主流的浏览器

和Vue.js一样，vue-resource除了不支持IE 9以下的浏览器，其他主流的浏览器都支持。


## 支持Promise API和URI Templates

Promise是ES6的特性，Promise的中文含义为“先知”，Promise对象用于异步计算。
URI Templates表示URI模板，有些类似于ASP.NET MVC的路由模板。

## 支持拦截器

拦截器是全局的，拦截器可以在请求发送前和发送请求后做一些处理。
拦截器在一些场景下会非常有用，比如请求发送前在headers中设置access_token，或者在请求失败时，提供共通的处理方式。

# vue-resource使用

## 引入vue-resource

```js
<script src="js/vue.js"></script>
<script src="js/vue-resource.js"></script>
```
## 基本语法

引入vue-resource后，可以基于全局的Vue对象使用http，也可以基于某个Vue实例使用http。
```js
// 基于全局Vue对象使用http
Vue.http.get('/someUrl', [options]).then(successCallback, errorCallback);
Vue.http.post('/someUrl', [body], [options]).then(successCallback, errorCallback);

// 在一个Vue实例内使用$http
this.$http.get('/someUrl', [options]).then(successCallback, errorCallback);
this.$http.post('/someUrl', [body], [options]).then(successCallback, errorCallback);
```
在发送请求后，使用then方法来处理响应结果，then方法有两个参数，第一个参数是响应成功时的回调函数，第二个参数是响应失败时的回调函数。

then方法的回调函数也有两种写法，第一种是传统的函数写法，第二种是更为简洁的ES 6的Lambda写法：
```js
// 传统写法
this.$http.get('/someUrl', [options]).then(function(response){
    // 响应成功回调
}, function(response){
    // 响应错误回调
});


// Lambda写法
this.$http.get('/someUrl', [options]).then((response) => {
    // 响应成功回调
}, (response) => {
    // 响应错误回调
});
```
PS：做过.NET开发的人想必对Lambda写法有一种熟悉的感觉。

## 支持的HTTP方法
vue-resource的请求API是按照REST风格设计的，它提供了7种请求API：
```bash
get(url, [options])
head(url, [options])
delete(url, [options])
jsonp(url, [options])
post(url, [body], [options])
put(url, [body], [options])
patch(url, [body], [options])
```
除了jsonp以外，另外6种的API名称是标准的HTTP方法。当服务端使用REST API时，客户端的编码风格和服务端的编码风格近乎一致，这可以减少前端和后端开发人员的沟通成本。

客户端请求方法	服务端处理方法
```js
this.$http.get(...)     //Getxxx
this.$http.post(...)	//Postxxx
this.$http.put(...)	    //Putxxx
this.$http.delete(...)	//Deletexxx
```
## options对象

发送请求时的options选项对象包含以下属性：

|参数	|类型	|描述|
| ------------- |:-------------:| -----:|
|url	|string|	请求的URL|
|method	|string|	请求的HTTP方法，例如：'GET', 'POST'或其他HTTP方法|
|body	|Object, FormDatastring	|request body|
|params	|Object	|请求的URL参数对象|
|headers|	Object|	request header|
|timeout|	number|	单位为毫秒的请求超时时间 (0 表示无超时时间)|
|before	|function(request)|	请求发送前的处理函数，类似于jQuery的beforeSend函数|
|progress|	function(event)|	ProgressEvent回调处理函数|
|credientials	|boolean|	表示跨域请求时是否需要使用凭证|
|emulateHTTP	|boolean|	发送PUT, PATCH, DELETE请求时以HTTP POST的方式发送，并设置请求头的X-HTTP-Method-Override|
|emulateJSON	|boolean|	将request body以application/x-www-form-urlencoded content type发送|

## response对象

response对象包含以下属性：

| 方法	| 类型	| 描述|
| ------------- |:-------------:| -----:|
| text()| 	string	| 以string形式返回response body|
| json()	| Object	| 以JSON对象形式返回response body|
| blob()| 	Blob| 	以二进制形式返回response body|

| 属性	| 类型	| 描述|
| ------------- |:-------------:| -----:|
| ok	| boolean| 	响应的HTTP状态码在200~299之间时，该属性为true|
| status| 	number| 	响应的HTTP状态码|
| statusText| 	string	| 响应的状态文本|
| headers	| Object| 	响应头|

# CURD示例

## GET请求

```js
var demo = new Vue({
    el: '#app',
    data: {
        gridColumns: ['customerId', 'companyName', 'contactName', 'phone'],
        gridData: [],
        apiUrl: 'http://211.149.193.19:8080/api/customers'
    },
    ready: function() {
        this.getCustomers()
    },
    methods: {
        getCustomers: function() {
            this.$http.get(this.apiUrl)
                .then((response) => {
                    this.$set('gridData', response.data)
                })
                .catch(function(response) {
                    console.log(response)
                })
        }
    }
})
```

这段程序的then方法只提供了successCallback，而省略了errorCallback。
catch方法用于捕捉程序的异常，catch方法和errorCallback是不同的，errorCallback只在响应失败时调用，而catch则是在整个请求到响应过程中，只要程序出错了就会被调用。

在then方法的回调函数内，你也可以直接使用this，this仍然是指向Vue实例的：
```js
getCustomers: function() {
    this.$http.get(this.apiUrl)
        .then((response) => {
            this.$set('gridData', response.data)
        })
        .catch(function(response) {
            console.log(response)
        })
}
```

## JSONP请求

```js
getCustomers: function() {
    this.$http.jsonp(this.apiUrl).then(function(response){
        this.$set('gridData', response.data)
    })
}
```
## PUT请求

```js
updateCustomer: function() {
    var vm = this
    vm.$http.put(this.apiUrl + '/' + vm.item.customerId, vm.item)
        .then((response) => {
            vm.getCustomers()
        })
}
```
## Delete请求

```js
deleteCustomer: function(customer){
    var vm = this
    vm.$http.delete(this.apiUrl + '/' + customer.customerId)
        .then((response) => {
            vm.getCustomers()
        })
}
```