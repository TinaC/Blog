# Prototype

每个函数在创建之后，都具有一个prototype property, 指向prototype object, 最初的prototype object只有一个constructor属性， 又指回到了函数(Ninja)。

用这个函数(Ninja)创建的实例对象(ninja)的原型`[[Prototype]]`等于这个函数的原型对象(Ninja.prototype)。

```js
var Ninja = function ninjaConstructor() {}
Ninja.prototype // {constructor: f ninjaConstructor() {}}

var ninja = new Ninja()
ninja.__proto__ === Ninja.prototype // true
```

A prototype is an object to which the search for a particular property can be delegated to.

可以在原型对象中搜索属性


Every object can have a reference to its prototype, an object to which the search for a particular property can be delegated to, if the object itself doesn’t have that property.

每个对象都有一个到它的原型的引用，当查找属性时，如果对象本身不具有该属性，则会查找原型上是否有该属性

## 函数的普通调用 vs. 函数的构造调用

```js
function Ninja(){}
Ninja.prototype.swingSword = function(){
  return true;
};
const ninja1 = Ninja(); // undefined, 因为函数没有返回值
assert(ninja1 === undefined, "No instance of Ninja created.");

const ninja2 = new Ninja();  // 返回了一个Ninja实例对象
assert(ninja2 &&
 ninja2.swingSword &&
 ninja2.swingSword(),
 "Instance exists and method is callable." );

ninja.__proto__ === Ninja.prototype
ninja2.__proto__.constructor === Ninja
Ninja.prototype.constructor === Ninja
```

## instance property vs. prototype property

如果有同名的property, 优先instance property (废话，是原型链啊)

```js
function Ninja(){
 this.swung = false;
 this.swingSword = function(){
   return !this.swung;
 };
}
Ninja.prototype.swingSword = function(){
  return this.swung;
};
const ninja1 = new Ninja();
const ninja2 = new Ninja();
const ninja3 = new Ninja();  //会创建三个相同的instance property / method, 浪费内存, 这种情况还是用原型属性好

assert(ninja1.swingSword(), "Called the instance method, not the prototype method.");
```

object instance与函数原型之间的引用关系，是在object instance创建的时候建立的。

如果建立后构造函数原型被修改，instance还是保存原来的原型引用。新创建的instance的原型才会更新

关于如何访问instance property & prototype property, 见[Enumerability and ownership of properties](https://github.com/TinaC/Blog/blob/master/JavaScript/properties_enumerability_ownership.md)

## 原型继承

`SubClass.prototype = new SuperClass()`
`Ninja.prototype = new Person()`

because the prototype of the SubClass instance(`ninja1.__proto__`) will be an instance of the SuperClass(`Person`), which has a prototype with all the properties of SuperClass, and which will in turn have a prototype pointing to an instance of its superclass, and on and on.

`ninja.__proto__ === Ninja.prototype === new Person()`

`new Person()`这个实例的原型上有Person的所有属性

`ninja.__proto__.__proto__ === Ninja.prototype.__proto__ === (new Person()).__proto__ === Person.prototype`

`ninja.__proto__.__proto__.__proto__ === (Person.prototype).__proto__ === Object.prototype` 因为Person.prototype是一个Object

![prototype](/assets/article_images/2018/prototype.png)

还需要修复被override的constructor, 否则constructor就是Person了:

```js
function Person(){}
Person.prototype.dance = function(){};

function Ninja(){}
Ninja.prototype = new Person();

Object.defineProperty(Ninja.prototype, "constructor", {
  enumerable: false,
  value: Ninja,
  writable: true
});

var ninja = new Ninja();
```


## [Object.create()](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/create)

[FunFunFunction](https://www.youtube.com/watch?v=CDFN1VatiJA)
Douglas Crockford写了Object.create(), 为了替代js中的new, new是假装类继承

> creates a new object, using an existing object as the prototype of the newly created object.
`Object.create()`会创建一个对象，并把传入的对象设置为新创建的对象的原型

```js
var Person = {
  name: "myname",
  speak: function() {
    console.log(this.name)
  }
}

var ninja = Object.create(Person);
ninja.__proto__ === Person // true
Person.isPrototypeOf(ninja) // true
ninja.name // "myname"
"name" in ninja // true
ninja.hasOwnProperty("name") // false
```

```js
// pre-ES6
// throws away default existing `Bar.prototype`
Bar.prototype = Object.create( Foo.prototype );

// ES6+
// modifies existing `Bar.prototype`
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```

> If you care about performance you should avoid setting the [[Prototype]] of an object. Instead, create a new object with the desired [[Prototype]] using `Object.create()`.
使用Object.create(). 而不是setPrototypeOf().
[`Object.prototype.__proto__`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto) & [Object.setPrototypeOf()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf)
因为性能不好。setPrototypeOf()只适合用来讲解prototype原理...

```js
// 先创建一个对象，再修改它的原型性能很差
function objectCreate(proto) {
  const obj = {}
  Object.setPrototypeOf(obj, proto)
  return obj
}

// 所以直接创建一个具有proto原型的对象
// 简化polyfill
Object.create = function (proto, propertiesObject) {
  function F() {}
  F.prototype = proto;

  // (new F()).__proto__ === F.prototype === proto
  return new F();
};
```

* Object.create(SuperClass) vs. Object.create(SuperClass.prototype)

```js
function Shape() {
  this.x = 0;
  this.y = 0;
  this.speak = function() {
    console.log("this is an instance function")
  }
}
// Shape.speak = ...

Shape.prototype.move = function(x, y) {
  this.x += x;
  this.y += y;
  console.info('this is a prototype function');
};

function Rectangle() {
  Shape.call(this); // call super constructor.
}

function Triangle() {
  Shape.call(this); // call super constructor.
}

// (new F()).__proto__ === F.prototype === Shape.prototype
// Rectangle.prototype === new F()
// rect.__proto__.__proto__ === Rectangle.prototype.__proto__  === (new F()).__proto__ === Shape.prototype
Rectangle.prototype = Object.create(Shape.prototype);

// (new F()).__proto__ === F.prototype === Shape
// Triangle.prototype === new F()
// tri.__proto__.__proto__ === (Triangle.prototype).__proto__ === (new F()).__proto__ === Shape
Triangle.prototype = Object.create(Shape);
// Rectangle.prototype.constructor = Rectangle;

var rect = new Rectangle();
var tri = new Triangle();

rect.__proto__.__proto__  ===  Shape.prototype
tri.__proto__.__proto__ === Shape

rect.constructor // f Shape
tri.constructor // f Function

rect.speak // f speak
tri.speak // f speak

rect.move // f move
tri.move // undefined, 无法访问Shape的原型方法
```

![object_create](/assets/article_images/2018/object_create.png)
