# Js SetTimeout问题

## 问题：

面试常会有的问题：

1. 给一个循环，里面是 setTimrout 函数，让函数依次每隔1秒输出一次

```javascript
for(var i=0;i<5;i++) { setTimeout(() => console.log(i), 1000); }
```

这个部分的输出结果是：

>  1秒之后输出5个5

注意这里 var 声明的变量，所以 i 属于全局作用域，而 setTimeout 的第一个函数是回调函数，并且无参数，所以在执行回调的时候全局变量 i 都已经变成了5

2. 同样的问题，如果去掉延时的1000参数，即：

```javascript
for(var i=0;i<5;i++) { setTimeout(() => console.log(i)); }
```

这部分的输出结果是：

> 立刻输出5个5

这是因为，虽然延时是0，但是 setTimeout 还是启动了异步操作，所以在循环过程中启动的，加入了循环结束后，所以在循环结束之后才会执行。

## 解决方法1：let

让 var 变成 let

```javascript
for(let i=0;i<5;i++) { setTimeout(() => console.log(i)); } // 立即输出0 1 2 3 4
```

```javascript
for(let i=0;i<5;i++) { setTimeout(() => console.log(i), 1000); } //等1秒立刻输出0，1，2，3，4
```

这时因为 let 的作用域是函数块，所以每次 i 的作用域都是for 循环，并且每次 console.log(i)都引用到了 i 变量，因此 i的每个值都会被保存下来，在 setTimeout 结束前都不会被释放。



## 解决方法2： 构造作用域

```javascript
for(var i=0;i<5;i++) { 
    setTimeout(
        (function(){
            console.log(i)
        })(i) ); 
}//立即输出0，1，2，3，4
```

```javascript
for(var i=0;i<5;i++) { 
    setTimeout(
        (function(){
            console.log(i)
        })(i), 1000 ); 
}
//立刻输出0，1，2，3，4
```

```javascript
for(var i=0;i<5;i++) { 
    setTimeout(
        (function(){
            console.log(i)
        })(i), i*1000 ); 
}
//立刻输出0，1，2，3，4
```

可以看到上面的缺点是无法控制其每隔1秒输出一次，而是都立即执行，这也是立即执行函数的效果。我们用立即执行函数的参数 i,使得其立刻输出

```javascript
for (var i = 0; i < 5; i++) { 
 	(function(i){      //立刻执行函数
		setTimeout(function (){
			console.log(i);  
		 },1000);  
 	})(i);	
}
// 延时1秒输出0,1,2,3,4
```



这种构造作用域的方法不会导致立刻输出，而是给 setTimeout 构造了一个作用域，因此每个 i 都可以在作用域内被保存下来，直到回调函数执行完毕。

```javascript
for (var i = 0; i < 5; i++) { 
 	(function(){      //立刻执行函数
		setTimeout(function (){
			console.log(i);  
		 },1000);  
 	})(i);	
}
// 1秒后连续输出5个5
```

**注意这里有个坑，把自治性函数的参数去掉，那么这个构造的作用域中就找不到 i，于是只能去全局作用域找，而全局作用域的 i 已经执行完毕，成了5，所以结果为1秒后连续输出5个5**



## 终极目标

这个函数向做的，应该是每隔1秒，输出一次，但是我们的回调函数排在 for 循环之后，执行的时候是没有异步的，也就是等待1秒是(基本)同时执行的,因为是立即执行函数，在1秒结束后，回调函数形成的队列也是依次进行的，但是结束时间基本一致。所以在1秒结束后连续输出，而不是每隔1秒输出一次，想要每隔1秒输出，需要控制延时的值

```javascript
for (var i = 0; i < 5; i++) { 
 	(function(i){      //立刻执行函数
		setTimeout(function (){
			console.log(i);  
		 },i*1000);  
 	})(i);	
}
//每隔1秒输出一个，0，1，2，3，4
```

```javascript
for (let i = 0; i<5;i++){
    setTimeout(()=>console.log(i), i*1000);
}
//每隔1秒输出一个，0，1，2，3，4
```



## 一些坑

```javascript
for (var i = 0; i < 5; i++) { 
 	(function(i){      //立刻执行函数
		setTimeout(function (i){
			console.log(i);  
		 },i*1000);  
 	})(i);	
}
//  每隔1秒输出一个 undefined
```

 这里我的理解是，setTimeout 是不能携带参数的，携带参数之后，屏蔽了构造的作用域中的 i，因此会导致输出 undefined