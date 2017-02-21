
---
title:  vuex代码样例
category:  vue
tags: vue
---
这个文章主要的作用是熟悉 vue vue-router vuex 在初期搭建项目的时候该怎么配置，以及怎么去互相配合，xx.vue文件该怎么写，如何创建和使用组件，vuex的state，mutations，actions，getters怎么配合，形成一个完整的流程。
<!--more-->
# 路由配置
```js
import App from '../App'

export default [{
    path: '/',
    component: App,
    children: [{
        path: '',
        component: r => require.ensure([], () => r(require('../page/home')), 'home')
    }, {
        path: '/item',
        component: r => require.ensure([], () => r(require('../page/item')), 'item')
    }, {
        path: '/score',
        component: r => require.ensure([], () => r(require('../page/score')), 'score')
    }]
}]
```
# 配置actions
```js
import ajax from '../config/ajax'

export default {
    addNum({ commit, state }, id) {
        commit('REMBER_ANSWER', { id })
        if (state.itemNum < state.itemDetail.length) {
            commit('ADD_ITEMNUM', {
                num: 1
            })
        }
    },

    getData({ commit, state }) {
        ajax('GET', 'http://operating-activities.putao.com/happyfriday?active_topic_id=4').
        then(res => {
            commit('GET_DATA', {
                res
            })
        })
    },

    initializeData({ commit }) {
        commit('INITIALIZE_DATA')
    }
}
```
# mutations
```js
const GET_DATA = 'GET_DATA'
const ADD_ITEMNUM = 'ADD_ITEMNUM'
const REMBER_ANSWER = 'REMBER_ANSWER'
const REMBER_TIME = 'REMBER_TIME'
const INITIALIZE_DATA = 'INITIALIZE_DATA'
const GET_USER_INFORM = 'GET_USER_INFORM'

export default {
    [GET_DATA](state, payload) {
        if (payload.res.httpStatusCode == 200) {
            state.itemDetail = payload.res.topiclist;
        }
    },

    [GET_USER_INFORM](state, payload) {
        state.user_id = payload.res.users_id;
    },

    [ADD_ITEMNUM](state, payload) {
        state.itemNum += payload.num;
    },

    [REMBER_ANSWER](state, payload) {
        state.answerid[state.itemNum] = payload.id;
    },

    [REMBER_TIME](state) {
        state.timer = setInterval(() => {
            state.allTime++;
        }, 1000)
    },

    [INITIALIZE_DATA](state) {
        state.itemNum = 1;
        state.allTime = 0;
    },
}
```
# 创建store
```js
import Vue from 'vue'
import Vuex from 'vuex'
import mutations from './mutations'
import actions from './action'


Vue.use(Vuex)

const state = {
    level: '第一周',
    itemNum: 1,
    allTime: 0,
    timer: '',
    itemDetail: [],
    answerid: {}
}

export default new Vuex.Store({
    state,
    actions,
    mutations
})
```
# 创建vue实例
```js
import Vue from 'vue'
import VueRouter from 'vue-router'
import routes from './router/router'
import store from './store/'

Vue.use(VueRouter)
const router = new VueRouter({
    routes
})

new Vue({
    router,
    store,
}).$mount('#app')
```