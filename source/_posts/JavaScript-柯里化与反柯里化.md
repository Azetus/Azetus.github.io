---
title: 'JavaScript: 柯里化与反柯里化'
date: 2022-07-02 15:35:25
tags:
    - JavaScript
    - 柯里化与反柯里化
categories:
    - JavaScript
---

-   柯里化：柯里化是将一个 N 元函数转换为 N 个一元函数，它持续的返回一个新的函数，直到所有的参数用尽位置，然后柯里化链中的最后一个函数被返回并且执行时，才会全部执行
-   是一种函数转换，多元函数转换为多个一元函数

# 一个最简单的例子

```javascript
function calcSum(num1, num2, num3) {
    return num1 + num2 + num3
}

function curryCalcSum(num1) {
    return function (num2) {
        return function (num3) {
            return num1 + num2 + num3
        }
    }
}

console.log('calcSum:', calcSum(3, 4, 5))
console.log('curryCalcSum:', curryCalcSum(3)(4)(5))
```

# 实现通用柯里化

-   接收一个需要柯里化的方法
-   存放每次函数调用的参数
-   参数数目不够原函数参数数目，不调用原函数，返回新的函数继续接收下一个参数，反之调用参数

```javascript
var curry = function (fn) {
    var args = Array.prototype.slice.call(arguments, 1)
    return _curry.apply(this, [fn, fn.length].concat(args))
}

function _curry(fn, len) {
    var oArgs = Array.prototype.slice.call(arguments, 2)
    return function () {
        var args = oArgs.concat(Array.prototype.slice.call(arguments))
        if (args.length >= len) {
            return fn.apply(this, args)
        } else {
            return _curry.apply(this, [fn, len].concat(args))
        }
    }
}
```

```javascript
function calcSum(num1, num2, num3) {
    return num1 + num2 + num3
}
const calcSumCurry = curry(calcSum, 3)
calcSumCurry(4, 5)
calcSumCurry(4)(5)
```

## 变体

-   对传入的参数数目做一个判断

```javascript
var slice = Array.prototype.slice
var curry = function (fn, length) {
    var args = slice.call(arguments, 2)
    return _curry.apply(this, [fn, length || fn.length].concat(args))
}

function _curry(fn, len) {
    var oArgs = slice.call(arguments, 2)
    return function () {
        var args = oArgs.concat(slice.call(arguments))
        if (arguments.length === 0) {
            if (args.length >= len) {
                return fn.apply(this, args)
            }
            return console.warn('curry:参数长度不足')
        } else {
            return _curry.apply(this, [fn, len].concat(args))
        }
    }
}

function calcSum() {
    return [...arguments].reduce((pre, value, index) => {
        return pre + value
    }, 0)
}
```

## 柯里化的作用

-   参数复用、逻辑复用
-   延迟计算/执行

示例：

```javascript
var slice = Array.prototype.slice
var curry = function (fn, length) {
    var args = slice.call(arguments, 2)
    return _curry.apply(this, [fn, length || fn.length].concat(args))
}

function _curry(fn, len) {
    var oArgs = slice.call(arguments, 2)
    return function () {
        var args = oArgs.concat(slice.call(arguments))
        if (args.length >= len) {
            return fn.apply(this, args)
        } else {
            return _curry.apply(this, [fn, len].concat(args))
        }
    }
}

function log(logLevel, msg) {
    console.log(`${logLevel}:${msg}:::${Date.now()}`)
}

//柯里化log 方法
const curryLog = curry(log)

const debugLog = curryLog('debug')

const errLog = curryLog('error')

//复用参数debug
debugLog('testDebug1')
debugLog('testDebug2')

//复用参数error
errLog('testError1')
errLog('testError2')
```

# 偏函数

-   固定一部分参数，然后产生更小单元的函数
-   分两次传递参数

```javascript
function partial(fn) {
    const args = [].slice.call(arguments, 1)
    return function () {
        const newArgs = args.concat([].slice.call(arguments))
        return fn.apply(this, newArgs)
    }
}

function calcSum(num1, num2, num3) {
    return num1 + num2 + num3
}
const pCalcSum = partial(calcSum, 10)

console.log(pCalcSum(11, 12))
```

## 偏函数与柯里化的区别

-   柯里化是将一个多参数函数转换为单参数的函数，将一个 N 元函数转换为 N 个 1 元函数
-   偏函数是固定一部分参数（一个或多个参数），将一个 N 元函数转换成一个 N-X 元函数

# 反柯里化

-   扩大适用性，使原来作为特定对象所拥有的功能的函数可以被任意对象所用

```javascript
function unCurry(fn) {
    return function (context) {
        return fn.apply(context, Array.prototype.slice.call(arguments, 1))
    }
}
```

## 示例

数组的 push 方法

```javascript
Function.prototype.unCurry = function () {
    const self = this
    return function () {
        return Function.prototype.call.apply(self, arguments)
    }
}

const push = Array.prototype.push.unCurry()
const obj = {}
push(obj, 4, 5, 6)
console.log(obj)
```

对象变成了一个类数组

```
{ '0': 4, '1': 5, '2': 6, length: 3 }
```

## 其他实现版本

```javascript
Function.prototype.unCurry = function () {
    var self = this
    return function () {
        return Function.prototype.call.apply(self, arguments)
    }
}
Function.prototype.unCurry = function () {
    return this.call.bind(this)
}
Function.prototype.unCurry = function () {
    return (...args) => this.call(...args)
}
```

## 反柯里化的使用场景

-   借用数组方法
-   复制数组
-   发送事件

### 复制数组

```javascript
Function.prototype.unCurry = function () {
    const self = this
    return function () {
        return Function.prototype.call.apply(self, arguments)
    }
}

const clone = Array.prototype.slice.unCurry()

var a = [1, 2, 3]

var b = clone(a)

console.log('a==b:', a === b)
console.log(a, b)
```

### 发送事件

```javascript
Function.prototype.unCurry = function () {
    const self = this
    return function () {
        return Function.prototype.call.apply(self, arguments)
    }
}

const dispatch = EventTarget.prototype.dispatchEvent.unCurry()

window.addEventListener('event-x', (ev) => {
    console.log('event-x', ev.detail) // event-x ok
})

dispatch(window, new CustomEvent('event-x', { detail: 'ok' }))
```
