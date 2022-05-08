---
title: JavaScript：数据类型细节
date: 2022-05-08 12:55:28
tags:
    - JavaScript
    - 数据类型
categories:
    - JavaScript进阶
---

# 1 数据类型的陷阱

## 判断是不是 Object

```JavaScript
function isObject(obj) {
    if(typeof obj === 'object'){
        return true
    }
    return false
}
```

-   第一个问题：上面的方法有什么问题？

    > `typeof null` 返回的值也是`object`

-   第二个问题：为什么`typeof null` 返回的值是`object`

让我们追溯到 JavaScript 的第一个版本，此时一个值在栈中占用**32 个位的存储单元**，而其分为两个部分，第一部分为**3 位的标记位**剩余的位存储数据。此时 JavaScript 只有 5 种数据类型。

> 000: object  
> 001: interger  
> 010: double  
> 100: string  
> 110: boolean

而 null 是特殊的一类，其 32 位机器码全部为 0，所以，其标记位也自然全部为 0。

-   第三个问题：为什么不修复这个问题？

问题存在的位置过于底层，牵一发而动全身。

## 一元运算符+转为数字

```JavaScript
function toNumber(val) {
    return +val
  }
```

ES5 中没有问题，但是在 ES6 中会报错，因为 ES6 引入了`BigInt`与`Symbol`数据类型，二者无法被转变为`number`。

## 位转移为数字

```JavaScript
function toNumber(val){
    return val >> 0
}
function toNumber2(val){
    return val >>> 0
}
```

这个代码会有什么问题？当输入一个非常大的数的时候会返回什么？

```JavaScript
// 超大的数
toNumber(Number.MAX_SAFE_INTEGER)   // -1
toNumber2(Number.MAX_SAFE_INTEGER)  // 4294967295
```

为什么会这样？

### 无符号位移的情况

```JavaScript
// toNumber2(Number.MAX_SAFE_INTEGER)  => 4294967295
var val = Number.MAX_SAFE_INTEGER.toString(2)
// 11111111111111111111111111111111111111111111111111111
var val1 = val.substring(0,32)
// 11111111111111111111111111111111
var num = parseInt(val1, 2)
// 4294967295
```

当进行右移位运算的时候，会将数字当做一个 **32 位**整数进行计算的，当长度超出 **32 位时**会进行截取。

### 有符号位移的情况

> 有符号数字，最高位为符号位。

十进制变二进制：原码 -> 反码 -> 加一（补码）  
二进制变十进制：减一 -> 反码 -> 原码

```JavaScript
var val = Number.MAX_SAFE_INTEGER.toString(2)
// 11111111111111111111111111111111111111111111111111111

var val1 = val.substring(0,32)
// 11111111111111111111111111111111
```

截取前 32 位后减一

`11111111111111111111111111111110`

之后再取反码

`00000000000000000000000000000001 = 1`

最终等于 1，因为其最高位 1 是负数。

> 此外这种方法对于 BigInt 和 Symbol 也是无效的！

### 补充知识，关于 JavaScript 的补码

JavaScript 将数字存储为 64 位浮点数，但所有按位运算都以 32 位二进制数执行。在执行位运算之前，JavaScript 将数字转换为 32 位有符号整数。执行按位操作后，结果将转换回 64 位 JavaScript 数。

对于有符号的整数，JavaScript 为了方便底层识别和运算，二进制使用补码。正数的补码就是原码本身，而负数的反码为其原码除符号位以外的各位按位取反，并在末位加一。

> 正数：补码 = 原码；  
> 负数：补码 = 除符号位取反码+1;

以按位取反 `~` 运算符为例：

`~5 -> -6`

```
00000000000000000000000000000101 (5)
11111111111111111111111111111010 (~5 = -6)
11111111111111111111111111111011 (-5 即 ~5+1)
```

有符号整数使用最左边的位作为减号。

所以：
`~x` -> `-(x+1)`

