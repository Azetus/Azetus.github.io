---
title: 记一次 Redux Toolkit 的使用
date: 2021-12-17 15:54:10
tags:
  - React
---

# 初识 Redux Toolkit

按照官方文档的描述，Redux Toolkit 是一个官方提供用于 Redux 高效开发，有想法的、功能齐全的工具包。提供了一些列的 API 函数以简化 Redux 的使用。

我这一次的使用场景是使用 React 制作一个简单的 Task App，包含用户登录，创建、编辑待办事项等功能。使用 Redux 对组件的状态与用户的数据进行统一管理。

## 文档链接

Official Site：`https://redux-toolkit.js.org/`

# 在项目中引入 Redux Toolkit

之前项目中已经安装了`redux`与`react-redux`,直接输入`npm install @reduxjs/toolkit`安装 Redux Toolkit。

## 创建 Store

Redux Toolkit 中包含了一个新的 Api `configureStore()` 他包含了原本 redux `createStore()`的功能，同时可以设置一些额外的选项。

### configureStore()

```javascript
const store = configureStore({
  reducer: MyReducer,
  middleware: [thunk, ...MyMiddlewares],
  devTools: true
});
```

将 Reducer 函数作为 reducer 的字段传递进对象中，还可以设置中间件 middleware，在创建 Store 的时候可以将`devTools:`设置为`true`方便测试的时候使用浏览器的 Redux DevTools 进行调试。

### 使用 combineReducers() 合并 reducers

和字面意思一样，`combineReducers()`能把一个由多个不同 reducer 函数作为 value 的 对象合并成为一个总的 reducers 函数。然后可以对这个 reducers 调用 `createStore()`。合并后的 reducers 可以调用各个子 reducer，并把他们的结果合并成一个 state 对象。state 对象的结构由传入的多个 reducer 的 key 决定。

```javascript
const rootReducer = combineReducers({
  reducerA: reducerA,
  reducerB: reducerB
});
```

这里直接快进到最后完成时。项目中有两个名为`listState`与`user`的 slice，分别管理用户输入的待办事项与用户信息、登录状态等。

```javascript
const rootReducer = combineReducers({
  todolist: todolistReducer,
  user: userSlice
});
```

### 关于数据持久化

如果单使用 redux，用户在刷新页面的时候数据就会丢失，这里使用`redux-persist`对数据进行持久化保存，`redux-persist`实现数据持久化依靠的还是 html5 的 localstorage。
我的整个 store 的结构大概是这样子

```
rootReducer
  ├── listState
  |   ├── data
  |   ├── firstTime
  |   └── loading:
  |
  └── user
      ├── loading
      ├── username
      ├── error
      ├── token
      └── uid
```

我只想保存用户的 username, uid 以及表示用户登录状态的 jwt token，因此需要设置嵌套的 persist 只将我想缓存的部分保存在浏览器缓存中，在嵌套的设置中设置两层黑名单来实现这一点，设置如下：

```javascript
import storage from 'redux-persist/lib/storage';
const userPresistConfig = {
  key: 'user',
  storage: storage,
  blacklist: ['loading', 'error']
};
const presistConfig = {
  key: 'root',
  storage: storage,
  blacklist: ['todolist', 'user']
};
```

其中`userPresistConfig`是 userSlice 的 persist 设置，缓存机制 storage 代表数据将缓存在浏览器的 localStorage 中。在 blacklist 中设置 loading 与 error 字段，表示这两个数据不用缓存。而`presistConfig`表示的是 rootReducer 的缓存设置，将 todolist 与 user 一起设置进黑名单中。最终被缓存的数据只有 user 中的 username, token 与 uid。

设置的应用：

```javascript
// 创建嵌套 presist
const rootReducer = combineReducers({
  todolist: todolistReducer,
  user: persistReducer(userPresistConfig, userSlice)
});

const persistedReducer = persistReducer(presistConfig, rootReducer); // 浏览器缓存

const store = configureStore({
  reducer: persistedReducer,
  devTools: true
});

// 在 index ReactDOM.render() 中使用组件
const persistor = persistStore(store);

const rootstore = { store, persistor };
export default rootstore;
```

最后为了设置生效，还要再 index.js 中使用`<PersistGate>`对 APP 进行包裹，其中`loading={}`可以传入一个自定义的组件，在读取缓存时展示

```javascript
// index。js
import { Provider } from 'react-redux';
import rootstore from './redux/store.js';

import { PersistGate } from 'redux-persist/lib/integration/react';
import Loading from './components/loading/Loading';

ReactDOM.render(
  <React.StrictMode>
    <Provider store={rootstore.store}>
      <PersistGate loading={<Loading />} persistor={rootstore.persistor}>
        <App />
      </PersistGate>
    </Provider>
  </React.StrictMode>,
  document.getElementById('root')
);
```

### 自定义中间件

