---
title: 学习笔记：Vue 知识点
date: 2022-02-23 10:31:08
tags:
  - Vue
categories:
  - Vue
---

# Vue 知识点 (1)

## 1. v-show 和 v-if 的区别

- v-show 通过 CSS display 控制显示和隐藏
- v-if 组件真正的渲染和销毁，而不是显示和隐藏
- 频繁切换显示用 v-show，否则用 v-if
  > 如果涉及到权限(例如 vip 权限)、安全，页面展示的情况下用 v-if

v-if 有更高的切换消耗 （切换时需要修改 DOM 结构）
v-show 有更高的初始渲染消耗 （无论初始为 true 还是 false 都会进行渲染，之后通过 css 控制）

## 2. 为何在 v-for 中用 key

- 必须用 key，且不能是 index 和 random
- diff 算法中需要通过 tag 和 key 来判断，是否是 sameNode
- 减少渲染次数，提升渲染性能

## 3. 描述 Vue 组件生命周期（父子组件）

### 单组件生命周期图

![life_cycle](https://vue3js.cn/docs/zh/images/lifecycle.png)

- 挂载阶段
  > beforeCreate created 实例初始化，只是存在内存中的 js 变量  
  > beforeMount mounted 在网页上绘制完成，渲染完成
- 更新阶段
  > beforeUpdate updated
- 销毁阶段
  > beforeUnmount unmounted

### 父子组件生命周期关系

- 挂载阶段

  - 创建及初始化阶段由外到内（先父组件后子组件）
  - 渲染阶段由内到外（先子组件后父组件）

  > 先创建及初始化父组件  
  > 子组件渲染完毕后，父组件才算完成渲染

![父子组件生命周期01](vue_lifecycle_01.png)

- 更新阶段

  - before update 由外到内
  - updated 由内到外

> 父组件的 data 先被修改，并更新子组件  
> 子组件更新渲染完毕后，父组件才算完成更新

![父子组件生命周期02](vue_lifecycle_02.png)

- 销毁阶段

  - 子组件销毁后，父组件才能销毁

![父子组件生命周期03](vue_lifecycle_03.png)

## 4. Vue 组件如何通讯（常见情况）

- 父子组件 props 和 `this.$emit`
- 自定义事件 `event.$no` `event.$off` `event.$emit`
- Vuex

## 5. 描述组件渲染和更新的过程

![vue_process](vue_process.png)

### 初次渲染过程

1. 解析模板为 render 函数（或在开发环境已完成，vue-loader）
2. 触发响应式，监听 data 属性 getter setter
   > 执行 render 函数会触发(touch) getter (如果模板用到的话)  
   > 并生成、收集依赖，并观察起来(watcher)
3. 执行 render 函数，生成 vnode，patch(elem,vnode)

### 更新过程

1. 修改 data ，触发 setter （此前在 getter 中已被监听）
   > Notify Watcher
2. 重新执行 render 函数，生成 newVnode
3. patch(vnode, newVnode)
   > diff 算法计算出 vnode,newVnode 的最小差异并之后更新在 DOM 上

## 6. 双向数据绑定 v-model 的实现原理

- input 元素的 `value = this.name`
- 绑定 input 事件 `this.name = $event.target.value`
- data 更新触发 re-render

## 7. 对 MVVM 的理解

> Model View ViewModel

![vue_mvvm](https://012.vuejs.org/images/mvvm.png)

> 图片引用自 vue 官方文档

通过 VM 将视图(view)和数据(model)连接起来，通过修改 Model 中的数据可以直接修改视图。

> 视图 - template  
> 数据 - data()
> VM - 连接层

## 8. computed 有何特点

- 缓存，data 不变不会重新计算
- 提高性能

## 9. 为何组件 data 必须是一个函数

- 组件本质上是一个 class，使用组件时是对 class 进行实例化
- 如果 data 不是一个函数，组件实例化时所有的数据全会变成一样的
- data 是函数，在组件实例化时都会执行 data()函数，data 就会在闭包之中
- 因此修改 data 时组件间就不会相互影响

## 10. ajax 请求应该放在哪个生命周期

- 放在 mounted 中（整个渲染完成，DOM 也加载完成）
- JS 本质是**单线程**，ajax 是异步获取数据
- 放在 mounted 之前没有用，只会让逻辑更加混乱
  > 没有渲染完，异步数据依旧在 pending 中，没有实际的效果

## 11. 如何将组建所有的 props 传递给子组件？

- `$props`
- `<User v-bind= "$props"/>`

## 12. 如何自己实现 v-model

```html
<template>
  <input
    type="text"
    :value="text"
    @input="$emit('change',$event.target.value)"
  />
  <!-- 
    1. 使用 :value 而不是 v-model
    2. change 和 model.event 对应起来即可，可以自己修改命名
   -->
</template>

<script>
  export default {
    model: {
      prop: 'text', // 对应到 props text
      event: 'change'
    },
    props: {
      text: String,
      default() {
        return '';
      }
    }
  };
</script>
```

## 13. 多个组件有相同的逻辑，如何抽离？

- mixin
- 以及 mixin 的一些缺点
  - 变量来源不明确，不利于阅读
  - 多 minxin 可能会造成命名冲突
  - minxin 和组件可能会出现多对多的关系，复杂度较高

## 14. 何时要使用异步组件？

- 加载大组件
- 路由异步加载

## 15. 何时需要使用 keep-alive？

- 缓存组件，不需要重复渲染
- 如多个静态 tab 页切换
- 优化性能

## 16. 何时需要使用 beforeDestory（Vue3 中为 beforeUnmount）

- 解绑自定义事件 event.$off
- 清除定时器
- 解绑自定义的 DOM 事件，如 window scroll 等

## 17. 什么是作用域插槽

子组件作用域中有一个 data，让父组件也能获取。

子组件

```html
<template>
  <a :herf="url">
    <slot :slotData="website">
      {{website.subTitle}}
      <!-- 默认值显示 subTitle ，即父组件不传内容时 -->
    </slot>
  </a>
</template>
<script>
  export default {
    props: ['url'],
    data() {
      return {
        website: {
          url: 'http://xxxxxx.com',
          title: 'XXX',
          subTitle: 'xxx'
        }
      };
    }
  };
</script>
```

父组件

```html
<ScopedSlotDemo :url="website.url">
  <template v-slot="slotProps">
    <!-- {{/* slotProps名字可以自定义 */}}  -->
    {{ slotProps.slotData.title }}
  </template>
</ScopedSlotDemo>
```

在子组件 slot 定义一个动态属性`:slotData="website"`，并对应到子组件 data。父组件调用子组件时，在子组件标签内添加一个`<template>`，在`<template>`中使用`v-slot`拿到数据。

## 18. Vuex 中 action 和 mutation 有何区别

- **action 中处理异步**， mutation 不可以
- mutation 做原子操作（每次一个操作）
- **aciton 可以整合多个 mutation**

## 19. Vue-router 常用的路由模式

- hash 默认
  - `window.onhashchange`
- H5 history（需要服务端支持，不管什么 url 都返回 index.html）
  - `history.pushState` 和 `window.onpopstate`
- 两者比较
  - to B 的系统推荐用 hash，简单易用，对 url 规范不敏感
  - to C 的系统，可以考虑选择 H5 history，但需要服务端支持
    > 例如需要考虑搜索引擎优化时
  - 能选简单的，就别用复杂的，要考虑成本和收益

## 20. 如何配置 Vue-router 异步加载

使用 import 异步组件的形式

```javascript
const UserDetails = () => import('./views/UserDetails');

const router = createRouter({
  // ...
  routes: [{ path: '/users/:id', component: UserDetails }]
});
```

## 21. 请用 Vnode 描述一个 DOM 结构

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

Vnode 结构的形式没有一定的结构，但必须包括三点，元素的标签，元素的属性与数据，子节点。

- 元素的标签，其命名没有固定格式，可以叫 tag 或者叫 sel。
- 标签的属性或者说是数据，即 props，包含 className，id 等，也可以包含 style 等其他属性与数据，style 或其他属性，比如绑定的事件也可以剥离出去新建一个层级进行表示。
- 最后是该节点的子节点可以用一个数组来表示[]，但是结构一定要规范。

## 22. 监听 data 变化的核心 API 是什么

- `Object.defineProperty`
- 深度监听与监听数组

### API 基本使用，getter 与 setter

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

> get:属性的 getter 函数，当访问该属性时，会调用此函数。  
> set:属性的 setter 函数，当属性值被修改时，会调用此函数。

### 深度监听

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

### 监听数组

要点：重新定义原型 arrProto 在不污染全局原型的情况下扩展数组的方法

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

### `Object.defineProperty`

- 基础 API - `Object.defineProperty` 的使用
- 如何监听对象（深度监听），监听数组
- `Object.defineProperty`的缺点:
  - 深度监听，需要递归到底，一次性计算量大
  - API 无法监听新增属性/删除属性，例如`data.x=100`（所以 Vue2.0 需要 Vue.set Vue.delete）
  - 无法原生监听数组，需要特殊处理

## 23. 描述响应式原理

> - 组件 data 的数据一旦发生变化，立刻触发视图的更新
> - 实现数据驱动视图的第一步
> - 考察 Vue 原理的第一道题

- 监听 data 变化 （参考 22）
- 结合组件渲染和更新的流程

![vue_process](vue_process.png)

## 24. diff 算法的时间复杂度

- O(n)
- 是在 O(n^3)基础上调整优化而来

### diff 算法的优化

- 第一，遍历 tree1；第二，遍历 tree2
- 第三，排序
- 1000 个节点，要计算 1 亿次，算法是不可用的

### 如何优化时间复杂度到 O(n)

1. 只比较同一层级，不跨级比较
2. tag 不相同，则直接删掉重建，不再深度比较
3. tag 和 key，两者都相同，则认为是相同节点，不再深度比较

> 为何 v-for 要是用 key？

如果不使用 key 或使用随机数，在更新 vdom 时，不变的节点只能全部删掉重建。key 使用 index 时，若新的 vdom 中节点顺序发生了改变，也只能将节点删掉重建（index 仅仅按顺序赋值 0 1 2 3 4）。

**认为是相同节点的条件：tag 和 key 两者都相同**

> 如果使用了 key，在对比 children 时，可以根据 tag 与 key 快速得出哪些新旧节点是相同的，直接移动即可。

## 25. 简述 diff 算法过程

> 挑一种描述即可，重点是过程

- patch(elem,vnode) 和 patch(vnode,newVnode)
- patchVnode 和 addVnodes 和 removeVnodes
- updateChildren (key 的重要性)

## 26. Vue 为何是异步渲染，$nextTick 有何用？

- 异步渲染（以及合并 data 修改），以调高渲染性能
- $nextTick 在 DOM 更新完之后，触发回调

## 27. Vue 常见的新能优化方式

- 合理使用 v-show 和 v-if
- 合理使用 computed （缓存）
- v-for 时加 key，以及避免和 v-if 同时使用
  > v-for 优先级比 v-if 更高，循环出来的元素每一个都要进行 v-if 计算
- 自定义事件、DOM 事件及时销毁
- 合理使用异步组件
- 合理使用 keep-alive
- data 层级尽量扁平，不要太深 （深度监听需要一次性遍历完成）
- 使用 vue-loader 在开发环境做模板编译（预编译）
- webpack 层面的优化
- 前端通用的性能优化，如图片懒加载
- 使用 SSR

---

# Vue 知识点 (2)

## Vue3

- Vue3 比 Vue2 有什么优势？
- 描述 Vue3 生命周期
- 如何看待 Composition API 和 Options API？
- 如何理解 ref toRef 和 toRefs？
- Vue3 升级了哪些重要功能？
- Composition API 如何实现代码逻辑复用？
- Vue3 如何实现响应式？
- watch 和 watchEffect 的区别是什么？
- setup 中如何组件实例？ （this）
- Vue3 为何比 Vue2 快？
- Vite 是什么？
- Composition API 和 React Hooks 的对比

## Vue3 比 Vue2 有什么优势？

- 性能更好
- 体积更小
- 更好 typescript 支持
- 更好的代码组织
- 更好的逻辑抽离
- 更多新功能

# Vue3 生命周期

- Options APT 生命周期
- Composition API 生命周期

## Options APT 生命周期

- `beforeDestory` 改为 `beforeUnmount`
- `destoryed` 改为 `unmounted`
- 其他沿用 Vue2 的生命周期

## Composition API 生命周期

```html
<script>
  import {
    onBeforeMount,
    onMounted,
    onBeforeUpdate,
    onUpdated,
    onBeforeUnmount,
    onUnmounted
  } from 'vue';
  export default {
    // 等于 beforeCreate 和 created
    setup() {
      console.log('setup');

      onBeforeMount(() => {
        console.log('onBeforeMount');
      });
      onMounted(() => {
        console.log('onMounted');
      });
      onBeforeUpdate(() => {
        console.log('onBeforeUpdate');
      });
      onUpdated(() => {
        console.log('onUpdated');
      });
      onBeforeUnmount(() => {
        console.log('onBeforeUnmount');
      });
      onUnmounted(() => {
        console.log('onUnmounted');
      });
    }
  };
</script>
```

# Vue3 升级了哪些重要功能

- creatAPP
- emits 属性
- 生命周期
- 多事件处理
- Fragment
- 移除 .sync
- 异步组件的写法
- 移除 filter
- Teleport
- Suspense
- Composition API

## 1. creatAPP

```javascript
// Vue2.x
const app = new Vue({
  /* 选项 */
});

// Vue3
const app = Vue.creatApp({
  /* 选项 */
});
```

```javascript
// Vue2.x
Vue.use(/* ... */);
Vue.mixin(/* ... */);

// Vue3
app.use(/* ... */);
app.mixin(/* ... */);
```

## 2. emits 属性

> 父组件将事件传递给子组件，由子组件调用

父组件

```html
<HelloWord :msg="msg" @onSayHello="sayHello" />
```

子组件

```javascript
export default {
  name: 'HelloWord',
  props: {
    msg: String
  },
  emits: ['onSayHello'],
  setup(props, { emit }) {
    emit('onSayHello', 'bbb');
  }
};
```

## 3. 多事件处理

```html
<!-- 在 methods 里定义 one two 两个函数 -->
<button @click="one($event),two($event)">Submit</button>
```

## 4. Fragment

Vue2.x 模板必须是单一根节点，大多时候需要在最外层用一个`<div>`进行包裹。
Vue3 则没有这个限制。

```html
<!-- Vue2.x 组件模板 -->
<template>
  <div>
    <div class="blog-post">
      <h3>{{ title }}</h3>
      <div v-html="content"></div>
    </div>
  </div>
</template>
```

```html
<!-- Vue3 组件模板 -->
<template>
  <h3>{{ title }}</h3>
  <div v-html="content"></div>
</template>
```

## 5. 移除 .sync

```html
<!-- Vue2.x -->
<MyComponent v-bind:title.sync="title" />

<!-- Vue3 -->
<MyComponent v-model:title="title" />
```

## 6. 异步组件

Vue2.x

```javascript
new Vue({
  // ...
  components: {
    'my-component': () => import('./my-async-component.vue')
  }
});
```

Vue3

```javascript
import { creatApp, defineAsyncComponent } from 'vue';

creatApp({
  // ...
  components: {
    AsyncComponent: defineAsyncComponent(() =>
      import('./my-async-component.vue')
    )
  }
});
```

## 7. 移除 filter

以下 filter 在 Vue3 中已不可用

```html
<!-- 在双花括号中 -->
{{ message | capitalize }}

<!-- 在 v-bind 中 -->
<div v-bind:id="rawId | formatId"></div>
```

## 8. Teleport

> 可以把一些组件放在外面，比如直接挂在 body 下

```html
<!-- data 中设置 modalOpen: false -->
<button @click="modelOpen=true">Open full screen model</button>

<teleport to="body">
  <div v-if="modalOpen" class="modal">
    <div>
      teleport 弹窗 （父元素是 body）
      <button @click="modelOpen=false">Close</button>
    </div>
  </div>
</teleport>
```

## 9. Suspense

允许等待整个组件树处理完毕而不是单个组件。`default` 插槽里的节点会尽可能展示出来。如果不能，则展示 `fallback` 插槽里的节点。

```html
<Suspense>
  <template #default>
    <Test1 />
    <!-- 是一个异步组件 -->
  </template>

  <!-- #fallback 是一个具名插槽。
    即 Suspense 组件内部，有两个 slot，
    其中一个名为 fallback-->

  <template #fallback> Loading... </template>
</Suspense>
```

## 10. Composition API

- reactive
- ref 相关
- readonly
- watch 和 watchEffect
- setup
- 生命周期钩子函数

# v-model 参数的用法

## `.sync`修饰符

对一个 prop 进行“双向绑定”，即为子组件绑定一个属性，子组件在修改这个属性时可以同步到父组件中。

```javascript
// 子组件
this.$emit('update:title', newTitle);
```

```html
<!-- 父组件 -->
<text-document
  v-bind:title="doc.title"
  v-on:update:title="doc.title = $event"
></text-document>
```

`.sync`修饰符就是这种模式的缩写。

```html
<text-document v-bind:title.sync="doc.title"></text-document>
```

## `v-model`参数

Vue3 中放弃了`.sync`修饰符，而用`v-model`参数代替。

```html
<!-- 父组件 -->
<ChildComponent v-model:title="pageTitle" />

<!-- 是以下的简写: -->

<ChildComponent :title="pageTitle" @update:title="pageTitle = $event" />
```

```html
<!-- 子组件 -->
<template>
  <input
    :value="pageTitle"
    @input="$emit('update:pageTitle',$event.target.value)"
  />
</template>
<script>
  export default {
    name: 'ChildComponent',
    props: {
      pageTitle: String
    }
  };
</script>
```

1. 父组件使用`v-model`为子组件绑定一个属性
2. 子组件用 props 接收
3. 子组件在模板中使用该属性
4. 并且绑定一个触发事件`$emit('update:pageTitle',$event.target.value)`，事件的名称为`'update:pageTitle'`，事件的参数是`$event.target.value`

# Vue3 为何比 Vue2 快

- Proxy 响应式
- PatchFlag
- hoistStatic
- cacheHandler
- SSR 优化
- tree-shaking

## PatchFlag 静态标记

- 编译模板时，动态节点做标记
- 标记，分为不同的类型，如 TEXT PROPS
- diff 算法时，可以区分静态节点，以及不同类型的动态节点

### 代码演示

```html
<div>
  <span>Hello World!</span>
  <span>{{ msg }}</span>
  <span :class="name">Azem</span>
  <span :id="age" :msg="msg">{{ age }}</span>
</div>
```

```javascript
import {
  createElementVNode as _createElementVNode,
  toDisplayString as _toDisplayString,
  normalizeClass as _normalizeClass,
  openBlock as _openBlock,
  createElementBlock as _createElementBlock
} from 'vue';

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (
    _openBlock(),
    _createElementBlock('div', null, [
      _createElementVNode('span', null, 'Hello World!'),
      _createElementVNode(
        'span',
        null,
        _toDisplayString(_ctx.msg),
        1 /* TEXT */
      ),
      _createElementVNode(
        'span',
        {
          class: _normalizeClass(_ctx.name)
        },
        'Azem',
        2 /* CLASS */
      ),
      _createElementVNode(
        'span',
        {
          id: _ctx.age,
          msg: _ctx.msg
        },
        _toDisplayString(_ctx.age),
        9 /* TEXT, PROPS */,
        ['id', 'msg']
      )
    ])
  );
}
```

> 通过 PatchFlag 标记，对动态节点进行快速比较。  
> Vue3 对 diff 算法的优化更多的集中于整体流程，而非算法本身，例如 PatchFlag 是对算法输入的变更与优化。

![PatchFlag](patchflag.png)

## hoistStatic

- 将静态节点的定义，提升到父作用域，缓存起来
- 多个相邻的静态节点，会被合并起来
- 典型的用空间换时间的优化策略

### 代码演示

> 将静态节点的定义，提升到父作用域，缓存起来

```html
<div>
  <span>Hello World!</span>
  <span>Hello World!</span>
  <span>Hello World!</span>
  <span>{{ msg }}</span>
</div>
```

```javascript
import {
  createElementVNode as _createElementVNode,
  toDisplayString as _toDisplayString,
  openBlock as _openBlock,
  createElementBlock as _createElementBlock
} from 'vue';

const _hoisted_1 = /*#__PURE__*/ _createElementVNode(
  'span',
  null,
  'Hello World!',
  -1 /* HOISTED */
);
const _hoisted_2 = /*#__PURE__*/ _createElementVNode(
  'span',
  null,
  'Hello World!',
  -1 /* HOISTED */
);
const _hoisted_3 = /*#__PURE__*/ _createElementVNode(
  'span',
  null,
  'Hello World!',
  -1 /* HOISTED */
);

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (
    _openBlock(),
    _createElementBlock('div', null, [
      _hoisted_1,
      _hoisted_2,
      _hoisted_3,
      _createElementVNode(
        'span',
        null,
        _toDisplayString(_ctx.msg),
        1 /* TEXT */
      )
    ])
  );
}
```

### 当静态节点到达一定数量时

> 合并多个相邻的静态节点

```html
<div>
  <span>Hello World!</span>
  <!-- 插入更多的 <span>Hello World!</span> -->
  <span>Hello World!</span>
  <span>{{ msg }}</span>
</div>
```

```javascript
import {
  createElementVNode as _createElementVNode,
  toDisplayString as _toDisplayString,
  createStaticVNode as _createStaticVNode,
  openBlock as _openBlock,
  createElementBlock as _createElementBlock
} from 'vue';

const _hoisted_1 = /*#__PURE__*/ _createStaticVNode(
  '<span>Hello World!</span><span>Hello World!</span><span>Hello World!</span><span>Hello World!</span><span>Hello World!</span><span>Hello World!</span><span>Hello World!</span><span>Hello World!</span><span>Hello World!</span><span>Hello World!</span><span>Hello World!</span><span>Hello World!</span><span>Hello World!</span>',
  13
);

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (
    _openBlock(),
    _createElementBlock('div', null, [
      _hoisted_1,
      _createElementVNode(
        'span',
        null,
        _toDisplayString(_ctx.msg),
        1 /* TEXT */
      )
    ])
  );
}
```

## CacheHandler

- 缓存事件

### 代码演示

```html
<div>
  <span @click="clickHandler">Hello World!</span>
</div>
```

```javascript
import {
  createElementVNode as _createElementVNode,
  openBlock as _openBlock,
  createElementBlock as _createElementBlock
} from 'vue';

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (
    _openBlock(),
    _createElementBlock('div', null, [
      _createElementVNode(
        'span',
        {
          onClick:
            _cache[0] ||
            (_cache[0] = (...args) =>
              _ctx.clickHandler && _ctx.clickHandler(...args))
        },
        'Hello World!'
      )
    ])
  );
}
```

> 一个典型的缓存写法：

```javascript
onClick: _cache[0] ||
  (_cache[0] = (...args) => _ctx.clickHandler && _ctx.clickHandler(...args));
```

## SSR 优化

- 静态节点直接输出字符串，绕过了 vdom
- 动态节点，还是需要动态渲染

### 代码演示

```html
<div>
  <span>Hello World!</span>
  <span>Hello World!</span>
  <span>Hello World!</span>
  <span>{{ msg }}</span>
</div>
```

```javascript
import { mergeProps as _mergeProps } from 'vue';
import {
  ssrRenderAttrs as _ssrRenderAttrs,
  ssrInterpolate as _ssrInterpolate
} from '@vue/server-renderer';

export function ssrRender(
  _ctx,
  _push,
  _parent,
  _attrs,
  $props,
  $setup,
  $data,
  $options
) {
  const _cssVars = { style: { color: _ctx.color } };
  _push(
    `<div${_ssrRenderAttrs(
      _mergeProps(_attrs, _cssVars)
    )}><span>Hello World!</span><span>Hello World!</span><span>Hello World!</span><span>${_ssrInterpolate(
      _ctx.msg
    )}</span></div>`
  );
}
```

## tree shaking

- 编译时，根据不同情况，引入不同的 API
- 即根据模板内容，动态 import 需要的内容

### 只有静态节点的情况

```html
<div>
  <span>Hello World!</span>
</div>
```

```javascript
import {
  createElementVNode as _createElementVNode,
  openBlock as _openBlock,
  createElementBlock as _createElementBlock
} from 'vue';
```

### 有插值的情况

```html
<div>
  <span>Hello World!</span>
  <span>{{ msg }}</span>
</div>
```

```javascript
import {
  createElementVNode as _createElementVNode,
  toDisplayString as _toDisplayString,
  openBlock as _openBlock,
  createElementBlock as _createElementBlock
} from 'vue';
```

### 稍复杂一点的情况

```html
<div>
  <span v-if="msg">Hello World!</span>
  <span :class="text">{{ msg }}</span>
  <input v-model="data" />
</div>
```

```javascript
import {
  openBlock as _openBlock,
  createElementBlock as _createElementBlock,
  createCommentVNode as _createCommentVNode,
  toDisplayString as _toDisplayString,
  normalizeClass as _normalizeClass,
  createElementVNode as _createElementVNode,
  vModelText as _vModelText,
  withDirectives as _withDirectives
} from 'vue';
```

## 小结

- Proxy 响应式
  > 执行递归的时机：生成响应式对象时 -> 获取到哪一层时执行到哪一层
- PatchFlag
  > 对动态节点添加标记
- hoistStatic
  > 将静态节点的定义，提升到父作用域，缓存起来
  > 合并多个相邻的静态节点
- cacheHandler
  > 缓存事件
- SSR 优化
  > 静态节点直接输出字符串，绕过了 vdom
- tree-shaking
  > 根据模板内容，进行动态 import

# Vite 是什么

- 一个前端打包工具，Vue 作者发起的项目
- 借助 Vue 的影响力，发展较快，和 webpack 竞争
- 优势：开发环境下无需打包，启动快

## Vite 为何启动快？

- 开发环境用 ES6 Module，无需打包 —— 非常快
  > webpack 需要打包后生成 ES5 代码再交给浏览器执行
- 生产环境使用 rollup，并不会快很多

> Vite

```html
<body>
  <script type="module">
    import add from './src/add.js';
    const res = add(1, 2);
    console.log('add res', res);
  </script>
</body>
```

# Composition API 和 React Hooks 对比

- 前者 setup 只会被调用一次，而后者函数（函数组件）会被多次调用
- 前者无需 useMemo useCallback（缓存数据，缓存函数），因为 setup 只调用一次
- 前者无需顾虑调用顺序，而后者需要保证 hooks 的顺序一致
- 前者 reactive + ref 比后者 useState，要难理解
