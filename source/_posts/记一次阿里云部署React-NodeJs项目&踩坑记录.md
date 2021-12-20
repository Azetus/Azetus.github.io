---
title: 记一次阿里云部署React+NodeJs项目&踩坑记录
date: 2021-12-17 16:14:24
tags:
  - 踩坑
  - 阿里云
  - React
  - NodeJs
---

# 前言

前一阵做了一个简单的 Task-App，前端使用的是 React，后端使用的是 Nodejs+Express 框架，数据库选用 MySQL。做好之后想部署在自己的阿里云上，部署过程中踩了一坑，便想记录下来，免得以后遇到同样的问题还得再从头排查。

# 使用 Docker 对 React 项目进行容器化部署

部署前端到阿里云上的时候，想试试使用 Docker 进行容器化部署，顺便学习一下 Docker 的使用，于是按照文档尝试了一下打包。于是在项目根目录里添加了如下的 Dockerfile。

```Dockerfile
FROM node:14 as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY public public/
COPY src src/
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build/ /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

因为我的电脑用的是 Win10 家庭版，没法在电脑上直接打包，于是我选择直接将除了`node_modules`的其他项目文件先传到阿里云上，在阿里云上直接打包。

先 cd 到项目根目录，然后执行打包指令。

`docker build -t todo-list-app .`

之后直接运行在 8085 端口上。

`docker run -d -p 8085:80 todo-list-app`

但是在部署完成后测试的时候发现访问首页的时候一切正常，一旦进入二级页面后再刷新，就会返回 nginx 的 404 页面。而之前在本地测试的时候一切正常。

既然返回的是 nginx 的 404 页面，那问题应该出在 nginx 的配置上。应该在 Docker 打包镜像的时候对 nginx 镜像的配置文件进行修改，否则镜像打包时会默认使用 nginx 镜像中的 default.conf，它将尝试将 url 路径匹配到不存在的静态文件夹/文件，而不是像单页面应用那样正确的渲染。

最终的解决方案是在项目的根目录下创建一个名为`nginx.conf`的文件

```conf
server {
    listen 80;
    location / {
        root /usr/share/nginx/html/;
        index index.html index.htm;
        try_files $uri $uri/ /index.html;
    }
    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        root /user/share/nginx/html;
    }
}
```

监听 docker 镜像内部的 80 端口，添加`try_files`，让 nginx 按指定的 file 顺序查找存在的文件，并使用第一个找到的文件进行请求处理。

最后再 Dockerfile 中添加两部，将 nginx 镜像中的默认配置`default.conf`删除，再将包含添加配置的`nginx.conf`拷贝进去。最后的配置如下：

```Dockerfile
# 拉取node镜像打包React项目
FROM node:14 as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY public public/
COPY src src/
RUN npm run build

# 创建并运行Nginx服务器，并将打包好的文件复制粘贴到服务器文件夹
FROM nginx:alpine
COPY --from=build /app/build/ /usr/share/nginx/html
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

# CentOS7 安装 MySQL 后无法启动服务

我使用的阿里云的操作系统是 CentOS 7.9 64 位，安装上 MySQL 后提示`Failed to start mysql.server.service: Unit not found.`。

起初还以为是 MySQL 版本的问题，卸载后换成了 el7 的安装包但依旧是无法启动服务，最后查了一圈发现 CentOS 7 开始放弃了 MySQL 转而启用了由开源社区维护的 MariaDB，好在 MariaDB 是完全兼容 MySQL，包括 API 和命令行。

于是直接安装 MariaDB 替代 MySQL。

```
# yum install mariadb-server -y
# systemctl start mariadb.service //启动服务
# systemctl enable mariadb.service //开机启动服务
# mysql -u root -p
```

最后经过测试，NodeJS 的 mysql 库也能完全兼容。

# 为 PM2 日志添加时间

使用 PM2 对后端的 NodeJS 项目进行进程守护。默认的日志位置如下：

`/root/.pm2/logs/`

但日志默认是不带时间的，需要手动添加。

首先使用`pm2 list`列出所有 pm2 管理的进程，获取进程 id。

然后使用：

`pm2 restart [id] --log-date-format="YYYY-MM-DD HH:mm Z"`

为日志添加时间

或者在项目启动时添加。

`pm2 start npm --name [别名] -- run prd --log-date-format="YYYY-MM-DD HH:mm Z"`

# 解决 MySQL Error: Connection lost

在测试时，发现服务器崩溃，并在错误日志中打印了如下异常。

```
Error: Connection lost: The server closed the connection.
    at Protocol.end (/home/todo-list-backend/node_modules/mysql/lib/protocol/Protocol.js:112:13)
    at Socket.<anonymous> (/home/todo-list-backend/node_modules/mysql/lib/Connection.js:94:28)
    at Socket.<anonymous> (/home/todo-list-backend/node_modules/mysql/lib/Connection.js:526:10)
    at Socket.emit (events.js:387:35)
    at endReadableNT (internal/streams/readable.js:1317:12)
    at processTicksAndRejections (internal/process/task_queues.js:82:21)
Emitted 'error' event on Connection instance at:
    at Connection._handleProtocolError (/home/todo-list-backend/node_modules/mysql/lib/Connection.js:423:8)
    at Protocol.emit (events.js:375:28)
    at Protocol._delegateError (/home/todo-list-backend/node_modules/mysql/lib/protocol/Protocol.js:398:10)
    at Protocol.end (/home/todo-list-backend/node_modules/mysql/lib/protocol/Protocol.js:116:8)
    at Socket.<anonymous> (/home/todo-list-backend/node_modules/mysql/lib/Connection.js:94:28)
    [... lines matching original stack trace ...]
    at processTicksAndRejections (internal/process/task_queues.js:82:21) {
  fatal: true,
  code: 'PROTOCOL_CONNECTION_LOST'
}
```

根据日志信息初步分析，是 MySQL 连接出现问题，当过几个小时没有访问的 MySQL 的时候，MySQL 会自动断开连接，这个问题的原因是 MySQL 有一个 wait_time 当超过这个时间的时候连接会丢失，再去请求 MySQL 的时候会连接不上 MySQL 服务。

首先尝试复现问题，先重启服务器，连接上 MySQL 后对网页进行测试，测试没有问题后，直接重启服务器的 MariaDB 服务。

`systemctl restart mariadb.service`

再次刷新网页，在错误日志中得到相同的报错。可以确定是数据库自动断开连接的问题。

解决方法：

之前使用的是`mysql.createConnection()`的方法创建的连接，改用使用`mysql.createPool()`这个 API 创建连接池。

```javascript
const con = mysql.createPool(MYSQL_CONF);

function exec(sql) {
  const promise = new Promise((resolve, reject) => {
    con.getConnection(function (err, connection) {
      if (err) {
        reject(err);
      } else {
        connection.query(sql, (err, result) => {
          if (err) {
            reject(err);
            return;
          }
          resolve(result);
        });
        // 在 query 执行完毕后加上 connection.release() 释放连接
        connection.release();
      }
    });
  });
  return promise;
}
```
