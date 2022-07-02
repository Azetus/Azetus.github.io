---
title: JavaScript：数组小技巧（上）
date: 2022-05-22 19:33:03
tags:
    - JavaScript
    - 数组
categories:
    - JavaScript
---

# 先看 2 小个问题

1. 如果快速的批量制造数据
2. 两个数组去重

## 批量制造数据 - for

-   思路：for 循环，push

```javascript
function createData() {
    const data = []
    for (let i = 0; i < 1000; i++) {
        data.push({
            name: `name${i + 1}`
        })
    }
    return data
}
```

## 批量制造数据 - map

-   思路：创造空数组，然后 map

```javascript
function createData() {
    return new Array(1000).map((v, i) => ({
        name: `name${i + 1}`
    }))
}
```

结果

```
[ <1000 empty items> ]
```

`new Array(n)`初始化的数组中的每一项为`empty`，不会参与到后续的 map 操作中。

## 批量制造数据 - fill + map

```javascript
function createData() {
    return new Array(1000).fill(null).map((v, i) => ({ name: `name${i + 1}` }))
}
```

## 批量制造数据 - Array.from

-   思路：Array.from 第二个初始化函数返回数据

```javascript
function createData() {
    return Array.from({ length: 1000 }, (v, i) => ({ name: `name${i + 1}` }))
}
```

## 数组去重 - Set

-   思路：Set 的特性

```javascript
const arr1 = [1, 2, 3]
const arr2 = [3, 4, 5]

console.log(new Set([...arr1, ...arr2]))
```

## 数组去重 - 非 Set

-   思路：for 遍历，indexOf 判断是否存在
    -   无法去重 NaN

```javascript
const arr1 = [1, 2, 3]
const arr2 = [3, 4, 5]

function mergeArray(arr1, arr2) {
    // 克隆
    var arr = arr1.slice(0)
    var v
    for (let i = 0; i < arr2.length; i++) {
        v = arr2[i]
        // 这个操作，
        // 详情参见4.2位运算符的妙用：奇偶数，色值换算，换值， 编码等
        if (~arr.indexOf(v)) {
            continue
        }
        arr.push(v)
    }
    return arr
}

console.log(mergeArray(arr1, arr2))
```

```javascript
const arr1 = [1, 2, 3, NaN]
const arr2 = [3, 4, 5, NaN]
console.log(mergeArray(arr1, arr2))
```

```
[
  1, 2, 3, NaN,
  4, 5, NaN
]
```

## 对象去重

-   不是判断引用，而是判断属性的值是否相等
-   思路：for 遍历，find | findIndex 判断是否存在

```javascript
function mergeArray(arr1, arr2) {
    // 克隆
    var arr = arr1.slice(0)
    var v
    for (var i = 0; i < arr2.length; i++) {
        v = arr2[i]
        // 这个操作，
        // 详情参见4.2位运算符的妙用：奇偶数，色值换算，换值， 编码等
        if (~arr.findIndex((el) => el.id === v.id)) {
            continue
        }
        arr.push(v)
    }
    return arr
}
```

---

# 数组方法使用注意事项

## 序

-   各个数组的长度是多少？

```javascript
const arr1 = [1]
const arr2 = [1, ,]
const arr3 = new Array('10')
const arr4 = new Array(10)
```

```
arr1 length: 1
arr2 length: 2
arr3 length: 1
arr4 length: 10
```

## 数组空元素

```javascript
const arr2 = [1, ,]
```

```
[ 1, <1 empty item> ]
```

-   empty: 数组的空位，指数组的某一位置没有任何值，简单来说，就是数组上没有对应的属性
-   一般的遍历，自动跳过空位，如 forEach, reduce 等
-   基于值进行运算，空位的值作为 undefined
    -   find, findIndex, includes 等,indexOf 除外
    -   被作为迭代时，参与 Object.entries, 扩展运算符，for...of 等
-   Join 和 toString 时，空位作为空字符串处理

## 稀疏数组

-   有空元素的数组，就是稀疏数组
    -   V8 引擎访问稀疏数组时使用的是散列表，访问速度相对会慢一点

## 如何避免创建稀疏数组

-   `Array.apply(null,Array(3))`
-   `[...new Array(3)]`
-   `Array.from(Array(3))`

## 数组不会自动添加分号

-   `(, [, +, -, /`其中为一行代码的开头，解析时不会自动添加分号

```javascript
const objA = { a: 1 }['a']
// 1 =>{a:1}['a']

const objB = ['a']['a']
// undefined
```

## indexOf 与 includes

| 方法     | 返回值  | 是否能找到 NaN | [ , , ] | undefined |
| -------- | ------- | -------------- | ------- | --------- |
| indexOf  | number  | ×              | ×       | √         |
| includes | boolean | √              | √       | √         |

```javascript
const array2 = [1, ,]
console.log('array.includes ,,:', array2.includes(undefined))
console.log('array.indexOf ,,:', array2.indexOf(undefined) > -1)

const array3 = [undefined]
console.log('array.includes undefined:', array3.includes(undefined))
console.log('array.indexOf undefined:', array3.indexOf(undefined) > -1)
```

```
array.includes ,,: true
array.indexOf ,,: false
array.includes undefined: true
array.indexOf undefined: true
```

如何判断空位还是 undefined

-   使用`hasOwnProperty`

## 数组可变长度问题

-   length 代表数组中元素个数，数组额外附加属性不计算在内
-   length 可以改写，可以通过修改 length 改变数组长度
-   数组操作不存在越界，找不到下标，返回 undefined

## 数组查找或者过滤

![5-3-1](5-3-1.png)

## 改变自身的方法

-   pop, shift, splice
-   unshift, push, sort, reverse
-   ES6:copyWithin, fill

## sort 注意事项

-   默认按照 ASII 马先后顺序排序（10 会排在 2 前面）

## 数组操作非线性存储问题

![5-3-2](5-3-2.png)

```javascript
myArray[10000] = '1'
```

![5-3-3](5-3-3.png)

-   数组默认存储为线性存储
-   存储结构改变必然产生不必要代价，尽量线性递增

## delete 误区

-   delete 删除数组元素，后面元素不会补齐，delete 只删除引用

```javascript
const array = [1, 2, 3, 4, 5]
delete array[2]
```

```
[ 1, 2, <1 empty item>, 4, 5 ]
```

## push vs concat

-   大量操作时尽量使用 push

## 注意事项

1. 不要使用负数索引
2. 尽量指定数组大小
3. 使用连续赋值
