# MDN bind polyfill

[You-Dont-Know-JS this](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch2.md#determining-this)

https://stackoverflow.com/questions/40764184/unable-to-understand-prototype-in-bind-polyfill

## 第1版

```js
function mybind(fn, newThis) {
  return function() {
    // 通过闭包保存了obj1为newThis, new调用的时候也不能修改this, 不符合规范
    fn.apply(newThis, arguments)
  }
}

function test(text1, text2) {
  this.text = text1;
  console.log(`this: ${this}`);
}

var obj1 = {name: "obj1"}
var proxyTest1 = mybind(test, obj1);
proxyTest1("111")
console.log(obj1.text) // 111

// 问题1 调用mybind -> test, 本来new的时候this是创建的新对象，但是mybind把this修改为了obj1, 与this的优先级顺序不符合
var proxyTest2 = new proxyTest1("2222");
console.log(obj1.text) // 2222, 期望还是111
console.log(proxyTest2.text) // undefined, 期望是2222
```

[this的优先级](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch2.md#determining-this):
1. new binding: `var bar = new foo()`
2. explicit binding: `var bar = foo.call(obj2)`
3. implicit binding: `var bar = obj1.foo()`
4. default binding, `undefined`(strict mode) / global object: `var bar = foo()`

new binding的优先级比explicit binding优先级高的好处在于，可以使用bind(this, arg1, arg2)预设一些函数的参数，在使用new的时候再传入其他的参数。叫partial application, 是柯里化currying的一种

```js
function foo(something) {
  this.a = something;
}

var obj1 = {};
var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // 2

var baz = new bar( 3 );
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```

## 第1.1版

问题2. 函数的参数丢了，v1也有这个问题

```JS
Function.prototype.bind = function(context) {
// 一定要返回一个函数
//return this.apply(context,arguments);

  var fn = this;
  return function() {
    // 这时候的arguments只是调用的时候的arguments, bind的arguments并没有保存
    fn.apply(context, arguments);
  };
}

function test(text1, text2) {
  console.log(this.text + ' ' + text1 + ' ' + text2);
}

var proxyTest1 = test.bind({text: 'this'}, 'text1');
var proxyTest2 = test.bind({text: 'this'}, 'text1', 'text2');

// bind传入的arguments不见了
proxyTest1('text2') // this text2 undefined
proxyTest2() // this undefined undefined
```

## 第2版: 解决参数问题

```js
Function.prototype.bind = function(context) {
  var slice = Array.prototype.slice;

  // 这时的arguments是bind时候传入的，copy arguments, 去掉第一个参数(this)
  var aArgs = slice.call(arguments, 1);
  var fn = this; // now this is the original function, 对象的隐式binding
  return function() {
    // 这时的arguments是调用bind返回的函数的时候传入的, 保证bind & 调用()的时候的参数都能传入
    fn.apply(context, aArgs.concat(slice.call(arguments)));
  };
}

function test(text1, text2) {
  console.log(this.text + ' ' + text1 + ' ' + text2);
  console.log(`arguments: ${arguments[2]}`);
}

var proxyTest1 = test.bind({text: 'this'}, 'text1');
var proxyTest2 = test.bind({text: 'this2'}, 'text1', 'text2');

proxyTest1('text2') // this text1 text2 / undefined
proxyTest2('text3') // this text1 text2 / text3
proxyTest2() // this text1 text2
```

## 第3版: 解决this binding优先级的问题

* [this的优先级](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch2.md#determining-this):
1. new binding: `var bar = new foo()`
2. explicit binding: `var bar = foo.call(obj2)`
3. implicit binding: `var bar = obj1.foo()`
4. default binding, `undefined`(strict mode) / global object: `var bar = foo()`

new binding的优先级比explicit binding优先级高的好处在于，可以使用bind(this, arg1, arg2)预设一些函数的参数，在使用new的时候再传入其他的参数。叫partial application, 是柯里化currying的一种

```js
function foo(something) {
  this.a = something;
}

var obj1 = {};
var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // 2

var baz = new bar( 3 );
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```

---

先复习原型链

* [`this instanceof fNOP ? this : oThis` 的作用](https://stackoverflow.com/questions/5774070/mozillas-bind-function-question)

`fToBind.apply(this instanceof fNOP ? this : oThis,
   aArgs.concat(Array.prototype.slice.call(arguments)));`

作用是:
= 允许以构造函数的形式调用the bound function时，不会被绑定到原来的对象。
= 当使用new调用时，函数还是会以unbound version运行, this是new新创建的对象
= new binding优先级高于hard binding
`new bound(123)` 和 `bound(123)`的this不一样

问题在于，如何区分use fBound as a function & use fBound as a constructor ?

```js
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    // 调用.bind的必须是一个函数
    if (typeof this !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }

    var slice = Array.prototype.slice,
      aArgs   = slice.call(arguments, 1),
      fToBind = this,
      // fNOP.prototype == {constructor: f},  新创建的函数都具有一个prototype property, 指向prototype object, 最初的prototype object只有一个constructor属性
      fNOP    = function() {},
      fBound  = function() {
        // instanceof 查看this的原型链中是否有fNOP.prototype, 由于下面的代码设置了原型链，所以所有通过new fBound()调用生成的新对象，都会有
        return fToBind.apply(this instanceof fNOP ? this : oThis,
          aArgs.concat(slice.call(arguments)));
      };

    // fToBind
    if (this.prototype) {
      // Function.prototype doesn't have a prototype property
      // fNOP作为一个intermediate class, 把this.prototype({constructor: f})保存在了fNOP.prototype
      fNOP.prototype = this.prototype;
      // 也是一种继承方法，这两个对象的原型指向了同一个对象，更改同步(live-update)，非特殊情况不建议使用
    }
    // fBound 继承了fNOP, fNOP又继承了fToBind(this)
    fBound.prototype = new fNOP(); // 等同于new fToBind(), 为啥不直接这样用呢？

    return fBound;
  };
}

function test(text1, text2) {
  this.name = text2
  console.log(this.text + ' ' + text1 + ' ' + text2);
  console.log(`arguments: ${arguments[2]}`);
}
var obj1 = {text: 'bindthis'};
var proxyTest1 = test.bind(obj1, 'text1');
var proxyTest4 = proxyTest1('test5'); // proxyTest4 is undefined, obj1.name == 'test5'

var proxyTest2 = new proxyTest1('test3'); // `this` is proxyTest2, the `new` created object
console.log(proxyTest2.name)
```

`this instanceof fNOP`等同于`Object.getPrototypeOf(this) == nop.prototype`
`this instanceof fNOP` == true, 说明是new fBound()构造函数调用, 则使用this, 即new新创建的object
`this instanceof fNOP` == false, 说明是fBound()普通函数调用, 则使用oThis, 即bind传入的this

```js
var fToBind = this;
var fBound = function() {
  return fToBind.apply(this instanceof fToBind ? this: oThis, aArgs.concat(slice.call(arguments)));
}
// 这种方法的问题在于，new fToBind()的时候会执行fToBind里面的逻辑，也就是每次bind会执行一次，肯定不行。所以需要创建一个空函数fNOP来保存fToBind的原型
fBound.prototype = new fToBind();
```

## 遗留问题

```js
var obj1 = {text: 'bindthis'};
var proxyTest1 = test.bind(obj1, 'text1');
proxyTest1.prototype // undefined, 规范里规定bind返回的函数是没有.prototype属性的

var bindinstance = new proxyTest1()
bindinstance instanceof test // true, test的.prototype会代替proxyTest1.prototype
// bindinstance.__proto__ === test.prototype
```

但是使用我们的Polyfill返回的函数是有prototype的
```js
proxyTest1.prototype // test {}
(new proxyTest1()).__proto__.__proto__ = proxyTest1.prototype.__proto__ === (new test()).__proto__ === test.prototype // true
// because
fBound.prototype = new fNOP();
proxyTest1.prototype = new test();

var bindinstance = new proxyTest1()
bindinstance instanceof test // true, 这个是一样的
```
