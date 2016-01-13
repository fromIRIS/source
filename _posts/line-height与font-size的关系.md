title: "line-height与font-size的关系"
date: 2015-12-22 11:14:34
categories: css
tags: [line-highe]
---

##line-height与font-size的关系

> 对于P元素，当文字超出一行时，决定每行之间的间隙的css是line-height和font-size

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;解释：P元素中有两行文字，每一行是一个`inlineBox`,其高对应的css是`line-height`。在`inlineBox`里有一个区域叫做`contentArea`，这里显示的是文字，其高对应的css是`font-size`。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**浏览器自动将contentArea区域设置在inlineBox区域的垂直居中位置。**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;所以，当inlineBox的高大于contentArea的高，在视图里展现的样子是两行文字之间有一定的间隙，间隙的大小取决于两者高度的差值。

> 接下来讲一下css属性中line-height继承问题。

当在css中要继承父元素的line-height时，有几种写法：
- `1、`px   
-  `2、`%
-  `3、`num 
-  `4、`normal(1.2)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`px` 当在父元素上使用px时，其子元素都会不顾自己的font-size而继承px的确切值。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这样造成的影响是子元素的inlineBox的高度都是相同，而子元素的font-size不同的话就会造成各个文字区块的行间距不同，影响美观。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`&` 是将此时元素的font-size乘以%得到现有的line-hight，同px，子元素继承固定值。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`normal`相当于数字1.2 ，子元素同样继承固定的line-height。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`num`子元素只继承num这个数值，但具体line-height数值会跟每个子元素的自身font-size相乘得出line-height。

| element | font-size | line-height   |after line-height|
| :-------- | --------:| :------: |----|
| body   |   14 |  1.2  |28px|
|p|30|1.2|36|
|h1|24|1.2|28|


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这样的效果是每个文字区块中的文字段落的行间距都是根据自己的font-size得出的，整体文字排版的行间距很统一。