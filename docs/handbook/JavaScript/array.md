---
title: JS——手写数组Api
date: 2022-12-06
author: 倒数
tags:
  - Array
categories:
  - javaScript
isShowComments: true
subSidebar: auto
---

# 1、前言

在我们 conding 中，我们都用过数组这个数据结构，我们除了使用了数组自带的增删改查方法，还会使用带数组身上各类的 API 方法，下面我就来介绍数组身上的各类 api 方法。

# 2、迭代属性的 API 方法

## 2.1 Array.every()

#### 1. evrry()原理：

`every()` 方法接受一个回调函数作为参数，它是检查数组中的所有元素是否都通过了我们的限制条件，`every()` 方法对数组中存在的每个元素都会去执行一次：

- 如果找到函数返回 false 值的数组元素，`every()` 返回 false，并且不检查剩余的元素
- 如果没有出现 false，`every()` 返回 true
- `every()` 不改变原始数组

#### 2. evrry()方法的手写实现：

```js
const singer = [
  { name: "111", num: 20 },
  { name: "222", num: 25 },
  { name: "333", num: 19 },
  { name: "444", num: 10 },
  { name: "555", num: 0 },
];
Array.prototype.my_every = function (callback) {
  for (let i = 0; i < this.length; i++) {
    if (!callback(this[i], i, this)) {
      return false;
    }
  }
  return true;
};

const newSinger = singer.my_every((item, index, arr) => {
  return item.num > 10; // singer[4].num < 10
});
console.log(newSinger); //false
```

## 2.2 Array.some()

### 1.some()原理

`some()` 方法也接受一个回调函数作为参数，与`every()`相反，它是检查数组中的任何元素是否通过我们的限制条件，`some()` 方法对数组中存在的每个元素执行一次函数：

- 如果找到函数返回真值的数组元素，some() 返回 ture,并且不检查剩余的元素
- 否则返回 false
- `some()` 不改变原始数组。

### 2.some()方法的手写实现

```js
const singer = [
  { name: "111", num: 20 },
  { name: "222", num: 25 },
  { name: "333", num: 19 },
  { name: "444", num: 10 },
  { name: "555", num: 0 },
];
Array.prototype.my_some = function (callback) {
  for (let i = 0; i < this.length; i++) {
    if (callback(this[i], i, this)) {
      return true;
    }
  }
  return false;
};
const newSinger = singer.my_some((item, index, arr) => {
  return item.num > 20; // singer[1].num > 20
});
console.log(newSinger); // true
```

## 2.3 Array.map()

### 1.map()原理

`map()`方法接收三个参数，分别为 `item`（值）、`index`（下标）、`arr`（原数组），它使用为每个数组元素调用函数的结果创建新数组， 且函数一定要有`return`，并不会改变原始数组。

### 2.map()方法的手写实现

```js
const singer = [
  { name: "111", num: 20 },
  { name: "222", num: 25 },
  { name: "333", num: 19 },
  { name: "444", num: 10 },
  { name: "555", num: 0 },
];
Array.prototype.my_map = function (callback) {
  const res = [];
  for (let i = 0; i < this.length; i++) {
    res.push(callback(this[i], i, this));
  }
  return res;
};

const newSinger = singer.my_map((item, index, arr) => {
  return item.num > 20;
});
console.log(newSinger); // [ false, true, false, false, false ]
```

## 2.4 Array.reduce()

### 1.reduce()原理

`reduce()` 方法接受两个参数，第一个参数是一个回调函数，这个回调函数接受四个参数，分别为`pre`(上次函数的执行结果)、`next`(值)、`index`（下标）、`arr`（原数组），第二个参数为第一次执行回调函数时`pre`的初始值。

如果我们没有给第二个参数，则`pre`为数组的第一个值，`next`为数组的第二个值。`reduce()` 方法也不会改变原始数组。一般我们会使用`reduce`方法实现数组的累加。

### 2.reduce()方法的手写实现

```js
const singer = [
  { name: "111", num: 20 },
  { name: "222", num: 25 },
  { name: "333", num: 19 },
  { name: "444", num: 10 },
  { name: "555", num: 1 },
];
Array.prototype.my_reduce = function (callback, ...args) {
  let pre,
    start = 0;
  if (args.length) {
    pre = args[0];
  } else {
    pre = this[0];
    start = 1;
  }
  for (let i = start; i < this.length; i++) {
    pre = callback(pre, this[i], i, this);
  }
  return pre;
};
const newSinger = singer.my_reduce((pre, next, index, arr) => {
  console.log(pre); //0 20 45 64 74
  return pre + next.num;
}, 0);

console.log(newSinger); // 75
```

## 2.5 Array.filter()

### 1.filter()原理

`filter()` 方法接收三个参数，分别为`item`、`index`、`arr`，他创建了一个新的数组，其中填充了所有满足我们的限制条件的元素。`filter()`也不会改变原始数组。

