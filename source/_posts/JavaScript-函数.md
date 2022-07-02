---
title: 'JavaScript: 函数'
date: 2022-07-02 15:25:07
tags:
    - JavaScript
    - 函数
categories:
    - JavaScript
---

# 1 执行上下文， IIFE, 闭包，作用域，变量提升，暂时性死区重要概念一览

## 执行上下文

-   JavaScript 代码被解析和执行时的环境
-   这个是从程序的角度出发的

-   执行上下文 Execution Context
    -   this
    -   变量环境 var
    -   词法环境 let const
    -   外部环境

![6-1-1](6-1-1.png)

-   查找顺序

![6-1-2](6-1-2.png)

### 上下文

-   全局执行上下文
-   函数执行上下文
-   eval 函数执行上下文

### 执行栈

> 也可称为调用栈，用栈的结构存储程序执行过程中创建的所有执行上下文

```JavaScript
function outer() {
    inner()
}
function inner() {
    // ...
}
outer()
```

1. inner 上下文
2. outer 上下文
3. 全局上下文

## 作用域

作用域：一个独立的区域。主要的用途就是隔离变量

## 执行上下文 vs 作用域

|            | 创建时间         | 运行机制 |
| ---------- | ---------------- | -------- |
| 执行上下文 | 运行时创建       | 动态     |
| 作用域     | 函数创建时已确定 | 静态     |

## 变量提升

-   变量提升：访问“后”声明的变量

## IIFE

-   IIFE：Immediately Invoked Function
    Expressions，立即调用函数表达式
    > 是一个在定义时就会立即执行的 JavaScript 函数
-   第一部分是包围在 圆括号运算符 () 里的一个匿名函数，这个匿名函数拥有独立的词法作用域。这不仅避免了外界访问此 IIFE 中的变量，而且又不会污染全局作用域。
-   第二部分再一次使用 () 创建了一个立即执行函数表达式，JavaScript 引擎到此将直接执行函数。

```JavaScript
(function(num1,num2){
    console.log(num1+num2);
})(7,9);

(function(num1,num2){
    console.log(num1+num2);
}(7,9));

+ function (num1,num2){
    console.log(num1+num2);
}(7,9)

void function (num1,num2){
    console.log(num1+num2);
}(7,9)
```

---

# 2 name, length，caller 等重要却少被关注的属性

## Function.name

-   Function.name 属性返回函数实例的名称
-   Function.name 属性的属性特性：
    -   writable —— false
    -   enumerable —— false
    -   configurable —— true

```JavaScript
function sum(num1,num2){
    return num1+num2
}
console.length(sum.name)
```

![6-2-1](./6-2-1.png)

主要用途

-   递归
-   调试和跟踪

## Function.length

-   length 指该函数有多少个必须要传入的参数，即形参的个数
    -   不包含剩余参数
    -   不包含含有默认值的参数
    -   仅包含第一个具有默认值之前的参数
    -   bind 之后的 length => length 减去 bind 的参数个数
-   arguments.length 是实际参数长度
-   Function.length 是形参长度

主要用途

-   柯里化

## Function.caller

-   <font color=red>非标准特性</font>
-   返回调用指定函数的函数
    -   全局作用域内被调用，返回 null
    -   函数内部作用域调用，指向调用它的那个函数
    -   严格模式下，caller、callee、arguments 都不可用

主要用途

-   调用栈信息收集
-   调用环境检查

栈信息收集

```JavaScript
function getStack(fn) {
    const stacks = [];
    let caller = fn.caller;
    while (caller) {
        stacks.unshift(caller.name);
        caller = caller.caller;
    }
    return stacks;
}

function a() {
    console.log("a")
    const stacks = getStack(a);
    console.log("stacks:", stacks);
}

function b() {
    a();
    console.log("b");
}


function c() {
    b();
    console.log("c")
}


c();
```

调用环境检查

```JavaScript
function getCaller(fun) {
    const caller = fun.caller;
    if (caller == null) {
        console.log("caller is global context");
    } else {
        console.log("caller.name:" + caller);
    }
    return fun.caller
}

function add() {
    getCaller(add)
}

add();
```

