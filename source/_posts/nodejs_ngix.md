
---
title:  Nodejs实现简单的集群负载
category:  nodejs
tags: nodejs
---


在Nodejs中使用集群还是不容易的。Javascript的单线程属性让nodejs下的应用很难使用现代机器的多核特性。比如下面的代码实现了一个http服务器的主干部分。这部分代码只会执行在一个线程上，不管这段代码运行的机器是单核的cpu还是1000个内核的cpu。
<!--more-->
```js
var http = require("http");
var port = parseInt(process.argv[2]);

http.createServer(function(request, response) {
  console.log("Request for:  " + request.url);
  response.writeHead(200);
  response.end("hello world\n");
}).listen(port);
```
# 使用多核特性

只需要一点修改，上面的代码就可以把cpu的所有核心都用起来。上面的示例代码将使用 cluster 模块重构。 cluster 模块可以让你很容易的创建多个分享端口的进程。每一个进程使用一个系统核心，也就是代码中的 numCPUs 变量中cpu核心的一个。每一个子进程都实现了HTTP server，并监听指定的端口。
```js
var cluster = require("cluster");
var http = require("http");
var numCPUs = require("os").cpus().length;
var port = parseInt(process.argv[2]);

if (cluster.isMaster) {
  for (var i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on("exit", function(worker, code, signal) {
    cluster.fork();
  });
} else {
  http.createServer(function(request, response) {
    console.log("Request for:  " + request.url);
    response.writeHead(200);
    response.end("hello world\n");
  }).listen(port);
}
```
# 多机器的均衡

使用cluster模块，你就可以更高效的使用硬件。然而，你还是被限制在单一的机器上。如果你的应用有客观的访问量，你最终还是把负载分部在不同的机器上。使用reverse proxy server可以把并发的访问负载到不同的服务器上。

Nodejitsu开发了 node-http-proxy 模块，一个开源的nodejs应用代理服务。使用以下命令可以安装这个模块：
```
npm install http-proxy
```
实际的使用可以参考以下代码。在这里例子中负载被分发到两台服务器上。首先测试反转代理，确保HTTP server运行在8080和8081两个端口上。接下来，运行反转代理，然后用浏览器访问这个代理。如果一切正常的话，你会发现请求被两个服务器交替处理。
```js
var proxyServer = require('http-proxy');
var port = parseInt(process.argv[2]);
var servers = [
  {
    host: "localhost",
    port: 8081
  },
  {
    host: "localhost",
    port: 8080
  }
];

proxyServer.createServer(function (req, res, proxy) {
  var target = servers.shift();

  proxy.proxyRequest(req, res, target);
  servers.push(target);
}).listen(port);
```
当然，这个例子只使用了一台机器。然而，如果你有多台机器的话，你可以在一台机器上运行反向代理服务器，其他的机器上运行HTTP server。

# 使用nginx负载均衡

使用nodejs写的反向代理有一个好处是你使用的技术都是一样的。但是，在生产环境下，更多使用的是nginx来处理负载均衡。nginx是一个开源的HTTP server和反向代理工具，尤其擅长处理静态文件，比如：CSS和HTML。因此，nginx常被用于处理站点的静态文件，和分发动态请求到nodejs的服务器上。

