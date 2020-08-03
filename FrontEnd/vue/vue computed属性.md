# vue computed

先来一个 computed 最简单的例子：

```javascript
<div id="app">
  <span @click="change">{{sum}}</span>
</div>
<script src="./vue2.6.js"></script>
<script>
  new Vue({
    el: "#app",
    data() {
      return {
        count: 1,
      }
    },
    methods: {
      change() {
        this.count = 2
      },
    },
    computed: {
      sum() {
        return this.count + 1
      },
    },
  })
</script>
```

这个例子很简单，刚开始的时候上面是2，点击之后变成3

## 初始化 computed

Vue会在初始化的时候对每个 computed 属性也用 watcher去包装起来(和之前所有正常属性一样)：

```javascript
if (opts.computed) { initComputed(vm, opts.computed); }
// 如果 new Vue(options)的 options 中存在computed 属性，则初始化 computed 属性。
```

进入 initComputed 方法看看

```javascript
var watchers = vm._computedWatchers = Object.create(null);

// 依次为每个 computed 属性定义
for (const key in computed) {
  const userDef = computed[key]
  watchers[key] = new Watcher(
      vm, // 实例
      getter, // 用户传入的求值函数 sum
      noop, // 回调函数 可以先忽视
      { lazy: true } // 声明 lazy 属性 标记 computed watcher
  )

  // 用户在调用 this.sum 的时候，会发生的事情
  defineComputed(vm, key, userDef)
}
```

首先创建了一个空对象 watchers，并且把每个 computed 中的 key，都作为这个行对象的一个key，而 value 则是一个 **watcher**, 即为每个计算属性都实现了一个 watcher。

其属性简化过后是这样的：

```javascript
{
    deps: [],
    dirty: true,
    getter: ƒ sum(),
    lazy: true,
    value: undefined
}
```

刚开始的时候 value 为 undefined，而 lazy 为 true，表示在需要的时候才会去计算，dirty 为 true，表示需要重新计算这个值， 可以看到上面代码最后有一个 definecomputed，代码如下：

```javascript
Object.defineProperty(vm, 'sum', { 
    get() {
        // 从刚刚说过的组件实例上拿到 computed watcher
        const watcher = this._computedWatchers && this._computedWatchers[key]
        if (watcher) {
          // ✨ 注意！这里只有dirty了才会重新求值
          if (watcher.dirty) {
            // 这里会求值 调用 get
            watcher.evaluate()
          }
          // ✨ 这里也是个关键 等会细讲
          if (Dep.target) {
            watcher.depend()
          }
          // 最后返回计算出来的值
          return watcher.value
        }
    }
})
```

即， 先根据_computedWatchers拿到这个属性的 watcher，查看 watcher 是否 dirty，如果是的话，就会执行 watcher.evaluate() 这个函数去重新计算值，如果 dirty 为 false 的时候，就直接读取值。在第一次进入的时候，因为默认值dirty 为 true，所以一定会求值一次。

而 evaluate 函数内容为：

```javascript
// 调用的是 watcher.evaluate(), 所以下面的 this 指的是 watcher
evaluate () {
  // 调用 get 函数求值
  this.value = this.get()  
  // 把 dirty 标记为 false， 计算过一次之后下次如果不是 dirty 就不用重新计算了
  this.dirty = false 
}
```

而上面的 get 函数为

```javascript
get () {
  pushTarget(this) // 全局的 Dep.target 是用一个栈来表示的，方便前进和回退Dep.target, 这里先把这个computed watcher 添加到这个栈中, 即这时候Dep.target 是 computed watcher
  let value
  const vm = this.vm
  try {
    value = this.getter.call(vm, vm) //
  } finally {
    popTarget()
  }
  return value
}
```



## 计算属性的更新

计算属性，即一定会依赖于一个已有的 data，那么如果这个已有的 data 变化了，如何通知这个 computed 属性改变呢。

因为在为 sum 这个computed 值创建 watcher 的时候，相当于给 sum  的 Dep 中添加了一个渲染 watcher，也就是说此时的 Dep 中是有一个渲染 watcher 的， 即如果 sum 改变的话，也应该要通知相应的渲染 watcher 来进行重新渲染，但是这个 watcher 还没有加入 sum 的 Dep 中。

