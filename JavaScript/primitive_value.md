
# Primitive type

6种基本数据类型，值都存在栈stack上，值不能修改(和C语言不一样，C的string可以修改)

* boolean
* number
* string
* undefined
* null
* symbol (ES6)

# typeof

![typeof](/assets/article_images/2018/typeof.jpg) 

```js
typeof true // "boolean"
typeof 1 // "number"
typeof "1" // "string"
typeof undefined //"undefined"
typeof null // "object", JS引擎的Bug，type tag是0的就是object, null是0x00, 就被误判成object了
typeof [] // "object", array is a object
typeof Symbol() // "symbal"
typeof Object // "function"

// typeof wrapper object还是返回object
var strPrimitive = "I am a string";
typeof strPrimitive;							// "string"
strPrimitive instanceof String;					// false

var strObject = new String( "I am a string" );
typeof strObject; 								// "object"
strObject instanceof String;					// true

// inspect the object sub-type
Object.prototype.toString.call( strObject );	// [object String]
```


# 参考资料

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof

[Why is typeof null "object"?](https://stackoverflow.com/questions/18808226/why-is-typeof-null-object) 因为bug

[Why does typeof function return “function”?](https://stackoverflow.com/questions/42467581/why-does-typeof-function-return-function) 标准是这样定义的

http://lucybain.com/blog/2015/js-data-types/