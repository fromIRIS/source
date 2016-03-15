title: "callback-hell"
date: 2016-03-15 15:47:04
tags: [callback]
---

## callback hell翻译
> 文章来源 http://callbackhell.com/

## 回调地狱
一篇如何进行javascript异步编程的指导文章。

什么是“回调地狱”？
javascript异步编程，或者说使用回调进行javascript编程，是很难凭直觉得到想要的返回值的。很多代码长的像下面一样：

```
fs.readdir(source, function (err, files) {
  if (err) {
    console.log('Error finding files: ' + err)
  } else {
    files.forEach(function (filename, fileIndex) {
      console.log(filename)
      gm(source + filename).size(function (err, values) {
        if (err) {
          console.log('Error identifying file size: ' + err)
        } else {
          console.log(filename + ' : ' + values)
          aspect = (values.width / values.height)
          widths.forEach(function (width, widthIndex) {
            height = Math.round(width / aspect)
            console.log('resizing ' + filename + 'to ' + height + 'x' + height)
            this.resize(width, height).write(dest + 'w' + width + '_' + filename, function(err) {
              if (err) console.log('Error writing file: ' + err)
            })
          }.bind(this))
        }
      })
    })
  }
})
```

看这个金字塔形状还有到处以`})`的结尾，是的！这就是著名的回调地狱。

造成回调地狱的原因在于，一些开发者想要用一种看起来是从上到下顺序执行的一种方式进行javascript编码。多数开发者都犯了这种错误。其他像`c/ruby/python`一样的语言不出意外地在执行时都是无论在第一行发生了什么，都会在第二行开始前结束，正如你接下来会看到的，javascript是不同的。

### 什么是回调？
回调只是使用javascript函数的一个约定俗成的名称。在javascript语法里没有专门的一个称为回调的名词。这只是一个约定俗成。跟大部分直接返回结果的函数不同，使用回调的函数会花费一些时间处理结果。`asynchronous`简称`async`只是意味着花费一些时间或发生在将来，而不是现在。通常情况下回调被使用在处理`I/O`上，类似于下载，读取文件，或处理大数据上等等。

当你调用一个普通的函数你可以使用他的返回值。

```
var result = multiplyTwoNumbers(5, 10)
console.log(result)
// 50 gets printed out
```

然而，使用回调的异步的函数不会立马得到返回值。

```
var photo = downloadPhoto('http://coolcats.com/cat.gif')
// photo is 'undefined'!
```

在这个例子gif图可以能花费大量时间下载，你并不像让你的程序在等待下载结束前暂停（又称 阻塞）。

相反，这段应该在下载后才执行的代码在一个函数中完成，这就是回调！你把这个回调传递给了`downloadPhoto`函数，这个函数在下载完成后会调用你的回调（e.g. 稍后调用），然后将值传递给`photo`参数。（出错的情况下会传值给error）

```
downloadPhoto('http://coolcats.com/cat.gif', handlePhoto)

function handlePhoto (error, photo) {
  if (error) console.error('Download error!', error)
  else console.log('Download finished', photo)
}

console.log('Download started')
```

开发者理解回调最大的障碍就是理解程序运行时的执行顺序。在这个例子中三件事情发生。第一件事`handlePhoto`函数被声明了，然后`downloadPhoto`函数被调用并将`handlePhoto`函数作为回调传递进来，最后`Download started`打印。

值得注意的是`handlePhoto`函数目前为止还没有被调用。这个函数只是被创建被当做回调被传递进`downloadPhoto`函数。但直到`downloadPhoto`函数完成他的任务前`handlePhoto`是不会被调用的。取决于网速这可能会花费大量时间。

这个例子目的是展示了两个重要的内容：
- `handlePhoto`回调只是存储一些稍后执行的代码块
- 代码执行的顺序将不会是从上到下的，这取决于什么时候完成。

### 我如何解决回调地狱？
回调地狱造成的原因是差劲的编码实践，幸运的是书写优美的代码没这么难！

#### 1.保持你的代码是有易懂的
以下是使用`browser-request`请求ajax的浏览器端js代码。

