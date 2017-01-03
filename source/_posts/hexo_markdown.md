
---
title:  hexo命令及Markdown语法
category:  hexo
tags: hexo
date: 2016-11-01
---

这篇我们要介绍两部分内容，Markdown与hexo的常用命令。
<!--more-->
# Markdown
hexo是使用Markdown编辑文章的，我写的这些文章也都是用这种标记语言完成的。所以，我们先从Markdown说起。

## 什么是Markdown

Markdown 是一种轻量级标记语言，创始人为约翰·格鲁伯和亚伦·斯沃茨。它允许人们“使用易读易写的纯文本格式编写文档，然后转换成有效的XHTML文档”。
## Markdown语法
```bash
1、分段： 两个回车
2、换行 两个空格 + 回车
3、标题 #~###### 井号的个数表示几级标题，即Markdown可以表示一级标题到六级标题
4、引用 >
5、列表 *，+，-，1.，选其中之一，注意后面有个空格
6、代码区块 四个空格开头
7、链接 [文字](链接地址)
8、图片 ![图片说明](图片地址)，图片地址可以是本地路劲，也可以是网络地址
9、强调 **文字**，__文字__，_文字_，*文字*
10、代码 ```，``
```
# hexo常用命令
我们在前面的已经略微的接触了一些hexo的命令，如hexo new "my blog"，hexo server等。下面来介绍一下我们经常会用到的hexo命令
## 新建
```bash
#新建的文件在hexo/source/_posts/my-blog.md
hexo new "my blog"
```
## 编译
```bash
#一般部署上去的时候都需要编译一下，编译后，会出现一个public文件夹，将所有的md文件编译成html文件
hexo generate
```

## 开启本地服务
```bash
hexo server #开启本地hexo服务用的
```
## 部署
```bash
hexo deploy
#部署到git上的时候，需要用这个命令
```
## 清除public
```bash
hexo clean
#当source文件夹中的部分资源更改过之后，特别是对文件进行了删除或者路径的改变之后，需要执行这个命令，然后重新编译。
```