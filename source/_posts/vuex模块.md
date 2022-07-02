---
title: Vuex 学习 0.2 模块
categories: Vue 
---

vuex 使用唯一的 store 对象来存储状态和操作状态的方法，但随着项目规模的增长，该对象的体量增大，包含多种状态，不便于管理。

vuex 支持将 store 分成多个模块，即让每一种数据都拥有自己独立的 state, mutations, actions 和 getters。 

### 模块的注册

一般创建单独的 modules 目录，一个模块一个文件，然后将module 导出，在index.js 中注册

目录结构如下 

```jsx
├─store
│  │ index.js
│  │
│  └─modules
│          m1.js
│          m2.js
```

模块注册

```jsx
// modules/m1.js
const m1 = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

// index.js 
import moduleA from "./modules/m1";

const store = createStore({
  modules: {
    moduleA
  }
})

```

### 局部状态（local state）

使用模块之后，不同模块拥有自己独立的 state

在模块内部注册的 getter， mutation，action， 注册方法与全局模式下相同，但默认传递的参数如 state， context.state，访问的是模块内部的 state

在组件中访问 state 时，需添加模块名前缀， 如 `$store.state.moduleA.localStateName`

下面是官方文档给的例子

```jsx
const moduleA = {
  state: () => ({
    count: 0
  }),
  mutations: {
    increment (state) {
      // `state` is the local module state
      state.count++
    }
  },
  getters: {
    doubleCount (state) {
      return state.count * 2
    }
  }
}
```

```jsx
const moduleA = {
  // ...
  actions: {
    incrementIfOddOnRootSum ({ state, commit, rootState }) {
      if ((state.count + rootState.count) % 2 === 1) {
        commit('increment')
      }
    }
  }
}
```

### 使用命名空间之后的 mutations，actions 和 getters

虽然模块具有自己的局部 state ，但其他操作  mutations，actions 和 getters 仍然默认全局注册。

比如 moduleA 和 moduleB 都有一个 setUser 的 mutation 操作，如果像之前一样在组件中提交 mutation ，`$store.commit(’setUser’, {name:’new user’})` , 则**两个组件的 setUser 都会响应**，将各自的局部状态 user 设置为 new user 

```jsx
// moudleA.js 
export default {
  //namespaced: true,
  state() {
    return {
      user: "user1",
    };
  },
  getter: {},
  mutations: {
    setUser(state, payload) {
      console.log("user1 === set new user");
      state.user = payload.name;
    },
  },
  actions: {},
};

//moduleB.js 
export default {
  //namespaced: true,
  state() {
    return {
      user: "user2",
    };
  },
  getter: {},
  mutations: {
    setUser(state, payload) {
      console.log("user2 === set new user");
      state.user = payload.name;
    },
  },
  actions: {},
};
```

更合理的操作是使用命名空间，即在模块中添加属性 `namespaced: true` ， 使每个模块都拥有自己的  getter， mutation 和 action 。使用命名空间之后的操作调用方法如下

```jsx
const store = createStore({
  modules: {
    account: {
      namespaced: true,

      // module assets
      state: () => ({ ... }), // module state is already nested and not affected by namespace option
      getters: {
        isAdmin () { ... } // -> getters['account/isAdmin']
      },
      actions: {
        login () { ... } // -> dispatch('account/login')
      },
      mutations: {
        login () { ... } // -> commit('account/login')
      },
	  }
	}
}
```

### 辅助映射函数的使用 （使用命名空间情况下）

使用方法（来自官方文档的例子）

第一个参数为模块名，第二个参数为映射选项，可以是数组或对象

```jsx
computed: {
  ...mapState('some/nested/module', {
    a: state => state.a,
    b: state => state.b
  }),
  ...mapGetters('some/nested/module', [
    'someGetter', // -> this.someGetter
    'someOtherGetter', // -> this.someOtherGetter
  ])
},
methods: {
  ...mapActions('some/nested/module', [
    'foo', // -> this.foo()
    'bar' // -> this.bar()
  ])
}

```
### CheatSheet

|  | 不使用命名空间 | 使用命名空间 |
| --- | --- | --- |
| state | $store.state.module1.stateName | $store.state.module1.stateName |
| getter | $store.state.getters.getterName | $store.state.getters[’module1/getterName’] |
| mutation | $store.commit(’mutaionName’, payload) | $store.commit(’module1/mutaionName’, payload) |
| action  | $store.dispatch(’actionName’, payload) | $store.dispatch(’module1/actionName’, payload) |

### 参考

[Modules | Vuex (vuejs.org)](https://vuex.vuejs.org/guide/modules.html)