# Js 模块化&& 加载机制

模块化是一种处理复杂系统分解为更好的可管理模块的方式。简单来说就是为了**解耦**, 简单开发，一个模块就是一部分实现特定功能的文件，可以更方便的使用别人的代码，例如需要什么模块，需要什么功能。



## CommonJs

CommonJs 是对于 node 的模块规范，前端的 webpack也对于commonJs 原生支持。即(**不能在浏览器环境直接使用 require**，require要么用 webpack 打包之后使用，要么在 node 下使用)

**特点** :

1. 模块输出的是一个值得拷贝，模块是**运行时加载，同步加载**
2. commonJs模块的顶层 this指向当前模块。

2个 API:

* require() : 记载所需要的模块
* module.exports或者 exports: 对外暴露的接口。

```javascript
//A.js
module.exports = {
  	a:1,
  	b:2
}

// B.js
require('./A.js')
```

* exports 是对 module.exports 的引用，exports 不能直接赋值，结果会是一个空对象，但是 module.exports 可以直接赋值。
* 一个文件不能写多个 module.exports, 如果写多个的话暴露的是最后一个。
* 如果没有写 module.exports 或者 exports, 而其他文件里面使用 require 引用，则得到的是一个{}空对象。



## AMD 规范

全称为 Asynchronous module definition, 中文名异步模块定义，它是一个在浏览器端模块开发的规范。

**特点** ：

1. 异步加载，不阻塞页面加载，能够并行加载多个模块，但是不能按需加载。必须提前加载所需依赖。

2个 API:

* define(id, [], callbacl): id表示 id，可选，[]表示是否依赖其他模块，可选，callback 为回调
* require([module], callback)：[callback]表示指定加载的模块，callback 表示一个回调函数。

还有一个配置属性 API

```javascript
require.config({

  baseUrl: //基本路径

  paths：// 对象，对外加载的模块名称  ： 键值关系，键：自定义模块名称，值 ：文件名或者文件路径(不要写文件后缀.js),可以是字符串，数组（如果第一个加载失败，会加载第二个）

  shim：//对象，配置非AMD 模式的文件，每个模块要定义（1）exports：值（指在js文件暴露出来的全局变量，如：window.a）（2）deps： 数组，表明该模块的依赖性

})
```



## Es6 Module

ES6到来,完全可以取代 CommonJS 和 AMD规范，成为浏览器和服务器通用的模块解决方案。

特点:

1. ES6 模块之中，顶层的`this`指向`undefined`，即不应该在顶层代码使用`this`。
2. 自动采用严格模式"use strict"。须遵循严格模式的要求
3. ES6 模块的设计思想是尽量的静态化，编译时加载”或者静态加载，编译时输出接口
4. ES6 模块**`export`**`、**`import`**`命令可以出现在模块的任何位置，但是必须处于模块顶层。如果处于块级作用域内，就会报错
5. ES6 模块输出的是值的**引用**

2个 API:

* export: 用于规定模块的对外接口，
* import: 用于输入其他模块提供的功能。

### export & import

export 可以输出变量，数组，class，等各种东西，输出的值只是一个引用。**注意在`<script>`标签中如果要使用 有 import/export 的文件时候，要写type = 'module'**

```javascript
<script scr = './index.js' type = 'module'>
```

export 文件:

```javascript
//A.js
function v1() {  }
function v2() {  }
class class1{}
var m = 1;
export {
  v1 as v1,
  v2 as v2,
  v2 as v3,
  m as m,
  class1 as c1,
};
```

import 命令:

```javascript
//B.js

//静态加载,只加载A.js 文件中三个变量，其他不加载
import {v1, v2, v3} from './A.js';

//import命令要使用as关键字，将输入的变量重命名。
import {v1 as fn1} from './A.js';

//整体加载模块
improt * as all from './A.js'
```

### export default

```javascript
export default function crc32() { // 输出
  // ...
}

import crc32 from 'crc32'; // 输入

// 第二组
export function crc32() { // 输出
  // ...
};

import {crc32} from 'crc32'; // 输入
```

可以看到，加了 default 和不加 default 的区别就是，在 import 的时候，加了 default 的可以不需要加大括号。如果不适用 default, 则需要加大括号。

这种写法也可以写成内联的形式:

```javascript
<script type="module">
  import utils from "./utils.js";

  // other code
</script>
```



在加载 js 的时候，如果`<script>`标签加了`type = 'module'`，则都是异步加载，不会阻塞浏览器。



## Js 加载机制 和阻塞问题

### `<script>`

什么都不加的 script 标签，在加载的时候，运行的时候，都会阻塞`<script>`标签下的 html 文档的解析

### `<script async>`

增加了 async 的`script`标签，会**异步加载**，即加载过程不会阻塞 html 文档解析，但是它是**加载完立即执行**，执行过程会阻塞 html 文档解析

### `<script defer>`

增加了 defer 的`script`标签，同样**异步加载**,  即加载过程也不会阻塞 html 文档解析，但是**加载完成后，不立即执行**, 而是等到 html 文档解析结束之后才运行, 即不会有阻塞 html 文档解析的过程。



一张神图！！一目了然！！

![](./img/56.png)



参考：

1. [前端模块化小总结—commonJs,AMD,CMD, ES6 的Module](https://www.cnblogs.com/beyonds/p/8992619.html)
2. [defer和async的区别]()(https://segmentfault.com/q/1010000000640869)

