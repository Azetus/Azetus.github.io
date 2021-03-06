---
title: JavaScript 作用域和闭包
date: 2021-07-15 16:56:03
tags:
  - JavaScript
  - 作用域
categories:
  - JavaScript
---

# JavaScript 作用域和闭包

### 1 闭包

> 重要概念：**自由变量**

- 当一个变量在当前作用域没有定义，但被使用了。
- **在其被定义的作用域中**向上级作用域，一层一层依次寻找，直到找到位置。
- 如果到全局作用域都没有找到，则报错 'xx is not defined'

```JavaScript
// 函数作为返回值
function create() {
  let a = 100; // 向上查找到函数 create 的作用域中 a = 100
  return function () {
    console.log(a); // 自由变量
  };
}
let fn = create();
let a = 200;
fn(); // 100
```

```JavaScript
// 函数作为参数
function print(fn) {
  let a = 200;
  fn();
}
let a = 100; // 向上查找到全局作用域 a = 100
function fn() {
  console.log(a);// 自由变量
}
print(fn) // 100
```

**自由变量的查找，是函数定义的地方，向上级作用域查找。而不是其被调用的作用域。**

# this 的赋值

> **重要概念：this 的取值是在函数执行时才被确定的**

### 1 作为普通函数

以函数的形式（包括普通函数、定时器函数、立即执行函数）调用时，this 的指向永远都是 window。比如 `fn1`();相当于 `window.fn1()`;

```JavaScript
function fn1() {
  console.log(this)
}
fn1() // window
```

### 2 使用 call apply bind 调用

1. **call**

> fn1.call(this 指向, 函数实参 1, 函数实参 2)

```JavaScript
function fn1() {
  console.log(this)
}
fn1.call({ x: 100 }) // { x: 100 }
```

`call()`可以立即调用一个函数，与此同时，他还可以改变这个函数内部的 this 指向。

```JavaScript
// 模拟 call
Function.prototype.myCall = function (context) {
  // 获取传入的对象（执行上下文）
  context = context || window; // [...arguments][0]

  // 获取 fn1.apply1() 中的 fn1
  context.func = this;

  // 将参数拆解为数组
  const args = [...arguments].slice(1);

  // 执行函数并使用 result 存储结果
  const result = context.func(...args);

  // 删除函数
  delete context.func;
  return result;
};

function fn1(a, b, c) {
  console.log('this', this);
  console.log(a, b, c);
  return 'this is fn1';
}
fn1.myCall({ x: 100 }, 10, 20, 30);
// this { x: 100 }
// 10 20 30
```

2. **bind**

> fn2 = fn1.bind(this 指向, 函数实参 1, 函数实参 2);

```JavaScript
function fn1() {
  console.log(this)
}
fn2 = fn1.bind({ x: 200 })
fn2() // { x: 200 }
```

`bind()` 方法不会调用函数，但是可以改变函数内部的 this 指向，并返回一个新的函数（指定 this 和指定实参的**原函数拷贝**）。

```JavaScript
// 模拟 bind
Function.prototype.myBind = function () {
  // 将参数拆解为数组
  const args = Array.prototype.slice.call(arguments);
  // 获取 this 指向
  const that = args.shift();
  // 获取 fn1.myBind() 中的 fn1
  const self = this;
  // 返回一个函数
  return function () {
    return self.apply(that, args);
  };
};

// bind demo
function fn1(a, b, c) {
  console.log('this', this);
  console.log(a, b, c);
  return 'this is fn1';
}

const fn2 = fn1.myBind({ x: 100 }, 10, 20, 30);
const res = fn2();
console.log(res);
// this { x: 100 }
// 10 20 30
// this is fn1
```

3. **apply**

> fn1.apply(this 指向, [函数实参 1, 函数实参 2])

`apply()`可以立即调用一个函数，与此同时，他还可以改变这个函数内部的 this 指向。传入的实参<u>必须是一个数组</u>。

```JavaScript
const arr1 = [3, 7, 10, 8]

const maxValue = Math.max.apply(Math, arr1); // 求数组 arr1 中元素的最大值
console.log(maxValue); // 10
```

```JavaScript
// 模拟 apply
Function.prototype.myApply = function (context) {
  // 获取传入的对象（执行上下文）
  context = context || window;
  // fn1.apply1() 中的 fn1
  context.func = this;
  // 将参数拆解为数组
  const args = arguments[1];
  // 执行函数并使用 result 存储结果
  const result = context.func(...args);
  // 删除函数
  delete context.func;

  return result;

};
function fn1(a, b, c) {
  console.log('this', this);
  console.log(a, b, c);
  return 'this is fn1';
}
fn1.myApply({ x: 100 }, [10, 20, 30]);
// this { x: 100 }
// 10 20 30
```

### 3 作为对象的方法调用

以方法的形式调用时，this 指向调用方法的那个对象。

```JavaScript
const monika = {
  name: '莫妮卡',
  sayHi() {
    // this 即当前对象
    console.log(this);
  },
  wait() {
    setTimeout(function () {
      // this === window
      console.log(this);
    });
  },
  waitAgain() {
    setTimeout(() => {
      // this 即当前对象
      console.log(this);
    }, 1000);
  }
};
```

- `monika.sayHi()` 调用该方法的对象是 monika，因此 this 指向 monika。

- `monika.wait()` 内 setTimeout 中为匿名函数，而这个匿名函数不是作为某个对象的方法来调用执行，是在全局执行，因此 this 指向 window。

- `monika.waitAgain()` 中箭头函数的 this，取其上级作用域的 this。

### 4 class 中的 this

```JavaScript
class People {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  sayHi() {
    console.log(`Hi, 我是${this.name}`);
  }
}
const monika = new People('莫妮卡', 19);
monika.sayHi(); // monika 对象
```

在 class 中的 this 指向实例对象。

> `monika.__proto__.sayHi();` 通过原型调用 sayHi()时，this 指向隐式原型，打印结果为’Hi, 我是 undefined'。
