---
title: flex布局下利用margin实现自动居中
date: 2021-07-08 16:23:43
tags:
  - css
  - margin
categories:
  - CSS
---

# flex 布局下利用 margin 实现自动居中

在传统 BFC（块格式化上下文）中，若子元素为 `margin:auto`，子元素只会在水平方向居中。如下所示：

HTML 代码：

```HTML
<div class="box1">
  <div class="box2"></div>
</div>
```

CSS 代码：

```CSS
.box1 {
  width: 600px;
  height: 600px;
  border: black solid 1px;
}
.box2 {
  width: 200px;
  height: 200px;
  margin: auto;
  background-color: black;
}
```

最终效果如下：

![img01](20210720164514.png)

这是因为，在 BFC 下，如果`margin-left`与`margin-right`同时为 `auto`，即代表二者相等，左右 margin 的计算值为元素剩余可用空间的一半，结果就是子元素在水平方向上居中。
而如果`margin-top`和`margin-bottom`同时为 `auto`，则他们的计算值都为 0，因此无法垂直居中

### FFC 下使用`margin:auto`对元素进行垂直居中

如果将上文中父元素 box1 设置为`display: flex`，则效果如下：

![img02](20210720170100.png)

原因是在于，flex 格式化上下文中，设置了`margin: auto`的元素，在通过 `justify-content` 和 `align-self` 进行对齐之前，任何可供子元素分配的剩余空间都会分配到该方向的 margin 中去，并且，`margin: auto` 的生效不仅是水平方向，垂直方向也会自动去分配这个剩余空间。

### 基于 flex 布局实现导航栏

假如使用以下代码实现一个网页顶部的导航栏，利用`justify-content: space-around`来调整导航栏中各元素的间距。

```HTML
<ul class="navigator">
  <li>nav01</li>
  <li>nav02</li>
  <li>nav03</li>
  <li>nav04</li>
  <li>nav05</li>
</ul>
```

```CSS
.navigator {
  display: flex;
  height: 50px;
  justify-content: space-around;
  background-color: black;
}
li {
  list-style: none;
  color: white;
}
```

![img03](20210720172014.png)

可以看到结果各元素在垂直方向上并没有居中对齐，这时如果在子元素的 css 代码中添加`margin: auto;`就能达到想要的效果

```CSS
.navigator {
  display: flex;
  height: 50px;
  background-color: black;
}
li {
  margin: auto;
  list-style: none;
  color: white;
}
```

![img04](20210720172811.png)

> _若不使用`margin: auto;`调整垂直居中，也可以设置子元素的`line-height`等于父元素`height`，这样也能呈现同样的效果_

> _此外,若子元素设置了`margin: auto;`,其父元素的`justify-content`将不会再生效_

#### 使用自动 margin 还可以实现元素的不规则两端对齐

导航栏的 dom 结构如下

```HTML
<ul class="navigator">
  <li>nav01</li>
  <li>nav02</li>
  <li>nav03</li>
  <li>nav04</li>
  <li class="signup">Sign Up</li>
  <li>Log In</li>
</ul>
```

若想要导航栏中 nav01 至 nav04 在导航栏左侧对齐，而登录和注册按钮在右侧对齐，可以在登录按钮即 Sign Up 的 CSS 样式中添加`margin-left: auto`，此时 Sign Up 的左侧 margin 的计算值将会自动添加其父元素水平方向剩余的可支配空间，从而实现我们想要的效果

```CSS
.navigator {
  display: flex;
}
.signup {
  margin-left: auto;
}
```

最终的效果如下：

![img05](20210720174355.png)
