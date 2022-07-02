---
title: JavaScript：类数组
date: 2022-05-22 19:38:17
tags:
    - JavaScript
    - 类数组
categories:
    - JavaScript
---

# 深入浅出类数组

-   什么是类数组？
-   类数组的特征？
-   常见的类数组？

# 创建数组的几种方式

-   数组对象字面量
    -   `const arr = [1, 2, 3]`
-   `new Array()`
-   `Array.from()` - ES6
-   `Array.of()` - ES6
-   `Array.prototype.slice`
-   `Array.prototype.concat`

# 什么是类数组

-   类数组：是一个有<font color=red>length 属性</font>和<font color=red>从零开始索引的属性</font>，但是没有 Array 的内置方法，比如 forEach()和 map()等的一种特殊**对象**

# 类数组特征

-   是一个普通对象
-   必须有 length 属性，可以有非负整数索引
-   本身不具备数组所具备的方法

# 常见的类数组

## arguments

-   arguments 对象

```javascript
function fn(a, b, c) {
    console.log('fn arguments:', arguments)
    console.log('fn type::', Object.prototype.toString.call(arguments))
}
```

![5-2-1](5-2-1.png)

## DOM 相关

-   NodeList，HTMLCollection，DOMTokenList 等

## 特殊 - 字符串

-   字符串：具备类数组的所有特性，<font color=red>但是类数组一般是指对象</font>

```javascript
var str = 'abc'
console.log(Array.from(str))
```

```
['a', 'b', 'c']
```

# 类数组和数组的区别

|               | 数组              | 类数组             |
| ------------- | ----------------- | ------------------ |
| toString 返回 | [object Array]    | [object Object]    |
| instanceof    | Array             | Object             |
| constructor   | [Function: Array] | [Function: Object] |
| Array.isArray | true              | false              |

# 代码判断类数组

```javascript
//使用 isFinite() 在检测无穷数：
function isArrayLikeObject(arr) {
    const typeStr = typeof arr
    // if (typeStr === 'string') {
    //     return true
    // }
    if (arr == null || typeStr !== 'object') return false

    const lengthMaxValue = Math.pow(2, 53) - 1
    if (!Object.prototype.hasOwnProperty.call(arr, 'length')) return false
    if (typeof arr.length != 'number') return false
    if (!isFinite(arr.length)) return false
    if (Array === arr.constructor) return false

    if (arr.length >= 0 && arr.length < lengthMaxValue) {
        return true
    } else {
        return false
    }
}
```

# 类数组转为数组

-   slice，concat 等
-   Array.from
-   Array.apply
-   复制遍历
-   ES6 扩展运算符

```javascript
const array1 = Array.prototype.slice.call(arrayLikeObj)

const array2 = Array.from(arrayLikeObj)

const array3 = Array.prototype.concat.apply([], arrayLikeObj)
const array4 = Array.apply(null, arrayLike)

const array5 = [...arrayLikeObj]
```

# 类数组的迭代器

-   1 借用数组的 Symbol。iterator
-   2 手动实现

```javascript
// 借用其他数组的Symbol.iterator
var arrayLikeObj = {
    length: 2,
    0: 1,
    1: 2,
    [Symbol.iterator]: [][Symbol.iterator]
}
console.log([...arrayLikeObj]) // [1,2]
```

```javascript
// 自己实现一个 Symbol.iterator
var arrayLikeObj = {
    length: 2,
    0: 1,
    1: 2,
    [Symbol.iterator]() {
        const self = this
        let index = 0
        return {
            next() {
                if (index < self.length) {
                    return {
                        value: self[index++],
                        done: false
                    }
                }
                return { value: undefined, done: true }
            }
        }
    }
}
```
