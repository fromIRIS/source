title: "The Principles of object-oriented javascript阅读笔记"
date: 2015-11-29 23:52:59
tags: [书籍阅读]
---

##The principles of object-oriented javascript阅读笔记
----------
原书作者：Nicholas C.Zakas
笔记：南洋

面向对象的语言有如下几种特性：
 - 封装 数据可以和操作数据的功能组织在一起。
 - 聚合 一个对象能够引用另一个对象
 - 继承 一个新创建的对象和另一个对象拥有同样的特性。
 - 多态 一个接口可悲多个对象实现。

### 第1章原始类型和引用类型
使用和理解对象是理解整个javascript的关键。
其他编程语言用栈存储原始类型，用堆储存引用类型。
javascript中的区别：
> 它使用一个变量对象追踪变量的生存期，原始值被直接保存在变量对象中，而引用值则作为一个指针保存在变量对象上，该指针指向实际对象在内存中的存储位置。

**1.2原始类型**
5种原始类型：
 - boolean
 - number
 - string
 - null
 - undefined

所有原始类型的值都有字面形式，&&&字面形式是不被保存在变量中的值。eg.
```
var name = "nicholas";
var num = 13;
```
每个含有原始值的变量使用自己的存储空间，而这些变量保存在变量对象中。
下面这个表格很好说明了这个现象。
<table>
   <tr>
   	<td>variable object</td>
   </tr>
   <tr>
      <td>color1</td>
      <td>“red"</td>
   </tr>
   <tr>
      <td>color2</td>
      <td>“red"</td>
   </tr>
</table>
 **1.2.1 鉴别原始类型**
 鉴别原始类型的最好方式就是`typeof`，对于前三项原始类型，对应的typeof就是1、boolean2、number3、string。
 而对于空类型
 ```
 console.log(typeof null) // object
 ```
 这个其实是一个错误， **判断一个值是否为空类型的最佳方法是直接跟null比较 ===**
 要值得注意的是，因为`==`会强制转换类型，所以 `undefined == null`，所以这里空类型的判断还是使用`==`

**1.2.2 原始方法**
字符串、数字、布尔也有方法，但是他们不属于对象（null undefined没有方法）。

**1.3 引用类型**
引用类型是指javascript中的对象。引用值是引用类型实例，也是对象的同义词。
```
var object = new Object();
```
引用类型不在变量中直接保存对象，所以上面的代码变量`object`实际上并不包含对象的实例，而是包含一个指向内存中实际对象所在的位置的指针（引用）。

**1.3.2 对象引用解除**
在不使用某个对象时最好将引用解除，引用解除最好的方式是将保存引用的变量赋值`null`

**1.3.3 添加删除属性**
在javascript中可以随时添加删除对象的属性，同一个对象的不同引用全部会随之改变。

**1.4 引用类型-内建类型**
 - Array
 - Date
 - Error
 - Function
 - Object
 - RegExp
都可以通过new 来实例化每个内建引用类型。内建类型也有字面量形式。

**1.6 鉴别引用类型**
1、function 鉴别函数还是可以用`typeof`返回值是  》》》 `function`
2、对于数组和object、可以用`instanceof`
```
cosole.log([] insyanceof Array)  //true
cosole.log({} insyanceof Object)  //true
```
**注：instanceof能识别继承，即Array继承自Object， 所以[] instanceof Object   //true**

**1.8 原始封装类型**
- String
- Number
- Boolean
当用到原始类型的方法时，才会生成对象，但是随后就会被销毁、
```
var name = "lvdada";
var firstChar = name.charAt(0);

// javascript引擎
var name = 'lvdada';
var temp = new String(name);
var firstChar = temp.charAt(0);
temp = null;
```


### 第2章 函数

函数其实就是对象，函数区别于其他对象的决定性特点在于函数存在一个被称为[[call]]的内部属性，这个属性不被代码访问，但定义了代码执行时的行为。
ECMA定义typeof操作符对任何具有[[call]]属性的对象返回"function"。由于函数是对象，所以其行为跟其他语言的行为不同，理解了函数的行为就是理解javascript的核心。

