---
title: 浅谈Promise
date: 2020-05-25 20:04:51
tags: JavaScript
categories: 笔记
---

# 浅谈Promise

#### promise诞生的原因

​	众所周知，javascript是一门单线程的语言，一次只对一个任务好，虽然单纯，但是当存在很多任务的时候，它只能一个一个来，处理完当下的任务才能进行下一个，当遇到比较耗时难缠的任务（像网络请求和定时器）时，后面的任务只能慢慢等着，效率会很低，会导致浏览器 '假死'，所以javascript出现了同步和异步两种任务执行方式，而promise正是异步的解决方案之一。

#### promise简述

​	promise是一个对象，用来表示异步操作的最终完成（或者失败）以及其值。它可以让你可以让异步方法也可以像同步方法那样返回值，但是并不是直接返回执行的最终结果，而是返回一个能描述未来将会发生什么的promise对象。

```javascript
let p = new Promise((resolve, reject) => {
  resolve(1)
})
console.log(p)
/* 
 Promise
 [[PromiseStatus]]: "resolved"
 [[PromiseValue]]: 1
*/
```

从上面的可以粗略的看出promise包含status和value，一个Promise有一下几个状态

- pending 初始状态，表示正在进行中
- fulfilled 成功状态，表示操作执行成功
- rejected 失败状态，表示操作执行失败

pending状态可能会转变为fulfilled状态并传递一个值给相应的状态处理方法，也有可能转变为rejected状态并传递错误信息，当其中任一种情况出现时，Promise 对象的 `then` 方法绑定的处理方法（handlers ）就会被调用（then方法包含两个参数：onfulfilled 和 onrejected，它们都是 Function 类型。当Promise状态为*fulfilled*时，调用 then 的 onfulfilled 方法，当Promise状态为*rejected*时，调用 then 的 onrejected 方法， 所以在异步操作的完成和绑定处理方法之间不存在竞争）。

因为then和catch返回的都是promise对象，所以他们可以链式调用

