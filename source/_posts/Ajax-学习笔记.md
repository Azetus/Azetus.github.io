---
title: Ajax 学习笔记
date: 2021-07-27 12:38:17
tags:
  - Ajax
  - 跨域
categories:
  - JS-Web-API学习笔记
---

# Ajax 核心 API

## 一个简易的 ajax

```JavaScript
function myAjax(url, successFn) {
  const xhr = new XMLHttpRequest();
  xhr.open('GET', url, true);
  xhr.onreadystatechange = function () {
    // 这里的函数异步执行，可参考之前JS基础中的异步模块
    if (xhr.readyState === 4) {
      if (xhr.status === 200) {
        successFn(xhr.responseText);
      }
    }
  };
  xhr.send(null);
}
```

> 使用 Promise 实现的情况

```JavaScript
function myAjax02(url) {
  const p = new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open('GET', url, true);
    xhr.onreadystatechange = function () {
      if (xhr.readyState === 4) {
        if (xhr.status === 200) {
          resolve(JSON.parse(xhr.responseText));
        } else if (xhr.status === 404) {
          reject(new Error('404 not found'));
        }
      }
    };
    xhr.send(null);
  });
  return p;
}

const url = 'xxxxxx'
myAjax02(url)
    .then(res => console.log(res))
    .catch(err => console.error(err))
```

## 知识点

- XMLHttpRequest
- 状态码
- 跨域：同源策略，跨域解决方案

## xhr.readyState

- 0 - (未初始化) 还没有调用 send()方法
- 1 - （载入）已调用 send()方法，正在发送请求
- 2 - （载入完成）send()方法执行完成，已经接收到全部响应内容
- 3 - （交互）正在解析响应内容
- 4 - （完成）响应内容解析完成，可以在客户端调用了

## xhr.status(http 状态码)

- 2xx - 成功处理请求，如 200
- 3xx - 需要重定向，浏览器直接转跳。如 301： 资源（网页等）被永久转移到其它 URL
- 4xx - 客户端请求错误，请求包含语法错误或无法完成请求。如 404：请求的资源（网页等）不存在
- 5xx - 服务器错误，如 500：内部服务器错误

## 常用工具

1. jQuery (适用于简单的请求，比如不存在链式调用的情况)
2. fetch (一个新的 API，要考虑兼容性)
3. axios

### fetch

默认返回 Promise

```JavaScript
fetch(url)
  .then(function (response) {
    return response.json();
  })
  .then(function (myJson) {
    console.log(myJson);
  });
```

> 当接收到一个代表错误的 HTTP 状态码时，从 fetch() 返回的 Promise 不会被标记为 reject， 即使响应的 HTTP 状态码是 404 或 500。相反，它会将 Promise 状态标记为 resolve （但是会将 resolve 的返回值的 ok 属性设置为 false ），仅当网络故障时或请求被阻止时，才会标记为 reject。  
> fetch() 可以不会接受跨域 cookies；你也可以使用 fetch() 建立起跨域会话。  
> fetch 不会发送 cookies。除非你使用了 credentials 的初始化选项。（自 2017 年 8 月 25 日以后，默认的 credentials 政策变更为 same-origin。Firefox 也在 61.0b13 版本中进行了修改）

---

# 跨域

## 知识点

- 什么是跨域（同源策略）

实现方式：

- JSONP
- CORS（服务端支持）

核心知识点：

- XMLHTTPRequest
- 状态码：readyState status
- 跨域：同源策略（如何绕过），JSONP，CORS

## 同源策略

- ajax 请求时，浏览器要求当前网页和 server 必须同源（安全）
- 同源：协议、域名、端口，三者必须一致
- 前端：`http://a.com:8080/`; server: `https://b.com/api/xxx`

**加载图片 css js 可无视同源策略**

- `<img />`可以用于统计打点，可使用第三方统计服务
- `<link />` `<script>` 可使用 CDN，CDN 一般都是外域
- `<script>` 可实现 JSONP

## 跨域

- 所有的跨域，都**必须**经过 server 端的允许和配合
- 未经 server 端允许就实现跨域，说明浏览器有漏洞，危险信号

## JSONP

- `<script>` 可以绕过跨域限制
- 服务器可以任意动态拼接数据返回
- 所以，`<script>`就可以获得跨域的数据，只要服务端愿意返回

```html
<script>
  window.abc = function (data) {
    console.log(data);
  };
</script>
<script src="./data/jsonp.js?callback=abc"></script>
```

```JavaScript
// jquery实现
$.ajax({
  url: './data/jsonp.js?callback=abc',
  dataType: 'jsonp',
  jsonpCallback: 'abc',
  success: function (data) {
    console.log(data);
  }
});
```

## CORS - 服务器设置 http header

- 需要服务端去做
