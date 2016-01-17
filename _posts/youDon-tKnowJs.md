title: "youDon'tKnowJs"
date: 2016-01-17 22:25:32
categories: 书籍阅读
tags: [js语言]
---

@(南洋)
## You don't konw JS 阅读心得
------by lyu dada
### javascript作用域
> 一门语言需要一套设计良好的规则来存储变量，并且之后可以方便的找到这些变量，这逃规则被称为作用域。

这也意味着当我们访问一个变量的时候，决定这个变量能否访问到的依据就是这个作用域。

#### 一、词法作用域
作用域共有两种主要的工作模型，第一种是最为普通的，被大多数编程语言（包括javascript）采用的`词法作用域`，另一种叫做`动态作用域`。而我们平时所提及的作用域，就是这里所说的`词法作用域`。

要了解词法作用域，必须要了解javascript引擎以及编译器的大概工作方式。一般程序中的源码在执行前会进行编译三步骤。
- 分词/语法分析
- 解析/语法分析
- 代码生成

而在分词/词法分析这个步骤，就已经确定了词法作用域。也就说作用域在我们书写代码的时候就已经确定了，引用书中的文字
> 词法作用域就是定义在词法阶段的作用域，换句话说，词法作用域是由你在写代码时将变量和块作用域写在哪里来决定的。

具体结合`编译器`、`作用域`、`引擎`来讲，编译器在分词阶段，针对特定的环境就会生成一个词法作用域，然后对源代码中的`var a = 3；`类似的声明进行识别，当遇到`var a`，编译器会询问作用域中是否有a变量，若无，则在作用域中新增一个a变量。编译完成之后，引擎执行编译后的代码，引擎在执行的过程中遇到`a`变量，会去作用域中查找是否有`a`变量，若有，则将`a`赋值2。对于`var a = 2；`一条语句会在两个过程中操作，正是变量提升现象的原因。（稍后讲到）

那什么时候会生成一个词法作用域呢？

#### 二、函数作用域
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/-1453026668155.png)

这幅图所展示的三个气泡，就代表了三个作用域，而编译器遇到一个函数定义，就会生成一个作用域。例如当编译器遇到`foo函数`，会创建一个作用域，再将这个函数内部的标识符（a/b/bar）放到词法作用域中。这个步骤在编译阶段就完成了。当js引擎执行`foo函数`的时候，遇到`a`变量，就会去询问早就创建好的作用域是否有`a`变量存在。

在作用域外，是无法访问作用域内的变量的。

例如
```
function foo() {
	var a = 3;
}
console.log(a); //undefied
```

**正是这个特性，可以被用来实现隐藏内部变量**
将重要变量声明放入一个函数声明的作用域中，可以防止被作用域外部的语句所引用甚至更改。

**根据函数作用域，可以引申出如何判断一个函数是函数声明还是一个函数表达式。**
最重要的区别是他们的名称标识符将会绑定在何处。

先声明一点，任何匿名函数都是可以添加名称标识符的。例如
```
setTimeout(function timer() {
	console.log(1)
}, 1000)
```

- 对于函数声明，名称标识符是绑定在当前作用域上的。即可在函数当前作用域调用这个名称标识符。
- 而函数表达式，名称标识符是绑定在自身的函数作用域中的。

按照这个区别，来看以下几个函数。
```
function foo1() {console.log(1)}
foo1(); // 1
```
```
var bar = function foo2() {console.log(1)}
foo2() // undefined
```
```
(function foo3() {console.log(1)})()
foo3() // undefined
```

以上的函数就只有`foo1`是函数声明。


#### 三、块作用域
在js语言中，除了函数，创建作用域的方式还可以通过块作用域。对于js而言，循环、ifelse块并没有创建块作用域的功能。

**通过ES3规范的`try/catch`的catch语句可以创建一个块作用域，其中声明的变量仅在catch中有效。**
而`try-catch`也正是`let`关键字的向前兼容方。
```
try {
 undefined(); // 执行一个非法操作来强制制造一个异常
} catch(err) {
	console.log(err);
}
console.log(err); // err not found
```


**ES6引入了`let`关键字，提供了除var以外的另一种变量声明方式，let为其声明的变量隐式地劫持了所在的块作用域。**
```
if (true) {
	{
		let bar = 3;
		bar = someting(bar);
		console.log(bar)
	}
}
console.log(bar) // undefined
```
作于的一个中括号起到划分块作用域的作用，显示的区别于`var`等变量。我们可能在之后会修改代码，看到这个中括号会直白的认识到这个是一个块作用域。

#### 四、变量提升
在第一节我已经提到了，对于`var a = 3;`这样一条语句，编译器通过分词、解析、最后生成机器可以读的代码。
> 而javascript实际上会将其看成两个声明：`var a`、`a = 3`。第一个声明在编译阶段进行，第二个赋值声明会留在原地等待执行。

所以在引擎工作去执行代码时，进入到函数作用域内时，首先会执行`var a`操作，而这个过程就好像变量从原先的位置被移动作用域最上面一样。
```
console.log(a); // undefined
var a = 3;
```
相当于
```
var a;
console.log(a); // undefined
a = 3;
```

