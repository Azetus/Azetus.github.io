---
title: JS-Web-API 存储
date: 2021-07-28 12:49:55
tags:
  - 存储
categories:
  - JS-Web-API学习笔记
---

# JS-Web-API 存储

## 知识点

- cookie
- localStorage 和 sessionStorage

## cookie

- 本身用于浏览器和 server 通讯
- 被"借用"到本地存储来
- 可用 `document.cookie = 'key = value'`来修改
  > document.cookie 一次只能追加一个键值对，新的键值对会追加进 cookir，相同的 key 值会覆盖

## cookie 的缺点

- 存储大小，最大 4KB
- http 请求时需要发送到服务端，增加请求数据量
- 只能用 `document.cookie = '...'` 来修改，太过简陋

## localStorage 和 sessionStorage

- HTML5 专门为存储而设计，最大可存 5Mb
- API 简单易用，`setItem`、`getItem`
- 不会随着 http 请求被发送出去

二者区别：

- localStorage 数据会永久存储，除非代码或手动删除
- sessionStorage 数据只存在于当前会话，浏览器关闭则清空
- 一般用 localStorage 会更多一些

## 小结

- 容量 cookie 最大 4kb， localStorage 和 sessionStorage 最大 5Mb
- API 易用性
- 是否会随 http 请求发送出去