![avatar](https://mdn.mozillademos.org/files/8633/promises.png)

#### 模拟promise

- ##### 雏形

  首先promise有三种状态：pending，fulfilled，rejected，拥有resolve（）和reject（）等基本方法，原型上还有then（）等方法。

  ```javascript
  class Promise {
      constructor(executor) {
          let _this = this
          this.status = 'pending'
          this.value = undefined
          this.reason = undefined
          try {
              executor(resolve, reject)
          } catch (err) {
              reject(err)
          } 
          // 如果不在pending状态，就不能对其进行修改。
          function resolve(value) {
              if(_this.status === 'pending') {
                  _this.status = 'fulfilled'
                  _this.value = value
              }
          }
          function reject(reason) {
              if(_this.status === 'pending') {
                  _this.status = 'rejected'
                  _this.reason = reason
              }
          }
      }
  }
  ```

  其中status代表着状态，value代表返回成功的值，reason代表返回失败的信息。

- ##### then

  promise中一个非常重要的原型方法，就是then，先简单接收一下promise传来的值

  ```javascript
  then(successCallback, failCallback) {
  	successCallback = typeof successCallback === 'function' ? 
  					  successCallback : 
  					  res => res
  	failCallback = typeof failCallback === 'function' ? 
  				   failCallback : 
  				   err => err
  	if(this.status === 'fulfilled') {
  		successCallback(this.value)
  	}
  	if(this.status === 'rejected') {
  		failCallback(this.reason)
  	}
  }
  ```

  让我们来简单测试一下吧

  ```javascript
  let p = new Promise((resolve, reject) => {
      resolve(1)
  })
  p.then(res => console.log(res)) // 1
  ```

  这样我们最初步的promise就完成了？让我们再来试试异步

  ```
  let p = new Promise((resolve, reject) => {
      setTimeout(() => {
      	resolve(1)
      }, 1000)
  })
  p.then(res => console.log(res))
  ```

  出错了，根据异步的特性知道，在resolve(1)执行之前p.then()已经执行了，当时的status还处于pending状态，所以我们对其进行处理就行

  ```javascript
  class Promise {
      constructor(executor) {
          let _this = this
          this.status = 'pending'
          this.value = undefined
          this.reason = undefined
          this.successCallbacks = []
          this.failCallbacks = []
          try {
              executor(resolve, reject)
          } catch (err) {
              reject(err)
          } 
          function resolve(value) {
              if(_this.status === 'pending') {
                  _this.status = 'fulfilled'
                  _this.value = value
                  _this.successCallbacks.forEach(callback => callback(value))
              }
          }
          function reject(reason) {
              if(_this.status === 'pending') {
                  _this.status = 'rejected'
                  _this.reason = reason
                  _this.failCallbacks.forEach(callback => callback(reason))
              }
          }
      }
      then(successCallback, failCallback) {
          successCallback = typeof successCallback === 'function' ? 
  					  successCallback : 
  					  res => res
  		failCallback = typeof failCallback === 'function' ? 
  				   	   failCallback : 
  				   	   err => err
          if(this.status === 'pending'){
              this.successCallbacks.push(successCallback)
              this.failCallbacks.push(failCallback)
          }
          if(this.status === 'fulfilled') {
              successCallback(this.value)
          }
          if(this.status === 'rejected') {
              failCallback(this.reason)
          }
      }
  }
  let p = new Promise((resolve, reject) => {
      setTimeout(() => {
          resolve(1)
      }, 0)
  })
  p.then(res => console.log(res), rej => console.log(rej)) // 1
  ```

  这样一个初步的promise就完成了

- ##### 完善

  - then方法必须返回一个promise对象，所以我们要对返回的值进行包装
  - 如果then中没有return语句，就用undefined包装成promise返回
  - 如果then方法出现异常，则调用reject跳转到下一个then的failCallback
  - 如果then中返回了一个promise，就返回这个promise
  - 如果then没传入任何方法，就继续向下传递

  ```javascript
class Promise {
    constructor(executor) {
        let _this = this
        this.status = 'pending'
        this.value = undefined
        this.reason = undefined
        this.successCallbacks = []
        this.failCallbacks = []
        try {
            executor(resolve, reject)
        } catch (err) {
            reject(err)
        } 
        function resolve(value) {
            if(_this.status === 'pending') {
                _this.status = 'fulfilled'
                _this.value = value
                _this.successCallbacks.forEach(callback => callback(value))
            }
        }
        function reject(reason) {
            if(_this.status === 'pending') {
                _this.status = 'rejected'
                _this.reason = reason
                _this.failCallbacks.forEach(callback => callback(reason))
            }
        }
    }
    then(successCallback, failCallback) {
        successCallback = typeof successCallback === 'function' ? 
					  successCallback : 
					  res => res
		failCallback = typeof failCallback === 'function' ? 
				   	   failCallback : 
				   	   err => err
        let promise = new Promise((resolve, reject) => {
            if(this.status === 'pending'){
                this.successCallbacks.push(() => resolve(successCallback(this.value)))
                this.failCallbacks.push(() => reject(failCallback(this.reason)))
            }
            if(this.status === 'fulfilled') {
                let t = successCallback(this.value)
                resolve(t)
            }
            if(this.status === 'rejected') {
                let t = failCallback(this.reason)
                reject(t)
            }
        })
        return promise
    }
}
let p = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve(1)
    }, 0)
})
console.log(p.then(res => res, res => res).then(res => console.log(res))) // 1
  ```
  
   这样就实现了then的一部分功能，但是根据then的返回值生成新的promise对象逻辑还比较多，所以我们可以把它从中抽离出来
  
  ```javascript
  function thenPromise(promise, t, resolve, reject) {
      // 判断返回的新的promise与 then方法返回值是否一致，如果一致，promise就会一直处在pending状态
      if(promise === t) {
          reject(new TypeError('error:循环引用'))
      }
      // 判断 t 是否为普通值,如果是普通值，直接包装成promise返回,如果不是，再进行下一步判断
      if (t !==null && (typeof t === 'function' || typeof t === 'object')) {
          // 判断 t 是否包含then方法
          try {
              let then = t.then
              // 如果then是一个function,就正常执行,否则作为普通的返回值返回
              if (typeof then === 'function') {
                  then.call(t, v => resolve(v))
              } else {
                  resolve(x)
              }
          } catch (err) {
              reject(err)
          }
  
      } else {
          resolve(t)
      }
  }
  ```
  
  这样，一个简单的promise就完成了。