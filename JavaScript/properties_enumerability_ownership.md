# Enumerability and ownership of properties

for..in

in

Object.prototype.propertyIsEnumerable()

Object.prototype.hasOwnProperty()


[Object.keys(), ES 5.1](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)

[Object.getOwnPropertyNames(), ES 5.1](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyNames)

[Object.getOwnPropertySymbols(), ES 6](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertySymbols)

[Object.getOwnPropertyDescriptors(), ES2017](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptors)


* 访问non-enumeration

in, obj.hasOwnProperty, Object.getOwnPropertyNames, Object.getOwnPropertyDescriptors

* 不访问non-enumeration

for...in, obj.propertyIsEnumerable, Object.keys,

* 访问原型链

in, for...in,

* 不访问原型链

obj.hasOwnProperty, obj.propertyIsEnumerable, Object.keys, Object.getOwnPropertyNames, Object.getOwnPropertyDescriptors

* 访问Symbol

in, obj.hasOwnProperty, obj.propertyIsEnumerable, Object.getOwnPropertyDescriptors(ES2017)

* 不访问Symbol

for..in, Object.keys(ES 5.1), Object.getOwnPropertyNames(ES 5.1)

```js
function Person() {}
Person.prototype.protoProp = "proto-prop-value";
var myObject = new Person()
myObject.normal = 2;

var mySymbol = Symbol.for("mySymbol");
myObject[mySymbol] = "my-symbol-value";

Object.defineProperty(
  myObject,
  "nonEnumerable",
  // make `b` non-enumerable
  { enumerable: false, value: "non-enumerable-value" }
);

"normal" in myObject // true
mySymbol in myObject // true
"protoProp" in myObject // true
"nonEnumerable" in myObject // true

// 原型链上的属性不访问
myObject.hasOwnProperty("normal") // true
myObject.hasOwnProperty("nonEnumerable") // true
myObject.hasOwnProperty(mySymbol) // true
myObject.hasOwnProperty("protoProp") // false

// nonEnumerable & 原型链上的属性不访问
myObject.propertyIsEnumerable("normal") // true
myObject.propertyIsEnumerable("nonEnumerable") // false
myObject.propertyIsEnumerable(mySymbol) // true
myObject.propertyIsEnumerable("protoProp") // false

// enumerable, non-Symbol, own properties
Object.keys(myObject) // ["normal"]

// non-Symbol, own properties
Object.getOwnPropertyNames(myObject) // ["normal", "nonEnumerable"]

// own properties
Object.getOwnPropertyDescriptors(myObject) // {"normal": {...}, "nonEnumerable": {...},"Symbol(mySymbol)": {...}}

// only include Symbol
Object.getOwnPropertySymbols(myObject) // [Symbal(mySymbol)]

for (var prop in myObject) {
   console.log( prop, myObject[prop] );
}
// normal, protoProp
```

## Reference
[Enumerability and ownership of properties, 有table总结
](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Enumerability_and_ownership_of_properties)