例子：  
57 的二进制表示为(1 个字节):00111001  
按位取反后(~57)的二进制: 11000110 此表示为十进制:-70  
这是一个负数，是有符号的数，负数在计算机里要用其补码来表示:补码=符号位以后按位取反再加 1  
所以-70(11000110)符号位以后按位取反后为(10111001) 再加 1 则为(10111010)  
换成十进制为:-58  
因此~57=-58

总结：当进行别的运算的时候，也需要将所有原码转换成补码，正数的原码=补码，负数的补码为反码+1，反码为原码除符号位取反，之后进行相应运算，（&、|、~、^、移动等），然后反向求原码，可以得到最后结果。

> 按位取反小窍门：

`!~x` -> `-1` 返回 `true` 其他均为 `false`  
`!!~x` -> `-1` 返回 `false` 而其他均为 `true`

可以判断后端 api 返回的 errno 是否为-1

## 字符串批量转换为整数

```JavaScript
const arr = ["1", "2", "3"];
arr.map(parseInt)
```

结果是什么？

`[ 1, NaN, NaN ]`

为什么？

其实上面的代码相当于

```JavaScript
['1', '2', '3'].map((val, index) => parseInt(val, index))
```

`parseInt`的第二个参数是进制。取值范围`2 ~ 36`

## if 条件判断

```JavaScript
const result = {};
// name存在
if(obj.name){
    result.name = obj.name;
}
return result;
```

本质是转为布尔值。  
falsy : `0` , `false`, `""`, `null`, `undefined`, `NaN`

也会被转为 `false`

应该使用`Object.hasOwnProperty()`

## 宽松比较

```JavaScript
console.log(null == 0) // false
console.log('0' == false) // true
```

本质：隐式转换

-   NaN 和任何值都不等，包括它自己
-   bigInt，Symbol 首先比较是否同类型
-   null，undefined
-   布尔值与其他类型（布尔值 -> 数字）
-   数字类型和字符串类型（字符串 -> 数字）
-   对象类型和原始类型相比较（对象类型 -> 原始类型）
    -   对象类型与对象类型（比较引用）

## 杂谈

-   `typeof`性能比`instanceof`性能高 2 倍 （百万级别）
-   null 和 undefined 实现机制完全不一样
    -   null 是一个关键字，而 undefined 是一个变量
-   判断是不是数字，NaN

---

# 2 数据类型的 8 种判断方式

## 第一种：typeof

-   主要用途：操作数的类型，只能识别基础数据类型和引用类型
-   **特别注意**：null，NaN，document.all
    -   null -> Object
    -   NaN -> Number
    -   document.all -> undefind
-   注意事项：已经不是绝对安全（暂时性死区）

```JavaScript
function log(){
    typeof a
    let a = 10
}
log()// Cannot access 'a' before initialization
```

## 第二种：constructor

-   原理：constructor 指向创建实例对象的构造函数（然后 toString）
-   注意事项：
    -   null 和 undefined 没有构造函数
    -   constructor 可以被改写
-   并不安全

```JavaScript
String.prototype.constructor = function a() {
    return {}
}
console.log("a".constructor)
```

可以被改写，因此十分不安全，只能作为辅助手段。

## 第三种：instanceof

-   原理：在原型链上查找，查到即是其示例
-   注意事项：
    -   右操作数必须是函数或者 class
    -   **多全局对象，例如多 window 之间**，此对象非彼对象
    -   例：iframe

```html
<script>
    var frame = window.frames[0]
    var isInstanceOf = [] instanceof frame.Array
    console.log('frame.Array', frame.Array)
    console.log('isInstanceOf', isInstanceOf)
</script>
```

## 第四种：isPrototypeOf

-   原理：是否出现在实例对象的原型链上
-   注意事项：
    -   能正常返回值的情况，基本等同于 instanceof

## 第五种：Object.prototype.toString

-   原理：通过函数的动态 this 特效，返回其数据类型
-   自定义对象如何获得`[object MyArray]`类型
    -   Symbol.toStringTag
-   `Object.prototype.toString.call(Boolean.prototype)`会返回什么
    -   Object

