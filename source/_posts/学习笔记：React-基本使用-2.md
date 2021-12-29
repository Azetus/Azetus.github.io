---
title: 学习笔记：React 基本使用(2)
date: 2021-08-31 14:56:10
tags:
  - React 学习笔记
categories:
  - React
---

# 知识点

- 组件和 props
- setState
- 生命周期

# 组件使用

- props 传递数据
- props 传递函数
- props 类型检查

## 父组件 TodoListDemo

**状态（数据）提升**

**在设计时，要将状态（数据）放在高一级的组件中进行统一管理。**

如下例子中，最外层父组件为`TodoListDemo`，以及子组件`Input`和`List`，而这三个组件共享一个数据`list`。因此需要将数据`list`放在父组件`TodoListDemo`中进行管理。

多个组件共享的数据，应该提升至级别最高的父组件中，由父组件进行管理，由父组件向子组件下发数据。由子组件进行渲染。

> 例如例子中的`List`子组件，只负责对接收到的数据进行渲染。

对于用户输入的情况，父组件只向子组件下发一个函数（事件），由**子组件执行**函数（事件），**父组件拼接**数据，并再将新的数据下发给负责渲染的子组件。

> 如例子中的`Input`组件，自身只负责接收父组件传递的函数`onSubmitTitle`并执行。而数据的拼接，则由父组件`TodoListDemo`进行。

```jsx
import React from 'react';

class TodoListDemo extends React.Component {
  constructor(props) {
    super(props);
    // 状态（数据）提升
    this.state = {
      list: [
        {
          id: 'id-1',
          title: '标题1'
        },
        {
          id: 'id-2',
          title: '标题2'
        },
        {
          id: 'id-3',
          title: '标题3'
        }
      ]
    };
  }
  render() {
    return (
      <div>
        <Input submitTitle={this.onSubmitTitle} />
        <List list={this.state.list} />
      </div>
    );
  }
  onSubmitTitle = (title) => {
    this.setState({
      list: this.state.list.concat({
        id: `id-${Date.now()}`,
        title
      })
    });
  };
}
```

## 子组件 List

直接使用 `const { list } = this.props;`接收父组件传递的 props

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

## props 类型检查

```jsx
import PropTypes from 'prop-types';
// props 类型检查
List.propTypes = {
  list: PropTypes.arrayOf(PropTypes.object).isRequired
};
// props 类型检查
Input.propTypes = {
  submitTitle: PropTypes.func.isRequired
};
```

## 传递函数

**父组件**

> 父组件传递的属性也可以是函数

```html
<!-- 作为属性传递 -->
<input submitTitle="{this.onSubmitTitle}" />
```

**子组件**

`const { submitTitle } = this.props;`

```jsx
class Input extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      title: ''
    };
  }
  render() {
    return (
      <div>
        <input value={this.state.title} onChange={this.onTitleChange} />
        <button onClick={this.onSubmit}>提交</button>
      </div>
    );
  }
  onTitleChange = (e) => {
    this.setState({
      title: e.target.value
    });
  };
  onSubmit = () => {
    const { submitTitle } = this.props;
    submitTitle(this.state.title); // 'abc'

    this.setState({
      title: ''
    });
  };
}
```

# setState

- **不可变值**
- 可能是异步更新
- 可能会被合并

## state 要在构造函数中定义

```jsx
import React from 'react';

class StateDemo extends React.Component {
  constructor(props) {
    super(props);

    // 第一，state 要在构造函数中定义
    this.state = {
      count: 0
    };
  }
  render() {
    return (
      <div>
        <p>{this.state.count}</p>
        <button onClick={this.increase}>累加</button>
      </div>
    );
  }
}
```

## 不可变值

不可变值：不能对 state 的值提前进行修改，同时在`setState`时不能影响之前 state 的值。

> 针对数组与对象时，选用那些不会改变现有数组、对象，而是返回新副本的方法。

### 使用 `setState()` 修改

```jsx
increase = () => {
  // 第二，不要直接修改 state ，使用不可变值 ----------------------------
  // this.state.count++ // 错误
  this.setState({
    count: this.state.count + 1 // SCU
  });
  // 操作数组、对象的的常用形式
};
```

### 数组

> 不能直接对 `this.state.list` 进行 `push` `pop` `splice` 等，这样违反不可变值。  
> 选用那些会返回新数组的 API，或者先生成 state 的副本，并对副本进行操作。

