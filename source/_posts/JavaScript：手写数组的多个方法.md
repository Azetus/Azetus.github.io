---
title: JavaScript：手写数组的多个方法
date: 2022-05-22 19:45:00
tags:
    - JavaScript
    - 数组
categories:
    - JavaScript
---

# Array.isArray

-   作用：确定传递的值是否是一个 Array

```javascript
const arr = ['1']
console.log(Array.isArray(arr))
// true
```

```javascript
const arr = ['1']
const proxyArr = new Proxy(arr, {})
console.log(Array.isArray(proxyArr))
// true
```

## 关于 Proxy

```javascript
const arr = ['1']
const proxy = new Proxy(arr, {})
const log = console.log

log('__proto__:', proxy.__proto__ === Array.prototype)
log('instanceof:', proxy instanceof Array)

console.log('-------------------')
log('toString', Object.prototype.toString.call(Proxy))

log('Proxy.prototype:', Proxy.prototype)

log('proxy instanceof Proxy:', proxy instanceof Proxy)
```

```
__proto__: true
instanceof: true
-------------------
toString [object Function]
Proxy.prototype: undefined
TypeError: Function has non-object prototype 'undefined' in instanceof check
```

-   Proxy 是函数，但本身没有 prototype
-   Proxy 不改变被代理对象的外在表现

## Array.isArray - Proxy

1. If Type(argument) is not Object, return false.
2. If argument is an Array exotic object, return true.
3. If argument is a Proxy exotic object, then
    1. If argument.[[ProxyHandler]] is null, throw a TypeError exception.
    2. Let target be argument.[[ProxyTarget]].
    3. Return ? IsArray(target).
4. Return false.

> 如果参数是一个 Proxy 代理对象，将 ProxyTarget 赋值给参数，然后再判断

## Array.isArray

1. Object.prototype.toString
2. typeof + instanceof
3. constructor

```javascript
Array.myIsArray = function (obj) {
    return Object.prototype.toString.call(obj) === '[object Array]'
}
```

```javascript
Array.myIsArray = function (obj) {
    if (typeof obj !== 'object' || obj === null) {
        return false
    }
    return obj instanceof Array
}
```

---

# Array.prototype.entries

-   作用：返回一个新的 Array Iterator 对象，该对象包含数组中每个索引的键值对
-   迭代器需要挂载在对象的`[Symbol.iterator]`上

```javascript
Array.prototype[Symbol.iterator] = function () {
    const O = Object(this)
    let index = 0
    const length = O.length

    function next() {
        if (index < length) {
            return { value: O[index++], done: false }
        }
        return { value: undefined, done: true }
    }

    return {
        next
    }
}

Array.prototype.entries = function () {
    const O = Object(this)
    const length = O.length
    var entries = []

    for (var i = 0; i < length; i++) {
        entries.push([i, O[i]])
    }

    const itr = this[Symbol.iterator].bind(entries)()
    return {
        next: itr.next,
        [Symbol.iterator]() {
            return itr
        }
    }
}

Array.prototype.keys = function () {
    const O = Object(this)
    const length = O.length
    var keys = []

    for (var i = 0; i < length; i++) {
        keys.push([i])
    }

    const itr = this[Symbol.iterator].bind(keys)()
    return {
        next: itr.next,
        [Symbol.iterator]() {
            return itr
        }
    }
}
```

---

# Array.prototype.includes

-   返回一个 boolean，用来判断一个数组是否包含一个指定的值
-   与 IndexOf 不同，可以准确识别 NaN
-   第二个参数 fromIndex，为从该索引处开始向后查找

```javascript
// Number.isNaN polyfill
Number.isNaN = function (value) {
    return typeof value === 'number' && isNaN(value)
}
Array.prototype.includes = function (item, fromIndex) {
    // call, apply调用，严格模式
    if (this == null) {
        throw new TypeError('无效的this')
    }
    var O = Object(this)

    var len = O.length >> 0
    if (len <= 0) {
        return false
    }

    const isNAN = Number.isNaN(item)
    for (let i = 0; i < len; i++) {
        if (O[i] === item) {
            return true
        } else if (isNAN && Number.isNaN(O[i])) {
            return true
        }
    }
    return false
}
```

-   实现第二个参数需要参照协议对传入的索引进行判断和处理
    > https://tc39.es/ecma262/#sec-array.prototype.includes
    -   需要注意的点
        -   协议标准
        -   正负 Infinity
        -   特殊值 NaN

```javascript
function ToIntegerOrInfinity(argument) {
    var num = Number(argument)
    // + 0 和 -0
    if (Number.isNaN(num) || num == 0) {
        return 0
    }
    if (num === Infinity || num == -Infinity) {
        return num
    }
    var inter = Math.floor(Math.abs(num))
    if (num < 0) {
        inter = -inter
    }
    return inter
}

Array.prototype.includes = function (item, fromIndex) {
    // call, apply调用，严格模式
    if (this == null) {
        throw new TypeError('无效的this')
    }
    var O = Object(this)

    var len = O.length >> 0
    if (len <= 0) {
        return false
    }

    var n = ToIntegerOrInfinity(fromIndex)
    if (fromIndex === undefined) {
        n = 0
    }
    if (n === +Infinity) {
        return false
    }
    if (n === -Infinity) {
        n = 0
    }

    var k = n >= 0 ? n : len + n
    if (k < 0) {
        k = 0
    }

    const isNAN = Number.isNaN(item)
    for (let i = k; i < len; i++) {
        if (O[i] === item) {
            return true
        } else if (isNAN && Number.isNaN(O[i])) {
            return true
        }
    }
    return false
}
```

