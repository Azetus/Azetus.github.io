---
title: JavaScript：深入对象
date: 2022-05-08 17:47:12
tags:
    - JavaScript
    - 对象
categories:
    - JavaScript进阶
---

# 1 对象的普通属性、排序属性和隐藏类

## 属性遍历

```JavaScript
var obj = {};

obj.p1 = "str1";
obj.p6 = "str6";
obj.p2 = "str2";

obj[1] = 'num1';
obj[6] = 'num6';
obj[2] = 'num2';

for (var p in obj) {
    console.log("property:", obj[p])
}
/**
    property: num1
    property: num2
    property: num6
    property: str1
    property: str6
    property: str2
*/
```

我们能发现什么？

-   两种属性：字符串作为键和数字作为键的属性
-   键被遍历的顺序似乎是有规律的

### 常规属性

-   键位字符串的属性
-   特点：<font color=red>**根据创建时的顺序排序**</font>

### 排序属性

-   属性键值为数字的属性
-   特点： <font color=red>**按照索引值大小升序排序**</font>

### 数字字符串属性是排序属性

```JavaScript
var obj = {};

obj['1'] = 'p1';
obj['6'] = 'p6';
obj['2'] = 'p2';

for (var p in obj) {
    console.log("property:", p)
}
/**
    property: 1
    property: 2
    property: 6
*/
```

可以类比字符串索引数组：

```JavaScript
var arr = [0, 1, 2, 3, 4];

console.log(arr["2"]);
console.log(arr["3"]);
console.log(arr["4"]);
```

### 为何设计常规属性和排序属性

-   提升属性的访问速度
-   两种线性数据结构保存  
    ![20220505203944](20220505203944.png)

-   elements -> 排序属性
-   properties -> 常规属性

V8 遍历对象时会先遍历**排序属性**，然后遍历**常规属性**。

### 对象内属性

-   何为对象内属性：被保存到对象自身的<font color=red>常规属性</font>
-   内属性的数量（10 个左右）
-   如何知道哪些属性时内属性？

1. 优先将常规属性转变为内属性
2. 当属性数量大于 10 时，将第 10 个之后的属性存入 properties，以下标顺序排序（线性数据结构）
3. 当常规数量再增多时，存入 properties 的属性变为非线性结构（慢属性）

### 对象属性的存储

排序属性 elements 有可能会被转变非线性结构储存。（哈希表）  
例：当排序只有 10 个时，对键为 10000 的属性进行赋值。

![20220505212434](20220505212434.png)

```JavaScript
foo[100000] = `${100000}-${Math.random()}`
```

![20220505212514](20220505212514.png)

## 隐藏类

-   什么是隐藏类
    -   描述了对象的属性布局
    -   属性名和对应的偏移量
-   隐藏类的作用
    -   从时间和空间两个维度提升速度

大多数的 Javascript 引擎会采用哈希表的方式来存取属性和寻找方法。而为了加快对象属性和方法在内存中的查找速度，V8 引擎引入了隐藏类(Hidden Class)的机制，起到给对象分组的作用。在初始化对象的时候，V8 引擎会创建一个隐藏类，随后在程序运行过程中每次增减属性，就会创建一个新的隐藏类或者查找之前已经创建好的隐藏类。每个隐藏类都会记录对应属性在内存中的偏移量，从而在后续再次调用的时候能更快地定位到其位置。

**动态添加和删除属性会导致隐藏类变动。**

### 如何保护隐藏类

-   初始化时保持属性顺序一致
-   一次性初始化完毕所有属性
-   谨慎使用`delete`

### 赋值 vs delete - 排序属性和普通属性

-   100000 个对象，25 个普通属性。delete 还是赋值快？
    -   赋值比 del 快一个数量级
-   100000 个对象，25 个排序属性。delete 还是赋值快？
    -   赋值比 del 快一倍左右

> 这是因为排序属性的访问比普通属性快。 一个可以通过偏移量计算访问，一个要 hash 计算访问。而每一次 delete 操作都会改变隐藏类，导致性能下降。

### 对象字面量

-   是创建对象的一种快捷方式（object literal)

```javascript
var obj = {
    name: 'myObject',
    id: 1
}
```

-   对应的还有：函数字面量、数组字面量等
-   **字面量的性能优于使用 new 构建**
    > 隐藏类：字面量创建对象会一次性构建隐藏类，而`new Object`创建时，需要通过`this.a = xxx; this.b = yyy`的形式创建，而每一次的动态赋值都会导致隐藏类变动。

---

# 2 属性来源，属性访问控制，属性冻结

## 属性来源

