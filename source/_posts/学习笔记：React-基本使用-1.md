---
title: 学习笔记：React 基本使用(1)
date: 2021-08-31 14:51:33
tags:
  - React 学习笔记
categories:
  - React
---

# 知识点

- JSX 基本使用
- 条件判断
- 渲染列表
- 事件
- 表单

# JSX 基本使用

- 变量、表达式
- class style
- 子元素和组件

```javascript
class JSXBaseDemo extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      name: 'Aliza',
      age: 25,
      flag: true
    };
  }
  render() {
    /* ......... */
    return; /* ......... */
  }
}
```

## 获取变量

> React 中获取变量使用单花括号 {}

```jsx
// 获取变量
const pElem = <p>{this.state.name}</p>;
return pElem;
```

## 使用表达式

```jsx
const exprElem = <p>{this.state.flag ? 'yes' : 'no'}</p>;
return exprElem;
```

## 子元素

```jsx
// 子元素
const imgElem = (
  <div>
    <p>我的头像</p>
    <img src="xxxx.png" />
    <img src={this.state.imgUrl} />
  </div>
);
return imgElem;
```

## class

> 关键词使用 `className`

```jsx
// class
const classElem = <p className="title">设置 css class</p>;
return classElem;
```

## style

> 内联写法，注意 {{ 和 }}  
> 使用单花括号 {} 包裹一个 js 对象

```jsx
// style
const styleData = { fontSize: '30px', color: 'blue' };
const styleElem = <p style={styleData}>设置 style</p>;
// 内联写法，注意 {{ 和 }}
// 使用单花括号 {} 包裹一个 js 对象
const styleElem = <p style={{ fontSize: '30px', color: 'blue' }}>设置 style</p>;
return styleElem;
```

## 原生 html

```jsx
// 原生 html
const rawHtml = '<span>富文本内容<i>斜体</i><b>加粗</b></span>';
const rawHtmlData = {
  __html: rawHtml // 注意，必须是这种格式
};
return rawHtml;

// --------------------------------------------------------

const rawHtmlElem = (
  <div>
    <p dangerouslySetInnerHTML={rawHtmlData}></p>
    <p>{rawHtml}</p>
  </div>
);
// return rawHtmlElem;
```

## 加载组件

```jsx
// 加载组件
const componentElem = (
  <div>
    <p>JSX 中加载一个组件</p>
    <hr />
    <List />
  </div>
);
return componentElem;
```

# 条件判断

- `if` `else`
- 三元表达式
- 逻辑运算符 `&&` `||`

```jsx
class ConditionDemo extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      theme: 'black'
    };
  }
  render() {
    const blackBtn = <button className="btn-black">black btn</button>;
    const whiteBtn = <button className="btn-white">white btn</button>;

    /* ....... */

    return; /* ... */
  }
}
```

```jsx
// if else
if (this.state.theme === 'black') {
  return blackBtn;
} else {
  return whiteBtn;
}
```

```jsx
// 三元运算符
return <div>{this.state.theme === 'black' ? blackBtn : whiteBtn}</div>;
```

```jsx
// &&
return <div>{this.state.theme === 'black' && blackBtn}</div>;

// true && expression 总是会返回 expression, 而 false && expression 总是会返回 false。
```

> 如果条件是 true，&& 右侧的元素就会被渲染，如果是 false，React 会忽略并跳过它

# 渲染列表

- **map**
  > 数组的 map 函数
- **key**
  > 可以参考 Vue 中的 key

```jsx
class ListDemo extends React.Component {
  constructor(props) {
    super(props);
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
      <ul>
        {
          /* 类似 vue 中的 v-for */
          this.state.list.map((item, index) => {
            // 这里的 key 和 Vue 的 key 类似，是必填项，而且不能是 index 或 random
            return (
              <li key={item.id}>
                index {index}; id {item.id}; title {item.title}
              </li>
            );
          })
        }
      </ul>
    );
  }
}
```

