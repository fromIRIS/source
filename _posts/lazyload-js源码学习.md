title: "lazyload.js源码学习"
date: 2015-09-28 00:59:55
tags:
---


##lazyload.js源码学习
```
/*
 * Lazy Load - jQuery plugin for lazy loading images
 *
 * Copyright (c) 2007-2013 Mika Tuupola
 *
 * Licensed under the MIT license:
 *   http://www.opensource.org/licenses/mit-license.php
 *
 * Project home:
 *   http://www.appelsiini.net/projects/lazyload
 *
 * Version:  1.8.5
 *
 */
(function($, window, document, undefined) {
    var $window = $(window);
    //基于jquery/zepto定义插件方法 lazyload
    $.fn.lazyload = function(options) {
        //this 指向了调用了lazyload得dom对象
        //一般是一个包含多个元素的对象
        var elements = this;
        var $container;
        
        /**
        * 定义了默认的配置对象
        * 在定义配置对象的时候最好将冒号对其 这样有益于阅读
        *
        **/
        var settings = {
            //起征点 图片在离屏幕什么地方开始load
            threshold       : 0,
            //可见区域外 搜索x个图片后停止搜索
            failure_limit   : 0,
            event           : "scroll",
            effect          : "show",
            container       : window,
            data_attribute  : "original",
            //是否跳过看不见的图片load
            skip_invisible  : true,
            appear          : null,
            load            : null
        };
        /**
        * function update
        * update函数的作用是在滚动后者resize屏幕时判断图片在屏幕中得位置，并作一些判断
        **/
        function update() {
            var counter = 0;
            //遍历需要懒加载的图片，根据当前屏幕相对文档的距离跟图片相对文档的距离的比较，去处理图片。
            elements.each(function() {
                var $this = $(this);
                //如果配置的是跳过看不见的图片 并且 这些图片设置隐藏 则跳出update方法
                if (settings.skip_invisible && $this.css("display") === "none") {
                    return;
                }
                if ($.abovethetop(this, settings) ||
                    $.leftofbegin(this, settings)) {
                        /* Nothing. */
                } else if (!$.belowthefold(this, settings) &&
                    !$.rightoffold(this, settings)) {
                        $this.trigger("appear");
                        /* if we found an image we'll load, reset the counter */
                        counter = 0;
                } else {
                    //
                    if (++counter > settings.failure_limit) {
                        return false;
                    }
                }
            });

        }
        /**
        * 如果有配置项写入 则将配置项extend到setting对象上。
        * 此处兼容了这个项目的之前的版本配置项的键名
        *
        **/
        
        if(options) {
            /* Maintain BC for a couple of versions. */
            if (undefined !== options.failurelimit) {
                options.failure_limit = options.failurelimit; 
                delete options.failurelimit;
            }
            if (undefined !== options.effectspeed) {
                options.effect_speed = options.effectspeed; 
                delete options.effectspeed;
            }

            $.extend(settings, options);
        }

        /* Cache container as jQuery as object. */
        $container = (settings.container === undefined ||
                      settings.container === window) ? $window : $(settings.container);

        /* Fire one scroll event per scroll. Not one scroll event per image. */
        if (0 === settings.event.indexOf("scroll")) {
            $container.on(settings.event, function(event) {
                return update();
            });
        }
        //这里的this就是要实现懒加载的所有图片集合 {object}
        /**
        * 给所有懒加载的图片遍历绑定事件 分2种情况
        * 1、setting配置scroll的时候
        * 绑定自定义的appear事件
        * 2、setting配置其他事件的时候 eq:click
        * 给图片绑定click事件，并且句柄设置为appear
        **/
        this.each(function() {
            var self = this;
            var $self = $(self);

            self.loaded = false;

            /* When appear is triggered load original image. */
            $self.one("appear", function() {
                if (!this.loaded) {
                    if (settings.appear) {
                        var elements_left = elements.length;
                        settings.appear.call(self, elements_left, settings);
                    }
                    $("<img />")
                        .on("load", function() {
                            $self
                                .hide()
                                .attr("src", $self.data(settings.data_attribute))
                                [settings.effect](settings.effect_speed);
                            self.loaded = true;

                            /* Remove image from array so it is not looped next time. */
                            var temp = $.grep(elements, function(element) {
                                return !element.loaded;
                            });
                            elements = $(temp);

                            if (settings.load) {
                                var elements_left = elements.length;
                                settings.load.call(self, elements_left, settings);
                            }
                        })
                        .attr("src", $self.data(settings.data_attribute));
                }
            });

            /* When wanted event is triggered load original image */
            /* by triggering appear.                              */

            /**
            * 如果配置项中得event不是scroll
            * 就需要给每个图片绑定一个事件
            * 比如click
            **/
            if (0 !== settings.event.indexOf("scroll")) {
                $self.on(settings.event, function(event) {
                    if (!self.loaded) {
                        $self.trigger("appear");
                    }
                });
            }
        });

        /* Check if something appears when window is resized. */
        $window.on("resize", function(event) {
            update();
        });
              
        /* With IOS5 force loading images when navigating with back button. */
        /* Non optimal workaround. */
        if ((/iphone|ipod|ipad.*os 5/gi).test(navigator.appVersion)) {
            $window.on("pageshow", function(event) {
                // if (event.originalEvent.persisted) {
                event = event.originalEvent || event;
                if (event.persisted) {
                    elements.each(function() {
                        $(this).trigger("appear");
                    });
                }
            });
        }

        /* Force initial check if images should appear. */
        $(window).on("load", function() {
            update();
        });
        
        return this;
    };

    /* Convenience methods in jQuery namespace.           */
    /* Use as  $.belowthefold(element, {threshold : 100, container : window}) */

    /**
    * $.belowthefold & $.rightoffold 
    * 是判断当前屏幕的最下方或最右方是否满足了加载图片的需求
    * false 触发trigge('appear')
    *
    *
    *
    **/
    $.belowthefold = function(element, settings) {
        var fold;
        
        if (settings.container === undefined || settings.container === window) {
            fold = $window.height() + $window[0].scrollY;
        } else {
            fold = $(settings.container).offset().top + $(settings.container).height();
        }

        return fold <= $(element).offset().top - settings.threshold;
    };
    
    $.rightoffold = function(element, settings) {
        var fold;

        if (settings.container === undefined || settings.container === window) {
            fold = $window.width() + $window[0].scrollX;
        } else {
            fold = $(settings.container).offset().left + $(settings.container).width();
        }

        return fold <= $(element).offset().left - settings.threshold;
    };
    

    /**
    * $.abovethetop & $.leftofbegin 
    * 判断当前屏幕的最上方或最左方是否已经超过了需要加载的图片的位置
    * true  nothing 
    *
    *
    *
    **/ 
    $.abovethetop = function(element, settings) {
        var fold;
        
        if (settings.container === undefined || settings.container === window) {
            fold = $window[0].scrollY;
        } else {
            fold = $(settings.container).offset().top;
        }

        return fold >= $(element).offset().top + settings.threshold  + $(element).height();
    };
    
    $.leftofbegin = function(element, settings) {
        var fold;
        
        if (settings.container === undefined || settings.container === window) {
            fold = $window[0].scrollX;
        } else {
            fold = $(settings.container).offset().left;
        }

        return fold >= $(element).offset().left + settings.threshold + $(element).width();
    };

    $.inviewport = function(element, settings) {
         return !$.rightoffold(element, settings) && !$.leftofbegin(element, settings) &&
                !$.belowthefold(element, settings) && !$.abovethetop(element, settings);
     };

    /* Custom selectors for your convenience.   */
    /* Use as $("img:below-the-fold").something() or */
    /* $("img").filter(":below-the-fold").something() which is faster */

    /**
    * 添加选择器
    * 通过筛选选择器可以方便的选出处于不同屏幕位置的图片
    * 在jquery的空间里配置了一些实用的选择器，可以借鉴
    *
    **/
    $.extend($.fn, {
        "below-the-fold" : function(a) { return $.belowthefold(a, {threshold : 0}); },
        "above-the-top"  : function(a) { return !$.belowthefold(a, {threshold : 0}); },
        "right-of-screen": function(a) { return $.rightoffold(a, {threshold : 0}); },
        "left-of-screen" : function(a) { return !$.rightoffold(a, {threshold : 0}); },
        "in-viewport"    : function(a) { return $.inviewport(a, {threshold : 0}); },
        /* Maintain BC for couple of versions. */
        "above-the-fold" : function(a) { return !$.belowthefold(a, {threshold : 0}); },
        "right-of-fold"  : function(a) { return $.rightoffold(a, {threshold : 0}); },
        "left-of-fold"   : function(a) { return !$.rightoffold(a, {threshold : 0}); }
    });

})($, window, document);

```
