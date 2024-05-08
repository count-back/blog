---
title: JS——继承
date: 2021-07-10
author: 倒数
tags:
  - 继承
categories:
  - javaScript
isShowComments: true
subSidebar: auto
---

# 1. 前言

首先我们要知道**继承是一种代码重用的形式**，**JavaScript 又是一门基于原型的语言**，这就意味着对象直接从其他对象继承。为了显著的减少软件开发的成本，提高代码的重用性，JavaScript 的继承方式也是我们要学会的知识。那下面我就来介绍 JavaScript 6 种常用的继承方式。

## 2. 原型链继承

原型链继承就是让一个构造函数的原型是继承于另一个构造函数的实例，这样这个构造函数 new 出来的实例对象就会有另一个构造函数的属性。

```js
SuperType.prototype.getSuperValue = function () {
  return this.property;
};
function SuperType(t) {
  this.property = t;
  this.like = {
    sport: "running",
    book: "js",
  };
  this.age = 18;
}
Type.prototype = new SuperType();
function Type(t) {
  this.typeproperty = t;
}
var instance1 = new Type(false);
instance1.age = 20;

console.log(instance1);
```

![图片.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/020c6f50ce674beb922da110445320e7~tplv-k3u1fbpfcp-watermark.image?)

如图所示，我们有一个`SuperType`构造函数，然后又创建了`Type`构造函数，同时让`Type`的原型赋值为`SuperType` new 出来的的实例对象，这样我们用`Type`new 出来的实例对象原型上就可访问到`SuperType`的属性。这也就是原型链继承。

- 优点：写法简单，容易理解
- 缺点：
- 1.会在子类实例上共享父类所有的引用类型实例属性
- 2.不能给父类构造函数传参

## 3. 经典继承(伪造对象）

经典继承就是在子类的构造函数内部调用父类构造函数，再使用`call()`方法将父类的构造函数绑定到子类上。

```js
SuperType.prototype.val = 123;
function SuperType() {
  this.colors = ["red", "blue", "green"];
}
function Type(t) {
  SuperType.call(this, t);
}
var instance1 = new Type();
instance1.colors.push("pink");
console.log(instance1);
var instance2 = new Type();
console.log(instance2);
```

![图片.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa83dc6b63f2487eb66a25925a9a1c01~tplv-k3u1fbpfcp-watermark.image?)

这样我们就是在子类构造函数`Type`继承到父类`SuperType`的`color`属性。

- 优点 -解决了原型链不能传参的问题
- 缺点
- 1.方法都在父构造函数中定义，无法实现函数复用
- 2.子类继承不到父类原型上的属性

## 4. 组合继承（伪经典继承）

组合继承就是将**原型链继承和经典继承组合结合起来**，使用原型链实现原型属性的继承，使用构造函数来实现对实例对象的继承。同时我们还可以实现函数的复用。

```js
SuperType.prototype.sayName = function () {
  console.log(this.name);
};
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
function Type(age, name) {
  this.age = age;
  SuperType.call(this, name);
}
Type.prototype = new SuperType();
Type.prototype.constructor = Type;
Type.prototype.sayAge = function () {
  console.log(this.age);
};
var instance1 = new Type(18, "zhangshan");
instance1.sayName();
instance1.sayAge();
console.log(instance1);
```

![图片.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76afb1edb42d4fb98d5bb4d83207ca57~tplv-k3u1fbpfcp-watermark.image?)

我们通过了原型链继承拿到了父类的原型上的属性，又通过经典继承拿到了父类实例对象的属性。

- 优点：
- 解决了原型链继承和经典继承的缺陷。
- 缺点：重复调用了两次父类的构造函数。一次是在创建子类原型时，另一次在子类构造函数内部

## 5. 原型式继承

原型式继承是借用一个函数传入一个对象作为参数，再在其内部创建一个临时的构造函数，然后将**参数作为这个构造函数的原型**，最后返回出这个构造函数的实例对象。

```js
function object(o) {
  function F() {}
  F.prototype = o;
  return new F();
}
var person = {
  name: "张三",
  age: 18,
  like: {
    sport: "running",
  },
};
var obj = object(person);
obj.like.sport = "singing";
console.log(obj);
```

![图片.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a6b4c149595429eaa79cc1bf7b90364~tplv-k3u1fbpfcp-watermark.image?)

这样，我们`obj`对象就继承到了`person`身上的属性。

- 优点：
- 不需要单独创建构造函数
- 缺点：
- 属性中包含的引用值始终会在相关对象间共享，同时子类实例不能向父类传参

## 6. 寄生式继承

寄生式继承是**创建一个仅用于封装继承过程的函数，该函数在内部以某种方式来增强对象，最后再返回对象**，优缺点和原型式继承相同。

```js
function createPerson(original) {
  var clone = Object.create(original);
  clone.say = function () {
    // 增强这个对象
    console.log("hello");
  };
  return clone;
}

let obj = createPerson({ name: "zhangsan" });
obj.say();
console.log(obj);
```

![图片.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d25a320b284f429582bf59d20647732b~tplv-k3u1fbpfcp-watermark.image?)

## 7. 寄生组合式继承

寄生组合式继承是通过借用构造函数来继承属性，通过原型链的混成形式来继承方法。本质上就是使用寄生式继承来继承父类的原型，然后再将结果指定给子类型的原型

```js
SuperType.prototype.sayName = function () {
  console.log(this.name);
};
SuperType.prototype.like = {
  a: 1,
  b: 2,
};
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
function Type(name, age) {
  this.age = age;
  SuperType.call(this, name);
}
var anotherPrototype = Object.assign(Type.prototype, SuperType.prototype);
Type.prototype = anotherPrototype; // new SuperType()
var instance1 = new Type("Tom", 12);
// instance1.colors.push('pink')
instance1.like.a = 11;
instance1.sayName();
console.log(instance1);
```

![图片.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a087f2f8202d44b88b9315c08ca25b48~tplv-k3u1fbpfcp-watermark.image?)

- 优点：
- 只调用了一次父类构造函数，并且避免了子类原型上创建多余的属性。还可以保持原型链不变。（**继承最优解**）
- 缺点：
- 代码复杂
