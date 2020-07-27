# Vue 双向绑定原理

首先一个新建一个 vue对象通常格式为：

```javascript
var vue = new Vue({
    el: '#vue-app',
        data: {
            word: 'Hello World!'
        },
        methods: {
            fun: function() {
                this.word = 'Hi';
            }
        }
})
```

首先看一下一个 MVVM 对象是怎么定义的，就类似于上面的 Vue 对象

```javascript
function MVVM(options) {
    this.$options = options || {}; // this.$options 就指代Vue(obj)中的那个 obj,可以有 data, method, computed...
    var data = this._data = this.$options.data;
    var me = this;

    // 数据代理
    // 实现 vm.xxx -> vm._data.xxx
    Object.keys(data).forEach(function(key) {
        me._proxyData(key); //代理 data，返回的其实是 this._data[key]
    });

    this._initComputed(); // 这部分不多说

    observe(data, this); // 观察者模式，检测数据变动

    this.$compile = new Compile(options.el || document.body, this)
}

MVVM.prototype = {
    constructor: MVVM,
    $watch: function(key, cb, options) {
        new Watcher(this, key, cb);
    },

    _proxyData: function(key, setter, getter) {
        var me = this;
        setter = setter || 
        Object.defineProperty(me, key, {
            configurable: false,
            enumerable: true,
            get: function proxyGetter() { //代理数据，对于每个  this.data[key]都返回this._data[key]
                return me._data[key];
            },
            set: function proxySetter(newVal) { // 修改数据则直接修改 this._data[key]
                me._data[key] = newVal;
            }
        });
    },

    _initComputed: function() { //初始化 computed，这部分这里不多说
        var me = this;
        var computed = this.$options.computed; //得到对应的 obj.computed
        if (typeof computed === 'object') {
            Object.keys(computed).forEach(function(key) {
                Object.defineProperty(me, key, {
                    get: typeof computed[key] === 'function' 
                            ? computed[key] 
                            : computed[key].get,
                    set: function() {}
                });
            });
        }
    }
};
```

上面的 observe 如下，下面还包含了 Dep，这个可以理解为发布订阅模式中的消息中心，会保留对应的订阅信息，每个数据(object 级别的)都有自己独立的一个 Dep来保留订阅，在对应的数据的 setter 中，如果数据改变，触发的也是这个数据的订阅的函数：

```javascript
function Observer(data) {
    this.data = data; //observer 的 this 也会留存数据
    this.walk(data); // walk 就是走一便数据，做数据劫持
}

Observer.prototype = {
    constructor: Observer,
    walk: function(data) { // walk 就是走一便数据，做数据劫持
        var me = this;
        Object.keys(data).forEach(function(key) {
            me.convert(key, data[key]); //都调用了 convert 这个 function
        });
    },
    convert: function(key, val) {
        this.defineReactive(this.data, key, val); // 都调用了 defineReactive 这个 function
    },

    defineReactive: function(data, key, val) {
        var dep = new Dep(); //消息中心构建
        var childObj = observe(val); // 递归观察子属性

        Object.defineProperty(data, key, {
            enumerable: true, // 可枚举
            configurable: false, // 不能再define
            get: function() {
                if (Dep.target) { // Dep.target 默认为null, 在 watcher 中，触发这个 get 方法，这个 get 方法就会执行，从而把一个 watcher 加入到 dep.sub 中
                    dep.depend(); // 执行到这里的时候 dep.target 指向一个 watcher，Dep.depend()把这个 watcher 添加到 dep.sub 中完成订阅
                }
                return val;
            },
            set: function(newVal) {
                if (newVal === val) {
                    return;
                }
                val = newVal;
                // 新的值是object的话，进行监听
                childObj = observe(newVal); //监听新 obj 属性
                // 通知订阅者
                dep.notify(); //属性发生改变，观察中心通知所有订阅此事件的 watcher
            }
        });
    }
};

function observe(value, vm) { // 这里是上一个函数块中的 observe，可以看到会返回一个 Observer 对象，这个对象会递归劫持所有 data 中的数据
    if (!value || typeof value !== 'object') {
        return;
    }

    return new Observer(value);
};


var uid = 0;

function Dep() {
    this.id = uid++;
    this.subs = [];
}

Dep.prototype = {
    addSub: function(sub) { //添加订阅
        this.subs.push(sub);
    },

    depend: function() { //调用上面的方法添加订阅
        Dep.target.addDep(this);
    },

    removeSub: function(sub) { //移除订阅
        var index = this.subs.indexOf(sub);
        if (index != -1) {
            this.subs.splice(index, 1);
        }
    },

    notify: function() { //调用所有和这个属性相关的watcher 的update()，update()会根据
        this.subs.forEach(function(sub) {
            sub.update();
        });
    }
};

Dep.target = null;
```

