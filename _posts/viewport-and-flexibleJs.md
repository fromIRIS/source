title: "viewport-and-flexibleJs"
date: 2016-01-28 17:56:39
tags: [viewport]
---

## viewport探索 flexible.js解读
http://www.quirksmode.org/mobile/viewports.html
http://www.quirksmode.org/mobile/viewports2.html
http://www.quirksmode.org/mobile/metaviewport/#t10
http://www.w3cplus.com/mobile/lib-flexible-for-html5-layout.html
> 这在这篇文章介绍了viewport的三种视口、以及等过此三视口分析了淘宝的flexible.js方案的实现原理。

现代浏览器中实现缩放的方式，无怪乎都是「拉伸」像素。

在屏幕上，首先介绍的两个概念。「css像素」、「设备像素」

css像素与写在样式表中定义的宽的度量单位是一致的。

可以这么理解，css像素和设备像素 是容纳的关系。在缩放倍数是200%的情况下，1个css像素容纳了4个设备像素。。
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453953104575.png) ![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453953119375.png) ![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453953129374.png)



缩放的作用就是改变一个css像素可以容纳设备像素的多少。

### PC端
在正常情况下，一css像素等于一设备像素。放大到200%的情况下，一个css像素等于四个设备像素。（宽2倍 高2倍）

#### window.innerWidth
屏幕、页面有很多属性。
- screen.width
- window.innerWidth
- document.documentElement.clientWidth
- document.documentElement.offsetwidth
- 等

而这些属性的值的单位就是像素pixels。区别就是其中一些属性值的度量单位是「设备pixels」而大部分是「css pixels」

window.innerWidth度量的是浏览器窗口的宽度。度量单位是css pixels。
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453617924515.png)
window.innerWidth的例子。

![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453618966971.png)

这是缩放100%的情况。header的宽1220px ，几乎沾满了浏览器屏幕宽度，window.innerWidth的值为1231px，根据这个现状很容易证明window.innerWidth的度量单位是css pixels。

现在将这个页面放大到200%。
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453620161800.png)

我们发现现在的window.innerWidth变为了刚才的一半。就是因为这个属性的度量单位是css 像素（包含多少css像素），衡量css像素的直观大小，跟页面中的dom的css大小比较就ok。

看我们的页面，header还是1220px没有变。但是在浏览器中展示出来的却只有刚才的一半。由直观上的判断，目前浏览器窗口范围内的css宽的值为之前的一半。而这个值 就是当前window.innerWidth的值的大小。

一个dom元素的css在数值上大小不变，而设备的像素大小是物理值，也不变，所以也符合之前提到的。缩放实际改变的是一个css像素容纳的设备像素的多少。


viewport，用来约束网站中最顶级包含块元素`<html>`。

默认「body」的宽度取自「HTML」，而「HTML」的宽度取自「viewport」，「viewport」的宽度正好等于「浏览器」窗口的宽度。

可以把「viewport」当成比「html」更高一层的元素，不管我们有没有给「html」设置宽，`document.documentElement.clientWidth`取到的都是「viewport」的宽。而`window.innerWidth`能取到「浏览器」窗口的宽。在PC端这两者的区别仅仅区分在`window.innerWidth`包含滚动条的宽。
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453959085296.png)

这个定义有一个问题出现，红色背景设为100%宽，内元素设置min-width980px，当屏幕缩小到980以下时，会出现横向滚动条，但把滚动条右移就会发现背景缺失了。这就是因为100%的宽其实最大就是浏览器窗口的宽。当浏览器窗口小于980px时，红色背景也会缩小到小于980px，所以右划会出现空白。
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-%E8%83%8C%E6%99%AF%E7%BC%BA%E5%A4%B1.gif)

#### offsetWidth
`document.documentElement.offsetWidth`这个属性取得是html的宽度。（IE中度量的是viewport）

如果将html当做一个块级元素来讲，html的宽默认为父元素的100%,高度默认被子元素撑开。
所以默认情况下，offsetWidth取得值是继承于viewport的宽，而viewport的宽等于浏览器窗口的宽。
而offsetHeight的值默认由子元素的高决定。
显式书写了html的height或者width的时候，offsetHeight/width取的就是显式设置的值。

