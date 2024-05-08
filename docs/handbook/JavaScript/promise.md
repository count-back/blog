---
title: JS——简化Promise的实现
date: 2022-08-29
author: 倒数
tags:
  - promise
categories:
  - javaScript
isShowComments: true
subSidebar: auto
---

# 第一步：搭建 Promise 框架

```js
(function (window) {
  // 值执行函数
  //在promise原型上挂载then方法
  myPromise.prototype.then = function (onResolved, onRejected) {};

  function MyPromise(executor) {
    // 构建promise函数

    function resolve(value) {} // promise内部的resolve函数

    function reject(value) {}

    try {
      executor(resolve, reject); //参数为实参
    } catch (error) {
      // 如果promise内代码执行出错，将错误抛出

      reject(error);
    }
  }

  window.myPromise = myPromise; //将promise挂载到window全局对象上
})(window);
```

首先我们要了解 Promise 函数中帮我们**干了啥事**：

- 1. 有一个 promise 构造函数
- 2. 可以在 promise 后可以接.then()函数
- 3. 接收了一个有两个参数的回调函数
- 4. 参数在 promise 中作为函数调用

这就是 promise 帮我干的事情，所以我们也就根据它帮我们干的事，我们来搭建**promise 框架**。

- 1. 先写一个 promise 构造函数
- 2. 在 promise 构造函数的原型上写.then()方法
- 3. 在 promise 函数把传进来的参数作为函数调用掉，同时将我们在 promise 写的函数作为实参传进去给回调函数使用
- 4. 在 promise 中定义这两个函数，供回调函数调用

# 第二步：明确 Promise 状态的变更

在我们使用 Promise，我们需要知道的一个很重要的知识点就是，Promise 现在是处于什么状态？首先我们要知道 Promise 有三种状态：

- pending
- resloved（fulfilled）
- rejected

接下来我们再来看看**Promise 状态的特点**：

1. `Promise`对象的状态不受外界影响。`Promise`对象代表一个异步操作，有三种状态：`pending`（进行中）、`fulfilled`（已成功）和`rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。

2. 一旦状态改变，就不会再变，任何时候都可以得到这个结果。`Promise`对象的状态改变，只有两种可能：从`pending`变为`fulfilled`和从`pending`变为`rejected`。

在明确完 Promise 的状态后，我们也就明白了我们需要在`resolve`和`reject`函数中需要去改变`Promise`的状态。

# 第三步：完善 Promise 构造函数

```js
function MyPromise(executor) {
  let self = this;
  self.status = "pending"; //  Promise的状态
  self.data = undefined; // resolve或reject函数调用传进来的值
  self.callbacks = []; // 在Promise未改变时，存.then函数中未执行的函数

  function resolve(value) {
    if (self.status !== "pending") {
      // 确定Promise未改变
      return;
    }
    self.status = "resolved"; // 改变Pormise的状态
    self.data = value; // 在resolve函数调用是将传进来的值存起来
    // 执行待执行的callback函数
    if (self.callbacks.length > 0) {
      setTimeout(() => {
        self.callbacks.forEach((callbackObj) => {
          callbackObj.onResolved(value);
        });
      });
    }
  }
  //原理同上
  function reject(value) {
    if (self.status !== "pending") {
      return;
    }
    self.status = "resolved";
    self.data = value;
    if (self.callbacks.length > 0) {
      setTimeout(() => {
        self.callbacks.forEach((callbackObj) => {
          callbackObj.onResolved(value);
        });
      });
    }
  }

  try {
    executor(resolve, reject);
  } catch (error) {
    //如果Promise内部出现错误，执行reject函数捕获错误并抛出。

    reject(error);
  }
}
```

在`Promise`构造函数中，我们首先要在`Promise`函数上挂上`状态`、`盛放传递的实参值得容器`、`盛放未执行函数的数组容器`这三个属性。接下来我们就需要完善我们`resolve`和`reject`函数的逻辑了。

- 首先我们要通过判断`Promise`函数的状态来确定我们的`resolve`或`reject`函数是否执行。
- 如果执行的话我们就**需要改变 Promise 的状态**了，同时将传进来的实参值存起来。方便 then 函数取用。
- 接下来我们就需要判断一下**callbacks 数组中还有没有因为 Promise 函数状态未改变而被挂起的函数**。我们需要将其执行掉。
- 最后将`resolve`和`reject`函数作为实参供回调函数里的参数调用。

# 第四步：完善 then 函数

```js
//then函数接受两个回调函数作为参数
myPromise.prototype.then = function(onResolved, onRejected) {
        // 把回调用对象包裹存在callbacks中
        let self = this
        if(self.status === 'pending') {
            this.callbacks.push({
                onResolved,
                onRejected
            })
        }else if (self.status === 'resolved') {
           setTimeout(() => {
             onResolved(self.data)
           });
        }else{
            setTimeout(() => {
                onRejected(self.data)
               });
        }
       })
