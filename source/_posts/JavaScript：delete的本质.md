---
title: 'JavaScript: 运算符'
date: 2022-05-21 14:34:34
tags:
    - JavaScript
    - 运算符
categories:
    - JavaScript
---

# 关于赋值

## 引子

普通函数调用时，this 指向的是调用函数的对象，下面的四个操作会返回什么？

```JavaScript
let name = "let的name";
const person = {
  name: "person的Name",
  getName() {
    return this.name
  }
};

const getName = person.getName;

const print = function (prefix, ...args) {
  console.log(prefix.padEnd(20, " ") + ":", ...args)
}


print("person.getName", person.getName());
print("getName", getName());
print("(person.getName)", (person.getName)());
print("(0, person.getName)", (0, person.getName)());
```

```
person.getName      : person的Name
getName             : undefined
(person.getName)    : person的Name
(0, person.getName) : undefined
```

### person.getName()

-   this:person
-   结果：person 的 name

### getName()

-   this：全局对象
-   答案：undefined (let 和 const 声明的对象不会挂载全局对象)

### (person.getName)()

-   有没有赋值操作？
-   如果有的话赋值给了谁？

### 引用 Reference Record

> tc39 ecma262:The Reference Record type is used to explain the behaviour of such operators as delete, typeof, the assignment operators, the super keyword and other language features. For example, the left-hand operand of an assignment is expected to produce a Reference Record.  
> A Reference Record is a resolved name or property binding

-   被解析的命名绑定（resolved name binding）
-   内部引用类型不是语言数据类型
-   用于解释诸如 delete、typeof 和赋值等操作符的行为
-   例如，赋值的<font color=red>左操作数</font>应该产生一个引用记录

它由 4 部分组成

| Field Name     | Value                                                                | Meaning                                                                                                                                                                                                                                                                                                                 |
| -------------- | -------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Base           | an ECMAScript language value, an Environment Record, or unresolvable | The value or Environment Record which holds the binding. A [[Base]] of unresolvable indicates that the binding could not be resolved.                                                                                                                                                                                   |
| ReferencedName | a String, a Symbol, or a Private Name                                | The name of the binding. Always a String if [[Base]] value is an Environment Record.                                                                                                                                                                                                                                    |
| Strict         | a Boolean                                                            | true if the Reference Record originated in strict mode code, false otherwise.                                                                                                                                                                                                                                           |
| ThisValue      | an ECMAScript language value or empty                                | If not empty, the Reference Record represents a property binding that was expressed using the super keyword; it is called a Super Reference Record and its [[Base]] value will never be an Environment Record. In that case, the [[ThisValue]] field holds the this value at the time the Reference Record was created. |

-   其表示的是对某个变量、数组元素或对象属性所在内存地址的引用，而非对其所在内存地址所保存值的引用。在 ECMAScript 中，赋值运算符的左侧只能是一个内存地址，而不能是一个值。（你可以把值保存在某个内存地址，而不能把值赋给另一个值）

#### 举个栗子

-   `var Num =10`
-   `10 = 10`

### 引用相关的两个重要操作

-   GetValue(V)：即取值操作，返回的是确定的值
-   PutValue(V,W)：设置值，对某个引用设置
-   PutValue <font color=red>要求第一个参数是引用</font>

#### `v = v`

-   可以理解为：`v=GetValue(v)`
-   v 在左手端的时候，他是引用
-   而 v 在右手端的时候，他是值

#### 分组运算符()

-   分组运算符里面可以是表达式，也可是字面量的值
-   <font color=red>此算法不将 GetValue 应用于计算 Expression（表达式）的结果。这样做的主要原因是，诸如 delete 和 typeof 等操作符可以应用于括号表达式</font>（delete 的本质是删除引用）

### (person.getName)()

-   没有产生 getValue 操作，没有发生取值操作，也没有赋值行为
-   this：person
-   答案：person 的 Name

### (0, person.getName)()

-   分组运算符
-   <font color=red>逗号运算符</font>

#### 逗号运算符

-   MDN：对每个操作数<font color=red>求值</font>（从左到右），并返回最后一个操作数的值

产生了复制操作，等同于`(const getName = person.Name)()`

-   this：全局对象，window
-   答案：undefined

#### typeof 未声明变量为什么不报错（非严格模式）

-   引用不可达（unresolvable），直接返回 undefined

---

# delete 语法的本质

先看一个例子:

```JavaScript
console.log("delete null     :", delete null);
console.log("delete 11       :", delete 11);
console.log("delete undefined:", delete undefined);

a = { c: 12 };
console.log("delete a        :", delete a);

var b = 12;
console.log("delete b        :", delete b);

console.log("delete xxxxxxxxx:", delete xxxxxxxxx);

var obj = ({})
console.log("delete .toString:", delete obj.toString);
console.log("obj.toString:", obj.toString);
```

```
delete null     : true
delete 11       : true
delete undefined: false
delete a        : true
delete b        : false
delete xxxxxxxxx: true
delete .toString: true
obj.toString: [Function: toString]
```

## delete 的返回值是什么？

-   Boolean 类型
-   true，并不代表删除成功，代表<font color=red>删除操作没有发生异常</font>
-   false，一定没有删除成功

## delete 不能删除哪些属性

-   任何用 var 声明的属性，不能从全局作用域或者函数作用域删除
-   任何用 let 或者 const 声明的变量，不能从它声明的作用域删除
-   不可配置（configurable: false）的属性不能被删除

## delete 删除原型上的属性

-   delete 不会遍历原型链
-   但是`delete Foo.prototype.bar`可以删除

## delete 删除的到底是什么

1. Let ref be the result of evaluating UnaryExpression.
2. ReturnIfAbrupt(ref).
3. If ref is not a Reference Record, return true.
4. If IsUnresolvableReference(ref) is true, then  
   a. Assert: ref.[[Strict]] is false.  
   b. Return true.
5. If IsPropertyReference(ref) is true, then  
   a. Assert: IsPrivateReference(ref) is false.  
   b. If IsSuperReference(ref) is true, throw a ReferenceError exception.  
   c. Let baseObj be ? ToObject(ref.[[Base]]).  
   d. Let deleteStatus be ? baseObj.[[Delete]](ref.[[ReferencedName]]).  
   e. If deleteStatus is false and ref.[[Strict]] is true, throw a TypeError exception.  
   f. Return deleteStatus.
6. Else,  
   a. Let base be ref.[[Base]].  
   b. Assert: base is an Environment Record.  
   c. Return ? base.DeleteBinding(ref.[[ReferencedName]]).

-   如果是常量和字面量，直接返回 true
-   如果不可达（unResolvable），返回 true

## delete 语法的本质是什么

-   删除的是操作表达式结果
-   <font color=red>值，字面量</font> —— 不操作，直接返回 true
-   <font color=red>引用类型</font>，删除引用

> 为何用 true 表示没有异常而不是操作成功：历史遗留问题，在 delete 推出时 JavaScript 还没要 try...catch 方法

## 严格模式

非严格模式是安全的，但严格模式下有几种情况会抛出异常

-   SyntaxError：变量，函数名，函数参数
-   TypeError：configurable: false
-   ReferenceError：典型的就是 delete super.property