---

# Array.from

> Array.from(arrayLike[, mapFn[, thisArg]])

-   arrayLike
    -   想要转换成数组的类数组对象或可迭代对象。
-   mapFn 可选
    -   如果指定了该参数，新数组中的每个元素会执行该回调函数。
-   thisArg 可选
    -   可选参数，执行回调函数 mapFn 时 this 对象。

## 特殊值处理

-   类数组特征
-   length 作为字符串处理
-   from 函数本身 this 也是可以被该写的
-   协议中数组长度理论最大值为 2^53-1，但 Chrome 浏览器最大长度为 2^32-1

```javascript
console.log('Array.from:', Array.from({}))
console.log('Array.from:', Array.from(''))
console.log('Array.from:', Array.from({ a: 1, length: '10' }))
console.log('Array.from:', Array.from({ a: 1, length: 'ss' }))
console.log('Array.from:', Array.from([NaN, null, undefined, 0]))

const max = Math.pow(2, 32)
console.log('Array.from:', Array.from({ 0: 1, 1: 2, length: max - 1 }))
console.log('Array.from:', Array.from({ 0: 1, 1: 2, length: max }))
```

```
Array.from: []
Array.from: []
Array.from: (10)[undefined]
Array.from: []
Array.from: [ NaN, null, undefined, 0 ]
```

```javascript
var maxSafeInteger = Math.pow(2, 32) - 1

var ToIntegerOrInfinity = function (value) {
    var number = Number(value)
    if (isNaN(number)) {
        return 0
    }
    if (number === 0 || !isFinite(number)) {
        return number
    }
    return (number > 0 ? 1 : -1) * Math.floor(Math.abs(number))
}

var ToLength = function (value) {
    var len = ToIntegerOrInfinity(value)
    return Math.min(Math.max(len, 0), maxSafeInteger)
}

var isCallable = function (fn) {
    return typeof fn === 'function' || toStr.call(fn) === '[object Function]'
}

Array.from = function (arrayLike, mapFn, thisArg) {
    var C = this

    //判断对象是否为空
    if (arrayLike == null) {
        throw new TypeError('Array.from requires an array-like object - not null or undefined')
    }
    //检查mapFn是否是方法
    if (typeof mapFn !== 'function' && typeof mapFn !== 'undefined') {
        throw new TypeError(mapFn + 'is not a function')
    }

    var items = Object(arrayLike)
    //判断 length 为数字，并且在有效范围内。
    var len = ToLength(items.length)
    if (len <= 0) return []

    var A = isCallable(C) ? Object(new C(len)) : new Array(len)

    for (var i = 0; i < len; i++) {
        var value = items[i]
        if (mapFn) {
            A[i] = typeof thisArg === 'undefined' ? mapFn(value, i) : mapFn.call(thisArg, value, i)
        } else {
            A[i] = value
        }
    }
    return A
}
```

---

# Array.prototype.flat

-   按照指定深度递归遍历数组，并将所有元素与遍历到的子数组中的元素合并为一个新数组返回

## 普通版本

-   简化版，无法指定递归深度
-   reduce 会跳过空位
-   递归使用 reduce 和 concat 只适合层级不深的情况

```javascript
const array = [1, [1, , ,]]

const flat = (arr) => {
    return arr.reduce((pre, cur) => {
        return pre.concat(Array.isArray(cur) ? flat(cur) : cur)
    }, [])
}

console.log(flat(array))
```

## 协议版本

-   判断长度
-   使用 push 替代 concat
-   如果是空位 push 一个 undefined
-   如果是数组再进行递归

```javascript
var has = Object.prototype.hasOwnProperty

var maxSafeInteger = Math.pow(2, 32) - 1

var toInteger = function (value) {
    const number = Number(value)
    if (isNaN(number)) {
        return 0
    }
    if (number === 0 || !isFinite(number)) {
        return number
    }
    return (number > 0 ? 1 : -1) * Math.floor(Math.abs(number))
}

var toLength = function (value) {
    var len = toInteger(value)
    return Math.min(Math.max(len, 0), maxSafeInteger)
}

var push = Array.prototype.push

Array.prototype.flat = function (deep) {
    var O = Object(this)
    var sourceLen = toLength(O.length)
    var depthNum = 1
    if (deep !== undefined) {
        depthNum = toLength(deep)
    }
    if (depthNum <= 0) {
        return O
    }
    var arr = []

    var val
    for (var i = 0; i < sourceLen; i++) {
        if (has.call(O, i)) {
            val = O[i]
            if (Array.isArray(val)) {
                push.apply(arr, val.flat(depthNum - 1))
            } else {
                arr.push(val)
            }
        } else {
            arr.push(undefined)
        }
    }

    return arr
}
```