## 第六种：鸭子类型检查

-   原理：检查自身，属性的类型或者执行结果的类型
-   例子：kindOf 与 p-is-promise
-   适用场景：候选方案

```JavaScript
function isPromise(value){
    return value instanceof Promise || (
        isObject(value) &&
        typeof value.then === 'function' &&
        typeof value.catch === 'function'
    )
}
```

## 第七种：Symbol.toStringTag

-   原理：Object.prototype.toString 会读取该值
-   适用场景：需自定义类型
-   注意事项：
    -   兼容性

```JavaScript
class MyArray {
    get [Symbol.toStringTag](){
        return "MyArray"
    }
}

var pf = console.log;
var a = new MyArray();
pf(Object.prototype.toString.call(a) )
```

## 第八种：等比较

-   原理：与某个固定值比较
-   适用场景：undefind、window、document、null

```JavaScript
// underscore源码
_.isUndefined = function(obj){
    return obj === void 0
}
```

## 总结

| 方法               | 基础数据类型 | 引用类型 | 注意事项                             |
| ------------------ | ------------ | -------- | ------------------------------------ |
| typeof             | √            | ×        | NaN, Object, document.all            |
| constructor        | 部分         | √        | 可以被改写                           |
| instanceof         | ×            | √        | 多窗口，右边必须是构造函数或者 class |
| isPrototypeOf      | ×            | √        | 小心 null 和 undefined               |
| toString           | √            | √        | 小心内置原型                         |
| 鸭子类型           | --           | √        | 不得已或者兼容时                     |
| Symbol.toStringTag | ×            | √        | 用于识别自定义对象                   |
| 等比较             | √            | √        | 特殊对象                             |

## 附：Redux kindOf.ts 源码

```JavaScript
export function miniKindOf(val: any): string {
  if (val === void 0) return 'undefined'
  if (val === null) return 'null'

  const type = typeof val
  switch (type) {
    case 'boolean':
    case 'string':
    case 'number':
    case 'symbol':
    case 'function': {
      return type
    }
  }

  if (Array.isArray(val)) return 'array'
  if (isDate(val)) return 'date'
  if (isError(val)) return 'error'

  const constructorName = ctorName(val)
  switch (constructorName) {
    case 'Symbol':
    case 'Promise':
    case 'WeakMap':
    case 'WeakSet':
    case 'Map':
    case 'Set':
      return constructorName
  }

  // other
  return Object.prototype.toString
    .call(val)
    .slice(8, -1)
    .toLowerCase()
    .replace(/\s/g, '')
}

function ctorName(val: any): string | null {
  return typeof val.constructor === 'function' ? val.constructor.name : null
}

function isError(val: any) {
  return (
    val instanceof Error ||
    (typeof val.message === 'string' &&
      val.constructor &&
      typeof val.constructor.stackTraceLimit === 'number')
  )
}

function isDate(val: any) {
  if (val instanceof Date) return true
  return (
    typeof val.toDateString === 'function' &&
    typeof val.getDate === 'function' &&
    typeof val.setDate === 'function'
  )
}

export function kindOf(val: any) {
  let typeOfVal: string = typeof val

  if (process.env.NODE_ENV !== 'production') {
    typeOfVal = miniKindOf(val)
  }

  return typeOfVal
}
```

---

# 3 ES6 中的 NaN

## NaN 和 Number.NaN

-   特点 1：typeof 返回的是数字
-   特点 2：自己与自己不相等
-   特点 3：不能被删除

```JavaScript
Object.getOwnPropertyDescriptor(window,"NaN")
// 结果
configurable: false
enumerable: false
value: NaN
writable: false
```

## isNaN()

-   isNaN：检查 toNumber 返回值，如果是 NaN 就返回 true，反之则返回 false

```JavaScript
// 语义化代码
const myIsNaN = function (val) {
    return Object.is(Number(val),NaN)
}
```

注意：并不是一个安全的方法，如果传入 bigInt 或者 Symbol 类型则会抛出错误

