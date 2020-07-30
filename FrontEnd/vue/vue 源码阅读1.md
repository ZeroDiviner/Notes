# Vue 源码阅读1

## 1. 基础使用

```javascript
<div id="app"></div>
<script src="https://cdn.jsdelivr.net/npm/vue@2.6.8/dist/vue.js"></script>
<script>
var vm = new Vue({
  el: '#app',
  data: {
    message: '选项合并'
  },
})
</script>
```

vue 的构造器函数就是上面调用的 new Vue()， 它在内部规定了只能通过 new 的方式而不是函数方式调用，打包之后是符合 UMD 规范的。

实例化 Vue 时候会传给构造函数一个对象，这个对象描述了用户自定义的行为，例如 data定义响应式的数据，computed 描述计算属性，components来注册组件，甚至于一些生命周期相关的钩子函数。除了这些之外，Vue 本身自带一些默认的选项，这些选项和用户自定义的选项会在后续一起参与到`Vue`实例的初始化中。

**Vue 的默认构造器资源配置**如下：

```javascript
Vue.options = {
  components: {
    KeepAlive: {}
    Transition: {}
    TransitionGroup: {}
  },
  directives: {
    model: {inserted: ƒ, componentUpdated: ƒ}
    show: {bind: ƒ, update: ƒ, unbind: ƒ}
  },
  filters: {}
  _base
}
```





在定义了构造函数之后，还对于 VUE 的原型定义了一系列的方法和属性

```javascript
// 定义Vue原型上的init方法(内部方法)
  initMixin(Vue);
  // 定义原型上跟数据相关的属性方法
  stateMixin(Vue);
  //定义原型上跟事件相关的属性方法
  eventsMixin(Vue);
  // 定义原型上跟生命周期相关的方法
  lifecycleMixin(Vue);
  // 定义渲染相关的函数
  renderMixin(Vue);
```



## 2. 定义原型上的属性和方法

即上面的5个函数，定义了原型上的一些方法属性

### initMixin()

```javascript
function initMixin (Vue) {
  Vue.prototype._init = function (options) {}
}
```

定义了**内部在实例化`Vue`时会执行的初始化代码**， 是一个内部使用的方法。

### stateMixin()

```javascript
function stateMixin (Vue) {
    var dataDef = {};
    dataDef.get = function () { return this._data };
    var propsDef = {};
    propsDef.get = function () { return this._props };
    {
      dataDef.set = function () {
        warn(
          'Avoid replacing instance root $data. ' +
          'Use nested data properties instead.',
          this
        );
      };
      propsDef.set = function () {
        warn("$props is readonly.", this);
      };
    }
    // 代理了_data,_props的访问
    Object.defineProperty(Vue.prototype, '$data', dataDef);
    Object.defineProperty(Vue.prototype, '$props', propsDef);
    // $set, $del
    Vue.prototype.$set = set;
    Vue.prototype.$delete = del;

    // $watch
    Vue.prototype.$watch = function (expOrFn,cb,options) {};
  }
```

定义了和数据相关的属性方法，例如可以通过实例上的\$prop, ​\$props访问到data 和 props

### eventMixin()

```javascript
function eventsMixin(Vue) {
  // 自定义事件监听
  Vue.prototype.$on = function (event, fn) {};
  // 自定义事件监听,只触发一次
  Vue.prototype.$once = function (event, fn) {}
  // 自定义事件解绑
  Vue.prototype.$off = function (event, fn) {}
  // 自定义事件通知
  Vue.prototype.$emit = function (event, fn) {
}
```

可以看到文档中的\$on,\$once,\$off,\$emit 都是在这里定义的

### lifecycleMixin() & renderMixin()

```javascript

function lifecycleMixin (Vue) {
    Vue.prototype._update = function (vnode, hydrating) {};

    Vue.prototype.$forceUpdate = function () {};

    Vue.prototype.$destroy = function () {}
  }

// 定义原型上跟渲染相关的方法
  function renderMixin (Vue) {
    Vue.prototype.$nextTick = function (fn) {};
    // _render函数，后面会着重讲
    Vue.prototype._render = function () {};
  }

```

