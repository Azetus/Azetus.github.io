---
title: 'JavaScript: call'
date: 2022-07-02 15:30:45
tags:
    - JavaScript
    - call
categories:
    - JavaScript
---

# 神奇的 call.call, call.call.call

```javascript
function a() {
    console.log(this, 'a')
}
function b() {
    console.log(this, 'b')
}

a.call(b)
```

a 被调用，this 指向 b

```javascript
function a() {
    console.log(this, typeof this, 'a')
}
function b() {
    console.log(this, typeof this, 'b')
}
a.call.call(b, 'b')
a.call.call.call(b, 'b')
a.call.call.call.call(b, 'b')
```

执行结果

```
[String: 'b'] object b
[String: 'b'] object b
[String: 'b'] object b
```

三个问题

-   为什么被调用的是 b 函数
-   为什么 this 指向`String{"b"}
-   为什么 3 个 call 的结果都一样

## 简单回顾 call 和 this

-   call：使用一个指定的 this 值和单独给出的一个或多个参数来调用一个函数。
-   this：执行上下文的一个变量
-   this：另外一种不严谨的说法，this 指向调用者

### call 的一种虚拟语法

`func.call(obj, ...args) === (obj.func = func; obj.func(...args))`

## 为什么 3 个 call 的结果都一样

-   a.call(b)：a 被调用
-   a.call.call(b)：a.call 被调用
-   a.call.call.call(b)：a.call.call 被调用

```javascript
a.call === Function.prototype.call // true
a.call === a.call.call // true
a.call === a.call.call.call // true
```

## 为什么被调用的是 b 函数

    a.call.call(b, 'b')
    (a.call).call(b, 'b')

1. 一个函数进行 call 调用，等同于在一个对象上执行该函数
    > 等于在 b 对象上调用 a.call 函数
2. a.call === Function.prototype.call
    > Function.prototype.call.call(b,'b')
3. 等同于在 b 调用 call 函数，this 参数为'b'
    > b.call('b')

## 为什么 this 指向`String{"b"}

-   this：非严格模式下会使用 Object 包装
-   this：严格模式下是任意值

## 万能的函数调用方法

-   `var call = Function.prototype.call.call.bind(Function.prototype.call); `

---

# 2 手写 call

-   某个方法进行 call 调用时，等同于吧方法作为 call 的第一个参数的某个属性，并进行调用
-   func.call(obj, ...args) === (obj.func = func; obj.func(...args))

## 版本 1

```javascript
Function.prototype.call = function (context) {
    context = context || window
    context['fn'] = this
    let arg = [...arguments].slice(1)
    const r = context['fn'](...arg)
    delete context['fn']
    return r
}
```

## 版本 2

```javascript
Function.prototype.call = function (context) {
    context = context == null || context == undefined ? window : new Object(context)
    context.fn = this
    var arr = []
    for (var i = 1; i < arguments.length; i++) {
        arr.push('arguments[' + i + ']')
    }
    var r = eval('context.fn(' + arr + ')')
    delete context.fn
    return r
}
```

## 存在的问题

-   this 是不是可以被调用
    > var obj = Object.create(Function.prototype)  
    > obj.call({})
-   undefined 是否安全
    > undefined 有可能会被改写  
    > void 0 更安全
-   window 作为默认上下文过于武断（node 环境中？）
-   eval 一定会被允许执行吗
-   delete context.fn 是否有副作用

### 补充：void 的其他用法

1. href

```html
<a href="javascript:void(0)">不要点击我</a>
```

2. 立即执行函数

```
void function (msg) {
    console.log(msg)
}('立即执行')
```

## 环境识别

-   浏览器环境
-   nodejs 环境
-   综合判断

```javascript
var getGlobal = function () {
    if (typeof self !== 'undefined') {
        return self // 浏览器中
    }
    if (typeof window !== 'undefined') {
        return window
    }
    if (typeof global !== 'undefined') {
        return global
    }
    throw new Error('unable to locate global object')
}
```

## 严格模式

-   是否支持严格模式
    -   全局上下文调用函数 this 指向 undefined
-   是否处于严格模式

### 是否支持严格模式

```javascript
var hasStrictMode = (function () {
    'use strict'
    return this == undefined
})()

console.log(hasStrictMode)
```

### 是否处于严格模式

```javascript
var isStrict = (function () {
    return this === undefined
})()

function test() {
    'use strict'
    console.log('isStrict:', isStrict)
    console.log(arguments.callee)
}

console.log(test.toString())
```

## 函数副作用

-   函数调用后，破坏了原对象

## 基于 eval 实现

![6-11-1](6-11-1.png)

