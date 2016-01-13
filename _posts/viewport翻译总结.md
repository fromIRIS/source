title: "viewport翻译总结"
date: 2015-12-22 11:20:57
categories: 翻译
tags: [viewport]
---


###viewport详解（翻译+总结）

来源：http://www.quirksmode.org/mobile/viewports.html
首先需要了解的是css像素并不等于设备像素，因为存在一些情况比如300px的div却占了1000x1000的屏幕，这两者的像素并不相等。下文会经常出现css像素设备像素等名词。
##pc端
**1、**`screen.width&screen.height`
这是显示器的设备像素大小，永远不变，并不是浏览器的，包含了整个屏幕。但并没有什么卵用。
**2、**`window.innerWidth`
浏览器的可视区域宽度，如果用户通过浏览器的zoom缩放，innerWidth也会随时变化。是通过css的像素进行测量的，而不是通过设备屏幕像素。
比如，300px宽的elem在zoom放大后占满了整个屏幕，但此时的innerWidth是300px；
但这个情况并不适合opera浏览器，且在移动端会有很严重的兼容性情况
**3、**`window.pageYOffset & window.pageXOffset`
获取浏览器滚动了多少像素，这个值也是根据css像素取得的，缩放时，浏览器会试图将当前视图固定，所以缩放时，pageYOffset并不会变化（或变化很小）。
**4、**`viewport`
默认情况下块级元素的宽度沾满了父级元素的100%，一直到html，而html的宽度占了浏览器宽度的100%
在理论上来讲，`<html>`的宽度被viewport的宽度限制。`<html>`的宽度占据了100%的viewport的宽度。而正巧，在pc端viewport的宽度就等于window的宽度。所以在pc端，viewport的宽度就等于`<html>`的宽度，也就等于浏览器的宽度.
同理，这里的浏览器宽度（window.innerWidth）是根据css像素度量的，所以也就出现了一个问题。
如果给一个div设置了100%的宽，这个具体的值是根据window的css像素算的，是body直接子辈。在zoom=100%的时候，div是整个浏览器的宽。但是如果zoom放大，此时window的宽度是根据css计算的，所以window宽度变小了，从而导致viewport的宽度值也变小了，从而导致div的宽度值也变小了，所以此时将页面朝右滚动，div会出现断层。
**html的宽度是由viewport的宽度限制的，可以把viewport看成一个节点，里面包裹着html**
**5、**`viewport的度量`
`document.documentElement.clientWidth`
这个属性只是显示了viewport的值，无视html的设置的宽度值。
**如此来说，`document.documentElement.clientWidth`跟`window.innerWidth`岂不是都能取得viewport的值？**
clientWidth不包含浏览器滚动条的宽度，而innerWidth却包含了滚动条的宽度。而且早期IE并不支持innerWidth。但这两者都涉及视口的宽度，跟文档宽度没多大关系。
**6、**`document.documentElement.offsetWidth`这个属性取得是html的宽度。在正常情况的`offsetWidth`跟`clientWidth`的值是一样的，但前者是html的宽度，后者是viewport的 宽度，前面还讲到了viewport是包裹html的关系，viewport是父元素。所以当给html设置宽度，比如400px，那这个时候两个值就将不同。
但`offsetHeight`跟`clientHeight`的值就不一样了，前者将是整个html内容的高度（有滚动条的情况包含看不见的内容的高度）
**7、**`事件event返回的坐标参数`
`pageX pageY`返回了相对于html的鼠标坐标，基于css像素。
`clientX clientY`返回了相对于viewport的鼠标坐标，基于css像素。
`screenX screenY`返回了相对于screen屏幕（不是浏览器）的鼠标坐标，基于设备像素。
我们一般90%的情况都会用到pageX， pageY。
**8、**`媒体查询`
媒体查询一般根据width进行判断。但官方有两种宽度模式可以选择。
`width & device-width`
这里的width指的的viewport宽度，也就是clientWidth，当然是基于css像素的，而device指的是设备宽度屏幕screen不是window，基于设备像素。
在pc端99.999%使用width。
##移动端
最重要的问题集中在css，尤其是度量viewport的尺寸。
html元素的宽取自layout viewport，那么不同机型浏览器的layout是不同的，ios980px，android800px。
最基础的是两个视角：
`visual viewport & layout viewport`
都是基于css度量。 
George Cummins在Stack Overflow上对基本概念给出了最佳解释：

