---
title: 学习笔记：JavaScript 异步(2)
date: 2021-07-21 14:57:48
tags:
  - JavaScript
  - 异步
  - Promise
  - async/await
  - 宏任务、微任务
categories:
  - JavaScript
---

# 学习笔记：JavaScript 异步(2)

```JavaScript
console.log('start')

setTimeout(function callback() {
  console.log('callback')
}, 5000);

console.log('end')
//  start -> end -> callback
```

![img01](8-4.gif)

## async/await

- async/await 依旧是基于异步与 event loop
- async/await 本质是语法糖（Generator 的语法糖）

```JavaScript
async function async1() {
  console.log('async1 start'); // 2
  await async2();

  // await 的后面，都可以看做是 异步回调(callback) 的内容
  console.log('async1 end'); // 5
  await async3();

    // 下面是第二个异步回调的内容
    console.log('async1 end 2'); // 7
}

async function async2() {
  console.log('async2'); // 3
}

async function async3() {
  console.log('async3'); // 6
}

console.log('script start'); // 1
async1();
console.log('script end'); // 4
// 同步代码执行完 开始 event loop，执行await async2()后面的代码
```

![img02](20210802201325.png)

## 宏任务 macroTask 和微任务 microTask

### 什么是宏任务，什么是微任务

```JavaScript
console.log(100);
setTimeout(() => {
  console.log(200);
});
new Promise((resolve,reject) => {
  console.log(300)
  resolve()
}).then(() => {
  console.log(400);
});
console.log(500);
// 100 300 500 400 200
```

![img06](img001.png)

- 宏任务：setTimeout, setInterval, Ajax, DOM 事件
- 微任务：Promise async/await

> 微任务的执行时机比宏任务要早。  
> 任务的一般执行顺序：同步任务 -> 微任务 -> 宏任务。  
> 在执行一个 Promise 对象的时候，当走完 resolve();之后，就会立刻把 .then()里面的代码加入到微任务队列当中。

### 二者相互嵌套的情况下

#### 例 1

```JavaScript
new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve();
    console.log(100);
  }, 0);
  console.log(200);
}).then((res) => {
  // 微任务
  console.log(300);
});
setTimeout(() => {
  console.log(400);
});
console.log(500);
// 200 500 100 300 400
```
![img07](img002.png)

在执行宏任务的过程中，创建了一个微任务。但是需要先把当前这个宏任务执行完，再去轮询异步任务的队列，进而执行微任务。

#### 例 2

```JavaScript
new Promise((resolve, reject) => {
  setTimeout(() => { 
    // 第一个进入队列的宏任务
    resolve();
    console.log(1);
  }, 0);
  console.log(2);
}).then((res) => {
  // 微任务
  setTimeout(() => { 
    // 第三个进入队列的宏任务
    console.log(3);
  });
  console.log(4);
});
setTimeout(() => { 
  // 第二个进入队列的宏任务
  console.log(5);
});
console.log(6);
// 2 6 1 4 5 3
```
![img08](img003.png)

1.  首先浏览器执行 js 进入第一个宏任务进入主线程, 遇到 setTimeout(1) 分发到宏任务 Event Queue 中
2.  遇到 console.log(2) 直接执行，并输出 **2**
3.  遇到第二个 setTimeout(5) 分发到宏任务 Event Queue 中
4.  遇到 console.log(6) 直接执行，并输出 **6**
5.  从宏任务队列中取出第一个 setTimeout 并执行 console.log(1) 输出 **1**
6.  resolve 执行完毕，将 .then() 中的内容分发到微任务 Event Queue 中
7.  当前有微任务，执行微任务，将宏任务 第三个 setTimeout(3) 加入宏任务队列，执行 console.log(4) 输出 **4**
8.  从宏任务队列中取出第二个 setTimeout 执行 console.log(5) 输出 **5**
9.  从宏任务队列中取出第三个 setTimeout 执行 console.log(3) 输出 **3**

## event loop 和 DOM 渲染

- JS 是单线程的，而且和 DOM 渲染共用一个线程。

![img03](20210803135224.png)

- 每次 Call Stack 清空（即每次轮询结束），即同步任务执行完。
- 都是 DOM 重新渲染的机会，DOM 结构如有改变则重新渲染。
- 然后再去触发下一次 event loop

