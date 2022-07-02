---
title: JavaScript：数组小技巧（下）
date: 2022-05-22 19:41:59
tags:
    - JavaScript
    - 数组
categories:
    - JavaScript
---

# 数据生成器

## 万能数据生成器

```javascript
const createValues = (creator, length = 10) => Array.from({ length }, creator)
```

## 随机数生成器

```javascript
const createRandomValues = (len) => createValues(Math.random, len)
```

## 序列生成器

```javascript
const createRange = (start, stop, step) => createValues((_, i) => start + i * step, (stop - start) / step + 1)
```

## 数据生成器

```javascript
function createUser(v, index) {
    return {
        name: `user-${index}`,
        age: (Math.random() * 100) >> 0
    }
}

const users = createValues(createUser, 100)
```

# 清空数组

-   `arr.splice(0)`
-   `arr.length = 0`

# 数组去重

-   `Array.from(new Set(arr))`
-   对于属性键和属性值都相同的对象没有用
    -   `Array.prototype.filter` + key 唯一

```javascript
function uniqueArray(arr = [], key) {
    const keyValues = new Set()
    let val
    return arr.filter((obj) => {
        val = obj[key]
        if (keyValues.has(val)) {
            return false
        }
        keyValues.add(val)
        return true
    })
}
```

# 求交集

-   `Array.prototype.filter` + includes

```javascript
function intersectSet(arr1, arr2) {
    return [...new Set(arr1)].filter((item) => arr2.includes(item))
}
```

-   引用类型：Array.prototype.forEach + Map + key 唯一
-   基础数据：Array.prototype.forEach + Map + filter

```javascript
// 引用类型
function intersect(arr1, arr2, key) {
    const map = new Map()
    arr1.forEach((val) => map.set(val[key]))

    return arr2.filter((val) => {
        return map.has(val[key])
    })
}

// 原始数据类型
function intersectBase(arr1, arr2) {
    const map = new Map()
    arr1.forEach((val) => map.set(val))

    return arr2.filter((val) => {
        return map.has(val)
    })
}
```

## 性能对比

对于原始数据类型 Set+filter 的<font color=red>成本远远高于</font> forEach + Map + filter

# 求差集

-   与求交集类似
    -   结果取反

```javascript
// 引用类型
function difference(arr1, arr2, key) {
    const map = new Map()
    arr1.forEach((val) => map.set(val[key]))

    return arr2.filter((val) => {
        return !map.has(val[key])
    })
}

// 原始数据类型
function differenceBase(arr1, arr2) {
    const map = new Map()
    arr1.forEach((val) => map.set(val))

    return arr2.filter((val) => {
        return !map.has(val)
    })
}
```

# 从数组中删除虚值

```javascript
const newArray = array.filter(Boolean)
```

# 取数组中最大值和最小值

-   `Math.max.apply(Math, numArray)`
-   `Math.min.apply(Math, numArray)`
    -   apply 第二个参数是一个数组，改变后的 this 指向 Math 自身

---

# reduce

-   `Array.prototype.reduce(callbackFn, initialValue)`

    -   callbackFn()

        -   previousValue：上一次调用 callbackFn 时的返回值。在第一次调用时，若指定了初始值 nitialValue，其值则为 initialValue，否则为数组索引为 0 的元素 array[0]。

        -   currentValue：数组中正在处理的元素。在第一次调用时，若指定了初始值 initialValue，其值则为数组索引为 0 的元素 array[0]，否则为 array[1]。

        -   currentIndex：数组中正在处理的元素的索引。若指定了初始值 initialValue，则起始索引号为 0，否则从索引 1 起始。
        -   array：用于遍历的数组。

    -   initialValue 可选
        -   作为第一次调用 callback 函数时参数 previousValue 的值。若指定了初始值 initialValue，则 currentValue 则将使用数组第一个元素；否则 previousValue 将使用数组第一个元素，而 currentValue 将使用数组第二个元素。

## queryString

-   什么是 queryString
    > https://xxx.com/search/<font color=red>?word=javascript</font>
-   作用：页面传递参数
-   规律：地址 url 问号(?)拼接的键值对

## 现代 Web API 查询 queryString

-   URLSearchParams
-   URL

