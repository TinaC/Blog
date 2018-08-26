
object 存在heap上

* Object
* Array
* Date
* RegExp
* Function
* 基本包装类型
  Boolean
  Number
  String
* 单体内置对象
  Global
  Math

[The most accurate way to check JS object's type?](https://stackoverflow.com/questions/7893776/the-most-accurate-way-to-check-js-objects-type)

Object.prototype.toString.call(JSON) // "[object JSON]"

```js
// Array，Date，Number，String，Boolean，Error甚至Object都是Function的一个实例
Array instanceof Function === true
Array instanceof Object === true
Object instanceof Object === true
Object instanceof Function === true
```

![object_function_prototype](/assets/article_images/2018/object_function_prototype.png)

....

#参考资料： 

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects 