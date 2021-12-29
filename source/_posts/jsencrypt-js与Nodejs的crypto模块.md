---
title: jsencrypt.js与NodeJs的crypto模块
date: 2021-12-29 15:06:53
tags:
  - NodeJs
  - jsencrypt
  - crypto
  - RSA
categories:
  - NodeJs
---

# 前言

之前搞了一个简单的 Task-App，前端部分使用的 React，后端部分使用的 express 框架，包含了用户注册、登录的功能，需要用到 RSA 加密算法为用户传输的密码进行加密处理，因为是第一次接触和使用，正好把使用过程和遇到的一些小问题记录下来，方便以后复习查阅。

# RSA 加密算法

RSA 加密算法是一种非对称加密算法，可以在不直接传递秘钥时完成解密，RSA 加密使用了"一对"密钥，分别是公钥和私钥，秘钥在生成时即是两两成对的。在使用时甲使用公钥加密，乙使用私钥解密。

只要将私钥保密，将公钥公开给任何用户，即是第三者得到了公钥，也无法解开使用公钥加密的密文。

同时 RSA 算法属于一种块加密算法，其特点便是**总是保持在一个固定长度的块上进行操作**，因此可接受的明文长度限制是由秘钥的 key length 决定的。所以在加密和解密的过程中需要分块进行，当分块后的内容小于 key length 时会自动填满。RSA 算法主要有三种填充方式：`RSA_NO_PADDING`、`RSA_PKCS1_PADDING`、`RSA_PKCS1_OAEP_PADDING`

## jsencrypt.js 的使用

jsencrypt.js 的使用十分简单，首先安装依赖`npm install jsencrypt`，然后在文件中引入`import JSEncrypt from 'jsencrypt'`。

jsencrypt.js **仅支持 RSA_PKCS1_PADDING 填充方式**

加密

```javascript
import { publicKey } from './res_public_key';

// 公钥加密
function rsaEncrypt(data) {
  const encode = new JSEncrypt();
  encode.setPublicKey(publicKey);
  const encodeData = encode.encrypt(data);
  return encodeData;
}

export default rsaEncrypt;
```

解密

```javascript
import { PrivateKey } from './res_private_key';

// 私钥解密
function rsaDecrypt(data) {
  const decode = new JSEncrypt();
  decode.setPrivateKey(publicKey);
  const decodeData = encode.decrypt(data);
  return decodeData;
}

export default rsadecrypt;
```

## crypto 私钥解密

crypto 是 NodeJs 原生自带的一个加密模块，使用它来对前端发送过来的密文进行解密。服务端持有的私钥以`.pem`的格式存储在服务器上，首先引入 crypto 与 fs 模块。

```javascript
const crypto = require('crypto');
const fs = require('fs');
```

然后从文件中读取私钥。

```javascript
const privateKey = fs.readFileSync('./utils/rsa_private_key.pem').toString();
```

使用`crypto.privateDecrypt(privateKey, buffer)`方法来对密文进行解密。

根据文档说明：

> If `privateKey` is not a `KeyObject`, this function behaves as if `privateKey` had been passed to `crypto.createPrivateKey()`. If it is an `object`, the `padding` property can be passed. Otherwise, this function uses `RSA_PKCS1_OAEP_PADDING`.

如果参数`privateKey`不是一个由`crypto.createPrivateKey()`方法生成的`KeyObject`，则等同于将该参数传入`crypto.createPrivateKey()`一样，如果参数是一个对象，则可以传入对象中的`padding`属性，否则将默认使用 `RSA_PKCS1_OAEP_PADDING` 填充方式。

在前文中提到过：jsencrypt.js **仅支持 RSA_PKCS1_PADDING 填充方式**

因此这里需要手动设置 crypto 解密私钥时使用的填充方式。否则无法解密密文。

```javascript
const privateKeyObj = {
  key: privateKey,
  padding: crypto.constants.RSA_PKCS1_PADDING
};

function decrypt(data) {
  const decodeData = crypto
    .privateDecrypt(privateKeyObj, Buffer.from(data, 'base64'))
    .toString('utf-8');
  return decodeData;
}
```

# 使用 MD5 加密存入数据库的用户密码

一般来说，用户信息数据库中的用户密码是不会直接使用明文进行存储的，如果数据库直接使用明文存储用户密码，若数据库泄露后，普通用户也可以很容易的使用泄露的明文密码登录他人的账户。因此在项目中使用 MD5 对用户密码进行加密处理。

MD5 是一种哈希算法，可以将任意长度的输入串经过计算得到固定长度的输出，而且只有在明文相同的情况下，才能等到相同的密文，并且这个算法是不可逆的。

在用户注册的场景下，将由 RSA 私钥解密出的用户的密码进行 MD5 加密后再存储进数据库，而在用户登录验证时，将私钥解密出的用户的密码进行 MD5 处理后与数据库中的记录进行比对，因为 MD5 明文相同才能得到相同密文的特性，直接使用密文与密文相匹配，匹配成功则返回登录成功。

依旧是用 NodeJs 自带的 crypto 库进行加密处理。

```javascript
const crypto = require('crypto');

// 密匙
const SECRET_KEY = 'MySecretKey';

// md5 加密
function md5(content) {
  let md5 = crypto.createHash('md5');
  return md5.update(content).digest('hex');
}

// 加密函数
function genPassword(password) {
  const str = `password=${password}&key=${SECRET_KEY}`;
  return md5(str);
}
```
