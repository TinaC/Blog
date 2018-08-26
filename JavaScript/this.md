# this的绑定规则

* 1. 默认绑定

非严格模式是window

严格模式是undefined

是否严格模式取决于函数的声明时

* 2. 隐式绑定

使用对象上下文去引用函数时，可以说在这个函数被调用时，这个对象包含了这个函数引用，这时隐式绑定规则就认为这个对象应该作为该方法的`this` binding. 

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

obj.foo(); // 2
```

隐式绑定经常发生绑定丢失：

1. 把函数当参数传递时丢失绑定。因为是一个隐式的引用赋值。
2. 函数回调丢失绑定: setTimeout, ajax callback

* 3. 显示绑定

* 3.1 hard binding: call() apply() bind()

> function.call(thisArg, arg1, arg2, ...)

> function.apply(thisArg, [argsArray])

> function.bind(thisArg[, arg1[, arg2[, ...]]])

https://stackoverflow.com/questions/15455009/javascript-call-apply-vs-bind

.bind() 在事件回调中常用，返回一个带有context的去调用原函数的函数。

.call(this, ) & .apply() 是立即调用。

```js
// bind的简单实现，完整版参考 MDN polyfill: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind#Polyfill
Function.prototype.bind = function(ctx) {
    var fn = this;
    return function() {
        fn.apply(ctx, arguments);
    };
};
```

* 3.2 API Call "Contexts"

提供了可选的this参数，原理和hard binding一样

`forEach(function, this)`

* 4. `new` binding

对函数的构造调用 construction calls of functions

```js
function foo(a) {
	this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a ); // 2
```

# this绑定规则之间的优先级

> `new` Binding > Explicit Binding > Implicit Binding > Deafault Binding(global object)

```js
// new binding vs implicit binding
function foo(a) {
	this.a = a;
}

var obj1 = {
	foo: foo
};

var obj2 = {};

obj1.foo( 2 );
console.log( obj1.a ); // 2

obj1.foo.call( obj2, 3 ); // hard binding
console.log( obj2.a ); // 3, hard binding wins 

var bar = new obj1.foo( 4 );
console.log( obj1.a ); // 2
console.log( bar.a ); // 4, new binding wins
```


```js
// new binding vs hard binding
var obj1 = {}

function foo(a) {
	this.a = a;
}

var bar = foo.bind(obj1); // hard binding
bar(2);
console.log(obj1.a) // 2

var baz = new bar(3); // new binding
console.log(baz.a) // 3 : new binding wins
```

![this_binding](/assets/article_images/2018/this_binding.png) 

# jQuery $.proxy()

.bind()在Function.prototype

$.proxy() 是一个函数

```js
// 使用
$(el).change(jQuery.proxy(function(event) {
    this.name = event.target.value;
}, this));

// 源代码
var isFunction = function isFunction( obj ) {
    // Support: Chrome <=57, Firefox <=52
    // In some browsers, typeof returns "function" for HTML <object> elements
    // (i.e., `typeof document.createElement( "object" ) === "function"`).
    // We don't want to classify *any* DOM node as a function.
    return typeof obj === "function" && typeof obj.nodeType !== "number";
};

jQuery.proxy = function( fn, context ) {
	var tmp, args, proxy;

  // 如果第二个参数是name, 说明传入的参数是(context,name) => (fn, context)
  // 就执行fn = context[name], context = context
	if ( typeof context === "string" ) {
		tmp = fn[ context ];
		context = fn;
		fn = tmp;
	}

	// Quick check to determine if target is callable, in the spec
	// this throws a TypeError, but we will just return undefined.
	if ( !isFunction( fn ) ) {
		return undefined;
	}

	// Simulated bind
	// jQuery.proxy( obj, "test", "a", "b", "c") );
	// args扔掉前两个，变成["a", "b", "c"]
	args = slice.call( arguments, 2 );
	proxy = function() {
	  // 如果没有context，就用default binding(window / global)
	  // slice是jQuery的简写， [].slice
		return fn.apply( context || this, args.concat( slice.call( arguments ) ) );
	};

	// Set the guid of unique handler to the same of original handler, so it can be removed
	proxy.guid = fn.guid = fn.guid || jQuery.guid++;

	return proxy;
};
```

[Underscore bind vs jQuery.proxy vs Native bind](https://stackoverflow.com/a/22860661/5238583)

> There is a slight difference between bind and proxy:
> Function.prototype.bind always returns a new function pointer. 总是返回一个新的函数指针
> jQuery.proxy only returns a new function if a proxy of the same args has not already been created.  只有同样参数的proxy没有被创建过，才会创建新的函数

```js
$(elm).on('click', doStuff.bind(thing)); //adds event handler
$(elm).off('click', doStuff.bind(thing)); //does not remove event handler as 2nd call of doStuff.bind(thing) always returns a new/different function 这种方法不能移除event handler,因为已经不是同一个函数

$(elm).on('click', $.proxy(doStuff, thing)); //adds handler
$(elm).off('click', $.proxy(doStuff, thing));//DOES remove handler, as a second call to $.proxy(doStuff, thing) is smart enough to know about similar use-cases

//Likewise, if you just passed 'thing.doStuff()' into the $.off() method, it would also work
```

# Underscore _.bind()

https://github.com/jashkenas/underscore/blob/1bfc9f1fb811060c745e7025ec55e35f87c1d43b/underscore.js#L767

```js
// Determines whether to execute a function as a constructor
// or a normal function with the provided arguments.
var executeBound = function(sourceFunc, boundFunc, context, callingContext, args) {
  if (!(callingContext instanceof boundFunc)) return sourceFunc.apply(context, args);
  var self = baseCreate(sourceFunc.prototype);
  var result = sourceFunc.apply(self, args);
  if (_.isObject(result)) return result;
  return self;
};

// Create a function bound to a given object (assigning `this`, and arguments,
// optionally). Delegates to **ECMAScript 5**'s native `Function.bind` if
// available.
_.bind = restArgs(function(func, context, args) {
  if (!_.isFunction(func)) throw new TypeError('Bind must be called on a function');
  var bound = restArgs(function(callArgs) {
    return executeBound(func, bound, context, this, args.concat(callArgs));
  });
  return bound;
});

// Optimize `isFunction` if appropriate. Work around some typeof bugs in old v8,
// IE 11 (#1621), Safari 8 (#1929), and PhantomJS (#2236).
var nodelist = root.document && root.document.childNodes;
if (typeof /./ != 'function' && typeof Int8Array != 'object' && typeof nodelist != 'function') {
  _.isFunction = function(obj) {
    return typeof obj == 'function' || false;
  };
}
```

参考资料:
[You Don't Know JS: this & Object Prototypes](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch1.md)
