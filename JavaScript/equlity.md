# Equlity

## [==: The Abstract Equlity Comparison Algorithm](https://www.ecma-international.org/ecma-262/5.1/#sec-11.9.3)

如果Type相等，则返回 === 的运算结果。如果type不相等，则执行coerce再比较

![IMAGE](/assets/article_images/2018/==.jpg)

## [===: The Strict Equality Comparison Algorithm](https://www.ecma-international.org/ecma-262/5.1/#sec-11.9.6)

step 1 先判断Type是否相等，如果不相等，直接返回false
step 2 如果Type(x) === Type(y) === Undefined(Undefined type只有undefined这一个值)，返回true

![IMAGE](/assets/article_images/2018/===.jpg)

## [Object.is(): The SameValue Algorithm](https://www.ecma-international.org/ecma-262/5.1/#sec-9.12)

> Object.is does no type conversion and no special handling for `NaN`, `-0`, and `+0` (giving it the same behavior as === except on those special numeric values).
也不做类型转化，除了对三个特殊值处理结果不同，其他都和 === 结果一样

```js
NaN === NaN // false
0 === -0 // true
+0 === -0 // true

Object.is(NaN, NaN) // true
Object.is(0, -0); // false
Object.is(+0, -0); // false

// Polyfill
if (!Object.is) {
  Object.is = function(x, y) {
    // SameValue algorithm
    if (x === y) { // Steps 1-5, 7-10
      // Steps 6.b-6.e: +0 != -0
      return x !== 0 || 1 / x === 1 / y;
      // Infinity === -Infinity , return false
    } else {
      // Step 6.a: NaN == NaN
      // NaN 是唯一一个不具有反射性的值(即不等于自身)
      return x !== x && y !== y;
    }
  };
}
```

![IMAGE](/assets/article_images/2018/sameValue.jpg)