可以看到上面说到了 Observe 中对于每个 obj 都有其消息中心 Dep，通过 Observer 的 get 方法和 Dep.target 属性把一个 watcher 添加到 Dep 中。那么下面看 watcher 是如何被调用的。

上面创建一个 MVVM 的最后一步如下图，可以看到是创建了一个 compiler

```javascript
this.$compile = new Compile(options.el || document.body, this)
```

Compile 代码如下图：

```javascript
function Compile(el, vm) {
    this.$vm = vm; //这里的 vm就等于 MVVM 对象
    this.$el = this.isElementNode(el) ? el : document.querySelector(el);//查询 el,找啊到对应的 node

    if (this.$el) {
        this.$fragment = this.node2Fragment(this.$el);// 构建虚拟 dom, 赋值给 this.$fragment
        this.init(); // this.$fragment下的所有子节点还都是原生节点，init()通过 compileElement(this.$fragment)初始化所有结点值。
        this.$el.appendChild(this.$fragment); //替换 dom
    }
}

Compile.prototype = {
    constructor: Compile,
    node2Fragment: function(el) {
        var fragment = document.createDocumentFragment(), //通过 document.createDocumentFragment 来创建虚拟 dom
            child;

        // 将原生节点拷贝到fragment
        while (child = el.firstChild) {
            fragment.appendChild(child);
        }

        return fragment;
    },

    init: function() { // 初始化结点
        this.compileElement(this.$fragment);
    },

    compileElement: function(el) {
        var childNodes = el.childNodes, // 对于$fragment 下的所有子节点
            me = this;

        [].slice.call(childNodes).forEach(function(node) {
            var text = node.textContent;
            var reg = /\{\{(.*)\}\}/;

            if (me.isElementNode(node)) { // 是 elementNode 类型的，就是大多数常见结点类型
                me.compile(node); //对 node 进行 compile

            } else if (me.isTextNode(node) && reg.test(text)) { //如果是文本类型并且是{{}}形式的
                me.compileText(node, RegExp.$1.trim()); //调用compileText 方法
            }

            if (node.childNodes && node.childNodes.length) {
                me.compileElement(node);//递归 compile 子属性
            }
        });
    },

    compile: function(node) {
        var nodeAttrs = node.attributes,
            me = this;

        [].slice.call(nodeAttrs).forEach(function(attr) {
            var attrName = attr.name;
            if (me.isDirective(attrName)) { //判断有没有 v-model,v-html 这样的
                var exp = attr.value;//属性的值，例如 v-model = 'name' ，那么 exp 就是 name
                var dir = attrName.substring(2); // dir 这是例如 ，v-model 对应 model
                // 事件指令
                if (me.isEventDirective(dir)) {//判断是否有 :onclick 这种事件
                    compileUtil.eventHandler(node, me.$vm, exp, dir);//传参分别是(node本身，vm 对象，attr 的值，事件名称)
                    // 普通指令
                } else { // 没有事件
                    compileUtil[dir] && compileUtil[dir](node, me.$vm, exp); //根据对应属性,例如 model, html 去找处理这个结点的对应方法
                }

                node.removeAttribute(attrName); // 去掉这个属性，v-model这种属性不能暴露在 html 中
            }
        });
    },

    compileText: function(node, exp) { //compileText方法, 这个 exp 是''
        compileUtil.text(node, this.$vm, exp); //调用compileUtil.text 方法， 处理 text 文本
    },

    isDirective: function(attr) { // 判断是否是 v-model, v-html 等等
        return attr.indexOf('v-') == 0;
    },

    isEventDirective: function(dir) { // 绑定是事件类型
        return dir.indexOf('on') === 0;
    },

    isElementNode: function(node) { //是 element 类型，Element, Text
        return node.nodeType == 1;
    },

    isTextNode: function(node) { // 判断是否是文本类型
        return node.nodeType == 3;
    }
};

// 指令处理集合
var compileUtil = {
    text: function(node, vm, exp) { // 对于 v-text 和 v-html 都用 Bind 方法
        this.bind(node, vm, exp, 'text');
    },

    html: function(node, vm, exp) {// 对于 v-text 和 v-html 都用 Bind 方法
        this.bind(node, vm, exp, 'html');
    },

    model: function(node, vm, exp) { // 对于 model 不仅用 bind方法，还要给其添加 eventListener 监控用户输入改变值
        this.bind(node, vm, exp, 'model');

        var me = this,
            val = this._getVMVal(vm, exp);
        node.addEventListener('input', function(e) {
            var newValue = e.target.value;
            if (val === newValue) {
                return;
            }

            me._setVMVal(vm, exp, newValue); // 传的值为 node对应 vm, this.$vm对应 exp，新值, 
            val = newValue;
        });
    },

    class: function(node, vm, exp) { // 对于 v-class, 调用 bind 方法
        this.bind(node, vm, exp, 'class');
    },

  
   //可以看到除了处理事件类型不会调用 bind, 其他符合要求的类型都会调用 bind 属性来新建 watcher 对其进行订阅。
    bind: function(node, vm, exp, dir) { // 根据传来的 dir 属性，例如 text, model, html 去找对应的处理器。
        var updaterFn = updater[dir + 'Updater'];

        updaterFn && updaterFn(node, this._getVMVal(vm, exp)); // 对应的替换事件，例如 v-text 替换内容，v-html 替换 html， v-class 替换增加 class

        new Watcher(vm, exp, function(value, oldValue) {
            updaterFn && updaterFn(node, value, oldValue); //新建一个 watcher，并且其回调函数为 updater 中对应的更新函数。
        });
    },

    // 事件处理
    eventHandler: function(node, vm, exp, dir) {
        var eventType = dir.split(':')[1],
            fn = vm.$options.methods && vm.$options.methods[exp];

        if (eventType && fn) {
            node.addEventListener(eventType, fn.bind(vm), false);
        }
    },

    _getVMVal: function(vm, exp) {
        var val = vm;
        exp = exp.split('.');
        exp.forEach(function(k) {
            val = val[k];
        });
        return val;
    },

    _setVMVal: function(vm, exp, value) { //三个参数分别对应这个 node, this.$vm, 新的值
        var val = vm;
        exp = exp.split('.');
        exp.forEach(function(k, i) {
            // 非最后一个key，更新val的值
            if (i < exp.length - 1) {
                val = val[k];
            } else {
                val[k] = value; // 这里是为什么不懂...
            }
        });
    }
};


var updater = {
    textUpdater: function(node, value) {
        node.textContent = typeof value == 'undefined' ? '' : value;
    },

    htmlUpdater: function(node, value) {
        node.innerHTML = typeof value == 'undefined' ? '' : value;
    },

    classUpdater: function(node, value, oldValue) {
        var className = node.className;
        className = className.replace(oldValue, '').replace(/\s$/, '');

        var space = className && String(value) ? ' ' : '';

        node.className = className + space + value;
    },

    modelUpdater: function(node, value, oldValue) {
        node.value = typeof value == 'undefined' ? '' : value;
    }
};
```

