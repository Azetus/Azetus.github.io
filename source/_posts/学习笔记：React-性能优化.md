---
title: 学习笔记：React 性能优化
date: 2021-09-13 15:07:43
tags:
  - React
categories:
  - React 学习笔记
---

# 知识点

- `shouldComponentUpdate` 简称 SCU
- PureComponent 和 React.memo
- 不可变值 `immutable.js`
- 组件的公共逻辑抽离

# SCU 基本用法

根据 `shouldComponentUpdate()` 的返回值，判断 React 组件的输出是否受当前 state 或 props 更改的影响。默认返回 `true`，即 state 每次发生变化组件都会重新渲染。

如果要手动编写此函数，可以将 `this.props` 与 `nextProps` 以及 `this.state` `与nextState` 进行比较，并返回 `false` 以告知 React 可以跳过更新。

```jsx
shouldComponentUpdate(nextProps, nextState) {
  if (nextState.count !== this.state.count) {
    return true // 可以渲染（默认值）
  }
  return false // 不重复渲染
}
```

**React 的默认行为：如果父组件更新，子组件也会无条件更新，无论子组件的数据是否有变化。**

## 示例

假设有这样一个`<TodoList>`组件，包含三个子组件`<Input>` `<List>` `<Footer>`。组件的 state 在父组件`<TodoList>`中，其中`<Input>`内有一个输入框，接收用户输入，`<List>`负责对列表数据进行渲染。`<Footer>`只接收一个不变的 props ，props 内容为一个固定的字符串，其内部生命周期`componentDidUpdate()`在子组件`<Footer>`更新时输出打印一个内容`'footer did update'`。

在页面为 TodoList 添加新内容时，可以发现，每次添加新内容时，浏览器控制台都会打印一次`'footer did update'`。说明即使子组件`<Footer>`的内容理论上永远也不会有任何变化，但依旧触发了更新与重复渲染。

> 对于 React 来说，其默认行为模式是，父组件更新时，子组件也会随之无条件更新

```jsx
render() {
  return (
    <div>
      <Input submitTitle={this.onSubmitTitle} />
      <List list={this.state.list} />
      <Footer text={this.state.footerInfo} length={this.state.list.length} />
    </div>
  );
}
```

```jsx
class Footer extends React.Component {
  constructor(props) {
    super(props);
  }
  render() {
    return (
      <p>
        {this.props.text}
        {this.props.length}
      </p>
    );
  }
  componentDidUpdate() {
    console.log('footer did update');
  }
  shouldComponentUpdate(nextProps, nextState) {
    if (
      nextProps.text !== this.props.text ||
      nextProps.length !== this.props.length
    ) {
      return true; // 可以渲染
    }
    return false; // 不重复渲染
  }
}
```

- React 默认行为：父组件有更新，子组件则无条件也更新，无论子组件的数据是否有变化
- 因此性能优化对于 React 更加重要（这种子组件更新机制与 Vue 有所不同）
- SCU 一定要每次都用吗？—— 需要的时候才优化

## SCU 与不可变值

`shouldComponentUpdate()`**必须配合不可变值使用**。如果使用错误的方法修改 state 的值，在 SCU 内部比较时，`nextProps`与`this.props`会相等，导致 SCU 一直返回`false`。因此不能对 state 的值提前进行修改，同时在`setState`时不能影响之前 state 的值。

```jsx
shouldComponentUpdate(nextProps, nextState) {
  // 使用 lodash 的 _.isEqual 做对象或者数组的深度比较（一次性递归到底）
  if (_.isEqual(nextProps.list, this.props.list)) {
    // 相等，则不重复渲染
    return false;
  }
  return true; // 不相等，则渲染
}
```

> 不推荐在 SCU 中队数据做深层比较

### 错误的写法

> 错误：在调用`setState`前就对 state 的值进行了修改，导致`nextProps`与`this.props`相等，SCU 无法执行正确的判断。

```jsx
//  错误的写法
this.state.list.push({
  id: `id-${Date.now()}`,
  title
});
this.setState({
  list: this.state.list
});
```

### 正确的写法

> 正确：在`setState`内对 state 进行修改，使用`concat` `slice`等不会修改原始数组，并且会返回新数组的方法。

```jsx
this.setState({
  list: this.state.list.concat({
    id: `id-${Date.now()}`,
    title
  })
});
```

## 小结

- SCU 默认返回`true`，即 React 默认重新渲染所有子组件
- 必须配合“不可变值”一起使用
- 有性能问题时再考虑使用 SCU

# PureComponent 与 memo

- PureComponent - 在 SCU 中实现了浅比较
  > 依旧需要配合不可变值
- class 组件使用 PureComponent
- 函数组件使用 memo
- 浅比较已适用于大部分情况（尽量不要做深比较）

## PureComponent

使用 `class 'ComponentName' extends React.PureComponent {}` 创建纯组件

> 相当于组件内部自带了 `shouldComponentUpdate() {/*浅比较*/}`

```jsx
class List extends React.PureComponent {
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
  // 相当于组件内部自带了 shouldComponentUpdate() {/*浅比较*/}
}
```

## memo

`React.memo`是高阶组件，适用于函数组件，若果组件在相同 props 的情况下返回相同的渲染结果，可以使用`React.memo`对组件进行包裹，以避免组件的重复渲染。

`React.memo`只检查 props 的变更，如果函数组件被 `React.memo` 包裹，且其实现中拥有 ` useState``，useReducer ` 或 `useContext` 的 Hook，当 state 或 context 发生变化时，它仍会重新渲染。

```jsx
const MyComponent = React.memo(function MyComponent(props) {
  /* 使用 props 渲染 */
});
```

