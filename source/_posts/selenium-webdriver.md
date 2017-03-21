---
title: selenium-webdriver进行自动化测试
category: test
tags: test
---

# 安装selenium-webdriver依赖包

```bash
npm install selenium-webdriver --save
```

# 下载Chrome插件
```
#去 http://chromedriver.storage.googleapis.com/index.html?path=2.24/ 下载
chromedriver_win32.zip
#下载驱动，并存放到一个目录中，例如：D:/driver/
#把这这个目录添加到系统环境变量PATH下面
```

<!--more-->
# 初始化一个测试文件
```js
var webdriver = require('selenium-webdriver'),
    By = webdriver.By,//查询元素组件
    until = webdriver.until;//等待事件

var driver = new webdriver.Builder()//初始化浏览器
    .forBrowser('chrome')
    .build();

var Mock = require('mockjs')//模拟数据

driver.get('https://www.baidu.com');//打开浏览器
driver.sleep(300)//等待300毫秒执行
driver.findElement(By.id('kw')).clear();//input元素清空
driver.findElement(By.id('kw')).sendKeys('webdriver');//查找id为kw的元素，赋值webdriver
driver.findElement(By.id('su')).click();//查找id为su的元素，点击
driver.wait(until.titleIs('webdriver_百度搜索'), 1000);//等待页面title成为‘xxxx’
driver.quit();//退出

```

# 常用API

## driver
```
driver.get('https://www.baidu.com');//打开浏览器
driver.sleep(300)//等待300毫秒执行
driver.findElement(By.id('kw')).clear();//input元素清空
driver.findElement(By.id('kw')).sendKeys('webdriver');//查找id为kw的元素，赋值webdriver
driver.findElement(By.id('su')).click();//查找id为su的元素，点击
driver.wait(until.titleIs('webdriver_百度搜索'), 1000);//等待页面title成为‘xxxx’
driver.quit();//退出

```

## By
```
# By.name(val) 获取name为val的元素
# By.id(val) 获取id为val的元素
# By.css(val) 获取css为val的元素,可以用样式查询
# By.xpath(val) 获取xpath元素，这个用的多 eg://button[@type='button']
```

