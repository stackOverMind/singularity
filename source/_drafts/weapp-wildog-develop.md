---
title: 微信小程序分析
tags: [微信,小程序,野狗]
---

微信小程序
* strict mode
* 不支持npm


源码解析

wxjsbridge
wxjscore
mina
Labrador

flex布局

# 框架

从分析开发者工具开始，开发者工具本身并不在研究范围之内。所以直接研究小程序架构相关的内容：

微信小程序框架的源代码都可以在这里找到： `Contents/Resources/app.nw/app/dist/weapp`
目录结构如下：
```
├── appservice
│   ├── ascheck.js
│   ├── asdebug
│   │   ├── const
│   │   ├── hijack
│   │   ├── jsbridge
│   │   └── utils
│   └── asdebug.js
├── commit
│   ├── build.js
│   ├── getallwxss.js
│   ├── initAppConfig.js
│   ├── initAppServiceJs.js
│   ├── pack.js
│   ├── unpack.js
│   └── upload.js
├── newquick
│   ├── app.js
│   ├── app.json
│   ├── app.wxss
│   ├── pages
│   │   ├── index
│   │   │   ├── index.js
│   │   │   ├── index.wxml
│   │   │   └── index.wxss
│   │   └── logs
│   │       ├── logs.js
│   │       ├── logs.json
│   │       ├── logs.wxml
│   │       └── logs.wxss
│   └── utils
│       └── util.js
├── onlinevendor
│   ├── version.json
│   ├── WAService.js
│   ├── WAWebview.js
│   ├── wcc
│   └── wcsc
├── tpl
│   ├── appserviceErrorTpl.js
│   ├── appserviceTpl.js
│   ├── errorTpl.js
│   └── pageFrameTpl.js
├── trans
│   ├── transConfigToPf.js
│   ├── transManager.js
│   ├── transWxmlToHtml.js
│   ├── transWxmlToJs.js
│   └── transWxssToCss.js
├── utils
│   ├── projectManager.js
│   ├── tools.js
│   └── vendorManager.js
├── vendor
└── weApp.js
```

文件都是压缩过，变量名也被替换过，所以需要两个过程还原

1. 格式化，使用 js-beautify 一次性完成
2. 猜测变量名并替换，这个过程比较有技巧，需要靠逻辑关联去猜测。。。

所以，先从 `weApp.js` 入手，还原后文件：
https://github.com/stackOverMind/wetoolsource/blob/master/app_.js

这部分代码只有一个逻辑：作为一个server提供给客户端相应的js文件。
从这一点来看微信小程序还是个bs结构，运行依赖于一个提供静态资源的server，而不是把所有的资源打包到一起，这与微信宣称的无需更新是吻合的。而这部分代码依然不是小程序运行时环境，我们需要继续深入。






## 架构



## 线程


微信小程序的view和logic是运行在不同线程上的：

![](mina-lifecycle.png)

## 通信



# 误解：小程序的UI是原生的

不管是从一开始流出的内幕来看还是从现在技术发展的趋势来看，小程序的UI都应该是原生的。一开始我也这样推测。后来，随着深入了解原生UI的react native 并与小程序做对比后，产生了很多疑问。

* RN只支持很少的css，而weapp却宣称支持大多数css
* RN不支持canvas，而weapp却支持

分析原因有几种可能：

1. 微信比FB NB，使用native实现了一套跟浏览器内核差不多的渲染引擎
2. 微信比FB NB，实现native与html混合布局，页面中有的组件是HTML，有的组件是native
3. 微信没有比谁NB，所有的组件都是html5

怎么看都觉得第三种可能更靠谱一点，但是只是怀疑，并没有证据，知道看到这篇文章：
http://www.voidcn.com/blog/walid1992/article/p-6247160.html

![](./940_8d1_d7d.jpg)

很好理解，android有个开发者选项，中间可以打开一个显示渲染边界的开关。只要是native渲染的组件都会显示边界，但是html却不会，因为在操作系统看来，真个webview都是一个组件，这个组件内部要怎么显示是webview的工作，与系统无关。我们发现除了上面和下面的导航，中间的内容全是webview。