```
var form = document.querySelector('form')
form.onsubmit = function (submitEvent) {
  var name = document.querySelector('input').value
  request({
    uri: "http://example.com/upload",
    body: name,
    method: "POST"
  }, function (err, response, body) {
    var statusMessage = document.querySelector('.status')
    if (err) return statusMessage.value = err
    statusMessage.value = body
  })
}
```

这段代码有两个匿名函数，让我们给他名字！

```
var form = document.querySelector('form')
form.onsubmit = function formSubmit (submitEvent) {
  var name = document.querySelector('input').value
  request({
    uri: "http://example.com/upload",
    body: name,
    method: "POST"
  }, function postResponse (err, response, body) {
    var statusMessage = document.querySelector('.status')
    if (err) return statusMessage.value = err
    statusMessage.value = body
  })
}
```

正如我们所能看到的给函数命名非常简单并且有立显的好处：
- 由于描述性的函数名字使代码更易读。
- 当意外发生的时候你可以得到确切函数名提及的堆栈信息而不是「匿名函数」
- 能够移动函数或者通过名字引用函数

现在让我们把函数移动到程序的顶部：

```
document.querySelector('form').onsubmit = formSubmit

function formSubmit (submitEvent) {
  var name = document.querySelector('input').value
  request({
    uri: "http://example.com/upload",
    body: name,
    method: "POST"
  }, postResponse)
}

function postResponse (err, response, body) {
  var statusMessage = document.querySelector('.status')
  if (err) return statusMessage.value = err
  statusMessage.value = body
}
```

