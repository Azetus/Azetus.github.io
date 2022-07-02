---
title: JavaScript：对象的拷贝
date: 2022-05-08 19:56:44
tags:
    - JavaScript
    - 对象
categories:
    - JavaScript
---

# 对象的浅拷贝

-   只克隆对象的第一层级
-   如果属性值时原始数据类型，拷贝其值，也就是我们常说的值拷贝
-   如果属性值是引用类型，拷贝其内存地址，也就是我们常说的引用拷贝

## 常用的浅拷贝

-   ES6 扩展运算符`{...obj}`
-   `Object.assign({}, obj)`
-   `for...in`和其他的一层遍历复制

## 数组常用的浅拷贝

-   ES6 扩展运算符`[...arr]`
-   `arr.slice(0)`
-   `[].concat(arr)`

# 深拷贝

-   克隆对象的每个层级
-   如果属性值是原始数据类型，拷贝其值，也就是我们常说的值拷贝
-   如果属性值是引用类型，递归克隆

# 方法 1 SON.parse + JSON.stringify

利用 JSON 对象方法

```javascript
function clone(obj) {
    return JSON.parse(JSON.stringify(obj))
}
```

-   优点：原生方法，简单易用
-   局限性：
    -   只能复制普通键的属性，Symbol 类型无能为力
    -   循环引用对象，比如 Window 不能复制
    -   特殊值类型，函数、Date、Rege、Blob 等类型不能复制
    -   性能差

# 方法 2 消息通讯

消息通讯 - BroadcastChannel 等

-   window.postMessage
-   BroadcastChannel
-   Shared Worker
-   Message Channel

```javascript
let chId = 0
function clone(data) {
    chId++
    var cname = `__clone__${chId}`
    var ch1 = new BroadcastChannel(cname)
    var ch2 = new BroadcastChannel(cname)
    return new Promise((resolve) => {
        ch2.addEventListener('message', (ev) => resolve(ev.data), { once: true })
        ch1.postMessage(data)
    })
}
```

-   局限性：
    -   循环引用对象，比如 Window 不能复制
    -   不能复制函数
    -   同步变成异步

# 方法 3 structuredClone()

> structuredClone(value)  
> structuredClone(value, { transfer })

参数:

-   value

    > 被克隆的对象。可以是任何结构化克隆支持的类型。

-   transfer 可选
    > 是一个 transferable objects (en-US) (可转移对象)的数组，里面的 值 并没有被克隆，而是被转移到被拷贝对象上。

局限性：

-   这是一个新的 API，从兼容性角度考虑，旧版浏览器不支持
-   Error 以及 Function 对象是不能被复制的，会抛出抛出的异常
-   DOM 节点同样无法复制
-   对象的某些特定参数也不会被保留
    -   RegExp 对象的 lastIndex 字段不会被保留
    -   属性描述符，setters 以及 getters（以及其他类似元数据的功能）同样不会被复制。例如，如果一个对象用属性描述符标记为 read-only，它将会被复制为 read-write，因为这是默认的情况下。
    -   原形链上的属性也不会被追踪以及复制。

# 方法 4

自己实现

1. 递归版本

```javascript
function deepClone(target, hash = new WeakMap()) {
    if (target === null) return target
    if (target instanceof Date) return new Date(target)
    if (target instanceof RegExp) return new RegExp(target)
    if (target instanceof HTMLElement) return target
    if (typeof target !== 'object') return target

    if (hash.get(target)) return hash.get(target)
    const cloneTarget = new target.constructor()
    hash.set(target, cloneTarget)

    if (target instanceof Map) {
        for (let [key, value] of target) {
            cloneTarget.set(key, deepClone(value, hash))
        }
        return cloneTarget
    }
    if (target instanceof Set) {
        for (let value of target) {
            cloneTarget.add(deepClone(value, hash))
        }
        return cloneTarget
    }

    Reflect.ownKeys(target).forEach((key) => {
        // 引入 Reflect.ownKeys，处理 Symbol 作为键名的情况
        cloneTarget[key] = deepClone(target[key], hash) // 递归拷贝每一层
    })
    return cloneTarget
}
```

2. 循环版本

```javascript
const { toString, hasOwnProperty } = Object.prototype

function hasOwnProp(obj, property) {
    return hasOwnProperty.call(obj, property)
}

function getType(obj) {
    return toString.call(obj).slice(8, -1).toLowerCase()
}

function isObject(obj) {
    return getType(obj) === 'object'
}

function isArray(arr) {
    return getType(arr) === 'array'
}

function isCloneObject(obj) {
    return isObject(obj) || isArray(obj)
}

function cloneDeep(x) {
    //使用WeakMap
    let uniqueData = new WeakMap()
    let root = x

    if (isArray(x)) {
        root = []
    } else if (isObject(x)) {
        root = {}
    }

    // 循环数组
    const loopList = [
        {
            parent: root,
            key: undefined,
            data: x
        }
    ]

    while (loopList.length) {
        // 深度优先
        const node = loopList.pop()
        const parent = node.parent
        const key = node.key
        const source = node.data

        // 初始化赋值目标，key为undefined则拷贝到父元素，否则拷贝到子元素
        let target = parent
        if (typeof key !== 'undefined') {
            target = parent[key] = isArray(source) ? [] : {}
        }

        // 复杂数据需要缓存操作
        if (isCloneObject(source)) {
            // 命中缓存，直接返回缓存数据
            let uniqueTarget = uniqueData.get(source)
            if (uniqueTarget) {
                parent[key] = uniqueTarget
                continue // 中断本次循环
            }

            // 未命中缓存，保存到缓存
            uniqueData.set(source, target)
        }

        if (isArray(source)) {
            for (let i = 0; i < source.length; i++) {
                if (isCloneObject(source[i])) {
                    // 下一次循环
                    loopList.push({
                        parent: target,
                        key: i,
                        data: source[i]
                    })
                } else {
                    target[i] = source[i]
                }
            }
        } else if (isObject(source)) {
            for (let k in source) {
                if (hasOwnProp(source, k)) {
                    if (isCloneObject(source[k])) {
                        // 下一次循环
                        loopList.push({
                            parent: target,
                            key: k,
                            data: source[k]
                        })
                    } else {
                        target[k] = source[k]
                    }
                }
            }
        }
    }

    uniqueData = null
    return root
}
```

# 方法 5

引入 lodash 等工具库
