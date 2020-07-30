# Vue tips

1. 为什么 v-for 和 v-if 不能同时使用？

> 因为 v-for 的优先级高于 v-if, 如果他们在同一级一起使用，那么可能造成每一个子元素都会判断一次 v-if， 如果目的是想要跳出循环，可以在外侧加容器添加 v-if。并且 v-if 的机制是为 true 时候添加 dom 元素，为 false 时候删除 dom 元素，因此频繁触发会导致效率降低。

2. 组件中的 data 为什么必须是一个函数

```javascript
data(){
  return {
    data1:'hello',
  }
}
```



> 因为组件可能被复用，如果是一个 function 的话，那么每个函数都可以保留一份返回对象的独立拷贝，如果没有这条规则，那么这个组件被多次复用的时候，对一个组件中的数据进行修改，那么另一个组件中的 data 也会被修改。

3. vue 如何检测数组变化

```javascript
const arrayProto = Array.prototype
// 创建一个空对象arrayMethods，并将arrayMethods的原型指向Array.prototype
export const arrayMethods = Object.create(arrayProto)

// 列出需要重写的数组方法名
const methodsToPatch = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]
// 遍历上述数组方法名，依次将上述重写后的数组方法添加到arrayMethods对象上
methodsToPatch.forEach(function (method) {
  // 保存一份当前的方法名对应的数组原始方法
  const original = arrayProto[method]
  // 将重写后的方法定义到arrayMethods对象上，function mutator() {}就是重写后的方法
  def(arrayMethods, method, function mutator (...args) {
    // 调用数组原始方法，并传入参数args，并将执行结果赋给result
    const result = original.apply(this, args)
    // 当数组调用重写后的方法时，this指向该数组，当该数组为响应式时，就可以获取到其__ob__属性
    const ob = this.__ob__
    let inserted
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2)
        break
    }
    if (inserted) ob.observeArray(inserted)
    // 将当前数组的变更通知给其订阅者
    ob.dep.notify()
    // 最后返回执行结果result
    return result
  })
})
```



> vue 对于数组的 prototype 中的一些方法进行了重写，例如'push',  'pop',  'shift',  'unshift',  'splice',  'sort',  'reverse'，在进行修改的时候通知其订阅者, 但是数组不能检测到 arr[index] = number 或者 arr.length = number 这种动作，即按 index 赋值和修改数组长度的操作是无法被检测到的。但是实际上是可以做到的，因为数组的索引也是属性，完全可以通过属性劫持来做到检测变化。
>
> 
> 对此尤大的回答是：性能代价和获得的用户体验不成正比 