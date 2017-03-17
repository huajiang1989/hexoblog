---
title: vue.js插件之vue-validator
category:  vue
tags: vue
---

在线地址：
```js
<script src="https://cdn.jsdelivr.net/vue.validator/2.1.6/vue-validator.min.js"></script>
```
<!--more-->
# v-validate 指令
用法如下:
```bash
v-validate[:field]="array literal | object literal | binding"
```
(1).可以通过 field 属性来指定字段名。（即目标的id）
(2).字面量（验证规则）
1）数组
```html
<validator name="validation">
  <form novalidate>
    Zip: <input type="text" v-validate:zip="['required']"><br />
    <div>
      <span v-if="$validation.zip.required">Zip code is required.</span>
    </div>
  </form>
</validator>
```
因为 required 验证器不要额外的参数，这样写更简洁。
2）对象
```html
<validator name="validation">
  <form novalidate>
ID: <input type="text" v-validate:id="{ required: true, minlength: 3, maxlength: 16 }"><br />
（或者使用严苛模式对象： ID: <input type="text" v-validate:id="{ minlength: { rule: 3 }, maxlength: { rule: 16 } }"><br />）
    <div>
      <span v-if="$validation.id.required">ID is required</span>
      <span v-if="$validation.id.minlength">Your ID is too short.</span>
      <span v-if="$validation.id.maxlength">Your ID is too long.</span>
    </div>
  </form>
</validator>
```
使用对象型字面量允许你为验证器指定额外的参数。
3）绑定
```js
new Vue({
  el: '#app',
  data: {
    rules: {
      minlength: 3,
      maxlength: 16
    }
  }
})
<div id="app">
  <validator name="validation">
    <form novalidate>
      ID: <input type="text" v-validate:id="rules"><br />
      <div>
        <span v-if="$validation.id.minlength">Your ID is too short.</span>
        <span v-if="$validation.id.maxlength">Your ID is too long.</span>
      </div>
    </form>
  </validator>
</div>
```
除数据属性外，也可以使用计算属性或事例方法来指定验证规则。

# 规则介绍
```bash
Build-in validators
You can be used build-in validators of below:
• required: whether the value has been specified
• pattern: whether the pattern of the regular expression
• minlength: whether the length of specified value is less than or equal minimum length
• maxlength: whether the length of specified value is less more or equal maximum length
• min: whether the specified numerical value is less than or equal minimum
• max: whether the specified numerical value is more than or equal maximum

• required: 是否是必须的
• pattern: 正则表达式
• minlength: 最小长度
• maxlength: 最大长度
• min: 最小值
• max: 最小值
```
# 可验证的表单元素
复选框 单选按钮 下拉列表