**2.1 声明还是表达式**
```
// 函数声明
function add (num2, num1) {
	return num1 + num2;
}
```
```
// 函数表达式
var a = function (num1, num2) {
	return num1 + num2'
};
```
函数声明与函数表达式的区别在于表达式的尾部要加上分号。
**这里也特别强调，除了函数声明以外的任何javascript语句的尾部都要加上分号，以免解析器解析错误。**
另外一个重大的区别在于函数声明会发生变量提升的现象，跟var a = 13；的变量提升一样，但不同于原始数据类型的提升，函数声明的提升不会赋值为undefined。
**所以这样的影响就是可以在函数声明之前调用函数。**

**2.3 参数**
函数有一个属性`length`存的是函数期望的参数个数。
```
function a (num1, num2) {
	return num2 + num2;
}
a.length //2
```

**2.4 重载**
其他语言的函数存在重载模式，而javascript不存在重载，那是因为函数其实是对象，是Function的实例，当第二次定义重名的函数其实就是覆盖上一个同名函数。若要实现重载的功能，要依赖arguments.length或者检查参数是否存在。

**2.5 对象方法**
我们可以在任何时候给对象添加或删除属性。如果属性的值是函数，那么就称这个属性为方法。

**2.5.1 this 对象**
javascript所有的函数作用域内都有一个this对象代表`调用`该函数的对象。注意这里不是定义该函数的对象。我们知道函数的&&&作用域是在函数定义时就决定了的，而this的查找的对象是调用时的对象。
这本书有个例子
```
function sayNameForAll () {
	console.log(this.name);
}
var person1 = {
	name: 'nicho',
	sayName: sayNameForAll
}
var person2 = {
	name: 'lvdada',
	sayName: sayNameForAll
}
var name = 'Mich';

person1.sayName() // 'nicho'
person2.sayName() // 'lvdada'

sayNameForAll() // 'Mich'
```
这个例子就说明了`this`在函数调用时才被设置为调用此函数的对象。

**2.5.2 改变this**
要改变`this`代表的对象有三种方法。
 - call()
 - apply()
 - bind() ECMA5

1、call
call是函数的一个方法，它以指定的this对象和具体的参数来**执行函数**
```
function sayNameForAll (label) {
	console.log(label + ':' this.name);
}
var person1 = {	
	name: 'ni'
}

sayNameForAll.call(person1, 'hi');  // 'hi:ni'
```
直白的解释就是在执行sayNameForAll函数的时候，本来this会指向调用此函数的对象，但现在被显示的修改为`person1对象`，再传入一些这个函数需要的参数。

2、apply
跟call几乎完全一样，唯一的区别在后面的参数是以数组的形式传递进去的。
```
sayNameForAll(person1, ['hi']);
```

3、bind
这个方法是ECMA5定义的，低版本浏览器IE8不支持。bind跟call和apply的最大的区别在于，call，apply直接调用了那个函数。而bind不是。它是对函数声明时就限制this指向的对象，当一个函数调用bind方法时，并不直接执行这个函数。
```
function sayNameForAll (label) {
	console.log(label + ':' + this.name);
}
var person1 = { name: 'nid'}
var sayNameForPerson1 = sayNameForAll.bind(person1, 'person1');
sayNameForPerson1()  // person1:nid

//需要注意的是bind也能显示传入参数，上面的例子，bind的时候传入了'person1'字符串，在调用函数时就不需要传参数了。
```

### 第3章 理解对象
> javascript中的对象是动态的，可以在代码执行的任意时刻发生改变。基于类的语言会根据类的定义锁定对象，javascript没有这种限制。


**3.1 定义属性**
 > 当一个属性第一次被添加给对象时，javascript在对象上调用一个名为[[Put]]的内部方法，这个方法会在对象上创建一个新节点来保存初始的值，每添加一个属性，对象的内部都会调用一次这个方法，调用[[Put]]的结果是在对象上创建了一个自有属性，这个自有属性表明指定的对象实例拥有该悻，该属性被直接保存在实例中，对该属性的所有操作都必须通过该对象进行。

