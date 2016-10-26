---
title: react native 中的桥连
date: 2016-10-26 11:27:55
tags: [react-native]
---

> 本文翻译自： http://tadeuzagallo.com/blog/react-native-bridge/

这篇文章假设你已经了解react-native 的基础，希望了解react-native内部javascript与native是如何通信的。

# 主要线程

在开始之前，记住一点，在react native 中有三个主要线程：

* `shadow queue`:布局运行的线程
* `main thread`：UIkit运行的线程
* `javascript thread`: js 代码运行的线程

另外，每个native module 都有自己的 `GCD Queue`，除非另有指定

*`shadow queue` 其实是个 GCD Queue,而不像名字中的线程*

# Native Modules

