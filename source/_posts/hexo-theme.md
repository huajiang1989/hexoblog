
---
title:  hexo主题设置
category:  hexo
tags: hexo
date: 2016-11-01
---

这篇文章要介绍一下hexo的主题的目录结构及主要文件的作用，并且以light主题为例，较为详细的了解如何优化自己的博客站点。
<!--more-->

# hexo的主题目录结构
```bash
|-- themes
   |-- landscape
   |-- light
```

我的目录结构，截图如下:
 ![haha](http://syxiaqj.github.io/image/introduce-hexo-theme/themeDir.png)
如上面看到的，有两个landscape，light主题，文件夹名称即为主题名称，还记得上一篇文章hexo主目录中的_config.yml文件Extensions部分中的theme:参数吗，那个参数的值便于这里的文件夹名称对应，值是哪个，便意味着哪个主题得到了你的“宠信”（哈哈）。
## 如何下载主题
hexo拥有丰富的主题供你选择：主题传送门
你可以手动下载其中的一款或者几款，然后拷贝到你的themes目录下，搞定！
或者，你cd到themes目录
```bash
cd 你的themes目录
```
然后运行一下命令
```bash
git clone 主题url
```
## 分析light主题
```bash
#目录结构
|-- _config.yml
|-- languages/
|-- layout/
|-- LICENSE
|-- README.md
|-- source/
_config.yml
```
## 主题配置文件
### languages
语言目录，用于保存语言文件
### layout
布局目录，目录结构
```bash
|-- _partial/
|-- _widget/
|-- archive.ejs *
|-- category.ejs *
|-- index.ejs *
|-- layout.ejs *
|-- page.ejs *
|-- post.ejs *
|-- tag.ejs *
```
以上打*的文件表示，每个主题至少需要这些文件。所有的主题都是用layout.ejs作为默认的布局文件，你也可以自定义布局文件。
_widget/： 小工具目录，在light主题中对右边栏的控制。
_partial/： 组件目录，给博客添加统计、评论等功能
hexo支持很多模板引擎，诸如EJS，Swig，Haml，Jade等，文件以什么后缀，即表示用的是哪个模板引擎，light主题用的是EJS模板。每种模板引擎的语法与使用，各位请点击链接查看，这里就不赘述了。
### source
主题资源目录，主题用到的CSS、Javascript等文件需要放在这个目录中，会被编译到hexo的public目录中。
如何进行编译，后面会介绍，我们先赶紧来优化一下自己的博客站点。
## 优化主题
_config.yml
主题配置配置文件，介绍主要参数
```bash
menu:        #站点导航栏 （标签: 路径）
  首页: /
  归档: /archives
  关于: /about

widgets:        #小工具 即站点的右边一栏 页面会按这里的顺序排列
- recent_posts
- category
- weibo_show
- blogroll

excerpt_link: 阅读全文   #默认是Read more 可以改成中文

twitter:    #墙内很少用此鸟，so可以删掉

addthis:    #这个也可以删掉
  enable: true

fancybox: true

baidu_analytics: true    #百度统计，天朝还是这个好使，没办法滴

google_analytics:        #默认统计，多么希望用这个统计，可惜

rss: /atom.xml        #RSS

comment_provider:
```
# 添加多说
light主题自带的是disqus，国内用户，当然不好使，还是用“多说”好。

1.先在多说注册
2.创建站点
3.完成后，在“工具”中，获取通用代码
```js
<!-- Duoshuo Comment BEGIN -->
<div class="ds-thread"></div>
<script type="text/javascript">
var duoshuoQuery = {short_name:"用户名"};
    (function() {
        var ds = document.createElement('script');
        ds.type = 'text/javascript';ds.async = true;
        ds.src = 'http://static.duoshuo.com/embed.js';
        ds.charset = 'UTF-8';
        (document.getElementsByTagName('head')[0]
        || document.getElementsByTagName('body')[0]).appendChild(ds);
    })();
    </script>
<!-- Duoshuo Comment END -->
```
4.用下面命令打开comment.ejs
```bash
vim themes/light/layout/_partial/comment.ejs
```
修改如下
```js
<% if (page.comments){ %>
<section id="comment">
    //多说通用代码
</section>
<% } %>
```
5.Over
# 添加分享
虽然某度，我基本不用，但是某度的分享，大家都用，我也就用用。
1.打开百度分享，获取代码。记得进行下一步按钮设置，可以对是否设置页面分享、图片分享、划词分享、按钮类型、风格、图标大小等功能。
2.在themes/light/layout/_partial/目录下，新建一个baidu_analytics.ejs文件
```bash
touch themes/light/layout/_partial/baidu_analytics.ejs
```
3.将获取的代码写入baidu_analytics.ejs文件
4.在themes/light/layout/layout.ejs文件中添加百度统计，添加到</body>标记前
```
<%- partial('_partial/baidu_analytics') %>
```
# 添加微博show

1、到微博开发平台获取代码
2、在themes/light/layout/_widget/目录下，新建一个weibo_show.ejs文件，并且将刚获取的代码，写入该文件
3、在themes/light/_config.yml文件中的widgets中添加weibo_show
```
widgets:
- weibo_show
```
见上面介绍_config.yml内容
# 添加友情链接
1、在themes/light/layout/_widget/目录下，新建blogroll.ejs
2、编辑文件
```html
<div class="widget tag">
<h3 class="title">友情链接</h3>
<ul class="entry">
<li>
    <a href="链接地址" title="title" target="_blank">显示名称</a>
</li>
</ul>
</div>
```
3、在themes/light/_config.yml文件中的widgets中添加blogroll
```
widgets:
- blogroll
```
见上面介绍_config.yml内容
# 添加“关于”
作为一个完整的博客站点，“关于”页是不能少的。下面，我们在导航栏上添加上这部分内容
1、新建一个页面
```bash
hexo new page "about"
```
2、编辑source/about/index.md，至于编辑什么，您随意。
3、在主题的配置文件themes/light/_config.yml中添加
```
menu:
  关于: /about
```
见上面介绍_config.yml内容