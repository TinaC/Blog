参考资料:

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures

https://codeburst.io/js-scope-static-dynamic-and-runtime-augmented-5abfee6223fe

https://css-tricks.com/javascript-scope-closures/

https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20%26%20closures/ch1.md

[从JS垃圾回收机制和词源来透视闭包](https://mp.weixin.qq.com/s/485GgpEt2c7uS-mY1cbA3w)

闭包是函数和声明该函数时的词法环境的结合
词法环境包含了这个闭包创建时所能访问的所有局部变量。
closure = function + it's lexical environment(local variables)

所以要理解闭包，首先要理解词法作用域； 要理解词法作用域，首先要理解作用域。

# 作用域 (Scope)

作用域是根据名称查找变量值的一套规则，[用途](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20%26%20closures/ch1.md)：
1. 隐藏内部实现
2. 规避冲突
是一种函数封装方式(?)。

[作用域可以粗略地分为三类](https://codeburst.io/js-scope-static-dynamic-and-runtime-augmented-5abfee6223fe)：
1. 静态作用域 (Static scope) / 词法作用域 (Lexical scope)
利用闭包实现，捕获词法环境中局部变量
author time，即通过源代码就能判断scope，取决于函数何处声明，与作用域链(lexical scope chain)相关

2. 动态作用域 (Dynamic scope)
利用调用栈来动态定义作用域，参考JS中的this值获取(ES6中的箭头函数就用的静态作用域)
run time，取决于函数被何处调用，与调用栈(call stack)相关

3. 运行时变化(?)作用域 (Runtime-augmented scope)
with, eval 影响代码优化(因为编译器无法预知)，尽量少用

# 词法作用域 (Lexical Scope)

作用域链的查找规则：
1. 引擎从当前的执行作用域开始查找变量
2. 如果找不到，就向上一级继续查找
3. 直到抵达最外层的全局作用域。
4. 如果没找到，严格模式下报错(ReferenceError: x is not defined), 非严格模式下创建一个变量？

MDN中lexical scope的例子, 函数嵌套组成了作用域链：

```js
function init() {
  var name = 'Mozilla'; // name is a local variable created by init
  function displayName() { // displayName() is the inner function, a closure
    alert(name); // use variable declared in the parent function
  }
  displayName();
}
init();
```

# 闭包 (Closure)

当函数可以记住并访问所在的词法作用域时，就产生了闭包，即使函数是在当前词法作用域之外执行。
无论使用何种手段将内部函数传递到所在的词法作用域外，它都会持有对原始定义作用域的引用，无论在何处执行这个函数都会使用闭包。
严格来说，要在它本身的词法作用域外执行的函数才叫闭包。否则就是用的词法作用域，而不是闭包。

```js
function makeFunc() {
  var name = 'Mozilla';
  function displayName() {
    alert(name);
  }
  return displayName; // 将内部函数传递到了所在的词法作用域外
}

var myFunc = makeFunc();
myFunc(); // 仍然包含对原始词法作用域的引用
```

之前我将作用域链和调用栈混淆了，看着两个例子：

* 作用域链看函数声明，author time
global scope -> foo
global scope -> bar

* 调用栈看函数调用，run time
global -> bar -> foo



```js
function foo() {
	console.log( a ); // 2
}

function bar() {
	var a = 3;
	foo();
}

var a = 2;

bar();
```

```js
function bar() {
  var a = 3;
  function foo() {
    console.log( a ); // 3
  }
  foo();
}

var a = 2;

bar();
```

由上可知，闭包保存了整个声明该函数时的词法环境，所以如果使用没有必要的函数嵌套，创建多余的闭包，会造成性能问题。
