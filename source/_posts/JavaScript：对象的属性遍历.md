---
title: JavaScript：对象的属性遍历
date: 2022-05-08 18:13:16
tags:
    - JavaScript
    - 对象
categories:
    - JavaScript进阶
---

# 属性的类型

-   普通属性
-   不可枚举的属性
-   原型属性
-   Symbol 属性
-   静态属性

```javascript
const symbolIsAnimal = Symbol.for('pro_symbol_attr_isAnimal')
const symbolSay = Symbol.for('pro_symbol_method_say')
const symbolSalary = Symbol.for('ins_symbol_attr_salary')

function Person(age, name) {
    this.ins_in_attr_age = age
    this.ins_in_attr_name = name

    this.ins_in_method_walk = function () {
        console.log('ins_method_walk')
    }
}

// 原型方法
Person.prototype.pro_method_say = function (words) {
    console.log('pro_method_say:', words)
}
Person.prototype[symbolSay] = function (words) {
    console.log('pro_symbol_method_say', words)
}

// 原型属性
Person.prototype[symbolIsAnimal] = true
Person.prototype.pro_attr_isAnimal = true

const person = new Person(100, '程序员')

//Symbol 属性
person[symbolSalary] = 6000
person['ins_no_enumerable_attr_sex'] = '男'

// sex 不可枚举
Object.defineProperty(person, 'ins_no_enumerable_attr_sex', {
    enumerable: false
})

Object.defineProperty(person, symbolSalary, {
    enumerable: false, // 无效的设置
    value: 999
})
```

## 小结

| 方法名                       | 普通属性 | 不可枚举属性 | Symbol 属性 | 原型属性 |
| ---------------------------- | -------- | ------------ | ----------- | -------- |
| for...in                     | √        | ×            | ×           | √        |
| Object.keys                  | √        | ×            | ×           | ×        |
| Object.getOwnPropertyNames   | √        | √            | ×           | ×        |
| Object.getOwnPropertySymbols | ×        | √            | √           | ×        |
| Reflect.ownKeys              | √        | √            | √           | ×        |

# 1 获取对象的全部静态属性

-   不要被静态属性误导
-   `Reflect.ownKeys` = `Object.getOwnPropertyNames` + `Object.getOwnPropertySymbols`

```javascript
let keys = Object.getOwnPropertyNames(_obj)
keys = keys.concat(Object.getOwnPropertySymbols(_obj))

const keys = Reflect.ownKeys(_obj)
```

# 2 获得原型上的所有属性

-   `for...in`? (拿不到不可枚举属性与 Symbol 属性)
-   递归
-   剔除内置属性

```JavaScript
class Grand {
    gName = "Grand";
    gGetName() {
        return this.gName;
    }
}
Grand.prototype[Symbol.for("gAge")] = "G-12";

class Parent extends Grand {
    pName = "123";
    pGetName() {
        return this.pName;
    }
}
Parent.prototype[Symbol.for("pAge")] = "G-11";

class Child extends Parent {
    cName = "123";
    cGetName() {
        return this.cName;
    }
}

const child = new Child();

// 递归
let result = [];
function logAllProperties(instance) {
    if (instance == null) return;
    let proto = instance.__proto__;
    while (proto) {
        result.push(...Reflect.ownKeys(proto));
        proto = proto.__proto__;
    }
}
logAllProperties(child);
console.log("result==", result);
```

# 3 获取所有不可枚举属性

-   如何知道某个属性不可枚举？
-   考不考虑原型上的不可枚举属性？
-   `getOwnPropertyDescriptor()` (会获得无关的描述符、开销大)
-   `Object.prototype.propertyIsEnumerable.call()` (推荐)

```JavaScript
function getNoEnumerable(_obj) {
    //获取原型对象
    const keys = Reflect.ownKeys(_obj);
    const result = keys.filter(key=> {
        return !Object.getOwnPropertyDescriptor(_obj, key).enumerable
    })
    return result;
}
```

```JavaScript
function getNoEnumerable(_obj) {
    //获取原型对象
    const keys = Reflect.ownKeys(_obj);
    const result = keys.filter(key=> {
        return !Object.prototype.propertyIsEnumerable.call(_obj, key)
    })
    return result;
}
```

# 栗子

```javascript
const proto = {
    name: 'p_parent',
    type: 'p_object',
    [Symbol.for('p_address')]: '地球'
}

const ins = Object.create(proto)
Object.defineProperty(ins, 'age', {
    value: 18
})
ins.sex = 1
ins[Symbol.for('say')] = function () {
    console.log('say')
}

const inKeys = []
for (let p in ins) {
    inKeys.push(p)
}

console.log(inKeys)
console.log(Reflect.ownKeys(ins))
```

```
[ 'sex', 'name', 'type' ]
[ 'age', 'sex', Symbol(say) ]
```

-   `for..in`可以遍历出原型属性，但无法遍历不可枚举的属性与 Symbol 属性
-   而`Reflect.ownKeys`无法遍历原型属性，但可以遍历不可枚举与 Symbol 属性
