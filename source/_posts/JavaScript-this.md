---
title: 'JavaScript: this'
date: 2022-07-02 15:27:55
tags:
    - JavaScript
    - this
categories:
    - JavaScript
---

# 函数的 this

-   this
-   绑定规则
    -   默认绑定
        -   非严格模式
        -   严格模式
    -   隐式绑定
    -   显式绑定
    -   new
    -   箭头函数
-   绑定优先级
    1. 箭头函数
    2. 显式绑定
    3. new
    4. 隐式绑定
    5. 默认绑定
-   锁定 this 的方式
    -   bind
    -   箭头函数
    -   属性绑定符

# this

-   执行上下文（global、function、eval）的一个属性
-   在非严格模式下下，总是指向一个对象
-   在严格模式下可以是任意值

# this 绑定规则

-   默认绑定
    -   非严格模式
    -   严格模式
-   隐式绑定
-   显式绑定
-   new
-   箭头函数

## 默认绑定：非严格模式

浏览器：this 指向 window  
nodejs：指向 global 对象

## 默认绑定：严格模式

浏览器：undefined  
nodejs：undefined

## 隐式绑定

-   作为某个对象的属性被调用的时候

```javascript
person.name()
```

-   EventTarget
    -   addEventListener 绑定函数的 this 指向调用 addEventListener 的对象
-   FileReader
-   XMLHttpRequest

## 显式绑定

显示表达谁该是 this

-   call (参数依次传递)
-   apply (参数为数组)
-   bind
    -   bind 可以多次绑定，<font color=red>但只有第一次生效</font>

bind 可以传递参数

```javascript
function add(num1, num2, num3, num4) {
    return num1 + num2 + num3 + num4
}

const add2 = add.bind(null, 10, 20)

console.log(add2(30, 40)) // 100
```

非严格模式下的 call

-   非严格模式下 call 传入`null`和`undefined`等同于默认绑定

## <font color=red>属性绑定符</font>

函数绑定运算符是并排的两个冒号（::），双冒号左边是一个对象，右边是一个函数。该运算符会自动将左边的对象，作为上下文环境（即 this 对象），绑定到右边的函数上面

```javascript
foo::bar
// 等同于
bar.bind(foo)

foo::bar(...arguments)
// 等同于
bar.apply(foo, arguments)
```

如果双冒号左边为空，右边是一个对象的方法，则等于将该方法绑定在该对象上面。

```javascript
var method = obj::obj.foo
// 等同于
var method = ::obj.foo

let log = ::console.log
// 等同于
var log = console.log.bind(console)
```

如果双冒号运算符的运算结果，还是一个对象，就可以采用链式写法。

```javascript
// 连续使用
function getPerson() {
    return this.person
}

function getName() {
    return this.name
}

var obj = {
    person: {
        name: 'Tom'
    }
}

obj::getPerson()::getName()
```

## new

-   实例化一个函数或者 ES6 的 class
-   对于 Function，return 会影响返回值
    -   return 非对象，实际返回系统内部的对象
    -   return 对象，实际返回该对象

```javascript
function MyObject() {
    this.name = 'myObject'
}

function MyObject2() {
    this.name = 'myObject'
    return {
        name: 'myObject2'
    }
}

function MyObject3() {
    this.name = 'myObject3'
    return undefined
}

console.log(new MyObject())
console.log(new MyObject2())
console.log(new MyObject3())
```

补充：new 的本质

```javascript
function newObject(constructor) {
    var args = Array.prototype.slice.call(arguments, 1)
    var obj = {} // 1
    obj.__proto__ = constructor.prototype // 2
    var res = constructor.apply(obj, args) //3
    return res instanceof Object ? res : obj //4
}
```

## 箭头函数

-   没有自己的 this、arguments、super、new.target
-   使用自己所处的执行上下文的 this（继承自上层）
-   适合需要匿名函数的地方

### 一般情况

```javascript
var name = '全局的name'
var getName = () => this.name
// 全局

var person = {
    name: 'person的name',
    getName: () => this.name
}
// 全局

var person2 = {
    name: 'person2的name',
    getPerson() {
        return {
            getName: () => this.name
        }
    }
}
// person2
```

### 嵌套的情况下

```javascript
class Person {
    constructor(name) {
        this.name = name
    }

    getName() {
        return {
            getName2: () => ({
                getName3: () => ({
                    getName4: () => this.name
                })
            })
        }
    }
}

var p = new Person('person的name')
console.log(p.getName().getName2().getName3().getName4())
// person的name
```

### 上层 this 改变的情况

箭头函数的 this 继承自上层的 this，因此当上层的 this 改变时，箭头函数的 this 也会跟着一起改变

```javascript
// 浏览器中执行
var name = 'global.name'
var person = {
    name: 'person.name',
    getName() {
        return () => this.name
    }
}

console.log(person.getName()()) // person.name

console.log(person.getName.call({ name: 'name' })()) // name
```

# 绑定优先级

1. 箭头函数
2. 显式绑定
3. new
4. 隐式绑定
5. 默认绑定

# 锁定 this 的方法

-   bind
-   箭头函数