另外函数声明也会发生变量提升的现象（连实际函数值也提升，即可以在函数声明前调用）。而行数表达式`var a = function foo1() {}`发生提升的是a变量，函数本身不会发生提升。
```
foo(); // 不是ReferenceError 而是 TypeError
var foo = function bar() {}
```

**ReferenceError  TypeError**
这是两个错误标记，第一个错误标记是查询变量时，若在作用域中查找不到这个变量则发出，第二个标记是能查找到变量（即使是endefined），但是这个变量被错误的调用（比如对null，undefined进行调用），发出。


### 作用域闭包
#### 一、经典的闭包
> 闭包是基于词法作用域书写代码时所产生的自然结果。

基于词法作用域产生的结果，这有点类似于词法作用域的产生条件。这也意味着闭包在书写代码的时候就已经形成了。

看一个最经典的闭包例子
```
function foo () {
	var a = 1;
	function bar () {
		console.log(a); //1
	}
	return bar;
}
var baz = foo();
baz();
```

基于这个经典的例子，结合书中的话
> 一个函数在定义时的词法作用域以外的地方被调用，可以记住并访问原先所在的词法作用域时，就产生了闭包。也即被返回出去的函数被调用时依然持有对该作用域的引用。这个引用就是闭包。

先确定一点，javascript中函数是可以作为值被传递的。基于这个特性，有多种方法可以行成闭包。只要在一个作用域中，将函数作为值传递到另一个词法作用域中并调用，就会形成闭包。
```
function foo() {
	var a = 2;
	function baz() {
		console.log(a);
	}
	bar(baz);
}
function bar(fn) {
	fn();
}
// 回调传递函数
```

```
var fn;
function foo() {
	var a = 2;
	function baz() {
		console.log(a);
	}
	fn = baz;
}
function bar() {
	fn();
}
foo();
bar(); //2
// 间接传递函数
```
无论通过何种手段将内部函数传递到所在的词法作用域以外，它都会持有对原始定义作用域的引用，无论在何处执行这个函数都会使用闭包。

#### 二、回调 == 闭包

再看上一节，回调中传递函数的例子。
```
function foo() {
	var a = 2;
	function baz() {
		console.log(a);
	}
	bar(baz);
}
function bar(fn) {
	fn();
}
// 回调传递函数
```
是将函数当做值并作为参数传递给函数。再来看
```
function wait(message) {
	setTimeout(function timer () {
		console.log(message); // hello world
	}, 1000)
}
wait('hello world');
```
`setTimeout`作为js内置的工具函数，将`timer 函数`当做值传进去，在setTimeout定义函数内对传进来的`timer`进行了调用。类似于
```
function setTimeout(fn) {
	// 延迟多少毫秒
	fn();
}
```
回调函数`timer`在另一个词法作用域内调用，但是能访问原先作用域内的参数（message）。

类似jquery中的事件绑定，涉及到传递回调函数，就都有闭包的产生！

#### 三、闭包在循环中的表现
最令人困惑的闭包表现就是在循环中了。像我们刚刚提及到的setTimeout、事件绑定等回调函数都会产生闭包。
```
for(var i = 1; i <= 5; i++) {
	setTimeout(function timer() {
		console.log(i);
	}, i*1000)
}
```
这个循环的本意是想间隔1秒打印1、2、3、4、5，结果却每隔1秒输出了5次6！
结合在第二节中对setTimeout函数的解析，这个误区将很快解开。

首先要明白for循环没有块作用域的概念，即在这个循环中5次迭代都是在同一个作用域中进行的。
要清楚`timer`函数不是在这个作用域中被调用的，它作为参数在其他的作用域中调用。
```
function timer() {
	console.log(i);
}
```
这个函数包括其中的形式参数`i`原原本本的被传递，在迭代过程中`i`不会被赋值。
而五次迭代完成后，共用的作用域中的`i`的值已经变成了6 。在其他作用域中的`timer`函数调用过程中需要查询`i`，因为产生了闭包，`i`的值会去原始的作用域中查找，即全是`6`。


得不到预期效果的错其实都在于for循环中共用一个作用域。想改进也很简单，即在迭代的过程中，创建对应的作用域。另外值得注意的一点是需要把每次迭代的`i`值传到作用域内。
```
for(var i = 1; i <= 5; i++) {
	(function (j) {
		setTimeout(function timer () {
			console.log(j)
		}, j* 1000)
	})(i)
}
```

#### 四、闭包的垃圾回收
本来一个变量被使用完之后就可以利用垃圾回收机制进行垃圾回收，但因为闭包的产生，阻止了这一行为。
```
function process(data) {
	//
}
var someReallyBigData = {};
process( someReallyBigData );
var $btn = $('.j_Btn');
$btn.on('click', function clicker() {});
```
这个例子中就是因为事件绑定机制中的传入了`clicker`回调函数，产生了闭包，引用着clicker所在的作用域，所以此处的someReallyBigData数据无法从内存中释放。

解决办法也有，声明一个块作用域，让引擎清楚的知道没有必要保存someReallyBigData饿了。
```
function process(data) {
	//
}
{
	let someReallyBigData = {};
	process( someReallyBigData );
}
var $btn = $('.j_Btn');
$btn.on('click', function clicker() {});
```

阅读心得，转载请注明出处。