这两个函数都是对于生命周期渲染方法的定义

## 3. 定义全局静态属性方法

除了原型链上的方法，还定义了许多全局的 API 方法，这些方法都在initGlobalAPI中定义

例如：

1. 为源码里的`config`配置做一层代理，可以通过`Vue.config`拿到默认的配置，并且可以修改它的属性值，具体哪些可以配置修改，可以先参照官方文档。
2. 定义内部使用的工具方法，例如警告提示，对象合并等。
3. 定义`set,delet,nextTick`方法，本质上原型上也有这些方法的定义。
4. 对`Vue.components,Vue.directive,Vue.filter`的定义，这些是默认的资源选项，后续会重点分析。
5. 定义`Vue.use()`方法
6. 定义`Vue.mixin()`方法
7. 定义`Vue.extend()`方法



## 4. Vue的实例化阶段

从构造器定义可知，实例化`Vue`做的核心操作便是执行`_init`方法进行初始化。初始化包括:

* 选项合并配置
* 初始化生命周期
* 初始化事件中心
* 乃至构建数据响应式系统

关键的第一部就是如何选项合并，即**将用户传入的 new Vue(obj)中的 obj 和 Vue 自身的默认选项的合并**。在其中还会对于用户传入的数据合法性进行检查，例如：

* components 检查: 检查用户没有用 html 保留的标签注册组件名
* props 检查: 正确的两种 props 方式为1，`{ props: ['a', 'b', 'c'] }`, 2.{ props: { a: { type: 'String', default: 'prop校验' } }}
* inject/provide 检查: 略
* directive 检查: 略

在校验之后，合并之前，还要提到一个东西，子类构造器，即 Vue 支持extend，类似于

```javascript
var Parent = Vue.extend({
  data() {
    test: '父类'，
    test1: '父类1'
  }
})
var Child = Parent.extend({
  data() {
    test: '子类',
    test2: '子类1'
  }
})
var vm = new Child().$mount('#app');
console.log(vm.$data);
// 结果 
{
  test: '子类',
  test1: '父类1',
  test2: '子类1'
}
```

因为这个子类的原因，所以产生了不同的合并策略，基于父子类的合并。



## 5. 合并策略

1. `Vue`针对每个规定的选项都有定义好的合并策略，例如`data,component,mounted`等。如果合并的子父配置都具有相同的选项，则只需要按照规定好的策略进行选项合并即可。
2. 由于`Vue`传递的选项是开放式的，所有也存在传递的选项没有自定义选项的情况，这时候由于选项不存在默认的合并策略，所以处理的原则是有子类配置选项则默认使用子类配置选项，没有则选择父类配置选项。

合并大概可以分为下面几类:

* 用`data,props,computed`等选项定义实例数据
* 用`mounted, created, destoryed`等定义生命周期函数
* 用`components`注册组件
* 用`methods`选项定义实例方法



按照合并策略，合并一共可以分为5大类

* **常规选项合并**: 例如 el, data属性的合并
* **自带资源选项合并**: components, directive, filter属性的合并
* **生命周期钩子合并**
  * 如果子类和父类都拥有相同钩子选项，则将子类选项和父类选项合并。
  * 如果父类不存在钩子选项，子类存在时，则以数组形式返回子类钩子选项。
  * 当子类不存在钩子选项时，则以父类选项返回。
  * 子父合并时，是将子类选项放在数组的末尾，这样在执行钩子时，永远是父类选项优先于子类选项执行。
* **`watch`选项合并**
* **`props,methods, inject, computed`类似选项合并**



## 总结

总结一下，第一部分做了构造函数的初始化，并且定义了 Vue 原型链上的一些方法和一些全局的静态属性和方法。在构造函数中，首先要对于用户传入的参数进行合法性检测，之后对于用户选项和 Vue 实例默认自带的选项根据规则进行合并(因为有子类的存在)。

