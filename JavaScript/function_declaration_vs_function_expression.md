# 函数表达式 vs. 函数声明

[Medium: Function Declarations vs. Function Expressions ](https://medium.com/@mandeep1012/function-declarations-vs-function-expressions-b43646042052)

> 如果function出现在statement的第一个词，则是一个函数声明，否则就是一个函数表达式。所以IIFE用括号表示是一个函数表达式。
A Function Declaration defines a named function variable without requiring variable assignment. 

* 函数声明 Function Declarations

1. 一定是有函数名的, 类似于var bar;

```js
function bar() {
    return 3;
}
```

2. 在JS解析时进行函数提升, 也类似于var，因此在同一个作用域内，不管函数声明在哪里定义，该函数都可以进行调用。

3. 返回值是undefined

* 函数表达式 Function Expressions 

1. 可以是匿名的

```js
// 变量赋值
var bar = function() {
    return 3;
};
// baz不能在外层scope被访问
var bar = function baz() {
    return 3;
};
```

2. 不提升。函数表达式的值是在JS运行时确定，在interpreter执行到这行代码时。

3. 返回值是这个函数

```js
foo(); // TypeError: foo is not a function. foo已经定义了，只是值这时是undefined
bar(); // ReferenceError: bar is not defined

var foo = function bar() {
	// ...
};

//等同于
foo(); // TypeError
bar(); // ReferenceError

var foo;

foo = function() {
	var bar = ...self...
	// ...
}
```

PS:
函数表达式和函数声明都可以在函数内部用函数名访问自己

```js
function bar() {
  bar() //能访问，只是会死循环
  return 3;
}

var foo = function baz() {
  console.log(foo) // log function
  console.log(baz) // log function, 但是chrome debugger hover上去是baz is not defined. Bug?
}
```

## Function Expression的优势:

* As closures
* As arguments to other functions
* As Immediately Invoked Function Expressions (IIFE)

[Quick Tip: Function Expressions vs Function Declarations](
https://www.sitepoint.com/function-expressions-vs-declarations/)

## 函数提升

> 函数声明提升，函数表达式不提升, var提升

* Similar to the var statement, **function declarations** are hoisted to the top of other code. 
* **Function expressions** aren’t hoisted, which allows them to retain a copy of the local variables from the scope where they were defined.

## Quiz

[Function Declarations vs. Function Expressions 有题目](https://javascriptweblog.wordpress.com/2010/07/06/function-declarations-vs-function-expressions/)


* Quiz 1

```js
function foo(){
    function bar() {
        return 3;
    }
    return bar();
    // 函数声明被提升
    function bar() {
        return 8;
    }
}
alert(foo()); // 8

//等同于
function foo(){
    //define bar once
    function bar() {
        return 3;
    }
    //redefine it, hoist
    function bar() {
        return 8;
    }
    //return its invocation
    return bar(); //8
}
alert(foo());
```

* Quiz 2

```js
//Variable Declarations get hoisted but their Assignment Expressions don’t. 
function foo(){
    var bar = function() {
        return 3;
    };
    return bar();
    // var变量提升，函数表达式不提升。等同于var bar = undefined.
    var bar = function() {
        return 8;
    };
}
alert(foo());

//等同于
//**Simulated processing sequence for Question 2**
function foo(){
    //a declaration for each function expression
    var bar = undefined;
    var bar = undefined;
    //first Function Expression is executed
    bar = function() {
        return 3;
    };
    // Function created by first Function Expression is invoked
    return bar();
    // second Function Expression unreachable
}
alert(foo()); //3
```

* Quiz 3

```js
//foo gets hoisted.
alert(foo());//3
// 函数声明被提升
function foo(){
    var bar = function() {
        return 3;
    };
    return bar();
    var bar = function() {
        return 8;
    };
}

function foo(){
    return bar();
    var bar = function() {
        return 3;
    };
    var bar = function() {
        return 8;
    };
}
alert(foo());
```

---

## Conditonally created function

> 条件创建(if)函数声明在不同浏览器实现中可能有不同表现, 所以如果需要conditional function creation, 使用函数表达式。
[MDN function declaration
](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function#Conditionally_created_functions)

```js
var hoisted = "foo" in this;
console.log(`'foo' name ${hoisted ? "is" : "is not"} hoisted. typeof foo is ${typeof foo}`);
if (false) {
  function foo(){ return 1; }
}

// In Chrome: 
// 'foo' name is hoisted. typeof foo is undefined
// 
// In Firefox:
// 'foo' name is hoisted. typeof foo is undefined
//
// In Edge:
// 'foo' name is not hoisted. typeof foo is undefined
// 
// In Safari:
// 'foo' name is hoisted. typeof foo is function
```