```javascript
const urlSP = new URLSearchParams(location.search)
function getQueryString(key) {
    return urlSP.get(key)
}
```

```javascript
const urlObj = new URL(location.href)
function getQueryString(key) {
    return urlObj.searchParams.get(key)
}
```

-   `urlObj.searchParams instanceof URLSearchParams` => `true`

## 兼容性的场景

### queryString 规律

-   "?" 之后
-   "&" 分割得到一组键值对
-   "=" 分割键和值

![5-5-1](5-5-1.png)

```javascript
const urtObj = location.search
    .slice(1)
    .split('&')
    .filter(Boolean)
    .reduce(function (obj, cur) {
        var arr = cur.split('=')
        if (arr.length != 2) {
            return obj
        }
        obj[decodeURIComponent(arr[0])] = decodeURIComponent(arr[1])
        return obj
    }, {})

function getQueryString(key) {
    return urtObj[key]
}
```

## 另一个例子：折上折

-   优惠 1：9 折
-   优惠 2：满 200 减 50
-   优惠 3：...
-   如果后期还要添加新的优惠方式？
    -   使用...rest 与 reduce 方法

```javascript
// 最后传入的方法先执行
function compose(...funcs) {
    if (funcs.length === 0) {
        return (arg) => arg
    }
    return funcs.reduce(
        (a, b) =>
            (...args) =>
                b(a(...args))
    )
}

function discount(x) {
    console.log('discount')
    return x * 0.9
}
function reduce(x) {
    console.log('reduce')
    return x > 200 ? x - 50 : x
}
function discountPlus(x) {
    console.log('discountPlus')
    return x * 0.95
}
const getPrice = compose(discount, reduce, discountPlus)

const print = console.log

print(getPrice(200))
print(getPrice(250))
```

## 另一个场景

1. 登录
2. 获取用户信息
3. 获取用户订单

-   await 排比句

```javascript
function login(...args):Promise<any>{
    // ...
}
function getUserInfo(...args):Promise<any>{
    // ...
}
function getOrders(...args):Promise<any>{
    // ...
}

function async orders(logindata){
    const loginRes = await login(logindata)
    const userRes = await getUserInfo(loginRes)
    const orderRes = await getOrders(userRes)
}
```

-   promise 链式调用

```javascript
Promise.resolve(initData)
    .then((data) => login(data))
    .then((data) => getUserInfo(data))
    .then((data) => getOrders(data))
    .then((data) => console.log('orders', data))
```

-   使用 reduce 实现
    -   将初始化数据作为 initData 使用`Promise.resolve()`包装后传入 reduce 的第二个参数
    -   reduce 的第二个参数是作为第一次调用 callback 函数时参数 previousValue 的值，使用 Promise 链式调用的方式，调用下一个方法

```javascript
function login(data) {
    return new Promise((resolve) => {
        // ...
    })
}

function runPromises(promiseCreators, initData) {
    return promiseCreators.reduce((promise, next) => {
        return promise.then((data) => next(data))
    }, Promise.resolve(initData))
}

runPromises([login, getUserInfo, getOrders], initData).then((res) => {
    console.log('res', res)
})
```

## 数组分组

-   数据按照某个特性归类
    -   传入 fn 的返回值必须能作为对象的键

```javascript
const hasOwn = Object.prototype.hasOwnProperty
function group(arr, fn) {
    // 不是数组
    if (!Array.isArray(arr)) {
        return arr
    }
    // 不是函数
    if (typeof fn !== 'function') {
        throw new TypeError('fn必须是一个函数')
    }
    var v
    return arr.reduce((obj, cur, index) => {
        v = fn(cur, index)
        if (!hasOwn.call(obj, v)) {
            obj[v] = []
        }
        obj[v].push(cur)
        return obj
    }, {})
}

// 按照长度分组
let result = group(['apple', 'pear', 'orange', 'peach'], (v) => v.length)
console.log(result)

// 按照份数分组
result = group(
    [
        {
            name: 'tom',
            score: 60
        },
        {
            name: 'Jim',
            score: 40
        },
        {
            name: 'Nick',
            score: 88
        }
    ],
    (v) => v.score >= 60
)
```
