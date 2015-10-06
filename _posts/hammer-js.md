title: "hammerjs的初次学习"
categories: javascript
tags: [javascript]
description: hammerjs的初次学习
---

##hammerjs的初次学习
###Hammer.js 手势库的学习与应用。
> 上个礼拜公司交代的任务中有一个需求，商品详细页面的图片点击后可以缩放拖拽。本来想用原生的api取写，但是出来的效果兼容性很差，在双击，拖拽，放大绑定在同一元素的情况下，很容易误触发不相关的动作，比如在双指缩放时一不小心就触发了双击事件，能力还未够，所以在hammer.js官网学习了一下手势库，完成了那个需求。

**1、hammerjs概括**
hammerjs一共有6种手势，`pan`(拖拽)，`pinch`(缩放)，`press`(长按) `rotate`(旋转)  `swipe`(滑动) `tap`(点)。这6个手势几乎涵盖了所有日常所需！
**2、用法**
需要实例化一个对象。
我这里提供一个我在项目中使用的通用写法。
我们写移动端一般都包含zepto的底层工具库。
```
var wrap = $('.wrap')[0];
//定义一个元素节点，注意这里不能是jq或者zepto对象
var wrapHammer = new Hammer(wrap);
//把需要绑定事件的节点传入，实例化。
var singletap = new Hammer.Tap();
//实例化手势对象
var doubletap = new Hammer.Tap({event: 'doubletap', taps: 2});
var pinch = new Hammer.Pinch();
var pan = new Hammer.Pan();
var swipe = new Hammer.Swipe();
```
在实例化手势对象的时候可以传参数，不同的手势对象有不同的参数可以选择。具体可以参考官网。
例如想看`pan`在实例化的时候可以填什么参数，可以查看http://hammerjs.github.io/recognizer-pan/ 。有一个框表示了传入的对象里可以填什么参数。
**这里比较特殊的是Tap**，不同数量的点击，需要实例化不同的对象。
**接下来**
```
wrapHammer.add(singletap);
wrapHammer.add(doubletap);
			  singletap.requireFailure(doubletap);			doubletap.requireFailure(singletap);
			  
wrapHammer.add(pinch);
wrapHammer.add(pan);
wrapHammer.add(swipe)
//在实例出来的节点对象上add上一步实例化的手势对象，此时一个节点上将绑定了很多手势。
```
```
wrapHammer.on('tap', hideWrap);
wrapHammer.on('doubletap', doubleScale);
wrapHammer.on('pinchmove', scaleImg);
wrapHammer.on('pinchend', scaleEndImg);
wrapHammer.on('panstart', moveStartImg);
wrapHammer.on('panmove', moveImg);
wrapHammer.on('panend', moveEndImg);
//第三步就是绑定事件了
```
**注意**第三步中的这些事件名，可以在官网上对应手势页面的events中找到。例如若要查看`pinch`则
http://hammerjs.github.io/recognizer-pinch/

**3、注意点**
紧跟以上三步，基本就能做出基本的手势页面了。
接下来再介绍几个需要特殊设置的地方。
`1`如果想要缩放与旋转同时触发，即边旋转边缩放。可以
```
var pinch = new Hammer.Pinch();
var rotate = new Hammer.Rotate();
pinch.recognizeWith(rotate);
```
官网页面：http://hammerjs.github.io/recognize-with/
`2`在doubletap时会在之前触发tap，此时可以设置
```
singleTap.requireFailure(doubleTap);
```
如此设置后在触发tap后稍微有些许停顿，以触发double。
`3`在官网或者网上经常看到`pan` `swipe` 在默认情况下只能水平移动，如果要开启还需要设置。但采用我上面的写法，就不需要设置了，默认情况下上下左右都是打开的。
http://hammerjs.github.io/recognizer-pan/ 
direction的默认值是DIRECTION_ALL，而且在最下面的note也说明了当采用`new Hanmmer()`实例化就默认水平，而我们是`new Hammer.pan()` 实例化。
`pinch`道理也同，不需要再配置就可以用了。
`4`在`on`的事件绑定中回调的event返回，最常用的也就是
```
ev.type
ev.deltaX
ev.deltaY
ev.scale
```

链接推荐：
https://cdn.rawgit.com/hammerjs/hammer.js/master/tests/manual/visual.html
http://www.cnblogs.com/iamlilinfeng/p/4239957.html

原创文章，转载请注明出处！
