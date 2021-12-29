---
title: 学习笔记：React-Redux
date: 2021-09-13 15:14:33
tags:
  - React 学习笔记
categories:
  - React
---

# Redux 使用

- 单向数据流
- react-redux
- 异步 action
- 中间件

# Redux 单向数据流

## 基本概念

- store state
- action
- reducer

## 概述

- `dispatch(action)`
- reducer -> newState
- subscribe 触发通知

## 基本使用

- 创建一个 reducer `(state, action) => state`
- 遵循不可变值的原则，state 变化时要返回一个全新的对象
- 使用 `createStore()` ，传入 reducer 作为参数创建 store
- 订阅更新
- 修改时通过 `state.dispatch()` dispatch 一个 action
- reducer 接收到 action 并判断执行何种操作

### 演示

> 引用自 Redux 文档

```jsx
import { createStore } from 'redux';

/**
 * 这是一个 reducer，形式为 (state, action) => state 的纯函数。
 * 描述了 action 如何把 state 转变成下一个 state。
 *
 * state 的形式取决于你，可以是基本类型、数组、对象、
 * 甚至是 Immutable.js 生成的数据结构。惟一的要点是
 * 当 state 变化时需要返回全新的对象，而不是修改传入的参数。
 *
 * 下面例子使用 `switch` 语句和字符串来做判断，但你可以写帮助类(helper)
 * 根据不同的约定（如方法映射）来判断，只要适用你的项目即可。
 */
function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1;
    case 'DECREMENT':
      return state - 1;
    default:
      return state;
  }
}

// 创建 Redux store 来存放应用的状态。
// API 是 { subscribe, dispatch, getState }。
let store = createStore(counter);

// 可以手动订阅更新，也可以事件绑定到视图层。
store.subscribe(() => console.log(store.getState()));

// 改变内部 state 惟一方法是 dispatch 一个 action。
// action 可以被序列化，用日记记录和储存下来，后期还可以以回放的方式执行
store.dispatch({ type: 'INCREMENT' });
// 1
store.dispatch({ type: 'INCREMENT' });
// 2
store.dispatch({ type: 'DECREMENT' });
// 1
```

# react-redux

- `<Provider>`
- connect
- mapStateToProps
- mapDispatchToProps

# redux 处理异步

- redux-thunk
- redux-promise
- redux-saga

## 同步 action

```jsx
// 同步 action
export const addTodo = (text) => {
  // 返回 action 对象
  return {
    type: 'ADD_TODO',
    id: nextTodoId++,
    text
  };
};
```

## 异步 action

```jsx
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from './reducer/index';

// 创建 store 时，作为中间件引入 redux-thunk
const store = createStore(rootReducer, applyMiddleware(thunk));
```

```jsx
// 异步 action
export const addTodoAsync = (text) => {
  // 返回函数，其中有 dispatch 参数
  return (dispatch) => {
    // ajax 异步获取数据
    fetch(url).then((res) => {
      // 执行异步 action
      dispatch(addTodo(res.text));
    });
  };
};
```

# redux 中间件

![middleware](middleware.png)

```jsx
import { applyMiddleware, createStore } from 'redux';
import createLogger from 'redux-logger';
import thunk from 'redux-thunk';
const logger = createLogger();

// 会按顺序执行
const store = createStore(reducer, applyMiddleware(thunk, logger));
```

## redux 中间件 - logger 实现

```jsx
// 修改 dispatch， 增加logger
let next = store.dispatch;
store.dispatch = function dispatchAndLog(action) {
  console.log('dispatching', action);
  next(action);
  console.log('next state', store.getState());
};
```

# redux 数据流图

![redux_pic](redux_pic.png)

# 总结

- 基本概念
- 单向数据流
- react-redux
- 异步 action
- 中间件
