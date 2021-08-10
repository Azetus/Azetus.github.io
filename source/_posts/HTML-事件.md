---
title: HTML 事件
date: 2021-07-23 12:35:13
tags:
  - 事件
categories:
  - JS-Web-API学习笔记
---

# 事件绑定和事件冒泡

## 知识点

- 事件绑定
- 事件冒泡
- 事件代理

## 事件绑定

```JavaScript
const btn = document.getElementById('btn1')
btn.addEventListener('click',event=>{
  console.log('clicked')
})
```

**通用的绑定函数**

```JavaScript
function bindEvent(elem, type, fn) {
  elem.addEventListener(type, fn);
}
const a = document.getElementById('link1');
bindEvent(a, 'click', (e) => {
  e.preventDefault(); // 阻止默认行为
  alert('clicked');
});
```

## 事件冒泡

`event.stopPropagation()`

```HTML
<body>
  <div id="div1">
    <p id="p1">激活</p>
    <p id="p2">取消</p>
    <p id="p3">取消</p>
    <p id="p4">取消</p>
  </div>
  <div id="div2">
    <p id="p5">取消</p>
    <p id="p6">取消</p>
  </div>
</body>
```

```JavaScript
const p1 = document.getElementById('p1');
const body = document.body;
bindEvent(p1, 'click', (e) => {
  e.stopPropagation(); // 该方法将停止事件的传播，阻止它被分派到其他 Document 节点。
  alert('激活');
});
bindEvent(body, 'click', (e) => {
  alert('取消');
});
```

---

# 事件代理

- 代码简洁
- 减少浏览器内存占用
- 不要滥用（适用于瀑布流等）

```JavaScript
function bindEvent(elem, type, selector, fn) {
  if (fn == null) {
    fn = selector;
    selector = null;
  }
  elem.addEventListener(type, (event) => {
    const target = event.target;
    if (selector) {
      // 代理绑定
      if (target.matches(selector)) {
        fn.call(target, event);
      }
    } else {
      // 普通绑定
      fn.call(target, event);
    }
  });
}
```

---

# 小结

描述事件冒泡的流程

- 基于 DOM 树形结构
- 事件会顺着触发元素往上冒泡
- 应用场景：代理

无限下拉列表，如何监听每个列表元素点击

- 事件代理
- 用 e.target 获取触发元素
- 用 matches 判断是否是触发元素
