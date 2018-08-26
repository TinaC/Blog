在JS中，当obejcts, strings等被创建时，值自动被创建，而当它们不被使用了，就会“自动”被释放。释放的过程称为垃圾回收。

自动垃圾回收的重点在于如何判断哪些分配的内存已经不再被需要。

## Reference

For instance, a JavaScript object has a reference to its prototype (implicit reference) and to its properties values (explicit reference).

显式引用 object -> properties values

隐式引用 object -> prototype

## Refrenece-counting garbage collection 引用计数

“an object is not needed anymore” (undecidable 算法无解) -> "an object has no other objects referencing it"

一个对象只要没有引用指向它，则可以视为可回收的

限制： 循环引用 

例子： IE 6 and 7 对DOM对象使用了引用计数垃圾回收，所以循环引用就会造成内存泄露

```js
var div;
window.onload = function() {
  div = document.getElementById('myDivElement');
  div.circularReference = div;
  div.lotsOfData = new Array(10000).join('*');
};
```

# Mark-and-swap algorithm 标记回收

"an object is not needed anymore" -> "an object is unreachable".

object has zero reference => object being unreachable

object being unreachable !=> object has zero reference 不能得出

将global object作为root, 垃圾回收器会从这些roots开始，找到所有被引用的对象，以及被这些对象引用的对象...直到找到所有reachable的对象，回收所有non-reachable的对象。

在2012，所有现代浏览器都换成了标机回收算法，generational/incremental/concurrent/parallel garbage collection这些都是在标机回收算法基础上的改进。


参考资料：
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management