```HTML
<script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-1.9.0.min.js"></script>
<body>
  <div id="container"></div>
</body>
<script>
  const $p1 = $('<p>第一段文字</p>');
  const $p2 = $('<p>第二段文字</p>');
  const $p3 = $('<p>第三段文字</p>');

  $('#container').append($p1).append($p2).append($p3);

  console.log('length', $('#container').children().length);

  alert('本次 call stack 结束，DOM结构已更新，但尚未触发渲染');
</script>
// alert 会阻断 js 执行，也会阻断 DOM 渲染
```

> 控制台会先输出`length 3`然后弹窗显示 alert 内容，关闭弹窗后网页才会显示添加的 3 个 `<p>` 标签及其内容。

## 微任务和宏任务的区别

- 宏任务：DOM 渲染<font color='red'>后</font>触发，如 setTimeout
- 微任务：DOM 渲染<font color='red'>前</font>触发，如 Promise

若将上方代码修改为

```JavaScript
// 宏任务 DOM 渲染后触发
setTimeout(() => {
  console.log('length_2', $('#container').children().length); //3
  alert('setTimeout');
});

// 微任务 DOM 渲染前触发
Promise.resolve().then(() => {
  console.log('length_1', $('#container').children().length); //3
  alert('Promise then');
});
```

> 页面空白，控制台先输出`length_1 3`,然后 alert 弹窗显示 'Promise then'。  
> 页面显示出 3 个`<p>` 标签的内容。  
> 之后控制台输出`length_2 3`，最后 alert 弹窗显示 'setTimeout'。

![img04](20210803135541.png)

- 微任务是 ES6 语法规定的
- 宏任务是由浏览器规定的

## 总结

1. 执行全局代码，这些同步代码有一些是同步语句，有一些是异步语句（比如 setTimeout 等），异步代码移入队列中；
2. 全局代码执行完毕后，调用栈 (Call Stack) 会清空；
3. 从微队列 (microtask queue) 中取出位于队首的回调任务，放入调用栈 (Call Stack) 中执行，执行完后 (microtask queue) 长度减 1；
4. 继续取出位于微队列 (microtask queue) 队首的任务，放入调用栈 (Call Stack) 中执行，以此类推，直到把微队列 (microtask queue) 中的所有任务都执行完毕。**如果在执行微任务 (microtask) 的过程中，又产生了微任务 (microtask)，那么会加入到队列的末尾，也会在这个周期被调用执行，<font color='red'>直至微任务队列为空</font>**；
5. 微队列 (microtask queue) 中的所有任务都执行完毕，此时微队列 (microtask queue) 为空队列，调用栈 (Call Stack) 也为空；
6. 尝试触发 DOM 渲染，如果 DOM 结果有改变则触发 DOM 渲染；
7. 取出宏队列 (macrotask queue) 中位于队首的任务，放入 Stack 中执行；
8. **宏队列 (macrotask queue) <font color='red'>一次</font>只从队列中取<font color='red'>一个</font>任务执行，若产生新的宏任务，则加入到宏任务队列的队尾**；
9. 执行完毕后，调用栈 Stack 为空；
10. 重复第 3-9 个步骤；

```JavaScript
async function async1() {
  console.log('async1 start'); // 同步 -2-
  await async2(); // 同步
  console.log('async1 end'); // 第 1 个 微任务 -6-
}

async function async2() {
  console.log('async2'); // 同步 -3-
}

console.log('script start'); // 同步 -1-

setTimeout(function () {
  // 宏任务
  console.log('setTimeout'); // -8-
}, 0);

async1();

// 初始化 Promise 时，传入的函数会立即被执行
new Promise(function (resolve) {
  console.log('promise1'); // 同步 -4-
  resolve();
}).then(function () {
  console.log('promise2'); // 第 2 个 微任务 -7-
});

console.log('script end'); // 同步 -5-
// 1. 同步代码执行完毕（event loop - call stack 被清空）
// 2. 执行微任务 （微任务队列中所有的任务都会被依次取出来执行，直至微任务队列为空）
// 3. 尝试触发 DOM 渲染
// 4. 触发 event loop，执行宏任务 （宏队列一次只从队列中取一个任务执行，执行完后就去执行微任务队列中的任务）
```

![img05](20210803142340.png)