```javascript
var hasStrictMode = (function () {
    'use strict'
    return this == undefined
})()

var isStrictMode = function () {
    return this === undefined
}

var getGlobal = function () {
    if (typeof self !== 'undefined') {
        return self
    }
    if (typeof window !== 'undefined') {
        return window
    }
    if (typeof global !== 'undefined') {
        return global
    }
    throw new Error('unable to locate global object')
}

function isFunction(fn) {
    return typeof fn === 'function' || Object.prototype.toString.call(fn) === '[object Function]'
}

function getContext(context) {
    // 是否是严格模式
    var isStrict = isStrictMode()
    // 没有严格模式，或者有严格模式但不处于严格模式
    if (!hasStrictMode || (hasStrictMode && !isStrict)) {
        return context === null || context === void 0 ? getGlobal() : Object(context)
    }

    // 严格模式下, 妥协方案
    return Object(context)
}

Function.prototype.call = function (context) {
    // 不可以被调用
    if (!isFunction(this)) {
        throw new TypeError(this + ' is not a function')
    }

    // 获取上下文
    var ctx = getContext(context)

    // 更为稳妥的是创建唯一ID, 以及检查是否有重名
    var propertyName = '__fn__' + Math.random() + '_' + new Date().getTime()
    var originVal
    var hasOriginVal = isFunction(ctx.hasOwnProperty) ? ctx.hasOwnProperty(propertyName) : false
    if (hasOriginVal) {
        originVal = ctx[propertyName]
    }

    ctx[propertyName] = this

    // 采用string拼接
    var argStr = ''
    var len = arguments.length
    for (var i = 1; i < len; i++) {
        argStr += i === len - 1 ? 'arguments[' + i + ']' : 'arguments[' + i + '],'
    }
    var r = eval('ctx["' + propertyName + '"](' + argStr + ')')

    // 还原现场
    if (hasOriginVal) {
        ctx[propertyName] = originVal
    } else {
        delete ctx[propertyName]
    }

    return r
}

// 测试
function log() {
    console.log('name:', this.name)
}

log.call({ name: 'name' })
```

## 基于 new Function 的实现

思路

```javascript
function createFun(argsLength) {
    // return ctx[propertyName](arg1, arg2, arg3,...)
    // 拼接函数
    var code = 'return ctx[propertyName]('
    // 拼接参数, 第二个起是参数
    for (var i = 0; i < argsLength; i++) {
        if (i > 0) {
            code += ','
        }
        code += 'args[' + i + ']'
    }
    code += ')'
    return new Function('ctx', 'propertyName', 'args', code)
}
```

```
console.log(createFun(3).toString())

/*
ƒ anonymous(ctx,propertyName,args
) {
return ctx[propertyName](args[0],args[1],args[2])
}
*/
```

### 最终实现

```javascript
var hasStrictMode = (function () {
    'use strict'
    return this == undefined
})()

var isStrictMode = function () {
    return this === undefined
}

var getGlobal = function () {
    if (typeof self !== 'undefined') {
        return self
    }
    if (typeof window !== 'undefined') {
        return window
    }
    if (typeof global !== 'undefined') {
        return global
    }
    throw new Error('unable to locate global object')
}

function isFunction(fn) {
    return typeof fn === 'function' || Object.prototype.toString.call(fn) === '[object Function]'
}

function getContext(context) {
    var isStrict = isStrictMode()

    if (!hasStrictMode || (hasStrictMode && !isStrict)) {
        return context === null || context === void 0 ? getGlobal() : Object(context)
    }
    // 严格模式下, 妥协方案
    return Object(context)
}

function createFun(argsLength) {
    // return ctx[propertyName](arg1, arg2, arg3,...)
    // 拼接函数
    var code = 'return ctx[propertyName]('

    // 拼接参数, 第二个起是参数
    for (var i = 0; i < argsLength; i++) {
        if (i > 0) {
            code += ','
        }
        code += 'args[' + i + ']'
    }
    code += ')'

    return new Function('ctx', 'propertyName', 'args', code)
}

Function.prototype.call = function (context) {
    // 不可以被调用
    if (typeof this !== 'function') {
        throw new TypeError(this + ' is not a function')
    }

    // 获取上下文
    var ctx = getContext(context)

    // 更为稳妥的是创建唯一ID, 以及检查是否有重名
    var propertyName = '__fn__' + Math.random() + '_' + new Date().getTime()
    var originVal
    var hasOriginVal = isFunction(ctx.hasOwnProperty) ? ctx.hasOwnProperty(propertyName) : false
    if (hasOriginVal) {
        originVal = ctx[propertyName]
    }

    ctx[propertyName] = this

    var argArr = []
    var len = arguments.length
    for (var i = 1; i < len; i++) {
        argArr[i - 1] = arguments[i]
    }

    var r = createFun(len - 1)(ctx, propertyName, argArr)

    // 还原现场
    if (hasOriginVal) {
        ctx[propertyName] = originVal
    } else {
        delete ctx[propertyName]
    }
    return r
}
```
