#### 回顾 promise

传统的异步编程解决方案有：回调函数、事件。

promise 对象的特点：

- 对象的状态不受外界影响。有三种状态：`pending（进行中）`、`fulfilled（已成功）`和`rejected（已失败）`。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。
- 一旦状态改变，就不会再变，任何时候都可以得到这个结果。只有两种可能的状态改变：从`pending`变为`fulfilled`和从`pending`变为`rejected`。只要这两种情况发生，状态就凝固了，不会再变，这时称为 `resolved（已定型）`。改变若已经发生，再对 promise 对象添加回调函数，也会立即得到这个结果。
- 可以将异步操作以同步操作的流程表达出来，避免层层嵌套的回调函数，即回调地狱。

promise 对象的缺点：

- 无法取消 promise，一旦新建它就会立即执行，无法中途取消。
- promise 内部抛出的错误，不会反应到外部，需要你在外部利用`promise.prototype.catch()`来获取
- 在处于`pending`状态时，无法得知目前进展到哪个阶段（刚刚开始还是即将完成）

`promise.prototype.then`方法返回的是一个新的 promise 实例。

以上总结来自：[ECMAScript 6 入门 | 阮一峰](http://es6.ruanyifeng.com/#docs/promise)（基础知识看它）

#### 手写简易版 promise

```javascript
// 三个状态
const PENDING = 'pending'
const RESOLVED = 'resolved'
const REJECTED = 'rejected'

// promise对象构造函数，参数是一个函数 fn，fn将会在构造函数里面执行
function myPromise(fn) {
    // 保存this对象
    const that = this
    // 一开始 promise 的状态应为 pending
    that.state = PENDING
    // 保存传入 resolve 或者 reject 的值，也即结果
    that.value = null
    // 保存 then 的回调，用于状态改变时使用，因为执行完 promise 时状态可能还是等待中
    that.resolvedCallbacks = []
	that.rejectedCallbacks = []
    
    // resolve 函数
    function resolve(value) {
        // 判断状态是否为等待中，上面说过只有等待态才可以改变状态
        if (that.state === PENDING) {
            // 将当前状态更改为对应状态
            that.state = RESOLVED
            // 将传入的结果赋值给 value
            that.value = value
            // 遍历回调数组并执行
            that.resolvedCallbacks.map(cb => cb(that.value))
        }
    }
    // reject 函数，相类似
    function reject(value) {
        if (that.state === PENDING) {
            that.state = REJECTED
            that.value = value
            that.rejectedCallbacks.map(cb => cb(that.value))
        }
    }
}

// 这里是执行传入来的函数，并且将之前两个函数当做参数传进去
// 如果传进来的用户自定义的函数执行报错，需要捕获错误并且执行reject
try {
    fn(resolve, reject)
} catch(e) {
    reject(e)
}

// 实现 then 函数
myPromise.prototype.then = function(onFulfilled, onRejected) {
    const that = this
    // 判断两个参数是否为函数类型，因为这两个参数是可选参数
    // 若参数不是函数类型时，就创建一个函数赋值给对应的参数，同时也实现了透传
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : V => v
    onRejected = typeof onRejected === 'function' ? onRejected : r => { throw r }
    if (that.state ==== PENDING) {
        // 还在等待态的时候，先把函数推入到回调函数数组
        that.resolvedCallbacks.push(onFulfilled)
        that.rejectedCallbacks.push(onRejected)
    }
    if (that.state === RESOLVED) {
        onFulfilled(that.value)
    }
    if (that.state === REJECTED) {
        onRejected(that.value)
    }
}
```

#### 参考

yck 的小册

[BAT前端经典面试问题：史上最最最详细的手写Promise教程](https://juejin.im/post/5b2f02cd5188252b937548ab)（根据promisesA+规范来实现的，也很不错）