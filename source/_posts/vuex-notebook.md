---
title:  vuex 创建一个笔记本项目
category:  vue
tags: vue
---

我们会通过这个例子解释相应的概念，以及 Vuex 所要解决的问题：如何管理一个包含许多组件的大型应用。我们假定这个例子使用以下四个组件：最外层(App.vue),左边(toolbar),中间(noteslist),右边(editor)
<!--more-->
![](http://upload-images.jianshu.io/upload_images/1444794-d346609162594f9b.png?%0AimageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# App.vue
app 组件，它包含了另外三个子组件：

- toolbar
- noteslist
- editor
```html
<template>
  <div id="app">
    <toolbar></toolbar>
    <notes-list></notes-list>
    <editor></editor>
  </div>
</template>

<script>
import Toolbar from './components/toolbar'
import NotesList from './components/noteslist'
import Editor from './components/editor'

export default {
  components: {
    Toolbar, NotesList, Editor
  }
}
</script>
<style type="text/css">
@import url(https://fonts.googleapis.com/css?family=Raleway:400,300);
html, #app {
  height: 100%;
}
body {
  margin: 0;
  padding: 0;
  border: 0;
  height: 100%;
  max-height: 100%;
  position: relative;
}
#toolbar {
  float: left;
  width: 80px;
  height: 100%;
  background-color: #30414D;
  color: #767676;
  padding: 35px 25px 25px 25px;
}
#notes-list {
  float: left;
  width: 300px;
  height: 100%;
  background-color: #F5F5F5;
  font-family: 'Raleway', sans-serif;
  font-weight: 400;
}
#list-header {
  padding: 5px 25px 25px 25px;
}
#list-header h2 {
  font-weight: 300;
  text-transform: uppercase;
  text-align: center;
  font-size: 22px;
  padding-bottom: 8px;
}
#notes-list .container {
  height: calc(100% - 137px);
  max-height: calc(100% - 137px);
  overflow: auto;
  width: 100%;
  padding: 0;
}
#notes-list .container .list-group-item {
  border: 0;
  border-radius: 0;
}
.list-group-item-heading {
  font-weight: 300;
  font-size: 15px;
}
#note-editor {
  height: 100%;
  margin-left: 380px;
}
#note-editor textarea {
  height: 100%;
  border: 0;
  border-radius: 0;
}
#toolbar i {
  font-size: 30px;
  margin-bottom: 35px;
  cursor: pointer;
  opacity: 0.8;
  transition: opacity 0.5s ease;
}
#toolbar i:hover {
  opacity: 1;
}
.starred {
  color: #F7AE4F;
}
</style>
```

component/toolbar.vue
```html
<template>
  <div id="toolbar">
    <i class="glyphicon glyphicon-plus"></i>
    <i class="glyphicon glyphicon-star"></i>
    <i class="glyphicon glyphicon-remove"></i>
  </div>
</template>
<script>
export default {}
</script>
```
component/noteslist.vue
```html
<template>
  <div id="notes-list">
    <div id="list-header">
      <h2>Notes | coligo</h2>
      <div class="btn-group btn-group-justified" role="group">
        <!-- All Notes button -->
        <div class="btn-group" role="group">
          <button type="button" class="btn btn-default">
            All Notes
          </button>
        </div>
        <!-- Favorites Button -->
        <div class="btn-group" role="group">
          <button type="button" class="btn btn-default">
            Favorites
          </button>
        </div>
      </div>
    </div>
    <!-- render notes in a list -->
    <div class="container">
      <div class="list-group">
        <a class="list-group-item" href="#">
          <h4 class="list-group-item-heading">
            标题列表
          </h4>
        </a>
      </div>
    </div>
  </div>
</template>
<script>
export default {}
</script>
```
component/editor.vue
```html
<template>
  <div id="note-editor">
    <textarea class="form-control">
    </textarea>
  </div>
</template>

<script>
export default {}
</script>
```
# 第一步：加入 store

store 存储应用所需的数据。所有组件都从 store 中读取数据，在我们开始之前，先用 npm 安装 vuex
```bash
$ npm install --save vuex
```
建立一个新的文件，在根目录下创建 vuex/store.js
```js
import Vue from 'vue'
import Vuex from 'vuex'

// 使用 vuex
Vue.use(Vuex)

// 创建一个对象来保存应用启动时的初始状态
const state = {
  // TODO 放置初始状态
}

// 创建一个对象存储一系列我们接下来要写的 mutation 函数
const mutations = {
    // TODO 放置我们的状态变更函数
}

// 整合初始状态和变更函数，我们就得到了我们所需的 store
// 至此，这个 store 就可以链接到我们的应用中
export default new Vuex.Store({
  state, mutations
})
```
我们需要将创建的 store 让整个项目发现，所以这个时候需要修改 main.js
修改 main.js，注入 store
```js
import Vue from 'vue'

import App from './App'
import store from './vuex/store'

new Vue({
  template: '<App/>',
  store,
  components: { App }
}).$mount('#app')
```
# 第二步：创建 action

action 是一种专门用来被 component 调用的函数，action 函数能够通过分发相应的 mutation 函数，来触发对 store 的更新。action 也可以先从 HTTP 后端或 store 中读取其他数据之后再分发更新事件。

创建一个新文件 vuex/action.js，然后写入相关函数
```js
// 新增笔记
export const addNote = ({ commit, state }) => {
  commit('ADD_NOTE')
}

// 修改笔记
export const editNote = ({ commit, state }, e) => {
  commit('EDIT_NOTE', e.target.value)
}

// 删除笔记
export const deleteNote = ({ commit, state }) => {
  commit('DELETE_NOTE')
}

// 更新当前选中笔记
export const updateActiveNote = ({ commit, state }, note) => {
  commit('SET_ACTIVE_NOTE', note)
}

// 选中模块按钮（All note、Favorites）
export const toggleFavorite = ({ commit, state }) => {
  commit('TOGGLE_FAVORITE')
}
```
回顾一下我们刚刚添加的内容背后所潜藏的一些有趣的点：

- 我们有了一个新对象 vuex.actions，包含着新的 action；
- 我们没有指定特定的 store，object，state 等等。Vuex 会自动把它们串联起来；
- 我们可以用 this.addNote () 在任何方法中调用此 action；
- 我们也可以通过 @click 参数调用它，与使用其他普通的 Vue 组件方法并无二致；
- 我们给 action 起名叫 addNote，但是在具体使用时，我们可以根据需要进行重新命名
# 第三步：创建 state 和 mutation

我们在 vuex/actions.js 文件里，添加了相关的 mutation ，但是我们还没有写它所对应的具体操作，现在就让我们来将这些方法暴露出来。

修改 vuex/store.js
```js
import Vue from 'vue'
import Vuex from 'vuex'
import * as actions from './actions'
import * as getters from './getters'

// 使用 vuex
Vue.use(Vuex)

// 创建一个对象来保存应用启动时的初始状态
const state = {
  // TODO 放置初始状态
  count: 0,
  notes: [],
  activeNote: []
}

// 创建一个对象存储一系列我们接下来要写的 mutation 函数
const mutations = {
  // TODO 放置我们的状态变更函数
  increment (state, amount) {
    state.count = state.count + amount
  },
  decrement (state, amount) {
    state.count = state.count - amount
  },
  DECREMENT (state, amount) {
    state.count = state.count - amount
  },
  ADD_NOTE (state) {
    console.log(state)
    const newNote = {
      text: 'New Note',
      favorite: false
    }
    state.notes.push(newNote)
    state.activeNote = newNote
  },

  EDIT_NOTE (state, text) {
    state.activeNote.text = text
  },

  DELETE_NOTE (state) {
    state.notes.$remove(state.activeNote)
    state.activeNote = state.notes[0]
  },

  TOGGLE_FAVORITE (state) {
    state.activeNote.favorite = !state.activeNote.favorite
  },

  SET_ACTIVE_NOTE (state, note) {
    state.activeNote = note
  }
}

// 整合初始状态和变更函数，我们就得到了我们所需的 store
// 至此，这个 store 就可以链接到我们的应用中
export default new Vuex.Store({
  actions,
  getters,
  state,
  mutations
})
```
# 第四步： 创建 getter

在 store 中的数据，我们可以通过创建一个新的文件 getter 来统一获取方法，这样子不仅便于管理，有时候很多地方使用同一个方法，此时我们不需要修改一大堆页面，只需要修改 getter.js 中的方法实现就可以。

创建一个新文件 vuex/getter.js ，然后编写代码：
```js
/**
 * 在 ES6 里你可以这样写
 * export const getCount = state => state.count
 */
export const notes = state => state.notes

export const activeNote = state => state.activeNote

export const activeNoteText = state => state.activeNote.text
```
# 第五步：在组件中获取数据，并且调用 action 方法

修改 vuex/toolbar.vue
```html
<template>
  <div id="toolbar">
    <i @click="addNote" class="glyphicon glyphicon-plus"></i>
    <i @click="toggleFavorite"
      class="glyphicon glyphicon-star"
      :class="{starred: activeNote.favorite}"></i>
    <i @click="deleteNote" class="glyphicon glyphicon-remove"></i>
  </div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'

export default {
  computed: mapGetters({
    activeNote: 'activeNote'
  }),
  methods: mapActions([
    'addNote',
    'deleteNote',
    'toggleFavorite'
  ])
}
</script>
```
修改 vuex/noteslist.vue
```html
<template>
  <div id="notes-list">

    <div id="list-header">
      <h2>Notes | coligo</h2>
      <div class="btn-group btn-group-justified" role="group">
        <!-- All Notes button -->
        <div class="btn-group" role="group">
          <button type="button" class="btn btn-default"
            @click="show = 'all'"
            :class="{active: show === 'all'}">
            All Notes
          </button>
        </div>
        <!-- Favorites Button -->
        <div class="btn-group" role="group">
          <button type="button" class="btn btn-default"
            @click="show = 'favorites'"
            :class="{active: show === 'favorites'}">
            Favorites
          </button>
        </div>
      </div>
    </div>
    <!-- render notes in a list -->
    <div class="container">
      <div class="list-group">
        <a v-for="note in filteredNotes"
          class="list-group-item" href="#"
          :class="{active: activeNote === note}"
          @click="updateActiveNote(note)">
          <h4 class="list-group-item-heading">
            {{note.text.trim().substring(0, 30)}}
          </h4>
        </a>
      </div>
    </div>

  </div>
</template>
<script>
import { mapGetters, mapActions } from 'vuex'

export default {
  data () {
    return {
      show: 'all'
    }
  },
  computed: {
    ...mapGetters([
      'notes', 'activeNote'
    ]),
    filteredNotes () {
      if (this.show === 'all') {
        return this.notes
      } else if (this.show === 'favorites') {
        return this.notes.filter(note => note.favorite)
      }
    }
  },
  methods: mapActions([
    'updateActiveNote'
  ])
}
</script>
```
修改 vuex/editor.vue
```html
<template>
  <div id="note-editor">
    <textarea
      :value="activeNoteText"
      @input="editNote"
      class="form-control">
    </textarea>
  </div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'

export default {
  computed: {
    ...mapGetters([
      'activeNoteText'
    ])
  },
  methods: {
    ...mapActions([
      'editNote'
    ])
  }
}
</script>
```
这个时候，运行下你的程序，它可以正常工作了。
```bash
$ npm run dev
```
最后，总结下编写该案例时遇到的坑：

注意该案例使用的是 vue2.0 和 vuex2.0，安装插件时请不要装错；
如果使用 ES6，babel，那么请在 .babelrc 中 使用 stage-3 和 transform-object-rest-spread;
```json
{
   "presets": ["es2015", "stage-3"],
   "plugins": ["transform-runtime", "transform-object-rest-spread"],
   "comments": false
}
```