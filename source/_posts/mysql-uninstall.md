
---
title:  windows7下启动mysql服务出现服务名无效的原因及解决方法
category: mysql
tags: mysql
---
问题原因：mysql服务没有安装。
解决办法： 在 mysql bin目录下 以管理员的权限 执行 mysqld -install命令
然后仍然以管理员的权限 net start mysql 开启Mysql服务了。
附卸载mysql服务的方法。
1、以管理员的权限 net stop mysql ，关闭mysql服务
2、以管理员的权限 mysqld -remove ，卸载mysql服务