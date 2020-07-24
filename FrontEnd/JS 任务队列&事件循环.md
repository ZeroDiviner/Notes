# JS 任务队列&事件循环

Js是单线程的，所谓单线程，就是依次只能完成一个任务，前面一个完成，才能执行后面一个。

但是这种模式会出现问题，例如一个任务长时间无相应，后面等待的任务就只能卡在这个地方。

为了解决这个问题，Js 将任务执行模式分成了两种，**同步**和**异步**

* 同步模式：就是上面描述的，一个任务结束后再执行下一个任务。

* 异步模式： 每个函数都有一个或者多个回调函数，前一个任务结束后不是执行队列上的后一个任务，而是执行回调函数，后一个任务则是不等待前一个任务的回调函数而执行。所以执行顺序和任务的排列顺序是不一致的。



## 异步任务队列

### SetTimeout

SetTimeout 会执行一个异步的函数，即使第二个参数是0.

```javascript
var timeout1 = setTimeout(function() {
  console.log(2);
}, 0);
 
console.log(1);
 
var timeout2 =setTimeout(function() {
  console.log(3);
},0)
//1
//2
//3
```

可以看到同步队列中只有输出1，所以1先输出，而后 setTimeout 注册的两个异步事件2先注册，所以先输出，最后输出3。

>  SetTimeout 有一个最小事件间隔，这个间隔足够 console.log(1)执行完毕。
>
> 在苹果机上的最小时间间隔是**10ms**，在Windows系统上的最小时间间隔大约是**15ms**。Firefox中定义的最小时间间隔是**10ms**，而HTML5规范中定义的最小时间间隔是**4ms**。



### Promise

```javascript
setTimeout(function(){
    console.log(2);
},0);
 
new Promise(function(resolve){
    console.log(3);
    resolve();
    console.log(4);
}).then(function(){
    console.log(5);
});
 
console.log(6);
 
setTimeout(function(){
    console.log(7);
},0);
 
console.log(8);

// 3
// 4
// 6
// 8
// 5
// 2
// 7
```

可以看到上面 同步的第一部分为 promise 的第一部分，因为 promise 声明即执行，所以**3，4**输出，resolve()的作用是触发其异步机制，即resolve 或者 reject，而不会影响后续的同步事件。

下来同步事件输出**6，8**,  而后剩下的是两个 setTimeout 和一个 promise，而 promise.then 的优先级大于 setTimeout, 因此先输出**5**，然后2比7先注册，因此才输出**2，7**



### SetImmediate 和process.nextTick()

SetImmediate 和 process.nextTick都是 Node 的语句

```javascript
setImmediate(function(){
    console.log(1);
},0);
setTimeout(function(){
    console.log(2);
},0);
new Promise(function(resolve){
    console.log(3);
    resolve();
    console.log(4);
}).then(function(){
    console.log(5);
});
console.log(6);
process.nextTick(function(){
    console.log(7);
});
console.log(8);
// 3
// 4
// 6
// 8
// 7
// 5
// 2
// 1
```

上面可以看出，同步部分输出为3，4，6，8。

异步部分，第一个输出的是 nextTick 的 **7**，因此其优先级最高。

第二个输出的是 Promise 的回调 **5**，因此其优先级其次。

第三个输出的是 SetTimeout 的 **2**， 优先级待定

第四个输出的是 SetImmediate 的**1**，优先级待定





## 总结 

优先级 ：

<b style = 'color:red'>process.nextTick > promise.then >setImmediate > setTimeout </b>



在 Js 引擎中，可以把任务按照性质分成两种： **宏任务**，**微任务**

