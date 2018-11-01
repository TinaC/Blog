# IIFE

* Explain why the following doesn't work as an IIFE: `function foo(){ }();`

IIFE需要用()包裹，其他符号也行，只是不能用function开头，function开头是函数声明，IIFE需要函数表达式。

```js
(function () {
    // logic here
})();

(function IIFE(){ })()
```

## 为啥需要函数表达式? 

> The key thing about JavaScript expressions is that they return values. In both cases above the return value of the expression is the function.
函数表达式会返回值，在上面的两个例子中，函数表达式返回了这个函数
这样才能才能被后一个()执行

```js
(function () {
}) // return a function

function declaration(){ } // return undefined
```

## IIFE 的作用

IIFE’s are used to help prevent your functions and variables from affecting the global scope. All the properties within are scoped to the anonymous function.
IIFE中的函数和变量都在IIFE这个匿名(不一定吧)函数的作用域中，不会影响全局作用域

The primary reason to use an IIFE is to obtain data privacy. Because JavaScript's `var` scopes variables to their containing function, any variables declared within the IIFE cannot be accessed by the outside world.
IIFE主要作用是保证数据隐私。由于var定义的变量作用域是存在于包裹它的函数, 所以IIFE内定义的变量不能被外部环境访问。

```js
function myImmediateFunction () {
  var foo = "bar";

  // Outputs: "bar"
  console.log(foo);
}

myImmediateFunction();

// ReferenceError: foo is not defined
console.log(foo);
```

和上面这种方法效果一样，但是这种方法缺点在于：
1. 全局空间增加了myImmediateFunction这个变量，增加了命名冲突的可能性
2. 代码可读性不强 the intentions of this code aren't as self-documenting as an IIFE. 
3. 还能被调用多次 because it is named and isn't self-documenting it might accidentally be invoked more than once.

现在有let, 还有模块了，IIFE可以退休了。

## Reference 

[An Introduction to IIFEs - Immediately Invoked Function Expressions](http://adripofjavascript.com/blog/drips/an-introduction-to-iffes-immediately-invoked-function-expressions.html)

[You Don't Know JS - IIFE](https://github.com/getify/You-Dont-Know-JS/blob/1026e4d56521890a13e73e08f5ffa1b9c70039c8/up%20%26%20going/ch2.md#immediately-invoked-function-expressions-iifes)
