---
title: 'JavaScript: 动态解析和执行函数'
date: 2022-07-02 15:37:39
tags:
    - JavaScript
    - 动态解析和执行函数
categories:
    - JavaScript
---

-   eval
-   new Function

# eval

-   功能：会将传入的字符串当做 JavaScript 代码进行执行
-   语法：eval(string)
-   基本使用`eval('2+2')`

## 使用场景

-   系统内部的 setTimeout 或者 setInterval
-   JSON 字符串转对象
-   前端模板
-   动态生成函数或者变量
-   有需要跳出严格模式的场景
-   其他场景

## setTimeout 和 setInterval

-   若参数为字符串则会尝试放入 eval 中执行

```JavaScript
setTimeout('console.log("setTimeout:", Date.now())', 1000)
setInterval('console.log("setInterval", Date.now())', 3000)
```

## JSON 字符串转对象

```JavaScript
var jsonStr = `{a:1, b:1}`;
var obj = eval('(' + jsonStr + ')' );
console.log("eval json:", obj, typeof obj);
```

## 生成函数

```javaScript
var sumAdd = eval(`(function add(num1, num2){
    return num1 + num2
}
)`)
console.log('sumAdd:', sumAdd(10,20))
```

## 数字数组相加

```javaScript
var arr = [1,2,3,7,9];
var r = eval(arr.join('+'))
console.log("数组相加:", r);
```

## 获取全局对象

```JavaScript
// 获取全局对象
// 逗号运算符 + 间接调用 => return eval("this")
var globalThis = (function(){ return (void 0,eval)("this")})();

console.log("globalThis:", globalThis)
```

## 注意事项

-   安全性（XSS 攻击）
-   调试困难（配合 debugger 关键字）
-   性能低
    -   如果一定要使用 eval，尽量提前将函数创建出来，然后再调用，而不是反复进行创建
-   不好把握
-   可读性维护性差

## eval 的直接调用和间接调用

直接调用

-   eval
-   (eval)
-   eval = window.eval <font color=red>（不能改名）</font>
-   {eval} = window
-   with({eval})

间接调用

-   除直接调用外的任何调用方式
    -   `(0, eval)('name')` 分组运算符
    -   `var eval2 = window.eval` 改名
    -   `{ eval: eval2 } = window` 解构改名

| 调用方式 | 作用域         | 是否严格模式 | this       |
| -------- | -------------- | ------------ | ---------- |
| 直接调用 | 正常的作用域链 | 继承当前     | 当前作用域 |
| 间接调用 | 只有全局作用域 | 非严格模式   | 全局       |

### 作用域链

```javaScript
var name = "全局的name";
function test() {
    var name = "local的name"
    // 直接调用
    console.log(eval(`name; debugger`));
    // 间接调用
    console.log(window.eval(`name`))
}

test();
```

```
local的name
全局的name
```

### this 指向

直接调用

```javaScript
var a = {
    show:function(){
      eval('console.log(this)');
   }
}
a.show();    //{show: ƒ}对象
```

间接调用

```javaScript
var a = {
    show:function(){
      var test = eval;
      test('console.log(this)');
   }
}
a.show();    //Window 对象
```

# new Function

-   作用：创建一个新的 Function
-   语法：`new Function([arg1 [, arg2[, ...argN]],] functionBody)`
-   基本使用：`new Function("a", "b", "return a + b")(10, 20)`

## 经典案例

-   webpack 的事件通知系统 tapable
-   fast-json-stringify

### tapable

> https://github.com/webpack/tapable/blob/master/lib/HookCodeFactory.js

```javaScript
// ......
case "sync":
    fn = new Function(
        this.args(),
        '"use strict";\n' +
            this.header() +
            this.contentWithInterceptors({
                onError: err => `throw ${err};\n`,
                onResult: result => `return ${result};\n`,
                resultReturns: true,
                onDone: () => "",
                rethrowIfPossible: true
            })
    );
    break;
case "async":
    fn = new Function(
        this.args({
            after: "_callback"
        }),
        '"use strict";\n' +
            this.header() +
            this.contentWithInterceptors({
                onError: err => `_callback(${err});\n`,
                onResult: result => `_callback(null, ${result});\n`,
                onDone: () => "_callback();\n"
            })
    );
    break;
// ......
```

### fast-json-stringify

-   需要使用者提前设置好 options
-   根据设置生成预设好的函数并调用

```javaScript
result = (new Function('schema', code))(root)
```

## 注意事项

-   new Function 基于全局环境创建
    -   this 默认也指向全局但可以改变
-   方法的 name 属性是'anonymous'

## 经典应用

-   获取全局 this
    ```javaScript
    var globalObj = new Function('return this')();
    ```
-   在线代码运行器
-   模板引擎

```html
<body>
    <div id="template">
        <div>名字:${name}</div>
        <div>年龄:${age}</div>
        <div>性别:${sex}</div>
        <div>性别:${c.b}</div>
        <div>商品:${products.join(",")}</div>
    </div>
    <script>
        function parse(source, data) {
            return new Function(
                'data',
                `
                with(data){
                    return \`${source}\`
                }
                `
            )(data)
        }
        const result = parse(template.innerHTML, {
            name: '帅哥',
            age: 18,
            sex: '男',
            products: ['杯子', '瓜子'],
            c: {
                b: '哈哈'
            }
        })
        template.innerHTML = result
    </script>
</body>
```

# eval VS new Function

<table>
    <tr>
        <th></th>
        <th>相同点</th>
        <th>不同点</th>
    </tr>
    <tr>
        <td>eval</td>
        <td rowspan="2">
        1. 动态编译和执行<br>
        2. 调试困难<br>
        3. 可读性可维护性差<br>
        4. 可以被禁用
        </td>
        <td>
        1. 直接调用和间接调用运行环境不同<br>
        2. 安全性低<br>
        3. 不仅仅局限于创建函数
        </td>
    </tr>
    <tr>
        <td>new Function</td>
        <td>1. 被创建于全局环境</td>
    </tr>
</table>

# 补充：创建一个函数，动态执行异步代码

```javaScript
var AsyncFunction = Object.getPrototypeOf(async function () { }).constructor;

function createAsyncFun(...args) {
    return new AsyncFunction(...args);
}

var fn = createAsyncFun(`
    const res = await fetch("/");
    console.log("res:");
    return res.text();
`)

fn().then((res) => {
    console.log("res:", res);
}).catch((err) => {
    console.log("err:", err);
})
```
