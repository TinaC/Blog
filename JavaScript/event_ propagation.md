# event propagation 事件传播

>  element.addEventListener('click', listener, useCapture);
useCapture 默认是false, 表示使用bubble

## event phase

在DOM事件标准中，定义了事件传播的3个阶段

1. capturing phase 捕获阶段，事件从dom tree的上方向下方传递
2. target phase 目标阶段，事件到达目标元素
3. bubbling phase 冒泡阶段，事件从该元素向上传递。即触发子元素中注册的事件，再触发父元素中注册的事件。

![event_phase](/assets/article_images/2018/event_phase.png)

event.eventPhase可以获取当前阶段：
* none: 0 (Event.NONE)
* capture: 1 (Event.CAPTURING_PHASE)
* target: 2 (Event.AT_TARGET)
* bubble: 3 (Event.BUBBLING_PHASE)

通常在注册listener的时候已经知道event会在哪个phase发生，所以这个属性并不常用

## event.target

当事件发生时，最深嵌套(the most nested)的元素被标记为目标元素(target element). 可以通过event.target获取。
目标元素在冒泡/捕获过程不会变化。
但是handler的this会变化，是当前handler注册的元素。

## event.stopPropagation()

event.stopPropagation()只是阻止事件向上/向下被捕获，但是元素如果有多个handler，还是都会执行。

event.stopImmediatePropagation() 可以阻止所有别的handler执行

不要轻易使用event.stopPropagation(), 有些分析工具需要捕捉用户点击事件，如果阻止冒泡，document.addEventListener('click'…)这些listener都会失效。

## 例子

因为capture是第一阶段，所以注册在capture阶段的事件总是最先执行的。

点击最里层的“bubble”元素，触发的事件顺序：

![event_bubble_example](/assets/article_images/2018/event_bubble_example.jpg) 

## 历史

上古时代：

Netscape 支持capturing

Microsoft 支持bubbling

IE < 9 只支持冒泡，但是IE9+ & 所有现代浏览器两种都能支持。

事件冒泡的性能在复杂DOMs中可能稍微慢一些。

参考资料：

https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener

https://stackoverflow.com/questions/4616694/what-is-event-bubbling-and-capturing

https://javascript.info/bubbling-and-capturing