title: "seajs模块加载器在IE9及以下window.onload失效"
date: 2016-01-22 13:21:11
tags: [seajs]
---

## seajs在IE9及以下的window.load事件调用失败

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最近在在项目中遇到一个情况。项目使用seajs作为模块加载器，在js文件中的define回调中我们把函数都放在`$(function () {})`回调中去执行，意味着在文档加载完成后执行。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但是这个页面是商品详情页，包含着大量的详情图片，众所周知，加载的图片都会影响最后的load时间。

![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/seajs-load-1453433652471.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;就是最后红色的load时间，而这个时间的长短会影响页面加载旋转菊花的时间，![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/seajs-load-1453433717727.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;而这个旋转菊花的转态非常影响用户体验（很多人看这个菊花转太久就直接走人了）

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为了体验上的改进，项目中的详情页的商品详情区块都采用异步加载的方式加载。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但是在`$(function () {})`中异步加载商品详情区块并没有什么用，还是会影响最后的load时间。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;所以最后采用的是在define回调中使用`window.onload`方法，强制把商品详情区块在window.onload之后执行，这样加载的图片就不会影响load时间了。

```
addEventWhenOnload(window,'load',function(){
	// 更多活动信息加载
    new LoadActivity().init();
    // 商品详情信息加载
    new LoadDetails().init();
    new DetailTab().init();
})
```

在现代浏览器上很正常:smile:
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/seajs-load-1453435088012.png)

商品详情区块的图片都在红线之后加载（红线代表load时间）。

到这应该就圆满结束了。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但是万恶的IE8、IE9的问题又出来了。在IE8/IE9下异步加载的区块都没执行了。相应的页面中详情图片区块都空了。

一番搜索之后，问题找到了seajs上。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;seajs模块加载器的实现方法是动态的生成一个script，然后对这个script设置`async`属性。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对这个async属性做个简单的介绍，就是这个外部脚本不会阻塞domtree的构建，异步下载，一旦下载就会执行。
具体的可以看我翻译的一篇文章
http://lvdada.org/2015/11/12/%E7%A0%B4%E8%AF%91%E5%85%B3%E9%94%AE%E6%B8%B2%E6%9F%93%E8%B7%AF%E5%BE%84%EF%BC%88%E7%BF%BB%E8%AF%91%EF%BC%89/

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个机制本来对于我们之前的方案也没有影响，即使
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/seajs-load-1453439293592.png)
`item-detail.js`这个异步脚本会在蓝线之后再加载执行，但是jquery的`$(function () {})`的实现会判断目前文档是否到了domready状态，若执行的时候已经过了domready状态，就会立即执行。

但是`async`**只支持IE10及以上！**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;就是这个问题，在IE8、9浏览器上加载标有了`async`的外部脚本会什么时候开始加载执行呢，我们看实际的timeline。

![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/seajs-load-1453437393053.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;黄框就是加了`async`的脚本的加载执行时间，注意看图，加载的时间在load之后。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;现在就能清晰的知道原因了，业务js脚本在IE8/9下在load之后才加载，那在脚本中绑定的window.onload事件肯定就绑定不到了。

### 总结
在seajs中还是慎用window.onload啊。。