### 2.filter()方法的手写实现

```js
const singer = [
  { name: "111", num: 20 },
  { name: "222", num: 25 },
  { name: "333", num: 19 },
  { name: "444", num: 10 },
  { name: "555", num: 0 },
];
Array.prototype.my_filter = function (callback) {
  const res = [];
  for (let i = 0; i < this.length; i++) {
    callback(this[i], i, this) && res.push(this[i]);
  }
  return res;
};
const newSinger = singer.filter((item, index, arr) => {
  return item.num > 15;
});
console.log(newSinger);
// [ //   { name: '111', num: 20 }, //   { name: '222', num: 25 }, //   { name: '333', num: 19 } // ]
```

## 2.6 Array.forEach()

### 1.forEach()原理

`forEach()` 方法一个回调函数作为参数，`forEach()`方法按顺序为数组中的每个元素调用一次函数。

### 2.forEach()方法的手写实现

```js
const singer = [
  { name: "111", num: 20 },
  { name: "222", num: 25 },
  { name: "333", num: 19 },
  { name: "444", num: 10 },
  { name: "555", num: 0 },
];
Array.prototype.my_forEach = function (callback) {
  for (let i = 0; i < this.length; i++) {
    callback(this[i], i, this);
  }
};
singer.my_forEach((item, index, arr) => {
  console.log(item, index);
});
// { name: '111', num: 20 } 0
// { name: '222', num: 25 } 1
// { name: '333', num: 19 } 2
// { name: '444', num: 10 } 3
// { name: '555', num: 0 } 4
```

# 3、查找属性的 API 方法

## 3.1 Array.find()

### 1.find()原理

`find()` 方法接收三个参数，分别为 `val`（你要查找元素）、`start`（开始查找的位置）、`end`（结束的查找的位置）,它返回数组中**第一个**通过限制的的元素的值。`find()` 方法对数组中存在的每个元素执行一次函数：

- 如果找到函数返回 true 值的数组元素，则 find() 返回该数组元素的值（并且不检查剩余值）
- 否则返回 undefined
- `find()` 不会改变原始数组。

### 2.find()方法的手写实现

```js
const singer = [
  { name: "111", num: 20 },
  { name: "222", num: 25 },
  { name: "333", num: 19 },
  { name: "444", num: 10 },
  { name: "555", num: 0 },
];
Array.prototype.my_find = function (callback) {
  for (let i = 0; i < this.length; i++) {
    if (callback(this[i], i, this)) {
      return this[i];
    }
  }
  return undefined;
};
const res = singer.my_find((item, index, arr) => {
  return item.num > 20;
});

console.log(res); // { name: '222', num: 25 }
```

## 3.2 Array.findIndex()

### 1.findIndex()原理

`findIndex()` 方法与`find（）`方法原理相同，如果找到了返回对应**下标值**，否则返回`-1`

### 2.findIndex()方法的手写实现

```js
const singer = [
  { name: "111", num: 20 },
  { name: "222", num: 25 },
  { name: "333", num: 19 },
  { name: "444", num: 10 },
  { name: "555", num: 0 },
];
Array.prototype.my_findIndex = function (callback) {
  for (let i = 0; i < this.length; i++) {
    if (callback(this[i], i, this)) {
      return i;
    }
  }
  return -1;
};
const res = singer.findIndex((item, index, arr) => {
  return item.num > 30;
});
console.log(res); // -1
```

# 4、其他的好用 API 方法

## 4.1 Array.include()

### 1.includes()原理

`includes()` 方法接收两个参数，第一个参数为数组是否包含的元素，第二个参数为从数组某个位置开始向后查找，如果第二个个参数为负数，则从负数加上数组长度的位置开始。如果数组包含元素，则此方法返回 `true`，否则返回 `false`。且该方法区分大小写。

使用`includes()`方法时，我们要考虑当我们第一个参数为`NaN`时，由于`NaN != NaN`，所以在我们手写实现时，我们将这种情况考虑进去

### 2.includes()方法的手写实现

```js
let obj = { name: "111", num: 25 };
const singer = [
  { name: "222", num: 20 },
  obj,
  { name: "333", num: 19 },
  { name: "444", num: 10 },
  { name: "555", num: 0 },
];
const nums = [1, 2, 3, 4, 5, NaN];
Array.prototype.my_includes = function (val, start = 0) {
  if (start < 0) start = this.length + start;
  const isNaN = Number.isNaN(val);
  for (let i = start; i < this.length; i++) {
    if (this[i] === val || (isNaN && Number.isNaN(this[i]))) {
      // NaN === NaN
      return true;
    }
  }
  return false;
};
const res = nums.my_includes(NaN, 0);
console.log(res); // true
```

## 4.2 Array.join()

### 1.join()原理

`join()` 方法将数组作为字符串返回。元素将由指定的分隔符分隔。默认分隔符是逗号 `,`。`join()` 方法不会改变原始数组。