## ES6 的 Number.isNaN

-   Number.isNaN：判断一个值是否是数字，并且值等于 NaN

```JavaScript
// 语义化代码
Number.myIsNaN = function(val){
    if(typeof val !=="number"){
        return false
    }
    return Object.is(val,NaN)
}
```

## 严格判断 NaN 汇总

-   ES6 `Number.isNaN()`
-   自身比较（自身不等于自身）
-   Object.is
-   typeof + NaN

```JavaScript
function isNaNVal(val) {
    return Object.is(val, NaN);
}

function isNaNVal(val) {
    return val !== val;
}

function isNaNVal(val) {
    return typeof val === 'number' && isNaN(val)
}

// 综合垫片
if (!("isNaN" in Number)) {
    Number.isNaN = function(val) {
        return typeof val === 'number' && isNaN(val)
    }
}
```

## 通过陷阱看本质

1. 例子

```JavaScript
var arr = [NaN]
arr.indexOf(NaN) // -1
arr.includes(NaN) // true
```

includes：调用内部的`Number::sameValueZero`  
indexOf：调用内部的`Number::equal`

---

# 4 数值千分位的 6 种方法

对一个数值进行千分位，即`938765432.02 → 938,765,432.02`

## (1) 数值转字符串遍历

-   整体思路：数字转字符串，整数部分低位往高位遍历。

```
938765432.02
         ^
```

1. 数字转字符串，字符串按照`.`分割
2. 整数部分拆成字符串数组，并倒序
3. 遍历，按照每 3 位添加`,`号
4. 拼接整数部分+小数部分

```JavaScript
function format_with_array(number) {
    // 转为字符串，并按照.拆分
  var arr = (number + '').split('.');
    // 整数部分再拆分
  var int = arr[0].split('');
    // 小数部分
  var fraction = arr[1] || '';
    // 返回的变量
  var r = "";
  var len = int.length;
    // 倒叙并遍历
  int.reverse().forEach(function (v, i) {
        // 非第一位并且是位值是3的倍数， 添加 ","
      if (i !== 0 && i % 3 === 0) {
          r = v + "," + r;
      } else {
            // 正常添加字符
          r = v + r;
      }
  })
    // 整数部分和小数部分拼接
  return r + (!!fraction ? "." + fraction : '');
}

const print = console.log;
print(format_with_array(938765432.02));
// 938,765,432.02
```

## (2) 字符串+subString 截取

-   整体思路，数字转字符串，整数部分高位往低位遍历，三位分数

1. 数字转字符串，字符串按照`.`分割
2. 整数部分对 3 求模，获取多余部分
3. 按照 3 截取，添加`,`号
4. 拼接整数部分+小数部分

```JavaScript
function format_with_substring(number) {
    // 数字转字符串, 并按照 .分割
  var arr = (number + '').split('.');
  var int = arr[0] + '';
  var fraction = arr[1] || '';

  // 多余的位数
  var f = int.length % 3;
  // 获取多余的位数，f可能是0， 即r可能是空字符串
  var r = int.substring(0, f);
  // 每三位添加","和对应的字符
  for (var i = 0; i < Math.floor(int.length / 3); i++) {
      r += ',' + int.substring(f + i * 3, f + (i + 1) * 3)
  }

  //多余的位数，上面
  if (f === 0) {
      r = r.substring(1);
  }
  // 整数部分和小数部分拼接
  return r + (!!fraction ? "." + fraction : '');
}


const print = console.log;
print(format_with_substring(938765432.02));
```

## (3) 除法+求模

-   整体思路：求模的值添加`,`，求余值（是否大于 1）计算是否够结束

1. 值对 1000 求模，获得最高三位
2. 值除以 1000，值是否大于 1 判定是否结束
3. 重复 1 和 2 直到退出
4. 拼接整数部分+小数部分

