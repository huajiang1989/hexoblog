---
title: 使用node和git搭建Hexo博客
category: hexo
tags: hexo
---
安装Hexo
----------
```bash
  #修改npm下载路径
  $ npm install -g cnpm --registry=https://registry.npm.taobao.org
  #全局安装hexo
  $ npm install -g hexo-cli
```
<!--more-->
----------
本地运行hexo
-------------
```bash
#初始化hexo
$ hexo init
#安装依赖包
$ npm install
#运行hexo,以后要在本地运行博客只要输入该命令即可
$ hexo s -g
# 停止运行
#按住Ctrl+C键即可停止

```
hexo配置信息
---
```js
#博客名称
title: Mr.hua
#副标题
subtitle:subtitle
#简介
description: your site interdution
#博客作者
author: Mr.hua
#博客语言
language: zh-CN
#时区
timezone:

#博客地址,与申请的GitHub一致
url: http://huajiang1989.github.io
root: /
#博客链接格式
permalink: :year/:month/:day/:title/
permalink_defaults:

source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: true
  tab_replace:

default_category: uncategorized
category_map:
tag_map:

#日期格式
date_format: YYYY-MM-DD
time_format: HH:mm:ss

#分页，每页文章数量
per_page: 10
pagination_dir: page

#博客主题
theme: landscape

#发布设置
deploy:
  type: git
  #huajiang1989改为你的github用户名
  repository: https://github.com/huajiang1989/huajiang1989.github.io.git
  branch: master
```