要实现nginx的负载均衡，只需要安装nginx，之后把nodejs服务器作为upstream resource添加在配置文件中。配置文件的路劲一般是{nginx-root}/conf/nginx.conf，{nginx-root}是nginx安装的根目录。整个的配置文件请参考下面的示例。当然，我们只需要用到其中的一小部分。
```
#user  nobody;
worker_processes  1;
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid		logs/nginx.pid;

events {
	worker_connections  1024;
}
http {
	include	   mime.types;
	default_type  application/octet-stream;

	#log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
	#				  '$status $body_bytes_sent "$http_referer" '
	#				  '"$http_user_agent" "$http_x_forwarded_for"';
	#access_log  logs/access.log  main;

	sendfile		on;
	#tcp_nopush	 on;
	#keepalive_timeout  0;
	keepalive_timeout  65;
	#gzip  on;
	upstream node_app {
	  server 127.0.0.1:8080;
	  server 127.0.0.1:8081;
	}
	server {
		listen	   80;
		server_name  localhost;
		#charset koi8-r;
		#access_log  logs/host.access.log  main;

		location / {
			root   html;
			index  index.html index.htm;
		}
		location /foo {
		  proxy_redirect off;
		  proxy_set_header   X-Real-IP			$remote_addr;
		  proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
		  proxy_set_header   X-Forwarded-Proto $scheme;
		  proxy_set_header   Host				   $http_host;
		  proxy_set_header   X-NginX-Proxy	true;
		  proxy_set_header   Connection "";
		  proxy_http_version 1.1;
		  proxy_pass		 http://node_app;
		}
		#error_page  404			  /404.html;

		# redirect server error pages to the static page /50x.html
		#
		error_page   500 502 503 504  /50x.html;
		location = /50x.html {
			root   html;
		}
		# proxy the PHP scripts to Apache listening on 127.0.0.1:80
		#
		#location ~ \.php$ {
		#	proxy_pass   http://127.0.0.1;
		#}
		# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
		#
		#location ~ \.php$ {
		#	root		   html;
		#	fastcgi_pass   127.0.0.1:9000;
		#	fastcgi_index  index.php;
		#	fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
		#	include		fastcgi_params;
		#}
		# deny access to .htaccess files, if Apache's document root
		# concurs with nginx's one
		#
		#location ~ /\.ht {
		#	deny  all;
		#}
	}
	# another virtual host using mix of IP-, name-, and port-based configuration
	#
	#server {
	#	listen	   8000;
	#	listen	   somename:8080;
	#	server_name  somename  alias  another.alias;
	#	location / {
	#		root   html;
	#		index  index.html index.htm;
	#	}
	#}
	# HTTPS server
	#
	#server {
	#	listen	   443;
	#	server_name  localhost;
	#	ssl				  on;
	#	ssl_certificate	  cert.pem;
	#	ssl_certificate_key  cert.key;
	#	ssl_session_timeout  5m;
	#	ssl_protocols  SSLv2 SSLv3 TLSv1;
	#	ssl_ciphers  HIGH:!aNULL:!MD5;
	#	ssl_prefer_server_ciphers   on;
	#	location / {
	#		root   html;
	#		index  index.html index.htm;
	#	}
	#}
}
```
如前文所述，本教程只会涉及到整个配置文件的一部分。第一个需要关注的部分如下所示。
```js
upstream node_app {
  server 127.0.0.1:8080;
  server 127.0.0.1:8081;
}
```
这部分的配置定义了一个 upstream 服务器，名称为 node_app 。对这个服务器的请求会分配到就两个ip地址上（这里只用端口区分了一下）。

只是定义了一个upstream服务器还没有告诉nginx如何使用它。因此，我们必须使用如下的指令顶一个路由的规则。使用这个路由，任何的到/foo的请求都会被代理到之前配置的nodejs服务器上。
```js
location /foo {
  proxy_redirect off;
  proxy_set_header   X-Real-IP            $remote_addr;
  proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
  proxy_set_header   X-Forwarded-Proto $scheme;
  proxy_set_header   Host                   $http_host;
  proxy_set_header   X-NginX-Proxy    true;
  proxy_set_header   Connection "";
  proxy_http_version 1.1;
  proxy_pass         http://node_app;
}
```
# 最后

本教程旨在介绍如何把单线程的Nodejs应用运行在多台机器的多个核心上。你也可以学到如何使用nodejs或者nginx建立一个负载均衡。当然本文不是深入的介绍如何在产品环境下运行的。因此，如果你使用的是nginx，还有很多其他的可以做的，比如缓存，来提高系统性能。你也会需要使用forever，如果崩溃的话这个工具可以重启你的nodejs进程。