```javascript
if (watcher.dirty) {
            // 这里会求值 调用 get
    watcher.evaluate() // 里面调用了 this.get， 把Dep.target 指向这个 computed watcher
}
```



而如果 dirty 为 true 的时候，computed watcher调用了 evaluate 方法，而 evaluate 中调用了 watcher 的 get 方法，相当于把 Dep.target = 这个 computed watcher. 

之后通过用户传入的 sum 属性计算computed 的返回结果:

```javascript
value = this.getter.call(vm, vm) // 就是用户传入的 sum 函数 = count + 1
```

注意这里访问了 count 变量，而触发了 count 的 observer 中的 get 属性，从而将这个 watcher 添加到了 count 属性的订阅Dep 中。(通过addDep函数)

```javascript
addDep (dep: Dep) {
  ..前面部分略
 
  this.deps.push(dep) // 会把 count 的 Dep 存放在 watcher 的属性deps中
  // 又带着 watcher 自身作为参数
  // 回到 dep 的 addSub 函数了
  dep.addSub(this) // 把 watcher 添加到 dep 中
}
```

这样一来， sum 的watcher中存有 count 的 dep，count的 dep 中存有  sum的watcher，两个属性相互依赖，即这时候的 watcher 为：

```javascript
{
    deps: [ count的dep ],
    dirty: false, // 求值完了 所以是false
    value: 2, // 1 + 1 = 2
    getter: ƒ sum(),
    lazy: true
}
```

这时候count 的 dep 中为:

```javascript
{
    subs: [ sum的计算watcher ]
}
```

这时候 对'sum'属性劫持还没有结束，还有剩下的一部分

```javascript
get() {
          // 此时函数执行到了这里
          if (Dep.target) { // 因为上一步结束之后 popTarget，所以现在 Dep.target 指向渲染 count 的watcher
            watcher.depend()
          }
          return watcher.value
        }
    }
```

因此通过 watcher.depend()把这个渲染 watcher 加入 count 的Dep 中, 此时 count 的 Dep 为

```javascript
{
    subs: [ sum的计算watcher，渲染watcher ]
}
```

这样一来当 count 改变的额时候，就会触发 sum 的计算 watcher，从而触发更新



## 如何更新

```javascript
update () {
  if (this.lazy) {
    this.dirty = true
  }
}
```

可以看到，computed watcher更新函数只有一句话，即把这个值得属性设置为 dirty = true，等待下次调用。

而渲染 watcher 的更新其实就是根据 render 函数生成的vnode 重新渲染视图，在 render 过程中，一定会访问到 sum 这个值，于是又回到了 sum 的定义上

```javascript
Object.defineProperty(vm, 'sum', { 
    get() {
        const watcher = this._computedWatchers && this._computedWatchers[key]
        if (watcher) {
          // ✨上一步中 dirty 已经置为 true, 所以会重新求值
          if (watcher.dirty) {
            watcher.evaluate()
          }
          if (Dep.target) {
            watcher.depend()
          }
          // 最后返回计算出来的值
          return watcher.value
        }
    }
})
```

根据上面的总结，只有当依赖属性出现变化的时候才会将 dirty 设置为true，其他情况都是直接调用，不再重复计算



## 总结

* 和data 中的属性一样，vue 给每个 computed 属性(例如 sum)也都定义了一个 watcher，watcher 中保存了用户传入的计算方法， lazy, dirty, deps等属性
* 同时劫持了sum 属性的 get 方法，在其中得到对应的 watcher，将其赋给 Dep.target，然后通过用户传入的计算方法(sum = count + 1), 访问 count 属性的 get 方法，在 get 方法中把computed watcher 加入 count 的 Dep 中。 
* 系统通过一个 targetStack 来保留 Dep.target 的值，方便前进后退，当 computed watcher 已经加入到count 的 Dep 中后，执行 targetStack.pop()， 使得 Dep.target = 渲染 watcher
* 在判断 Dep.target 是否存在，并将渲染 watcher 也添加到 count 的 Dep.sun 中。
* 当值发生改变的时候 computed watcher 将dirty 设为 true, 当访问到 sum 的时候将进行重新计算。



参考:

1. [Vue 的计算属性真的会缓存吗？（保姆级教学，原理深入揭秘)](https://zhuanlan.zhihu.com/p/129017942?utm_source=wechat_session&utm_medium=social&utm_oi=984754533125271552&utm_content=first)