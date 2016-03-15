title: "the-3th-viewport-translate"
date: 2016-01-27 20:16:20
tags: [viewport]
---

原文：http://www.quirksmode.org/mobile/metaviewport/#t10
### 第三种viewport
多年前 我提到了移动端的浏览器有两种viewport：visual viewport很layout viewport。如果有必要可以重读这几篇文章。[文章](http://www.quirksmode.org/mobile/viewports2.html)
我假设这两个viewport的知识大家已经掌握了。

#### ideal viewport
结果表明存在第三种viewport，我把他叫做ideal viewport。这个viewport能将设备上的页面设置能最理想的尺寸。因此，每种设备的ideal viewport的尺寸是不一样的。

在老的便宜的没有retina屏幕的机型上，ideal viewport跟物理像素一致，但这不是必须的。更新物理像素更高的设备保持旧的ideal viewport，因为这是最理想最适合设备的。

直到iphone4s，不管是否有retina屏，他的ideal viewport都是320x480。因为320x480对于在这种iphone上的页面来说，是最理想的。

关于idealviewport 最重要的两点内容：
1. layout viewport能被设置成ideal viewport。`width=device-width和initial-scale=1`指令可以做到。
2. 所有的`scale`指令都是相对于ideal viewport的。无视layout viewport设置了多少，所以`maximum-scale=3`意味着最大的缩放值是ideal viewport的300%

#### 寻找ideal viewport
读取到ideal viewport的值在某些情况下是有用的。
是的，你可以读取到。给页面设置一个如下显示的meta标签，然后使用`document.documentElement.clientWidth`读取到当前设备的ideal viewport。
```
<meta name="viewport" content="width=device-width,initial-scale=1">
```

如果上面这个方法你不能选择而导致你不能读取到ideal viewport的值，我希望`screen.width`可以帮助到你，但是只有BlackBerry能给出正确的信息。其他浏览器会得出不同的无用的信息。

开放的问题：`screen.width`应该取得ideal viewport的值吗
赞成：这个属性至少应该取得有用的信息
反对：ideal viewport 跟物理像素一样无用

![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/viewport-1453693967543.png)

#### layout viewport
在呈现页面之前，浏览器需要知道layout viewpoint的宽是多少。这个viewport的值是相对于css声明来计算的，例如`width：20%`

在没有任何指令的情况下，移动端浏览器会自己选择一个默认值，大部分是980px。这无关对错，只是浏览器供应商自己的选择。

当你在meta viewport标签里设置了width=400或者其他任何数值，layout viewport会被设置成这个值，我们已经知道了。

然而Android WebKit’s and IE 的最小layout viewport是320px，当设置的值小于320，会自动变为ideal viewport的值。

有一种情况是layout viewport跟ideal viewport的值相同，这发生在设置了`width=device-width or initial-scale=1`。这是复杂的，因为safari和ie10有一些bugs，使用`inittial-scale`也是有陷阱的，但是这是普通的规则。

#### 最小、最大的尺寸
最大的layout viewport的宽度是10000像素，我不是十分相信这个数字，因为浏览器不会允许你缩小到这个数值，到目前为止我是接受这个官方数字的。

最小的layout viewport的值大约是1/10的ideal viewport，这也是最大的缩放等级。（layout viewport 永远不会变的比最小的visual viewport还要小）例外：Android WebKit and IE最小的layout viewport不会小于320px

#### 缩放
zoom缩放是狡猾的。理论上听起来很简单：决定用户缩放的缩放因素包含两个方面：
1. 我们不能直接读取缩放因素，取而代之的是我们不得不读取与缩放因素互相关联的visual viewport的宽度。
所以最小的缩放因素决定了最大的visual viewport的宽度，反之亦然。
2. 结果表明所有的缩放因素都是相对于ideal viewport的，不管当前的layout viewport的值是多少。

然后就是关于名字的讨论，用Apple的语言来讲，zoom就是scale，类似meta viewport的指令`initial-scale, minimum-scale, and maximum-scale`，其他浏览器厂商被迫遵从为了保持对已经兼容了iphone的页面的兼容性。

这三个指令预示着缩放因素，initial-scale=2意味着缩放到200%的ideal viewport的宽度。

#### 公式
首先让我们定义公式：
```
visual viewport width = ideal viewport width / zoom factor
zoom factor = ideal viewport width / visual viewport width
```

因此，ideal viewport为320px设置了缩放值2我们会得到160px的的visual viewport。layout viewport的宽度不参与计算。

#### 最小、最大的缩放因素
浏览器支持的最小和最大的缩放因素是多少呢？

首先会有一个限制，visual viewport永远不会比layout viewport要宽，所以大多数情况下最下的缩放因素为`ideal viewport width / layout viewport width`





#### initial-scale
设置initial-scale指令实际上做了两件事：
1. 把页面的初始缩放因素设置了一个有意义的值，是相对于ideal viewport进行计算的，产生了visual viewport的宽度。
2. 根据刚刚计算得到的visual viewport的宽度值去设置layout viewport 的宽度值。

让我们看看一台竖屏的iphone（译者：4s），我们设置`initial-scale=2`，并且不设置其他的指令。这个指令将visual viewport的宽设置为160px（320 /2），这应该不会让你感到惊讶。这就是缩放指令工作的方式。

然而，这个指令同样让layout viewport的宽设置为160px，所以我们有一个缩放值最小状态下160px宽的页面。（visual viewport 不会变的比layout viewport更大，所以不能再缩小了）

但是这没什么意义，如果要问我的意见我可能会说：完全没有任何意义。然而，毫无疑问的是浏览器的行为就是这样的。

#### 浏览器bugs
对于安卓，明显的，安卓允许在其值等于1的情况下，`initial-scale`去设置layout viewport的宽。并且不能设置`width`的值。所以只有在没有其他指令的情况下`initial-scale=1`的设置会启到作用。

对于IE，他提供了错误的ideal viewport（320x320和不是320x480），而且IE假装所有initial-scale的值都为1，所有对于IE来讲设置initial-scale的值是无用的。

#### 指令的混淆
 因为`initial-scale`能设置layout viewport的宽，你现在可以创建一个有争议的指令
 ```
 <meta name="viewport" coontent="initial-scale=1, width=400">
 ```
 现在会发生什么？浏览器得到混淆的指令，我们再回到4s上，
 1. `initial-scale=1`告诉浏览器设置layout viewport的宽为320px
 2. `width=400`告诉浏览器设置layout viewport 的宽为400px

翻译文章！经验有限，慎用！