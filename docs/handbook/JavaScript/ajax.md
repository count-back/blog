---
title: JS——手写ajax
date: 2022-09-25
author: 倒数
tags:
  - javaScript
categories:
  - javaScript
isShowComments: true
subSidebar: auto
---

# 前言

在我们手撸 ajax 之前，我们先来 Ajax 吧！

### Ajax 是什么

在我们写前端代码时，`我们是不是都苦恼着如何做前后端互通呢？`而今天的主角就是帮我们完成这件事的。所以 Ajax 的作用就是用于浏览器与服务器之间使用异步数据传输(HTTP 请求),做到局部请求以实现局部刷新。让我们可以请求到服务端的数据用于渲染浏览器端。

### Ajax 的作用

1. 不刷新页面而更新网页（`局部刷新`）
2. 在页面加载后从服务器`请求`数据
3. 在页面加载后从服务器`接受`数据
4. 在后端向服务器发送数据

## 原生 JS 向后端请求方式

```js
//原生js提供的接口请求方法
let xhr = new XMLHttpRequest();
xhr.open(method, url, async, [user - name], [user - pass]);
xhr.onreadystatechange = function () {
  //监听
  if (xhr.readyState === 4 && xhr.status === 200) {
    console.log(xhr.responseText);
  }
};
xhr.send([请求主题]);
```

原生 JS 提供了一个 **XMLHttpRequest** 方法。我们直接 new 一个 xhr 实例对象。然后我们直接调用**XMLHttpRequest**上的 open 方法。同时在`open`方法中携带 method、url、async（async=false 为同步执行）、[user-name]、[user-pass]（可选、url 的认证资格)。

#### http 请求方式（method）

- GET: 从服务器获取数据（向服务器推送数据）
- POST: 向服务器推送数据（从服务器获取数据）
- DELETE：删除服务器的某些内容
- PUT： 向服务器存放一些内容
- HEAD:   只想获取服务器返回的响应头信息，不要响应体
- OPTIONS: 一般使用它向服务器发送一个探测请求，如果返回的信息，说明客户端和服务器建立了连接，可以继续请求
- TRACE:   基于 cross - domain 进行跨域请求的时候

之后我们继续可以调用**XMLHttpRequest** 的`onreadystatechange`方法用来监听的状态。根据状态码来判断请求是否成功和返回数据。

### readyState 的状态

- 0: 刚刚创建 xhr
- 1：执行了 open 这个操作
- 2：请求已经发送，响应头已经被客户端接受
- 3: 响应主题正在返回     // 还没被接收
- 4：响应主体已经被客户端接收

### http 状态码

- 1xx: 请求已经接受，正在处理
- 2xx：成功。请求被成功接受
- 3xx：成功，资源已经重定向了  // 换了个 url
- 4xx：客户端错误
- 5xx: 服务端错误

最后我们调用**XMLHttpRequest** 的`send`方法根据情况向后端传递参数。这样我们既完成了原生的 JS 请求方式。

### 我们来一个实例

```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <!--  原生的请求方式 -->
    <script>
        let xhr = new XMLHttpRequest()
        xhr.open('GET', 'https://www.fastmock.site/mock/39ac87de3060aa2bb2ba20a0ff375c81/cat-movie/mostLike')
        xhr.onreadystatechange = function () {     //监听
            if (xhr.readyState === 4 && xhr.status === 200) {
                console.log(JSON.parse(xhr.responseText));
            }
        }
        xhr.send()
    </script>
</body>
</html>
```

![屏幕截图(86).png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b688435a52f438b8addde0927f72499~tplv-k3u1fbpfcp-watermark.image?)

## 直接使用封装好的 Ajax 方法

我们先来看一看封装好的 Ajax 方法的使用方式，我们同样是可以向后端请求到数据，而且会你原生 JS 的请求方式更加的便捷。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
       
    <meta charset="UTF-8" />
       
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
       
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
       
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.1/jquery.min.js"></script>
       
    <title>Document</title>
  </head>
  <body>
        <
    <script>
      $.ajax({
        url: "https://www.fastmock.site/mock/39ac87de3060aa2bb2ba20a0ff375c81/cat-movie/mostLike",
        method: "GET",
        dataType: "json",
        data: null,
        async: true,
        cache: true,
        success: (res) => {
          console.log(res);
        },
      });
    </script>
  </body>
</html>
```

![屏幕截图(87).png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a22c35a3c2344e18b6647c71ee9d0c3f~tplv-k3u1fbpfcp-watermark.image?)

## 手撸一个简化版 Ajax

在看完直接封装的 Ajax 方法，那我们是否好奇它是怎么实现的呢？那接下来我们就来自己封装一个简化版的 Ajax 来看看他的实现原理吧。

```html
HTML代码：
<!DOCTYPE html>
<html lang="en">
  <head>
       
    <meta charset="UTF-8" />
       
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
       
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
       
    <script src="ajax.js"></script>
       
    <title>Document</title>
  </head>
  <body>
       
    <script>
      ajax({
        url: "https://www.fastmock.site/mock/39ac87de3060aa2bb2ba20a0ff375c81/cat-movie/mostLike",
        method: "GET",
        dataType: "json",
        data: {
          name: "abc",
          age: 10,
        },
        async: true,
        cache: true,
        success: (res) => {
          console.log(res);
        },
      });
    </script>
  </body>
</html>
```

```js
JS代码部分: function ajax(options) {
  let {
    // 结构前端参数
    url,
    method = "GET",
    data = null,
    dataType = "JSON",
    async = true,
    cache = true,
    success,
    error,
  } = options;
  let xhr = new XMLHttpRequest(); // 处理参数
  if (method.toUpperCase() === "GET") {
    // 处理不同的请求方式
    if (data) {
      url += "?";
      for (let key in data) {
        url += `${key}=${data[key]}&`;
      }
      xhr.open(method, url, async);
      xhr.send();
    }
  } else {
    //post
    xhr.open(method, url, async);
    xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded"); // 请求头
    xhr.send(data);
  }
  xhr.onreadystatechange = () => {
    // 监听请求是否成功，并返回数据
    if (xhr.readyState === 4) {
      if (!/^(2|3)\d{2}$/.test(xhr.status)) {
        error && error(xhr.statusText, xhr);
      }
      let result = handleDataType(xhr); //格式化处理
      success && success(result, xhr);
    }
  };
  function handleDataType(xhr) {
    //拿到前端需要的数据格式，进行格式化处理
    dataType = dataType.toUpperCase();
    let result = xhr.responseText;
    switch (dataType) {
      case "TEXT":
        break;
      case "JSON":
        result = JSON.parse(result);
        break;
      case "XML":
        result = xhr.responseXML;
        break;
      default:
    }
    return result;
  }
}
```

在通过简单的封装，我们自己的封装的 Ajax 也能实现 JQ 里封装好的 ajax 的部分功能。

![屏幕截图(88).png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed9e978676e3481c95585742e50918b5~tplv-k3u1fbpfcp-watermark.image?)
