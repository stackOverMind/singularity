---
title: 记录将微信小程序的API适配到W3C标准
date: 2016-10-20 11:16:04
tags:
---


# 关于标准化
标准化是现代化生产的基础，所有厂商公用一套标准可以大大降低生产成本。所以世界上有各种各样的标准化组织，软件或互联网行业也是一样，发展成熟的标志就是所有的厂商都遵循一套标准。

微信小程序虽然支持commonjs,却不支持npm，另外API完全是微信私有的标准。这让众多的已有模块和开发者想说爱你不容易。为了更好的复用已有模块，我开始了一个项目，用来把相同功能的API适配到标准w3c协议上。

# 项目地址
 https://github.com/stackOverMind/WeApp-adapter

# 遇到问题

* 不可能适配所有的API，微信小程序并不是浏览器环境
* 能够适配的API也不能完美适配

# 具体流程

## 项目搭建
由于工作的需要，首先进行的是对websocket的适配。项目搭建还是使用自己比较熟悉和流行的webpack。注意到很重要的一点信息：微信小程序支持es6的class，并且微信小程序默认在 `strict mode`运行。所以我决定直接使用es6编写业务代码。

webpack.config.js:
<!-- more -->
```js
module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'weapp-adapter.js',
    path: './',
    libraryTarget: "commonjs2",
  },
  module: {
    loaders: [
      {
        test: /\.js$/,
        exclude: /(node_modules|bower_components)/,
        loader: 'babel', // 'babel-loader' is also a valid name to reference
        query: {
          presets: ['es2015']
        }
      }
    ]
  }
};
```

## 适配websocket

所有实现尽可能遵循这篇文档 https://www.w3.org/TR/2011/WD-websockets-20110419/
首先根据IDL定义class

```
interface WebSocket {
  readonly attribute DOMString url;

  // ready state
  const unsigned short CONNECTING = 0;
  const unsigned short OPEN = 1;
  const unsigned short CLOSING = 2;
  const unsigned short CLOSED = 3;
  readonly attribute unsigned short readyState;
  readonly attribute unsigned long bufferedAmount;

  // networking
           attribute Function onopen;
           attribute Function onmessage;
           attribute Function onerror;
           attribute Function onclose;
  readonly attribute DOMString protocol;
  void send(in DOMString data);
  void close();
};
WebSocket implements EventTarget;
```

值得注意的是WebSocket实现`EventTarget`。好在`EventTarget`已经有人帮我实现了，拿来就用。

然后就是注意行为了，比如初始化的时候在什么情况下抛什么错误，状态如何改变。事件何时触发，事件中的参数如何表示等等。总体而言，适配websocket的过程还是比较简单的。

有一些东西我给回避了，比如bufferedAmount。

# 适配XHR

先留坑，以后填









