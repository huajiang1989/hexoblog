---
title:  vuex 2.0 知识点详解
category:  vue
tags: vue
---

Vuex 是一个专门为 Vue.js 应该程序开发的状态管理模式，它类似于 Redux 应用于 React 项目中，他们都是一种 Flux 架构。相比 Redux，Vuex 更简洁，学习成本更低。

<!--more-->

注：本文针对 Vuex 2.0 的语法，目前通过 npm 默认下载的版本为 1.0+ ，想引入 2.0 版本可以通过 script 标签引入。
```html
<script src="https://unpkg.com/vuex@2.0.0"></script>
```
习惯使用 ES6 中 import 方法的可以暂时通过解构赋值的方式引入 Vuex 中的方法。
```js
import { mapState, mapGetters } from 'Vuex';
//替换为：
let { mapState, mapGetters } = Vuex;
```

Vuex 的核心内容主要就是 State、Getters、Mutations、Actions 这四部分，也非常好理解。

# State
首先看如何申明一个 store

```js
import Vue from 'Vue';
import Vuex from 'Vuex';

Vue.use(Vuex);

let store = new Vuex.Store({
    state: {
        stateA: 'a',
        stateB: 'b',
        stateC: 'c'
    }
});

console.log(store.state.stateA); // a
```

在 store 中的 state 对象，可以理解为 Vue 实例中的 data 对象，它用来保存最基本的数据。

## 在 Vue 中获取 store 中的状态

```js
let app = new Vue({
　　 el: '#demo',
    template: `<h1>{{myState}}</h1>`,
    computed: {
         myState() {
            return store.state.stateA;
        }
    }
});
```

最简单的方式就是通过 Vue 中的计算属性(computed) 来将 store 中的状态映射为 Vue 的数据。但是当数据多时这种方法明显效率过低，所以 Vuex 中提供了 mapState 方法用于批量映射 store 中的状态。

首先必须在 Vue 中注册 store 选项，这样整个 store 就从根节点注册到 Vue 下的每一个子组件里了。

```js
import { mapState } from 'Vuex';

let app = new Vue({
    el: '#demo',
    store,
    data: {
        local: 'L'
    },
    computed: mapState({
        stateA: state => state.stateA,
        stateB: 'stateB',
        stateC(state) {
            return state.stateC + this.local;
        }
    })
});
```
上例中，a. 可以通过 ES6 中的箭头函数进行数据的映射，b. 当计算属性的名称与 state 的属性名一致时可能直接通过字符串赋值，c. 当需要引用上下文中的 data 属性实，只能通过常规函数来使 this 生效。

如果所有计算属性的名称都与 state 一致，可以在 mapState 中以数组的方式进行映射。如果 Vue 中已经存在计算属性，可以通过 ES6 的展开操作符 (...) 进行组合。

```js
let app = new Vue({
    el: '#demo',
    store,
    computed: {
        local() {
             return 'Local';
        },
        ...mapState(['stateA', 'stateB', 'stateC'])
    }
});
```


# Getters
当需要对 store 中的数据进行处理，或者需要对处理后的数据进行复用，就可以使用 Getters 来处理，Getters 也可以理解为 Vue 中的计算属性 (computed)。

```js
let store = new Vuex.Store({
    state: {
        nowDate: new Date()
    },
    getters: {
        dateFormat(state, getters) {
            let date = state.nowDate;
            return `${date.getFullYear()}-${date.getMonth()+1}-${date.getDate()} / ${date.getHours()}:${date.getMinutes()}`;
        }
    }
});

console.log('The time is now:', store.getters.dateFormat); // The time is now: 2017-2-10 / 17:28
```
getters 接收两个参数，1. state、2. getters 本身，可以引用其他 getters。与 mapState 类似，Getters 也拥有 mapGetters 方法，用于批量映射。

```js
let { mapGetters } from 'Vuex';

let comonent = {
    computed: {
        ...mapGetters([
            'nowDate'
        ])
    }
};
```


# Mutations
在 Vue 中，data 值是可以直接被更改的。但是在 Vuex 中，不能直接对 state 进行操作，唯一的方法就是提交 mutation。mutation 可以理解为 Vue 中的 method 事件，只不过调用 mutation 需要特别的方法。

```js
let store = new Vuex.Store({
    state: {
        count: 0
    },
    mutations: {
        addCount(state) {
            state.count ++;
        }
    }
});

store.commit('addCount');

console.log(store.state.count); // 1
```
每一个 mutation 都有一个字符串的事件类型和一个回调函数。通常在回调函数中进行状态更改，通过 store.commit 触发事件。

## 传参

