• static
• relative
• fixed
• absolute
• inherit

* position: static 默认值
不会被 top, bottom, left, right影响，总是按照页面正常流来放置
	
* position: relative
可以设置 top, bottom, left, right
对于position: relative的元素设置top等，是相对于原来的位置

* position: fixed
总是相对于viewport来放置，也就是说即使页面滚动也保持在同样的位置，
可以设置 top, bottom, left, right

* position: absolute
相对于设置了position的第一个父元素进行定位。如果没有设置了position的祖先（position不是static），就相对于document body，跟着页面滚动

Z-index：
规定元素的堆叠的顺序
在没有设置z-index的情况下，后面的HTML代码放在上面
