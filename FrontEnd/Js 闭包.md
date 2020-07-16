# Js 闭包

##### 定义：`闭包是指有权访问另一个函数作用域中的变量的函数`

例如下面的函数：

```javascript
function f1(){
        var n = 123;
        function f2(){    //f2是一个闭包
            alert(n)
        }    
        return f2;
}
var myFun = outFun();
myFun()
```

上面的函数中，f2可以读取 f1中的变量，因为 f2是 f1的子函数，于是把 f2返回，在外界用一个全局变量接着，那么全局变量就可以访问f1函数中的变量啦。

**因为f1是 f2的父函数，而 f2被赋值给了一个全局变量，因此f2始终存在于内存中，而f2的存在是依赖于 f1的，所以 f1也始终在内存中，不会因为调用结束而被回收。**



##  闭包场景1: 封装不同函数

```javascript
function add(x){
    return function(y){
        return x + y;
    };
}

var add_4 = add(4)
var add_6 = add(6)

console.log(add_4(5)) //9
console.log(add_4(5)) // 11
```

可以看到一个 function，根据参数不同可以返回两个效果不一样的 function。



## 闭包场景2：SetTimeout

因为 SetTimeout 的第一个参数应该是一个函数，但是这个函数是不允许传参数的，所以可以利用闭包来防止这个问题。

```javascript
function al(n){
  return function(){
      alert(n)
  }
}
var func = al(5)
SetTimeout(al, 1000)
```



## 闭包场景3：回调函数

```javascript
function changeSize(size){
        return function(){
            document.body.style.fontSize = size + 'px';
        };
}

var size12 = changeSize(12);
var size14 = changeSize(14);
var size16 = changeSize(16);

document.getElementById('size-12').onclick = size12;
document.getElementById('size-14').onclick = size14;
document.getElementById('size-16').onclick = size16;
```

可以看到上面根据不同的 size，返回了不同的回调函数，实质上还是和之前的用法一样的。



## 闭包场景4： 封装变量

```javascript
var counter = (function(){
        var privateCounter = 0; //私有变量
        function change(val){
            privateCounter += val;
        }
        return {
            increment:function(){   //三个闭包共享一个词法环境
                change(1);
            },
            decrement:function(){
                change(-1);
            },
            value:function(){
                return privateCounter;
            }
        };
})();

console.log(counter.value());//0
counter.increment();//1
counter.increment();//2
```

上面的 counter = 一个自执行函数，函数的返回值是一个对象变量，对象的每个属性都对应一个 function，而 counter 中有自己私有的 privateCounter 变量，和change函数，外界是访问不到的



## 闭包缺点

闭包会使得函数中的变量一直存储在变量中，导致内存损耗。

因为闭包中的子函数引用了其父函数的变量，而子函数又被外部变量引用。

在 Js 中，如果一个变量不再被引用，那么GC(垃圾回收机制)会回收它，如果两个变量互相引用，那么 GC 会回收它，但是对于上面这种场景，父函数是不会被回收的。



## 总结一下：

函数闭包就是别的函数能够拿到另一个函数的私有变量，原理是通过函数的作用域链，能够访问其父级和以上的作用域。其作用大致分为两种，

一种是通过函数工厂

```javascript
function factory(parameter){
  	return function(){
        // do something with this parameter
    }
}
```

根据传给factory 函数参数的不同，让返回的 function 能够做不同的事情。



另一种是通过上面场景4的方法，封装一个变量，让其拥有私有属性。 本质上还是通过返回函数来实现的。





参考：[JS闭包的理解及常见应用场景](https://www.cnblogs.com/Renyi-Fan/p/11590231.html)