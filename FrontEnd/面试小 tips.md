1. Q: window.onload()和 document.ready()的区别：

   >  A: window.onload()要等页面上所有的元素都加载完才触发，包括图片，大的表单等等，而 document.ready 触发时间是dom 元素加载完之后就会触发

2. Q: transition+ transform 和 animation 的区别：

   > transition 更强调于转化和过渡，长被应用于 hover 属性上，而 animation 则是动画，结合 keyframe 可以定制中间每一帧的状态，transition 可以看成是只有两帧 的动画，而 animation 可以做的要强大的多。

3. Q: 异步请求在 vue 生命周期的哪个里面完成

   > A: created或者 mounted，created 的优势是早，但是如果需要 dom 操作类型的则需要在 mounted 中发起，如果只是数据更改类型的则在 created 中完成就可以

4. Q: 第一次加载页面会触发哪些钩子？

   >  A: beforeCreate， created， beforeMount， mounted