还有个情况就是设置html为100%的时候，实际上这个意思就是将html的高设置为viewport的高的100%，同理，viewport的高等于浏览器窗口的高。可以用window.innerHeight取得。

![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453627362062.png)

#### pc端小结
1. `window.innerWidth`的度量单位是「css像素」，表象的概念就是当前浏览器窗口包含了多少css像素大小的dom。
2.  默认「body」的宽度取自「HTML」，而「HTML」的宽度取自「viewport」，「viewport」的宽度正好等于「浏览器」窗口的宽度。
### 移动端
- layout viewport
- visual viewport

#### layout viewport和visual viewport
layout viewport的概念其实跟pc端的viewport是一样的，**是作为html的”上层“元素**。将宽继承给html。html内的各元素都是以layout viewport为基准进行布局的。但跟pc端不同的是，pc端的viewport的宽是由浏览器的窗口的宽决定的，用户可以手动拖动窗口改变宽的大小。但是移动端的不同平台的浏览器呈现不同的layout viewport。

ios980px，android800px。

将visual viewport想象成覆盖手机屏幕的一个框，这个框带有类似pc端缩放的功能，而且这个框的度量单位也是css像素。这就意味着，在layout viewport不变的情况，我们能看到多少css像素的东西，取决于这个框的缩放程度。默认情况下。大多数移动端浏览器会将visual viewport这个框缩放到与layout viewport相同。

拿ios设备举例，layout viewport固定为980px，默认打开页面的情况下，visual viewport会将这个框缩放到980px。这样我们就能看到全部的内容了。

![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453640203641.png) ![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453909523104.png)

默认情况下html元素的宽取自layout viewport，那么不同机型浏览器的layout是不同的，ios980px，android800px。

所以在不设置任何条件的情况下，一个宽度是1220px的div在ios模拟器下是这样的。

![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453633924318.png)

html的宽继承自layout viewport，980px。多出的240px在layout viewport之外，需要通过滑动屏幕才能见到。


在pc端我们通过`document.documentElement.clientWidth`取得viewport的宽度。

但是移动端有layout和visual两个viewport，该怎么取值?

之前讲到了layout viewport的概念跟pc端的viewport相近，都是来约束html元素的宽的。即html元素是以layout viewport作为参考系进行布局的。所以这里的`document.documentElement.clientWidth`能获取到layout viewport的宽度尺寸。

在pc端是通过`window.innerWidth`来度量浏览器窗口的宽的，而在移动端，之前讲到的visual viewport模拟的那个框，就相当于浏览器的窗口。所以在移动端可以通过`window.innerWidth`来取得visual viewport的宽。其值会根据缩放的程度而改变。读到的值为当前屏幕上x方向的css像素的值。

在pc端我们总结过`window.innerWidth`和`document.documentElement.clientWidth`在缩放不同的情况下只是相差一个滚动条的宽度。但是在移动端，`window.innerWidth（visual viewport）`和`document.documentElement.clientWidth（layout viewport）`在缩放的情况下值是不同的。原因在于移动端的`document.documentElement.clientWidth（layout viewport）`总是固定的。

#### viewport meta标签
```
width：控制 layout viewport 的大小。
height：和 width 相对应，指定高度。
initial-scale：初始缩放比例，也即是当页面第一次 load 的时候缩放比例。
maximum-scale：允许用户缩放到的最大比例。
minimum-scale：允许用户缩放到的最小比例。
user-scalable：用户是否可以手动缩放
```

width 设置layout viewport的宽度

为什么要设置这个属性？

![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-%E6%94%BE%E5%A4%A7.gif)

我们之前提到过在原始的页面上visual viewport会自动将视口缩放到与layout viewport同宽，这样就能看到全部的内容，用户自然会放大页面，但是遇到较长的文字段落需要将屏幕左右滑动才能阅读完全。
在没有viewport meta标签之前可以这样优化。

![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-%E6%94%BE%E5%A4%A7%E4%BC%98%E5%8C%96.gif)

将html的宽度设置为375px,这样文字就能在一个页面内显示完全，放大的时候不需要左右滑动了。

