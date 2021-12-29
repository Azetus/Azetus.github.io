---
title: HTTP 学习笔记
date: 2021-08-01 12:52:47
tags:
  - http
  - http状态码
  - 缓存
  - http methods
categories:
  - http
---

# http 知识点

- http 状态码
- http method
- Restful API
- http headers
- http 缓存策略

---

# http 状态码

- 状态码分类
- 常见状态码
- 关于协议和规范

## 状态码分类

- `1xx` 服务器收到请求
- `2xx` 请求成功，如 200
- `3xx` 重定向，如 302
- `4xx` 客户端错误，如 404
- `5xx` 服务端错误，如 500

## 常见状态码

- `200` 成功
- `301` 永久重定向（配合 location，浏览器自动处理，会记住该地址后续自动重定向，例如域名更换的情况）
- `302` 临时重定向（配合 location，浏览器自动处理，再次访问时仍会访问原地址，例如搜索引擎点击搜索结果转跳时）
- `304` 资源未被修改（例：服务器端的资源与上次请求时发送的相同）
- `404` 资源未找到（url 错误）
- `403` 没有权限（例：未登录）
- `500` 服务器错误（例：服务器内部错误等）
- `504` 网关超时（例：访问的服务器与上游服务器通讯时未得到及时响应）

## 关于协议和规范

- 是一个约定
- 要求大家遵守执行

---

# http methods

- 传统的 methods
- 现在的 methods
- Restful API

## 传统的 methods

- `get` 获取服务器数据
- `post` 向服务器提交数据
- 简单的网页功能，就这两个操作

## 现在的 methods

- `get` 获取数据
- `post` **新建**数据
- `patch`/`put` 更新数据 （局部更新/更新整个资源）
- `delete` 删除数据

## Restful API

- 一种新的 API 设计方法（早已推广使用）
- 传统 API 设计：把每个 url 当做一个功能
- Restful API 设计：把每个 url 当做一个唯一的资源

### 如何设计成一个资源？

- 尽量不用 url 参数
- 用 method 表示操作类型

#### 不使用 url 参数

- 传统 API 设计：`/api/list?pageIndex=2`
- Restful API 设计: `/api/list/2`

#### 用 method 表示操作类型

传统 API 设计:

- 新建：`post` 请求 `/api/create-blog`
- 更新：`post` 请求 `/api/update-blog?id=100`
- 获取：`get` 请求 `/api/get-blog?id=100`

Restful API 设计：

- 新建：`post` 请求 `/api/blog`
- 更新：`patch` 请求 `/api/blog/100`
- 获取：`get` 请求 `/api/blog/100`

---

# http headers

- 常见的 Request Headers
- 常见的 Response Headers

## Request Headers

- `Accept` 浏览器可接收的数据格式
- `Accept-Encoding` 浏览器可接收的压缩算法，如 gzip
- `Accept-Language` 浏览器可以接收，如 zh-CN
- `Connection:keep-alive` 一次 TCP 连接重复使用
- `cookie`
- `Host` 请求的域名
- `User-Agent` (UA) 浏览器信息
- `Content-type` 发送数据的格式，如 `application/json`

## Response Headers

- `Content-type` 返回的数据格式，如 `application/json`
- `Content-length` 返回的数据大小，多少字节
- `Content-Encoding` 返回数据的压缩算法，如 gzip
- `Set-cookie` 更改 cookie

## 自定义 Headers

- 自定义 Request Headers 在前端向后端发送请求时添加
- 自定义 Response Headers 在后端返回时由后端添加

## 缓存相关的 Headers

- `Cache-Control` `Expires`
- `Last-Modified` `If-Modified-Since`
- `Etag` `if-None-Match`

---

# http 缓存

- 关于缓存的介绍
- **http 缓存策略（强制缓存+协商缓存）**
- 刷新操作方式，对缓存的影响

## 关于缓存

- 什么是缓存？
  > 一些没有必要重复获取的资源，不再重新获取（本地或服务端暂存）
- 为什么需要缓存？
  > CPU 计算、页面渲染都是很快的，整个环节中较慢的环节是网络请求，同时网络请求也是不稳定的。（信号，网络速度）  
  > 优化是需要尽量减少网络请求的体积和数量。（木桶效应）
- 哪些资源可以被缓存？ **静态资源(js css img)**
  > html 一般是不能缓存的，因为随时可能更新。  
  > 业务数据一般也是不能缓存的，例如博客文章，评论信息。  
  > 联动：webpack 打包静态资后生成的 hash 值

---

# http 缓存 - 强制缓存

## Cache-Control

- 在 Response Headers 中
- 控制强制缓存的逻辑
- 例如 `Cache-Control: max-age = 3153600`(单位是秒)

![http_image_01](http_image_01.png)

> 浏览器初次向服务端请求资源时，如果服务端判断该资源可以使用缓存存储则会在 Response Header 中添加 Cache-Control。  
> 当浏览器再次请求该资源时，则从缓存中直接查找。（若未失效的情况下）  
> 若缓存已失效，则再次向浏览器请求。

## Cache-Control 的值

- max-age
- no-cache (不用强制缓存，交给服务端处理，缓存时需要进行校验)
- no-store (不使用缓存，只从服务器获取资源)
- private (只允许最终用户做缓存)
- public (允许中间路由和代理做缓存)

## 关于 Expires

- 同在 Response Headers 中
- 同为控制缓存过期
- 已被 Cache-Control 代替

---

# http 缓存 - 协商缓存 (对比缓存)

- 服务器端缓存策略
- 服务器判断客户端缓存资源，是否和服务端资源一致
- 一致则返回 304 (资源未被修改)，否则返回 200 和最新的资源

![http_image_02](http_image_02.png)

## 资源标识

- 在 Response Headers 中，有两种
- `Last-Modified` 资源最后的修改时间
- `Etag` 资源的唯一标识（字符串，类似人类的指纹）

## Last-Modified 和 Etag

- 会优先使用`Etag`
- `Last-Modified`只能精确到秒级
- 如果资源被重复生成，而内容不变，则 `Etag` 更精确(Etag 不会变)

![http_image_03](http_image_03.png)

---

# 刷新操作对缓存的影响

- 正常操作：地址栏输入 url，跳转连接，前进后退等
- 手动刷新：F5，点击刷新按钮，右击菜单刷新
- 强制刷新：ctrl + F5

## 不同刷新操作，不同的缓存策略

- 正常操作：强制缓存<font color='#1FDD1F'>有效</font>，协商缓存<font color='#1FDD1F'>有效</font>
- 手动刷新：强制缓存<font color='red'>失效</font>，协商缓存<font color='#1FDD1F'>有效</font>
- 强制刷新：强制缓存<font color='red'>失效</font>，协商缓存<font color='red'>失效</font>