把layout viewport想像成为一张不会变更大小或者形状的大图。现在想像你有一个小一些的框架，你通过它来看这张大图。（译者：可以理解为「管中窥豹」）这个小框架的周围被不透明的材料所环绕，这掩盖了你所有的视线，只留这张大图的一部分给你。你通过这个框架所能看到的大图的部分就是visual viewport。当你保持框架（缩小）来看整个图片的时候，你可以不用管大图，或者你可以靠近一些（放大）只看局部。你也可以改变框架的方向，但是大图（layout viewport）的大小和形状永远不会变。

也看一下Chris给出的解释。

visual viewport是页面当前显示在屏幕上的部分。用户可以通过滚动来改变他所看到的页面的部分，或者通过缩放来改变visual viewport的大小。
visual的值是不能大于layout的。
当缩放的时候，`visual viewport`会变化，因为视觉上能容纳的css像素在变化，但`layout viewport`保持不会。

**9、移动端viewport的度量**
在移动端，用`clientWidth`度量`layout viewport`的宽度，用`innerWidth`度量`visual viewport`的宽度。在不设置viewport标签的时候，ios的`layout viewport`都为980px，而且浏览器都会默认将`visual viewport`放大到跟`layout viewport`一样大，所以默认情况下手机屏幕能看到整个网页的内容（缩小版）。
很明显的当zoomin或zoomout时候，`visual viewport`会时刻改变，因为容纳的css像素是不同的。
**10、screen.width**
在pc端这个属性基本没用，但在移动端，这个值代表了手机的屏幕的设备像素大小。这个尺寸大小在chrome的模拟器里对各个机型都有标识。而且这个值跟ideal viewport是一样的。
**11、window.pageXOffset & window.pageYOffset**
visual 相对于 layout的位置，这个解释起来其实跟pc端的是一样的，因为这个滚动的值是根据css像素取度量的，不管怎么缩放，css像素值是不会变的。
**12、document.documentElement.offsetHeight**整个html元素的高度。同pc端的用法
**13、meta viewport**
```
<meta name="viewport" content="width=320">
```
这里的width的作用就是重置layout viewport的值。
`initial-scale`设置初始页面的缩放和layout viewport的宽度。
`device-width`这个值可以设置layout viewport，取得是idea viewport的值。
 `Android WebKit’s`的最小的layout viewport是320px，所以当设置viewport的width为320px以下时，浏览器将会自动设置width为ideal的宽度。

**14、ideal viewport**
有两点是ideal最重要的：
1、layout viewport 可以被设置成idael 的值，`width=divice-width & initial-scale=1`
这两者有兼容性问题，所以要设置成ideal需要将两者都写上。
2、最大最小缩放值都是根据ideal viewport的值确定的。 所以 maximum-scale=3 意味着最大的缩放值是 300% of the ideal viewport.。
**15、缩放ZOOM**
我们不能直接在js中取得缩放的值。
下面有两条计算公式
```
visual viewport width = ideal viewport width / zoom factor
zoom factor = ideal viewport width / visual viewport width
```
layout viewport不参与这个计算
**16、initial-scale**
设置这个命令实际上做了两件事：
1、设置了初始缩放值zoom，根据上面的公式，再计算出visual viewport的宽度。
2、根据算出来的visual值就等于layout viewport的值。
所以当在viewport里只设置了initial-scale，计算出来的visual跟layout是一样的。

**注意**initial-scale的设置在安卓原生浏览器上是没用的，在安卓原生上只能=1是有效的，并且不能设置width
,但这个情况也只存在部分安卓原生浏览器上，很奇怪。


翻译文章！