---
title: 学习笔记：Composition API
date: 2021-08-20 19:37:04
tags:
  - Vue3
  - Composition API
categories:
  - Vue 学习笔记
---

# Composition API

- 更好的代码组织
- 更好的逻辑复用
- 更好的类型推导

```html
<script>
  // 将原本的功能进行拆分：

  // 将list相关功能进行封装
  const listRelativeEffect = () => {
    // ...
  };

  // 将inputValue相关功能进行封装
  const inputRelativeEffect = () => {
    // ...
  };

  const app = Vue.createApp({
    setup() {
      // 流程的调度中转
      // 从不同的地方找到需要的东西
      const { list, addItemToList } = listRelativeEffect();
      const { inputValue, handleInputValueChange } = inputRelativeEffect();

      return {
        list,
        addItemToList,
        inputValue,
        handleInputValueChange
      };
    },
    template: `
            <div>
                <!-- ... -->
            </div>
        `
  });

  const vm = app.mount('#root');
</script>
```

**如何选择**

- 不建议共用，会引起混乱
- 小型项目、业务逻辑简单，用 Options API
- 中大型项目、逻辑复杂，用 Composition API

> Composition API 是为了解决复杂业务逻辑而设计  
> Composition API 就像 Hooks 在 React 中的地位

## 小结

- Composition API 带来了什么？
  - 更好的代码组织
  - 更好的逻辑复用
  - 更好的类型推导
- Composition API 和 Options API 如何选择？
  - 不建议共用，会引起混乱
  - 小型项目、业务逻辑简单，用 Options API
  - 中大型项目、逻辑复杂，用 Composition API

# Composition API 如何实现逻辑复用

- 抽离逻辑代码到一个函数
- 函数命名约定为 useXxxx 格式（React Hooks 也是）
- 在 setup 中引用 useXxxx 函数

## 代码演示

获取光标在页面的位置。可以将逻辑抽离出去，并放到一个 js 文件中，在使用时`import xxxx form './xxx'`引入。

```javascript
import { reactive, ref, onMounted, onUnmounted } from 'vue';

function useMousePosition(params) {
  const xRef = ref(0);
  const yRef = ref(0);

  // ref 修改值时使用 .value
  function update(e) {
    xRef.value = e.pageX;
    yRef.value = e.pageY;
  }

  onMounted(() => {
    console.log('useMousePosition mounted');
    window.addEventListener('mousemove', update);
  });

  onUnmounted(() => {
    console.log('useMousePosition unMounted');
    window.removeEventListener('mousemove', update);
  });

  // 使用 ref 实现响应式的写法
  return {
    xRef,
    yRef
  };
  /*
    // 直接返回一个 reactive 对象的写法
    const state = reactive({
      x: 0,
      y: 0
    });

    function update(e) {
      state.x = e.pageX;
      state.y = e.pageY;
    }
    return {
      state
    };
  */
}
```

```html
<template>
  <p>mouse position {{xRef}} {{yRef}}</p>
</template>

<script>
  // import { reactive } from "vue";
  import useMousePosition from './useMousePosition'

  export default {
    name: 'MousePosition'
  setup() {
      const { xRef, yRef } = useMousePosition()
      /*
        // 若方法返回一个 reactive 对象时的写法
        // 模板应改为 state.x 和 state.y
        const state = useMousePosition2()
        return {
            state
        }
      */
      return {
        xRef,
        yRef
      }
    }
  }
</script>
```
