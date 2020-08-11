# Canvas 和 Svg

## Canvas:

Canvas 是 H5新出来的标签，在 HTML 中写\<canvas\>标签，在 js 中获取画布，并使用 getContext('2d')这个绘制工具在画布中绘制图形。

canvas 是逐个像素进行渲染的，一旦绘制完成，浏览器就不会再关注。

## Svg(scalable vector graphics):

svg 是一种用XML 描述的2D 图形语言，它基于 XML，这意味着SVG Dom 中每个元素都是可用的。甚至都可以添加 js事件处理器，大小变化的时候，浏览器能够自动重现图形。

svg 是基于dom 操作的。所以不适合复杂度高的游戏应用。

```xml
<svg width="100%" height="100%"  >
        <circle cx="300" cy="60" r="50" stroke="#ff0" stroke-width="3" fill="red" />
</svg>
```



## 区别

* canvas 基于**位图**(位图由像素点组成，无限放大位图可以看到马赛克一样的像素点)，而 svg 基于**矢量图**(基于计算机指令来描述的一幅图，即线条或者形状，当放大的时候对应命令比例也会放大，所以不会出现像素点，因此矢量图和分辨率无关)。
* canvas用 js 绘制2D 图形，而  svg 基于 xml
* canvas 依赖于分辨率，而 svg 不依赖于分辨率
* canvas 适合高交互相应的游戏应用或者基于一些插件(Echarts)做图表类应用，而 svg 适用于静态图片，例如地图



## HTML 和 XML

HTML 和 XML 实际上是兄弟关系，都是SGML的特定子集。



## XML(ExtentsibleMarkup Language)

XML被设计用来描述数据，其焦点是数据的内容。

```xml
<note><to>Tove</to><from>Jani</from><heading>Reminder</heading><body>Don't forget me this weekend!</body></note>
```

可以看到 xml 只是用来被单纯的包含和传递信息，XML 中的标签没有预定义，想要使用必须自己去定义，并且对于这些特定形式的 XML，要自己按照自己的规则进行解析。

在用途上来说，XML 类似于 json，都是传递数据的一种格式。



## HTML(HyperTextMark-upLanguage)

HTML被设计用来显示数据，其焦点是数据的外观。

在html中不区分大小写，在xml中严格区分。

在HTML中，可以拥有不带值的属性名。在XML中，所有的属性都必须带有相应的值。