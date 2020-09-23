1. Q: window.onload()和 document.ready()的区别：

   >  A: window.onload()要等页面上所有的元素都加载完才触发，包括图片，大的表单等等，而 document.ready 触发时间是dom 元素加载完之后就会触发

2. Q: transition+ transform 和 animation 的区别：

   > transition 更强调于转化和过渡，长被应用于 hover 属性上，而 animation 则是动画，结合 keyframe 可以定制中间每一帧的状态，transition 可以看成是只有两帧 的动画，而 animation 可以做的要强大的多。

3. Q: 异步请求在 vue 生命周期的哪个里面完成

   > A: created或者 mounted，created 的优势是早，但是如果需要 dom 操作类型的则需要在 mounted 中发起，如果只是数据更改类型的则在 created 中完成就可以

4. Q: 第一次加载页面会触发哪些钩子？

   >  A: beforeCreate， created， beforeMount， mounted

5. Q: vue 中 v-html 会导致什么问题

   > V-html更新的是元素的 innerHTML 。内容按普通 HTML 插入，相当于直接操作了dom 不会作为 Vue 模板进行编译，因此可能会导致 xss 攻击， 这部分 html 没有经过 vue 编译器模板编译，因此单组件中的 scoped 样式不会被应用。

6. Q：vue-router 匹配优先级(有时候同一个路径可以匹配多个路由)

   > 谁先定义的就匹配谁，谁最先定义，则优先级最高。

7. Q: Seo 是什么

   > A: Search engine optimisation搜索引擎优化

8. Q: window.requestAnimationFrame 和 setInterval 16的区别

   > A: 浏览器刷新频率为1秒60帧，级16.67ms会刷新一次，16的话并不是严丝合缝的。
   >
   > 1. 并且 requestAnimationFrame会把每一帧的dom 操作集中起来去集中回流重绘，并且回流重绘事件紧跟刷新时间。
   > 2. setInterval 是异步的，如果有其他线程干扰可能会导致丢帧
   > 3. 对于隐藏或者不可见元素，requestAnimationFrame 不会回流重绘，减少 cpu，gpu 使用量
   >
   > 使用 requestAnimationFrame ： let id = window.requestAnimationFrame(animation), 其中 animation 是定义好的动画函数 
   >
   > 停止这个动画: cancelAnimationFrame(id) // id 就是上面定义的

9. CSS 动画和 js 动画的区别

   > css 动画用的是类似 requestAnimationFrame 的方法，集中 dom 操作，使得回流重绘频率紧跟刷新频率，且强制使用 GPU 加速，代码相对简单，但是对于动画控制不如 js 强。兼容性不如 js 动画
   >
   > js 动画功能强大，能够实现各种操作，但是因为是异步关系，如果有干扰线程，那么可能会导致丢帧。并且 js 动画大多兼容性没有问题，并且 js 动画比 css 动画多了一个解析的过程，性能不如 css 动画好。

10. js的 map 函数接收几个参数

    > map 函数接收一个参数，为一个 function，表示对于每个数组中的元素如何操作，类似 python 中的 lambda，这个函数接收3个参数，分别是 : 当前元素，当前元素下标，整个数组。

11. Promise 如何在外部 resolve

    > ```javascript
    > var Obj = {
    > ​	success:null,
    > ​	error: null
    > } 
    > 
    > new Promise((resolve, reject)=>{
    > ​	Obj.success = resolve;
    > ​	Obj.error =  reject
    > }).then((data)=>{
    > ​	console.log(`data == ${data}`)
    > })
    > Obj.success('Helloworld')
    > // data == Helloworld
    > ```

12. [] == ![] 为 true, 而 {} == !{} 为 false

    > 首先，！的优先级要高于 == ，因此先计算的是右边，因此右边相当于是 false
    >
    > 计算右边完成之后，对于以 Boolean 的变量和一个引用类型变量，二者比较会遵循一个原则：
    >
    > 先调用引用对象的 valueOf 方法，如果不可以的话，就调用 引用对象的 toString()方法
    >
    > 而 []调用 toString()方法结果为**""**, 因此相当于 false, 所以[] == ![]
    >
    > 但是{} 调用 toString()方法结果为"[object Object]", 因此相当于是 true == false, 所以{} == !{}为 false

13. 上传的图片如何可视化

    > 可以转换为 base64格式进行可视化，img的 src 可以接受 base64编码，canvas 下有一个 API 叫做canvas.toDataURL()， 用于把图片转换为 base64，转化后的图片是8位的位图图片。
    
14. 浏览器关闭前执行事件怎么完成

    > 有一个事件叫做 onBeforeUnload，所绑定的事件可以在关闭之前执行

15. head 中有哪些标签

    > \<script\>, \<style\>, \<link\>, \<meta\>, \<noscript\>, \<title\>

16. \<link\>标签的作用

    > rel="stylesheet" 表示 css样式表
    >
    > rel="canonical" href="..." 搜索引擎
    >
    > rel = 'prefetch' 浏览器预先拉取资源
    >
    > rel = 'dns-prefetch'预先拉取 dns 资源
    >
    > rel = 'icon'表示页面的 icon

17. \<meta\>标签的作用

    > http-equiv = 'key-words' content = 'key1, key2' 用于浏览器 seo 关键字部分
    >
    > http-equiv = 'description' content = '描述' 用于浏览器 seo 网页描述
    >
    > http-equiv="Refresh" content="2；URL=http://" 自动刷新并重定向
    >
    > http-equiv="cache-control" content="no-cache, must-revalidate" 删除缓存，再访问需要重新下载。
    >
    > name="viewport" content="width=device-width, initial-scale=1, user-scalable=no, minimal-ui" 初始化设备尺寸

