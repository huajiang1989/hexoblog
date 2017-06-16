---
title:  Nodejs 部署到阿里云全过程
category:  nodejs
tags: nodejs
---
整个部署过程学到了不少东西，记录一下。
```
参考了以下文章：
http://itbilu.com/other/relate/NJ2TJohl.html
https://segmentfault.com/a/1190000004051670
http://borninsummer.com/2015/06/17/notes-on-developing-nodejs-webapp/
https://bbs.aliyun.com/read/146189.html
https://segmentfault.com/q/1010000003937372
```
<!--more-->
# 1. 到阿里云购买云服务器 ECS 。
```
https://www.aliyun.com/product/ecs
```
如果是在校学生，在淘宝有实名认证，且在学信网有注册，可以试试抢学生的首月优惠套餐。https://www.aliyun.com/act/aliyun/campus.html
作为一个穷逼+不熟悉服务器配置的菜鸟。选了最便宜的套餐：
CPU： 1核 / 内存： 1024 MB / 带宽：1Mbps / 操作系统： CentOS 7.0
购买环节会设置 ssh 登陆密码，记下密码。

在最后的付费环节，使用推荐码可以享受9折。我的推荐码 no4qx1
登陆到阿里云，查看购买的实例。
注意公网 IP，下一步会用到


购买的实例
# 2. 登陆服务器

打开 Terminal， 输入 ssh root@公网IP 登陆服务器。首次登陆会询问公钥，yes 即可。关于 ssh 登陆，具体可以看
```
http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html
```
这篇文章。

登陆服务器后。这里对于我这个 Linux 菜鸟有个大坑………就是 Linux 系统常见的目录结构和文件放置区域。
使用 root 用户身份登陆后，会直接进入到下图 红色箭头标出的 root 目录下。先 cd .. 跳转到上一层, 再 ls -a ，就可以看到类似下图的目录结构了。


图片引用自《鸟哥的Linux》
# 3.安装 node 和 mongodb
```
node -- 编译后二进制文件应在/usr/local/bin/node 下
mongodb -- 安装在/usr/local/mongodb 下
```
下面就一步一步来，首先升级CentOS
```
yum -y update
```

升级后，跳转到 /usr/local/src , 这个文件夹通常用来存放软件源代码
```
cd /usr/local/src
```
下载 nodejs 代码，也可以使用scp命令直接上传，因为下载实在太慢了。
```
wget http://nodejs.org/dist/v0.12.5/node-v0.12.5.tar.gz
```
解压
```
tar -xzvf node-v0.12.5.tar.gz
```
进入解压后的文件夹
```
cd node-v0.12.5
```
执行配置脚本来进行编译预处理
```
./configure
```
编译源代码
```
make
```
当编译完成后，需要使之在系统范围内可用, 编译后的二进制文件将被放置到系统路径，默认情况下，Node二进制文件应该放在/user/local/bin/node文件夹下
```
make install
```
安装 express 和 forever，这两个模块都推荐 global 安装
```
npm -g install express forever
```
建立超级链接, 不然 sudo node 时会报 "command not found"
```
sudo ln -s /usr/local/bin/node /usr/bin/node
sudo ln -s /usr/local/lib/node /usr/lib/node
sudo ln -s /usr/local/bin/npm /usr/bin/npm
sudo ln -s /usr/local/bin/node-waf /usr/bin/node-waf
sudo ln -s /usr/local/bin/forever /usr/bin/forever
```
Nodejs到这里就基本安装完成了。

 下面来安装mongodb

软件安装位置：/usr/local/mongodb
数据存放位置：/var/mongodb/data
日志存放位置：/var/mongodb/logs
首先下载安装包
···
cd /usr/local
wget http://fastdl.mongodb.org/linux/mongodb-linux-x86_64-2.4.9.tgz
···
解压安装包，重命名文件夹为mongodb
```
taz zxvf mongodb-linux-x86_64-2.6.0.tgz
mv mongodb-linux-x86_64-2.6.0 mongodb
```
创建数据和日志存放目录
```
mkdir /var/mongodb
mkdir /var/mongodb/data
mkdir /var/mongodb/logs
```
打开rc.local文件，添加CentOS开机启动项：
```
vim /etc/rc.d/rc.local
```
将mongodb启动命令追加到本文件中，让mongodb开机自启动：
```
/usr/local/mongodb/bin/mongod --dbpath=/var/mongodb/data --logpath /var/mongodb/logs/log.log -fork
```
关闭 vim 后，直接手动启动mongodb
```
/usr/local/mongodb/bin/mongod --dbpath=/var/mongodb/data --logpath /var/mongodb/logs/log.log -fork
```
看到类似的信息，说明已启动成功。我在这里发了个傻，以为26308是port号，导致后面设置port时折腾了好久。其实这里的 forked process 和 port 号是两个东西， 这个是程序本身在Server上的进程。
```
forked process: 26308
```
启动mongo shell
```
cd /usr/local/mongodb/bin/
./mongo
```
在 mongo shell 中创建管理员及数据库
```json
use admin //admin 数据库
db.createUser({
user: "用户名",
pwd:"登陆密码",
roles:["userAdminAnyDatabase"] //超级管理员
})

use databaseFoo //nodeapp 要连接的数据库
db.createUser({
user: "用户名",
pwd:"登陆密码",
roles:["readWrite"] //读写权限
})
```
到这里 mongodb 基本已经安装设置完成了。具体数据的迁移导入可自行研究。

# 4.配置及启动node app

我们把 nodeapp 的程序放在 /home 下
```
cd /home
```
我使用 http://git.oschina.net/ 管理代码。它的私有库是免费的。基本操作和 github 一样。
复制代码：
```
git clone https://git.oschina.net/xxxxxxx/nodeapp.git   //你的repo地址
```
进入 nodeapp 文件夹
```
cd nodeapp
```
（若后续代码变更，提交到 git repo 后直接git pull即可部署代码）

安装nodeapp的所有依赖
```
npm install
```

在启动文件 ( 我的是 app.js ) 中设置数据库连接
```
vim app.js
```
数据库连接类似下面的格式，由于数据库安装在同一服务器，因此 host 为127.0.0.1：
```js
var dbUrl = 'mongodb://用户名:登陆密码@127.0.0.1/databaseFoo';
mongoose.connect(dbUrl)
```
这里要注意，如果直接 npm start 或 node app.js 启动，则一旦退出 ssh 远程登陆，nodeapp 就会停止运行。因此我们使用 forever 启动 nodeapp。
```
NODE_ENV=production forever start app.js
```
在尝试启动的过程中，一般会有很多报错。按照报错一点一点配置即可……

在蹚过无数坑后，项目部署成功。用浏览器打开 公网IP:端口号 即可看到 nodeapp 的首页

DNS 设置过程以后再补。