但是在初始化的时候内容太小，不易于阅读。

于是苹果为了解决这个问题提出了viewport meta。

目的之一就是可以手动的设置layout viewport的值。

既然在移动端用不了这么大的像素宽度，那干脆就把layout viewport的值减小，visual viewport也会自动缩放在屏幕上显示所有的layout viewport的内容。

但是我们之前也讲到了layout viewport的宽是限制HTML元素的宽度的。所以html元素内的元素的宽需要设置不大于layout viewport的宽度值，才能保证完全显示在visual viewport内。

![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453910179713.png)
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453910238490.png)
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453910323569.png)

此时对于viewport meta标签里只设置了width=300字段，依靠缩放的现象，visual viewport 和 layout viewport的宽度一致


根据这个现象我们只要根据设计稿中规划好的宽度，进行viewport width的宽度相同的设置就可以了。依靠缩放原理就可以在全部机型上呈现一样的宽度。（高度是根据宽度自适应的）

但是目前的viewport只设置了width，现在的页面用户还是可以进行手动缩放的，为了禁止手动缩放，需要增加一个字段
```
<meta name="viewport" content="width=375" user-scalable=no>
```
但是这个`user-scalable=no`字段给部分安卓原生浏览器及部分安卓webview的自动缩放功能带来了限制。
本例中设置的layout viewport width为375px。

![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-%E5%AE%89%E5%8D%93%E5%B7%A6%E5%8F%B3%E6%BB%91%E5%8A%A8.gif)

本来应该在屏幕上正好填充内容的，现在出现了左右滚动条。
这个现象的原因就是layout viewport的宽度由viewport meta的width值设置成了375px，但是visual viewport的值只有360px，所以屏幕上只能显示360px的内容，剩下的需要左右滑动。

至于这里为什么是360px，而不是其他的值。
来看看ideal viewport的概念。

#### ideal viewport 
最开始提到了visual viewport 和 layout viewport的概念。这两个值都是跟页面实际的大小有关的，但是这个ideal viewport，用最直观的话来讲，就是每一种机型所对应的屏幕尺寸，当然也是css像素来度量的。

![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453861548105.png)
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453861770598.png)

拿Note2举例，就是刚刚gif里展示的那台机子，ideal viewport的值就是360px，而这个值与缩放值和visual viewport是息息相关的。是作为一个基准值。  （「ideal viewport」「缩放值」「visual viewport」三者息息相关）

```
visual viewport width = ideal viewport width / zoom factor
```
这个zoom factor缩放值之前也讲到过，在大部分情况下是自动计算的。拿note2举例，layout viewport的宽设置了375px（没有设置user-scalable=no），当前的ideal viewport的宽为360px。
```
<meta name="viewport" content="width=375">
```
页面自动缩放visual viewport会计算得为375。所以此时在屏幕的visual viewport能看到所有的layout viewport的宽度的内容。

![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453865034011.png)


一旦设置了
```
<meta name="viewport" content="width=375, user-scalable=no">
```
之后，在部分安卓原生浏览器以及安卓webview上，缩放值就不会自动计算了，而是取固定值1。
根据上述的计算公式，
visual viewport width =  360 / 1 = 360px
即这个时候页面呈现如下
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453865456333.png)


既然缩放值不会自动缩放，那么可以通过js去动态设置缩放值，看看能不能修复这个问题。

再来看viewport meta的一个属性：`initial-scale` 这个值可以显式地设置缩放值。记着，根据上述的公式，缩放值是相对ideal viewport width进行缩放的。

一个特殊的width的值`width=device-width`。设置这个值，浏览器会将当前页面的layout viewport的宽度设置为设备的ideal viewport的宽度。而layout viewport的值我们可以通过`document.documentElement.clientWidth`取得。

设置initial-scale指令实际上做了两件事：
1. 把页面的初始缩放因素设置了一个有意义的值，是相对于ideal viewport进行计算的，产生了visual viewport的宽度。
2. 根据刚刚计算得到的visual viewport的宽度值去设置layout viewport 的宽度值。