我的项目中有一个名为`todolistMiddleWare`的中间件，用于处理用户输入、编辑待办事项时，向服务器异步发送请求。按照文档，在调用`configureStore()`时将中间件传入。middleware 应接收一个数组，包含使用的中间件。

```javascript
const store = configureStore({
  reducer: persistedReducer,
  middleware: [todolistMiddleWare],
  devTools: false
});
```

但是这里如果只传入自定义中间件的话会立即报错。文档中是这样的解释的：

> If this option is provided, it should contain all the middleware functions you want added to the store. configureStore will automatically pass those to applyMiddleware.
> If not provided, configureStore will call getDefaultMiddleware and use the array of middleware functions it returns.

`configureStore()`会提供默认的中间件，但如果这里手动传入参数的话，redux 只会使用传入的中间件，并覆盖默认配置，因此这里还要从 redux-thunk 引入 thunk 一并传入数组。

```javascript
const store = configureStore({
  reducer: persistedReducer,
  middleware: [thunk, todolistMiddleWare],
  devTools: true
});
```

或者使用 toolkit 中内置的`getDefaultMiddleware()`函数直接获取默认中间件，函数返回一个数组，包含了所有默认中间件，使用数组的`.concat()`或者 ES6 解构数组的方式，将自定义中间传入并返回一个新数组。

```javascript
const store = configureStore({
  reducer: persistedReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(todolistMiddleWare),
  devTools: true
});
```

**tips**：使用中可能会出现 Warning，'A non-serializable value was detected in an action'。此时需要手动设置 thunk 的 extraArgument 并将 serializableCheck 设置为 `false`。

```javascript
const store = configureStore({
  reducer: persistedReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      thunk: {
        extraArgument: todolistMiddleWare
      },
      serializableCheck: false
    }).concat(todolistMiddleWare),
  devTools: true
});
```

### 最终整个 store 的代码如下：

```javascript
import { configureStore } from '@reduxjs/toolkit';
import { combineReducers } from 'redux';
import todolistReducer from './listState/slice';
import userSlice from './user/slice';

// 中间件，处理todolist各种操作中的异步请求
import { todolistMiddleWare } from './middleware/todolistMiddleWare';

// 浏览器缓存
import storage from 'redux-persist/lib/storage';
import { persistStore, persistReducer } from 'redux-persist';

// 创建嵌套 presist 设置
// 本地只保存 uid, username 与 token 防止请求失败用户再次刷新时，loading 依旧为 true
const userPresistConfig = {
  key: 'user',
  storage: storage,
  blacklist: ['loading', 'error']
};
const presistConfig = {
  key: 'root',
  storage: storage,
  blacklist: ['todolist', 'user']
};

// 创建嵌套 presist
const rootReducer = combineReducers({
  todolist: todolistReducer,
  user: persistReducer(userPresistConfig, userSlice)
});

const persistedReducer = persistReducer(presistConfig, rootReducer); // 浏览器缓存

const store = configureStore({
  reducer: persistedReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      thunk: {
        extraArgument: todolistMiddleWare
      },
      serializableCheck: false
    }).concat(todolistMiddleWare),

  devTools: false
});

// 在 index ReactDOM.render() 中使用组件
const persistor = persistStore(store);

const rootstore = { store, persistor };
export default rootstore;
```

将整个 store export 出去之后不要忘了在 index.js 中使用

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import { Provider } from 'react-redux';
import rootstore from './redux/store.js';

import { PersistGate } from 'redux-persist/lib/integration/react';
import Loading from './components/loading/Loading';

ReactDOM.render(
  <React.StrictMode>
    <Provider store={rootstore.store}>
      <PersistGate loading={<Loading />} persistor={rootstore.persistor}>
        <App />
      </PersistGate>
    </Provider>
  </React.StrictMode>,
  document.getElementById('root')
);
```

## 创建 Slice

slice 是 redux toolkit 定义的一个概念，就是一系列相关数据的状态管理，包含 action，state，reducer 三个部分，在页面较简单时，可以认为每个页面的状态就是一个 slice；在页面较复杂时，可以把一些相关状态放在一个 slice 中。可以实现模块化的封装。所有的相关操作都独立在一个文件中完成。~~不然的话 action，reducer，state 散落在各个文件中~~

### 使用 createSlice()

调用`createSlice()`，函数接受初始化 state 和对象化映射后的 reducer，可以将 store 以切片 slice 的方式分割成为不同的部分，每个部分都会独立、而且自动生成相对应的 action 与 state 对象。

官方的示例代码如下：

```javascript
import { createSlice } from '@reduxjs/toolkit';

const initialState = { value: 0 };

