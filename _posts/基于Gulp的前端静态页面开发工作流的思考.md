title: "基于Gulp的前端静态页面开发工作流的思考"
date: 2015-12-07 01:09:39
categories: 前端工程
tags: [gulp]
---
##基于Gulp的前端静态页面开发工作流的思考

写静态页面的时候最常见的脚手架就是下方这样的形式，本地新建一个项目文件，然后再在里面新建js、less文件夹，配合一个less处理，js压缩混淆的可视化工具，边写边处理，写完后再到tinyPng上压缩一下图片，需要将图片地址换成cdn的还需要将图片手动传到服务器上，最后大功告成！

```
root
---js/
---src/
---less/
---css/
---images/
---html
```
但是可视化工具比如`codekit`的一个缺点就是可拓展性不高，局限于软件本身的功能。使用`gulp`以及强大的插件机制，可以带来无限的可能。
首先的问题就是项目文件目录该如何规划。如果只是上面提到的形式，生产代码跟发布代码混在同一个文件下，容易混淆。如果针对生产环境和发布环境分别对应一个文件夹就会清晰很多，如果两个文件夹的下属目录名称也相同，html文件引用资源的相对路径也就不需要改了。
先来看下项目脚手架的形式
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/NYGulp1.png)
- `src`为生产源代码，在编写的时候的代码都应该放入这个文件目录下，包括html文件。
- `dist`为上线的代码，里面存放的都是从src文件夹里压缩编译后的优化代码，包括html文件。
- `node_modules`存放的是相关于gulp的npm包。
- `config.json`有一些关于部署或者gulp需要的配置信息。
- `gulpfile.js`gulp的入口文件。
- `package.json`gulp所需插件的包管理。

再来展开看一下`src`目录。
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/NYGulp2.png)
生产源代码文件src下的目录结构
- `css` 同文件下less编译后得到。
- `images` 图片
-  `js` js文件
-  `less`less文件
-  `lib` 第三方库
-  `index.html`  主页面

再来看下`dist`文件目录
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/NYGulp3.png)
- `css`  src下的css文件压缩后得到
- `images` src下的images压缩得到
-  `js`  src下的js文件压缩混淆得到
-  `lib` 第三方库
-  `index.html`  主页面


----------
这篇文章假设大家已经基本了解了gulp如何安装使用，以及nodejs，npm包的使用。
http://markpop.github.io/2014/09/17/Gulp%E5%85%A5%E9%97%A8%E6%95%99%E7%A8%8B/
不清楚gulp初始使用的同学可以看下这边文章，跟着做基本就懂了几个大概的插件用法。


----------
首先从此github上clone下来这个脚手架 https://github.com/fromIRIS/NYGulp
然后`npm install`安装所有需要的gulp插件。

----------
### 代码编写阶段
我对于gulp的使用流程是这样思考的，在编写代码的时候，并不需要时刻对js进行压缩，也不需要对图片进行压缩，唯一要做的是对less文件的编译（因为我平时工作都是less，所以本文css预处理器的基于less的）。当然也可以对js进行代码风格的检查。
所以在编写代码的时候我们只需开一个gulp默认的命令行并在内部进行css文件的监听就好。
```
gulp
```
```
gulp.task("serve", ["less", "js-watch", "html"], function() {
    browserSync.init({
        server : "./src"
    });

    gulp.watch(srcPath.LESS, ["less"]);
    gulp.watch(srcPath.JS, ["js-watch"]);
    gulp.watch(srcPath.HTML, ["html"]);
    gulp.watch(srcPath.HTML).on("change", function() {
        browserSync.reload;
    });
});
gulp.task("less", function() {
    gulp.src(srcPath.LESS)
        .pipe(less())
        .pipe(gulp.dest(srcPath.CSS))
        .pipe(browserSync.stream());
})
gulp.task("js-watch", function() {
    gulp.src(srcPath.JS)
    .pipe(browserSync.stream());
})

gulp.task("html", function() {
    gulp.src(srcPath.HTML)
    .pipe(browserSync.stream());
})

gulp.task("default", ["serve"])
```
但是现在在编写代码的同时怎么还能持续的`command+R`呢，必须要用上时时刷新浏览器的功能啊，所以在代码编写的阶段，还要用到强大的`browser-sync`。这个插件最神奇的地方莫过于在一个终端操作，其他终端都能实时变化。比如在pc端填写表单，H5端也能实时的自动填写表单。
![][gif]
[gif]:http://7xpcne.com1.z0.glb.clouddn.com/NYGulp4120131606.gif
### 发布阶段 /1
代码写完了，在各个端都测试过觉得样式交互都ok了，感觉可以放到测试环境看看了，这个时候就可以对js文件进行压缩混淆，对css文件进行压缩，对图片进行压缩了。
这个时候在这个项目的根目录下执行这样的语句。
```
gulp build
```
此时根目录下就会多了一个`dist`文件夹，就是上文提到的那个。这个`dist`文件夹里的文件都是用于线上/测试环境的，不能在这个文件夹里直接修改代码。


### 发布阶段 /2
如果需要这个项目中使用`sprite雪碧图`，可以换一个命令，目前NYGulp只支持到PC的雪碧图自动生成。这个命令可以根据特定的图片文件夹生成一张sprite图片，并且自动将css文件中的`background-image`的地址替换掉，自动设置背景的`position`属性。
图片需要放在这个`src/images/slice` 目录下。**是必须的啊**
```
gulp buildspritePc
```


### 更多思考
一个前端静态项目的js、less、图片的编译压缩通过gulp都自动进行着，最后还有一个问题，也是这个脚手架的最后一公里，就是如何自动化地根据项目中html文件中图片的相对路径将图片上传到公司所在的静态资源服务器，然后将相对路径自动替换成cdn地址。如果能做到这样，对于一个静态活动页面，就真正的做到了自动化，只用进行代码的书写，无需再处理图片的压缩上传。
https://github.com/amfe/article/issues/8 , 事实上阿里的这个团队对前端自动化流的工具的研究已经很多了，从这篇文章里能找到解决思路，这也是我相继续深入的。
对于这样一个脚手架，在实际使用中已经较爽，可以利用 [Yeoman](http://yeoman.io/) 编写一个脚手架generator，这样在一个目录下执行一个命令就可以直接生成这个文件目录，免去了每次都要复制粘贴整个目录文档。


----------


参考资料： 
browser-sync使用 http://segmentfault.com/a/1190000002919912
百度前端脚手架文档： https://github.com/ecomfe/spec/blob/master/directory.md#level1
手淘前端自动化： https://github.com/amfe/article/issues/8
JGulp脚手架： http://segmentfault.com/a/1190000002658165
完！

原创文章，转载请注明出处！