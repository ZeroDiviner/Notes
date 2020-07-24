# Es6 Proxy & Reflect

# Proxy

Proxy 翻译为代理，可以理解为对于目标的一些事件，架设一些拦截，任何对于这个事件的触发和访问都会触发这个拦截。下面是一个简单的 proxy:

```javascript
var obj = new Proxy({}, {
  get: function (target, key, receiver) {
    console.log(`getting ${key}!`);
    return Reflect.get(target, key, receiver);
  },
  set: function (target, key, value, receiver) {
    console.log(`setting ${key}!`);
    return Reflect.set(target, key, value, receiver);
  }
});

obj.k = 1 // setting k!
```

上面这个 function 相当于是对于一个空的 object 的 get 和 set 方法进行了拦截，并在想要得到和设置值的时候触发



通常写法：

```javascript
//通常定义一个 Proxy 会需要两个参数，一个是 target，一个是handler，两个都是 object，如：
var target = {}
var handler = {}
var proxy = new Proxy(target, handler);
//上面这个情况就是对于一个空的 object，不进行任何设置。
```

一般 target 可以选择自定的objecct，handler 同样是 object，支持的操作有：

* **get(target, propKey, receiver)**：拦截对象属性的读取，比如`proxy.foo`和`proxy['foo']`。最后一个参数`receiver`是一个对象，可选，参见下面`Reflect.get`的部分。

  * ```javascript
    // get 方法可以继承，如下图使用 create 方法来创建一个实例
    let proto = new Proxy({}, {
      get(target, propertyKey, receiver) {
        console.log('GET '+propertyKey);
        return target[propertyKey];
      }
    });
    
    let obj = Object.create(proto);
    obj.xxx = 1
    console.log(obj.xxx) // GET xxx
    ```

* **set(target, propKey, value, receiver)**: 拦截对象属性的设置，比如`proxy.foo = v`或`proxy['foo'] = v`，返回一个布尔值。

  * ```javascript
    let validator = {
      set: function(obj, prop, value) {
        if (prop === 'age') {
          if (!Number.isInteger(value)) {
            throw new TypeError('The age is not an integer');
          }
          if (value > 200) {
            throw new RangeError('The age seems invalid');
          }
        }
    
        // 对于age以外的属性，直接保存
        obj[prop] = value;
      }
    };
    
    let person = new Proxy({}, validator);
    person.age = 100;
    
    person.age // 100
    person.age = 'young' // 报错
    person.age = 300 // 报错
    ```

    

* **has(target, propKey)**: 拦截`propKey in proxy`的操作，以及对象的`hasOwnProperty`方法，返回一个布尔值。

* **deleteProperty(target, propKey)**: 拦截`delete proxy[propKey]`的操作，返回一个布尔值。

* **ownKeys(target)**: 拦截`Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`，返回一个数组。该方法返回对象所有自身的属性，而`Object.keys()`仅返回对象可遍历的属性。

* **getOwnPropertyDescriptor(target, propKey)**: 拦截`Object.getOwnPropertyDescriptor(proxy, propKey)`，返回属性的描述对象。

* **defineProperty(target, propKey, propDesc)**: 拦截`Object.defineProperty(proxy, propKey, propDesc）`、`Object.defineProperties(proxy, propDescs)`，返回一个布尔值。

* **preventExtensions(target)**: 拦截`Object.preventExtensions(proxy)`，返回一个布尔值。

* **getPrototypeOf(target)**: 拦截`Object.getPrototypeOf(proxy)`，返回一个对象。

* **isExtensible(target)**: 拦截`Object.isExtensible(proxy)`，返回一个布尔值。

* **setPrototypeOf(target, proto)**: 拦截`Object.setPrototypeOf(proxy, proto)`，返回一个布尔值。

  如果目标对象是函数，那么还有两种额外操作可以拦截。

* **apply(target, object, args)**: 拦截 Proxy 实例作为函数调用的操作，比如`proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`。

* **construct(target, args)**: 拦截 Proxy 实例作为构造函数调用的操作，比如`new proxy(...args)`。



## Reflect 概述

Reflect 的所有方法都是上面 proxy 的方法，设计 Reflect 的目的有：

1. 将`Object`对象的一些明显属于语言内部的方法（比如`Object.defineProperty`），放到`Reflect`对象上。

2. 修改某些Object方法的返回结果，让其变得更合理。比如，`Object.defineProperty(obj, name, desc)`在无法定义属性时，会抛出一个错误，而`Reflect.defineProperty(obj, name, desc)`则会返回`false`。

   ```javascript
   // 之前的写法
   try {
     Object.defineProperty(target, property, attributes);
     // success
   } catch (e) {
     // failure
   }
   ```

   ```javascript
   // 现在的写法
   if (Reflect.defineProperty(target, property, attributes)) {
     // success
   } else {
     // failure
   }
   ```

3. 让`Object`操作都变成函数行为。某些`Object`操作是命令式，比如`name in obj`和`delete obj[name]`，而`Reflect.has(obj, name)`和`Reflect.deleteProperty(obj, name)`让它们变成了函数行为。

   ```javascript
   // 老写法
   'assign' in Object //true
   
   // 新写法
   Reflect.has(Object, 'assign') //true
   ```

4. `Reflect`对象的方法与`Proxy`对象的方法一一对应，只要是`Proxy`对象的方法，就能在`Reflect`对象上找到对应的方法。这就让`Proxy`对象可以方便地调用对应的`Reflect`方法，完成默认行为，作为修改行为的基础。也就是说，不管`Proxy`怎么修改默认行为，你总可以在`Reflect`上获取默认行为。

   ```javascript
   var loggedObj = new Proxy(obj, {
     get(target, name) {
       console.log('get', target, name);
       return Reflect.get(target, name);
     },
     deleteProperty(target, name) {
       console.log('delete' + name);
       return Reflect.deleteProperty(target, name);
     },
     has(target, name) {
       console.log('has' + name);
       return Reflect.has(target, name);
     }
   });
   ```

   