值得注意的是函数声明在文件的最底部声明，这多亏了[函数提升](https://gist.github.com/maxogden/4bed247d9852de93c94c).


#### 2.模块化
这是最重要的部分：每个开发者都有能力创建模块（类似第三方库）。引用`Isaac Schlueter`（一个node项目） 「写一些各司其职的小模块，然后把他们装配在一个大模块里去产生更大的作用。」You can't get into callback hell if you don't go there

我们把上面提到的实例代码拿来拆分成几个文件来转变成模块。我会展示一个浏览器端或服务端（或两者）可以工作的一个模块形式。

接下来一个叫做`formuploader.js`的新文件包含了之前的两个函数。

```
module.exports.submit = formSubmit

function formSubmit (submitEvent) {
  var name = document.querySelector('input').value
  request({
    uri: "http://example.com/upload",
    body: name,
    method: "POST"
  }, postResponse)
}

function postResponse (err, response, body) {
  var statusMessage = document.querySelector('.status')
  if (err) return statusMessage.value = err
  statusMessage.value = body
}
```

`module.exports`是一个工作在node，electron。和使用了browserify的浏览器端的nodejs模块系统的一个例子。

现在我们有了`formuploader.js`，这个文件在被编译之后作为一个script标签被引入页面中。我们只需要去依赖这个文件然后使用它。写下来的范例就是我们的应用如何使用这个文件，

```
var formUploader = require('formuploader')
document.querySelector('form').onsubmit = formUploader.submit
```

现在我们的程序只有两行，这有一下好处：
- 对于新人来说是易于理解的 -- 他们不需要去理解`formuploader`函数的具体内容。
- `formuploader`不需要被拷贝就可以在其他地方使用也可以很方便的在github或npm中共享。

#### 3.处理每个error
有不同种类的错误：syntax errors 由程序员自己造成（通常是在启动程序时被捕获到），runtime errors 由程序员造成（代码中有个bug导致的出错），platform errors 由一些不合法的文件造成，硬盘失败，网络连接错误等等。这个章节主要为了讲述这几个类型的错误。

前两个规则主要关于如何使你的代码可读性高，但这个规则是关于让你的代码更可靠。当我们在处理一些分发下来、默默进行的任务定义的回调时，回调又会在不经意间成功完成或者失败退出。任何一个有经验的开发者都会告诉你你永远都不会知道错误在哪里时候发生，所以你最好企划好错误一直在发生。

对于回调而言处理errors最流行的方法就是nodejs风格，第一个参数永远都是为了error准备的。

```
var fs = require('fs')

 fs.readFile('/Does/not/exist', handleFile)

 function handleFile (error, file) {
   if (error) return console.error('Uhoh, there was an error', error)
   // otherwise, continue on and use `file` in your code
 }
```

第一个参数设置为error是个普遍的约定为了鼓励你记得去处理error。如何error是第二个参数你可能就会像`function handle (file){}`这样写代码而忘记了处理error。

代码风格检查器也能配置帮助你记住处理回调的错误信息。

#### 总结
1. 不要嵌套函数。给匿名函数命名并移动到程序的上层。
2. 使用[变量提升](https://gist.github.com/maxogden/4bed247d9852de93c94c) 去调用下层的函数。
3. 处理回调中的每个错误处理，使用类似于`standard`这样的代码检查工具帮助你记住去处理回调中的错误处理。
4. 创建可重复使用的函数并放到模块中减少理解你的代码的认知难度。将你的代码拆分成一些小的模块以帮助你处理错误信息。编写测试用例，强制自己创建可靠的并带有文档的公共API的代码。这也能帮助你重构。


避免回调地狱的最重要的一个方面就是将函数从嵌套中移出来，这样代码的流程将更容易读懂，新手也不用啃过所有回调函数的具体内容来理解整个程序是怎么工作的了。

对于创建模块有几个规则：
- 首先从将重复使用的代码装入函数中
- 当你的函数（或者是跟某一主题相近的一组函数）到达一定大小后，将他们移到另一个文件然后通过`module.exports`去引用。你可以通过相对路劲去引用。
- 如果你有一段代码是可以在很多项目中通用的，给这段代码加上readme，测试用例，`package.json`然后发布到github和npm上去。对于这个举动有列不完的好处！
- 一个好的模块是小体量的但是针对一个问题。
- 模块中的单独的一个文件不应该超过150行。
- 一个模块不应该有超过一个嵌套的包含js文件的文件夹，如果发生了，可能是你赋予这个模块做了太多的事情。
- 咨询更多有经验的程序员向你展示好模块的例子直到你对其有一定概念，如果这需要花费很多时间，只能证明这可能不是一个好模块。

#### 更多阅读
尝试阅读我的[更多](https://github.com/maxogden/art-of-node#callbacks) 或者尝试[nodeschool](http://nodeschool.io/)教程。

同时也可以尝试[browserify-handbook](https://github.com/substack/browserify-handbook) ，里面有许多书写模块代码的例子。


### `promises/generators/ES6`如何？
在寻找高级解决方案之前，记得回调是javascript的记基础（因为他们就是函数），你应该在目光转移到高级语法特性之前首先要学会读写回调。因为理解这些高级语法特性都是在理解回调的基础上进行的。如果你还不能写可维护性的回调代码，那么久继续加油吧~

如果你真的想让你的异步代码是从上往下走的，这有一些你可以尝试的好用的方法。请注意这些可能会涉及性能问题或者是跨平台的问题，所以确保你做了功课。

promise是一种看起来仿佛是从上往下执行的一种异步代码编写方式。由于`try/catch`风格的错误信息处理方式可以让promise处理多种错误信息。

generators能让你「暂停」一个单独的函数而不影响整体程序的运行。理解起来有些困难但这种方式能让你的异步代码看起来是从上到下的风格。尝试一下[watt](https://github.com/mappum/watt)

async函数是ES7的一个新特性，这个特性把generator和promise包装成了更高级的语法，如果听起来感兴趣的话就尝试下吧。

个人而言我写90%饿异步代码都是使用回调来写，当需求复杂了我会引入类似于[run-parallel](https://github.com/feross/run-parallel) [run-series](https://github.com/feross/run-series)这样的东西。我并不觉得callback、promise或其他概念对我而言有什么区别。最大的影响来自于保持代码的简单，不要嵌套，将代码拆分成小模块。

不管你选择了什么方法，记得总是要处理每一个回调错误信息，并保持代码简单。

**牢记， 只有你自己能防止回调地狱的产生，当然了，还有森林大火**

你可以在[github](http://github.com/maxogden/callback-hell)上找到源码。


翻译文章~欢迎转载~