# 验证结果
字段验证结果
```bash
• valid: 字段有效时返回 true,否则返回 false。
• invalid: valid 的逆.
• touched: 字段获得过焦点时返回 true,否则返回 false。
• untouched: touched 的逆.
• modified: 字段值与初始值不同时返回 true,否则返回 false。
• dirty: 字段值改变过至少一次时返回 true,否则返回 false。
• pristine: dirty 的逆.
• errors: 字段无效时返回存有错误信息的数据，否则返回 undefined。
全局结果
• valid: 所有字段都有效时返回 true,否则返回 false。
• invalid: 只要存在无效字段就返回 true,否则返回 false。
• touched: 只要存在获得过焦点的字段就返回 true,否则返回 false。
• untouched: touched 的逆。
• modified: 只要存在与初始值不同的字段就返回 true,否则返回 false。
• dirty: 只要存在值改变过至少一次的字段就返回 true,否则返回 false。
• pristine: 所有字段都没有发生过变化时返回 true,否则返回 false。
• errors: 有无效字段时返回所有无效字段的错误信息，否则返回 undefined。
```
验证结果保存在如下结构中:
```json
{// top-level validation properties
  valid: true,
  invalid: false,
  touched: false,
  undefined: true,
  dirty: false,
  pristine: true,
  modified: false,
  errors: [{
    field: 'field1', validator: 'required', message: 'required field1'
  }, ... {
    field: 'fieldX', validator: 'customValidator', message: 'invalid fieldX'
  }],
  // field1（id） validation
  field1: {
    required: false, // build-in validator, return `false` or `true`
    email: true, // custom validator
    url: 'invalid url format', // custom validator, if specify the error message in validation rule, set it
    ...
    customValidator1: false, // custom validator
    // field validation properties
    valid: false,
    invalid: true,
    touched: false,
    undefined: true,
    dirty: false,
    pristine: true,
    modified: false,
    errors: [{
      validator: 'required', message: 'required field1'
    }]
  },
  ...
  // fieldX validation
  fieldX: {
    min: false, // validator
    ...
    customValidator: true,
    // fieldX validation properties
    valid: false,
    invalid: true,
    touched: true,
    undefined: false,
    dirty: true,
    pristine: false,
    modified: true,
    errors: [{
      validator: 'customValidator', message: 'invalid fieldX'
    }]
  },
}
```
验证结果会关联到验证器元素上。在上例中，验证结果保存在 $validation1 下，$validation1 是由 validator 元素的 name 属性值加 $ 前缀组成。支持把验证字段分组：
```html
<validator name="validation1" :groups="['user', 'password']">
  username: <input type="text" group="user" v-validate:username="['required']"><br />
  password: <input type="password" group="password" v-validate:password1="{ minlength: 8, required: true }"/><br />
password (confirm): <input     type="password"     group="password"     v-validate:password2="{ minlength: 8, required: true }"/>
  <div class="user">
    <span v-if="$validation1.user.invalid">Invalid yourname !!</span>
  </div>
  <div class="password">
    <span v-if="$validation1.password.invalid">Invalid password input !!</span>
  </div>
</validator>
```
# 指定错误消息
有时候你只需要输出部分错误消息，此时你可以通过 group 或 field 属性来指定这部分验证结果。
• group: 指定组的错误消息 (例如 $validation.group1.errors)
• field: 指定字段的错误消息 (例如 $validation.field1.errors)
错误消息
错误消息可以直接在验证规则中指定，同时可以在 v-show 和 v-if 中使用错误消息：
```html
  <span v-if="$validation1.password.required">{{ $validation1.password.required }}</span>
 <span v-if="$validation1.password.minlength">{{ $validation1.password.minlength }}</span>
```
也可以在 v-for 指令中使用错误消息：
```html
<ul>
      <li v-for="error in $validation1.errors">
        <p>{{error.field}}: {{error.message}}</p>
      </li>
</ul>
```
# 自定义验证器
## 全局注册
可以使用 Vue.validator 方法注册自定义验证器。
提示: Vue.validator asset 继承自 Vue.js 的 asset 管理系统.
通过下例中的 email 自定义验证器详细了解 Vue.validator 的使用方法：
// Register email validator function.
```js
Vue.validator('email', function (val) {
  return /^(([^<>()[\]\\.,;:\s@\"]+(\.[^<>()[\]\\.,;:\s@\"]+)*)|(\".+\"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/.test(val)
})
new Vue({
  el: '#app'
  data: {
    email: ''
  }
})
<div id="app">
  <validator name="validation1">
    address: <input type="text" v-validate:address="['email']"><br />
    <div>
      <p v-show="$validation1.address.email">Invalid your mail address format.</p>
    </div>
  <validator>
</div>
```
## 局部注册
可以通过组件的 validators 选项注册只能在组件内使用的自定义验证器。
自定义验证器是通过在组件的 validators 下定义验证通过返回真不通过返回假的回调函数来实现。
下例中注册了 numeric 和 url 两个自定义验证器：
```js
new Vue({
  el: '#app',
  validators: { // `numeric` and `url` custom validator is local registration
    numeric: function (val/*,rule*/) {
      return /^[-+]?[0-9]+$/.test(val)
    },
    url: function (val) {
      return /^(http\:\/\/|https\:\/\/)(.{4,})$/.test(val)
    }
  },
  data: {
    email: ''
  }
})
<div id="app">
  <validator name="validation1">
    username: <input type="text" v-validate:username="['required']"><br />
    email: <input type="text" v-validate:address="['email']"><br />
    age: <input type="text" v-validate:age="['numeric']"><br />
    site: <input type="text" v-validate:site="['url']"><br />
    <div class="errors">
      <p v-if="$validation1.username.required">required username</p>
      <p v-if="$validation1.address.email">invalid email address</p>
      <p v-if="$validation1.age.numeric">invalid age value</p>
      <p v-if="$validation1.site.url">invalid site uril format</p>
    </div>
  <validator>
</div>
```
# 延迟初始化
如果在 validator 元素上设置了 lazy 属性，那么验证器直到 $activateValidator() 被调用时才会进行初始化。这在待验证的数据需要异步加载时有用，避免了在得到数据前出现错误提示。下例中在得到评论内容后验证器才开始工作；如果不设置 lazy 属性，在得到评论内容前会显示错误提示。
```html
<!-- comment component -->
<div>
  <h1>Preview</h1>
  <p>{{comment}}</p>
  <validator lazy name="validation1">
    <input type="text" :value="comment" v-validate:comment="{ required: true, maxlength: 256 }"/>
    <span v-if="$validation1.comment.required">Required your comment</span>
    <span v-if="$validation1.comment.maxlength">Too long comment !!</span>
    <button type="button" value="save" @click="onSave" v-if="valid">
  </validator>
</div>
```
```js
Vue.component('comment', {
  props: {
    id: Number,
  },
  data: function () {
    return { comment: '' }
  },
  activate: function (done) {
    var resource = this.$resource('/comments/:id');
    resource.get({ id: this.id }, function (comment, stat, req) {
      this.comment =  comment.body
      // activate validator
      this.$activateValidator()
      done()
    }.bind(this)).error(function (data, stat, req) {
      // handle error ...
      done()
    })
  },
  methods: {
    onSave: function () {
      var resource = this.$resource('/comments/:id');
      resource.save({ id: this.id }, { body: this.comment }, function (data, stat, req) {
        // handle success
      }).error(function (data, sta, req) {
        // handle error
      })
    }
  }
})
```
# 自定义验证时机
默认情况下，vue-validator 会根据 validator 和 v-validate 指令自动进行验证。然而有时候我们需要关闭自动验证，在有需要时手动触发验证。
initial
当 vue-validator 完成初始编译后，会根据每一条 v-validate 指令自动进行验证。如果你不需要自动验证，可以通过 initial 属性或 v-validate 验证规则来关闭自动验证。
(1)initial
```html
<input id="username" type="text" initial="off" v-validate:username="['required', 'exist']">
```
(2) v-validate 验证规则
detect-blur and detect-change
vue-validator 会在检测到表单元素(input, checkbox, select 等)上的 DOM 事件(input, blur, change)时自动验证。此时，可以使用 detect-change 和 detect-blur 属性：
```html
<input id="username" type="text"
          detect-change="off" detect-blur="off" v-validate:username="{
          required: { rule: true, message: 'required you name !!' }
        }" />
```
# 异步验证
当在需要进行服务器端验证，可以使用异步验证，如下例：
```html
<template>
  <validator name="validation">
    <form novalidate>
      <h1>user registration</h1>
      <div class="username">
        <label for="username">username:</label>
        <input id="username" type="text"
          detect-change="off" v-validate:username="{
          required: { rule: true, message: 'required your name !!' },
          exist: { rule: true, initial: 'off' }
        }" />
        <span v-if="checking">checking ...</span>
      </div>
      <div class="errors">
        <validator-errors :validation="$validation"></validator-errors>
      </div>
      <input type="submit" value="register" :disabled="!$validation.valid" />
    </form>
  </validator>
</template>
```
```js
function copyOwnFrom (target, source) {
  Object.getOwnPropertyNames(source).forEach(function (propName) {
    Object.defineProperty(target, propName, Object.getOwnPropertyDescriptor(source, propName))
  })
  return target
}
function ValidationError () {
  copyOwnFrom(this, Error.apply(null, arguments))
}
ValidationError.prototype = Object.create(Error.prototype)
ValidationError.prototype.constructor = ValidationError
// exmpale with ES2015
export default {
  validators: {
    data () {
      return { checking: false }
    },
    exist (val) {
      this.vm.checking = true // spinner on
      return fetch('/validations/exist', {
        method: 'post',
        headers: {
          'Accept': 'application/json',
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          username: val
        })
      }).then((res) => {
        this.vm.checking = false // spinner off
        return res.json()
      }).then((json) => {
        return Object.keys(json).length > 0
          ? Promise.reject(new ValidationError(json.message))
          : Promise.resolve()
      }).catch((error) => {
        if (error instanceof ValidationError) {
          return Promise.reject(error.message)
        } else {
          return Promise.reject('unexpected error')
        }
      })
    }
  }
}
```
异步验证接口在异步验证时，可以使用如下两类接口：

