# Css 面试题总结

1. 介绍下 css 盒子模型，和怪异盒模型

   > CSS 盒模型从内到外依次是 content, padding, border, margin, 其中 width 和 height 指定的是 content 的宽高
   >
   > 怪异盒子模型 的 width 和 height 指定的是 content+padding+border

2. box-sizing 属性

   > box-sizing 属性可选的为 content-box 和 border-box
   >
   > content-box 表示使用标准盒模 模型， width 只指定 content 的宽
   >
   > padding-box 也是合法属性，表示 width 表示 content + padding
   >
   > border-box 表示使用怪异盒模型， width 指定 content + padding + border 的宽

3. css 选择器有哪些

   > 从优先级最高到最低有, 和对应的优先级算法权值
   >
   > * !important  	权重最高
   > * 内联样式          1000
   > * id 选择器          100
   > * class 选择器     10
   > * 伪类选择器a:hover  （伪类选择器和类选择器权重相同）
   > * 标签选择器        1 (伪元素选择器和标签选择器权重相同)
   > * 相邻选择器h1+p 
   > * 子选择器 ul >p
   > * 后代选择器 li a

4. 哪些属性可以继承，哪些不可以

   > 可以继承的：font-size, font-family, color
   >
   > 不可继承: width, height, border, padding, margin

5. CSS伪类和伪元素

   > Css 伪类使用单个冒号：，伪元素使用双冒号::
   >
   > Css 伪类是辅助 css 选择器，给某些元素的特定属性时附加样式
   >
   > Css 伪元素是给指定元素添加某个元素，例如在元素前或者元素后添加
   >
   > 伪类有： :hover, :visited, enabled, disabled, :first-child
   >
   > css3新增伪类： :first-of-type, n-th-child(2), :only-child 
   >
   > 伪元素有: ::before, ::after, ::first-line, ::first-letter

6. Display 有哪些值并解释

   > inline 内联样式，默认样式
   >
   > none 隐藏
   >
   > block 块级元素
   >
   > table  表格
   >
   > list-item 列表项目
   >
   > inline-block 行内 block

7. position 的值

   > static: 默认文档流位置
   >
   > relative: 根据自己在文档流的位置相对定位
   >
   > absolute: 根据距离其最近的一个父级不为 static 的元素定位，脱离文档流
   >
   > fixed： 根据浏览器视口定位

8. css3新属性有哪些

   > RGBA, 透明度
   >
   > Background-image
   >
   > background-origin(content-box, padding-box, border-box) 
   >
   > background-size 
   >
   > background repeat
   >
   > Word-wrap: break-word
   >
   > Text-shadow 和 box-shadow
   >
   > Border-radius 和 border-image
   >
   > @ media screen

9. Css 中 visibility 的collapse 属性，在不同浏览器中有什么区别

   > 当一个元素的 visibility 被设置为 collapse 的时候，对于一般元素来说和 hidden 一样
   >
   > 在 chrome 中， collapse 和 hidden 没区别
   >
   > 在 IE, firefox, opera 中，和 display: none 没区别

10. Visibility: hidden 和 display: none 的区别

    > display: none 表示隐藏元素，并且在文档流中不分配空间
    >
    > visibility: hidden 表示隐藏元素，但是在文档流中保留原来的空间

11. Position 和display ，float 叠加会怎么样

    > position 决定了元素的定位方式，float 定义元素向哪个方向浮动， display 决定了元素应该生成的框的类型。
    >
    > 优先级别， position: absolute/fixed 是最高的，有这种脱离文档流的属性的时候，float 是不起作用的。float 元素自动变更为 display: block

12. 浮动是什么，什么时候需要清除浮动，清除浮动的方式

    > 浮动元素碰到包含它的边框或者其他浮动元素的时候停止，浮动元素不在文档流中，所以包含浮动元素的框感觉像浮动元素不存在一样，浮动在块框之上。
    >
    > 浮动可能带来的问题：
    >
    > 	1. 父级元素无法撑开
    >  	2. 和浮动元素同级的元素会紧随其后
    >  	3. 如果不是第一个元素浮动，那么它之前的元素也都需要设置浮动
    >
    > 清除浮动的方法：
    >
    > 1. 父级元素设置高度
    > 2. Overflow: hidden或者 auto， 触发一个 BFC
    > 3. 最后添加一个空 div，设置 clear:both
    > 4. 伪元素::after 设置 clear: both，原理同上一个
    > 5. 父级元素设置 zoom

