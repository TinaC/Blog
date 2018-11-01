列举一些 live coding 常用的 DOM API

# DOM element & jQuery element 之间的转化

[How can I convert a DOM element to a jQuery element?](https://stackoverflow.com/questions/625936/how-can-i-convert-a-dom-element-to-a-jquery-element)

```js
var elm = document.createElement("div");
// $() 不仅接受selector作为参数，也可以接受jQuery element & DOM element
var jelm = $(elm);//convert to jQuery Element

var htmlElm = jelm[0];//convert to HTML Element

// 拿到第一个div
$("div")[0]
$("div").get(0)

// 拿到所有div(s)
$("div").get()

// 返回一个数组，包含所有div(s), 但是还有selector & prevObject等properties
$("div")
```