函数需要实现一个返回签名为 function (resolve, reject) 如同 promise 一样的函数的自定义验证器。函数参数解释如下：
• 验证结果
o 成功时: resolve
o 失败时: reject

promise需要实现一个返回 promise 的自定义验证器。根据验证结果来 resolve 或 reject。
使用错误消息
如上例所示，在服务器端验证错误发生时，可以使用服务器端返回的错误消息。
验证器函数 context
验证器函数 context 是绑定到 Validation 对象上的。Validation 对象提供了一些属性，这些属性在实现特定的验证器时有用。
vm 属性
暴露了当前验证所在的 vue 实例。
the following ES2015 example:
```js
new Vue({
  data () { return { checking: false } },
  validators: {
    exist (val) {
      this.vm.checking = true // spinner on
      return fetch('/validations/exist', {
        // ...
      }).then((res) => { // done
        this.vm.checking = false // spinner off
        return res.json()
      }).then((json) => {
        return Promise.resolve()
      }).catch((error) => {
        return Promise.reject(error.message)
      })
    }
  }
})
```
el 属性暴露了当前验证器的目标 DOM 元素。下面展示了结合 International Telephone Input jQuery 插件使用的例子：
```js
new Vue({
  validators: {
    phone: function (val) {
      return $(this.el).intlTelInput('isValidNumber')
    }
  }
})
```