13. Margin塌陷，以及怎么解决

    > margin 塌陷出现在BFC 中，同属于一个 BFC 的两个 box 在垂直方向会出现 margin 重叠的情况。
    >
    > 解决方法: 在其中一个外面添加一个 div，并触发使其作为另一个 BFC

14. 媒体查询

    > @media only and screen (max-device-width: 200px){
    >
    > ​	/* */
    >
    > }

15. css优化的方式

    > 1. 避免链式选择符
    > 2. 避免不必要命名空间
    > 3. 避免重复
    > 4. 最好使用语义化的名字和类名，例如一个 class 最好描述它是什么
    > 5. 避免 ！important

16. 浏览器如何解析 css选择器

    > **从右边向左边解析**，先找到最右边的节点，对于每个节点，向上寻找其父节点直到根节点或者满足条件的匹配规则，就结束这个分支的遍历，大量不符合规则的样式在第一部就可以被筛掉。
    >
    > 如果是从左向右解析，则可能到达最后一步发现匹配失败，造成浪费大量性能。
    >
    > 在 CSS 解析完成后形成 CSSOM 树，与 DOM 树一起组成渲染树。

17. 网页中的字体应该使用奇数还是偶数

    > 这部分是自己的理解： 偶数字体，因为默认行高为字号* 1.5如果是奇数的话会出现行高不是整数的情况，如果是偶数例如14，则是14 * 1.5 = 21

18. 元素竖向的百分比是相对于容器的高度吗

    > 当百分比设定宽度的时候，是相对于父容器的高度设定的，但是对于一些竖直方向距离的属性，例如 padding-top, padding-bottom, margin-top, margin-bottom，设定百分比的时候，**依据的依然是容器的宽度**。

19. 响应式原理

    > 设计一个网页，但是能够兼容多个端，移动/pc, 而不是为每个终端做一个特定的版本。通过媒体查询来实现，页面的头部必须有
    >
    > ```javascript
    > <meta name = 'viewport' content = "width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
    > ```

20. 如何让 chrome 支持小于12px 的字体

    > ```css
    > p{
    > 	font-size:10px;
    >   transform: scale(0.8)//0.8是缩放比例
    > }
    > ```

21. li 和 li 之间有看不见的空白是什么原因引起的，怎么解决

    > 行框的排列会收到中间的空白(回车空格)影响，解决方法可以:
    >
    > 1. 把<li>都写在同一行
    > 2. li 设置float: left
    > 3. url 设置 font-size:0

22. 怎么理解 margin 负值

    > margin 表示一个元素向外扩张的区域，当 margin 为负数的时候表示其他元素可以侵占本元素的距离。

23. display: inline-block什么时候会显示间隙

    > 1. 有空格的时候，移除空格
    > 2. margin 正值得时候，可以使用负值 margin
    > 3. 有时候可以使用 font-size:0 解决

24. png, jpg, gif, webp

    > Png: portable network graphics, 一种无损压缩格式，压缩比高，色彩好
    >
    > jpg: 一种失真压缩方法，破坏性的压缩
    >
    > Gif: 一种位图文件格式，8位色彩可以实现动画效果
    >
    > Webp: 压缩率只有jpg 的2/3, 大小比 png 小45%， 压缩时间需要更久，兼容性不太好

25. style 写在 body 前和 body 后的区别

    > 由于 html 是逐行加载解析，如果写在 body 前就是先加载样式。
    >
    > 如果写在 body 后，则解析到尾部的 css 样式的时候，会导致浏览器停止之前的渲染，等待加载完毕后重新渲染

26. css的 overflow 属性的各种处理

    > scroll：必然会出现滚动条
    >
    > auto: 当子元素大于父元素的时候出现滚动条
    >
    > visible: 超出父元素页显示
    >
    > Hidden：超出部分截断

27. 文本超出显示省略号的 css

    > overflow: hidden
    >
    > text-overflow: ellipsis
    >
    > white-space:no-wrap

28. 块级元素和行内元素

    > 块级元素总是在新的一行开始，高度，宽度，边距都可以控制
    >
    > 行内元素和其他元素同在一行内，宽高不可变，宽度就是其文字或者图片的宽度，只能容纳文本或者其他行内元素(margin 和 padding 只有左右有效，上下无效)
    
29. 块级元素和行内元素举例：

    > 块级：form, div, header, footer, nav, ul, ol，p, h
    >
    > 行内: span, b, em, i, li, input, textarea img, a