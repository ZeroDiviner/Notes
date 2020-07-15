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



