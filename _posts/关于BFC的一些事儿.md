title: "关于BFC的一些事儿"
date: 2015-06-08 22:58:17
categories: css
tags: [BFC]
---

##**关于BFC的一些事儿**：
阅读：穆乙 http://www.cnblogs.com/pigtail/
http://www.iyunlu.com/view/css-xhtml/55.html

`什么是BFC`
BFC（Block Formatting Context），中文为块状格式化上下文。
按我的理解，就是触发BFC环境的元素在文档流中各自独立，谁也不碍着谁，这个元素的子元素还是按照普通的文档流进行排列。
**触发BFC常见的有几种方式**：overflow不为visible，float不为none， display的值为table-cell, table-caption, inline-block中的任何一个， position为absolute
在IE中，个人理解BFC是hasLayout，而触发hasLayout的简便方法就是zoom: 1;

**触发BFC的好处主要在两方面，第一点是清除浮动，第二点在于处理上下margin的塌陷。**

`BFC与清除浮动`
在需要清除浮动的时候
`1`overflow: hidden
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/BFC1433772289741.png)
**解释如下：**
根据 CSS2.1 规范第 10.6.3 部分的高度计算规则，在进行普通流中的块级非替换元素的高度计算时，浮动子元素不参与计算。

同时 CSS2.1 规范第10.6.7部分的高度计算规则，在计算生成了 block formatting context 的元素的高度时，其浮动子元素应该参与计算。

所以，触发外部容器BFC，高度将重新计算。比如给outer加上属性overflow:hidden触发其BFC。

`2`触发BFC的元素与浮动元素不会重叠
若两个Div排列，第一个div浮动，此时两个div会重叠，但将第二个div触发BFC，则此两个div分开（横向）


`BFC与margin上下塌陷`
在普通流中的两个元素，不管是在同级还是包含关系中，margin-top和margin-bottom接触就会塌陷只留下较大方。
**即使父元素触发BFC，里面的正常流中的子元素之间还是会发生塌陷。**
因为触发BFC的两个元素相对独立，所以解决了塌陷，这个比较好理解。
对于嵌套元素之间的塌陷，即下图：
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/BFC1433775283824.png)

**合理解释为：**
根据 CSS 2.1 8.3.1 Collapsing margins 第三条，生成 block formatting context 的元素不会和在流中的子元素发生空白边折叠。
所以在父元素上触发BFC即可。


####**清除浮动的两种方法**：
`1`
```
.container:before,
.container:after {
  content:"";
  display:table;
}
.container:after {
  clear:both;
}
.container {
  zoom:1; /* For IE 6/7 (trigger hasLayout) */
}
```
`2`
```
.container {
    overflow: hidden; /* Clearfix! */
    zoom: 1;  /* Triggering "hasLayout" in IE */
    display: block; /* Element must be a block to wrap around contents. Unnecessary if only using block-level elements. */
}
```



原创文章，转载请注明出处！
