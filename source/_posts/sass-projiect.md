
---
title:  管理Sass项目文件结构
category:  sass
tags: sass
---
# 构建你的结构体系
CSS预处理器的特点之一是可以把你的代码分割成很多个文件，而且不会影响性能。这都要归功于Sass的@import命令，只要在你的开发环境下，你调用不管多少文件，最终将编译出一个CSS样式文件。

开始将你的CSS文件分割成多个文件和文件夹。
<!--more-->
# 文件夹构建
文件夹的创建是必不可少的。

你在创建CSS的架构的时候不只是把所有的Sass文件放在一个文件夹下，你会将他们分类。

下面的示例屏示的是我将如何组织我的Sass文件：
```bash
sass/
|
|– base/
|   |– _reset.scss       # Reset/normalize
|   |– _typography.scss  # Typography rules
|   ...                  # Etc…
|
|– components/
|   |– _buttons.scss     # Buttons
|   |– _carousel.scss    # Carousel
|   |– _cover.scss       # Cover
|   |– _dropdown.scss    # Dropdown
|   |– _navigation.scss  # Navigation
|   ...                  # Etc…
|
|– helpers/
|   |– _variables.scss   # Sass Variables
|   |– _functions.scss   # Sass Functions
|   |– _mixins.scss      # Sass Mixins
|   |– _helpers.scss     # Class & placeholders helpers
|   ...                  # Etc…
|
|– layout/
|   |– _grid.scss        # Grid system
|   |– _header.scss      # Header
|   |– _footer.scss      # Footer
|   |– _sidebar.scss     # Sidebar
|   |– _forms.scss       # Forms
|   ...                  # Etc…
|
|– pages/
|   |– _home.scss        # Home specific styles
|   |– _contact.scss     # Contact specific styles
|   ...                  # Etc…
|
|– themes/
|   |– _theme.scss       # Default theme
|   |– _admin.scss       # Admin theme
|   ...                  # Etc…
|
|– vendors/
|   |– _bootstrap.scss   # Bootstrap
|   |– _jquery-ui.scss   # jQuery UI
|   ...                  # Etc…
|
|
`– main.scss             # primary Sass file
```
正如你所看到的，在根目录底下只有一个main.scss文件，其他.scss文件都根据不同的分类放在对应的文件夹中，只是这些.scss文件前面都有一个下划线(_)，用来告诉Sass，这些.scss文件只是局部，不通过@import是不应该被编译出.css文件。事实上，它们是导入和合并文件的基本文件而以。

一个文件可以解决所有问题，一个文件可以找到他们，一个文件给他们带来了所有的一切，Sass只是将他们合并在一起。——@J.R.R. Tolkien

接下来，我们依次来看结构中的每一个文件目录。

## Base

base/文件夹包含了一些有关于你的项目中一些模板相关。在这里，你可以看到reset样式(或者Normalize.css,或者其他)，也有一些关于文本排版方面的，当然根据不同的项目会有一些其他的文件。
```js
_reset.scss或_normalize.scss
_typography.scss
```
## Helpers

helpers/文件夹（有的地方也称其为utils/）主要包含了项目中关于Sass的工具和帮助之类。在里面放置了我们需要使用的_function.scss，和_mixin.scss。在这里还包含了一个_variables.scss文件（有的地方也称其为_config.scss），这里包含项目中所有的全局变量（比如排版本上的，配色方案等等）。
```bash
_variables.scss
_mixin.scss
_function.scss
_placeholders.scss#(也有称为_helpers.scss)
```
## Layout

layout/文件夹(有时也称为partials/)中放置了大量的文件，每个文件主要用于布局方面的，比如说"header"，“footer”等。他也会包括_grid.scss文件，用来创建网格系统。
```js
_grid.scss
_header.scss
_footer.scss
_sidebar.scss
_forms.scss
```
导航文件（_navigation.scss）文件放在这里也有意义，虽然我将他放在了components/文件夹中。但是我想将其放在layout/文件夹中更好些，当然最后还是由你自己来决定。

## Components

对于一些小组件，都放在了components/文件夹（通常也称为modules/），layout/是一个宏观的（定义全局的线框），components/是一个微观的。它里面放了一些特定的组件，比如说slider，loading，widget或者其他的小组件。通常components/目录下的都是一些小组件文件。
```js
_media.scss
_carousel.scss
_thumbnails.scss
```
## Page

如果你需要针对一些页面写特定的样式，我想将他们放在page/文件夹中是非常酷的，并且以页面的名称来命名。例如，你的首页需要制作一个特定的样式，那么你就可以在page/文件夹中创建一个名叫_home.scss文件。
```js
_home.scss
_contact.scss
```
根据你自己的布署，你可以根据自己的需求调用这些文件，避免与其他样式文件合并在一起。这真的主取决于你自己，在我工作的地方，我是不允 许这样的事情发生，只在需要的页面调用需要的文件。比如说，我们首页有一个特定的布局样式，编译出来的CSS大约有200行代码。为了防止每个页面加载这 些代码，我只在主页文件上引用这个文件。

## Themes

如果你像我一样要为一个大型的网站制作多个主题，那么有一个theme/文件夹是非常有意义的。你可以把主题相关的文件放在这个文件夹中。这绝对跟具体的项目有关，你只要觉得跟主题相关的，有必要引入。
```js
_theme.scss
_admin.scss
```
## Vendors

最后一个但并非不重要，创建vendors/文件夹，主要用来包含来自外部的库和框架的CSS文件。比如Bootstrap,jQueryUI，FancyCarouselSliderjQueryPowered等等。把这些文件放在同一个文件夹中，你可以说，嘿，这些代码不是我的，不是我写的，跟我无关。

例如：
```js
bootstrap.scss
jquery-ui.scss
select2.scss
```
从另一个角度来说，在我平时工作中，还创建了一个vendors-extensions/文件夹，用来放置一些覆盖从外部引入进来的库和框架中的小组件。例如，我们可以在_bootstrap.scss文件中用来覆盖Bootstrap框架中的一些小组件。这为了避免和外部直接引来的组件升级造成的冲突，或许这不是一个很好的方案。

大致就是这些，但不同的项目可能会不一样，但我可以肯定，你们都有了这样的一个概念。在文件夹中嵌套一个文件夹，这样的做法我一直不太反对，但我不 太喜欢这样的方式。我发现，在大多数情况之下，只需一个层级就足足够，既保证结构的简洁与清晰，而且不复杂。但话又说回来，如果你觉得你的项目有必要嵌套 更深层次的文件夹，你也可以自由的发挥。

温馨提示：如果你觉得你的架构并不能向大家说明SCSS文件夹的架构，你可以在根目录下创建一个README.md文件（或者在main.scss文件中一步一步说明）解释。

文件很酷？
有一个问题常被人问到“多少文件才算是很多文件呢？”我常回答“再多文件都不算多”。拆分成多个文件的宗旨是帮助你组织你的代码。如果你觉得某事值得拆分成多个文件，可以自由的拆分。正如CHRIS COYIER在《Sass Style Guide》中所说：

拆分成尽可能多的小文件是有道理的。——@CHRIS COYIER

不过，我建议不把单个组件拆分成多个文件，除非你有很好的理由这样做。通常我更倾向于一个组件一个文件。俗话说“没有更多，只有更少”。用一个简洁语义化的名称，用来表示模块的名称。这样我们就可以通过查找名称找到你需要的东西