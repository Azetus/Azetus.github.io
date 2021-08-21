---
title: Vue3 的生命周期
date: 2021-08-20 19:28:13
tags:
  - Vue3
  - 生命周期
categories:
  - Vue 学习笔记
---

# Vue3 生命周期

- Options API 生命周期
- Composition API 生命周期

## Options API 生命周期

- `beforeDestory` 改为 `beforeUnmount`
- `destoryed` 改为 `unmounted`
- 其他沿用 Vue2 的生命周期

## Composition API 生命周期

`setup` 是围绕 `beforeCreate` 和 `created` 生命周期钩子运行的。原来在 Vue 2 中的 `beforeCreate` 和 `created` 中编写的代码可以直接写在 `setup`函数中。

> 1. 在执行 `setup` 时，组件实例其实尚未被创建。因此在`setup`中是无法通过 this 获取到组件实例。  
> 2. `data` `computed` `methods`等选项也无法在`setup`函数中获取到（可以在`onMounted`生命周期中获取）

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
