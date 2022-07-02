---
title: JavaScript：搞懂对象的隐式类型转换
date: 2022-05-08 18:14:46
tags:
    - JavaScript
    - 对象
categories:
    - JavaScript
---

# 显式类型转换

-   显式转换：通过 JS 定义的转换方式进行转换
-   String, Object 等
-   parseInt, parseFloat 等
-   显式 toString

# 隐式转换

-   隐式转换：编译器自动完成类型转换的方式就称为隐式转换
-   总是<font color=red>期望</font>返回基本类型值

```
[] + ''
!'str'
```

# 什么时候会发生隐式类型转换

-   二元`+`运算符
-   关系运算符`>`, `<`, `>=`, `<=`, `==`
-   逻辑运算符`!`, `if/while`, 三目条件
-   属性键遍历，`for...in`等
-   模板字符串

> 预期的数据类型和传入的数据类型不一致的时候

一个例子：

```javascript
const obj = {
    value: 10,
    toString: function () {
        return this.value + 10
    },
    valueOf: function () {
        return this.value
    }
}

obj[obj] = obj.value

console.log('keys:', Object.keys(obj))
console.log('${obj}:', `${obj}`)
console.log('obj + 1:', obj + 1)
console.log('obj + "":', obj + '')
/**
    keys: [ '20', 'value', 'toString', 'valueOf' ]
    ${obj}: 20
    obj + 1: 11
    obj + "": 10
*/
```

# 对象隐式转换三大扛把子

-   `Symbol.toPrimitive`
-   `Object.prototype.valueOf`
-   `Object.prototype.toString`

# 对象隐式转换规则

-   如果`[Symbol.toPrimitive](hint)`存在，优先调用，无视`valueOf`和`toString`方法
-   否则，如果期望值是 <font color=red>"string"</font> —— 先调用`obj.toString()`如果返回不是原始值，继续调用`obj.valueOf()`
-   否则，如果期望值是<font color=red>"number"</font>或<font color=red>"default"</font> —— 先调用`obj.valueOf()`如果返回不是原始值，继续调用`obj.toString()`

## 一个问题

-   如果未定义`[Symbol.toPrimitive]`，期望值是"string",而`toString()`与`valueOf()`都没返回原始值？
    > 抛出异常

```javascript
const obj = {
    value: 10,
    valueOf() {
        return this
    },
    toString() {
        return this
    }
}
console.log(10 + obj)
// TypeError: Cannot convert object to primitive value
```

# `[Symbol.toPrimitive](hint)`

> hint 参数并非手动输入，而是引擎根据上下文自行判断（最初为 default，之后根据推断设定）

-   hint - "string"
-   hint - "number"
-   hint - "default"

## hint - "string"

-   window.alert(obj)
-   模板字符串<code>\`${obj}\`</code>
-   作为属性键`test[obj]=123`

```javascript
const obj = {
    [Symbol.toPrimitive](hint) {
        if (hint == 'number') {
            return 'hint is number'
        }
        if (hint == 'string') {
            return 'hint is string'
        }
        return true
    }
}
// alert, 浏览器
window.alert(obj) // hint is string
// ${}
console.log(`${obj}`) // hint is string
// 属性键
obj[obj] = 123
console.log(Object.keys(obj)) // hint is string
```

## hint - "number"

-   一元`+`，位移
-   `-`, `*`, `/`, 关系运算符
-   `Math.pow`, `String.prototype.slice`等很多内部方法

```javascript
const obj = {
    [Symbol.toPrimitive](hint) {
        if (hint == 'number') {
            return 10
        }
        if (hint == 'string') {
            return 'hello'
        }
        return true
    }
}

// 一元+
console.log('一元+:', +obj)

// 位移运算符
console.log('位移运算符', obj >> 0)

//除减算法, 没有 + 法，之前已经单独说过转换规则
console.log('减法:', 5 - obj)
console.log('乘法:', 5 * obj)
console.log('除法:', 5 / obj)

//逻辑 大于，小于，大于等于, 没有等于， 有自己的一套规则
console.log('大于:', 5 > obj)
console.log('大于等于:', 5 >= obj)

// 其他期望是整数的方法
console.log('Math.pow:', Math.pow(2, obj))
/**
    一元+: 10
    位移运算符 10
    减法: -5
    乘法: 50
    除法: 0.5
    大于: false
    大于等于: false
    Math.pow: 1024
*/
```

## hint - "default"

-   二元`+`
-   `==`, `!=`
    > 二元与非严格逻辑判断 String +/==/!= Number

```javascript
const obj = {
    [Symbol.toPrimitive](hint) {
        if (hint == 'number') {
            return 10
        }
        if (hint == 'string') {
            return 'hello'
        }
        // default
        return true
    }
}

