# Bind, Call, apply, this

[toc]

this 指向在调用的时候才能够决定，在定义的时候无法确定。

## This 指向问题

```javascript
var a = {
    user:"追梦子",
    fn:function(){
        console.log(this.user);
    }
}
var b = a.fn;
b();
>>> undefined

```

```javascript
var a = {
    user:"追梦子",
    fn:function(){
        console.log(this.user);
    }
}
a.fn();
>>> "追梦子"
```

* 上面这两个例子中，第一个是先声明了 b 变量，指向了 a 中的 fn, 但是b 是挂载在 window 下的一个变量，所以使用 b()的话，this 指向的是 window，而 window 中没有 user 这个属性，因此输出 undefined
* 在第二个例子中，直接用定义的 a 调用 fn，因此 fn 指向的是调用它的 a，a 下属性 user 为"追梦子"，因此输出追梦子



## Call，Apply，bind的定义

三个函数都可以改变 this 指针的指向，不同的是：

* Call 函数： 立即执行，并且可以传递参数，但是参数需要一个一个传
* Apply 函数： 立即执行，可以传递参数，传参形式是一个数组
* Bind 函数：不立即执行，而是构造函数直接复制并返回成一个新的函数，并且改变新的函数的this的指向。



## Call 函数 & call 函数传参

```javascript
var a = {
    user:"追梦子",
    fn:function(){
        console.log(this.user); //追梦子
    }
}
var b = a.fn;
b.call(a);
>>> "追梦子"
```

上面这个函数中，先声明了 b变量，挂载在了window 下，之后调用 call 函数，使得this 的指向从 window 指向了 a，a 下的 user 是"追梦子", 所以输出为追梦子。

```javascript
var a = {
    user:"追梦子",
    fn:function(e,ee){
        console.log(this.user); //追梦子
        console.log(e+ee); //3
    }
}
var b = a.fn;
b.call(a,1,2);
>>> "追梦子"
>>> 3
```

这个部分使用 call 绑定 this 指针到 a 上，因此第一行可以输出追梦子，同时传入了两个参数1，2，因此可以输出为3。



## Apply & apply 函数传参

```javascript
var a = {
    user:"追梦子",
    fn:function(){
        console.log(this.user); //追梦子
    }
}
var b = a.fn;
b.apply(a);
>>> 追梦子
```

apply 方法和 call 方法有点相似，也是改变 this 指针指向，在一个参数的时候是

```javascript
var a = {
    user:"追梦子",
    fn:function(e,ee){
        console.log(this.user); //追梦子
        console.log(e+ee); //11
    }
}
var b = a.fn;
b.apply(a,[10,1]);
>>> 追梦子
>>> 11
```

apply 和 call 最大的区别，即 apply 的参数传入一定要是一个数组形式的

## Bind 和 Call 第一个参数为 Null

```javascript
var a = {
    user:"追梦子",
    fn:function(){
        console.log(this); //Window {external: Object, chrome: Object, document: document, a: Object, speechSynthesis: SpeechSynthesis…}
    }
}
var b = a.fn;
b.apply(null);
>> window
```





## Bind & bind 函数传参

Bind 和其他两个方法有些不同，即不会立刻执行函数

```javascript
var a = {
    user:"追梦子",
    fn:function(){
        console.log(this.user);
    }
}
var b = a.fn;
b.bind(a);
>> 没有任何输出
```

如果给这个 bind(a)后面加括号变成 bind(a)(),那么函数就相当于是 call 函数了

即：

```javascript
var a = {
    user:"追梦子",
    fn:function(){
        console.log(this.user);
    }
}
var b = a.fn;
b.bind(a)();
>>> 追梦子
```

因为 bind 这个函数返回一个改变指针的 function，所以下面的 function 其实和上面的是相同的

```javascript
var a = {
    user:"追梦子",
    fn:function(){
        console.log(this.user); //追梦子
    }
}
var b = a.fn;
var c = b.bind(a);
c();
>>> 追梦子
```

bind 同样可以传递多个参数，并且可以在绑定的时候传递一部分，在执行的时候再传递剩下的。

```javascript
var a = {
    user:"追梦子",
    fn:function(e,d,f){
        console.log(this.user); //追梦子
        console.log(e,d,f); //10 1 2
    }
}
var b = a.fn;
var c = b.bind(a,10);
c(1,2);
>>> 追梦子
>>> 10 1 2
```

可以看到上面的函数 bind 的时候除了要绑定的 a，只传递了一个10，执行的时候才传递1，2两个参数, 注意10这个参数在执行的过程中是不能覆盖的，如果执行的时候还传递三个参数，那么就会出现：

```javascript
var a = {
    user:"追梦子",
    fn:function(e,d,f){
        console.log(this.user); //追梦子
        console.log(e,d,f); //10 1 2
    }
}
var b = a.fn;
var c = b.bind(a,10);
c(12,1,2);
>>> 追梦子
>>> 10 12 1
```

可以看到12，1变成参数，即函数只接受两个参数，对于多余参数不会理睬。

