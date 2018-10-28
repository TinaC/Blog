> JS Engine会在解释intepret JS 代码之前对其进行编译compile, 编译阶段中会找到所有的声明，并用合适的作用域将它们关联起来,放入内存中。
允许在声明之前使用变量。

1. hoisting is per-scope. 每个作用域都会进行提升操作
2. Function declarations are hoisted. But function expressions are not. 函数声明会被提升，但是函数表达式却不会被提升。(提升的是var，只是`var foo`提升了以后赋值也是undefined, undefined is not a function)
`foo(); var foo = function bar() {}`
报错是
> TypeError: foo is not a function

3. 在重复声明中，变量var会首先被提升，然后才是函数声明, 函数声明会覆盖var赋值

```js
foo(); // 2 

var foo = 1; // Chrome里会warning: already declared

function foo() {
	console.log(2);
}

foo = function() {
	console.log(3);
};

//等同于
var foo = 1;

function foo() {
	console.log( 2 );
}

foo(); // 2

foo = function() {
	console.log( 2 );
};
```
