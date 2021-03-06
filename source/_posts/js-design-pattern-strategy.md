---
title: js设计模式-策略模式
category:  js-design-pattern
tags: js-design-pattern
---


策略模式的定义是：定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。

使用策略模式的优点如下：

优点：
1. 策略模式利用组合，委托等技术和思想，有效的避免很多if条件语句。

2. 策略模式提供了开放-封闭原则，使代码更容易理解和扩展。

3. 策略模式中的代码可以复用。
<!--more-->
# 理解javascript中的策略模式
## 使用策略模式计算奖金

下面的demo是我在书上看到的，但是没有关系，我们只是来理解下策略模式的使用而已，我们可以使用策略模式来计算奖金问题；

比如公司的年终奖是根据员工的工资和绩效来考核的，绩效为A的人，年终奖为工资的4倍，绩效为B的人，年终奖为工资的3倍，绩效为C的人，年终奖为工资的2倍；现在我们使用一般的编码方式会如下这样编写代码：

```js
var calculateBouns = function(salary,level) {
    if(level === 'A') {
        return salary * 4;
    }
    if(level === 'B') {
        return salary * 3;
    }
    if(level === 'C') {
        return salary * 2;
    }
};
// 调用如下：
console.log(calculateBouns(4000,'A')); // 16000
console.log(calculateBouns(2500,'B')); // 7500
```
第一个参数为薪资，第二个参数为等级；

代码缺点如下：

calculateBouns 函数包含了很多if-else语句。

calculateBouns 函数缺乏弹性，假如还有D等级的话，那么我们需要在calculateBouns 函数内添加判断等级D的if语句；

算法复用性差，如果在其他的地方也有类似这样的算法的话，但是规则不一样，我们这些代码不能通用。

## 使用组合函数重构代码

组合函数是把各种算法封装到一个个的小函数里面，比如等级A的话，封装一个小函数，等级为B的话，也封装一个小函数，以此类推；如下代码：

```js
var performanceA = function(salary) {
    return salary * 4;
};
var performanceB = function(salary) {
    return salary * 3;
};

var performanceC = function(salary) {
    return salary * 2
};
var calculateBouns = function(level,salary) {
    if(level === 'A') {
        return performanceA(salary);
    }
    if(level === 'B') {
        return performanceB(salary);
    }
    if(level === 'C') {
        return performanceC(salary);
    }
};
// 调用如下
console.log(calculateBouns('A',4500)); // 18000
```
代码看起来有点改善，但是还是有如下缺点：

calculateBouns 函数有可能会越来越大，比如增加D等级的时候，而且缺乏弹性。

## 使用策略模式重构代码

策略模式指的是 定义一系列的算法，把它们一个个封装起来，将不变的部分和变化的部分隔开，实际就是将算法的使用和实现分离出来；算法的使用方式是不变的，都是根据某个算法取得计算后的奖金数，而算法的实现是根据绩效对应不同的绩效规则；

一个基于策略模式的程序至少由2部分组成，第一个部分是一组策略类，策略类封装了具体的算法，并负责具体的计算过程。第二个部分是环境类Context，该Context接收客户端的请求，随后把请求委托给某一个策略类。我们先使用传统面向对象来实现；

如下代码：

```js
var performanceA = function(){};
performanceA.prototype.calculate = function(salary) {
    return salary * 4;
};
var performanceB = function(){};
performanceB.prototype.calculate = function(salary) {
    return salary * 3;
};
var performanceC = function(){};
performanceC.prototype.calculate = function(salary) {
    return salary * 2;
};
// 奖金类
var Bouns = function(){
    this.salary = null;    // 原始工资
    this.levelObj = null;  // 绩效等级对应的策略对象
};
Bouns.prototype.setSalary = function(salary) {
    this.salary = salary;  // 保存员工的原始工资
};
Bouns.prototype.setlevelObj = function(levelObj){
    this.levelObj = levelObj;  // 设置员工绩效等级对应的策略对象
};
// 取得奖金数
Bouns.prototype.getBouns = function(){
    // 把计算奖金的操作委托给对应的策略对象
    return this.levelObj.calculate(this.salary);
};
var bouns = new Bouns();
bouns.setSalary(10000);
bouns.setlevelObj(new performanceA()); // 设置策略对象
console.log(bouns.getBouns());  // 40000

bouns.setlevelObj(new performanceB()); // 设置策略对象
console.log(bouns.getBouns());  // 30000
```
如上代码使用策略模式重构代码，可以看到代码职责更新分明，代码变得更加清晰。

