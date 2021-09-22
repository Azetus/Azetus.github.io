---
title: '字符串匹配: KMP 算法'
date: 2021-09-22 16:11:03
tags:
  - 学习笔记
  - KMP
  - 字符串匹配
categories:
  - JavaScript学习笔记
---

# KMP 算法

> Knuth-Morris-Pratt 算法

现有一字符串“DBC ABCDABD DABCDABF FCUI”，其中是否包含一个子串“ABCDABF”？

## (1)

![kmp_01](KMP_01.gif)
首先将字符串“DBC ABCDABD DABCDABF FCUI”的首字母与模式串“ABCDABF”的首字母比较，发现不匹配时，将模式串后移一位。

## (2)

![kmp_02](KMP_02.png)

在字母不匹配时不断将模式串后移一位，直至字符串与模式串的首字母匹配。

![kmp_03](KMP_03.gif)

接着比较字符串和搜索词的下一个字符，直到有一位不匹配或者完全匹配。

## (3)

![kmp_04](KMP_04.png)

在例子中的情况里，在匹配到最后一位时，字符串最后一位为'D'，而模式串最后一位为'F'，二者不匹配。

![kmp_05](KMP_05.png)

此时可以将模式串直接向后移动一位继续匹配，这样的暴力搜索解法可行，但是效率不高，因为无法有效利用已经搜索过的前 6 个字母。

## (4)

> 前缀、后缀与部分匹配值

前缀与后缀：“前缀”是指除了最后一个字符外，一个字符串的全部头部组合，而同理，“后缀”是指除了第一个字符外，一个字符串全部尾部的组合。

部分匹配值就是指一个字符串“前缀”与“后缀”的最长共有元素长度。

> 例如"ABCDABF"的子串"A"前后缀为空寂，共有长度为 0  
> 子串"AB"的前缀为`[A]` 后缀为`[B]`，共有长度为 0  
> 以此类推至子串"ABCDAB"的前缀组合为`[A,AB,ABC,ABCD,ABCDA]`后缀组合为`[BCDAB,CDAB,DAB,AB,B]`，共有元素为"AB"长度为 2

以此可以得出一张部分匹配表

![kmp_06](KMP_06.png)

## (4)

![kmp_07](KMP_07.png)
![kmp_08](KMP_08.png)

在例子中，最后一位不匹配，但前六位"ABCDAB"是匹配的，可以利用部分匹配表。得到最后一个匹配的字符"B"对应的部分匹配值是 2。因此可以按如下公式得到，模式串下一次匹配时应向后移动 4 位。

> 移动位数 = 已匹配字符数 - 对应的部分匹配值

# JavaScript 实现

## 1 生成模式串的部分匹配表

首先我们需要根据模式串构建模式串的部分匹配表。
假设模式串为"ABABCABAA"可以得到下表：

|          i           |  0  |  1  |  2  |  3  |  4  |  5  |  6  |  7  |  8  |
| :------------------: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
|      target[i]       |  A  |  B  |  A  |  B  |  C  |  A  |  B  |  A  |  A  |
| len, prefix_table[i] |  0  |  0  |  1  |  2  |  0  |  1  |  2  |  3  |  1  |

可以看出如果我们想要得到`prefix_table[i]` 我们只需要将`target[len]`和`target[i]`进行比较即可，假设此时 `i = 7`，上一位的 `len = 2`，可以得到`target[2] === A === target[7]`，此时，我们得到`target[len]===target[i]`,然后对`len++`，记录`prefix_table[i]===len`,i 指针后移即可。

现在，我们已经得到了 i=7 的值，还剩下 i=8。我们同样按照上面的思路来思考。比较`target[8]--->A`与`target[3]---->B`，发现不匹配。将 len 侧移后再比较一轮。

```JavaScript
function genPrefixTable(target) {
  let prefix_table = [];
  let len = 0; // 最长公共前后缀长度初始化为0
  prefix_table[0] = len;
  let n = target.length;
  let i = 1;
  while (i < n) {
    if (target[len] === target[i]) {
      len++;
      prefix_table[i] = len;
      i++;
    } else {
      if (len > 0) {
        // 侧移
        len = prefix_table[len - 1]; //缩小最长公共前后缀的长度
        // 再下一轮 while 循环时再比较一次
      } else {
        prefix_table[i] = 0;
        i++;
      }
    }
  }
  return prefix_table;
}
```

## 2 得到 next 数组

字符不匹配时

> 模式串位移的位数 = 已匹配字符数 - 不匹配字符上一位字符对应的部分匹配值。

因此，再出现不匹配情况时，我们不用去关心失配位字符的情况，而是需要考虑上一位字符。因此可以将最大长度表右移一位，首位赋值-1，变得到了 next 数组。

最终我们在计算模式串位移位数可以得到：

> 移动位数 = 失配字符位置 - 失配字符对应的 next 数组的值

| target[i] |  A  |  B  |  A  |  B  |  C  |  A  |  B  |  A  |  A  |
| :-------: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
|  next[i]  | -1  |  0  |  0  |  1  |  2  |  0  |  1  |  2  |  3  |

```JavaScript
function genNextArr(prefix_table) {
  let next = new Array(prefix_table.length).fill(0);
  for (var i = prefix_table.length - 1; i > 0; i--) {
    next[i] = prefix_table[i - 1];
  }
  next[0] = -1;
  return next;
}
```

## 3 KMP 算法

使用之前构建的函数生成 next 数组，利用两个指针分别遍历模板字符串`str`和模式字符串`target`。在双指针都小于两个字符串的长度时不停的循环遍历。在匹配时，双指针同步右移。在出现失配情况时，根据 next 数组，得到此时`j = next[j]`。

![kmp_09](KMP_09.png)
![kmp_10](KMP_10.png)

此时指针`i = 12`，指针`j = 8`，模式串整体右移 5 位。对于两个指针相当于`i`不变，`j = next[j] -> 3`，若出现`j -> -1`的情况，则两个指针整体右移一位。

```JavaScript
function KMP(str, target) {
  let prefix_table = genPrefixTable(target);
  let next = genNextArr(prefix_table);
  let i = 0; // str 指针
  let j = 0; // target 指针
  while (i < str.length && j < target.length) {
    if (str[i] === target[j]) {
      i++;
      j++;
    } else {
      j = next[j]; // 右移
      if (j === -1) {
        // 这部分可以移到第一个 if 中 ——> if(str[i] === target[j] || j === -1)
        i++;
        j++;
      }
    }
  }
  if (j === target.length) {
    return i - j;
  } else {
    return -1;
  }
}
```

# 优化后的 KMP 算法

直接构建 next 数组，节省一部分内存空间。

```JavaScript
function genNextArr(target) {
  let i = 0;
  let j = -1;
  const next = new Array(target.length).fill(0);
  next[0] = -1;
  while (i < target.length - 1) {
    if (j === -1 || target[i] === target[j]) {
      i++;
      j++;
      next[i] = j;
    } else {
      j = next[j];
    }
  }
  return next;
}
```

KMP 部分基本无变化

```JavaScript
function KMP(str, target) {
  const next = Next(target);
  let i = 0; // str 指针
  let j = 0; // target 指针
  while (i < str.length && j < target.length) {
    if (str[i] === target[j] || j === -1) {
      i++;
      j++;
    } else {
      j = next[j]; //右移
    }
  }
  if (j === target.length) {
    return i - j;
  } else {
    return -1;
  }
}
```
