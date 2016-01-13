title: "http+缓存"
date: 2015-12-22 11:12:07
categories: 前端
tags: [http]
---

##HTTP+缓存

引用：http://www.bokeyy.com/post/200-ok-from-cache-vs-304-not-modified.html
http://www.cnblogs.com/zhwl/archive/2012/02/28/2371691.html
http://div.io/topic/854下方hefangshi的评论

> http缓存概念

1、当web请求抵达缓存时，如果本地有“已缓存的”副本，就可以直接从本地存储设备，而不是从原始远程服务器上拉去数据。
2、与缓存相关的header
**request**
 - Cache-Control: max-age=0 以秒为单位
 - If-Modifiled-Since: xxxxxx 缓存文件的最后修改时间
 - If-None-Match: "676767xxxx" 缓存文件的Etag值
 - Cache-Control: no-cache 不使用缓存
 - Pragma: no-cache 不使用缓存

**response**
 - Cache-Control: public  响应被缓存，并且在多用户间共享
 - Cache-Control: private  响应只能作为私有缓存，不能在用户之间共享
 - Cache-Control:no-cache  提醒浏览器要从服务器提取文档进行验证
 - Cache-Control:no-store  绝对禁止缓存（用于机密，敏感文件
 - Cache-Control: max-age=60  60秒之后缓存过期（相对时间）
 - Date: Mon, 19 Nov 2012 08:39:00 GMT  当前response发送的时间
 - Expires: Mon, 19 Nov 2012 08:40:01 GMT 缓存过期的时间（绝对时间）
 - Last-Modified: Mon, 19 Nov 2012 08:38:01 GMT  服务器端文件的最后修改时间
 - ETag: "20b1add7ec1cd1:0"  服务器端文件的Etag值


3、直接使用缓存，不去服务器验证
**按F5刷新浏览器跟在地址栏输入网址回车，这两个行为是不一样的。**
 - 按F5浏览器会去web服务器验证缓存。
 - 而在地址栏输入网址，浏览器会直接使用有效的缓存，不会发httprequest去服务器。--缓存命中
 

> Status Code:200 OK (from cache) vs Status Code:304 Not Modified

如果浏览器进行资源请求，使用了缓存就无外乎以上两种response情况。
区别：
 - 200OK： 是浏览器没有跟服务器确认，直接使用了浏览器缓存。
 - 304 NOT Modified 是浏览器跟服务器多确认了一次缓存有效性，再用的缓存。

**1、两者触发的时机区别**
总结：200 OK (from cache) 是直接点击链接访问，输入网址按回车访问也能触发，后退也触发；而 304 Not Modified 是刷新页面时触发。

**2、具体原因：**
浏览器的缓存机制分为两块，也是规范中的4.2Freshness和4.3的Validation
这两块跟服务器的返回response头信息中的几个参数有关。
Freshness
 - Expires 缓存过期具体时间  --http1
 - cache Contrpl：max-age=60（注秒）缓存有效时间 --http1.1
 
解释：第一次浏览器请求资源后得到respone中的这两个参数信息，当再一次请求时浏览器会拿本地时间跟expires比较（http1），若判断此时的缓存是新鲜的，则直接调用缓存中的数据。状态码显示200ok
Validation
 - Last-Modifiled 数据最后修改的时间
 - ETag 类似于md5

解释：当第一步的Freshness验证不通过后，浏览器就会请求服务器，并将这两个参数传过去，如果服务器验证服务器端的数据没有过时，则返回304，如果数据过时了，则返回200和新的内容。

比如http://119.showjoy.com/activity/119meeting command+r刷新跟地址栏回车进入页面相同的资源，状态码不同，load的时间也不同。