## Javascript版本的策略模式

```js
//代码如下：
var obj = {
        "A": function(salary) {
            return salary * 4;
        },
        "B" : function(salary) {
            return salary * 3;
        },
        "C" : function(salary) {
            return salary * 2;
        }
};
var calculateBouns =function(level,salary) {
    return obj[level](salary);
};
console.log(calculateBouns('A',10000)); // 40000
```
可以看到代码更加简单明了；

策略模式指的是定义一系列的算法，并且把它们封装起来，但是策略模式不仅仅只封装算法，我们还可以对用来封装一系列的业务规则，只要这些业务规则目标一致，我们就可以使用策略模式来封装它们；

# 表单效验

比如我们经常来进行表单验证，比如注册登录对话框，我们登录之前要进行验证操作：比如有以下几条逻辑：

用户名不能为空

密码长度不能小于6位。

手机号码必须符合格式。

比如HTML代码如下：

```html
<form action = "http://www.baidu.com" id="registerForm" method = "post">
        <p>
            <label>请输入用户名：</label>
            <input type="text" name="userName"/>
        </p>
        <p>
            <label>请输入密码：</label>
            <input type="text" name="password"/>
        </p>
        <p>
            <label>请输入手机号码：</label>
            <input type="text" name="phoneNumber"/>
        </p>
</form>
```
我们正常的编写表单验证代码如下：

```js
var registerForm = document.getElementById("registerForm");
registerForm.onsubmit = function(){
    if(registerForm.userName.value === '') {
        alert('用户名不能为空');
        return;
    }
    if(registerForm.password.value.length < 6) {
        alert("密码的长度不能小于6位");
        return;
    }
    if(!/(^1[3|5|8][0-9]{9}$)/.test(registerForm.phoneNumber.value)) {
        alert("手机号码格式不正确");
        return;
    }
}
```
但是这样编写代码有如下缺点：

1.registerForm.onsubmit 函数比较大，代码中包含了很多if语句；

2.registerForm.onsubmit 函数缺乏弹性，如果增加了一种新的效验规则，或者想把密码的长度效验从6改成8，我们必须改registerForm.onsubmit 函数内部的代码。违反了开放-封闭原则。

3. 算法的复用性差，如果在程序中增加了另外一个表单，这个表单也需要进行一些类似的效验，那么我们可能又需要复制代码了；

下面我们可以使用策略模式来重构表单效验；

第一步我们先来封装策略对象；如下代码：

```js
var strategy = {
    isNotEmpty: function(value,errorMsg) {
        if(value === '') {
            return errorMsg;
        }
    },
    // 限制最小长度
    minLength: function(value,length,errorMsg) {
        if(value.length < length) {
            return errorMsg;
        }
    },
    // 手机号码格式
    mobileFormat: function(value,errorMsg) {
        if(!/(^1[3|5|8][0-9]{9}$)/.test(value)) {
            return errorMsg;
        }
    }
};
```
接下来我们准备实现Validator类，Validator类在这里作为Context，负责接收用户的请求并委托给strategy 对象，如下代码：

