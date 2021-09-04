---
title: 学习笔记：React 基本使用(4)
date: 2021-09-04 20:25:38
tags:
  - React
categories:
  - React 学习笔记
---

# 知识点

- React Context
- 异步组件

# React Context

- 公共信息（语言、主题）如何传递给每个组件？
- 用 props 太繁琐
- 用 redux 小题大做
- 逻辑并不复杂，但需要一层一层向下传递

## 核心 API

- `React.createContext()`创建 Context，传入初始值
- 父组件使用`.Provider`与`value`设置动态值，可以配合`setState`修改
- 底层 class 组件`Component.contextType`或`static contextType`获取 Context
- 底层函数组件使用`<ThemeContext.Consumer>`包裹函数，函数接收 value 参数获取 Context

## 演示

最外层组件为 APP，第二层为 Toolbar，第三层为 ThemedButton 和 ThemeLink。
将最外层组件作为 context 数据的生产方，同时作为修改方（状态提升）。其下层的组件作为数据的消费方（使用者）。

1. 使用`React.createContext()`创建 Context，并填入默认值
2. 在最外层使用`<ThemeContext.Provider value={this.state.theme}>`包裹子组件，配合 value 实现最外层组件对 Context 的修改
3. 中间层若不使用 Context 数据则不作任何操作
4. 底层组件使用 Context
   - class 组件
     > class 组件在创建后使用 `ThemedButton.contextType = ThemeContext`指定 contextType 读取当前的 theme context。  
     > 也可以在组件内部使用`static contextType = ThemeContext`读取
     > 在组件内部使用 Context，`const theme = this.context` React 会往上找到最近的 theme Provider，然后使用它的值。
   - 函数组件
     > 函数组件没有实例，没有 this
     > 直接使用`<ThemeContext.Consumer>`进行包裹
     > 在内部使用函数获取 context 的 value

```jsx
import React from 'react';

// 创建 Context 填入默认值（任何一个 js 变量）
const ThemeContext = React.createContext('light');

// 最外层组件
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      theme: 'light'
    };
  }
  render() {
    return (
      <ThemeContext.Provider value={this.state.theme}>
        <Toolbar />
        <hr />
        <button onClick={this.changeTheme}>change theme</button>
      </ThemeContext.Provider>
    );
  }
  changeTheme = () => {
    this.setState({
      theme: this.state.theme === 'light' ? 'dark' : 'light'
    });
  };
}

//  ------------------------------------------------

// 底层组件 - 函数组件
function ThemeLink(props) {
  // const theme = this.context // 会报错。函数式组件没有实例，即没有 this

  // 函数式组件可以使用 Consumer
  return (
    <ThemeContext.Consumer>
      {(value) => <p>link's theme is {value}</p>}
    </ThemeContext.Consumer>
  );
}

// 底层组件 - class 组件
class ThemedButton extends React.Component {
  // 指定 contextType 读取当前的 theme context。
  // static contextType = ThemeContext
  // 也可以用 ThemedButton.contextType = ThemeContext
  render() {
    const theme = this.context; // React 会往上找到最近的 theme Provider，然后使用它的值。
    return (
      <div>
        <p>button's theme is {theme}</p>
      </div>
    );
  }
}
ThemedButton.contextType = ThemeContext; // 指定 contextType 读取当前的 theme context。

//  ------------------------------------------------

// 中间的组件再也不必指明往下传递 theme 了。
function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
      <ThemeLink />
    </div>
  );
}

export default App;
```

# React 异步加载组件

- `import()`
- `React.lazy`
- `React.Suspense`

## 演示

1. 使用`React.lazy(() => import('./ContextDemo'))`引入组件
2. `<React.Suspense fallback={<div>Loading...</div>}>`包裹异步组件
3. fallback 中可以加入等待组件加载时的显示内容

```jsx
import React from 'react';

const ContextDemo = React.lazy(() => import('./ContextDemo'));

class App extends React.Component {
  constructor(props) {
    super(props);
  }
  render() {
    return (
      <div>
        <p>引入一个动态组件</p>
        <hr />
        <React.Suspense fallback={<div>Loading...</div>}>
          <ContextDemo />
        </React.Suspense>
      </div>
    );
  }
}

export default App;
```