-   静态属性，例如：`Object.assign()` （不需要实例化即可使用）
-   原型属性，例如：`Object.prototype.toString()` （来自于原型链，不来自实例本身）
-   实例属性，例如：`function Person (name){ this.name = name }`

```javascript
function Person(name, age) {
    // 原型属性
    this.name = name
    this.age = age
    this.getName = function () {
        return name
    }
}
// 实例属性
Person.prototype.getAge = function () {
    return this.age
}

var person = new Person()
```

```javascript
class Person {
    constructor(name, age) {
        this.name = name
        this.age = age
    }
    // 实例属性
    getName = () => {
        return this.name
    }
    // 原型属性
    getAge() {
        return this.age
    }
}
const hasOwn = Object.hasOwnProperty
const print = console.log

var person = new Person()
print('getName:', hasOwn.call(person, 'getName')) // true
print('getAge:', hasOwn.call(person, 'getAge')) // false
```

## 属性描述符

```javascript
const obj = {}
Object.defineProperty(obj, 'name', {
    value: 'Azetus'
})

const des = Object.getOwnPropertyDescriptor(obj, 'name')

console.log('name:', des)
/**
    name: {
    value: 'Azetus',
    writable: false,
    enumerable: false,
    configurable: false
    }
*/
```

设置属性 `Object.defineProperty` `Object.defineProperties`

读取属性描述符 `Object.getOwnPropertyDescriptor` `Object.getOwnPropertyDescriptors`

-   configurable：可配置（是否可以删除，属性描述是否可以更改）
-   enumerable：是否可枚举（是否会出现在 for...in）
-   value：值
-   writable：是否可以被更改
-   get, set：访问器函数

    > 大体可以分为两种：

-   数据属性：value, writable, configurable, enumerable
-   访问器属性：get, set, configurable, enumerable

### 一个例子

先看一下代码

```javascript
const obj = {}

Object.defineProperty(obj, 'name', {})

obj.name = 1

console.log(obj.name) // undefind

console.log(Object.getOwnPropertyDescriptor(obj, 'name'))
```

`Object.defineProperty` 第三个参数传入空对象时，其默认值为

```javascript
{
  value: undefined,
  writable: false,
  enumerable: false,
  configurable: false
}
```

假如我们尝试修改描述符

#### case 01

```javascript
const obj = {}

Object.defineProperty(obj, 'name', {
    writable: true,
    configurable: false
})

// 尝试修改描述符信息
Object.defineProperty(obj, 'name', {
    writable: false
})

// 读取信息
console.log(Object.getOwnPropertyDescriptor(obj, 'name'))
/**
    {
    value: undefined,
    writable: false,
    enumerable: false,
    configurable: false
    }
*/
```

可以看到，在 configurable 为 false 时，我们成功的将 name 属性的 writable 从 true 修改成了 false。

> 根据 tc39 ecma262 的文档对 configurable 的描述：  
> If false, attempts to delete the property, change it from a data property to an accessor property or from an accessor property to a data property, or make any changes to its attributes <font color=red>(other than replacing an existing [[Value]] or setting [[Writable]] to false)</font> will fail.

如果为 false，尝试删除属性、将数据属性变为访问器属性或反之、以及对该属性描述符做任何更改将会报错。<font color=red>(除了变更属性的 value 或将 Writable 改为 false 之外)</font>

如果一个描述符不具有 value, writable, get 和 set 中的任意一个键，那么它将被认为是一个数据描述符。如果一个描述符同时拥有 value 或 writable 和 get 或 set 键，则会产生一个异常。

#### case 02

以下代码会报错

```javascript
const obj = {}

Object.defineProperty(obj, 'name', {
    configurable: false,
    writable: false
})

// 尝试修改描述符信息
Object.defineProperty(obj, 'name', {
    writable: true
})
// 会报错

// 读取信息
console.log(Object.getOwnPropertyDescriptor(obj, 'name'))
```

### Object.defineProperty 的缺点

-   无法监听数组变化
-   只能劫持对象的属性，如果属性也是一个对象，需要进行递归操作

### 对象的可扩展 - Object.preventExtensions

-   Object.preventExtensions：对象变得不可扩展，也就是永远不能再添加新的属性
-   Object.isExtensible：判断一个对象是否可以扩展

### 对象的封闭 - Object.seal

-   Object.seal：阻止添加新的属性 + <font color=red>属性标记为不可配置</font>
-   Object.isSealed：判断一个对象是否被封闭

### 对象的冻结 - Object.freeze

-   Object.freeze：不能添加新属性 + 不可配置 + 不能修改值
-   Object.isFrozen：判断一个对象是否被冻结

### 总结