```JavaScript
function format_with_mod(number) {
    var n = number;
    var r = "";
    var temp;
    do {
      	// 求模的值，用于获取高三位，这里可能有小数
        mod = n % 1000;
      	// 值是不是大于1，是继续的条件
        n = n / 1000;
      	// 高三位
        temp = ~~mod;
      	// 1. 填充 : n>1 循环未结束， 就要填充为比如，1 => 001,
        // 不然 1 001， 就会变成 '11',
      	// 2. 拼接 ","
        r =  (n >= 1 ?`${temp}`.padStart(3, "0"): temp) + (!!r ? "," + r : "")
    } while (n >= 1)

    var strNumber = number + "";
    var index = strNumber.indexOf(".");
  	// 拼接小数部分，
    if (index >= 0) {
        r += strNumber.substring(index);
    }
    return r;
}

const print = console.log;
print(format_with_mod(38765432.02));
```

## (4) 正则先行断言等

| 名字                   | 表达式          | 作用                      |
| ---------------------- | --------------- | ------------------------- |
| 先行断言（前瞻）       | `exp1(?=exp2)`  | 查找 exp2 前面的 exp1     |
| 后行断言（后顾）       | `(?<=exp2)exp1` | 查找 exp2 后面的 exp1     |
| 正向否定查找（负前瞻） | `exp1(?!exp2)`  | 查找后面不是 exp2 的 exp1 |
| 反向否定查找（负后顾） | `(?<!exp2)exp1` | 查找前面不是 exp2 的 exp1 |

```JavaScript
const print = console.log;
print(/hello (?=[a-z]+)/.test("hello a"));  // true
print(/hello (?=[a-z]+)/.test("hello 1"));  // false
```

数字 3 个一组，查找它前面 1-3 位的数字，`.replace(reg, '$&,')`匹配到的内容+","号。
replace 函数的第二个参数 newvalue 比较特殊，它有一下几个特殊字符串：

\$\$ 直接量符号(就是当做'$$'字符用)  
\$\& 与正则相匹配的字符串
\$` 匹配字符串左边的字符  
\$\’ 匹配字符串右边的字符  
\$1,\$2,\$3,…,\$n 匹配结果中对应的分组匹配结果

```JavaScript
function format_with_regex(number) {
    var reg = /\d{1,3}(?=(\d{3})+$)/g;
    return (number + '').replace(reg, '$&,');
}

// 987, 654, 321

const print = console.log;
print(format_with_regex(987654321));
```

`.replace(reg, callback)` 中的 callback
|变量名| 代表的值|
|---|---|
|match| 匹配的子串。（对应于上述的$&。）|
|p1,p2, ... |假如 replace()方法的第一个参数是一个 RegExp 对象，则代表第 n 个括号匹配的字符串。（对应于上述的$1，$2 等。）例如，如果是用 /(\a+)(\b+)/ 这个来匹配，p1 就是匹配的 \a+，p2 就是匹配的 \b+。|
|offset |匹配到的子字符串在原字符串中的偏移量。（比如，如果原字符串是 'abcd'，匹配到的子字符串是 'bc'，那么这个参数将会是 1）|
|string| 被匹配的原字符串。|
|NamedCaptureGroup| 命名捕获组匹配的对象|

精确的参数个数依赖于 replace() 的第一个参数是否是一个正则表达式（RegExp）对象，以及这个正则表达式中指定了多少个括号子串，如果这个正则表达式里使用了命名捕获， 还会添加一个命名捕获的对象

```JavaScript
function replacer(match, p1, p2, p3, offset, string) {
  // p1 is nondigits, p2 digits, and p3 non-alphanumerics
  return [p1, p2, p3].join(' - ');
}
var newString = 'abc12345#$*%'.replace(/([^\d]*)(\d*)([^\w]*)/, replacer);
console.log(newString);  // abc - 12345 - #$*%
```

```JavaScript
function format_with_regex(number) {
    var reg = /\d{1,3}(?=(\d{3})+$)/g;
    return (number + '').replace(reg, function(match, ...args){
      console.log(match, ...args);
      return match + ','
    });
}
format_with_regex(987654321)
// 987 321 0 987654321
// 654 321 3 987654321
```

```JavaScript
function format_with_regex(number) {
    var reg = /(\d)(?=(?:\d{3})+$)/g
    return (number + '').replace(reg, '$1,');
}
```

## (5) ECMA 规范

> `Intl.NumberFormat`

-   语法：`new Intl.NumberFormat([locales[,options]])`
-   `Intl.NumberFormat`是对语言敏感的格式化数字类的构造器类
-   基本功能：国际化的数字处理方案，它可以用来显示不同国家对数字的处理偏好。

```JavaScript
function format_with_Intl(number, minimumFractionDigits, maximumFractionDigits) {
    minimumFractionDigits = minimumFractionDigits || 2;
    maximumFractionDigits = (maximumFractionDigits || 2);
    maximumFractionDigits = Math.max(minimumFractionDigits, maximumFractionDigits);

    return new Intl.NumberFormat('en-us', {
        maximumFractionDigits: maximumFractionDigits || 2,
        minimumFractionDigits: minimumFractionDigits || 2
    }).format(number)
}


