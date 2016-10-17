---
title: 记录一个JS严格模式遇到的问题
date: 2016-10-13 15:43:32
tags: [javascript,前端]
var123: 222
---

场景：为微信小程序定制engine.io
描述：
```js
(function(){return this})()
```
engine.io 使用webpack + babel 编译，其中一些依赖模块的写法并没有考虑 `use strict` 的情况。
上述代码在`use strict` 的情况下返回`undefined` 而不是全局对象。而webpack中使用 babel却自动在每个模块上自动添加`use strict` 。导致一些奇怪现象的发生。

经过测试，在微信小程序中，默认是启用strict 模式的，而webpack并没有解决这个问题，简直 pain in the ass，先保留这个问题，留在以后解决。

