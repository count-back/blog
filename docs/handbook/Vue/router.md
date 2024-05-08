---
title: Vue——router实现原理
date: 2020-04-10
author: 倒数
tags:
  - router
categories:
  - Vue
isShowComments: true
subSidebar: auto
---

# 前言

在我们了解 router 的实现原理之前，我们先来聊聊 vue 的路由跳转到底是怎样的吧！

首先，我们知道 vue 创建的项目其实是一个单页应用。在我们对 vue 项目进行打包以后，我们所有的代码都会被成一份 HTML 和一份 JS，那我们创建的那么多.vue 文件是如何实现通信的呢？

这时我们今天的主角就登场啦，vue-router 就是 vue 官方定义的路由插件，其作用有如下两点：

- 路由用户设定访问路径的，将路径和组件映射起来。
- 在 vue-router 的单页面应用中，页面的路径的改变就是组件的切换

# 实现一个 router

## vue 官方版 router

首先，我们先来看看官方版 router，官方版 router 的实现步骤有五步。我们先来分析分析五步都在干什么。

![2FHSKT%ANM{2P~AGW$C.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/911177446e004cddbec29233e85fa500~tplv-k3u1fbpfcp-watermark.image?)

1：第一步我们在我们的 router 文件夹下的 index.js 文件中引入了‘vue-router’插件，将其引入到我们的项目中。

![X5O.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d80ada1ba8354457bfe5a61dc2400974~tplv-k3u1fbpfcp-watermark.image?)
2：第二步我们就要在我们的 Vue 实例对象中，use 掉我们引入的插件，使我们可以在这个实例对象中可以使用 VueRouter。

![RU.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed957c1ec4a64a36a71956b315a5dce9~tplv-k3u1fbpfcp-watermark.image?)

3. 第三步我们在 index.js 定义了 routes 数组给用户来定义其要路由跳转的路径和对路径进行重定向，如上图，第一个对象就是对路由进行重定向，第二个对象就是定义路由跳转的路径。

![UNO{(J{I%4%I}_Y32%CQ%QI.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4b583ab090f474ea680dbfc5cba178f~tplv-k3u1fbpfcp-watermark.image?)

![63XVLG9A_U1E%IKYEU8G(~J.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/afd6b1251bf24f22bd3553de128143d7~tplv-k3u1fbpfcp-watermark.image?)

4.第四步我们就要是 new 出我们的路由实例对象，选择跳转模式和根路径，同时把我们定义的 routes 数组给到路由实例对象，再将其抛出，之后我们就可以在 main.js 中引入引入我们抛出的路由对象了。

![VWBH0N.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7744cabdc9f6499aa2bd079a54e7ddd5~tplv-k3u1fbpfcp-watermark.image?) 5.第五步是最重要的一步，我们要调用 router 对象中内置的 router-link 和 router-view 组件，让其可以在页面中显示出来。这样我们就可以肆无忌惮的跳转页面了。

## 自己实现一个 VueRouter

在我们看完官方版路由后，我们就该疑惑了，在我们引入的这个 vue-router 组件中，他是怎样让我们可以如此轻松的实现路由跳转的呢？

这时我们就要提出疑惑了，VueRouter 这个构造函数中到底创建了什么方法，可以让我们将模式，根路径，跳转路径三个参数传递进去就可以帮我们实现路由跳转的呢？

在官方版 VueRouter 构造函数中，我们可以看出我们需要处理的由三个东西，第一个就是处理路由跳转的模式（mode），第二个就要处理的就是路由跳转的路径（routes），第三个就要处理路由的根路径和跳转组件。

### VueRouter 构造函数的实现

下面我们来自己创建 VueRouter 构造函数，同时我们再来分析分析他的实现原理。

```js
class HistoryRoute {
  // 保存当前路径状态
  constructor() {
    this.current = null; // 用来记录当前路由的路径的变量
  }
}
class VueRouter {
  constructor(options) {
    this.mode = options.mode || "hash"; // 用来记录路由模式的边疆
    this.routes = options.routes; // 跳转路径数组
    this.routesMap = this.createMap(this.routes); // 路径对象
    this.history = new HistoryRoute();
    this.init();
  }
  createMap(routes) {
    // 当前的路由对象
    return routes.reduce((pre, current) => {
      pre[current.path] = current.component; // '/' : Home, '/about' : About
      return pre;
    }, {});
  }
  init() {
    if (this.mode == "hash") {
      // 根据路由跳转的方式，改变路由的实例对象
      location.hash ? "" : (location.hash = "/");
      window.addEventListener("load", () => {
        this.history.current = location.hash.slice(1);
      });
      window.addEventListener("hashchange", () => {
        this.history.current = location.hash.slice(1);
      });
    } else {
      location.pathname ? "" : (location.pathname = "/");
      window.addEventListener("load", () => {
        this.history.current = location.pathname;
      });
      window.addEventListener("popstate", () => {
        this.history.current = location.pathname;
      });
    }
  }
}
```

### 在 Vue 实例对象中混入 Vue-Router

```js
let Vue = null; // vue 的实例对象
VueRouter.install = function (params) {
  // 在VueRouter实例对象的原型上挂上install方法
  Vue = params; // console.log(Vue);
  Vue.mixin({
    //将在其他地方写的api方法混入vue实例对象中
    beforeCreate() {
      if (this.$options && this.$options.router) {
        // 根组件
        this._root = this;
        this._router = this.$options.router;
        Vue.util.defineReactive(this, "xxx", this._router.history);
      } else {
        //子组件
        this._root = this.$parent && this.$parent._root;
      }
      Object.defineProperty(this, "$router", {
        get() {
          return this._root.router;
        },
      });
      Object.defineProperty(this, "$route", {
        get() {
          return this._root.router.history.current;
        },
      });
    },
  });
  Vue.component("router-link", {
    // 自带的组件注册  声明全局组件
    props: {
      to: String,
    },
    render(h) {
      let mode = this._self._root._router.mode;
      let to = mode === "hash" ? "#" + this.to : this.to;
      return h("a", { attrs: { href: to } }, this.$slots.default);
    },
  });
  Vue.component("router-view", {
    render(h) {
      let current = this._self._root._router.history.current; // 当前路由地址 '/about'
      let routeMap = this._self._root._router.routesMap; // 所有路由对象
      return h(routeMap[current]);
    },
  });
};
export default VueRouter;
```

这样我们直接调用我们自己的 MyRouter 函数，实现路由跳转，下面我们来看看效果：

![QQ录屏20221010223312_1.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26b73dd0ecd44183bf538580dca2347d~tplv-k3u1fbpfcp-watermark.image?)
