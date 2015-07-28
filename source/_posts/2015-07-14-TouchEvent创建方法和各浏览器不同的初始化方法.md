title: TouchEvent创建方法和各浏览器不同的初始化方法
date: 2015-07-14 21:49:21
tags:
- touchEvent
- initTouchEvent
---
## TouchEvent事件创建

```
//!Document.prototype.createTouch by w3c spec createTouch (https://developer.mozilla.org/ja/docs/Web/API/DocumentTouch.createTouch)
if (!Document.prototype.createTouch) {
    Document.prototype.createTouch = function(view, target, identifier, pageX, pageY,
                                              screenX, screenY, clientX, clientY,
                                              radiusX, radiusY, rotationAngle, force) {
        return {
            clientX       : clientX,
            clientY       : clientY,
            force         : force,
            identifier    : identifier,
            pageX         : pageX,
            pageY         : pageY,
            radiusX       : radiusX,
            radiusY       : radiusY,
            rotationAngle : rotationAngle,
            screenX       : screenX,
            screenY       : screenY,
            target        : target,
        };
    };
}

//!Document.prototype.createTouchList by w3c spec createTouchList (https://developer.mozilla.org/ja/docs/Web/API/DocumentTouch.createTouch)
if (!Document.prototype.createTouchList) {
    (function(global) {
        function TouchList(touches) {
            this.length = 0;
            if (!touches) {
                return this;
            }
            // list type argument
            else if (touches.length) {
                var touch;

                for (var i = 0, iz = touches.length; i < iz; i++) {
                    touch = touches[i];
                    this[i] = touch;
                }
                this.length = iz;
            }
            else {
                this[0] = touches;
                this.length = 1;
            }
        }
        TouchList.prototype = {
            constructor: TouchList,
            identifiedTouch: function(id) {
                var that = this;

                for (var key in that) if (!global.isNaN(global.parseInt(key)) && that.hasOwnProperty(key))  {
                    if (that[key].identifier == id) {
                        return that[key];
                    }
                }
                return undefined;
            },
            item: function(index) {
                return this[index];
            }
        };

        Document.prototype.createTouchList = function(touches) {
            return new TouchList(touches);
        };
    })(window);
}
```
参考：

[DocumentTouch,createTouch](https://developer.mozilla.org/ja/docs/Web/API/DocumentTouch.createTouch)

[DocumentTouch.createTouchList](https://developer.mozilla.org/ja/docs/Web/API/DocumentTouch.createTouch)

## initTouchEvent初始化事件

### Chrome, Opera

参考: [Chromium Source](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/events/TouchEvent.cpp&q=initTouchEvent&sq=package:chromium&type=cs&l=58)

```
var touch = document.createTouch(/* 省略 */);
var touches = document.createTouchList(/* 省略 */);
var event = document.createEvent('TouchEvent');

event.initTouchEvent(
    touches,              // {TouchList} touches
    touches,              // {TouchList} targetTouches
    touches,              // {TouchList} changedTouches
    'tap',                // {String}    type
    document.defaultView, // {Window}    view
    0,                    // {Number}    screenX
    0,                    // {Number}    screenY
    0,                    // {Number}    clientX
    0,                    // {Number}    clientY
    false,                // {Boolean}   ctrlKey
    false,                // {Boolean}   alrKey
    false,                // {Boolean}   shiftKey
    false                 // {Boolean}   metaKey
);
```

### Safari

参考: [Safari Developer Library - TouchEvent Class Reference](https://developer.apple.com/library/safari/documentation/UserExperience/Reference/TouchEventClassReference/TouchEvent/TouchEvent.html#//apple_ref/javascript/instm/TouchEvent/initTouchEvent)

```
var touch = document.createTouch(/* 省略 */);
var touches = document.createTouchList(/* 省略 */);
var event = document.createEvent('TouchEvent');

event.initTouchEvent(
    'tap',                // {String}    type
    true,                 // {Boolean}   canBubble
    true,                 // {Boolean}   cancelable
    document.defaultView, // {Window}    view
    1,                    // {Number}    detail
    0,                    // {Number}    screenX
    0,                    // {Number}    screenY
    0,                    // {Number}    clientX
    0,                    // {Number}    clientY
    false,                // {Boolean}   ctrlKey
    false,                // {Boolean}   altKey
    false,                // {Boolean}   shiftKey
    false,                // {Boolean}   metaKey
    touches,              // {TouchList} touches
    touches,              // {TouchList} targetTouches
    touches,              // {TouchList} changedTouches
    0,                    // {Number}    scale(0 - 1)
    0                     // {Number}    rotation
);
```

### Firefox

参考: [mozilla-central mozilla/dom/events/TouchEvent.h](http://lxr.mozilla.org/mozilla-central/source/dom/events/TouchEvent.h#108)

```
var touch = document.createTouch(/* 引数省略 */);
var touches = document.createTouchList(/* 引数省略 */);
var event = document.createEvent('TouchEvent');

event.initTouchEvent(
    'tap',                // {String} type
    true,                 // {Boolean} canBubble
    true,                 // {Boolean} cancelable
    document.defaultView, // {Window} view
    1,                    // {Number} detail
    false,                // {Boolean} ctrlKey
    false,                // {Boolean} altKey
    false,                // {Boolean} shiftKey
    false,                // {Boolean} metaKey
    touches,              // {TouchList} touches
    touches,              // {TouchList} targetTouches
    touches               // {TouchList} changedTouches
);
```

## 总结

- Android2.x, 3.x

	使用`document.createEvent('MouseEvent')`创建Touch事件，使用`initMouseEvent`初始化事件。TouchEvent添加`touches, targetTouches, changedTouches`
	
- Android4.x (Stock Browser & Chrome)

	使用`document.createEvent('TouchEvent')`创建，`initTouchEvent`使用**Chrome**的初始化方式。
	
- iOS

	使用`document.createEvent('TouchEvent')`创建，`initTouchEvent`使用**Safari**的初始化方式。
	
- PC Chrome & Opera

	同Android4.x。
	
- PC Safari

	同Android2.x, 3.x。
	
- PC Firefox

	`initTouchEvent`使用**Firefox**的初始化方式。
	
- PC Internet Exproler

	同Android2.x, 3.x。