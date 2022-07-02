---
title: Vuex 学习 0.1
categories: Vue 
---


[Vuex 4 Crash Course - 2021](https://www.youtube.com/watch?v=y7DQhNs9Azw)

https://vuex.vuejs.org/zh/guide/

## vuex 是什么

> Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式 + 库**。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

什么是状态（**state**）？ 即 Vue 应用中的数据

Vuex 将组件的共享状态抽取出来，以一个全局的单例模式进行管理

Vuex 的两个要点：

- 读取状态
- 修改状态

任意组件都能够读取到 Vuex 存储的状态

修改状态的唯一方式是通过提交事先定义好的 mutation（行为），这就对状态的改变方式进行了限制，同时也变得可预测

为什么使用vuex ： 当多个视图需要用到同一份数据的时候，能够避免复杂传参

## 创建 store

使用 `createStore` 方法创建一个全局的 `Store`（”仓库“） 对象。

```jsx
import { createStore } from "vuex";

export default createStore({
  state() {
		return {}
	},
  getters: {},
  mutations: {},
  actions: {},
  modules: {},
});
```

这个仓库有以下特点：

- 任意组件都可以访问到 store 中的状态
- 状态存储是响应式的
- 改变 store 中状态的唯一途径是显示的提交（commit） mutation

## 在组件中读取 state

方法一 ：访问 `$store` 实例   `{{$state.store.statNname}}`

方法二（更简便） ：使用 `mapState` 辅助函数将 `state` 映射到计算属性中

使用方法

mapState接收一个对象或者数组，返回一个对象，对象中包含多个计算属性值

使用对象作为参数能够以别名的方式访问 state

当映射的计算属性的名称与 state 的子节点名称相同时，直接传字符串数组

结合展开语法使用

```jsx
computed: {
  localComputed () { /* ... */ },
  // 使用对象展开运算符将此对象混入到外部对象中
  ...mapState([
	  // 映射 this.count 为 store.state.count
	  'count'
])
}
// 在单独构建的版本中辅助函数为 Vuex.mapState
import { mapState } from 'vuex'

export default {
  // ...
  computed: mapState({
    // 箭头函数可使代码更简练
    count: state => state.count,

    // 传字符串参数 'count' 等同于 `state => state.count`
    countAlias: 'count',

    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}
```

## 修改store中的状态

更改 Vuex 的 store 中的状态的唯一方法是提交 mutation

1. 注册Mutation
2. 调用 Mutation

> 注意：mutation 必须是同步函数

### mutation 注册

```jsx
mutations: {
  increment (state, payload) { // state 自动传入， payload 一般是一个对象
    state.count += payload.amount   
  }
}
```

### mutation 调用 （commit）

方法一： `this.$store.commit('xxx'，payload)`

方法二 （推荐）： 使用 `mapMutations` 辅助函数将 mutation 映射到组件的 methods 中

```jsx
methods: {
    ...mapMutations(["addToCounter", "subtractFromCounter"]),
  },
```

## Action 实现异步请求

action 最重要的是能执行异步操作

mutation 中只能包含同步操作，**异步操作**由action来完成

1. action 注册
2. action 调用（dispatch）

### Action 注册

Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象

通过contex 来读取state 或者提交mutation修改state

```jsx
const store = createStore({
  //....
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})
```

### Action 调用（dispatch）

方法一 ：`$store.dispatch('xxx')`

方法二：使用 `mapActions`辅助函数将 action 映射到 组件的 methods 中

```jsx
export default {
  // ...
  methods: {
    ...mapActions([
      'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`

      // `mapActions` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
    ]),
    ...mapActions({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
    })
  }
}
```

## Getter —— store 中的computed 方法

Store 中的 Getter 类似于组件中的计算属性，是为了获取原始数据经过筛选处理之后的结果

最常见的case， 需要对列表数据进行过滤筛选；

如果在多个组件中都需要进行同样的筛选，就需要重复写 computed 函数

所以使用 getter， 把 getter 同样映射到计算属性中当成数据来用

1. 注册 getter
2. 访问 getter

### 注册：返回数据或者查询函数

```jsx
getters: {
    doneTodos (state) {
      return state.todos.filter(todo => todo.done)
    }
  }

getters: {
  // ...
  getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  }
}
```

### 访问

方法一 ：`store.getters.`getterName

方法二：使用 `mapGetters` 辅助函数将 store 中的 getter 映射到局部**计算属性**

```jsx
//通过属性访问，当getter返回一个数据时
store.getters.dountTodoCount

//通过方法访问，当getter 返回一个方法时
store.getters.getTodoById(2)
```

## CheatSheet

|          | 直接访问/调用                             | 辅助函数     | 辅助函数映射位置 |
| -------- | ----------------------------------------- | ------------ | ---------------- |
| State    | $store.state.property_name                | mapState     | computed         |
| Mutation | $store.commit(type, payload)              | mapMutations | methods          |
| Action   | $store.dispatch(type, payload) / 或 (obj) | mapActions   | methods          |
| Getter   | $store.getters.getterName                 | mapGetters   | computed         |
