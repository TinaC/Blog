

```css
/* <display-outside> values: specify the element's outer display type, which is esssentially it's role in flow layout */
/* 让元素像<div> / <section> */
display: block; 
/* default value */
display: inline; 
...

/* <display-inside> values, inner display type, which defines the type of formatting context that lays out its contents (assuming it is a non-replaced element). */
display: table;
display: flex;
display: grid;
display: flow-root;
...

/* <display-legacy> block-level + inline-level */
/* 等同于inline flow-root*/
display: inline-block;
```

## inline

margin & padding只有水平方向的生效， width & height不生效，但是在chrome styles box里还能看见

## inline block

和inline的区别在于可以设置width & height & margin & padding了

## block

通常是
* container: div, section, ul
* text blocks: p, h1

如果没有设置width， block element是父元素的全部宽度

参考资料：

https://css-tricks.com/almanac/properties/d/display/

https://developer.mozilla.org/en-US/docs/Web/CSS/display