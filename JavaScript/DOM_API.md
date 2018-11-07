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

---

# Create Element & Attribute

## document.createElement & document.createTextNode

```js
var heading = document.createElement("h1"); // <h1></h1>
var heading_text = document.createTextNode("Big Head!"); // Big Head!
heading.appendChild(heading_text); // 返回Big Head! (heading_text)
// heading: <h1>Big Head!</h1>
```

## document.createAttribute

https://developer.mozilla.org/en-US/docs/Web/API/Document/createAttribute

```js
var node = document.createElement("div");
var a = document.createAttribute("my_attr");
var a_empty = document.createAttribute("attr");
a.value = "newVal";
node.setAttributeNode(a); // <div my_attr="newVal"></div>
node.setAttributeNode(a_empty);  // <div my_attr="newVal" attr></div>
```

## 对应的jQuery

> `$("<div>")`
`$("<div></div>")`
`$(document.createElement('div'))`
https://stackoverflow.com/questions/268490/jquery-document-createelement-equivalent

> `$('#someid').attr('name', 'value')`
`$("#someid").removeAttr("name")`
for DOM properties like checked, disabled and readonly, the proper way to do this (as of JQuery 1.6) is to use `prop`
`$('#someid').prop('disabled', true)`
`$('#someid').removeProp('disabled')`

---

# Append & Prepend Element

## [ParentNode.append()](https://developer.mozilla.org/en-US/docs/Web/API/ParentNode/append) vs [Node.appendChild()](https://developer.mozilla.org/en-US/docs/Web/API/Node/appendChild)

ParentNode.append()是新API IE不支持

```js
var p = document.createElement("p");
document.body.appendChild(p);

var parent = document.createElement("div");
parent.append("Some text");

console.log(parent.textContent); // "Some text"
```

ParentNode.append() allows you to also append DOMString object,
Node.appendChild() only accepts Node objects.

ParentNode.append() has no return value
Node.appendChild() returns the appended Node object

ParentNode.append() can append several nodes and strings
Node.appendChild() can only append one node.

## [ParentNode.prepend()](https://developer.mozilla.org/en-US/docs/Web/API/ParentNode/prepend)

## [Node.insertBefore()](https://developer.mozilla.org/en-US/docs/Web/API/Node/insertBefore)

---

# Get Element Value

https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent#Differences_from_innerText

> 继承关系:
EventTarget <- Node <- Element <- HTMLElement <- HTMLInputElement
返回的内容多少:
Element.innerHTML > Node.textContent(推荐使用) > HTMLElement.innerText

## Node.textContent

The Node.textContent property represents the text content of a node and its descendants

```js
var span = document.createElement("span")

span.innerHTML = "60"
span.textContent= 80 // 覆盖了前一个

span // <span>80</span>
span.innerHTML
span.innerText
span.textContent // 都返回"80" 拿出来是string
```

## [Element.innerHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML)

可能会引起cross-site scripting attack, 建议使用`Node.textContent`而不是`Element.innerHTML`, `Node.textContent`不会parse传入的内容为HTML, 而是作为raw text插入。

```JS
// 虽然<script>不会执行
const name = "<script>alert('I am John in an annoying alert!')</script>";
// 但是还有其他方式执行JS
const name = "<img src='x' onerror='alert(1)'>";

el.innerHTML = name; // harmless in this case

var div = document.createElement("div")
div.innerHTML = "<h1>Properties of the DOM <span>load</span> Event Object</h1>"
div.textContent // Node.textContent: "Properties of the DOM load Event Object"
div.innerHTML // Element.innerHTML: "<h1>Properties of the DOM <span>load</span> Event Object</h1>"
```

## Node.textContent vs. HTMLElement.innerHTML

innerHTML返回html

textContent性能更好，也能避免xss attacks

## [Node.textContent vs. HTMLElement.innerText](https://stackoverflow.com/questions/35213147/difference-between-textcontent-vs-innertext)

*  Node.textContent把script & style里面的内容都能拿出来

```js
div.innerHTML = "<h1>Properties of the DOM <span>load</span> <script>function test() {}</script><style>table{ border-collapse: collapse;}</style>  Event Object</h1>"

div.innerText // "Properties of the DOM load function test() {}table{ border-collapse: collapse;}  Event Object"
div.textContent // 同上

// 这样拿出来结果是一样的，要是html里面创建的例子才可以？
// https://github.com/TinaC/FE_Notes/blob/master/js/dom/get_value.html
var html = document.getElementsByTagName("html")[0];

console.log(html.innerText); // "Hello"
console.log(html.textContent); // 多了: title, style, display: none 元素
```

*  innerText不会返回隐藏元素，textContent会

```js
div.innerHTML = "<h1>Properties of the DOM <span style='display: none;'>load</span>Event Object</h1>"

div.innerText // "Properties of the DOM loadEvent Object"
div.textContent // "Properties of the DOM loadEvent Object"
```

*  innerText is aware of CSS styling, 触发reflow. textContent不会
*  innerText was non-standard, while textContent was standardized earlier.

## [HTMLInputElement.value](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement#Properties)

`var value = document.getElementById("input").value;` // 获取input value

value: Returns / Sets the current value of the control.

```JS
// 在html里面这么写
<input type='text' id="input">content</input>
// 会被浏览器修改为, input闭标签被省略掉了
<input type="text" id="input">content

// 因为content没有包含在input里
var input = document.getElementsByTagName("input")[0]
input.textContent // ""
// 直接赋值是可以的
input.textContent = "TEST"
input.textContent // "TEST"
```