> 默认情况下`React.memo`只会做浅层比较，可以使用第二个参数来手动控制对比过程。

```jsx
function MyComponent(props) {
  /* 使用 props 渲染 */
}
function areEqual(prevProps, nextProps) {
  /*
    如果将 nextProps 传入 render 方法的返回结果
    与将 prevProps 传入 render 方法返回的结果一致，则返回 true
    否则返回 false
  */
}
export default React.memo(MyComponent, areEqual);
```

# immutable.js

- 彻底拥抱“不可变值”
- 基于共享数据（不是深拷贝），速度好
- 有一定学习和迁移成本，按需使用

```javascript
const map1 = immutable.Map({ a: 1, b: 2, c: 3 });
const map2 = map1.set('b', 50);
map1.get('b'); // 2
map2.get('b'); // 50
```

# 性能优化 - 总结

- 面试重点，且涉及 React 设计理念
- SUC PureComponent, memo, immutable.js
- 按需使用
- state 层级设计

# React 组件公共逻辑的抽离

- mixin，已被 React 弃用
- 高阶组件 HOC (Higher Order Component)
- Render Props

## 高阶组件

> HOC 并不是一个功能，而是一种模式（类似工厂）。

高阶组件是参数为组件，返回值为新组件的函数。定义一个更高级的组件包裹子组件，实现公共逻辑并传递给子组件。

```jsx
const HOCFactory = (Component) => {
  class HOC extends React.Component {
    // 再次定义多个组件的公共逻辑
    render() {
      return <Component {...this.props} />; // 返回拼装的结果
    }
  }
  return HOC;
};
const EnhancedComponent1 = HOCFactory(WrappedComponent1);
const EnhancedComponent2 = HOCFactory(WrappedComponent2);
```

### 高阶组件的使用方法

> 鼠标滑动时获取鼠标位置

在`withMouse`中实现获取鼠标位置逻辑，并传入到 APP 组件中，APP 从 props 中获取 mouse 的值，并在页面中渲染显示。用`withMouse()`包裹 APP 组件并返回。

1. 定义一个 HOC 函数，参数接收一个 Component
2. 在 HOC 函数内部定义组件，并在此组件内放置 state，编写公共逻辑
3. HOC 函数内组件的 render 中对传入的参数组件 Component 进行包裹，在包裹层添加公共事件
4. 向包裹层内部的 Component 透传 props ，用属性的形式传递需要的 state
5. HOC 函数最终将这个组件返回
6. 在需要使用公共逻辑时，用 HOC 函数对组件进行包裹，使组件可以使用公共逻辑

```jsx
import React from 'react';

// 高阶组件
const withMouse = (Component) => {
  // 定义组件 withMouseComponent
  class withMouseComponent extends React.Component {
    // state 放在高阶组件中
    constructor(props) {
      super(props);
      this.state = { x: 0, y: 0 };
    }

    // 公共逻辑
    handleMouseMove = (event) => {
      this.setState({
        x: event.clientX,
        y: event.clientY
      });
    };

    render() {
      return (
        <div style={{ height: '500px' }} onMouseMove={this.handleMouseMove}>
          {/* 1. 透传所有 props 2. 增加 mouse 属性 */}
          <Component {...this.props} mouse={this.state} />
        </div>
      );
    }
  }
  return withMouseComponent;
};

const App = (props) => {
  const a = props.a;
  const { x, y } = props.mouse; // 接收 mouse 属性
  return (
    <div style={{ height: '500px' }}>
      <h1>
        The mouse position is ({x}, {y})
      </h1>
      <p>{a}</p>
    </div>
  );
};

export default withMouse(App); // 返回高阶函数
```

### redux connect 是高阶组件

```jsx
import { connect } from 'react-redux';

// connect 是高阶组件
const VisibleTodoList = connect(mapStateToProps, mapDispatchToProps)(TodoList);

export default VisibleTodoList;
```

## Render Props

Render Props 的核心思想：通过一个函数将 class 组件的 state 作为 props 传递给纯函数组件

```jsx
class Factory extends React.Component {
  constructor(){
    this.state = {
      /* state 即多个组件的公共逻辑的数据 */
    }
  }

  /* 公共逻辑 */
  /* 修改 state */

  render() {
    return <div>{this.props.render(this.state)}</div>
  },
}
```

```jsx
const APP = () => {
  <Factory render={
    /* render 是一个函数组件 */
    (props) => <p>{props.a} {props.b}</p>
  }>
}
```

### 演示

```jsx
import React from 'react';
import PropTypes from 'prop-types';

class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove = (event) => {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  };

  render() {
    return (
      <div style={{ height: '500px' }} onMouseMove={this.handleMouseMove}>
        {/* 将当前 state 作为 props ，传递给 render （render 是一个函数组件） */}
        {this.props.render(this.state)}
      </div>
    );
  }
}
Mouse.propTypes = {
  render: PropTypes.func.isRequired // 必须接收一个 render 属性，而且是函数
};

const App = (props) => (
  <div style={{ height: '500px' }}>
    <p>{props.a}</p>
    <Mouse
      render={
        /* render 是一个函数组件 */
        ({ x, y }) => (
          <h1>
            The mouse position is ({x}, {y})
          </h1>
        )
      }
    />
  </div>
);

/**
 * 即，定义了 Mouse 组件，只有获取 x y 的能力。(定义公共能力)
 * 至于 Mouse 组件如何渲染，App 说了算，通过 render prop 的方式告诉 Mouse 。
 */

export default App;
```

## 小结

- HOC：模式简单，但会增加组件层级
- Render Props：代码简洁，学习成本较高
- 按需使用
