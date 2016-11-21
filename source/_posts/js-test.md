---
title: js-test
date: 2016-01-01 15:40:39
tags:
---



1. 请写下这段代码的输出结果


```js
for(var i = 1;i<10;i++){
    setTimeout(function(){
        console.log(i)
    })
}
```

2. 请用ES6的 class 改写下面代码

```js
var A = function(p1){
    this.p1 = p1
}
A.prototype.hello = function(){
    console.log('hello world')
}
```
3. 输出以下代码结果

```
console.log(null === undefined )
console.log(!!"" == !1)
```

4. 输出以下代码结果

```js
var b = function(){
    console.log(this)
}
a.b = b;
a.b()
b()
b.call()
b.call(a)

```

5. 输出以下代码的结果

```js
[1,2,3,4,56].fiter((a) => {
    return a<5
}).map((a){
    return a*a
}).reduce(function(a,b){
    return a + b
})

```

6. 写出堆排序的（伪）代码