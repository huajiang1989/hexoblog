
---
title:  淘宝npm镜像
category:  npm
tags: npm
---

镜像使用方法（三种办法任意一种都能解决问题，建议使用第三种，将配置写死，下次用的时候配置还在）:

# 通过config命令
```js
npm config set registry https://registry.npm.taobao.org
npm info underscore （如果上面配置正确这个命令会有字符串response）
```
# 命令行指定
```bash
npm --registry https://registry.npm.taobao.org info underscore
```
# 编辑 ~/.npmrc 加入下面内容
```bash
registry = https://registry.npm.taobao.org
```
搜索镜像: https://npm.taobao.org

建立或使用镜像,参考: https://github.com/cnpm/cnpmjs.org