---
title: 学习笔记：React 基本使用(3)
date: 2021-09-03 16:41:51
tags:
  - React
categories:
  - React 学习笔记
---

# 高级特性

- 函数组件
- 非受控组件
- Portals

# 函数组件

## class 组件

```jsx
class List extends React.Component {
  constructor(props) {
    super(props);
  }
  render() {
    const { list } = this.props;

    return (
      <ul>
        {list.map((item, index) => {
          return (
            <li key={item.id}>
              <span>{item.title}</span>
            </li>
          );
        })}
      </ul>
    );
  }
}
```

## 函数组件

- 纯函数，输入 props，输出 JSX
- 没有实例，没有生命周期，没有 state ，不包含逻辑
- 不能扩展其他方法

```jsx
function List(props) {
  const { list } = props;
  return (
    <ul>
      {list.map((item, index) => {
        return (
          <li key={item.id}>
            <span>{item.title}</span>
          </li>
        );
      })}
    </ul>
  );
}
```

## 如何选择

- 若组件只接收输入的 props 并输出一个 JSX，选择函数组件
- 如果组件包含其他逻辑，必须需要设置 state，需要调用声明周期，选择 class 组件

# 非受控组件

- ref
- defaultValue defaultChecked
- 手动操作 DOM 元素

## ref

`this.nameInputRef = React.createRef()`
需要使用`React.createRef()`创建 ref ，并赋值给`this`的一个属性。在 render 中使用`ref={this.nameInputRef}`使用 ref。  
在使用时，通过`this.nameInputRef.current`获取当前 DOM 节点。

## defaultValue

在 render 中使用 `defaultValue` 而不是 `value`，并且没有 `onChange` 事件，因此在`<input>` 中输入的内容并不会改变 state。
而 `alert` 使用的值是直接从 DOM 中获取的，因此可以弹出输入的内容。

> `<input>` value -> defaultValue

```jsx
import React from 'react';

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      name: 'Monika',
      flag: true
    };
    this.nameInputRef = React.createRef(); // 创建 ref
    this.fileInputRef = React.createRef();
  }
  render() {
    // input defaultValue
    return (
      <div>
        {/* 使用 defaultValue 而不是 value ，使用 ref */}
        <input defaultValue={this.state.name} ref={this.nameInputRef} />
        {/* state 并不会随着改变 */}
        <span>state.name: {this.state.name}</span>
        <br />
        <button onClick={this.alertName}>alert name</button>
      </div>
    );
  }
  alertName = () => {
    const elem = this.nameInputRef.current; // 通过 ref 获取 DOM 节点
    alert(elem.value); // 不是 this.state.name
  };
}
```

## defaultChecked

> `<input type="checkbox">` checked -> defaultChecked

```jsx
return (
  <div>
    <input type="checkbox" defaultChecked={this.state.flag} />
  </div>
);
```

## file

> 当无法使用 state，必须使用 DOM 操作时。比如获取文件名时。

```jsx
import React from 'react';

class App extends React.Component {
  constructor(props) {
    super(props);
    this.fileInputRef = React.createRef();
  }
  render() {
    // file
    return (
      <div>
        <input type="file" ref={this.fileInputRef} />
        <button onClick={this.alertFile}>alert file</button>
      </div>
    );
  }
  alertFile = () => {
    const elem = this.fileInputRef.current; // 通过 ref 获取 DOM 节点
    alert(elem.files[0].name);
  };
}
```

## 小结

- 必须手动操作 DOM 元素，setState 实现不了
- 比如文件上传`<input type=file>`
- 某些富文本编辑器，需要传入 DOM 元素
- 如何选择：
  - 优先使用受控组件，复合 React 设计原则（数据驱动视图）
  - 必须操作 DOM 时，在使用非受控组件

# Portals

- React 组件默认会按照既定层次嵌套渲染
- 如何让组件渲染到父组件以外

## 示例

### 父组件

```jsx
return (
  <div>
    <PortalsDemo>Modal 内容</PortalsDemo>
  </div>
);
```

### 子组件

> `this.props.children` 可以获取组件内的东西（类似 vue slot）

#### 正常渲染的情况

会嵌套在很多层`<div>`内部。

```jsx
import React from 'react';
import './style.css';

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {};
  }
  render() {
    // 正常渲染
    return (
      <div className="modal">
        {this.props.children} {/* 类似 vue slot */}
      </div>
    );
  }
}

export default App;
```

```css
.modal {
  position: fixed;
  width: 300px;
  height: 100px;
  top: 100px;
  left: 50%;
  margin-left: -150px;
  background-color: #000;
  /* opacity: .2; */
  color: #fff;
  text-align: center;
}
```

#### 使用 Portal 的情况

引入`ReactDOM`，并使用`ReactDOM.createPortal()`创建 Portal，其第二个参数为元素需要渲染的 DOM 节点位置。此时该组件将被挂载在`<body>`下第一层级。

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import './style.css';

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {};
  }
  render() {
    // 使用 Portals 渲染到 body 上。
    // fixed 元素要放在 body 上，有更好的浏览器兼容性。
    return ReactDOM.createPortal(
      <div className="modal">{this.props.children}</div>,
      document.body // DOM 节点
    );
  }
}

export default App;
```

## Portals 的使用场景

- 父组件设置了`overflow: hidden`，父组件使用了 BFC ，限制了子组件的展示
- 父组件 z-index 值太小
- `position: fixed;` 的组件需要放在 body 第一层级
- 主要针对 CSS 兼容性和布局相关的问题
