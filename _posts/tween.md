title: "tweenMax基础api的逐条随意翻译"
categories: 动画库
tags: [jsLib]
---
###tweenMax基础api的逐条随意翻译

`1`tween没有内置的选择器引擎，在没有载入jquery等类库的情况下只能通过`getElementById`进行选择。
`2.ticker`
```
TweenMax.ticker.addEventListener("tick", myFunction);

function myFunction(event) {
//executes on every tick after the core engine updates
}
```
`3.delay()`
```
var currentDelay = myAnimation.delay(); //gets current delay
myAnimation.delay(2); //sets delay
```
dalay方法可以设置或者获取某一个实例的延迟。
数值是以秒为单位的。
在设置了延迟的tween中，一切开始的动画都在delay之后开始，除了from()方法，from方法是不管延迟的，直接开始，除非设置了immediateRender:false参数在vars对象里。
```
TweenMax ( target:Object, duration:Number, vars:Object ) ;
//就是设置运动参数的vars对象。
```
`4.TweenMax.delayedCall()`
定义一个实例，这个实例表示延迟了多少秒后调用一个函数，而且回调函数可以传参数，
```
//calls myFunction after 1 second and passes 2 parameters:
TweenMax.delayedCall(1, myFunction, ["param1", "param2"]);

function myFunction(param1, param2) {
    //do stuff
}
//即如果这个方法写在文档加载后，即代表文档加载后过1秒执行myFunction函数，并且可以传一个数组的参数进去
```
`5.duration(time)`
获取tween实例的动画时长或者设置其动画时长
```
var currentDuration = myAnimation.duration(); //gets current duration
myAnimation.duration(2); //sets duration
```
`6.endTime()`
只返回tween实例结束动画的时间
`7.eventCallback( type:String, callback:Function, params:Array, scope:* )`
作用跟在tween的vars对象里直接写上事件是一样的，区别是这样做可以解除这个事件的绑定。
```
//the following two lines produce IDENTICAL results:
var myAnimation = new TweenLite(mc, 1, {x:100, onComplete:myFunction, onCompleteParams:["param1","param2"]});
myAnimation.eventCallback("onComplete", myFunction, ["param1","param2"]);
```
如果要给一个tween实例绑定不同的多个事件，则可以
```
myAnimation.eventCallback("onComplete", completeHandler).eventCallback("onUpdate", updateHandler, ["param1","{self}"])
//链式调用
```
如何解除绑定
```
//deletes the onUpdate
myAnimation.eventCallback("onUpdate", null);
```
**需要注意的是一个事件只能有一个callback，如果写多个callback那将会覆盖**
`8、TweenMax.fromTo( target:Object, duration:Number, fromVars:Object, toVars:Object )`
需要注意的是一般的vars的属性写在fromVars里，而特殊的类似onComplete等方法需要写在toVars对象里。
`9、TweenMax.getAllTweens( includeTimelines:Boolean ) : Array`
返回一个数组，包含所以的tween实例，参数可以选择是否返回timeline实例，如果想要实现对所有的实例同意操作，比如暂停，开始等，可以考虑使用`TimelineLite.exportRoot()`
`10、TweenMax.getTweensOf( target:Object, onlyActive:Boolean );`
在当前寻找到操作元素对象是Object的tween，并返回。也可能返回多个tween的数组.若onlyActive为true则只返回正在运行的tween。
```
TweenMax.to(myObject1, 1, {x:100});
TweenMax.to(myObject2, 1, {x:100});
TweenMax.to([myObject1, myObject2], 1, {opacity:0});

var a1 = TweenMax.getTweensOf(myObject1); //finds 2 tweens
var a2 = TweenMax.getTweensOf([myObject1, myObject2]); //finds 3 tweens
```
`11、TweenMax.globalTimeScale( value:Number );`
设置全部tween的速度，大于1则表示速度越大。
`12、.invalidate()`不懂
`13、.isActive( ) : Boolean`
可以查看这个tween是否是在运动中，没开始，结束，暂停都不算在运动中。
`14、TweenMax.isTweening(target: object)`
Reports whether or not a particular object is actively tweening. If a tween is paused, is completed, or hasn't started yet, it isn't considered active.
`15、.kill()`
看下方官方给出的示例代码就能清楚的知道.kill()方法的用途
```
// kill the entire animation:
myAnimation.kill();
 
//kill only the "x" and "y" properties of the animation (all targets):
myAnimation.kill({x:true, y:true});
 
//kill all parts of the animation related to the target "myObject" (if the tween has multiple targets, the others will not be affected):
myAnimation.kill(null, myObject);
 
//kill only the "x" and "y" properties of animations of the target "myObject":
myAnimation.kill({x:true, y:true}, myObject);
  
//kill only the "opacity" properties of animations of the targets "myObject1" and "myObject2":
myAnimation.kill({opacity:true}, [myObject1, myObject2]);
```
`16、TweenMax.killAll( complete:Boolean, tweens:Boolean, delayedCalls:Boolean, timelines:Boolean );`
`17、TweenMax.killChildTweensOf( parent:Object, complete:Boolean );`
去除父元素下的所有tween，可选的complete若为true则先完成动画 再kill。
`18、TweenMax.lagSmoothing()`不懂
`19、.pause( atTime:*, suppressEvents:Boolean ) : *`
如果括号内有秒计数的数字，则表示运动到atTime秒后在暂停
`20、TweenMax.pauseAll( tweens:Boolean, delayedCalls:Boolean, timelines:Boolean );`
括号内可选的属性的默认值都是true 表示默认暂停这些所有
`21、paused(boolean)`
getter or setter 判断当前tween实例是否处于停止状态。
下方代码的toggles比较有用
```
var paused = myAnimation.paused(); //gets current paused state
myAnimation.paused( true ); //sets paused state to true (just like pause())
myAnimation.paused( !myAnimation.paused() ); //toggles the paused state
```
`22、play(from:num, suppressEvents:Boolean)`
from值默认情况下是在元素当前的位置开始动。设置的数值代表直接从第几分钟开始运动
```
//begins playing from wherever the playhead currently is:
myAnimation.play();
//begins playing from exactly 2-seconds into the animation:
myAnimation.play(2);
//begins playing from exactly 2-seconds into the animation but doesn't suppress events during the initial move:
myAnimation.play(2, false);
```
`23、.progress( value:Number, suppressEvents:Boolean )`
指示tween跳转到进程的某一个阶段（0-1），比如0.5就代表跳到进程的一半。
```
var progress = myTween.progress(); //gets current progress
myTween.progress( 0.25 ); //sets progress to one quarter finished
```
`24、.repeat( value:Number )`
可以写在vars对象里
```
var repeat = myTween.repeat(); //gets current repeat value<br />
myTween.repeat(2); //sets repeat to 2
```
`25、.repeatDelay( value:Number )`
延迟几秒再重复
可以写在vars对象里
```
TweenMax.to(mc, 1, {x:100, repeat:2, repeatDelay:1});
```
也可以当方法写
```
var repeatDelay = myTween.repeatDelay(); //gets current repeatDelay value
myTween.repeatDelay(2); //sets repeatDelay to 2
```
`26、.restart( includeDelay:Boolean, suppressEvents:Boolean ) `
第一个参数表示是否包含上一次的延迟效果，如果为true，则表示重复的时候还有继承上一次的延迟，即触发的时候会延迟几秒再开始动作。
`27、.reverse( from:*, suppressEvents:Boolean )`
往相反的方向运动，如果将值设置为0，则从先前运动的最后时刻开始往反方向运动。如果是-1，则跳到距离终点还有1秒的时刻开始掉头运动。
```
//reverses playback from wherever the playhead currently is:
 myAnimation.reverse();
 
 //reverses playback from exactly 2 seconds into the animation:
 myAnimation.reverse(2);
 
 //reverses playback from exactly 2 seconds into the animation but doesn't suppress events during the initial move:
myAnimation.reverse(2, false);
 
//reverses playback from the very END of the animation:
myAnimation.reverse(0);
  
//reverses playback starting from exactly 1 second before the end of the animation:
myAnimation.reverse(-1);
 
//flips the orientation (if it's forward, it will go backward, if it is backward, it will go forward):
if (myAnimation.reversed()) {
    myAnimation.play();
} else {
    myAnimation.reverse();
}
 
//flips the orientation using the reversed() method instead (shorter version of the code above):
myAnimation.reversed( !myAnimation.reversed() );
```
`28、.reversed( value:Boolean )`
判断是否在相反的方向，用法跟pasued()一样
```
var rev = myAnimation.reversed(); //gets current orientation
myAnimation.reversed( true ); //sets the orientation to reversed
myAnimation.reversed( !myAnimation.reversed() ); //toggles the orientation
```
`29、TweenMax.set( target:Object, vars:Object ) `
```
TweenMax.set(myObject, {x:100, y:50, opacity:0});
//
TweenMax.to(myObject, 0, {x:100, y:50, opacity:0});
```
`30、.time()`
获取或设置tween的当前运动的时间点
`30、.timeScale()`
获取或设置当前tween的速度缩放数值


原整理文章，转载请注明出处！
