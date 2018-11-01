## Primitive type: 6种简单数据类型、基本数据类型

Primitive values are data that are stored on the stack. primitive的值存在stack上
Primitive value is stored directly in the location that the variable accesses.
immutable values 不能修改，和C语言不一样，C的string可以修改

* Boolean 
* Number
* String
* Undefined
* Null
* Symbol (ES6)

```js
typeof undefined     === "undefined"; // true
typeof true          === "boolean";   // true
typeof 42            === "number";    // true
typeof "42"          === "string";    // true
typeof { life: 42 }  === "object";    // true

// added in ES6!
typeof Symbol()      === "symbol";    // true

// 判断null, null is the only primitive value that is "falsy" (aka false-like) but that also returns "object" from the typeof check.
var a = null;
(!a && typeof a === "object"); // true

// Object(native or host and does implementent [[Call]])
// 标准规定了，实现了[[Call]]的object 返回function (Callable Object)
typeof new Function === "function"; // true
typeof new Function() === "function"; // true
typeof Object === "function"; // true

typeof [] === "object"; // true
// typeof wrapper object还是object
typeof new Array() === "object"; // true
typeof new String() === "object"; // true
```

typeof相比primitive type就多了个function
![IMAGE](quiver-image-url/203155F2914E7B195BFDE5D5133380CC.jpg =1638x860)

[Why does typeof function return “function”?](https://stackoverflow.com/questions/42467581/why-does-typeof-function-return-function)

```js
var strPrimitive = "I am a string";
typeof strPrimitive;							// "string"
strPrimitive instanceof String;					// false

var strObject = new String( "I am a string" );
typeof strObject; 								// "object"
strObject instanceof String;					// true

// inspect the object sub-type
Object.prototype.toString.call( strObject );	// [object String]
```

## 参考资料：

[MDN Data structures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures)
[MDN typeof](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof)

typeof的返回值可能是什么
https://stackoverflow.com/questions/18808226/why-is-typeof-null-object
http://lucybain.com/blog/2015/js-data-types/

---

## [`undefined` vs "undeclared"](https://github.com/getify/You-Dont-Know-JS/blob/1026e4d56521890a13e73e08f5ffa1b9c70039c8/types%20%26%20grammar/ch1.md#undefined-vs-undeclared)

>An "undefined" variable is one that has been declared in the accessible scope, but at the moment has no other value in it. 
在作用域中声明了，但是还没有赋值
an "undeclared" variable is one that has not been formally declared in the accessible scope.
在作用域中没有声明

```js
var a;

a; // undefined
b; // ReferenceError: b is not defined

typeof a; // "undefined"
// special safety guard in the behavior of typeof
typeof b; // "undefined"

// 利用typeof这种特性，可以用于检查变量是否被定义过
// oops, this would throw an error!
if (DEBUG) {
	console.log( "Debugging is starting" );
}

// this is a safe existence check
if (typeof DEBUG !== "undefined") {
	console.log( "Debugging is starting" );
}

// 也可以通过检查全局对象的方式，但是全局对象不总是window
if (window.DEBUG) {
	// ..
}
```