```js
var Validator = function(){
    this.cache = [];  // 保存效验规则
};
Validator.prototype.add = function(dom,rule,errorMsg) {
    var str = rule.split(":");
    this.cache.push(function(){
        // str 返回的是 minLength:6
        var strategy = str.shift();
        str.unshift(dom.value); // 把input的value添加进参数列表
        str.push(errorMsg);  // 把errorMsg添加进参数列表
        return strategys[strategy].apply(dom,str);
    });
};
Validator.prototype.start = function(){
    for(var i = 0, validatorFunc; validatorFunc = this.cache[i++]; ) {
        var msg = validatorFunc(); // 开始效验 并取得效验后的返回信息
        if(msg) {
            return msg;
        }
    }
};
```
Validator类在这里作为Context，负责接收用户的请求并委托给strategys对象。上面的代码中，我们先创建一个Validator对象，然后通过validator.add方法往validator对象中添加一些效验规则，validator.add方法接收3个参数，如下代码：
```js
validator.add(registerForm.password,'minLength:6','密码长度不能小于6位');
```
registerForm.password 为效验的input输入框dom节点；

minLength:6： 是以一个冒号隔开的字符串，冒号前面的minLength代表客户挑选的strategys对象，冒号后面的数字6表示在效验过程中所必须验证的参数，minLength:6的意思是效验 registerForm.password 这个文本输入框的value最小长度为6位；如果字符串中不包含冒号，说明效验过程中不需要额外的效验信息；

第三个参数是当效验未通过时返回的错误信息；

当我们往validator对象里添加完一系列的效验规则之后，会调用validator.start()方法来启动效验。如果validator.start()返回了一个errorMsg字符串作为返回值，说明该次效验没有通过，此时需要registerForm.onsubmit方法返回false来阻止表单提交。下面我们来看看初始化代码如下：

```js
var validateFunc = function(){
    var validator = new Validator(); // 创建一个Validator对象
    /* 添加一些效验规则 */
    validator.add(registerForm.userName,'isNotEmpty','用户名不能为空');
    validator.add(registerForm.password,'minLength:6','密码长度不能小于6位');
    validator.add(registerForm.userName,'mobileFormat','手机号码格式不正确');

    var errorMsg = validator.start(); // 获得效验结果
    return errorMsg; // 返回效验结果
};
var registerForm = document.getElementById("registerForm");
registerForm.onsubmit = function(){
    var errorMsg = validateFunc();
    if(errorMsg){
        alert(errorMsg);
        return false;
    }
}
```
下面是所有的代码如下：

```js
var strategys = {
    isNotEmpty: function(value,errorMsg) {
        if(value === '') {
            return errorMsg;
        }
    },
    // 限制最小长度
    minLength: function(value,length,errorMsg) {
        if(value.length < length) {
            return errorMsg;
        }
    },
    // 手机号码格式
    mobileFormat: function(value,errorMsg) {
        if(!/(^1[3|5|8][0-9]{9}$)/.test(value)) {
            return errorMsg;
        }
    }
};
var Validator = function(){
    this.cache = [];  // 保存效验规则
};
Validator.prototype.add = function(dom,rule,errorMsg) {
    var str = rule.split(":");
    this.cache.push(function(){
        // str 返回的是 minLength:6
        var strategy = str.shift();
        str.unshift(dom.value); // 把input的value添加进参数列表
        str.push(errorMsg);  // 把errorMsg添加进参数列表
        return strategys[strategy].apply(dom,str);
    });
};
Validator.prototype.start = function(){
    for(var i = 0, validatorFunc; validatorFunc = this.cache[i++]; ) {
        var msg = validatorFunc(); // 开始效验 并取得效验后的返回信息
        if(msg) {
            return msg;
        }
    }
};

var validateFunc = function(){
    var validator = new Validator(); // 创建一个Validator对象
    /* 添加一些效验规则 */
    validator.add(registerForm.userName,'isNotEmpty','用户名不能为空');
    validator.add(registerForm.password,'minLength:6','密码长度不能小于6位');
    validator.add(registerForm.userName,'mobileFormat','手机号码格式不正确');

    var errorMsg = validator.start(); // 获得效验结果
    return errorMsg; // 返回效验结果
};
var registerForm = document.getElementById("registerForm");
registerForm.onsubmit = function(){
    var errorMsg = validateFunc();
    if(errorMsg){
        alert(errorMsg);
        return false;
    }
};
```
如上使用策略模式来编写表单验证代码可以看到好处了，我们通过add配置的方式就完成了一个表单的效验；这样的话，那么代码可以当做一个组件来使用，并且可以随时调用，在修改表单验证规则的时候，也非常方便，通过传递参数即可调用；

