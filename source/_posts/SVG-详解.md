---
title: SVG 详解
date: 2020-09-20 23:22:13
tags:
  - SVG
categories:
  - HTML
---

## 什么是SVG

SVG 是 Scalable Vector Graphics 的缩写。字面意思就是可伸缩矢量图形。他相对于传统的格式的图片（如png，jpg等）最大的优势就是他是可以**无限缩放**的。无论放大多少倍，他的图片都不会出现模糊的问题。而且他的兼容性也很好。除了 IE8 及以下，其他浏览器都对其提供了良好的支持。

## SVG 语法

SVG 使用 XML 来定义它的图像。常用的元素有：

- text: 创建一个文字元素
- circle: 创建一个圆形
- ellipse： 创建一个椭圆
- rect: 创建一个矩形
- line: 创建一条直线
- path: 创建一个路径
- textPath: 创建一个文本路径
- polygon: 创建一个多边形
- g: 创建一个元素组

### text

属性x，y代表文本绘制的起始点

```
<text x="10" y="10">I am a text</text>
```

### circle

属性如下

- cx: 圆心x坐标
- cy：圆形y坐标
- r：圆半径
- fill：圆填充色

```
<circle cx="20" cy="20" r="10" fill="#e0e0e0" />
```

### ellipse

属性如下

- cx： 椭圆圆心x坐标
- cy： 椭圆圆心y坐标
- rx： 椭圆横向半径
- ry： 椭圆纵向半径

### rect

属性如下

- x： 矩形左上角x坐标
- y： 矩形左上角y坐标
- width： 矩形横向长度
- height：矩形纵向长度

### line

属性如下

- x1: 直线起点x坐标
- y1: 直线起点y坐标
- x2: 直线终点x坐标
- y2: 直线终点y坐标

### path

有一个属性d，在这个属性中使用一些命令来绘制图形，命令如下

- M： 代表 moveto，接受一组x，y坐标，把绘制点移动到这个位置
- L： 代表 lineto，接受一组x，y坐标，从上一点开始到这个参数的点绘制一条直线
- H： 代表 horizontal lineto，接受一个x坐标，类似L，但是是绘制一条纵向的直线
- V： 代表 vertical lineto，接受一个y坐标，类似L，但是是绘制一条横向的直线
- C： 代表 curveto，接受三组x，y坐标，绘制一个三次贝塞尔曲线
- S： 代表 smooth curveto，接受两组x，y坐标，一个点某一侧的控制点是它另一侧的控制点的对称（以保持斜率不变），是简写的C命令
- Q： 代表 quadratic Bézier curve，接受两组x，y坐标，绘制一条二次贝塞尔曲线
- T： 代表 smooth quadratic Bézier curveto，接受一组x，y坐标，类似S命令，是Q命令的简写，可以只写终点自动推测中间的控制点。但是之前必须有Q命令或者T命令，否则将会成为一条直线
- A： 代表 elliptical Arc，A命令有许多参数，是用来生成一段曲线的
- Z： 代表 closepath，不接受参数，代表将终点和起点相连，闭合路径

```
<path d="M 10 10 L 20 20 H 30 V 40 Z" />
```

### textPath

接受的参数和path相同。然后文字将在这段路径上显示

```
<textPath d="M 10 10 L 20 20 H 30 V 40 Z">i am a text path</textPath>
```

### polygon

有一个属性 points，接受多个x，y坐标组

```
<polygon points="10,10 20,20 35,20" />
```

### g

将多个元素分组

```
<g id="group">
  <path d="M10 10 L20 20" />
  <rect x="20" y="20" width="10" height="10">
</g>
```

## SVG 常用属性

- fill: SVG 的填充颜色。在改变svg颜色的时候不能用color，而是要用fill。当然，你可以将fill属性设为currentColor，那么 SVG 将使用 color 属性的颜色进行渲染
- fill-opacity： 填充颜色的不透明度
- stroke: SVG 线条的颜色
- stroke-width: SVG 线条的宽度

## 使用JS，CSS操作SVG

当 SVG 使用内联的方式加载时（不是通过img等标签通过外部引入），我们可以通过JS或者CSS来操作SVG。SVG 中的每一个标签和正常的DOM标签一样，都拥有class，id等属性。可以通过设置CSS来改变样式，同时，他们也可以被类似于`document.querySelector` 等操作DOM的API来进行改变。通过这样，我们可以实现一些很炫酷的SVG动画等等功能。

## viewport 和 viewBox

### viewport

是指 SVG 可见区域，也就是 SVG 实际渲染的大小

```
<svg width="200" height="200"></svg>
```

这里定义了 SVG 长宽为 200px，这个200px的大小就是 SVG 的 viewport

### viewBox

viewBox 是指渲染的画布的区域范围

viewBox 接受4个参数：x, y, width, height 定义了一个矩形。举一个例子

```
<svg viewBox="0,0,50,50">
  <rect x="10" y="10" width="10" height="10">
</svg>
```

这个 SVG 渲染出来的结果是什么呢？我们假设这个 SVG 设置长宽为100。那么展示的矩形起点在20，20处，长宽为20,20。

也就是说，我们想象 SVG 有一个无限大的画布，里面的元素可以在这个任意大的画布中进行作画。但是我这个 SVG 向外界展示的区域是一个有限的区域，我们对 SVG 缩放的范围，渲染的范围就是这个区域。这个区域的名字就是 viewBox。

### preserveAspectRatio

但是我们设置的 SVG 的长宽比和 viewBox 的长宽比并不一定是一样的。这时候就需要 preserveAspectRatio 属性来出马了。他由两个值组成，第一个值表示 viewBox 和 viewport 的对其方式，第二个值表示长宽比如何维持。

第一个值有10种

- 一种是none，这时候 SVG 的长宽比将失真，强制使 viewBox 长宽适应 viewport 的长宽
- 另外的9种，他们分别定义了x，y方向上，以起点，终点和中点三个方向进行对其，3*3 共 9 种

第二个值：

- meet： 保持 viewBox 纵横比缩放， viewBox 尽可能放大占满 viewport。也就是说，整个整个 viewBox 区域都是可见的，小于 viewport 的大小
- slice： 保持 viewBox 纵横比缩放，同时比例小的方向放大填满 viewport。也就是说，部分 viewBox 区域可能会超出 viewport 的区域变得不可见
