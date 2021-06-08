---
title: Promise实现原理
date: 2021-06-08 14:30:16
tags:
---

Promise 是异步编程的一种解决方案。使用 Promise对象，可以将异步操作以同步操作的流程表达出来，避免了层层嵌套回调函数。
使用Promise可以使得异步方法可以像同步方法那样返回值：异步方法并不会立即返回最终的值，而是会返回一个 promise，以便在未来某个时候把值交给使用者。  


### Promise的基本用法
```
const promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});

promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```
Promise有三个状态:
- 待定（pending）: 初始状态，既没有被兑现，也没有被拒绝。
- 已兑现（fulfilled）: 意味着操作成功完成。
- 已拒绝（rejected）: 意味着操作失败。

Promise一定是处于这三个状态的其中之一。待定状态的 Promise 对象要么会通过一个值被兑现（fulfilled），要么会通过一个原因（错误）被拒绝（rejected）。当这些情况之一发生时，我们用 promise 的 then 方法排列起来的相关处理程序就会被调用。

![image](https://mdn.mozillademos.org/files/8633/promises.png)


###  发布/订阅模式   
Promise 是通过发布订阅模式来实现的，发布/订阅模式 是通过信号的发送时机来决定什么时候执行其对相应的任务实现 ：

```
class EventCenter {
  // 定义事件中心
  constructor() {
      this.events = {}
  }
  // 发布器
  on(evt, handler) {
      // 检测事件信号是否存在，当存在时不做操作，不存在时创建给予这个信号的方法存储器（数组）
      this.events[evt] = this.events[evt] || []
          // 将传入的方法放入数组中
      this.events[evt].push({
          handler: handler
      })
  }
  // 订阅器
  fire(evt, params) {
      // 检测当前被订阅的信号是否存在，存在则执行其内的所有方法
      if (!this.events[evt]) {
          return
      }
      for (let i = 0; i < this.events[evt].length; i++) {
          this.events[evt][i].handler(params)
      }
  }
}
let center = new EventCenter()
center.on('event', function(data) {
  console.log('event执行了第一个任务')
})
center.on('event', function(data) {
  console.log('event执行了第二个任务')
})
center.fire('event')
// event执行了第一个任务
// event执行了第二个任务
```


### Promise的实现原理
首先 new Promise 创建实例，在 then() 中收集回调， 异步操作完成后调用 resolve/reject 再执行回调。  
如果传入的 executor 是同步任务，在执行 then 的时候，已经先执行 resolve/reject，就直接执行传入的回调函数

**Promise的基本实现：**
```
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';
class MyPromise {
    constructor(executor) {
        this.status = PENDING
        this.result = undefined
        this.resolveQueues = []
        this.rejectQueues = []
        executor(this.resolve,this.reject)
    }
    resolve = (value) => {
        if(this.status !== PENDING) return
        this.result = value
        this.status = FULFILLED
        this.resolveQueues.length && this.resolveQueues.shift()(this.result)
    }
    reject = (value) => {
        if(this.status !== PENDING) return
        this.result = value
        this.status = REJECTED
        this.rejectQueues.length && this.rejectQueues.shift()(this.result)
    }
    then(onResolved, onRejected) {
        switch(this.status) {
            case PENDING: {
                this.resolveQueues.push(onResolved)
                this.rejectQueues.push(onRejected)
            }
            case FULFILLED:
              onResolved(this.result)
              break
            case REJECTED:
              onRejected(this.result)
              break
        }
    }
}
```
测试
```
const promise = new MyPromise((resolve,reject) => {
    setTimeout(() => {
        resolve('test')
    },1000)
     
})
promise.then(res => {
    console.log(res)
})
```
过1秒输出了 test

参考：   
https://github.com/YvetteLau/Blog/issues/2  
https://zhuanlan.zhihu.com/p/28105119  
https://segmentfault.com/a/1190000022614627
