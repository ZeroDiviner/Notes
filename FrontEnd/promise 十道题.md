# Primise 十道题

1. 

```javascript
const promise = new Promise((resolve, reject) => {
  console.log(1)
  resolve()
  console.log(2)
})
promise.then(() => {
  console.log(3)
})
console.log(4)
// 1
//2
//4
//3
```

resolve 是同步执行的不会导致异步

2. 

```javascript
const promise1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('success')
  }, 1000)
})
const promise2 = promise1.then(() => {
  throw new Error('error!!!')
})

console.log('promise1', promise1)
console.log('promise2', promise2)

setTimeout(() => {
  console.log('promise1', promise1)
  console.log('promise2', promise2)
}, 2000)
// promise1 Promise(<pending>)
// promise2 Promise(<pending>)
// promise1 Promise success
// promise2 Promise {error: Error error!!!}
```

3.

```javascript
const promise = new Promise((resolve, reject) => {
  resolve('success1')
  reject('error')
  resolve('success2')
})

promise
  .then((res) => {
    console.log('then: ', res)
  })
  .catch((err) => {
    console.log('catch: ', err)
  })

// then: success1
```

resolve 和 reject 只有一次有效，多次无效

4. 

```javascript
Promise.resolve(1)
  .then((res) => {
    console.log(res)
    return 2
  })
  .catch((err) => {
    return 3
  })
  .then((res) => {
    console.log(res)
  })
// 1
// 2
```

5. 

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log('once')
    resolve('success')
  }, 1000)
})

const start = Date.now()
promise.then((res) => {
  console.log(res, Date.now() - start)
})
promise.then((res) => {
  console.log(res, Date.now() - start)
})
// once
// success 1005
// success 1007
Promise 对象的状态一经改变，那么 then 和 catch 可以调用多次，拿到的都是变化之后的值。
```

6. 

```javascript
Promise.resolve()
  .then(() => {
    return new Error('error!!!')
  })
  .then((res) => {
    console.log('then: ', res)
  })
  .catch((err) => {
    console.log('catch: ', err)
  })
then: Error: error!!!
    at Promise.resolve.then (...)
    at ...
```

return 方法无法抛出 error，只能使用 throw，或者 reject

7. 

```javascript
const promise = Promise.resolve()
  .then(() => {
    return promise
  })
promise.catch(console.error)
// TypeError: Chaining cycle detected for promise #<Promise>
//    at <anonymous>
//    at process._tickCallback (internal/process/next_tick.js:188:7)
//    at Function.Module.runMain (module.js:667:11)
//    at startup (bootstrap_node.js:187:16)
//    at bootstrap_node.js:607:3

```

then或者 catch 的返回值会自动用 promise包裹，所以返回值不能是 promise，会造成死循环。

8. 

```javascript
Promise.resolve(1)
  .then(2)
  .then(Promise.resolve(3))
  .then(console.log)
// 1
```

.then和.catch 的期望参数是函数，传入非函数会发生值穿透

9. 

```javascript
Promise.resolve()
  .then(function success (res) {
    throw new Error('error')
  }, function fail1 (e) {
    console.error('fail1: ', e)
  })
  .catch(function fail2 (e) {
    console.error('fail2: ', e)
  })
// fail2: Error : error
```

then 中和其成功回调同级别的 fail 回调不能捕捉到同级别 success抛出的错误，需要到下一轮 then 的 fail 中捕获或者尾部的 catch。

10 

```javascript
process.nextTick(() => {
  console.log('nextTick')
})
Promise.resolve()
  .then(() => {
    console.log('then')
  })
setImmediate(() => {
  console.log('setImmediate')
})
console.log('end')
// end
// nextTick
// then
// setImmediate
```

参考: [Promise 必知必会（十道题）](https://juejin.im/post/6844903509934997511)

