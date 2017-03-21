---
title:  webdriver-元素定位
category:  test
tags: test
---

webdriver提供了丰富的API，有多种定位策略：id,name,css选择器，xpath等，其中css选择器定位元素效率相比xpath要高些，使用id，name属性定位元素是最可靠，效率最高的一种办法。
<!--more-->
　　1、工具选择：在我们开发测试脚本的过程中各个浏览器给我们也提供了方便定位元素的工具，我比较喜欢使用firefox的firebug工具，也是目前很多开发测试人员比较热衷的选择，原因是firefox是唯一能够集成selenium IDE的浏览器，并且firebug给用户提供了丰富的扩展组件，我们可以根据自己的需要来选择，一般情况下，使用firebug+firefinder就足够使用了，firefinder支持xpath以及css选择器定位元素的功能，很方便帮助我们调试测试脚本
　　2、元素定位的方法：findElement() 与 findElements()
　　findElement() 该方法返回基于指定查询条件的webElement对象，或抛出不符合条件的异常  eg：driver.findElement(By.id("userID"));
　　findElements() 该方法返回指定查询条件的WebElement的对象集合，或返回null
　　3、WebElement对象提供的各种定位元素策略
```js
ID：driver.findElement(By.id(<elementID>))
Name：driver.findElement(By.name(<elementName>))
className：driver.findElement(By.className(<elementClassName>))
tagName：driver.findElement(By.tagName(<htmlTagName>))
linkText：driver.findElement(By.linkText(<linkText>))
partialLinkText：driver.findElement(By.partialLinkText(<partialLinkText>))
css：driver.findElement(By.cssSelector(<cssSelector>))
xpath：driver.findElement(By.xpath(<xpathQuery>))
```
　　4、webelement类提供了诸多方法，在我们开发脚本过程中如何选择最可靠，效率最高的方法，使用id，name是首选，因为他们在html标签中是唯一的，所以是最可靠的
```js
　　ID：driver.findElement(By.id("username"))
　　name：driver.findElement(By.name("username"))
　　class：driver.findElement(By.className("username"))
```
　　多学一招：WebElement类支持查询子类元素，如果页面中存在重复元素，但在不同div中，我们可以先定位到其父元素，然后定位其子元素，方法如下：
```js
WebElement hello = driver.findElement(By.id("div1")).findElement(By.lindText("hello"));
```
　　5、使用WebElements定位多个相似的元素，比如页面中存在五个单选按钮,他们有相同的class属性，值为：myRadio，我们想对五个按钮循环操作，我们可以把它们全部取出来放到集合中，然后做循环操作，如下：
```js
List<WebElement> radios = driver.findElements(By.className("myRadio"));
for(int i = 0;i<radios.size();i++){
radios.get(i).click();
}
```
　　其他定位方法与操作id，name类似，这里不再赘述，接下来我着重对css选择器与Xpath描述下
# WebDriver By类
1. 使用相对路径定位元素
　　如，我们要定为DOM中的input元素，我们可以这样操作，不考虑其在DOM中的位置，但这样做存在一定弊端，当DOM中存在多个input元素时，该方法总返回DOM中的第一个元素，这并不是我们所期待的
　　eg：WebElement username = driver.findElement(By.cssSelector("input"));
　　另外，为了使用这种方法更准确的定位元素，我们可以结合该元素的其他属性来实现精确定位的目的
2. 结合id来定位
    driver.findElement(By.cssSelector("input#username"));
在标签与id之间使用#连接，如果对css了解的朋友一看就知道为什么会这样写了，不了解也没关系，只要记住这种写法就OK了
　　另外该方法也可简写为driver.findElement(By.cssSelector("#username")); 有点儿类似于id选择器
3. 使用元素的任何属性来定位元素
```js
　　driver.findElement(By.cssSelector("标签名[属性名='属性值']"));
```
4. 匹配部分属性值
^=
```js
driver.findElement(By.cssSelector("标签名[属性名^='xxx']"));
```
5. 匹配属性值以xxx开头的元素
$=
```js
driver.findElement(By.cssSelector("标签名[属性名$='xxx']"));
```
6. 匹配属性值以xxx结尾的元素
*=
```js
driver.findElement(By.cssSelector("标签名[属性名^='xxx']"));  匹配属性值包含xxx的元素
```
7. 使用相对+绝对路径方法
```js
　　driver.findElement(By.cssSelector("div#login>input"))
```
该方法中“div#login>input” 首先通过相对路径定位到id为login的div元素，然后查找其子元素input（绝对路径）
# 使用xpath
相比cssSelector，xpath是我比较常用的一种定位元素的方式，因为它很方便，缺点是，消耗系统性能
　　1、使用绝对路径定位元素
```js
　　driver.findElement(By.xpath("/html/body/div/form/input"))
```
　　2、使用相对路径定位元素
　　driver.findElement(By.xpath("//input"))   返回查找到的第一个符合条件的元素
　　3、使用索引定位元素，索引的初始值为1，注意与数组等区分开
　　driver.findElement(By.xpath("//input[2]"))   返回查找到的第二个符合条件的元素
　　4、结合属性值来定位元素
```js
　　driver.findElement(By.xpath("//input[@id='username']"));
　　driver.findElement(By.xpath("//img[@alt='flowr']"));
```
　　5、使用逻辑运算符，结合属性值定位元素,and与or
```js
　　driver.findElement(By.xpath("//input[@id='username' and @name='userID']"));
```
　　6、使用属性名来定位元素
```js
　　driver.findElement(By.xpath("//input[@button]"))
```
　　7、类似于cssSlector，使用部分属性值匹配元素
```js
starts-with()    driver.findElement(By.xpath("//input[stars-with(@id,'user')]"))
ends-with        driver.findElement(By.xpath("//input[ends-with(@id,'name')]"))
contains()        driver.findElement(By.xpath("//input[contains(@id,"ernam")]"))
```
　　8、使用任意属性值匹配元素
```js
　　driver.findElement(By.xpath("//input[@*='username']"))
```
　　9、使用xpath轴来定位元素
　　这里略了，详见w3school.com
# 使用innerText
　　1、使用cssSelector查找innerText定位元素
```js
　　driver.findElement(By.cssSelector("span[textContent='新闻']"));
```
　　2、使用xpath的text函数
```js
　　driver.findElement(By.xpath("//span[contains(text(),'hello')]"))  //包含匹配
　　driver.findElement(By.xpath("//span[text()='新闻']"))     //绝对匹配
```