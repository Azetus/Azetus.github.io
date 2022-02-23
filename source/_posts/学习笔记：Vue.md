---
title: 学习笔记：Vue 基础
date: 2022-02-23 10:16:17
tags:
  - Vue
categories:
  - Vue
---

# Vue 基本使用

- 基本使用，组件使用 —— 常用，必须会
- 高级特性 —— 不常用，但体现深度
- Vuex 和 Vue-router 使用

## 指令、插值

- 插值、表达式
- 指令、动态属性
- v-html：会有 xss 风险，会覆盖子组件

## computed 和 watch

- computed 有缓存，data 不变则不会重新计算
- watch 如何深度监听
- watch 监听引用类型，拿不到 oldVal

## class 和 style

- 使用动态属性
- 使用驼峰式写法

## 条件渲染

- v-if、v-else 的用法，可使用变量，也可以使用 === 表达式
- v-if 和 v-show 的区别？
  > v-if 判断不显示的内容不会出现在 DOM 结构  
  > v-show 使用`display: none;`隐藏不显示内容
- v-if 和 v-show 的使用场景？
  > v-if 适用于更新不频繁的内容（一次性）
  > v-show 适用于频繁切换的场景

## 循环（列表）渲染

- 遍历对象也可以用 v-for
- key 的重要性
  > 数据结构中最好有 id
- **v-for 和 v-if 不能一起使用**
  > 先 v-for 循环，所有子对象循环出的元素都会有 v-if 判断，会严重影响性能

## 事件

```html
<button @click="increment1">+1</button>
<button @click="increment2(2, $event)">+2</button>
```

```javascript
increment1(event){ ... }
increment2(val,event){ ... }
```

- event 参数，自定义参数
  > 携带自定义参数时，若需要 event 对象则需要手动传递参数
- 事件修饰符，按键修饰符
- **观察**事件被绑定的位置
  > 是原生的 event 对象  
  > 事件被挂载到当前元素

## 事件修饰符

```html
<!-- 阻止单击事件继续传播 -->
<a @click.stop="doThis"></a>

<!-- 提交事件不再重载页面 -->
<form @submit.prevent="onSubmit"></form>

<!-- 修饰符可以串联 -->
<a @click.stop.prevent="doThat"></a>

<!-- 只有修饰符 -->
<form @submit.prevent></form>

<!-- 添加事件监听器时使用事件捕获模式 -->
<!-- 即内部元素触发的事件先在此处理，然后才交由内部元素进行处理 -->
<div @click.capture="doThis">...</div>

<!-- 只当在 event.target 是当前元素自身时触发处理函数 -->
<!-- 即事件不是从内部元素触发的 -->
<div @click.self="doThat">...</div>
```

> 摘录自 Vue3 中文文档

## 按键修饰符

```html
<!-- Alt + Enter -->
<input @keyup.alt.enter="clear" />

<!-- Ctrl + Click -->
<div @click.ctrl="doSomething">Do something</div>
<!-- 即使 Alt 或 Shift 被一同按下时也会触发 -->
<button @click.ctrl="onClick">A</button>

<!-- 有且只有 Ctrl 被按下的时候才触发 -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- 没有任何系统修饰符被按下的时候才触发 -->
<button @click.exact="onClick">A</button>
```

> 摘录自 Vue3 中文文档

## 表单

- v-model
  > 双向数据绑定
- 常见表单项 textarea checkbox radio select
  ```html
  <textarea v-model="desc"></textarea>
  <!-- 注意，<textarea>{{desc}}</textarea> 是不允许的！！！ -->
  ```
- 修饰符 lazy number trim
  > trim 截取前后空格  
  > lazy 防抖
  > number 转换为数字

# Vuex 基本概念

- state
- getters
- action (异步操作)
- mutation

## 用于 Vue 组件

- dispatch
- commit
- mapState
- mapGetters
- mapActions
- mapMutations

