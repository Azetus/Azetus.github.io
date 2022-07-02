---
title: 'JavaScript: 链式调用'
date: 2022-07-02 15:36:55
tags:
    - JavaScript
    - 链式调用
categories:
    - JavaScript
---

# 链式调用的本质

## 熟知的案例

-   jQuery
-   数组
-   ES6 异步 fetch
-   EventEmitter

## 链式调用的本质

-   返回对象本身
-   或者返回同类型的实例对象

## 链式调用的优点

-   可读性强，语义好理解
-   代码简洁
-   易维护

## 链式调用的缺点

-   调试不方便
-   消耗大

## 链式调用适用场景

-   需要多次计算或者赋值
-   逻辑上有特定的顺序
-   相似业务的集中处理

## 示例：写一个计算器

-   第一种写法：返回本身
-   第二种写法：返回同类型对象实例

### 每次调用返回自身

```JavaScript
class Calculator {
    constructor(val) {
        this.val = val;
    }

    double() {
        this.val = this.val * 2;
        return this
    }

    add(num) {
        this.val = this.val + num;
        return this
    }

    minus(num) {
        this.val = this.val - num;
        return this
    }

    multi(num) {
        this.val = this.val * num;
        return this
    }

    divide(num) {
        this.val = this.val / num;
        return this
    }

    pow(num) {
        this.val = Math.pow(this.val, num);
        return this
    }

    // ES5 getter, 表现得像个属性，实则是一个方法
    get value() {
        return this.val;
    }
}

const cal = new Calculator(10);

const val = cal.add(10) // 20
    .minus(5) // 15
    .double() // 30
    .multi(10) // 300
    .divide(2) // 150
    .pow(2)   // 22500
    .value;
console.log(val); // 22500
```

### 每次调用返回同类型实例

```JavaScript
class Calculator {
    constructor(val) {
        this.val = val;
    }

    double() {
        const val = this.val * 2;
        return new Calculator(val)
    }

    add(num) {
        const val = this.val + num;
        return new Calculator(val)
    }

    minus(num) {
        const val = this.val - num;
        return new Calculator(val)
    }

    multi(num) {
        const val = this.val * num;
        return new Calculator(val)
    }

    divide(num) {
        const val = this.val / num;
        return new Calculator(val)
    }

    pow(num) {
        const val = Math.pow(this.val, num);
        return new Calculator(val)
    }

    get value() {
        return this.val;
    }
}

const cal = new Calculator(10);

const val = cal.add(10) // 20
    .minus(5) // 15
    .double() // 30
    .multi(10) // 300
    .divide(2) // 150
    .pow(2)   // 22500
    .value;
console.log(val); // 22500
```

### 其他类似的方案

-   compose 和 pipe

```javascript
function double(val) {
    return val * 2
}

function add(val, num) {
    return val + num
}

function minus(val, num) {
    return val - num
}

function multi(val, num) {
    return val * num
}

function divide(val, num) {
    return val / num
}

function pow(val, num) {
    return Math.pow(val, num)
}

function pipe(...funcs) {
    if (funcs.length === 0) {
        return (arg) => arg
    }

    if (funcs.length === 1) {
        return funcs[0]
    }
    return funcs.reduceRight(
        (a, b) =>
            (...args) =>
                a(b(...args))
    )
}

const cal = pipe(
    (val) => add(val, 10),
    (val) => minus(val, 5),
    double,
    (val) => multi(val, 10),
    (val) => divide(val, 2),
    (val) => pow(val, 2)
)

console.log(cal(10))
```
