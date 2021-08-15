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

# Vue 2.0 的响应式

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

## 总结

- 基础 API - `Object.defineProperty`
- 如何监听对象（深度监听），监听数组
- `Object.defineProperty`的缺点:
  - 深度监听，需要递归到底，一次性计算量大 （因此设计 data 的数据结构尽量扁平）
  - API 无法监听新增属性/删除属性，例如`data.x=100`（所以 Vue2.0 需要 Vue.set Vue.delete）
  - 无法原生监听数组，需要特殊处理