```jsx
increase = () => {
  // 不可变值（函数式编程，纯函数） - 数组
  const list5Copy = this.state.list5.slice();
  list5Copy.splice(2, 0, 'a'); // 中间插入/删除
  this.setState({
    list1: this.state.list1.concat(100), // 追加
    list2: [...this.state.list2, 100], // 追加
    list3: this.state.list3.slice(0, 3), // 截取
    list4: this.state.list4.filter((item) => item > 100), // 筛选
    list5: list5Copy // 其他操作
  });
};
```

### 对象

> 不能直接对 this.state.obj 进行属性设置，这样违反不可变值。  
> 通过`Object.assgin` 或是**解构**，生成新的对象，避免对旧对象产生影响。

```jsx
this.setState({
  obj1: Object.assign({}, this.state.obj1, { a: 100 }),
  obj2: { ...this.state.obj2, a: 100 }
});
```

## `setState` 可能是异步更新

setState 有可能是异步，也有可能是同步。

- 直接使用`this.setState` —— **异步**
- 在`setTimeout`和自定义 DOM 事件 —— **同步**

### 直接使用`this.setState`

直接使用`this.setState`，并在`setState`尝试接获取 state 的结果时，获取的是当前值。而`setState`修改后的渲染过程是存在异步的（参考 Vue $nextTick - DOM）。可以在`setState`的第二个参数中使用回调函数获取最新的 state 值。

```jsx
increase = () => {
  // 第三，setState 可能是异步更新（有可能是同步更新）
  this.setState(
    {
      count: this.state.count + 1
    },
    () => {
      // 联想 Vue $nextTick - DOM
      console.log('count by callback', this.state.count); // 回调函数中可以拿到最新的 state
    }
  );
  console.log('count', this.state.count); // 异步的，拿不到最新值
};
```

### 在 setTimeout 中使用 `setState`

setTimeout 中 setState 是同步的，不用回调函数就可以拿到最新的值

```jsx
increase = () => {
  // setTimeout 中 setState 是同步的
  setTimeout(() => {
    this.setState({
      count: this.state.count + 1
    });
    console.log('count in setTimeout', this.state.count);
  }, 0);
};
```

### 自己定义的 DOM 事件，setState 是同步的。

> componentDidMount

```jsx
bodyClickHandler = () => {
  this.setState({
    count: this.state.count + 1
  });
  console.log('count in body event', this.state.count);
};
componentDidMount() {
  // 自己定义的 DOM 事件，setState 是同步的
  document.body.addEventListener('click', this.bodyClickHandler);
}
componentWillUnmount() {
  // 及时销毁自定义 DOM 事件
  document.body.removeEventListener('click', this.bodyClickHandler);
  // clearTimeout
}
```

## `setState` 可能会被合并

state 异步更新的话，更新前会被合并

### 传入对象，会被合并

> 在异步更新时，类似于`Object.assign({ count: 1 }, { count: 1 })`，更新会被合并，只执行 1 次。

```jsx
increase = () => {
  // 传入对象，会被合并（类似 Object.assign ）。执行结果只一次 +1
  this.setState({
    count: this.state.count + 1
  });
  this.setState({
    count: this.state.count + 1
  });
  this.setState({
    count: this.state.count + 1
  });
};
```

### 传入函数，不会被合并

> 函数可以接收两个参数，`prevState`即执行操作前旧的 state，以及`props`

```jsx
increase = () => {
  // 传入函数，不会被合并。执行结果是 +3
  this.setState((prevState, props) => {
    return {
      count: prevState.count + 1
    };
  });
  this.setState((prevState, props) => {
    return {
      count: prevState.count + 1
    };
  });
  this.setState((prevState, props) => {
    return {
      count: prevState.count + 1
    };
  });
};
```

# React 组件生命周期

- 单组件生命周期
- 父子组件生命周期，和 Vue 的一样

## 常用生命周期

- 挂载时 `componentDidMount`
  > ajax 请求等操作  
  > Vue - mounted
- 更新时 `componentDidUpdate`
  > `setState()` 触发 render  
  > Vue - updated
- 卸载时 `componentWillUnmount`
  > 销毁自定义事件，清空定时器等  
  > Vue - beforeUnmount

![react_life_cycle_01](react_life_cycle.png)

## 不常用生命周期

- `shouldComponentUpdate` 更新时是否需要继续渲染
  > 性能优化

根据 `shouldComponentUpdate()` 的返回值，判断 React 组件的输出是否受当前 state 或 props 更改的影响，当 props 或 state 发生变化时，`shouldComponentUpdate()` 会在渲染执行之前被调用。返回值默认为 `true`。首次渲染或使用 `forceUpdate()` 时不会调用该方法。

![react_life_cycle_02](react_life_cycle_02.png)
