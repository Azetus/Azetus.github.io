---
title: 'JavaScript: 组合与继承'
date: 2022-07-02 15:34:04
tags:
    - JavaScript
    - 组合与继承
categories:
    - JavaScript
---

# 1 深入浅出原型链

## 原型解决了什么问题

-   共享数据，减少空间占用，节省内存
-   实现继承

如果没有原型

```JavaScript
function Person(name, age) {
    this.name = name;
    this.age = age;
    this.getName = function () {
        return this.name;
    }
    this.getAge = function () {
        return this.age
    }
}

var person = new Person();
var person2 = new Person();

console.log(person.getName === person2.getName)
// false
```

## 原型三件套

-   prototype
-   constructor
-   \_\_proto\_\_

### prototype

-   无处不在
-   本质是一个普通对象

```JavaScript
var obj = {};
console.log(obj.toString === Object.prototype.toString);
// true
```

### constructor

-   实例的构造函数

```JavaScript
var obj = {};
console.log(obj.constructor === Object);
// true
```

### \_\_proto\_\_

-   \_\_proto\_\_属性是一个访问器属性（一个 getter 函数和一个 setter 函数）, 暴露了通过它访问的对象的内部[[Prototype]] (一个对象或 null)。
-   构造函数的原型 prototype
-   推荐使用：Object.getPrototypeof

```JavaScript
var obj = {};
console.log(obj.__proto__ === obj.constructor.prototype)
// true
console.log(Object.getPrototypeOf(obj) === obj.__proto__)
// true
```

### 谁都有谁

-   prototype 属性本质是一个普通对象
-   普通对象有\_\_proto\_\_属性
-   普通函数或者 class 既有 prototype 属性，又有\_\_proto\_\_属性

### 小结

-   prototype
    -   函数或者 class 的共享属性
    -   作用
        -   节约内存
        -   实现继承
    -   本质是一个普通的对象
-   constructor
    -   实例对象的构造函数
    -   可以被改写
    -   普通对象，其在原型上
-   \_\_proto\_\_
    -   构造函数的 prototype 原型
    -   Object.prototype.\_\_proto\_\_ === null
    -   基于此形成原型链

## 原型链

```JavaScript
function Person(name, age){
	this.name = name;
  this.age = age;
}

Person.prototype.getName = function(){
	return this.name;
}


Person.prototype.getAge = function (){
	return this.age
}

var person = new Person();
```

![6-6-1](./6-6-1.png)

### 原型链的尽头都是 null

Object.prototype.\_\_proto\_\_ === null

### 理解原型链

1. prototype 本质是一个普通对象
2. 函数.prototype.constructor 指向函数自身

![6-6-2](./6-6-2.png)

```JavaScript
function Person(name, age) {
    this.name = name;
    this.age = age;
}
Person.prototype.getName = function () {
    return this.name;
}
Person.prototype.getAge = function () {
    return this.age
}

var person = new Person();
```

![6-6-3](./6-6-3.png)

## 纯净对象

-   没有原型的对象
-   创建方法：`var obj = Object.create(null)`

优点：

-   空间上，少了原型链的信息，必然节约空间
-   时间上，没有原型链，查找一步到位

## 总结

-   函数最终的本质上是对象
-   普通对象都有 constructor，指向自己的构造函数，可以被改变，但不安全
-   函数和 class 的 prototype.constructor 指向函数自身
-   Function, Object, RegExp, Error 等本质是函数， Function.constructor === Function
-   普通对象都有\_\_proto\_\_，其等于构造函数的原型，推荐使用 Object.getPrototypeOf
-   所有普通函数的构造函数都是 Function, ES6 有出现的新函数种类 AsyncFunction，GeneratorFunction
-   原型链的尽头都是 null：Object.prototype.\_\_proto\_\_ === null
-   Function.\_\_proto\_\_ 指向 Function.prototype

## 小知识点

-   普通对象的两次\_\_proto\_\_是 null
-   普通函数的三次\_\_proto\_\_是 null
-   如果是经历过 n 次显式继承，被实例化的普通对象，n+3 层的\_\_proto\_\_是 null

---

# 2 组合和继承

## 组合 （has-a 关系）

-   在一个类/对象内使用其他的类/对象
-   has-a：包含关系，体现的是整体和部分的思想
-   黑盒复用：对象内部细节不可见，调用者只需要知道怎么使用

```javascript
class Logger {
    log() {
        console.log(...arguments)
    }
    error() {
        console.error(...arguments)
    }
}

class Reporter {
    constructor(logger) {
        this.logger = logger || new Logger()
    }
    report() {
        // TODO:
        this.logger.log('report')
    }
}

var reporter = new Reporter()
reporter.report()
```

### 组合的优点

-   功能相对独立，松耦合
-   扩展性好
-   复合单一职责，复用性好
-   支持动态组合，即程序运行中组合
-   具备按需组装的能力

### 组合的缺点

-   使用上相比继承，更加复杂一些
-   容易产生过多的类/对象

## 继承（is-a）关系