![vuex_imgs](https://next.vuex.vuejs.org/vuex.png)

> 图片引用自 vuex 文档

# Vue-router 使用

- 面试考点并不多（前提是熟悉 Vue）
- 路由模式（hash、H5 history）
- 路由配置（动态路由、懒加载）

## Vue-router 路由模式

- hash 模式（默认），如`http://abc.com/#/user/20`
- H5 history 模式，如`http://abc.com/user/20`
- 后者需要 server 端支持，因此无特殊需求可选择前者

## 动态路由配置

```javascript
const user = {
  template: '<div>User {{$route.params.id}}</div>'
};

const router = new VueRouter({
  routers: [
    // 动态路径参数 以冒号开头。能命中'/user/10' '/user/20'等格式的路由
    { path: '/user/:id', component: User }
  ]
});
```

## 懒加载

```javascript
const UserDetails = () => import('./views/UserDetails');

const router = createRouter({
  // ...
  routes: [{ path: '/users/:id', component: UserDetails }]
});
```

# 前端路由原理

- 稍微复杂一点的 SPA，都需要路由
- vue-router 是 vue 全家桶的标配之一

## 回顾

- 回顾 vue-router 的路由模式
- hash
- H5 history （需要后端支持）

---

## hash

### 网页 url 组成部分

```javascript
// http://127.0.0.1:8881/01-hash.html?a=100&b=20#/aaa/bbb
location.protocol; // 'http:'
location.hostname; // '127.0.0.1'
location.host; // '127.0.0.1:8881'
location.port; // '8881'
location.pathname; // '/01-hash.html'
location.search; // '?a=100&b=20'
location.hash; // '#/aaa/bbb'
```

### hash 的特点

- hash 变化会触发网页跳转，即浏览器的前进、后退
- hash 变化不会刷新页面，SPA 必需的特点
- hash 永远不会提交到 server 端（前端自生自灭）

## H5 history

- 用 url 规范的路由，但转跳时不刷新页面
- history.pushState
- window.onpopstate

### H5 history 模式

- `https://mySite.com/xxx` 刷新页面
- `https://mySite.com/xxx/yyy` 前端跳转，不刷新页面
- `https://mySite.com/xxx/yyy/zzz` 前端跳转，不刷新页面
- 需要后端配合（无论访问什么路由，后端都返回 index.html）

### 小结

- hash - `window.onhashchange`
- H5 history - `history.pushState` 和 `window.onpopstate`
- H5 history 需要后端支持

## 两者选择

- to B 的系统推荐用 hash，简单易用，对 url 规范不敏感
- to C 的系统，可以考虑选择 H5 history，但需要服务端支持
  > 例如需要考虑搜索引擎优化时
- 能选简单的，就别用复杂的，要考虑成本和收益

# 组件 渲染/更新 过程

- 一个组件渲染到页面，修改 data 触发更新（数据驱动视图）
- 其背后的原理与要点

## 回顾

- 响应式：监听 data 属性的 getter/setter
- 模板编译：模板到 render 函数，再到 vnode
- vdom：patch(elem, vnode)和 patch(vnode, newVnode)

## 知识点

- 初次渲染过程
- 更新过程
- 异步渲染

---

## 初次渲染过程

1. 解析模板为 render 函数（或在开发环境已完成，vue-loader）
2. 触发响应式，监听 data 属性 getter setter
   > 执行 render 函数会触发(touch) getter (如果模板用到的话)  
   > 并生成、收集依赖，并观察起来(watcher)
3. 执行 render 函数，生成 vnode，patch(elem,vnode)

## 更新过程

1. 修改 data ，触发 setter （此前在 getter 中已被监听）
   > Notify Watcher
2. 重新执行 render 函数，生成 newVnode
3. patch(vnode, newVnode)
   > diff 算法计算出 vnode,newVnode 的最小差异并之后更新在 DOM 上

![vue_process](vue_process.png)

> 图片来自 vue 官方文档

## 异步渲染

- 回顾 `$nextTick()`
- 汇总 data 的修改，一次性更新视图
- 减少 DOM 操作次数，提高性能

## 小结

- 渲染和响应式的关系
- 渲染和模板编译的关系
- 渲染和 vdom 的关系
- 初次渲染过程
- 更新过程
- 异步渲染

# Vue 原理 (1)

- 组件化
- 响应式
- vdom 和 diff
- 模板编译
- 渲染过程
- 前端路由

## 组件化基础

- 数据驱动视图（MVVM，setState）
- asp jsp php 已经有组件化
- nodejs 中也有类似的组件化
- 传统组件，只是静态渲染，更新还要依赖于操作 DOM
- 数据驱动视图 - Vue MVVM
- 数据驱动视图 - React setState

## Vue MVVM

> Model View ViewModel

![vue_mvvm](https://012.vuejs.org/images/mvvm.png)

> 图片引用自 vue 官方文档

通过 VM 将视图(view)和数据(model)连接起来，通过修改 Model 中的数据可以直接修改视图。

> 视图 - template  
> 数据 - data()
> VM - 连接层

## 小结

- 组件化
- 数据驱动视图
- MVVM

---

# Vue 响应式

> - 组件 data 的数据一旦发生变化，立刻触发视图的更新
> - 实现数据驱动视图的第一步
> - 考察 Vue 原理的第一道题

- 核心 API - `Object.defineProperty`
- 如何实现响应式
- `Object.defineProperty` 的一些缺点（Vue3.0 启用 Proxy）

## Proxy 有兼容性问题

- Proxy 兼容性不好，且无法 polyfill
- Vue2.x 还会存在一段时间，所以都得学

## Object.defineProperty 基本用法

```javascript
const data = {};
let name = 'Aliza';
Object.defineProperty(data, 'name', {
  get: function () {
    console.log('get');
    return name;
  },
  set: function (newVal) {
    console.log('set');
    name = newVal;
  }
});
```

```javascript
console.log(data.name);
data.name = 'Monika';
```

> get:属性的 getter 函数，当访问该属性时，会调用此函数。  
> set:属性的 setter 函数，当属性值被修改时，会调用此函数。

- 监听对象，如何监听数组？
- 复杂对象，深度监听
- 几个缺点

## 如何深度监听 data 变化

实现深度监听

```javascript
// 触发更新视图
function updateView() {
  console.log('视图更新');
}

// 准备数据
const data = {
  name: 'zhangsan',
  age: 20,
  info: {
    address: '北京' // 需要深度监听
  },
  nums: [10, 20, 30]
};
```

```javascript
// 监听数据
observer(data);

// 监听对象属性
function observer(target) {
  if (typeof target !== 'object' || target === null) {
    // 不是对象或数组
    return target;
  }

  // 重新定义各个属性（for in 也可以遍历数组）
  for (let key in target) {
    defineReactive(target, key, target[key]);
  }
}

// 重新定义属性，监听起来
function defineReactive(target, key, value) {
  // 深度监听
  observer(value);

  // 核心 API
  Object.defineProperty(target, key, {
    get() {
      return value;
    },
    set(newValue) {
      if (newValue !== value) {
        // 深度监听
        observer(newValue);

        // 设置新值
        // 注意，value 一直在闭包中，此处设置完之后，再 get 时也是会获取最新的值
        value = newValue;

        // 触发更新视图
        updateView();
      }
    }
  });
}
```

`Object.defineProperty`的缺点

- 深度监听，需要递归到底，一次性计算量大
- `Object.defineProperty`API 无法监听新增属性/删除属性，例如`data.x=100`（所以 Vue2.0 需要 Vue.set Vue.delete）

## 如何监听数组变化

```javascript
// 重新定义数组原型
const oldArrayProperty = Array.prototype;
// 创建新对象，原型指向 oldArrayProperty ，再扩展新的方法不会影响原型
const arrProto = Object.create(oldArrayProperty);
// 对新数组扩展方法
['push', 'pop', 'shift', 'unshift', 'splice'].forEach((methodName) => {
  arrProto[methodName] = function () {
    updateView(); // 触发视图更新

    oldArrayProperty[methodName].call(this, ...arguments);
    // 执行push()时就类似于Array.prototype.push.call(this, ...arguments)
  };
});
```

```javascript
// 监听对象属性
function observer(target) {
  if (typeof target !== 'object' || target === null) {
    // 不是对象或数组
    return target;
  }

  // 直接修改 Array.prototype，会污染全局的 Array 原型
  // 所以需要利用 arrProto 来扩展方法
  // Array.prototype.push = function () {
  //     updateView()
  //     ...
  // }

  if (Array.isArray(target)) {
    // 改变数组的原型指向
    // 次数组再次调用数组方法时会先去arrProto中扩展的方法里寻找
    target.__proto__ = arrProto;
  }

  // 重新定义各个属性（for in 也可以遍历数组）
  for (let key in target) {
    defineReactive(target, key, target[key]);
  }
}
```

`Object.defineProperty`的缺点

- 无法原生监听数组，需要特殊处理

## 小结

- 基础 API - `Object.defineProperty`
- 如何监听对象（深度监听），监听数组
- `Object.defineProperty`的缺点:
  - 深度监听，需要递归到底，一次性计算量大
  - API 无法监听新增属性/删除属性，例如`data.x=100`（所以 Vue2.0 需要 Vue.set Vue.delete）
  - 无法原生监听数组，需要特殊处理

---

# 虚拟 DOM( Virtual DOM ) 和 diff

- vdom 是实现 vue 和 React 的重要基石
- diff 算法是 vdom 最核心、最关键的部分
- vdom 是一个热门话题

**技术背景**

- DOM 操作**非常**耗费性能
- 以前使用 jQuery，可以自行控制 DOM 操作的时机，手动调整
- Vue 和 React 都是数据驱动视图，如何有效控制 DOM 操作？

## 解决方案 - vdom

- 有了一定复杂度，想减少计算次数比较难
- 将计算，更多的转移为 JS 计算，因为 JS 执行速度很快
- vdom - 用 JS 模拟 DOM 结构，计算出最小的变更，操作 DOM

> DOM 操作依旧是必不可少的，但 vdom 可以尽可能减少无用的 DOM 操作

## 用 JS 模拟 DOM 结构

```html
<div id="div1" class="container">
  <p>vdom</p>
  <ul style="font-size: 20px">
    <li>a</li>
  </ul>
</div>
```

```javascript
{
  tag: 'div',
  props:{
    className: 'container',
    id: 'div1'
  },
  children:[
    {
      tag: 'p',
      children: 'vdom'
    },
    {
      tag: 'ul',
      props: {style: 'font-size: 20px'},
      children: [
        {
          tag: 'li',
          children: 'a'
        }
      ]
    }
  ]
}
```

> 所有 XML 语言都可以用 JSON 或是 JS 对象的形式表示

## 通过 snabbdom 学习 vdom

- 简洁强大的 vdom 库，易学易用
- Vue 参考它实现的 vdom 和 diff
- https://github.com/snabbdom/snabbdom
- Vue3.0 重写了 vdom 的代码，优化了性能
- 但 vdom 的基本理念不变，面试考点也不变
- React vdom 具体实现和 Vue 也不同，但不妨碍统一学习

---

# diff 算法

- diff 算法是 vdom 中最核心、最关键的部分
- diff 算法能在日常使用 vue React 中体现出来（如 key）
- diff 算法是前端热门话题

## diff 算法概述

- diff 即对比，是一个广泛的概念，如 linux diff 命令、git diff 等
- 两个 js 对象也可以做 diff，如https://github.com/cujojs/jiff
- 两棵树做 diff，如这里的 vdom diff

![diff_tree](diff_tree.png)

## 树 diff 的时间复杂度 O(n^3)

- 第一，遍历 tree1；第二，遍历 tree2
- 第三，排序
- 1000 个节点，要计算 1 亿次，算法是不可用的

## 优化时间复杂度到 O(n)

- 只比较同一层级，不跨级比较
- tag 不相同，则直接删掉重建，不再深度比较
- tag 和 key，两者都相同，则认为是相同节点，不再深度比较

**1. 只比较同一层级**

![diff_tree_01](diff_tree_01.png)

**2. tag 不相同，则直接删掉重建，不再深度比较**

![diff_tree_02](diff_tree_02.png)

## 不适用 key 和使用 key

![diff_key](diff_keys.png)

如果不使用 key 或使用随机数，在更新 vdom 时，不变的节点只能全部删掉重建。key 使用 index 时，若新的 vdom 中节点顺序发生了改变，也只能将节点删掉重建（index 仅仅按顺序赋值 0 1 2 3 4）。

> **认为是相同节点的条件：tag 和 key 两者都相同**

如果使用了 key，在对比 children 时，可以根据 tag 与 key 快速得出哪些新旧节点是相同的，直接移动即可。

## 小结

- patchVnode
- addVnodes removeVnodes
- updateChildren（key 的重要性）

---

# vdom 和 diff - 总结

- 关注基本流程和原理
- vdom 核心概念很重要： h、vnode、patch、diff、key 等
- vdom 存在的价值更加重要：数据驱动视图，控制 DOM 操作

---

# Vue 原理 (2)

# 模板编译

- 模板是 Vue 开发中最常用的部分，即与使用相关联的原理
- 它不是 html，有指令、插值、JS 表达式
- 面试不会直接问，但会通过“组件渲染和更新过程”考察

> template -> vnode

- 前置知识： JS 的 with 语法
- vue template complier 将模板便以为 render 函数
- 执行 render 函数生成 vnode

## with 语法

```javascript
const obj = { a: 100, b: 200 };

console.log(obj.a); // 100
console.log(obj.b); // 200
console.log(obj.c); // undefined

// 使用 with，能改变 {} 内自由变量的查找方式
// 将 {} 内的自由变量，当做 obj 的属性来查找
with (obj) {
  console.log(a); // 100
  console.log(b); // 200
  console.log(c); // 会报错！
}
```

- 改变 {} 内自由变量的查找方式，当做 obj 的属性来查找
- 如果找不到匹配的 obj 属性，就会报错
- with 要慎用，它打破了作用域规则，易读性变差

## 编译模板

- 模板不是 html，有指令、插值、JS 表达式，能实现判断、循环
- html 是标签语言，只有 JS 才能实现判断、循环（图灵完备的）
- 因此，模板一定是转换为了某种 JS 代码，即编译模板

vue-template-compiler 示例

```javascript
const compiler = require('vue-template-compiler');
// 插值
const template = `<p>{{message}}</p>`;
// with(this){return createElement('p',[createTextVNode(toString(message))])}
// 编译
const res = compiler.compile(template);
console.log(res.render);
// with(this){return _c('p',[_v(_s(message))])}
```

> `this` -> `new Vue({...})`

```javascript
const template = `
    <div id="div1" class="container">
        <img :src="imgUrl"/>
    </div>
`;
// with (this) {
//   return _c('div', { staticClass: 'container', attrs: { id: 'div1' } }, [
//     _c('img', { attrs: { src: imgUrl } })
//   ]);
// }
```

```javascript
with (this) {
  return _c(
    'tag-name',
    {
      // class-name, attrs ....
    },
    [
      // childrenNodes1, childrenNodes2
    ]
  );
}
```

- 模板编译为 render 函数，执行 render 函数返回 vnode
- 基于 vnode 再执行 patch 和 diff
- 使用 webpack vue-loader 会在开发环境下编译模板（重要）

# Vue 组件中使用 render 代替 template

```javascript
Vue.component('heading', {
  // template:`xxxx`,
  render: function (createElement) {
    return createElement('h' + this.level, [
      createElement(
        'a',
        {
          attrs: {
            name: 'headerId',
            href: '#' + 'headerId'
          }
        },
        'this is a tag'
      )
    ]);
  }
});
```

- 有些复杂情况中，不能用 template，可以考虑用 render
- React 一直都用 render（没有模板）

# 小结

- with 语法
- 模板到 render 函数，再到 vnode，再到渲染和更新
- Vue 组件可以用 render 代替 template

---

# Vue 原理 - 总结

**知识点：**

- 组件化
- **响应式**
- **vdom 和 diff**
- **模板编译**
- 渲染过程
- 前端路由

## 组件化

- 组件化的历史
- 数据驱动视图
- MVVM

## 响应式

- `Object.defineProperty`
- 监听对象（深度），监听数组
- `Object.defineProperty`的缺点（Vue3 使用 Proxy）

## vdom 和 diff

- 应用背景
- vnode 结构
- snabbdom 使用： vnode h patch

## 模板编译

- with 语法
- 模板编译为 render 函数
- 执行 render 函数生成 vnode

## 组件渲染/更新过程

- 初次渲染过程
- 更新过程
- **异步渲染**

## 前端路由原理

- hash
- H5 history
- 两者对比
