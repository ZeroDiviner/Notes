# Vue 生命周期

vue 生命周期的8个阶段

源码中调用生命周期都使用的是`callHook`方法， 文件位置在vue/src/core/instance/

```javascript
export function callHook (vm: Component, hook: string) {
  // #7573 disable dep collection when invoking lifecycle hooks
  pushTarget()
  const handlers = vm.$options[hook]
  if (handlers) {
    for (let i = 0, j = handlers.length; i < j; i++) {
      try {
        handlers[i].call(vm)
      } catch (e) {
        handleError(e, vm, `${hook} hook`)
      }
    }
  }
  if (vm._hasHookEvent) {
    vm.$emit('hook:' + hook)
  }
  popTarget()
}
```

callHook 的逻辑相当简单，传入字符串hook, 去 vm.$options[hook]拿到对应的回调函数数组，然后遍历执行，执行的时候上下文为 vm

对应的执行：

```javascript
Vue.prototype._init = function (options?: Object) {
  // ...
  initLifecycle(vm) // 准备好生命周期相关的各种函数，并挂载在 vm.$options 下
  initEvents(vm)  // 准备 vue 中好事件相关的各种函数，例如 $on, $off, $once $emit..
  initRender(vm) //准备好渲染函数
  callHook(vm, 'beforeCreate')
  initInjections(vm) // resolve injections before data/props
  initState(vm)
  initProvide(vm) // resolve provide after data/props
  callHook(vm, 'created')
  // ...
}
```



1. 在 beforeCreate 之前：
   *  准备生命周期相关的函数
   * 准备事件处理的函数( on, emit, once 等等)
   * 准备 render 函数
2. **beforeCreate**:  可以看到 beforeCreate 之后才进行双向绑定，因此**这里不能访问到数据相关的东西**
3. 在 Created 之前：
   * initState实例化数据，双向绑定等等。绑定 props, data, methods, watch, computed 等属性
4. created: data, methods, watch, computed 都已经完成操作，**最早可以访问数据相关的钩子函数**。
5. beforeMount: 执行到这个位置的时候，template 和 render 函数应该已经都准备好了，Vnode 已经保存了虚拟 dom
6. Mounted: 执行到这里的时候已经挂载完毕，是最早可以访问到真实 dom 的时机
7. beforeUpdate： 当执行这个钩子时，页面中的显示的数据还是旧的，data中的数据是更新后的， 页面还没有和最新的数据保持同步
8. updated：页面显示的数据和data中的数据已经保持同步了，都是最新的
9. beforeDestory：Vue实例从运行阶段进入到了销毁阶段，这个时候上所有的 data 和 methods ， 指令， 过滤器 ……都是处于可用状态。
10. destroyed： 这个时候上所有的 data 和 methods ， 指令， 过滤器 ……都是处于不可用状态。组件已经被销毁了。

参考

1. [生命周期](https://ustbhuangyi.github.io/vue-analysis/v2/components/lifecycle.html#beforedestroy-destroyed)