* 宏任务：按优先级顺序排列）: `script`(你的全部JS代码，“同步代码”）, `setTimeout`, `setInterval`, `setImmediate`, `I/O`,`UI rendering`
* 微任务：按优先级顺序排列）:`process.nextTick`,`Promises`（这里指浏览器原生实现的 Promise）, `Object.observe`, `MutationObserver`

浏览器环境中的事件循环：

1. 第一个事件循环，先执行`script`中的所有同步代码（即 **macrotask** 中的第一项任务）
2. 取出 **microtask** 中的全部任务执行（先清空**process.nextTick**队列，再清空**promise.then**队列）
3. 下一个事件循环，再回到 **macrotask** 取其中的下一项任务
4. 再重复2
5. 反复执行事件循环…

参考：[JavaScript任务队列的顺序机制（事件循环](https://blog.csdn.net/happyqyt/article/details/90644667)



## 例题

1. 

```javascript
async function async1() {
 
  console.log('async1 start');
 
  await async2();
 
  console.log('async1 end');
 
}
 
async function async2() {
 
  console.log('async2');
 
}
 
console.log('script start');
 
setTimeout(function() {
 
    console.log('setTimeout1');
 
}, 200);
 
setTimeout(function() {
 
    console.log('setTimeout2');
 
    new Promise(function(resolve) {
 
        resolve();
 
    }).then(function() {
 
        console.log('then1')
 
    })
 
    new Promise(function(resolve) {
 
        console.log('Promise1');
 
        resolve();
 
    }).then(function() {
 
        console.log('then2')
 
    })
 
},0)
 
async1();
 
new Promise(function(resolve) {
 
    console.log('promise2');
 
    resolve();
 
  }).then(function() {
 
    console.log('then3');
 
  });
 
console.log('script end');
// 'script start'
// 'async1 start'
// 'async2'
// 'promise2'
// 'script end'
// 'async1 end'
// 'then3'
// 'setTimeout2'
// 'Promise1'
// 'then1'
// 'then2'
// 'setTimeout1'
```

上面的部分中，await 相当于是在同步队列执行，执行结束之后的部分的优先级是最高的。

2.

```javascript
function read () {
  console.log(1)
  setTimeout(function () {
    console.log(2)
    setTimeout(function () {
      console.log(3)
    })
  })
  setTimeout(function () {
    console.log(4)
  })
  console.log(5)
}
// 1
// 5
// 2
// 4
// 3
```



3. ```javascript
   console.log(1)
   setTimeout(() => {
     console.log('setTimeout1')
     Promise.resolve('p').then((res) => console.log(res))
   }, 0);
   setTimeout(() => {
     console.log('setTimeout2')
   }, 0);
   // 1
   // 'setTimeout1'
   // 'p'
   // 'setTimeout2'
   
   ```

4. ```javascript
   setImmediate(() => {
     console.log('setImmediate1')
     setTimeout(() => {
       console.log('setTimeout1')
     }, 0);
   })
   setTimeout(() => {
     console.log('setTimeout2')
     setImmediate(() => {
       console.log('setImmediate2')
     });
   }, 0)
   // 'setImmediate1'
   // 'setTimeout2'
   // 'setTimeout1'
   // 'setImmediate2'
   ```

5. ```javascript
   let fs = require('fs')
   fs.readFile('./test.js', () => {
     console.log('fs')
     setTimeout(() => {
       console.log('setTimeout')
     }, 0)
     setImmediate(() => {
       console.log('setImmediate')
     })
   })
   // fs
   // setImmediate
   // setTimeout
   ```

6. ```javascript
   setTimeout(()=>{
    
   console.log('log-timeout');
    
   }, 0);
    
   process.nextTick(()=>{
    
   console.log('tick')
    
   })
    
   const promise = new Promise((resolve)=>{
    
   console.log('log-promise')
    
   resolve('promise resolve');
    
   });
    
   (async () => {
    
   console.log('async start');
    
   const str = await promise;
    
   console.log(str);
    
   })()
    
   promise.then(()=>{
    
   console.log('log-promise1-then');
    
   });
    
   console.log('log-end');
   // 'log-promise'
   // 'async start'
   // 'log-end'
   // 'tick'
   // 'promise resolve'
   // 'log-promise1-then'
   // 'log-timeout'
   
   ```

7. ```javascript
   process.nextTick(() => {
       console.log('nextTick')
   })
   Promise.resolve().then(()=> {
       console.log('promise1');
   }).then(()=> {
     console.log('promise2');
   });
   setImmediate(() => {
       console.log('setImmediate')
   }) 
   console.log('end') 
   // end
   // 'nextTick'
   // 'promise1'
   // 'promise2'
   // 'setImmediate'
   
   ```

8. ```javascript
   async function async1(){
     console.log('async1 start')
     await async2()
     console.log('async1 end')
   }
   async function async2(){
     console.log('async2')
   }
   console.log('script start')
   setTimeout(function(){
     console.log('setTimeout') 
   },0)  
   async1();
   new Promise(function(resolve){
     console.log('promise1')
     resolve();
   }).then(function(){
     console.log('promise2')
   })
   console.log('script end')
   // 'script start'
   // 'async1 start'
   // 'async2'
   // 'promise1'
   // 'script end'
   // 'async1 end'
   // 'promise2'
   // 'setTimeout'
   ```

   