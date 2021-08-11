---
title: 学习笔记：Vue 组件
date: 2021-08-11 15:36:52
tags:
  - Vue 组件
categories:
  - Vue 学习笔记
---

# Vue 组件使用

知识点：

- props 和 $emit
- 组件间通讯 - 自定义事件
- 组件生命周期

## props 和 $emit

父子组件间通信，使用 props 传递数据。

父组件：

```html
<input @add="addHandler" /> <List :list="list" @delete="deleteHandler" />
```

子组件

```html
<ul>
  <li v-for="item in list" :key="item.id">
    {{item.title}}

    <button @click="deleteItem(item.id)">删除</button>
  </li>
</ul>
```

```javascript
export default {
  // props: ['list']
  props: {
    // prop 类型和默认值
    list: {
      type: Array,
      default() {
        return [];
      }
    }
  },
  methods: {
    deleteItem(id) {
      this.$emit('delete', id);
    },
    addTitleHandler(title) {
      // eslint-disable-next-line
      console.log('on add title', title);
    }
  }
};
```

## 自定义事件

兄弟间组件通信可以使用自定义事件。

event.js

```javascript
import Vue from 'vue';

export default new Vue();
```

绑定自定义事件

```javascript
import event from './event';
export default {
  methods: {
    addTitleHandler(title) {
      // eslint-disable-next-line
      console.log('on add title', title);
    }
  },
  mounted() {
    event.$on('onAddTitle', this.addTitleHandler);
  },
  beforeDestroy() {
    // 及时销毁自定义事件，否则可能造成内存泄露
    event.$off('onAddTitle', this.addTitleHandler);
  }
};
```

调用自定义事件

```javascript
import event from './event';
export default {
  data() {
    return {
      title: ''
    };
  },
  methods: {
    addTitle() {
      // 调用父组件的事件
      this.$emit('add', this.title);

      // 调用自定义事件
      event.$emit('onAddTitle', this.title);

      this.title = '';
    }
  }
};
```

> 自定义事件要及时销毁！否则可能造成内存泄露。

# 生命周期（单个组件）

![life_cycle](https://vue3js.cn/docs/zh/images/lifecycle.png)

> 图片引用自 vue3 文档

- 挂载阶段
  > beforeCreate created 实例初始化，只是存在内存中的 js 变量  
  > beforeMount mounted 在网页上绘制完成，渲染完成
- 更新阶段
  > beforeUpdate updated
- 销毁阶段
  > beforeUnmount unmounted

# 生命周期（父子组件）

挂载阶段

- 创建及初始化阶段由外到内（先父组件后子组件）
- 渲染阶段由内到外（先子组件后父组件）

> 先创建及初始化父组件  
> 子组件渲染完毕后，父组件才算完成渲染

![父子组件生命周期01-1](vue_lifecycle_01-1.png)

![父子组件生命周期01](vue_lifecycle_01.png)

更新阶段

- before update 由外到内
- updated 由内到外

> 父组件的 data 先被修改，并更新子组件  
> 子组件更新渲染完毕后，父组件才算完成更新

![父子组件生命周期02](vue_lifecycle_02.png)

销毁阶段

- 子组件销毁后，父组件才能销毁

![父子组件生命周期03](vue_lifecycle_03.png)