## arguments.callee

-   包含正在执行的函数
-   严格模式禁止使用
-   起源：匿名函数递归问题

```JavaScript
function sumTotal(n) {
    if (n == 1) return 1;
    return sumTotal(n - 1) + n;
};
const result = [5, 10, 20].map(sumTotal)
```

如果使用匿名函数实现上述功能

```JavaScript
const result = [5, 10, 20].map(function (n) {
    if (n == 1) return 1;
    return arguments.callee(n - 1) + n;
});
```

---

# 3 纯函数，副作用，高阶函数等函数式编程概念

## 编程范式 - 面向过程编程

-   特点：主要采取过程调用或函数调用的方式来进行流程控制。流程则包括还有一系列运算步骤的过程，子程序，方法或函数来控制
-   代表：C 语言等

## 编程范式 - 面向对象编程

-   特点：将对象作为程序的基本单元，将程序和数据封装其中，以提高软件的重用性、灵活性和扩展性，对象里的程序可以访问及经常修改对象相关联的数据。在面向对象程序编程里，计算机程序被设计成彼此相关的对象
-   代表：Python, C++, Java, C#等

## 编程范式 - 函数式编程

-   特点：函数式编程更加强调程序执行的结果而非执行的过程，倡导利用若干简单的执行单元让计算结果不断渐进，逐层推到复杂的运算，而不是设计一个复杂的执行过程
-   代表：Haskell, Scala

## 函数式编程的优点

-   代码简洁、优雅
-   语法灵活、复用性高
-   容易测试
-   易升级
-   并发友好
-   可维护性好

## 纯函数

-   定义：纯函数就是相同的输入，永远得到相同的输出，并且没有任何副作用
-   同入同出
-   没有副作用

## 纯函数的优点

-   安全：无副作用，不破坏外面的状态
-   可测试：入参固定，输出固定，好断言
-   可缓存：同入同出，便于缓存，提升效率

## 函数的副作用

-   定义：函数调用时，除了返回函数值之外，还对外界产生了附加的影响
-   修改了变量
-   修改了入参
-   输出了日志（包含了引用）
-   操作了 DOM
-   发送了 Http 请求
-   操作客户端存储
-   与 serverce worker, iframe 通讯

## 高阶函数

-   定义：一个接收函数作为参数或者函数作为输出的函数
-   特征：函数入参，函数作为返回值，满足任一条件即可

例如：

-   Array.prototype.filter
-   Array.prototype.find
-   Array.prototype.map

应用：

-   柯里化
-   Function。prototype.bind

### 高阶组件

定义：包装了另外一个组件的组件

```javascript
function PropsHOC(WrappedComponent) {
    return class extends React.Component {
        render() {
            return <WrappedComponent {...this.props} />
        }
    }
}
```

## 其他函数式编程的概念

-   compose 组合
-   pipe 管道
-   偏函数
-   柯里化
-   chain 链式调用

### 补充：幂等性与纯函数

-   数学中的幂等性

    -   `foo(x)` 将产生与 `foo(foo(x))`、`foo(foo(foo(x)))` 等相同的输出

-   接口的幂等性

    -   对接口而言：幂等性实际上就是接口可重复调用，在调用方多次调用的情况下，接口最终得到的结果是一致的。比如，在 App 中下订单的时候，点击确认之后，没反应，就又点击了几次。在这种情况下，如果无法保证该接口的幂等性，那么将会出现重复下单问题
    -   http 方法的幂等：指的是同样的请求被执行一次与连续执行多次的效果是一样的，服务器的状态也是一样的（注意，只是服务器状态，和服务器返回状态无关，例如 GET 应当是幂等的，而 POST 和 PUT 则不是）

-   程序的幂等性
    -   一个函数执行多次皆返回相同的结果
    -   个函数被调用多次时，保证内部状态的一致性
    -   和纯函数相比，幂等主要强调多次调用，对内部的状态的影响是一样的（但多次调用返回值可能不同）。而纯函数，主要强调相同的输入，多次调用，输出也相同且无副作用。<font color=red>纯函数一定是幂等的</font>