而当一个已有的属性被替换为一个新值时，调用的是名为[[Set]]的方法。

**3.2 属性探测**
我们经常会如此判断一个对象的属性是否存在
```
if (person.age) {
	// do something with age
}
```
如果person.age = 0，这个属性是存在的，但是在判断时候会是`false`.判断属性是否存在的方式是使用`in`操作符或者是`hasOwnProperty方法`。
用法：
 `"name" in person` 在person对象中是否存在name键名。
 `person.hasOwnProperty('name')`在person对象中是否存在name键名。
 两者的区别在于，in会检索原型上的键名，也能找到不可枚举的键名，而hasOwnProperty只检索自有属性。

**3.4 属性枚举**
`for in`会循环枚举一个对象所有的可枚举属性，也会跑到原型上去枚举那些可以被枚举的属性。
`Object.keys()`返回一个对象的所有自有属性（返回数组类型）


----------


### 第4章 构造函数和原型对象
**4.1 构造函数**
```
var person1 = new Person();
```
> 每个实例在创建时都自动拥有一个constructor属性，这个属性包含了一个指向其构造函数的引用。

eg
```
cosnole.log(person1.constructor === Person); //true
```
但是这种判断最好不要用在判断某对象是否是一构造函数的实例，因为实例的`constructor`属性是来自原型的，可能会被覆盖。
```
person1.hasOwnProperty('constructor')  //false
```

**4.2 原型对象**
> 几乎所有的函数都有一个名为 `prototype`的属性，该属性是一个原型对象，所有创建的对象实例共享该原型对象。且这些对象实例可以访问原型对象的属性。

比如，`hasOwnProperty()`方法就存在Object.prototype原型对象中，而new出来的或者字面形式的对象都能调用该方法。

这里由个方法可以鉴别一个原型属性
```
function hasPrototypeProperty (object, name) {
	return name in object && !object.hasOwnProperty(name);
}
```

**4.2.1 [[Prototype]]属性**
> 一个对象实例通过内部属性[[Prototype]]跟踪其原型对象，该属性是一个指向该实例使用的原型对象的指针。

`[[prototype]]`属性读取方法
```
var object = {};
var prototype = Object.getPrototypeOf(object);
console.log(prototype === Object.prototype); //true
```

也有一种方法检测某个对象是否是另一个对象的原型对象。
```
var object = {};
console.log(Object.prototype.isPrototypeOf(object)); // true
```

当访问一个对象的某个属性时，现在这个对象的自有属性中查找，若找不到，再去其原型对象中寻找。
**无法给一个对象的原型对象的属性赋值**。
即给一个对象添加一个跟原型同名的属性时，只是增加一个自有属性，而不会去原型对象中去改写属性。

**4.2.2 在构造函数中使用原型对象**
直接用一个对象字面量替代原型对象是非常流行且方便的方式，但存在一个问题。
```
function Person (name) {
	this.name = name;
}
Person.prototype = {
	sayName: function () { console.log(this.name) },
	toString: function () {console.log('')}
}
```
此时
```
var person1 = new Person('ldd');

console.log(person1.constructor === Person); //false
console.log(person1.constructor === Object); //true
```

> 为什么会发生这样的现象呢，上面解释过constructor是来自原型的constructor属性。当一个函数被创建时，它的prototype属性也被创建，即这个函数的原型对象，而原型对象具有constructor属性，并指向该函数。当用对象字面量改写了原型对象时，其constructor属性被置为Object，所以实例的constructor属性也指向了Object

改进方法
```
Person.prototype = {
	constructor: Person, // 改进方法
	sayName: function () { console.log(this.name) },
	toString: function () {console.log('')}
}
```

