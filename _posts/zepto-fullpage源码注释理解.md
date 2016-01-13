title: "zepto.fullpage源码注释理解"
date: 2015-08-03 01:07:28
categories: 源码
tags: [zepto.fullpage]
---

###zepto.fullpage的源码逐段注释理解
在工作中经常用到这个插件，去做h5端的全屏滚动，但不知道具体实现原理，所以就阅读了源码而且做了注释。
```

 //将传入的Zepto 和 windo 当做参数传入，这样可以提高文档内的节点搜索速度
(function($, window, undefined) {
    //判断zepto是否引用
    if (typeof $ === 'undefined') {
        throw new Error('zepto.fullpage\'s script requires Zepto');
    }
    //在手指滑动的时候阻止了屏幕默认的滚动
    $(document).on('touchmove', function(e) {
        e.preventDefault();
    });
    //初始化参数
    var fullpage = null;
    var d = {
        page: '.page',
        start: 0,
        duration: 500,
        loop: false,
        drag: false,
        dir: 'v',
        change: function(data) {},
        beforeChange: function(data) {},
        afterChange: function(data) {},
        orientationchange: function(orientation) {}
    };
    //判断是否有循环 返回页面序号
    function fix(cur, pagesLength, loop) {
        if (cur < 0) {
            return !!loop ? pagesLength - 1 : 0;
        }

        if (cur >= pagesLength) {
            return !!loop ? 0 : pagesLength - 1;
        }


        return cur;
    }
    //移动wpinner
    function move($ele, dir, dist) {
        var xPx = "0px" , yPx = "0px";
        if(dir === 'v') yPx = dist+"px";
        else xPx = dist + "px";
        $ele.css({
            '-webkit-transform' : 'translate3d(' + xPx + ', ' + yPx + ', 0px);',
            'transform' : 'translate3d(' + xPx + ', ' + yPx + ', 0px);'
        });
    }
    //初始化函数
    function init(option) {
        //将传入的option对象跟d深度合并，传给o
        var o = $.extend(true, {}, d, option);
        //这里的this是new Fullpage实例出来的对象
        var that = this;
        that.curIndex = -1;
        that.o = o;

        that.startY = 0;
        //这个用来判断页面是否在移动中
        that.movingFlag = false;

        that.$this.addClass('fullPage-wp');
        //传进来节点对象的父节点对象
        that.$parent = that.$this.parent();
        //子元素中page对象的集合
        that.$pages = that.$this.find(o.page).addClass('fullPage-page fullPage-dir-' + o.dir);
        //有多少个page
        that.pagesLength = that.$pages.length;
        //在实例化时就调用原型上的初始化方法
        //调节页面高度或宽度
        that.update();
        //初始化绑定事件
        that.initEvent();
        that.status = 1;
    }

    function Fullpage($this, option) {
        this.$this = $this;
        //init中定义的属性转移到Fullpage
        init.call(this, option);
    }
    //给Fullpage添加原型方法
    $.extend(Fullpage.prototype, {
        //更新page的高或宽，初始化出现的页面是第几页
        update: function() {
            //这里的this就是实例Fullpage后的对象，
            //$.extend 合并对象给Fullpage的原型
            if (this.o.dir === 'h') {
                this.width = this.$parent.width();
                this.$pages.width(this.width);
                this.$this.width(this.width * this.pagesLength);
            }
            //将page的高度设置成wp容器的高度
            this.height = this.$parent.height();
            this.$pages.height(this.height);
            //调用原型上的moveTo方法，将page初始化到第某页
            this.moveTo(this.curIndex < 0 ? this.o.start : this.curIndex);
        },
        //
        initEvent: function() {
            //将this传给that，不然下面的事件绑定的this会指向错误
            var that = this;
            var $this = that.$this;
            //通过给wpinner绑定touchstart和touchend来做手势滑动效果
            $this.on('touchstart', function(e) {
                if (!that.status) {return 1;}
                //e.preventDefault();
                if (that.movingFlag) {
                    return 0;
                }

                that.startX = e.targetTouches[0].pageX;
                that.startY = e.targetTouches[0].pageY;
            });
            $this.on('touchend', function(e) {
                if (!that.status) {return 1;}
                //e.preventDefault();
                if (that.movingFlag) {
                    return 0;
                }

                var sub = that.o.dir === 'v' ? e.changedTouches[0].pageY - that.startY : e.changedTouches[0].pageX - that.startX;
                var der = (sub > 30 || sub < -30) ? sub > 0 ? -1 : 1 : 0;
                //调用原型的moveTo方法
                that.moveTo(that.curIndex + der, true);
            });
            //拖拽的效果是手指滑动的时候页面跟着滑动，end时候触发moveTo方法
            if (that.o.drag) {
                $this.on('touchmove', function(e) {
                    if (!that.status) {return 1;}
                    //e.preventDefault();
                    if (that.movingFlag) {
                        that.startX = e.targetTouches[0].pageX;
                        that.startY = e.targetTouches[0].pageY;
                        return 0;
                    }

                    var y = e.changedTouches[0].pageY - that.startY;
                    //这里的两个if判断当页面在第一页或者最后一页时，页面只会移动一点恢复原状
                    if( (that.curIndex==0 && y>0) || (that.curIndex===that.pagesLength-1 && y<0) ) y/=2;
                    var x = e.changedTouches[0].pageX - that.startX;
                    if( (that.curIndex==0 && x>0) || (that.curIndex===that.pagesLength-1 && x<0) ) x/=2;
                    var dist = (that.o.dir === 'v' ? (-that.curIndex * that.height + y) : (-that.curIndex * that.width + x));
                    $this.removeClass('anim');
                    move($this, that.o.dir, dist);
                });
            }

            // 翻转屏幕提示
            // ==============================             
            window.addEventListener("orientationchange", function() {
                if (window.orientation === 180 || window.orientation === 0) {
                    that.o.orientationchange('portrait');
                }
                if (window.orientation === 90 || window.orientation === -90) {
                    that.o.orientationchange('landscape');
                }
            }, false);

            window.addEventListener("resize", function() {
                that.update();
            }, false);
        },

        start: function() {
            this.status = 1;
        },
        stop: function() {
            this.status = 0;
        },
        moveTo: function(next, anim) {
            var that = this;
            var $this = that.$this;
            var cur = that.curIndex;
            next = fix(next, that.pagesLength, that.o.loop);

            if (anim) {
                $this.addClass('anim');
            } else {
                $this.removeClass('anim');
            }
            //判断下一页的序号不是当前页··是否是最后一页了
            //在move之前调用beforeChange方法
            if (next !== cur) {
                that.o.beforeChange({
                    next: next,
                    cur: cur
                });
            }

            that.movingFlag = true;
            //改变当前序列号
            that.curIndex = next;
            //移动wpinner
            move($this, that.o.dir, -next * (that.o.dir === 'v' ? that.height : that.width));
            //在move之后马上调用change方法
            if (next !== cur) {
                that.o.change({
                    prev: cur,
                    cur: next
                });
            }
            //在move之后duration时间之后调用afterChange方法
            window.setTimeout(function() {
                //在每次调用moveTo多少时间后令movingFlag = false
                that.movingFlag = false;
                if (next !== cur) {
                    that.o.afterChange({
                        prev: cur,
                        cur: next
                    });
                    that.$pages.removeClass('cur').eq(next).addClass('cur');
                }
            }, that.o.duration);
        },
        movePrev: function(anim) {
            this.moveTo(this.curIndex - 1, anim);
        },
        moveNext: function(anim) {
            this.moveTo(this.curIndex + 1, anim);
        }
    });
    //这句话解释起来就是在$的原型上写一个fullpage方法，所以this就是实例化出来的对象，在zepto中$(this)就是节点对象
    $.fn.fullpage = function(option) {
        if (!fullpage) {
            fullpage = new Fullpage($(this), option);
        }
        return this;
    };
    $.fn.fullpage.version = '0.3.1';
    //暴露方法
    $.each(['update', 'moveTo', 'moveNext', 'movePrev', 'start', 'stop'], function(key, val) {
        $.fn.fullpage[val] = function() {
            if (!fullpage) {
                return 0;
            }
            fullpage[val].apply(fullpage, [].slice.call(arguments, 0));
        };
    });
}(Zepto, window));

```

原创文章！转载请注明出处