# 事件

- bind this
- 关于 event 参数
- 传递自定义参数

## this - 使用 bind

> 点击事件的 bind this，尽量写在 constructor 中，初始化时只用执行一次既可以完成 bind 操作

```jsx
class EventDemo extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      name: 'zhangsan',
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

    // 修改方法的 this 指向
    this.clickHandler1 = this.clickHandler1.bind(this);
  }
  render() {
    // this - 使用 bind
    return <p onClick={this.clickHandler1}>{this.state.name}</p>;
  }
  clickHandler1() {
    // this 默认是 undefined
    this.setState({
      name: 'lisi'
    });
  }
}
```

## this - 使用静态方法

> 不使用 bind。静态方法，this 指向当前实例

```jsx
render() {
  // this - 使用静态方法
  return <p onClick={this.clickHandler2}>
    clickHandler2 {this.state.name}
  </p>
}
// 静态方法，this 指向当前实例
clickHandler2 = () => {
  this.setState({
    name: 'lisi'
  });
};
```

## event

1. event 是 SyntheticEvent ，模拟出来 DOM 事件所有能力
2. event.nativeEvent 是原生事件对象
3. 所有的事件，都被挂载到 document 上，作为附加的处理器，即事件委托 （React 16 中）
   > <font color=#FF0000>注意：在 React 17 中依旧会使用事件委托机制，但挂载的位置变更为了 react 树的根 DOM 容器中，指向 Root Element</font>
4. 和 DOM 事件不一样，和 Vue 事件也不一样
   > Vue 中的 event 是原生的，事件会被挂载到当前元素  
   > event 其实是 React 封装的。可以看 `__proto__.constructor` 是 SyntheticEvent 组合事件

```jsx
render() {
  // event
  return <a href="https://imooc.com/" onClick={this.clickHandler3}>
    click me
  </a>
}
clickHandler3 = (event) => {
  event.preventDefault(); // 阻止默认行为
  event.stopPropagation(); // 阻止冒泡
  console.log('target', event.target); // 指向当前元素，即当前元素触发
  console.log('current target', event.currentTarget); // 指向当前元素（假象）

  // 注意: event 其实是 React 封装的。可以看 __proto__.constructor 是 SyntheticEvent 组合事件
  console.log('event', event); // 不是原生的 Event ，原生的事件是 MouseEvent
  console.log('event.__proto__.constructor', event.__proto__.constructor);

  // 原生: event 如下。其 __proto__.constructor 是 MouseEvent
  console.log('nativeEvent', event.nativeEvent);
  console.log('nativeEvent target', event.nativeEvent.target); // 指向当前元素 render 中的 <a> 标签，即当前元素触发
  console.log('nativeEvent current target', event.nativeEvent.currentTarget);
  // React 16 中指向 document
  // 而React 17 中指向 Root Element
};
```

1. 在原生的 event 中：

   - `event.target` 返回触发事件的元素
   - `event.currentTarget` 返回绑定事件的元素

   而 React 的合成事件 SyntheticEvent 的 target 与 currentTarget 都指向触发的元素

2. React 对事件绑定做了优化，借助事件冒泡 React 其实只要给 `Document.rootElement` 绑定 `onClick` 就可以处理 App 内所有元素的 click 事件，React 自己再处理事件的冒泡与分发机制。这就是为什么 `syntheticEvent.nativeEvent` 上的 `currentTarget` 一直会是 document 根元素。
3. 即原生事件挂载到 document 根元素，但是由当前元素触发
4. <font color=#FF0000>注意：React 17 开始，事件不会再绑定在 document 上，而是绑定在 Root 组件</font>
5. 有利于多个 React 版本并存，如微前端

## React 17 中的 Event

**在 React 17 中，React 将不再向 document 附加事件处理器。而会将事件处理器附加到渲染 React 树的根 DOM 容器中**

