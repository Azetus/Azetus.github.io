---
title: 学习笔记：虚拟 DOM 和 diff
date: 2021-08-18 10:08:51
tags:
  - VDOM
categories:
  - 学习笔记
---

# 虚拟 DOM( Virtual DOM ) 和 diff

**技术背景**

- DOM 操作**非常**耗费性能
- 使用 jQuery，可以自行控制 DOM 操作的时机，手动调整
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

## Vue2 的 vue-template-compiler

### with 语法

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

### vue-template-compiler 的模板编译

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
- 使用 webpack vue-loader 会在开发环境下编译模板

---

# vdom diff

![diff_tree](diff_tree.png)

## 树结构 diff 的时间复杂度 O(n^3)

- 第一，遍历 tree1；第二，遍历 tree2
- 第三，排序
- 1000 个节点，要计算 1 亿次，这样的算法是不可用的

## 优化时间复杂度到 O(n)

1. 只比较同一层级，不跨级比较
2. tag 不相同，则直接删掉重建，不再深度比较
3. tag 和 key，两者都相同，则认为是相同节点，不再深度比较

**1. 只比较同一层级**

![diff_tree_01](diff_tree_01.png)

**2. tag 不相同，则直接删掉重建，不再深度比较**

![diff_tree_02](diff_tree_02.png)

## 不使用 key 和使用 key

![diff_key](diff_keys.png)

如果不使用 key 或使用随机数，在更新 vdom 时，不变的节点只能全部删掉重建。key 使用 index 时，若新的 vdom 中节点顺序发生了改变，也只能将节点删掉重建（index 仅按顺序赋值，如： 0 1 2 3 4）。

> **认为是相同节点的条件：tag 和 key 两者都相同**

如果使用了 key，在对比 children 时，可以根据 tag 与 key 快速得出哪些新旧节点是相同的，直接移动即可。
