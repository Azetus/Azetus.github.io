---
title: 学习笔记：JavaScript 异步(1)
date: 2021-06-12 14:55:46
tags:
  - JavaScript
  - 异步
  - Promise
  - async/await
categories:
  - JavaScript学习笔记
---
# 学习笔记：JavaScript 异步(1)

## Event Loop (事件循环/事件轮询)

- JavaScript 是单线程运行的。
- 异步要基于回调来实现。
- event loop 就是异步回调的实现原理。

### JavaScript 如何执行

- 从前到后，逐行执行。
- 如果某一行报错，则停止下面代码的执行。
- **先执行完同步代码，再执行异步**。

```JavaScript
console.log('start')

setTimeout(function callback() {
  console.log('callback')
}, 5000);

console.log('end')
//  start -> end -> callback
```

![img01](8-4.gif)

1. 同步代码，一行一行放在 Call Stack 执行。
2. 遇到异步，会先“记录”下，等待时机（定时、网络请求等）。
3. 时机到了，移入到 Callback Queue 中。
4. 如果 Call Stack 为空（即同步代码执行完） Event Loop 开始工作。
5. 轮询查找 Callback Queue，如果有则移动到 Call Stack 中执行。
6. 然后继续轮询查找。

**DOM 事件和 event loop**

```HTML
<body>
  <button id="btn">点击</button>
</body>
<script >
  console.log('start');
  $('#btn').click(function cb(e) {
    console.log('btn clicked');
  });
  console.log('end');
</script>
```

> `$('#btn').click()`是*同步执行* 的。  
> 会进入 Callback Queue 的是回调函数 `cb`。  
> DOM 事件也使用回调，基于 event loop。

## Promise

### Promis 的三种状态

- **pending** 还在过程中
- **fulfilled** 已解决
- **rejected** 失败
- pending -> fulfilled 或 pending -> rejected
- 变化是不可逆的。

### 状态的表现

- pending 状态，不会触发`.then` 和`.catch`
- resolved 状态，会触发后续的 `.then`回调函数。
- rejected 状态，会触发后续`.catch`回调函数。

### then 和 catch 改变状态

- then 正常返回 resolved，里面有报错则返回 rejected
  rejected
- catch 正常返回 resolved，里面有报错则返回 rejected
  rejected

```JavaScript
const p1 = Promise.resolve().then(() => {
  return 100;
});

console.log('p1', p1); // fulfilled

const p2 = Promise.resolve().then(() => {
  throw new Error('then error');
});

console.log('p2', p2); // rejected

p2.then(() => {
  console.log('456'); // 不会执行
}).catch((err) => {
  console.error('err100', err); // err100 Error: then error
});
```
![img02](20210803152423.png)

```JavaScript
const p3 = Promise.reject('p3 error').catch((err) => {
  console.error(err); // p3 error
});

console.log('p3', p3); // fulfilled 触发 then 回调

p3.then(() => {
  console.log(300); // 300
});

const p4 = Promise.reject('p4 error').catch((err) => {
  throw new Error('catch err'); // catch err
});

console.log('p4', p4); // rejected 触发 catch 回调

p4.then(() => {
  console.log(400); // 不会执行
}).catch(() => {
  console.log('some error'); // 'some error'
});
```
![img03](2021080315.png)

## async/await

- Promise then catch 链式调用，但也是基于回调函数。
- async/await 用同步语法编写异步代码。*(不能改变异步的本质)*
- async 的返回值是 Promise 实例对象。
- await 可以得到异步结果。
- await 必须由 async 函数包裹。

```JavaScript
function loadImg(src) {
  return new Promise((resolve, reject) => {
    const img = document.createElement('img');
    img.onload = () => {
      resolve(img);
    };
    img.onerror = () => {
      const err = new Error(`加载失败 ${src}`);
      reject(err);
    };
    img.src = src;
  });
}

const src1 = './image1.png';
const src2 = './image2.PNG';

async function loadImg2() {
  const img2 = await loadImg(src2);
  return img2;
}

(async function () {
  const img1 = await loadImg(src1);
  console.log(img1.height, img1.width);
  const img2 = await loadImg2();
  console.log(img2.height, img2.width);
})();
``
```

## async/await 和 Promise 的关系

- 执行 async 函数，返回的是 Promise 对象。
- await 相当于 Promise 的 then。
- try...catch 可捕获异常，代替了 Promise 的 catch。

```JavaScript
async function fn1() {
  return 100; // 相当于 return Promise.resolve(100)
}
const res1 = fn1(); // 执行 async 函数，返回的是一个 Promise 对象
console.log('res1', res1); // Promise 对象
res1.then((data) => {
  console.log('data', data); // 100
});

(async function () {
  const p1 = Promise.resolve(200);
  const data1 = await p1; // await 相当于 Promise then
  console.log('data1', data1);
})();

(async function () {
  const data2 = await 300; // await Promise.resolve(400)
  console.log('data2', data2);
})();

(async function () {
  const data3 = await fn1(); // await Promise.resolve(400)
  console.log('data3', data3);
})();

(async function () {
  const p4 = Promise.reject('err 01'); // rejected 状态
  try {
    const res = await p4;
    console.log(res);
  } catch (err) {
    console.error(err); // try...catch 相当于 promise catch
  }
})();
```
![img04](20210803152006.png)

> await 相当于 Promise 中的 then

await 无法得到处于 rejected 状态的异步结果

```JavaScript
(async function () {
  const p5 = Promise.reject('err 02'); // rejected 状态
  const res5 = await p5; // await -> then 无法得到异步结果
  console.log('res5', res5);
})();
```
## for ... of

- for ... in （以及 forEach、 for）是常规的同步遍历
- for ... of 常用于异步的遍历
```JavaScript
function muti(num) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(num * num);
    }, 1000);
  });
}

const nums = [1, 2, 3];

nums.forEach(async (i) => {
  const res = await muti(i);
  console.log(res);
});
// 1 4 9会同时打印出来
// forEach 是同步执行，将 nums 遍历完后同步放入 muti 中，计时器在同一时刻开始计时

(async () => {
  for (let i of nums) {
    const res = await muti(i);
    console.log(res);
  }
})();
// 1 4 9 间隔1秒顺序出现，for...of 在异步中会依次顺序执行
```