### 2.join()方法的手写实现

```js
const arr = [1, 2, 3, 4, "hello"];
Array.prototype.my_join = function (s = ",") {
  let str = "";
  for (let i = 0; i < this.length; i++) {
    if (i === this.length - 1) {
      str += this[i];
    } else {
      str += this[i] + s;
    } // str += i === this.length - 1 ? this[i] : this[i] + s // 简洁版
  }
  return str;
};
const str = arr.my_join();
console.log(str);
```

## 4.3 Array.flat()

### 1.flat()原理

`flat()`方法会按照一个可指定的深度递归遍历数组，并将所有元素与遍历到的子数组中的元素合并为一个新数组返回。其接收一个参数**默认值为 1**，表示需要深度遍历几次，超出数组维度则指挥遍历**数组维度-1**次,使其成为以为数组

### 2.flat()方法的手写实现

```js
const arr = [1, 2, [3, 4, [5]], 6, { a: 7 }];
Array.prototype.myflat = function (num = 1) {
  let newArr = this;
  let i = 0;
  while (newArr.some((item) => Array.isArray(item)) && i < num) {
    newArr = [].concat(...newArr);
    i++;
  }
  return newArr;
};

const newArr = arr.myflat(4);

console.log(newArr); // [ 1, 2, 3, 4, 5, 6, { a: 7 } ]
```

## 4.4 Array.splice()

### 1.splice()原理

`splice()`方法可以接收任意个参数，它的第一个参数为切割数组的起始位置（start），第二个参数为我们需要切割的长度（length），而后面的参数则为我们需要添加到原数组上的值。splice 会改变原来的数组。在我们手写`splice()`方法时，我们要注意以下几点：

1. splice 方法是先切割数组，再向其添加我们的想要向原数组的值
2. splice 方法会改变原数组，同时也会返回出一个新数组
3. 当我们传进来的第一个参数为复数时，我们需要从（start+数组长度）的位置开始切割
4. 当我们切割的长度大于原数组长度的最大值时，我们要改变新数组的长度，去除新数组后面的`undefined`值

### 2.splice()方法的手写实现

```js
const singer = [
  { name: "111", num: 20 },
  { name: "222", num: 25 },
  { name: "333", num: 19 },
  { name: "444", num: 10 },
  { name: "555", num: 0 },
];

Array.prototype.my_splice = function (start, length, ...args) {
  const result = [],
    targetArr = []; // 新数组 原数组
  // 判断第一个参数是否小于零
  if (start < 0) {
    start = this.length + start;
  }
  // 生成新数组同时将原数组剩余的值和要添加的值给到 targetArr
  for (let i = 0; i < this.length; i++) {
    if (i === start) {
      for (let j = start; j < start + length; j++) {
        result.push(this[j]);
        i = j;
      }
      targetArr.push(...args);
      if (!length) {
        targetArr.push(this[i]);
      }
    } else {
      targetArr.push(this[i]);
    }
  }
  // 给原数组赋值
  for (let i = 0; i < targetArr.length; i++) {
    this[i] = targetArr[i];
  }
  // 去除新数组上的undefined
  if (start + length > this.length) {
    if (start === 0) {
      result.length = this.length;
    } else {
      result.length = this.length - 1 - start;
    }
  }
  this.length = targetArr.length;
  return result;
};
const arr = singer.my_splice(-2, 10, "hello", "world", "123");
console.log(singer);
// [
//   { name: '111', num: 20 },
//   { name: '222', num: 25 },
//   { name: '333', num: 19 },
//   'hello',
//   'world',
//   'laoli'
// ]
console.log(arr);
// [ { name: '444', num: 10 }, { name: '555', num: 0 } ]
```

## 4.5 Array.fill()

### 1.fill()原理

`fill()` 方法接受三个参数，分别是分别为 `val`（你要填充元素）、`start`（开始查找的位置）、`end`（结束的查找的位置），它是向数组中填充指定的元素。可以指定开始和结束填充的位置。如果未指定，则将填充所有元素。`fill()` 会覆盖原数组。要注意的一点是：**当我们填充一个引用类型的数据时，当我们改变了该引用类型的值时，数组的值也会改变**

### 2.fill()方法的手写实现

```js
let obj = { name: "666", num: 12 };
let singer = [
  { name: "111", num: 20 },
  { name: "222", num: 25 },
  { name: "333", num: 19 },
  { name: "444", num: 10 },
  { name: "555", num: 0 },
];
Array.prototype.my_fill = function (val, start = 0, end) {
  end = end || this.length;
  for (let i = start; i < end; i++) {
    this[i] = val;
  }
  return this;
};
singer.my_fill(obj, 0, 1);
console.log(singer[0]); //{ name: '666', num: 12 }

obj.num = 13;
console.log(singer[0]); // { name: '666', num: 12 }
```
