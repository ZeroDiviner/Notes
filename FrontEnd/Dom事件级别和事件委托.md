# Dom事件级别

**事件**： 是文档或浏览器窗口中发生的一些特定的交互瞬间。**JavaScript与HTML之间的交互是通过事件实现的**。

**事件流**：是事件在目标元素和其祖先元素之间的触发顺序。

W3C 规定，一个事件分为三个阶段：

* 捕获阶段： 事件从顶层元素到达目标元素的父元素
* 目标阶段： 事件到达元素，如果指定不冒泡，则就会在这里终止
* 冒泡阶段： 事件从目标元素父元素一直向上到顶层元素 window



## Dom 0级

通过 一个属性 onclick将一个函数赋值给它，这样的缺点是**一个处理程序无法同时绑定多个处理函数**

```javascript
<button id="btn" type="button"></button>

<script>

    var btn = document.getElementById('btn');

    btn.onclick = function() {

        alert('Hello World');

    }

    // btn.onclick = null; 解绑事件

</script>
```

## Dom1级

不存在

## Dom 2级

定义了 addEventListener 和 removeEventListener来绑定和解绑事件。

语法：

target.addEventListener(type, listener, options);

* type： 表示事件类型，例如'click'
* Listner: 表示触发事件时候的回调函数
* options:可选，可用选项如下
  * capture： true 或则 false，默认是 false，表示事件会在冒泡阶段触发，如果是 true，则代表事件会在捕获阶段触发
  * once： 如果为 true 则代表该事件处理只会触发一次。
  * passive： 如果为 true，表示 listener 永远不会调用 preventDefault(). 如果调用则会收到控制台警告。

```javascript
<button id="btn" type="button"></button>

<script>

    var btn = document.getElementById('btn');

    function showFn() { alert('Hello World'); }

    btn.addEventListener('click', showFn, false);

  // btn.removeEventListener('click', showFn, false); 解绑事件

</script>(IE8级以下版本只支持冒泡型事件)
```



## Dom3级

在DOM2级事件的基础上添加了更多的事件类型，全部类型如下：

* UI事件，当用户与页面上的元素交互时触发，如：load、scroll
* 焦点事件，当元素获得或失去焦点时触发，如：blur、focus
* 鼠标事件，当用户通过鼠标在页面执行操作时触发如：dbclick、mouseup
* 滚轮事件，当使用鼠标滚轮或类似设备时触发，如：mousewheel
* 文本事件，当在文档中输入文本时触发，如：textInput
* 键盘事件，当用户通过键盘在页面上执行操作时触发，如：keydown、keypress
* 合成事件，当为IME（输入法编辑器）输入字符时触发，如：compositionstart
* 变动事件，当底层DOM结构发生变化时触发，如：DOMsubtreeModified
* 同时DOM3级事件也**允许使用者自定义一些事件**。



## event 对象

event 对象有许多属性和方法

1. preventDefault(): 阻止默认行为。例如点击 submit 的时候，阻止表单提交
2. stopPropagation(): 停止事件冒泡，即停在目标阶段，防止事件冒泡带来的负面影响（在当前事件的处理程序中 return false 也可以阻止冒泡）
3. stopImmediatePropagation (): 阻止后续事件，除了阻止冒泡之外，如果当前事件被绑定了多个处理程序的时，后续处理程序也会终止
4. currentTarget 返回当前事件所绑定的对象
5. target：返回当前触发事件的对象，不一定是绑定事件的对象。



## 事件委托

事件委托又叫做事件代理，从 Js 设计上来讲，事件委托就是**利用事件冒泡**，只指定一个事件处理程序， 就可以管理某一个类型的所有事件。

为什么需要事件委托？

> 加入我们有100个 li 标签，我们需要处理每个标签的点击事件，那么就需要用 for 循环，给每个li 标签增加点击事件，而页面上的事件处理程序的数量越多，性能就会越低。因为需要不断的和 dom交互，而交互越多引起的回流和重绘就越多，而使用事件委托可以将多个事件浓缩成一个，只需要一次 dom 交互，注册的事件也更少了，性能也会更好。

事件委托原理?

> 事件委托是依靠冒泡来实现的，所谓事件冒泡就是从最深的节点开始，向 document 传递的过程。假如有 div>ul>li>a 这样的结构，我们对于多个 li 标签，可以给 ul 绑定事件来监听，这样在事件冒泡阶段父元素 ul 就会监听到事件发生。
>
> 可以直接在 ul 上添加事件监听
>
> ```javascript
> let ul = document.getElementById('ul')
> ul.addEventListener('click',(e)=>{}) // 这里第三个参数不存在的时候默认为 false,表示在冒泡阶段触发
> ```
>
> 但是这样的话因为 ul 也有点击事件，所以当点击 ul 的时候也会触发，这种情况可以在函数中判断 target，看看是否等于 li
>
> ```javascript
> let ul = document.getElementById('ul')
> ul.addEventListener('click',(e)=>{
>   	e = e||window.event;
>   	let target = e.target;
>   	if(target.nodeName.toLowerCase() == 'li'){
> 				// event content
> 		}
> })
> ```
>
> 这样一来只需要注册一次，就可以监听所有 li 标签。
>
> 事件委托还有一个好处，即当用 for 循环遍历 li 标签进行监听的时候，当有新添加的元素的时候，新添加元素是无法被监听的，但是如果在父元素监听事件，则新添加的元素也可以被监听。