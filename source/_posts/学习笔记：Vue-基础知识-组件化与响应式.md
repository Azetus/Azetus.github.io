---
title: 学习笔记：Vue 基础知识 - 组件化与响应式
date: 2021-08-15 19:53:32
tags:
  - Vue 组件
  - 响应式
categories:
  - Vue 学习笔记
---

# Vue 的组件化与响应式

## 组件化基础

- 传统组件，只是静态渲染，更新还要依赖于操作 DOM
- 数据驱动视图
  - Vue - MVVM
  - React - setState

## Vue MVVM

> Model View ViewModel

![vue_mvvm](https://012.vuejs.org/images/mvvm.png)

> 图片引用自 vue 官方文档

通过 VM 将视图(view)和数据(model)连接起来，通过修改 Model 中的数据可以直接修改视图。

# Vue 2.x 的响应式

> 组件 data 的数据一旦发生变化，立刻触发视图的更新  
> 实现数据驱动视图的第一步

- 核心 API - `Object.defineProperty`
- 如何实现响应式
- `Object.defineProperty` 的一些缺点（Vue3.0 启用 Proxy）

## Proxy 有兼容性问题

- Proxy 兼容性不好，且无法 polyfill

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
  - 深度监听，需要递归到底，一次性计算量大 （因此设计 data 的数据结构尽量扁平）
  - API 无法监听新增属性/删除属性，例如`data.x=100`（所以 Vue2.0 需要 Vue.set Vue.delete）
  - 无法原生监听数组，需要特殊处理

# Vue 3 的响应式

## Proxy

- 基本使用
- Reflect
- 实现响应式

> Proxy 对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）。  
> 创建响应式对象时，使用带有 getter 和 setter 的处理程序遍历其所有 property 并将其转换为 Proxy。  
> Proxy 是一个包含另一个对象或函数并允许你对其进行拦截的对象。

示例：

```javascript
const data = {
  name: 'Aliza',
  age: 20
};
```

创建一个 Proxy 对象，并添加 get 与 set 方法，当查询或修改 proxyData 时便会调用创建时设置的 get 与 set 方法，从而实现响应式。同时，对 proxyData 的修改也会被同步到其原始对象 data 上。

```javascript
const proxyData = new Proxy(data, {
  get(target, key, receiver) {
    const result = Reflect.get(target, key, receiver);
    console.log('get', key);
    return result; // 返回结果
  },
  set(target, key, val, receiver) {
    const result = Reflect.set(target, key, val, receiver);
    console.log('set', key, val);
    return result; // 是否设置成功
  },
  deleteProperty(target, key) {
    const result = Reflect.deleteProperty(target, key);
    console.log('delete property', key);
    return result; // 是否删除成功
  }
});
```

数组的情况：

```javascript
const data = ['a', 'b', 'c'];

const proxyData = new Proxy(data, {
  get(target, key, receiver) {
    // 只监听处理本身（非原型的）属性，原型的属性不处理
    // 若没有这一步会打印出 'get push'
    const ownKeys = Reflect.ownKeys(target);
    if (ownKeys.includes(key)) {
      console.log('get', key); // 监听
    }

    const result = Reflect.get(target, key, receiver);
    return result; // 返回结果
  },
  set(target, key, val, receiver) {
    // 重复的数据，不处理
    if (val === target[key]) {
      return true;
    }

    const result = Reflect.set(target, key, val, receiver);
    console.log('set', key, val);
    // console.log('result', result) // true
    return result; // 是否设置成功
  },
  deleteProperty(target, key) {
    const result = Reflect.deleteProperty(target, key);
    console.log('delete property', key);
    // console.log('result', result) // true
    return result; // 是否删除成功
  }
});
```

### 一些 Proxy 相关的 API

#### `new Proxy(target, handler)`

1. `target`

&emsp;Proxy 会对 target 对象进行包装。它可以是任何类型的对象，包括内置的数组，函数甚至是另一个代理对象。

2. `handler`

&emsp;它是一个对象，它的属性提供了某些操作发生时所对应的处理函数。

#### `handler(target, key, val, receiver)`

1. `target`

&emsp;目标对象（即例子中的 data）

2. `key`

&emsp;将被设置的属性名（data.name data.age）

3. `val`

&emsp;新属性值

4. `receiver`

&emsp;最初被调用的对象。通常是 proxy 本身，但 handler 的 set 方法也有可能在原型链上，或以其他方式被间接地调用（因此不一定是 proxy 本身）。

> **比如**：假设有一段代码执行 `obj.name = "jen"`， obj 不是一个 proxy，且自身不含 name 属性，但是它的原型链上有一个 proxy，那么，那个 proxy 的 set() 处理器会被调用，而此时，obj 会作为 receiver 参数传进来。

### Reflect 的作用

- 和 Proxy 的能力一一对应（API 与参数相同）
- 规范化、标准化、函数式
  > `'a' in obj` vs `Reflect.has(obj,'a')`  
  > `delete obj.a` vs `Reflect.deleteProperty(obj, 'a')`
- 代替 Object 上的工具函数
  > `Object.getOwnPropertyNames(obj)` vs ` Reflect.ownKeys(obj)`

## Proxy 实现响应式

- 设置代理配置
- 实现深度监听

```javascript
// 创建响应式
function reactive(target = {}) {
  if (typeof target !== 'object' || target == null) {
    // 不是对象或数组，则返回
    return target;
  }

  // 代理配置
  const proxyConf = {
    get(target, key, receiver) {
      // 只处理本身（非原型的）属性
      const ownKeys = Reflect.ownKeys(target);
      if (ownKeys.includes(key)) {
        console.log('get', key); // 监听
      }

      const result = Reflect.get(target, key, receiver);

      // 深度监听
      // 只有在用到的时候才执行递归，性能提升
      // 本质是，获取到哪一层，哪一层才会获得响应式
      return reactive(result);
    },
    set(target, key, val, receiver) {
      // 重复的数据，不处理
      if (val === target[key]) {
        return true;
      }

      const ownKeys = Reflect.ownKeys(target);
      if (ownKeys.includes(key)) {
        console.log('已有的 key', key);
      } else {
        console.log('新增的 key', key);
      }

      const result = Reflect.set(target, key, val, receiver);
      console.log('set', key, val);
      // console.log('result', result) // true
      return result; // 是否设置成功
    },
    deleteProperty(target, key) {
      const result = Reflect.deleteProperty(target, key);
      console.log('delete property', key);
      // console.log('result', result) // true
      return result; // 是否删除成功
    }
  };

  // 生成代理对象
  const observed = new Proxy(target, proxyConf);
  return observed;
}
```

> 为何 Proxy 实现深度监听的性能更好?

利用 Proxy 使用递归实现对对象的深度监听时，只有在被监听对象的深层属性被用到时才进行递归。对于多层嵌套的对象，获取到哪一层时，那一层才会获得响应性。而`Object.defineProperty`实现深度监听时，需要在创建响应式对象时对对象进行完整的递归。

如果使用 Proxy 对如下对象进行深度监听时。

```javascript
const data = {
  str: 'proxy',
  num: {
    count: 5,
    a: {
      b: {
        c: 100
      }
    }
  }
};
```

若仅调用 proxyData 时，其 num 属性依旧是一个普通对象，但当调用 proxyData.num.a.b 时，递归才会执行到`{b: {c: 100}}`，深层的 b 属性才会获得响应性。

## 小结

- Proxy 实现的深度监听，性能更好
- Proxy 可监听 新增/删除 属性
- Proxy 可监听数组变化
