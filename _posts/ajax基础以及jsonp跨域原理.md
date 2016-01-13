title: "ajax基础以及jsonp跨域原理"
date: 2015-12-22 11:07:01
categories: js
tags: [ajax]
---

##ajax详解

参考：
http://segmentfault.com/a/1190000000490034
http://stackoverflow.com/questions/6792878/jquery-ajax-error-function
http://www.cnblogs.com/rainman/archive/2011/02/20/1959325.html

全称：Asynchronous JavaScript and XML（异步的JavaScript 和XML）

>**XMLHttpRequest**
> 简称xhr，是ajax最底层最核心的对象。

- 兼容性
```
function createXHR () {
	if (window.XMLHttpRequest) {
		return new XMLHttpRequest();
	} else if (window.ActiveXObject){
		return new ActiveXObject('Microsoft.XMLHTTP');
	} ese {
	alert('此浏览器不支持XHR')
	}
}
```
- XMLHttpRequest方法
 - `open(param1, param2, param3)`
XMLHttpRequest调用的第一个方法。
param1：string | "GET" | "POST" 代表将要通过什么方式进行数据交互。
param2：string | url 代表请求的地址。如果是GET请求，则将查询参数追加到url末尾，但是每一个名值对要经过encodeURIComponent()的编码。如果是post请求则不需要追加参数，直接写url。
param3：boolum | true | false true | 代表请求是异步的，不等待数据返回。
 - `send(param1)`
必须接受一个参数。如果是get请求要传递的数据已经在open里写过了，此处就为null。如果是post请求，则此时的参数写要传递的data。
 - `onreadystateChange()`
绑定readystateChange事件，当XMLHttpRequest的属性readystate变化时触发此事件。
 - `setRequestHeader()` 设置请求时的http头信息，说明数据的格式。这个方法只在post请求时用到，并且要在open、send方法之间调用。
- XMLHttpRequest属性
 - `responseText`
 - `responseXML`
 - `status`从服务器响应回来的http状态码
 - `readystate`服务器传回数据的状态，当为4时代表数据传回完毕。


> http简介

- ![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/ajax1.png)
- ![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/ajax2.png)
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/ajax3.png)


> 跨域 + 如何处理

出于安全方面的考虑，当前url下加载的页面的javascript无法访问其他地址上的数据。
```
http:// www.  showjoy.net   :8080   /u/orderRefund.html
```
![Alt text](http://7xpcne.com1.z0.glb.clouddn.com/ajax4.png)
(协议+子域名+主域名+端口)任意一个变化代表了域的不同。
- 如何处理跨域的问题？
 - jsonp （只针对get请求方式）?
 维基百科的定义是：JSONP（JSON with Padding）是资料格式 JSON 的一种“使用模式”，可以让网页从别的网域要资料。
 **jsonp跨域的原理：**
  1. 服务端（php）
	```
	$json = $_GET['callback'] . '(' . $json . ')';
	echo $json
	
	```
	
  2.  定义一个foo()函数
  3. 动态的生成一个script标签，src地址存放请求地址,带上查询的参数和`callback=foo`
  ```
  <script type="text/javascript" src="http://www.nanyang.com/user?id=123?callback=foo"></script>
  ```
   4. 此时服务器会返回来一个foo(data)，调用已经存在的foo函数。
   jquery中的jsonp也是这种原理。
	  ```
	  $.ajax({
             url: "http://nanyang.com/user?id=111",
             dataType: "jsonp",
             jsonp: "callback",//传递给请求处理程序，调用内部的方法，用以获得jsonp回调函数名的参数名(一般默认为:callback)，其实跟ajax已经没有关系了。
             jsonpCallback:"foo",//自定义的jsonp回调函数名称，默认为jQuery自动生成的随机函数名，也可以写"?"，jQuery会自动为你处理数据
             success: function(json){
                 alert('==');
             },
             error: function(){
                 alert('fail');
             }
         });
	  ```
   
  -  window.name + iframe跨域方法
```
1/在应用页面（a.com/app.html）中创建一个iframe，把其src指向数据页面（b.com/data.html）。
数据页面会把数据附加到这个iframe的window.name上，data.html代码如下：
<script type="text/javascript">
    window.name = 'I was there!';    // 这里是要传输的数据，大小一般为2M，IE和firefox下可以大至32M左右
                                     // 数据格式可以自定义，如json、字符串
</script>
2/在应用页面（a.com/app.html）中监听iframe的onload事件，在此事件中设置这个iframe的src指向本地域的代理文件（代理文件和应用页面在同一域下，所以可以相互通信）。app.html部分代码如下：
<script type="text/javascript">
    var state = 0, 
    iframe = document.createElement('iframe'),
    loadfn = function() {
        if (state === 1) {
            var data = iframe.contentWindow.name;    // 读取数据
            alert(data);    //弹出'I was there!'
        } else if (state === 0) {
            state = 1;
            iframe.contentWindow.location = "http://a.com/proxy.html";    // 设置的代理文件
        }  
    };
    iframe.src = 'http://b.com/data.html';
    if (iframe.attachEvent) {
        iframe.attachEvent('onload', loadfn);
    } else {
        iframe.onload  = loadfn;
    }
    document.body.appendChild(iframe);
</script>
3/获取数据以后销毁这个iframe，释放内存；这也保证了安全（不被其他域frame js访问）。
<script type="text/javascript">
    iframe.contentWindow.document.write('');
    iframe.contentWindow.close();
    document.body.removeChild(iframe);
</script>

```

   
完！

原创文章，转载注明出处~