所以微信小程序是HTML5的。native中间可以嵌入webview，但HTML5中不能嵌入native，至少目前的技术不能，因为这涉及到谁来掌管layout和渲染。

如果简单的说微信小程序是HTML5 好像又不那么合适，至少在上边导航和下面导航使用的是native控件。那么我们简单的还原下微信的设计思路：

* 能用native的地方都用native
* native与html不能混排,（因为技术做不到）但fix定位可以

这样，我们就得出了能够用native的地方
* 上方和下方 navbar
* action-sheet
* modal
* toast
* loding

# strict mode

微信小程序默认在严格模式下运行，其中有些代码运行的行为是不一致的。回顾一下strict mode

* 为了防止意外创建全局变量，对没有声明的变量直接赋值会报错

```js
"use strict";
                       // 假如有一个全局变量叫做mistypedVariable
mistypedVaraible = 17; // 因为变量名拼写错误
                       // 这一行代码就会抛出 ReferenceError
```

* 在正常模式下静默失败的情况会报错,例如，给不可写变量赋值

```js
"use strict";

// 给不可写属性赋值
var obj1 = {};
Object.defineProperty(obj1, "x", { value: 42, writable: false });
obj1.x = 9; // 抛出TypeError错误
```

* 删除不可删除的属性会报错,删除变量会报错

```js
"use strict";
delete Object.prototype; // 抛出TypeError错误
```

...
...

* 在全局作用域中调用this会返回undefined！

这是我遇到的一个坑，如果使用webpack来为小程序的依赖库打包有可能也会遇到这个坑。

如果你的lib库使用了global变量，那么打包后的文件看起来是这个样子的

```js
...
function(global){
//your code
}(exports,function(){return this}())
...

```

在普通模式下，这个没有任何问题，但在strict 模式下 `global`会变成`undefined`。所以如果你的库在小程序中失效了，除了环境的原因外，你应该检查下是不是strict mode造成的。


```js


var array = []
ref.on('child_added',function(snapshot,preKey){
    if(array.length == 0 || prKey = array[array.lenght-1][".key"]){
        array.push({".value":snapshot.val(),".key":snapshot.key()})
    }
    var index = array.findIndex(function(el){
        if(el[".key"] === preKey){
            return true
        }
        return false
    })
    array.splice(index,0,{".value":snapshot.val(),".key":snapshot.key()})
})


```

```js

ref.push({
  'topic':'大象的鼻子为什么这么长'，
  'content':'xxxxxxxxxxx',
  'author': 'liu-jin'
})

```

```js

ref.orderByPriority().limitToLast(20).on('child_added')
.then((snapshot) => {
	console.log(snapshot.val())
})

```

```js

wilddog.sync().ref("/users/john/rank").transaction(function(currentRank) {
   // If currentRank = null, 直接返回默认值 0。
   if (currentRank == null) {
        return 0;
   } 
   return currentRank+1;
});

```

```js

ref.on('child_added',function(snapshot,prKey){
    //prKey 新添加的这条数据的上一条Key是什么
    var value = snapshot.val()
    var key = snapshot.key()
    //此处修改view
})

ref.on('child_removed',function(snapshot,prKey){
    //prKey 新删除的这条数据的上一条Key是什么
    var value = snapshot.val()
    var key = snapshot.key()
    //此处修改view
})

wilddog.sync().ref('players').
    orderByChild('score').
    limitToLast(10).
    on('value',function(snapshot){
        snapshot.forEach(function(snapshot){
            console.log(snapshot.val())
        })
    })

// user1 端代码
Wilddog.sync().ref('user-a/status').onDisconnect().set('offline')
var connectedRef = wilddog.sync().ref("/.info/connected");
connectedRef.on("value", function(snap) {
  if (snap.val() === 'true') {
    console.log("I am online");
    Wilddog.sync().ref('user-a/status').set('offline')
  } else {
    console.log("I am offline");
  }
});

// user2 端代码
Wilddog.sync().ref('user-a/status').on('value',function(snapshot){
    if(snapshot.val()=='offline'){
        console.log('user-A is offline')
    }
    else{
        console.log('user-A is online')
    }
})



```