-   是 is-a 继承关系
-   白盒复用：需要了解父类的实现细节，从而决定怎么重写父类的方法

### 继承的优点

-   初始化简单，子类自动具备父类的能力
-   无需显式初始化父类

### 继承的缺点

-   继承层级多，会导致代码混乱，可读性变差
-   耦合紧
-   扩展性相对组合较差

## 组合和继承的最终目的

-   逻辑复用，代码复用

## 多态

-   事物在运行过程中存在不同的状态
-   多态形成条件
    -   需要有继承关系
    -   子类重写父类的方法
    -   父类指向子类

```Typescript
class Animal {
    eat(){
        console.log("Animal is eating")
    }
}

class Person extends Animal {
    eat(){
        console.log("Person is eating")
    }
}

var animal: Animal = new Animal();
animal.eat()

var person: Animal = new Person();
person.eat()
```

## 如何选择继承和组合

-   有多态需求的时候，考虑使用继承
-   如果有多重继承的需求，考虑使用组合
-   既有多态又有多重继承，考虑使用继承+组合

## ES5 中的继承方式

-   原型链继承
-   构造函数继承
-   原型式继承
-   组合继承
-   寄生式继承
-   寄生组合继承

### 寄生组合继承

借用构造函数来继承属性，通过原型链的方式来继承方法，而不需要为子类指定原型而调用父类的构造函数，我们需要拿到的仅仅是父类原型的一个副本。因此可以通过传入子类和父类的构造函数作为参数

1. 首先创建父类原型的一个复本
2. 并为其添加 constrcutor
3. 最后赋给子类的原型。

这样避免了调用两次父类的构造函数，为其创建多余的属性。

```javascript
function Animal(options) {
    this.age = options.age || 0
    this.sex = options.sex || 1
    this.testProperties = [1, 2, 3]
}

Animal.prototype.eat = function (something) {
    console.log('eat:', something)
}

function Person(options) {
    // 初始化父类, 独立各自的属性
    Animal.call(this, options)
    this.name = options.name || ''
}

// 设置原型
Person.prototype = Object.create(Animal.prototype)
// 修复构造函数
Person.prototype.constructor = Person

Person.prototype.eat = function eat(something) {
    console.log(this.name, ':is eating', something)
}
Person.prototype.walk = function walk() {
    console.log(this.name, ':is waking')
}

var person = new Person({ sex: 1, age: 18, name: '小红' })
person.eat('大米')
person.walk()

person.testProperties.push('4')

var person2 = new Person({ sex: 1, age: 18, name: '小红' })
console.log(person2.testProperties)
```

### 寄生组合继承解决的问题

-   各个实例的属性独立，不会发生修改一个实例，影响另外一个实例
-   实例化过程中没有多余的函数调用
-   原型上的 constructor 属性指向正确的构造函数

```javascript
function inheritPrototype(child, parent) {
    let prototype = Object.create(parent.prototype) // 创建对象
    prototype.constructor = child // 增强对象
    Child.prototype = prototype // 赋值对象
}
```

```javascript
function Parent(name) {
    this.name = name
    this.friends = ['rose', 'lily', 'tom']
}

Parent.prototype.sayName = function () {
    console.log(this.name)
}

function Child(name, age) {
    Parent.call(this, name)
    this.age = age
}

inheritPrototype(Child, Parent)
Child.prototype.sayAge = function () {
    console.log(this.age)
}
```

## 继承的一种变体

-   mixin：混入
-   把属性拷贝到原型，让其实例也有相应的属性

```javascript
class Logger {
    log() {
        console.log('Logger::', ...arguments)
    }
}

class Animal {
    eat() {
        console.log('Animal:: is eating')
    }
}

class Person extends Animal {
    walk() {
        console.log('Person:: is walking')
    }
}

const whiteList = ['constructor']
function mixin(targetProto, sourceProto) {
    const keys = Object.getOwnPropertyNames(sourceProto)
    keys.forEach((k) => {
        if (whiteList.indexOf(k) <= 0) {
            targetProto[k] = sourceProto[k]
        }
    })
}

mixin(Person.prototype, Logger.prototype)
```

## ES6 实现继承

-   extends 关键字
-   super 访问父类

```javascript
class Animal {
    constructor(options) {
        this.age = options.age || 0
        this.sex = options.sex || 1
    }

    eat(something) {
        console.log('eat:', something)
    }
}

class Person extends Animal {
    // 私有变量
    #friends = []

    constructor(options) {
        super(options)
        this.name = options.name || name
    }
    // 原型上的方法
    eat(something) {
        console.log(this.name, 'eat:', something)
    }
    run() {
        return `${this.name}正在跑步`
    }
    // 实例上的方法
    say = () => {
        console.log('say==', say)
    }
}
```

在原型上添加非方法的属性

```javascript
Animal.prototype.name = 'prototype的name'
```

### ES6 继承注意点

-   构造函数 this 使用前，必须先调用 super 方法
-   注意箭头函数形式的属性
-   class 若想在原型上添加非函数的属性，还得依赖 prototype
