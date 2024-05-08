---
title: JS——手写对象Api
date: 2022-12-07
author: 倒数
tags:
  - Object
categories:
  - javaScript
isShowComments: true
subSidebar: auto
---

# 1、前言

# 2. Object.entries()

`Object.entries()`方法返回一个给定对象自身可枚举属性(不是原型上的属性)的键值对数
组,`Object.entries()`返回一个`数组`，其元素是与直接在`object`上找到的`可枚举属性键值对相对应的数组`。属性的顺序与通过手动循环对象的属性值所给出的顺序相同。

简化版：`Object.entries()`方法返回一个每个键值对在一个数组里的二维数组，且不会返回原型上的方法。

实现：

```js
Object.prototype.like = "running";
const obj = new Object({
  name: "123",
  age: 12,
  sex: "1",
});
Object.prototype.my_entries = function (obj) {
  const res = [];
  for (let key in obj) {
    obj.hasOwnProperty(key) && res.push([key, obj[key]]);
  }
  return res;
};
const newObj = Object.my_entries(obj);
console.log(newObj); // [ [ 'name', '123' ], [ 'age', 12 ], [ 'sex', '1' ] ]
```

## 3. Object.fromEntries()

`Object.fromEntries()`方法是把键值对列表转换为一个对象，与`Object.entries() `相对。`Object.fromEntries()` 方法 s 是把键值对列表转换为一个对象。我们可以使用它将**map 或者数组**等其他类型转为对象类型。

实现：

```js
const obj = [
  ["name", "112"],
  ["age", 18],
  ["sex", "123"],
];
const map = new Map(obj);
Object.prototype.my_fromEntries = function (arr) {
  const ss = {};
  for (let [key, value] of arr) {
    ss[key] = value;
  }
  return ss;
};

const newObj = Object.my_fromEntries(obj); // { name: '112', age: 18, sex: '123' }
const newObj1 = Object.my_fromEntries(map); // { name: '112', age: 18, sex: '123' }
console.log(newObj);

console.log(newObj1);
```

## 4. Object.keys()

`Object.keys()`方法为遍历对象上的可枚举自身的属性，返回一个表示给定对象的所有可枚举属性的字符串数组。

简化版：遍历对象上所有的键，但不包含原型上的键, 并返回一个由键组成的字符串数组。

实现：

```js
Object.prototype.like = "running";
const obj = new Object({
  name: "123",
  age: 18,
  sex: "1",
});
Object.prototype.my_keys = function (obj) {
  let res = [];
  for (let key in obj) {
    // hasOwnProperty()方法:某个属性是否是自己直接拥有的属性，而不是从原型链上继承的
    if (obj.hasOwnProperty(key)) {
      res.push(key);
    }
  }
  return res;
};
const keys = Object.my_keys(obj);
console.log(keys); // [ 'name', 'age', 'sex' ]
```

## 5. Object.values()

我们既然知道了`Object.keys()`方法的作用，那自然可以得知`Object.values()`方法就是遍历对象上的可枚举自身的属性的值，同样返回的是一个由所有可枚举属性的值组成的一个数组。

实现：

```js
Object.prototype.like = "running";
const obj = new Object({
  name: "123",
  age: 18,
  sex: "1",
});
Object.prototype.my_values = function (obj) {
  let res = [];
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      res.push(obj[key]);
    }
  }
  return res;
};
const values = Object.values(obj);
console.log(values); // [ '123', 18, '1' ]
```

# 5. Object.is()

`Object.is()`方法判断两个值是否是相同的值。我们知道在 JS 中万物皆对象, 所以在使用`Object.is()`方法时，我们要注意几点：

- 两个值都是 undefined
- 两个值都是 null
- 两个值都是 true 或者都是 false
- 两个值是由相同个数的字符按照相同的顺序组成的字符串
- 两个值指向同一个对象
- 两个值都是数字并且
  - 都是正零 +0
  - 都是负零 -0
  - 都是 NaN
  - 都是除零和 NaN 外的其它同一个数字

这些情况，`Object.is()`返回的都是`true`。所以在我们实现`Object.is()`方法时，我们需要考虑如果接收的参数不是对象的情况。

实现；

```js
Object.prototype.like = "running";
const obj1 = new Object({
  name: "111",
  age: 18,
  sex: "1",
});
const obj2 = new Object({
  name: "1",
  age: 19,
  sex: "3",
});
const obj3 = obj1;
Object.prototype.my_is = function (x, y) {
  if (x === y) {
    return x !== 0 || 1 / x === 1 / y;
  }
  return x !== x && y !== y;
};
const res = Object.my_is(obj1, obj2);
const res1 = Object.my_is(obj1, obj3);
const res2 = Object.my_is(NaN, NaN);
const res3 = Object.my_is(undefined, undefined);
const res4 = Object.my_is(null, null);
const res5 = Object.my_is(true, true);
const res6 = Object.my_is("1", "1");
const res7 = Object.my_is(+0, +0);
const res8 = Object.my_is(-0, +0);
const res9 = Object.my_is(0, 0);
console.log(res); // false
console.log(res1); // true
console.log(res2); // true
console.log(res3); // true
console.log(res4); // true
console.log(res5); // true
console.log(res6); // true
console.log(res7); // true
console.log(res8); // false
console.log(res9); // false
```

# 6. Object.assign()

**`Object.assign()`  方法用于将所有可枚举属性的值从一个或多个源对象复制到目标对象。它将返回目标对象。** 简单来说，`Object.assign()`方法用来复制对象的可枚举属性到目标对象，利用这个特性我们可以实现对象属性的合并。

`Object.assign()`方接收多个参数，第一个参数成为目标对象，后面所有的参数都为源对象，它把源对象的可枚举属性复制到目标对象。

实现：

```js
Object.prototype.like = "running";
let obj1 = new Object({
  name: "123",
  age: 18,
  sex: "1",
  a: 111,
});
let obj2 = new Object({
  name: "121",
  age: 19,
  sex: "2",
});
Object.prototype.my_assign = function (...args) {
  let obj = args[0];
  for (let i = 0; i < args.length; i++) {
    for (let key in args[i]) {
      if (args[i].hasOwnProperty(key)) {
        obj[key] = args[i][key];
      }
    }
  }
  return obj;
};
let obj3 = Object.my_assign(obj1, obj2);
console.log(obj3); // { name: '121', age: 19, sex: '2', a: 111 }
console.log(obj3 === obj1); // true
```
