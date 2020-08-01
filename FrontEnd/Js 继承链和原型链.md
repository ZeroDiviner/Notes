# Js 继承链和原型链

Js 中，每个实例对象(object)都有一个私有属性\_\_proto\_\_,指向它的构造函数的原型对象，该原型对象也有一个 \_\_proto

\_\_,层层向上，直到等于**null** 为止, 因此**原型链的顶端其实是 null**.



几乎所有 JavaScript 中的对象都是位于原型链顶端的 [`Object`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object) 的实例。



## 属性继承

Js 的对象就是一个动态的包，它又一个指向自己原型对象的链，当搜索一个属性的时候，例如

```javascript
a = function(){
  console.log(1);
}
a.attr
```

不仅仅在 a 上搜索，同时还在 a 的原型上搜索，还在 a 的原型的原型上搜索，直到 null 或者找到这个属性。



## 基于原型链的继承

```javascript
let  f = function(){
  this.a = 1;
  this.b = 2;
}
let o = new f()


f.prototype.b = 3
f.prototype.c = 4
console.log(o);
>>> f {a: 1, b: 2} 
		a: 1
		b: 2
		__proto__: Object
    	b: 3
			c: 4
			constructor: ƒ ()
			__proto__: Object
console.log(o.b) // 2
console.log(o.c) // 4
```

可以看到给o 这个对象本身有a，b 两个属性，因此给其原型赋值 b并不会修改其 b 的值，同时给 c 赋值，因为 o 上本身没有 c 这个属性，因此会沿着原型链一直向上找，找到其 prototype 的属性 c等于4，于是返回4



## 继承方法

在 js 中，任何函数都可以当做是对象的属性而添加到对象上，当继承了这个函数的时候，函数内部的 this 指针指向的是当前继承这个函数的对象。

```javascript
var o = {
  a: 2,
  m: function(){
    return this.a + 1;
  }
};
// 这里就是声明了一个对象 o，有一个属性 a = 2，同时一个方法 m
console.log(o.m()) // 3 o 这里调用 m，返回2+1 = 3
var p = Object.create(o) // 这个方法就是创建一个继承自 o 的对象 p
console.log(p)
>>> {}
			__proto__:
      	a: 2
				m: ƒ ()
				__proto__: Object
// 可以看到这时 p 自己的属性是空的，继承自 o 的属性有 a 和方法 m

p.a = 4 // 自己的属性，会覆盖掉原型链上的属性 a
console.log(p.m()) // 5 因为调用 m 的对象为 p，因此 this 指针指向调用的对象，所以结果为4+1 = 5

```



## 在 Js 中使用原型链

```javascript
function doSomething(){}
doSomething.prototype.foo = "bar"; // add a property onto the prototype
var doSomeInstancing = new doSomething();
doSomeInstancing.prop = "some value"; // add a property onto the object
console.log( doSomeInstancing );
```

结果为：

```javascript
{
    prop: "some value",
    __proto__: {
        foo: "bar",
        constructor: ƒ doSomething(),
        __proto__: {
            constructor: ƒ Object(),
            hasOwnProperty: ƒ hasOwnProperty(),
            isPrototypeOf: ƒ isPrototypeOf(),
            propertyIsEnumerable: ƒ propertyIsEnumerable(),
            toLocaleString: ƒ toLocaleString(),
            toString: ƒ toString(),
            valueOf: ƒ valueOf()
        }
    }
}
```

可以看到doSomeInstancing 是doSomething的一个实例，第四行给这个实例增加了一个属性 prop,值为'some value'所以在下面的输出中可以看到他自己的属性有 prop。

在第二行中， 给doSomething 的 prototype上挂了一个新属性 foo = 'bar'， 因此在输出的时候，可以看到其 prototpe 上挂载的 foo属性。

new 方法就是基于上面修改的prototype，来产生一个 doSomething 的实例。因此doSomeInstancing实际上是 doSomething 的实现，而 doSomething 这个实例本身没有挂载任何属性。

因此：

```javascript
console.log("doSomeInstancing.prop:      " + doSomeInstancing.prop);
console.log("doSomeInstancing.foo:       " + doSomeInstancing.foo);
console.log("doSomething.prop:           " + doSomething.prop);
console.log("doSomething.foo:            " + doSomething.foo);
console.log("doSomething.prototype.prop: " + doSomething.prototype.prop);
console.log("doSomething.prototype.foo:  " + doSomething.prototype.foo);
```

