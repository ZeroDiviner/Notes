# Vue watch

watch是一个侦听的动作，用来观察和响应 Vue 实例上的数据变动。官网上的例子

```javascript
<div id="watch-example">
  <p>
    Ask a yes/no question:
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>
var watchExampleVM = new Vue({
  el: '#watch-example',
  data: {
    question: '',
    answer: 'I cannot give you an answer until you ask a question!'
  },
  watch: {
    // 如果 `question` 发生改变，这个函数就会运行
    question: function (newQuestion, oldQuestion) {
      this.answer = 'Waiting for you to stop typing...'
    }
  },
})
```

使用 watch 选项允许我们执行异步操作 (访问一个 API)，限制我们执行该操作的频率，并在我们得到最终结果前，设置中间状态。这些都是计算属性无法做到的。



## 和 computed 的异同点

相同： computed和watch都起到监听/依赖一个数据，并进行处理的作用

异同：它们其实都是vue对watcher的实现，只不过**computed主要用于对同步数据的处理，watch则主要用于观测某个值的变化去完成一段开销较大的复杂业务逻辑**。



能用computed的时候优先用computed，避免了多个数据影响其中某个数据时多次调用watch的尴尬情况。



## Watch 的高级用法

1. handler和 immediate 方法

在 watch 中下面这样的写法

```javascript
watch:{
	question: function (newvalue, oldvalue){
    	// do something
  }
}
```

 相当于

```javascript
watch: {
	question:{
			handler: function(newvalue, oldvalue){
					//do something
			},
      immediate: true, // 默认为 false
      deep: true // 默认为 false
	}
}
```

Immediate表示在初次渲染的时候并不会调用这个 watch 的方法，只有在值发生改变的时候才会进行调用。

2. deep 属性表示是否开启深度监听，如果不开启的话，那么只有给一个 object 赋值的时候可以触发，例如

```javascript
data(){
	return {
			obj:{
				a: '123'
			}
	}
}
// 当 deep 为 false 的时候
// obj.a = '456' 不能触发 watch
// obj = {a:'456'} 才可以
```

即，当 deep 为 false 的时候不能检测到一个 object 内部属性的变化，只有当整个 object 变化的时候才能检测到。这个时候如果加上 deep 属性那么，当给 obj.a 修改值得时候就可以检测到了。

当 deep 属性为 true 的时候，会在对象中一层一层向下遍历，利用源码中定义的 traverse方法

```javascript
/* @flow */

import { _Set as Set, isObject } from '../util/index'
import type { SimpleSet } from '../util/index'
import VNode from '../vdom/vnode'

const seenObjects = new Set()

/**
 * Recursively traverse an object to evoke all converted
 * getters, so that every nested property inside the object
 * is collected as a "deep" dependency.
 */
export function traverse (val: any) {
  _traverse(val, seenObjects)
  seenObjects.clear()
}

function _traverse (val: any, seen: SimpleSet) {
  let i, keys
  const isA = Array.isArray(val)
  if ((!isA && !isObject(val)) || Object.isFrozen(val) || val instanceof VNode) {
    return
  }
  if (val.__ob__) {
    const depId = val.__ob__.dep.id
    if (seen.has(depId)) {
      return
    }
    seen.add(depId)
  }
  if (isA) {
    i = val.length
    while (i--) _traverse(val[i], seen)
  } else {
    keys = Object.keys(val)
    i = keys.length
    while (i--) _traverse(val[keys[i]], seen)
  }
}


```

traverse方法递归每一个对象或者数组，触发它们的getter，使得对象或数组的每一个成员都被依赖收集，形成一个深的依赖关系，同时遍历过程中会把子响应式对象通过它们的 dep.id 记录到 seenObjects，避免以后重复访问。

deep 属性给每个层级都加上了 watcher，性能开销就会很大了。可以使用字符串属性来进行优化：

```javascript
watch: {
    'obj.a': {
      handler(val) {
       console.log('obj.a changed')
      },
      immediate: true
      // deep: true
    }
  }

```



参考:

1. [Vue.js的computed和watch是如何工作的？](https://juejin.im/post/6844903667884097543)