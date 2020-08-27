# keep-alive 及其原理

Keep-alive 本身是一个**抽象组件**, 但是它自身不会渲染一个 dom 元素，也不会出现在父组件链中。keep-alive包裹动态组件的时候，会缓存不活动的组件实例，而不是销毁它们。

Keep-alive可以避免组件的反复创建和渲染，有效提升系统性能，总得来说，keep-alive用于保存组件的渲染状态。

## keep-alive用法

* 在动态组件中使用

```javascript
<keep-alive :include="whiteList" :exclude="blackList" :max="amount">
     <component :is="currentComponent"></component>
</keep-alive>
```

* 在vue-router 中使用

```javascript
<keep-alive :include="whiteList" :exclude="blackList" :max="amount">
    <router-view></router-view>
</keep-alive>
```

其中，`include`定义缓存白名单(缓存内部包含的组件)，`exclude`定义缓存黑名单(不缓存内部包含的组件)，`max`表示缓存组件的上限数量，超出的话使用 LRU 规则置换。



## 源码部分

```javascript
// src/core/components/keep-alive.js
export default {
  name: 'keep-alive',
  abstract: true, // 判断当前组件虚拟dom是否渲染成真实dom的关键
  props: {
      include: patternTypes, // 缓存白名单
      exclude: patternTypes, // 缓存黑名单
      max: [String, Number] // 缓存的组件
  },
  created() {
     this.cache = Object.create(null) // 缓存虚拟dom
     this.keys = [] // 缓存的虚拟dom的键集合
  },
  destroyed() {
    for (const key in this.cache) {
       // 删除所有的缓存
       pruneCacheEntry(this.cache, key, this.keys)
    }
  },
 mounted() {
   // 实时监听黑白名单的变动
   this.$watch('include', val => {
       pruneCache(this, name => matched(val, name))
   })
   this.$watch('exclude', val => {
       pruneCache(this, name => !matches(val, name))
   })
 },

 render() {
    // 先省略...
 }
}
```

这个组件中有一个属性叫做 `abstract`, 当这个属性为 true 的时候，vue 渲染的时候就不会渲染实体dom。

`Keep-alive`源码中又三个生命周期函数:

* Created: 初始化两个对象分别缓存VNode(虚拟DOM)和VNode对应的键集合
* destroyed: 删除this.cache中缓存的VNode实例。使用了 pruneCache 方法，原理是调用了组件的destroy 钩子函数。
* mounted: 在mounted这个钩子中对include和exclude参数进行监听，然后实时地更新（删除）this.cache对象数据。

### render

```javascript
render () {
  const slot = this.$slots.defalut
  const vnode: VNode = getFirstComponentChild(slot) // 找到第一个子组件对象
  const componentOptions : ?VNodeComponentOptions = vnode && vnode.componentOptions
  if (componentOptions) { // 存在组件参数
    // check pattern
    const name: ?string = getComponentName(componentOptions) // 组件名
    const { include, exclude } = this
    if (// 条件匹配
      // not included
      （include && (!name || !matches(include, name))）||
      // excluded
        (exclude && name && matches(exclude, name))
    ) {
        return vnode
    }
    
    const { cache, keys } = this
    // 定义组件的缓存key
    const key: ?string = vnode.key === null ? componentOptions.Ctor.cid + (componentOptions.tag ? `::${componentOptions.tag}` : '') : vnode.key
     if (cache[key]) { // 已经缓存过该组件
        vnode.componentInstance = cache[key].componentInstance
        remove(keys, key)
        keys.push(key) // 调整key排序
     } else {
        cache[key] = vnode //缓存组件对象
        keys.push(key)
        if (this.max && keys.length > parseInt(this.max)) {
          //超过缓存数限制，将第一个删除
          pruneCacheEntry(cahce, keys[0], keys, this._vnode)
        }
     }
     
      vnode.data.keepAlive = true //渲染和执行被包裹组件的钩子函数需要用到
 }
 return vnode || (slot && slot[0])
}
```

1. 第一步： 获取keep-alive包裹的第一个组件对象及其名称
2. 第二步：根据 exclude 和 include进行条件匹配。决定是否缓存，不匹配则直接返回组件实例，否则执行第三部。
3. 第三部:  根据组件 ID 和 tag 生成对应 key，在缓存对象中查找是否存在，如果存在则返回并且更新 key 的位置(LRU用)，不存在则执行第四部
4. 第四步：在this.cache 中存储该组件实例并保存 key 值。之后检查缓存实例是否超过 max，如果是则调用 LRU
5. 第五步：最后一步很重要，将组件的 keep-alive 值置为 true



参考:

1. [keep-alive实现原理](https://www.jianshu.com/p/9523bb439950)