console.log('相加:', 5 + obj) // 6
console.log('等等与:', 5 == obj) // false
console.log('不等于:', 5 != obj) // true
```

# 误区

-   两个对象`===`, `!==` 是否触发隐式转换
    > 不会，两个对象比较的是引用
-   两个对象`==`, `!=`是否触发隐式转换
    > <font color=red>不会</font>  
    > 只有一个对象与非对象比较时才会触发隐式转换

## toString 与 valueOf

如果期望值是"string" —— 先调用`obj.toString()`
如果期望值是"number" —— 先调用`obj.valueOf()`

```javascript
const user = {
    name: 'John',
    age: 10,
    toString() {
        return this.name
    },
    valueOf() {
        return this.age
    }
}

console.log('user:', +user) // 10 期望是数字
console.log('user:', `${user}`) // John 期望是字符串
```

### valueOf 不返回数字时

当`valueOf()`不再返回数字，继续调用`toString()`

```javascript
const user = {
    name: 'John',
    age: 10,
    toString() {
        return this.name
    }
    // valueOf方法不存在时表现相同
    valueOf() {
        // return this.age;
        return this;
    }
}

console.log('user:', +user) // user: NaN
```

### toString 不返回字符串时

1. 当对象存在 `toString()` 方法，但该方法不返回字符串时继续调用 `valueOf()`

```javascript
const user = {
    name: 'John',
    age: 10,
    toString() {
        // return this.name;
        return this
    },
    valueOf() {
        return this.age
    }
}

console.log('user:', `${user}`) // user: 10
```

2. 当对象不存在 `toString()` 方法时，执行 Object 原型上的 `Object.prototype.toString()`

```javascript
const user = {
    name: 'John',
    age: 10,

    valueOf() {
        return this.age
    }
}

console.log('user:', `${user}`) // [object Object]
```

3. 当对象及其原型上都不存在 `toString()` 方法时，继续调用 `valueOf()`

```javascript
const user = {
    name: 'John',
    age: 10,

    valueOf() {
        return this.age
    }
}
Object.prototype.toString = undefined
console.log('user:', `${user}`) // user: 10
```

# 特殊的 Date

-   hint 是<font color=red>default</font>，是优先调用 toString，然后再调用 valueOf

```javascript
const date = new Date()

console.log('date toString:', date.toString())
console.log('date valueOf:', date.valueOf())
// date toString: Sun May 08 2022 14:32:38 GMT+0800 (中国标准时间)
// date valueOf: 1651991558618

console.log(`date number:`, +date) // 期望是数字
console.log(`date str:`, `${date}`) // 期望是字符串
// date number: 1651991558618
// date str: Sun May 08 2022 14:32:38 GMT+0800 (中国标准时间)

console.log(`date +:`, date + 1) // 二元加法，hint 为 default
// date +: Sun May 08 2022 14:32:38 GMT+0800 (中国标准时间)1
```

# 总结

-   如果`[Symbol.toPrimitive](hint)`存在，优先调用，无视`valueOf`和`toString`方法
-   否则，如果期望值是 <font color=red>"string"</font> —— 先调用`obj.toString()`如果返回**不是原始值**，继续调用`obj.valueOf()`
-   否则，如果期望值是<font color=red>"number"</font>或<font color=red>"default"</font> —— 先调用`obj.valueOf()`如果返回**不是原始值**，继续调用`obj.toString()`

<table>
    <tr>
        <td>hint</td>
        <td>判断依据</td>
        <td>toPrimitive 不存在时</td>
    </tr>
    <tr>
        <td>String</td>
        <td><code>window.alert(obj)</code><br>模板字符串<code>`${obj}`</code><br>作为属性键<code>test[obj]=123</code></td>
        <td>先调用<code>obj.toString()</code>如果返回不是原始值，继续调用<code>obj.valueOf()</code></td>
    </tr>
    <tr>
        <td>Number</td>
        <td>一元<code>+</code>，位移<br><code>-</code>, <code>*</code>, <code>/</code>, 关系运算符<br><code>Math.pow</code>, <code>String.prototype.slice</code>等很多内部方法</td>
        <td rowspan='2'>先调用<code>obj.valueOf()</code>如果返回不是原始值，继续调用<code>obj.toString()</code></td>
    </tr>
    <tr>
        <td>default</td>
        <td>二元<code>+</code><br><code>==</code>, <code>!=</code> 二元非严格逻辑判断</td>
    </tr>
</table>

# 最开始的例子

```javascript
const obj = {
    value: 10,
    toString: function () {
        return this.value + 10
    },
    valueOf: function () {
        return this.value
    }
}

