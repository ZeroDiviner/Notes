# Webworker 实现 js 多线程

JavaScript 一直是属于单线程环境，我们无法同时运行两个 JavaScript 脚本。但是如果可以使用多线程的话那么业务场景将会大变，并且可以合理的利用 cpu 的多核机制。

WebWorker 并不是传统意义上的多线程，线程可以共享用户地址空间，线程之间不会共享任何资源，并且**线程之间唯一的通信方式就是一个基于事件监听机制的 message**



## 实例化一个 web worker

实例化很简单，只需要 new 一个 worker全局对象就可以，这个构造函数接受一个 filepathname String 参数，用于指定 Worker 脚本文件的路径；

```js
var worker = new Worker('./worker.js');
```

当实例运行了一个 worker 线程之后，两个线程实际上是完全独立运行的。他们之间的通信是通过事件监听的 message 来实现的，在 main.js 中，注册这个 worker 并监听它

```javascript
//main.js
var worker = new Worker('./worker.js');
worker.addEventListener('message', function (e) { // 监听 message 事件
  console.log('MAIN: ', 'RECEIVE', e.data);
});

//发消息给 worker.js
worker.postMessage('Hello Worker, I am main.js'); // 注意这里调用的是 worker 上挂载的函数，而在 worker.js 中 postmessage 相当于是一个全局函数
```

 而在 worker.js 中

```javascript
//worker.js
console.log('WORKER TASK: ', 'running');
onmessage = function (e) { 
  console.log('WORKER TASK: ', 'RECEIVE', e.data);
  // 发送数据事件
  postMessage('Hello, I am Worker');
}
// onmessage 是一个全局函数
```

如果某个时刻，不想让这个 worker 继续工作了，就可以调用

```javascript
worker.terminate();
```



## 错误处理

我们还需要对于 worker 可能出现的错误进行监听和处理

```javascript
//main.js
var worker = new Worker('./worker.js');

//监听 message
worker.addEventListener('message', function (e) {
  console.log('MAIN: ', 'RECEIVE', e.data);
});

// 监听 error
worker.addEventListener('error', function (e) {
  console.log('MAIN: ', 'ERROR', e);
  console.log('MAIN: ', 'ERROR', 'filename:' + e.filename + '---message:' + e.message + '---lineno:' + e.lineno);
});

worker.postMessage({ // 调用 worker的 postmessage 函数
  m: 'Hello Worker, I am main.js'
});

```



## Webworker 的环境和作用域

Worker 线程中没有 window 对象，也无法访问 dom 对象，所以一般只能执行 js的计算操作。

可以使用的 API 有:

* `setTimeout()， clearTimeout()， setInterval()， clearInterval()`：有了设计个函数，就可以在 Worker 线程中执行定时操作了；
* `XMLHttpRequest` 对象：意味着我们可以在 Worker 线程中执行 **ajax** 请求；
* `navigator` 对象：可以获取到 ppName，appVersion，platform，userAgent 等信息；
* `location` 对象（只读）：可以获取到有关当前 URL 的信息；



## 在 worker 中加载脚本

可以在worker 环境中通过 worker 的全局函数importScripts()加载外部脚本到当前 Worker 脚本中，并且他们都会运行, 

在 main.js 中创建一个 worker

```javascript
// main.js
var worker = new Worker('./worker1.js');
```

 在 worker1.js 中

```javascript
// worker1.js
console.log('hello, I,m worker 1');
importScripts('worker2.js', 'worker3.js'); // importScripts 接收多个字符串参数，表示脚本地址
// 或者
// importScripts('worker2.js');
// importScripts('worker3.js');
```

worker2.js  & worker3.js

```javascript
// worker2.js
console.log('hello, I,m worker 2');
//worker3.js
console.log('hello, I,m worker 3');
```

在 console 中可以看到他们都输出了



## Shared Worker

对于 Webworker 来说，一个 tab 页面只能对应一个web worker 线程，是相互独立的。 即 tab 页和 web worker 一一对应。

SharedWorker提供了能让不同 tab 共享一个Worker脚本线程。

```javascript
//main.js
var myWorker = new SharedWorker("worker.js");


myWorker.port.start();


myWorker.port.postMessage("hello, I'm main");


myWorker.port.onmessage = function(e) {
  console.log('Message received from worker', e);
}
```



```javascript
// worker.js
onconnect = function(e) {
  var port = e.ports[0];


  port.addEventListener('message', function(e) {
    var workerResult = 'Result: ' + (e.data[0] * e.data[1]);
    port.postMessage(workerResult);
  });


  port.start();
}
```



## 兼容性

不管是 webWorker 还是 Service Worker，大部分浏览器都还没有实现对其的实现

参考

1. [JavaScript 中的多线程 -- Web Worker](https://zhuanlan.zhihu.com/p/25184390)

