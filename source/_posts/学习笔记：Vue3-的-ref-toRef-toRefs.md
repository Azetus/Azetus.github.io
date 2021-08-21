---
title: '学习笔记：Vue3 的 ref, toRef, toRefs'
date: 2021-08-21 16:37:16
tags:
  - Vue3
  - ref
  - toRef
  - toRefs
categories:
  - Vue 学习笔记
---

# ref

- 生成值类型的响应式数据
- 可用于模板和 reactive
- 通过`.value`修改值

### 使用示例：

```html
<template>
  <p>ref demo {{ageRef}} {{state.name}}</p>
</template>

<script>
  import { ref, reactive } from 'vue';

  export default {
    name: 'Ref',
    setup() {
      const ageRef = ref(18); // 值类型 响应式
      const nameRef = ref('Monika');

      const state = reactive({
        name: nameRef
      });

      setTimeout(() => {
        console.log('ageRef', ageRef.value);

        ageRef.value = 25; // .value 修改值
        nameRef.value = 'Aliza';
      }, 1500);

      return {
        ageRef,
        state
      };
    }
  };
</script>
```

### 利用 ref 获取 dom 元素

> 由于`setup()`函数在创建组件之前执行（等于 beforeCreate 和 created），因此如要执行 dom 操作，应将相关代码写在生命周期函数内。

```html
<template>
  <p ref="elemRef">一行文字</p>
</template>

<script>
  import { ref, onMounted } from 'vue';

  export default {
    name: 'RefTemplate',
    setup() {
      const elemRef = ref(null);

      onMounted(() => {
        console.log('ref template', elemRef.value.innerHTML, elemRef.value);
      });

      return {
        elemRef
      };
    }
  };
</script>
```

# toRef

- 针对一个响应式对象（reactive 封装）的 prop （属性）
- 创建一个 ref，具有响应式
- 两者保持引用关系 （应用类型赋值）

> 普通对象实现响应式直接使用`reactive`  
> `reactive`的某一个属性单独实现响应式使用`toRef`

```html
<template>
  <p>toRef demo - {{ageRef}} - {{state.name}} {{state.age}}</p>
</template>

<script>
  import { ref, toRef, reactive } from 'vue';

  export default {
    name: 'ToRef',
    setup() {
      const state = reactive({
        age: 18,
        name: 'Monika'
      });

      const age1 = computed(() => {
        return state.age + 1;
      });
      console.log(age1); // ComputedRefImpl

      // toRef 如果用于普通对象（非响应式对象），产出的结果不具备响应式
      // const state = {
      //     age: 20,
      //     name: 'Aliza'
      // }

      const ageRef = toRef(state, 'age');

      setTimeout(() => {
        state.age = 20;
      }, 1500);

      setTimeout(() => {
        ageRef.value = 21; // .value 修改值
      }, 3000);

      return {
        state,
        ageRef
      };
    }
  };
</script>
```

# toRefs

- 将响应式对象（reactive 封装）转换为普通对象
- 对象的每个 prop（属性）都是对应的 ref
- 两者保持引用关系

> 直接解构 reactive 对象，会使其属性失去响应式

```html
<template>
  <p>toRefs demo {{age}} {{name}}</p>
</template>

<script>
  import { ref, toRef, toRefs, reactive } from 'vue';

  export default {
    name: 'ToRefs',
    setup() {
      const state = reactive({
        age: 18,
        name: 'Monika'
      });

      const stateAsRefs = toRefs(state); // 将响应式对象，变成普通对象

      // return { ...stateAsRefs };
      // const { age: ageRef, name: nameRef } = stateAsRefs
      // 每个属性，都是 ref 对象
      // return {
      //     ageRef,
      //     nameRef
      // }

      setTimeout(() => {
        state.age = 25;
      }, 1500);

      return stateAsRefs;
    }
  };
</script>
```

# ref toRef 和 toRefs 的使用场景

### 合成函数返回响应式对象

```javascript
function useFeatureX() {
  const state = reactive({
    x: 1,
    y: 2
  });

  // 运行逻辑

  // 返回时转换为 ref
  return toRefs(state);
}
```

```javascript
export default {
  setup() {
    // 可以在不失去响应性的情况下结构
    const { x, y } = useFeatureX();

    return {
      x,
      y
    };
  }
};
```

### 使用场景

- 用 reactive 做对象的响应式，用 ref 做值类型的响应式
- `setup()`中返回 `toRefs(state)` ，或者 `toRef(state, 'xxx')`
- ref 的变量命名都用 `xxxRef`
- 合成函数返回响应式对象时，使用 toRefs

# 总结

1. 为何需要 ref ？
2. 为何需要 .value ？
3. 为何需要 toRef 和 toRefs ？

## 1. 为何需要 ref ？

- **返回值类型，会丢失响应式**
- 如在 `setup`， `computed`， 合成函数，都有可能返回值类型
- 因此如果 Vue 不定义 ref，用户将自造 ref，反而混乱

## 2. 为何需要.value ?

- ref 是一个对象（不丢失响应式），value 存储值
- 通过 .value 属性的 get 和 set 实现响应式
- 用于模板、reactive 时，不需要.value,其他情况都需要

**错误的做法**

```javascript
//  错误
function computed(getter) {
  let value = 0;
  setTimeout(() => {
    value = getter();
  }, 1500);
  return value;
}
let a = 0;
let a = computed(() => 100);
/*
  let a = 100
  let b = a
  a = 200
  b // 100
*/
```

> value 是值类型，而非引用类型。异步执行后 100 赋值给 value，但此时 value 已经与 a 没有任何关系，无法做到引用传递。

**正确的做法**

```javascript
//  正确
function computed(getter) {
  const ref = {
    value: null
  };
  setTimeout(() => {
    ref.value = getter();
  }, 1500);
  return ref;
}
let a = {};
let a = computed(() => 100);
/*
  const obj1 = {x:100}
  const obj2 = obj1
  obj1.x = 200
  obj2.x // 200
*/
```

> 对象形式能够保持并传递响应式

## 3. 为何需要 toRef 和 toRefs？

- 初衷：在不丢失响应式的情况下，把对象数据**分散/扩散**（实现解构）
- 前提：针对的是响应式对象（reactive 封装的），非普通对象
- 注意：**不创造**响应式，而是**延续**响应式