输出为

```javascript
doSomeInstancing.prop:      some value		// 实例自己的属性
doSomeInstancing.foo:       bar						// prototype 的属性
doSomething.prop:           undefined			// doSomething 本身自己没挂载属性
doSomething.foo:            undefined			// doSomething 本身自己没挂载属性
doSomething.prototype.prop: undefined			// 在 prototype 上无法访问其实例对象自己挂载的属性
doSomething.prototype.foo:  bar						// prototype 自己的属性是可以访问的
```



## 使用不同方法创建的对象和生成原型链

对象

```javascript
var o = {a: 1};
// o 这个对象继承了Object.prototype 的所有属性
// o 自身是没有hasOwnProperty的属性的
// hasOwnProperty 是 Object.prototype 的属性
// 因此 o 继承了 Object.prototype 的 hasOwnProperty
// Object.prototype的原型为 null
原型链为： o ---> Object.prototype ---> null
```

数组

```javascript
var a = ["yo", "whadup", "?"];
// 数组继承于Array.prototype 
// (Array.prototype 中包含 indexOf, forEach 等方法)
原型链为：a ---> Array.prototype ---> Object.prototype ---> null
```

函数

```javascript
function f(){
  return 2;
}
// 函数都继承于 Function.prototype
// (Function.prototype 中包含 call, bind等方法)
原型链为：f ---> Function.prototype ---> Object.prototype ---> null
```

Create创建的

```javascript
var a = {a: 1}; 
原型链：a ---> Object.prototype ---> null

var b = Object.create(a);
原型链：b ---> a ---> Object.prototype ---> null

var c = Object.create(b);
原型链: c ---> b ---> a ---> Object.prototype ---> null

```



## 性能问题

在原型链上查找某个属性是比较复杂的过程，试图访问不存在的属性将会遍历整个原型链。 遍历对象的属性的时候，原型链上每个可枚举属性都会被枚举出来。



因此要检查对象是否具备自己定义的属性，可以用从 Object.prototype 上继承的 hasOwnProperty方法(这时唯一一个不会遍历原型链的方法)。

```javascript
console.log(g.hasOwnProperty('nope'));
// false
console.log(g.hasOwnProperty('vertices'));
// true
```



`.prototype` 是用于类的，而 `Object.getPrototypeOf()` 是用于实例的（instances），两者功能一致。



## New 问题

所谓 new 函数就是新建一个给定类的实例

```javascript
function Person (name) {
    this.name = name;
}
p = new Person('bob')
console.log(p.__proto__ == Person.prototype) //true
```

自己实现一个 new函数

```javascript
function new_(name){
  		c = {}
  		c.__proto__ = Person.prototype
  		Person.call(c,name);
}
p = new_('bob')
console.log(p)
>>> Person{name: 'bob'}
```

实现一个更 general 一点的

```javascript
 function new_(constructor){
   		c = {};
   		c.__proto__ = constructor.prototype;
			constructor.apply(c, Array.prototype.slice.call(arguments, 1));
   		return c;
 }
p = new_(Person, 'bob')
console.log(p)
>>> Person{name: 'bob'}
```







## 面试题

```javascript
let a = {}
let b = new Object()
let c = function(){}
function d(){
  
}
let e = new d()
let f = new c()
console.log(typeof(c)) //function
console.log(typeof(d)) //function


```

对象分为普通对象和函数对象，上面的 c, d 都是函数对象，对于函数对象 typeof 返回的就是 function。

函数对象是有 prototype 属性的，普通对象只有\_\_proto\_\_属性, 所有对象都有\_\_proto\_\_属性.

普通对象的\_\_proto\_\_属性指向其构造函数的 prototype, 例如

```javascript
console.log(a.__proto__ == Object.prototype) //true ========>      a={}
console.log(b.__proto__ == Object.prototype) //true ========>      b = new Object()
console.log(c.__proto__ == Function.prototype) // true ========>   c = function(){}
console.log(e.__proto__ == d.prototype) // true ========>          e = new d()
```



因此对应构造函数也就是

```javascript
console.log(a.constructor == Object) //true
console.log(b.constructor == Object) // true
console.log(c.constructor == Function)  // true
console.log(e.constructor == d) // true
```



