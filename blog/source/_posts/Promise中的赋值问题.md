---
title: Promise中的赋值问题
date: 2020-09-21 13:32:29
tags: JavaScript
categories: JavaScript
---

# Promise.prototype.then中的赋值问题

在使用promise的过程中，我遇到了一个很有意思的问题，具体如下

```javascript
let a
function func() {
    let p = new Promise((resolve, result) => {
    	resolve(1)
    })
    p.then(res => a = res)
    console.log(a)
}

func() // undefined
```

我很好奇，为什么会取不到值呢？突然想起来，好像之前做项目的时候，使用axios拿到结果也没办法赋值的情况，当时因为时间紧急，也没有解决这个问题。现在又遇到同样的问题，缘，妙不可言？我思考了一段时间，排除了作用域的问题，因为在then中是可以访问到的，并且也不存在私有作用域，再从代码执行的角度取看它，果不其然，问题找到了，根据**事件循环**可以知道，promise是异步的，then是微任务，要等主代码块执行完毕才会在任务栈中取事件执行，问题找到了就很好解决了，使用同步的方法重写一下，我偏爱async / await。

```javascript
let a
async function func() {
    let p = new Promise((resolve, result) => {
    	resolve(1)
    })
    await p.then(res => a = res)
    console.log(a)
}
func() // 1
```

