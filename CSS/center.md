
# 水平居中

* inline or inline-* elements(texts or links)

`text-align: center` 设置在父元素container

```html
<!-- text虽然没有标签也是子元素 document.createTextNode("text") -->
<header style="text-align: center">
  This text is centered
</header>

<nav role="navigation" style="text-align: center">
  <a href="#0">One</a>
  <a href="#0">Two</a>
  <a href="#0">Three</a>
  <a href="#0">Four</a>
</nav>
```

* block element

`margin: 0 auto` 设置给block element

没有width的时候block元素是full width不需要center

```js
<div style="width: 200px; margin: 0 auto;">
  I'm a block element
</div>
```

* more than one block level element

多个inline-block div水平居中

```css
.inline-block-center {
  text-align: center;
}
.inline-block-center div {
  display: inline-block;
}

// or 
.flex-center {
  display: flex;
  justify-content: center;
}
```

# 垂直居中

* inline or inline-* elements(texts or links) single line

```css
<!-- 1. padding-top = padding-bottom-->
.link {
  padding-top: 30px;
  padding-bottom: 30px;
}

<!--  2. 设置line-height = height -->
.center-text-trick {
  height: 100px;
  line-height: 100px;
  /* 保证只有1行 */
  white-space: nowrap; 
}
```

* inline or inline-* elements(texts or links) single line

1. padding-top = padding-bottom

2. display: table + display: table-cell

3. vertical-align: middle

参考资料：

https://css-tricks.com/centering-css-complete-guide/ 

https://www.w3.org/Style/Examples/007/center.en.html 