> 现在总结一下 `实例` `构造函数` `原型对象` 三者之间关系。
> 1、实例 通过内部的`[[prototype]]`属性指向原型对象
> 2、原型对象 通过constructor属性指向 构造函数
> 3、构造函数 通过prototype属性指向 原型对象
> 发现实例与构造函数之间没有明显的交流。


**4.2.3 改变原型对象**
> [[Prototpe]]属性只是包含了一个指向原型对象的指针，任何对原型对象的改变都立即反映到所有引用它的对象实例上。**这意味着给原型对象添加的新成员都可以立即被已经存在的对象实例使用**、


----------


### 第5章 继承
**5.1 原型链和Object.prototype**
**原型链：**对象实例继承了原型对象的属性，因为原型对象也是一个对象，它也有自己的原型对象并继承其属性。
所有对象都继承自Object.prototype

**5.2 对象继承**
> 对象继承是最简单的继承类型，只需要指定一个对象是另一个对象的[[prototype]]。对象字面量形式会隐式的指定Object.prototype为另一个对象的[[prototype]]，当然也可以通过Object.create()方法显示的指定。

实现的方式在**5.3节**会提到

**5.3 构造函数继承**
> 上文中讲到，几乎所有的函数都有prototype属性（原型对象），且这个属性可以被修改和替换。当这个函数被创建时，该`prototype`属性被自动设置为一个新的继承自Object.prototype的对象，而且这个对象有一个自有属性constructor。
其实看看javascript引擎做的事情就好理解了。
```
// 当我们写下下面这行代码时
function YourConstructor () {}

// javascript engine 会做这样的事情
YourConstructor.prototype = Object.create(Object.prototype, {
	constructor: {
		configurable: true,
		enumerable: true,
		value: YourConstructor,
		writeable: true
	}
});

```
 意思是当我们创建一个构造函数时，js引擎会自动的创建一个[[prototype]]为Object.prototype的，并且包含一个constructor属性的对象，赋值给构造函数的prototype属性。
 这样之后，就形成了2级原型链，**实例继承自构造函数的原型对象，而这个原型对象又继承自Object.prototype**。

> 由于prototype属性是可以重写的，可以通过改写它来改变原型链。

```
function Constructor1 () {}
Constructor1.prototype.add = function () {};

function Constructor2 () {}
Constructor2.prototype = new Constructor1();
Constructor2.prototype.constructor = Constructor2;
```

这里将Constructor2的原型对象置为了Constructor1的实例，跟上面js引擎的效果类似，原本Constructor2的原型对象应该是一个[[prototype]]为Object.prototype的对象，现在被赋值成了一个[[prototype]]为Constructor1.prototype的对象实例，根据原型链的概念，这样形成了继承。
**这里需要提起的是`new Constructor2 instanceof Constructor1` 这里就不能解释为new Constructor2 是 Constructor1的实例，因为`instanceof`是根据原型链判断的。**

这种继承方式是有问题的，因为`Constructor2.prototype = new Constructor1();` 如果Constructor1里有引用类型的值，比如数组，在Constructor2的原型对象里也会存在这个属性。而原型对象中的引用类型的属性是被所有实例共用的。

所以我们这里修改一下，不要带入被继承对象的内部属性。

```
function Constructor2 () {}

Contructor2.prototype = Object.create(Contructor1.prototype, {
	constructor: {
		value: Constructor
	}
});
```
我们在这里将Constructor2.prototype设置成一个只有constructor属性的并且[[prototype]]指向Constructor1.prototype的新对象，这样就不会带入污染物了。

但是通过原型链继承我们就拿不到父类的内部属性了，下面的代码可以解决。这种方式被称为伪类继承。
```
function Constructor1 () {}
Constructor1.prototype.add = function () {};

function Constructor2 () {
	Constructor1.call(this);
	// 如此就可将Constructor1的属性复制到Constructor2了
}

// 将Constructor1原型对象的属性复制来
Contructor2.prototype = Object.create(Contructor1.prototype, {
	constructor: {
		value: Constructor
	}
});

// 注意接下来Constructor2原型对象不能携程对象字面量的形式
```
