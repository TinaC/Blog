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

实例于函数原型之间的引用关系，是在对象实例创建的时候建立的。

如果建立后构造函数原型被修改，实例还是保存原来的原型引用。新创建的实例的原型才会更新

关于如何访问instance property & prototype property, 见[Enumerability and ownership of properties](https://github.com/TinaC/Blog/blob/master/JavaScript/properties_enumerability_ownership.md)