obj[obj] = obj.value
console.log('keys:', Object.keys(obj))
// 作为属性键 hint-String
// keys: [ '20', 'value', 'toString', 'valueOf' ]
console.log('${obj}:', `${obj}`)
// 模板字符串 hint-String
// ${obj}: 20
console.log('obj + 1:', obj + 1)
// 二元加法 hint-Default valueOf已有返回值 (-> toString)
// obj + 1: 11
console.log('obj + "":', obj + '')
// 二元加法 hint-Default valueOf已有返回值 (-> toString)
// obj + "": 10
```

## 例子 1

```javascript
const arr = [4, 10]
arr[Symbol.toPrimitive] = function (hint) {
    return hint
}
arr.valueOf = function () {
    return this
}

const obj = {}
```

```
+ arr + obj + arr + obj
"NaN[object Object]default[object Object]"
{} + arr
NaN
```

1. `+ arr`一元加法，hint 为 number（返回的 hint 是一个 string），因此`+ arr`等于`+"number"`，显然最终结果是`NaN`。
2. 然后计算`NaN + obj`，hint 为 default，先调用 valueOf 然后调用 toString，相当于`"NaN" + "[object Object]"`,最终结果为`"NaN[object Object]"`
3. `"NaN[object Object]" + arr`，hint 为 default，由于`arr[Symbol.toPrimitive]`返回 hint，等于`"NaN[object Object]" + "default"`
4. `"NaN[object Object]default" + {}`显而易见，最终结果为`"NaN[object Object]default[object Object]"`

## 例子 2

```javascript
const val = [] == ![]
console.log([+val, [] + 1] == [1, 1] + [])
console.log([+val, [] + 1] == [1, '1'])
```

1. `const val = [] == ![]`，将一个数组转换为布尔值，是真值，再对真值取反为`false`，`const val = [] == false`
2. 一个布尔值与其他值进行不严格等比较，两边转换为原始数据类型 Number，因此第一步转变为`const val = "" == 0`，最终转变为`const val = 0 == 0`，最后`val`被赋值为`true`
    > const val = true

-   对于`[+val, [] + 1] == [1, 1] + []`
    1. `+val`一元加法，hint 为 number，真值 true 转为数字 1
    2. `[] + 1`二元加法，hint 为 default，先 valueOf 并没有返回原始值，再调用 toString 返回空字符串，最终等于`""+1`
    3. 左侧数组为`[1, "1"]`
        > [1, "1"] == [1, 1] + []
    4. 右侧`[1, 1] + []`二元加法，hint 为 default，先 valueOf 然后 toString，最终等于`"1,1" + ""`
        > [1, "1"] == "1,1"
    5. 最终表达式为`[1, "1"] == "1,1"`，<font color=red>一边为引用类型另一边为基础类型</font> ，需要隐式转换，hint 为 default，先 valueOf 然后 toString，表达式转换为`"1,1" == "1,1"`
    6. 最终返回`true`
-   对于`[+val, [] + 1] == [1, '1']`
    1. 同上，左侧数组为`[1, "1"]`
        > `[1, "1"] == [1, "1"]`
    2. 请注意，<font color=red>这里两边均为引用类型，相当于`===`，两个引用类型时，比较的是引用</font>，因此返回`false`

## 例子 3

```javascript
const val = (+{} + [])[+[]]
console.log(val)
```

```
N
```

> `(+{} + [])[+[]]`

1.  `+{} => NaN`
2.  `(NaN + [])[+[]]`
3.  [] 隐式转换 ''
4.  `(NaN + '')[+[]]`
5.  NaN + '' => 'NaN'
6.  `('NaN')[+[]]`
7.  +[] => 0
8.  `('NaN')[0]`
9.  'N'

## 例子 4

```javascript
const obj = {},
    objA = { propertyA: 'A' },
    objB = { propertyB: 'B' }

obj[objA] = 'objectA'
obj[objB] = 'ObjectB'

for (let [k, v] of Object.entries(obj)) {
    console.log('k:', k, ', v:', v)
}
```

```
k: [object Object] , v: ObjectB
```

-   使用对象作为对象属性的键值，转换结果为字符串`[object Object]`
-   对象键的特性：本质上是字符串，如果是数字，用数字和数字字符串一致
-   Object.entries：迭代器，能获取键值对数组

```javascript
const obj = {},
    objA = {
        propertyA: 'A',
        toString() {
            return 'objA'
        }
    },
    objB = {
        propertyB: 'B',
        valueOf() {
            return 'objB'
        }
    }

obj[objA] = 'objectA'
obj[objB] = 'ObjectB'

for (let [p, v] of Object.entries(obj)) {
    console.log('p:', p, ', v:', v)
}
```

```
p: objA , v: objectA
p: [object Object] , v: ObjectB
```
