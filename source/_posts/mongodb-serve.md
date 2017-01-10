
---
title:  MongoDB安装成为Windows服务
category: MongoDB
tags: MongoDB
---

MongoDB安装成为Windows服务
使用以下命令将MongoDB安装成为Windows服务。
笔者的MongoDB目录为D:\Program Files\mongodb

切换到D:\Program Files\mongodb\bin>
```bash
mongod --logpath "D:\Program Files\mongodb\data\logs.txt" --logappend --dbpath "D:\Program Files\mongodb\data" --directoryperdb --serviceName "MongoDB" --serviceDisplayName "MongoDB" --install
```
输入以上命令。

<!--more-->
```bash
Creating service MongoDB.
Service creation successful.
Service can be started from the command line via 'net start "MongoDB"'.
```


该命令行指定了日志文件：D:\Program Files\mongodb\data\logs.tx，日志是以追加的方式输出的；

数据文件目录：D:\Program Files\mongodb\data，并且参数--directoryperdb说明每个DB都会新建一个目录；



Windows服务的名称：MongoDB；


最后是安装参数：--install，与之相对的是--remove

启动MongoDB：net start MongoDB

停止MongoDB：net stop MongoDB


注意：遇到问题请查看日志文件
```bash
mongodb exception in initAndListen: 12596 old lock file, terminating解决方法
```
错误信息如下：
```bash
exception in initAndListen: 12596 old lock file, terminating
```
解决方法

1.删除data目录中的.lock文件

2.mongod.exe --repair

3.启动mongod就可以了