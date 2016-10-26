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

下面是一个例子， `Persion` 模块接收js的调用并且也调用js

```obj-c
@interface Person : NSObject <RCTBridgeModule>
@end

@implementation Logger

RCT_EXPORT_MODULE()

RCT_EXPORT_METHOD(greet:(NSString *)name)
{
  NSLog(@"Hi, %@!", name);
  [_bridge.eventDispatcher sendAppEventWithName:@"greeted"
                                           body:@{ @"name": name }];
}

@end
```

我们将集中注意到两个宏上 `RCT_EXPORT_MODULE` 和`RCT_EXPORT_METHOD`。

## `RCT_EXPORT_MODULE([js_name])`

定义如下：

```obj-c
#define RCT_EXPORT_MODULE(js_name) \
  RCT_EXTERN void RCTRegisterModule(Class); \
  + (NSString \*)moduleName { return @#js_name; } \
  + (void)load { RCTRegisterModule(self); }

```

这段代码做了什么：

* 首先声明`RCTRegisterModule` 是个extern function，意味着这个函数的实现在编译时不可见，在连接时才可见
* 声明了一个方法`moduleName`，返回可选的宏参数`js_name`。如果你在js中想用其他的名字来调用这个方法
* 声明一个`load` 方法（当app被载入到内存里的时候，它会对每个`class`调用`load`）

## `RCT_EXPORT_METHOD(method)`

这个宏更“有意思”。这个宏没有做任何事情，只是重新定义了一个方法，重新定义了一个方法名。

```obj-c
+ (NSArray *)__rct_export__120
{
  return @[ @"", @"log:(NSString *)message" ];
}

```

