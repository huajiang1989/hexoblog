---
title:  nvm安装与常用命令
category:  nvm
tags: nvm
---
nvm 是nodejs的版本管理工具，可以很方便的进行node版本切换
<!--more-->
# 在windows下用nvm 安装node

## nvm 下载
nvm 的下载地址：https://github.com/coreybutler/nvm-windows 。选择第一个 nvm-noinstall.zip ，然后解压在系统盘（一般开发相关的文件我都放C盘，但是放别的盘也是可以的）。我放的目录路径是C:\dev\nvm。解压出来的文件有：
```
 +  elevate.cmd
 +  elevate.vbs
 +  install.cmd
 +  LICENSE
 +  nvm.exe
```
## nvm 安装

双击 install.cmd ，是以控制台形式显示的，第一下直接按回车，然后会在C盘根目录产生settings.txt，把这个文件放进刚刚解压的那个目录，然后修改settings.txt内容，改成下面那样：
```
 root: C:\dev\nvm
 path: C:\dev\nodejs
 arch: 64
 proxy: none
 node_mirror: http://npm.taobao.org/mirrors/node/
 npm_mirror: https://npm.taobao.org/mirrors/npm/
```
但是有些人很不幸，这个方法行不通，因为打开 install.cmd按下回车后，显示拒绝访问注册表路径，并弹出一个settings.txt。这时候，你只要淡定地叉掉那个文本以及控制台，然后在刚刚的目录里新建一个文件settings.txt，最后把上面的内容复制进去就可以了。
```
root ： nvm的存放地址
path ： 存放指向node版本的快捷方式，使用nvm的过程中会自动生成。一般写的时候与nvm同级。
arch ： 电脑系统是64位就写64,32位就写32
proxy ： 代理
```
##  nvm 配置

以控制台方法执行成功的，在环境变量里会自动配置了 NVM_HOME 和 NVM_SYMLINK ，这时候只要修改相应的路径就行了。
直接创建settings文件的可以在环境变量里创建 NVM_HOME 和 NVM_SYMLINK，并添加路径
要是嫌弃可视化界面打开环境变量的步骤太麻烦，可以直接使用  windows+r =>
```
sysdm.cpl
NVM_HOME： C:\dev\nvm
NVM_SYMLINK ： C:\dev\nodejs
在PATH里加上;%NVM_HOME%;%NVM_SYMLINK%;。
```
一键控制台install的还要检查 环境变量 PATH 上的路径有没有添加C:\dev\nvm以及C:\dev\nodejs,有的话就删掉。
## 检测安装结果

打开控制台，输入：nvm -v,若是出现版本信息，则安装。若报错，那就重新把步骤再捋一遍。

检查环境变量是否配置成功：可以在控制台输入：set [环境变量名]，查看路径是否填写错误
## 使用node

控制台下载 => 输入：nvm install [版本号]，下载最新版的可以直接输nvm install latest
下载完成后，在控制台输入：nvm use [版本号]。即使用这个版本号的node了。在use后，上面所说的nodejs文件夹就自动生成了。（在use之前是没有的哦）

# 在linux下安装nvm

## 安装nvm
```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.1/install.sh | bash
```
或者
```
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.31.1/install.sh | bash
```

```
## 安装brew
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

## 安装nvm
$ brew install nvm
```
shell用了zsh，所以还要在~/.zshrc 配置文件里添加nvm PATH
```
## 添加nvm PATH
 export NVM_DIR="~/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
```

## 使用淘宝镜像安装nodejs
```
NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node nvm install 6.2.0
```

# nvm 常用命令

```

nvm install ## 安装指定版本，可模糊安装，如：安装v4.4.0，既可nvm install v4.4.0，又可nvm install 4.4

nvm uninstall ## 删除已安装的指定版本，语法与install类似

nvm use ## 切换使用指定的版本node

nvm ls ## 列出所有安装的版本

nvm ls-remote ## 列出所以远程服务器的版本（官方node version list）

nvm current ## 显示当前的版本

nvm alias ## 给不同的版本号添加别名

nvm unalias ## 删除已定义的别名

nvm reinstall-packages ## 在当前版本node环境下，重新全局安装指定版本号的npm包
```