给某个文本输入框添加多种效验规则，上面的代码我们可以看到，我们只是给输入框只能对应一种效验规则，比如上面的我们只能效验输入框是否为空，validator.add(registerForm.userName,'isNotEmpty','用户名不能为空');但是如果我们既要效验输入框是否为空，还要效验输入框的长度不要小于10位的话，那么我们期望需要像如下传递参数：
```js
validator.add(registerForm.userName,[{strategy:’isNotEmpty’,errorMsg:’用户名不能为空’}，{strategy: 'minLength:6',errorMsg:'用户名长度不能小于6位'}])
```
我们可以编写代码如下：

```js
// 策略对象
var strategys = {
    isNotEmpty: function(value,errorMsg) {
        if(value === '') {
            return errorMsg;
        }
    },
    // 限制最小长度
    minLength: function(value,length,errorMsg) {
        if(value.length < length) {
            return errorMsg;
        }
    },
    // 手机号码格式
    mobileFormat: function(value,errorMsg) {
        if(!/(^1[3|5|8][0-9]{9}$)/.test(value)) {
            return errorMsg;
        }
    }
};
var Validator = function(){
    this.cache = [];  // 保存效验规则
};
Validator.prototype.add = function(dom,rules) {
    var self = this;
    for(var i = 0, rule; rule = rules[i++]; ){
        (function(rule){
            var strategyAry = rule.strategy.split(":");
            var errorMsg = rule.errorMsg;
            self.cache.push(function(){
                var strategy = strategyAry.shift();
                strategyAry.unshift(dom.value);
                strategyAry.push(errorMsg);
                return strategys[strategy].apply(dom,strategyAry);
            });
        })(rule);
    }
};
Validator.prototype.start = function(){
    for(var i = 0, validatorFunc; validatorFunc = this.cache[i++]; ) {
    var msg = validatorFunc(); // 开始效验 并取得效验后的返回信息
    if(msg) {
        return msg;
    }
    }
};
// 代码调用
var registerForm = document.getElementById("registerForm");
var validateFunc = function(){
    var validator = new Validator(); // 创建一个Validator对象
    /* 添加一些效验规则 */
    validator.add(registerForm.userName,[
        {strategy: 'isNotEmpty',errorMsg:'用户名不能为空'},
        {strategy: 'minLength:6',errorMsg:'用户名长度不能小于6位'}
    ]);
    validator.add(registerForm.password,[
        {strategy: 'minLength:6',errorMsg:'密码长度不能小于6位'},
    ]);
    validator.add(registerForm.phoneNumber,[
        {strategy: 'mobileFormat',errorMsg:'手机号格式不正确'},
    ]);
    var errorMsg = validator.start(); // 获得效验结果
    return errorMsg; // 返回效验结果
};
// 点击确定提交
registerForm.onsubmit = function(){
    var errorMsg = validateFunc();
    if(errorMsg){
        alert(errorMsg);
        return false;
    }
}
```
注意：如上代码都是按照书上来做的，都是看到书的代码，最主要我们理解策略模式实现，比如上面的表单验证功能是这样封装的代码，我们平时使用jquery插件表单验证代码原来是这样封装的，为此我们以后也可以使用这种方式来封装表单等学习；