const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    increment(state) {
      state.value++;
    },
    decrement(state) {
      state.value--;
    },
    incrementByAmount(state, action) {
      state.value += action.payload;
    }
  }
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;
export default counterSlice.reducer;
```

在示例中，reducers 中的函数直接修改了 state 的值，而不是重建新的 state，似乎违反了 immutable 的原则。

但 redux/toolkit 内置了 Immer 功能，这里的 state 是一个 Proxy，允许我们把 state 从 immutabe 转化为 mutable 从而可以直接更改 State，对他的直接更改，会在底层转化为一个新的 immutable state。当然也可以不适用 immer 功能， 直接在函数中 return 一个新建的 state。

传入的对象中需要包含 3 个参数：

1. name：slice 的命名空间，生成的动作类型常量将使用它作为前缀('name/actionName')
2. initialState：state 的初始值
3. reducers：定义的 action，内置了 immer 插件。

最终创建的 Slice.js 如下（省略了部分代码）：

```javascript
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';
import axios from 'axios';

const todolistSlice = createSlice({
  name: 'todolist',
  initialState: {
    data: [],
    firstTime: true,
    loading: false
  },
  reducers: {
    ADD_TODO: (state, action) => {
      const newTodo = action.payload;
      state.data.push(newTodo);
    },

    REMOVE_TODO: (state, action) => {
      const removeTodoArr = state.data.filter(
        (todo) => todo.id !== action.payload.delTodoId
      );
      const newState = Object.assign(state, { data: removeTodoArr });
      return newState;
    }
    // ......
    // ......
  }
});

export const {
  ADD_TODO,
  REMOVE_TODO
  // ......
} = todolistSlice.actions;
export default todolistSlice.reducer;
```

### 使用 createAsyncThunk() 与 extraReducers 创建异步 action

官方文档中的示例如下：

```javascript
const fetchUserById = createAsyncThunk(
  'users/fetchByIdStatus',
  async (userId, thunkAPI) => {
    const response = await userAPI.fetchById(userId);
    return response.data;
  }
);
```

`createAsyncThunk()` 方法可以创建一个异步的 action，他接收三个参数，第一个参数是一个字符串定义 action type，第二个参数为返回 Promise 类型的回调函数，第三个参数为 options。回调函数接收的第二个参数为`thunkAPI`，其中包含了例如`getState()`获取当前 state 的方法。

action 执行时，会产生对应的三个状态：pending(进行中)、fulfilled(成功)、rejected(失败)。需要创建对应的 extraReducers 对其结果进行监听。在 extraReducers 中添加`[action类型.状态.type](state, action) {}`进行接收。

```javascript
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';
import axios from 'axios';

export const getInitialList = createAsyncThunk(
  'todolist/getInitialList',
  async (paramaters, thunkAPI) => {
    const api = 'http://localhost:3030/api/list/getTodolist';
    const uid = thunkAPI.getState().user.uid;
    const token = thunkAPI.getState().user.token;
    const { data } = await axios.get(api, {
      headers: {
        uid: uid,
        Authorization: `Bearer ${token}`
      }
    });
    return data;
  }
);

const todolistSlice = createSlice({
  name: 'todolist',
  initialState: {...},
  reducers: {...},
  extraReducers: {
    [getInitialList.pending.type](state) {
      state.loading = true;
    },
    [getInitialList.fulfilled.type](state, action) {
      if (action.payload.errno === 0) {
        console.log('getInitialList fulfilled');
        state.loading = false;
        state.firstTime = false;
        state.data = action.payload.data;
      } else {
        state.loading = false;
      }
    },
    [getInitialList.rejected.type](state, action) {
      state.loading = false;
      console.log('getInitialList fail');
    }
  }
});

export const {
  // ...
} = todolistSlice.actions;
export default todolistSlice.reducer;
```

### 在 React 组件中使用

在组件中 dispatch 使用 redux-toolkit 创建的 action 与原本没有什么区别。无论是同步 action 还是异步 action。

首先在函数组件中引入 useDispatch 与想要使用的 action。在需要的地方 dispatch action。

```javascript
import { useDispatch } from 'react-redux';
import { REMOVE_TODO, getInitialList } from '../../redux/listState/slice';

function Component() {
  const dispatch = useDispatch();

  dispatch(REMOVE_TODO('传递的 payload'));
  dispatch(getInitialList('传递的 paramaters'));

  return <div>
    <!-- ... -->
  </div>;
}
export default Component;
```

### 创建自定义中间件

在使用 configureStore 创建 store 的时候传入了一个自定义中间件 todolistMiddleWare。中间件的创建方式如下，定义一个`(store) => (next) => (action) => {}`函数并 export 出去。在函数中可以使用`store.getState()`来获取当前 store 的 state。函数内部可以使用`switch`语法针对不同 action 进行不同的操作，在最后需要调用`next(action)`执行下一个中间件或继续执行 slice 中的 reducer。

```javascript
export const todolistMiddleWare = (store) => (next) => (action) => {
  switch (action.type) {
    case 'todolist/ADD_TODO':
      // ...
      break;
    case 'todolist/REMOVE_TODO':
      break;
    // .....
    default:
      break;
  }
  next(action);
};
```
