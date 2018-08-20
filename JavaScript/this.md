参考资料:
[You Don't Know JS: this & Object Prototypes](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch1.md)

this的绑定规则

# 1. 默认绑定

非严格模式是window
严格模式是undefined, 是否严格模式取决于函数的声明时

# 2. 隐式绑定



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

