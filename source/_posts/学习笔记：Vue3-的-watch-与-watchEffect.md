---
title: 学习笔记：Vue3 的 watch 与 watchEffect
date: 2021-08-22 19:06:50
tags:
  - Vue3
  - watch
  - watchEffect
categories:
  - Vue 学习笔记
---

# watch 和 watchEffect 的区别

- 两者都可以监听 data 属性变化
- watch 需要明确监听哪个属性
- watchEffect 会根据其中的属性，自动监听其变化

## 代码演示

```javascript
import { reactive, ref, toRefs, watch, watchEffect } from 'vue';

export default {
  name: 'Watch',
  setup() {
    const numberRef = ref(100);
    const state = reactive({
      name: 'Aliza',
      age: 25
    });

    // watch 和 watchEffect

    return {
      numberRef,
      ...toRefs(state)
    };
  }
};
```

### watch

```javascript
watch(
  numberRef,
  (newNumber, oldNumber) => {
    console.log('ref watch', newNumber, oldNumber);
  }
  // , {
  //     immediate: true // 初始化之前就监听，可选
  // }
);

setTimeout(() => {
  numberRef.value = 200;
}, 1500);

watch(
  // 第一个参数，确定要监听哪个属性
  () => state.age,

  // 第二个参数，回调函数
  (newAge, oldAge) => {
    console.log('state watch', newAge, oldAge);
  },

  // 第三个参数，配置项
  {
    immediate: true // 初始化之前就监听，可选
    // deep: true // 深度监听
  }
);

setTimeout(() => {
  state.age = 18;
}, 1500);
setTimeout(() => {
  state.name = 'Monika';
}, 3000);
```

### watchEffect

```javascript
watchEffect(() => {
  // 初始化时，一定会执行一次（收集要监听的数据）
  console.log('watchEffect');
});

watchEffect(() => {
  console.log('state.age', state.age);
  console.log('state.name', state.name);
});

setTimeout(() => {
  state.age = 18;
}, 1500);
setTimeout(() => {
  state.name = 'Monika';
}, 3000);
```

watchEffect **初始化时，一定会执行一次**。收集要监听的数据。
