---
title: vue-hackernews-2.0 项目学习笔记
categories: Vue 
---

项目地址  [https://github.com/vuejs/vue-hackernews-2.0](https://github.com/vuejs/vue-hackernews-2.0)

在线 demo  [https://vue-hn.herokuapp.com/top](https://vue-hn.herokuapp.com/top)

这是尤雨溪在2019年用 vue-2 写的一个 demo app，目的是为了展示 vue的一些功能

项目使用了 vue + vue-router + vuex ，并将不同模块代码进行分离。

麻雀虽小，五脏俱全。虽然有些早了，但有不少值得学习的地方

## 项目运行：使用 nvm 切换 node 版本

遇到的第一个问题： node 版本过高，不兼容

[(24条消息) nvm介绍、nvm下载安装及使用_前端菜鸡小卒的博客-CSDN博客_nvm](https://blog.csdn.net/qq_30376375/article/details/115877446?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165529798716782390523354%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165529798716782390523354&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-115877446-null-null.142^v16^rank_v33,157^v14^new_3&utm_term=nvm&spm=1018.2226.3001.4187)

使用 nvm 对不同node版本进行管理

nvm就是为解决这个问题而产生的，他可以方便的在同一台设备上进行多个node版本之间切换

踩坑：nvm use 命令需要以**管理员**身份运行，否则报错

```jsx
exit status 1: ��û���㹻��Ȩ��ִ�д˲�����
```

## 路由懒加载

项目的所有路由定义，都是用的懒加载。

涉及知识点包括 **模块的动态导入** 和 **vue-router 中路由懒加载的使用方法**

### 模块的动态导入

ES6 模块的导入方式有两种，第一种的静态导入，即在开头使用 import 关键字导入。语法简单且严格，路径只能是字符串。

第二种是动态导入，即使用 `import()` 表达式。（注意：`import()` 表达式并不是一个函数）

动态导入的两个要点 

- import 表达式返回一个 Promise ，resolve 值为一个模块对象，包含了所有导出
- 当只有一个默认导出时，使用 .default 属性访问

例子1 

```jsx
import(modulePath)
  .then(obj => <module object>)
  .catch(err => <loading error, e.g. if no such module>)
```

例子2 

```jsx
// module.js 
export default {
  name: "zzy",
  sayHi() {
    console.log("hello");
  },
};

//main.js 
import("./module.js").then((m) => {
  console.log(m.default.name);
});
```

例子3 

```jsx
//module.js 
export function sayBye() {
  console.log("bye");
}
export const money = "ssssss";

export default {
  name: "zzy",
  sayHi() {
    console.log("hello");
  },
};

//main.js 
import("./module.js").then((m) => {
  console.log(m);
});

//输出 
// [Module: null prototype] {
//     default: { name: 'zzy', sayHi: [Function: sayHi] },
//     money: 'ssssss',
//     sayBye: [Function: sayBye]
//   }
```

### Vue router 使用懒加载

`component` (和 `components`) 配置接收一个返回 Promise 组件的函数，并且 Vue Router **只会在第一次进入页面时才会获取这个函数**，然后使用缓存数据。

所以使用方法如下 

```jsx
// 将
// import UserDetails from './views/UserDetails'
// 替换成
const UserDetails = () => import('./views/UserDetails')

const router = createRouter({
  // ...
  routes: [{ path: '/users/:id', component: UserDetails }],
})
```

## 使用工厂函数创建路由所需的页面组件

在hacker-news 项目中，由于Top， Ask，Job 等页面的逻辑和页面样式基本相同，只是要展示的数据类型不同。所以只需要写一个页面，并使用 props 确定要加载的数据类型即可。

但是在 vue-router 路由定义时，又不能向组件传递 prop 参数来指定页面的数据类型，所以定义一个工厂函数，结合渲染函数，实现根据数据类型创建不同的 ItemListPage 页面。

### 渲染函数 （render function）基础

创建组件时，有两种方式告诉Vue要生成什么样的HTML

- template 模板  （大部分时候建议）
- render 函数 （当创建模板需要使用 JS 进行处理时）

例子：以下两种方式效果是一样的

```jsx
// render-example.vue 
<script>
import { h } from "@vue/runtime-core";
export default {
  render() {
    return h("div", {}, "hello word");
  },
};
</script>

//template-example.vue 
<template>
  <div>hello word</div>
</template>

<script>
export default {};
</script>

```

render 函数生成并返回 VNode 对象， 借助全局方法 `h()` 实现

`h()` 方法接收三个参数：

- type  标签名字符串 |  一个组件
- props
- children   字符串 | 子 VNode 数组 | 有插槽的对象（比如 `this.$slot.default()`）

在hacker-news 项目中，由于Top， Ask，Job 等页面的逻辑和页面样式基本相同，只是要展示的数据类型不同。所以只需要写一个页面，并使用 props 确定要加载的数据类型即可。

但是在 vue-router 路由定义时，又不能向组件传递 prop 参数来指定页面的数据类型，所以使用一个工厂函数，来根据数据类型创建不同的 ItemListPage 页面。

方法如下：

### 实现步骤

**定义工厂函数**

根据 type 返回一个组件对象。相当于对 ItemList.vue 包装了一层

使用 render 函数代替模板进行创建，这样就能传入 props ：type （注意：vue3 传 props 语法略有不同）

```jsx
//定义工厂函数 createListView.js
import ItemList from './ItemList.vue'

export default function createListView (type) {
  return {
    name: `${type}-stories-view`,
		//...
    render (h) {
      return h(ItemList, { props: { type }})   // 核心：render 函数
    }
  }
}

```

**在路由中加载组件**

定义`createListView` 函数，返回一个函数，满足 component 属性的值要求

即：`component` 配置接收一个返回 Promise 组件的函数，实现路由的懒加载

当然也可以将  `createListView` 工厂函数导入，然后挨个创建需要的页面作为component 的属性值

但这样就不是路由懒加载了

```jsx
const createListView = (id) => () =>
  import("../views/CreateListView").then((m) => m.default(id));

export function createRouter() {
  return new Router({
    mode: "history",
    routes: [
      { path: "/top/:page(\\d+)?", component: createListView("top") }, 
      { path: "/new/:page(\\d+)?", component: createListView("new") },
      { path: "/show/:page(\\d+)?", component: createListView("show") },
      { path: "/ask/:page(\\d+)?", component: createListView("ask") },
      { path: "/job/:page(\\d+)?", component: createListView("job") },
      { path: "/item/:id(\\d+)", component: ItemView },
      { path: "/user/:id", component: UserView },
      { path: "/", redirect: "/top" },
    ],
  });
}
```

静态加载

```jsx
import createListPage from '../views/CreateListView' 

//..
	routes:[
		{ path: "/top/:page(\\d+)?", component: createListPage("top") }
	]
```

## 数据角度 —— Vuex 基础

[Vuex 学习 0.1 ](https://zzy2005137.github.io/blog/2022/06/27/vuex%E5%9F%BA%E7%A1%80/)

[Vuex 学习0.2 模块 ](https://zzy2005137.github.io/blog/2022/06/27/vuex%E6%A8%A1%E5%9D%97/)

## 其他

### 动态路由参数设置为可选

问题： 带参数的动态路由，在没有参数时如何正常显示 ？ 

路由参数后面加 `?` 代表可选参数

[路由的匹配语法 | Vue Router (vuejs.org)](https://router.vuejs.org/zh/guide/essentials/route-matching-syntax.html#%E5%8F%AF%E9%80%89%E5%8F%82%E6%95%B0)

```jsx
const routes = [
  // 匹配 /users 和 /users/posva
  { path: '/users/:userId?' },
  // 匹配 /users 和 /users/42
  { path: '/users/:userId(\\d+)?' },
]
```

### ItemList页面逻辑

beforeMount ：调用vuex action，发送网络请求，获取items数据。根据 page 参数设置当前展示页和当前展示 items

监听 page 变化，重新加载items数据

### vuex-router-sync

在 vuex 的getter 里发现一个疑惑的点

在设置当前要展示的items列表时，用到了当前路由的 page 参数， 访问方法如下

```jsx
//store/index.js 
const page = Number(state.route.params.page) || 1;
```

问题：为什么state 能够访问到 route ？ 

原因：在 app.js 中，使用了 `vuex-router-sync` 第三方包， 用于把 Vue Router 当前的 `$route`
同步为 Vuex 状态的一部分。

[https://github.com/vuejs/vuex-router-sync/blob/master/README.zh-cn.md](https://github.com/vuejs/vuex-router-sync/blob/master/README.zh-cn.md)

```jsx
import { sync } from 'vuex-router-sync'

// sync the router with the vuex store.
// this registers `store.state.route`
sync(store, router)
```

通过查询资料，要在 vuex 中访问到 vue router 可以将 router 对象导入 vuex 

访问当前路由用法如下

```jsx
import router from "@/router";
return router.currentRoute.value.params;
```