```js
// ...
mutations: {
    addCount(state, n) {
        state.count += n;
    }
}

store.commit('addCount', 10);
```
 这种方式有一个问题，一旦参数多了，就必须保证传入的顺序不能出错，可读性也很低。所以通常将参数以对象的形式传入，同时 mutaion 的事件类型字符串也可以使用对象的形式传入。

```js
// ...
mutations: {
    addCount(state, params) {
        state.count += params.num;
    }
}

store.commit('addCount', {
    num: 10
});

store.commit({
    type: 'addCount',
    num: 10
});
```
这里有一个规则需要遵守，在 mutation 中更改 state 应该以新对象替换老对象，不要在直接原对象上直接修改。*熟悉 React 的朋友们应该知道，在使用 setState 更新状态时也是同样的规则。

通过 ES6 的展开操作符可以很容易的完成。
```js
state.obj = { ...state.obj, newState: 123 };
```
## 在 Vue 组件中提交 mutaion
```js
this.$store.commit('xxx');
```
在组件中可以通过上述方法提交 commit，不过 Vuex 提供了 mapMutations 方法用于将 mutation 映射到组件中的 method 中。与 mapState、mapGetters 相似，这里就不再赘述了。

```js
import { mapMutations } from 'vuex'

const component = {
    // ...
    methods: {
        ...mapMutations([
            'addCount' // 映射 this.addCount() 为 this.$store.commit('addCount')
        ]),
        ...mapMutations({
            add: 'addCount' // 映射 this.add() 为 this.$store.commit('addCount')
        })
    }
}
```
## mutation 必须是同步函数

我们先试着写一个异步的 mutation ，看看会发生什么。

```js
// ...
mutations: {
    asyncAdd(state) {
        setTimeout(() => {
            state.count ++;
        }, 2000);
    }
}

store.commit('asyncAdd');
```
经测试，在 mutaion 里进行异步操作也是会生效的，2秒之后 state.count 确实发生改变。

那为什么还要强调 mutation 必须是同步的呢？因为在触发 commit 之后，回调函数还没有被调用，所以这次 mutation 的修改也是无法被调试工具所记录的。

如何对 state 进行异步操作呢，就要使用下面的 Action 了。



# Actions
Action 类似于 mutation，不同在于：

1. Action 不直接更改状态，而是提交 mutation

2. Action 可以包含任何异步操作

```js
const store = new Vuex.Store({
    state: {
        count: 0
    },
    mutations: {
        addCount(state) {
            state.count ++;
        }
    },
    actions: {
       asyncAdd(context) {
           setTimeout(() => {
               context.commit('addCount');
           }, 2000);
       }
    }
})
```
Action 中的回调函数会接收一个上下文 context 对象，它包含了当前 store 中所有属性和方法，但其不是 store 本身。你可以通过 context.commit 来提交 mutation，也可以通过 context.state 与 context.getters 来获取 state 和 getters。

当需要多次调用 commit 时，可以使用 ES6 的语法对传入的参数进行解构。

```js
actions: {
    asyncAdd({ commit }) {
        commit('addCount');
    }
}
```
## 分发 Action 与传参

Action 是通过 store.dispatch 方法来触发，传参方式与 store.commit 类似。

```js
store.dispatch('asyncAdd');

store.dispatch('asyncAdd', {
    num: 10
});

store.dispatch({
    type: 'asyncAdd',
    num: 10
});
```
## 在 Vue 组件中分发 Action
```js
this.$store.dispatch('xxx');
```
可以使用上述方式，同时 Vuex 中也提供了 mapActions 方法用于批量映射于组件中的 methods 中，与 mapMutations 类似。

```js
import { mapActions } from 'vuex'

const component = {
    // ...
    methods: {
        ...mapActions([
            'asyncAdd' // 映射 this.asyncAdd() 为 this.$store.dispatch('asyncAdd')
        ]),
        ...mapActions({
            add: 'asyncAdd' // 映射 this.add() 为 this.$store.dispatch('asyncAdd')
        })
    }
}
```
## 组合 Actions

既然 Action 是异步行为，那我们可以使用 ES6 中的 Promise 对象进行组合。

```js
const store = {
    actions: {
        asyncActionA({ commit }) {
            return new Promise((resolve, reject) => {
                setTimeout(() => {
                    commit('asyncAdd');
                    resolve();
                }, 2000);
            });
        },
        asyncActionB({ dispatch }, params) {
            return dispatch('asyncActionA').then(() => {
                console.log('Action complete at: ', params.date);
            });
        }
    }
}

store.dispatch('asyncActionB', {
    date: (new Date()).getTime() // 2秒后打印 Action complete at: xxxxxxxx (当前时间毫秒数)
});
```