```

在 then 函数中，我们首要的任务就是判断**Promise 的状态**，来确定我们我们的回调函数该怎么执行。总共有三种情况：

- 1.如果状态为 pending 时，我们就要将 then 函数中的回调函数挂起。
- 2.如果状态为 resolved 时，我们就需要自己将 onResolved 回调函数执行掉
- 3.如果状态为 rejected 时，我们即执行 onRejected 回调函数

这样封装完，我们就可以实现 promise.then 的效果吗？下面我们来看看效果：

![屏幕截图(94).png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c59410c171bb449c89b03a57e5d81561~tplv-k3u1fbpfcp-watermark.image?)

我们可以看到，我们成功的打印出了`onResolved 1`, 这也就说明我们写的`Promise`没有问题, 但我们封装成这样就够了吗？

# 第五步：实现 then 函数后可以继续接 then 函数

当我们把 Promise 封装成这样显然是不够的，因为我们还没有实现在 then 函数后在接.then(), 所以我们还需要完善我们的 then 函数，让我们的 then 函数中也可以接.then()。

```js
myPromise.prototype.then = function (onResolved, onRejected) {
  // 把回调用对象包裹存在callbacks中
  let self = this;
  return new myPromise((resolve, reject) => {
    if (self.status === "pending") {
      this.callbacks.push({
        onResolved,
        onRejected,
      });
    } else if (self.status === "resolved") {
      setTimeout(() => {
        const result = onResolved(self.data);
        if (result instanceof myPromise) {
          // 没有额外的return
          result.then(
            // 为了将result的状态变更成resolved
            (val) => {
              resolve(val);
            },
            (err) => {
              reject(err);
            }
          );
        } else {
          resolve(result);
        }
      });
    } else {
      setTimeout(() => {
        onRejected(self.data);
      });
    }
  });
};
```

如果我们想在`.then后面在接.then`的话，首先我们就需要在我们`第一个then`中有返回值，这样我们才可以在我们`第二个.then`中拿到前面的值，所以第一件事，我们需要在我们的**then 函数中返回出一个 Pormise 对象**。
接下来，我们就要去处理一下我们第一个.then 中的回调函数，所以在我们的 then 函数中干了第二件事，**声明了一个 result 承接.then 中的返回值**，然后又在我们的 then 函数中干了第三件事，**判断 result 的类型，如果 result 是一个 Promise 对象，我们就去把在去调用一次 then 函数，否则我们就直接执行 resolve 即可**。

接下来我们就来看看效果：

![屏幕截图(95).png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc7f9d63d6544f8e82aa5f961adbdcae~tplv-k3u1fbpfcp-watermark.image?)

如上图，我们成功的打印出了 `ok res2`，也就说明，我们实现了在我们封装的 Promise 可以在后面接.then。

# 总结

写到这里，我们也就顺利的实现了`简化版Promise`的封装，也实现其主要功能，虽然我们没有封装的 Promise 没有官方版那么强大，但是我们可以清晰的看到`Promise`的原理

# 补充（完整代码）

```js
(function (window) {
  myPromise.prototype.then = function (onResolved, onRejected) {
    let self = this;
    return new myPromise((resolve, reject) => {
      if (self.status === "pending") {
        this.callbacks.push({
          onResolved,
          onRejected,
        });
      } else if (self.status === "resolved") {
        setTimeout(() => {
          const result = onResolved(self.data);
          if (result instanceof myPromise) {
            // 没有额外的return
            result.then(
              // 为了将result的状态变更成resolved
              (val) => {
                resolve(val);
              },
              (err) => {
                reject(err);
              }
            );
          } else {
            resolve(result);
          }
        });
      } else {
        setTimeout(() => {
          onRejected(self.data);
        });
      }
    });
  };
  function myPromise(executor) {
    let self = this;
    self.status = "pending";
    self.data = undefined;
    self.callbacks = [];
    function resolve(value) {
      if (self.status !== "pending") {
        return;
      }
      self.status = "resolved";
      self.data = value;
      if (self.callbacks.length > 0) {
        setTimeout(() => {
          self.callbacks.forEach((callbackObj) => {
            callbackObj.onResolved(value);
          });
        });
      }
    }
    function reject(value) {
      if (self.status !== "pending") {
        return;
      }
      self.status = "reject";
      self.data = value;
      if (self.callbacks.length > 0) {
        setTimeout(() => {
          self.callbacks.forEach((callbackObj) => {
            callbackObj.onRejected(value);
          });
        });
      }
    }
    try {
      executor(resolve, reject);
    } catch (error) {
      reject(error);
    }
  }
  window.myPromise = myPromise;
})(window);
```
