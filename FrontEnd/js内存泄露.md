# Js 内存泄露

js 的内存管理是有自动的垃圾回收机制的，但是即使是这样，还是会碰到一系列的内存泄露问题。

## Js 回收机制

大多数 GC 都使用一个叫做 mark and sweep 的算法，由以下几个步骤组成

1. 垃圾回收机制建立一个'根节点'列表，通常是那些引用被保留在代码中的全局变量，在 js 中 window就是可以作为根节点
2. window 对象会一直存在，并且被标记为活动的，所有子节点也都被递归检查过，每个可以从根节点访问到的变量都不是垃圾。
3. 即可以从根节点访问到的内存就不是垃圾，否则就是垃圾。



## 3种常见的js 内存泄露

1. ### 意外的全局变量

```javascript
function foo(arg) {
    bar = 'this is a hidden global variable';
}　
```

这个内部的 bar 变量相当于是window.bar = '...'， 这种变量泄露可以通过 use strict; 来避免

另一种全局变量为:

```javascript
function foo() {
    this.variable = "potential accidental global";
}
foo()
// 没有使用 new 操作符，因此这里的 this 指向 window，所以变量泄露了
```

2. ### 遗漏的定时器和回调函数

```javascript
var someResource = getData();
setInterval(function() {
    varnode = document.getElementById('Node');
    if(node) {
        // Do stuff with node and someResource.
        node.innerHTML = JSON.stringify(someResource));
    }
}, 1000);
```

函数周期性执行，代表内部的应用变量页都不能被回收，如果定时器不被清理，那么内部可能会存在很大的对象的引用，也不能被清理。因此当不使用定时器时，显示的清除他是很必要的。

点击事件造成的内存泄露:

```javascript
varelement = document.getElementById('button');
  
function onClick(event) {
    element.innerHtml = 'text';
}
  
element.addEventListener('click', onClick);
// Do stuff
element.removeEventListener('click', onClick);
element.parentNode.removeChild(element);
```

当点击事件绑定的元素被从 dom 中移除的时候，相应的函数等就应该被撤销，但是在一些老的浏览器中是不会这样处理的，需要像上面一样显示的移除。

3. ### Dom 之外的引用

```javascript
varelements = {
    button: document.getElementById('button'),
    image: document.getElementById('image'),
    text: document.getElementById('text')
};
  
function doStuff() {
    image.src = 'http://some.url/image';
    button.click();
    console.log(text.innerHTML);
    // Much more logic
}
  
function removeButton() {
    // The button is a direct child of body.
    document.body.removeChild(document.getElementById('button'));
  
    // At this point, we still have a reference to #button in the global
    // elements dictionary. In other words, the button element is still in
    // memory and cannot be collected by the GC.
}
```

有的时候会吧 dom 元素引用保存在数据结构，例如数组或者字典中，但是要记得清理这些引用，因为当 dom 元素本身被从 dom 树中移除之后，如果还有其他引用，会导致这部分内存无法被回收。

4. ### 闭包

参考:

1. [JavaScript 内存泄露问题](https://www.cnblogs.com/onepixel/p/8832776.html)

