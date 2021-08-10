---
title: CCS 布局相关知识点
date: 2021-07-10 16:35:43
tags:
  - css
categories:
  - css学习笔记
---

# CCS 布局相关知识点

### 1 盒模型宽度的计算

> 请问如下代码中， div1 的 offsetWidth 是多大？

```HTML
<style>
  #div1 {
    width:100px;
    padding: 10px;
    border: 1px solid #ccc;
    margin: 10px;
  }
</style>

<div id='div1'>This is div1</div>
```

offsetWidth = ( 内容宽度 + 内边距 + 边框 )，无外边距。
因此该元素的 offsetWidth = ( width + 2 _ padding + 2 _ border) = 122px

> 若想要使 offsetWidth 等于 100px，可以添加`box-sizing: border-box`。

### 2 margin 重叠的问题

> 请问如下代码中， AAA 和 BBB 之间的距离是多少？（15px）

```HTML
<style>
  p {
    font-size: 16px;
    line-height: 1;
    margin-top: 10px;
    margin-bottom: 15px;
  }
</style>

<p>AAA</p>
<p></p>
<p></p>
<p></p>
<p>BBB</p>
```

- 相邻元素的 `margin-top` 和 `margin-bottom` 会发生重叠。
- 空白内容的`<p></p>`也会重叠。

### 3 margin 负值的问题

> 对 margin 的 top left right bottom 设置负值会有什么样的效果？

1. 对`margin-top` 和 `margin-left` 设置负值时，**_元素自身会向上、向左移动_**。
2. 对`margin-right` 设置负值，右侧元素左移，**_自身不受影响_**。
3. 对`margin-bottom` 设置负值，下方元素上移，**_自身不受影响_**。

### 4 BFC 的理解与应用

> 什么是 BFC？如何应用？

- Block format context ，块级格式化上下文。
- 一块独立的渲染区域，内部元素的渲染不会影响边界以外的元素。

**形成 BFC 的常见条件**

- `float` 不是 `none`
- `overflow` 不是 `visible`
- `position` 是 `absolute` 或 `fixed`
- `display` 是 `flex` 和 `inline-block` 等

**BFC 的常见应用**

- 清除浮动

```HTML
<div>
  <div class="container bfc">
    <img src="...." class="left" style="margin-left: 10px;">
    <p class="bfc">一段文字</p>
  </div>
</div>
```

```HTML
<style>
  .container {
    background-color: #f1f1f1;
  }
  .left {
    float: left;
  }
  .bfc {
    overflow: hidden; /* 触发元素 BFC */
  }
</style>
```

### 5 float 布局

> 如何实现圣杯布局和双飞翼布局

**圣杯布局和双飞翼布局的目的**

- 三栏布局，中间一栏最先加载和渲染。（内容最重要）
- 两侧内容固定，中间内容随着宽度自适应。
- 一般用于 PC 网页

**1. 圣杯布局**

![img01](20210801163126.png)

```HTML
<div id="header">this is header</div>
  <div id="container" class="clearfix">
    <div id="center" class="column">this is center</div>
    <div id="left" class="column">this is left</div>
    <div id="right" class="column">this is right</div>
  </div>
<div id="footer">this is footer</div>
```

```HTML
<style type="text/css">
  body {
    min-width: 550px;
  }
  #header {
    text-align: center;
    background-color: #f1f1f1;
  }

  /* 中间部分设置 padding 为两边留白 */
  #container {
    padding-left: 200px;
    padding-right: 150px;
  }
  /* 使用 float 布局 */
  #container .column {
    float: left;
  }

  #center {
    background-color: #ccc;
    width: 100%;
  }
  #left {
    position: relative;
    right: 200px; /* 相对于自身移动 200px */
    background-color: yellow;
    width: 200px;
    margin-left: -100%; /* 设置 -100% 向左移动至重合 */
  }
  #right {
    background-color: red;
    width: 150px;
    margin-right: -150px;
  }

  #footer {
    text-align: center;
    background-color: #f1f1f1;
  }

  /* 手写 clearfix */
  .clearfix:after {
    content: '';
    display: table;
    clear: both;
  }
</style>
```

**2. 双飞翼布局**

![img02](20210801163147.png)

```HTML
<div id="main" class="col">
  <div id="main-wrap">
    this is main
  </div>
</div>
<div id="left" class="col">
  this is left
</div>
<div id="right" class="col">
  this is right
</div>
```

```HTML
<style type="text/css">
    body {
        min-width: 550px;
    }
    .col {
        float: left;
    }

    #main {
        width: 100%;
        height: 200px;
        background-color: #ccc;
    }
    #main-wrap {
        /* 使用 margin 为两边留白 */
        margin: 0 190px 0 190px;
    }

    #left {
        width: 190px;
        height: 200px;
        background-color: #0000FF;
        margin-left: -100%;
    }
    #right {
        width: 190px;
        height: 200px;
        background-color: #FF0000;
        margin-left: -190px;
    }
</style>
```

**技术总结**

- 使用 `float` 布局。
- 两侧使用`margin`负值，以便和中间内容横向重叠。
- 防止中间内容被两侧覆盖，一个使用`padding`一个使用`margin`。

> clearfix

```CSS
.clearfix:after {
  content: '';
  display: table;
  clear: both;
}
/* 兼容 IE 低版本 */
.clearfix {
  *zoom: 1;
}
```

给浮动元素的容器添加一个 clearfix 的 class，然后给这个 class 添加一个`:after` 伪元素实现元素之后添加一个看不见的块元素（Block element），设置`clear: both;`属性清理浮动。

### 6 flex 布局

容器属性：

- `flex-direction` 主轴方向
- `justify-content` 主轴对齐方式
- `align-items` 交叉轴对齐方式
- `flex-wrap` 是否换行

子元素属性：

- `align-self` 子元素在交叉轴的对齐方式
- `flex` 默认值为`0 1 auto`，是`flex-grow`、`flex-shrink` 和 `flex-basis` 属性的简写

> 使用 flex 实现一个三点骰子

![img03](20210801163240.png)

```HTML
<div class="box">
    <span class="item"></span>
    <span class="item"></span>
    <span class="item"></span>
</div>
```

```HTML
<style type="text/css">
    .box {
        width: 200px;
        height: 200px;
        border: 2px solid #ccc;
        border-radius: 10px;
        padding: 20px;
    /* 使用 flex 布局 */
        display: flex;
    /* 两端对齐，三个点横向平均分布 */
        justify-content: space-between;
    }
    .item {
        display: block;
        width: 40px;
        height: 40px;
        border-radius: 50%;
        background-color: #666;
    }
    /* 第二项在垂直方向居中对齐 */
    .item:nth-child(2) {
        align-self: center;
    }
    /* 第三项在垂直方向尾对齐 */
    .item:nth-child(3) {
        align-self: flex-end;
    }

</style>
```
