title: "viewport中的width、inital-scale简介"
date: 2015-07-12 23:02:56
categories: 前端
tags: [viewport]
---
##web移动端开发
###viewport
1、css的一个像素只是一个抽象的单位，在不同的设备中代表的设备物理像素是不同的。
2、`devicePixelRatio`一个css像素代表多少个设备像素
3、
**浏览器默认的viewport** 称为`layout viewport` ,且为比屏幕要宽的viewport，一般都为980px，可以通过以下代码获取
```
document.documentElement.clientWidth
```
**浏览器可视区域的宽度**：`visual viewport`可以通过以下代码获取
```
window.innerWidth
```
**浏览器理想宽度**：`idea viewport`
- 不同的设备有不同的idea viewport，所有的iphone的idea都是320px（iphone6以前）。查看最新`各设备idea viewport`网址http://viewportsizes.com/
- `idea viewport` 的宽度就是移动设备的屏幕宽度，在css中设置某元素的宽度为`idea viewport`的宽度（单位px），那么这个元素在屏幕中就是现实100%

4、![Alt text](./1436681520937.png)
**target-densititydpi**已经被安卓废除，避免使用
5、`initial-scale`这个缩放初始值是相对于设备的`idea viewport`的值进行缩放的，也就是说
```
<meta name="viewport" content="initial-scale=1"> 
//等效于
<meta name="viewport" content="width=devive-width"
//
```
**两者各有一个bug，所以生产时最好将两个一起写上**
但如果同时出现`width`和`initial-scale`
```
<meta name="viewport" content="width=500, initial-scale=1">
```
并不是根据先后顺序比较，而是通过谁大取谁，比如设备是iphone5，`idea viewport`是320px，则此时`initial-scale=1`(320px)，所以此时viewport取`width=500`
6、**关于initial-scale缩放的默认值以及其理论**
`visual viewport宽度 = idea viewport宽度 / 当前设置的缩放值`
`当前缩放值 = idea viewport宽度 / visual viewport宽度`
**注**这个理论不适合安卓原生浏览器
**关于默认值：（iphone及ipad下）**
在安卓浏览器下的`initial-scale`并不能默认设置自己的值，但ios下`initial-scale`会自动根据情况设置初始值（不写`initial-scale`）。
例：
```
<meta name="viewport" content="width=375">
```
此标签只写了`width=375`属性，表明了`visual viewport=375`而且`375`是iphone6的`idea viewport`，所以此时css设置样式`width: 375px`在6就是完全填充。但如何设备是苹果5，其`idea viewport`是`320`，根据理论，得出`initial-scale`的默认值为`320/375`。页面会自动缩放这样不会出现滚动条，这样就可以在`320`的视角下无障碍浏览375的内容。
###viewport总结
目前移动开发中关于页面布局有三种方式
- 1.缩放 ，利用iPhone的initial-scale默认值会自动变化
- 2.width=device-width，布局采用百分比 
- 3.rem 

上文的`initical-scale`的默认值的应用就是第一种方式的详解。
