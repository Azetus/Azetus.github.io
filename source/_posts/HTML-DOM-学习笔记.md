---
title: HTML DOM 学习笔记
date: 2021-07-22 12:30:52
tags:
  - DOM
categories:
  - JS-Web-API学习笔记
---

# 知识点

**DOM 操作 （Document Object Module)**

- DOM 属于那种数据结构
- DOM 操作的常用 API
- attribute 和 property 的区别
- 一次性插入多个 DOM 节点，考虑性能

- DOM 的本质
- DOM 节点的操作
- DOM 结构的操作
- DOM 性能

---

# DOM 的本质

```xml
<?xml version="1.0" encoding="UTF-8"?>
<note>
  <to>Tove</to>
  <from>Jani</from>
  <heading>Reminder</heading>
  <body>Don't forget me this weekend!</body>
  <other>
    <a></a>
    <b></b>
  </other>
</note>
```

```HTML
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Document</title>
</head>
<body>
  <div>
    <p>this is p</p>
  </div>
</body>
</html>
```

> HTML 是一种特定的 XML，规定了可以使用的各个标签。

**DOM 的本质是一棵树**
浏览器从 HTML 文件解析出的一棵树

---

# DOM 节点操作

- 获取 DOM 节点
- attribute
- property

## 获取 DOM 节点

```JavaScript
const div1 = document.getElementById('div1') // 元素
const divList = document.getElementById('div') // 集合
console.log(divList.length)
console.log(divList[0])

const containerList = document.getElementsByClassName('container') // 集合

const pList = document.querySelectorAll('p') // 集合
```

## DOM 节点的 property

```JavaScript
const pList = document.querySelectorAll('p') // 集合

const p = pList[0]
p.style.width = '100px'     // 修改样式
console.log(p.style.width)  // 获取样式
p.className = 'red'         // 修改class
console.log(p.className)    // 获取 class

// 获取 nodeName 和 nodeType
console.log(p.nodeName)
console.log(p.nodeType)
```

> 获取 DOM 元素通过 JS 属性的方式进行操作

## DOM 节点的 attribute

> 修改的是 DON 节点的属性

## 小结

- property：修改对象属性，不会提现到 html 结构中（尽量用此种方式）
- attribute：修改 html 属性，会改变 html 结构
- 两者都有可能引起 DOM 重新渲染

---

# DOM 结构操作

- 新增/插入节点
- 获取子元素列表，获取父元素
- 删除子元素

```HTML
<body>
  <div id="div1" class="container">
    <p>一段文字 1</p>
    <p id="p2">一段文字 2</p>
    <p>一段文字 3</p>
  </div>
  <div id="div2" class="container">
    <img src="./w8h09ebV24.png" />
  </div>
</body>
```

```JavaScript
const div1 = document.getElementById('div1');
const div2 = document.getElementById('div2');

// 添加新节点
const p1 = document.createElement('p');
p1.innerHTML = 'this is p1';
div1.appendChild(p1); // 添加新创建的元素
// 移动已有节点
const p2 = document.getElementById('p2');
div2.appendChild(p2);

// 获取父元素
console.log(p2.parentNode);

// 获取子元素列表
const div1ChildNodes = div1.childNodes;
console.log(div1ChildNodes);
console.log(div1ChildNodes[0].nodeName, div1ChildNodes[0].nodeType);
console.log(div1ChildNodes[1].nodeName, div1ChildNodes[1].nodeType);

const div1ChildNodesP = Array.from(div1.childNodes).filter((child) => {
  if (child.nodeType === 1) {
    return true;
  }
  return false;
});
console.log(div1ChildNodesP);

// 删除节点
div1.removeChild(div1ChildNodesP[0]);
```

---

# DOM 性能

- DOM 操作非常"昂贵"，避免频繁的 DOM 操作。
- 对 DOM 查询做缓存。
- 将频繁操作改为一次性操作。

```HTML
<body>
  <ul id="list"></ul>
</body>
```

```JavaScript
const list = document.getElementById('list');

// 创建一个文档片段，此时还没要插入到 DOM 树种
const frag = document.createDocumentFragment();

// 执行插入
for (let i = 0; i < 10; i++) {
  const li = document.createElement('li');
  li.innerHTML = `List item ${i + 1}`;

  // 先插入文档片段中
  frag.appendChild(li);
}

// 都完成之后，再统一插入到 DOM 结构中
list.appendChild(frag);
```

---

# 总结

**DOM 是哪种数据结构**

- 树（DOM 树）

**DOM 操作常用 API**

- DOM 节点操作
- DOM 结构操作
- attribute 和 property

**attribute 和 property 的区别**

- property：修改对象属性，不会体现到 html 结构中
- attribute：修改 html 属性，会改变 html 结构
- 二者都有可能引起 DOM 重新渲染

**一次性插入多个节点，考虑性能**

- 使用文档片段，一次性插入 DOM 树种。

```JavaScript
const frag = document.createDocumentFragment()
```