| 方法                     | 新增属性 | 修改描述符            | 删除属性 | 更改属性值 |
| ------------------------ | -------- | --------------------- | -------- | ---------- |
| Object.preventExtensions | 不可     | 可                    | 可       | 可         |
| Object.seal              | 不可     | 不可(writable 有例外) | 不可     | 可         |
| Object.freeze            | 不可     | 不可(writable 有例外) | 不可     | 不可       |

---

# 3 访问对象的原型

## 1. prototype

-   prototype 是一个对象
-   原型会形成原型链，原型链上查找属性比较耗时，访问不存在的属性会访问整个原型链

### 两个问题

-   class 的 ES5 实现？

![img01](3-3-1.png)

-   toString 的怪异现象

```javascript
var proto = Boolean.prototype
console.log(typeof proto)
console.log(Object.prototype.toString.call(proto))
/**
    object
    [object Boolean]
*/
```

> Else if O has a [[BooleanData]] internal slot, let builtinTag be "Boolean".

## 2. \_\_proto\_\_

-   构造函数的原型
-   null 以外的对象均有\_\_proto\_\_属性
-   Function，class 的实例有 prototype 以及\_\_proto\_\_属性
-   **普通函数祖上三代必为 null**

### 两个问题

-   \_\_proto\_\_是实例对象的自身属性还是原型上的属性？
    -   **原型上的**
-   普通对象祖上第几代\_\_proto\_\_为 null
    -   **两代**

## 3. instanceof

-   检测构造函数的 prototype 属性是否出现在某个实例对象的原型链上
-   手写 instanceof
-   `Object instanceof Function`, `Function instanceof Object`

### 手写一个简单的 instanceof

```javascript
function instanceOf(instance, cclass) {
    let proto = instance.__proto__
    let prototype = cclass.prototype

    while (proto) {
        if (proto === prototype) return true
        proto = proto.__proto__
    }
    return false
}

class Parent {}
class Child extends Parent {}
class CChild extends Child {}
class Luren {}
var cchild = new CChild()
```

### `Object instanceof Function`, `Function instanceof Object`

二者均为 True

```javascript
const getType = (val) => Object.prototype.toString.call(val)
function getPrototypeChains(instance) {
    const chains = []
    let proto = instance.__proto__
    chains.push(getType(proto))
    while (proto) {
        proto = proto.__proto__
        chains.push(getType(proto))
    }
    return chains
}

const print = console.log
print(getPrototypeChains(Function))
print(getPrototypeChains(Object))
/**
    [ '[object Function]', '[object Object]', '[object Null]' ]
    [ '[object Function]', '[object Object]', '[object Null]' ]
*/
```

## 4. getPrototypeOf

-   返回对象的原型
-   `Object.getPrototypeOf`, `Reflect.getPrototypeOf`
-   内部先 toObject 转换，注意 null 和 undefined

## 5. setPrototypeOf

-   指定对象的原型
-   `Object.setPrototypeOf`, `Reflect.setPrototypeOf`
-   原型的尽头是 null

## 6. isPrototypeOf

-   一个对象是否存在于另一个对象的原型链上
-   `Object.isPrototypeOf`, `Object.prototype.isPrototypeOf`, `Function.isPrototypeOf`, `Reflect.isPrototypeOf`

```javascript
Object.isPrototypeOf({}) // false
Object.prototype.isPrototypeOf({}) // true
Reflect.isPrototypeOf({}) // false
Function.isPrototypeOf({}) // false
```

## 其他

-   `Object.create`

## 总结

![img02](3-3-2.png)

# 栗子

```javascript
let a = { n: 1 }
a.x = a = { n: 2 }

// 求a.x
console.log(a.x)
```

```
undefined
```

```javascript
let a = { n: 1 }
let b = a
a.x = a = { n: 2 }
console.log(a.x)
console.log(b)
```

```
undefined
{ n: 1, x: { n: 2 } }
```

-   JavaScript 连续赋值时从右向左执行
-   `.`的优先级比`=`高
-   对象属于引用类型

对于`a.x = a = { n: 2 }`

1. 先执行`a.x`，为堆内存中的对象新增了一个属性，键为 x 值为 undefined，此时`a.x`的部分指向堆内存中的对象`{n:1, x: undefined}`的属性 `x`
2. 连续赋值从右向左执行，`a = { n: 2 }`，a 指向的对象变为了堆内存中的`{ n: 2 }`
3. 继续向左执行，将此时 a 指向的对象`{ n: 2 }`，赋值给对象`{n:1, x: undefined}`的属性 `x`，对象变为`{ n: 1, x: { n: 2 } }`
4. 赋值完毕，此时 a 指向 `{ n: 2 }`，并没有属性 x，打印显示 undefined
5. b 指向的对象从始至终没有改变，`{ n: 1, x: { n: 2 } }`