有了这些概念，开始我们的测试。
```
<meta name="viewport" content="width=device-width, initial-scale = 1, user-scalable=no" />// 目的是讲当前页面的layout viewport设置成ideal viewport width
var idaelViewport = document.documentElement.clientWidth; // 取得当前页面的ideal viewport width
var visualViewport = 375;
alert(idaelViewport)
var zoomView = idaelViewport/visualViewport;  // 动态计算缩放值
$('.j_Viewport').attr('content', 'user-scalable=no, initial-scale='+zoomView ); // 动态设置viewport meta
alert(zoomView)
```


问题是在某些安卓下initial-scale在设置成1的情况下才能通过计算设置layout viewport 和 visual viewport的值，所在在这个方案里这个也是失败的。

基于上述的知识点，再来看淘宝出品flexible.js的实现原理就很简单了。

flexible.js一共做了这么几件事，我们拿iphone5举例子，
iphone5的ideal viewport width为320。
1. 根据设备定义`dpr`
```
if (isIPhone) {
   // iOS下，对于2和3的屏，用2倍的方案，其余的用1倍方案
   if (devicePixelRatio >= 3 && (!dpr || dpr >= 3)) {                
       dpr = 3;
   } else if (devicePixelRatio >= 2 && (!dpr || dpr >= 2)){
       dpr = 2;
   } else {
       dpr = 1;
   }
} else {
   // 其他设备下，仍旧使用1倍的方案
   dpr = 1;
}
scale = 1 / dpr;
```
将iphone的不同型号划分为1、2、3
将安卓的dpr统一设为1
iphone5环境下，此时dpr=2，scale=0.5



2. 给html设置一个data-dpr属性
```
docEl.setAttribute('data-dpr', dpr);
```

3. 动态设置viewport meta
```
 metaEl = doc.createElement('meta');
 metaEl.setAttribute('name', 'viewport');
 metaEl.setAttribute('content', 'initial-scale=' + scale + ', maximum-scale=' + scale + ', minimum-scale=' + scale + ', user-scalable=no');
 if (docEl.firstElementChild) {
     docEl.firstElementChild.appendChild(metaEl);
 } else {
     var wrap = doc.createElement('div');
     wrap.appendChild(metaEl);
     doc.write(wrap.innerHTML);
 }
```
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453885901203.png)

```
<meta name="viewport" content="initial-scale=0.5, maximum-scale=0.5, minimum-scale=0.5, user-scalable=no">
```
现在在iphone5的环境，单独设置initial-scale=0.5有两个作用，之前也提到。根据公式
```
visual viewport width = ideal viewport width / zoom factor
```
可以得到visual viewport width=320/0.5 = 640
就是讲此时页面的visual viewport width和layout viewport width的值都设置为640px。

同时将html的宽也设置成640px。
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453886405621.png)

现在这个库已经把当前设备上的html宽度设置好了，不能缩放，visual viewport width和layout viewport width的值也相同，完全展示。

接下来就是要处理页面中的元素的大小了。

4. 利用rem设置页面元素大小
```
function refreshRem(){
   var width = docEl.getBoundingClientRect().width;
   if (width / dpr > 540) {
       width = 540 * dpr;
   }
   var rem = width / 10;
   docEl.style.fontSize = rem + 'px';
   flexible.rem = win.rem = rem;
}
```

这段代码中的`docEl.getBoundingClientRect().width;`取得的就是html的宽，也就是当前页面的最大宽。
`var rem = width / 10;`
然后将宽度/10得到一个基准数，在iphone下，这个值就rem=640/10 = 64。
`docEl.style.fontSize = rem + 'px';`将hmtl元素的fontSize设置为64px。

rem的工作方式就是相对于html元素的fontSize值进行计算的。

这个库的功能大概已经完成了，几段逻辑最后的目的就是将hmtl的fontSize值设置成64px，这该如何工作？

![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-IMG_3850.JPG)

我们一般拿到的设计稿都是750px的，默认将设计稿分成10份，将75px的像素对应一个rem，量的多少的像素相应的做一下换算，若量的的一张图片为30px，则对应的rem为30/75=0.4rem。然后在对应的页面里定义这个图片的宽就为0.4rem。

这就是flexible.js的工作方式。

原创文章，转载请注明出处。
