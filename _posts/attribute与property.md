title: "attribute与property"
date: 2015-10-12 22:42:58
categories: js
tags: [property&attribute]
---
##Dom节点Attribute与Property

> 前段时间在尚妆请假OA系统中接触到了大量`checkbox/select/input`等表单元素，在使用jq的attr遇到了一些问题，所以了解了一下这几者的关系，顺便理了理dom的一些基础。

1、在IE<9中,浏览器会把所有的property和attribute强制映射，即property = attribute
tip：通过getAttribute获取到的值都是字符串，
但通过property获取的值可以是任何类型。

2、每个dom节点都是一个对象，既然是js对象，就跟对象一样式是有属性和方法的。而且原则上这些属性方法都是在js中出现供js调用的。但是不同的节点总会有几个默认的常见的属性出现在html上（不是页面上,是指在查看控制台中的element），比如a标签的`href`，img标签的`src`，以及多数标签的`id`。
当然方法也是可以出现在html上的，比如`onclick onblur等`
```
var list = [];
for (var key in document.body) {
	list.push(key)
}
console.log(list.join('\n'))
//这个方法可以在控制台查看某个dom对象的所有属性跟方法。
```

3、而出现在html中的属性在dom中（不是对象）就不叫`property`了，而叫`attribute`。
`property`和`attribute`存在**某种程度上**的映射关系，即改变其中之一，相对应的也会改变。

4、property跟attribute的读写方式不同，
```
var a = document.getElementById('test');
//property
》get a.id;
》set a.id = 'es';
//attribute
》get a.getAttribute('id')
》set a.setAttribute('id', 'es');
```

5、既然第四点内容里对于两种属性都有读写，那改变了其中一个（例如property），另一个属性会不会也改呢？接下来就来解释一下，这里面有一些特殊的点，记住就好。

property属性非常多，而attribute并不多，常用的有`name, id, scr, href, value, checked等`
- 对于这些attribute属性，通过setAttribute改变值时，都能同步到property上（即改变了attribute，htmldom直接显示改了， 然后去dom节点对象里查看property也变了）
- 而通过第4点中的property`a.id`写方法，对于`value、checked`属性来说，并不改变相对应的attribute的值。这也意味着改了property，但html中的value值却不变。
- **但是**： 对于input中的value和checked这两个属性来讲，property获取的是页面的显示，而attribute获取的是html中的属性值。
- 所以用户在输入框中输入，我们在js中要获取到就要使用property。
- 特别说明checkbox的property属性是checked = true，checkbox = false。而在html中写attribute时若要初始勾选就

```
<input type="checkbox" id="checkbox" checked>
```
6、还有个select表单元素的value属性，对于select元素而言，一般没有value的attribute属性，而property获取到的就是选中的下拉框option的value属性。
而对于option元素，其value的attribute和property属性都只指向html中的value值，不会涉及到**s或a**这些显示在页面上的值，这点不要跟input的输入框混淆了。
```
<select id="select">
 	<option value="22">s</option>
	<option value="1">a</option>
</select>
```

**综上所述**：除了input中的type=text的value和type=checkbox的checked属性外，其他属性用property和attribute差别不大。但是对于value和checked还是使用property比较方便，因为我们大多数都是想获取到用户输入了什么。对应到jq中就是使用其prop()方法。


还有二个特殊的属性，在映射中名字不一样。
> attribute   <==> property

1. class  <==>  className
2. data-x  <==>  dataset.x

这两个属性都是互通的。