// 使用默认配置选项 性能更高
function format_with_Intl(number) {
    return new Intl.NumberFormat('en-us').format(number)
}


const print = console.log;
print(format_with_Intl(938765432.02));
```

## (6) toLocaleString

-   语法：`numObj.toLocaleString([locales[,options]])`
-   功能：其能把数字转为特定语言环境下的表示字符串

```JavaScript
function format_with_toLocaleString(number, minimumFractionDigits, maximumFractionDigits) {
    minimumFractionDigits = minimumFractionDigits || 2;
    maximumFractionDigits = (maximumFractionDigits || 2);
    maximumFractionDigits = Math.max(minimumFractionDigits, maximumFractionDigits);

    return number.toLocaleString("en-us", {
        maximumFractionDigits: maximumFractionDigits || 2,
        minimumFractionDigits: minimumFractionDigits || 2
    })
}

// 性能更高
function format_with_toLocaleString(number) {
    return number.toLocaleString("en-us")
}


const print = console.log;
print(format_with_toLocaleString(938765432.02));
```

## 性能对比

-   T0
    -   求模添加','，求余计算是够结束
-   T1
    -   subString
    -   正则
    -   字符串遍历
-   T2
    -   toLocaleString
    -   Intl.NumberFormat

---

# 5 [] + [], [] + {}, {} + [], {} + {}

## 本质： 二元操作符+规则

-   如果操作数是对象，则对象会转换为原始值
-   如果其中一个操作数是字符串的话，另一个操作数也会转换为字符串，进行字符串连接
-   否则，两个操作数都将转换成数字或 NaN，进行加法操作

## 对象转换为原始数据类型的值

-   `Symbol.ToPrimitive`
-   `Object.prototype.valueOf`
-   `Object.prototype.toString`

## []的原始值

-   `typeof [][Symbol.ToPrimitive]` undefined （没有这个方法）
-   `[].valueOf()` 返回 [ ]
-   `[].toString()` 返回 ''

## {}的原始值

-   `typeof {}[Symbol.ToPrimitive]` undefined （没有这个方法）
-   `{}.valueOf() 或 ({}).valueOf()` 返回 { }
-   `({}).toString()` 返回 '[object Object]'

## 问题的答案

1. [] + []  
   等同于`[].toString() + [].toString()`  
   等同于`'' + ''`  
   返回`''`
2. [] + {}  
   等同于`[].toString() + ({}).toString()`  
   等同于`'' + '[object Object]'`
   返回`'[object Object]'`
3. {} + []  
   **在浏览器中**  
   等同于`{}; + []`
   等同于` + []`  
   返回 0
4. {} + {}
    - **在 Chrome 中**
        - Chrome 花括号开头和结尾的语句会自动添加括号包裹
        - 等同于`({}) + ({})`
        - `'[object Object][object Object]' `
    - **在其他浏览器中**
        - 等同于`{}; + {}`
        - 等同于` + '[object Object]'`
        - 返回 NaN
