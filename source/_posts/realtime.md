---
title: web实时化（iweb ppt内容）
date: 2016-09-24 00:16:13
tags: [前端,实时化]
---

本来说好的周日更新，但一直没时间。周一又是一周最忙的一天，所以长话短说，尽量简单阐述观点

以下正文：

# 非实时的web


`word wide web` (以下简称web)是1989年 由 TimBL在 [CERN](https://en.wikipedia.org/wiki/CERN)发明的。web现在已经是一个内容平台，应用平台。但是想要了解web为什么是现在的这种工作机制就必须回到本源，web其实只是被设计成为一个信息管理系统(information management system)。对于一个信息关系系统而言，不需要持续的交互，只需要一次性下载文档就可以，所以http协议的特点是：

* 单向访问，只支持客户端主动访问服务端，而服务端不能把消息主动推送给客户端
* 无连接，一次请求就建立一个连接，一次连接只服务一个请求
* 无状态，连接本身不带有上下文信息，服务端也不保存

> 对于历史的部分感兴趣的话可以wikipedia:
> https://en.wikipedia.org/wiki/Tim_Berners-Lee



![](/img/ppt-realtime-1.png)

![](/img/ppt-realtime-2.png)

![](/img/ppt-realtime-3.png)

![](/img/ppt-realtime-4.png)

![](/img/ppt-realtime-5.png)

![](/img/ppt-realtime-6.png)

![](/img/ppt-realtime-7.png)

![](/img/ppt-realtime-8.png)