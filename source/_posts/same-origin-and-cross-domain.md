---
title: 同源和跨域
date: 2018-10-17 23:16:26
categories:
  - FE
tags:
  - Browser
---

实际开发中，经常会遇到需要发起跨域请求的时候，总的来说还是有不少解决办法的，说到需要跨域的原因就不得不提同源政策了，还有跨域的究极解法 CORS。

<!-- more -->

### 什么是同源政策（`Same-Origin Policy`）

- 协议相同

- 域名相同

- 端口相同

> 只要协议、域名、端口有任何一个不同，都被当作是不同的源。

更详细的解释可以看[这里](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)

### 为什么要有同源政策

同源策略限制了从同一个源加载的文档或脚本如何与来自另一个源的资源进行交互。这是一个用于隔离潜在恶意文件的重要安全机制。而且保护用户隐私信息，防止身份伪造等(读取 Cookie)。

### document.domain

对于主域相同，子域不同的情况，可以设置 `document.domain` 来规避同源政策。

```Javascript
// 对于 blog.tinykid.org 和 dev.tiny.org，可以通过如下设置获取 cookie
document.domain = 'tinykid.org'
```

### location.hash

这种方式是子框架具有修改父框架 src 的 hash 值，通过这个属性进行传递数据，且更改 hash 值，页面不会刷新。但是传递的数据的字节数是有限的。

```Javascript
// 父窗口
iframe = document.createElement('iframe')
iframe.style.display = 'none'
var state = 0

iframe.onload = function() {
  if (state === 1) {
    // 获取数据
    var data = window.location.hash
    // 销毁 iframe
    iframe.contentWindow.document.write('')
    iframe.contentWindow.close()
    document.body.removeChild(iframe)
  } else if (state === 0) {
    state = 1
    iframe.contentWindow.location = 'http://tinykid.org/xxx.html'
  }
}
document.body.appendChild(iframe)


// xxx 页面中
parent.location.hash = "data";
```

### window.postMessage

这个方法是 HTML5 中引入的一个新 API，用于跨域的父子窗口通信，不受同源策略限制。通过它可以实现对存储的读写，DOM 的操作等。

```Javascript
// 弹出一个新窗口
var popup = window.open('http://child.com');

//父窗口向子窗口发送消息
popup.postMessage('Hello World!', 'http://child.com');

//子窗口向父窗口发送消息
window.opener.postMessage('Nice to see you', 'http://parent.com');

//父子都可以监听 message 事件响应
window.addEventListener('message', function(e) {
  // do something
}, false);
```

### WebSocket

WebSockets 是一个可以创建和服务器间进行双向会话的高级技术。通过这个 API 你可以向服务器发送消息并接受基于事件驱动的响应，这样就不用向服务器轮询获取数据了。更多信息点[这里](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSockets_API)。

### JSONP

JSONP 虽然很好用，但是只支持 Get 方法，其思路是 script 标签是没有同源限制的，所以可以利用这点来发起跨域请求。看代码：

```Javascript
;(function(global) {
  var id = 0,
    container = document.getElementsByTagName('head')[0]

  function jsonp(options) {
    if (!options || !options.url) return

    var scriptNode = document.createElement('script'),
      data = options.data || {},
      url = options.url,
      callback = options.callback,
      fnName = 'jsonp' + id++

    // 添加回调函数
    data['callback'] = fnName

    // 拼接url
    var params = []
    for (var key in data) {
      params.push(encodeURIComponent(key) + '=' + encodeURIComponent(data[key]))
    }
    url = url.indexOf('?') > 0 ? url + '&' : url + '?'
    url += params.join('&')
    scriptNode.src = url

    // 传递的是一个匿名的回调函数，要执行的话，暴露为一个全局方法
    global[fnName] = function(ret) {
      callback && callback(ret)
      container.removeChild(scriptNode)
      delete global[fnName]
    }

    // 出错处理
    scriptNode.onerror = function() {
      callback &&
        callback({
          error: 'error'
        })
      container.removeChild(scriptNode)
      global[fnName] && delete global[fnName]
    }

    scriptNode.type = 'text/javascript'
    container.appendChild(scriptNode)
  }

  global.jsonp = jsonp
})(this)
```

### CORS

CORS 可以说是跨域的究极解法方法，由浏览器自动完成。这个机制允许 Web 应用服务器进行跨域访问控制，从而使跨域数据传输得以安全进行。浏览器支持在 API 容器中（例如 XMLHttpRequest 或 Fetch ）使用 CORS，以降低跨域 HTTP 请求所带来的风险。具体描述可以看这里。

由于 MDN 已经讲解的非常仔细了，我就说一下注意事项。

对于客户端，需要注意设置 `xhr.withCredentials = true`，不然无法携带 cookie。

对于服务端需要返回两个关键字段，`Access-Control-Allow-Origin` 和 `Access-Control-Allow-Credentials`，这样基本就能顺利完成跨域请求。
