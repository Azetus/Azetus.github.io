---
title: JavaScript 原型与原型链
date: 2021-07-14 16:38:13
tags:
  - JavaScript
  - 原型链
categories:
  - JavaScript
---

# JavaScript 原型与原型链

### 1 继承

创建一个类(class)，包含一个属性`name`和一个方法`eat()`

```JavaScript
class People {
    constructor(name) {
        this.name = name
    }
    eat() {
        console.log(`${this.name} eat something`)
    }
}
```

利用继承的方法创建子类。

```JavaScript
// 子类
class Student extends People {
  constructor(name, number, age) {
    super(name);
    this.number = number;
    this.age = age;
  }
  sayHi() {
    console.log(`姓名 ${this.name} 学号 ${this.number}`);
  }
}

```

通过 new 创建一个类的对象/实例

```JavaScript
const monika = new Student('Monika', 100, 18)
```

### 2 原型

当使用了 new 之后，构造函数的执行会经历以下流程：

1. 开辟内存空间，在内存中创建一个新的空对象。
2. 将 `this` 指向这个空对象
3. 指向构造函数中的代码，为新对象添加属性与方法，如例子中通过`this.name = name`来赋值。
4. 返回这个新对象

> 通过 new Student 构建的新对象 monika

![img01](20210731141949.png)

- 所有的引用类型（数组、对象、函数），都有一个 **\_\_proto\_\_** 属性，属性值是一个普通的对象。**\_\_proto\_\_** 的含义是**_隐式原型_**。
- 所有的函数（不包括数组、对象），都有一个 **prototype** 属性，属性值是一个普通的对象。 **prototype** 的含义是**_显式原型_**。（实例没有这个属性）
- 所有的引用类型（数组、对象、函数），**\_\_proto\_\_** 属性指向它的对应的 class 的 **prototype** 值。

![img02](20210731143026.png)

基于原型的执行规则：

- 获取属性 `monika.name` 或执行方法 `monika.sayHi()` 时
- 现在自身的属性和方法中寻找
- 若未找到对应的属性或方法，则去 `__proto__` 中查找

### 3 原型链

如果在 **1** 中的代码添加

```JavaScript
monika.sayHi()
monika.eat()
```

会得到以下结果：

![img03](20210731155312.png)

上面的代码中，通过`monika.eat()`直接调用了 `eat()` 方法，这是因为它通过原型链，去`__proto__` 的 `__proto__` 里找到了 `People`，而 `People` 是有 `eat()`方法的。

> 当试图获取一个对象的某个属性时，如果这个对象本身没有这个属性，那么会去它的 `__proto__` 中寻找（即它的构造函数的`prototype`），并顺着原型链一直向上查找。

![img03](20210731152646.png)

### instanceof

`instanceof` 运算符用于检测构造函数的 `prototype` 属性是否出现在某个实例对象的原型链上。

![img04](20210731155723.png)

例如第三行的判断逻辑是：从 monika 的`__proto__`一层一层往上找，看能否对应到 `People.prototype`。
