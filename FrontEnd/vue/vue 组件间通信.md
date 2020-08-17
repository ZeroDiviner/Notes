# Vue组件间通信



## 父子组件通信(prop/emit)

父组件

```javascript
// 父组件中
<template>
  <div class="section">
    <com-article :articles="articleList" @onEmitIndex="onEmitIndex"></com-article>
    <p>{{currentIndex}}</p>
  </div>
</template>

<script>
import comArticle from './test/article.vue'
export default {
  name: 'HelloWorld',
  components: { comArticle },
  data() {
    return {
      currentIndex: -1,
      articleList: ['红楼梦', '西游记', '三国演义']
    }
  },
  methods: {
    onEmitIndex(idx) {
      this.currentIndex = idx
    }
  }
}
</script>

```

子组件

```javascript
<template>
  <div>
    <div v-for="(item, index) in articles" :key="index" @click="emitIndex(index)">{{item}}</div>
  </div>
</template>

<script>
export default {
  props: ['articles'],
  methods: {
    emitIndex(index) {
      this.$emit('onEmitIndex', index)// 这里触发的事件名称要和上面@后面的一致。这里表示触发 onEmitIndex 事件，父组件监听 onEmitIndex 事件
    }
  }
}
</script>

```



## 父子组件通信(\$parent/$children)

通过\$parent 和 ​\$Children 可以访问父组件 和子组件的实例，也就是，可以直接拿到 data 属性。

这只是一种应急的方法，推荐使用 props/emit 来传递值

```javascript
// 父组件中
<template>
  <div class="hello_world">
    <div>{{msg}}</div>
    <com-a></com-a>
    <button @click="changeA">点击改变子组件值</button>
  </div>
</template>

<script>
import ComA from './test/comA.vue'
export default {
  name: 'HelloWorld',
  components: { ComA },
  data() {
    return {
      msg: 'Welcome'
    }
  },

  methods: {
    changeA() {
      // 获取到子组件A
      this.$children[0].messageA = 'this is new value'
    }
  }
}
</script>

```

子组件

```javascript
<template>
  <div class="com_a">
    <span>{{messageA}}</span>
    <p>获取父组件的值为:  {{parentVal}}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      messageA: 'this is old'
    }
  },
  computed:{
    parentVal(){
      return this.$parent.msg;
    }
  }
}
</script>
```





## 父子组件通信(provide/inject)

下面关系中 A 是 B 的父组件，B 是 C 的父组件

当有很深的嵌套关系的时候可以使用 provide 和 inject, 这样不管多深的嵌套，都能拿到值。

```javascript
// A.vue

<template>
  <div>
	<comB></comB>
  </div>
</template>

<script>
  import comB from '../components/test/comB.vue'
  export default {
    name: "A",
    provide: { 	// 在父亲组件中 provide 一个值，在子组件中用 inject 接收
      for: "demo"
    },
    components:{
      comB
    }
  }
</script>
```

```javascript
// B.vue

<template>
  <div>
    {{demo}}
    <comC></comC>
  </div>
</template>

<script>
  import comC from '../components/test/comC.vue'
  export default {
    name: "B",
    inject: ['for'], // 在这个位置用 inject 来接收 for 属性
    data() {
      return {
        demo: this.for
      }
    },
    components: {
      comC
    }
  }
</script>

```

```javascript
// C.vue (B 的子组件)
<template>
  <div>
    {{demo}}
  </div>
</template>

<script>
  export default {
    name: "C",
    inject: ['for'], // 在这个位置用 inject 来接收 for 属性，这样
    data() {
      return {
        demo: this.for
      }
    }
  }
</script>

```

## 兄弟组件通信(eventBus)

初始化一个 eventBus，顾名思义，这个 eventBus 就是连通两个组件的一个信息通道。

```javascript
// event-bus.js
import Vue from 'vue'
export const EventBus = new Vue()
```

假设有两个兄弟组件，在父组件中的调用为:

```javascript
<template>
  <div>
    <show-num-com></show-num-com>
    <addition-num-com></addition-num-com>
  </div>
</template>

<script>
import showNumCom from './showNum.vue'
import additionNumCom from './additionNum.vue'
export default {
  components: { showNumCom, additionNumCom }
}
</script>
```

在其中一个组件中发送事件, 两个组件都会 import 之前定义的那个 eventBus, 一个组件在这个 eventBus 上 \$emit事件以发送信息，另一个组件在这个 eventBus 上$on 监听事件以接收信息。

```javascript
<template>
  <div>
    <button @click="additionHandle">+加法器</button>    
  </div>
</template>

<script>
import {EventBus} from './event-bus.js'
console.log(EventBus)
export default {
  data(){
    return{
      num:1
    }
  },

  methods:{
    additionHandle(){// emit参数接收两个参数，一个是事件名称，第二个是传递的参数
      EventBus.$emit('addition', {
        num:this.num++
      })
    }
  }
}
</script>
```

在另一个组件中接收事件

```javascript
// showNum.vue 中接收事件

<template>
  <div>计算和: {{count}}</div>
</template>

<script>
import { EventBus } from './event-bus.js'
export default {
  data() {
    return {
      count: 0
    }
  },

  mounted() { // $on 同时接收两个参数，第一个也是事件的名称，另一个是一个回调函数，回调函数的参数为传递过来的信息。
    EventBus.$on('addition', param => {
      this.count = this.count + param.num;
    })
  }
}
</script>
```



参考:

1. [vue中8种组件通信方式, 值得收藏!](https://juejin.im/post/6844903887162310669)