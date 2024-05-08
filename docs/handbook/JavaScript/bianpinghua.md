---
title: JS——扁平化
date: 2022-07-09
author: 倒数
tags:
  - js
categories:
  - javaScript
isShowComments: true
subSidebar: auto
---

# 1. 前言

相信我们在刷算法时，都做过把多维数组变成一维数组， 我们有很多办法，例如递归、flat（）函数、正则。既然数组可以扁平化为一维数组，那如果我们想把对象扁平化该怎么做呢？那我们就来介绍几种方法去实现数组和对象的扁平化。

# 2. 数组扁平化

题目描述： 给定你一个多维数组，例如：**[ 1, 2, [3, 4, [5]], 6, {a: 7} ]**，让我们将多维数组变为一维数组。

```js
[ 1, 2, [3, 4, [5]], 6, {a: 7} ] ---> [ 1, 2, 3, 4, 5, 6, { a: 7 }
```

## 2.1 flat()方法

首先我们第一个想到的就 ES10 新增的`flat()`方法, 它可以帮助我们实现数组扁平化。实现如下：

```js
const arr = [1, 2, [3, 4, [5]], 6, { a: 7 }];

const newArr = arr.flat(1); // [ 1, 2, 3, 4, [ 5 ], 6, { a: 7 } ]
console.log(newArr);
const newArr1 = arr.flat(2); // [ 1, 2, 3, 4, 5, 6, { a: 7 } ]
console.log(newArr1);
const newArr2 = arr.flat(3); // [ 1, 2, 3, 4, 5, 6, { a: 7 } ]
console.log(newArr2);
```

`flat()`方法，可以直接帮我们实数组的扁平化，而且`falt()`方法还接受一个参数供我们选择将数组变为我们想要的维数。

## 2.2 reduce() 方法+递归

那如果没有`flat()`方法， 我们还有其他的方法实现了数组扁平化吗？这是，我们还有一种很容易想到的办法，那就是递归。那我们就是使用` reduce() 方法+递归`的方式实现吧。

```js
const arr = [1, 2, [3, 4, [5]], 6, { a: 7 }];
let myFlat = (arr) => {
  return arr.reduce((pre, next) => {
    return pre.concat(Array.isArray(next) ? myFlat(next) : next);
  }, []);
};
let newArr = myFlat(arr);
console.log(newArr); // [ 1, 2, 3, 4, 5, 6, { a: 7 } ]
```

在`reduce() 方法+递归`方式里，我们需要想到的就是，当我们循环碰到是数组的话，我们就需要递归的执行，然后将我们递归的结果在拼接成一个新的以为数组。

**小提示：如果不熟系这两种方法的小伙伴可以去看[# 数组各种好用 API 的手写](https://juejin.cn/post/7174043835880701966)文章。**

## 2.3 不使用递归

那到这里，很多小伙伴就该问了，使用了递归的性能还是不够好，那有不使用递归的方法吗？那到这里，我就再来实现一个不使用递归的方法。

```js
const arr = [1, 2, [3, 4, [5]], 6, { a: 7 }];
let myFlat1 = (arr, num = 1) => {
  let newArr = arr;
  let i = 0;
  while (arr.some((item) => Array.isArray(item)) && i < num) {
    newArr = [].concat(...newArr);
    i++;
  }
  return newArr;
};

let newArr1 = myFlat1(arr, 2);
console.log(newArr1); // [ 1, 2, 3, 4, 5, 6, { a: 7 } ]
```

我们自己利用了数组身上的`some()`方法，去将遍历数组每个元素，再将符合条件的元素结构出来进行拼接，我们也实现了数组的扁平化。

**小提示：这里我们还可以想到了一个办法，那就是利用正则表达式，去除数组左右两边的数组再进行拼接，小伙伴可以自己去试试哦**

# 3. 对象扁平化

在介绍完数组的扁平化之后，我们就该到我们的对象扁平化了。那我们该如何实现对象的扁平化呢？

题目描述：

```js
const obj = {
    a: {
      b: 1,
      c: 2,
      d: { e: 5 }
    }
    b: [1, 3, { a: 2, b: 3 }]
    c: 3
  }
----->
// {
  //   a.b: 1,
  //   a.c: 2,
  //   a.d.e: 5
  //   b[0]: 1,  
  //   b[1]: 3,  
  //   b[2].a: 2,  
  //   b[2].b: 3,  
  //   c: 3
  // }
```

在看数组扁平化之后，我们不难看出实现扁平化的核心原理，就是当我们在循环数组时碰到是数组，我们该怎么处理？我们做的是将数组里的值拿出来，再拼接起来。

那如果是对象的扁平化呢？对象的键该怎么处理呢？那我就先来实现一下吧。

```js
const obj = {
  a: {
    b: 1,
    c: 2,
    d: { e: 5 },
  },
  b: [1, 3, { a: 2, b: 3 }],
  c: 3,
};

function flatten(obj) {
  let res = {};
  function dfs(cur, prefix) {
    // prefix参数是为了拿到保留上一级的键
    if (typeof cur === "object" && cur !== null) {
      // 递归结束条件
      if (Array.isArray(cur)) {
        // 拿到数组的拼接方式
        cur.forEach((item, index) => {
          dfs(item, `${prefix}[${index}]`);
        });
      } else {
        for (let key in cur) {
          // 拿到对象的拼接方式
          dfs(cur[key], `${prefix}${prefix ? "." : ""}${key}`); // 将上一级的键与这一级的键进行拼接
        }
      }
    } else {
      res[prefix] = cur;
    }
  }
  // 第一次
  dfs(obj, "");
  return res;
}
console.log(flatten(obj));
// { //   'a.b': 1, //   'a.c': 2, //   'a.d.e': 5, //   'b[0]': 1, //   'b[1]': 3, //   'b[2].a': 2, //   'b[2].b': 3, //   c: 3 // }
```

在对象的扁平化中，需要拿到对象的键，将对象的键进行一个拼接，然后我们再给对象的键去赋值既实现了对象的扁平化。
