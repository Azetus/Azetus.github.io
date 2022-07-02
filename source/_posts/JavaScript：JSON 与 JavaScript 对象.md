---
title: JavaScript：JSON 与 JavaScript 对象
date: 2022-05-08 18:44:42
tags:
    - JavaScript
    - 对象
    - JSON
categories:
    - JavaScript
---

# JSON 对象

-   严格意义上 JSON 是一种文本协议
-   而 JS 环境中全局作用域下，指名为 JSON 的 Object 对象

以下对象时 JSON 对象么？

> 不是，这是一个对象字面量，JSON 与 JS 对象不同，JSON 键值只能是字符串

```javascript
var obj = {
    name: 'tom',
    [Symbol.for('sex')]: 1
}
```

# JSON 格式

-   JSON 是一种轻量级的、<font color=red>基于文本的</font>、与语言无关的语法，用于定义数据交换格式
-   他来源于 ECMAScript，但是独立于编程语言

# JSON 特征

-   JSON 本质就是一串<font color=red>字符串</font>，使用特定的符号标注
-   `{}`表示对象
-   `[]`表示数组
-   `""`双引号内是属性键或值

## JSON 键

-   只能是字符串
-   **必须**用双引号包裹

## JSON 值

只能包括以下值

1. Object
2. Array
3. Number (只能是十进制)
4. String
5. true
6. false
7. null

# JSON.parse()

> `JSON.parse(text[, reviver])`

-   第二个参数函数(可选) `reviver(key, value)`
-   key -> 当前转换的属性键
-   value -> 当前转换的属性值
-   如果 reviver 返回 undefined，则当前属性会从所属对象中删除，如果返回了其他值，则返回的值会成为当前属性新的属性值
    > 可以用来过滤数据，拦截一些不想通过网络传输或存储的数据。
-   遍历顺序：从最最里层的属性开始，一级级往外，最终到达顶层，也就是解析值本身
    > 对于深层对象，遇到子对象时，先遍历解析子对象的属性，再解析子对象自身  
    > 当遍历到最顶层的值（解析值）时，传入 reviver 函数的 key 参数会是空字符串 "" 和当前的解析值（有可能已经被修改过了），当前的 this 值会是 {"": 修改过的解析值}
-   this：在调用过程中，当前属性所属的对象会作为 this 值

```javascript
const jsonStr = `{
    "1": 1,
    "2": 2,
    "3": {
        "4": 4,
        "5": 5,
        "6": {
            "7": 7
        }
    }
}`
JSON.parse(jsonStr, function (k, v) {
    console.log(k)
    return v
})
```

> 输出顺序为：1 2 4 5 7 6 3 ""

```javascript
JSON.parse(obj, function (k, v) {
    console.log('key:', k, ',this:', this)
    return v
})
```

```
"key:" "1" ",this:" Object { 1: 1, 2: 2, 3: Object { 4: 4, 5: 5, 6: Object { 7: 7 } } }
"key:" "2" ",this:" Object { 1: 1, 2: 2, 3: Object { 4: 4, 5: 5, 6: Object { 7: 7 } } }
"key:" "4" ",this:" Object { 4: 4, 5: 5, 6: Object { 7: 7 } }
"key:" "5" ",this:" Object { 4: 4, 5: 5, 6: Object { 7: 7 } }
"key:" "7" ",this:" Object { 7: 7 }
"key:" "6" ",this:" Object { 4: 4, 5: 5, 6: Object { 7: 7 } }
"key:" "3" ",this:" Object { 1: 1, 2: 2, 3: Object { 4: 4, 5: 5, 6: Object { 7: 7 } } }
"key:" "" ",this:" Object { : Object { 1: 1, 2: 2, 3: Object { 4: 4, 5: 5, 6: Object { 7: 7 } } } }
```

# JSON.stringify()

> `JSON.stringify(value[, replacer [, space]])`

-   第二个参数(可选) replacer：过滤属性或处理值
    > replacer?: (this: any, key: string, value: any) => any  
    > replacer?: (number | string)[] | null
-   第三个参数(可选) space：美化输出格式
    > space?: string | number | null

以如下对象为例：

```javascript
var person = {
    name: 'Azetus',
    age: 27,
    birth: '1995'
}
```

## 第二个参数 replacer

-   如果该参数是一个函数：则在序列化过程中，被序列化的值的每个属性都会经过该函数的转换和处理；
-   如果改参数是一个数组，则只有包含在这个数组中的属性名才会被序列化到最终的 JSON 字符串中；
-   如果该参数为 null 或者未提供，则对象所有的属性都会被序列化。

### 传入函数

```javascript
var jsonString = JSON.stringify(person, function (key, value) {
    if (typeof value === 'string') {
        return undefined
    }
    return value
})
console.log(jsonString)
```

```
{"age":27}
```

### 传入数组

```javascript
console.log(JSON.stringify(person, ['name', 'age']))
```

```
{"name":"Azetus","age":27}
```

## 第三个参数 space

-   如果参数是个数字：它代表有多少的空格；上限为 10，该值若小于 1，则意味着没有空格；
-   如果该参数为字符串（当字符串长度超过 10 个字母，取其前 10 个字母），该字符串将被作为空格；
-   如果该参数没有提供（或者为 null），将没有空格。

```javascript
const a = JSON.stringify(person)
console.log(a)
```

```
{"name":"Azetus","age":27,"birth":"1995"}
```

```javascript
const c = JSON.stringify(person, null, '\t')
console.log(c)
```

```
{
	"name": "Azetus",
	"age": 27,
	"birth": "1995"
}
```

## 转换规则 - undefined, Symbol, function

-   作为对象属性值，自动忽略
-   作为数组，序列化返回 null
-   单独序列化时，返回 undefined

## 其他转换规则

-   Date 返回 ISO 字符串
-   循环引用会报错
-   NaN, Infinity, null 都会作为 null
-   BigInt 报错
-   Map, Set, WeakMap 等对象，仅序列化可枚举属性

## toJSON

如果对象拥有`toJSON`方法，`toJSON`会覆盖对象默认的序列化行为

```javascript
var product = {
    name: '牙膏',
    count: 10,
    orderDetail: {
        createTime: 1632996519781,
        orderId: 8632996519781
    },
    toJSON() {
        return {
            name: '牙膏'
        }
    }
}

console.log(JSON.stringify(product))
```

```
'{"name":"牙膏"}'
```
