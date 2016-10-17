---
title: 记录一个JS严格模式遇到的问题
date: 2016-10-13 15:43:32
tags: [javascript,前端]
var123: 222
---

# 场景 
为微信小程序定制engine.io
# 描述

```js
(function(){return this})()
```
engine.io 使用webpack + babel 编译，其中一些依赖模块的写法并没有考虑 `use strict` 的情况。
上述代码在`use strict` 的情况下返回`undefined` 而不是全局对象。而webpack中使用 babel却自动在每个模块上自动添加`use strict` 。导致一些奇怪现象的发生。

经过测试，在微信小程序中，默认是启用strict 模式的，而webpack并没有解决这个问题，简直 pain in the ass，先保留这个问题，留在以后解决。

# update 20161017





问题是由

```js
...
function(global){
//your code
}(exports,function(){return this}())

...

```
代码快引起，所以一个思路是去掉这个闭包
<!-- more --> 
webpack 提供了 node.global 选项，如果把这个选项关闭的话webpack 将不再提供 global对象的闭包。也就是说直接暴露目标环境的global对象给程序。之前我的理解有偏差，要给global一个特殊的变量。实际上global是标准内置的。所以理论上可以在任何环境中使用global对象。

> 参考: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects Standard built-in objects
> The term "global objects" (or standard built-in objects) here is not to be confused with the global object. Here, global objects refer to objects in the global scope (but only if ECMAScript 5 strict mode is not used; in that case it returns undefined). The global object itself can be accessed using the this operator in the global scope. In fact, the global scope consists of the properties of the global object, including inherited properties, if any.

所以我把webpack的配置文件做一下修改
```
  node:{
    global:false//默认为true，
  },
```
加上这个选项后webpack就不会在生成的代码中加入这个恼人的闭包了。


# NEXT webpack为什么默认给一个闭包？

TODO

