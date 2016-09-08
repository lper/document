#rem好处
实用而论，不想了解具体原理的同学只要记住一点即可：在配置好环境的情况下，拿到设计稿并发现需要使用固定宽高比元素的时候，在css中做好px（设计稿中的px）与rem换算就行（即便这个过程也有不少偷懒的办法,减轻工作量）.
##关于rem的基础知识
这部分比较懒,可以参考

1. [知乎-css3的字体大小单位rem到底好在哪？](https://www.zhihu.com/question/21504656 "知乎-css3的字体大小单位rem到底好在哪？")
2. [移动端页面适配方案](http://ybshare.coding.io/share/flexible.htm "移动端页面适配方案") (主要参考的这个，还看不明白的可以看看)

##rem解决了什么问题?
针对文博在线微网站,rem能解决什么问题?他有什么好处呢?

我们是否遇到这样的情况:一个按钮,它的宽度可以做到自适应地随着设备的变化而去响应,但是高度呢?
![](https://raw.githubusercontent.com/wweggplant/blog/master/docs/images/17750930371867.png)

再比如一个两列的图片列表（这个图表达的有些问题，还没想好该怎么表达~~）:

![](https://raw.githubusercontent.com/wweggplant/blog/master/docs/images/17750922181547.png)

我们发现在没有使用特殊处理的情况下,想要图片按照给定的比例去显示是比较困难的.主要的原因是在css中height属性是不会轻易按照百分比去计算.之前项目中的pc站借助了javascript才搞定(当然最后发现没有什么实际的作用).但是在移动端,这个问题变的有意义了.

可能有人对rem兼容性有疑问,请看[这里](http://caniuse.com/#search=rem)

![rem兼容性](https://raw.githubusercontent.com/wweggplant/blog/master/docs/images/17220326519147.png)

可以说在手机端使用,应该是没有问题的.淘宝的手机站已经采取了rem的方案.

##rem实战
下面说说如何在实战中使用rem.首先我们设定,1rem等于当前设备状态下(不论横屏还是竖屏)的宽度的10%.

**设计稿到css**

以640设计稿举例,图中左右两个300px×150px的item

![](https://raw.githubusercontent.com/wweggplant/blog/master/docs/images/17200603827591.png)

那么我们分成10份后,1rem=64px

![](https://raw.githubusercontent.com/wweggplant/blog/master/docs/images/18310113443436.png)


如果是原来的写法的话

    .rem-block-wrap{
    float:left;
    width: 300px;
    height:150px;
    padding:10px;
    box-sizing: content-box;
	}
	.rem-block{
	    width: 100%;
	    height: 100%;
	    background: #fff;
	}

转换成rem写法:

	.rem-block-wrap{
    float:left;
    width: 4.6875rem; /*300px*/
    height:2.34375rem;/*150px*/
    padding:.15625rem;/*10px*/
	}
	.rem-block{
	    width: 100%;
	    height: 100%;
	    background: #fff;
	}

针对字体的话,保留了使用px,因为有的时候我们并不希望字体出现奇怪的值

	.example-item:nth-child(2),
	.example-item:nth-child(2) button{
	  font-size: 12px;
	}
	[data-dpr="2"] .example-item:nth-child(2),
	[data-dpr="2"] .example-item:nth-child(2) button
	{
	  font-size: 24px;
	}


在dpr等于2,且缩放为0.5的情况下(<meta name="viewport" content="initial-scale=0.5,maximum-scale=0.5,minimum-scale=0.5,user-scalable=no">),24px其实相当于之前的12px,这么做的目的主要是消除ios系统下的1px偏差,了解具体,请点击[这里](http://wweggplant.github.io/blog/example/rem.html)


css文件编写完毕后,在实际的html中我们需要计算真实设备中html的font-size的值,关键的代码如下:


**计算font-size和缩放的比例**

这里也是参考了 [别人的代码 ](http://www.meow.re/demo/screen-adaptation-in-mobileweb/mobile-util.js):

	window.mobileUtil = (function(win, doc) {
    var UA = navigator.userAgent,
        isAndroid = /android|adr/gi.test(UA),
        isIos = /iphone|ipod|ipad/gi.test(UA) && !isAndroid, // 据说某些国产机的UA会同时包含 android iphone 字符
        isMobile = isAndroid || isIos;  // 粗略的判断

    return {
        isAndroid: isAndroid,
        isIos: isIos,
        isMobile: isMobile,

        isNewsApp: /NewsApp\/[\d\.]+/gi.test(UA),
        isWeixin: /MicroMessenger/gi.test(UA),
        isQQ: /QQ\/\d/gi.test(UA),
        isYixin: /YiXin/gi.test(UA),
        isWeibo: /Weibo/gi.test(UA),
        isTXWeibo: /T(?:X|encent)MicroBlog/gi.test(UA),

        tapEvent: isMobile ? 'tap' : 'click',

        /**
         * 缩放页面
         */
        fixScreen: function() {
            var metaEl = doc.querySelector('meta[name="viewport"]'),
                metaCtt = metaEl ? metaEl.content : '',
                matchScale = metaCtt.match(/initial\-scale=([\d\.]+)/),
                matchWidth = metaCtt.match(/width=([^,\s]+)/);

            if ( !metaEl ) { // REM
                var docEl = doc.documentElement,
                    maxwidth = docEl.dataset.mw || 750, // 每 dpr 最大页面宽度
                    dpr = isIos ? Math.min(win.devicePixelRatio, 3) : 1,
                    scale = 1/dpr,
                    tid;

                docEl.removeAttribute('data-mw');
                docEl.dataset.dpr = dpr;
                metaEl = doc.createElement('meta');
                metaEl.name = 'viewport';
                metaEl.content = fillScale(scale);
                docEl.firstElementChild.appendChild(metaEl);

                var refreshRem = function() {
                    var width = docEl.getBoundingClientRect().width;
                    if (width / dpr > maxwidth) {
                        width = maxwidth * dpr;
                    }
                    var rem = width / 10;
                    docEl.style.fontSize = rem + 'px';
                };

                win.addEventListener('resize', function() {
                    clearTimeout(tid);
                    tid = setTimeout(refreshRem, 300);
                }, false);
                win.addEventListener('pageshow', function(e) {
                    if (e.persisted) {
                        clearTimeout(tid);
                        tid = setTimeout(refreshRem, 300);
                    }
                }, false);

                refreshRem();
            } else if ( isMobile && !matchScale && ( matchWidth && matchWidth[1] != 'device-width' ) ) { // 定宽
                var width = parseInt(matchWidth[1]),
                    iw = win.innerWidth || width,
                    ow = win.outerWidth || iw,
                    sw = win.screen.width || iw,
                    saw = win.screen.availWidth || iw,
                    ih = win.innerHeight || width,
                    oh = win.outerHeight || ih,
                    ish = win.screen.height || ih,
                    sah = win.screen.availHeight || ih,
                    w = Math.min(iw,ow,sw,saw,ih,oh,ish,sah),
                    scale = w / width;

                if ( scale < 1 ) {
                    metaEl.content = metaCtt + ',' + fillScale(scale);
                }
            }

            function fillScale(scale) {
                return 'initial-scale=' + scale + ',maximum-scale=' + scale + ',minimum-scale=' + scale + ',user-scalable=no';
            }


            if (!isMobile) {
                alert("请在现代浏览器(chrome,火狐)的开发者模式下,模拟移动端设计查看");
            }
        },

        /**
         * 转href参数成键值对
         * @param href {string} 指定的href，默认为当前页href
         * @returns {object} 键值对
         */
        getSearch: function(href) {
            href = href || win.location.search;
            var data = {},reg = new RegExp( "([^?=&]+)(=([^&]*))?", "g" );
            href && href.replace(reg,function( $0, $1, $2, $3 ){
                data[ $1 ] = $3;
            });
            return data;
        }
    };
	})(window, document);
	// 默认直接适配页面
	mobileUtil.fixScreen();

这段代码主要有两个作用:

1. 计算出适应宽度的meta标签的内容,`<meta name="viewport" content="initial-scale=x,maximum-scale=x,minimum-scale=x,user-scalable=no">`
2. 计算html节点的font-size


效果如下图:
![](https://raw.githubusercontent.com/wweggplant/blog/master/docs/images/17540427041802.png)

具体请点击[demo](http://wweggplant.github.io/blog/example/rem.html)

**px转rem**

到这里就会有一个问题,那就是px转到rem需要过程,这个工作想想就头大,不过这里我提供了几种方式减轻这个工作.

1.Sublime Text 2/3

效果如下图:

![](http://img4.07net01.com/upload/images/2015/08/12/1655152121636501.gif)

具体情况请[点击](http://www.07net01.com/2015/08/900796.html)

2.gulp插件

使用postcss中的postcss-px2rem插件,具体情况请[点击](http://www.jianshu.com/p/b130293511af)

    phpStorm中也可以整合gulp

##什么情况下使用rem?

1. 整体的布局还是使用百分比
2. 使用rem的最佳场景是,遇到例如多列带有图片的列表,常常需要图片固定宽高比例
3. 字体一般情况建议使用px