```jsx
const rootNode = document.getElementById('root');
ReactDOM.render(<App />, rootNode);
```

![react17_event](https://zh-hans.reactjs.org/static/bb4b10114882a50090b8ff61b3c4d0fd/1e088/react_17_delegation.png)

## 传递参数

用 bind(this, a, b)

```jsx
render() {
  // 传递参数 - 用 bind(this, a, b)
  return (
    <ul>
      {this.state.list.map((item, index) => {
        return (
          <li
            key={item.id}
            onClick={this.clickHandler4.bind(this, item.id, item.title)}
          >
            index {index}; title {item.title}
          </li>
        );
      })}
    </ul>
  );
}
// 传递参数
clickHandler4(id, title, event) {
  console.log(id, title);
  console.log('event', event); // 最后追加一个参数，即可接收 event
}
```

# 表单

- 受控组件
- input textarea select 用 value
- checkbox radio 用 checked

## 受控组件

> 受控：input 的 value 受`state.name`控制  
> input 和 `state.name` 的值相关联，可通过控制`state.name`的值来控制 input 的值

input checkbox 等默认为非受控组件，其输入框内部的值受用户控制，与 React 无关，可以设置一个 defaultValue。而受控组件需要主动维护内部的`state`状态，因此可以通过为非受控组件添加`value`属性并配合`setState`与`onChange`将其转换为受控组件，实现`state`状态对组件的控制。

受控组件可以通过`onChange`事件控制用户输入，使用正则表达式等手段过滤用户的不合理输入。

```jsx
class FormDemo extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      name: 'Aliza',
      info: 'personal info',
      city: 'beijing',
      flag: true,
      gender: 'male'
    };
  }
  render() {
    // 受控组件
    return (
      <div>
        <p>{this.state.name}</p>
        <label htmlFor="inputName">姓名：</label> {/* 用 htmlFor 代替 for */}
        <input
          id="inputName"
          value={this.state.name}
          onChange={this.onInputChange}
        />
      </div>
    );
  }
  onInputChange = (e) => {
    this.setState({
      name: e.target.value
    });
  };
}
```

## textarea

> 必须使用`value={ /* ... */ }`

```jsx
render() {
  // textarea - 必须使用 value
  return <div>
    <textarea value={this.state.info} onChange={this.onTextareaChange}/>
    <p>{this.state.info}</p>
  </div>
}
onTextareaChange = (e) => {
  this.setState({
    info: e.target.value
  })
}
```

## select

> 使用`value={ /* ... */ }`

```jsx
render() {
  // select - 使用 value
  return <div>
    <select value={this.state.city} onChange={this.onSelectChange}>
      <option value="beijing">北京</option>
      <option value="shanghai">上海</option>
      <option value="shenzhen">深圳</option>
    </select>
    <p>{this.state.city}</p>
  </div>
}
onTextareaChange = (e) => {
  this.setState({
    info: e.target.value
  });
};
```

## checkbox

> 使用 `checked={ /* ... */ }`

```jsx
render() {
  // checkbox
  return (
    <div>
      <input
        type="checkbox"
        checked={this.state.flag}
        onChange={this.onCheckboxChange}
      />
      <p>{this.state.flag.toString()}</p>
    </div>
  );
}
onCheckboxChange = () => {
  this.setState({
    flag: !this.state.flag
  });
};
```

## radio

> 使用 `checked={ /* ... */ }`

```jsx
render() {
  // radio
  return <div>
    male <input type="radio" name="gender" value="male" checked={this.state.gender === 'male'} onChange={this.onRadioChange}/>
    female <input type="radio" name="gender" value="female" checked={this.state.gender === 'female'} onChange={this.onRadioChange}/>
    <p>{this.state.gender}</p>
  </div>
}
onRadioChange = (e) => {
  this.setState({
    gender: e.target.value
  });
};
```