可以看到，一个 compiler 

* 先用 document.createDocumentFragment 来创建了一个虚拟dom, 之后
* 根据其 node 类型还有node 的属性类型对其进行了初始化。
* 即如果是 Text 且没有属性则直接赋值，如果是 element 则对其 child 递归 compile。
* 如果判断到了有符合标准的属性，例如 v-model, v-html, v-text 等等，则对其进行 bind，如果绑定的是 v-model 还要对该 node 进行 input 监听，如果用户输入并修改了值，则要同时修改对应$vm.data 中的值
* bind 方法的内容是： 先从 updater 中找到对应改变应该调用的方法。然后新建一个 watcher，并把方法作为 watcher 的回调函数。这样一来，如果data 中的值变动，Observer 中的 setter 就会调用对应的 Dep 进行 notify，对应的 watcher 收到信息，就会执行相应的回调函数，这样一来就会重新渲染。

那么就来看最后一部分 Watcher

```javascript
function Watcher(vm, expOrFn, cb) {
    this.cb = cb; // 回调函数，在上一步 bind 后面传入
    this.vm = vm; //vm 也保存一份$vm
    this.expOrFn = expOrFn;
    this.depIds = {};

    if (typeof expOrFn === 'function') {
        this.getter = expOrFn;
    } else {
        this.getter = this.parseGetter(expOrFn.trim());
    }

    this.value = this.get(); // ！！！ 重要的地方，调用自己的 get 方法， 这个方法能够让自己添加到对应 Dep 的 sublist 中
}

Watcher.prototype = {
    constructor: Watcher,
    update: function() { // 触发 watcher 的更新功能，调用 watcher.cb 来更新对应的dom
        this.run();
    },
    run: function() {
        var value = this.get();
        var oldVal = this.value;
        if (value !== oldVal) {
            this.value = value;
            this.cb.call(this.vm, value, oldVal);
        }
    },
    addDep: function(dep) {
        // 1. 每次调用run()的时候会触发相应属性的getter
        // getter里面会触发dep.depend()，继而触发这里的addDep
        // 2. 假如相应属性的dep.id已经在当前watcher的depIds里，说明不是一个新的属性，仅仅是改变了其值而已
        // 则不需要将当前watcher添加到该属性的dep里
        // 3. 假如相应属性是新的属性，则将当前watcher添加到新属性的dep里
        // 如通过 vm.child = {name: 'a'} 改变了 child.name 的值，child.name 就是个新属性
        // 则需要将当前watcher(child.name)加入到新的 child.name 的dep里
        // 因为此时 child.name 是个新值，之前的 setter、dep 都已经失效，如果不把 watcher 加入到新的 child.name 的dep中
        // 通过 child.name = xxx 赋值的时候，对应的 watcher 就收不到通知，等于失效了
        // 4. 每个子属性的watcher在添加到子属性的dep的同时，也会添加到父属性的dep
        // 监听子属性的同时监听父属性的变更，这样，父属性改变时，子属性的watcher也能收到通知进行update
        // 这一步是在 this.get() --> this.getVMVal() 里面完成，forEach时会从父级开始取值，间接调用了它的getter
        // 触发了addDep(), 在整个forEach过程，当前wacher都会加入到每个父级过程属性的dep
        // 例如：当前watcher的是'child.child.name', 那么child, child.child, child.child.name这三个属性的dep都会加入当前watcher
        if (!this.depIds.hasOwnProperty(dep.id)) {
            dep.addSub(this);
            this.depIds[dep.id] = dep;
        }
    },
    get: function() {  // 这一步就是创建一个 watcher 时候主动调用的，先把 Dep.target 指向自己
        Dep.target = this;//用 Dep.target 缓存自己， Dep 这里是一个全局的 Object，没有实例化
        var value = this.getter.call(this.vm, this.vm); //强行调用 observe中的 get，因为之前已经对所有属性进行了 observe，这一步把自己添加到对应的 Dep 实例的订阅队列中，watcher 有对应的 callback
        Dep.target = null; // 之后把 Dep.target 置空，这样在  observer.get 执行的时候就不会进入 if 判断
        return value;
    }

    parseGetter: function(exp) {
        if (/[^\w.$]/.test(exp)) return; 

        var exps = exp.split('.');

        return function(obj) {
            for (var i = 0, len = exps.length; i < len; i++) {
                if (!obj) return;
                obj = obj[exps[i]];
            }
            return obj;
        }
    }
};
```

可以看到上面这部分中：

* 创建 Watcher 之后主动调用了其 get 方法，其 get 方法中把 Dep.target 指向了自己，先缓存着
* 之后强行调用对应属性的 get 属性，于是把这个 watcher 加入了对应的 Dep 的 subList，
* 当对应属性修改的时候，对应属性的 setter 会触发watcher 的 callback
* watcher 的 callback 是在 compiler 中传递过来的，修改对应的dom 元素值。

参考: [剖析Vue实现原理 - 如何实现双向绑定mvvm](https://github.com/DMQ/mvvm)

